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
| 3 | 👤 Cliente | Na etapa de entrega, confirma endereço na **combobox** (padrão já selecionado, outro salvo ou **novo**) |
| 3a | Sistema | Endereço novo usado no pedido é **salvo** no perfil |
| 4 | 👤 Cliente | Escolhe **forma de pagamento**: uma salva (autofill) ou **cadastra na hora** (mock — ex.: rótulo + últimos dígitos simulados) |
| 5 | 👤 Cliente | Vê a **taxa de entrega fixa** da loja no resumo (mesmo valor para qualquer endereço) |
| 6 | 👤 Cliente | Clica em **Finalizar Pedido** |
| 7 | Sistema | Revalida itens, loja e limites diários |
| 8 | Sistema | Confirma **pagamento simulado** e registra pedido + cópia do endereço |
| 9 | 👤 Cliente | Vê **"Pedido recebido!"** com **`display_number`** — detalhe em [FLUX-HAPPY-21-checkout-pagamento](FLUX-HAPPY-21-checkout-pagamento.md) |
| 10 | 👨‍🍳 Cozinha | Pedido na **fila esquerda** do KDS ([FLUX-HAPPY-11-item-cozinha](FLUX-HAPPY-11-item-cozinha.md)) |
| 11 | 👨‍🍳 Cozinha | Seleciona pedido na lista → **Preparar** → prepara → **Concluir preparo** → anota `display_number` na embalagem ([FLUX-HAPPY-11](FLUX-HAPPY-11-item-cozinha.md)) |
| 12 | 👤 Cliente | Timeline: **"Em preparo"** |
| 13 | 🛵 Motoboy | **Fila de prontos** com `display_number` |
| 14 | 🛵 Motoboy | Confere embalagem, **Aceitar pedido** → **Acompanhamento** (nome, telefone, WhatsApp, Waze, Maps) |
| 14a | Sistema | Exibe na tela de entrega ativa do motoboy: endereço completo (rua, número, complemento, bairro, ponto de referência) + **atalhos de navegação** e **atalho de contato** |
| 14b | 🛵 Motoboy | Toca em **🗺️ Waze** ou **🗺️ Google Maps** — o app de navegação escolhido abre com o destino já preenchido |
| 14c | 🛵 Motoboy | Pode tocar em **💬 WhatsApp do cliente** — abre conversa no WhatsApp com mensagem de contexto pré-preenchida |
| 15 | 👤 Cliente | Vê na timeline: **"Pedido saiu para entrega"** 🚀 + **🛵 WhatsApp do motoboy** (e **Fale com a loja** continua visível) |
| 16 | 🛵 Motoboy | **Finalizar entrega** → volta à fila de prontos |
| 17 | 👤 Cliente | Timeline: **"Pedido entregue! ✅"** |

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
