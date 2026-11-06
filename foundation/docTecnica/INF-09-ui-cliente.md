# INF-09 — App cliente (Android)

**Referenciado por:** [DB-04](DB-04-catalogo-checkout-cliente.md) · [INF-02 § 11](INF-02-http-vs-websocket.md) · [FLUX-HAPPY-22](../fluxos/happypaths/FLUX-HAPPY-22-cardapio-categorias.md)

*[← Índice docTecnica](README.md)*

Perfil **cliente** — autocadastro e login ([INF-03](INF-03-autenticacao.md)). Foco: **cardápio**, **carrinho**, **checkout**, **acompanhamento** e **Fale conosco**.

---

## 1. Telas principais

| Tela | Conteúdo e contratos |
|------|----------------------|
| **Cardápio** | `GET /menu` — seções por categoria ([DB-04 § 1](DB-04-catalogo-checkout-cliente.md)); produto com foto, nome, preço, `+` quantidade; itens pausados/bloqueados cinza |
| **Carrinho** | Estado local ([DB-04 § 6](DB-04-catalogo-checkout-cliente.md)); editar quantidades; subtotal |
| **Checkout / pagamento** | Endereço (lista salva + novo), taxa fixa de entrega, forma de pagamento, **Finalizar** → `POST /orders` ([FLUX-HAPPY-21](../fluxos/happypaths/FLUX-HAPPY-21-checkout-pagamento.md)) |
| **Pedido recebido (sucesso)** | `#display_number`, link para acompanhar |
| **Acompanhar pedido** | `GET /orders/{id}` + WebSocket da timeline ([INF-01](INF-01-tempo-real.md)); estados em linguagem natural ([FLUX-HAPPY-06](../fluxos/happypaths/FLUX-HAPPY-06-timeline.md)) |
| **Fale conosco** | WhatsApp da loja (`GET /store/contact-links`); na timeline do pedido, mesmo canal com texto incluindo `#display_number` ([DB-01](DB-01-canal-atendimento.md)) |
| **Histórico** | `GET /orders/me`; refazer — [DB-04 § 8](DB-04-catalogo-checkout-cliente.md) |
| **Conta** | Endereços, formas de pagamento, logout |

---

## 2. O que o cliente não tem (MVP)

- **Sem** acesso a timers operacionais internos (fila cozinha, balcão) nem colunas de SLA — só **timeline** e mensagens de estado.
- **Sem** contato direto com o motoboy por WhatsApp no app; contato é **loja** ([DB-01](DB-01-canal-atendimento.md)).

---

## 3. Sessão de compra e loja fechada

- `shoppingSessionId` em `GET /menu` quando loja aberta; permite finalizar após fechamento dentro do TTL ([DB-04 § 7](DB-04-catalogo-checkout-cliente.md)).
- Sem sessão válida e loja fechada: cardápio cinza — [FLUX-SAD-14](../fluxos/sadPaths/FLUX-SAD-14-loja-fechada.md).

---

*[← Índice docTecnica](README.md)*
