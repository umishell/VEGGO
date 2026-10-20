# FLUX-HAPPY-12-horario
### Gerente configura horários e fechamentos programados

**Perfis:** 🔑

[← Índice happy path](README.md)

---

**Perfis:** 🔑 Gerente  
**Veja também:** [FLUX-SAD-13-loja-emergencia](../sadPaths/FLUX-SAD-13-loja-emergencia.md) — fechamento de emergência · [FLUX-SAD-14-loja-fechada](../sadPaths/FLUX-SAD-14-loja-fechada.md) — o que o cliente vê quando a loja está fechada

```
Gerente configura horários semanais e/ou intervalos especiais
→ sistema abre e fecha automaticamente → cliente vê countdown em tempo real
```

---

## Parte 1 — Horário semanal padrão

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Acessa **Configurações → Horário de funcionamento** |
| 2 | 🔑 Gerente | Define abertura e fechamento para cada dia da semana (ex: Seg–Sex 18h–23h) |
| 3 | 🔑 Gerente | Marca os dias fechados (ex: domingo) |
| 4 | Sistema | Jobs automáticos abrem (`ABERTA`) e fecham (`FECHADA_HORARIO`) nos horários definidos |
| 5 | 👤 Cliente | Vê countdown até a próxima abertura quando a loja está fechada |

---

## Parte 2 — Fechamentos e aberturas especiais programados

Para eventos pontuais — intervalo de uma hora, dia feriado, folga do dia — sem alterar a grade semanal.

| # | Quem | O que acontece |
|---|------|----------------|
| 6 | 🔑 Gerente | Acessa **Configurações → Eventos especiais** |
| 7 | 🔑 Gerente | Cria um evento: data, horário de início (`starts_at`), horário de fim (`ends_at`), tipo e descrição opcional |
| 8 | Sistema | Salva o evento em `store_schedules` |
| 9 | Sistema | No horário `starts_at`, muda para `FECHADA_PROGRAMADA` |
| 10 | 👤 Cliente | Vê banner: **"Loja em pausa temporária"** + countdown até `ends_at` |
| 11 | Sistema | No horário `ends_at`, volta a `ABERTA` automaticamente |
| 12 | 👤 Cliente | Banner some, cardápio volta ao normal |

> 💡 O histórico de pedidos do dia **não é resetado** na reabertura. Pedidos feitos antes do intervalo continuam visíveis no painel do gerente.

---

**Tipos de evento programado:**

| Tipo | Descrição |
|------|-----------|
| `CLOSURE` | Fecha a loja em horário específico e reabre automaticamente |
| `OPENING` | Abre a loja fora do horário semanal (ex: evento especial numa segunda normalmente fechada) |

---

## Parte 3 — Comportamento dos jobs

| Status resultante | Fonte |
|-----------------|-------|
| `ABERTA` | Job de abertura semanal ou fim de `CLOSURE` em `store_schedules` |
| `FECHADA_HORARIO` | Job de fechamento semanal |
| `FECHADA_PROGRAMADA` | Início de `CLOSURE` em `store_schedules` |
| `FECHADA_EMERGENCIA` | Ação manual do gerente |

> `FECHADA_EMERGENCIA` **não** é desfeita por jobs automáticos. Só o gerente reabre.

---

**Tabela-resumo dos fechamentos:**

| | Automático de horário | Programado especial | Emergência |
|---|---|---|---|
| Status | `FECHADA_HORARIO` | `FECHADA_PROGRAMADA` | `FECHADA_EMERGENCIA` |
| Motivo exibido | Não | Descrição opcional | Sim, obrigatório |
| Countdown | Sim (próxima abertura) | Sim (`ends_at`) | Se `reopen_at` definido |
| Reabre | Job automático | Job automático | Gerente manualmente |

---

> 📌 **Nota técnica resumida:**  
> Tabela `store_hours` para grade semanal. Tabela `store_schedules` para eventos especiais. Jobs verificam os dois, com `store_schedules` tendo prioridade. `next_opens_at` é calculado no servidor e enviado ao cliente — o frontend só exibe o timer.  
> → Detalhes: [`../../docTecnica/DB-02-disponibilidade-itens.md`](../../docTecnica/DB-02-disponibilidade-itens.md)
