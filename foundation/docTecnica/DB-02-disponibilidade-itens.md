# DB-02 — Disponibilidade de Itens e Status da Loja

**Referenciado por:**  
→ [FLUX-HAPPY-10-item-gerente](../fluxos/happypaths/FLUX-HAPPY-10-item-gerente.md) · [FLUX-HAPPY-11-item-cozinha](../fluxos/happypaths/FLUX-HAPPY-11-item-cozinha.md) · [FLUX-HAPPY-12-horario](../fluxos/happypaths/FLUX-HAPPY-12-horario.md) · [FLUX-HAPPY-14-frete](../fluxos/happypaths/FLUX-HAPPY-14-frete.md) · [FLUX-SAD-01-item-cinza](../fluxos/sadPaths/FLUX-SAD-01-item-cinza.md) · [FLUX-SAD-13-loja-emergencia](../fluxos/sadPaths/FLUX-SAD-13-loja-emergencia.md)

*[← Índice docTecnica](README.md)*

PostgreSQL. Sem coleção paralela para a cozinha ou para o gerente.

---

## 1. Itens — tabela `products`

| Coluna | Tipo | Regra |
|--------|------|-------|
| `availability` | enum | `DISPONIVEL`, `PAUSADO_DIA`, `BLOQUEADO` |
| demais colunas do produto | — | nome, descrição, preço, categoria, foto |

Não há flags `available` e `out_today`.

| Estado | Quem grava | Meia-noite | Lista da cozinha e do gerente | Cardápio do cliente |
|--------|------------|------------|--------------------------------|---------------------|
| `DISPONIVEL` | Reativação | Permanece | Visível, pode vender | Normal |
| `PAUSADO_DIA` | Cozinha ou gerente | Job volta para `DISPONIVEL` | Visível | Cinza. "Não está sendo produzido hoje." Não entra no carrinho |
| `BLOQUEADO` | Cozinha ou gerente | Não mexe | Visível | Cinza. "Indisponível no momento." Não entra no carrinho |

Cozinha e gerente reativam os dois estados. Só o gerente executa `DELETE` do produto. Aí a linha some do banco e das três telas.

`order_items.product_name` e `order_items.unit_price` são cópia no checkout. O histórico sobrevive ao `DELETE`.

Job:

```sql
UPDATE products
   SET availability = 'DISPONIVEL'
 WHERE availability = 'PAUSADO_DIA';
```

`BLOQUEADO` não entra nesse `UPDATE`.

---

## 2. Status da loja — tabela `stores`

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `store_status` | enum | `ABERTA`, `FECHADA_HORARIO`, `FECHADA_PROGRAMADA`, `FECHADA_EMERGENCIA` |
| `closed_reason` | string, nulo | Obrigatório em `FECHADA_EMERGENCIA` |
| `manager_message` | string, nulo | Mensagem adicional opcional do gerente visível ao cliente |
| `reopen_at` | timestamp, nulo | Previsão de reabertura em `FECHADA_EMERGENCIA`. Nulo = sem previsão |
| `closed_at` | timestamp, nulo | Auditoria do fechamento manual |
| `whatsapp_number` | string | Usado só no link `wa.me` |
| `accept_alert_minutes` | int | Alerta de pedido parado em `RECEBIDO`. Não muda o estado |
| `allocation_timeout_minutes` | int | `ALOCADO` volta a `PRONTO` |
| `next_opens_at` | timestamp, nulo | Calculado pelo servidor; enviado ao cliente para o countdown. Nunca gravado pelo frontend |

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

`store_schedules` tem prioridade sobre `store_hours`. Um `CLOSURE` com `starts_at` no futuro fecha a loja naquele instante e a reabre no `ends_at` sem resetar o histórico de pedidos do dia.

---

**Jobs:**

| Job | Efeito |
|-----|--------|
| Abertura semanal | `store_status = ABERTA` se `store_hours.is_open_day` e não há `CLOSURE` ativo |
| Fechamento semanal | `store_status = FECHADA_HORARIO` |
| Início de `CLOSURE` | `store_status = FECHADA_PROGRAMADA` |
| Fim de `CLOSURE` | `store_status = ABERTA` (retorna ao turno sem resetar histórico) |
| Início de `OPENING` | `store_status = ABERTA` fora do horário padrão |
| Fim de `OPENING` | `store_status = FECHADA_HORARIO` |
| Meia-noite | `PAUSADO_DIA` → `DISPONIVEL` em `products` |

`FECHADA_EMERGENCIA` não é desfeita por nenhum job. Só o gerente reabre manualmente.

Checkout e adição ao carrinho exigem `store_status = ABERTA`.

---

## 3. Zonas — tabela `delivery_zones`

| Campo | Tipo |
|-------|------|
| `id` | UUID |
| `store_id` | UUID |
| `name` | string |
| `cep_start` | string |
| `cep_end` | string |
| `fee` | numeric |

O checkout grava a taxa encontrada na linha do pedido. CEP fora de todas as faixas não cria pedido.

---

*[← Índice docTecnica](README.md)*
