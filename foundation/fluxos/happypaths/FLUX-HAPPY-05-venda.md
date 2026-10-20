# FLUX-HAPPY-05-venda
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
| 3 | 👤 Cliente | Confirma o endereço de entrega (endereço salvo ou digita novo) |
| 3a | Sistema | Se o cliente digitou um endereço novo, **salva automaticamente** no perfil dele |
| 4 | 👤 Cliente | Vê o valor do frete calculado automaticamente pela região |
| 5 | 👤 Cliente | Clica em **Finalizar Pedido** |
| 6 | Sistema | Verifica que todos os itens ainda estão disponíveis |
| 7 | Sistema | Confirma o pagamento (simulado) e registra o pedido |
| 8 | 👤 Cliente | Vê a tela de confirmação: **"Pedido recebido!"** com número do pedido |
| 9 | 👨‍🍳 Cozinha | Recebe o novo pedido na tela da cozinha (KDS) com um sinal sonoro |
| 10 | 👨‍🍳 Cozinha | Clica em **Aceitar**. O pedido sai da fila e vai para `EM_PREPARO` — é nesse clique que o preparo começa |
| 11 | 👤 Cliente | Vê na timeline: **"Em preparo"** |
| 12 | 👨‍🍳 Cozinha | Clica em **Marcar como pronto** quando embalou o pedido |
| 13 | 🛵 Motoboy | Vê o pedido aparecer na **fila de disponíveis** no app dele |
| 14 | 🛵 Motoboy | Clica em **Assumir** — o pedido é bloqueado só para ele |
| 15 | 👤 Cliente | Vê na timeline: **"Motoboy a caminho para retirar"** |
| 16 | 🛵 Motoboy | Chega na loja, pega o pedido, clica em **Saiu para entrega** |
| 17 | 👤 Cliente | Vê na timeline: **"Pedido saiu para entrega"** |
| 18 | 🛵 Motoboy | Chega na casa do cliente, entrega o pedido, clica em **Confirmar entrega** |
| 19 | 👤 Cliente | Vê na timeline: **"Pedido entregue! ✅"** — evento de venda encerrado |

**Linha do tempo resumida:**

```
CLIENTE         COZINHA              MOTOBOY
   │                │                   │
   ├─ monta carrinho│                   │
   ├─ finaliza ─────┤                   │
   │      aceita e prepara              │
   │           pronto├───────────────── fila
   │                │              assume
   │                │         retira ──┤
   │                │    entrega ──────┤
   ✅ entregue       │                  │
```
