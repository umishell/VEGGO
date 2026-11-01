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
| 1 | 👤 Cliente | No cardápio, toca em **X-Burger** e clica **Adicionar ao carrinho** |
| 2 | Sistema | Adiciona o item com quantidade 1. O ícone flutuante do carrinho aparece com o total |
| 3 | 👤 Cliente | Navega até **Bebidas**, toca em **Coca-Cola** e clica **Adicionar** |
| 4 | Sistema | Adiciona a bebida. Carrinho exibe "2 itens" e o subtotal atualizado |

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
| 12 | 👤 Cliente | **Fecha o app** com X-Burger (×2) no carrinho |
| 13 | 👤 Cliente | Reabre o app minutos depois |
| 14 | Sistema | Restaura o carrinho exatamente como estava — itens e quantidades |
| 15 | 👤 Cliente | Continua de onde parou, direto para o checkout |

> 💡 O carrinho fica salvo enquanto a sessão do cliente está ativa. Fazer **logout** descarta o carrinho.

---

**Veja também:**  
→ [FLUX-HAPPY-05-venda](FLUX-HAPPY-05-venda.md) — checkout completo  
→ [FLUX-SAD-02-item-carrinho](../sadPaths/FLUX-SAD-02-item-carrinho.md) — item do carrinho fica indisponível antes do checkout  
→ [FLUX-SAD-04-carrinho-sessao](../sadPaths/FLUX-SAD-04-carrinho-sessao.md) — o que acontece ao fazer logout com itens no carrinho
