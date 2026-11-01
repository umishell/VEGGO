# INF-01 — Atualizações em Tempo Real (WebSocket)

**Referenciado por:**  
→ [FLUX-HAPPY-06-timeline](../fluxos/happypaths/FLUX-HAPPY-06-timeline.md)

*[← Índice docTecnica](README.md)*

Alinhado a [`pitch_delivery.md`](../pitch_delivery.md).

---

## 1. Decisão

A timeline do pedido usa **WebSocket do Spring**, no mesmo processo do monólito. O browser fala só com o **gateway Fastify**, que faz proxy do endpoint de WebSocket para o Spring.

PostgreSQL é a fonte da verdade do estado. Redis continua restrito a sessão, denylist e logout. Redis não publica evento de pedido.

Não entra Socket.io, broker Node nem broker de outro framework. Não há segundo banco.

---

## 2. Caminho

```
Browser  --wss-->  Fastify /ws  -->  Spring WebSocket
                                      |
                                      +-- lê e grava PostgreSQL
```

O broker é interno ao processo Spring. Um único container não precisa de broker externo.

---

## 3. Eventos

Destino: o pedido `order:{orderId}`, a cozinha `kitchen:{storeId}`, a fila `delivery:{storeId}`.

Payload mínimo:

```text
order.status_changed  { orderId, newStatus, priority, occurredAt }
order.incident_alert  { orderId, incidentAlert, incidentMessage }
```

Publica quem muda estado: cozinha (aceitar já é `EM_PREPARO`), motoboy, gerente e o próprio sistema (imprevisto em rota). Não existe evento de recusa de pedido. O campo `priority` viaja no `status_changed` para o KDS ordenar a fila. O `incidentMessage` viaja no `incident_alert` para exibição na timeline do cliente.

Inscrições:

| Cliente | Destino |
|---------|---------|
| Tela de acompanhamento do cliente | `order:{orderId}` do pedido dele |
| KDS | `kitchen:{storeId}` |
| Fila do motoboy | `delivery:{storeId}` |

O servidor só assina o cliente no destino se a policy do pedido autorizar.

---

## 4. App fechado

A conexão cai quando a aba fecha. Ao abrir de novo, o cliente busca o estado por REST e abre outro WebSocket. O que vale é a linha no PostgreSQL.

A cozinha reconecta automaticamente sem intervenção do cozinheiro (backoff + HTTP GET ao reconectar). Detalhes completos em INF-02.

---

**Protocolo detalhado por transição de estado, lifecycle da cozinha, heartbeat, reconexão e payloads:**  
→ [INF-02-http-vs-websocket.md](INF-02-http-vs-websocket.md)

---

*[← Índice docTecnica](README.md)*
