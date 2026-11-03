# FLUX-HAPPY-15-disputa
### Dois motoboys disputam o mesmo pedido — um assume

**Perfis:** 🛵 🛵

[← Índice happy path](README.md)

---

**Perfis:** 🛵 Motoboy A · 🛵 Motoboy B

```
Pedido fica "Pronto" → dois motoboys clicam **Aceitar pedido** ao mesmo tempo → só um consegue
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy A | Vê o pedido na fila e clica **Aceitar pedido** |
| 2 | 🛵 Motoboy B | Ao mesmo tempo, clica **Aceitar pedido** no mesmo pedido |
| 3 | Sistema | Processa os dois cliques; o Motoboy A chegou primeiro |
| 4 | 🛵 Motoboy A | Recebe confirmação: **"Pedido #0042 é seu!"** — pedido vai direto para `SAIU_PARA_ENTREGA` |
| 5 | 🛵 Motoboy B | Recebe aviso: **"Esse pedido já foi assumido"** — vê a fila atualizada |
| 6 | Sistema | O pedido #0042 sai da fila de disponíveis — Motoboy B só vê outros pedidos |

> 💡 **Aceitar pedido** dispara `POST /take`: lock e transição para `SAIU_PARA_ENTREGA` na mesma transação. Quem chega primeiro no servidor ganha; o outro recebe HTTP 409.
