# FLUX-HAPPY-13-usuarios
### Gerente cadastra e desativa cozinha e motoboy

**Perfis:** 🔑

[← Índice happy path](README.md)

---

**Perfis:** 🔑 Gerente · 👨‍🍳 Cozinha · 🛵 Motoboy

```
Gerente cria o usuário da loja → a pessoa entra com o próprio login
→ gerente desativa quando ela sai da operação
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Abre **Usuários da loja** e clica em **Novo** |
| 2 | 🔑 Gerente | Informa nome, e-mail, senha inicial e o perfil: **Cozinha** ou **Motoboy** |
| 3 | Sistema | Cria o usuário vinculado a esta loja |
| 4 | 👨‍🍳 Cozinha ou 🛵 Motoboy | Entra no `web-operacao` e vê só o menu do próprio perfil |
| 5 | 🔑 Gerente | Quando a pessoa não opera mais, clica em **Desativar** |
| 6 | Sistema | Encerra a sessão e impede novo login |
| 7 | 🛵 Motoboy | Se estava na fila, sai da fila. Pedido já `ALOCADO` ou `SAIU_PARA_ENTREGA` com ele precisa ser resolvido antes: [FLUX-SAD-16-entrega-desiste](../sadPaths/FLUX-SAD-16-entrega-desiste.md) ou [FLUX-SAD-18-rota-gerente](../sadPaths/FLUX-SAD-18-rota-gerente.md) |

> Desativar a conta não é o mesmo que o motoboy encerrar o turno. O turno é ele mesmo, no app: [FLUX-HAPPY-16-turno](FLUX-HAPPY-16-turno.md). A conta desativada não consegue iniciar turno.
