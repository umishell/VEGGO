# FLUX-HAPPY-10-item-gerente
### Gerente muda a disponibilidade de um item

**Perfis:** 🔑

[← Índice happy path](README.md)

---

**Perfis:** 🔑 Gerente  
**Veja também:** [FLUX-HAPPY-11-item-cozinha](FLUX-HAPPY-11-item-cozinha.md) — cozinha opera só o KDS; disponibilidade de item é **só aqui (gerente)**.

```
Gerente escolhe o item → pausa o dia, bloqueia, arquiva, reativa ou apaga
→ pausado/bloqueado: cliente vê cinza
→ arquivado: some da vista do cliente e da cozinha, mas continua no banco
→ apagado: some do banco definitivamente
```

Itens `PAUSADO_DIA` e `BLOQUEADO` **ficam cinza** no cardápio do cliente. Continuam visíveis no **Cardápio** do gerente. `ARQUIVADO` **não aparece** para o cliente — só o gerente na seção "Itens arquivados" (`GET /store/products?archivedOnly=true`).

Na lista **Cardápio**, o gerente filtra por **categoria** (combobox) e busca por nome — [INF-04 § 4](../../docTecnica/INF-04-ui-operacao-cliente.md) · `GET /store/products?categoryId=…` ([INF-02 § 11.2](../../docTecnica/INF-02-http-vs-websocket.md)).

---

## Limite diário de pedidos (não é estoque)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | No CRUD do item, define **Limite padrão por dia** (`default_daily_order_limit`) — quantas unidades podem ser vendidas por dia operacional; renova sozinho à meia-noite |
| 2 | 🔑 Gerente | Na operação do dia, vê **vendido / restante / limite efetivo de hoje** |
| 3 | 🔑 Gerente | Pode **aumentar ou reduzir só hoje** — `PATCH /store/products/{id}/daily-cap` (`effective_limit`) — ex.: sobrou insumo → sobe o teto; estragou faltou → baixa sem mudar o padrão de amanhã |
| 4 | 👤 Cliente | No cardápio/carrinho, quantidade máxima respeita o **restante**; checkout recusa se estourar (409) |
| 5 | 👨‍🍳 Cozinha | Avisa o gerente se faltou insumo — **não** altera limite nem disponibilidade no sistema |
| 6 | 👨‍🍳 → 🔑 | Se **não dá** para produzir um item (falta ingrediente), cozinha **avisa o gerente**; o gerente aplica **Pausar para o dia** (`PAUSADO_DIA`) — cinza, sem depender só de zerar o limite numérico |

> Limite diário **conta pedidos**, não grama/litro em estoque. Ver DB-02 § 1.2. Checkout recusado: [FLUX-SAD-24-limite-dia](../sadPaths/FLUX-SAD-24-limite-dia.md).

---

## Pausar para o dia — `PAUSADO_DIA`

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Abre **Cardápio** e escolhe o item |
| 2 | 🔑 Gerente | Clica em **Pausar para o dia** |
| 3 | Sistema | Grava `PAUSADO_DIA` |
| 4 | 👤 Cliente | Vê o item cinza: **"Não está sendo produzido hoje."** Não consegue pedir |
| 5 | 👤 Cliente | Vê o item cinza no app |
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
| 5 | 🔑 Gerente | Reativa manualmente quando puder produzir de novo |

---

## Arquivar — some da vista, mas fica no banco

Usado quando o gerente quer **ocultar** o item temporária ou indefinidamente sem perdê-lo. O produto continua cadastrado e pode ser reativado a qualquer momento.

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Clica em **Arquivar produto** e confirma |
| 2 | Sistema | Grava `ARQUIVADO`. O item sai da lista do cliente e da lista da cozinha |
| 3 | 👤 Cliente | O produto **não aparece mais** no cardápio — sem cinza, sem texto |
| 4 | 👤 Cliente | Deixa de ver o item (oculto) |
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
| 4 | — | Cozinha (KDS) não lista cardápio — item já não entra em novos pedidos |
| 5 | Sistema | Pedidos antigos continuam legíveis: o nome e o preço foram copiados na hora da compra |

> ⚠️ Use **Remover** somente para produtos que não voltarão mais. Para ocultar temporariamente, prefira **Arquivar**.

A cozinha não tem as ações Arquivar, Desarquivar nem Remover.

---

> 📌 **Nota técnica resumida:**  
> Coluna `products.availability` (enum: `DISPONIVEL`, `PAUSADO_DIA`, `BLOQUEADO`, `ARQUIVADO`). Job da meia-noite só desfaz `PAUSADO_DIA`. Cliente: `GET /menu` exclui `ARQUIVADO`. Gerente: `GET /store/products?archivedOnly=true` na seção arquivados; demais ações via `PATCH /store/products/{id}`.  
> → [DB-02 § 1](../../docTecnica/DB-02-disponibilidade-itens.md) · [INF-02 § 11](../../docTecnica/INF-02-http-vs-websocket.md) · Rodapé estatísticas: [DB-03 § 6.2](../../docTecnica/DB-03-telemetria.md) · [INF-04 § 5](../../docTecnica/INF-04-ui-operacao-cliente.md)
