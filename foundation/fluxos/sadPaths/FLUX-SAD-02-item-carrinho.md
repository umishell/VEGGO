# FLUX-SAD-02-item-carrinho
### Item do carrinho fica indisponível antes do checkout

**Gravidade:** 🟠 · **Perfis:** 👤 🔑

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente · 🔑 Gerente ou 👨‍🍳 Cozinha  
**Causa:** O item entrou no carrinho `DISPONIVEL` e depois foi para `PAUSADO_DIA` ou `BLOQUEADO`.

```
Item no carrinho → alguém pausa ou bloqueia → linha fica cinza → checkout daquele item trava
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Adiciona o X-Burger ao carrinho |
| 2 | 🔑 Gerente ou 👨‍🍳 Cozinha | Pausa para o dia ou bloqueia o X-Burger |
| 3 | 👤 Cliente | Volta ao carrinho |
| 4 | Sistema | Mantém a linha e a pinta de cinza, com o mesmo texto do cardápio |
| 5 | 👤 Cliente | Lê o aviso e **não** consegue finalizar enquanto essa linha estiver no carrinho |
| 6 | 👤 Cliente | Remove a linha e segue com o resto, ou escolhe outro item |

Se o gerente **apagou** o produto, a linha sai do carrinho com o aviso **"Este produto foi removido do cardápio."**

> **Impacto:** Médio. Nada é cobrado pelo item indisponível. A linha cinza explica o motivo; o sistema não apaga a pausa em silêncio.
