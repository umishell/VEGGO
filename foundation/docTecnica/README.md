# Documentação Técnica — VEGGO Delivery

Decisões de tecnologia, infraestrutura e esquemas de banco. A definição do MVP está em [`pitch_delivery.md`](../pitch_delivery.md). Os fluxos em `foundation/fluxos/` só apontam para cá.

Stack fixa: PostgreSQL como único banco da aplicação, Redis só para sessão, gateway Fastify, WebSocket no Spring. O front não é redefinido aqui.

---

## Índice

| ID | Documento | Tema |
|----|-----------|------|
| [DB-01](DB-01-canal-atendimento.md) | Canal de Atendimento (Fale Conosco) | Link `wa.me`; sem tabela de mensagens; `whatsapp_number` em PostgreSQL |
| [DB-02](DB-02-disponibilidade-itens.md) | Itens, loja e zonas | `DISPONIVEL`, `PAUSADO_DIA`, `BLOQUEADO`, exclusão física; `store_status`; `delivery_zones` |
| [ORD-01](ORD-01-estados-pedido.md) | Máquina de estados do pedido | `RECEBIDO` até `ENTREGUE`; cancelamento antes do preparo; imprevisto no mesmo pedido; timeouts |
| [INF-01](INF-01-tempo-real.md) | WebSocket — decisão de stack | Spring no monólito, proxy no Fastify, PostgreSQL como fonte da verdade |
| [INF-02](INF-02-http-vs-websocket.md) | HTTP vs WebSocket por transição | Protocolo por ator e estado; lifecycle da cozinha bidirecional; heartbeat; reconexão; payloads |
| [INF-03](INF-03-autenticacao.md) | Autenticação por plataforma | JWT em cookie (Angular web) vs header Bearer (Android Kotlin); EncryptedSharedPreferences; OkHttp interceptor/authenticator; Fastify gateway; Argon2id |
| [DB-03](DB-03-telemetria.md) | Telemetria e estatísticas | Queries de agregação em PostgreSQL; painel do dia em tempo real; histórico por período |

---

## Fluxos que referenciam esta documentação

| Fluxo | Referencia |
|-------|-----------|
| [FLUX-HAPPY-06-timeline](../fluxos/happypaths/FLUX-HAPPY-06-timeline.md) — Timeline em tempo real | INF-01 |
| [FLUX-HAPPY-10-item-gerente](../fluxos/happypaths/FLUX-HAPPY-10-item-gerente.md) — Gerente pausa, bloqueia ou apaga item | DB-02 |
| [FLUX-HAPPY-11-item-cozinha](../fluxos/happypaths/FLUX-HAPPY-11-item-cozinha.md) — Cozinha pausa ou bloqueia item | DB-02 |
| [FLUX-HAPPY-12-horario](../fluxos/happypaths/FLUX-HAPPY-12-horario.md) — Horários e fechamentos programados | DB-02 |
| [FLUX-HAPPY-18-painel](../fluxos/happypaths/FLUX-HAPPY-18-painel.md) — Painel de pedidos do dia | INF-01 · DB-03 |
| [FLUX-HAPPY-19-stats](../fluxos/happypaths/FLUX-HAPPY-19-stats.md) — Estatísticas e telemetria | DB-03 |
| [FLUX-HAPPY-14-frete](../fluxos/happypaths/FLUX-HAPPY-14-frete.md) — Zonas de frete | DB-02 |
| [FLUX-SAD-13-loja-emergencia](../fluxos/sadPaths/FLUX-SAD-13-loja-emergencia.md) — Fechamento de emergência | DB-02 |
| [FLUX-SAD-19-rota-motoboy](../fluxos/sadPaths/FLUX-SAD-19-rota-motoboy.md) — Motoboy reporta imprevisto em rota | DB-01 · ORD-01 |
| [FLUX-SAD-19-rota-motoboy](../fluxos/sadPaths/FLUX-SAD-19-rota-motoboy.md) — Motoboy cancela em rota | DB-01 · ORD-01 |
| [FLUX-SAD-21-aviso-cliente](../fluxos/sadPaths/FLUX-SAD-21-aviso-cliente.md) — Aviso ao cliente e Fale Conosco | DB-01 |
| [FLUX-SAD-18-rota-gerente](../fluxos/sadPaths/FLUX-SAD-18-rota-gerente.md) — Protocolo de imprevisto em rota (prioridade) | ORD-01 · DB-01 |
| [FLUX-SAD-12-cozinha-fila](../fluxos/sadPaths/FLUX-SAD-12-cozinha-fila.md) — Alerta de aceite atrasado | ORD-01 |
| [FLUX-HAPPY-17-whatsapp](../fluxos/happypaths/FLUX-HAPPY-17-whatsapp.md) — Fale Conosco via WhatsApp | DB-01 |

---

## Convenções desta pasta

- Prefixos: `DB-` banco · `ORD-` estados de pedido · `INF-` infraestrutura e autenticação.
- História de quem clica em quê fica nos fluxos. Aqui só fica a regra técnica.
