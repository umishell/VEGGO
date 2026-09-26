# FLUX-HAPPY-12
### Motoboy encerra turno (fica inativo)

**Perfis:** 🛵

[← Índice happy path](README.md)

---

**Perfis:** 🛵 Motoboy

```
Fim do expediente → motoboy se marca como inativo → sai da fila de entregas
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | No app, clica em **"Encerrar turno"** ou desativa o status de disponível |
| 2 | Sistema | Remove o motoboy da fila de disponíveis |
| 3 | Sistema | Novos pedidos não serão mais mostrados para ele |
| 4 | 🛵 Motoboy | Pode fechar o app e ir embora |
| 5 | 🔑 Gerente | No painel, vê o motoboy como **inativo** na lista da loja |

> 💡 Se o motoboy já tiver um pedido **em andamento**, precisa finalizá-lo antes de encerrar o turno.

---

---

*Próximo documento: [`cenarios_Problemas_Erros.md`](../cenarios_Problemas_Erros.md) — todos os cenários de falha, erro e exceção.*
