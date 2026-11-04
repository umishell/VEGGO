# INF-03 — Autenticação por Plataforma

> **Escopo:** estratégia de tokens JWT, armazenamento seguro, refresh automático e autenticação de WebSocket para cada superfície do sistema VEGGO.
>
> **Relacionado:** [INF-02-http-vs-websocket.md](INF-02-http-vs-websocket.md) — quando e como cada ator se comunica | [ORD-01-estados-pedido.md](ORD-01-estados-pedido.md) — máquina de estados

---

## 1. Plataformas e estratégia de token

O sistema possui **quatro perfis** em **duas plataformas distintas**:

| Perfil | Aplicação | Plataforma | Estratégia de token |
|--------|-----------|------------|---------------------|
| `FUNCIONARIO_COZINHA` | `web-operacao` | Angular (navegador web) | JWT em cookie `HttpOnly` |
| `GERENTE_LOJA` | `web-operacao` | Angular (navegador web) | JWT em cookie `HttpOnly` |
| `CLIENTE` | `app-cliente` | Android nativo (Kotlin) | JWT no cabeçalho `Authorization: Bearer` |
| `MOTOBOY` | `app-motoboy` | Android nativo (Kotlin) | JWT no cabeçalho `Authorization: Bearer` |

**Regra central:** nenhum app guarda JWT em `localStorage` ou `sessionStorage`. Nenhum app mobile guarda JWT em `SharedPreferences` simples.

---

## 2. Web Angular — Cozinha e Gerente (`web-operacao`)

### 2.1 Tokens em cookies HttpOnly

Dois cookies são emitidos pelo Fastify na resposta de login:

| Cookie | Conteúdo | TTL | Flags |
|--------|----------|-----|-------|
| `access_token` | JWT com `userId`, `roles`, `storeId` | 15 minutos | `HttpOnly; Secure; SameSite=Strict` |
| `refresh_token` | JWT opaco (só `jti` + `sub`) | 7 dias | `HttpOnly; Secure; SameSite=Strict; Path=/auth/refresh` |

- **`HttpOnly`:** JavaScript do Angular **não consegue ler** esses cookies. Proteção total contra XSS — um script malicioso não consegue roubar os tokens.
- **`Secure`:** o cookie só é enviado em conexões HTTPS.
- **`SameSite=Strict`:** o cookie não é enviado em requisições cross-site. Elimina a maioria dos vetores de CSRF.
- O `refresh_token` usa `Path=/auth/refresh` para que ele **só seja enviado** para a rota de renovação, nunca para a API de negócio.

### 2.2 Proteção CSRF (dupla camada)

`SameSite=Strict` já é suficiente para proteger contra CSRF em cenários modernos, mas o Fastify adiciona uma segunda camada via **Double Submit Cookie**:

1. No login, além dos cookies `HttpOnly`, o Fastify emite um cookie `csrf_token` **sem** `HttpOnly` (lido pelo JavaScript).
2. O Angular lê esse valor e o envia no cabeçalho `X-CSRF-Token` em todas as requisições mutantes (`POST`, `PUT`, `PATCH`, `DELETE`).
3. O Fastify valida que o `X-CSRF-Token` do cabeçalho **coincide** com o valor do cookie `csrf_token`. Se divergir: HTTP 403.

O atacante de CSRF nunca consegue ler o cookie `csrf_token` por causa da política de mesma origem do browser, portanto não consegue incluir o cabeçalho correto.

### 2.3 Refresh token rotation (web)

```
Angular faz request → Fastify detecta access_token expirado → HTTP 401
         │
Angular interceptor detecta 401
         │
         ▼
POST /auth/refresh
  Cookie: refresh_token=<token_antigo>

Fastify:
  1. valida assinatura do refresh_token
  2. verifica Redis: token está na denylist? → rejeita (HTTP 401 → forçar login)
  3. verifica session_version: token.ver == Redis user:{userId}:session_version? → ok
  4. gera novo par (access_token + refresh_token)
  5. adiciona refresh_token_antigo ao Redis denylist com TTL = tempo de vida restante
  6. retorna novo par em cookies HttpOnly

Angular refaz a request original (o interceptor já injeta os novos cookies automaticamente)
```

