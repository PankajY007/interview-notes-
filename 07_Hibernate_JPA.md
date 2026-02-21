# 07 — Hibernate & JPA (Interview Ready)

> **Level:** 3-4 Years Experience | **Priority:** 🔴 Critical  
> Covers: JPA, Entity Mapping, Relationships, N+1 Problem, Caching, Transactions, Query Methods

---

## 📌 Table of Contents

1. [JPA vs Hibernate](#1-jpa-vs-hibernate)
2. [Entity Mapping](#2-entity-mapping)
3. [Relationships](#3-relationships)
4. [Fetch Types — Lazy vs Eager](#4-fetch-types)
5. [N+1 Problem & Solutions](#5-n1-problem)
6. [Spring Data JPA Queries](#6-spring-data-jpa-queries)
7. [Transactions](#7-transactions)
8. [Caching](#8-caching)
9. [Auditing](#9-auditing)
10. [Interview Questions & Answers](#10-interview-questions--answers)

---

## 1. JPA vs Hibernate

| Feature | JPA | Hibernate |
|---------|-----|-----------|
| What | Specification (interface) | Implementation (ORM framework) |
| Annotations | `javax.persistence.*` / `jakarta.persistence.*` | `org.hibernate.*` |
| Query | JPQL | HQL (superset of JPQL) |
| Analogy | JDBC is a spec | MySQL Driver is implementation |

```
JPA (spec) ←── Hibernate (implementation)
            ←── EclipseLink (alternative implementation)
            ←── OpenJPA (alternative implementation)
```

---

## 2. Entity Mapping

```java
@Entity
@Table(name = "employees", uniqueConstraints = {
    @UniqueConstraint(columnNames = {"email"})
})
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // auto-increment
    private Long id;

    @Column(name = "full_name", nullable = false, length = 100)
    private String name;

    @Column(unique = true, nullable = false)
    private String email;

    @Enumerated(EnumType.STRING) // store as string, NOT ordinal
    @Column(nullable = false)
    private Department department;

    @Column(precision = 10, scale = 2)
    private BigDecimal salary;

    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "created_at", updatable = false)
    private Date createdAt;

    @Lob // Large Object — CLOB/BLOB
    private String bio;

    @Transient // NOT persisted to DB
    private int age;

    @Embedded
    private Address address;
}

@Embeddable
public class Address {
    private String street;
    private String city;
    private String zipCode;
}

// Generation Strategies
// IDENTITY — DB auto-increment (MySQL, PostgreSQL)
// SEQUENCE — DB sequence (Oracle, PostgreSQL) — preferred for batch inserts
// TABLE — JPA manages sequence table
// AUTO — JPA picks strategy
```

### ⚠️ Pitfall: `EnumType.ORDINAL` vs `EnumType.STRING`
```java
// ORDINAL (default) — stores 0, 1, 2...
// If you reorder enum values → DATA CORRUPTION
// ALWAYS use EnumType.STRING
@Enumerated(EnumType.STRING) // stores "ADMIN", "USER", etc.
private Role role;
```

---

## 3. Relationships

### One-to-Many / Many-to-One

```java
@Entity
public class Department {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Employee> employees = new ArrayList<>();

    // Helper methods for bidirectional consistency
    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);
    }

    public void removeEmployee(Employee employee) {
        employees.remove(employee);
        employee.setDepartment(null);
    }
}

@Entity
public class Employee {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY) // ALWAYS use LAZY for ManyToOne
    @JoinColumn(name = "department_id", nullable = false)
    private Department department;
}
```

### Many-to-Many

```java
@Entity
public class Student {
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>(); // Use Set to avoid duplicates
}

@Entity
public class Course {
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
}
```

### One-to-One

```java
@Entity
public class User {
    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "profile_id", referencedColumnName = "id")
    private UserProfile profile;
}
```

### Cascade Types
| Type | Meaning |
|------|---------|
| `PERSIST` | Save child when parent is saved |
| `MERGE` | Update child when parent is updated |
| `REMOVE` | Delete child when parent is deleted |
| `REFRESH` | Refresh child when parent is refreshed |
| `DETACH` | Detach child when parent is detached |
| `ALL` | All of the above |

**⚠️ `orphanRemoval = true`** — deletes child when removed from parent's collection (even without deleting parent).

---

## 4. Fetch Types

| Fetch Type | Behavior | Default For |
|-----------|----------|-------------|
| `EAGER` | Load immediately with parent | `@ManyToOne`, `@OneToOne` |
| `LAZY` | Load only when accessed | `@OneToMany`, `@ManyToMany` |

```java
// ⚠️ LazyInitializationException — most common JPA error
@Service
public class EmployeeService {
    public EmployeeDTO getEmployee(Long id) {
        Employee emp = repository.findById(id).orElseThrow();
        // Session is closed after this method returns

        // If controller tries to access lazy collection:
        emp.getDepartment().getName(); // LazyInitializationException!
    }
}

// ✅ Fix 1: Use DTO projection (BEST)
// ✅ Fix 2: JOIN FETCH in query
// ✅ Fix 3: @EntityGraph
// ❌ Fix 4: Open-in-View (spring.jpa.open-in-view=false recommended)
```

---

## 5. N+1 Problem

### The Problem
```java
// 1 query to fetch all departments
List<Department> departments = departmentRepository.findAll();
// SELECT * FROM departments → returns 10 departments

// N queries to fetch employees for EACH department
for (Department dept : departments) {
    dept.getEmployees().size(); // each access triggers:
    // SELECT * FROM employees WHERE department_id = 1
    // SELECT * FROM employees WHERE department_id = 2
    // ... (10 more queries!)
}
// Total: 1 + 10 = 11 queries (N+1)
```

### Solutions

```java
// ✅ Solution 1: JOIN FETCH (JPQL)
@Query("SELECT d FROM Department d JOIN FETCH d.employees")
List<Department> findAllWithEmployees();
// Single query with JOIN

// ✅ Solution 2: @EntityGraph
@EntityGraph(attributePaths = {"employees"})
List<Department> findAll();
// Spring Data adds JOIN FETCH automatically

// ✅ Solution 3: @BatchSize (Hibernate)
@OneToMany(mappedBy = "department")
@BatchSize(size = 20) // loads 20 collections per batch
private List<Employee> employees;
// Instead of N queries, does ceil(N/20) queries

// ✅ Solution 4: DTO Projection (BEST for read-only)
@Query("SELECT new com.app.dto.DeptSummary(d.name, COUNT(e)) " +
       "FROM Department d LEFT JOIN d.employees e GROUP BY d.name")
List<DeptSummary> getDepartmentSummaries();
```

---

## 6. Spring Data JPA Queries

### Derived Query Methods
```java
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    // Auto-generated SQL from method name!
    List<Employee> findByDepartment(String department);
    List<Employee> findByNameContainingIgnoreCase(String name);
    List<Employee> findBySalaryGreaterThan(Double salary);
    List<Employee> findByDepartmentAndSalaryBetween(String dept, Double min, Double max);
    Optional<Employee> findByEmail(String email);
    boolean existsByEmail(String email);
    long countByDepartment(String department);
    List<Employee> findTop5ByOrderBySalaryDesc();
    
    // With pagination
    Page<Employee> findByDepartment(String department, Pageable pageable);
}
```

### JPQL Queries
```java
@Query("SELECT e FROM Employee e WHERE e.salary > :minSalary AND e.department = :dept")
List<Employee> findHighPaidInDept(@Param("dept") String dept, @Param("minSalary") Double salary);

// Update/Delete
@Modifying
@Transactional
@Query("UPDATE Employee e SET e.salary = e.salary * :multiplier WHERE e.department = :dept")
int updateSalaryByDept(@Param("dept") String dept, @Param("multiplier") double multiplier);
```

### Native SQL Queries
```java
@Query(value = "SELECT * FROM employees WHERE department = :dept", nativeQuery = true)
List<Employee> findByDeptNative(@Param("dept") String dept);
```

### Specifications (Dynamic Queries)
```java
public class EmployeeSpecifications {
    public static Specification<Employee> hasDepartment(String dept) {
        return (root, query, cb) -> dept == null ? null : cb.equal(root.get("department"), dept);
    }

    public static Specification<Employee> salaryGreaterThan(Double salary) {
        return (root, query, cb) -> salary == null ? null : cb.greaterThan(root.get("salary"), salary);
    }
}

// Usage — dynamic where clause
List<Employee> results = repository.findAll(
    Specification.where(hasDepartment("IT"))
        .and(salaryGreaterThan(50000.0))
);
```

---

## 7. Transactions

```java
@Service
public class TransferService {

    @Transactional // default: Propagation.REQUIRED, Isolation.DEFAULT
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        Account from = accountRepo.findById(fromId).orElseThrow();
        Account to = accountRepo.findById(toId).orElseThrow();

        from.debit(amount);
        to.credit(amount);

        accountRepo.save(from);
        accountRepo.save(to);
        // If any exception → ALL changes rolled back
    }
}
```

### Propagation Types
| Type | Behavior |
|------|----------|
| `REQUIRED` (default) | Join existing or create new |
| `REQUIRES_NEW` | Always create new (suspend existing) |
| `MANDATORY` | Must have existing (throws if none) |
| `SUPPORTS` | Join if exists, run without if not |
| `NOT_SUPPORTED` | Run without transaction |
| `NEVER` | Throws if transaction exists |

### Isolation Levels
| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|-----------|---------------------|-------------|
| READ_UNCOMMITTED | ✅ Possible | ✅ Possible | ✅ Possible |
| READ_COMMITTED | ❌ Prevented | ✅ Possible | ✅ Possible |
| REPEATABLE_READ | ❌ Prevented | ❌ Prevented | ✅ Possible |
| SERIALIZABLE | ❌ Prevented | ❌ Prevented | ❌ Prevented |

### ⚠️ Common @Transactional Pitfalls

```java
// Pitfall 1: Self-invocation — proxy is bypassed!
@Service
public class OrderService {
    public void processOrder(Order order) {
        saveOrder(order); // ❌ @Transactional IGNORED — same class call
    }

    @Transactional
    public void saveOrder(Order order) { ... }
}
// ✅ Fix: Call from another bean, or use self-injection

// Pitfall 2: Checked exceptions don't trigger rollback by default
@Transactional // only rolls back on RuntimeException
public void process() throws IOException {
    throw new IOException("fail"); // ❌ NOT rolled back!
}

@Transactional(rollbackFor = Exception.class) // ✅ Fix
public void process() throws IOException { ... }

// Pitfall 3: Must be on public methods
@Transactional
private void process() { } // ❌ IGNORED — not public
```

---

## 8. Caching

### First-Level Cache (Session Cache)
- Enabled by default
- Scoped to a single Hibernate `Session` / JPA `EntityManager`
- Same entity queried twice in one session → returns cached instance

### Second-Level Cache (Shared)
```java
// Enable in application.yml
spring:
  jpa:
    properties:
      hibernate:
        cache:
          use_second_level_cache: true
          region.factory_class: org.hibernate.cache.ehcache.EhCacheRegionFactory

@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Product {
    @Id private Long id;
    private String name;
}
```

### Spring Cache Abstraction
```java
@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public Product getProduct(Long id) {
        return repository.findById(id).orElseThrow(); // DB hit only first time
    }

    @CachePut(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        return repository.save(product); // updates cache with new value
    }

    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(Long id) {
        repository.deleteById(id); // removes from cache
    }

    @CacheEvict(value = "products", allEntries = true)
    public void clearCache() { } // clears entire cache
}
```

---

## 9. Auditing

```java
@Configuration
@EnableJpaAuditing
public class JpaConfig { }

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

// All entities extend BaseEntity
@Entity
public class Employee extends BaseEntity {
    @Id @GeneratedValue private Long id;
    private String name;
}

// Provide current user for @CreatedBy/@LastModifiedBy
@Component
public class AuditorAwareImpl implements AuditorAware<String> {
    @Override
    public Optional<String> getCurrentAuditor() {
        return Optional.of(SecurityContextHolder.getContext()
            .getAuthentication().getName());
    }
}
```

---

## 10. Interview Questions & Answers

### Q1: What is the N+1 problem? How do you solve it?
**A:** When loading a parent entity with a lazy collection, Hibernate fires 1 query for parents + N queries for each parent's children. Solutions: **JOIN FETCH** (JPQL), **@EntityGraph**, **@BatchSize**, or **DTO projections**. Best practice: always check Hibernate SQL logs during development.

### Q2: Explain `@Transactional` and its pitfalls.
**A:** Creates a proxy that manages transaction begin/commit/rollback. **Pitfalls:** (1) Self-invocation bypasses proxy, (2) only RuntimeException triggers rollback by default, (3) must be on public methods, (4) read-only flag `@Transactional(readOnly = true)` optimizes read queries.

### Q3: Difference between `save()` and `saveAndFlush()`?
**A:** `save()` marks entity for persistence but may defer the SQL. `saveAndFlush()` immediately writes to DB. Use `saveAndFlush()` when you need the generated ID immediately or when subsequent logic depends on the DB state.

### Q4: What is `@MappedSuperclass` vs `@Inheritance`?
**A:** `@MappedSuperclass` — shared fields, NO table for parent (fields go into child tables). `@Inheritance` — actual table hierarchy. Strategies: `SINGLE_TABLE` (one table, discriminator column), `TABLE_PER_CLASS` (separate tables), `JOINED` (normalized, joined on ID).

### Q5: What is the difference between `persist()` and `merge()`?
**A:** `persist()` — makes a new transient entity managed (INSERT). `merge()` — copies state of detached entity to a managed copy (UPDATE or INSERT). `persist()` throws if entity has an ID; `merge()` handles both new and existing entities.

### Q6: How does dirty checking work in Hibernate?
**A:** Hibernate keeps a snapshot of entity state when loaded. At flush time, it compares current state with snapshot. If different → generates UPDATE SQL automatically. No need to explicitly call `save()` for managed entities within a transaction.

---

> **Next:** [08_Microservices_Design_Patterns.md](08_Microservices_Design_Patterns.md) →
