# FLUX-HAPPY-15-disputa
### Dois motoboys disputam o mesmo pedido — um assume

**Perfis:** 🛵 🛵

[← Índice happy path](README.md)

---

**Perfis:** 🛵 Motoboy A · 🛵 Motoboy B

```
Pedido fica "Pronto" → dois motoboys clicam "Assumir e sair" ao mesmo tempo → só um consegue
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy A | Vê o pedido #0042 na fila e clica **Assumir e sair** |
| 2 | 🛵 Motoboy B | Ao mesmo tempo, também vê o pedido #0042 e clica **Assumir e sair** |
| 3 | Sistema | Processa os dois cliques; o Motoboy A chegou primeiro |
| 4 | 🛵 Motoboy A | Recebe confirmação: **"Pedido #0042 é seu!"** — pedido vai direto para `SAIU_PARA_ENTREGA` |
| 5 | 🛵 Motoboy B | Recebe aviso: **"Esse pedido já foi assumido"** — vê a fila atualizada |
| 6 | Sistema | O pedido #0042 sai da fila de disponíveis — Motoboy B só vê outros pedidos |

> 💡 "Assumir e sair" é **um único passo**: lock e transição para `SAIU_PARA_ENTREGA` acontecem na mesma transação. Não existe estado intermediário "assumido mas ainda na loja". Quem chega primeiro no servidor ganha; o outro recebe HTTP 409.
