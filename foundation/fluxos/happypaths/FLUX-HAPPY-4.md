# FLUX-HAPPY-4
### Venda completa — do carrinho à entrega na porta

**Perfis:** 👤 👨‍🍳 🛵

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente · 👨‍🍳 Cozinha · 🛵 Motoboy

> Este é o **fluxo principal** do sistema. Tudo corre bem, sem nenhum problema.

```
Cliente monta carrinho → finaliza pedido → cozinha prepara → motoboy entrega
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Abre o carrinho com os itens escolhidos |
| 2 | 👤 Cliente | Revisa o resumo: itens, quantidades e valores |
| 3 | 👤 Cliente | Confirma o endereço de entrega |
| 4 | 👤 Cliente | Vê o valor do frete calculado automaticamente pela região |
| 5 | 👤 Cliente | Clica em **Finalizar Pedido** |
| 6 | Sistema | Verifica que todos os itens ainda estão disponíveis |
| 7 | Sistema | Confirma o pagamento (simulado) e registra o pedido |
| 8 | 👤 Cliente | Vê a tela de confirmação: **"Pedido recebido!"** com número do pedido |
| 9 | 👨‍🍳 Cozinha | Recebe o novo pedido na tela da cozinha (KDS) com um sinal sonoro |
| 10 | 👨‍🍳 Cozinha | Clica em **Aceitar** — começa a contagem da cozinha |
| 11 | 👤 Cliente | Vê na timeline: **"Pedido aceito pela loja"** |
| 12 | 👨‍🍳 Cozinha | Clica em **Iniciar preparo** quando começa a montar o lanche |
| 13 | 👨‍🍳 Cozinha | Clica em **Marcar como pronto** quando embalou o pedido |
| 14 | 🛵 Motoboy | Vê o pedido aparecer na **fila de disponíveis** no app dele |
| 15 | 🛵 Motoboy | Clica em **Assumir** — o pedido é bloqueado só para ele |
| 16 | 👤 Cliente | Vê na timeline: **"Motoboy a caminho para retirar"** |
| 17 | 🛵 Motoboy | Chega na loja, pega o pedido, clica em **Saiu para entrega** |
| 18 | 👤 Cliente | Vê na timeline: **"Pedido saiu para entrega"** |
| 19 | 🛵 Motoboy | Chega na casa do cliente, entrega o pedido, clica em **Confirmar entrega** |
| 20 | 👤 Cliente | Vê na timeline: **"Pedido entregue! ✅"** — evento de venda encerrado |

**Linha do tempo resumida:**

```
CLIENTE         COZINHA              MOTOBOY
   │                │                   │
   ├─ monta carrinho│                   │
   ├─ finaliza ─────┤                   │
   │           aceita│                  │
   │           prepara                  │
   │           pronto├───────────────── fila
   │                │              assume
   │                │         retira ──┤
   │                │    entrega ──────┤
   ✅ entregue       │                  │
```
