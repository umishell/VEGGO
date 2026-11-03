# DB-03 — Telemetria, Estatísticas e Painel do Gerente

**Referenciado por:**  
→ [FLUX-HAPPY-18-painel](../fluxos/happypaths/FLUX-HAPPY-18-painel.md) — Painel de pedidos do dia em tempo real  
→ [FLUX-HAPPY-19-stats](../fluxos/happypaths/FLUX-HAPPY-19-stats.md) — Estatísticas e telemetria da operação

*[← Índice docTecnica](README.md)*

---

## 1. Princípio

Não há tabela de agregação separada no MVP. Todas as métricas são derivadas das tabelas operacionais (`orders`, `order_items`, `order_events`, `users`, `delivery_persons`) via queries SQL. O PostgreSQL é a única fonte.

---

## 1.1 Dia operacional (painel do dia e “hoje”)

**Definição única:** pedidos cujo `created_at`, convertido para o fuso `stores.timezone`, cai na **data civil de hoje** nesse mesmo fuso.

| Situação | Entra no painel / resumo “hoje”? |
|----------|----------------------------------|
| Pedido antes de uma pausa `FECHADA_PROGRAMADA` e depois da reabertura **no mesmo dia civil** | Sim — mesma lista |
| Loja reabre após pausa programada | A lista **não** é apagada (não é “desde a última abertura”) |
| Meia-noite no fuso da loja | Novo dia operacional; painel e totais “hoje” passam a outra data |
| Pedido criado ontem ainda em preparo hoje | Continua visível no painel **de ontem**; no painel **de hoje** só entram pedidos com `created_at` na data de hoje |

Parâmetro REST `date=today` significa exatamente essa data civil no `timezone` da loja. Histórico por período (`FLUX-HAPPY-19-stats`) agrupa por `DATE(created_at AT TIME ZONE timezone)`, não por turno de abertura.

Expressão reutilizável ( `:storeTz` = `stores.timezone` ):

```sql
(created_at AT TIME ZONE :storeTz)::date = (CURRENT_TIMESTAMP AT TIME ZONE :storeTz)::date
```

---

## 2. Queries por tela

### Tela 1 — Resumo do dia (atualização em tempo real)

Os totais do dia são recomputados a cada evento de estado recebido pelo servidor, ou solicitados por REST no carregamento. Não são armazenados em cache no MVP. Usam o **dia operacional** (§ 1.1).

```sql
-- Pedidos do dia operacional por estado
SELECT status, COUNT(*) AS qty
  FROM orders
 WHERE store_id = :storeId
   AND (created_at AT TIME ZONE :storeTz)::date
       = (CURRENT_TIMESTAMP AT TIME ZONE :storeTz)::date
 GROUP BY status;

-- Faturamento do dia operacional (apenas entregues)
SELECT COALESCE(SUM(total_amount), 0) AS revenue
  FROM orders
 WHERE store_id = :storeId
   AND status = 'ENTREGUE'
   AND (created_at AT TIME ZONE :storeTz)::date
       = (CURRENT_TIMESTAMP AT TIME ZONE :storeTz)::date;
```

---

### Tela 2 — Histórico por período

```sql
-- Volume e faturamento agrupado por dia (com fuso da loja — alinhado ao §1.1)
SELECT (created_at AT TIME ZONE :storeTz)::date AS day,
       COUNT(*) AS total_orders,
       COUNT(*) FILTER (WHERE status = 'ENTREGUE') AS delivered,
       COUNT(*) FILTER (WHERE status = 'CANCELADO') AS cancelled,
       COALESCE(SUM(total_amount) FILTER (WHERE status = 'ENTREGUE'), 0) AS revenue,
       ROUND(
         100.0 * COUNT(*) FILTER (WHERE status = 'CANCELADO') / NULLIF(COUNT(*), 0),
         1
       ) AS cancellation_rate_pct
  FROM orders
 WHERE store_id = :storeId
   AND (created_at AT TIME ZONE :storeTz)::date BETWEEN :startDate AND :endDate
 GROUP BY (created_at AT TIME ZONE :storeTz)::date
 ORDER BY day;
-- :startDate e :endDate são datas civis (DATE) já convertidas pelo chamador para o fuso da loja
-- :storeTz = stores.timezone (ex: 'America/Sao_Paulo')
```

> ⚠️ **Correção de fuso aplicada:** a versão anterior usava `DATE(created_at)` sem fuso, o que podia classificar pedidos no dia errado para lojas em fuso UTC-3. Agora todos os agrupamentos por dia usam `(created_at AT TIME ZONE :storeTz)::date`, igual ao §1.1.

