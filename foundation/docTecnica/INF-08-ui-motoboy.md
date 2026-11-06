# INF-08 — App motoboy (Android)

**Referenciado por:** [ORD-01](ORD-01-estados-pedido.md) · [FLUX-HAPPY-20](../fluxos/happypaths/FLUX-HAPPY-20-motoboy-app.md) · [INF-02 § 4](INF-02-http-vs-websocket.md)

*[← Índice docTecnica](README.md)*

Perfil **motoboy** — JWT + indicador de **turno ativo** (`motoboyAtivo`). Identidade visual distinta dos botões da cozinha: **Aceitar pedido** = `POST /orders/{id}/take`, não confundir com **Preparar** do KDS.

---

## 1. Telas e fluxos

### 1.1 Fila de prontos

| Conteúdo | Regra |
|----------|--------|
| Lista | Pedidos `PRONTO` sem motoboy alocado |
| Destaque | `display_number` (#…) |
| Dado oculto na lista | Nome do cliente (conferência pelo **recibo** no pacote + app ao abrir detalhe) |
| Ação | Toque → tela de confirmação / detalhe mínimo antes do take |

### 1.2 Aceitar pedido (`take`)

- HTTP `POST /orders/{id}/take` + `Idempotency-Key`.
- Efeito: `PRONTO → SAIU_PARA_ENTREGA`, `dispatch_started_at`, `balcao_wait_seconds`, associação `motoboy_id`.
- Navega para **Acompanhamento da entrega**.

### 1.3 Acompanhamento da entrega

| Bloco | Conteúdo |
|-------|----------|
| Cliente | Nome, telefone |
| Timer | **Cronômetro** desde `dispatch_started_at` até **Finalizar entrega** ([ORD-01 § 5.6](ORD-01-estados-pedido.md)) |
| Navegação | Atalhos **Waze** / **Google Maps** (`GET /orders/{id}/navigation-links`) |
| Contato | **WhatsApp** e demais links conforme [DB-01](DB-01-canal-atendimento.md) |
| **Finalizar entrega** | `POST /orders/{id}/deliver` → `ENTREGUE`; grava `delivered_at`, `delivery_duration_seconds`, `total_fulfillment_seconds` |
| **Reportar imprevisto** | `POST /orders/{id}/incident/report` — alerta gerente; **não** muda estado até o gerente confirmar ([INF-02](INF-02-http-vs-websocket.md)) |

### 1.4 Pós-entrega

- Volta à **Fila de prontos** com lista atualizada.

---

## 2. O que o motoboy não vê (MVP)

- Pedidos `RECEBIDO` / `EM_PREPARO` (fila da cozinha).
- Edição de cardápio ou painel financeiro.

---

*[← Índice docTecnica](README.md)*
