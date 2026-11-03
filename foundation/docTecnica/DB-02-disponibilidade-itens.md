# DB-02 — Disponibilidade de Itens e Status da Loja

**Referenciado por:**  
→ [FLUX-HAPPY-10-item-gerente](../fluxos/happypaths/FLUX-HAPPY-10-item-gerente.md) · [FLUX-HAPPY-11-item-cozinha](../fluxos/happypaths/FLUX-HAPPY-11-item-cozinha.md) (KDS) · [FLUX-HAPPY-12-horario](../fluxos/happypaths/FLUX-HAPPY-12-horario.md) · [FLUX-SAD-01-item-cinza](../fluxos/sadPaths/FLUX-SAD-01-item-cinza.md) · [FLUX-SAD-13-loja-emergencia](../fluxos/sadPaths/FLUX-SAD-13-loja-emergencia.md) · [INF-02 § 11](INF-02-http-vs-websocket.md)

*[← Índice docTecnica](README.md)*

PostgreSQL. Sem coleção paralela para a cozinha ou para o gerente.

---

## 1. Itens — tabela `products`

Entidade **única** do cardápio: gerente faz CRUD; app cliente lista; checkout copia snapshot para `order_items` ([DB-04](DB-04-catalogo-checkout-cliente.md) § 3). HTTP: [INF-02](INF-02-http-vs-websocket.md) § 11.

| Coluna | Tipo | Regra |
|--------|------|-------|
| `id` | UUID | PK |
| `store_id` | UUID | FK `stores` — todo produto pertence a uma loja |
| `category_id` | UUID | FK `product_categories` (mesma loja) — [DB-04](DB-04-catalogo-checkout-cliente.md) § 1 |
| `name` | varchar(120) | Nome no cardápio e snapshot `order_items.product_name` |
| `description` | text, nulo | Texto longo; exibido no detalhe do produto (cliente) e no formulário gerente |
| `unit_price_cents` | int | Preço unitário em centavos (≥ 0) |
| `image_url` | varchar(2048), nulo | URL pública da foto (upload gera URL — INF-02 § 11) |
| `availability` | enum | `DISPONIVEL`, `PAUSADO_DIA`, `BLOQUEADO`, `ARQUIVADO` |
| `default_daily_order_limit` | int, nulo | Padrão de unidades vendidas por dia operacional; ver § 1.2 |
| `sort_order` | int, default 0 | Ordem **dentro da categoria** no app cliente (empate: `name`) |
| `created_at` | timestamptz | Auditoria |
| `updated_at` | timestamptz | Auditoria |

Índices sugeridos: `(store_id, category_id, availability)`, `(store_id, sort_order)`.

Não há flags `available` e `out_today`.

| Estado | Quem grava | Meia-noite | Cozinha vê? | Gerente vê? | Cardápio do cliente |
|--------|------------|------------|-------------|-------------|---------------------|
| `DISPONIVEL` | Reativação | Permanece | ✅ Visível | ✅ Visível | Normal, pode pedir |
| `PAUSADO_DIA` | **Gerente** | Job volta para `DISPONIVEL` | ❌ (KDS não gerencia cardápio) | ✅ Visível | Cinza. "Não está sendo produzido hoje." Não entra no carrinho |
| `BLOQUEADO` | **Gerente** | Não mexe | ❌ | ✅ Visível | Cinza. "Indisponível no momento." Não entra no carrinho |
| `ARQUIVADO` | **Só o gerente** | Não mexe | ❌ Não aparece | ✅ Seção separada "Itens arquivados" | ❌ Não aparece |

**`ARQUIVADO`** é o estado de ocultação administrativa. O item permanece cadastrado no banco, não aparece no cardápio do cliente nem na lista da cozinha. O gerente acessa e gerencia os itens arquivados em seção dedicada e pode desarquivá-los (volta a `DISPONIVEL`) ou excluí-los definitivamente.

Só o gerente altera `PAUSADO_DIA`, `BLOQUEADO`, arquiva, desarquiva e executa `DELETE` permanente do produto. Quando arquivado, a linha sai das listas operacionais mas **não** é apagada do banco.

`order_items.product_name` e `order_items.unit_price` são cópia no checkout. O histórico sobrevive ao `DELETE`.

**Imprevisto em rota:** o sistema **não** classifica linha a linha (descarte vs devolução). Motoboy e gerente resolvem **presencialmente / WhatsApp** o que descartar ou trazer de volta; o app só recoloca o **mesmo pedido** em `RECEBIDO` com prioridade ([ORD-01](ORD-01-estados-pedido.md) § 4).

---

## 1.2 Limite diário de pedidos por produto (não é estoque)

**Não há módulo de estoque** (sem saldo físico, lote ou entrada/saída de mercadoria). Há **controle de quantas unidades de cada produto podem ser vendidas no dia operacional** — ferramenta de planejamento para a cozinha estimar insumos; o sistema só conta **pedidos confirmados**.

