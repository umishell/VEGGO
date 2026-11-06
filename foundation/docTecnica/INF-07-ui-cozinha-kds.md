# INF-07 — Cozinha — KDS (`web-operacao`)

**Referenciado por:** [ORD-01 § 5](ORD-01-estados-pedido.md) · [FLUX-HAPPY-11](../fluxos/happypaths/FLUX-HAPPY-11-item-cozinha.md)

*[← Índice docTecnica](README.md)*

Perfil **cozinha** (funcionário com escopo loja). Aplicativo **web** em modo paisagem, dois painéis. Comunicação: WebSocket STOMP para `accept` / `ready` ([INF-02 § 4](INF-02-http-vs-websocket.md)).

---

## 1. Finalidade

- Ver a fila **`RECEBIDO`** + **`EM_PREPARO`**.
- **Selecionar** um pedido (clique na linha) sem mudar estado.
- **Preparar** (`RECEBIDO → EM_PREPARO`) e **Concluir preparo** (`EM_PREPARO → PRONTO`).
- **Imprimir recibo** na maquininha e grampear no pacote **antes** de concluir ([DB-04 § 2](DB-04-catalogo-checkout-cliente.md)).

---

## 2. Layout (resumo visual)

```
┌─────────────────────────┬──────────────────────────────┐
│  FILA (N linhas)        │  DETALHE DO PEDIDO SELECIONADO │
│  #42 ⚡ recebido        │  [ Preparar ]  [ Concluir prep. ] │
│  #43   em preparo ◀     │        ⏱ mm:ss                 │
│  #44   recebido         │  itens: …                      │
└─────────────────────────┴──────────────────────────────┘
```

---

## 3. Elementos e regras

| Elemento | Comportamento |
|----------|----------------|
| Lista esquerda | Uma linha por pedido `RECEBIDO` ou `EM_PREPARO`; ordenação [ORD-01 § 5.1](ORD-01-estados-pedido.md); badge **N** = quantidade de linhas no pedido |
| Seleção | Destaque visual; **não** altera estado sozinha |
| **Preparar** | Só com linha `RECEBIDO` selecionada; envia `accept` → grava `prep_started_at` e `allocation_wait_seconds` ([DB-04 § 2.5](DB-04-catalogo-checkout-cliente.md)) |
| **Timer** | Após `EM_PREPARO`: cronômetro desde `prep_started_at`; atualização a cada segundo ([ORD-01 § 5.3](ORD-01-estados-pedido.md)) |
| **Concluir preparo** | Só em `EM_PREPARO`; grava `prep_completed_at`, `prep_duration_seconds`; pedido **some** do KDS (`PRONTO`) |
| Detalhe — itens | Somente leitura (`order_items`) |
| Recibo | Fluxo obrigatório antes de concluir (impressão térmica + grampo) |

Pedidos **`PRONTO` e seguintes** não aparecem no KDS.

---

## 4. O que o KDS *não* faz (MVP)

- Não lista cardápio para edição.
- Não mostra endereço completo do cliente (fica no app do motoboy após o take).
- Não dispara alerta automático por demora na fila `RECEBIDO` ([ORD-01 § 6](ORD-01-estados-pedido.md)).

---

*[← Índice docTecnica](README.md)*
