# DB-02 — Disponibilidade de Itens e Status da Loja

**Referenciado por:**  
→ [FLUX-HAPPY-10-item-gerente](../fluxos/happypaths/FLUX-HAPPY-10-item-gerente.md) · [FLUX-HAPPY-11-item-cozinha](../fluxos/happypaths/FLUX-HAPPY-11-item-cozinha.md) · [FLUX-HAPPY-12-horario](../fluxos/happypaths/FLUX-HAPPY-12-horario.md) · [FLUX-HAPPY-14-frete](../fluxos/happypaths/FLUX-HAPPY-14-frete.md) · [FLUX-SAD-01-item-cinza](../fluxos/sadPaths/FLUX-SAD-01-item-cinza.md) · [FLUX-SAD-13-loja-emergencia](../fluxos/sadPaths/FLUX-SAD-13-loja-emergencia.md)

*[← Índice docTecnica](README.md)*

PostgreSQL. Sem coleção paralela para a cozinha ou para o gerente.

---

## 1. Itens — tabela `products`

| Coluna | Tipo | Regra |
|--------|------|-------|
| `availability` | enum | `DISPONIVEL`, `PAUSADO_DIA`, `BLOQUEADO`, `ARQUIVADO` |
| `item_type` | enum | `PREPARED_FOOD`, `SEALED_BEVERAGE` — classifica o destino do item em caso de imprevisto em rota |
| demais colunas do produto | — | nome, descrição, preço, categoria, foto |

Não há flags `available` e `out_today`.

| Estado | Quem grava | Meia-noite | Cozinha vê? | Gerente vê? | Cardápio do cliente |
|--------|------------|------------|-------------|-------------|---------------------|
| `DISPONIVEL` | Reativação | Permanece | ✅ Visível | ✅ Visível | Normal, pode pedir |
| `PAUSADO_DIA` | Cozinha ou gerente | Job volta para `DISPONIVEL` | ✅ Visível | ✅ Visível | Cinza. "Não está sendo produzido hoje." Não entra no carrinho |
| `BLOQUEADO` | Cozinha ou gerente | Não mexe | ✅ Visível | ✅ Visível | Cinza. "Indisponível no momento." Não entra no carrinho |
| `ARQUIVADO` | **Só o gerente** | Não mexe | ❌ Não aparece | ✅ Seção separada "Itens arquivados" | ❌ Não aparece |

**`ARQUIVADO`** é o estado de ocultação administrativa. O item permanece cadastrado no banco, não aparece no cardápio do cliente nem na lista da cozinha. O gerente acessa e gerencia os itens arquivados em seção dedicada e pode desarquivá-los (volta a `DISPONIVEL`) ou excluí-los definitivamente.

Cozinha reativa `PAUSADO_DIA` e `BLOQUEADO`. Só o gerente arquiva, desarquiva e executa `DELETE` permanente do produto. Quando arquivado, a linha sai das listas operacionais mas **não** é apagada do banco.

`order_items.product_name` e `order_items.unit_price` são cópia no checkout. O histórico sobrevive ao `DELETE`.

**`item_type`** é usado pelo protocolo de imprevisto em rota para classificar automaticamente os itens do pedido:
- `PREPARED_FOOD` → descartado (lanche, batata, sobremesa etc.)
- `SEALED_BEVERAGE` → pendência de devolução (lata, garrafa)

O gerente define `item_type` ao criar ou editar o produto.

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
| `closed_at` | timestamp, nulo | Auditoria do fechamento manual |
| `whatsapp_number` | string | Usado só no link `wa.me` |
| `accept_alert_minutes` | int | Alerta de pedido parado em `RECEBIDO`. Não muda o estado |
| `next_opens_at` | timestamp, nulo | Calculado pelo servidor; enviado ao cliente para o countdown. Nunca gravado pelo frontend |
| `timezone` | string | Fuso IANA da loja (ex.: `America/Sao_Paulo`). Define o **dia operacional** do painel e das métricas “hoje” |

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
