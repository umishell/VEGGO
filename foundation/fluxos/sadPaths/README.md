# Menu — Sad Path

Problemas, erros e exceções · como o sistema reage.

O número segue a mesma história do happy path: cardápio, carrinho, pagamento, endereço, cancelamento, loja e entrega. A palavra depois do número é o tema. Quando existe um happy path do mesmo assunto, a palavra se repete (`item`, `cancela`, `disputa`, `frete`, `endereco`).

| Gravidade | Significado |
|-----------|-------------|
| 🟡 Leve | Inconveniente, resolve rápido |
| 🟠 Médio | Interrompe o fluxo |
| 🔴 Grave | Afeta pedido ou pagamento |

| ID | Fluxo | Gravidade | Perfis |
|----|-------|-----------|--------|
| [FLUX-SAD-01-item-cinza](FLUX-SAD-01-item-cinza.md) | Item indisponível no cardápio — fica cinza | 🟡 | 👤 |
| [FLUX-SAD-02-item-carrinho](FLUX-SAD-02-item-carrinho.md) | Item do carrinho fica indisponível antes do checkout | 🟠 | 👤 🔑 |
| [FLUX-SAD-03-item-checkout](FLUX-SAD-03-item-checkout.md) | Item pausado no checkout — servidor recusa | 🔴 | 👤 🔑 |
| [FLUX-SAD-04-carrinho-sessao](FLUX-SAD-04-carrinho-sessao.md) | Cliente sai do app com itens no carrinho | 🟡 | 👤 |
| [FLUX-SAD-05-checkout-fecha](FLUX-SAD-05-checkout-fecha.md) | Cliente fecha app no meio do checkout | 🟡 | 👤 |
| [FLUX-SAD-06-pagamento](FLUX-SAD-06-pagamento.md) | Pagamento falha no checkout | 🔴 | 👤 |
| [FLUX-SAD-07-endereco-vazio](FLUX-SAD-07-endereco-vazio.md) | Cliente sem endereço tenta finalizar | 🟡 | 👤 |
| [FLUX-SAD-08-frete-fora](FLUX-SAD-08-frete-fora.md) | CEP fora da zona de entrega | 🟠 | 👤 |
| [FLUX-SAD-09-cancela-tarde](FLUX-SAD-09-cancela-tarde.md) | Cliente tenta cancelar após preparo iniciado | 🟠 | 👤 |
| [FLUX-SAD-10-cancela-saiu](FLUX-SAD-10-cancela-saiu.md) | Cliente tenta cancelar após motoboy sair | 🔴 | 👤 🛵 |
| [FLUX-SAD-11-cancela-loja](FLUX-SAD-11-cancela-loja.md) | Gerente cancela um pedido específico | 🔴 | 🔑 👤 |
| [FLUX-SAD-12-cozinha-fila](FLUX-SAD-12-cozinha-fila.md) | Cozinha demora a aceitar o pedido | 🟠 | 👨‍🍳 🔑 👤 |
| [FLUX-SAD-13-loja-emergencia](FLUX-SAD-13-loja-emergencia.md) | Gerente fecha a loja em emergência (fechamento antecipado) | 🔴 | 🔑 👤 👨‍🍳 |
| [FLUX-SAD-14-loja-fechada](FLUX-SAD-14-loja-fechada.md) | Loja fechada — banner, visual cinza e countdown | 🟠 | 👤 |
| [FLUX-SAD-15-disputa](FLUX-SAD-15-disputa.md) | Dois motoboys — um perde a corrida | 🟠 | 🛵 🛵 |
| [FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md) | Gerente aciona imprevisto em rota — pedido refaz com prioridade | 🔴 | 🔑 🛵 👤 👨‍🍳 |
| [FLUX-SAD-19-rota-motoboy](FLUX-SAD-19-rota-motoboy.md) | Motoboy reporta imprevisto em rota — gerente confirma e aciona refazimento | 🔴 | 🛵 🔑 👤 👨‍🍳 |
| [FLUX-SAD-20-sem-motoboy](FLUX-SAD-20-sem-motoboy.md) | Nenhum motoboy ativo para assumir | 🔴 | 👨‍🍳 🔑 |
| [FLUX-SAD-21-aviso-cliente](FLUX-SAD-21-aviso-cliente.md) | Cliente notificado de imprevisto — aviso na timeline e Fale Conosco | 🟠 | 👤 |
| [FLUX-SAD-22-acesso-pedido](FLUX-SAD-22-acesso-pedido.md) | Cliente tenta acessar pedido alheio | 🔴 | 👤 |
| [FLUX-SAD-23-acesso-loja](FLUX-SAD-23-acesso-loja.md) | Motoboy tenta pedido de outra loja | 🟠 | 🛵 |

