# INF-02 — HTTP vs WebSocket: Protocolo por Transição de Estado

**Referenciado por:**  
→ [FLUX-HAPPY-05-venda](../fluxos/happypaths/FLUX-HAPPY-05-venda.md) · [FLUX-HAPPY-06-timeline](../fluxos/happypaths/FLUX-HAPPY-06-timeline.md) · [FLUX-HAPPY-09-item-novo](../fluxos/happypaths/FLUX-HAPPY-09-item-novo.md) · [FLUX-HAPPY-10-item-gerente](../fluxos/happypaths/FLUX-HAPPY-10-item-gerente.md) · [FLUX-HAPPY-18-painel](../fluxos/happypaths/FLUX-HAPPY-18-painel.md) · [FLUX-HAPPY-22-cardapio-categorias](../fluxos/happypaths/FLUX-HAPPY-22-cardapio-categorias.md)  
→ [FLUX-SAD-18-rota-gerente](../fluxos/sadPaths/FLUX-SAD-18-rota-gerente.md) · [FLUX-SAD-19-rota-motoboy](../fluxos/sadPaths/FLUX-SAD-19-rota-motoboy.md) · [FLUX-SAD-21-aviso-cliente](../fluxos/sadPaths/FLUX-SAD-21-aviso-cliente.md)

*[← Índice docTecnica](README.md)*

Complementa [INF-01-tempo-real.md](INF-01-tempo-real.md) (decisão de stack) com o protocolo detalhado por transição de estado.

---

## 1. Regra geral de decisão

| Protocolo | Quando usar |
|-----------|-------------|
| **HTTP REST** | Ação iniciada pelo ator que gera mudança de estado: checkout, cancelar, assumir entrega, confirmar entrega, acionar imprevisto, fechar loja, turno do motoboy. Resposta síncrona com status HTTP. Suporta `Idempotency-Key` por cabeçalho. |
| **WebSocket (STOMP)** | **Receber:** servidor empurra mudanças de estado para quem está inscrito no topic. **Enviar ações de domínio:** somente a cozinha (aceitar → `EM_PREPARO`, marcar pronto → `PRONTO`) via STOMP `SEND` no mesmo canal (seção 4). |

Não existe polling. Quem **não** é cozinha: WebSocket só para **leitura em tempo real**; toda escrita que muda pedido/loja/turno vai por **HTTP**.

| Ator | WebSocket recebe (subscribe) | WebSocket envia (STOMP SEND / ação) | HTTP escreve |
|------|------------------------------|-------------------------------------|--------------|
| Cozinha (KDS) | Sim — `kitchen:{storeId}` | **Sim** — aceitar, marcar pronto | Não (nessas transições) |
| Cliente (timeline) | Sim — `order:{orderId}` | **Não** | Sim — checkout, cancelar |
| Motoboy | Sim — `delivery:{storeId}` | **Não** | Sim — assumir/sair, entregar, reportar imprevisto, turno |
| Gerente (painel) | Sim — `kitchen:{storeId}` | **Não** | Sim — cancelar pedido, imprevisto, loja, etc. |

Ou seja: **vários perfis abrem WebSocket**, mas **só a cozinha manda comandos de mudança de estado pelo WebSocket**. Gerente e motoboy também ficam “conectados o dia todo” na operação, porém quando clicam em algo que altera o sistema, usam REST.

---

## 2. Mapeamento: transição → protocolo → canais notificados

