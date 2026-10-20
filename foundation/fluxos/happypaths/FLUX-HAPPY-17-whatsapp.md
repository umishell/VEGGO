# FLUX-HAPPY-17-whatsapp
### Cliente acessa Fale Conosco — redirecionamento para WhatsApp da loja

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente  
**Contextos:** Pode ser acionado em duas situações distintas:

| Contexto | De onde vem |
|----------|------------|
| **Imprevisto de entrega** | Dentro da caixa de aviso ❗ na tela completa do pedido (ver [FLUX-SAD-21-aviso-cliente](../sadPaths/FLUX-SAD-21-aviso-cliente.md)) |
| **Contato geral** | Botão Fale Conosco acessível no dashboard principal do cliente, a qualquer momento |

```
Cliente clica em Fale Conosco → app abre WhatsApp da loja → cliente envia mensagem
```

---

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Clica no botão **💬 Fale Conosco** (na caixa de aviso de imprevisto ou no dashboard) |
| 2 | Sistema | Abre o link externo `wa.me/[número da loja]` |
| 3 | 👤 Cliente | WhatsApp é aberto no dispositivo — direto na conversa com o número da loja |
| 4 | 👤 Cliente | Digita e envia sua mensagem normalmente pelo WhatsApp |
| 5 | 🔑 Gerente | Recebe e responde pelo WhatsApp — fora do app de delivery |

---

> **Decisão de produto:** O atendimento ao cliente é feito **inteiramente pelo WhatsApp**, fora do app. O app não armazena, não processa e não exibe mensagens. Isso elimina a necessidade de construir um sistema de chat interno, reduz complexidade técnica e aproveita uma ferramenta já familiar para o gerente.

---

> 📌 **Nota técnica resumida:**  
> O botão Fale Conosco é um simples link `wa.me/[número]`. Nenhum banco de mensagens, nenhum WebSocket, nenhuma tela de chat no app.  
> → Decisão documentada em: [`../../docTecnica/DB-01-canal-atendimento.md`](../../docTecnica/DB-01-canal-atendimento.md)
