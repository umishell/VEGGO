# FLUX-HAPPY-04-endereco
### Cliente gerencia endereços salvos

**Perfis:** 👤

[← Índice happy path](README.md)

---

**Perfis:** 👤 Cliente  
**Veja também:** [FLUX-SAD-07-endereco-vazio](../sadPaths/FLUX-SAD-07-endereco-vazio.md) — o primeiro endereço nasce no checkout e fica salvo.

```
Cliente abre Endereços → escolhe o padrão, edita, apaga ou adiciona
→ no próximo checkout a lista já está lá
```

---

## Campos de um endereço

| Campo | Obrigatório | Exemplo | Para que serve |
|-------|-------------|---------|----------------|
| **CEP** | Sim | `80210-170` | Determina zona de frete |
| **Rua** | Sim | `Rua das Flores` | — |
| **Número** | Sim | `42` | — |
| **Bairro** | Sim | `Centro` | — |
| **Complemento** | Não | `Apto 8, Bloco C` | Ajuda o motoboy a encontrar a unidade dentro do edifício |
| **Ponto de referência** | Não | `Em frente ao mercado Boa Vista` | Ajuda o motoboy em ruas de difícil acesso |

> Complemento e ponto de referência aparecem no link de navegação enviado ao motoboy e na tela de entrega dele.

---

## Passo a passo

| # | Quem | O que acontece |
|---|------|----------------|
| 1 | 👤 Cliente | Abre **Endereços** no perfil |
| 2 | 👤 Cliente | Vê todos os endereços já usados em compras, com um marcado como padrão |
| 3 | 👤 Cliente | Marca outro como **Padrão** |
| 4 | Sistema | O checkout seguinte abre com esse endereço selecionado |
| 5 | 👤 Cliente | Edita rua, número, complemento, bairro, CEP ou ponto de referência de um endereço |
| 6 | Sistema | Se o CEP novo estiver fora da zona, avisa e não deixa salvar. Ver [FLUX-SAD-08-frete-fora](../sadPaths/FLUX-SAD-08-frete-fora.md) |
| 7 | 👤 Cliente | Apaga um endereço que não quer mais |
| 8 | Sistema | Tira o endereço da lista. Pedidos antigos continuam mostrando o endereço daquela entrega |
| 9 | 👤 Cliente | No checkout, digita um endereço novo (incluindo complemento e ponto de referência, se quiser) |
| 10 | Sistema | Usa esse endereço na compra e **salva** na lista |

> O endereço gravado no pedido é uma **cópia completa** (incluindo complemento e ponto de referência). Editar ou apagar o cadastro não reescreve entrega já feita.
