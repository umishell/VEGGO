# FLUX-HAPPY-09-item-novo
### Gerente cadastra novo produto no cardápio

**Perfis:** 🔑

[← Índice happy path](README.md)

---

**Perfis:** 🔑 Gerente

```
Chegou um novo hambúrguer especial → gerente cria o produto → aparece no cardápio
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Acessa o painel e clica em **Cardápio → Novo produto** |
| 2 | 🔑 Gerente | Preenche: nome, descrição, preço, **categoria** ([DB-04](../../docTecnica/DB-04-catalogo-checkout-cliente.md) § 1.1), foto |
| 3 | 🔑 Gerente | Marca como **"Disponível"** e clica em **Salvar** |
| 4 | Sistema | `POST /store/products` (INF-02 § 11.3) — cria linha em `products` com `availability = DISPONIVEL` ([DB-02 § 1](../../docTecnica/DB-02-disponibilidade-itens.md)) |
| 5 | 👤 Cliente | `GET /menu` — vê o novo produto na categoria correta |

> 📌 Colunas `products`: [DB-02 § 1](../../docTecnica/DB-02-disponibilidade-itens.md) · API: [INF-02 § 11](../../docTecnica/INF-02-http-vs-websocket.md) · Tela gerente: [INF-04 § 4](../../docTecnica/INF-04-ui-operacao-cliente.md)
