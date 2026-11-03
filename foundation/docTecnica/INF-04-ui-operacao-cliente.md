# INF-04 — Layout de telas (KDS, motoboy, cliente, gerente)

**Referenciado por:** fluxos HAPPY-09, HAPPY-10, HAPPY-11, HAPPY-18, HAPPY-19, HAPPY-20, HAPPY-21, HAPPY-22 · [ORD-01](ORD-01-estados-pedido.md) § 5

*[← Índice docTecnica](README.md)*

Especificação de **IHC + contrato de dados** para implementação Angular / Android. Regras de estado em ORD-01; transporte em INF-02; métricas em DB-03 § 6.

---

## 1. KDS cozinha (`web-operacao`)

Layout **dois painéis** (desktop/tablet horizontal). Vários pedidos **`EM_PREPARO`** simultâneos.

```
┌─────────────────────────┬──────────────────────────────┐
│  PEDIDOS (N)            │  DETALHE DO SELECIONADO      │
│  #1042  ⚡  · recebido  │  ┌ Preparar          Concluir preparo ┐
│  #1043    · em preparo ◀│  │        ⏱ 08:42                  │
│  #1044    · recebido    │  └──────────────────────────────────┘
│  (clique = seleção)     │  #1043 · linhas order_items…           │
└─────────────────────────┴──────────────────────────────┘
```

| Elemento | Regra |
|----------|--------|
| **Esquerda** | Uma **linha por pedido** `RECEBIDO` ou `EM_PREPARO`. Ordem: ORD-01 § 5.1. Badge **N** = total de linhas |
| **Seleção** | Clique na linha → destaque (cor/borda). **Não** muda estado. Detalhe à direita segue o selecionado |
| **Estilo por estado** | `RECEBIDO`: visual “aguardando preparo”. `EM_PREPARO`: borda/cor distinta (lista + detalhe) |
| **Detalhe — timer** | Centro superior: cronômetro desde `prep_started_at` enquanto `EM_PREPARO`; congela ao concluir (some com o pedido) |
| **Preparar** (↖ superior esquerdo) | Visível se selecionado `RECEBIDO`. STOMP accept → `EM_PREPARO`. Some após clicar |
| **Concluir preparo** (↗ superior direito) | Visível se selecionado `EM_PREPARO`. STOMP ready → `PRONTO`; linha e detalhe **somem** |
| **Linhas do pedido** | `order_items`: nome × qty (somente leitura). Sem ação por linha |
| **Embalagem** | Antes de concluir, anotar `display_number` no pacote |

Estados `PRONTO`+ **não** aparecem no KDS (INF-02). **Nunca** auto-preparar ao selecionar ou ao abrir a tela.

---

## 2. App motoboy — fila e aceite no balcão

| Tela | Conteúdo |
|------|----------|
| **Fila de prontos** | Lista `PRONTO` com **`display_number`** em destaque (e opcional horário). Sem nome do cliente na lista |
| Fluxo físico | Motoboy compara código na **embalagem** com o app → seleciona o pedido → **Aceitar pedido** |
| **Aceitar pedido** | HTTP `POST /orders/{id}/take` (Idempotency-Key) → `SAIU_PARA_ENTREGA` + aloca motoboy → navega para **Acompanhamento da entrega** |
| **Acompanhamento** | Nome do cliente, telefone, botão **WhatsApp**, **Waze**, **Google Maps** (`GET /orders/{id}/navigation-links`, `contact-links`) |
| **Finalizar entrega** | `POST /orders/{id}/deliver` → `ENTREGUE` → volta à **Fila de prontos** |

Rótulo **Aceitar pedido** no app motoboy — distinto do botão **Preparar** do KDS (INF-02 § 4).

---

## 3. App cliente — cardápio e checkout

