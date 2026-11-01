# FLUX-HAPPY-17-whatsapp
### Fale Conosco e contatos de entrega — redirecionamento para WhatsApp

**Perfis:** 👤 🛵

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente · 🛵 Motoboy  
**Contextos:** Três situações distintas, com links diferentes.

| # | Contexto | Quem aciona | Visível quando |
|---|----------|-------------|----------------|
| 1 | **Cliente → Loja (geral)** | Botão 💬 Fale com a loja no dashboard ou tela do pedido | Sempre |
| 2 | **Cliente → Loja (imprevisto)** | Botão 💬 Fale com a loja dentro do aviso ⚡ de imprevisto | Imprevisto ativo |
| 3 | **Cliente → Motoboy** | Botão 🛵 Contatar motoboy na timeline do pedido | Só em `SAIU_PARA_ENTREGA` |
| 4 | **Motoboy → Cliente** | Botão 💬 WhatsApp do cliente na tela de entrega ativa | Só em `SAIU_PARA_ENTREGA` |

```
Usuário toca no botão → backend monta link → app abre WhatsApp externo com contexto pré-preenchido
```

---

## Contexto 1 — Cliente → Loja (contato geral)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Toca em **💬 Fale com a loja** no dashboard ou no card de qualquer pedido |
| 2 | Sistema | Abre `wa.me/[número_loja]` sem texto pré-preenchido |
| 3 | 👤 Cliente | WhatsApp abre na conversa da loja — digita livremente |
| 4 | 🔑 Gerente | Recebe e responde pelo WhatsApp, fora do app |

---

## Contexto 2 — Cliente → Loja (após imprevisto em rota)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Dentro do aviso ⚡ na timeline, toca em **💬 Fale com a loja** |
| 2 | Sistema | Abre `wa.me/[número_loja]?text=Olá,+tive+um+problema+com+meu+pedido+%23[orderId]...` |
| 3 | 👤 Cliente | WhatsApp abre com a mensagem de contexto já preenchida (editável) — o gerente já sabe o número do pedido |

---

## Contexto 3 — Cliente → Motoboy (durante a entrega)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Na timeline, com pedido em **`SAIU_PARA_ENTREGA`**, toca em **🛵 Contatar motoboy** |
| 2 | Sistema | Chama `GET /orders/{id}/contact-links` e retorna o link do motoboy |
| 3 | Sistema | Abre `wa.me/[telefone_motoboy]?text=Olá,+sou+o+cliente+do+pedido+%23[orderId].+Posso+falar?` |
| 4 | 🛵 Motoboy | Recebe no próprio WhatsApp pessoal — fora do app de delivery |
| 5 | Sistema | Botão desaparece quando o pedido muda para `ENTREGUE` ou volta a `RECEBIDO` (imprevisto) |

---

## Contexto 4 — Motoboy → Cliente (durante a entrega)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Na tela de entrega ativa, toca em **💬 WhatsApp do cliente** |
| 2 | Sistema | Chama `GET /orders/{id}/contact-links` e retorna o link do cliente |
| 3 | Sistema | Abre `wa.me/[telefone_cliente]?text=Olá+[nome_cliente],+sou+o+motoboy+do+seu+pedido+%23[orderId].+Estou+a+caminho!` |
| 4 | 👤 Cliente | Recebe no WhatsApp pessoal |
| 5 | Sistema | Botão desaparece ao clicar em **Confirmar entrega** |

---

> **Privacidade:** O número do cliente é visível **somente** ao motoboy alocado ao pedido dele, em `SAIU_PARA_ENTREGA`. O número do motoboy é visível **somente** ao dono do pedido, em `SAIU_PARA_ENTREGA`. O endpoint `contact-links` aplica FGAC antes de retornar os links — qualquer outro perfil ou estado recebe 403.

---

> **Sem `whatsapp_number` da loja:** o botão "Fale com a loja" fica oculto. Ver configuração em [FLUX-HAPPY-12-horario](FLUX-HAPPY-12-horario.md) Parte 0.  
> **Sem telefone do cliente:** o botão "WhatsApp do cliente" fica oculto na tela do motoboy.  
> **Sem telefone do motoboy:** o botão "Contatar motoboy" fica oculto na timeline do cliente.

---

**Veja também:**  
→ [FLUX-HAPPY-06-timeline](FLUX-HAPPY-06-timeline.md) — botões de contato na timeline  
→ [FLUX-HAPPY-05-venda](FLUX-HAPPY-05-venda.md) — atalhos do motoboy na tela de entrega ativa  
→ [FLUX-SAD-21-aviso-cliente](../sadPaths/FLUX-SAD-21-aviso-cliente.md) — imprevisto e Fale Conosco  
→ [`../../docTecnica/DB-01-canal-atendimento.md`](../../docTecnica/DB-01-canal-atendimento.md) — links, campos e endpoints

---

> 📌 **Nota técnica resumida:**  
> Todos os links são montados pelo backend. O frontend nunca concatena telefone ou número de pedido diretamente. Endpoint único: `GET /orders/{id}/contact-links` retorna um objeto com os links disponíveis para o perfil autenticado no estado atual do pedido. O parâmetro `context` controla o texto pré-preenchido.  
> → Detalhes: [`../../docTecnica/DB-01-canal-atendimento.md`](../../docTecnica/DB-01-canal-atendimento.md)