**Rotation**: cada refresh_token só funciona uma vez. Se um atacante roubar e usar o refresh_token antes do usuário legítimo, na próxima renovação do usuário legítimo o Fastify detecta o token antigo na denylist e invalida a sessão inteira (proteção contra roubo de refresh_token).

### 2.4 Logout e "deslogar todos os dispositivos"

| Ação | Mecanismo |
|------|-----------|
| Logout simples | `POST /auth/logout` → Fastify adiciona o `jti` do refresh_token atual ao Redis denylist |
| Deslogar todos | `POST /auth/logout-all` → Fastify incrementa `user:{userId}:session_version` no Redis → todos os tokens com versão antiga são rejeitados |

### 2.5 WebSocket com cookie (Angular) — handshake automático

O browser envia os cookies **automaticamente** no handshake HTTP Upgrade do WebSocket, sem que o Angular precise fazer nada especial:

```ts
// @stomp/rx-stomp — o cookie é enviado automaticamente
const config: InjectableRxStompConfig = {
  brokerURL: 'wss://api.veggo.com/ws', // Fastify proxy
  connectHeaders: { /* sem Authorization aqui — o cookie vai junto */ },
  heartbeatIncoming: 10000,
  heartbeatOutgoing: 10000,
};
```

Fastify valida o `access_token` do cookie antes de aceitar o upgrade. Se inválido: HTTP 401 e o WebSocket nunca abre. Se o token expirar **durante** a sessão WebSocket:

- Fastify fecha a conexão com código `4001 Unauthorized`
- Angular intercepta o `closeEventCode` no listener de `onStompError`
- Angular executa o refresh (POST /auth/refresh)
- Angular re-abre o WebSocket com os novos cookies (automático)
- Angular busca estado atual via HTTP GET antes de re-assinar os canais

---

## 3. Mobile Android Kotlin — Cliente e Motoboy (`app-cliente` / `app-motoboy`)

Apps nativos Android **não usam cookies**. O JWT é enviado como cabeçalho HTTP.

### 3.1 TTL de sessão — `CLIENTE` vs demais mobile

| Perfil | `access_token` | `refresh_token` | Comportamento |
|--------|----------------|-----------------|---------------|
| `CLIENTE` (`app-cliente`) | 15 minutos | **90 dias (~3 meses)** | Rotation no refresh; usuário permanece logado salvo **logout** ou revogação (`session_version`) |
| `MOTOBOY` (`app-motoboy`) | 15 minutos | 7 dias | Idem rotation |

Login do cliente: Fastify emite par JWT; app persiste em EncryptedSharedPreferences. Enquanto o refresh for válido, reabrir o app **não** exige login — carrinho segue regras DB-04 § 6 (TTL **3 h** independente dos 90 dias de login).

### 3.2 Armazenamento seguro — EncryptedSharedPreferences

```kotlin
// build.gradle.kts
implementation("androidx.security:security-crypto:1.1.0-alpha06")
```

```kotlin
// TokenStore.kt
class TokenStore(context: Context) {
    private val prefs: SharedPreferences = EncryptedSharedPreferences.create(
        context,
        "veggo_tokens",
        MasterKey.Builder(context)
            .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
            .build(),
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )

    fun saveTokens(access: String, refresh: String) {
        prefs.edit()
            .putString("access_token", access)
            .putString("refresh_token", refresh)
            .apply()
    }

    fun getAccessToken(): String? = prefs.getString("access_token", null)
    fun getRefreshToken(): String? = prefs.getString("refresh_token", null)

    fun clearTokens() = prefs.edit().clear().apply()
}
```

**Por que EncryptedSharedPreferences?**
- Usa AES-256-GCM para criptografar valores e AES-256-SIV para criptografar chaves.
- A **master key** fica no **Android Keystore** (hardware-backed em dispositivos modernos), nunca exposta ao app.
- Mesmo que o arquivo `.xml` seja extraído de um dispositivo rooteado, os dados permanecem cifrados sem a master key do Keystore.
- Substitui completamente `SharedPreferences` simples, que guarda em texto plano.

### 3.3 Envio automático do JWT — OkHttp Interceptor

