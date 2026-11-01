# FLUX-HAPPY-05-venda
### Venda completa — do carrinho à entrega na porta

**Perfis:** 👤 👨‍🍳 🛵

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente · 👨‍🍳 Cozinha · 🛵 Motoboy

> Este é o **fluxo principal** do sistema. Tudo corre bem, sem nenhum problema.

```
Cliente monta carrinho → finaliza pedido → cozinha prepara → motoboy entrega
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Abre o carrinho com os itens escolhidos |
| 2 | 👤 Cliente | Revisa o resumo: itens, quantidades e valores |
| 3 | 👤 Cliente | Confirma o endereço de entrega (endereço salvo ou digita novo — incluindo complemento e ponto de referência) |
| 3a | Sistema | Se o cliente digitou um endereço novo, **salva automaticamente** no perfil dele |
| 4 | 👤 Cliente | Vê o valor do frete calculado automaticamente pela região |
| 5 | 👤 Cliente | Clica em **Finalizar Pedido** |
| 6 | Sistema | Verifica que todos os itens ainda estão disponíveis |
| 7 | Sistema | Confirma o pagamento (simulado) e registra o pedido com a **cópia completa do endereço** |
| 8 | 👤 Cliente | Vê a tela de confirmação: **"Pedido recebido!"** com número do pedido |
| 9 | 👨‍🍳 Cozinha | Recebe o novo pedido na tela da cozinha (KDS) com um sinal sonoro |
| 10 | 👨‍🍳 Cozinha | Clica em **Aceitar**. O pedido sai da fila e vai para `EM_PREPARO` — é nesse clique que o preparo começa |
| 11 | 👤 Cliente | Vê na timeline: **"Em preparo"** |
| 12 | 👨‍🍳 Cozinha | Clica em **Marcar como pronto** quando embalou o pedido |
| 13 | 🛵 Motoboy | Vê o pedido aparecer na **fila de disponíveis** no app dele |
| 14 | 🛵 Motoboy | Clica em **Assumir e sair** — o pedido vai direto para `SAIU_PARA_ENTREGA` (lock e transição em único passo) |
| 14a | Sistema | Exibe na tela de entrega ativa do motoboy: endereço completo (rua, número, complemento, bairro, ponto de referência) + **atalhos de navegação** e **atalho de contato** |
| 14b | 🛵 Motoboy | Toca em **🗺️ Waze** ou **🗺️ Google Maps** — o app de navegação escolhido abre com o destino já preenchido |
| 14c | 🛵 Motoboy | Pode tocar em **💬 WhatsApp do cliente** — abre conversa no WhatsApp com mensagem de contexto pré-preenchida |
| 15 | 👤 Cliente | Vê na timeline: **"Pedido saiu para entrega"** 🚀 + botão **💬 Contatar motoboy** |
| 16 | 🛵 Motoboy | Chega na casa do cliente, entrega o pedido, clica em **Confirmar entrega** |
| 17 | 👤 Cliente | Vê na timeline: **"Pedido entregue! ✅"** — botão "Contatar motoboy" desaparece |

---

**Linha do tempo resumida:**

```
CLIENTE              COZINHA              MOTOBOY
   │                    │                    │
   ├─ monta carrinho    │                    │
   ├─ finaliza ─────────┤                    │
   │        aceita e prepara                 │
   │                pronto ─────────────── fila
   │                    │           assume e sai
   │                    │      [nav + contato] │
   🚀 saiu p/ entrega   │                    │
   [botão: contatar]    │                    │
   ✅ entregue           │                    │
```

---

**Atalhos de navegação (tela do motoboy — `SAIU_PARA_ENTREGA`):**

| Atalho | Link gerado pelo backend | Quando some |
|--------|--------------------------|-------------|
| 🗺️ **Waze** | `https://waze.com/ul?q=[endereço+completo]` | Ao confirmar entrega |
| 🗺️ **Google Maps** | `https://maps.google.com/?q=[endereço+completo]` | Ao confirmar entrega |
| 💬 **WhatsApp do cliente** | `wa.me/[phone_cliente]?text=Olá+[nome],+sou+o+motoboy+do+seu+pedido...` | Ao confirmar entrega |

> O backend monta esses links via `GET /orders/{id}/navigation-links` (FGAC: só o motoboy alocado, em `SAIU_PARA_ENTREGA`). O app abre o link externo — sem GPS interno, sem rastreamento.

---

**Atalho de contato para o cliente (tela de timeline — `SAIU_PARA_ENTREGA`):**

| Atalho | Link | Quando aparece | Quando some |
|--------|------|----------------|-------------|
| 💬 **Fale com a loja** | `wa.me/[phone_loja]` | A qualquer momento | Nunca some |
| 🛵 **Contatar motoboy** | `wa.me/[phone_motoboy]?text=Olá,+sou+o+cliente+do+pedido+%23[id]...` | Quando `SAIU_PARA_ENTREGA` | Quando `ENTREGUE` |

> Os links são construídos pelo backend via `GET /orders/{id}/contact-links` (FGAC: só o dono do pedido; só retorna `motoboy` quando `SAIU_PARA_ENTREGA`).

---

**Veja também:**  
→ [FLUX-HAPPY-06-timeline](FLUX-HAPPY-06-timeline.md) — detalhe da timeline do cliente  
→ [FLUX-HAPPY-17-whatsapp](FLUX-HAPPY-17-whatsapp.md) — todos os contextos de contato  
→ [FLUX-HAPPY-04-endereco](FLUX-HAPPY-04-endereco.md) — campos do endereço (complemento, ponto de referência)
