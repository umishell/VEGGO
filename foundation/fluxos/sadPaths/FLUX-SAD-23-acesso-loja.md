# FLUX-SAD-23-acesso-loja
### Motoboy tenta pedido de outra loja

**Gravidade:** 🟠 · **Perfis:** 🛵

[← Índice sad path](README.md)

---

**Perfis:** 🛵 Motoboy
**Causa:** Motoboy está cadastrado em uma loja e tenta manipular a fila de outra.

```
Motoboy tenta assumir pedido fora da sua loja → sistema bloqueia
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Tenta acessar ou assumir pedido de uma loja diferente da qual está vinculado |
| 2 | Sistema | Verifica se o pedido pertence à loja do motoboy |
| 3 | Sistema | **Pedido de outra loja — acesso negado** |
| 4 | 🛵 Motoboy | Recebe: **"Você não tem permissão para este pedido."** |
| 5 | Sistema | Responde **403** |

> **Impacto:** Nenhum operacional. A autorização por atributo (FGAC: `lojaId` do motoboy vs `lojaId` do pedido) garante o isolamento entre lojas — mesmo com uma única instância do sistema.