### Tabelas

**`products` — padrão que renova todo dia:**

| Coluna | Tipo | Regra |
|--------|------|-------|
| `default_daily_order_limit` | int, nulo | Máximo de **unidades** (soma de `quantity` em pedidos pagos) permitidas **por dia operacional**. `NULL` = sem teto (ilimitado no dia) |

**`product_daily_caps` — cap efetivo do dia (pode divergir do padrão):**

| Coluna | Tipo | Regra |
|--------|------|-------|
| `store_id` | UUID | FK `stores` |
| `product_id` | UUID | FK `products` |
| `operational_date` | date | Data civil no fuso `stores.timezone` (mesma regra do painel — DB-03 § 1.1) |
| `effective_limit` | int, nulo | Teto do **dia atual**; gerente altera a qualquer momento. Job da meia-noite copia de `default_daily_order_limit` |
| `units_sold` | int, default 0 | Unidades já comprometidas em pedidos com pagamento confirmado neste dia (incrementa no checkout atômico) |

**PK / unicidade:** `UNIQUE (store_id, product_id, operational_date)`.

**Restante do dia:** `remaining = effective_limit - units_sold` quando `effective_limit` não é nulo; senão ilimitado.

### Comportamento

| Ação | Quem | Efeito |
|------|------|--------|
| Definir padrão diário | 🔑 Gerente | Edita `default_daily_order_limit` no CRUD do cardápio |
| Ajustar só **hoje** | 🔑 Gerente | `PATCH /store/products/{id}/daily-cap` → `effective_limit` (aumenta se sobrou insumo; reduz se faltou ou estragou) — **libera ou restringe pedidos na hora**, sem mudar o padrão de amanhã |
| Meia-noite (fuso loja) | Job | Nova linha ou reset: `effective_limit := default_daily_order_limit`, `units_sold := 0` por produto. Independente do job que desfaz `PAUSADO_DIA` |
| Checkout | Cliente | Exige `availability = DISPONIVEL`, loja `ABERTA` e, se houver teto, `quantity <= remaining`. Estoura → HTTP 409 (`ITEM_UNAVAILABLE` ou `DAILY_LIMIT_EXCEEDED`). Fluxos: [FLUX-SAD-03](../fluxos/sadPaths/FLUX-SAD-03-item-checkout.md) · [FLUX-SAD-24](../fluxos/sadPaths/FLUX-SAD-24-limite-dia.md) |
| Ingrediente acabou / não dá para produzir | 👨‍🍳 avisa · 🔑 age | Cozinha **não** pausa cardápio: o gerente coloca **`PAUSADO_DIA`** (“não produzido hoje”, cinza). Ver [FLUX-HAPPY-11-item-cozinha](../fluxos/happypaths/FLUX-HAPPY-11-item-cozinha.md) (KDS) |

Painel **Cardápio** do gerente exibe **padrão**, **vendido hoje**, **restante** e **limite efetivo de hoje**, com **filtro por categoria** (INF-04 § 4). Cozinha no KDS vê apenas pedidos (não CRUD de disponibilidade).

Job:

```sql
UPDATE products
   SET availability = 'DISPONIVEL'
 WHERE availability = 'PAUSADO_DIA';
-- BLOQUEADO e ARQUIVADO não entram nesse UPDATE
```

`BLOQUEADO` e `ARQUIVADO` não entram nesse `UPDATE`.

---

## 2. Status da loja — tabela `stores`

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `store_status` | enum | `ABERTA`, `FECHADA_HORARIO`, `FECHADA_PROGRAMADA`, `FECHADA_EMERGENCIA` |
| `closed_reason` | string, nulo | Obrigatório em `FECHADA_EMERGENCIA` |
| `manager_message` | string, nulo | Mensagem adicional opcional do gerente visível ao cliente |
| `reopen_at` | timestamp, nulo | Previsão de reabertura em `FECHADA_EMERGENCIA`. Nulo = sem previsão |
| `closed_at` | timestamp, nulo | Momento do fechamento manual (campo operacional) |
| `whatsapp_number` | string | Usado só no link `wa.me` |
| `next_opens_at` | timestamp, nulo | Calculado pelo servidor; enviado ao cliente para o countdown. Nunca gravado pelo frontend |
| `timezone` | string | Fuso IANA da loja (ex.: `America/Sao_Paulo`). Define o **dia operacional** do painel e das métricas “hoje” |
| `fixed_delivery_fee_cents` | int | **Taxa de entrega fixa** da loja (≥ 0). Gerente edita em **Configurações → Loja** ([FLUX-HAPPY-12](../fluxos/happypaths/FLUX-HAPPY-12-horario.md)). Checkout copia o valor para `orders.delivery_fee_cents` (snapshot). **Não** há zonas, faixas de CEP nem frete variável no MVP |

**O countdown é calculado pelo servidor** a partir de `store_hours` ou `store_schedules` e enviado ao cliente. O frontend recebe um timestamp absoluto e exibe a diferença.

