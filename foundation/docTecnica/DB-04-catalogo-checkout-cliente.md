# DB-04 — Cardápio (categorias), checkout, endereços e pagamento simulado

**Referenciado por:**  
→ [FLUX-HAPPY-02-login](../fluxos/happypaths/FLUX-HAPPY-02-login.md) · [FLUX-HAPPY-03-carrinho](../fluxos/happypaths/FLUX-HAPPY-03-carrinho.md) · [FLUX-HAPPY-04-endereco](../fluxos/happypaths/FLUX-HAPPY-04-endereco.md) · [FLUX-HAPPY-05-venda](../fluxos/happypaths/FLUX-HAPPY-05-venda.md) · [FLUX-HAPPY-21-checkout-pagamento](../fluxos/happypaths/FLUX-HAPPY-21-checkout-pagamento.md) · [FLUX-HAPPY-22-cardapio-categorias](../fluxos/happypaths/FLUX-HAPPY-22-cardapio-categorias.md) · [FLUX-HAPPY-11-item-cozinha](../fluxos/happypaths/FLUX-HAPPY-11-item-cozinha.md) · [FLUX-HAPPY-20-motoboy-app](../fluxos/happypaths/FLUX-HAPPY-20-motoboy-app.md)

*[← Índice docTecnica](README.md)*

PostgreSQL. Complementa [DB-02](DB-02-disponibilidade-itens.md) (`products` § 1, caps § 1.2, loja).

Esquema físico de **`products`**: [DB-02 § 1](DB-02-disponibilidade-itens.md). Rotas HTTP: [INF-02 § 11](INF-02-http-vs-websocket.md).

**Fora do MVP:** cupom/promoção; **zonas de frete** (`delivery_zones`). Total do pedido = subtotal das linhas + **taxa de entrega fixa** da loja ([§ 2](#2-pedido--código-visível--orders), `stores.fixed_delivery_fee_cents` em [DB-02 § 2](DB-02-disponibilidade-itens.md)).

---

## 1. Categorias do cardápio — `product_categories`

Colunas mínimas:

| Coluna | Tipo | Regra |
|--------|------|-------|
| `id` | UUID | PK |
| `store_id` | UUID | FK `stores` |
| `slug` | enum ou varchar | Identificador estável (ver § 1.1) |
| `display_name` | varchar | Rótulo na UI do cliente |
| `sort_order` | int | Ordem das colunas/seções no app |
| `active` | boolean | Gerente pode ocultar seção vazia |

`products.category_id` → FK `product_categories`.

### 1.1 Slugs MVP (seed da loja demo)

| `slug` | `display_name` (exemplo UI) |
|--------|-----------------------------|
| `HAMBURGUER_TRADICIONAL` | Hambúrgueres tradicionais |
| `HAMBURGUER_PREMIUM` | Hambúrgueres premium |
| `HAMBURGUER_VEGETARIANO` | Hambúrgueres vegetarianos |
| `HAMBURGUER_VEGANO` | Hambúrgueres veganos |
| `ACOMPANHAMENTO` | Acompanhamentos (ex.: porção de batata para uma pessoa) |
| `ADICIONAL_HAMBURGUER` | Adicionais — hambúrguer |
| `ADICIONAL_MOLHO` | Adicionais — molhos |
| `PORCAO` | Porções |
| `DOCE` | Doces |
| `SALGADO` | Salgados |
| `BEBIDA` | Bebidas |

Query do app cliente: `GET /menu` (INF-02 § 11) — produtos com `availability != ARQUIVADO`, agrupados por `sort_order` da categoria e `products.sort_order`. Itens `PAUSADO_DIA` / `BLOQUEADO` aparecem cinza (DB-02).

---

## 2. Pedido — código visível — `orders`

| Coluna | Tipo | Regra |
|--------|------|-------|
| `display_number` | varchar(16) | **Código curto** exibido no KDS, embalagem e app motoboy (ex.: `#1042`). Gerado no checkout, **único por loja + dia operacional** |
| `subtotal_cents` | int | Soma das linhas (`order_items.line_total_cents`) |
| `delivery_fee_cents` | int | Cópia de `stores.fixed_delivery_fee_cents` no momento do checkout (snapshot) |
| `total_cents` | int | `subtotal_cents + delivery_fee_cents` |
| `delivery_address_snapshot` | jsonb | Cópia completa do endereço no momento do pedido |
| `prep_started_at` | timestamptz, nulo | Início do preparo (`RECEBIDO → EM_PREPARO`) — ORD-01 § 5.4 |
| `prep_completed_at` | timestamptz, nulo | Fim do preparo (`EM_PREPARO → PRONTO`) |
| `prep_duration_seconds` | int, nulo | Duração congelada ao concluir; exibida no painel gerente |

**Operação:** cozinha **escreve `display_number` na embalagem** após concluir preparo. Motoboy compara embalagem ↔ lista no app antes de aceitar.

Índice: `UNIQUE (store_id, operational_date, display_number)` (data operacional = fuso `stores.timezone`, mesma regra DB-03).

---

## 3. Linhas do pedido — `order_items`

| Coluna | Tipo | Regra |
|--------|------|-------|
| `order_id` | UUID | FK |
| `product_id` | UUID | FK (pode ser nulo se produto apagado depois) |
| `product_name` | varchar | Snapshot checkout |
| `unit_price_cents` | int | Snapshot checkout |
| `quantity` | int | ≥ 1 |
| `line_total_cents` | int | `unit_price_cents * quantity` |

**KDS (painel direito):** lista somente leitura — `product_name` + `quantity`. Conclusão é **Concluir preparo** no pedido (`EM_PREPARO → PRONTO`), não por linha.

---

## 4. Endereços salvos — `customer_addresses`

| Coluna | Tipo | Regra |
|--------|------|-------|
| `user_id` | UUID | FK `users` (cliente) |
| `label` | varchar, nulo | Ex.: "Casa", "Trabalho" |
| `is_default` | boolean | Um default por cliente |
| `cep`, `street`, `number`, `district` | varchar | Obrigatórios |
| `complement`, `reference_point` | varchar, nulo | Motoboy / maps |

Checkout: combobox pré-seleciona `is_default = true`. Novo endereço no checkout → `INSERT` após pedido confirmado (mesma transação do checkout).

---

## 5. Formas de pagamento simuladas — `customer_payment_methods`

MVP **mock** — não integra adquirente.

| Coluna | Tipo | Regra |
|--------|------|-------|
| `user_id` | UUID | FK cliente |
| `label` | varchar | Ex.: "Cartão principal" |
| `is_default` | boolean | Autofill no checkout |
| `method_type` | enum | `CREDIT_CARD`, `DEBIT_CARD`, `PIX` (simulado) |
| `last_four` | char(4), nulo | Exibição "•••• 4242" |
| `token_simulated` | varchar | Identificador interno; **não** PAN real |

Checkout: combobox + opção "Usar outro / novo" → formulário manual → persiste se cliente marcar "Salvar".

Pagamento confirmado: `payments.status = CONFIRMADO` via `PaymentSimulationService` (ver FLUX-SAD-06).

---

## 6. Carrinho (cliente)

**Sem tabela no servidor no MVP** — estado local no app (`SharedPreferences` / DataStore) enquanto sessão ativa. Validação de preço, disponibilidade e limite diário só no **checkout** (DB-02 § 1.2).

---

*[← Índice docTecnica](README.md)*
