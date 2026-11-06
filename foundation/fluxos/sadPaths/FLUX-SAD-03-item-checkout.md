# FLUX-SAD-03-item-checkout
### Checkout recusado no servidor — item indisponível

**Gravidade:** 🔴 · **Perfis:** 👤 🔑

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente · 🔑 Gerente (causador, se pausou/bloqueou item)  
**Causa:** Entre montar o carrinho e clicar **Finalizar**, o item deixou de estar **`DISPONIVEL`** (pausa, bloqueio, arquivamento ou exclusão).

```
Cliente clica "Finalizar" → servidor revalida tudo → recusa atômica → nada cobrado
```

O checkout **sempre** revalida no commit (não confia só no que o app mostrou segundos antes). **Não** há validação de estoque nem limite diário por produto — ver [DB-02 § 1.2](../../docTecnica/DB-02-disponibilidade-itens.md).

| Validação | Se falhar | Código HTTP / negócio |
|-----------|-----------|------------------------|
| Loja `ABERTA` **ou** `shoppingSessionId` válido | Sem direito de checkout | `STORE_CLOSED` |
| Item `DISPONIVEL` (não pausado/bloqueado/arquivado) | Linha inválida | **`ITEM_UNAVAILABLE`** |
| Pagamento simulado | Rollback | `PAYMENT_FAILED` — [FLUX-SAD-06-pagamento](FLUX-SAD-06-pagamento.md) |

Qualquer falha → **transação inteira revertida** — nenhum pedido, nenhuma cobrança.

---

## Cenário principal — item pausado no último segundo

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Monta o carrinho com X-Burger e Coca-Cola |
| 2 | 👤 Cliente | Clica em **Finalizar Pedido** |
| 3 | 🔑 Gerente | No mesmo instante, pausou o X-Burger (`PAUSADO_DIA` ou `BLOQUEADO`) |
| 4 | Sistema | Detecta que o X-Burger não está `DISPONIVEL` |
| 5 | Sistema | **Cancela a operação inteira** — HTTP **409** · **`ITEM_UNAVAILABLE`** |
| 6 | 👤 Cliente | Vê: **"Não foi possível finalizar. X-Burger ficou indisponível. Revise seu carrinho."** |
| 7 | 👤 Cliente | Volta ao carrinho, remove ou ajusta a linha e tenta de novo |

---

> **Impacto:** Alto na experiência, mas **seguro financeiramente**. O cliente não paga por produto que a loja não pode honrar.

> ⚠️ **Checkout atômico:** ou tudo passa na mesma transação (pedido + pagamento simulado), ou nada persiste.

---

> 📌 **Nota técnica resumida:**  
> `POST /orders` com `Idempotency-Key`. Ordem sugerida na transação: validar disponibilidade e regra de loja/sessão → simular pagamento → gravar pedido e `order_items`. Problem Details RFC 7807 com `type`/`code` por motivo.  
> → [`../../docTecnica/DB-02-disponibilidade-itens.md`](../../docTecnica/DB-02-disponibilidade-itens.md) · [`../../pitch_delivery.md`](../../pitch_delivery.md)
