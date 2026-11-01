# FLUX-HAPPY-02-login
### Login e navegação no cardápio

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente

```
Cliente já tem conta → faz login → navega pelo cardápio
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Abre o app, vê a tela de login |
| 2 | 👤 Cliente | Digita e-mail e senha, clica **Entrar** |
| 3 | Sistema | Verifica os dados e libera o acesso |
| 4 | 👤 Cliente | Vê o cardápio organizado por categorias (ex.: Hambúrgueres, Bebidas, Sobremesas) |
| 5 | 👤 Cliente | Toca em um produto para ver foto, descrição e preço |
| 6 | 👤 Cliente | Navega à vontade pelas categorias |

> 💡 Itens `PAUSADO_DIA` e `BLOQUEADO` **aparecem cinza** no cardápio — o cliente vê que o produto existe mas não pode pedir. Itens `ARQUIVADO` não aparecem (ocultos administrativamente pelo gerente). O cliente só adiciona ao carrinho itens `DISPONIVEL`.
