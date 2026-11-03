# FLUX-HAPPY-02-recuperacao-senha
### Cliente recupera senha por e-mail

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente  
**Veja também:** [FLUX-HAPPY-02-login](FLUX-HAPPY-02-login.md) · [FLUX-HAPPY-13-usuarios](FLUX-HAPPY-13-usuarios.md) — reset de **funcionário** é só com gerente presente, não usa e-mail.

```
Esqueci a senha → e-mail com código → confirma código → nova senha (2×) → login
```

---

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Na tela de login, toca **Esqueci minha senha** |
| 2 | 👤 Cliente | Informa o **e-mail** cadastrado |
| 3 | Sistema | Envia e-mail com **código de confirmação** (TTL curto; uso único) |
| 4 | 👤 Cliente | Abre o app na tela **Informe o código** e digita o código recebido |
| 5 | Sistema | Valida o código; se OK, libera a etapa de nova senha |
| 6 | 👤 Cliente | Digita a **nova senha** e **confirma a nova senha** (dois campos) |
| 7 | Sistema | Grava hash Argon2id, invalida o código e incrementa versão de sessão |
| 8 | 👤 Cliente | É redirecionado ao **login** (ou entra automaticamente, conforme UX do app) |

> Resposta genérica se o e-mail não existir (“Se houver conta, enviamos instruções”) — evita enumerar usuários.

> 📌 **API (MVP):** `POST /auth/password-reset/request` · `POST /auth/password-reset/confirm-code` · `POST /auth/password-reset/complete` — detalhe em [`../../docTecnica/INF-03-autenticacao.md`](../../docTecnica/INF-03-autenticacao.md) § 6.4.
