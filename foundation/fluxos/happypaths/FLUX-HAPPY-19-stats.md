# FLUX-HAPPY-19-stats
### Gerente acessa estatísticas e telemetria da operação

**Perfis:** 🔑

[← Índice happy path](README.md)

---

**Perfis:** 🔑 Gerente

```
Gerente abre Estatísticas → escolhe o período e a visão
→ vê volumes, tendências, itens mais pedidos e comportamento dos clientes
```

> Esta área é somente leitura — não modifica pedidos, itens ou usuários.

---

## Telas disponíveis

### Tela 1 — Resumo do dia

Acessada também pelo painel principal; complementa o [FLUX-HAPPY-18-painel](FLUX-HAPPY-18-painel.md).

| Indicador | Descrição | MVP? |
|-----------|-----------|------|
| Total de pedidos do dia | Contagem de todos os pedidos (inclusive cancelados, com indicador separado) | ✅ |
| Faturamento do dia | Soma dos pedidos `ENTREGUE` (não conta cancelados) | ✅ |
| Ticket médio | Valor médio por pedido entregue | ✅ |
| Pedidos em andamento agora | Contagem em tempo real | ✅ |
| Tempo médio da cozinha | `EM_PREPARO` → `PRONTO` | ✅ |
| Tempo médio de entrega | `SAIU_PARA_ENTREGA` → `ENTREGUE` | ✅ |

---

### Tela 2 — Histórico por período

| Filtro | Opções |
|--------|--------|
| Período | Dia, semana, mês, intervalo personalizado |
| Agrupamento | Por dia / por semana / por mês |

| Métrica | Descrição |
|---------|-----------|
| Volume de pedidos | Gráfico de barras com pedidos por período |
| Faturamento | Receita por período |
| Taxa de cancelamento | % de pedidos cancelados no período |
| Novos clientes | Clientes que fizeram o primeiro pedido no período |
| Clientes recorrentes | Clientes com mais de 1 pedido no período |

---

### Tela 3 — Itens do cardápio

| Métrica | Descrição | MVP? |
|---------|-----------|------|
| Itens mais pedidos | Ranking por quantidade vendida no período | ✅ |
| Itens que mais faturam | Ranking por receita gerada | ✅ |
| Itens com maior taxa de cancelamento | Itens que aparecem em pedidos cancelados | ✅ |
| Horário de pico por item | Em qual faixa de horário cada item é mais pedido | 🔜 pós-MVP |

---

### Tela 4 — Clientes

| Métrica | Descrição | MVP? |
|---------|-----------|------|
| Total de clientes cadastrados | Contagem | ✅ |
| Clientes ativos no período | Com pelo menos 1 pedido no período | ✅ |
| Novos clientes no período | Primeira compra dentro do intervalo | ✅ |
| Clientes recorrentes | Mais de 1 pedido no período | ✅ |
| Clientes que mais pedem | Ranking por volume e por valor | ✅ |
| Frequência de pedido por cliente | Distribuição (histograma) | 🔜 pós-MVP |
| Retenção mês-a-mês | % que voltou no mês seguinte | 🔜 pós-MVP |

---

### Tela 5 — Operação (motoboys e cozinha)

| Métrica | Descrição | MVP? |
|---------|-----------|------|
| Pedidos por motoboy | Entregas feitas por cada motoboy no período | ✅ |
| Imprevistos em rota por motoboy | Contagem e % por entregador | ✅ |
| Tempo médio de entrega por motoboy | Da retirada (`SAIU_PARA_ENTREGA`) à confirmação (`ENTREGUE`) | ✅ |
| Tempo médio da cozinha | `EM_PREPARO` → `PRONTO` | ✅ |
| Tempo médio aguardando motoboy | `PRONTO` → `SAIU_PARA_ENTREGA` | ✅ |
| Tempo médio por turno | Cozinha separada por turno do dia | 🔜 pós-MVP |

---

> 💡 Os dados das Telas 2–5 são históricos e **não atualizam em tempo real** — são gerados sob demanda com filtro de período.  
> 💡 A Tela 1 (resumo do dia) atualiza automaticamente, igual ao painel de pedidos.  
> 💡 Métricas marcadas como **🔜 pós-MVP** não têm query em DB-03 e não serão implementadas no MVP — a tela pode exibir essas linhas como "em breve".

---

> 📌 **Nota técnica resumida:**  
> As métricas das Telas 2–5 são geradas por queries de agregação em PostgreSQL, com índices nos campos de data e status. Todos os agrupamentos por dia usam `(created_at AT TIME ZONE stores.timezone)::date` para respeitar o fuso da loja. Não há tabela de pré-agregação no MVP — as queries rodam sob demanda. Para períodos longos, o tempo de resposta pode aumentar; paginação e cache de resultado podem ser adicionados fora do MVP.  
> → SQL completo de todas as métricas ✅: [`../../docTecnica/DB-03-telemetria.md`](../../docTecnica/DB-03-telemetria.md)
