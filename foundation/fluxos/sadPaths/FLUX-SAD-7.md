# FLUX-SAD-7
### Pagamento falha no checkout

**Gravidade:** 🔴 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente
**Causa:** Simulação de pagamento falha (representa falha de gateway real).

```
Cliente finaliza → pagamento recusado → nenhum pedido é criado → cliente tenta de novo
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Clica em **Finalizar Pedido** |
| 2 | Sistema | Tenta confirmar o pagamento |
| 3 | Sistema | Pagamento **falha** (erro na simulação) |
| 4 | Sistema | **Rollback completo** — nenhum pedido é criado, nenhum registro fica pela metade |
| 5 | 👤 Cliente | Vê: **"Não foi possível processar o pagamento. Tente novamente."** |
| 6 | 👤 Cliente | Carrinho continua intacto — pode tentar finalizar de novo |

> **Impacto:** Alto, mas **seguro**. O "tudo ou nada" garante: sem pagamento confirmado, sem pedido criado. Nunca haverá pedido "fantasma" sem pagamento.
