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
| 2 | 🔑 Gerente | Preenche: nome, descrição, preço, categoria, foto e **tipo do produto** (`Alimento preparado` ou `Bebida em embalagem fechada`) |
| 3 | 🔑 Gerente | Marca como **"Disponível"** e clica em **Salvar** |
| 4 | Sistema | O produto é criado e aparece imediatamente no cardápio do cliente |
| 5 | 👤 Cliente | Ao abrir o app, já vê o novo produto listado |

> 📌 O campo **tipo do produto** (`item_type`) é obrigatório e define o destino do item caso ocorra um imprevisto em rota: alimentos preparados são descartados; bebidas em embalagem fechada ficam como pendência de devolução do motoboy. Pode ser alterado na edição do produto a qualquer momento.  
> → Regra técnica: [`../../docTecnica/DB-02-disponibilidade-itens.md`](../../docTecnica/DB-02-disponibilidade-itens.md)  
> → Protocolo de imprevisto: [`../../docTecnica/ORD-01-estados-pedido.md`](../../docTecnica/ORD-01-estados-pedido.md)
