# 08 — Microservices & Design Patterns (Interview Ready)

> **Level:** 3-4 Years Experience | **Priority:** 🔴 Critical  
> Covers: SOLID, GoF Design Patterns, Microservices Architecture, Patterns, Communication

---

## 📌 Table of Contents

1. [SOLID Principles](#1-solid-principles)
2. [Design Patterns — Creational](#2-creational-patterns)
3. [Design Patterns — Structural](#3-structural-patterns)
4. [Design Patterns — Behavioral](#4-behavioral-patterns)
5. [Microservices Architecture](#5-microservices-architecture)
6. [Microservices Patterns](#6-microservices-patterns)
7. [Inter-Service Communication](#7-inter-service-communication)
8. [Interview Questions & Answers](#8-interview-questions--answers)

---

## 1. SOLID Principles

### S — Single Responsibility Principle
> A class should have only **one reason to change**.

```java
// ❌ BAD — does everything
class UserService {
    void createUser(User user) { }
    void sendEmail(User user) { }        // email responsibility
    void generateReport(User user) { }    // reporting responsibility
}

// ✅ GOOD — each class has one job
class UserService { void createUser(User user) { } }
class EmailService { void sendEmail(User user) { } }
class ReportService { void generateReport(User user) { } }
```

### O — Open/Closed Principle
> Open for **extension**, closed for **modification**.

```java
// ❌ BAD — must modify class to add new shape
class AreaCalculator {
    double calculate(Object shape) {
        if (shape instanceof Circle c) return Math.PI * c.radius * c.radius;
        if (shape instanceof Rectangle r) return r.width * r.height;
        // Adding Triangle? Must modify this method!
    }
}

// ✅ GOOD — extend without modifying
interface Shape { double area(); }
class Circle implements Shape { double area() { return Math.PI * r * r; } }
class Rectangle implements Shape { double area() { return w * h; } }
class Triangle implements Shape { double area() { return 0.5 * b * h; } } // just add new class
```

### L — Liskov Substitution Principle
> Subtypes must be **substitutable** for their base types without breaking behavior.

```java
// ❌ BAD — Square breaks Rectangle's contract
class Rectangle {
    void setWidth(int w) { this.width = w; }
    void setHeight(int h) { this.height = h; }
}
class Square extends Rectangle {
    void setWidth(int w) { this.width = w; this.height = w; } // violates!
}

// ✅ GOOD — use composition or separate hierarchy
interface Shape { double area(); }
record Rectangle(double width, double height) implements Shape { ... }
record Square(double side) implements Shape { ... }
```

### I — Interface Segregation Principle
> Clients should not be forced to depend on interfaces they don't use.

```java
// ❌ BAD — fat interface
interface Worker {
    void work();
    void eat();    // Robot doesn't eat!
    void sleep();  // Robot doesn't sleep!
}

// ✅ GOOD — segregated interfaces
interface Workable { void work(); }
interface Eatable { void eat(); }
interface Sleepable { void sleep(); }

class Human implements Workable, Eatable, Sleepable { ... }
class Robot implements Workable { ... } // only what it needs
```

### D — Dependency Inversion Principle
> High-level modules should depend on **abstractions**, not concrete implementations.

```java
// ❌ BAD — tightly coupled to MySQL
class OrderService {
    private MySQLOrderRepository repo = new MySQLOrderRepository();
}

// ✅ GOOD — depends on abstraction
class OrderService {
    private final OrderRepository repo; // interface
    OrderService(OrderRepository repo) { this.repo = repo; } // injected
}
// Can swap MySQLOrderRepository, MongoOrderRepository, etc.
```

---

## 2. Creational Patterns

### Singleton
```java
// Thread-safe Singleton using enum (Joshua Bloch recommended)
public enum AppConfig {
    INSTANCE;
    private String dbUrl;
    public String getDbUrl() { return dbUrl; }
}

// Double-checked locking
public class ConnectionPool {
    private static volatile ConnectionPool instance;
    private ConnectionPool() { }
    public static ConnectionPool getInstance() {
        if (instance == null) {
            synchronized (ConnectionPool.class) {
                if (instance == null) instance = new ConnectionPool();
            }
        }
        return instance;
    }
}
// In Spring: All beans are Singleton by default!
```

### Factory Method
```java
public interface Notification { void send(String message); }
class EmailNotification implements Notification { ... }
class SMSNotification implements Notification { ... }
class PushNotification implements Notification { ... }

public class NotificationFactory {
    public static Notification create(String type) {
        return switch (type.toUpperCase()) {
            case "EMAIL" -> new EmailNotification();
            case "SMS" -> new SMSNotification();
            case "PUSH" -> new PushNotification();
            default -> throw new IllegalArgumentException("Unknown type: " + type);
        };
    }
}
```

### Builder
```java
// For objects with many parameters
public class User {
    private final String name;       // required
    private final String email;      // required
    private final int age;           // optional
    private final String phone;      // optional

    private User(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.age = builder.age;
        this.phone = builder.phone;
    }

    public static class Builder {
        private final String name;
        private final String email;
        private int age;
        private String phone;

        public Builder(String name, String email) {
            this.name = name;
            this.email = email;
        }

        public Builder age(int age) { this.age = age; return this; }
        public Builder phone(String phone) { this.phone = phone; return this; }
        public User build() { return new User(this); }
    }
}

User user = new User.Builder("John", "john@email.com")
    .age(30)
    .phone("+1234567890")
    .build();

// With Lombok: @Builder annotation does this automatically
```

---

## 3. Structural Patterns

### Adapter
```java
// Convert one interface to another
interface MediaPlayer { void play(String filename); }
interface AdvancedPlayer { void playVLC(String filename); }

class VLCPlayer implements AdvancedPlayer {
    public void playVLC(String filename) { /* VLC logic */ }
}

class MediaAdapter implements MediaPlayer {
    private AdvancedPlayer advancedPlayer;
    MediaAdapter(AdvancedPlayer player) { this.advancedPlayer = player; }
    public void play(String filename) { advancedPlayer.playVLC(filename); }
}
// Real-world: Adapting legacy mainframe APIs to REST
```

### Decorator
```java
interface Coffee { double cost(); String description(); }

class BasicCoffee implements Coffee {
    public double cost() { return 5.0; }
    public String description() { return "Basic Coffee"; }
}

class MilkDecorator implements Coffee {
    private final Coffee coffee;
    MilkDecorator(Coffee coffee) { this.coffee = coffee; }
    public double cost() { return coffee.cost() + 1.5; }
    public String description() { return coffee.description() + " + Milk"; }
}

Coffee order = new MilkDecorator(new BasicCoffee());
// "Basic Coffee + Milk" = $6.50
// Real-world: Java I/O streams (BufferedReader wraps FileReader)
```

### Proxy
```java
// Control access to an object
interface UserService { User getUser(Long id); }

class UserServiceImpl implements UserService {
    public User getUser(Long id) { return db.findById(id); }
}

class UserServiceProxy implements UserService {
    private UserServiceImpl service;
    private Cache cache;

    public User getUser(Long id) {
        User cached = cache.get(id);
        if (cached != null) return cached;
        User user = service.getUser(id);
        cache.put(id, user);
        return user;
    }
}
// Real-world: Spring AOP uses proxies for @Transactional, @Cacheable
```

---

## 4. Behavioral Patterns

### Strategy
```java
interface PaymentStrategy { void pay(double amount); }
class CreditCardPayment implements PaymentStrategy { ... }
class UPIPayment implements PaymentStrategy { ... }
class WalletPayment implements PaymentStrategy { ... }

class PaymentProcessor {
    private PaymentStrategy strategy;
    void setStrategy(PaymentStrategy strategy) { this.strategy = strategy; }
    void processPayment(double amount) { strategy.pay(amount); }
}

// Usage — swap algorithms at runtime
PaymentProcessor processor = new PaymentProcessor();
processor.setStrategy(new UPIPayment());
processor.processPayment(100.0);
```

### Observer
```java
interface EventListener { void onEvent(String event); }

class EventBus {
    private Map<String, List<EventListener>> listeners = new HashMap<>();

    void subscribe(String eventType, EventListener listener) {
        listeners.computeIfAbsent(eventType, k -> new ArrayList<>()).add(listener);
    }

    void publish(String eventType, String data) {
        listeners.getOrDefault(eventType, List.of())
            .forEach(l -> l.onEvent(data));
    }
}
// Real-world: Spring Application Events, Kafka consumers
```

### Template Method
```java
abstract class DataProcessor {
    // Template method — defines algorithm skeleton
    public final void process() {
        readData();
        transform();
        validate();
        save();
    }

    abstract void readData();
    abstract void transform();

    void validate() { /* default validation */ }
    abstract void save();
}

class CSVProcessor extends DataProcessor {
    void readData() { /* read CSV */ }
    void transform() { /* parse CSV */ }
    void save() { /* save to DB */ }
}
```

---

## 5. Microservices Architecture

### Monolith vs Microservices

| Feature | Monolith | Microservices |
|---------|----------|---------------|
| Deployment | Single unit | Independent per service |
| Scaling | Entire app | Individual services |
| Tech stack | Single | Polyglot possible |
| Complexity | Simple initially | Complex from start |
| Data | Shared DB | DB per service |
| Team | Single team | Small autonomous teams |
| Failure | Whole app down | Isolated failures |

### Microservices Architecture Diagram
```
                    ┌──────────────┐
                    │  API Gateway │
                    │  (Routing,   │
                    │   Auth, Rate │
                    │   Limiting)  │
                    └──────┬───────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
┌───────▼─────┐    ┌──────▼──────┐    ┌──────▼──────┐
│ User Service│    │Order Service│    │Payment Svc  │
│  (Port 8081)│    │ (Port 8082) │    │ (Port 8083) │
└──────┬──────┘    └──────┬──────┘    └──────┬──────┘
       │                  │                  │
  ┌────▼────┐        ┌───▼────┐        ┌───▼────┐
  │ User DB │        │Order DB│        │Pay DB  │
  │(Postgres)│       │(MySQL) │        │(Mongo) │
  └─────────┘        └────────┘        └────────┘

  ┌──────────────────────────────────────────┐
  │        Message Broker (Kafka/RabbitMQ)    │
  │  Event-driven async communication        │
  └──────────────────────────────────────────┘

  ┌──────────────────────────────────────────┐
  │     Service Registry (Eureka/Consul)      │
  │     Config Server (Spring Cloud Config)   │
  └──────────────────────────────────────────┘
```

---

## 6. Microservices Patterns

### API Gateway Pattern
```
Client → API Gateway → Microservices
- Single entry point
- Request routing
- Authentication/Authorization
- Rate limiting
- Load balancing
- Request/Response transformation

Tools: Spring Cloud Gateway, Kong, AWS API Gateway
```

### Circuit Breaker Pattern (Resilience4j)
```java
@Service
public class PaymentService {

    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentService", fallbackMethod = "paymentFallback")
    public PaymentResponse processPayment(PaymentRequest request) {
        return paymentGateway.charge(request); // external call
    }

    public PaymentResponse paymentFallback(PaymentRequest request, Throwable t) {
        log.error("Payment service unavailable: {}", t.getMessage());
        return new PaymentResponse("QUEUED", "Payment queued for retry");
    }
}

// Circuit Breaker States:
// CLOSED → normal operation, monitoring failures
// OPEN → failures exceed threshold, fail fast (no calls to service)
// HALF_OPEN → allows limited requests to test if service recovered
```

### Saga Pattern (Distributed Transactions)
```
// Instead of 2-phase commit, use compensating transactions

// CHOREOGRAPHY — events trigger next step
Order Created → [Payment Service listens] → Payment Done → [Inventory listens] → ...
If Payment fails → Order Service listens → cancels order

// ORCHESTRATION — central coordinator
Saga Orchestrator:
  1. Create Order → if fail → END
  2. Reserve Inventory → if fail → Cancel Order
  3. Process Payment → if fail → Release Inventory, Cancel Order
  4. Ship Order → SUCCESS
```

### CQRS (Command Query Responsibility Segregation)
```
WRITES (Commands) → Write DB (PostgreSQL, normalized)
                     ↓ (events via Kafka)
READS (Queries) → Read DB (Elasticsearch, denormalized, fast)

// Separate models for read vs write
// Write model: normalized, consistent
// Read model: denormalized, optimized for queries
```

### Other Essential Patterns
| Pattern | Purpose |
|---------|---------|
| **Service Discovery** | Services find each other (Eureka, Consul) |
| **Config Server** | Centralized configuration |
| **Distributed Tracing** | Track requests across services (Zipkin, Jaeger) |
| **Bulkhead** | Isolate failures (separate thread pools) |
| **Event Sourcing** | Store events instead of current state |
| **Strangler Fig** | Gradually migrate monolith to microservices |
| **Sidecar** | Deploy helper alongside main service |

---

## 7. Inter-Service Communication

### Synchronous (REST / gRPC)
```java
// RestTemplate (legacy)
RestTemplate restTemplate = new RestTemplate();
User user = restTemplate.getForObject("http://user-service/users/{id}", User.class, userId);

// WebClient (reactive, preferred)
WebClient webClient = WebClient.create("http://user-service");
Mono<User> user = webClient.get()
    .uri("/users/{id}", userId)
    .retrieve()
    .bodyToMono(User.class);

// Feign Client (declarative)
@FeignClient(name = "user-service")
public interface UserClient {
    @GetMapping("/users/{id}")
    User getUser(@PathVariable Long id);
}
```

### Asynchronous (Kafka / RabbitMQ)
```java
// Kafka Producer
@Service
public class OrderEventProducer {
    @Autowired
    private KafkaTemplate<String, OrderEvent> kafkaTemplate;

    public void publishOrderCreated(Order order) {
        OrderEvent event = new OrderEvent("ORDER_CREATED", order);
        kafkaTemplate.send("order-events", order.getId().toString(), event);
    }
}

// Kafka Consumer
@Service
public class InventoryEventConsumer {
    @KafkaListener(topics = "order-events", groupId = "inventory-group")
    public void handleOrderEvent(OrderEvent event) {
        if ("ORDER_CREATED".equals(event.getType())) {
            inventoryService.reserveStock(event.getOrder());
        }
    }
}
```

---

## 8. Interview Questions & Answers

### Q1: Explain SOLID principles with real-world examples.
**A:** *[Cover each principle with the examples from Section 1 above]*

### Q2: What is the difference between Saga and 2-Phase Commit?
**A:** **2PC** — coordinator locks all resources, ensures all-or-nothing (strong consistency, poor availability). **Saga** — sequence of local transactions with compensating actions for rollback (eventual consistency, better availability). Saga is preferred in microservices because 2PC doesn't work well across distributed services.

### Q3: How does a Circuit Breaker work?
**A:** States: **CLOSED** (normal, counting failures) → **OPEN** (failure threshold exceeded, requests fail-fast immediately) → **HALF-OPEN** (after timeout, allows limited requests to test recovery). If test requests succeed → CLOSED. If they fail → OPEN again. Prevents cascading failures.

### Q4: When would you NOT use microservices?
**A:** Small team (<5 developers), simple domain, startup/MVP phase, tight deadlines, when you don't need independent scaling. Microservices add complexity (distributed transactions, network latency, monitoring). Start with a well-structured monolith, split when needed.

### Q5: What is the Strangler Fig pattern?
**A:** Gradually replace parts of a monolith with microservices. Route new requests to the new service, old requests to the monolith. Over time, the monolith "strangles" and dies. Named after strangler fig trees that grow around host trees.

---

> **Next:** [09_SQL_PostgreSQL.md](09_SQL_PostgreSQL.md) →
