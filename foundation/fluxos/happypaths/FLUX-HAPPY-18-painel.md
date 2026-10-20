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

> Esta tela é diferente do KDS da cozinha. Na cozinha, o pedido desaparece quando concluído. Aqui, **todos os pedidos do dia permanecem visíveis**, com o estado atual destacado.

---

## Passo a passo — visão do painel do dia

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Abre **Painel → Pedidos do dia** |
| 2 | Sistema | Lista todos os pedidos criados desde a última abertura da loja, por ordem de criação (mais recente no topo) |
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
| `ALOCADO` | 🟣 Roxo | Motoboy a caminho |
| `SAIU_PARA_ENTREGA` | 🟤 Âmbar | Em entrega |
| `ENTREGUE` | 🟢 Verde | Entregue |
| `CANCELADO` | ⬜ Cinza | Cancelado |
| Imprevisto em rota | 🔴 Vermelho | Imprevisto — sendo refeito |

---

## Filtros disponíveis

| Filtro | Opções |
|--------|--------|
| Estado | Todos / por estado individual |
| Período | Desde a abertura / últimas X horas |
| Motoboy | Todos / por motoboy específico |
| Valor | Faixa de valor do pedido |
| Texto | Número do pedido ou nome do cliente |

---

> 💡 O painel do dia não reseta na reabertura de um fechamento programado. O histórico do dia continua acumulando pedidos.  
> 💡 O gerente pode usar esse painel para cobrar a cozinha (pedido parado em `RECEBIDO` há muito tempo) sem precisar ir ao KDS.

---

> 📌 **Nota técnica resumida:**  
> Mesma conexão WebSocket do monólito Spring. O gerente se inscreve no canal `kitchen:{storeId}` para receber todos os eventos de estado do dia. A listagem inicial vem por REST (`GET /orders?date=today`). Os eventos chegam e atualizam o estado do item na tabela sem recarregar.  
> → Detalhes: [`../../docTecnica/INF-01-tempo-real.md`](../../docTecnica/INF-01-tempo-real.md) · [`../../docTecnica/DB-03-telemetria.md`](../../docTecnica/DB-03-telemetria.md)
