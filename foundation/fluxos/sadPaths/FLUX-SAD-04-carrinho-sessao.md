# FLUX-SAD-04-carrinho-sessao

### Cliente sai do app com itens no carrinho

**Gravidade:** 🟡 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente
**Causa:** Interrupção comum do dia a dia — ligação, notificação, etc.

```
Carrinho montado → app fechado → app reaberto → carrinho salvo (se sessão ativa)
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Adiciona X-Burger e Coca ao carrinho |
| 2 | 👤 Cliente | Fecha o app sem finalizar |
| 3 | Sistema | Mantém o carrinho salvo na sessão ativa do cliente |
| 4 | 👤 Cliente | Reabre o app — vê o carrinho com os mesmos itens |
| 5 | 👤 Cliente | Finaliza normalmente |

**Cenário com logout:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Adiciona itens ao carrinho |
| 2 | 👤 Cliente | Faz logout do app |
| 3 | Sistema | **Descarta o carrinho** — sessão encerrada |
| 4 | 👤 Cliente | Faz login novamente — carrinho está vazio |
| 5 | 👤 Cliente | Precisa montar o carrinho de novo |

> **Resumo:**
>
> | Situação | Carrinho salvo? |
> |----------|-----------------|
> | Fechou o app (sessão ativa) | ✅ Sim |
> | Ficou sem internet | ✅ Sim |
> | Fez logout | ❌ Não |
> | Sessão expirou | ❌ Não |
