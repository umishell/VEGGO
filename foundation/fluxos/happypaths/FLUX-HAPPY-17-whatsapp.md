# FLUX-HAPPY-17-whatsapp
### Fale conosco e WhatsApp na entrega

**Perfis:** 👤 🛵

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente · 🛵 Motoboy  

O app **não** tem chat interno. Os atalhos abrem o **WhatsApp** (ou apps de mapa, no motoboy) com link montado pelo backend.

| Onde no app | Botão (rótulo sugerido) | Endpoint |
|-------------|-------------------------|----------|
| Menu **Fale conosco** | 💬 **WhatsApp da loja** | `GET /store/contact-links` |
| **Acompanhar pedido** (timeline) | 💬 **Fale com a loja** | `GET /orders/{id}/contact-links` → `store` |
| Tela do motoboy em rota | 💬 **WhatsApp do cliente** | `GET /orders/{id}/contact-links` → `customer` |

```
Usuário toca no atalho → backend devolve wa.me → app abre WhatsApp externo
```

---

## Fale conosco — WhatsApp da loja (sem pedido aberto)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | No menu (ou perfil), abre **Fale conosco** |
| 2 | 👤 Cliente | Vê texto de apoio (horário, dúvidas sobre pedido, etc.) |
| 3 | 👤 Cliente | Toca **💬 WhatsApp da loja** |
| 4 | Sistema | Chama `GET /store/contact-links` e abre o link retornado (**sem** número de pedido no texto — contato geral) |
| 5 | 👤 Cliente | Conversa com a loja no WhatsApp — fora do app |

> Mesmo número configurado pelo gerente em **Configurações → Loja** (`stores.whatsapp_number`). Ver [FLUX-HAPPY-12-horario](FLUX-HAPPY-12-horario.md) Parte 0.

---

## Acompanhar pedido — Fale com a loja (sempre)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Na timeline de **qualquer** pedido ativo ou histórico recente, vê **💬 Fale com a loja** |
| 2 | Sistema | `GET /orders/{id}/contact-links` → campo `store` |
| 3 | Sistema | Abre WhatsApp da loja com **mensagem pré-preenchida** citando o **número do pedido** (`display_number`, ex. `#42`) para o gerente identificar o contexto |

Botão **permanece visível** em todos os estados (`RECEBIDO` … `ENTREGUE`). **Não** há atalho direto ao WhatsApp do motoboy para o cliente — quem precisa falar com o entregador pede à **loja** por esse canal.

---

## Cliente → Loja após imprevisto ⚡

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | No aviso ⚡ da timeline, toca **Fale com a loja** |
| 2 | Sistema | Mesmo fluxo do `store`: link da loja com texto que inclui o pedido e o contexto do imprevisto |

---

## Motoboy → Cliente (durante a entrega)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Na tela de acompanhamento, toca **💬 WhatsApp do cliente** |
| 2 | Sistema | `GET /orders/{id}/contact-links` → `customer` |
| 3 | Sistema | Abre WhatsApp com saudação e número do pedido |

---

> **Privacidade:** O **cliente** não vê o telefone nem link direto ao WhatsApp do **motoboy**. O **motoboy alocado** em `SAIU_PARA_ENTREGA` recebe `customer` para contatar o cliente. FGAC no backend — 403 fora dessas condições.

> **Sem `whatsapp_number`:** atalhos da **loja** ocultos ou “Contato indisponível”. **Sem `phone` do cliente:** botão **WhatsApp do cliente** oculto para o motoboy.

---

**Veja também:**  
→ [FLUX-HAPPY-06-timeline](FLUX-HAPPY-06-timeline.md)  
→ [FLUX-SAD-21-aviso-cliente](../sadPaths/FLUX-SAD-21-aviso-cliente.md)  
→ [`../../docTecnica/DB-01-canal-atendimento.md`](../../docTecnica/DB-01-canal-atendimento.md)

---

> 📌 **Nota técnica:** Frontend **nunca** monta `wa.me` manualmente. **Fale conosco** geral: `GET /store/contact-links`. **Timeline** (pedido): `GET /orders/{id}/contact-links` → `store` com texto do pedido.