| Transição | Ator | Protocolo da ação | Canais WebSocket publicados |
|-----------|------|-------------------|-----------------------------|
| → `RECEBIDO` (checkout) | Cliente | HTTP POST `/orders` | `kitchen:{storeId}` |
| → `EM_PREPARO` (Preparar) | Cozinha | **WebSocket** `@MessageMapping /orders/{id}/accept` | `order:{orderId}` · `kitchen:{storeId}` |
| → `PRONTO` (Concluir preparo) | Cozinha | **WebSocket** `@MessageMapping /orders/{id}/ready` | `order:{orderId}` · `kitchen:{storeId}` · `delivery:{storeId}` |
| → `SAIU_PARA_ENTREGA` (aceitar pedido / take) | Motoboy | **HTTP POST** `/orders/{id}/take` + `Idempotency-Key` | `order:{orderId}` · `kitchen:{storeId}` · `delivery:{storeId}` |
| → `ENTREGUE` (confirmar) | Motoboy | HTTP POST `/orders/{id}/deliver` | `order:{orderId}` · `kitchen:{storeId}` |
| → `CANCELADO` (cliente) | Cliente | HTTP DELETE `/orders/{id}` | `order:{orderId}` · `kitchen:{storeId}` |
| → `CANCELADO` (gerente) | Gerente | HTTP POST `/orders/{id}/cancel` | `order:{orderId}` · `kitchen:{storeId}` |
| → `RECEBIDO` ⚡ (imprevisto) | Gerente | HTTP POST `/orders/{id}/incident` | `order:{orderId}` · `kitchen:{storeId}` · `delivery:{storeId}` |

**Por que a cozinha usa WebSocket para ações de estado:** a KDS mantém conexão aberta durante todo o turno. Enviar ações pelo mesmo canal evita o overhead de abrir nova conexão TCP a cada clique, e se encaixa nativamente no modelo STOMP bidirecional do Spring.

**Por que motoboy usa HTTP para aceitar pedido (`/take`):** lock `SELECT … FOR UPDATE`, Idempotency-Key, HTTP 409 em disputa. UI: **Aceitar pedido** → `SAIU_PARA_ENTREGA` + tela de acompanhamento (INF-04 § 2).

---

## 3. Canais STOMP e assinantes

O Spring WebSocket usa STOMP como protocolo de mensageria sobre WebSocket. O Fastify faz proxy do endpoint `/ws` para o Spring (veja INF-01). O broker é in-process — sem broker externo.

| Canal (topic STOMP) | Quem assina | O que chega |
|---------------------|-------------|-------------|
| `/topic/order/{orderId}` | Tela de timeline do cliente | Toda mudança de estado do pedido; `incident_alert` + `incidentMessage` |
| `/topic/kitchen/{storeId}` | KDS da cozinha · Painel do gerente | Novos pedidos `RECEBIDO`; mudanças de estado de qualquer pedido da loja; eventos de imprevisto |
| `/topic/delivery/{storeId}` | App do motoboy | Pedidos que entram na fila (`PRONTO`); pedidos que saem da fila (`SAIU_PARA_ENTREGA`, `CANCELADO`) |

Publicação no servidor:

```java
// Spring Service
simpMessagingTemplate.convertAndSend("/topic/kitchen/" + storeId, payload);
simpMessagingTemplate.convertAndSend("/topic/order/" + orderId, payload);
simpMessagingTemplate.convertAndSend("/topic/delivery/" + storeId, payload);
```

O servidor só permite que um cliente assine um canal se a policy do pedido autorizar (FGAC verificada no handshake via Fastify e no `ChannelInterceptor` do Spring).

---

## 4. Kitchen — conexão WebSocket bidirecional

A cozinha é o único ator com WebSocket aberto nos dois sentidos. O cozinheiro não clica em "conectar" — o Angular abre a conexão automaticamente quando a tela do KDS carrega.

### 4.1 Abertura e handshake

```
Cozinheiro abre a URL da loja
  └─► Angular carrega · ngOnInit do componente KDS executa
        └─► StompClient.activate() (lib: @stomp/rx-stomp)
              └─► HTTP GET /ws  ──►  Fastify
                    │
                    Fastify lê o cookie HttpOnly (JWT)
                    Valida assinatura + perfil (FUNCIONARIO_COZINHA ou GERENTE_LOJA)
                    │
                    ✅ Retorna HTTP 101 Switching Protocols  (RFC 6455)
                    │
                    Fastify faz proxy da conexão WebSocket para o Spring
                    │
                    STOMP CONNECT frame  ──►  Spring WebSocketMessageBroker
                    │
                    ✅ STOMP CONNECTED — canal aberto, permanece assim por horas
                    │
                    Angular assina /topic/kitchen/{storeId}
```

