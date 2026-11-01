# FLUX-HAPPY-01-cadastro
### Cadastro e primeiro acesso

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente

```
Cliente abre o app pela primeira vez
→ Vê tela de boas-vindas com dois botões: "Entrar" e "Criar conta"
→ Clica em "Criar conta"
→ Preenche nome, e-mail e senha
→ Confirma e vê a tela inicial do cardápio
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Abre o app e vê a tela de boas-vindas |
| 2 | 👤 Cliente | Clica em **Criar conta** |
| 3 | 👤 Cliente | Preenche **nome completo**, **e-mail**, **telefone** e **senha** |
| 4 | 👤 Cliente | Clica em **Cadastrar** |
| 5 | Sistema | Valida os dados, cria a conta e faz login automático |
| 6 | 👤 Cliente | É redirecionado direto para o **cardápio** — já logado |

> 💡 Na próxima vez que abrir o app, o cliente já estará logado automaticamente (sessão ativa).

> 📞 O **telefone** é usado exclusivamente para o motoboy entrar em contato durante a entrega. Fica visível ao motoboy apenas enquanto o pedido estiver em `SAIU_PARA_ENTREGA`. Não é exibido publicamente.
