# FLUX-SAD-10-cancela-saiu
### Cliente tenta cancelar após motoboy sair

**Gravidade:** 🔴 · **Perfis:** 👤 🛵

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente · 🛵 Motoboy
**Causa:** Pedido já está `SAIU_PARA_ENTREGA` — impossível cancelar.

```
Lanche a caminho → cliente não pode mais cancelar → pedido segue até a entrega
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Retirou o pedido na loja e clicou **Saiu para entrega** |
| 2 | 👤 Cliente | Tenta cancelar o pedido pelo app |
| 3 | Sistema | Bloqueio total — sem opção de cancelamento |
| 4 | 👤 Cliente | Vê: **"Pedido a caminho — cancelamento não é possível neste momento."** |
| 5 | 🛵 Motoboy | Continua o percurso normalmente |
| 6 | 👤 Cliente | Recebe o pedido |

> **Impacto:** Alto. Não há estorno neste estado. O cliente fala com a loja pelo WhatsApp: [FLUX-HAPPY-17-whatsapp](../happypaths/FLUX-HAPPY-17-whatsapp.md).
