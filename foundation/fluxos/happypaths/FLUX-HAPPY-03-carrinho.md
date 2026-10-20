# FLUX-HAPPY-03-carrinho
### Montagem do carrinho e saída do app (carrinho salvo)

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente

```
Cliente adiciona itens → fecha o app → volta → carrinho ainda está lá
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | No cardápio, toca em **X-Burger** e clica **Adicionar** |
| 2 | 👤 Cliente | Adiciona também uma **Coca-Cola** |
| 3 | 👤 Cliente | Vê o ícone do carrinho com "2 itens" no canto da tela |
| 4 | 👤 Cliente | **Fecha o app** (vai atender uma ligação, por exemplo) |
| 5 | 👤 Cliente | Reabre o app minutos depois |
| 6 | Sistema | Restaura o carrinho exatamente como estava |
| 7 | 👤 Cliente | Vê X-Burger e Coca-Cola ainda no carrinho, prontos para finalizar |

> 💡 O carrinho fica salvo enquanto a sessão do cliente está ativa. Se o cliente fizer logout, o carrinho é descartado.
