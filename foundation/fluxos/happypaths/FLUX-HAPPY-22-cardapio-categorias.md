# FLUX-HAPPY-22-cardapio-categorias
### Cliente navega o cardápio por categorias

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente  
**Veja também:** [FLUX-HAPPY-03-carrinho](FLUX-HAPPY-03-carrinho.md) · [DB-04-catalogo-checkout-cliente](../../docTecnica/DB-04-catalogo-checkout-cliente.md) § 1

```
Login → cardápio em colunas/seções → adiciona quantidade → carrinho
```

---

## Seções do cardápio (MVP)

O app lista produtos **disponíveis para pedido** (`DISPONIVEL`) agrupados nesta ordem (seed DB-04):

| Seção | Exemplos |
|-------|----------|
| Hambúrgueres tradicionais | Clássicos da casa |
| Hambúrgueres premium | Linha especial |
| Hambúrgueres vegetarianos | Sem carne animal |
| Hambúrgueres veganos | 100% plant-based |
| Acompanhamentos | Porção de batata **para uma pessoa** |
| Adicionais — hambúrguer | Extra proteína, queijo etc. |
| Adicionais — molhos | Molhos extras |
| Porções | Porções compartilhadas |
| Doces | Sobremesas |
| Salgados | Salgados |
| Bebidas | Refrigerantes, sucos |

Itens `PAUSADO_DIA` / `BLOQUEADO` aparecem **cinza** na seção (não entram no carrinho). `ARQUIVADO` não aparece.

---

## Interação

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | App carrega cardápio via `GET /menu` ([INF-02 § 11.1](../../docTecnica/INF-02-http-vs-websocket.md)) |
| 2 | 👤 Cliente | Rola ou alterna entre seções |
| 2 | 👤 Cliente | Em cada produto, define **quantidade** e toca **Adicionar** |
| 3 | Sistema | Atualiza badge do carrinho e subtotal local |
| 4 | 👤 Cliente | Abre o **carrinho** para revisar ([FLUX-HAPPY-03-carrinho](FLUX-HAPPY-03-carrinho.md)) |

Quantidade máxima respeita **limite diário restante** quando houver teto (DB-02 § 1.2).