---

**Tabela `store_hours` — grade semanal:**

| Campo | Tipo |
|-------|------|
| `store_id` | UUID |
| `day_of_week` | `MON` … `SUN` |
| `opens_at` | time |
| `closes_at` | time |
| `is_open_day` | boolean |

---

**Tabela `store_schedules` — eventos especiais programados:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `id` | UUID | |
| `store_id` | UUID | |
| `type` | enum | `CLOSURE` (fecha) ou `OPENING` (abre fora do padrão) |
| `starts_at` | timestamp | Início do evento |
| `ends_at` | timestamp | Fim do evento (reabertura automática) |
| `description` | string, nulo | Descrição opcional visível ao cliente |
| `applied` | boolean | Impede o job de reprocessar evento já executado |

Um `CLOSURE` fecha na `starts_at` e, no `ends_at`, o sistema **recalcula** o status — não assume `ABERTA` cegamente. Pausa programada **não** zera pedidos do painel do gerente (mesmo dia operacional; ver `DB-03`).

---

## 2.1 Resolução efetiva de `store_status`

Nenhum job grava `store_status` com valor fixo, exceto transições pontuais abaixo. Depois de cada gatilho, o backend chama **`resolveStoreStatus(store_id, at)`**, que devolve o status correto para o instante `at`.

**Regra 0 — emergência:** se `store_status = FECHADA_EMERGENCIA`, **`resolveStoreStatus` não roda** (jobs de horário e de `store_schedules` saem sem alterar a loja). Só a ação manual **Reabrir loja** do gerente chama `resolveStoreStatus` em seguida.

**Prioridade quando não está em emergência** (do mais restritivo ao menos):

| Ordem | Condição em `at` | Status resultante |
|-------|------------------|-------------------|
| 1 | Existe `CLOSURE` com `starts_at ≤ at < ends_at` | `FECHADA_PROGRAMADA` |
| 2 | Existe `OPENING` com `starts_at ≤ at < ends_at` | `ABERTA` |
| 3 | Dia/fechamento em `store_hours`: fora da janela ou `is_open_day = false` | `FECHADA_HORARIO` |
| 4 | Dentro da janela semanal e sem `CLOSURE` ativo | `ABERTA` |

**Sobreposição de eventos programados (MVP):**

- **`CLOSURE` + `OPENING` no mesmo intervalo:** vale o `CLOSURE` (loja fechada programada).
- **Dois `CLOSURE` sobrepostos:** permanece `FECHADA_PROGRAMADA` até não restar nenhum `CLOSURE` ativo; o countdown usa o `ends_at` do evento que termina **por último** entre os ativos.
- **Dois `OPENING` sobrepostos:** permanece `ABERTA` enquanto qualquer `OPENING` estiver ativo.
- **Emergência durante pausa programada:** o gerente grava `FECHADA_EMERGENCIA` por cima; jobs ignoram até reabertura manual; ao reabrir, `resolveStoreStatus` pode voltar a `FECHADA_PROGRAMADA` se o `CLOSURE` ainda estiver vigente.

`next_opens_at` é derivado do próximo instante em que `resolveStoreStatus` retornaria `ABERTA` (próximo fim de `CLOSURE`, próximo início de janela semanal ou de `OPENING`, conforme o caso).

---

**Jobs (gatilhos → efeito):**

| Job | Gatilho | Efeito |
|-----|---------|--------|
| Tick horário / abertura semanal | `opens_at` do dia em `store_hours` | Se **não** `FECHADA_EMERGENCIA`: `store_status := resolveStoreStatus(store_id, now())` |
| Tick horário / fechamento semanal | `closes_at` do dia em `store_hours` | Idem |
| Início de `CLOSURE` | `starts_at`, `applied = false` | Marca evento aplicado; se **não** emergência: `FECHADA_PROGRAMADA` (equivalente a resolver com CLOSURE ativo) |
| Fim de `CLOSURE` | `ends_at` | Se **não** emergência: `store_status := resolveStoreStatus(...)` — pode ser `ABERTA` **ou** `FECHADA_HORARIO` |
| Início / fim de `OPENING` | `starts_at` / `ends_at` | Se **não** emergência: `resolveStoreStatus(...)` |
| Meia-noite (`timezone` da loja) | 00:00 local | `PAUSADO_DIA` → `DISPONIVEL` em `products`. **Não** altera `store_status` por si só; opcionalmente roda `resolveStoreStatus` se o relógio cruzou abertura/fechamento |

Checkout e adição ao carrinho exigem `store_status = ABERTA` **no momento da requisição** (após `resolveStoreStatus` se o job acabou de rodar).

**Fora do MVP:** tabela `delivery_zones`, frete por CEP/bairro e bloqueio por área de entrega. Ver taxa fixa em `stores.fixed_delivery_fee_cents` acima.

---

*[← Índice docTecnica](README.md)*
