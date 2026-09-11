# Elementos obrigatórios — barra de aprovação TADS/UFPR

Visão geral. O que falta aqui vira argumento de reprovação ou de “CRUD com pele”.

---

## Stack

- Kotlin + Spring Boot + JPA (backend)
- Angular (frontend)
- Fastify como API gateway / BFF
- Redis (sessão, denylist, logout forçado)
- PostgreSQL (modelo relacional normalizado)
- Monólito modular (módulos de domínio no mesmo deploy Spring)
- HTTPS em todo o tráfego browser → gateway → backend (TLS no gateway; cookies só com `Secure`)
- Gateway na frente; browser nunca fala direto com o Spring em produção acadêmica

---

## Identidade, sessão e autorização

- Cadastro / autenticação de usuários
- Senhas com Argon2id (nunca bcrypt/SHA; hash só no Spring)
- Mais de um perfil (RBAC)
- FGAC (autorização por recurso/atributo, não só por role)
- JWT em par: access + refresh, ambos em cookies HttpOnly + `Secure` + `SameSite`
- Refresh com rotação
- Redis para denylist / versão de sessão / logout forçado (inclusive “deslogar todos os dispositivos”)
- CSRF consciente do modelo cookie-to-cookie via gateway

---

## API HTTP

- REST até Richardson nível 3 (HATEOAS)
- Links hipermídia nos recursos (`self`, transições de estado, ações permitidas ao perfil)
- Verbos e status corretos; sem RPC disfarçado de POST único
- Erros padronizados (Problem Details)
- Paginação, filtros e idempotência nos POST críticos
- Contratos versionáveis; OpenAPI gerada a partir do Spring
- Catálogo documental de requests/responses JSON (front ↔ gateway ↔ back)

---

## Natureza transacional (não negociável)

- Regras de negócio além de CRUD
- Máquinas de estado nos agregados centrais
- Transações ACID no PostgreSQL
- Concorrência: lock otimista (e pessimista onde o domínio exigir)
- Integridade relacional (PK, FK, unique, checks)
- Operações compostas atômicas (reserva, lance, liquidação, alocação, estorno)
- Idempotency keys nos fluxos que o usuário pode repetir

---

## Arquitetura

- Monólito modular: módulos por domínio, não por “tela”
- Camadas limpas no Spring (aplicação / domínio / infra)
- Gateway Fastify: auth cookies, agregação, CORS, rate limit básico, roteamento HATEOAS-friendly
- Sem microserviços no TCC (custo sem ganho de banca)
- Auditoria das transações críticas (quem, quando, o quê, estado anterior/novo)

---

## Dados

- DER normalizado (3FN como regra; desnormalizar só com justificativa)
- Entidades de instância (nada de “item genérico” quando o domínio exige unicidade)
- Script SQL de criação do banco (schema completo, reproduzível)
- Flyway: migrações versionadas + seed (perfis, dados de demonstração da banca)
- Soft constraints de negócio no banco, não só no Angular

---

## IHC (Angular)

- Fluxos por persona/perfil (menu e ações mudam com FGAC)
- Navegação dirigida pelos links da API (UI não inventa transição ilegal)
- Estados vazios, erro e loading
- Heurísticas básicas: consistência, feedback, prevenção de erro, controle do usuário
- Telas que expõem o fluxo transacional (não só cadastros)

---

## Qualidade mínima de entrega

- Testes das transações críticas (serviço + concorrência)
- Docker Compose: Spring + PostgreSQL + Redis + Fastify + Angular
- Logs com correlação no fluxo gateway → backend
- README de subida local e recorte de escopo (o que está fora)

---

## Artefatos da banca

- Casos de uso (principal, alternativo, exceção)
- Histórias de usuário
- Diagramas de sequência (fluxos transacionais críticos)
- Diagrama de banco lógico (e físico / DER quando a banca pedir)
- Diagrama de classes
- Máquinas de estado (Mermaid)
- Personas e matriz de permissões (RBAC + FGAC)
- Contratos da API (incluindo rels hipermídia)
- Catálogo de requests e responses JSON esperados (front ↔ back, inclusive erros e links HATEOAS)
- Protótipo/fluxo de telas
- Script SQL de criação do banco + Flyway com seed
- Decisões explícitas: HTTPS, Argon2id, cookies, Redis, monólito, Richardson 3
