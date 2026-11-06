# INF-06 — Telas do gerente (`web-operacao`)

**Referenciado por:** [ORD-01 § 5.7](ORD-01-estados-pedido.md) · [DB-04 § 2.2](DB-04-catalogo-checkout-cliente.md) · [DB-03 § 6.1](DB-03-telemetria.md) · [FLUX-HAPPY-18](../fluxos/happypaths/FLUX-HAPPY-18-painel.md)

*[← Índice docTecnica](README.md)*

Perfil **GERENTE_LOJA** (JWT). Objetivo: **CRUD de cardápio**, **painel operacional do dia**, **estatísticas**, ações excepcionais no pedido (cancelar, imprevisto em rota). Dados em tempo real: canal `kitchen:{storeId}` ([INF-01](INF-01-tempo-real.md)).

---

## 1. Mapa de telas (MVP)

| Área / rota (conceitual) | Função |
|--------------------------|--------|
| **Painel → Pedidos do dia** | Lista todos os pedidos do **dia operacional** com estado, timers do segmento ativo e durações já fechadas; detalhe do pedido; cancelamento e imprevisto |
| **Cardápio** | CRUD e disponibilidade de produtos ([DB-02 § 1](DB-02-disponibilidade-itens.md)) |
| **Estatísticas** | Visão por período (histórico, gráficos) — [DB-03 § 2](DB-03-telemetria.md) · [FLUX-HAPPY-19](../fluxos/happypaths/FLUX-HAPPY-19-stats.md) |
| **Horário / loja** (se aplicável ao MVP da build) | Pausas programadas, taxa de entrega — [FLUX-HAPPY-12](../fluxos/happypaths/FLUX-HAPPY-12-horario.md) |
| **Usuários** (funcionários) | Cadastro presencial — [FLUX-HAPPY-13](../fluxos/happypaths/FLUX-HAPPY-13-usuarios.md) |

---

## 2. Painel → Pedidos do dia (foco em SLA e reclamação do cliente)

### 2.1 O que o gerente precisa enxergar

- **Estado atual** do pedido (badge colorido — [FLUX-HAPPY-18](../fluxos/happypaths/FLUX-HAPPY-18-painel.md)).
- **Um cronômetro por linha** mostrando há **quanto tempo** o pedido está **na etapa atual** (não só o total já fechado). Regra por `status`: [ORD-01 § 5.7](ORD-01-estados-pedido.md).
- **Durações já gravadas** nas colunas abaixo (formato `mm:ss` ou minutos inteiros no rodapé), para comparar com o “agora” quando o pedido já passou de fase.

### 2.2 Colunas da tabela principal

Ordem sugerida (esquerda → direita). Ajuste visual livre desde que **estado + timer ativo** fiquem visíveis sem scroll horizontal em desktop padrão.

| Coluna | Conteúdo |
|--------|----------|
| **#** | `display_number` |
| **Cliente** | Nome (ou “Cliente”) |
| **Status** | Rótulo + cor ([FLUX-HAPPY-18](../fluxos/happypaths/FLUX-HAPPY-18-painel.md)) |
| **Timer fase** | Cronômetro do segmento atual ([ORD-01 § 5.7](ORD-01-estados-pedido.md)); `—` se `ENTREGUE` ou `CANCELADO` |
| **Fila → prep.** | `allocation_wait_seconds` após **Preparar**; `—` até lá |
| **Preparo** | `prep_duration_seconds` após **Concluir preparo**; enquanto `EM_PREPARO`, repetir o mesmo valor do timer ou `—` + só timer |
| **Balcão** | `balcao_wait_seconds` após **take**; enquanto `PRONTO`, mostrar **live** `now - prep_completed_at` |
| **Entrega** | `delivery_duration_seconds` após **deliver**; enquanto `SAIU_PARA_ENTREGA`, cronômetro desde `dispatch_started_at` |
| **Total** | `total_fulfillment_seconds` quando `ENTREGUE`; até lá pode mostrar `now - created_at` como “tempo desde pedido” opcional |
| **Motoboy** | Nome ou `—` |
| **Valor** | `total_cents` |

Fonte de dados: `GET /orders?storeId=&date=today&…` com os campos de [DB-04 § 2.2](DB-04-catalogo-checkout-cliente.md). Atualização em tempo real: eventos STOMP no tópico da loja ([INF-02](INF-02-http-vs-websocket.md)).

### 2.3 Detalhe do pedido (clique na linha)

- Linhas `order_items`, endereço snapshot, timeline resumida (`order_events`).
- Ações: **Cancelar** (conforme ORD-01 § 2), **Acionar imprevisto em rota** se `SAIU_PARA_ENTREGA` ([ORD-01 § 4](ORD-01-estados-pedido.md)).

### 2.4 Rodapé — estatísticas do período (abaixo da tabela)

Mesmo período selecionável que [DB-03 § 6.1](DB-03-telemetria.md) (`today` / `7d` / `30d`). Cards sugeridos:

| Card | Métrica |
|------|---------|
| Pedidos (volume) | Contagem no período |
| Média **fila → cozinha** | `AVG(allocation_wait_seconds)` |
| Média **preparo** | `AVG(prep_duration_seconds)` |
| p95 **preparo** | Percentil 95 de `prep_duration_seconds` |
| Média **balcão** (pronto → take) | `AVG(balcao_wait_seconds)` |
| Média **entrega** (rua) | `AVG(delivery_duration_seconds)` |
| Média **total** (checkout → entregue) | `AVG(total_fulfillment_seconds)` |
| Taxa de cancelamento | `CANCELADO` / total |

Implementação: `GET /store/stats/orders-summary` (ou nome estável na API) — ver DB-03 § 6.1.

---

## 3. Cardápio gerente

Comportamento completo: [INF-04](INF-04-ui-operacao-cliente.md) § 4 resumido — CRUD, filtros, rodapé de top itens ([DB-03 § 6.2](DB-03-telemetria.md)).

---

*[← Índice docTecnica](README.md)*
