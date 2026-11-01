# FLUX-SAD-21-aviso-cliente
### Cliente notificado de imprevisto — aviso na timeline e Fale Conosco

**Gravidade:** 🟠 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente  
**Causa:** Disparado após o gerente acionar o protocolo de imprevisto em rota ([FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md) ou [FLUX-SAD-19-rota-motoboy](FLUX-SAD-19-rota-motoboy.md)).

```
Gerente aciona imprevisto → pedido volta a RECEBIDO com prioridade
→ cliente vê status atualizar + mensagem do gerente na timeline em tempo real
→ pode usar Fale Conosco a qualquer momento
```

---

**O que o cliente vê — na ordem:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | Sistema | Evento de imprevisto dispara: `incident_alert = true`, `priority = true`, `incident_message` definido |
| 2 | Sistema | WebSocket empurra o evento `order.incident_alert` para o canal `order:{orderId}` do cliente em tempo real |
| 3 | 👤 Cliente | Na timeline do pedido, o estado muda de volta para **"Pedido recebido"** com ⚡ tag de **prioridade** |
| 4 | Sistema | A mensagem do gerente aparece em destaque como novo evento na timeline (ex: *"Houve um imprevisto com a entrega. Seu pedido foi colocado na fila da cozinha com prioridade e logo chegará! 🙏"*) |
| 5 | 👤 Cliente | No card simplificado da tela principal, vê o indicador ⚡ e o estado "Recebido — prioridade" |
| 6 | 👤 Cliente | **Não pode cancelar** o pedido (`cancelamento_bloqueado = true`) |
| 7 | 👤 Cliente | Pode clicar em **💬 Fale Conosco** a qualquer momento |

---

**Parte 2 — Fale Conosco:**

| # | Quem | O que acontece |
|---|------|----------------|
| 8 | 👤 Cliente | Clica em **💬 Fale Conosco** — disponível no card do pedido ou no dashboard principal |
| 9 | Sistema | Abre o link externo `wa.me/[número da loja]` |
| 10 | 👤 Cliente | WhatsApp abre pronto para enviar mensagem |
| 11 | 🔑 Gerente | Atende pelo WhatsApp — fora do app de delivery |

> O atendimento acontece **fora do app**. O app não armazena nem processa mensagens.

---

**Veja também:**  
→ [FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md) — gerente aciona o imprevisto diretamente  
→ [FLUX-SAD-19-rota-motoboy](FLUX-SAD-19-rota-motoboy.md) — motoboy reporta, gerente confirma  
→ [FLUX-HAPPY-17-whatsapp](../happypaths/FLUX-HAPPY-17-whatsapp.md) — fluxo completo do Fale Conosco

---

> **Impacto:** Médio. A mensagem do gerente e o indicador ⚡ de prioridade comunicam o problema de forma clara e positiva — o cliente sabe que o pedido não foi abandonado e está sendo refeito com urgência.

---

> 📌 **Nota técnica resumida:**  
> `incident_alert: boolean` e `incident_message: string` ficam no registro do pedido (`orders`). O WebSocket empurra o evento `order.incident_alert { orderId, incidentAlert, incidentMessage }` para o canal `order:{orderId}`. O frontend exibe `incidentMessage` como novo item da timeline, com ⚡ e destaque visual. O Fale Conosco é um link externo `wa.me` — sem banco de mensagens, sem chat interno.  
> → Detalhes: [`../../docTecnica/DB-01-canal-atendimento.md`](../../docTecnica/DB-01-canal-atendimento.md) · [`../../docTecnica/INF-01-tempo-real.md`](../../docTecnica/INF-01-tempo-real.md)