**Detalhe de segurança crítico:** o browser não permite enviar cabeçalhos HTTP customizados no upgrade WebSocket pela API nativa. O JWT não vai no `Authorization` header — vai no cookie `HttpOnly + Secure` que o browser envia automaticamente. O Fastify valida esse cookie **antes** de aceitar o upgrade. Se o cookie for inválido ou o perfil for incorreto, o Fastify retorna HTTP 401 e o WebSocket nunca é estabelecido. O Spring não repete a validação: confia que tudo que chegou pelo Fastify já foi autenticado.

**Biblioteca Angular recomendada:** `@stomp/rx-stomp` (wrapper RxJS sobre `@stomp/stompjs`).

```typescript
// Exemplo de configuração no Angular (KDS component)
const stompConfig: RxStompConfig = {
  webSocketFactory: () => new WebSocket('wss://loja.com/ws'),
  connectHeaders: {}, // JWT vem via cookie, não header
  heartbeatIncoming: 10000,
  heartbeatOutgoing: 10000,
  reconnectDelay: 3000,
};
rxStompService.configure(stompConfig);
rxStompService.activate(); // ← chamado no ngOnInit
```

### 4.2 Sentido servidor → cozinha (recebimento)

Quando o cliente faz checkout, o Spring:

1. Executa o checkout atômico via HTTP.
2. Chama `simpMessagingTemplate.convertAndSend("/topic/kitchen/{storeId}", payload)`.
3. O broker entrega o frame STOMP para todos os assinantes do topic — incluindo a KDS.
4. Angular recebe via `rxStompService.watch('/topic/kitchen/{storeId}')` e insere o novo card na fila, sem recarregar a página.

**Payload recebido pela cozinha (novo pedido):**

```json
{
  "type": "order.new",
  "orderId": "550e8400-e29b-41d4-a716-446655440000",
  "items": [
    { "name": "X-Burguer", "quantity": 2 },
    { "name": "Coca-Cola", "quantity": 1 }
  ],
  "priority": false,
  "createdAt": "2026-11-01T03:00:00Z"
}
```

Se `priority: true` (imprevisto em rota), o KDS posiciona a linha no topo com indicador ⚡.

**Lista KDS:** pedidos `RECEBIDO` e `EM_PREPARO`, ordenação `priority DESC`, `created_at ASC`. Clique na linha = **seleção** (detalhe à direita); **não** inicia preparo.

### 4.3 Sentido cozinha → servidor (envio de ações de estado)

A cozinha **não abre nova requisição HTTP** para preparar ou concluir. Usa `rxStompService.publish()` no mesmo canal já aberto.

**Preparar (`RECEBIDO → EM_PREPARO`)** — botão ↖ no detalhe; destino STOMP `/accept`:

```typescript
rxStompService.publish({
  destination: '/app/orders/550e8400/accept',
  body: JSON.stringify({ acceptedAt: new Date().toISOString() }),
});
```

```java
@MessageMapping("/orders/{orderId}/accept")
@Transactional
public void startPrep(@DestinationVariable String orderId,
                      SimpMessageHeaderAccessor headerAccessor) {
    Order order = orderRepo.findByIdForUpdate(orderId);
    if (order.getStatus() != RECEBIDO) {
        sendError(headerAccessor, "INVALID_STATE", orderId);
        return;
    }
    Instant now = Instant.now();
    order.setStatus(EM_PREPARO);
    order.setPrepStartedAt(now);
    orderRepo.save(order);
    orderEventRepo.save(new OrderEvent(orderId, RECEBIDO, EM_PREPARO, actorId));
    messagingTemplate.convertAndSend("/topic/order/" + orderId,
        new StatusChangedPayload(orderId, EM_PREPARO));
    messagingTemplate.convertAndSend("/topic/kitchen/" + storeId,
        new StatusChangedPayload(orderId, EM_PREPARO));
}
```

**Concluir preparo (`EM_PREPARO → PRONTO`)** — botão ↗; destino `/ready`:

```typescript
rxStompService.publish({ destination: '/app/orders/550e8400/ready', body: '{}' });
```

