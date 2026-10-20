# FLUX-HAPPY-16-turno
### Motoboy encerra turno e inicia o turno no dia seguinte

**Perfis:** 🛵

[← Índice happy path](README.md)

---

**Perfis:** 🛵 Motoboy

```
Fim do expediente → motoboy encerra turno → fica inativo
→ no dia seguinte abre o app → vê tela de "Iniciar turno" → clica → entra na fila
```

---

## Parte 1 — Encerrar turno

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | No app, clica em **"Encerrar turno"** |
| 2 | Sistema | Remove o motoboy da fila de disponíveis — status `INATIVO` |
| 3 | Sistema | Novos pedidos não são mais exibidos para ele |
| 4 | 🛵 Motoboy | Pode fechar o app e ir embora |
| 5 | 🔑 Gerente | No painel, vê o motoboy como **inativo** na lista |

> ⚠️ Se o motoboy tiver um pedido **em andamento**, deve finalizá-lo antes de encerrar o turno.

---

## Parte 2 — Iniciar turno no dia seguinte

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 6 | 🛵 Motoboy | Abre o app no dia seguinte |
| 7 | Sistema | Detecta que o status é `INATIVO` (sessão ativa, turno não iniciado) |
| 8 | Sistema | Exibe a tela inicial do motoboy com o botão **"Iniciar turno"** em destaque |
| 9 | 🛵 Motoboy | Clica em **"Iniciar turno"** |
| 10 | Sistema | Status muda para `DISPONÍVEL` — motoboy entra na fila de entregas |
| 11 | 🛵 Motoboy | Pedidos disponíveis começam a aparecer no app normalmente |
| 12 | 🔑 Gerente | No painel, vê o motoboy como **ativo** |

> 💡 O status **não reinicia automaticamente** ao abrir o app — o motoboy precisa clicar em "Iniciar turno" conscientemente. Isso evita que o app o coloque na fila sem ele estar pronto para trabalhar.

> 💡 O app mantém a sessão logada entre os dias — o motoboy **não precisa fazer login** toda vez, apenas iniciar o turno.

---

> 📌 **Nota técnica resumida:**  
> O status do motoboy (`INATIVO` / `DISPONÍVEL` / `EM_ENTREGA`) é persistido na tabela `delivery_persons`. O app lê esse status ao abrir e renderiza a tela adequada. O turno não tem reset automático — é sempre uma ação explícita do motoboy.
