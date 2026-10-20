# FLUX-HAPPY-14-frete
### Gerente cadastra zonas de entrega e frete

**Perfis:** 🔑

[← Índice happy path](README.md)

---

**Perfis:** 🔑 Gerente · 👤 Cliente

```
Gerente define faixa de CEP e taxa → checkout do cliente usa essa taxa
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 🔑 Gerente | Abre **Zonas de entrega** |
| 2 | 🔑 Gerente | Cria uma zona: nome, CEP inicial, CEP final e valor do frete |
| 3 | Sistema | Salva a faixa |
| 4 | 👤 Cliente | No checkout, informa um CEP dentro da faixa |
| 5 | Sistema | Mostra o frete daquela zona e grava o valor no pedido |
| 6 | 🔑 Gerente | Altera o valor da taxa |
| 7 | Sistema | Pedidos novos usam a taxa nova. Pedido já criado mantém a taxa que foi cobrada |

CEP sem faixa: [FLUX-SAD-08-frete-fora](../sadPaths/FLUX-SAD-08-frete-fora.md).

---

> 📌 **Nota técnica resumida:**  
> Tabela `delivery_zones`. A taxa copiada no pedido não muda se a zona for editada depois.  
> → [`../../docTecnica/DB-02-disponibilidade-itens.md`](../../docTecnica/DB-02-disponibilidade-itens.md)
