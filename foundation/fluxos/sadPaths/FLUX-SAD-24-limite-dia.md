# FLUX-SAD-24-limite-dia
### Limite diário do produto esgotado no checkout

**Gravidade:** 🟠 · **Perfis:** 👤 🔑

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente · 🔑 Gerente (pode liberar mais unidades)  
**Causa:** O carrinho pede mais unidades do que o **restante do dia** (`effective_limit - units_sold`) para aquele produto — concorrência entre clientes, limite baixo ou gerente reduziu o teto durante o dia.

```
Cliente finaliza → servidor valida teto diário → recusa com DAILY_LIMIT_EXCEEDED → nada cobrado
```

> Não é falta de **estoque físico** no sistema — é o **teto de pedidos confirmados no dia operacional**. Ver [DB-02 § 1.2](../../docTecnica/DB-02-disponibilidade-itens.md).

---

## Cenário A — Restante zerou antes de finalizar

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Coloca 2× X-Burger no carrinho (o app mostrava restante ≥ 2) |
| 2 | Sistema | Outros pedidos confirmados no dia consomem o restante — ou o gerente **reduz** `effective_limit` |
| 3 | 👤 Cliente | Clica em **Finalizar Pedido** |
| 4 | Sistema | No checkout atômico: `units_sold + quantity > effective_limit` → **rollback completo** |
| 5 | Sistema | HTTP **409** · código **`DAILY_LIMIT_EXCEEDED`** · mensagem com nome do produto e **restante atual** (ex.: 0 ou 1) |
| 6 | 👤 Cliente | Vê: *"Não foi possível finalizar. Limite de pedidos de hoje para X-Burger esgotado (restam 0). Ajuste o carrinho."* |
| 7 | 👤 Cliente | Reduz quantidade ou remove a linha e tenta de novo |

---

## Cenário B — Dois clientes no mesmo instante (última unidade)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente A e B | Restante do dia = **1** unidade de X-Burger |
| 2 | 👤 Cliente A | Finaliza primeiro — checkout OK, `units_sold` incrementa |
| 3 | 👤 Cliente B | Finaliza em seguida com 1× X-Burger |
| 4 | Sistema | Segundo checkout falha com **`DAILY_LIMIT_EXCEEDED`** — nenhum pedido duplicado além do limite |
| 5 | 👤 Cliente B | Ajusta o carrinho (outro item ou espera o gerente aumentar o teto) |

> Mesma ideia do lock no motoboy: validação **no commit** do checkout, com transação e checagem da linha `product_daily_caps` (ou equivalente).

---

## O que o gerente pode fazer

| Situação | Ação |
|----------|------|
| Ainda dá para produzir mais hoje | **Cardápio → Limite de hoje** — aumenta `effective_limit` (não muda o padrão de amanhã) |
| Não dá mais para produzir o item | **Pausar para o dia** (`PAUSADO_DIA`) — cliente vê cinza; ver [FLUX-HAPPY-10-item-gerente](../happypaths/FLUX-HAPPY-10-item-gerente.md) |
| Limite esgotado e operação encerrada para o item | Pausar o dia ou reduzir limite a `units_sold` |

---

**Veja também:**  
→ [FLUX-SAD-03-item-checkout](FLUX-SAD-03-item-checkout.md) — recusa por item pausado/bloqueado (`ITEM_UNAVAILABLE`)  
→ [FLUX-SAD-02-item-carrinho](FLUX-SAD-02-item-carrinho.md) — item pausado no carrinho antes do checkout  
→ [FLUX-HAPPY-10-item-gerente](../happypaths/FLUX-HAPPY-10-item-gerente.md) — limite padrão e ajuste do dia

---

> **Impacto:** Médio na experiência; **seguro** — nada é cobrado se o teto não comporta a quantidade.

---

> 📌 **Nota técnica resumida:**  
> Checkout `@Transactional`: para cada linha, `availability = DISPONIVEL` **e** `quantity <= remaining`. Falha de limite → Problem Details `409` / `DAILY_LIMIT_EXCEEDED` com `productId`, `productName`, `remainingToday`. Distinto de `ITEM_UNAVAILABLE` (pausa/bloqueio).  
> → [`../../docTecnica/DB-02-disponibilidade-itens.md`](../../docTecnica/DB-02-disponibilidade-itens.md) § 1.2
