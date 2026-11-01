# FLUX-SAD-15-disputa
### Dois motoboys — um perde a corrida

**Gravidade:** 🟠 · **Perfis:** 🛵 🛵

[← Índice sad path](README.md)

---

**Perfis:** 🛵 Motoboy A · 🛵 Motoboy B
**Causa:** Dois entregadores clicam em "Assumir" ao mesmo tempo.

```
Dois motoboys veem o mesmo pedido → um clica "Assumir e sair" → o outro recebe aviso claro
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy A | Vê pedido #0042 disponível e clica **Assumir e sair** |
| 2 | 🛵 Motoboy B | No mesmo instante, clica **Assumir e sair** no mesmo pedido |
| 3 | Sistema | Processa os dois; o Motoboy A chegou primeiro |
| 4 | 🛵 Motoboy A | Confirma: **"Pedido #0042 assumido!"** — pedido vai direto para `SAIU_PARA_ENTREGA` |
| 5 | 🛵 Motoboy B | Recebe: **"Esse pedido já foi assumido por outro motoboy."** |
| 6 | 🛵 Motoboy B | A lista de pedidos é atualizada — ele vê apenas pedidos ainda disponíveis |

> **Impacto:** Baixo. Motoboy B perde a corrida mas não há confusão — o sistema é claro e a fila é atualizada na hora.
