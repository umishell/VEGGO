# FLUX-SAD-12-cozinha-fila
### Cozinha demora a aceitar o pedido

**Gravidade:** 🟠 · **Perfis:** 👨‍🍳 🔑 👤

[← Índice sad path](README.md)

---

**Perfis:** 👨‍🍳 Cozinha · 🔑 Gerente · 👤 Cliente  
**Causa:** O pedido está `RECEBIDO` além do prazo de alerta e ninguém aceitou.

```
Pedido na fila → prazo de alerta → gerente é avisado → pedido continua na fila
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Finaliza o pedido. Estado `RECEBIDO` |
| 2 | 👨‍🍳 Cozinha | O pedido está na fila de espera |
| 3 | Sistema | Depois de `accept_alert_minutes`, avisa o gerente: **"Pedido #[número] aguardando aceite."** |
| 4 | Sistema | **Não** cancela, **não** estorna e **não** recusa |
| 5 | 👤 Cliente | Continua vendo **Pedido recebido**. O botão **Cancelar** segue disponível, porque o preparo não começou |
| 6 | 🔑 Gerente | Cobra a cozinha ou aceita ele mesmo no KDS |
| 7 | 👨‍🍳 Cozinha | Clica em **Aceitar**. O pedido vai para `EM_PREPARO` |

> Pedido que já entrou não é recusado por timeout.

---

> 📌 **Nota técnica resumida:**  
> `stores.accept_alert_minutes` só dispara aviso. O estado permanece `RECEBIDO`.  
> → [`../../docTecnica/ORD-01-estados-pedido.md`](../../docTecnica/ORD-01-estados-pedido.md)
