# FLUX-SAD-6
### Cozinha recusa o pedido

**Gravidade:** 🔴 · **Perfis:** 👤 👨‍🍳

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente · 👨‍🍳 Cozinha
**Causa:** Loja lotada, fechamento antecipado, falha de ingredientes — cozinha não pode atender.

```
Pedido chega → cozinha recusa → cliente é notificado e pagamento estornado
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Faz o pedido normalmente — pagamento confirmado |
| 2 | 👨‍🍳 Cozinha | Recebe o pedido no painel da cozinha |
| 3 | 👨‍🍳 Cozinha | Avalia que não pode atender e clica em **Recusar** |
| 4 | Sistema | Cancela o pedido e inicia o estorno do pagamento automaticamente |
| 5 | 👤 Cliente | Recebe a notificação: **"Pedido recusado pela loja. Seu pagamento será estornado."** |
| 6 | 👤 Cliente | Vê o pedido como **"Recusado"** no histórico com o estorno registrado |

> **Impacto:** Alto para o cliente, mas o pagamento é devolvido. O sistema garante que não fica dinheiro cobrado por pedido não atendido.
