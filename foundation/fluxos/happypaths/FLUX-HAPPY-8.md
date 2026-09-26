# FLUX-HAPPY-8
### Dois motoboys disputam o mesmo pedido — um assume

**Perfis:** 🛵 🛵

[← Índice happy path](README.md)

---

**Perfis:** 🛵 Motoboy A · 🛵 Motoboy B

```
Pedido fica "Pronto" → dois motoboys clicam "Assumir" ao mesmo tempo → só um consegue
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy A | Vê o pedido #0042 na fila e clica **Assumir** |
| 2 | 🛵 Motoboy B | Ao mesmo tempo, também vê o pedido #0042 e clica **Assumir** |
| 3 | Sistema | Processa os dois cliques; o Motoboy A chegou primeiro |
| 4 | 🛵 Motoboy A | Recebe confirmação: **"Pedido #0042 é seu!"** — vai buscar na loja |
| 5 | 🛵 Motoboy B | Recebe aviso: **"Esse pedido já foi assumido"** — vê a fila atualizada |
| 6 | Sistema | O pedido #0042 sai da fila de disponíveis — Motoboy B só vê outros pedidos |

> 💡 Isso garante que **nunca** dois motoboys saiam para o mesmo endereço com o mesmo pedido.
