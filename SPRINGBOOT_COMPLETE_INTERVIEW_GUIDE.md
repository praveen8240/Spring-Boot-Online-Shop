# Spring Boot — Complete Interview Preparation Guide
### From Zero to 100% Interview-Ready — Everything in One File

> App runs on: `http://localhost:8082`
> H2 Console: `http://localhost:8082/h2-console` (JDBC URL: `jdbc:h2:mem:shopdb`)

---

## Table of Contents

**Part 1 — Core Concepts (In Your Project)**
- [Stage 1 — Spring Core & DI](#stage-1--spring-core-fundamentals)
- [Stage 2 — JPA & Database](#stage-2--database-layer-jpa)
- [Stage 3 — REST API Layer](#stage-3--rest-api-layer)
- [Stage 4 — Exception Handling](#stage-4--error-handling)
- [Stage 5 — AOP](#stage-5--cross-cutting-concerns-aop)
- [Stage 6 — Events](#stage-6--spring-events)
- [Stage 7 — Scheduling](#stage-7--scheduling)
- [Stage 8 — Profiles](#stage-8--profiles)
- [Stage 9 — Testing](#stage-9--testing)
- [Stage 10 — Actuator](#stage-10--production-monitoring-actuator)

**Part 2 — Security (In Your Project)**
- [Security Overview](#security-overview)
- [BCrypt & Password Hashing](#step-3--password-hashing-bcrypt)
- [UserDetailsService](#step-4--userdetailsservice-the-bridge)
- [JWT Deep Dive](#step-5--jwt-the-token)
- [JwtFilter](#step-6--jwtfilter-checking-every-request)
- [SecurityConfig](#step-7--securityconfig-the-master-rules)
- [Auth Controller](#step-8--authcontroller-register-and-login)
- [Role-Based Access](#step-9--role-based-access-preauthorize)

**Part 3 — Missing Topics (Interview Must-Know)**
- [Bean Lifecycle](#1--bean-lifecycle)
- [Pagination & Sorting](#2--pagination--sorting)
- [CORS](#3--cors-configuration)
- [Async Processing](#4--async--asynchronous-processing)
- [HTTP Clients](#5--calling-external-apis-resttemplate--webclient--openfeign)
- [Caching](#6--caching)
- [JPA Auditing](#7--jpa-auditing)
- [Filters & Interceptors](#8--filters--interceptors)
- [Test Slices](#9--test-slices)
- [@Transactional Deep Dive](#10--transactional-deep-dive)
- [N+1 Problem](#11--lazy-vs-eager-loading--n1-problem)
- [Flyway Migrations](#12--database-migrations-flyway)
- [Circuit Breaker](#13--circuit-breaker-resilience4j)
- [Auto-Configuration Internals](#14--spring-boot-auto-configuration-internals)
- [Refresh Tokens](#15--refresh-tokens--jwt-best-practices)
- [DTO Mapping](#16--dto-mapping-mapstruct)
- [Microservices Concepts](#17--microservices-concepts)
- [Quick-Fire Q&A](#18--spring-boot-internals-quick-questions)
- [Final Checklist](#-final-checklist)

---

# PART 1 — CORE CONCEPTS

---

## Stage 1 — Spring Core Fundamentals
> *Understand how Spring manages objects before writing any web code*

---

### 1.1 — What is IoC and Dependency Injection?

| | |
|---|---|
| **Open this file** | `config/DIConceptsDemo.java` |
| **Read sections** | Top comment block → `NotificationService` interface → `@Bean` methods |
| **Key question** | Why don't we write `new EmailNotificationService()`? Who creates it? |

**Concepts learned:**
- IoC Container (Spring creates and manages objects)
- `@Configuration` + `@Bean` — manual bean registration
- `@Primary` — default bean when multiple of same type exist
- `@Qualifier` — pick a specific bean by name
- 3 injection styles: constructor / setter / field

**Interview Q&A:**

| Question | Answer |
|----------|--------|
| What is IoC? | Inversion of Control — you don't create objects, Spring creates and manages them |
| Why prefer constructor injection? | Makes dependencies explicit, allows immutability (`final`), easier to test |
| `@Primary` vs `@Qualifier`? | `@Primary` sets the default; `@Qualifier("name")` picks a specific one explicitly |
| What is `@Component`? | Generic stereotype; `@Service`, `@Repository`, `@Controller` are specializations |

---

### 1.2 — Auto-Detection with Stereotypes

| | |
|---|---|
| **Open this file** | `service/impl/ProductServiceImpl.java` |
| **Read sections** | Class-level annotations, constructor |
| **Key question** | How does Spring know to create `ProductServiceImpl` without a `@Bean` method? |

**Concepts learned:**
- `@Service`, `@Repository`, `@Component` — Spring auto-detects these via `@ComponentScan`
- Constructor injection (recommended pattern)
- `@Transactional` — what it does and when to use `readOnly = true`

---

### 1.3 — Bean Scopes

| | |
|---|---|
| **Open this file** | `config/DIConceptsDemo.java` |
| **Read sections** | `CartService` (singleton) and `RequestContext` (prototype) `@Bean` methods |
| **Key question** | Run the app — how many times is `CartService` constructor printed? |

**Concepts learned:**
- `singleton` — one shared instance per ApplicationContext (default)
- `prototype` — new instance every injection
- When to use each

**Interview Q&A:**

| Question | Answer |
|----------|--------|
| Spring Singleton vs GoF Singleton? | Spring = one per ApplicationContext; GoF = one per JVM |
| When to use prototype? | Stateful beans — shopping carts, request-specific processing |
| Does `@PreDestroy` run on prototype beans? | ❌ No — Spring doesn't manage prototype destruction |

---

### 1.4 — Configuration Properties

| | |
|---|---|
| **Open this file** | `config/AppProperties.java` then `resources/application.properties` |
| **Then open** | `controller/InfoController.java` |
| **Hit endpoint** | `GET http://localhost:8082/api/info` |
| **Key question** | What are the 3 ways to read a property? Which is best and why? |

**Concepts learned:**
- `@ConfigurationProperties(prefix="app")` — type-safe grouped config (best)
- `@Value("${key}")` — single value injection
- `Environment` — programmatic property access
- `application.properties` structure

---

## Stage 2 — Database Layer (JPA)
> *How Spring talks to a database without writing SQL*

---

### 2.1 — Mapping a Class to a Database Table

| | |
|---|---|
| **Open this file** | `model/Product.java` |
| **Then open** | `model/Order.java` |
| **Key question** | Run the app, open H2 Console — do you see `PRODUCTS` and `ORDERS` tables? |

**Concepts learned:**
- `@Entity`, `@Table` — class becomes a DB table
- `@Id`, `@GeneratedValue` — auto-increment primary key
- `@Column` — customize column constraints
- `@ManyToOne`, `@JoinColumn` — foreign key relationship
- `@Enumerated(EnumType.STRING)` — store enum as text

---

### 2.2 — Querying the Database (Zero SQL)

| | |
|---|---|
| **Open this file** | `repository/ProductRepository.java` |
| **Then open** | `repository/OrderRepository.java` |
| **Hit endpoint** | `GET http://localhost:8082/api/products/category/Electronics` |
| **Key question** | How does `findByCategory()` work with no implementation written? |

**Concepts learned:**
- `JpaRepository<Entity, ID>` — free CRUD methods
- Query method naming: `findBy`, `findByXAndY`, `findByXLessThan`
- `@Query` — custom JPQL when naming isn't enough
- `nativeQuery = true` — raw SQL option

---

### 2.3 — Transactions

| | |
|---|---|
| **Open this file** | `service/impl/OrderServiceImpl.java` → `placeOrder()` method |
| **Key question** | If saving the order fails after deducting stock, what happens to stock? |

**Concepts learned:**
- `@Transactional` — wraps multiple DB operations in one atomic unit
- COMMIT on success, ROLLBACK on exception
- Why `placeOrder()` needs a transaction (stock + order are two operations)

---

## Stage 3 — REST API Layer
> *Exposing your data as HTTP endpoints*

---

### 3.1 — Building REST Endpoints

| | |
|---|---|
| **Open this file** | `controller/ProductController.java` |
| **Hit endpoints** | All endpoints in Postman → `Products` folder |
| **Key question** | What is the difference between `@PathVariable` and `@RequestParam`? |

**Concepts learned:**
- `@RestController` = `@Controller` + `@ResponseBody`
- `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`
- `@PathVariable` — from URL: `/products/{id}`
- `@RequestParam` — from query string: `/products?name=iPhone`
- `@RequestBody` — from JSON body
- `ResponseEntity<T>` — control status code (200, 201, 404...)

**Interview Q&A:**

| Question | Answer |
|----------|--------|
| PUT vs PATCH? | PUT replaces entire resource; PATCH updates specific fields |
| Idempotent methods? | GET, PUT, DELETE — same call multiple times = same result |
| Safe methods? | GET, HEAD, OPTIONS — no side effects on server |
| Status codes: Create/NotFound/ValidationError? | 201 / 404 / 400 |

---

### 3.2 — Input Validation

| | |
|---|---|
| **Open this file** | `dto/ProductRequest.java` |
| **Then open** | `exception/GlobalExceptionHandler.java` → `handleValidationErrors()` |
| **Hit endpoint** | `POST /api/products` with blank `name` field |
| **Key question** | What annotation in the controller triggers the validation? |

**Concepts learned:**
- DTO (Data Transfer Object) vs Entity — why they're separate
- `@NotBlank`, `@Size`, `@Positive`, `@Min`, `@Max` — constraint annotations
- `@Valid` on `@RequestBody` — triggers validation
- `MethodArgumentNotValidException` — thrown when validation fails

---

### 3.3 — Standard Response Format

| | |
|---|---|
| **Open this file** | `dto/ApiResponse.java` |
| **Key question** | How does every endpoint return the same `{ success, message, data }` structure? |

**Concepts learned:**
- Generic wrapper class `ApiResponse<T>`
- Static factory methods (`success()`, `error()`)
- `@JsonInclude(NON_NULL)` — skip null fields in JSON

---

## Stage 4 — Error Handling
> *One place to handle all exceptions across all controllers*

---

### 4.1 — Custom Exceptions + Global Handler

| | |
|---|---|
| **Open this file** | `exception/ResourceNotFoundException.java` |
| **Then open** | `exception/GlobalExceptionHandler.java` |
| **Hit endpoint** | `GET http://localhost:8082/api/products/9999` |
| **Key question** | Without `@RestControllerAdvice`, where would you put the try-catch? |

**Concepts learned:**
- `@ResponseStatus(HttpStatus.NOT_FOUND)` on custom exceptions
- `@RestControllerAdvice` — intercepts exceptions from ALL controllers
- `@ExceptionHandler(XyzException.class)` — handles a specific exception type
- Exception hierarchy: 404, 400, 500 handlers

---

## Stage 5 — Cross-Cutting Concerns (AOP)
> *Logic that applies to many methods — written once, applied everywhere*

---

### 5.1 — Aspect Oriented Programming

| | |
|---|---|
| **Open this file** | `aspect/LoggingAspect.java` |
| **Hit any endpoint** | Watch the IntelliJ console output |
| **Key question** | Which code in `ProductServiceImpl` prints the `[AOP @Before]` line? (answer: none!) |

**Concepts learned:**
- Why AOP exists — removes cross-cutting boilerplate (logging, timing, security)
- `@Aspect`, `@Component` — declare an aspect
- Pointcut expression — `execution(* com.example..service.impl.*.*(..))` — which methods to intercept
- `@Before` — runs before the method
- `@After` — runs after (always, even on exception)
- `@AfterReturning` — runs after successful return, can inspect return value
- `@Around` — full control, must call `proceed()` to run the real method

**Interview Q&A:**

| Question | Answer |
|----------|--------|
| What is a Join Point? | A point in execution where an aspect can be applied (usually a method call) |
| What is a Pointcut? | Expression that selects which join points to intercept |
| What is Advice? | The actual code that runs at a join point (`@Before`, `@After`, etc.) |
| What is Weaving? | Process of applying aspects to target objects — Spring uses proxy-based weaving |

---

## Stage 6 — Spring Events
> *Decouple actions that happen as a side effect of business logic*

---

### 6.1 — Publish and Listen to Events

| | |
|---|---|
| **Open this file** | `events/OrderPlacedEvent.java` then `events/OrderEventListener.java` |
| **Then open** | `service/impl/OrderServiceImpl.java` → `placeOrder()` |
| **Hit endpoint** | `POST http://localhost:8082/api/orders` |
| **Key question** | Does `OrderServiceImpl` know anything about email sending? |

**Concepts learned:**
- `ApplicationEvent` — base class for custom events
- `ApplicationEventPublisher.publishEvent()` — fire an event
- `@EventListener` — react to an event (Observer Pattern)
- Loose coupling — service fires event, listener does the work independently

---

## Stage 7 — Scheduling
> *Run tasks automatically on a timer*

---

### 7.1 — Scheduled Tasks

| | |
|---|---|
| **Open this file** | `scheduler/ReportScheduler.java` |
| **Observe** | Console — minute report prints every 60 seconds |
| **Key question** | What happens if a scheduled method throws an exception? |

**Concepts learned:**
- `@EnableScheduling` — activates scheduling (on main class)
- `@Scheduled(fixedRate=)` — every N milliseconds from app start
- `@Scheduled(fixedDelay=)` — N milliseconds after previous run finishes
- `@Scheduled(cron=)` — precise timing with cron expression
- `initialDelay` — wait before first run

**Interview Q&A:**

| Question | Answer |
|----------|--------|
| `fixedRate` vs `fixedDelay`? | `fixedRate`: every N ms regardless of execution time; `fixedDelay`: N ms AFTER last run ends |
| Cron `"0 0 9 * * MON-FRI"`? | Every weekday at 9:00 AM |
| Are scheduled tasks run in parallel by default? | No — single thread; use `@EnableAsync` + `@Async` on the method for parallel execution |

---

## Stage 8 — Profiles
> *Different behaviour for dev, test, and production environments*

---

### 8.1 — Environment Profiles

| | |
|---|---|
| **Open this file** | `resources/application-dev.properties` and `application-prod.properties` |
| **Then open** | `config/AppConfig.java` — `@Profile("dev")` on `seedDemoData` |
| **Key question** | If you change `spring.profiles.active=prod`, will demo data still be seeded? |

**Concepts learned:**
- `spring.profiles.active=dev` — activate a profile
- `application-{profile}.properties` — overrides for that profile
- `@Profile("dev")` on a `@Bean` — bean only exists in that profile
- Dev vs Prod differences: H2 console, SQL logging, error details

**Dev profile settings:**
```properties
spring.jpa.show-sql=true
server.error.include-message=always
server.error.include-stacktrace=on_param
app.environment=DEVELOPMENT
```

**Prod profile settings:**
```properties
spring.jpa.show-sql=false
spring.jpa.hibernate.ddl-auto=validate
spring.h2.console.enabled=false
logging.level.root=WARN
server.error.include-stacktrace=never
app.environment=PRODUCTION
```

---

## Stage 9 — Testing
> *Verify your code works without running the whole app manually*

---

### 9.1 — Unit Tests (fast, isolated)

| | |
|---|---|
| **Open this file** | `test/.../ProductServiceTest.java` |
| **Run** | Right-click → Run in IntelliJ |
| **Key question** | Does this test need a running database? Why not? |

**Concepts learned:**
- `@ExtendWith(MockitoExtension.class)` — activates Mockito (no Spring context)
- `@Mock` — fake object that returns what you tell it to
- `@InjectMocks` — real class with mocks injected
- `when().thenReturn()` — stub a return value
- `verify()` — assert a method was (or wasn't) called
- AssertJ: `assertThat().isEqualTo()`, `assertThatThrownBy()`

---

### 9.2 — Integration Tests (realistic, full stack)

| | |
|---|---|
| **Open this file** | `test/.../ProductControllerIntegrationTest.java` |
| **Run** | Right-click → Run in IntelliJ |
| **Key question** | What is the difference between this test and the unit test above? |

**Concepts learned:**
- `@SpringBootTest` — loads the full ApplicationContext
- `@AutoConfigureMockMvc` — simulates HTTP without a real server
- `MockMvc.perform()` — fire a fake HTTP request
- `jsonPath("$.data[0].name")` — assert JSON response fields
- `@Transactional` on test class — rolls back DB after each test

---

## Stage 10 — Production Monitoring (Actuator)
> *Built-in endpoints to monitor a live application*

---

### 10.1 — Actuator Endpoints

| | |
|---|---|
| **Open this file** | `resources/application.properties` → `management.*` section |
| **Hit endpoints** | Postman → `Actuator` folder |

| Endpoint | Shows |
|----------|-------|
| `/actuator/health` | App + DB status |
| `/actuator/info` | App name, version |
| `/actuator/metrics` | All metric names |
| `/actuator/metrics/jvm.memory.used` | JVM heap usage |
| `/actuator/beans` | Every bean in the context |
| `/actuator/env` | All properties and their values |

---

# PART 2 — SPRING SECURITY

---

## Security Overview

```
CLIENT                          SERVER
  │                               │
  │── POST /auth/register ────────▶  Save user (BCrypt password)
  │◀── { token: "eyJhbG..." } ─────  Return JWT token
  │                               │
  │── POST /auth/login ───────────▶  Verify password
  │◀── { token: "eyJhbG..." } ─────  Return JWT token
  │                               │
  │── GET /api/products ──────────▶  JwtFilter reads token
  │   Header: Bearer eyJhbG...       Validates it, sets user
  │◀── { data: [...products] } ────  Controller runs
  │                               │
  │── GET /api/admin/dashboard ───▶  JwtFilter validates token
  │   Header: Bearer eyJhbG...       SecurityConfig checks role
  │◀── 403 Forbidden ──────────────  ROLE_USER? Blocked!
```

| Error | Meaning | When |
|-------|---------|------|
| `401 Unauthorized` | "Who are you? No token sent." | Missing or invalid token |
| `403 Forbidden` | "I know you, but you can't do this." | Wrong role |

---

## Step 1 — Roles

- `ROLE_USER` → can browse products, place orders
- `ROLE_ADMIN` → can also create/delete products, see admin panel
- Spring Security convention: role names **must** start with `ROLE_`

> `hasRole('ADMIN')` works because Spring auto-adds `ROLE_` prefix.
> Use `hasAuthority('ROLE_ADMIN')` if you want exact string match.

---

## Step 2 — User Entity

- `User` is a normal JPA `@Entity` — just like `Product`
- The `password` field stores a **BCrypt hash**, never the real password
- `@Column(unique = true)` on `username` — no two users with same name

```sql
SELECT * FROM USERS;
-- password shows $2a$10$... — unreadable. BCrypt is one-way.
```

---

## Step 3 — Password Hashing: BCrypt

```
"secret123"  ──BCrypt.encode()──▶  "$2a$10$N9qo8uLOickgx2ZMRZoMye..."
"secret123"  ──BCrypt.encode()──▶  "$2a$10$XqzR7l5mDFkV6YQ2PwZhBe..."  ← different each time!

BCrypt.matches("secret123", "$2a$10$N9qo8uLOickgx2ZMRZoMye...") ──▶ true ✅
BCrypt.matches("wrongpass", "$2a$10$N9qo8uLOickgx2ZMRZoMye...") ──▶ false ❌
```

The `10` in `$2a$10$` is the **work factor** — how many rounds of hashing.
Higher = slower = harder to crack. 10 is the default.

**Interview Q&A:**

| Question | Answer |
|----------|--------|
| Why does BCrypt generate different hashes for the same password? | It includes a random **salt** in the hash; `matches()` extracts and reuses the salt |
| How does login work if we can't un-hash the password? | BCrypt re-hashes what the user types and **compares** the two hashes |

---

## Step 4 — UserDetailsService: The Bridge

Spring Security has no idea about your `User` entity. `UserDetailsServiceImpl` is the **translator**:

```
Spring Security asks: "Give me the user named 'praveen'"
         ↓
UserDetailsServiceImpl.loadUserByUsername("praveen")
         ↓
userRepository.findByUsername("praveen")  ← query YOUR database
         ↓
Convert: User entity → Spring's UserDetails object
         ↓
Spring Security gets: username + hashed password + roles
```

`loadUserByUsername()` is called in TWO places:
1. During **login** — AuthenticationManager calls it for password check
2. In **JwtFilter** — called after extracting username from token

---

## Step 5 — JWT: The Token

A JWT has three parts separated by `.`:
```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJwcmF2ZWVuIiwiaWF0IjoxNzA1MzE2MDAwfQ.SflKxwRJSMeKKF2QT4fw
│────── Part 1 ──────│ │──────────────── Part 2 ────────────────────────│ │───── Part 3 ─────│
     Header                    Payload (username, expiry)                      Signature
```

- **Part 1 — Header:** Algorithm used (`HS256`)
- **Part 2 — Payload:** Who the token belongs to + expiry (**NOT secret! Anyone can read it**)
- **Part 3 — Signature:** Proves nobody tampered with it (uses your secret key)

| Method | When called | Does what |
|--------|------------|-----------|
| `generateToken(userDetails)` | After login/register | Creates and signs a new JWT |
| `extractUsername(token)` | In JwtFilter | Reads username from Part 2 |
| `isTokenValid(token, userDetails)` | In JwtFilter | Checks username matches + not expired |

---

## Step 6 — JwtFilter: Checking Every Request

**Runs on EVERY single HTTP request.**

```
Step 1: Read "Authorization" header → "Bearer eyJhbGciOiJIUzI1NiJ9..."
Step 2: No header? → skip, pass to next filter
Step 3: Strip "Bearer " → get just the token
Step 4: extractUsername(token) → "praveen"
Step 5: loadUserByUsername("praveen") → load from DB
Step 6: isTokenValid(token, userDetails) → true/false
Step 7: Create UsernamePasswordAuthenticationToken
Step 8: SecurityContextHolder.setAuthentication(authToken)
        → "praveen is logged in for THIS request"
Step 9: filterChain.doFilter() → pass to controller
```

**SecurityContextHolder** is thread-local storage for "who is currently logged in".
It lives only for the duration of one request, then gets cleared automatically.

---

## Step 7 — SecurityConfig: The Master Rules

**5 things configured here:**

**1. CSRF disabled** — REST APIs use tokens in headers, not cookies → CSRF irrelevant
```java
.csrf(AbstractHttpConfigurer::disable)
```

**2. Stateless sessions** — Server remembers NOTHING between requests
```java
.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
```

**3. URL access rules (first match wins — ORDER MATTERS)**
```java
.requestMatchers("/auth/**").permitAll()
.requestMatchers(HttpMethod.GET, "/api/products/**").authenticated()
.requestMatchers(HttpMethod.POST, "/api/products/**").hasAuthority("ROLE_ADMIN")
.anyRequest().authenticated()
```

**4. Add JwtFilter before default Spring login filter**
```java
.addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
```

**5. AuthenticationProvider** — wires `UserDetailsServiceImpl` + `BCryptPasswordEncoder`

---

## Step 8 — AuthController: Register and Login

**REGISTER flow:**
```
POST /auth/register  { username: "praveen", password: "pass123" }
   ↓ Check username not taken
   ↓ Hash password → passwordEncoder.encode("pass123") → "$2a$10$..."
   ↓ Save User to DB
   ↓ Generate token → jwtUtil.generateToken(userDetails)
   ↓ Return token to client
```

**LOGIN flow:**
```
POST /auth/login  { username: "praveen", password: "pass123" }
   ↓ authenticationManager.authenticate(username, password)
      → loadUserByUsername("praveen") → load from DB
      → BCrypt.matches("pass123", storedHash) → true or false
      → throws BadCredentialsException if wrong
   ↓ Credentials correct → generate token
   ↓ Return token to client
```

---

## Step 9 — Role-Based Access: @PreAuthorize

**Way 1 — URL-level in SecurityConfig (blocks at filter level — controller never runs):**
```java
.requestMatchers("/api/admin/**").hasAuthority("ROLE_ADMIN")
```

**Way 2 — Method-level (fine-grained):**
```java
@PreAuthorize("hasRole('ADMIN')")
public ApiResponse<?> dashboard(@AuthenticationPrincipal UserDetails currentUser) {
    currentUser.getUsername();  // "admin" — injected by Spring from SecurityContextHolder
}
```
Requires `@EnableMethodSecurity` in `SecurityConfig`.

**Tests:**
```
Register as admin → GET /api/admin/dashboard → ✅ 200 OK
Register as user  → GET /api/admin/dashboard → ❌ 403 Forbidden
No token at all   → GET /api/admin/dashboard → ❌ 401 Unauthorized
```

---

## Security Common Errors

| Error | HTTP Code | Reason | Fix |
|-------|-----------|--------|-----|
| No token sent | `401` | Authorization header missing | Add `Bearer <token>` header |
| Token expired | `401` | 24 hours passed | Login again to get a fresh token |
| Token tampered | `401` | Signature invalid | Use the original token |
| Wrong role | `403` | ROLE_USER calling ROLE_ADMIN endpoint | Login as admin |
| Username taken | `400` | Already registered | Use a different username |
| Wrong password | `401` | BCrypt mismatch | Check your password |

---

## Full Request Trace (Admin creates product)

```
Request: POST /api/products
Header:  Authorization: Bearer eyJhbG...
Body:    { "name": "iPad", "price": 599, ... }

FILE 1: JwtFilter.java
  → reads "eyJhbG..." from header
  → calls JwtUtil.extractUsername() → "admin"
  → calls UserDetailsServiceImpl.loadUserByUsername("admin") → loads from DB
  → calls JwtUtil.isTokenValid() → true
  → creates UsernamePasswordAuthenticationToken
  → sets it in SecurityContextHolder

FILE 2: SecurityConfig.java
  → request matches: POST /api/products/**
  → rule: hasAuthority("ROLE_ADMIN")
  → checks SecurityContextHolder → user has ROLE_ADMIN ✅
  → allows request through

FILE 3: ProductController.java
  → createProduct() method runs
  → calls ProductServiceImpl

FILE 4: ProductServiceImpl.java
  → saves to DB → returns new Product

Response: 201 Created { "success": true, "data": { "id": 11, "name": "iPad" } }
```

---

# PART 3 — MISSING INTERVIEW TOPICS

> Topics NOT in your project — know these to be 100% ready

---

## Gap Summary Table

| # | Topic | Interview Frequency |
|---|-------|--------------------|
| 1 | Bean Lifecycle (`@PostConstruct`, `@PreDestroy`) | ⭐⭐⭐⭐⭐ |
| 2 | Pagination & Sorting | ⭐⭐⭐⭐⭐ |
| 3 | CORS Configuration | ⭐⭐⭐⭐ |
| 4 | `@Async` / Async Processing | ⭐⭐⭐⭐ |
| 5 | RestTemplate / WebClient / Feign | ⭐⭐⭐⭐⭐ |
| 6 | Caching (`@Cacheable`, `@CacheEvict`) | ⭐⭐⭐⭐ |
| 7 | JPA Auditing | ⭐⭐⭐⭐ |
| 8 | Filters & Interceptors | ⭐⭐⭐⭐ |
| 9 | Test Slices (`@WebMvcTest`, `@DataJpaTest`) | ⭐⭐⭐⭐ |
| 10 | `@Transactional` deep dive (propagation, isolation) | ⭐⭐⭐⭐⭐ |
| 11 | Lazy vs Eager Loading + N+1 Problem | ⭐⭐⭐⭐⭐ |
| 12 | Flyway / Liquibase DB Migrations | ⭐⭐⭐⭐ |
| 13 | Circuit Breaker / Resilience4j | ⭐⭐⭐ |
| 14 | Spring Boot Auto-Configuration internals | ⭐⭐⭐⭐ |
| 15 | Refresh Tokens & Token Blacklisting | ⭐⭐⭐⭐ |
| 16 | DTO Mapping (MapStruct) | ⭐⭐⭐⭐ |
| 17 | Microservices concepts | ⭐⭐⭐⭐ |

---

## 1 — Bean Lifecycle

> One of the top 5 most asked Spring interview questions. Know the exact order.

### Full Lifecycle Order

```
1.  Spring creates bean instance (calls constructor)
2.  Spring injects dependencies (@Autowired fields/constructor args)
3.  BeanNameAware.setBeanName() — if bean implements it
4.  BeanFactoryAware.setBeanFactory() — if bean implements it
5.  ApplicationContextAware.setApplicationContext() — if bean implements it
6.  BeanPostProcessor.postProcessBeforeInitialization()
7.  @PostConstruct method runs  ← YOUR INIT LOGIC HERE
8.  InitializingBean.afterPropertiesSet() — if bean implements it
9.  @Bean(initMethod = "customInit") — if declared
10. BeanPostProcessor.postProcessAfterInitialization()
11. ✅ Bean is READY — lives in context
12. ... app runs ...
13. @PreDestroy method runs  ← YOUR CLEANUP LOGIC HERE
14. DisposableBean.destroy() — if bean implements it
15. @Bean(destroyMethod = "customCleanup") — if declared
```

### Code Example

```java
@Component
public class DatabaseConnectionPool {

    private List<Connection> pool;

    public DatabaseConnectionPool() {
        System.out.println("1. Constructor called — object created");
    }

    @PostConstruct
    public void init() {
        // ✅ Safe to use injected dependencies here
        pool = new ArrayList<>();
        System.out.println("2. @PostConstruct — pool initialized");
    }

    @PreDestroy
    public void cleanup() {
        pool.forEach(c -> c.close());
        System.out.println("3. @PreDestroy — pool closed");
    }
}
```

### Interview Q&A

| Question | Answer |
|----------|--------|
| Why can't you do initialization in the constructor? | Dependencies aren't injected yet when the constructor runs |
| `@PostConstruct` vs `CommandLineRunner`? | `@PostConstruct` runs per-bean after injection; `CommandLineRunner` runs once after full context is ready |
| When does `@PreDestroy` NOT run? | On prototype-scoped beans and on `System.exit()` without shutdown hook |

---

## 2 — Pagination & Sorting

> Any real app needs this. Almost always asked if JPA is discussed.

```java
// Controller with manual params
@GetMapping("/products")
public Page<Product> getProducts(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(defaultValue = "name") String sortBy,
    @RequestParam(defaultValue = "asc") String direction) {

    Sort sort = direction.equalsIgnoreCase("asc")
        ? Sort.by(sortBy).ascending()
        : Sort.by(sortBy).descending();

    Pageable pageable = PageRequest.of(page, size, sort);
    return productRepository.findAll(pageable);
}

// Or use @PageableDefault (simpler)
@GetMapping("/products")
public Page<Product> getProducts(
    @PageableDefault(size = 10, sort = "name", direction = Sort.Direction.ASC)
    Pageable pageable) {
    return productRepository.findAll(pageable);
}
```

**What `Page<T>` returns:**
```json
{
  "content": [ {...}, {...} ],
  "totalElements": 150,
  "totalPages": 15,
  "number": 0,
  "size": 10,
  "first": true,
  "last": false
}
```

**URL:** `GET /products?page=0&size=10&sort=price,desc`

| Question | Answer |
|----------|--------|
| `Page` vs `Slice`? | `Page` runs a COUNT query for total; `Slice` just knows if next page exists — faster for infinite scroll |

---

## 3 — CORS Configuration

> Every frontend-backend split project needs this.

**What is CORS?** Browser security blocks a React app on `localhost:3000` from calling your API on `localhost:8082`. CORS tells the browser "it's okay."

```java
// WAY 1: Per-controller
@CrossOrigin(origins = "http://localhost:3000")
@RestController
public class ProductController { ... }

// WAY 2: Global (recommended)
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("https://myapp.com", "http://localhost:3000")
            .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
            .allowedHeaders("*")
            .allowCredentials(true)
            .maxAge(3600);
    }
}

// WAY 3: With Spring Security (must also enable in security chain)
http.cors(cors -> cors.configurationSource(request -> {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("http://localhost:3000"));
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
    config.setAllowedHeaders(List.of("*"));
    return config;
}))
```

| Question | Answer |
|----------|--------|
| What is a preflight request? | Browser sends `OPTIONS` request first to check if the real request is allowed |
| Does `@CrossOrigin(origins = "*")` work with credentials? | ❌ No — wildcard `*` can't be combined with `allowCredentials = true` |
| Postman vs browser CORS? | Postman never enforces CORS; browsers always do |

---

## 4 — `@Async` / Asynchronous Processing

```java
// Step 1: Enable
@SpringBootApplication
@EnableAsync
public class Application { }

// Step 2: Mark method async
@Service
public class EmailService {

    @Async
    public CompletableFuture<String> sendWelcomeEmail(String email) {
        Thread.sleep(3000); // doesn't block the caller
        return CompletableFuture.completedFuture("Email sent to " + email);
    }

    @Async  // fire and forget
    public void sendNotification(String message) { }
}

// Step 3: Custom thread pool (production must-have)
@Bean(name = "emailTaskExecutor")
public Executor emailTaskExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(5);
    executor.setMaxPoolSize(20);
    executor.setQueueCapacity(100);
    executor.setThreadNamePrefix("email-async-");
    executor.initialize();
    return executor;
}

@Async("emailTaskExecutor")
public void sendEmail() { ... }
```

| Question | Answer |
|----------|--------|
| `@Async` doesn't work — why? | 1) `@EnableAsync` missing, 2) self-invocation bypasses proxy, 3) method not public |
| Default thread pool? | `SimpleAsyncTaskExecutor` — creates new thread every call (**not production-safe**) |

---

## 5 — Calling External APIs (RestTemplate / WebClient / OpenFeign)

### RestTemplate (traditional)

```java
@Service
public class PaymentService {

    private final RestTemplate restTemplate;

    public PaymentService(RestTemplateBuilder builder) {
        this.restTemplate = builder
            .connectTimeout(Duration.ofSeconds(5))
            .readTimeout(Duration.ofSeconds(10))
            .build();
    }

    public PaymentResponse chargeCard(PaymentRequest request) {
        return restTemplate.postForObject(
            "https://payment-api.com/charge", request, PaymentResponse.class);
    }

    public ResponseEntity<Product> getWithHeaders(Long id) {
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth("token123");
        HttpEntity<Void> entity = new HttpEntity<>(headers);
        return restTemplate.exchange(
            "https://api.example.com/products/" + id, HttpMethod.GET, entity, Product.class);
    }
}
```

### WebClient (reactive, preferred for new projects)

```java
@Service
public class InventoryService {

    private final WebClient webClient;

    public InventoryService(WebClient.Builder builder) {
        this.webClient = builder.baseUrl("https://inventory-service").build();
    }

    public Mono<Stock> checkStock(Long productId) {
        return webClient.get().uri("/stock/{id}", productId)
            .retrieve().bodyToMono(Stock.class);
    }

    public Stock checkStockSync(Long productId) {
        return checkStock(productId).block(); // synchronous in non-reactive apps
    }
}
```

### OpenFeign (best for microservices)

```java
@SpringBootApplication
@EnableFeignClients
public class Application { }

@FeignClient(name = "inventory-service", url = "http://inventory-service")
public interface InventoryClient {
    @GetMapping("/api/stock/{productId}")
    Stock getStock(@PathVariable Long productId);

    @PostMapping("/api/stock/reserve")
    void reserveStock(@RequestBody ReserveRequest request);
}

// Just inject and use — no implementation needed
@Service
public class OrderService {
    private final InventoryClient inventoryClient; // Spring auto-injects Feign proxy
}
```

| Question | Answer |
|----------|--------|
| `RestTemplate` vs `WebClient`? | `RestTemplate` is synchronous/blocking (deprecated); `WebClient` is reactive/non-blocking |
| When to use Feign? | In microservices — cleaner, declarative, integrates with Eureka for service discovery |

---

## 6 — Caching

```java
@SpringBootApplication
@EnableCaching
public class Application { }

@Service
public class ProductService {

    // Caches result. Next call with same ID returns from cache (no DB hit)
    @Cacheable(value = "products", key = "#id")
    public Product getById(Long id) {
        System.out.println("DB HIT for id=" + id); // only prints once per id
        return productRepository.findById(id).orElseThrow();
    }

    // Removes specific entry from cache after update
    @CacheEvict(value = "products", key = "#id")
    public Product update(Long id, ProductRequest req) {
        return productRepository.save(...);
    }

    // Removes ALL entries in the "products" cache
    @CacheEvict(value = "products", allEntries = true)
    public void clearAllProductCache() { }

    // Updates the cache with the new return value (ALWAYS runs the method)
    @CachePut(value = "products", key = "#product.id")
    public Product save(Product product) {
        return productRepository.save(product);
    }
}
```

**Production (Redis):**
```properties
spring.cache.type=redis
spring.data.redis.host=localhost
spring.data.redis.port=6379
spring.cache.redis.time-to-live=600000
```

| Question | Answer |
|----------|--------|
| Default cache implementation? | `ConcurrentHashMap` — in-memory, no eviction, not production-safe |
| `@Cacheable` vs `@CachePut`? | `@Cacheable` skips method on cache hit; `@CachePut` ALWAYS runs method and updates cache |
| Why doesn't `@Cacheable` work on private methods? | Spring proxy is bypassed — same issue as `@Transactional` and `@Async` |

---

## 7 — JPA Auditing

> Any real-world entity needs `createdAt`/`updatedAt`. Shows you've built production apps.

```java
@SpringBootApplication
@EnableJpaAuditing
public class Application { }

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String updatedBy;
}

@Entity
public class Product extends BaseEntity {
    // createdAt, updatedAt, createdBy, updatedBy auto-managed
}

// Provide current user for @CreatedBy / @LastModifiedBy
@Component
public class AuditorAwareImpl implements AuditorAware<String> {
    @Override
    public Optional<String> getCurrentAuditor() {
        return Optional.ofNullable(SecurityContextHolder.getContext())
            .map(SecurityContext::getAuthentication)
            .filter(Authentication::isAuthenticated)
            .map(Authentication::getName);
    }
}
```

---

## 8 — Filters & Interceptors

> Know the difference — interviewers always test this.

| | Filter (Servlet) | Interceptor (Spring MVC) |
|---|---|---|
| Level | Servlet container level | Spring MVC level |
| Runs before | DispatcherServlet | Controller method |
| Spring context access | Only via `@Autowired` with `@Component` | Full Spring context |
| Works with | All requests including static files | Only DispatcherServlet requests |
| Interface | `OncePerRequestFilter` | `HandlerInterceptor` |
| Use for | Auth token check, CORS, request logging | Role checks, logging, response time |

**Filter Example:**
```java
@Component
@Order(1)
public class RequestLoggingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
            HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        long start = System.currentTimeMillis();
        try {
            filterChain.doFilter(request, response);
        } finally {
            long duration = System.currentTimeMillis() - start;
            System.out.printf("%s %s → %d (%dms)%n",
                request.getMethod(), request.getRequestURI(),
                response.getStatus(), duration);
        }
    }
}
```

**Interceptor Example:**
```java
@Component
public class AuthInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) {
        // return true = proceed; return false = stop
        String role = (String) req.getSession().getAttribute("role");
        if ("ADMIN".equals(role)) return true;
        res.setStatus(403);
        return false;
    }

    @Override
    public void afterCompletion(HttpServletRequest req, HttpServletResponse res,
                                Object handler, Exception ex) {
        // cleanup after response sent
    }
}

// Register interceptor
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AuthInterceptor())
            .addPathPatterns("/api/admin/**")
            .excludePathPatterns("/api/auth/**");
    }
}
```

---

## 9 — Test Slices

> Your project only has `@SpringBootTest`. Know the lightweight alternatives.

### `@WebMvcTest` — Controller only (no DB, no service beans)

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean  // replaces the real bean in context
    private ProductService productService;

    @Test
    void getProduct_returnsOk() throws Exception {
        when(productService.findById(1L)).thenReturn(new Product(1L, "iPhone", 999.0, "Electronics", 50));

        mockMvc.perform(get("/api/products/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.data.name").value("iPhone"));
    }

    @Test
    @WithMockUser(roles = "ADMIN")  // simulate logged-in admin
    void createProduct_asAdmin_returns201() throws Exception {
        mockMvc.perform(post("/api/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""{"name":"iPad","price":599.0,"category":"Electronics","stock":10}"""))
            .andExpect(status().isCreated());
    }
}
```

### `@DataJpaTest` — Repository only (real in-memory DB)

```java
@DataJpaTest
class ProductRepositoryTest {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private TestEntityManager em;

    @Test
    void findByCategory_returnsMatchingProducts() {
        em.persist(new Product(null, "iPhone", 999.0, "Electronics", 10));
        em.persist(new Product(null, "Samsung", 799.0, "Electronics", 5));
        em.persist(new Product(null, "Nike Shoes", 100.0, "Apparel", 20));
        em.flush();

        List<Product> electronics = productRepository.findByCategory("Electronics");
        assertThat(electronics).hasSize(2);
    }
}
```

| Annotation | Loads | Use When |
|---|---|---|
| `@SpringBootTest` | Full context | Full integration test |
| `@WebMvcTest` | Web layer only | Testing controllers (fast) |
| `@DataJpaTest` | JPA/DB layer only | Testing repositories (fast) |
| `@MockBean` | Replaces real bean with mock | Isolating a layer |

---

## 10 — `@Transactional` Deep Dive

> Your project uses `@Transactional` but doesn't cover propagation/isolation — senior-level questions.

### Propagation

```java
// REQUIRED (default) — join existing tx or create new one
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() {
    methodB(); // uses methodA's transaction
}

// REQUIRES_NEW — always create a new tx (suspends outer tx)
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void auditLog() {
    // Even if outer tx rolls back, THIS COMMITS separately
    // Use for: audit logs, notifications that must persist
}

// NESTED — save point inside existing tx
@Transactional(propagation = Propagation.NESTED)
public void riskyOperation() { }
```

### Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-------|-----------|--------------------|--------------|-|
| `READ_UNCOMMITTED` | ✅ possible | ✅ possible | ✅ possible | Fastest |
| `READ_COMMITTED` (most DBs default) | ❌ prevented | ✅ possible | ✅ possible | Good |
| `REPEATABLE_READ` | ❌ prevented | ❌ prevented | ✅ possible | Slower |
| `SERIALIZABLE` | ❌ prevented | ❌ prevented | ❌ prevented | Slowest |

### Self-Invocation Problem (critical!)

```java
@Service
public class OrderService {

    public void processOrder() {
        placeOrder();  // ❌ @Transactional WON'T work! self-call bypasses Spring proxy
    }

    @Transactional
    public void placeOrder() { ... }
}
// Fix: inject the service itself, or split into two classes
```

---

## 11 — Lazy vs Eager Loading & N+1 Problem

> The N+1 problem is a critical real-world JPA pitfall. Almost always asked in senior interviews.

### Loading Types

```java
@ManyToOne(fetch = FetchType.EAGER)  // default — loads related entity immediately (JOIN)
@ManyToOne(fetch = FetchType.LAZY)   // recommended — loads only when accessed
@OneToMany(mappedBy = "order", fetch = FetchType.LAZY)  // OneToMany default is LAZY
```

### N+1 Problem

```java
// 🔴 N+1 — 1 query for all orders + 1 query per order for user
List<Order> orders = orderRepository.findAll(); // SELECT * FROM orders → 100 rows
for (Order o : orders) {
    System.out.println(o.getUser().getName()); // SELECT * FROM users WHERE id=? × 100 times!
}
// Total: 101 queries — very slow!

// ✅ Fix 1: JOIN FETCH in JPQL
@Query("SELECT o FROM Order o JOIN FETCH o.user")
List<Order> findAllWithUser();

// ✅ Fix 2: @EntityGraph
@EntityGraph(attributePaths = {"user", "items"})
List<Order> findAll();

// ✅ Fix 3: DTO projection (most performant)
@Query("SELECT new com.example.dto.OrderSummary(o.id, u.name) FROM Order o JOIN o.user u")
List<OrderSummary> findOrderSummaries();
```

---

## 12 — Database Migrations (Flyway)

> `spring.jpa.hibernate.ddl-auto=create-drop` is for learning only. Production uses Flyway.

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

```sql
-- src/main/resources/db/migration/V1__create_products_table.sql
CREATE TABLE products (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    category VARCHAR(100),
    stock INT DEFAULT 0
);

-- V2__add_product_description.sql
ALTER TABLE products ADD COLUMN description TEXT;
```

```properties
spring.flyway.enabled=true
spring.jpa.hibernate.ddl-auto=validate   # Flyway manages schema, JPA only validates
```

| Question | Answer |
|----------|--------|
| `V1__` vs `R1__` prefix? | `V` = versioned (runs once); `R` = repeatable (runs when checksum changes) |
| What if Flyway detects schema mismatch? | App fails to start — prevents running against wrong schema |
| Flyway vs Liquibase? | Flyway uses SQL files (simpler); Liquibase uses XML/YAML changelogs (more features) |

---

## 13 — Circuit Breaker (Resilience4j)

> Prevents cascade failures in microservices.

```java
@Service
public class PaymentService {

    @CircuitBreaker(name = "payment", fallbackMethod = "paymentFallback")
    @Retry(name = "payment")
    public CompletableFuture<String> chargeCard(PaymentRequest request) {
        return CompletableFuture.supplyAsync(() -> externalApi.charge(request));
    }

    public CompletableFuture<String> paymentFallback(PaymentRequest request, Throwable t) {
        return CompletableFuture.completedFuture("PAYMENT_DEFERRED");
    }
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      payment:
        sliding-window-size: 10
        failure-rate-threshold: 50       # open if 50% fail
        wait-duration-in-open-state: 10s
  retry:
    instances:
      payment:
        max-attempts: 3
        wait-duration: 1s
```

**Circuit States:**
```
CLOSED → (failure rate ≥ threshold) → OPEN → (wait duration) → HALF-OPEN → test call succeeds → CLOSED
                                                                           → test call fails   → OPEN
```

---

## 14 — Spring Boot Auto-Configuration Internals

> "How does Spring Boot work under the hood" — always asked.

```
@SpringBootApplication
├── @EnableAutoConfiguration
│   └── Reads: META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
│       └── ~160 auto-configuration classes listed there
│           Each class annotated with @Conditional annotations
│           e.g.: @ConditionalOnClass(DataSource.class)
│                 @ConditionalOnMissingBean(DataSource.class)
│                 → Only creates DataSource bean if:
│                   1. DataSource class is on classpath AND
│                   2. No DataSource bean already defined (lets you override)
└── @ComponentScan — scans your code
```

**Key `@Conditional` Annotations:**
```java
@ConditionalOnClass(DataSource.class)         // Only if class exists on classpath
@ConditionalOnMissingBean(DataSource.class)   // Only if NO bean of this type exists
@ConditionalOnProperty(name = "spring.cache.type", havingValue = "redis")
@Profile("production")
```

**How to debug:**
```properties
debug=true   # prints conditions report at startup
# Or: GET /actuator/conditions
```

| Question | Answer |
|----------|--------|
| Why don't we need `@EnableJpaRepositories`? | `JpaRepositoriesAutoConfiguration` is auto-configured when Spring Data JPA is on classpath |
| How to override auto-configuration? | Define your own `@Bean` of the same type — `@ConditionalOnMissingBean` skips the default |
| What is a Starter? | Curated group of dependencies (e.g. `spring-boot-starter-web` = spring-webmvc + tomcat + jackson) |
| What does `@SpringBootApplication` expand to? | `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` |

---

## 15 — Refresh Tokens & JWT Best Practices

### Refresh Token Flow

```
1. LOGIN: Client sends credentials
   Server returns: { access_token (15min), refresh_token (7 days) }

2. API CALL: Client sends access_token in Authorization header

3. ACCESS TOKEN EXPIRED:
   Client sends refresh_token to POST /auth/refresh
   Server validates → issues new access_token (user stays logged in)

4. LOGOUT:
   Client deletes tokens locally
   Server blacklists refresh_token (Redis/DB lookup on each request)
```

```java
@PostMapping("/auth/refresh")
public AuthResponse refresh(@RequestBody RefreshRequest request) {
    String refreshToken = request.getRefreshToken();

    if (!jwtUtil.isTokenValid(refreshToken))
        throw new UnauthorizedException("Invalid refresh token");

    if (tokenBlacklist.contains(refreshToken))
        throw new UnauthorizedException("Token revoked");

    String username = jwtUtil.extractUsername(refreshToken);
    UserDetails user = userDetailsService.loadUserByUsername(username);
    String newAccessToken = jwtUtil.generateToken(user);

    return new AuthResponse(newAccessToken, refreshToken);
}
```

| Question | Answer |
|----------|--------|
| JWT structure? | `header.payload.signature` — base64 encoded, NOT encrypted |
| Why not store JWT in localStorage? | Vulnerable to XSS; prefer `httpOnly` cookie |
| When to use short vs long expiry? | Access token: 15min; Refresh token: 7-30 days |

---

## 16 — DTO Mapping (MapStruct)

**Why DTOs?**
- Avoid exposing DB entities (passwords, internal IDs) in API responses
- API contract stays stable even if DB schema changes

```java
// Entity — never expose directly
@Entity
public class User {
    private Long id;
    private String username;
    private String password;      // ← NEVER expose this!
    private String email;
    private Role role;
}

// Response DTO
public record UserResponse(Long id, String username, String email, String role) {}

// MapStruct mapper (compile-time code generation — zero runtime overhead)
@Mapper(componentModel = "spring")
public interface UserMapper {

    @Mapping(source = "role", target = "roleName")
    @Mapping(target = "password", ignore = true)  // never map passwords
    UserResponse toResponse(User user);

    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    User toEntity(CreateUserRequest request);

    List<UserResponse> toResponseList(List<User> users);
}
```

---

## 17 — Microservices Concepts

> Even if you're not building microservices, interviewers test your awareness.

```
Client
  │
  ▼
API Gateway (Spring Cloud Gateway)
  │  ← single entry point, auth, rate limiting, routing
  ├────────────────────────────────────────┐
  ▼                                        ▼
Product Service                        Order Service
  │                                        │
  └──────────── communicate via ───────────┘
                HTTP (Feign/RestTemplate) or Kafka/RabbitMQ
```

| Component | Library | Purpose |
|-----------|---------|---------|
| **Service Registry** | Spring Cloud Eureka | Services register by name, not IP |
| **API Gateway** | Spring Cloud Gateway | Single entry point, routing, auth |
| **Load Balancing** | Spring Cloud LoadBalancer | Distribute calls across instances |
| **Config Server** | Spring Cloud Config | Centralized config for all services |
| **Circuit Breaker** | Resilience4j | Prevent cascade failures |
| **Distributed Tracing** | Micrometer + Zipkin | Trace a request across services |
| **Message Broker** | Kafka / RabbitMQ | Async event-driven communication |

**Eureka registration:**
```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
spring:
  application:
    name: product-service   # registered under this name
```

**Feign + Eureka (auto-discovers service IP):**
```java
@FeignClient(name = "order-service")  // name from Eureka registry
public interface OrderClient {
    @GetMapping("/api/orders/{id}")
    Order getOrder(@PathVariable Long id);
}
```

---

## 18 — Spring Boot Internals Quick Questions

| Question | Answer |
|----------|--------|
| What does `@SpringBootApplication` expand to? | `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` |
| What is an embedded server? | Tomcat/Jetty packaged inside the JAR — no external server install needed |
| How to change from Tomcat to Jetty? | Exclude `spring-boot-starter-tomcat`, add `spring-boot-starter-jetty` |
| What is a fat JAR? | JAR containing all dependencies — runnable with `java -jar app.jar` |
| `BeanFactory` vs `ApplicationContext`? | `BeanFactory` is basic DI; `ApplicationContext` extends it with events, i18n, AOP |
| What is `ContextRefreshedEvent`? | Fired when ApplicationContext is initialized or refreshed |
| Singleton vs Prototype? | Singleton = one per ApplicationContext; Prototype = new instance every injection |
| `@Lazy` annotation? | Delays bean creation until first use (instead of at startup) |
| `@Component` vs `@Bean`? | `@Component` on your own classes; `@Bean` in `@Configuration` for third-party classes |

---

## 19 — JPA Advanced Quick Reference

| Question | Answer |
|----------|--------|
| `@OneToMany` default fetch? | `LAZY` |
| `@ManyToOne` default fetch? | `EAGER` |
| `cascade = CascadeType.ALL`? | Propagates persist/merge/remove to child entities |
| `orphanRemoval = true`? | Deletes child entity when removed from parent's collection |
| `@Version`? | Optimistic locking — prevents lost updates in concurrent edits |
| JPQL vs SQL? | JPQL uses entity class names and fields; SQL uses table and column names |
| `findById` vs `getReferenceById`? | `findById` = SELECT now; `getReferenceById` = proxy, SELECT on first access |
| `@Embeddable`? | Maps value object (no own table, no ID) into parent entity's table |
| What is persistence context? | First-level cache (within one transaction) — same ID = same object reference |

---

## 20 — Security Advanced Quick Reference

| Question | Answer |
|----------|--------|
| JWT structure? | `header.payload.signature` — base64 encoded, not encrypted |
| `hasRole('ADMIN')` vs `hasAuthority('ROLE_ADMIN')`? | `hasRole` auto-prefixes `ROLE_`; `hasAuthority` uses exact string |
| What is `SecurityContextHolder`? | Thread-local store for the currently authenticated user |
| `OncePerRequestFilter` purpose? | Ensures the filter runs only once per request (not on forwards/includes) |
| CSRF — when to disable? | For stateless REST APIs using JWT — no session/cookies → CSRF irrelevant |
| Why BCrypt generates different hashes? | Includes random **salt** in the hash; `matches()` extracts and reuses the salt |
| `@PreAuthorize` vs URL-level security? | `@PreAuthorize` is method-level (finer control); URL-level is in SecurityConfig |

---

## Quick Concept Lookup

| I want to understand... | Go to file |
|-------------------------|------------|
| How Spring creates objects | `DIConceptsDemo.java` |
| How to call the DB without SQL | `ProductRepository.java` |
| How to build a REST endpoint | `ProductController.java` |
| How to validate request input | `ProductRequest.java` + `GlobalExceptionHandler.java` |
| How to handle errors in one place | `GlobalExceptionHandler.java` |
| How to add logging without touching every method | `LoggingAspect.java` |
| How transactions work | `OrderServiceImpl.java` → `placeOrder()` |
| How to decouple side effects | `OrderPlacedEvent.java` + `OrderEventListener.java` |
| How to run code at startup | `AppConfig.java` → `seedDemoData()` |
| How to run code on a schedule | `ReportScheduler.java` |
| How to read from application.properties | `AppProperties.java` + `InfoController.java` |
| How to have different config per environment | `application-dev/prod.properties` |
| How to write a unit test | `ProductServiceTest.java` |
| How to write an integration test | `ProductControllerIntegrationTest.java` |
| How JWT auth works end to end | `JwtFilter.java` → `JwtUtil.java` → `SecurityConfig.java` |
| How roles are enforced | `SecurityConfig.java` + `AdminController.java` |

---

## ✅ Final Checklist — Can You Answer Without Notes?

### Core Spring
- [ ] Explain IoC and why it matters
- [ ] 3 injection types — which is preferred and why?
- [ ] `@Primary` vs `@Qualifier`
- [ ] Bean lifecycle — what order does `@PostConstruct` run in?
- [ ] Singleton vs Prototype scope

### JPA & Database
- [ ] Explain the N+1 problem and 3 ways to fix it
- [ ] `@Transactional` propagation types — REQUIRED vs REQUIRES_NEW
- [ ] Lazy vs Eager loading — which is default for `@ManyToOne`?
- [ ] Why use Flyway instead of `ddl-auto=update` in production?
- [ ] What is optimistic locking and when to use `@Version`?

### REST & Web
- [ ] How does `@Valid` work? What exception is thrown on failure?
- [ ] How does `@ControllerAdvice` intercept exceptions?
- [ ] What is CORS and when does it happen?
- [ ] `@WebMvcTest` vs `@SpringBootTest` — when to use each?
- [ ] `@Cacheable` vs `@CachePut` — what's the difference?

### Security
- [ ] Explain JWT flow end to end (register → login → use token → expiry)
- [ ] What's the difference between 401 and 403?
- [ ] Why disable CSRF for REST APIs?
- [ ] `@PreAuthorize` vs URL patterns in SecurityConfig
- [ ] `hasRole('ADMIN')` vs `hasAuthority('ROLE_ADMIN')`?

### Advanced
- [ ] How does auto-configuration work internally?
- [ ] What is a Circuit Breaker? When does it open/close?
- [ ] Explain the Filter → Interceptor → Controller request flow
- [ ] Why does `@Async` / `@Transactional` / `@Cacheable` NOT work on private methods or self-calls?
- [ ] `RestTemplate` vs `WebClient` vs `OpenFeign` — when to use which?
