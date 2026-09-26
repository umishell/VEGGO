# FLUX-SAD-4
### Cliente tenta cancelar após preparo iniciado

**Gravidade:** 🟠 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente · 👨‍🍳 Cozinha
**Causa:** A janela de cancelamento se fechou — o lanche já está sendo feito.

```
Cliente quer cancelar → botão não existe mais → sistema explica o porquê
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Faz o pedido |
| 2 | 👨‍🍳 Cozinha | Aceita o pedido e clica em **Iniciar preparo** |
| 3 | 👤 Cliente | Muda de ideia e abre a tela do pedido para cancelar |
| 4 | Sistema | O botão **Cancelar** não aparece mais (o preparo já começou) |
| 5 | 👤 Cliente | Vê a mensagem: **"Cancelamento não disponível — seu pedido já está sendo preparado."** |
| 6 | 👤 Cliente | Não consegue cancelar; o pedido segue normalmente até a entrega |

> **Impacto:** Médio. Cliente fica frustrado, mas a loja não pode desmontar um lanche pela metade.
>
> 💡 **Regra de negócio:** cancelamento só é permitido enquanto o pedido está em `RECEBIDO` ou `ACEITO` (cozinha aceitou mas não iniciou). Após `EM_PREPARO`, a opção desaparece.
