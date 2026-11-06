# FLUX-HAPPY-06-timeline
### Cliente acompanha pedido em tempo real

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente

```
Após finalizar, o cliente abre a tela de acompanhamento
→ cada mudança de estado chega automaticamente, sem precisar atualizar a tela
→ contato com a loja (WhatsApp) sempre disponível na timeline
```

---

## Passo a passo

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Após o pedido ser feito, toca em **"Acompanhar pedido"** |
| 2 | Sistema | Abre a timeline do pedido e estabelece conexão em tempo real com o servidor |
| 3 | 👤 Cliente | Vê estado inicial: **Pedido recebido** ✅ |
| 4 | 👨‍🍳 Cozinha | Clica em **Aceitar** na fila — pedido vai para `EM_PREPARO` |
| 5 | Sistema | Empurra a atualização para o app do cliente **automaticamente** |
| 6 | 👤 Cliente | Vê na timeline: **Em preparo** 🍳 |
| 7 | 👨‍🍳 Cozinha | Clica em **Marcar como pronto** |
| 8 | 👤 Cliente | Vê: **Pronto para retirada** 📦 |
| 9 | 🛵 Motoboy | Clica **Aceitar pedido** — `SAIU_PARA_ENTREGA` |
| 10 | 👤 Cliente | Vê: **Pedido saiu para entrega** 🚀 |
| 10a | Sistema | Na área de ações da timeline: só **💬 Fale com a loja** (sempre visível) — WhatsApp da loja com texto que inclui o **número do pedido** |
| 11 | 🛵 Motoboy | Clica em **Confirmar entrega** |
| 12 | 👤 Cliente | Vê: **Pedido entregue! ✅** — evento de venda encerrado |

---

## Botões de contato visíveis por estado

| Botão | Visível em | Some em |
|-------|------------|---------|
| 💬 **Fale com a loja** | WhatsApp da loja; na timeline o link inclui **contexto do pedido** (`display_number`) | Nunca |

> Se ocorrer um imprevisto e o pedido voltar a `RECEBIDO`, o cliente continua usando **Fale com a loja** (sempre visível). O **motoboy** pode contatar o **cliente** pelo app de entrega; o cliente **não** recebe link direto ao WhatsApp do motoboy.

---

**Timeline de estados:**

```
Pedido recebido → Em preparo → Pronto → Saiu para entrega → Entregue
                                               ↑
                              [Fale com a loja — sempre]
```

> 💡 Sem rastreamento GPS no mapa — só a timeline de estados.  
> 💡 O cliente **não precisa atualizar a tela manualmente** — as mudanças chegam automaticamente via WebSocket.  
> 💡 Se o cliente fechar o app e reabrir, a timeline exibe o estado atual e o botão de contato correto.

---

**Veja também:**  
→ [FLUX-HAPPY-17-whatsapp](FLUX-HAPPY-17-whatsapp.md) — **Fale conosco** e WhatsApp (cliente ↔ loja; motoboy ↔ cliente)  
→ [FLUX-SAD-21-aviso-cliente](../sadPaths/FLUX-SAD-21-aviso-cliente.md) — o que acontece quando há imprevisto  

---

> 📌 **Nota técnica resumida:**  
> As atualizações em tempo real chegam via **WebSocket** (canal `order:{orderId}`). Cada transição de estado emite um evento com o novo status.  
> O link **Fale com a loja** vem de `GET /orders/{id}/contact-links` → `store` (FGAC: só o dono do pedido).  
> → Decisão de stack: [`../../docTecnica/INF-01-tempo-real.md`](../../docTecnica/INF-01-tempo-real.md)  
> → Canais e payloads: [`../../docTecnica/INF-02-http-vs-websocket.md`](../../docTecnica/INF-02-http-vs-websocket.md)  
> → Contatos e links: [`../../docTecnica/DB-01-canal-atendimento.md`](../../docTecnica/DB-01-canal-atendimento.md)
