# DB-04 — Cardápio (categorias), checkout, endereços e pagamento simulado

**Referenciado por:**  
→ [FLUX-HAPPY-02-login](../fluxos/happypaths/FLUX-HAPPY-02-login.md) · [FLUX-HAPPY-03-carrinho](../fluxos/happypaths/FLUX-HAPPY-03-carrinho.md) · [FLUX-HAPPY-08-historico](../fluxos/happypaths/FLUX-HAPPY-08-historico.md) · [FLUX-HAPPY-04-endereco](../fluxos/happypaths/FLUX-HAPPY-04-endereco.md) · [FLUX-HAPPY-05-venda](../fluxos/happypaths/FLUX-HAPPY-05-venda.md) · [FLUX-HAPPY-21-checkout-pagamento](../fluxos/happypaths/FLUX-HAPPY-21-checkout-pagamento.md) · [FLUX-HAPPY-22-cardapio-categorias](../fluxos/happypaths/FLUX-HAPPY-22-cardapio-categorias.md) · [FLUX-HAPPY-11-item-cozinha](../fluxos/happypaths/FLUX-HAPPY-11-item-cozinha.md) · [FLUX-HAPPY-20-motoboy-app](../fluxos/happypaths/FLUX-HAPPY-20-motoboy-app.md)

*[← Índice docTecnica](README.md)*

PostgreSQL. Complementa [DB-02](DB-02-disponibilidade-itens.md) (`products` § 1, § 1.2, loja).

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

Toda a loja é **vegana**; não há categorias separadas por “vegetariano” ou “vegano” — a distinção é por linha de produto (ex.: tradicional vs premium), não por dieta.

| `slug` | `display_name` (exemplo UI) |
|--------|-----------------------------|
| `HAMBURGUER_TRADICIONAL` | Hambúrgueres tradicionais |
| `HAMBURGUER_PREMIUM` | Hambúrgueres premium |
| `ACOMPANHAMENTO` | Acompanhamentos (ex.: porção de batata para uma pessoa) |
| `ADICIONAL_HAMBURGUER` | Adicionais — hambúrguer |
| `ADICIONAL_MOLHO` | Adicionais — molhos |
| `PORCAO` | Porções |
| `DOCE` | Doces |
| `SALGADO` | Salgados |
| `BEBIDA` | Bebidas |

Query do app cliente: `GET /menu` (INF-02 § 11) — produtos com `availability != ARQUIVADO`, agrupados por `sort_order` da categoria e `products.sort_order`. Itens `PAUSADO_DIA` / `BLOQUEADO` aparecem cinza (DB-02).

---

## 2. Pedido — `orders` (UUID + código do dia)

Cada pedido é **uma linha** em `orders`, mapeada **1:1** para a entidade de domínio **`Order`** no backend (Spring JPA) e para o DTO exposto nas APIs (`OrderResponse`, payloads WebSocket). Filhas: **`order_items`** (linhas) e **`order_events`** (timeline — [ORD-01 § 8](ORD-01-estados-pedido.md)).

### 2.1 Dois identificadores

| Identificador | Coluna / JSON | Onde usar |
|---------------|---------------|-----------|
| **UUID do pedido** | `orders.id` · API `id` | Rotas `/orders/{id}`, FKs, STOMP `/topic/order/{orderId}`, locks, imprevisto (**mesmo** pedido reutiliza o UUID) |
| **Código do dia** | `orders.display_number` · API `displayNumber` | UI humana: confirmação pós-checkout, KDS, **recibo na embalagem**, fila do **motoboy** (conferência física). **Não** substitui o UUID nas URLs |

O motoboy **nunca** digita UUID: na fila `PRONTO` vê **`displayNumber`** (ex.: `#42`), confere com o **recibo grampeado no pacote** (e o **`display_number`** nele) e o backend associa a ação ao pedido via **`id`** (UUID retornado na lista / no take). **Nome do cliente** e **`display_number`** conferem pelo recibo (ver § 2.2).

### 2.2 Tabela `orders` — colunas MVP