```java
@MessageMapping("/orders/{orderId}/ready")
@Transactional
public void markReady(@DestinationVariable String orderId, ...) {
    Order order = orderRepo.findByIdForUpdate(orderId);
    if (order.getStatus() != EM_PREPARO) {
        sendError(..., "INVALID_STATE", orderId);
        return;
    }
    Instant now = Instant.now();
    order.setStatus(PRONTO);
    order.setPrepCompletedAt(now);
    if (order.getPrepStartedAt() != null) {
        order.setPrepDurationSeconds(
            (int) Duration.between(order.getPrepStartedAt(), now).getSeconds());
    }
    orderRepo.save(order);
    // publica order + kitchen + delivery
}
```

**Como o Angular sabe se a ação deu certo:** assina `/user/queue/errors` ao inicializar. Se o servidor enviar um `ErrorPayload`, o Angular exibe um toast e, se necessário, reconcilia o estado via HTTP GET.

```typescript
rxStompService.watch('/user/queue/errors').subscribe(msg => {
  const error = JSON.parse(msg.body);
  showToast(`Ação inválida: ${error.code} — pedido ${error.orderId}`);
  refreshOrderState(error.orderId); // HTTP GET /orders/{id}
});
```

### 4.4 Conflito entre dois cozinheiros

Se dois cozinheiros clicarem **Preparar** no mesmo pedido quase simultaneamente:

1. O `@MessageMapping` de `accept` executa dentro de `@Transactional` com `SELECT FOR UPDATE`.
2. O primeiro commita: pedido vai para `EM_PREPARO`.
3. O segundo encontra o estado `EM_PREPARO` e envia `ErrorPayload` de volta via `/user/queue/errors` para aquele cozinheiro.
4. O KDS do segundo cozinheiro remove o card da fila (já que o status_changed chegou via `/topic/kitchen/{storeId}`).

### 4.5 Heartbeat e reconexão automática

**Heartbeat STOMP** — mantém o túnel TCP vivo durante períodos sem mensagens (ex: cozinha ociosa):

```java
// Spring WebSocketConfig
@Override
public void configureMessageBroker(MessageBrokerRegistry registry) {
    registry.enableSimpleBroker("/topic", "/queue")
            .setHeartbeatValue(new long[]{10000, 10000}); // send/receive em ms
    registry.setApplicationDestinationPrefixes("/app");
}
```

```typescript
// Angular StompConfig
heartbeatIncoming: 10000,  // Spring envia ping a cada 10s
heartbeatOutgoing: 10000,  // Angular envia ping a cada 10s
```

**Ciclo de reconexão automática quando a internet oscila:**

```
WebSocket fecha  ──►  @stomp/rx-stomp detecta onclose
                  ──►  aguarda reconnectDelay (padrão: 3s; pode usar backoff: 3s → 6s → 12s → 30s)
                  ──►  tenta reconectar (novo handshake com cookie JWT)
                  ──►  ✅ conectado
                  ──►  HTTP GET /orders?storeId={storeId}&status=RECEBIDO,EM_PREPARO
                       → reconstrói lista KDS + seleção com estado atual do PostgreSQL
                  ──►  re-assina /topic/kitchen/{storeId}
                  ──►  operação normal — cozinheiro não percebeu nada
```

O passo de HTTP GET ao reconectar é **obrigatório**: durante a janela offline, pedidos podem ter mudado de estado. O PostgreSQL é a fonte da verdade; o WebSocket é apenas o canal de entrega das notificações.

**Ciclo de vida completo da sessão da cozinha:**

```
[ Início do turno ]
  Abre a URL  ──►  Angular carrega  ──►  ngOnInit  ──►  StompClient.activate()
                                                                │
                                                    HTTP 101 + STOMP CONNECT
                                                                │
                                                    Canal aberto e estável

[ Durante o dia ]
  Servidor ──► Angular   push de novos pedidos, atualizações de estado  ◄──┐
  Angular  ──► Servidor   "Aceitar" · "Marcar Pronto" via STOMP SEND   ────┘

[ Queda de internet ]
  onclose  ──►  backoff  ──►  HTTP GET (reconstrói estado)  ──►  re-assina WS

[ Fim do turno ]
  Cozinheiro fecha a aba  ──►  TCP FIN  ──►  Spring limpa a sessão STOMP
```