```sql
-- Novos clientes no período (primeira compra dentro do intervalo)
SELECT COUNT(*) AS new_clients
  FROM (
    SELECT client_id
      FROM orders
     WHERE store_id = :storeId
     GROUP BY client_id
    HAVING MIN((created_at AT TIME ZONE :storeTz)::date) BETWEEN :startDate AND :endDate
  ) first_orders;

-- Clientes recorrentes (mais de 1 pedido no período)
SELECT COUNT(*) AS returning_clients
  FROM (
    SELECT client_id
      FROM orders
     WHERE store_id = :storeId
       AND (created_at AT TIME ZONE :storeTz)::date BETWEEN :startDate AND :endDate
     GROUP BY client_id
    HAVING COUNT(*) > 1
  ) recurring;
```

---

### Tela 3 — Itens mais pedidos

```sql
-- Ranking de itens por quantidade
SELECT oi.product_name,
       COUNT(*) AS times_ordered,
       SUM(oi.quantity) AS total_units,
       SUM(oi.unit_price * oi.quantity) AS revenue
  FROM order_items oi
  JOIN orders o ON o.id = oi.order_id
 WHERE o.store_id = :storeId
   AND o.status = 'ENTREGUE'
   AND o.created_at BETWEEN :start AND :end
 GROUP BY oi.product_name
 ORDER BY total_units DESC;
```

---

### Tela 4 — Clientes

```sql
-- Clientes ativos no período
SELECT COUNT(DISTINCT client_id) AS active_clients
  FROM orders
 WHERE store_id = :storeId
   AND created_at BETWEEN :start AND :end;

-- Top clientes por volume
SELECT client_id,
       u.name,
       COUNT(*) AS orders_count,
       SUM(total_amount) FILTER (WHERE status = 'ENTREGUE') AS total_spent
  FROM orders o
  JOIN users u ON u.id = o.client_id
 WHERE o.store_id = :storeId
   AND o.created_at BETWEEN :start AND :end
 GROUP BY client_id, u.name
 ORDER BY orders_count DESC
 LIMIT 20;
```

---

### Tela 5 — Operação

```sql
-- Tempo médio cozinha: preferir orders.prep_duration_seconds (ORD-01 § 5.5)
SELECT AVG(prep_duration_seconds) / 60.0 AS avg_kitchen_minutes
  FROM orders o
 WHERE o.store_id = :storeId
   AND o.prep_duration_seconds IS NOT NULL
   AND (o.prep_completed_at AT TIME ZONE :storeTz)::date BETWEEN :startDate AND :endDate;

-- Fallback legado via order_events (se prep_duration_seconds nulo em pedidos antigos):
-- AVG(EXTRACT(EPOCH FROM (e_pronto.occurred_at - e_preparo.occurred_at)) / 60) ...

-- Tempo médio de entrega: SAIU_PARA_ENTREGA → ENTREGUE
SELECT AVG(EXTRACT(EPOCH FROM (e_entregue.occurred_at - e_saiu.occurred_at)) / 60) AS avg_delivery_minutes
  FROM order_events e_saiu
  JOIN order_events e_entregue ON e_entregue.order_id = e_saiu.order_id
  JOIN orders o ON o.id = e_saiu.order_id
 WHERE e_saiu.new_status = 'SAIU_PARA_ENTREGA'
   AND e_entregue.new_status = 'ENTREGUE'
   AND o.store_id = :storeId
   AND (e_saiu.occurred_at AT TIME ZONE :storeTz)::date BETWEEN :startDate AND :endDate;

-- Pedidos por motoboy no período
SELECT e_saiu.changed_by_id AS motoboy_id,
       u.name AS motoboy_name,
       COUNT(*) AS deliveries_count,
       COUNT(*) FILTER (WHERE EXISTS (
         SELECT 1 FROM order_events ie
          WHERE ie.order_id = e_saiu.order_id
            AND ie.reason LIKE 'IMPREVISTO%'
       )) AS incidents_count
  FROM order_events e_saiu
  JOIN users u ON u.id = e_saiu.changed_by_id
  JOIN orders o ON o.id = e_saiu.order_id
 WHERE e_saiu.new_status = 'SAIU_PARA_ENTREGA'
   AND o.store_id = :storeId
   AND (e_saiu.occurred_at AT TIME ZONE :storeTz)::date BETWEEN :startDate AND :endDate
 GROUP BY e_saiu.changed_by_id, u.name
 ORDER BY deliveries_count DESC;

-- Tempo médio aguardando motoboy: PRONTO → SAIU_PARA_ENTREGA
SELECT AVG(EXTRACT(EPOCH FROM (e_saiu.occurred_at - e_pronto.occurred_at)) / 60) AS avg_wait_motoboy_minutes
  FROM order_events e_pronto
  JOIN order_events e_saiu ON e_saiu.order_id = e_pronto.order_id
  JOIN orders o ON o.id = e_pronto.order_id
 WHERE e_pronto.new_status = 'PRONTO'
   AND e_saiu.new_status = 'SAIU_PARA_ENTREGA'
   AND o.store_id = :storeId
   AND (e_pronto.occurred_at AT TIME ZONE :storeTz)::date BETWEEN :startDate AND :endDate;
```

