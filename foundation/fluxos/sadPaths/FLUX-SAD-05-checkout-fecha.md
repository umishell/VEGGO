# FLUX-SAD-05-checkout-fecha
### Cliente fecha app no meio do checkout

**Gravidade:** 🟡 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente
**Causa:** App fechado, internet caiu ou cliente voltou antes de confirmar.

```
Cliente estava na tela de checkout → fechou o app → nenhum pedido foi criado
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Estava na tela de revisão do pedido (ainda não clicou em "Finalizar") |
| 2 | 👤 Cliente | Fecha o app ou perde a conexão |
| 3 | Sistema | **Nada é criado** — o pedido só existe após clicar "Finalizar" |
| 4 | 👤 Cliente | Reabre o app — vê o **carrinho intacto** com os mesmos itens |
| 5 | 👤 Cliente | Pode continuar de onde parou e finalizar normalmente |

> **Impacto:** Nenhum. O cliente não perde nada, nenhuma cobrança ocorre. O checkout só "acontece" quando o servidor confirma — fechar o app antes não tem efeito.
