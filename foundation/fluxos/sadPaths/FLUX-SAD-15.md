# FLUX-SAD-15
### Gerente desaloca motoboy em trânsito

**Gravidade:** 🔴 · **Perfis:** 🔑 🛵 👤

[← Índice sad path](README.md)

---

**Perfis:** 🔑 Gerente · 🛵 Motoboy · 👤 Cliente
**Causa:** Situação excepcional — acidente do motoboy, problema de segurança.

```
Motoboy saiu → gerente intervém → pedido volta a PRONTO → novo motoboy assume
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Assumiu o pedido, está com status `ALOCADO` ou `SAIU_PARA_ENTREGA` |
| 2 | 🔑 Gerente | Acessa o painel e vai em **Pedidos em andamento** |
| 3 | 🔑 Gerente | Clica em **Desalocar motoboy** no pedido específico |
| 4 | Sistema | Pedido volta ao estado **PRONTO** |
| 5 | 🛵 Motoboy | Recebe aviso no app: **"Você foi removido deste pedido."** |
| 6 | 👤 Cliente | A timeline para em "Pronto" — sem avanço por enquanto |
| 7 | 🛵 Novo motoboy | Vê o pedido na fila e pode assumir |
| 8 | 👤 Cliente | Timeline avança quando o novo motoboy assume |

> **Impacto:** Alto. Atraso significativo na entrega. Toda a ação fica registrada na auditoria.
