# FLUX-HAPPY-08-historico
### Histórico de pedidos e refazer pedido

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente

```
Meus pedidos → lista do mais recente ao mais antigo → detalhe → refazer no carrinho
```

---

## Lista e detalhe

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Abre **Meus pedidos** / **Histórico** |
| 2 | Sistema | `GET /orders/me` — ordenação **`created_at DESC`** |
| 3 | 👤 Cliente | Vê número do dia (`displayNumber`), data, total e status final |
| 4 | 👤 Cliente | Toca em um pedido |
| 5 | Sistema | `GET /orders/{id}` — itens, endereço snapshot, timeline resumida |
| 6 | 👤 Cliente | Só vê **seus** pedidos (FGAC no servidor) |

---

## Refazer pedido

| # | Quem | O que acontece |
|---|------|----------------|
| 7 | 👤 Cliente | Toca **Refazer pedido** |
| 8 | Sistema | Copia linhas (`productId`, `quantity`) para o carrinho — substitui carrinho atual; novo `cartStartedAt` |
| 9 | Sistema | Se loja **fechada** e **sem** sessão de compra válida → bloqueia com aviso (mesma regra de entrada — [FLUX-SAD-14](../sadPaths/FLUX-SAD-14-loja-fechada.md)) |
| 10 | 👤 Cliente | Ajusta carrinho se algum item ficou indisponível (checkout ainda valida — [FLUX-SAD-03](../sadPaths/FLUX-SAD-03-item-checkout.md)) |
| 11 | 👤 Cliente | Segue para checkout normal ([FLUX-HAPPY-21](FLUX-HAPPY-21-checkout-pagamento.md)) |

> 💡 **Refazer** não reutiliza o UUID do pedido antigo — cria um **novo** pedido no checkout.

---

> 📌 [DB-04 § 8](../../docTecnica/DB-04-catalogo-checkout-cliente.md) · [INF-02 § 11](../../docTecnica/INF-02-http-vs-websocket.md)
