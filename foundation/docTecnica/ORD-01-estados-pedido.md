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
| `EM_PREPARO` | Cozinha aceitou; itens visíveis no painel direito do KDS para conferência e embalagem | 👨‍🍳 Cozinha ou 🔑 Gerente |
| `PRONTO` | Preparo concluído; aguardando motoboy no balcão / fila do app | 👨‍🍳 Cozinha ou 🔑 Gerente |
| `SAIU_PARA_ENTREGA` | Motoboy **aceitou** o pedido no app (após conferir `display_number` na embalagem) | 🛵 Motoboy |
| `ENTREGUE` | Entrega confirmada | 🛵 Motoboy |
| `CANCELADO` | Encerrado com estorno, se o pagamento estava confirmado | 👤 Cliente ou 🔑 Gerente, conforme a janela |

Não existe `RECUSADO`, `PENDENTE` nem `ACEITO`. Aceitar o pedido da fila é a transição `RECEBIDO → EM_PREPARO`.

> **Pré-condição para aceite no app motoboy (`POST /orders/{id}/take`):** turno ativo (`motoboyAtivo = true`); pedido `PRONTO` sem motoboy; motoboy confere **`display_number`** na embalagem. UI: botão **Aceitar pedido** → tela de acompanhamento. Ver [FLUX-HAPPY-20-motoboy-app](../fluxos/happypaths/FLUX-HAPPY-20-motoboy-app.md) · INF-04 § 2.

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
   │ motoboy aceita no app (take)
   ▼
SAIU_PARA_ENTREGA ── imprevisto ─► RECEBIDO (mesmo id, priority, cancelamento bloqueado)
   │ entrega
   ▼
ENTREGUE
```

---

## 4. Imprevisto em rota

Um único `order_id` para o cliente. O motivo `IMPREVISTO_ENTREGA` aparece na **timeline** do pedido; isso é evento, não um estado permanente.

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

**Itens físicos (fora do sistema):** após o imprevisto, o motoboy **contata o gerente** (WhatsApp, telefone ou reporte no app). O gerente **não** marca no sistema o que foi descartado ou devolvido. Na prática: descarta comida que não volta, devolve depois itens duráveis (ex.: bebidas lacradas) **presencialmente** — decisão operacional, sem campos `DISCARDED`, pendência de devolução ou `item_type` no MVP.

O gerente só **aciona imprevisto** no painel: mesmo `order_id` → `RECEBIDO` + `priority = true` + mensagem ao cliente. A cozinha refaz conforme as linhas do pedido no KDS.

Quando o mesmo pedido chega a `ENTREGUE`, `incident_alert` volta a `false`.

---

## 5. Fila do KDS — ordem, seleção e vários preparos

Pedidos **`RECEBIDO`** e **`EM_PREPARO`** aparecem na **lista esquerda** do KDS até irem a **`PRONTO`**. A cozinha pode ter **vários** pedidos `EM_PREPARO` ao mesmo tempo.

**Seleccionar ≠ preparar:** clicar numa linha só abre o **detalhe à direita**. Nenhum pedido entra em preparo automaticamente (nem o primeiro da fila).

### 5.1 Ordenação da lista KDS

| Critério | Ordem |
|----------|--------|
| `priority` | `true` antes de `false` (⚡ imprevisto) |
| Entre iguais | `created_at ASC` (ordem de chegada) |

Inclui pedidos `RECEBIDO` e `EM_PREPARO`. Ao concluir (`→ PRONTO`), o pedido **some** da lista e do detalhe da cozinha.

### 5.2 Iniciar e concluir preparo

| Ação (UI detalhe, canto **superior esquerdo**) | Estado | Efeito |
|------------------------------------------------|--------|--------|
| **Preparar** | `RECEBIDO` | `→ EM_PREPARO`; timeline cliente **Em preparo**; grava `prep_started_at`; inicia **timer** no detalhe; estilo da linha na lista + painel direito muda (borda/cor “em preparo”); botão esquerdo some |
| **Concluir preparo** (canto **superior direito**) | `EM_PREPARO` | `→ PRONTO`; para timer; grava `prep_completed_at` e `prep_duration_seconds`; pedido some do KDS; fila motoboy |

| Regra | Detalhe |
|-------|---------|
| Qual pedido preparar | Qualquer `RECEBIDO` **selecionado** — **não** exige ser o primeiro da lista |
| Alternar pedidos | Clicar outra linha na lista → detalhe à direita troca de pedido (timer e botões do pedido selecionado) |
| Servidor | Rejeita **Preparar** se status ≠ `RECEBIDO`; rejeita **Concluir** se status ≠ `EM_PREPARO` (`INVALID_STATE`) |

Protocolo STOMP: INF-02 § 4 (`/accept` → preparar, `/ready` → concluir).

### 5.3 Timer de preparo (cozinha)

| Regra | Detalhe |
|-------|---------|
| Onde | Centro **superior** do painel de detalhe (direita) |
| Quando corre | Enquanto pedido `EM_PREPARO` e timer não congelado |
| Origem | `prep_started_at` (servidor) — UI atualiza a cada segundo |
| Ao concluir | Timer **congela** no valor final; persiste em `orders.prep_duration_seconds` |

### 5.4 Tabela `orders` — tempo de preparo

| Coluna | Tipo | Regra |
|--------|------|--------|
| `prep_started_at` | `timestamptz`, nulo | Preenchido em `RECEBIDO → EM_PREPARO` |
| `prep_completed_at` | `timestamptz`, nulo | Preenchido em `EM_PREPARO → PRONTO` |
| `prep_duration_seconds` | int, nulo | Segundos congelados ao concluir: `prep_completed_at - prep_started_at` (arredondamento inteiro para cima no MVP) |

### 5.5 KDS — painéis (resumo)

| Painel | Conteúdo |
|--------|----------|
| Esquerda | Linhas `RECEBIDO` + `EM_PREPARO`; contador; seleção por clique (destaque visual) |
| Direita | Detalhe do **selecionado**: `display_number`, timer (se `EM_PREPARO`), linhas `order_items`, **Preparar** (↖) / **Concluir preparo** (↗) |
| Embalagem | Anotar `display_number` antes de concluir |

Layout: [INF-04-ui-operacao-cliente.md](INF-04-ui-operacao-cliente.md) § 1.

---

## 6. Timeout (resumo)

Pedidos `RECEBIDO` permanecem na fila **indefinidamente** até a cozinha clicar **Preparar** ou até cancelamento manual. **Não** há alerta automático ao gerente por demora na cozinha nem timeout que cancele o pedido.

---

## 7. Pagamento

| Evento | Pagamento |
|--------|-----------|
| Checkout ok | `CONFIRMADO` no mesmo transaction do `RECEBIDO` |
| Simulação falha | Rollback. Nenhum pedido |
| `CANCELADO` | `ESTORNADO` se estava `CONFIRMADO` |
| Imprevisto em rota | Permanece `CONFIRMADO` |

---

## 8. Timeline do pedido — `order_events`

Persistência **técnica** para a timeline do cliente, mensagens de imprevisto e algumas métricas (DB-03). **Não** há tela ou módulo de “auditoria” no produto.

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

A linha do pedido (`order_items`) guarda `product_name`, `unit_price`/`unit_price_cents` e `quantity` copiados no checkout. Ver [DB-04-catalogo-checkout-cliente.md](DB-04-catalogo-checkout-cliente.md).

---

*[← Índice docTecnica](README.md)*