---

## 5. Timeline do cliente (app-cliente Android Kotlin) — somente leitura via WebSocket

O cliente nunca envia mensagens pelo WebSocket. Todas as ações são HTTP.

| Ação | Protocolo |
|------|-----------|
| Checkout | HTTP POST `/orders` |
| Cancelar pedido | HTTP DELETE `/orders/{id}` |
| Abrir tela de acompanhamento | HTTP GET `/orders/{id}` → abre WebSocket OkHttp, assina `/topic/order/{orderId}` |
| Receber atualização de estado | WebSocket push do servidor |
| Fechar / minimizar app | WebSocket fecha (`WebSocketListener.onClosed` OkHttp) |

**Reabrir o app / tela:** o app Android não mantém WebSocket persistente como o KDS web da cozinha. Ao abrir a tela de acompanhamento, o app busca o estado atual via HTTP GET e só então abre o WebSocket. O que vale é sempre o PostgreSQL.

**Payload recebido pelo cliente — atualização de estado:**

```json
{
  "type": "order.status_changed",
  "orderId": "550e8400-e29b-41d4-a716-446655440000",
  "newStatus": "EM_PREPARO",
  "priority": false,
  "occurredAt": "2026-11-01T03:05:00Z"
}
```

**Payload recebido pelo cliente — imprevisto em rota:**

```json
{
  "type": "order.incident_alert",
  "orderId": "550e8400-e29b-41d4-a716-446655440000",
  "newStatus": "RECEBIDO",
  "incidentAlert": true,
  "priority": true,
  "incidentMessage": "Houve um imprevisto com a entrega. Seu pedido foi colocado com prioridade na fila da cozinha e logo chegará! 🙏"
}
```

O app Android usa **OkHttp WebSocket** (`OkHttpClient.newWebSocket(request, listener)`). Ao contrário do browser, o Android pode enviar o JWT no cabeçalho `Authorization: Bearer <token>` durante o handshake — o Fastify valida esse cabeçalho antes de aceitar o upgrade. Detalhes de autenticação por plataforma em [INF-03-autenticacao.md](INF-03-autenticacao.md).

---

## 6. Motoboy (app-motoboy Android Kotlin) — subscrição + ações HTTP

O motoboy combina: assina o canal `delivery:{storeId}` para saber de novos pedidos em tempo real, mas usa HTTP para todas as ações de estado.

| Ação | Protocolo | Detalhe técnico |
|------|-----------|-----------------|
| Ver fila de pedidos `PRONTO` | WebSocket `/topic/delivery/{storeId}` | Push ao entrar na fila; push ao sair (outro motoboy assumiu) |
| Iniciar turno | HTTP POST `/delivery-persons/shift/start` | Motoboy entra como disponível |
| Aceitar pedido | **HTTP POST** `/orders/{id}/take` + `Idempotency-Key` | `SELECT … FOR UPDATE` — HTTP 200 ou 409 |
| Confirmar entrega | HTTP POST `/orders/{id}/deliver` | `SAIU_PARA_ENTREGA → ENTREGUE` |
| Reportar imprevisto | HTTP POST `/orders/{id}/incident/report` | Cria evento de alerta em `order_events`; notifica gerente via `kitchen:{storeId}`. **Não muda o estado** — só o gerente confirma |
| Encerrar turno | HTTP POST `/delivery-persons/shift/end` | Sai da fila disponível |

**Detalhe de aceite com lock (UI: Aceitar pedido):**

```
Motoboy A e B clicam "Aceitar pedido" no mesmo pedido #1042

  Motoboy A  ──►  HTTP POST /orders/0042/take  (Idempotency-Key: uuid-a)
  Motoboy B  ──►  HTTP POST /orders/0042/take  (Idempotency-Key: uuid-b)

  Spring:
    BEGIN TRANSACTION
    SELECT * FROM orders WHERE id='0042' FOR UPDATE  ← bloqueia a linha
    IF status != PRONTO → rejeita
    SET status = SAIU_PARA_ENTREGA, motoboy_id = motoboy_a_id
    INSERT INTO order_events(...)
    COMMIT

    Motoboy A  ◄──  HTTP 200 { orderId, newStatus: "SAIU_PARA_ENTREGA" }
    Motoboy B  ◄──  HTTP 409 { error: "ORDER_ALREADY_TAKEN" }

  Spring publica:
    /topic/order/{orderId}      → cliente vê "Saiu para entrega"
    /topic/kitchen/{storeId}    → KDS remove o card
    /topic/delivery/{storeId}   → outros motoboys veem o pedido sumir da fila
```

