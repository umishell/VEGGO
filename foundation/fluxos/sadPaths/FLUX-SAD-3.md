# FLUX-SAD-3
### Item pausado no checkout — servidor recusa

**Gravidade:** 🔴 · **Perfis:** 👤 🔑

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente · 🔑 Gerente (causador)
**Causa:** Entre o clique em "Finalizar" e o servidor processar, o gerente pausou o item. Caso raro mas real.

```
Cliente clica "Finalizar" → servidor detecta item indisponível → recusa e explica
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Monta o carrinho com X-Burger e Coca-Cola |
| 2 | 👤 Cliente | Clica em **Finalizar Pedido** |
| 3 | 🔑 Gerente | No mesmo instante, pausou o X-Burger no painel |
| 4 | Sistema | Ao processar o pedido, verifica que X-Burger está pausado |
| 5 | Sistema | **Cancela a operação inteira** — nenhum pedido é criado, nenhum valor é cobrado |
| 6 | 👤 Cliente | Vê a mensagem: **"Não foi possível finalizar. X-Burger ficou indisponível. Revise seu carrinho."** |
| 7 | 👤 Cliente | Volta ao carrinho, remove o X-Burger e pode tentar de novo |

> **Impacto:** Alto na experiência, mas **seguro financeiramente**. O cliente não paga por produto que não pode receber.
>
> ⚠️ Isso é o **checkout atômico** em ação: ou tudo funciona, ou nada é cobrado.
