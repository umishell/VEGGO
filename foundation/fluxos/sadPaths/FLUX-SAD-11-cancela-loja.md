# FLUX-SAD-11-cancela-loja
### Gerente cancela um pedido específico

**Gravidade:** 🔴 · **Perfis:** 🔑 👤

[← Índice sad path](README.md)

---

**Perfis:** 🔑 Gerente · 👤 Cliente · 👨‍🍳 Cozinha  
**Causa:** Um pedido já recebido não pode ser concluído — falta de luz no meio do preparo, erro operacional desse pedido.  
**Não é** o fechamento da loja. Fechar a loja bloqueia pedido novo e deixa os atuais seguirem: [FLUX-SAD-13-loja-emergencia](FLUX-SAD-13-loja-emergencia.md).

```
Um pedido em andamento → gerente cancela esse pedido → cozinha para esse pedido → cliente recebe estorno
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👨‍🍳 Cozinha | O pedido está em `RECEBIDO`, `EM_PREPARO` ou `PRONTO` sem motoboy |
| 2 | 🔑 Gerente | Abre o pedido e clica em **Cancelar pedido** |
| 3 | 🔑 Gerente | Informa o motivo e confirma |
| 4 | Sistema | Muda para `CANCELADO` e estorna o pagamento confirmado |
| 5 | 👨‍🍳 Cozinha | Esse pedido sai da tela |
| 6 | 👤 Cliente | Recebe **"Seu pedido foi cancelado pela loja. Motivo: [motivo]. Pagamento estornado."** |
| 7 | 👤 Cliente | Vê o pedido como **Cancelado** no histórico |

> Pedido em `SAIU_PARA_ENTREGA` não é cancelado com estorno: o gerente aciona o protocolo de imprevisto e o mesmo pedido volta à fila com prioridade ([FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md)).

> **Impacto:** Alto. Estorno e auditoria com responsável, horário, motivo e estado anterior.
