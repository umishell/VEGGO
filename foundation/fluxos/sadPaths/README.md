# Menu — Sad Path

Problemas, erros e exceções · como o sistema reage.

| Gravidade | Significado |
|-----------|-------------|
| 🟡 Leve | Inconveniente, resolve rápido |
| 🟠 Médio | Interrompe o fluxo |
| 🔴 Grave | Afeta pedido ou pagamento |

| ID | Fluxo | Gravidade | Perfis |
|----|-------|-----------|--------|
| [FLUX-SAD-1](FLUX-SAD-1.md) | Item indisponível no cardápio | 🟡 | 👤 |
| [FLUX-SAD-2](FLUX-SAD-2.md) | Item do carrinho fica indisponível antes do checkout | 🟠 | 👤 🔑 |
| [FLUX-SAD-3](FLUX-SAD-3.md) | Item pausado no checkout — servidor recusa | 🔴 | 👤 🔑 |
| [FLUX-SAD-4](FLUX-SAD-4.md) | Cliente tenta cancelar após preparo iniciado | 🟠 | 👤 |
| [FLUX-SAD-5](FLUX-SAD-5.md) | Cliente tenta cancelar após motoboy sair | 🔴 | 👤 🛵 |
| [FLUX-SAD-6](FLUX-SAD-6.md) | Cozinha recusa o pedido | 🔴 | 👤 👨‍🍳 |
| [FLUX-SAD-7](FLUX-SAD-7.md) | Pagamento falha no checkout | 🔴 | 👤 |
| [FLUX-SAD-8](FLUX-SAD-8.md) | Dois motoboys — um perde a corrida | 🟠 | 🛵 🛵 |
| [FLUX-SAD-9](FLUX-SAD-9.md) | Motoboy desiste de pedido assumido | 🟠 | 🛵 👨‍🍳 |
| [FLUX-SAD-10](FLUX-SAD-10.md) | Cliente fecha app no meio do checkout | 🟡 | 👤 |
| [FLUX-SAD-11](FLUX-SAD-11.md) | Cliente sai do app com itens no carrinho | 🟡 | 👤 |
| [FLUX-SAD-12](FLUX-SAD-12.md) | Cliente sem endereço tenta finalizar | 🟡 | 👤 |
| [FLUX-SAD-13](FLUX-SAD-13.md) | CEP fora da zona de entrega | 🟠 | 👤 |
| [FLUX-SAD-14](FLUX-SAD-14.md) | Gerente cancela pedido já aceito | 🔴 | 🔑 👤 |
| [FLUX-SAD-15](FLUX-SAD-15.md) | Gerente desaloca motoboy em trânsito | 🔴 | 🔑 🛵 👤 |
| [FLUX-SAD-16](FLUX-SAD-16.md) | Nenhum motoboy ativo para assumir | 🔴 | 👨‍🍳 🔑 |
| [FLUX-SAD-17](FLUX-SAD-17.md) | Cliente tenta acessar pedido alheio | 🔴 | 👤 |
| [FLUX-SAD-18](FLUX-SAD-18.md) | Motoboy tenta pedido de outra loja | 🟠 | 🛵 |

---

## Resumo dos Cenários por Perfil

### 👤 Cliente
| ID | Situação |
|----|---------|
| FLUX-SAD-1 | Item some do cardápio — nem vê |
| FLUX-SAD-2 | Item do carrinho pausado — aviso e remoção |
| FLUX-SAD-3 | Checkout recusado por item pausado no servidor |
| FLUX-SAD-4 | Não consegue cancelar (preparo iniciado) |
| FLUX-SAD-5 | Não consegue cancelar (motoboy saiu) |
| FLUX-SAD-7 | Pagamento falha — nada cobrado |
| FLUX-SAD-10 | Fechou app no checkout — nada criado |
| FLUX-SAD-11 | Carrinho salvo se sessão ativa; perdido no logout |
| FLUX-SAD-12 | Sem endereço — sistema pede antes de finalizar |
| FLUX-SAD-13 | CEP fora da zona — pedido bloqueado |
| FLUX-SAD-17 | Acesso negado a pedido alheio |

### 👨‍🍳 Cozinha
| ID | Situação |
|----|---------|
| FLUX-SAD-6 | Recusa o pedido — cliente estornado |
| FLUX-SAD-16 | Pedido pronto mas sem motoboy disponível |

### 🛵 Motoboy
| ID | Situação |
|----|---------|
| FLUX-SAD-8 | Perde a corrida para outro motoboy |
| FLUX-SAD-9 | Desiste de pedido assumido — volta à fila |
| FLUX-SAD-15 | Desalocado pelo gerente |
| FLUX-SAD-18 | Tenta pedido de outra loja — bloqueado |

### 🔑 Gerente
| ID | Situação |
|----|---------|
| FLUX-SAD-6 | Cozinha (ou gerente) recusa pedido com estorno |
| FLUX-SAD-14 | Cancela pedido em andamento com estorno |
| FLUX-SAD-15 | Desaloca motoboy em trânsito |
| FLUX-SAD-16 | Gerencia ausência de motoboy disponível |

---

[Fluxos felizes](../happypath/README.md)
