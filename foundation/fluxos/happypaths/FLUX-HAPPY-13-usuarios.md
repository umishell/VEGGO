# FLUX-HAPPY-13-usuarios
### Gerente cadastra, controla e desativa funcionários (cozinha e motoboy)

**Perfis:** 🔑

[← Índice happy path](README.md)

---

**Perfis:** 🔑 Gerente · 👨‍🍳 Cozinha · 🛵 Motoboy

```
Somente o CLIENTE se autocadastra no app
→ cozinha e motoboy entram só pelo gerente, na tela Funcionários
→ senha criada pelo funcionário (gerente não vê)
→ gerente conclui o cadastro
```

> **Tela Funcionários** no `web-operacao` (listagem, cadastro, desativar, reset de senha). Cozinha e motoboy **não** têm fluxo “Criar conta” nos apps.  
> **Cozinha:** após login, qualquer conta com perfil Cozinha abre o **KDS** compartilhado da loja. O gerente **não** designa quem opera a fila no MVP.

---

## Parte 1 — Cadastro presencial (nova conta)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Abre **Funcionários → Novo funcionário** |
| 2 | 🔑 Gerente | Preenche dados **visíveis ao gerente**: nome, e-mail, perfil (**Cozinha** ou **Motoboy**). Motoboy: **telefone** obrigatório |
| 3 | 🔑 Gerente | Clica em **Continuar — definir senha com o funcionário** |
| 4 | Sistema | Exibe passo **“Senha do funcionário”**: campo de senha em modo **privado para o funcionário** (o gerente não vê caracteres; idealmente o funcionário segura o teclado/mouse ou usa o próprio celular na etapa, se o fluxo for mobile) |
| 5 | 👨‍🍳 ou 🛵 Funcionário | Digita e confirma a **própria senha** sem o gerente enxergar |
| 6 | 🔑 Gerente | Clica em **Concluir cadastro** |
| 7 | Sistema | Persiste o usuário com hash Argon2id **somente no Spring**; vincula à loja e ao perfil |
| 8 | 👨‍🍳 Cozinha | Entra no `web-operacao` com e-mail + senha que definiu |
| 9 | 🛵 Motoboy | Entra no **`app-motoboy`** com e-mail + senha que definiu |

> O gerente **nunca** informa nem recebe a senha em texto. Não há “senha inicial” temporária definida pelo gerente.

---

## Parte 2 — Reset de senha (funcionário presente)

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👨‍🍳 ou 🛵 Funcionário | Esqueceu a senha ou precisa trocar — vai **presencialmente** ao gerente |
| 2 | 🔑 Gerente | Abre **Funcionários** → seleciona a pessoa → **Resetar senha** |
| 3 | Sistema | Mesmo padrão do cadastro: funcionário digita a **nova senha** (gerente não vê) |
| 4 | 🔑 Gerente | Confirma a operação |
| 5 | Sistema | Grava novo hash, invalida sessões anteriores (`session_version` / denylist — ver INF-03) |
| 6 | 👨‍🍳 ou 🛵 Funcionário | Faz login com a nova senha |

> **Cliente** usa autocadastro e recuperação por e-mail: [FLUX-HAPPY-02-recuperacao-senha](FLUX-HAPPY-02-recuperacao-senha.md). Reset de senha de **cliente** não é feito pelo gerente.

---

## Parte 3 — Desativar funcionário

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Quando a pessoa não opera mais, clica em **Desativar** |
| 2 | Sistema | Verifica se o motoboy tem pedido em `SAIU_PARA_ENTREGA` |
| 2a | *(sem entrega ativa)* | Encerra sessões, impede novo login, remove da fila se motoboy |
| 2b | *(com entrega ativa)* | **Bloqueia.** Resolver imprevisto antes: [FLUX-SAD-18-rota-gerente](../sadPaths/FLUX-SAD-18-rota-gerente.md) |

> Desativar ≠ encerrar turno do motoboy: [FLUX-HAPPY-16-turno](FLUX-HAPPY-16-turno.md).

---

> 📌 **Nota técnica resumida:**  
> `POST /store/staff` (gerente) — corpo em duas etapas ou um POST final após passo de senha enviado já hasheado **não**: a senha trafega só TLS e é hasheada no Spring. Reset: `POST /store/staff/{id}/password-reset`. Autocadastro: `POST /auth/register` restrito a role `CLIENTE`.  
> → [`../../docTecnica/INF-03-autenticacao.md`](../../docTecnica/INF-03-autenticacao.md) § 6.1–6.3
