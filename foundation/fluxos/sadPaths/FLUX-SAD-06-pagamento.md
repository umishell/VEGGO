# FLUX-SAD-06-pagamento
### Pagamento falha no checkout

**Gravidade:** 🔴 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente  
**Causa:** Simulação de pagamento falha (representa recusa de gateway real em produção).

```
Cliente finaliza → simulação recusa → rollback completo → nenhum pedido criado → tenta de novo
```

---

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Clica em **Finalizar Pedido** |
| 2 | Sistema | Chama o serviço de simulação de pagamento |
| 3 | Sistema | Simulação **recusa** a transação |
| 4 | Sistema | **Rollback completo** — nenhum pedido é criado, nenhum registro fica pela metade |
| 5 | 👤 Cliente | Vê a mensagem: *"Não foi possível processar o pagamento. Tente novamente."* |
| 6 | 👤 Cliente | Carrinho continua intacto — pode tentar finalizar de novo |

> **Impacto:** Alto, mas **seguro**. O "tudo ou nada" garante: sem pagamento confirmado, sem pedido criado. Nunca haverá pedido sem pagamento.

---

## Como acionar a falha na demonstração (banca / testes)

O serviço de pagamento simulado lê a propriedade `payment.simulation.mode` do ambiente Spring:

| Valor | Comportamento |
|-------|---------------|
| `ALWAYS_SUCCEED` | Toda tentativa retorna aprovado (padrão de produção) |
| `ALWAYS_FAIL` | Toda tentativa retorna recusado — **usar na demo do sad path** |
| `RANDOM` | Falha com ~10 % de probabilidade (útil para testes de carga) |

**Para a banca:** basta iniciar a stack com a variável de ambiente:

```
PAYMENT_SIMULATION_MODE=ALWAYS_FAIL
```

ou alterar `application-demo.properties`:

```properties
payment.simulation.mode=ALWAYS_FAIL
```

Após demonstrar o sad path, troca para `ALWAYS_SUCCEED` e o checkout volta ao normal — sem alterar código, sem reiniciar o banco.

> 📌 **Nota técnica resumida:**  
> `PaymentSimulationService` é um bean Spring com comportamento condicionado pela propriedade `payment.simulation.mode`. O valor vem de variável de ambiente ou `application.properties`. Nenhuma lógica de "cartão de teste" no frontend; a decisão é inteiramente no servidor. A propriedade é exposta no `application-demo.properties` do perfil de demonstração carregado no Compose de apresentação.
