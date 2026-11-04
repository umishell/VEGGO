# FLUX-HAPPY-03-carrinho
### Montagem do carrinho — adicionar, ajustar e remover itens

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente

```
Cliente adiciona itens → ajusta quantidades → fecha o app → volta → carrinho ainda está lá
```

---

## Parte 1 — Adicionar itens

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | No cardápio (loja **aberta** na entrada), toca em **X-Burger** e clica **Adicionar ao carrinho** |
| 2 | Sistema | Primeiro item grava `cartStartedAt`; associa `shoppingSessionId` ([DB-04 § 6–7](../../docTecnica/DB-04-catalogo-checkout-cliente.md)); quantidade 1 no carrinho |
| 3 | 👤 Cliente | Navega até **Bebidas**, toca em **Coca-Cola** e clica **Adicionar** |
| 4 | Sistema | Carrinho exibe totais atualizados |

---

## Parte 2 — Ajustar quantidade e remover

| # | Quem | O que acontece |
|---|------|----------------|
| 5 | 👤 Cliente | Abre o carrinho e vê os itens listados com controles **−** e **+** ao lado de cada um |
| 6 | 👤 Cliente | Toca em **+** ao lado do X-Burger |
| 7 | Sistema | Quantidade vai para 2. Subtotal e total atualizam em tempo real |
| 8 | 👤 Cliente | Toca em **−** ao lado da Coca-Cola |
| 9 | Sistema | Quantidade volta para 0 → item é **removido** do carrinho automaticamente |
| 10 | 👤 Cliente | Pode também tocar no ícone 🗑️ ao lado de qualquer item para remover diretamente (independente da quantidade) |
| 11 | Sistema | Item removido. Se o carrinho ficar vazio, exibe mensagem: *"Seu carrinho está vazio"* com atalho para o cardápio |

> As alterações no carrinho são locais (sem chamada ao servidor). O servidor valida disponibilidade e preço no **checkout** — não a cada ajuste.

---

## Parte 3 — Salvar e retomar

| # | Quem | O que acontece |
|---|------|----------------|
| 12 | 👤 Cliente | **Fecha ou mata o app** com itens no carrinho |
| 13 | 👤 Cliente | Reabre dentro de **3 horas**, ainda logado |
| 14 | Sistema | Restaura carrinho e sessão de compra (se válida) |
| 15 | 👤 Cliente | Pode ir ao checkout — **mesmo se a loja fechou** nesse intervalo ([FLUX-SAD-14](../sadPaths/FLUX-SAD-14-loja-fechada.md) cenário E) |

> 💡 Carrinho persiste ao fechar app; **logout** limpa; após **3 h** desde o 1º item o carrinho zera ([FLUX-SAD-04](../sadPaths/FLUX-SAD-04-carrinho-sessao.md)). Login dura **~3 meses** (refresh token).

---

**Veja também:**  
→ [FLUX-HAPPY-05-venda](FLUX-HAPPY-05-venda.md) — checkout completo  
→ [FLUX-SAD-02-item-carrinho](../sadPaths/FLUX-SAD-02-item-carrinho.md) — item do carrinho fica indisponível antes do checkout  
→ [FLUX-SAD-04-carrinho-sessao](../sadPaths/FLUX-SAD-04-carrinho-sessao.md) — o que acontece ao fazer logout com itens no carrinho
