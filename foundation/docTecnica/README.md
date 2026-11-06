# Documentação Técnica — VEGGO Delivery

Decisões de tecnologia, infraestrutura e esquemas de banco. A definição do MVP está em [`pitch_delivery.md`](../pitch_delivery.md). **Ordem de implementação para IA:** [`IMP-01-ordem-implementacao-ai.md`](../IMP-01-ordem-implementacao-ai.md). Os fluxos em `foundation/fluxos/` só apontam para cá.

Stack fixa: **backend Spring Boot em Kotlin** com **Spring Data JPA**, **Hibernate** e **PostgreSQL** (detalhe e peculiaridades do Kotlin em [INF-05](INF-05-kotlin-spring-jpa.md)); Redis só para sessão; gateway Fastify; WebSocket no Spring. O front não é redefinido aqui.

---

## Índice

| ID | Documento | Tema |
|----|-----------|------|
| [DB-01](DB-01-canal-atendimento.md) | Canal de Atendimento | WhatsApp loja (`GET /store/contact-links`) · por pedido (`GET /orders/.../contact-links`) · navegação motoboy |
| [DB-02](DB-02-disponibilidade-itens.md) | Itens e loja | Tabela **`products`** (§ 1); disponibilidade (§ 1.2); **`store_status`**; **`fixed_delivery_fee_cents`** |
| [ORD-01](ORD-01-estados-pedido.md) | Máquina de estados do pedido | `RECEBIDO` até `ENTREGUE`; KDS multi-preparo + timer; cancelamento; imprevisto |
| [INF-01](INF-01-tempo-real.md) | WebSocket — decisão de stack | Spring no monólito, proxy no Fastify, PostgreSQL como fonte da verdade |
| [INF-02](INF-02-http-vs-websocket.md) | HTTP vs WebSocket por transição | Protocolo por ator e estado; **§ 11 CRUD cardápio** (`/menu`, `/store/products`); lifecycle cozinha |
| [INF-03](INF-03-autenticacao.md) | Autenticação por plataforma | JWT web/mobile; **autocadastro só cliente**; reset cliente por e-mail; cadastro e reset de funcionário pelo gerente |
| [DB-03](DB-03-telemetria.md) | Telemetria e estatísticas | Queries de agregação; painel do dia; histórico |
| [DB-04](DB-04-catalogo-checkout-cliente.md) | Cardápio, checkout, cliente | **`orders`**; carrinho local + **`client_shopping_sessions`**; histórico / refazer |
| [INF-04](INF-04-ui-operacao-cliente.md) | Layout (sumário) KDS, motoboy, cliente, cardápio gerente | Índice visual; **detalhe por perfil** nos INF-06 … INF-09 |
| [INF-05](INF-05-kotlin-spring-jpa.md) | Backend Kotlin | Spring Boot · Spring Data JPA · Hibernate · PostgreSQL · plugins Kotlin |
| [INF-06](INF-06-ui-gerente.md) | UI gerente (`web-operacao`) | Painel do dia (timers + colunas de tempo); cardápio; estatísticas embutidas |
| [INF-07](INF-07-ui-cozinha-kds.md) | UI cozinha — KDS | Dois painéis; Preparar / Concluir; timer de preparo; recibo |
| [INF-08](INF-08-ui-motoboy.md) | UI motoboy (`app-motoboy`) | Fila prontos; take; acompanhamento + timer de entrega; deliver |
| [INF-09](INF-09-ui-cliente.md) | UI cliente (`app-cliente`) | Cardápio, carrinho, checkout, timeline, Fale conosco |

---

## Fluxos que referenciam esta documentação

| Fluxo | Referencia |
|-------|-----------|
| [FLUX-HAPPY-06-timeline](../fluxos/happypaths/FLUX-HAPPY-06-timeline.md) — Timeline em tempo real | INF-01 |
| [FLUX-HAPPY-09-item-novo](../fluxos/happypaths/FLUX-HAPPY-09-item-novo.md) · [FLUX-HAPPY-10-item-gerente](../fluxos/happypaths/FLUX-HAPPY-10-item-gerente.md) — Cardápio, pausa, bloqueio, arquivar | DB-02 · INF-02 § 11 · INF-04 § 4 |
| [FLUX-HAPPY-02-recuperacao-senha](../fluxos/happypaths/FLUX-HAPPY-02-recuperacao-senha.md) — Reset senha cliente | INF-03 § 6.4 |
| [FLUX-HAPPY-13-usuarios](../fluxos/happypaths/FLUX-HAPPY-13-usuarios.md) — Funcionários (cadastro e senha presencial) | INF-03 § 6.1–6.3 |
| [FLUX-HAPPY-11-item-cozinha](../fluxos/happypaths/FLUX-HAPPY-11-item-cozinha.md) — Cozinha opera KDS | ORD-01 § 5 |
| [FLUX-HAPPY-20-motoboy-app](../fluxos/happypaths/FLUX-HAPPY-20-motoboy-app.md) — Navegação app motoboy | ORD-01 |
| [FLUX-HAPPY-12-horario](../fluxos/happypaths/FLUX-HAPPY-12-horario.md) — Horários e fechamentos programados | DB-02 |
| [FLUX-HAPPY-18-painel](../fluxos/happypaths/FLUX-HAPPY-18-painel.md) — Painel de pedidos do dia | INF-01 · DB-03 · INF-06 |
| [FLUX-HAPPY-19-stats](../fluxos/happypaths/FLUX-HAPPY-19-stats.md) — Estatísticas e telemetria | DB-03 |
| [FLUX-HAPPY-12-horario](../fluxos/happypaths/FLUX-HAPPY-12-horario.md) — Horários, config loja e taxa de entrega fixa | DB-02 |
| [FLUX-SAD-13-loja-emergencia](../fluxos/sadPaths/FLUX-SAD-13-loja-emergencia.md) — Fechamento de emergência | DB-02 |
| [FLUX-SAD-19-rota-motoboy](../fluxos/sadPaths/FLUX-SAD-19-rota-motoboy.md) — Motoboy reporta imprevisto em rota | DB-01 · ORD-01 |
| [FLUX-SAD-19-rota-motoboy](../fluxos/sadPaths/FLUX-SAD-19-rota-motoboy.md) — Motoboy cancela em rota | DB-01 · ORD-01 |
| [FLUX-SAD-21-aviso-cliente](../fluxos/sadPaths/FLUX-SAD-21-aviso-cliente.md) — Aviso ao cliente e Fale Conosco | DB-01 |
| [FLUX-SAD-18-rota-gerente](../fluxos/sadPaths/FLUX-SAD-18-rota-gerente.md) — Protocolo de imprevisto em rota (prioridade) | ORD-01 · DB-01 |
| [FLUX-SAD-03-item-checkout](../fluxos/sadPaths/FLUX-SAD-03-item-checkout.md) — Checkout 409 (item indisponível) | DB-02 · INF-02 § 11.4 |
| [FLUX-HAPPY-21-checkout-pagamento](../fluxos/happypaths/FLUX-HAPPY-21-checkout-pagamento.md) · [FLUX-HAPPY-22-cardapio-categorias](../fluxos/happypaths/FLUX-HAPPY-22-cardapio-categorias.md) | DB-04 · INF-02 § 11 · INF-04 § 3–4 |

---

## Convenções desta pasta

- Prefixos: `DB-` banco · `ORD-` estados de pedido · `INF-` infraestrutura e autenticação.
- História de quem clica em quê fica nos fluxos. Aqui só fica a regra técnica.
