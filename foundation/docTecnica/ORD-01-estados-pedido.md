# ORD-01 — Máquina de Estados do Pedido

**Referenciado por:**  
→ [FLUX-SAD-09-cancela-tarde](../fluxos/sadPaths/FLUX-SAD-09-cancela-tarde.md) · [FLUX-SAD-11-cancela-loja](../fluxos/sadPaths/FLUX-SAD-11-cancela-loja.md) · [FLUX-SAD-18-rota-gerente](../fluxos/sadPaths/FLUX-SAD-18-rota-gerente.md) · [FLUX-SAD-19-rota-motoboy](../fluxos/sadPaths/FLUX-SAD-19-rota-motoboy.md) · [FLUX-HAPPY-07-cancela](../fluxos/happypaths/FLUX-HAPPY-07-cancela.md)

*[← Índice docTecnica](README.md)*

Alinhado a [`pitch_delivery.md`](../pitch_delivery.md). O nome do estado inicial é `RECEBIDO`.

---

## 1. Estados do pedido

| Estado | Descrição | Quem transiciona |
|--------|-----------|-----------------|
| `RECEBIDO` | Pedido criado e pago. Já está na fila da cozinha. Ninguém aceitou ainda | Sistema, no checkout |
| `EM_PREPARO` | Cozinha aceitou o pedido da fila. O preparo começa nesse clique | 👨‍🍳 Cozinha ou 🔑 Gerente |
| `PRONTO` | Embalado, aguardando motoboy | 👨‍🍳 Cozinha ou 🔑 Gerente |
| `SAIU_PARA_ENTREGA` | Motoboy assumiu e saiu com o pedido (transição única) | 🛵 Motoboy |
| `ENTREGUE` | Entrega confirmada | 🛵 Motoboy |
| `CANCELADO` | Encerrado com estorno, se o pagamento estava confirmado | 👤 Cliente ou 🔑 Gerente, conforme a janela |

Não existe `RECUSADO`, `PENDENTE` nem `ACEITO`. Aceitar o pedido da fila é a transição `RECEBIDO → EM_PREPARO`.

> **Pré-condição para "Assumir e sair":** o motoboy precisa ter turno ativo (`delivery_persons.status = DISPONIVEL`), representado por `motoboyAtivo = true` no JWT. Turno não iniciado (`INATIVO`) bloqueia a ação no servidor. Ver [FLUX-HAPPY-16-turno](../fluxos/happypaths/FLUX-HAPPY-16-turno.md).

---

## 2. Quem cancela

| Estado | 👤 Cliente | 🔑 Gerente |
|--------|------------|------------|
| `RECEBIDO` | Sim. O pedido sai da fila da cozinha | Sim, com estorno |
| `EM_PREPARO` | Não | Sim, com estorno. Exceção de um pedido, não o fechamento da loja |
| `PRONTO` sem motoboy | Não | Sim, com estorno |
| `SAIU_PARA_ENTREGA` | Não | Acionar imprevisto: volta a `RECEBIDO` com prioridade. Não estorna |
| `ENTREGUE` | Não | Não |

`cancelamento_bloqueado = true` impede **somente o cliente** de cancelar em `RECEBIDO`. O **gerente não é bloqueado** por este campo — pode cancelar o pedido com estorno a qualquer momento, mesmo após imprevisto em rota, se não houver como refazer o pedido.

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
   │ motoboy assume e sai (transição única)
   ▼
SAIU_PARA_ENTREGA ── imprevisto ─► RECEBIDO (mesmo id, priority, cancelamento bloqueado)
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
| `priority` | `true` — pedido vai ao topo da fila do KDS |
| `incident_alert` | `true` — marca o pedido como tendo tido imprevisto; exibe aviso na timeline do cliente |
| `incident_message` | mensagem do gerente para o cliente (gravada no pedido; exibida via WebSocket) |
| `cancelamento_bloqueado` | `true` até `ENTREGUE` — cliente não pode cancelar |
| `remake_seq` | incrementa |
| `payment_status` | permanece `CONFIRMADO` |

**Status do motoboy após imprevisto confirmado:** o motoboy envolvido retorna automaticamente a `DISPONIVEL` (turno continua ativo; ele pode assumir outros pedidos). O sistema grava `INCIDENT_CONFIRMED` em `order_events` com o `motoboy_id` antes de zerálo no pedido.

**Prioridade no KDS:** pedido com `priority = true` aparece no topo da fila `RECEBIDO`. A cozinha vê o indicador ⚡. Pedidos já em `EM_PREPARO` não são interrompidos — o pedido prioritário aguarda sua vez para ser aceito normalmente pelo próximo cozinheiro livre.

O KDS lista os itens desse `remake_seq` sem nome, telefone ou endereço do cliente. O endereço segue no pedido para o próximo motoboy.

**Regra de destino dos itens após imprevisto em rota:**

A classificação usa `products.item_type` (definido pelo gerente no cadastro do produto):

| `item_type` | Exemplos | Destino | Como o gerente acompanha |
|-------------|----------|---------|--------------------------|
| `PREPARED_FOOD` | Lanche, batata, sobremesa | **Descartados** — fora da loja, não retornam. Auditoria registra `ITEM_DISCARDED` por `order_id + remake_seq` | Detalhe do pedido no painel — linha por item com status `DISCARDED` |
| `SEALED_BEVERAGE` | Lata, garrafa | **Pendência de devolução** — `order_events` registra `BEVERAGE_RETURN_PENDING` com `order_id`, `motoboy_id` e lista de itens pendentes | Detalhe do pedido no painel — badge "Devolução pendente". Quando o motoboy devolve pessoalmente, o gerente clica **Devolução recebida** gravando `BEVERAGE_RETURN_CONFIRMED` |

O gerente **não precisa de tela separada**: o detalhe do pedido (acessível pelo painel do dia, `FLUX-HAPPY-18-painel`) mostra os itens com seus status e o botão de confirmar devolução quando houver pendência. Não há módulo de estoque — é apenas registro de auditoria.

Quando o mesmo pedido chega a `ENTREGUE`, `incident_alert` volta a `false`.

---

## 5. Timeout

| Situação | Efeito |
|----------|--------|
| Cozinha não tira o pedido da fila | Permanece `RECEBIDO`. Depois de `accept_alert_minutes` o gerente recebe alerta |

O prazo `accept_alert_minutes` fica na tabela `stores`. Não existe timeout que cancele o pedido automaticamente — apenas alerta.

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
