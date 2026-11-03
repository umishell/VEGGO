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
| 5 | Sistema | Revalida loja, itens, limites; aplica `fixed_delivery_fee_cents` da loja; simula pagamento |
| 6 | Sistema | Cria pedido `RECEBIDO` + `display_number`; salva endereço/pagamento novos se marcado |
| 7 | 👤 Cliente | Tela **Pedido recebido!** com número (`display_number`) e link para acompanhar |

---

## Quando algo falha

| Situação | UX |
|----------|-----|
| Item indisponível / limite / loja fechada | Modal ou tela com **explicação** + código (`ITEM_UNAVAILABLE`, `DAILY_LIMIT_EXCEEDED`, `STORE_CLOSED`, …) — ver sad paths |
| Pagamento simulado recusado | Mensagem clara; carrinho intacto — [FLUX-SAD-06-pagamento](../sadPaths/FLUX-SAD-06-pagamento.md) |

> Checkout `@Transactional`: falha em qualquer etapa → **nenhum** pedido parcial.
