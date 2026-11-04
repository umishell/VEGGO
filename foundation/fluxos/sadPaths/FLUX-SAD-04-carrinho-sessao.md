# FLUX-SAD-04-carrinho-sessao

### Carrinho — persistência, TTL 3 h e logout

**Gravidade:** 🟡 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente  
**Veja também:** [FLUX-HAPPY-03-carrinho](../happypaths/FLUX-HAPPY-03-carrinho.md) · [DB-04 § 6–7](../../docTecnica/DB-04-catalogo-checkout-cliente.md)

```
Monta carrinho → fecha ou mata o app → volta logado → carrinho intacto (até 3 h)
Logout → carrinho zerado
3 h desde o primeiro item → carrinho limpo automaticamente
```

---

## Passo a passo — fechar app

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Adiciona itens ao carrinho (grava `cartStartedAt`) |
| 2 | 👤 Cliente | Fecha o app ou reinicia o telefone |
| 3 | Sistema | Persiste carrinho no dispositivo (DataStore) |
| 4 | 👤 Cliente | Reabre dentro de **3 h**, ainda logado |
| 5 | Sistema | Restaura itens e quantidades; mantém `shoppingSessionId` se válido |

---

## Logout intencional

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Toca **Sair** / logout |
| 2 | Sistema | Limpa tokens **e** carrinho local; revoga `client_shopping_sessions` |
| 3 | 👤 Cliente | Próximo login começa com carrinho vazio |

> Logout **antes** de 3 h **sempre** descarta o carrinho — não importa quantos itens havia.

---

## TTL de 3 horas

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | Sistema | Ao abrir app ou checkout, se `now > cartStartedAt + 3 h` |
| 2 | Sistema | Limpa carrinho; sessão de compra deixa de valer para checkout |
| 3 | 👤 Cliente | Vê carrinho vazio; se loja fechou entretanto, aplica [FLUX-SAD-14-loja-fechada](FLUX-SAD-14-loja-fechada.md) |

---

> **Resumo**
>
> | Situação | Carrinho |
> |----------|----------|
> | Fechou / matou app (logado) | ✅ Mantém (≤ 3 h) |
> | Sem internet | ✅ Mantém local |
> | Logout | ❌ Limpa |
> | > 3 h desde 1º item | ❌ Limpa |
> | Login expirado (~3 meses sem refresh) | ❌ Precisa login; carrinho segue regra acima se ainda no prazo |
