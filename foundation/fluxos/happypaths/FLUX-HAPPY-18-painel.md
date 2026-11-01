# FLUX-HAPPY-18-painel
### Gerente acompanha pedidos do dia em tempo real

**Perfis:** 🔑

[← Índice happy path](README.md)

---

**Perfis:** 🔑 Gerente

```
Gerente abre o painel do dia → vê todos os pedidos com estado colorido
→ atualizam em tempo real → filtra e monitora a operação
```

> Esta tela é diferente do KDS da cozinha. Na cozinha, o pedido desaparece quando concluído. Aqui, **todos os pedidos do dia operacional atual** permanecem visíveis, com o estado atual destacado. “Dia operacional” = data civil no fuso da loja (meia-noite a meia-noite), **não** “desde a última abertura”.

---

## Passo a passo — visão do painel do dia

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Abre **Painel → Pedidos do dia** |
| 2 | Sistema | Lista todos os pedidos **criados hoje** (dia operacional da loja), por ordem de criação (mais recente no topo) |
| 3 | 🔑 Gerente | Vê cada pedido com: número, cliente, itens resumidos, horário, valor, estado atual e indicador de cor |
| 4 | 🔑 Gerente | Qualquer mudança de estado em qualquer pedido aparece na tela **automaticamente**, sem recarregar |
| 5 | 🔑 Gerente | Clica em um pedido para ver o detalhe completo — itens, endereço, histórico de estados |

---

## Indicadores visuais por estado

| Estado | Cor | Texto exibido |
|--------|-----|---------------|
| `RECEBIDO` | 🟡 Amarelo | Na fila — aguardando cozinha |
| `EM_PREPARO` | 🟠 Laranja | Em preparo |
| `PRONTO` | 🔵 Azul | Pronto — aguardando motoboy |
| `SAIU_PARA_ENTREGA` | 🟤 Âmbar | Em entrega |
| Imprevisto em rota (priority) | 🔴 Vermelho + ⚡ | Imprevisto — refazendo com prioridade |
| `ENTREGUE` | 🟢 Verde | Entregue |
| `CANCELADO` | ⬜ Cinza | Cancelado |

---

## Filtros disponíveis

| Filtro | Opções |
|--------|--------|
| Estado | Todos / por estado individual |
| Período | Hoje (dia operacional) / últimas X horas |
| Motoboy | Todos / por motoboy específico |
| Valor | Faixa de valor do pedido |
| Texto | Número do pedido ou nome do cliente |

---

> 💡 Pausa programada (`FECHADA_PROGRAMADA`) **não** zera a lista: pedidos feitos antes e depois da pausa no **mesmo dia civil** continuam juntos. Só a **meia-noite** (fuso da loja) inicia um painel vazio para o dia seguinte.  
> 💡 O gerente pode usar esse painel para cobrar a cozinha (pedido parado em `RECEBIDO` há muito tempo) sem precisar ir ao KDS.

---

> 📌 **Nota técnica resumida:**  
> O gerente assina o canal WebSocket `kitchen:{storeId}`. A listagem inicial vem por REST (`GET /orders?date=today`). Cada mudança de estado de qualquer pedido chega via push e atualiza a linha na tabela sem recarregar. Ações pontuais (cancelar pedido, acionar imprevisto) são HTTP POST.  
> → Decisão de stack: [`../../docTecnica/INF-01-tempo-real.md`](../../docTecnica/INF-01-tempo-real.md)  
> → Protocolo por ator, canais e payloads: [`../../docTecnica/INF-02-http-vs-websocket.md`](../../docTecnica/INF-02-http-vs-websocket.md)  
> → Queries de agregação: [`../../docTecnica/DB-03-telemetria.md`](../../docTecnica/DB-03-telemetria.md)
