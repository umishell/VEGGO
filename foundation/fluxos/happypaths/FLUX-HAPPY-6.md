# FLUX-HAPPY-6
### Gerente pausa item do cardápio durante o dia

**Perfis:** 🔑

[← Índice happy path](README.md)

---

**Perfis:** 🔑 Gerente

```
Gerente percebe que acabou o pão brioche → pausa o X-Burger → item some do cardápio
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Acessa o painel da loja e vai em **Cardápio** |
| 2 | 🔑 Gerente | Localiza o **X-Burger** na lista |
| 3 | 🔑 Gerente | Clica no toggle **"Disponível"** → muda para **"Pausado"** |
| 4 | Sistema | Remove imediatamente o X-Burger do cardápio visível ao cliente |
| 5 | 🔑 Gerente | Vê o item marcado como pausado na lista interna (ainda existe, só não aparece para o cliente) |
| 6 | 🔑 Gerente | Quando o pão chegar, volta e ativa de novo em um clique |

> 💡 O item pausado **não é excluído** — só fica invisível para quem compra.
