# FLUX-SAD-01-item-cinza
### Item indisponível no cardápio

**Gravidade:** 🟡 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente  
**Causa:** Gerente marcou `PAUSADO_DIA` ou `BLOQUEADO` (cozinha avisou falta de insumo, se for o caso).

```
Item continua na lista, cinza, com texto → cliente não consegue pedir
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente ou 👨‍🍳 Cozinha | Pausa para o dia ou bloqueia o item |
| 2 | 👤 Cliente | Abre o cardápio e **ainda vê** o item |
| 3 | Sistema | Desenha o card cinza |
| 4 | 👤 Cliente | Se for pausa do dia, lê **"Não está sendo produzido hoje."** |
| 5 | 👤 Cliente | Se for bloqueio, lê **"Indisponível no momento."** |
| 6 | 👤 Cliente | Não há botão de adicionar. O item não entra no carrinho |

Item **apagado** pelo gerente não passa por esta tela: ele não está mais na lista.

> **Impacto:** Baixo. O cliente entende por que não pede, em vez de achar que o produto foi tirado do cardápio para sempre.

**Resolução:** reativação manual pelo **gerente**. `PAUSADO_DIA` também volta sozinho na meia-noite. `BLOQUEADO` não.

---

## Diferença entre cinza e invisível

| Situação | O cliente vê o item? | Por que |
|----------|---------------------|---------|
| `PAUSADO_DIA` | ✅ Sim — cinza | Ingrediente acabou hoje; produto voltará amanhã |
| `BLOQUEADO` | ✅ Sim — cinza | Produto temporariamente suspenso; volta quando gerente/cozinha reativar |
| `ARQUIVADO` | ❌ Não aparece | Gerente optou por ocultar o item completamente (sem dar ciência ao cliente). Apenas gerente vê na seção "Itens arquivados" |
| Apagado (físico) | ❌ Não existe | Excluído permanentemente — sem volta |

> **Regra de UX:** itens cinzas comunicam ao cliente que o produto existe mas está temporariamente fora. `ARQUIVADO` é para itens que não devem nem criar expectativa — produtos descontinuados na prática, em reformulação, ou sazonais sem previsão de retorno próximo.
