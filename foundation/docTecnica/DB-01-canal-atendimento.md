# DB-01 — Canal de Atendimento ao Cliente (Fale Conosco)

**Referenciado por:**  
→ [FLUX-SAD-18-rota-gerente](../fluxos/sadPaths/FLUX-SAD-18-rota-gerente.md) · [FLUX-SAD-19-rota-motoboy](../fluxos/sadPaths/FLUX-SAD-19-rota-motoboy.md) · [FLUX-SAD-21-aviso-cliente](../fluxos/sadPaths/FLUX-SAD-21-aviso-cliente.md) · [FLUX-HAPPY-17-whatsapp](../fluxos/happypaths/FLUX-HAPPY-17-whatsapp.md)

*[← Índice docTecnica](README.md)*

---

## 1. Decisão de produto

O canal de atendimento "Fale Conosco" **não é implementado dentro do app**. O cliente é redirecionado para o **WhatsApp da loja** via link externo.

**Motivação:**
- Não há chat dentro do sistema: sem tabela de mensagens, sem tela de conversa e sem WebSocket para mensagem. O WebSocket do Spring existe só para a timeline do pedido (`INF-01`). Tudo persiste em PostgreSQL.
- O gerente já usa o WhatsApp no dia a dia — sem curva de aprendizado, sem novo painel.
- O cliente também já usa WhatsApp — UX familiar, sem fricção.
- Reduz escopo técnico sem perda de funcionalidade real para o usuário.

---

## 2. Implementação

O botão **💬 Fale Conosco** é um link externo no formato:

```
wa.me/[código do país][número da loja]
```

**Exemplos:**
```
wa.me/5511999999999
wa.me/5511999999999?text=Olá,%20tive%20um%20problema%20com%20meu%20pedido
```

O parâmetro `text` (opcional) permite pré-preencher a mensagem no WhatsApp — útil no contexto de imprevisto de entrega para já informar o contexto ao gerente.

**Contextos de uso:**

| Contexto | Parâmetro `text` sugerido |
|----------|--------------------------|
| Imprevisto de entrega (via caixa de aviso) | `"Olá, tive um problema com meu pedido #[número]. Poderia me ajudar?"` |
| Contato geral (via dashboard) | *(vazio — cliente digita livremente)* |

---

## 3. O que o app armazena (e o que não armazena)

| Elemento | Armazenado no app? |
|----------|-------------------|
| Flag `incident_alert` no pedido | ✅ Sim — controla o ícone ❗ no card do cliente |
| Conteúdo das mensagens trocadas | ❌ Não — fica no WhatsApp |
| Histórico de conversas | ❌ Não — fica no WhatsApp |
| Status de leitura das mensagens | ❌ Não |
| Banco `conversations` / `messages` | ❌ Não criado |

---

## 4. Configuração do número

O número de WhatsApp da loja deve ser configurável pelo gerente (ou hardcoded na configuração da loja no banco). Não deve ser fixo no código-fonte.

**Sugestão de armazenamento:**

| Campo | Tabela | Tipo |
|-------|--------|------|
| `whatsapp_number` | `stores` | string (ex: `"5511999999999"`) |

---

*[← Índice docTecnica](README.md)*
