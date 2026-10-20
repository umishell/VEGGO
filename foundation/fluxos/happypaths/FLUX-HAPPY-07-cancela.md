# FLUX-HAPPY-07-cancela
### Cliente cancela pedido antes do preparo

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente · 👨‍🍳 Cozinha

```
Pedido ainda na fila da cozinha → cliente cancela → sai da fila → pagamento estornado
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Abre o pedido em **Meus pedidos** |
| 2 | Sistema | Mostra **Cancelar** somente se o estado é `RECEBIDO` |
| 3 | 👤 Cliente | Confirma o cancelamento |
| 4 | Sistema | Muda para `CANCELADO` e estorna o pagamento |
| 5 | 👨‍🍳 Cozinha | O pedido sai da fila |
| 6 | 👤 Cliente | Vê **"Pedido cancelado. Pagamento estornado."** |

> A janela fecha quando a cozinha clica em **Aceitar**. Esse clique leva o pedido a `EM_PREPARO`. Ver [FLUX-SAD-09-cancela-tarde](../sadPaths/FLUX-SAD-09-cancela-tarde.md).

> Pedido que voltou para a fila por imprevisto em rota não mostra **Cancelar**: o cliente já pagou e o lanche está sendo refeito.
