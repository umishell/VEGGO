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

## Parte 0 — Dados gerais da loja (configuração inicial)

| # | Quem | O que acontece |
|---|------|----------------|
| 0a | 🔑 Gerente | Acessa **Configurações → Loja** |
| 0b | 🔑 Gerente | Informa ou atualiza o **número de WhatsApp** da loja (campo obrigatório; formato: `5511999999999`) |
| 0c | 🔑 Gerente | Informa ou atualiza o **fuso horário** da loja (ex.: `America/Sao_Paulo`) |
| 0d | Sistema | Salva em `stores.whatsapp_number` e `stores.timezone`. A partir daí o botão Fale Conosco usa esse número |

> ⚠️ Sem `whatsapp_number` preenchido, o botão **Fale Conosco** não aparece no app do cliente (ou exibe aviso de "suporte indisponível"). O número deve ser configurado antes do primeiro pedido.

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
| 11 | Sistema | No horário `ends_at`, chama **`resolveStoreStatus`**: pode voltar a `ABERTA` ou a `FECHADA_HORARIO`, conforme a grade semanal naquele instante |
| 12 | 👤 Cliente | Banner some, cardápio volta ao normal |

> 💡 O painel do gerente usa o **dia operacional** (data civil no fuso da loja). A pausa programada **não** apaga pedidos já listados — antes e depois do intervalo, no mesmo dia, continuam juntos. Ver [FLUX-HAPPY-18-painel](FLUX-HAPPY-18-painel.md) e `DB-03`.

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
| `ABERTA` | Dentro do horário semanal, janela `OPENING`, ou `resolveStoreStatus` após fim de `CLOSURE` quando a grade ainda permite |
| `FECHADA_HORARIO` | Fora da grade semanal, fim de `OPENING`, ou `resolveStoreStatus` após fim de `CLOSURE` quando já passou do horário |
| `FECHADA_PROGRAMADA` | Início de `CLOSURE` ativo em `store_schedules` |
| `FECHADA_EMERGENCIA` | Ação manual do gerente — jobs **não** alteram |

> `FECHADA_EMERGENCIA` **não** é desfeita por jobs automáticos. Só o gerente reabre.

---

**Tabela-resumo dos fechamentos:**

| | Automático de horário | Programado especial | Emergência |
|---|---|---|---|
| Status | `FECHADA_HORARIO` | `FECHADA_PROGRAMADA` | `FECHADA_EMERGENCIA` |
| Motivo exibido | Não | Descrição opcional | Sim, obrigatório |
| Countdown | Sim (próxima abertura) | Sim (`ends_at`) | Se `reopen_at` definido |
| Reabre | Job (`resolveStoreStatus`) | Job (`resolveStoreStatus` no `ends_at`) | Gerente manualmente |

---

> 📌 **Nota técnica resumida:**  
> Tabela `store_hours` para grade semanal. Tabela `store_schedules` para eventos especiais. Jobs chamam **`resolveStoreStatus`** (prioridade: emergência intacta → `CLOSURE` ativo → `OPENING` → horário semanal). `next_opens_at` é calculado no servidor e enviado ao cliente — o frontend só exibe o timer.  
> → Detalhes: [`../../docTecnica/DB-02-disponibilidade-itens.md`](../../docTecnica/DB-02-disponibilidade-itens.md)
