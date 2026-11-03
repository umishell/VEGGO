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
| **CEP** | Sim | `80210-170` | Localização da entrega (motoboy / maps) |
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
| 4 | Sistema | No checkout, a combobox já vem com o endereço **padrão** selecionado |
| 5 | 👤 Cliente | Edita rua, número, complemento, bairro, CEP ou ponto de referência de um endereço |
| 6 | 👤 Cliente | Apaga um endereço que não quer mais |
| 7 | Sistema | Tira o endereço da lista. Pedidos antigos continuam mostrando o endereço daquela entrega |
| 8 | 👤 Cliente | No checkout, abre a combobox: escolhe endereço salvo, mantém o padrão ou toca **Adicionar novo** |
| 9 | 👤 Cliente | Se adicionar novo, preenche campos (complemento e referência opcionais) |
| 10 | Sistema | Usa o endereço escolhido na compra e **salva** na lista se for novo |

> O endereço gravado no pedido é uma **cópia completa** (incluindo complemento e ponto de referência). Editar ou apagar o cadastro não reescreve entrega já feita.

**Formas de pagamento (mock MVP):** o cliente pode salvar rótulos simulados no perfil (autofill no checkout) ou informar uma nova forma só na finalização — mesmo padrão da combobox de endereços. Ver [FLUX-HAPPY-05-venda](FLUX-HAPPY-05-venda.md).
