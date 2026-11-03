# FLUX-HAPPY-11-item-cozinha
### KDS — lista à esquerda, detalhe à direita, vários preparos

**Perfis:** 👨‍🍳

[← Índice happy path](README.md)

---

**Perfis:** 👨‍🍳 Cozinha  
**Veja também:** [FLUX-HAPPY-10-item-gerente](FLUX-HAPPY-10-item-gerente.md) · [INF-04-ui-operacao-cliente](../../docTecnica/INF-04-ui-operacao-cliente.md) § 1

```
Esquerda: linhas RECEBIDO + EM_PREPARO (⚡ prioridade, depois chegada)
Clique = seleciona (não prepara sozinho)
Direita: detalhe + Preparar (↖) / Concluir preparo (↗) + timer
Concluir → PRONTO → some da cozinha → motoboy vê na fila
```

---

## Layout

| Área | Conteúdo |
|------|----------|
| **Esquerda** | Uma linha por pedido ativo na cozinha. Contador **N pedidos** |
| **Seleção** | Clique muda cor/borda da linha e carrega o **detalhe à direita** |
| **Direita — topo** | **Preparar** (esquerda) e **Concluir preparo** (direita); centro: **timer** quando em preparo |
| **Direita — corpo** | `display_number` + linhas `order_items` (somente leitura) |

---

## Passo a passo

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | Sistema | Checkout cria pedido `RECEBIDO`; aparece como linha na lista |
| 2 | 👨‍🍳 Cozinha | Clica na linha do pedido — detalhe abre; pedido **ainda** `RECEBIDO` |
| 3 | 👨‍🍳 Cozinha | Clica **Preparar** (↖) |
| 4 | Sistema | `RECEBIDO → EM_PREPARO`; timeline cliente **Em preparo**; timer inicia; estilo “em preparo” na lista e no detalhe; **Preparar** some; **Concluir preparo** aparece |
| 5 | 👨‍🍳 Cozinha | Pode selecionar **outra linha** na lista para alternar detalhe (vários pedidos em preparo ao mesmo tempo) |
| 6 | 👨‍🍳 Cozinha | Embala e anota **`display_number`** no pacote |
| 7 | 👨‍🍳 Cozinha | Clica **Concluir preparo** (↗) |
| 8 | Sistema | Timer congela; grava `prep_duration_seconds`; `→ PRONTO`; linha e detalhe **somem** do KDS |
| 9 | 🛵 Motoboy | Pedido na **fila de prontos** com o mesmo `display_number` |

---

> 📌 **Técnica:** ORD-01 § 5 · `prep_*` e timer DB-04 § 2 · WebSocket INF-02 § 4 · métricas gerente DB-03 § 6
