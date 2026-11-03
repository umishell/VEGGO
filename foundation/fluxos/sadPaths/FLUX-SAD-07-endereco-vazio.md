# FLUX-SAD-07-endereco-vazio
### Cliente sem endereço tenta finalizar

**Gravidade:** 🟡 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente  
**Causa:** Primeiro pedido do cliente — nenhum endereço salvo ainda.

```
Cliente tenta finalizar → sistema pede endereço → cliente cadastra
→ taxa de entrega fixa no resumo → finaliza → endereço salvo automaticamente para os próximos pedidos
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Clica em **Finalizar Pedido** |
| 2 | Sistema | Detecta que não há endereço de entrega cadastrado |
| 3 | Sistema | Exibe formulário: **"Informe seu endereço de entrega"** |
| 4 | 👤 Cliente | Preenche rua, número, bairro e CEP (obrigatórios) + complemento e ponto de referência (opcionais, mas ajudam o motoboy a encontrar a casa) |
| 5 | Sistema | Mostra subtotal, **taxa de entrega fixa** (`fixed_delivery_fee_cents`) e total |
| 6 | 👤 Cliente | Confirma e finaliza normalmente |
| 7 | Sistema | **Salva o endereço automaticamente** no perfil do cliente |
| 8 | 👤 Cliente | Nos próximos pedidos, o endereço já aparece pré-selecionado na tela de finalização |

---

**Comportamento em pedidos futuros — endereço novo digitado manualmente:**

| # | Quem | O que acontece |
|---|------|----------------|
| A | 👤 Cliente | Já tem endereço salvo, mas digita um **novo endereço manualmente** na tela de finalização |
| B | Sistema | Utiliza o novo endereço para este pedido |
| C | Sistema | **Salva o novo endereço** na lista de endereços do cliente automaticamente |
| D | 👤 Cliente | Nos próximos pedidos, pode escolher entre todos os endereços salvos |

> 💡 Todo endereço utilizado para finalizar um pedido é salvo. O cliente acumula sua lista de endereços sem precisar acessar as configurações do perfil.

> **Impacto:** Mínimo. É uma etapa extra no primeiro pedido, não um erro. A partir do segundo pedido, o fluxo é mais ágil. Editar, apagar e escolher o padrão: [FLUX-HAPPY-04-endereco](../happypaths/FLUX-HAPPY-04-endereco.md).
