# FLUX-SAD-03-item-checkout
### Checkout recusado no servidor — item indisponível ou limite do dia

**Gravidade:** 🔴 · **Perfis:** 👤 🔑

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente · 🔑 Gerente (causador, se pausou item)
**Causa:** Entre montar o carrinho e clicar **Finalizar**, a regra no servidor mudou — item deixou de estar vendável **ou** a quantidade pedida não cabe no **restante diário**.

```
Cliente clica "Finalizar" → servidor revalida tudo → recusa atômica → nada cobrado
```

O checkout **sempre** revalida no commit (não confia só no que o app mostrou segundos antes):

| Validação | Se falhar | Código HTTP / negócio |
|-----------|-----------|------------------------|
| Loja `ABERTA` | Pedido bloqueado | `STORE_CLOSED` |
| Item `DISPONIVEL` (não pausado/bloqueado) | Linha inválida | **`ITEM_UNAVAILABLE`** |
| Soma das quantidades ≤ **restante diário** (`product_daily_caps`) | Teto do dia | **`DAILY_LIMIT_EXCEEDED`** — ver [FLUX-SAD-24-limite-dia](FLUX-SAD-24-limite-dia.md) |
| Pagamento simulado | Rollback | `PAYMENT_FAILED` — [FLUX-SAD-06-pagamento](FLUX-SAD-06-pagamento.md) |

Qualquer falha → **transação inteira revertida** — nenhum pedido, nenhuma cobrança, `units_sold` não incrementa.

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

## Cenário relacionado — limite diário

Se o item ainda está `DISPONIVEL` mas **não há restante** para a quantidade do carrinho, o servidor responde **`DAILY_LIMIT_EXCEEDED`** (mensagem distinta). Passo a passo completo: [FLUX-SAD-24-limite-dia](FLUX-SAD-24-limite-dia.md).

---

> **Impacto:** Alto na experiência, mas **seguro financeiramente**. O cliente não paga por produto que a loja não pode honrar no dia.

> ⚠️ **Checkout atômico:** ou tudo passa na mesma transação (pedido + pagamento simulado + caps), ou nada persiste.

---

> 📌 **Nota técnica resumida:**  
> `POST /orders` com `Idempotency-Key`. Ordem sugerida na transação: lock caps → validar availability e remaining → simular pagamento → gravar pedido → incrementar `units_sold`. Problem Details RFC 7807 com `type`/`code` por motivo.  
> → [`../../docTecnica/DB-02-disponibilidade-itens.md`](../../docTecnica/DB-02-disponibilidade-itens.md) § 1.2 · [`../../pitch_delivery.md`](../../pitch_delivery.md)