O `Idempotency-Key` garante que um duplo clique do mesmo motoboy (rede instável) não crie dois commits. O Spring verifica o key em cache (Redis) antes de processar.

---

## 7. Gerente (web-operacao Angular) — painel em tempo real + ações HTTP

| Ação | Protocolo |
|------|-----------|
| Painel do dia em tempo real | WebSocket `/topic/kitchen/{storeId}` — mesma inscrição da cozinha |
| Cancelar pedido | HTTP POST `/orders/{id}/cancel` |
| Acionar imprevisto em rota | HTTP POST `/orders/{id}/incident` |
| Fechar loja em emergência | HTTP POST `/stores/{id}/close` |
| Reabrir loja manualmente | HTTP POST `/stores/{id}/reopen` |
| Criar fechamento/abertura programada | HTTP POST `/stores/{id}/schedules` |

O gerente assina o mesmo `/topic/kitchen/{storeId}` que a cozinha. A diferença está na UI Angular: o gerente vê todos os estados (`RECEBIDO` até `ENTREGUE`, inclusive `CANCELADO` e pedidos com `priority`); o KDS da cozinha filtra e exibe só até `PRONTO` (o card some quando vai para `SAIU_PARA_ENTREGA`).

---

## 8. Fluxo de imprevisto — propagação do evento ⚡

```
Gerente clica "Acionar imprevisto"
  │
  ▼
HTTP POST /orders/{id}/incident
  { reason: "Pneu furado", incidentMessage: "Seu pedido está sendo refeito..." }
  │
  ▼
Spring @Transactional
  status              = RECEBIDO
  priority            = true
  incident_alert      = true
  incident_message    = "Seu pedido está sendo refeito..."
  cancelamento_bloqueado = true
  motoboy_id          = null
  remake_seq          = remake_seq + 1
  INSERT order_events (SAIU_PARA_ENTREGA → RECEBIDO, motivo: IMPREVISTO_ENTREGA)
  │
  ▼
Spring publica para três canais:

  /topic/order/{orderId}
  {
    "type": "order.incident_alert",
    "orderId": "...",
    "newStatus": "RECEBIDO",
    "priority": true,
    "incidentAlert": true,
    "incidentMessage": "Seu pedido foi colocado com prioridade..."
  }

  /topic/kitchen/{storeId}
  {
    "type": "order.priority_requeue",
    "orderId": "...",
    "newStatus": "RECEBIDO",
    "priority": true,
    "items": [{ "name": "X-Burguer", "quantity": 2 }]
  }

  /topic/delivery/{storeId}
  {
    "type": "order.removed_from_queue",
    "orderId": "..."
  }
```

**KDS após receber `order.priority_requeue`:** reinsere o card no topo da lista `RECEBIDO`, ordenada por `priority DESC, createdAt ASC`. O cozinheiro vê o indicador ⚡ e sabe que é prioritário.

---

## 9. Tratamento de erros no canal WebSocket (cozinha)

| Situação | Resposta do servidor | Comportamento Angular |
|----------|----------------------|-----------------------|
| **Preparar** pedido que não é `RECEBIDO` | `/user/queue/errors` `{ code: "INVALID_STATE", orderId }` | Toast; reconcilia via GET |
| **Concluir preparo** pedido que não é `EM_PREPARO` | Idem | Idem |
| Dois cozinheiros **Preparar** o mesmo pedido | Segundo recebe `INVALID_STATE` | Toast; lista atualiza via broadcast |
| JWT expirou durante o turno | Fastify fecha o WebSocket com código `4001 Unauthorized` | Angular detecta close code → redireciona para login |
| Queda de internet (close sem código) | — | Backoff de reconexão → HTTP GET reconcilia estado → re-assina |
| Spring reinicia (deploy) | Conexão cai com `1001 Going Away` | Backoff → reconecta após o Spring voltar |

