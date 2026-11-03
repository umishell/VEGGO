# FLUX-SAD-18-rota-gerente
### Gerente aciona imprevisto em rota — pedido refaz com prioridade

**Gravidade:** 🔴 · **Perfis:** 🔑 🛵 👤 👨‍🍳

[← Índice sad path](README.md)

---

**Perfis:** 🔑 Gerente · 🛵 Motoboy · 👤 Cliente · 👨‍🍳 Cozinha  
**Causa:** Motoboy saiu com o pedido e teve um imprevisto — não consegue entregar nem devolver os itens à loja. O motoboy está incapacitado ou não tem como reportar; o gerente detecta o problema e aciona o protocolo.  
**Veja também:** [FLUX-SAD-19-rota-motoboy](FLUX-SAD-19-rota-motoboy.md) — quando o motoboy tem acesso ao app e reporta o imprevisto antes do gerente agir.

```
Motoboy saiu → imprevisto → gerente detecta → aciona refazimento com prioridade
→ pedido vai ao topo do KDS → cozinha refaz → novo motoboy entrega → cliente recebe aviso
```

---

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Saiu com o pedido — status `SAIU_PARA_ENTREGA` |
| 2 | 🛵 Motoboy | Imprevisto acontece. Não consegue entregar nem devolver os itens à loja |
| 3 | 🔑 Gerente | Percebe o problema: cliente reclama, tempo de entrega passou, ou contato externo (ligação do motoboy, notificação) |
| 4 | 🔑 Gerente | Abre o painel → localiza o pedido → clica em **Acionar imprevisto** |
| 5 | 🔑 Gerente | Preenche o motivo (campo obrigatório — aparece na timeline do pedido) e opcionalmente edita a mensagem para o cliente |
| 6 | 🔑 Gerente | Confirma a ação |
| 7 | Sistema | Muda o pedido: `status = RECEBIDO`, `priority = true`, `incident_alert = true`, `incident_message` gravado, `motoboy_id = null`, `cancelamento_bloqueado = true`, `remake_seq++` |
| 8 | Sistema | O pedido aparece no **topo da fila do KDS** com indicador ⚡, antes de outros `RECEBIDO`, depois de qualquer `EM_PREPARO` já em andamento |
| 9 | 👨‍🍳 Cozinha | Vê o pedido com ⚡ no topo — refaz os itens. O nome do cliente **não aparece** no KDS |
| 10 | 🔑 Gerente | No painel, o pedido exibe cor 🔴 vermelho com ⚡ (imprevisto em refazimento) |

---

**Notificação ao cliente:**

| # | Quem | O que acontece |
|---|------|----------------|
| 11 | Sistema | Envia evento via WebSocket: `order.incident_alert` com `incidentMessage` |
| 12 | 👤 Cliente | O pedido volta a **"Pedido recebido"** na timeline com ⚡ tag de **prioridade** |
| 13 | Sistema | Exibe a mensagem do gerente em destaque na timeline (padrão: *"Houve um imprevisto com a entrega do seu pedido. Ele foi colocado na fila da cozinha com prioridade e logo chegará. Pedimos desculpas pelo transtorno! 🙏"*) |
| 14 | 👤 Cliente | **Não pode cancelar** o pedido (`cancelamento_bloqueado = true`) |
| 15 | 👤 Cliente | Pode clicar em **💬 Fale Conosco** a qualquer momento para abrir o WhatsApp da loja |

---

**Itens físicos (fora do app):**

O sistema **não** registra descarte nem devolução linha a linha. Motoboy e gerente combinam (WhatsApp, telefone ou presencial): descartar comida que não volta, devolver depois itens duráveis à loja quando possível. O app só recoloca o pedido na fila da cozinha com ⚡.

| 🔄 Pedido | Mesmo `order_id`, `RECEBIDO` com `priority = true` — topo da fila do KDS com ⚡ |
| 🛵 Próximo motoboy | Quando `PRONTO`, qualquer motoboy disponível pode assumir normalmente |

**Se a cozinha não aceitar o pedido de volta:**

O pedido fica em `RECEBIDO` com prioridade. `cancelamento_bloqueado` impede **somente o cliente** de cancelar. O gerente pode cancelar com estorno a qualquer momento pelo painel — clica em **Cancelar pedido** no detalhe, informa o motivo, e o cliente recebe o estorno e aviso via WebSocket.

---

**Veja também:**  
→ [FLUX-SAD-19-rota-motoboy](FLUX-SAD-19-rota-motoboy.md) — motoboy reporta o imprevisto pelo app antes do gerente agir  
→ [FLUX-SAD-21-aviso-cliente](FLUX-SAD-21-aviso-cliente.md) — experiência completa do cliente ao receber o aviso  
→ [FLUX-HAPPY-17-whatsapp](../happypaths/FLUX-HAPPY-17-whatsapp.md) — Fale Conosco via WhatsApp

---

> **Impacto:** Muito alto. Atraso significativo, custo de produto (lanche descartado) e retrabalho da cozinha. Motivo e mensagem ficam visíveis na timeline do pedido e no detalhe do painel.

---

> 📌 **Nota técnica resumida:**  
> O mesmo `order_id` volta a `RECEBIDO` com `priority = true` e `incident_message` definido. A fila do KDS ordena por `priority DESC, created_at ASC` — pedidos com prioridade aparecem antes. `cancelamento_bloqueado = true` impede o cliente de cancelar. O `incident_message` viaja via WebSocket `order.incident_alert` para o canal `order:{orderId}`. O Fale Conosco é `wa.me/[número]`.  
> → Detalhes: [`../../docTecnica/ORD-01-estados-pedido.md`](../../docTecnica/ORD-01-estados-pedido.md) · [`../../docTecnica/DB-01-canal-atendimento.md`](../../docTecnica/DB-01-canal-atendimento.md)
