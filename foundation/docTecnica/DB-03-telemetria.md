# DB-03 — Telemetria, Estatísticas e Painel do Gerente

**Referenciado por:**  
→ [FLUX-HAPPY-18-painel](../fluxos/happypaths/FLUX-HAPPY-18-painel.md) — Painel de pedidos do dia em tempo real  
→ [FLUX-HAPPY-19-stats](../fluxos/happypaths/FLUX-HAPPY-19-stats.md) — Estatísticas e telemetria da operação

*[← Índice docTecnica](README.md)*

---

## 1. Princípio

Não há tabela de agregação separada no MVP. Todas as métricas são derivadas das tabelas operacionais (`orders`, `order_items`, `order_events`, `users`, `delivery_persons`) via queries SQL. O PostgreSQL é a única fonte.

---

## 2. Queries por tela

### Tela 1 — Resumo do dia (atualização em tempo real)

Os totais do dia são recomputados a cada evento de estado recebido pelo servidor, ou solicitados por REST no carregamento. Não são armazenados em cache no MVP.

```sql
-- Pedidos do dia por estado
SELECT status, COUNT(*) AS qty
  FROM orders
 WHERE store_id = :storeId
   AND created_at::date = CURRENT_DATE
 GROUP BY status;

-- Faturamento do dia (apenas entregues)
SELECT COALESCE(SUM(total_amount), 0) AS revenue
  FROM orders
 WHERE store_id = :storeId
   AND status = 'ENTREGUE'
   AND created_at::date = CURRENT_DATE;
```

---

### Tela 2 — Histórico por período

```sql
-- Volume e faturamento agrupado por dia
SELECT DATE(created_at) AS day,
       COUNT(*) AS total_orders,
       COUNT(*) FILTER (WHERE status = 'ENTREGUE') AS delivered,
       COUNT(*) FILTER (WHERE status = 'CANCELADO') AS cancelled,
       SUM(total_amount) FILTER (WHERE status = 'ENTREGUE') AS revenue
  FROM orders
 WHERE store_id = :storeId
   AND created_at BETWEEN :start AND :end
 GROUP BY day
 ORDER BY day;
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
-- Tempo médio cozinha: RECEBIDO → PRONTO
SELECT AVG(EXTRACT(EPOCH FROM (e_pronto.occurred_at - e_recebido.occurred_at)) / 60) AS avg_kitchen_minutes
  FROM order_events e_recebido
  JOIN order_events e_pronto ON e_pronto.order_id = e_recebido.order_id
 WHERE e_recebido.new_status = 'RECEBIDO'
   AND e_pronto.new_status = 'PRONTO'
   AND e_recebido.occurred_at::date = CURRENT_DATE;
```

---

## 3. Painel do dia em tempo real — `FLUX-HAPPY-18-painel`

O gerente recebe os mesmos eventos WebSocket da cozinha e do motoboy, pelo canal `kitchen:{storeId}`. A diferença é só no frontend:

| KDS (cozinha) | Painel do gerente |
|---------------|-------------------|
| Mostra só `RECEBIDO` | Mostra todos os estados |
| Remove o card ao aceitar | Mantém o card, muda cor |
| Sem filtro | Filtros de estado, motoboy, valor, período |

A listagem inicial do dia vem de:
```
GET /orders?storeId={id}&date=today&page=1&size=50
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
