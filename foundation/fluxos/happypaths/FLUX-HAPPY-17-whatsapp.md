# FLUX-HAPPY-17-whatsapp
### nos contacte, Fale conosco e WhatsApp na entrega

**Perfis:** 👤 🛵

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente · 🛵 Motoboy  

O app **não** tem chat interno. Todos os atalhos abrem o **WhatsApp** (ou apps de mapa, no motoboy) com link montado pelo backend.

| Onde no app | Botão (rótulo sugerido) | Endpoint |
|-------------|-------------------------|----------|
| Menu **nos contacte** / **Fale conosco** | 💬 **WhatsApp da loja** | `GET /store/contact-links` |
| **Acompanhar pedido** (timeline) | 💬 **Fale com a loja** | `GET /orders/{id}/contact-links` → `store` |
| **Acompanhar pedido** em `SAIU_PARA_ENTREGA` | 🛵 **WhatsApp do motoboy** | `GET /orders/{id}/contact-links` → `motoboy` |
| Tela do motoboy em rota | 💬 **WhatsApp do cliente** | `GET /orders/{id}/contact-links` → `customer` |

```
Usuário toca no atalho → backend devolve wa.me → app abre WhatsApp externo
```

---

## nos contacte — WhatsApp da loja (sem pedido aberto)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | No menu (ou perfil), abre **nos contacte** / **Fale conosco** |
| 2 | 👤 Cliente | Vê texto de apoio (horário, dúvidas sobre pedido, etc.) |
| 3 | 👤 Cliente | Toca **💬 WhatsApp da loja** |
| 4 | Sistema | Chama `GET /store/contact-links` e abre o link retornado |
| 5 | 👤 Cliente | Conversa com a loja no WhatsApp — fora do app |

> Mesmo número configurado pelo gerente em **Configurações → Loja** (`stores.whatsapp_number`). Ver [FLUX-HAPPY-12-horario](FLUX-HAPPY-12-horario.md) Parte 0.

---

## Acompanhar pedido — Fale com a loja (sempre)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Na timeline de **qualquer** pedido ativo ou histórico recente, vê **💬 Fale com a loja** |
| 2 | Sistema | `GET /orders/{id}/contact-links` → campo `store` |
| 3 | Sistema | Abre WhatsApp; texto pode incluir `display_number` ou id do pedido (editável) |

Botão **permanece visível** em todos os estados (`RECEBIDO` … `ENTREGUE`).

---

## Acompanhar pedido — WhatsApp do motoboy (só na entrega)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Pedido em **`SAIU_PARA_ENTREGA`** — aparece **🛵 WhatsApp do motoboy** (ou **Contatar motoboy**) |
| 2 | Sistema | `GET /orders/{id}/contact-links` → campo `motoboy` (só neste estado) |
| 3 | Sistema | Abre `wa.me/[telefone_motoboy]?text=...` com contexto do pedido |
| 4 | Sistema | Botão **some** em `ENTREGUE`, cancelamento ou imprevisto (motoboy desvinculado) |

Detalhe na timeline: [FLUX-HAPPY-06-timeline](FLUX-HAPPY-06-timeline.md).

---

## Cliente → Loja após imprevisto ⚡

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | No aviso ⚡ da timeline, toca **Fale com a loja** |
| 2 | Sistema | Link `store` com mensagem pré-preenchida citando o pedido e o imprevisto |

---

## Motoboy → Cliente (durante a entrega)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Na tela de acompanhamento, toca **💬 WhatsApp do cliente** |
| 2 | Sistema | `GET /orders/{id}/contact-links` → `customer` |
| 3 | Sistema | Abre WhatsApp com saudação e número do pedido |

---

> **Privacidade:** Telefone do motoboy só para o **dono** do pedido em `SAIU_PARA_ENTREGA`. Telefone do cliente só para o **motoboy alocado** no mesmo estado. FGAC no backend — 403 fora dessas condições.

> **Sem `whatsapp_number`:** atalhos da **loja** ocultos ou “Contato indisponível”. **Sem `phone` do motoboy:** botão do motoboy oculto na timeline do cliente.

---

**Veja também:**  
→ [FLUX-HAPPY-06-timeline](FLUX-HAPPY-06-timeline.md)  
→ [FLUX-SAD-21-aviso-cliente](../sadPaths/FLUX-SAD-21-aviso-cliente.md)  
→ [`../../docTecnica/DB-01-canal-atendimento.md`](../../docTecnica/DB-01-canal-atendimento.md)

---

> 📌 **Nota técnica:** Frontend **nunca** monta `wa.me` manualmente. Loja geral: `GET /store/contact-links`. Por pedido: `GET /orders/{id}/contact-links`.
