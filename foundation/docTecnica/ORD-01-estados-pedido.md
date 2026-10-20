# ORD-01 — Máquina de Estados do Pedido

**Referenciado por:**  
→ [FLUX-SAD-09-cancela-tarde](../fluxos/sadPaths/FLUX-SAD-09-cancela-tarde.md) · [FLUX-SAD-16-entrega-desiste](../fluxos/sadPaths/FLUX-SAD-16-entrega-desiste.md) · [FLUX-SAD-11-cancela-loja](../fluxos/sadPaths/FLUX-SAD-11-cancela-loja.md) · [FLUX-SAD-18-rota-gerente](../fluxos/sadPaths/FLUX-SAD-18-rota-gerente.md) · [FLUX-SAD-19-rota-motoboy](../fluxos/sadPaths/FLUX-SAD-19-rota-motoboy.md) · [FLUX-SAD-17-entrega-prazo](../fluxos/sadPaths/FLUX-SAD-17-entrega-prazo.md) · [FLUX-HAPPY-07-cancela](../fluxos/happypaths/FLUX-HAPPY-07-cancela.md)

*[← Índice docTecnica](README.md)*

Alinhado a [`pitch_delivery.md`](../pitch_delivery.md). O nome do estado inicial é `RECEBIDO`.

---

## 1. Estados do pedido

| Estado | Descrição | Quem transiciona |
|--------|-----------|-----------------|
| `RECEBIDO` | Pedido criado e pago. Já está na fila da cozinha. Ninguém aceitou ainda | Sistema, no checkout |
| `EM_PREPARO` | Cozinha aceitou o pedido da fila. O preparo começa nesse clique | 👨‍🍳 Cozinha ou 🔑 Gerente |
| `PRONTO` | Embalado, sem motoboy | 👨‍🍳 Cozinha ou 🔑 Gerente |
| `ALOCADO` | Motoboy assumiu e ainda não saiu. Lanche está na loja | 🛵 Motoboy |
| `SAIU_PARA_ENTREGA` | Motoboy saiu com o pedido | 🛵 Motoboy |
| `ENTREGUE` | Entrega confirmada | 🛵 Motoboy |
| `CANCELADO` | Encerrado com estorno, se o pagamento estava confirmado | 👤 Cliente ou 🔑 Gerente, conforme a janela |

Não existe `RECUSADO`, `PENDENTE` nem `ACEITO`. Aceitar o pedido da fila é a transição `RECEBIDO → EM_PREPARO`.

---

## 2. Quem cancela

| Estado | 👤 Cliente | 🔑 Gerente |
|--------|------------|------------|
| `RECEBIDO` | Sim. O pedido sai da fila da cozinha | Sim, com estorno |
| `EM_PREPARO` | Não | Sim, com estorno. Exceção de um pedido, não o fechamento da loja |
| `PRONTO` sem motoboy | Não | Sim, com estorno |
| `ALOCADO` | Não | Desalocar volta para `PRONTO`. Não cancela e não estorna |
| `SAIU_PARA_ENTREGA` | Não | Imprevisto: volta para `RECEBIDO`. Não estorna |
| `ENTREGUE` | Não | Não |

`cancelamento_bloqueado = true` impede o cliente de cancelar mesmo em `RECEBIDO`. O imprevisto em rota liga essa flag até `ENTREGUE`.

O fechamento da loja (`FLUX-SAD-13-loja-emergencia`) não percorre estas transições. Pedido já criado continua.

---

## 3. Diagrama

```
RECEBIDO ── cliente ou gerente cancela ─► CANCELADO + estorno
   │ cozinha aceita (prepara)
   ▼
EM_PREPARO ── gerente cancela ─► CANCELADO + estorno
   │
   ▼
PRONTO ── gerente cancela (sem motoboy) ─► CANCELADO + estorno
   │ motoboy assume
   ▼
ALOCADO ── desiste, gerente desaloca ou timeout ─► PRONTO
   │ retira
   ▼
SAIU_PARA_ENTREGA ── imprevisto ─► RECEBIDO (mesmo id, cancelamento bloqueado)
   │ entrega
   ▼
ENTREGUE
```

---

## 4. Imprevisto em rota

Um único `order_id` para o cliente. A auditoria grava o motivo `IMPREVISTO_ENTREGA`; isso é evento, não um estado permanente.

| Campo | Depois do imprevisto |
|-------|----------------------|
| `status` | `RECEBIDO` |
| `motoboy_id` | `null` |
| `incident_alert` | `true` até `ENTREGUE` |
| `cancelamento_bloqueado` | `true` até `ENTREGUE` |
| `remake_seq` | incrementa |
| `payment_status` | permanece `CONFIRMADO` |

O KDS lista os itens desse `remake_seq` sem nome, telefone ou endereço do cliente. O endereço segue no pedido para o próximo motoboy.

| Item | Regra |
|------|-------|
| Lanche que estava na rota | Descartado. A auditoria registra perda operacional. Não há tela de estoque |
| Bebida | Pendência de devolução do motoboy (`order_id`, `motoboy_id`, `pending`). Sem módulo de estoque |

Quando o mesmo pedido chega a `ENTREGUE`, `incident_alert` volta a `false`.

---

## 5. Timeout

| Situação | Efeito |
|----------|--------|
| Cozinha não tira o pedido da fila | Permanece `RECEBIDO`. Depois de `accept_alert_minutes` o gerente recebe alerta |
| Motoboy não retira em `ALOCADO` | Depois de `allocation_timeout_minutes` o pedido volta a `PRONTO` e o lanche continua na loja |

Os dois prazos ficam na tabela `stores`.

---

## 6. Pagamento

| Evento | Pagamento |
|--------|-----------|
| Checkout ok | `CONFIRMADO` no mesmo transaction do `RECEBIDO` |
| Simulação falha | Rollback. Nenhum pedido |
| `CANCELADO` | `ESTORNADO` se estava `CONFIRMADO` |
| Imprevisto em rota | Permanece `CONFIRMADO` |

---

## 7. Auditoria — `order_events`

| Campo | Tipo |
|-------|------|
| `order_id` | UUID |
| `previous_status` | enum |
| `new_status` | enum |
| `changed_by_role` | `system`, `kitchen`, `delivery`, `manager`, `client` |
| `changed_by_id` | UUID |
| `reason` | string, nulo fora de cancelamento, timeout e imprevisto |
| `occurred_at` | timestamp |

Índices: `order_id`, `occurred_at DESC`.

A linha do pedido (`order_items`) guarda `product_name` e `unit_price` copiados no checkout.

---

*[← Índice docTecnica](README.md)*
