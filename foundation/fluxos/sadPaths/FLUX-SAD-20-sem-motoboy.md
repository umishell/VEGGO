# FLUX-SAD-20-sem-motoboy
### Nenhum motoboy ativo para assumir

**Gravidade:** 🔴 · **Perfis:** 👨‍🍳 🔑

[← Índice sad path](README.md)

---

**Perfis:** 👨‍🍳 Cozinha · 🔑 Gerente · 👤 Cliente
**Causa:** Pedido ficou `PRONTO` mas todos os motoboys estão ocupados ou inativos.

```
Pedido pronto → fila vazia de motoboys → gerente gerencia a situação
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👨‍🍳 Cozinha | Marca pedido como **Pronto** |
| 2 | Sistema | Pedido entra na fila de disponíveis — mas não há motoboy ativo |
| 3 | 👨‍🍳 Cozinha | Pedido permanece em `PRONTO` na fila — ninguém assume |
| 4 | 🔑 Gerente | Vê no painel um aviso de **pedido aguardando motoboy há X minutos** |
| 5 | 🔑 Gerente | Ativa um motoboy adicional ou chama alguém de folga |
| 6 | 🛵 Novo motoboy | Entra online e vê o pedido na fila — assume normalmente |
| 7 | 👤 Cliente | Recebe atraso, mas o pedido segue sem cancelamento |

> **Impacto:** Alto. O sistema não pode cancelar automaticamente — o gerente precisa intervir manualmente. O pedido fica "travado" em `PRONTO` até ser assumido.
