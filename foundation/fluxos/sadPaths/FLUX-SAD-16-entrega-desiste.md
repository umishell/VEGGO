# FLUX-SAD-16-entrega-desiste
### Motoboy desiste de pedido assumido antes de sair da loja

**Gravidade:** 🟠 · **Perfis:** 🛵 👤 🔑

[← Índice sad path](README.md)

---

**Perfis:** 🛵 Motoboy · 👤 Cliente · 🔑 Gerente  
**Causa:** Motoboy assumiu o pedido mas teve um imprevisto **antes de sair da loja** — emergência pessoal, problema com a moto no pátio, desistência.  
**Status no momento:** `ALOCADO` — o motoboy ainda **não saiu**; o lanche está **fisicamente na loja**.

> ⚠️ Este fluxo é exclusivo para imprevistos **antes da saída**.  
> Se o motoboy já saiu com o pedido, veja: [FLUX-SAD-19-rota-motoboy](FLUX-SAD-19-rota-motoboy.md) (auto-cancelamento em trânsito) ou [FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md) (gerente desaloca motoboy incapacitado em trânsito).

```
Motoboy assume → ainda na loja → desiste → pedido volta à fila
→ lanche continua na loja → próximo motoboy assume e pega o mesmo lanche
```

---

## Cenário A — Motoboy cancela sozinho

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Assumiu o pedido — status `ALOCADO`; ainda está na loja ou a caminho dela |
| 2 | 🛵 Motoboy | Surge um imprevisto (emergência pessoal, moto com problema no pátio, etc.) |
| 3 | 🛵 Motoboy | Abre o app e clica em **Desistir do pedido** |
| 4 | Sistema | Libera o pedido — volta ao status **`PRONTO`** na fila de motoboys |
| 5 | 👤 Cliente | Timeline **não avança** — permanece em *"Motoboy a caminho para retirar"* ou *"Pronto para retirada"* |
| 6 | 🛵 Outro motoboy | Vê o pedido na fila e clica em **Assumir** |
| 7 | 🛵 Outro motoboy | Chega na loja e pega o **mesmo lanche** — já embalado, pronto para sair |
| 8 | 👤 Cliente | Timeline avança normalmente quando o novo motoboy confirma a saída |

**O lanche:**

| Item | O que acontece |
|------|----------------|
| 🍔 Lanche | **Permanece na loja** — entregue ao próximo motoboy sem refazimento |
| 🥤 Bebidas | **Permanecem na loja** — entregues ao próximo motoboy junto com o lanche |

> Nenhum produto é desperdiçado. Nenhum pedido de refazimento é gerado.

---

## Cenário B — Gerente desaloca o motoboy (antes da saída)

Usado quando o gerente precisa intervir — ex.: motoboy ficou sem resposta dentro da loja, ou situação operacional exige troca.

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Assumiu o pedido — status `ALOCADO`; ainda está na loja |
| 2 | 🔑 Gerente | Acessa o painel → **Pedidos em andamento** → localiza o pedido |
| 3 | 🔑 Gerente | Clica em **Desalocar motoboy** e confirma com motivo |
| 4 | Sistema | Libera o pedido — volta ao status **`PRONTO`** na fila |
| 5 | 🛵 Motoboy | Recebe aviso no app: *"Você foi removido deste pedido."* |
| 6 | 👤 Cliente | Timeline não avança — permanece em *"Pronto para retirada"* |
| 7 | 🛵 Outro motoboy | Vê o pedido na fila, assume e pega o **mesmo lanche** na loja |
| 8 | 👤 Cliente | Timeline avança normalmente |

> **Diferença-chave do Cenário B vs. FLUX-SAD-18-rota-gerente:** aqui o motoboy **ainda não saiu** — o lanche está na loja, não há descarte, não há refazimento e **não é acionado** o ícone ❗ para o cliente (apenas um atraso normal de redistribuição).

---

**Veja também:**  
→ [FLUX-SAD-19-rota-motoboy](FLUX-SAD-19-rota-motoboy.md) — motoboy já saiu e cancela em trânsito (lanche descartado, pedido refeito)  
→ [FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md) — motoboy incapacitado em trânsito, gerente desaloca (lanche descartado, pedido refeito)

---

> **Impacto:** Médio. Atraso na entrega, mas **sem desperdício de produto** e sem retrabalho da cozinha. O cliente percebe apenas uma demora extra na chegada do motoboy.
