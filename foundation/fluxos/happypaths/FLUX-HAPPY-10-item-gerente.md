# FLUX-HAPPY-10-item-gerente
### Gerente muda a disponibilidade de um item

**Perfis:** 🔑

[← Índice happy path](README.md)

---

**Perfis:** 🔑 Gerente  
**Veja também:** [FLUX-HAPPY-11-item-cozinha](FLUX-HAPPY-11-item-cozinha.md) — a cozinha faz as duas pausas, sem apagar o produto.

```
Gerente escolhe o item → pausa o dia, bloqueia, reativa ou apaga
→ cliente vê cinza, ou o item some de vez
```

O item regular **não some** do cardápio quando fica indisponível. Fica cinza, com texto, e não entra no carrinho. Continua na lista da cozinha e na lista do gerente.

---

## Pausar para o dia — `PAUSADO_DIA`

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Abre **Cardápio** e escolhe o item |
| 2 | 🔑 Gerente | Clica em **Pausar para o dia** |
| 3 | Sistema | Grava `PAUSADO_DIA` |
| 4 | 👤 Cliente | Vê o item cinza: **"Não está sendo produzido hoje."** Não consegue pedir |
| 5 | 👨‍🍳 Cozinha | Vê o mesmo estado na lista dela |
| 6 | Sistema | Na meia-noite o item volta a `DISPONIVEL` |
| 7 | 🔑 Gerente | Pode reativar antes, em um clique |

---

## Bloquear até alguém reativar — `BLOQUEADO`

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Clica em **Bloquear temporariamente** |
| 2 | Sistema | Grava `BLOQUEADO` |
| 3 | 👤 Cliente | Vê o item cinza: **"Indisponível no momento."** Não consegue pedir |
| 4 | Sistema | A meia-noite **não** muda esse estado |
| 5 | 🔑 Gerente ou 👨‍🍳 Cozinha | Só volta a `DISPONIVEL` quando um dos dois reativa |

---

## Apagar — some do banco

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Clica em **Remover produto** e confirma |
| 2 | Sistema | Apaga a linha do produto |
| 3 | 👤 Cliente | O item deixa de existir na lista. Não fica cinza |
| 4 | 👨‍🍳 Cozinha | Também deixa de ver o item |
| 5 | Sistema | Pedidos antigos continuam legíveis: o nome e o preço foram copiados na hora da compra |

A cozinha não tem essa ação.

---

> 📌 **Nota técnica resumida:**  
> Coluna `products.availability`. Job da meia-noite só desfaz `PAUSADO_DIA`.  
> → [`../../docTecnica/DB-02-disponibilidade-itens.md`](../../docTecnica/DB-02-disponibilidade-itens.md)