---

## 10. Diagrama de ciclo de vida resumido

```
═══════════════════════════════════════════════════════════════════
  COZINHA (Angular web-operacao)
═══════════════════════════════════════════════════════════════════
  [ início do turno ]  →  ngOnInit  →  StompClient.activate()
                                              │
                                  HTTP 101 + STOMP CONNECT
                                              │
                          (canal aberto e estável por horas)
                                              │
  [ durante o dia ]
    Servidor ──►  Cozinha   STOMP push: novos pedidos, atualizações  ◄──────┐
    Cozinha  ──►  Servidor  STOMP send: aceitar, marcar pronto       ───────┘
                                              │
  [ queda de rede ]  →  onclose  →  backoff  →  HTTP GET  →  re-assina WS
  [ fim do turno ]   →  fecha aba  →  TCP FIN  →  sessão STOMP encerrada

═══════════════════════════════════════════════════════════════════
  CLIENTE (app-cliente Android Kotlin)
═══════════════════════════════════════════════════════════════════
  checkout  →  HTTP POST /orders  →  pedido criado
  abre timeline  →  HTTP GET /orders/{id}  →  OkHttp WS  →  assina order:{id}
  JWT no header Authorization do WS handshake (não em cookie)
  eventos chegam via push  →  UI atualiza
  fecha app  →  OkHttp WS fecha (onClosed)

═══════════════════════════════════════════════════════════════════
  MOTOBOY (app-motoboy Android Kotlin)
═══════════════════════════════════════════════════════════════════
  inicia turno  →  HTTP POST /shift/start  →  assina delivery:{storeId}
  vê pedido PRONTO via push  →  clica assumir
  HTTP POST /orders/{id}/take + Idempotency-Key  →  200 (ganha) ou 409
  HTTP POST /orders/{id}/deliver  →  ENTREGUE

═══════════════════════════════════════════════════════════════════
  GERENTE (Angular web-operacao)
═══════════════════════════════════════════════════════════════════
  assina kitchen:{storeId}  →  vê todos os pedidos em tempo real
  ações pontuais  →  HTTP POST (cancelar, imprevisto, fechar loja)
  cardápio CRUD   →  HTTP /store/products* (§ 11)
```

---

## 11. Cardápio — produtos e limites (HTTP REST)

Todas as rotas abaixo passam pelo **Fastify** (JWT). `store_id` vem do token do funcionário (`GERENTE_LOJA`) ou da loja fixa do tenant no app cliente. Esquema: [DB-02 § 1–1.2](DB-02-disponibilidade-itens.md) · categorias [DB-04 § 1](DB-04-catalogo-checkout-cliente.md). UI gerente: [INF-04 § 4](INF-04-ui-operacao-cliente.md).

### 11.1 App cliente — cardápio público

| Método | Rota | Quem | Resposta |
|--------|------|------|----------|
| GET | `/menu` | Cliente autenticado | Categorias ativas (`product_categories.active = true`) ordenadas por `sort_order`. Em cada categoria, produtos com `availability != ARQUIVADO`, ordenados por `products.sort_order`, `name`. Campos por produto: `id`, `name`, `description`, `unitPriceCents`, `imageUrl`, `availability`, `remainingToday` (nulo se ilimitado), `categorySlug`, `categoryDisplayName`. Itens cinza (`PAUSADO_DIA`, `BLOQUEADO`) incluídos com flag para UI. **Raiz:** `fixedDeliveryFeeCents` — cópia de `stores.fixed_delivery_fee_cents` (taxa de entrega fixa; não varia por CEP) |

Não expõe `default_daily_order_limit` bruto — só `remainingToday` derivado de `product_daily_caps` do dia operacional.

### 11.2 Gerente — listagem e filtros

