# FLUX-HAPPY-21-checkout-pagamento
### Finalizar pedido — endereço, taxa de entrega e pagamento mock

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente  
**Veja também:** [FLUX-HAPPY-04-endereco](FLUX-HAPPY-04-endereco.md) · [FLUX-SAD-03-item-checkout](../sadPaths/FLUX-SAD-03-item-checkout.md) · [FLUX-SAD-06-pagamento](../sadPaths/FLUX-SAD-06-pagamento.md) · [DB-04-catalogo-checkout-cliente](../../docTecnica/DB-04-catalogo-checkout-cliente.md)

```
Carrinho → Finalizar → tela pagamento → sucesso ou erro explicado
```

---

## Tela de pagamento (checkout)

| Bloco | Comportamento |
|-------|----------------|
| **Endereço de entrega** | Combobox com endereços salvos; **padrão** pré-selecionado. Abaixo, campos para **novo endereço** (CEP, rua, número, bairro, complemento, referência) |
| **Resumo** | Subtotal, **Entrega** (taxa fixa da loja — `GET /menu` · DB-02), **total** (`subtotal + entrega`) |
| **Forma de pagamento** | Combobox cartões/formas **salvas** (autofill) ou preenchimento **manual** para usar/cadastrar novo no ato |
| **Finalizar pedido** | Dispara checkout atômico (mock de pagamento no servidor) |

---

## Passo a passo feliz

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | No carrinho, toca **Finalizar pedido** |
| 2 | 👤 Cliente | Confirma ou altera endereço; vê taxa de entrega e total |
| 3 | 👤 Cliente | Escolhe forma de pagamento |
| 4 | 👤 Cliente | Toca **Finalizar pedido** |
| 5 | Sistema | Revalida itens, limites e pagamento — **não** bloqueia só porque a loja fechou se `shoppingSessionId` válido ([DB-04 § 7](../../docTecnica/DB-04-catalogo-checkout-cliente.md)) |
| 6 | Sistema | Cria pedido `RECEBIDO` (`orders.id` UUID + `display_number` do dia); salva endereço/pagamento novos se marcado |
| 7 | 👤 Cliente | Tela **Pedido recebido!** com **`display_number`** (#…) e link para acompanhar (timeline usa UUID em `/orders/{id}`) |

---

## Quando algo falha

| Situação | UX |
|----------|-----|
| Item indisponível / limite | Modal ou tela com **explicação** + código (`ITEM_UNAVAILABLE`, `DAILY_LIMIT_EXCEEDED`, …) |
| Sem sessão e loja fechada | `STORE_CLOSED` — [FLUX-SAD-14](../sadPaths/FLUX-SAD-14-loja-fechada.md) |
| Pagamento simulado recusado | Mensagem clara; carrinho intacto — [FLUX-SAD-06-pagamento](../sadPaths/FLUX-SAD-06-pagamento.md) |

> Checkout `@Transactional`: falha em qualquer etapa → **nenhum** pedido parcial. Esquema: [DB-04 § 2](../../docTecnica/DB-04-catalogo-checkout-cliente.md).
