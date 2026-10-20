# FLUX-SAD-21-aviso-cliente
### Cliente notificado de imprevisto — ícone de alerta e Fale Conosco via WhatsApp

**Gravidade:** 🟠 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente  
**Causa:** Disparado automaticamente pelos fluxos [FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md) ou [FLUX-SAD-19-rota-motoboy](FLUX-SAD-19-rota-motoboy.md) quando ocorre um imprevisto na entrega.

```
Imprevisto detectado → ícone ❗ aparece no card do cliente
→ cliente clica → caixa de aviso → Fale Conosco → abre WhatsApp da loja
```

---

**Parte 1 — Ícone de alerta no card simplificado:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | Sistema | Ativa `incident_alert: true` no pedido afetado (via FLUX-SAD-18-rota-gerente ou FLUX-SAD-19-rota-motoboy) |
| 2 | 👤 Cliente | Na tela principal, o card simplificado do pedido exibe o ícone **🔴 ❗** no canto |
| 3 | 👤 Cliente | **Não vê texto explicativo no card** — apenas o ícone vermelho de exclamação |

---

**Parte 2 — Tela completa do pedido:**

| # | Quem | O que acontece |
|---|------|----------------|
| 4 | 👤 Cliente | Clica no card simplificado do pedido |
| 5 | Sistema | Abre a tela completa de detalhes do pedido |
| 6 | Sistema | Exibe **caixa de aviso em destaque no topo** da tela (antes dos detalhes do pedido) |
| 7 | 👤 Cliente | Lê a mensagem: *"Houve um imprevisto com a entrega do seu pedido. Seu lanche está sendo refeito e logo chegará. Pedimos desculpas pelo inconveniente! 🙏"* |
| 8 | 👤 Cliente | Vê o botão **💬 Fale Conosco** dentro da caixa de aviso |

---

**Parte 3 — Fale Conosco (via imprevisto ou via dashboard):**

| # | Quem | O que acontece |
|---|------|----------------|
| 9 | 👤 Cliente | Clica em **💬 Fale Conosco** — dentro da caixa de aviso do pedido **ou** no dashboard principal |
| 10 | Sistema | Redireciona o cliente para o **WhatsApp da loja** via link externo (`wa.me/[número]`) |
| 11 | 👤 Cliente | WhatsApp abre com a conversa da loja pronta para digitar |
| 12 | 👤 Cliente | Envia sua mensagem pelo próprio WhatsApp |

> O atendimento acontece **fora do app**, diretamente no WhatsApp. O app não armazena nem gerencia mensagens.

---

**Veja também:**  
→ [FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md) — gerente desaloca motoboy incapacitado (dispara este fluxo)  
→ [FLUX-SAD-19-rota-motoboy](FLUX-SAD-19-rota-motoboy.md) — motoboy cancela em trânsito (dispara este fluxo)  
→ [FLUX-HAPPY-17-whatsapp](../happypaths/FLUX-HAPPY-17-whatsapp.md) — fluxo completo do Fale Conosco via WhatsApp

---

> **Impacto:** Médio — afeta a experiência do cliente. O ícone ❗ e a caixa de aviso comunicam o problema sem necessidade de ação imediata. O Fale Conosco conecta o cliente à loja pelo WhatsApp sem adicionar complexidade ao sistema.

---

> 📌 **Nota técnica resumida:**  
> Flag `incident_alert: boolean` no registro do pedido controla a exibição do ícone `❗`. O botão Fale Conosco é um link externo `wa.me/[número]` — sem banco de mensagens, sem WebSocket, sem lógica de chat no app.  
> → Detalhes: [`../../docTecnica/DB-01-canal-atendimento.md`](../../docTecnica/DB-01-canal-atendimento.md)
