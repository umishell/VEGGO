# FLUX-HAPPY-20-motoboy-app
### Motoboy — fila de prontos, aceite no balcão e acompanhamento

**Perfis:** 🛵

[← Índice happy path](README.md)

---

**Perfis:** 🛵 Motoboy  
**Veja também:** [FLUX-HAPPY-16-turno](FLUX-HAPPY-16-turno.md) · [FLUX-HAPPY-05-venda](FLUX-HAPPY-05-venda.md) · [INF-04-ui-operacao-cliente](../../docTecnica/INF-04-ui-operacao-cliente.md) § 2

```
Fila PRONTO (display_number) → conferir recibo no pacote (display_number + nome) → Aceitar pedido → acompanhamento → Finalizar entrega → volta à fila
```

---

## Fila de prontos (tela principal)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Com turno ativo, vê lista de pedidos **`PRONTO`** com **`display_number`** em destaque |
| 2 | 👨‍🍳 Cozinha | Já **grampeou o recibo** no pacote: mesmo **`display_number`**, nome do cliente, itens/valores, total e data ([FLUX-HAPPY-11-item-cozinha](FLUX-HAPPY-11-item-cozinha.md)) |
| 3 | 🛵 Motoboy | No balcão, confere **`display_number`** e **nome do cliente** no **recibo** com a **fila** no app, seleciona o pedido correto |
| 4 | 🛵 Motoboy | Toca **Aceitar pedido** |
| 5 | Sistema | `POST /orders/{id}/take` → `SAIU_PARA_ENTREGA`; abre **Acompanhamento da entrega** |

---

## Acompanhamento da entrega

| Campo / ação | Detalhe |
|--------------|---------|
| Nome do cliente | `users.name` do dono do pedido |
| Telefone | Visível na tela |
| **WhatsApp** | Link com contexto do pedido |
| **Waze** / **Google Maps** | Endereço completo do snapshot |
| **Finalizar entrega** | `ENTREGUE` → retorno automático à **Fila de prontos** |

> Durante entrega ativa, o app permanece no acompanhamento até finalizar ou protocolo de imprevisto (SAD-18/19).

---

## Histórico

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Aba **Histórico** — pedidos que já **aceitou** (`SAIU_PARA_ENTREGA`+) |
| 2 | Sistema | Mostra entregues (`ENTREGUE`) e cancelados pelo gerente após aceite |

Não lista pedidos só vistos na fila sem aceitar (ex.: disputa — [FLUX-HAPPY-15-disputa](FLUX-HAPPY-15-disputa.md)).
