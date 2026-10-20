# FLUX-SAD-18-rota-gerente
### Gerente desaloca motoboy incapacitado em trânsito (pedido refeito)

**Gravidade:** 🔴 · **Perfis:** 🔑 🛵 👤 👨‍🍳

[← Índice sad path](README.md)

---

**Perfis:** 🔑 Gerente · 🛵 Motoboy · 👤 Cliente · 👨‍🍳 Cozinha  
**Causa:** Motoboy teve acidente, perdeu a consciência ou perdeu o celular — **não consegue cancelar sozinho**; o gerente precisa intervir.  
**Veja também:** [FLUX-SAD-19-rota-motoboy](FLUX-SAD-19-rota-motoboy.md) — quando o próprio motoboy cancela a entrega em trânsito.

```
Motoboy saiu → fica incapacitado → gerente desaloca → lanche descartado
→ bebidas ficam pendentes de devolução → o mesmo pedido volta para a cozinha → cliente notificado com ❗
```

---

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Saiu para entrega — status `SAIU_PARA_ENTREGA`; lanche e bebidas estão com ele |
| 2 | 🔑 Gerente | Percebe ausência de contato, não resposta ou recebe alerta externo sobre o motoboy |
| 3 | 🔑 Gerente | Acessa o painel → **Pedidos em andamento** → localiza o pedido |
| 4 | 🔑 Gerente | Clica em **Desalocar motoboy** e seleciona o motivo: *"Imprevisto — motoboy incapacitado"* |
| 5 | 🔑 Gerente | Confirma a ação |
| 6 | Sistema | O **mesmo** pedido volta para `RECEBIDO`. Auditoria grava `IMPREVISTO_ENTREGA`. `incident_alert` liga. O cliente não pode mais cancelar este pedido |
| 7 | 🛵 Motoboy | Quando recuperar o acesso ao app recebe aviso: *"Você foi removido desta entrega."* |

---

**Protocolo de imprevisto — destino dos itens e do pedido:**

| Item | O que acontece |
|------|----------------|
| 🍔 Lanche | **Descartado** — estava com o motoboy. A auditoria registra a perda. Não há tela de estoque |
| 🥤 Bebidas | Pendência de devolução do motoboy. Sem módulo de estoque |
| 🔄 Pedido | Continua o **mesmo** número. Volta para a fila de espera da cozinha |
| 👨‍🍳 Cozinha | Vê os itens para refazer, **sem nome do cliente** |
| 🛵 Fila | Quando ficar `PRONTO` de novo, outro motoboy pode assumir |

---

**Notificação ao cliente:**

| # | Quem | O que acontece |
|---|------|----------------|
| 8 | Sistema | Ativa a flag `incident_alert` no pedido → ícone **🔴 ❗** aparece no card simplificado do pedido |
| 9 | 👤 Cliente | Vê o ícone vermelho de exclamação na tela principal — **sem texto, apenas o ícone** |
| 10 | 👤 Cliente | Clica no card simplificado do pedido |
| 11 | Sistema | Abre a tela completa do pedido com **caixa de aviso destacada no topo** |
| 12 | 👤 Cliente | Lê o aviso: *"Houve um imprevisto com a entrega do seu pedido. Seu lanche está sendo refeito e logo chegará. Pedimos desculpas pelo inconveniente! 🙏"* |
| 13 | 👤 Cliente | Vê o botão **💬 Fale Conosco** dentro da caixa de aviso |
| 14 | 👤 Cliente | (Opcional) Clica em **Fale Conosco** → redirecionado para o **WhatsApp da loja** |
| 15 | 👤 Cliente | Envia mensagem pelo próprio WhatsApp — atendimento fora do app |

**Veja também:**  
→ [FLUX-SAD-21-aviso-cliente](FLUX-SAD-21-aviso-cliente.md) — experiência completa do cliente ao receber a notificação e acessar o Fale Conosco  
→ [FLUX-HAPPY-17-whatsapp](../happypaths/FLUX-HAPPY-17-whatsapp.md) — fluxo do Fale Conosco via WhatsApp

---

> **Impacto:** Muito alto. Atraso significativo, custo de produto (lanche descartado) e retrabalho da cozinha. Toda ação fica registrada na auditoria com responsável e horário.

---

> 📌 **Nota técnica resumida:**  
> O mesmo `order_id` volta a `RECEBIDO`, com `incident_alert` e cancelamento do cliente bloqueado. `IMPREVISTO_ENTREGA` é o motivo na auditoria, não um segundo pedido. O Fale Conosco é um link `wa.me/[número]`.  
> → Detalhes completos: [`../../docTecnica/ORD-01-estados-pedido.md`](../../docTecnica/ORD-01-estados-pedido.md) · [`../../docTecnica/DB-01-canal-atendimento.md`](../../docTecnica/DB-01-canal-atendimento.md)
