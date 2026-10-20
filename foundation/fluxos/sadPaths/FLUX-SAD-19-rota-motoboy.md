# FLUX-SAD-19-rota-motoboy
### Motoboy cancela entrega em trânsito (auto-cancelamento)

**Gravidade:** 🔴 · **Perfis:** 🛵 👤 👨‍🍳 🔑

[← Índice sad path](README.md)

---

**Perfis:** 🛵 Motoboy · 👤 Cliente · 👨‍🍳 Cozinha · 🔑 Gerente  
**Causa:** Motoboy sofreu imprevisto em rota (pneu furado, falta de gasolina, mal súbito) e **ainda tem acesso ao celular** para cancelar ele mesmo.  
**Veja também:** [FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md) — quando o motoboy está incapacitado e o gerente precisa intervir.

```
Motoboy saiu → imprevisto em rota → auto-cancelamento no app
→ lanche descartado → bebidas ficam pendentes de devolução → o mesmo pedido volta para a cozinha → cliente notificado com ❗
```

---

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Está em rota — status `SAIU_PARA_ENTREGA`; lanche e bebidas estão com ele |
| 2 | 🛵 Motoboy | Sofre imprevisto: pneu furado, sem gasolina, mal súbito, etc. |
| 3 | 🛵 Motoboy | Abre o app e clica em **Cancelar entrega / Reportar imprevisto** |
| 4 | Sistema | Exibe tela de confirmação com campo obrigatório de motivo |
| 5 | 🛵 Motoboy | Seleciona o motivo (ex: *"Pneu furado"*, *"Sem combustível"*, *"Problema de saúde"*) e confirma |
| 6 | Sistema | O **mesmo** pedido volta para `RECEBIDO`. Auditoria grava `IMPREVISTO_ENTREGA` com o motivo. `incident_alert` liga. O cliente não pode mais cancelar este pedido |
| 7 | 🔑 Gerente | Recebe alerta no painel: *"Motoboy [nome] reportou imprevisto no pedido #XXXX — Motivo: [motivo]"* |

---

**Protocolo de imprevisto — destino dos itens e do pedido:**

| Item | O que acontece |
|------|----------------|
| 🍔 Lanche | **Descartado** — estava com o motoboy. A auditoria registra a perda. Não há tela de estoque |
| 🥤 Bebidas | Pendência de devolução do motoboy. Sem módulo de estoque |
| 🔄 Pedido | Continua o **mesmo** número. Volta para a fila de espera da cozinha |
| 👨‍🍳 Cozinha | Vê os itens para refazer, **sem nome do cliente** |
| 🛵 Fila | Quando ficar `PRONTO` de novo, outro motoboy pode assumir |

---

**Notificação ao cliente:**

| # | Quem | O que acontece |
|---|------|----------------|
| 8 | Sistema | Ativa a flag `incident_alert` no pedido → ícone **🔴 ❗** aparece no card simplificado |
| 9 | 👤 Cliente | Vê o ícone vermelho de exclamação na tela principal — **sem texto, apenas o ícone** |
| 10 | 👤 Cliente | Clica no card simplificado do pedido |
| 11 | Sistema | Abre a tela completa com **caixa de aviso em destaque no topo** |
| 12 | 👤 Cliente | Lê: *"Houve um imprevisto com a entrega do seu pedido. Seu lanche está sendo refeito e logo chegará. Pedimos desculpas pelo inconveniente! 🙏"* |
| 13 | 👤 Cliente | Vê o botão **💬 Fale Conosco** dentro da caixa de aviso |
| 14 | 👤 Cliente | (Opcional) Clica em **Fale Conosco** → redirecionado para o **WhatsApp da loja** |
| 15 | 👤 Cliente | Envia mensagem pelo próprio WhatsApp — atendimento fora do app |

**Veja também:**  
→ [FLUX-SAD-21-aviso-cliente](FLUX-SAD-21-aviso-cliente.md) — experiência completa do cliente ao receber a notificação e acessar o Fale Conosco  
→ [FLUX-HAPPY-17-whatsapp](../happypaths/FLUX-HAPPY-17-whatsapp.md) — fluxo do Fale Conosco via WhatsApp

---

> **Impacto:** Alto. Atraso, custo de produto (lanche descartado) e retrabalho da cozinha. O motivo fica registrado na auditoria, útil para acompanhar recorrência de imprevistos por motoboy.

---

> 📌 **Nota técnica resumida:**  
> Mesma regra do FLUX-SAD-18-rota-gerente: o mesmo `order_id` volta a `RECEBIDO`, com `incident_alert` e cancelamento bloqueado. O Fale Conosco é um link `wa.me/[número]`.  
> → Detalhes completos: [`../../docTecnica/ORD-01-estados-pedido.md`](../../docTecnica/ORD-01-estados-pedido.md) · [`../../docTecnica/DB-01-canal-atendimento.md`](../../docTecnica/DB-01-canal-atendimento.md)
