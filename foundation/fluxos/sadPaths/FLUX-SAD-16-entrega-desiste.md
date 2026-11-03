# FLUX-SAD-16-entrega-desiste
### Motoboy aceita no balcão e percebe impedimento

**Gravidade:** 🔴 · **Perfis:** 🛵 🔑 👤 👨‍🍳

[← Índice sad path](README.md)

---

**Perfis:** 🛵 Motoboy · 🔑 Gerente · 👤 Cliente · 👨‍🍳 Cozinha  
**Causa:** Motoboy assumiu o pedido (`SAIU_PARA_ENTREGA`) mas, ainda na loja ou logo depois de sair, percebe que não consegue prosseguir com a entrega (pneu furado no pátio, passou mal, moto que não liga etc.).  
**Diferença de SAD-18/19:** o motoboy ainda tem os itens e pode devolver; detectado **antes de se afastar**, mas o estado já é `SAIU_PARA_ENTREGA` — **Aceitar pedido** (`POST /take`) aloca e inicia entrega num único passo.

> Se o motoboy já se afastou da loja e o imprevisto ocorreu em rota: [FLUX-SAD-19-rota-motoboy](FLUX-SAD-19-rota-motoboy.md) (motoboy reporta) ou [FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md) (gerente age diretamente).

```
Motoboy Aceitar pedido → percebe impedimento → reporta ao gerente
→ gerente aciona protocolo de imprevisto → pedido volta para KDS com prioridade
→ itens são devolvidos à loja (motoboy ainda está por perto)
```

---

## Cenário A — Motoboy reporta pelo app

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Clicou **Aceitar pedido** — pedido está `SAIU_PARA_ENTREGA` |
| 2 | 🛵 Motoboy | Percebe que não pode sair (problema com a moto, mal súbito etc.), ainda está na loja ou no pátio |
| 3 | 🛵 Motoboy | Abre o app e clica em **Reportar imprevisto** |
| 4 | 🛵 Motoboy | Seleciona o motivo (ex: *"Problema com a moto"*) e confirma |
| 5 | Sistema | Registra evento `INCIDENT_REPORTED` e notifica gerente via `kitchen:{storeId}` |
| 6 | 🔑 Gerente | Vê alerta no painel: *"🚨 Imprevisto reportado — Pedido #XXXX · Motoboy [nome] · Motivo: [motivo]"* |
| 7 | 🔑 Gerente | Confirma e aciona o refazimento (igual ao FLUX-SAD-19) |
| 8 | Sistema | Pedido volta a `RECEBIDO` com `priority = true`; `motoboy_id = null` |
| 9 | 🛵 Motoboy | Devolve ou descarta itens **presencialmente** com o gerente — sem registro no app |

> 💡 Como o motoboy ainda está na loja, alimentos **podem ser reaproveitados** se em boas condições — decisão do gerente, fora do sistema. O app só recoloca o pedido na fila com prioridade e notifica o cliente.

---

## Cenário B — Gerente percebe antes do motoboy reportar

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🛵 Motoboy | Clicou **Aceitar pedido** mas não consegue sair da loja |
| 2 | 🔑 Gerente | Percebe a situação presencialmente ou por contato direto |
| 3 | 🔑 Gerente | Abre o pedido no painel → clica em **Acionar imprevisto** |
| 4 | 🔑 Gerente | Preenche o motivo e confirma |
| 5 | Sistema | Pedido volta a `RECEBIDO` com `priority = true`; `motoboy_id = null` |
| 6 | 🛵 Outro motoboy | Quando o pedido ficar `PRONTO` de novo, **Aceitar pedido** normalmente |

---

**Itens físicos (motoboy ainda na loja):**

Devolução ou descarte é **presencial** com o gerente. O app não diferencia alimento vs bebida nem exige confirmação no painel — só o refazimento na fila com ⚡.

---

**Veja também:**  
→ [FLUX-SAD-18-rota-gerente](FLUX-SAD-18-rota-gerente.md) — motoboy incapacitado em trânsito, gerente desaloca  
→ [FLUX-SAD-19-rota-motoboy](FLUX-SAD-19-rota-motoboy.md) — motoboy reporta imprevisto já em rota  
→ [FLUX-SAD-20-sem-motoboy](FLUX-SAD-20-sem-motoboy.md) — nenhum motoboy disponível para o novo refazimento

---

> **Impacto:** Alto — o pedido volta para a fila com prioridade e o cliente é notificado. Como o motoboy ainda está na loja, os itens têm chance maior de aproveitamento, mas o atraso total pode ser significativo.

---

> 📌 **Nota técnica resumida:**  
> Após `SAIU_PARA_ENTREGA`, o motoboy usa `POST /orders/{id}/incident/report` (SAD-19) ou o gerente usa `POST /orders/{id}/incident` (SAD-18). O estado resultante é idêntico: `RECEBIDO` com `priority = true`, `cancelamento_bloqueado = true`, `motoboy_id = null`. A distinção cenário A/B é operacional — o sistema não diferencia "impedimento na loja" de "impedimento em rota" na máquina de estados.  
> → [`../../docTecnica/ORD-01-estados-pedido.md`](../../docTecnica/ORD-01-estados-pedido.md)