| Coluna | Tipo | Regra |
|--------|------|--------|
| `id` | UUID | PK, gerado no checkout (`gen_random_uuid()` ou equivalente) |
| `store_id` | UUID | FK `stores` |
| `customer_id` | UUID | FK `users` (cliente) |
| `operational_date` | date | Dia operacional no fuso `stores.timezone` (mesma regra DB-03 § 1.1) |
| `display_number` | varchar(16) | Código **sequencial do dia** por loja (ver § 2.3). **Só dígitos decimais, sem zeros à esquerda** (sem padding): ex. `1`, `9`, `42`, `103`. Na UI: **#** + valor (ex. **#42**) |
| `status` | enum | Máquina ORD-01: `RECEBIDO` … `ENTREGUE`, `CANCELADO` |
| `payment_status` | enum | `CONFIRMADO` no checkout; `ESTORNADO` se cancelado com estorno |
| `priority` | boolean | `true` → topo da fila KDS (⚡ imprevisto) |
| `cancelamento_bloqueado` | boolean | Bloqueia cancelamento **do cliente** (ORD-01 § 2) |
| `incident_alert` | boolean | Aviso na timeline do cliente |
| `incident_message` | text, nulo | Texto do gerente ao cliente |
| `remake_seq` | int | Incrementa a cada imprevisto confirmado |
| `motoboy_id` | UUID, nulo | FK entregador após `take`; `null` em `RECEBIDO`/`PRONTO` sem dono |
| `subtotal_cents` | int | Soma das linhas |
| `delivery_fee_cents` | int | Snapshot de `stores.fixed_delivery_fee_cents` no checkout |
| `total_cents` | int | `subtotal_cents + delivery_fee_cents` |
| `delivery_address_snapshot` | jsonb | Cópia completa do endereço no momento do pedido |
| `last_queue_started_at` | timestamptz | Início da **fila da cozinha** para medir espera até **Preparar**: no checkout = `created_at`; após imprevisto (volta a `RECEBIDO`), atualiza para `now()` ([ORD-01 § 4](ORD-01-estados-pedido.md)) |
| `prep_started_at` | timestamptz, nulo | ORD-01 § 5.3 — início do preparo |
| `prep_completed_at` | timestamptz, nulo | Fim do preparo |
| `prep_duration_seconds` | int, nulo | `prep_completed_at - prep_started_at` (segundos), gravado ao ir a `PRONTO` |
| `dispatch_started_at` | timestamptz, nulo | Motoboy **aceitou** (`POST /orders/{id}/take`) → `SAIU_PARA_ENTREGA` |
| `delivered_at` | timestamptz, nulo | Motoboy confirmou entrega → `ENTREGUE` |
| `allocation_wait_seconds` | int, nulo | Espera na fila até a cozinha clicar **Preparar**: `prep_started_at - last_queue_started_at` (gravado na transição `RECEBIDO → EM_PREPARO`) |
| `balcao_wait_seconds` | int, nulo | Espera **pronto no balcão** até motoboy aceitar: `dispatch_started_at - prep_completed_at` (gravado em `take`) |
| `delivery_duration_seconds` | int, nulo | Rota do motoboy: `delivered_at - dispatch_started_at` (gravado em `deliver`) |
| `total_fulfillment_seconds` | int, nulo | Ciclo fechado: `delivered_at - created_at` (checkout até entregue; gravado em `deliver`) |
| `created_at` | timestamptz | Checkout |
| `updated_at` | timestamptz | Última mutação de estado/campos |

**Índices:**

- `UNIQUE (store_id, operational_date, display_number)` — dois pedidos no mesmo dia **não** compartilham código visível.
- `INDEX (store_id, status, priority DESC, created_at)` — KDS / fila motoboy.
- `INDEX (store_id, operational_date, created_at DESC)` — painel do dia.

**Operação — recibo no pacote:** antes de **Concluir preparo**, a cozinha **imprime** o recibo na **maquininha** da cozinha (impressora térmica acoplada ao fluxo do KDS — não é impressão manual do operador), **grampeia** ao pacote de entrega e só então toca **Concluir preparo**. O recibo contém, no mínimo: **`display_number`**, **nome do cliente**, **valores por item** (linhas do pedido), **total da compra** (alinhado ao snapshot do pedido: subtotal + taxa de entrega) e **data da compra** (dia operacional / data do pedido no fuso da loja). O motoboy confere **`display_number`** e **nome do cliente** entre o recibo, o pacote e a fila no app antes de **Aceitar pedido** (`POST /orders/{id}/take` com UUID do pedido).

### 2.3 Geração de `display_number` (reset diário)

Dentro da **mesma transação** do `POST /orders`:

1. Calcula `operational_date` = data civil atual no fuso da loja.
2. Obtém o próximo sequencial do dia para `(store_id, operational_date)` — ex.: linha em `store_daily_order_counters (store_id, operational_date, last_number)` com `SELECT … FOR UPDATE`, ou `MAX(display_number)` + 1 sob lock.
3. Atribui `display_number` como **string só com dígitos**, **sem padding** (próximo inteiro do dia em decimal: `'1'`, `'2'`, …, `'10'`, …; **nunca** `'001'` ou `'042'`). **Reinicia em `1`** a cada novo `operational_date`.
4. Persiste `orders` com `status = RECEBIDO`, `payment_status = CONFIRMADO`.

Não há prefixo obrigatório no banco; apps formatam **`#` + displayNumber** na UI.

### 2.4 API — checkout e leitura

**Resposta `201` de `POST /orders`** (campos mínimos):

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "displayNumber": "42",
  "status": "RECEBIDO",
  "totalCents": 5890,
  "operationalDate": "2026-11-03"
}
```

**`GET /orders/{id}`** — `{id}` é sempre **UUID**. Inclui `displayNumber`, linhas, endereço snapshot, timeline via eventos, **timestamps e durações** da § 2.2 (para o painel do gerente e export).

**Listagem gerente `GET /orders?storeId=…&date=today`** — cada item inclui `status`, marcas de tempo acima e, para pedidos **ativos**, os **segundos corridos** do **segmento atual** calculados no servidor ou no cliente a partir dos timestamps (ver [INF-06](INF-06-ui-gerente.md) § 2).

**Fila motoboy `GET /delivery/ready`** (ou equivalente): lista `{ id, displayNumber, createdAt, … }` — destaque visual em **`displayNumber`**.

Detalhe de validação checkout: [INF-02 § 11.4](INF-02-http-vs-websocket.md).

### 2.5 Gravação dos tempos (transições)

| Transição | Campos gravados |
|-----------|-----------------|
| Checkout | `created_at`, `last_queue_started_at = created_at`, `status = RECEBIDO` |
| `RECEBIDO → EM_PREPARO` | `prep_started_at`; `allocation_wait_seconds` |
| `EM_PREPARO → PRONTO` | `prep_completed_at`, `prep_duration_seconds` |
| `PRONTO → SAIU_PARA_ENTREGA` (`take`) | `dispatch_started_at`, `motoboy_id`; `balcao_wait_seconds` |
| `SAIU_PARA_ENTREGA → ENTREGUE` (`deliver`) | `delivered_at`; `delivery_duration_seconds`; `total_fulfillment_seconds` |
| Imprevisto → `RECEBIDO` de novo | `last_queue_started_at = now()`; timers do **novo** ciclo de cozinha usam esse instante para nova `allocation_wait_seconds` |

Pedidos `CANCELADO` antes do fim: durações parciais já gravadas permanecem; as não iniciadas ficam nulas.

### 2.6 Persistência dos tempos — coluna a coluna

Referência única para **o que cada campo representa** e **quando é gravado**. O mapa **timer ao vivo × `status`** do painel está em [ORD-01 § 5.7](ORD-01-estados-pedido.md). A tabela de **transições → campos** é a § 2.5 acima.

| Campo / marca | O que mede | Gravado em (transição ou evento) |
|---------------|------------|----------------------------------|
| `created_at` | Momento do checkout (início do ciclo do pedido) | Checkout → `RECEBIDO` |
| `last_queue_started_at` | Início da contagem da **fila até a cozinha** (espera até **Preparar**) | Checkout (= `created_at`) · imprevisto → `RECEBIDO` (`now()`) |
| `prep_started_at` | Cozinha iniciou o preparo | `RECEBIDO → EM_PREPARO` |
| `prep_completed_at` | Cozinha concluiu o preparo | `EM_PREPARO → PRONTO` |
| `prep_duration_seconds` | Duração do preparo (`prep_completed_at - prep_started_at`) | `EM_PREPARO → PRONTO` |
| `allocation_wait_seconds` | Espera na fila até **Preparar** (`prep_started_at - last_queue_started_at`) | `RECEBIDO → EM_PREPARO` |
| `dispatch_started_at` | Motoboy **aceitou** o pedido (início da rota) | `PRONTO → SAIU_PARA_ENTREGA` (`take`) |
| `balcao_wait_seconds` | Espera no balcão até o motoboy aceitar (`dispatch_started_at - prep_completed_at`) | `take` (com a transição acima) |
| `delivered_at` | Entrega confirmada ao cliente | `SAIU_PARA_ENTREGA → ENTREGUE` (`deliver`) |
| `delivery_duration_seconds` | Rota do motoboy (`delivered_at - dispatch_started_at`) | `deliver` |
| `total_fulfillment_seconds` | Ciclo completo (`delivered_at - created_at`) | `deliver` |

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

## 6. Carrinho (cliente — app)

**Sem tabela de linhas de carrinho no servidor no MVP.** Estado no dispositivo (**DataStore** / EncryptedSharedPreferences para tokens; carrinho pode usar DataStore dedicado).

| Campo local | Regra |
|-------------|--------|
| `items[]` | `productId`, `quantity`, snapshot opcional de nome/preço para exibição |
| `cartStartedAt` | Timestamp do **primeiro** item adicionado neste carrinho |
| `shoppingSessionId` | UUID emitido pelo servidor quando o cliente **entrou** com loja `ABERTA` (§ 7) |

| Evento | Carrinho |
|--------|----------|
| Fechar app / matar processo | **Mantém** itens e `cartStartedAt` |
| Reabrir app (JWT ainda válido) | Restaura carrinho; se `now > cartStartedAt + 3 h` → **limpa** carrinho e descarta `shoppingSessionId` |
| **Logout** intencional | **Limpa** carrinho; revoga sessão de compra no servidor (`POST /client/shopping-sessions/{id}/revoke`) |
| Passaram **3 h** desde `cartStartedAt` | App **limpa** carrinho ao abrir ou ao entrar no fluxo de checkout |

Validação de preço e `availability` continua só no **`POST /orders`** (não a cada `+`/`−` no carrinho). **Não** há validação de estoque nem limite diário por produto ([DB-02 § 1.2](DB-02-disponibilidade-itens.md)).

Fluxos: [FLUX-HAPPY-03](../fluxos/happypaths/FLUX-HAPPY-03-carrinho.md) · [FLUX-SAD-04](../fluxos/sadPaths/FLUX-SAD-04-carrinho-sessao.md).

---

## 7. Sessão de compra — `client_shopping_sessions`

Permite checkout **depois** que a loja fechou automaticamente, se o cliente **entrou** enquanto ainda estava `ABERTA`.

| Coluna | Tipo | Regra |
|--------|------|--------|
| `id` | UUID | PK — enviado ao app como `shoppingSessionId` |
| `user_id` | UUID | FK cliente |
| `store_id` | UUID | FK loja |
| `started_at` | timestamptz | Momento da emissão |
| `expires_at` | timestamptz | `started_at + 3 horas` (mesmo TTL do carrinho) |
| `revoked_at` | timestamptz, nulo | Preenchido no **logout** |

**Emissão:** no primeiro **`GET /menu`** (ou `POST /client/shopping-sessions`) com `store_status = ABERTA` após login — uma sessão ativa por cliente/loja; renovar `expires_at` se política futura exigir; no MVP o TTL é fixo a partir de `started_at`.

**Entrada bloqueada:** cliente que abre o app **com loja já fechada** e **sem** sessão não expirada → cardápio cinza, sem nova sessão ([FLUX-SAD-14](../fluxos/sadPaths/FLUX-SAD-14-loja-fechada.md)).

**Checkout `POST /orders`:**

- Cabeçalho ou corpo: `shoppingSessionId`.
- Servidor valida: existe, `user_id` do JWT, `revoked_at` nulo, `now < expires_at`.
- **Não** exige `store_status = ABERTA` nessa requisição.
- Sem sessão válida e loja fechada → `409 STORE_CLOSED`.
- Continua validando itens e pagamento ([FLUX-SAD-03](../fluxos/sadPaths/FLUX-SAD-03-item-checkout.md)).

**Resposta `GET /menu` (loja aberta):** inclui `shoppingSessionId`, `shoppingSessionExpiresAt`, `storeStatus`.

---

## 8. Histórico e refazer pedido

| Método | Rota | Regra |
|--------|------|--------|
| GET | `/orders/me` | Pedidos do cliente autenticado, **`created_at DESC`** (mais recente primeiro). Paginação MVP: `?limit=&cursor=` |
| GET | `/orders/{id}` | Detalhe + `order_items` snapshot; só se `customer_id` = JWT |
| POST | `/orders/{id}/reorder` | Monta payload de carrinho a partir das linhas históricas (`productId`, `quantity`); itens indisponíveis retornam lista `unavailableProductIds` — cliente ajusta antes do checkout |

Alternativa aceita no app: **`GET /orders/{id}`** + popular carrinho local manualmente (mesmas regras de gate § 6–7).

Fluxo: [FLUX-HAPPY-08](../fluxos/happypaths/FLUX-HAPPY-08-historico.md).

---

*[← Índice docTecnica](README.md)*
