# FLUX-HAPPY-11-item-cozinha
### Cozinha pausa ou bloqueia um item

**Perfis:** 👨‍🍳

[← Índice happy path](README.md)

---

**Perfis:** 👨‍🍳 Cozinha  
**Veja também:** [FLUX-HAPPY-10-item-gerente](FLUX-HAPPY-10-item-gerente.md) — o gerente faz as mesmas pausas e é o único que apaga o produto.  
**Veja também:** [FLUX-SAD-13-loja-emergencia](../sadPaths/FLUX-SAD-13-loja-emergencia.md) — fechar a loja inteira é só com o gerente.

A cozinha não recusa pedido que já entrou na fila. Se um ingrediente acabou, ela impede o **próximo** pedido daquele item.

```
Ingrediente acaba → cozinha pausa o dia ou bloqueia o item
→ cliente vê item cinza → pedido novo daquele item não entra
```

> A cozinha **não tem** as ações Arquivar, Desarquivar ou Remover produto — essas são exclusivas do gerente. Arquivar oculta o item completamente das interfaces; pausar/bloquear apenas o torna cinza no cardápio do cliente. Ver [FLUX-HAPPY-10-item-gerente](FLUX-HAPPY-10-item-gerente.md).

---

## Pausar para o dia

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👨‍🍳 Cozinha | Em **Disponibilidade**, escolhe o item e clica em **Pausar para o dia** |
| 2 | Sistema | Grava `PAUSADO_DIA` |
| 3 | 👤 Cliente | Vê o item cinza: **"Não está sendo produzido hoje."** |
| 4 | 🔑 Gerente | Vê o mesmo estado no painel |
| 5 | Sistema | Na meia-noite volta a `DISPONIVEL` |
| 6 | 👨‍🍳 Cozinha ou 🔑 Gerente | Qualquer um dos dois pode reativar antes |

---

## Bloquear temporariamente

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👨‍🍳 Cozinha | Clica em **Bloquear temporariamente** |
| 2 | Sistema | Grava `BLOQUEADO` |
| 3 | 👤 Cliente | Vê o item cinza: **"Indisponível no momento."** |
| 4 | Sistema | A meia-noite não reativa |
| 5 | 👨‍🍳 Cozinha ou 🔑 Gerente | Reativa manualmente quando puder produzir de novo |

---

> A cozinha não apaga produto e não fecha a loja. Pedido que já está na fila dela segue até a entrega, mesmo que o item seja pausado depois.

---

> 📌 **Nota técnica resumida:**  
> Mesma coluna `products.availability` do gerente.  
> → [`../../docTecnica/DB-02-disponibilidade-itens.md`](../../docTecnica/DB-02-disponibilidade-itens.md)
