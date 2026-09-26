# FLUX-SAD-1
### Item indisponível no cardápio

**Gravidade:** 🟡 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente
**Causa:** Gerente pausou o produto **ou** cozinha marcou como esgotado hoje.

```
Item some do cardápio → cliente nem chega a vê-lo (nem tenta comprar)
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente / 👨‍🍳 Cozinha | Marca produto como pausado ou esgotado |
| 2 | Sistema | Remove o item da listagem pública do cardápio |
| 3 | 👤 Cliente | Abre o cardápio — **não vê o item** na lista |
| 4 | 👤 Cliente | Não há frustração de clicar e não conseguir comprar — o item simplesmente não está lá |

> **Impacto:** Mínimo. O cliente não sabe que o item existe (ou sabe que acabou). Não há mensagem de erro — o produto simplesmente não está listado.

**Resolução:** Gerente reativa o produto quando disponível. Cozinha: reseta automaticamente na meia-noite.
