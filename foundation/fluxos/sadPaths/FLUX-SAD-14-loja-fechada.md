# FLUX-SAD-14-loja-fechada
### Loja fechada — experiência do cliente

**Gravidade:** 🟠 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente  
**Causa:** A loja está `FECHADA_HORARIO`, `FECHADA_PROGRAMADA` ou `FECHADA_EMERGENCIA`.  
**Veja também:** [FLUX-HAPPY-12-horario](../happypaths/FLUX-HAPPY-12-horario.md) — horário padrão e fechamentos programados · [FLUX-SAD-13-loja-emergencia](FLUX-SAD-13-loja-emergencia.md) — fechamento de emergência

```
Cliente abre o app → loja está fechada → tudo cinza + banner de aviso
→ não consegue adicionar ao carrinho → countdown mostra quando abre
```

---

## O que o cliente vê ao abrir o app

| Elemento | Comportamento |
|----------|--------------|
| **Banner no topo** | Faixa de aviso visível em todas as telas do app |
| **Cardápio** | Exibido todo em tom cinza — itens não clicáveis |
| **Botão "Adicionar"** | Desabilitado. O cliente não forma nem inicia o carrinho |
| **Carrinho ativo (visitante novo)** | Quem **não** tinha sessão de compra: cardápio cinza — não forma carrinho |
| **Compra em andamento** | Quem já tinha `shoppingSessionId` + carrinho dentro de **3 h** pode ir ao checkout **mesmo** com loja fechada (banner informativo opcional) |
| **Countdown** | Temporizador visual em tempo real (quando há horário de reabertura definido) |

---

## Cenário A — Fechamento de horário automático (`FECHADA_HORARIO`)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | Sistema | Fecha no horário configurado da semana |
| 2 | 👤 Cliente | Banner: **"Loja fechada no momento"** |
| 3 | Sistema | Exibe countdown: **"Abrimos em [HH:MM:SS]"** — conta regressiva até `opens_at` do próximo turno |
| 4 | 👤 Cliente | Vê o cardápio inteiro cinza e não consegue adicionar itens |
| 5 | Sistema | Ao zerar o countdown, status vira `ABERTA` automaticamente · banner some · cardápio volta ao normal |

---

## Cenário B — Fechamento programado (`FECHADA_PROGRAMADA`)

Gerente configurou um fechamento especial com horário de reabertura definido.

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | Sistema | Fecha no início do intervalo programado (`store_schedules`) |
| 2 | 👤 Cliente | Banner: **"Loja em pausa temporária"** |
| 3 | Sistema | Countdown: **"Voltamos em [HH:MM:SS]"** — conta até `ends_at` do evento |
| 4 | 👤 Cliente | Cardápio cinza, sem adição ao carrinho |
| 5 | Sistema | Ao zerar o countdown, reabre automaticamente · banner some · pedidos do dia permanecem no histórico |

---

## Cenário C — Fechamento de emergência com horário de retorno (`FECHADA_EMERGENCIA` + `reopen_at`)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Fechou em emergência e programou um horário de retorno |
| 2 | 👤 Cliente | Banner: **"Loja temporariamente fechada"** + motivo obrigatório do gerente |
| 3 | Sistema | Countdown: **"Previsto para reabrir em [HH:MM:SS]"** |
| 4 | 👤 Cliente | Cardápio cinza, sem adição ao carrinho |

---

## Cenário D — Fechamento de emergência sem horário de retorno (`FECHADA_EMERGENCIA` sem `reopen_at`)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Fechou em emergência. Sem previsão de retorno |
| 2 | 👤 Cliente | Banner com motivo obrigatório: **"Loja temporariamente fechada. Motivo: [motivo]."** |
| 3 | 🔑 Gerente | Pode ter deixado uma mensagem extra explicativa visível ao cliente |
| 4 | Sistema | Sem countdown. Somente **"Voltamos em breve."** |
| 5 | 👤 Cliente | Cardápio cinza, sem adição ao carrinho |
| 6 | 🔑 Gerente | Quando a situação normaliza, reabre manualmente |

---

## Cenário E — Loja fecha enquanto o cliente monta o carrinho

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Logado com loja `ABERTA`; recebe `shoppingSessionId`; monta carrinho |
| 2 | Sistema | Job de fechamento → `FECHADA_HORARIO` (ou pausa programada) |
| 3 | 👤 Cliente | Continua vendo carrinho / checkout — **não** perde sessão de compra até `expires_at` (**3 h**) |
| 4 | Sistema | `POST /orders` **não** exige loja aberta; valida itens + sessão |
| 5 | 👤 Cliente | Novo visitante que abre o app **agora** cai nos cenários A–D (cinza + countdown) |

---

> Pedidos já na cozinha antes do fechamento **não são afetados**.  
> O bloqueio cinza é para **entrada nova**; checkout “legacy” usa [DB-04 § 7](../../docTecnica/DB-04-catalogo-checkout-cliente.md).

> 📌 **Nota técnica resumida:**  
> `store_status` lido em tempo real via WebSocket. Se fechado: cardápio renderizado com flag `disabled`, banner injetado no topo. Countdown calcula diferença entre `now()` e `next_opens_at` (vem da API, computado no servidor). O frontend só exibe o timer. Para `FECHADA_EMERGENCIA`: `reopen_at` pode ser nulo; se nulo, não exibe countdown.  
> → Detalhes: [`../../docTecnica/DB-02-disponibilidade-itens.md`](../../docTecnica/DB-02-disponibilidade-itens.md)
