# FLUX-SAD-12
### Cliente sem endereço tenta finalizar

**Gravidade:** 🟡 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente
**Causa:** Primeiro pedido do cliente sem endereço salvo.

```
Cliente tenta finalizar → sistema pede endereço → cliente cadastra → finaliza
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Clica em **Finalizar Pedido** |
| 2 | Sistema | Detecta que não há endereço de entrega cadastrado |
| 3 | Sistema | Exibe formulário: **"Informe seu endereço de entrega"** |
| 4 | 👤 Cliente | Preenche rua, número, bairro e CEP |
| 5 | Sistema | Calcula o frete pela região e mostra o valor |
| 6 | 👤 Cliente | Confirma e finaliza normalmente |

> **Impacto:** Mínimo. É uma etapa extra, não um erro real. O sistema guia o cliente.
