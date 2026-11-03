# Menu — Happy Path

Hamburgueria delivery · fluxos **sem problemas**.

O número segue a ordem do acontecimento. A palavra depois do número é o tema. Quando um sad path trata do mesmo assunto, ele usa a mesma palavra (`item`, `cancela`, `disputa`, `frete`) e fica em `sadPaths`.

| Ícone | Perfil |
|-------|--------|
| 👤 | Cliente |
| 👨‍🍳 | Cozinha |
| 🛵 | Motoboy |
| 🔑 | Gerente |

| ID | Fluxo | Perfis |
|----|-------|--------|
| [FLUX-HAPPY-01-cadastro](FLUX-HAPPY-01-cadastro.md) | Cadastro e primeiro acesso | 👤 |
| [FLUX-HAPPY-02-login](FLUX-HAPPY-02-login.md) | Login e navegação no cardápio | 👤 |
| [FLUX-HAPPY-02-recuperacao-senha](FLUX-HAPPY-02-recuperacao-senha.md) | Cliente recupera senha por e-mail (código) | 👤 |
| [FLUX-HAPPY-03-carrinho](FLUX-HAPPY-03-carrinho.md) | Montagem do carrinho e saída do app (carrinho salvo) | 👤 |
| [FLUX-HAPPY-04-endereco](FLUX-HAPPY-04-endereco.md) | Cliente gerencia endereços salvos | 👤 |
| [FLUX-HAPPY-21-checkout-pagamento](FLUX-HAPPY-21-checkout-pagamento.md) | Checkout — endereço, taxa de entrega fixa, pagamento mock | 👤 |
| [FLUX-HAPPY-22-cardapio-categorias](FLUX-HAPPY-22-cardapio-categorias.md) | Cardápio por categorias e quantidades | 👤 |
| [FLUX-HAPPY-05-venda](FLUX-HAPPY-05-venda.md) | Venda completa — do carrinho à entrega na porta | 👤 👨‍🍳 🛵 |
| [FLUX-HAPPY-06-timeline](FLUX-HAPPY-06-timeline.md) | Cliente acompanha pedido em tempo real | 👤 |
| [FLUX-HAPPY-07-cancela](FLUX-HAPPY-07-cancela.md) | Cliente cancela pedido antes do preparo | 👤 |
| [FLUX-HAPPY-08-historico](FLUX-HAPPY-08-historico.md) | Cliente consulta histórico de pedidos anteriores | 👤 |
| [FLUX-HAPPY-09-item-novo](FLUX-HAPPY-09-item-novo.md) | Gerente cadastra novo produto no cardápio | 🔑 |
| [FLUX-HAPPY-10-item-gerente](FLUX-HAPPY-10-item-gerente.md) | Gerente pausa, bloqueia ou apaga item do cardápio | 🔑 |
| [FLUX-HAPPY-11-item-cozinha](FLUX-HAPPY-11-item-cozinha.md) | KDS — lista, seleção, vários preparos, timer | 👨‍🍳 |
| [FLUX-HAPPY-12-horario](FLUX-HAPPY-12-horario.md) | Gerente configura loja, horários, fechamentos e **taxa de entrega fixa** | 🔑 |
| [FLUX-HAPPY-13-usuarios](FLUX-HAPPY-13-usuarios.md) | Funcionários — cadastro presencial, reset de senha, desativar | 🔑 |
| [FLUX-HAPPY-15-disputa](FLUX-HAPPY-15-disputa.md) | Dois motoboys disputam o mesmo pedido — um assume | 🛵 🛵 |
| [FLUX-HAPPY-16-turno](FLUX-HAPPY-16-turno.md) | Motoboy encerra turno (fica inativo) | 🛵 |
| [FLUX-HAPPY-20-motoboy-app](FLUX-HAPPY-20-motoboy-app.md) | Motoboy — prontos, aceite no balcão, acompanhamento | 🛵 |
| [FLUX-HAPPY-17-whatsapp](FLUX-HAPPY-17-whatsapp.md) | **nos contacte** (WhatsApp loja) + timeline (loja e motoboy) | 👤 |
| [FLUX-HAPPY-18-painel](FLUX-HAPPY-18-painel.md) | Gerente acompanha pedidos do dia em tempo real | 🔑 |
| [FLUX-HAPPY-19-stats](FLUX-HAPPY-19-stats.md) | Gerente acessa estatísticas e telemetria da operação | 🔑 |

---

Índice na raiz (legado): [`cenarios_happyPath.md`](../../cenarios_happyPath.md).

[Fluxos com problemas](../sadPaths/README.md)