| Tela | Conteúdo |
|------|----------|
| Cardápio | Seções verticais por categoria (DB-04 § 1.1). Produto: foto, nome, preço, **+** quantidade. Dados: `GET /menu` (INF-02 § 11.1) |
| Carrinho | Linhas editáveis (−/+, remover), subtotal |
| Pagamento / checkout | Endereço: **combobox** endereços salvos + campos manuais; linha **Entrega** (taxa fixa); subtotal + total; pagamento: combobox cartões salvos ou novo; **Finalizar pedido** |
| **nos contacte** | Tela dedicada (menu): botão **💬 WhatsApp da loja** → `GET /store/contact-links` |
| **Acompanhar pedido** | Timeline + **💬 Fale com a loja** (sempre) + **🛵 WhatsApp do motoboy** só em `SAIU_PARA_ENTREGA` → `GET /orders/{id}/contact-links` |
| Pós-checkout | Sucesso **ou** tela/ modal de erro com código legível (`ITEM_UNAVAILABLE`, `PAYMENT_FAILED`, etc.) |

Autocadastro, login e reset senha: INF-03 § 6.1–6.4. Contatos: [DB-01](DB-01-canal-atendimento.md) · [FLUX-HAPPY-17](../fluxos/happypaths/FLUX-HAPPY-17-whatsapp.md).

---

## 4. Cardápio gerente (`web-operacao`)

Tela **Cardápio** — CRUD de `products` (DB-02 § 1). Transporte: INF-02 § 11.

| Elemento | Regra |
|----------|--------|
| **Filtro categoria** | Combobox alimentado por `GET /store/product-categories`. Opção **Todas** (`categoryId` omitido na query). Filtra a tabela via `GET /store/products?categoryId=…` |
| **Busca** | Campo `q` (nome parcial) na mesma listagem |
| **Colunas da tabela** | Nome, categoria (`display_name`), preço, `availability`, **limite padrão**, **vendido hoje**, **restante**, **limite efetivo hoje** |
| **Novo / editar** | Formulário: nome, descrição, categoria, preço, foto (`POST …/image` ou multipart no create), limite padrão, ordem na categoria (`sort_order`) |
| **Ações rápidas** | Pausar dia, bloquear, arquivar, reativar → `PATCH …/products/{id}` (`availability`) — [FLUX-HAPPY-10](../fluxos/happypaths/FLUX-HAPPY-10-item-gerente.md) |
| **Limite só hoje** | Controle numérico inline ou modal → `PATCH …/products/{id}/daily-cap` |
| **Itens arquivados** | Aba ou toggle **Ver arquivados** → `GET /store/products?archivedOnly=true` |
| **Rodapé estatísticas** | Bloco § 5 abaixo da tabela (itens mais vendidos, etc.) |

Categorias MVP são **seed fixo** (DB-04 § 1.1); gerente **não** cria categorias — só associa produto a uma existente.

---

## 5. Gerente — blocos de estatísticas embutidos

Padrão de UI: **tabela ou conteúdo principal** + **faixa inferior** (cards ou mini-gráficos) com métricas derivadas de DB-03. Somente leitura; atualiza ao carregar a tela e, no **Painel do dia**, também via WebSocket quando um pedido conclui preparo.

| Tela gerente | Bloco inferior (exemplos) | API / doc |
|--------------|---------------------------|-----------|
| **Painel → Pedidos do day** | Tempo médio preparo (hoje); pedidos hoje / semana / mês; média pedidos/dia (7d, 30d); p95 tempo preparo; cancelamentos hoje | `GET /store/stats/orders?…` · DB-03 § 6.1 |
| Tabela pedidos | Coluna **Tempo preparo** (`prep_duration_seconds` formatado mm:ss) por linha entregue ou já `PRONTO`+ | Campo em `GET /orders` |
| **Cardápio** | Top itens (unidades, receita); itens que mais esgotaram limite diário; mix por categoria | DB-03 § 6.2 |
| **Estatísticas** ([HAPPY-19](../fluxos/happypaths/FLUX-HAPPY-19-stats.md)) | Visão completa por período (já existente) | DB-03 § 2 |
| **Funcionários / motoboys** (futuro próximo) | Entregas por motoboy, tempo médio entrega | DB-03 § 2 Tela 5 |

Implementação MVP: queries sob demanda (sem cache). Período padrão do rodapé = **dia operacional** (DB-03 § 1.1), com seletor rápido 7d / 30d onde fizer sentido.

---

*[← Índice docTecnica](README.md)*
