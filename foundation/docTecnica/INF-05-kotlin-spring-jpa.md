# INF-05 — Backend Kotlin: Spring Boot, Spring Data JPA, Hibernate e PostgreSQL

**Referenciado por:** [`pitch_delivery.md`](../pitch_delivery.md) § 5 · [`IMP-01`](../IMP-01-ordem-implementacao-ai.md) · [índice docTecnica](README.md)

*[← Índice docTecnica](README.md)*

---

## 1. Stack normativa do monólito

O **backend da aplicação** (API REST, domínio, persistência, WebSocket de negócio) é implementado em:

| Camada | Tecnologia |
|--------|------------|
| Runtime / injeção / web | **Spring Boot** (linguagem **Kotlin**) |
| Persistência | **Spring Data JPA** |
| ORM | **Hibernate** (provedor JPA padrão do Spring Boot) |
| Banco | **PostgreSQL** (conexão JDBC via `spring.datasource` + driver PostgreSQL) |

**Fonte da verdade dos dados:** apenas PostgreSQL, conforme [DB-03](DB-03-telemetria.md) e demais docs `DB-*`.

Para usar Spring Boot com Spring Data JPA, Hibernate e PostgreSQL em **Kotlin**, é preciso alinhar a **configuração do projeto**, a **conexão com o banco** e a **estrutura das classes** para neutralizar o comportamento padrão do Kotlin (classes finais, construtores, `data class`).

---

## 2. Por que Kotlin exige cuidado com Spring e Hibernate

Em Kotlin, classes são **final** por padrão. **Spring** e **Hibernate** precisam de classes **abertas** (`open`) para:

- Criar **proxies** em tempo de execução (Spring para componentes transacionais e outros advices; Hibernate para **lazy loading** de associações).

Sem isso, recursos como proxies e carregamento tardio não se comportam como o framework espera.

O Hibernate também exige um **construtor sem argumentos** para materializar entidades — o compilador Kotlin não gera isso automaticamente para classes como o Java faz; o plugin JPA resolve em tempo de compilação (equivalente ao **kotlin-noarg**).

**Esclarecimento — recursos afetados pela classe `final`:** além do lazy loading, classes precisam ser “abertas” (via plugin ou `open`) para o funcionamento correto de **`@Transactional`** e da **criação de proxies** do Spring em geral — não só para Hibernate.

---

## 3. Gradle (`build.gradle.kts`) — plugins obrigatórios

No arquivo **`build.gradle.kts`**, incluir os plugins:

- **`org.jetbrains.kotlin.plugin.spring`**
- **`org.jetbrains.kotlin.plugin.jpa`**

**Papel de cada um:**

- **Plugin Spring:** o Kotlin gera classes **finais** por padrão; Spring e Hibernate esperam classes **abertas** para criar proxies e habilitar **lazy loading**. O plugin Spring trata isso abrindo automaticamente as classes alvo dos estereótipos Spring (e o ecossistema fica alinhado a **kotlin-allopen** com preset Spring).

- **Plugin JPA:** gera, em tempo de compilação, um **construtor sem argumentos** para entidades (`@Entity`, etc.) — exigência estrita do Hibernate. Corresponde ao que o **kotlin-noarg** faria com anotações JPA.

**Esclarecimento — nomes “debaixo do capô”:** os plugins “envelopados” acima são **`kotlin-spring`** e **`kotlin-jpa`**; por baixo usam explicitamente as ferramentas **kotlin-allopen** e **kotlin-noarg** com os presets adequados. Na documentação de build costuma-se citar só os plugins Gradle; aqui fica explícito **all-open** e **no-arg** como mecanismos reais.

---

## 4. Banco de dados: `application.yml` / `application.properties`

Configurar:

- **URL JDBC** do PostgreSQL, **driver**, **usuário** e **senha** (`spring.datasource.*`).
- Propriedades do **Hibernate/JPA** para o Spring gerir a conexão e o `EntityManagerFactory`, por exemplo:
  - estratégia de DDL alinhada ao projeto (ex.: `hibernate.hbm2ddl.auto` / `spring.jpa.hibernate.ddl-auto` — o valor exato depende de política de migrações, ex. Flyway/Liquibase);
  - **dialeto** PostgreSQL (`spring.jpa.database-platform` ou derivação automática).

O Spring Boot usa **HikariCP** como pool JDBC por padrão.

---

## 5. Entidades e repositórios

### 5.1 Entidades (`@Entity`)

- Usar a palavra-chave comum **`class`**, **não** `data class` como padrão para agregados com associações lazy.

**Por que evitar `data class` em entidades ricas (especialmente com lazy):**

- `data class` gera `equals()`, `hashCode()` e `toString()` automaticamente.
- Esses métodos podem **acessar propriedades de carregamento tardio** (lazy) fora de um contexto adequado, disparando **`LazyInitializationException`**.
- Também há risco de **loops** ou comportamento indesejado em `equals`/`hashCode` quando coleções ou grafos lazy entram na igualdade.

Ou seja: o problema técnico central não é só “loop”, e sim **equals/hashCode/toString gerados tocando lazy** fora da sessão.

- Anotar com `@Entity` e usar propriedades **mutáveis** (`var`) com valores padrão ou nulos onde o modelo exigir, para o Hibernate manipular a entidade durante o ciclo de vida.

### 5.2 Repositórios (Spring Data JPA)

- Definir **interfaces** que estendem `JpaRepository<SuaClasse, TipoDoId>` para herdar as operações de CRUD (e convenções de query derivada) do Spring Data JPA.

---

## 6. Resumo do checklist

| Item | Ação |
|------|------|
| Projeto | Plugins `org.jetbrains.kotlin.plugin.spring` e `org.jetbrains.kotlin.plugin.jpa` no `build.gradle.kts` |
| Conexão | PostgreSQL: JDBC URL, driver, credenciais, pool; Hibernate/JPA (dialeto, DDL/migrações) |
| Entidades | `class` + `@Entity` + `var` conforme modelo; **não** usar `data class` como regra para entidades com lazy |
| Repositórios | `JpaRepository<Entidade, Id>` |
| Motivação | Classes **open** (via plugin) para proxies Spring/Hibernate e lazy; **no-arg** para Hibernate |

---

## 7. Documentos relacionados

- Arquitetura geral: [pitch_delivery.md](../pitch_delivery.md) § 5.
- WebSocket e monólito: [INF-01-tempo-real.md](INF-01-tempo-real.md).
- Autenticação e gateway: [INF-03-autenticacao.md](INF-03-autenticacao.md).

---

*[← Índice docTecnica](README.md)*