```kotlin
// AuthInterceptor.kt
class AuthInterceptor(private val tokenStore: TokenStore) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val token = tokenStore.getAccessToken()
            ?: return chain.proceed(chain.request()) // não autenticado
        val request = chain.request().newBuilder()
            .header("Authorization", "Bearer $token")
            .build()
        return chain.proceed(request)
    }
}

// OkHttpClient configurado na injeção de dependência (Hilt/Koin)
val okHttpClient = OkHttpClient.Builder()
    .addInterceptor(AuthInterceptor(tokenStore))
    .authenticator(TokenAuthenticator(authApi, tokenStore))
    .build()
```

O `AuthInterceptor` adiciona o cabeçalho `Authorization: Bearer <access_token>` em **todas** as requisições. Nenhuma tela precisa se preocupar com tokens.

### 3.4 Refresh automático — OkHttp Authenticator

O `Authenticator` é chamado pelo OkHttp quando a resposta é HTTP 401 (token expirado):

```kotlin
// TokenAuthenticator.kt
class TokenAuthenticator(
    private val authApi: AuthApi,
    private val tokenStore: TokenStore
) : Authenticator {
    // evita loop de refresh (no máximo 1 tentativa)
    private val refreshInProgress = AtomicBoolean(false)

    override fun authenticate(route: Route?, response: Response): Request? {
        if (response.request.header("X-Auth-Retry") != null) return null
        if (!refreshInProgress.compareAndSet(false, true)) return null

        return try {
            val refreshToken = tokenStore.getRefreshToken() ?: return null
            val result = authApi.refresh(
                RefreshRequest(refreshToken = refreshToken)
            ).execute()

            if (result.isSuccessful) {
                val body = result.body() ?: return null
                tokenStore.saveTokens(body.accessToken, body.refreshToken)
                response.request.newBuilder()
                    .header("Authorization", "Bearer ${body.accessToken}")
                    .header("X-Auth-Retry", "1") // marca para não re-tentar
                    .build()
            } else {
                // refresh falhou (token expirado/revogado) → forçar login
                tokenStore.clearTokens()
                null // OkHttp propaga o 401 original
            }
        } finally {
            refreshInProgress.set(false)
        }
    }
}
```

O header `X-Auth-Retry` é interno (não chega ao Fastify) — serve apenas para evitar loop infinito de refresh caso o novo access_token também retorne 401.

### 3.5 WebSocket com header Authorization — OkHttp WebSocket

Apps Android, ao contrário do browser, podem enviar cabeçalhos customizados **durante o handshake** WebSocket:

```kotlin
// WebSocketManager.kt
class WebSocketManager(
    private val okHttpClient: OkHttpClient,
    private val tokenStore: TokenStore
) {
    private var webSocket: WebSocket? = null

    fun connect(orderId: String, onMessage: (String) -> Unit) {
        val token = tokenStore.getAccessToken() ?: return
        val request = Request.Builder()
            .url("wss://api.veggo.com/ws") // Fastify proxy
            .header("Authorization", "Bearer $token")
            .build()

        webSocket = okHttpClient.newWebSocket(request, object : WebSocketListener() {
            override fun onOpen(webSocket: WebSocket, response: Response) {
                // conectado — enviar frame STOMP CONNECT e subscribe
                webSocket.send(buildStompConnect())
                webSocket.send(buildStompSubscribe("/topic/order/$orderId"))
            }

            override fun onMessage(webSocket: WebSocket, text: String) {
                onMessage(text)
            }

            override fun onClosed(webSocket: WebSocket, code: Int, reason: String) {
                if (code == 4001) {
                    // Fastify recusou: token expirado durante a sessão
                    refreshAndReconnect(orderId, onMessage)
                }
            }

            override fun onFailure(webSocket: WebSocket, t: Throwable, response: Response?) {
                // queda de rede → backoff e reconexão
                scheduleReconnect(orderId, onMessage)
            }
        })
    }

    private fun refreshAndReconnect(orderId: String, onMessage: (String) -> Unit) {
        // executa refresh via Retrofit (fora do contexto OkHttp WS)
        // depois: busca estado HTTP GET → re-abre WS com novo token
        CoroutineScope(Dispatchers.IO).launch {
            // tokenStore já foi atualizado pelo TokenAuthenticator se houver requests HTTP paralelos
            // se não: chamar authApi.refresh() manualmente
            delay(500)
            connect(orderId, onMessage) // novo token já em tokenStore
        }
    }

    fun disconnect() = webSocket?.close(1000, "Usuário saiu")
}
```

