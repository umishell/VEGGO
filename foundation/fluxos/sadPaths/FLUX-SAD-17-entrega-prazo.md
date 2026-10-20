# FLUX-SAD-17-entrega-prazo
### Motoboy assume e não retira no prazo

**Gravidade:** 🟠 · **Perfis:** 🛵 👤

[← Índice sad path](README.md)

---

**Perfis:** 🛵 Motoboy · 👤 Cliente · 🔑 Gerente  
**Causa:** O pedido ficou `ALOCADO` e o motoboy não confirmou a retirada dentro do prazo da loja.

```
Motoboy assume → não retira → prazo estoura → pedido volta a PRONTO → outro motoboy pega o mesmo lanche
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Assume o pedido. Estado `ALOCADO`. O lanche continua na loja |
| 2 | Sistema | Conta o prazo `allocation_timeout_minutes` |
| 3 | 🛵 Motoboy | Não clica em **Saiu para entrega** |
| 4 | Sistema | Devolve o pedido para `PRONTO` e tira o motoboy |
| 5 | 🛵 Motoboy | Recebe **"O pedido voltou para a fila porque a retirada não foi confirmada."** |
| 6 | 🛵 Outro motoboy | Vê o pedido na fila e pode assumir. Pega o **mesmo** lanche |
| 7 | 👤 Cliente | A timeline não avança. Não há ícone de imprevisto e não há estorno |

> Este prazo só existe antes da saída. Depois que o motoboy saiu, o caminho é [FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md) ou [FLUX-SAD-19-rota-motoboy](FLUX-SAD-19-rota-motoboy.md).

---

> 📌 **Nota técnica resumida:**  
> `stores.allocation_timeout_minutes`. Transição `ALOCADO → PRONTO` com motivo de timeout na auditoria.  
> → [`../../docTecnica/ORD-01-estados-pedido.md`](../../docTecnica/ORD-01-estados-pedido.md)
