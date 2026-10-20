# FLUX-SAD-08-frete-fora
### CEP fora da zona de entrega

**Gravidade:** 🟠 · **Perfis:** 👤

[← Índice sad path](README.md)

---

**Perfis:** 👤 Cliente
**Causa:** Endereço do cliente está em bairro que a hamburgueria não atende.

```
Cliente informa endereço → sistema detecta fora da área → bloqueia o pedido
```

**Passo a passo:**

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Informa o CEP no checkout |
| 2 | Sistema | Verifica se o CEP pertence a alguma zona de entrega cadastrada |
| 3 | Sistema | **CEP não está em nenhuma zona de entrega** |
| 4 | 👤 Cliente | Vê: **"Infelizmente não entregamos nessa região ainda."** |
| 5 | 👤 Cliente | Botão "Finalizar" fica desabilitado — não pode prosseguir |
| 6 | 👤 Cliente | Pode tentar um endereço diferente (ex.: trabalho, casa de parente) |

> **Impacto:** Médio. Venda perdida, mas o sistema é honesto — não cria pedido para endereço que não pode atender.