**Diferença crítica em relação ao browser:**
- Browser: cookie vai automaticamente no handshake → sem controle do JS.
- Android: `Request.Builder().header("Authorization", ...)` → total controle do app.

### 3.6 Reconexão com backoff (mobile)

```kotlin
private var reconnectAttempts = 0

private fun scheduleReconnect(orderId: String, onMessage: (String) -> Unit) {
    val delayMs = minOf(3000L * (reconnectAttempts + 1), 30_000L) // 3s, 6s … 30s
    reconnectAttempts++
    CoroutineScope(Dispatchers.IO).launch {
        delay(delayMs)
        // reconcilia com PostgreSQL antes de re-assinar
        val currentState = ordersApi.getOrder(orderId).execute().body()
        // atualiza UI com currentState
        connect(orderId, onMessage) // re-abre WS
        reconnectAttempts = 0
    }
}
```

---

## 4. Fastify Gateway — diferenciação web vs mobile

O Fastify é o **único ponto de entrada** externo. Detecta a plataforma pelo local do token e aplica a mesma validação JWT independentemente:

### 4.1 Lógica de detecção

```
Fastify recebe request
  ├─ Cabeçalho Authorization: Bearer <token> presente?
  │   └─ SIM → plataforma mobile → extrai JWT do header
  └─ NÃO → tenta cookie
      └─ Cookie access_token presente?
          ├─ SIM → plataforma web → extrai JWT do cookie
          └─ NÃO → HTTP 401 Unauthorized
```

### 4.2 Plugin de autenticação Fastify

```js
// auth-plugin.js
import fp from 'fastify-plugin'
import jwt from '@fastify/jwt'

export default fp(async (fastify) => {
  fastify.decorate('authenticate', async (request, reply) => {
    let token

    // 1. mobile: Bearer header
    const authHeader = request.headers['authorization']
    if (authHeader?.startsWith('Bearer ')) {
      token = authHeader.slice(7)
    }
    // 2. web: cookie HttpOnly
    else if (request.cookies?.access_token) {
      token = request.cookies.access_token
    }

    if (!token) {
      return reply.code(401).send({ error: 'Unauthorized' })
    }

    try {
      const payload = fastify.jwt.verify(token)

      // verifica denylist no Redis
      const denied = await redis.get(`denylist:${payload.jti}`)
      if (denied) return reply.code(401).send({ error: 'Token revogado' })

      // verifica session_version
      const currentVersion = await redis.get(`user:${payload.sub}:session_version`)
      if (currentVersion && parseInt(currentVersion) > payload.ver) {
        return reply.code(401).send({ error: 'Sessão invalidada' })
      }

      // propaga identidade ao Spring via headers internos (nunca o JWT bruto)
      request.headers['x-user-id']    = payload.sub
      request.headers['x-user-roles'] = payload.roles.join(',')
      request.headers['x-store-id']   = payload.storeId ?? ''

    } catch (err) {
      return reply.code(401).send({ error: 'Token inválido' })
    }
  })
})
```

### 4.3 WebSocket: autenticação no handshake

O Fastify intercepta o upgrade HTTP antes de repassar ao Spring:

```js
// ws-proxy.js
fastify.get('/ws', { websocket: true }, async (connection, request) => {
  // 'authenticate' já rodou via hook onRequest — se chegou aqui, está autenticado
  // headers x-user-id / x-user-roles foram injetados
  proxyWebSocket(connection, request) // repassa ao Spring WS
})
```

O Fastify usa um hook `onRequest` com a função `authenticate` para todas as rotas, incluindo o upgrade `/ws`. Após o upgrade, os frames individuais não são revalidados — a autenticação do handshake é suficiente.

### 4.4 Headers internos ao Spring

O Spring **nunca vê o JWT**. Recebe apenas os headers internos já validados pelo Fastify:

