# FLUX-SAD-19-rota-motoboy
### Motoboy reporta imprevisto em rota — gerente confirma e aciona refazimento

**Gravidade:** 🔴 · **Perfis:** 🛵 🔑 👤 👨‍🍳

[← Índice sad path](README.md)

---

**Perfis:** 🛵 Motoboy · 🔑 Gerente · 👤 Cliente · 👨‍🍳 Cozinha  
**Causa:** Motoboy saiu com o pedido, sofreu um imprevisto e ainda tem acesso ao celular — reporta pelo app. O gerente recebe o alerta, valida e aciona o protocolo de refazimento.  
**Veja também:** [FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md) — quando o motoboy está incapacitado e o gerente precisa agir sem reporte.

```
Motoboy saiu → imprevisto → reporta no app → gerente recebe alerta
→ gerente confirma → pedido vai ao topo do KDS com prioridade → cliente notificado
```

---

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Está em rota — status `SAIU_PARA_ENTREGA` |
| 2 | 🛵 Motoboy | Sofre imprevisto: pneu furado, sem combustível, mal súbito, etc. Não consegue entregar nem devolver |
| 3 | 🛵 Motoboy | Abre o app e clica em **Reportar imprevisto** |
| 4 | 🛵 Motoboy | Seleciona o motivo (ex: *"Pneu furado"*, *"Sem combustível"*, *"Problema de saúde"*) e confirma |
| 5 | Sistema | Cria evento `INCIDENT_REPORTED` em `order_events` e envia alerta ao gerente via `kitchen:{storeId}` |
| 6 | 🔑 Gerente | Vê o alerta no painel: *"🚨 Imprevisto reportado — Pedido #XXXX · Motoboy [nome] · Motivo: [motivo]"* |
| 7 | 🔑 Gerente | Abre o pedido, revisa e clica em **Confirmar e acionar refazimento** |
| 8 | 🔑 Gerente | Edita opcionalmente a mensagem para o cliente (campo pré-preenchido com o texto padrão) |
| 9 | 🔑 Gerente | Confirma a ação |
| 10 | Sistema | Muda o pedido: `status = RECEBIDO`, `priority = true`, `incident_alert = true`, `incident_message` gravado, `motoboy_id = null`, `cancelamento_bloqueado = true`, `remake_seq++` |
| 11 | Sistema | O pedido aparece no **topo da fila do KDS** com ⚡, antes de outros `RECEBIDO`, depois de `EM_PREPARO` em andamento |
| 12 | 👨‍🍳 Cozinha | Vê o pedido com ⚡ no topo — refaz os itens sem ver o nome do cliente |

---

**Notificação ao cliente:**

| # | Quem | O que acontece |
|---|------|----------------|
| 13 | Sistema | Envia evento via WebSocket: `order.incident_alert` com `incidentMessage` |
| 14 | 👤 Cliente | O pedido volta a **"Pedido recebido"** na timeline com ⚡ tag de **prioridade** |
| 15 | Sistema | Exibe a mensagem do gerente em destaque na timeline (padrão: *"Houve um imprevisto com a entrega. Seu pedido foi colocado na fila da cozinha com prioridade e logo chegará! 🙏"*) |
| 16 | 👤 Cliente | **Não pode cancelar** — `cancelamento_bloqueado = true` |
| 17 | 👤 Cliente | Pode clicar em **💬 Fale Conosco** para abrir o WhatsApp da loja |

---

**Itens físicos (fora do app):**

Motoboy **contata o gerente** (reporte no app, WhatsApp ou telefone). O gerente **aciona refazimento** no painel; não há tela de “devolução recebida” nem status por item. Descarte/devolução física é acordada fora do sistema.

| 🔄 Pedido | Mesmo `order_id`, `RECEBIDO` com `priority = true` — topo da fila do KDS com ⚡ |
| 🛵 Fila | Quando `PRONTO`, qualquer motoboy disponível pode assumir |

**Se a cozinha não aceitar o pedido de volta:**

O pedido fica em `RECEBIDO` com prioridade. `cancelamento_bloqueado` impede **somente o cliente** de cancelar. O gerente pode cancelar com estorno pelo painel — clica em **Cancelar pedido**, informa o motivo, e o cliente recebe o estorno e aviso via WebSocket.

---

**Veja também:**  
→ [FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md) — gerente age diretamente sem reporte do motoboy  
→ [FLUX-SAD-21-aviso-cliente](FLUX-SAD-21-aviso-cliente.md) — experiência do cliente  
→ [FLUX-HAPPY-17-whatsapp](../happypaths/FLUX-HAPPY-17-whatsapp.md) — Fale Conosco via WhatsApp

---

> **Impacto:** Alto. O reporte rápido do motoboy permite o gerente agir antes mesmo de o cliente reclamar, reduzindo o tempo total de atraso.

---

> 📌 **Nota técnica resumida:**  
> O reporte do motoboy (`POST /orders/{id}/incident/report`) cria um evento em `order_events` e notifica o gerente via `kitchen:{storeId}`. O sistema não altera o pedido nesse momento — só após a confirmação do gerente. Isso garante validação antes de remover o motoboy e recolocar na fila. O `incident_message` viaja via WebSocket `order.incident_alert` para o canal `order:{orderId}`.  
> → Detalhes: [`../../docTecnica/ORD-01-estados-pedido.md`](../../docTecnica/ORD-01-estados-pedido.md) · [`../../docTecnica/DB-01-canal-atendimento.md`](../../docTecnica/DB-01-canal-atendimento.md)
