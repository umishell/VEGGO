# FLUX-SAD-14
### Gerente cancela pedido já aceito

**Gravidade:** 🔴 · **Perfis:** 🔑 👤

[← Índice sad path](README.md)

---

**Perfis:** 🔑 Gerente · 👤 Cliente · 👨‍🍳 Cozinha
**Causa:** Imprevisto grave — falta de luz, fechamento emergencial, erro operacional.

```
Pedido em andamento → gerente cancela → cozinha para → cliente recebe estorno
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👨‍🍳 Cozinha | Pedido está em `ACEITO` ou `EM_PREPARO` |
| 2 | 🔑 Gerente | Acessa o painel de gerenciamento de pedidos |
| 3 | 🔑 Gerente | Localiza o pedido e clica em **Cancelar pedido** (acesso exclusivo do gerente) |
| 4 | 🔑 Gerente | Confirma o cancelamento |
| 5 | Sistema | Cancela o pedido e **estorna o pagamento** automaticamente |
| 6 | 👨‍🍳 Cozinha | O pedido some da tela da cozinha |
| 7 | 👤 Cliente | Recebe notificação: **"Seu pedido foi cancelado pela loja. Pagamento estornado."** |
| 8 | 👤 Cliente | Vê o pedido como "Cancelado" no histórico com o estorno registrado |

> **Impacto:** Alto. Pagamento é devolvido. Ação registrada na auditoria (quem cancelou, quando, estado anterior).