| Header interno | Conteúdo | Exemplo |
|----------------|----------|---------|
| `X-User-Id` | UUID do usuário | `550e8400-e29b-41d4-a716-446655440000` |
| `X-User-Roles` | Lista separada por vírgula | `GERENTE_LOJA` |
| `X-Store-Id` | UUID da loja (vazio para `CLIENTE`) | `aaaa-bbbb-cccc` |

Este padrão é seguro **porque o Spring não é acessível diretamente pela internet**. Em produção acadêmica: Spring escuta em porta interna (ex: `8080`) não exposta no firewall; apenas o Fastify escuta externamente (porta `443`).

---

## 5. Spring Boot Kotlin — filtro de identidade interna

O Spring monta o `SecurityContext` a partir dos headers internos, sem revalidar JWT:

```kotlin
// InternalAuthFilter.kt
@Component
class InternalAuthFilter : OncePerRequestFilter() {
    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        chain: FilterChain
    ) {
        val userId  = request.getHeader("X-User-Id")
        val roles   = request.getHeader("X-User-Roles")
            ?.split(",")
            ?.map { SimpleGrantedAuthority("ROLE_${it.trim()}") }
            ?: emptyList()
        val storeId = request.getHeader("X-Store-Id")

        if (userId != null) {
            val details = mapOf("storeId" to storeId)
            val auth = UsernamePasswordAuthenticationToken(userId, details, roles)
            SecurityContextHolder.getContext().authentication = auth
        }
        chain.doFilter(request, response)
    }
}
```

Nos controllers, o Spring Security usa `@PreAuthorize("hasRole('GERENTE_LOJA')")` normalmente. O `storeId` está disponível via `authentication.credentials` para o FGAC.

---

## 6. Senhas — Argon2id

O hash de senha é executado **exclusivamente no Spring**. Nunca no Fastify, nunca nos apps.

```kotlin
// SecurityConfig.kt
@Bean
fun passwordEncoder(): PasswordEncoder = Argon2PasswordEncoder(
    16,   // saltLength (bytes)
    32,   // hashLength (bytes)
    4,    // parallelism
    65536, // memory (KB) = 64 MB
    3     // iterations
)
```

- **Argon2id** é o algoritmo vencedor do Password Hashing Competition (2015) e recomendado pelo OWASP para novos sistemas.
- O salt é gerado automaticamente e embutido no hash — nunca comparar senhas em texto plano.
- Parâmetros acima são conservadores para MVP; produção pode elevar `memory` para 128 MB.
- Nunca usar bcrypt, SHA-1, MD5 ou PBKDF2 para senhas em novos sistemas.

---

## 6.1 Quem pode se cadastrar

| Perfil | Cadastro |
|--------|----------|
| `CLIENTE` | **Autocadastro** — `POST /auth/register` no `app-cliente` (nome, e-mail, telefone, senha). Sem convite do gerente |
| `FUNCIONARIO_COZINHA` | **Somente gerente** — tela **Funcionários** no `web-operacao`. Sem endpoint público de registro |
| `MOTOBOY` | Idem cozinha |
| `GERENTE_LOJA` | Fora do MVP narrativo de fluxo (seed/demo ou provisionamento inicial) |

---

## 6.2 Cadastro de funcionário — senha fora da visão do gerente

Fluxo presencial (ver [FLUX-HAPPY-13-usuarios](../fluxos/happypaths/FLUX-HAPPY-13-usuarios.md)):

1. Gerente autenticado chama `POST /store/staff` com `{ name, email, role, phone? }` **sem** campo `password` definido pelo gerente.
2. Resposta inclui `staffSetupToken` de curta duração **ou** a UI web exibe etapa local onde o **funcionário** preenche `password` + `passwordConfirm` no mesmo request final `POST /store/staff/complete` — o Angular usa input type password com **painel voltado ao funcionário**; o gerente não recebe cópia do valor.
3. Spring recebe a senha em claro **uma vez** no TLS, aplica Argon2id, descarta plaintext.

Proibido: gerente enviar senha inicial fixa, gerente ver senha em e-mail ou log.

---

## 6.3 Reset de senha de funcionário

