# FLUX-HAPPY-10-item-gerente
### Gerente muda a disponibilidade de um item

**Perfis:** 🔑

[← Índice happy path](README.md)

---

**Perfis:** 🔑 Gerente  
**Veja também:** [FLUX-HAPPY-11-item-cozinha](FLUX-HAPPY-11-item-cozinha.md) — a cozinha faz as duas pausas, sem apagar o produto.

```
Gerente escolhe o item → pausa o dia, bloqueia, arquiva, reativa ou apaga
→ pausado/bloqueado: cliente vê cinza
→ arquivado: some da vista do cliente e da cozinha, mas continua no banco
→ apagado: some do banco definitivamente
```

Itens `PAUSADO_DIA` e `BLOQUEADO` **ficam cinza** no cardápio do cliente — o cliente vê que o produto existe, mas não pode pedir. Continuam visíveis na lista da cozinha e do gerente. `ARQUIVADO` **não aparece** para o cliente nem para a cozinha — só o gerente acessa na seção "Itens arquivados".

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

## Arquivar — some da vista, mas fica no banco

Usado quando o gerente quer **ocultar** o item temporária ou indefinidamente sem perdê-lo. O produto continua cadastrado e pode ser reativado a qualquer momento.

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Clica em **Arquivar produto** e confirma |
| 2 | Sistema | Grava `ARQUIVADO`. O item sai da lista do cliente e da lista da cozinha |
| 3 | 👤 Cliente | O produto **não aparece mais** no cardápio — sem cinza, sem texto |
| 4 | 👨‍🍳 Cozinha | Também deixa de ver o item na lista de disponibilidade |
| 5 | 🔑 Gerente | Vê o item na seção **"Itens arquivados"** — pode **Desarquivar** (volta a `DISPONIVEL`) ou **Remover definitivamente** |

> 💡 Use **Arquivar** para produtos sazonais, temporariamente fora de estoque de ingrediente base, ou que estão sendo reformulados. Assim o produto não vaza para o cliente como "cinza" e pode ser reativado sem precisar recadastrar tudo.

---

## Desarquivar — volta ao cardápio como `DISPONIVEL`

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Na seção "Itens arquivados", clica em **Desarquivar** |
| 2 | Sistema | Grava `DISPONIVEL`. O item volta a aparecer normalmente no cardápio e na lista da cozinha |
| 3 | 👤 Cliente | Vê o produto disponível |

---

## Apagar — exclusão física permanente

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Clica em **Remover produto** e confirma (disponível tanto para itens ativos quanto arquivados) |
| 2 | Sistema | Apaga a linha do produto do banco — **irreversível** |
| 3 | 👤 Cliente | O item deixa de existir. Não fica cinza |
| 4 | 👨‍🍳 Cozinha | Também deixa de ver o item |
| 5 | Sistema | Pedidos antigos continuam legíveis: o nome e o preço foram copiados na hora da compra |

> ⚠️ Use **Remover** somente para produtos que não voltarão mais. Para ocultar temporariamente, prefira **Arquivar**.

A cozinha não tem as ações Arquivar, Desarquivar nem Remover.

---

> 📌 **Nota técnica resumida:**  
> Coluna `products.availability` (enum: `DISPONIVEL`, `PAUSADO_DIA`, `BLOQUEADO`, `ARQUIVADO`). Job da meia-noite só desfaz `PAUSADO_DIA`. `ARQUIVADO` é filtrado nas queries do cardápio do cliente e da lista da cozinha (`WHERE availability != 'ARQUIVADO'`). Gerente usa `WHERE availability = 'ARQUIVADO'` na seção de itens arquivados.  
> → [`../../docTecnica/DB-02-disponibilidade-itens.md`](../../docTecnica/DB-02-disponibilidade-itens.md)
