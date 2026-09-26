# FLUX-SAD-9
### Motoboy desiste de pedido assumido

**Gravidade:** 🟠 · **Perfis:** 🛵 👨‍🍳

[← Índice sad path](README.md)

---

**Perfis:** 🛵 Motoboy · 👨‍🍳 Cozinha · 👤 Cliente
**Causa:** Motoboy teve um imprevisto após assumir mas antes de sair para entrega.

```
Motoboy assumiu → desiste → pedido volta à fila → outro motoboy assume
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Assumiu o pedido #0055 |
| 2 | 🛵 Motoboy | Teve um problema (pneu furado, emergência) e clica **Desistir** |
| 3 | Sistema | Libera o pedido — volta ao estado **PRONTO** na fila |
| 4 | 👨‍🍳 Cozinha | Pedido aparece de novo como disponível para motoboy |
| 5 | 👤 Cliente | A timeline não avança — permanece em "Pronto para retirada" |
| 6 | 🛵 Outro motoboy | Vê o pedido e pode assumir normalmente |
| 7 | 👤 Cliente | Vê a timeline avançar quando o novo motoboy assume e sai |

> **Impacto:** Médio. O cliente tem um atraso. O pedido não é cancelado — apenas redistribuído.
