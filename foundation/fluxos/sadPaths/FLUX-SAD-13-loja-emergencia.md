# FLUX-SAD-13-loja-emergencia
### Gerente fecha a loja em emergência (fechamento antecipado)

**Gravidade:** 🔴 · **Perfis:** 🔑 👤 👨‍🍳

[← Índice sad path](README.md)

---

**Perfis:** 🔑 Gerente · 👤 Cliente · 👨‍🍳 Cozinha  
**Causa:** Situação operacional grave impossibilita o funcionamento normal — muitos ingredientes em falta, problema de equipamento, emergência interna.  
**Frequência:** Rara — é um recurso de exceção.

> ⚠️ **Princípio fundamental:** a cozinha **não pode recusar pedidos que já chegaram**. A estratégia do sistema é **prevenir** que pedidos cheguem para itens indisponíveis, não recusá-los depois. Pedidos já recebidos seguem na cozinha. Cancelar um pedido específico, com estorno, é outra ação do gerente: [FLUX-SAD-11-cancela-loja](FLUX-SAD-11-cancela-loja.md). Fechar a loja não dispara esse cancelamento.

```
Cozinha identifica problema grave → alerta gerente
→ gerente fecha a loja com motivo → novos pedidos bloqueados → pedidos em andamento continuam
→ clientes veem motivo do fechamento → gerente reabre quando situação normaliza
```

---

## Camadas de prevenção (antes de chegar neste fluxo)

| Camada | Quem | O que faz | Quando usar |
|--------|------|-----------|-------------|
| **1 — Item individual** | 👨‍🍳 Cozinha | Marca item como "Esgotado hoje" ([FLUX-HAPPY-11-item-cozinha](../happypaths/FLUX-HAPPY-11-item-cozinha.md)) | Falta de um ingrediente específico |
| **2 — Item individual** | 🔑 Gerente | Pausa item do cardápio ([FLUX-HAPPY-10-item-gerente](../happypaths/FLUX-HAPPY-10-item-gerente.md)) | Mesmo que acima, com efeito permanente |
| **3 — Fechamento total** | 🔑 Gerente | **Este fluxo** | Múltiplos itens indisponíveis / impossível operar |

> A cozinha **não tem permissão** para fechar tudo sozinha. Se precisar de fechamento total, deve alertar o gerente.

---

## Passo a passo — fechamento de emergência

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👨‍🍳 Cozinha | Identifica que não é possível operar normalmente (ex: falta de vários ingredientes, equipamento parou) |
| 2 | 👨‍🍳 Cozinha | Já pausou os itens individuais que pode — mas a situação exige fechamento total |
| 3 | 👨‍🍳 Cozinha | Alerta o gerente pelo canal interno (fora do app) |
| 4 | 🔑 Gerente | Acessa o painel → **Operações** → **Encerrar atividades da loja** |
| 5 | Sistema | Exige preenchimento obrigatório: **"Informe o motivo do fechamento antecipado"** |
| 6 | 🔑 Gerente | Seleciona ou digita o motivo (ex: *"Falta crítica de ingredientes"*, *"Problema de equipamento"*, *"Emergência interna"*) |
| 7 | 🔑 Gerente | Confirma o fechamento |
| 8 | Sistema | Status da loja muda para `FECHADA_EMERGENCIA` |
| 9 | Sistema | **Novos pedidos são bloqueados** — checkout exibe aviso de loja fechada |
| 10 | Sistema | Exibe banner para todos os clientes: *"Loja temporariamente fechada. Motivo: [motivo]. Pedimos desculpas pelo inconveniente! Voltamos em breve."* |
| 11 | 👨‍🍳 Cozinha | Pedidos que já chegaram **continuam sendo preparados e entregues normalmente** |
| 12 | 🔑 Gerente | Quando a situação normaliza, clica em **"Reabrir loja"** |
| 13 | Sistema | Status volta para `ABERTA` — novos pedidos liberados |
| 14 | Sistema | Banner de fechamento removido do app |

---

## O que o cliente vê

| Momento | O que aparece no app |
|---------|---------------------|
| Antes do fechamento | Cardápio normal — pode pedir |
| Durante o fechamento | Banner de aviso com motivo · botão de finalizar pedido desabilitado |
| Após reabertura | Cardápio normal — pode pedir novamente |

---

**Veja também:**  
→ [FLUX-HAPPY-10-item-gerente](../happypaths/FLUX-HAPPY-10-item-gerente.md) — gerente pausa item individual (prevenção nível 2)  
→ [FLUX-HAPPY-10-item-gerente](../happypaths/FLUX-HAPPY-10-item-gerente.md) — gerente pausa item no cardápio (prevenção nível 1)  
→ [FLUX-HAPPY-12-horario](../happypaths/FLUX-HAPPY-12-horario.md) — loja abre e fecha automaticamente no horário configurado  
→ [FLUX-SAD-11-cancela-loja](FLUX-SAD-11-cancela-loja.md) — gerente cancela pedido já aceito (emergência extrema, último recurso)

---

> **Impacto:** Alto — novos clientes não conseguem pedir. Pedidos já em andamento não são afetados. O motivo visível ao cliente reduz a frustração e mantém a transparência.

---

> 📌 **Nota técnica resumida:**  
> Campo `store_status` na tabela `stores`: `ABERTA` | `FECHADA_HORARIO` | `FECHADA_EMERGENCIA`. Fechamento de emergência grava `closed_reason` (texto) e `closed_at` (timestamp). O banner do cliente lê `store_status` e `closed_reason` em tempo real.  
> → Detalhes: [`../../docTecnica/DB-02-disponibilidade-itens.md`](../../docTecnica/DB-02-disponibilidade-itens.md)
