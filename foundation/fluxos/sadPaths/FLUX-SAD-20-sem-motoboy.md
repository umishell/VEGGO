# FLUX-SAD-20-sem-motoboy
### Nenhum motoboy ativo para assumir

**Gravidade:** 🔴 · **Perfis:** 👨‍🍳 🔑 👤

[← Índice sad path](README.md)

---

**Perfis:** 👨‍🍳 Cozinha · 🔑 Gerente · 👤 Cliente  
**Causa:** Pedido ficou `PRONTO` mas todos os motoboys estão ocupados ou inativos.

```
Pedido pronto → fila vazia de motoboys → gerente gerencia a situação
→ Caminho A: ativa novo motoboy → pedido é assumido normalmente
→ Caminho B: não há saída → gerente cancela com estorno
```

---

## Caminho A — Gerente aciona novo motoboy

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👨‍🍳 Cozinha | Marca pedido como **Pronto** |
| 2 | Sistema | Pedido entra na fila de disponíveis — mas não há motoboy ativo |
| 3 | 🔑 Gerente | Vê no painel: aviso de **"Pedido aguardando motoboy há X minutos"** |
| 4 | 🔑 Gerente | Contata um motoboy de folga ou ativa um cadastrado |
| 5 | 🛵 Motoboy | Inicia o turno e assume o pedido normalmente |
| 6 | 👤 Cliente | Recebe atraso, mas o pedido segue sem cancelamento |

---

## Caminho B — Gerente cancela com estorno

Se não há perspectiva de motoboy disponível em tempo razoável, o gerente pode encerrar o pedido.

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | No painel, abre o pedido em `PRONTO` que aguarda há muito tempo |
| 2 | 🔑 Gerente | Clica em **Cancelar pedido** e informa o motivo (ex: *"Sem motoboy disponível"*) |
| 3 | Sistema | Muda o status para `CANCELADO` e inicia o estorno (`CONFIRMADO → ESTORNADO`) |
| 4 | Sistema | Envia evento WebSocket ao cliente: pedido cancelado com o motivo informado |
| 5 | 👤 Cliente | Vê na timeline: *"Pedido cancelado — [motivo]"* e recebe o estorno |
| 6 | 🔑 Gerente | O cancelamento fica registrado na auditoria com responsável e motivo |

> O gerente pode cancelar `PRONTO` **com ou sem** motoboy associado (sem motoboy é exatamente este cenário). Ver [ORD-01](../../docTecnica/ORD-01-estados-pedido.md) §2.

> **Impacto:** Alto. No Caminho A, apenas atraso. No Caminho B, o cliente perde o pedido mas recebe o estorno.

---

**Veja também:**  
→ [FLUX-SAD-11-cancela-loja](FLUX-SAD-11-cancela-loja.md) — gerente cancela qualquer pedido específico  
→ [FLUX-SAD-16-entrega-desiste](FLUX-SAD-16-entrega-desiste.md) — motoboy desiste antes de sair da loja
