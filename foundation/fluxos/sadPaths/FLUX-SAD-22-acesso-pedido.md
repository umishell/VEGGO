# FLUX-SAD-22-acesso-pedido
### Cliente tenta acessar pedido alheio

**Gravidade:** 🔴 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente (mal-intencionado ou por erro)
**Causa:** Tentativa de acessar URL de pedido de outro usuário.

```
Cliente tenta ver pedido que não é dele → sistema bloqueia → erro de permissão
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Tenta acessar diretamente a página de um pedido digitando um ID diferente na URL |
| 2 | Sistema | Verifica se o pedido pertence ao cliente logado |
| 3 | Sistema | **Pedido não pertence a este usuário — acesso negado** |
| 4 | 👤 Cliente | Vê: **"Pedido não encontrado"** (sem revelar que o pedido existe) |
| 5 | Sistema | Registra a tentativa de acesso indevido na auditoria |

> **Impacto:** Nenhum para o outro cliente. A segurança (FGAC) garante que cada usuário vê **apenas seus próprios pedidos**, mesmo que tente acessar IDs alheios diretamente.
