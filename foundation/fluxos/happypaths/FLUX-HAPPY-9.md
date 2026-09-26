# FLUX-HAPPY-9
### Cliente cancela pedido antes do preparo

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente

```
Cliente muda de ideia logo após o pedido → cancela → pagamento é estornado
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Entra em **"Meus pedidos"** e abre o pedido recém-feito |
| 2 | 👤 Cliente | Vê o botão **Cancelar** (ainda disponível pois a cozinha não iniciou o preparo) |
| 3 | 👤 Cliente | Clica em **Cancelar pedido** e confirma |
| 4 | Sistema | Cancela o pedido e estorna o pagamento automaticamente |
| 5 | 👤 Cliente | Vê a mensagem: **"Pedido cancelado. Pagamento estornado."** |
| 6 | 👨‍🍳 Cozinha | O pedido some da fila da cozinha |

> 💡 O cancelamento só está disponível até a cozinha **aceitar e iniciar o preparo**. Depois disso, o botão desaparece.
