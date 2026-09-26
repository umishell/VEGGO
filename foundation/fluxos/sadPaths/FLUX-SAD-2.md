# FLUX-SAD-2
### Item do carrinho fica indisponível antes do checkout

**Gravidade:** 🟠 · **Perfis:** 👤 🔑

[← Índice sad path](README.md)
---

**Perfis:** 👤 Cliente · 🔑 Gerente (causador)
**Causa:** Cliente colocou item no carrinho → gerente pausou o item **depois** → cliente volta ao carrinho.

```
Item estava no carrinho → foi pausado → cliente vê aviso no carrinho
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Adiciona **X-Burger** ao carrinho |
| 2 | 🔑 Gerente | Enquanto isso, pausa o **X-Burger** no painel |
| 3 | Sistema | O X-Burger some do cardápio público |
| 4 | 👤 Cliente | Volta ao carrinho ou recarrega a página |
| 5 | Sistema | Detecta que o X-Burger foi pausado e exibe aviso no carrinho |
| 6 | 👤 Cliente | Vê: **"⚠️ X-Burger ficou indisponível e foi removido do seu carrinho"** |
| 7 | 👤 Cliente | O item é removido automaticamente — carrinho atualizado |
| 8 | 👤 Cliente | Pode escolher outro item ou finalizar sem ele |

> **Impacto:** Médio. O cliente perde o item escolhido mas o sistema avisa claramente. Não há cobrança por produto indisponível.

**Alternativa (decisão de design):** Deixar o item no carrinho com linha riscada e botão "Remover" — impede finalizar até remover.
