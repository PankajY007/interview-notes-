# 05 — Spring Boot (Interview Ready)

> **Level:** 3-4 Years Experience | **Priority:** 🔴 Critical  
> Covers: IoC, DI, Auto-Configuration, Bean Lifecycle, AOP, Profiles, Actuator, Security

---

## 📌 Table of Contents

1. [Spring Core Concepts](#1-spring-core-concepts) — IoC, Spring vs Spring Boot
2. [Dependency Injection (DI)](#2-dependency-injection) — 3 Types, @Qualifier, @Primary, @Value
3. [Bean Lifecycle & Scopes](#3-bean-lifecycle--scopes) — @Bean vs @Component
4. [Spring Boot Auto-Configuration](#4-spring-boot-auto-configuration) — Starters, @Conditional
5. [Configuration & Profiles](#5-configuration--profiles) — @ConfigurationProperties, Profiles
6. [AOP (Aspect-Oriented Programming)](#6-aop)
7. [@Transactional Deep Dive](#7-transactional-deep-dive) — Propagation, Self-Invocation Pitfall
8. [@Async & @Scheduled](#8-async--scheduled)
9. [Application Events](#9-application-events)
10. [Spring Security Basics](#10-spring-security-basics) — JWT Flow
11. [Actuator & Monitoring](#11-actuator--monitoring)
12. [Exception Handling](#12-exception-handling)
13. [Testing in Spring Boot](#13-testing-in-spring-boot)
14. [Spring Boot Startup Process](#14-spring-boot-startup-process)
15. [Interview Questions & Answers](#15-interview-questions--answers) — 17 Q&As

---

## 1. Spring Core Concepts

### IoC (Inversion of Control)
Traditional: **You** create objects.  
Spring: The **container** creates and manages objects (beans).

```java
// Without IoC — tight coupling
public class OrderService {
    private PaymentService paymentService = new PaymentService(); // YOU create
}

// With IoC — loose coupling
@Service
public class OrderService {
    private final PaymentService paymentService; // Spring INJECTS

    @Autowired // constructor injection
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

### Spring vs Spring Boot

| Feature | Spring Framework | Spring Boot |
|---------|-----------------|-------------|
| Configuration | Manual XML/Java Config | Auto-Configuration |
| Server | Need external Tomcat | Embedded Tomcat/Jetty |
| Setup time | Hours | Minutes |
| Dependency mgmt | Manual | Starters |
| Production-ready | Manual setup | Actuator, health checks |

---

## 2. Dependency Injection

### 3 Types of Injection

```java
// 1. Constructor Injection ✅ RECOMMENDED
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;

    // @Autowired optional for single constructor (Spring 4.3+)
    public OrderService(PaymentService paymentService, InventoryService inventoryService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
    }
}
// Why preferred: immutable, testable, required deps can't be null

// 2. Setter Injection — for optional dependencies
@Service
public class NotificationService {
    private EmailService emailService;

    @Autowired(required = false)
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}

// 3. Field Injection ❌ AVOID in production
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository; // hard to test, hides dependencies
}
```

### @Qualifier and @Primary

```java
// When multiple beans of same type exist
public interface NotificationSender {
    void send(String message);
}

@Component("emailSender")
public class EmailSender implements NotificationSender { ... }

@Component("smsSender")
@Primary // default when no qualifier specified
public class SmsSender implements NotificationSender { ... }

@Service
public class AlertService {
    // Option 1: @Qualifier
    public AlertService(@Qualifier("emailSender") NotificationSender sender) { }

    // Option 2: @Primary will be used if no qualifier
}
```

### Stereotype Annotations

| Annotation | Purpose | Special Behavior |
|------------|---------|-----------------|
| `@Component` | Generic bean | Base annotation |
| `@Service` | Business logic | Semantic clarity |
| `@Repository` | Data access | Exception translation (SQL → Spring exceptions) |
| `@Controller` | MVC controller | Returns view name |
| `@RestController` | REST API controller | `@Controller` + `@ResponseBody` |
| `@Configuration` | Java-based config | Beans defined via `@Bean` methods |

### @Value — Injecting Properties

```java
@Component
public class AppConfig {
    @Value("${app.name:MyApp}")           // from properties, with default
    private String appName;

    @Value("${server.port}")               // no default — fails if missing
    private int port;

    @Value("#{systemProperties['java.version']}")  // SpEL expression
    private String javaVersion;

    @Value("${app.features:feature1,feature2}")  // comma-separated → List
    private List<String> features;

    @Value("${app.enabled:true}")          // boolean
    private boolean enabled;
}

// ⚠️ @Value vs @ConfigurationProperties:
// @Value — good for 1-2 simple properties
// @ConfigurationProperties — better for groups of related properties (type-safe, validated)
```

---

## 3. Bean Lifecycle & Scopes

### Bean Lifecycle

```
1. Spring Container starts
2. Bean Definition loaded (from annotations/XML/Java config)
3. Bean instantiated (constructor called)
4. Dependencies injected (@Autowired)
5. @PostConstruct method called
6. InitializingBean.afterPropertiesSet() called
7. Custom init-method called
8. ═══ BEAN IS READY TO USE ═══
9. @PreDestroy method called (on shutdown)
10. DisposableBean.destroy() called
11. Custom destroy-method called
```

```java
@Component
public class DatabaseInitializer {

    @PostConstruct
    public void init() {
        // runs after dependency injection
        // load initial data, validate config, etc.
        log.info("Database initialized");
    }

    @PreDestroy
    public void cleanup() {
        // runs before bean destruction
        // close connections, flush caches
        log.info("Cleaning up resources");
    }
}
```

### Bean Scopes

| Scope | Description | Usage |
|-------|-------------|-------|
| `singleton` | One instance per container (DEFAULT) | Stateless services |
| `prototype` | New instance every time requested | Stateful beans |
| `request` | One per HTTP request | Web apps |
| `session` | One per HTTP session | User-specific data |
| `application` | One per ServletContext | App-wide data |

```java
@Component
@Scope("prototype")
public class ShoppingCart {
    private List<Item> items = new ArrayList<>();
}

// ⚠️ Pitfall: Injecting prototype into singleton
@Service // singleton
public class OrderService {
    @Autowired
    private ShoppingCart cart; // SAME cart for all requests! Bug!

    // ✅ Fix: Use ObjectFactory or @Lookup
    @Autowired
    private ObjectFactory<ShoppingCart> cartFactory;

    public void createOrder() {
        ShoppingCart cart = cartFactory.getObject(); // new instance each time
    }
}
```

### @Bean vs @Component

```java
// @Component — class-level, auto-detected by component scanning
@Component
public class EmailService {
    // Spring auto-creates this bean
}

// @Bean — method-level, defined inside @Configuration class
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate(); // YOU control creation
    }

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper()
            .registerModule(new JavaTimeModule())
            .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    }
}
```

| | `@Component` | `@Bean` |
|---|---|---|
| Where | On the class itself | On a method in `@Configuration` |
| Auto-detected | ✅ By component scan | ❌ Must be in a config class |
| Use for | Your own classes | Third-party library classes |
| Customization | Limited | Full control over instantiation |
| Example | `@Service`, `@Repository` | `RestTemplate`, `ObjectMapper`, `DataSource` |
```

---

## 4. Spring Boot Auto-Configuration

### How it works
```
1. @SpringBootApplication = @Configuration + @EnableAutoConfiguration + @ComponentScan

2. Spring Boot reads META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
   (previously META-INF/spring.factories)

3. Each auto-config class has @ConditionalOn... annotations:
   @ConditionalOnClass — class exists on classpath
   @ConditionalOnBean — bean exists in context
   @ConditionalOnProperty — property is set
   @ConditionalOnMissingBean — no existing bean of this type

4. If conditions match → auto-configuration applies
   If you define your own bean → auto-config backs off (@ConditionalOnMissingBean)
```

### Starters
```xml
<!-- Adding a starter auto-configures everything -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- Brings: Tomcat, Spring MVC, Jackson, Validation -->
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <!-- Brings: Hibernate, Spring Data JPA, HikariCP connection pool -->
</dependency>
```

### Custom Auto-Configuration
```java
@Configuration
@ConditionalOnClass(DataSource.class)
@ConditionalOnProperty(name = "app.cache.enabled", havingValue = "true")
public class CacheAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("users", "orders");
    }
}
```

---

## 5. Configuration & Profiles

### application.yml / application.properties

```yaml
# application.yml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: ${DB_USER:postgres}     # env variable with default
    password: ${DB_PASS:password}
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

app:
  pagination:
    default-size: 20
    max-size: 100
```

### @ConfigurationProperties (Type-safe)

```java
@Component
@ConfigurationProperties(prefix = "app.pagination")
@Validated
public class PaginationConfig {
    @Min(1) @Max(50)
    private int defaultSize = 20;

    @Max(500)
    private int maxSize = 100;

    // getters & setters
}

// Usage
@Service
public class ProductService {
    private final PaginationConfig config;

    public ProductService(PaginationConfig config) {
        this.config = config;
    }

    public Page<Product> getProducts(int page) {
        int size = Math.min(requestedSize, config.getMaxSize());
        return repo.findAll(PageRequest.of(page, size));
    }
}
```

### Profiles

```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
  jpa:
    show-sql: true

# application-prod.yml
spring:
  datasource:
    url: jdbc:postgresql://prod-server:5432/mydb
  jpa:
    show-sql: false
```

```java
// Activate: java -jar app.jar --spring.profiles.active=prod

@Profile("dev")
@Configuration
public class DevConfig {
    @Bean
    public DataLoader dataLoader() {
        return new MockDataLoader(); // only in dev
    }
}
```

---

## 6. AOP

### Common Use Cases: Logging, Security, Transaction, Caching

```java
@Aspect
@Component
public class LoggingAspect {

    // Before advice
    @Before("execution(* com.app.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        log.info("Entering: {}.{}()",
            joinPoint.getTarget().getClass().getSimpleName(),
            joinPoint.getSignature().getName());
    }

    // Around advice (most powerful)
    @Around("@annotation(Timed)") // custom annotation
    public Object measureTime(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            Object result = pjp.proceed(); // execute method
            return result;
        } finally {
            long elapsed = System.currentTimeMillis() - start;
            log.info("{} took {} ms", pjp.getSignature().getName(), elapsed);
        }
    }

    // After throwing
    @AfterThrowing(pointcut = "execution(* com.app.service.*.*(..))", throwing = "ex")
    public void logException(JoinPoint jp, Exception ex) {
        log.error("Exception in {}: {}", jp.getSignature().getName(), ex.getMessage());
    }
}

// Custom annotation
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Timed { }

// Usage
@Service
public class OrderService {
    @Timed
    @Transactional
    public Order createOrder(OrderRequest request) { ... }
}
```

### AOP Terminology
| Term | Meaning |
|------|---------|
| **Aspect** | Cross-cutting concern (logging, security) |
| **Join Point** | Method execution point |
| **Advice** | Action taken (Before, After, Around) |
| **Pointcut** | Expression matching join points |
| **Weaving** | Linking aspects with target objects |

---

## 7. @Transactional Deep Dive

### Propagation Types

| Propagation | Behavior |
|-------------|----------|
| `REQUIRED` (default) | Join existing TX, or create new if none |
| `REQUIRES_NEW` | Always create new TX (suspends existing) |
| `MANDATORY` | Must run within existing TX, throws if none |
| `SUPPORTS` | Use existing TX if available, else run without |
| `NOT_SUPPORTED` | Run without TX (suspends existing) |
| `NEVER` | Must NOT run in TX, throws if one exists |
| `NESTED` | Create savepoint within existing TX |

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void logAudit(String action) {
    // Always in its own transaction — commits even if caller rolls back
    auditRepository.save(new AuditLog(action));
}
```

### ⚠️ The Self-Invocation Trap (MOST COMMON BUG)

```java
@Service
public class OrderService {

    @Transactional
    public void createOrder(OrderRequest req) {
        orderRepo.save(new Order(req));
        sendNotification(req);  // ⚠️ Self-invocation — @Transactional IGNORED!
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void sendNotification(OrderRequest req) {
        // This @Transactional has NO effect when called from within same class!
        // Because Spring AOP uses proxies — internal calls bypass the proxy
    }
}
```

**Why?** Spring wraps beans in a **proxy** that handles `@Transactional`. When you call a method within the same class, you're calling `this.method()` which skips the proxy.

**Fixes:**
1. **Extract to separate service** (best)
2. **Inject self:** `@Autowired private OrderService self; self.sendNotification(req);`
3. **Use `ApplicationContext.getBean()`** (less clean)

### Other @Transactional Pitfalls

```java
// ❌ Only works on PUBLIC methods
@Transactional
private void process() { } // IGNORED! Proxy can't intercept private

// ❌ Only rolls back on unchecked exceptions by default
@Transactional
public void process() throws IOException {
    throw new IOException(); // NOT rolled back!
}

// ✅ Fix: specify rollback for checked exceptions
@Transactional(rollbackFor = Exception.class)
public void process() throws IOException { }

// ❌ readOnly doesn't prevent writes — it's a HINT to the DB
@Transactional(readOnly = true)
public void update() {
    repo.save(entity); // Still works! But DB may optimize reads
}
```

---

## 8. @Async & @Scheduled

### @Async — Asynchronous Execution

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean
    public Executor asyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("Async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class EmailService {

    @Async  // runs in a separate thread
    public CompletableFuture<String> sendEmail(String to, String body) {
        // expensive I/O operation
        emailClient.send(to, body);
        return CompletableFuture.completedFuture("Sent");
    }
}

// ⚠️ Same self-invocation trap as @Transactional!
// Calling @Async from within same class → runs synchronously
```

### @Scheduled — Task Scheduling

```java
@Configuration
@EnableScheduling
public class ScheduleConfig { }

@Component
public class ScheduledTasks {

    @Scheduled(fixedRate = 60000)       // every 60 seconds
    public void syncData() {
        log.info("Syncing data...");
    }

    @Scheduled(fixedDelay = 30000)      // 30s after previous COMPLETES
    public void processQueue() {
        log.info("Processing queue...");
    }

    @Scheduled(cron = "0 0 2 * * ?")   // daily at 2 AM
    public void dailyCleanup() {
        log.info("Running daily cleanup");
    }

    @Scheduled(cron = "0 */15 * * * ?") // every 15 minutes
    public void healthCheck() { }
}
```

| | `fixedRate` | `fixedDelay` |
|---|---|---|
| Timer starts | When previous execution **starts** | When previous execution **ends** |
| Overlap possible? | ✅ Yes (if task takes longer than rate) | ❌ No |
| Use for | Periodic polling | Sequential processing |

---

## 9. Application Events

```java
// Custom event
public class OrderCreatedEvent {
    private final Long orderId;
    private final String customerEmail;

    public OrderCreatedEvent(Long orderId, String customerEmail) {
        this.orderId = orderId;
        this.customerEmail = customerEmail;
    }
    // getters
}

// Publishing
@Service
public class OrderService {
    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @Transactional
    public Order createOrder(OrderRequest req) {
        Order order = orderRepo.save(new Order(req));
        eventPublisher.publishEvent(new OrderCreatedEvent(order.getId(), req.getEmail()));
        return order;
    }
}

// Listening
@Component
public class OrderEventListener {

    @EventListener
    public void handleOrderCreated(OrderCreatedEvent event) {
        log.info("Order created: {}", event.getOrderId());
    }

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void sendConfirmation(OrderCreatedEvent event) {
        // Only fires AFTER transaction commits successfully
        emailService.sendConfirmation(event.getCustomerEmail());
    }

    @Async
    @EventListener
    public void asyncHandler(OrderCreatedEvent event) {
        // Runs in a separate thread — doesn't block the publisher
        notificationService.push(event);
    }
}
```

**Why use events?** Decouples services. The `OrderService` doesn't need to know about emails, notifications, or audit logs. New listeners can be added without modifying the publisher.

---

## 10. Spring Security Basics

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### JWT Flow
```
1. Client sends POST /login with credentials
2. Server validates, generates JWT token
3. Server returns JWT in response
4. Client sends JWT in Authorization header: Bearer <token>
5. Server validates JWT on each request (filter)
6. If valid → extract user, set SecurityContext
```

---

## 11. Actuator & Monitoring

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, env, loggers
  endpoint:
    health:
      show-details: always
```

| Endpoint | Purpose |
|----------|---------|
| `/actuator/health` | App health status |
| `/actuator/info` | App info (version, build) |
| `/actuator/metrics` | JVM, HTTP, custom metrics |
| `/actuator/env` | Environment properties |
| `/actuator/loggers` | View/change log levels at runtime |

```java
// Custom health indicator
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        if (isDatabaseUp()) {
            return Health.up().withDetail("database", "PostgreSQL").build();
        }
        return Health.down().withDetail("error", "Cannot connect to DB").build();
    }
}
```

---

## 12. Exception Handling

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse("NOT_FOUND", ex.getMessage(), LocalDateTime.now());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );
        return new ErrorResponse("VALIDATION_FAILED", errors.toString(), LocalDateTime.now());
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex) {
        log.error("Unexpected error", ex);
        return new ErrorResponse("INTERNAL_ERROR", "Something went wrong", LocalDateTime.now());
    }
}

public record ErrorResponse(String code, String message, LocalDateTime timestamp) {}
```

---

## 13. Testing in Spring Boot

```java
// Unit test — no Spring context
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @InjectMocks
    private OrderService orderService;

    @Test
    void shouldCreateOrder() {
        Order order = new Order("item1", 100.0);
        when(orderRepository.save(any(Order.class))).thenReturn(order);

        Order result = orderService.createOrder(new OrderRequest("item1", 100.0));

        assertNotNull(result);
        assertEquals("item1", result.getItem());
        verify(orderRepository, times(1)).save(any());
    }
}

// Integration test — full Spring context
@SpringBootTest
@AutoConfigureMockMvc
class OrderControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void shouldReturnOrderById() throws Exception {
        mockMvc.perform(get("/api/orders/1")
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.item").value("item1"));
    }

    @Test
    void shouldCreateOrder() throws Exception {
        String json = """
                {"item": "laptop", "price": 999.99}
                """;

        mockMvc.perform(post("/api/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(json))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.item").value("laptop"));
    }
}

// Slice tests — partial context
@WebMvcTest(OrderController.class)  // only web layer
@DataJpaTest                         // only JPA layer
```

---

## 14. Spring Boot Startup Process

```
1. main() calls SpringApplication.run()
2. SpringApplication instance created
   → Detects WebApplicationType (SERVLET, REACTIVE, NONE)
   → Loads ApplicationContextInitializers
   → Loads ApplicationListeners
3. Environment prepared (properties, profiles loaded)
4. ApplicationContext created (AnnotationConfigServletWebServerApplicationContext)
5. Bean definitions loaded (@ComponentScan + AutoConfiguration)
6. BeanFactoryPostProcessors run (modify bean definitions)
7. Beans instantiated and dependencies injected
8. BeanPostProcessors run (proxies created: AOP, @Transactional)
9. Embedded server started (Tomcat/Jetty)
10. ApplicationRunner / CommandLineRunner beans executed
11. Application is READY — ApplicationReadyEvent published
```

```java
// Run code at startup
@Component
public class StartupRunner implements CommandLineRunner {
    @Override
    public void run(String... args) {
        log.info("Application started with args: {}", Arrays.toString(args));
        // Load cache, validate config, seed data, etc.
    }
}

// Alternative
@Component
public class AppReadyListener {
    @EventListener(ApplicationReadyEvent.class)
    public void onReady() {
        log.info("Application is fully ready!");
    }
}
```

---

## 15. Interview Questions & Answers

### Q1: What is the difference between `@Component`, `@Service`, `@Repository`, and `@Controller`?
**A:** All are stereotype annotations that register beans. `@Component` is generic. `@Service` is for business logic (no special behavior, just semantic). `@Repository` adds **exception translation** (SQL exceptions → Spring DataAccessException). `@Controller` is for MVC (returns view names). `@RestController` = `@Controller` + `@ResponseBody`.

### Q2: Explain Spring Bean lifecycle.
**A:** Instantiation → DI → `@PostConstruct` → `afterPropertiesSet()` → custom init → **READY** → `@PreDestroy` → `destroy()` → custom destroy. Most commonly, developers use `@PostConstruct` for initialization and `@PreDestroy` for cleanup.

### Q3: What is the difference between `@SpringBootApplication` and `@EnableAutoConfiguration`?
**A:** `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`. It's a convenience annotation. `@EnableAutoConfiguration` tells Spring Boot to automatically configure beans based on classpath dependencies.

### Q4: How does Spring Boot auto-configuration work?
**A:** Spring Boot scans `AutoConfiguration.imports` for auto-config classes. Each class uses conditional annotations (`@ConditionalOnClass`, `@ConditionalOnMissingBean`) to decide whether to configure. If you define your own bean, auto-config backs off.

### Q5: What is the difference between `@RequestParam` and `@PathVariable`?
**A:** `@PathVariable` extracts from URL path: `/users/{id}`. `@RequestParam` extracts from query string: `/users?name=John`. PathVariable for resource identification, RequestParam for filtering/sorting.

### Q6: How do you handle circular dependencies?
**A:** Use `@Lazy` on one constructor parameter, restructure code to break the cycle (extract common logic to a third service), or use setter injection. Best practice: redesign to eliminate circular dependencies.

### Q7: What is `@Transactional` and how does it work?
**A:** Marks a method to run within a database transaction. Spring creates a proxy around the bean. When the method is called, the proxy starts a transaction, executes the method, and commits or rolls back on exception. **⚠️ Only works on public methods** called from outside the class (proxy limitation). Self-invocation within the same class bypasses the proxy.

---

### Q8: `@Bean` vs `@Component` — when to use which?
**A:**
- `@Component` — on YOUR classes. Auto-detected by component scanning.
- `@Bean` — on a METHOD inside `@Configuration` for third-party classes you can't annotate (e.g., `RestTemplate`, `ObjectMapper`, `DataSource`).
- `@Bean` gives you full control over instantiation, constructor arguments, and configuration.

---

### Q9: `@Value` vs `@ConfigurationProperties`?
**A:**
| | `@Value` | `@ConfigurationProperties` |
|---|---|---|
| Granularity | Single property | Group of related properties |
| Type-safe | ❌ String-based | ✅ Type-safe binding |
| Validation | ❌ Manual | ✅ `@Validated` + JSR-303 |
| Relaxed binding | ❌ Exact match | ✅ `my-prop` = `myProp` = `MY_PROP` |
| Best for | 1-2 simple values | Structured configuration |

---

### Q10: Explain the @Transactional self-invocation problem.
**A:** When a `@Transactional` method calls another `@Transactional` method **within the same class**, the second method's transactional behavior is IGNORED. This is because Spring AOP uses **proxies** — calling `this.method()` bypasses the proxy. Fix: extract the method to a separate `@Service`, or inject the bean into itself.

---

### Q11: `@RestController` vs `@Controller`?
**A:** `@Controller` returns a **view name** (resolved by ViewResolver for HTML/JSP). `@RestController` = `@Controller` + `@ResponseBody` — directly serializes the return value to JSON/XML in the HTTP response body. Use `@RestController` for REST APIs, `@Controller` for server-side rendered pages.

---

### Q12: What are Spring Boot starters?
**A:** Starters are dependency descriptors that bundle everything you need for a feature. Example: `spring-boot-starter-web` brings Tomcat, Spring MVC, Jackson, Validation. They simplify Maven/Gradle configuration — one dependency instead of 10+. You can also create custom starters.

---

### Q13: How to handle multiple environments (dev/staging/prod)?
**A:** Use **Spring Profiles**:
1. Create `application-dev.yml`, `application-prod.yml`
2. Activate via `spring.profiles.active=prod` (env var, CLI, or properties)
3. Use `@Profile("dev")` to load beans conditionally
4. Use `${DB_URL}` environment variables for secrets (never in properties files)
5. In production, use external config: config server, Vault, or K8s ConfigMaps

---

### Q14: What is `@Async` and how does it work?
**A:** `@Async` makes a method run in a **separate thread** from a thread pool. Requires `@EnableAsync` on a config class. Can return `void` or `CompletableFuture<T>`. Same proxy limitation as `@Transactional` — self-invocation won't work. Always configure a custom executor (default uses `SimpleAsyncTaskExecutor` which creates unlimited threads).

---

### Q15: How does the Spring Security filter chain work?
**A:**
1. Request enters `DelegatingFilterProxy` → `FilterChainProxy`
2. Matches a `SecurityFilterChain` based on URL
3. Passes through a chain of filters in order (15+ default filters)
4. Key filters: `UsernamePasswordAuthenticationFilter` (form login), `BasicAuthenticationFilter`, `BearerTokenAuthenticationFilter` (JWT)
5. Filter authenticates → sets `SecurityContext` (ThreadLocal)
6. `AuthorizationFilter` checks roles/authorities
7. If all pass → request reaches your controller

---

### Q16: `@SpringBootTest` vs `@WebMvcTest` vs `@DataJpaTest`?
**A:**
| | `@SpringBootTest` | `@WebMvcTest` | `@DataJpaTest` |
|---|---|---|---|
| Context | Full application | Web layer only | JPA layer only |
| Speed | Slowest | Fast | Fast |
| Beans loaded | All | Controllers, filters | Repositories, EntityManager |
| Dependencies | Real or `@MockBean` | Must `@MockBean` services | Embedded H2 DB |
| Use for | Integration tests | Controller tests | Repository tests |

---

### Q17: What happens when a Spring Boot application starts?
**A:** `main()` → `SpringApplication.run()` → detect app type (Servlet/Reactive) → load environment & profiles → create `ApplicationContext` → scan components & auto-config → instantiate beans → inject dependencies → create proxies (AOP) → start embedded server → run `CommandLineRunner`/`ApplicationRunner` → fire `ApplicationReadyEvent`. The entire process is customizable via listeners and initializers.

---

> **Next:** [06_REST_APIs.md](06_REST_APIs.md) →
