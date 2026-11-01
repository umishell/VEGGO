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
| 2 | 🔑 Gerente | Informa nome, e-mail, senha inicial e o perfil: **Cozinha** ou **Motoboy**. Para perfil Motoboy: também informa o **telefone** (obrigatório — usado pelo cliente para contato durante a entrega) |
| 3 | Sistema | Cria o usuário vinculado a esta loja |
| 4a | 👨‍🍳 Cozinha | Entra no `web-operacao` (Angular, no navegador) e vê só o menu de cozinha |
| 4b | 🛵 Motoboy | Entra no **`app-motoboy`** (app Android) com o e-mail e senha criados — **não** no `web-operacao` |
| 5 | 🔑 Gerente | Quando a pessoa não opera mais, clica em **Desativar** |
| 6 | Sistema | Verifica se o motoboy tem pedido em `SAIU_PARA_ENTREGA` |
| 6a | *(sem entrega ativa)* | Encerra a sessão, impede novo login, remove da fila se estava nela |
| 6b | *(com entrega ativa)* | **Bloqueia a ação.** Exibe: *"Este motoboy tem uma entrega em andamento. Resolva o imprevisto antes de desativar a conta."* Gerente deve usar [FLUX-SAD-18-rota-gerente](../sadPaths/FLUX-SAD-18-rota-gerente.md) para acionar o protocolo de imprevisto primeiro |

> Desativar a conta não é o mesmo que o motoboy encerrar o turno. O turno é ele mesmo, no app: [FLUX-HAPPY-16-turno](FLUX-HAPPY-16-turno.md). A conta desativada não consegue iniciar turno.
