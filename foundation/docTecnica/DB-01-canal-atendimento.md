# DB-01 — Canais de Contato e Navegação

**Referenciado por:**  
→ [FLUX-HAPPY-05-venda](../fluxos/happypaths/FLUX-HAPPY-05-venda.md) · [FLUX-HAPPY-06-timeline](../fluxos/happypaths/FLUX-HAPPY-06-timeline.md) · [FLUX-HAPPY-17-whatsapp](../fluxos/happypaths/FLUX-HAPPY-17-whatsapp.md) · [FLUX-SAD-18-rota-gerente](../fluxos/sadPaths/FLUX-SAD-18-rota-gerente.md) · [FLUX-SAD-19-rota-motoboy](../fluxos/sadPaths/FLUX-SAD-19-rota-motoboy.md) · [FLUX-SAD-21-aviso-cliente](../fluxos/sadPaths/FLUX-SAD-21-aviso-cliente.md)

*[← Índice docTecnica](README.md)*

---

## 1. Decisão de produto

O app não implementa chat interno. Todos os contatos entre cliente, motoboy e loja acontecem via **WhatsApp externo** ou **app de navegação externo**. O app apenas monta e entrega o link correto para cada contexto. Nada de mensagens, conversas ou histórico no banco do sistema.

---

## 2. Campos de telefone e configuração

| Campo | Tabela | Tipo | Obrigatório | Quem preenche |
|-------|--------|------|-------------|---------------|
| `whatsapp_number` | `stores` | string (ex: `"5511999999999"`) | Sim — sem ele o botão "Fale com a loja" fica oculto | Gerente em **Configurações → Loja** |
| `phone` | `users` (clientes) | string, nulo | Recomendado — sem ele o botão "WhatsApp do cliente" fica oculto para o motoboy | Cliente no cadastro |
| `phone` | `users` (motoboys) | string | Obrigatório para motoboy — sem ele o botão "Contatar motoboy" fica oculto para o cliente | Gerente ao criar o usuário motoboy |

> O campo `phone` de clientes e motoboys fica em `users.phone`. Motoboys são usuários como qualquer outro — só o perfil difere.

---

## 3. Endpoint unificado de links

```
GET /orders/{orderId}/contact-links
Authorization: Bearer {token}
```

**FGAC aplicada antes de retornar:**

| Perfil | Estado do pedido | Links retornados |
|--------|-----------------|-----------------|
| `CLIENTE` (dono) | Qualquer | `store` sempre; `motoboy` só em `SAIU_PARA_ENTREGA` |
| `MOTOBOY` (alocado) | `SAIU_PARA_ENTREGA` | `customer` |
| `GERENTE_LOJA` | Qualquer | `store`, `customer`, `motoboy` |
| Qualquer outro | — | 403 |

**Resposta de exemplo (cliente em `SAIU_PARA_ENTREGA`):**

```json
{
  "store":   "https://wa.me/5511999990000",
  "motoboy": "https://wa.me/5511888887777?text=Ol%C3%A1%2C+sou+o+cliente+do+pedido+%23abc123.+Posso+falar%3F"
}
```

**Resposta de exemplo (motoboy em `SAIU_PARA_ENTREGA`):**

```json
{
  "customer": "https://wa.me/5511777776666?text=Ol%C3%A1+Maria%2C+sou+o+motoboy+do+seu+pedido+%23abc123.+Estou+a+caminho%21"
}
```

> Se o telefone for `null`, o campo correspondente **não é incluído** na resposta (o frontend oculta o botão).

---

## 4. Endpoint de navegação (motoboy)

```
GET /orders/{orderId}/navigation-links
Authorization: Bearer {token}
```

**FGAC:** somente o motoboy alocado ao pedido, em `SAIU_PARA_ENTREGA`. Qualquer outro perfil ou estado → 403.

O backend monta o endereço a partir da cópia gravada no pedido:

```
{street}, {number} {complement}, {neighborhood}, {city}, {state}, Brasil
```

**Resposta:**

```json
{
  "waze":  "https://waze.com/ul?q=Rua+das+Flores%2C+42+Apto+8%2C+Centro%2C+...",
  "maps":  "https://maps.google.com/?q=Rua+das+Flores%2C+42+Apto+8%2C+Centro%2C+..."
}
```

> O endereço inclui `complement` e `reference_point` quando preenchidos — esses campos compõem a query do mapa e ficam visíveis na tela de entrega do motoboy antes de abrir o app de navegação.  
> O motoboy escolhe Waze ou Google Maps — o app exibe os dois botões sempre que ambos estão disponíveis.

---

## 5. Campos de endereço gravados no pedido (snapshot)

| Campo | Tipo | Obrigatório |
|-------|------|-------------|
| `street` | string | Sim |
| `number` | string | Sim |
| `complement` | string, nulo | Não |
| `neighborhood` | string | Sim |
| `cep` | string | Sim |
| `reference_point` | string, nulo | Não |
| *(cidade e estado)* | string | Sim — derivados do CEP no cadastro |

A cópia completa (incluindo `complement` e `reference_point`) fica na linha do pedido em `orders.delivery_address` (jsonb ou colunas dedicadas). Editar o endereço cadastrado não altera pedidos já criados.

---

## 6. O que o app armazena (e o que não armazena)

| Elemento | Armazenado no sistema? |
|----------|----------------------|
| `whatsapp_number` da loja | ✅ `stores.whatsapp_number` |
| `phone` do cliente | ✅ `users.phone` (nulo se não informado) |
| `phone` do motoboy | ✅ `users.phone` |
| Mensagens trocadas pelo WhatsApp | ❌ Ficam no WhatsApp |
| Histórico de conversas | ❌ Ficam no WhatsApp |
| Rotas percorridas | ❌ Ficam no Waze / Maps |
| `incident_alert` no pedido | ✅ `orders.incident_alert` |

---

*[← Índice docTecnica](README.md)*