| Método | Rota | Query | Uso |
|--------|------|-------|-----|
| GET | `/store/product-categories` | — | Dropdown de filtro e formulário (todas as categorias da loja, inclusive `active = false` para edição interna) |
| GET | `/store/products` | `categoryId` (UUID, opcional) · `availability` (enum, opcional, repetível) · `archivedOnly` (`true` → só `ARQUIVADO`) · `q` (busca parcial em `name`, opcional) | Tabela **Cardápio**; default exclui `ARQUIVADO` |
| GET | `/store/products/{productId}` | — | Detalhe + métricas do dia: `defaultDailyOrderLimit`, `effectiveLimitToday`, `unitsSoldToday`, `remainingToday` |

Resposta de lista (cada item): campos de `products` + caps do dia operacional atual.

### 11.3 Gerente — CRUD

| Método | Rota | Corpo / notas |
|--------|------|----------------|
| POST | `/store/products` | JSON: `{ name, description?, categoryId, unitPriceCents, defaultDailyOrderLimit?, sortOrder?, availability? }` — default `availability = DISPONIVEL`. Opcional **multipart** `image` no mesmo request → grava `image_url` |
| PATCH | `/store/products/{productId}` | Parcial: `name`, `description`, `categoryId`, `unitPriceCents`, `defaultDailyOrderLimit`, `sortOrder`, `availability` (pausa, bloqueio, arquivar, reativar) |
| POST | `/store/products/{productId}/image` | Multipart `image` — substitui foto; retorna `imageUrl` |
| DELETE | `/store/products/{productId}` | **Hard delete** só se gerente confirmar; pedidos antigos mantêm snapshot em `order_items`. Preferir `availability = ARQUIVADO` para ocultar |

**Autorização:** `GERENTE_LOJA` para escrita; cozinha **não** chama estas rotas.

### 11.4 Gerente — limite efetivo só hoje

| Método | Rota | Corpo |
|--------|------|-------|
| PATCH | `/store/products/{productId}/daily-cap` | `{ "effectiveLimit": 40 }` ou `{ "effectiveLimit": null }` (ilimitado **só hoje**) |

Upsert em `product_daily_caps` para `(store_id, product_id, operational_date = hoje)`. Não altera `products.default_daily_order_limit`.

**Erros comuns:**

| HTTP | Código | Quando |
|------|--------|--------|
| 404 | `PRODUCT_NOT_FOUND` | ID fora da loja |
| 409 | `DAILY_LIMIT_BELOW_SOLD` | Reduzir `effectiveLimit` abaixo de `units_sold` |
| 422 | `INVALID_CATEGORY` | `categoryId` de outra loja |

### 11.5 Checkout — validação de produto e entrega

`POST /orders` (corpo detalhado em DB-04 / HAPPY-21) resolve cada linha por `productId` + `quantity`:

- `availability` deve ser `DISPONIVEL`
- loja `ABERTA`
- se houver teto: `quantity <= remainingToday` (lock row `product_daily_caps` + incremento `units_sold` na mesma transação)
- `delivery_fee_cents := stores.fixed_delivery_fee_cents` (snapshot; **sem** lookup de CEP/zona)

Endereço de entrega é obrigatório (validação de campos — [FLUX-SAD-07](../fluxos/sadPaths/FLUX-SAD-07-endereco-vazio.md)); CEP **não** altera a taxa.

Falha → 409 `ITEM_UNAVAILABLE` ou `DAILY_LIMIT_EXCEEDED` ([FLUX-SAD-03](../fluxos/sadPaths/FLUX-SAD-03-item-checkout.md), [FLUX-SAD-24](../fluxos/sadPaths/FLUX-SAD-24-limite-dia.md)).

### 11.6 Gerente — configuração da loja (taxa de entrega)

| Método | Rota | Corpo |
|--------|------|-------|
| GET | `/store/settings` | — | Retorna `whatsappNumber`, `timezone`, **`fixedDeliveryFeeCents`** |
| PATCH | `/store/settings` | Parcial: mesmos campos (incl. `fixedDeliveryFeeCents` ≥ 0) |

Pedidos já criados **não** retroagem; só novos checkouts usam o valor atualizado. Ver [FLUX-HAPPY-12](../fluxos/happypaths/FLUX-HAPPY-12-horario.md) Parte 0.

---

*[← Índice docTecnica](README.md)*