---

## 3. Painel do dia em tempo real — `FLUX-HAPPY-18-painel`

O gerente recebe os mesmos eventos WebSocket da cozinha e do motoboy, pelo canal `kitchen:{storeId}`. A diferença é só no frontend:

| KDS (cozinha) | Painel do gerente |
|---------------|-------------------|
| Lista `RECEBIDO` + `EM_PREPARO`; some ao `PRONTO` | Todos os estados do dia permanecem na tabela |
| Seleção + **Preparar** / **Concluir preparo** | Somente leitura + exceções (cancelar, imprevisto) |
| Timer por pedido em preparo | Coluna **Tempo preparo** (`prep_duration_seconds`) |
| — | **Rodapé:** métricas § 6.1 (tempo médio, volume, etc.) |

A listagem inicial traz **todos os pedidos do dia operacional atual** (§ 1.1), ordenados por `created_at` desc.

```
GET /orders?storeId={id}&date=today&page=1&size=50
```

`date=today` resolve para a data civil em `stores.timezone`, com o mesmo predicado SQL da § 1.1.

---

## 6. Métricas embutidas (rodapé das telas gerente)

Queries sob demanda; ver também [INF-04 § 5](INF-04-ui-operacao-cliente.md).

### 6.1 Pedidos — painel do dia (`GET /store/stats/orders-summary`)

Parâmetros: `period=today|7d|30d|month` (default `today`), fuso `stores.timezone`.

| Métrica | Definição SQL (conceito) |
|---------|---------------------------|
| Pedidos no período | `COUNT(*)` em `orders` por data civil |
| Média pedidos/dia | Total no intervalo ÷ dias civis no intervalo |
| Pedidos hoje / semana / mês | Contagens com `(created_at AT TIME ZONE tz)::date` |
| Tempo médio preparo | `AVG(prep_duration_seconds)` onde not null, mesmo recorte de datas |
| p95 tempo preparo | Percentil 95 de `prep_duration_seconds` |
| Tempo médio fila antes preparo | `AVG(prep_started_at - created_at)` para pedidos que chegaram a preparo |
| Cancelamentos | `COUNT` status `CANCELADO` / total |

```sql
-- Tempo médio preparo (segundos) — preferir coluna materializada
SELECT ROUND(AVG(prep_duration_seconds)) AS avg_prep_sec,
       PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY prep_duration_seconds) AS p95_prep_sec
  FROM orders
 WHERE store_id = :storeId
   AND prep_duration_seconds IS NOT NULL
   AND (prep_completed_at AT TIME ZONE :storeTz)::date
       BETWEEN :startDate AND :endDate;

-- Volume por dia civil
SELECT (created_at AT TIME ZONE :storeTz)::date AS day,
       COUNT(*) AS orders_count
  FROM orders
 WHERE store_id = :storeId
   AND (created_at AT TIME ZONE :storeTz)::date BETWEEN :startDate AND :endDate
 GROUP BY 1 ORDER BY 1;
```

### 6.2 Cardápio — rodapé (`GET /store/stats/catalog`)

| Métrica | Uso |
|---------|-----|
| Top N itens por unidades | `order_items` + `ENTREGUE` no período |
| Top N por receita | Mesma base, `SUM(line_total_cents)` |
| Itens que bateram limite diário | Join `product_daily_caps` onde `units_sold >= effective_limit` |
| Mix por categoria | Agrupa por `product_categories.display_name` |

Reutiliza e estende § 2 Tela 3. Período default: **7 dias** no rodapé do cardápio; gerente pode alternar 7d / 30d.

### 6.3 Índice adicional

```sql
CREATE INDEX idx_orders_prep_completed ON orders(store_id, prep_completed_at)
  WHERE prep_duration_seconds IS NOT NULL;
```

---

## 4. Índices recomendados

```sql
CREATE INDEX idx_orders_store_date    ON orders(store_id, created_at);
CREATE INDEX idx_orders_status        ON orders(status);
CREATE INDEX idx_order_items_product  ON order_items(product_name);
CREATE INDEX idx_order_events_order   ON order_events(order_id, occurred_at);
```

---

## 5. Escopo do MVP

No MVP, as queries rodam sob demanda. Sem views materializadas, sem job de pré-agregação, sem cache de resultado.

Se o volume crescer fora do MVP: views materializadas refreshadas por job horário ou diário resolvem a maioria dos casos.

---

*[← Índice docTecnica](README.md)*
