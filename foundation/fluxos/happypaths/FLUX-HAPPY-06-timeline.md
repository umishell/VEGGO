# FLUX-HAPPY-06-timeline
### Cliente acompanha pedido em tempo real

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente

```
Após finalizar, o cliente abre a tela de acompanhamento
→ cada mudança de estado chega automaticamente, sem precisar atualizar a tela
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Após o pedido ser feito, toca em **"Acompanhar pedido"** |
| 2 | Sistema | Abre a timeline do pedido e estabelece conexão em tempo real com o servidor |
| 3 | 👤 Cliente | Vê estado inicial: **Pedido recebido** ✅ |
| 4 | 👨‍🍳 Cozinha | Clica em **Aceitar** na fila. O pedido vai para `EM_PREPARO` |
| 5 | Sistema | Empurra a atualização para o app do cliente **automaticamente** |
| 6 | 👤 Cliente | Vê na timeline: **Em preparo** 🍳 — sem precisar atualizar a tela |
| 7 | 👨‍🍳 Cozinha | Clica em **Marcar como pronto** |
| 8 | 👤 Cliente | Vê: **Pronto para retirada** 📦 |
| 9 | 🛵 Motoboy | Clica em **Assumir** e depois **Saiu para entrega** |
| 10 | 👤 Cliente | Vê: **Motoboy a caminho** 🛵 → **Saiu para entrega** 🚀 |
| 11 | 🛵 Motoboy | Clica em **Confirmar entrega** |
| 12 | 👤 Cliente | Vê: **Pedido entregue! ✅** |

**Timeline de estados:**

```
Pedido recebido → Em preparo → Pronto → Motoboy a caminho → Saiu → Entregue
```

> 💡 Sem rastreamento GPS no mapa — só a timeline de estados.  
> 💡 O cliente **não precisa atualizar a tela manualmente** — as mudanças chegam automaticamente via conexão em tempo real com o servidor.  
> 💡 Se o cliente fechar o app e reabrir, a timeline exibe o estado atual do pedido naquele momento.

---

> 📌 **Nota técnica resumida:**  
> As atualizações em tempo real são entregues via **WebSocket** (conexão persistente). Cada vez que um ator (cozinha, motoboy) muda o estado do pedido, o servidor emite um evento para o canal do pedido — o app do cliente, que está inscrito nesse canal, recebe e atualiza a UI imediatamente. Sem polling.  
> → Detalhes: [`../../docTecnica/INF-01-tempo-real.md`](../../docTecnica/INF-01-tempo-real.md)