- **Somente** `GERENTE_LOJA`: `POST /store/staff/{userId}/password-reset` com corpo `{ newPassword }` digitado pelo funcionário presente (mesma UX de privacidade do cadastro).
- Incrementa `user:{userId}:session_version` no Redis — derruba sessões antigas.
- Cliente final usa recuperação por e-mail no app — ver § 6.4 e [FLUX-HAPPY-02-recuperacao-senha](../fluxos/happypaths/FLUX-HAPPY-02-recuperacao-senha.md). Não passa pelo gerente.

---

## 6.4 Reset de senha do cliente (e-mail)

| Etapa | Endpoint | Regra |
|-------|----------|-------|
| Solicitar | `POST /auth/password-reset/request` `{ email }` | Gera código numérico/alfanumérico, TTL (~15 min), envia por e-mail. Resposta genérica |
| Validar código | `POST /auth/password-reset/confirm-code` `{ email, code }` | Retorna `resetToken` de curta duração se válido |
| Nova senha | `POST /auth/password-reset/complete` `{ resetToken, password, passwordConfirm }` | Argon2id; invalida código; incrementa `user:{userId}:session_version` |

Proibido: gerente resetar senha de cliente pelo backoffice no MVP.

---

## 7. Redis — papéis na autenticação

| Chave Redis | TTL | Propósito |
|-------------|-----|-----------|
| `denylist:{jti}` | TTL restante do token | Tokens revogados (logout, rotation) |
| `user:{userId}:session_version` | Permanente (até reset manual) | Controle de "deslogar todos os dispositivos" |
| `idempotency:{key}` | 24 horas | Idempotência de operações HTTP (ver INF-02 §6) |

O Redis **não guarda o JWT completo** — apenas o `jti` (JWT ID, campo padrão JWT) como referência. O payload fica sempre no token.

---

## 8. Diagrama de autenticação por plataforma

```
╔══════════════════════════════════════════════════════════════════╗
║  WEB (Angular — Cozinha e Gerente)                               ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  POST /auth/login { email, senha }                               ║
║    └─ Spring: Argon2id.verify → gera JWT                         ║
║    └─ Fastify: emite cookies HttpOnly (access + refresh)         ║
║                                                                  ║
║  Request HTTP ou WS Handshake                                    ║
║    └─ Browser envia cookies automaticamente                      ║
║    └─ Fastify: lê cookie → verifica JWT + Redis                  ║
║    └─ Fastify: injeta X-User-Id, X-User-Roles → Spring           ║
║                                                                  ║
║  Access token expirado (HTTP 401 ou WS 4001)                     ║
║    └─ Angular interceptor → POST /auth/refresh (cookie auto)     ║
║    └─ Fastify: rotation → novos cookies HttpOnly                 ║
║                                                                  ║
╠══════════════════════════════════════════════════════════════════╣
║  MOBILE (Android Kotlin — Cliente e Motoboy)                     ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  POST /auth/login { email, senha }                               ║
║    └─ Spring: Argon2id.verify → gera JWT                         ║
║    └─ Fastify: retorna { accessToken, refreshToken } em JSON     ║
║    └─ App: salva em EncryptedSharedPreferences                   ║
║                                                                  ║
║  Request HTTP                                                    ║
║    └─ OkHttp Interceptor: addHeader("Authorization", "Bearer ...")║
║    └─ Fastify: lê header → verifica JWT + Redis                  ║
║    └─ Fastify: injeta X-User-Id, X-User-Roles → Spring           ║
║                                                                  ║
║  Request WS (OkHttp WebSocket)                                   ║
║    └─ Request.Builder().header("Authorization", "Bearer ...")    ║
║    └─ Fastify: lê header no upgrade → verifica JWT               ║
║    └─ Proxy WS estabelecido após autenticação                    ║
║                                                                  ║
║  Access token expirado (HTTP 401)                                ║
║    └─ OkHttp Authenticator → POST /auth/refresh (Bearer header)  ║
║    └─ Fastify: rotation → { accessToken, refreshToken } em JSON  ║
║    └─ App: atualiza EncryptedSharedPreferences                   ║
║    └─ OkHttp: refaz request original com novo token              ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

---

*[← Índice docTecnica](README.md)*
