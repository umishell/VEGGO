# FLUX-HAPPY-7
### Cozinha marca item como "esgotado hoje"

**Perfis:** 👨‍🍳

[← Índice happy path](README.md)

---

**Perfis:** 👨‍🍳 Funcionário da Cozinha

```
Acabou a mussarela → cozinheiro marca o item como indisponível → reseta automaticamente no dia seguinte
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👨‍🍳 Cozinha | Na tela da cozinha, clica em **Gerenciar disponibilidade** |
| 2 | 👨‍🍳 Cozinha | Encontra **Smash Tradicional** e clica em **"Esgotado hoje"** |
| 3 | Sistema | Marca o item como indisponível para este dia |
| 4 | Sistema | O item some imediatamente do cardápio do cliente |
| 5 | Sistema | Na virada do dia (meia-noite), o item volta automaticamente ao cardápio |
| 6 | 👨‍🍳 Cozinha | No dia seguinte, o item já está disponível novamente sem precisar fazer nada |

> 💡 Diferença do gerente: a cozinha marca "acabou hoje" — efeito temporário. O gerente pausa — efeito manual (ele precisa reativar).
