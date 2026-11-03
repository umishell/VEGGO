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
| 2 | 🔑 Gerente | Pausa para o dia ou bloqueia o X-Burger |
| 3 | 👤 Cliente | Volta ao carrinho |
| 4 | Sistema | Mantém a linha e a pinta de cinza, com o mesmo texto do cardápio |
| 5 | 👤 Cliente | Lê o aviso e **não** consegue finalizar enquanto essa linha estiver no carrinho |
| 6 | 👤 Cliente | Remove a linha e segue com o resto, ou escolhe outro item |

Se o gerente **apagou** o produto, a linha sai do carrinho com o aviso **"Este produto foi removido do cardápio."**

> **Impacto:** Médio. Nada é cobrado pelo item indisponível. A linha cinza explica o motivo; o sistema não apaga a pausa em silêncio.

---

## Limite diário esgotado (item ainda `DISPONIVEL`)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Tinha 3× X-Burger no carrinho quando o **restante do dia** era 3 |
| 2 | Sistema | Outros pedidos ou redução do gerente deixam **restante menor que a quantidade** no carrinho |
| 3 | Sistema | App limita **+** no carrinho ao restante; linha pode exibir *"Restam N unidades hoje"* |
| 4 | 👤 Cliente | Se ainda tentar finalizar com quantidade inválida, o servidor recusa com **`DAILY_LIMIT_EXCEEDED`** — [FLUX-SAD-24-limite-dia](FLUX-SAD-24-limite-dia.md) · [FLUX-SAD-03-item-checkout](FLUX-SAD-03-item-checkout.md) |