---

## Resumo dos Cenários por Perfil

### 👤 Cliente

| ID | Situação |
|----|---------|
| FLUX-SAD-01-item-cinza | Item fica cinza no cardápio — não consegue pedir |
| FLUX-SAD-02-item-carrinho | Item do carrinho pausado — aviso e remoção |
| FLUX-SAD-03-item-checkout | Checkout recusado por item pausado no servidor |
| FLUX-SAD-04-carrinho-sessao | Carrinho salvo se sessão ativa; perdido no logout |
| FLUX-SAD-05-checkout-fecha | Fechou app no checkout — nada criado |
| FLUX-SAD-06-pagamento | Pagamento falha — nada cobrado |
| FLUX-SAD-07-endereco-vazio | Sem endereço — sistema pede antes de finalizar |
| FLUX-SAD-08-frete-fora | CEP fora da zona — pedido bloqueado |
| FLUX-SAD-09-cancela-tarde | Não consegue cancelar (preparo iniciado) |
| FLUX-SAD-10-cancela-saiu | Não consegue cancelar (motoboy saiu) |
| FLUX-SAD-14-loja-fechada | Loja fechada — carrinho bloqueado, visual cinza e countdown |
| FLUX-SAD-21-aviso-cliente | Vê pedido voltar a RECEBIDO com ⚡ prioridade e mensagem do gerente |
| FLUX-SAD-22-acesso-pedido | Acesso negado a pedido alheio |

### 👨‍🍳 Cozinha

| ID | Situação |
|----|---------|
| FLUX-SAD-12-cozinha-fila | Demora a aceitar — pedido permanece na fila, sem recusa |
| FLUX-SAD-13-loja-emergencia | Alerta gerente para fechar a loja — pedidos em andamento continuam |
| FLUX-SAD-20-sem-motoboy | Pedido pronto mas sem motoboy disponível |

### 🛵 Motoboy

| ID | Situação |
|----|---------|
| FLUX-SAD-15-disputa | Perde a corrida para outro motoboy — o vencedor vai direto a `SAIU_PARA_ENTREGA` |
| FLUX-SAD-18-rota-gerente | Gerente detecta imprevisto e aciona refazimento com prioridade |
| FLUX-SAD-19-rota-motoboy | Reporta imprevisto pelo app — gerente confirma e aciona refazimento |
| FLUX-SAD-23-acesso-loja | Tenta pedido de outra loja — bloqueado |

### 🔑 Gerente

| ID | Situação |
|----|---------|
| FLUX-SAD-11-cancela-loja | Cancela um pedido específico, com estorno — não é o fechamento da loja |
| FLUX-SAD-13-loja-emergencia | Fecha a loja em emergência com motivo — novos pedidos bloqueados |
| FLUX-SAD-18-rota-gerente | Aciona imprevisto em rota — pedido volta ao topo da fila com prioridade |
| FLUX-SAD-19-rota-motoboy | Recebe alerta do motoboy, valida e aciona refazimento com prioridade |
| FLUX-SAD-20-sem-motoboy | Gerencia ausência de motoboy disponível |
| FLUX-SAD-21-aviso-cliente | Cliente notificado de imprevisto — mensagem na timeline do pedido |

---

[Fluxos felizes](../happypaths/README.md)
