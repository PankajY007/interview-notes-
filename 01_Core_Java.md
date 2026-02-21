# 01 — Core Java (Interview Ready)

> **Level:** 3-4 Years Experience | **Priority:** 🔴 Critical  
> Covers: OOPs, Strings, Exception Handling, JVM Internals, Serialization, Cloning

---

## 📌 Table of Contents

1. [OOPs Concepts](#1-oops-concepts) — Four Pillars, Association, `this` vs `super`, Access Modifiers, Covariant Return, Diamond Problem
2. [String Deep Dive](#2-string-deep-dive) — Pool, Immutability, `+` Operator Internals
3. [Exception Handling](#3-exception-handling) — Hierarchy, Custom Exceptions, Try-with-Resources
4. [JVM Architecture & Memory](#4-jvm-architecture--memory) — Memory Areas, GC, ClassLoader
5. [Wrapper Classes & Autoboxing](#5-wrapper-classes--autoboxing)
6. [Serialization](#6-serialization)
7. [Cloning](#7-cloning)
8. [Miscellaneous Core Concepts](#8-miscellaneous-core-concepts) — `final`, `static`, `equals`/`hashCode`, Pass-by-Value, `instanceof`, Object class, Immutable Records
9. [Interview Questions & Answers](#9-interview-questions--answers) — 22 Q&As
10. [Mini Coding Exercises](#10-mini-coding-exercises)

---

## 1. OOPs Concepts

### 1.1 Four Pillars of OOP

#### **Encapsulation**
Wrapping data (fields) and code (methods) together. Restricting direct access to fields.

```java
public class BankAccount {
    private double balance; // hidden from outside

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount <= 0) throw new IllegalArgumentException("Amount must be positive");
        this.balance += amount;
    }

    public void withdraw(double amount) {
        if (amount > balance) throw new InsufficientFundsException("Not enough balance");
        this.balance -= amount;
    }
}
```

**Real-world scenario:** In production, you never expose internal state directly. A payment service wraps transaction logic — callers can't manipulate balance directly.

**⚠️ Pitfall:** Making all fields `public` "for convenience" during development breaks encapsulation. Always default to `private`.

---

#### **Abstraction**
Hiding complex implementation, showing only what's necessary.

```java
// Abstract class
public abstract class PaymentProcessor {
    // Template method pattern
    public final void processPayment(Order order) {
        validate(order);
        deductAmount(order);
        sendConfirmation(order);
    }

    protected abstract void validate(Order order);
    protected abstract void deductAmount(Order order);

    private void sendConfirmation(Order order) {
        System.out.println("Payment confirmed for order: " + order.getId());
    }
}

// Interface
public interface NotificationService {
    void sendNotification(String userId, String message);
    
    // Java 8+ default method
    default void sendBulkNotification(List<String> userIds, String message) {
        userIds.forEach(id -> sendNotification(id, message));
    }
}
```

**Interview Tip:** When asked "abstract class vs interface" — always mention:
| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Methods | Abstract + Concrete | Abstract + Default (Java 8+) + Static |
| Variables | Any type | `public static final` only |
| Constructor | ✅ Yes | ❌ No |
| Multiple Inheritance | ❌ No | ✅ Yes |
| When to use | IS-A with shared state | CAN-DO capability contract |

---

#### **Inheritance**

```java
public class Employee {
    protected String name;
    protected double baseSalary;

    public double calculatePay() {
        return baseSalary;
    }
}

public class Manager extends Employee {
    private double bonus;

    @Override
    public double calculatePay() {
        return baseSalary + bonus; // extends parent behavior
    }
}
```

**⚠️ Pitfall — Fragile Base Class Problem:**
```java
// Changing the parent class can break child classes unexpectedly
// Prefer composition over inheritance (Effective Java, Item 18)
public class InstrumentedSet<E> {
    private final Set<E> set; // COMPOSITION instead of extends HashSet
    private int addCount = 0;

    public InstrumentedSet(Set<E> set) {
        this.set = set;
    }

    public boolean add(E e) {
        addCount++;
        return set.add(e);
    }
}
```

---

#### **Polymorphism**

**Compile-time (Method Overloading):**
```java
public class Calculator {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }
    public int add(int a, int b, int c) { return a + b + c; }
}
```

**Runtime (Method Overriding):**
```java
public class Shape {
    public double area() { return 0; }
}

public class Circle extends Shape {
    private double radius;

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

// Runtime polymorphism
Shape shape = new Circle(5); // Reference: Shape, Object: Circle
shape.area(); // Calls Circle's area() — resolved at RUNTIME
```

**⚠️ Tricky Interview Question:**
```java
public class Parent {
    public static void greet() { System.out.println("Parent"); }
}

public class Child extends Parent {
    public static void greet() { System.out.println("Child"); }
}

Parent obj = new Child();
obj.greet(); // Output: "Parent" — static methods are NOT overridden, they are HIDDEN
```

---

### 1.2 Association, Aggregation, Composition

```java
// ASSOCIATION — loose relationship (Teacher teaches Student)
class Teacher {
    private String name;
}
class Student {
    private String name;
    private Teacher teacher; // can exist without teacher
}

// AGGREGATION — HAS-A, but child can exist independently
class Department {
    private List<Employee> employees; // employees exist without department
}

// COMPOSITION — HAS-A, child CANNOT exist without parent
class House {
    private final Room room; // room dies when house is destroyed

    public House() {
        this.room = new Room(); // created inside parent
    }
}
```

---

### 1.3 `this` vs `super` Keyword

```java
public class Animal {
    String name;

    Animal(String name) {
        this.name = name; // 'this' refers to current instance
    }

    void sound() {
        System.out.println("Generic sound");
    }
}

public class Dog extends Animal {
    String breed;

    Dog(String name, String breed) {
        super(name);       // calls parent constructor — MUST be first line
        this.breed = breed;
    }

    @Override
    void sound() {
        super.sound();     // calls parent's version
        System.out.println("Bark!");
    }

    void display() {
        System.out.println(this.name);  // inherited field via 'this'
    }
}
```

| Keyword | Purpose |
|---------|--------|
| `this` | Refers to **current object**. Used for: current instance fields, calling another constructor (`this()`), passing current object as argument |
| `super` | Refers to **parent class**. Used for: parent constructor (`super()`), parent method (`super.method()`), parent field access |

**⚠️ Pitfall:** `this()` and `super()` both must be the **first statement** in a constructor — so you can NEVER use both in the same constructor.

---

### 1.4 Access Modifiers

```
┌──────────────┬───────┬─────────┬──────────┬───────┐
│  Modifier    │ Class │ Package │ Subclass │ World │
├──────────────┼───────┼─────────┼──────────┼───────┤
│ private      │  ✅   │   ❌    │    ❌    │  ❌   │
│ default      │  ✅   │   ✅    │    ❌    │  ❌   │
│ protected    │  ✅   │   ✅    │    ✅    │  ❌   │
│ public       │  ✅   │   ✅    │    ✅    │  ✅   │
└──────────────┴───────┴─────────┴──────────┴───────┘
```

**⚠️ Common Confusion — `default` vs `protected`:**
- `default` (no keyword) = accessible within **same package only**
- `protected` = accessible within **same package + subclasses** (even in different packages)

```java
// Package: com.example.base
public class Parent {
    int defaultField = 1;        // default — package only
    protected int protectedField = 2; // subclass access even across packages
}

// Package: com.example.child (DIFFERENT package)
public class Child extends Parent {
    void test() {
        // System.out.println(defaultField);    // ❌ NOT accessible
        System.out.println(protectedField);     // ✅ accessible via inheritance
    }
}
```

---

### 1.5 Covariant Return Types

```java
public class Animal {
    public Animal create() {
        return new Animal();
    }
}

public class Dog extends Animal {
    @Override
    public Dog create() {   // ✅ Returns Dog instead of Animal — this is COVARIANT
        return new Dog();
    }
}

// Why it matters:
Dog dog = new Dog().create(); // No casting needed!
```

**Rule:** An overriding method can return a **subtype** of the return type declared in the parent. This is called a **covariant return type** (allowed since Java 5).

---

### 1.6 Diamond Problem & Interface Default Methods

```java
public interface Flyable {
    default void move() {
        System.out.println("Flying");
    }
}

public interface Swimmable {
    default void move() {
        System.out.println("Swimming");
    }
}

// Diamond Problem — which move() to use?
public class Duck implements Flyable, Swimmable {
    @Override
    public void move() {
        // MUST override to resolve conflict
        Flyable.super.move();    // explicitly choose
        Swimmable.super.move();  // or call both
        System.out.println("Duck moves!");
    }
}
```

**⚠️ Rules:**
1. If two interfaces have the **same default method**, the implementing class **MUST override** it — compiler error otherwise
2. A class method always wins over a default interface method
3. A more specific interface wins (sub-interface over super-interface)

---

## 2. String Deep Dive

### 2.1 String Pool & Immutability

```java
String s1 = "hello";        // Goes to String Pool
String s2 = "hello";        // Reuses from String Pool
String s3 = new String("hello"); // Goes to Heap (new object)

System.out.println(s1 == s2);      // true  (same pool reference)
System.out.println(s1 == s3);      // false (different objects)
System.out.println(s1.equals(s3)); // true  (same content)

String s4 = s3.intern();    // Moves to pool
System.out.println(s1 == s4); // true
```

**Why is String Immutable?**
1. **String Pool** — sharing wouldn't be safe if mutable
2. **Thread Safety** — immutable objects are inherently thread-safe
3. **Security** — database URLs, passwords stored as Strings
4. **Hashcode Caching** — HashMap keys rely on consistent hashcode
5. **Class Loading** — class names are Strings

### 2.2 String vs StringBuilder vs StringBuffer

| Feature | String | StringBuilder | StringBuffer |
|---------|--------|--------------|-------------|
| Mutability | Immutable | Mutable | Mutable |
| Thread-safe | Yes (immutable) | ❌ No | ✅ Yes (synchronized) |
| Performance | Slowest (creates new objects) | Fastest | Slower than StringBuilder |
| When to use | Few modifications | Single-threaded string building | Multi-threaded string building |

```java
// BAD — creates 1000 String objects
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i; // Each += creates new String object
}

// GOOD — single mutable object
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

### 2.3 Common String Pitfalls

```java
// Pitfall 1: == vs equals()
String a = new String("test");
String b = new String("test");
if (a == b) { } // FALSE! Always use .equals()

// Pitfall 2: NullPointerException
String name = null;
// name.equals("John"); // NPE!
"John".equals(name);    // Safe — returns false

// Pitfall 3: String concatenation in loops (O(n²))
// Always use StringBuilder in loops

// Pitfall 4: substring() memory leak (pre-Java 7u6)
// In older Java, substring shared the original char[] array
// Fixed in Java 7u6+ — substring creates new array
```

### 2.4 String `+` Operator Internals

```java
// What actually happens under the hood?
String result = "Hello" + " " + "World";

// Compiler optimization (compile-time constants):
// → Converted to: String result = "Hello World"; (single literal)

// With variables — compiler generates StringBuilder (Java 5-8):
String name = "John";
String greeting = "Hello " + name + "!";
// Becomes:
// String greeting = new StringBuilder().append("Hello ").append(name).append("!").toString();

// Java 9+ — uses invokedynamic (StringConcatFactory) for better performance
// → makeConcatWithConstants("Hello \u0001!")
```

**⚠️ Why `+` in loops is still BAD (even with StringBuilder optimization):**
```java
// Each iteration creates a NEW StringBuilder object!
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i; // new StringBuilder() per iteration → O(n²)
}

// Always use a SINGLE StringBuilder for loops:
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i); // O(n)
}
```

---

## 3. Exception Handling

### 3.1 Exception Hierarchy

```
Throwable
├── Error (DON'T catch these)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── VirtualMachineError
│
└── Exception
    ├── Checked Exceptions (MUST handle at compile time)
    │   ├── IOException
    │   ├── SQLException
    │   ├── FileNotFoundException
    │   └── ClassNotFoundException
    │
    └── RuntimeException / Unchecked (NOT required to handle)
        ├── NullPointerException
        ├── ArrayIndexOutOfBoundsException
        ├── IllegalArgumentException
        ├── IllegalStateException
        ├── ClassCastException
        └── ArithmeticException
```

### 3.2 Custom Exception (Production Pattern)

```java
// Base business exception
public abstract class BusinessException extends RuntimeException {
    private final String errorCode;

    protected BusinessException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }

    public String getErrorCode() {
        return errorCode;
    }
}

// Specific exception
public class OrderNotFoundException extends BusinessException {
    public OrderNotFoundException(Long orderId) {
        super("ORD_404", "Order not found with ID: " + orderId);
    }
}

// Usage in service
public class OrderService {
    public Order getOrder(Long id) {
        return orderRepository.findById(id)
            .orElseThrow(() -> new OrderNotFoundException(id));
    }
}

// Global exception handler (Spring Boot)
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(OrderNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleOrderNotFound(OrderNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(ex.getErrorCode(), ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        log.error("Unexpected error", ex);
        ErrorResponse error = new ErrorResponse("INTERNAL_ERROR", "Something went wrong");
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

### 3.3 Try-with-Resources (Java 7+)

```java
// OLD way — messy, error-prone
BufferedReader reader = null;
try {
    reader = new BufferedReader(new FileReader("data.txt"));
    String line = reader.readLine();
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (reader != null) {
        try {
            reader.close(); // close can also throw!
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// MODERN way — clean, auto-closes
try (BufferedReader reader = new BufferedReader(new FileReader("data.txt"))) {
    String line = reader.readLine();
} catch (IOException e) {
    log.error("Failed to read file", e);
}

// Custom AutoCloseable
public class DatabaseConnection implements AutoCloseable {
    @Override
    public void close() {
        // cleanup connection
    }
}
```

### 3.4 Exception Pitfalls

```java
// ❌ NEVER catch generic Exception silently
try {
    riskyOperation();
} catch (Exception e) {
    // swallowed — you'll never know something broke
}

// ❌ NEVER use exceptions for flow control
try {
    int value = Integer.parseInt(input);
} catch (NumberFormatException e) {
    // Using exception for validation — BAD
}

// ✅ Validate first
if (input.matches("\\d+")) {
    int value = Integer.parseInt(input);
}

// ❌ NEVER throw Exception from finally block
// It will suppress the original exception
```

---

## 4. JVM Architecture & Memory

### 4.1 JVM Memory Areas

```
┌─────────────────────────────────────────┐
│                JVM Memory               │
├─────────────┬───────────────────────────┤
│   HEAP      │  ┌─────────────────────┐  │
│  (shared)   │  │  Young Generation   │  │
│             │  │  ├── Eden Space     │  │
│             │  │  ├── Survivor S0    │  │
│             │  │  └── Survivor S1    │  │
│             │  ├─────────────────────┤  │
│             │  │  Old Generation     │  │
│             │  │  (Tenured)          │  │
│             │  └─────────────────────┘  │
├─────────────┼───────────────────────────┤
│  STACK      │  Thread-specific          │
│  (per       │  ├── Local variables      │
│   thread)   │  ├── Method calls         │
│             │  └── Partial results      │
├─────────────┼───────────────────────────┤
│  METASPACE  │  Class metadata           │
│  (Java 8+)  │  (replaced PermGen)       │
├─────────────┼───────────────────────────┤
│  PC Register│  Current instruction addr │
├─────────────┼───────────────────────────┤
│  Native     │  Native method calls      │
│  Method     │                           │
│  Stack      │                           │
└─────────────┴───────────────────────────┘
```

### 4.2 Garbage Collection

**GC Flow:**
1. New objects → **Eden Space**
2. Eden fills → **Minor GC** → survivors move to S0/S1
3. Objects surviving 15 GC cycles → promoted to **Old Generation**
4. Old Gen fills → **Major GC** (Stop-the-World pause)

**GC Algorithms:**
| GC | Best For | Flag |
|----|----------|------|
| Serial GC | Small apps, single-thread | `-XX:+UseSerialGC` |
| Parallel GC | Throughput-focused | `-XX:+UseParallelGC` |
| G1 GC | Balanced (default Java 9+) | `-XX:+UseG1GC` |
| ZGC | Ultra-low latency | `-XX:+UseZGC` |

### 4.3 ClassLoader Hierarchy

```
Bootstrap ClassLoader (loads rt.jar, core Java classes)
    ↓
Extension/Platform ClassLoader (loads ext/*.jar)
    ↓
Application ClassLoader (loads classpath)
    ↓
Custom ClassLoader (your custom loaders)
```

**Delegation Model:** Child asks parent first → if parent can't load, child tries.

---

## 5. Wrapper Classes & Autoboxing

```java
// Autoboxing & Unboxing
Integer a = 5;          // autoboxing: int → Integer
int b = a;              // unboxing: Integer → int

// ⚠️ TRICKY — Integer Cache (-128 to 127)
Integer x = 127;
Integer y = 127;
System.out.println(x == y);  // TRUE (cached)

Integer p = 128;
Integer q = 128;
System.out.println(p == q);  // FALSE (new objects)

// ⚠️ NullPointerException with unboxing
Integer value = null;
int result = value; // NPE! Can't unbox null
```

---

## 6. Serialization

```java
public class User implements Serializable {
    private static final long serialVersionUID = 1L; // version control

    private String name;
    private transient String password; // NOT serialized
    private static String company;     // NOT serialized (belongs to class)

    // Custom serialization
    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject();
        out.writeObject(encrypt(password)); // custom handling
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        this.password = decrypt((String) in.readObject());
    }
}
```

**⚠️ Pitfall:** If parent class is NOT Serializable but child IS, parent's fields won't be serialized. Parent must have a no-arg constructor.

---

## 7. Cloning

```java
// Shallow Clone — copies references, not objects
public class Employee implements Cloneable {
    private String name;
    private Address address; // reference type

    @Override
    protected Employee clone() throws CloneNotSupportedException {
        return (Employee) super.clone();
        // address will point to SAME object — shallow!
    }
}

// Deep Clone — copies everything
@Override
protected Employee clone() throws CloneNotSupportedException {
    Employee cloned = (Employee) super.clone();
    cloned.address = new Address(this.address); // new copy
    return cloned;
}

// Better alternative — Copy Constructor
public Employee(Employee other) {
    this.name = other.name;
    this.address = new Address(other.address);
}
```

---

## 8. Miscellaneous Core Concepts

### `final` keyword
```java
final int x = 10;         // constant variable
final class Utils { }      // cannot be extended
final void process() { }   // cannot be overridden

// ⚠️ final reference ≠ immutable object
final List<String> list = new ArrayList<>();
list.add("hello");         // ✅ allowed — object is mutable
// list = new ArrayList<>(); // ❌ compile error — can't reassign reference
```

### `static` keyword
```java
public class Counter {
    private static int count = 0; // shared across all instances

    static {
        // Static block — runs once when class loads
        System.out.println("Class loaded");
    }

    public Counter() {
        count++;
    }

    public static int getCount() {
        return count;
        // Cannot access instance members from static context
    }
}
```

### Java is Strictly Pass-by-Value

```java
// Primitive — value is copied
public static void modify(int x) {
    x = 100;
}
int a = 5;
modify(a);
System.out.println(a); // 5 — NOT changed

// Object — reference is copied (NOT the object itself)
public static void modify(StringBuilder sb) {
    sb.append(" World"); // modifies the SAME object ✅
}
StringBuilder s = new StringBuilder("Hello");
modify(s);
System.out.println(s); // "Hello World"

// But reassigning the reference does NOT affect the caller
public static void reassign(StringBuilder sb) {
    sb = new StringBuilder("New"); // local reference now points elsewhere
}
StringBuilder s2 = new StringBuilder("Original");
reassign(s2);
System.out.println(s2); // "Original" — NOT changed!
```

**Interview one-liner:** "Java passes the VALUE of the reference, not the reference itself. You can modify the object through the reference, but you can't make the caller's reference point to a different object."

---

### `instanceof` and Type Casting

```java
Animal animal = new Dog();

// Check type before casting
if (animal instanceof Dog) {
    Dog dog = (Dog) animal;   // safe downcast
    dog.bark();
}

// Java 16+ Pattern Matching (cleaner)
if (animal instanceof Dog dog) {
    dog.bark(); // no explicit cast needed — 'dog' is already cast
}

// ⚠️ ClassCastException without instanceof check
Animal cat = new Cat();
Dog dog = (Dog) cat; // 💥 ClassCastException at runtime!
```

| Cast Type | Direction | Safety |
|-----------|-----------|--------|
| **Upcasting** | Child → Parent | Always safe, implicit |
| **Downcasting** | Parent → Child | Risky, explicit, use `instanceof` |

---

### Object Class Methods (The Root of Everything)

Every Java class inherits from `Object`. Key methods:

```java
public class Object {
    public String toString()          // string representation
    public boolean equals(Object obj) // content equality
    public int hashCode()             // hash for collections
    public final Class<?> getClass()  // runtime class info
    protected Object clone()          // shallow copy
    public final void wait()          // thread waits on monitor
    public final void notify()        // wakes one waiting thread
    public final void notifyAll()     // wakes all waiting threads
    protected void finalize()         // called before GC (deprecated)
}
```

**⚠️ Common mistake:** Not overriding `toString()` — printing an object shows `ClassName@hashcode` instead of useful info.

```java
// Always override toString for debugging
@Override
public String toString() {
    return "Employee{id=" + id + ", name='" + name + "'}";
}
```

---

### Immutable Class with Java `record` (Java 16+)

```java
// Before records — verbose boilerplate
public final class Point {
    private final int x;
    private final int y;
    public Point(int x, int y) { this.x = x; this.y = y; }
    public int x() { return x; }
    public int y() { return y; }
    @Override public boolean equals(Object o) { /* ... */ }
    @Override public int hashCode() { return Objects.hash(x, y); }
    @Override public String toString() { return "Point[x=" + x + ", y=" + y + "]"; }
}

// With records — one line! Auto-generates constructor, getters, equals, hashCode, toString
public record Point(int x, int y) {}

// Usage
Point p = new Point(3, 5);
System.out.println(p.x());      // 3 (accessor method, NOT getX())
System.out.println(p);          // Point[x=3, y=5]

// Records can have:
public record Employee(String name, int age) {
    // Compact constructor for validation
    public Employee {
        if (age < 0) throw new IllegalArgumentException("Age can't be negative");
        name = name.trim(); // can modify before assignment
    }

    // Custom methods
    public String greeting() {
        return "Hi, I'm " + name;
    }
}
```

**Records are:** `final`, fields are `private final`, cannot extend other classes (but can implement interfaces).

---

### `equals()` and `hashCode()` Contract
```java
public class Employee {
    private int id;
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Employee emp = (Employee) o;
        return id == emp.id && Objects.equals(name, emp.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}
```

**Contract Rules:**
1. If `a.equals(b)` is `true` → `a.hashCode() == b.hashCode()` MUST be `true`
2. If `hashCode` is same → `equals` CAN be `false` (collision)
3. If you override `equals`, you MUST override `hashCode`

**⚠️ Breaking this contract corrupts HashMap/HashSet behavior!**

---

## 9. Interview Questions & Answers

### Q1: What is the difference between `==` and `.equals()`?
**A:** `==` compares **references** (memory addresses). `.equals()` compares **content/values**. For primitives, `==` compares values. For objects, always use `.equals()`. String Pool can make `==` work for string literals, but it's unreliable — always use `.equals()`.

---

### Q2: Why is String immutable in Java?
**A:** 
1. **String Pool optimization** — multiple references share the same string safely
2. **Thread safety** — no synchronization needed
3. **Security** — network connections, file paths, DB URLs use strings
4. **Hashcode caching** — computed once, reused (critical for HashMap keys)
5. **Class loading** — class names are strings, mutation would be a security risk

---

### Q3: What happens when you create `new String("hello")`?
**A:** 
- If "hello" doesn't exist in pool → **2 objects** created (one in pool, one in heap)
- If "hello" already exists in pool → **1 object** created (only in heap)
- `new` always creates a new object on the heap, regardless of the pool

---

### Q4: Explain JVM memory model.
**A:**
- **Heap** — shared memory for all objects. Divided into Young Gen (Eden + Survivors) and Old Gen
- **Stack** — per-thread, stores method frames, local variables, references
- **Metaspace** — stores class metadata (replaced PermGen in Java 8)
- **PC Register** — per-thread, holds address of current instruction
- Objects live on Heap; references to them live on Stack

---

### Q5: What is the difference between Checked and Unchecked exceptions?
**A:**
- **Checked** — must be handled at compile time (try-catch or throws). E.g., `IOException`, `SQLException`
- **Unchecked** — runtime exceptions, not forced to handle. E.g., `NullPointerException`, `ArrayIndexOutOfBoundsException`
- **Rule of thumb:** Use checked for recoverable conditions, unchecked for programming errors

---

### Q6: Can we override a static method?
**A:** No. Static methods belong to the class, not the object. What looks like overriding is actually **method hiding**. The method called depends on the reference type (compile time), not the object type (runtime).

---

### Q7: What is the `finalize()` method? Is it reliable?
**A:** `finalize()` is called by GC before reclaiming memory. It's **NOT reliable** because:
1. No guarantee when (or if) GC runs
2. Slows down GC
3. Can cause objects to be resurrected
4. **Deprecated since Java 9.** Use `try-with-resources` or `Cleaner` instead.

---

### Q8: What is `transient` keyword?
**A:** Marks a field to be excluded from serialization. Common use: passwords, calculated fields, sensitive data. When deserialized, `transient` fields get default values (null, 0, false).

---

### Q9: Explain `volatile` keyword.
**A:** Ensures a variable is read from **main memory**, not from thread's local cache. Provides **visibility** guarantee but NOT atomicity. Use for flags shared between threads. For compound operations, use `AtomicInteger` or `synchronized`.

---

### Q10: What is a marker interface?
**A:** An interface with no methods. It "marks" a class with a capability. Examples: `Serializable`, `Cloneable`, `Remote`. Modern alternative: annotations (`@Entity`, `@Deprecated`).

---

### Q11: Explain method dispatch — static vs dynamic binding.
**A:**
- **Static binding (early):** Compile time. For `private`, `static`, and `final` methods. Compiler knows exact method.
- **Dynamic binding (late):** Runtime. For overridden instance methods. JVM uses actual object type via vtable.

---

### Q12: What is an immutable class? How to create one?
**A:**
```java
public final class Money {                    // 1. Class is final
    private final BigDecimal amount;          // 2. Fields are private final
    private final String currency;
    private final List<String> tags;

    public Money(BigDecimal amount, String currency, List<String> tags) {
        this.amount = amount;                 // 3. Set in constructor
        this.currency = currency;
        this.tags = Collections.unmodifiableList(new ArrayList<>(tags)); // 4. Defensive copy
    }

    public BigDecimal getAmount() { return amount; }      // 5. No setters
    public String getCurrency() { return currency; }
    public List<String> getTags() { return Collections.unmodifiableList(tags); }
}
```

---

### Q13: What is the difference between `this` and `super`?
**A:**
- `this` refers to the **current object** — used to access current class fields, methods, or constructors (`this()`)
- `super` refers to the **parent class** — used to access parent fields, methods, or constructors (`super()`)
- Both `this()` and `super()` must be the first statement in a constructor, so you can't use both

---

### Q14: Difference between `final`, `finally`, and `finalize()`?
**A:**
| Keyword | Purpose |
|---------|--------|
| `final` | Makes variable constant, method non-overridable, class non-extendable |
| `finally` | Block that always executes after try-catch (cleanup code) |
| `finalize()` | Called by GC before object destruction (deprecated since Java 9) |

**Edge case:** `finally` does NOT execute if `System.exit()` is called or JVM crashes.

---

### Q15: Can an abstract class have all concrete methods?
**A:** Yes! An abstract class can have zero abstract methods. You'd do this to prevent direct instantiation while allowing subclassing. Example: `HttpServlet` — it has default implementations but you're expected to subclass it.

---

### Q16: What happens when two interfaces have the same default method?
**A:** The implementing class gets a **compile-time error** and MUST override the conflicting method. You can call a specific interface's version using `InterfaceName.super.method()`.

---

### Q17: `ClassNotFoundException` vs `NoClassDefFoundError`?
**A:**
| | `ClassNotFoundException` | `NoClassDefFoundError` |
|---|---|---|
| Type | Checked Exception | Error |
| When | `Class.forName()` or `ClassLoader.loadClass()` can't find class | Class was available at compile time but missing at runtime |
| Cause | Wrong classpath, missing JAR | Deleted JAR, static initializer failure |
| Fix | Check classpath, add dependency | Check build/deployment, verify all JARs present |

---

### Q18: `StackOverflowError` vs `OutOfMemoryError`?
**A:**
- **StackOverflowError** — Stack is full. Caused by infinite/deep recursion. Each method call adds a frame to the thread's stack.
- **OutOfMemoryError** — Heap is full. Caused by creating too many objects, memory leaks (holding references to unused objects), or loading huge data into memory.
- Both are `Error` (not `Exception`) — you generally shouldn't catch them.

---

### Q19: Heap vs Stack memory?
**A:**
| Feature | Stack | Heap |
|---------|-------|------|
| Stores | Local variables, method frames, references | Objects, instance variables |
| Thread | Per-thread (private) | Shared across all threads |
| Size | Small, fixed | Large, growable |
| Speed | Faster (LIFO allocation) | Slower (GC manages) |
| Error | `StackOverflowError` | `OutOfMemoryError` |
| Lifecycle | Freed when method returns | Freed by Garbage Collector |

---

### Q20: `Comparable` vs `Comparator`?
**A:**
```java
// Comparable — natural ordering, implemented BY the class itself
public class Employee implements Comparable<Employee> {
    @Override
    public int compareTo(Employee other) {
        return this.name.compareTo(other.name); // single sorting logic
    }
}
Collections.sort(employees); // uses compareTo

// Comparator — external, flexible, multiple sort strategies
Comparator<Employee> bySalary = Comparator.comparing(Employee::getSalary);
Comparator<Employee> byAge = Comparator.comparing(Employee::getAge).reversed();
Collections.sort(employees, bySalary);
```
| | `Comparable` | `Comparator` |
|---|---|---|
| Package | `java.lang` | `java.util` |
| Method | `compareTo()` | `compare()` |
| # of sorts | One (natural order) | Many (custom) |
| Modifies class? | Yes | No (external) |

---

### Q21: What is fail-fast vs fail-safe?
**A:**
- **Fail-fast:** Throws `ConcurrentModificationException` if collection is modified during iteration. E.g., `ArrayList`, `HashMap` iterators.
- **Fail-safe:** Works on a **copy** of the collection — no exception, but may not reflect latest changes. E.g., `CopyOnWriteArrayList`, `ConcurrentHashMap`.

```java
// Fail-fast — throws ConcurrentModificationException
List<String> list = new ArrayList<>(List.of("a", "b", "c"));
for (String s : list) {
    list.remove(s); // 💥 ConcurrentModificationException
}

// Safe way — use Iterator.remove()
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("b")) it.remove(); // ✅ safe
}
```

---

### Q22: Is Java pass-by-value or pass-by-reference?
**A:** Java is **always pass-by-value**. For primitives, the value is copied. For objects, the **reference (pointer) is copied** — not the object itself. This means:
- You CAN modify the object's state through the copied reference
- You CANNOT make the caller's variable point to a different object (reassigning inside a method has no effect on the caller)

---

## 10. Mini Coding Exercises

### Exercise 1: Reverse a String without using library methods
```java
public static String reverse(String input) {
    // Try solving before looking at the solution below
    char[] chars = input.toCharArray();
    int left = 0, right = chars.length - 1;
    while (left < right) {
        char temp = chars[left];
        chars[left] = chars[right];
        chars[right] = temp;
        left++;
        right--;
    }
    return new String(chars);
}
```

### Exercise 2: Check if two strings are anagrams
```java
public static boolean isAnagram(String s1, String s2) {
    if (s1.length() != s2.length()) return false;
    int[] freq = new int[26];
    for (char c : s1.toLowerCase().toCharArray()) freq[c - 'a']++;
    for (char c : s2.toLowerCase().toCharArray()) freq[c - 'a']--;
    for (int count : freq) {
        if (count != 0) return false;
    }
    return true;
}
```

### Exercise 3: Singleton Pattern (all 3 ways)
```java
// Way 1: Eager Initialization
public class EagerSingleton {
    private static final EagerSingleton INSTANCE = new EagerSingleton();
    private EagerSingleton() {}
    public static EagerSingleton getInstance() { return INSTANCE; }
}

// Way 2: Double-Checked Locking (Thread-safe, Lazy)
public class DCLSingleton {
    private static volatile DCLSingleton instance;
    private DCLSingleton() {}
    public static DCLSingleton getInstance() {
        if (instance == null) {
            synchronized (DCLSingleton.class) {
                if (instance == null) {
                    instance = new DCLSingleton();
                }
            }
        }
        return instance;
    }
}

// Way 3: Enum Singleton (Best — recommended by Joshua Bloch)
public enum EnumSingleton {
    INSTANCE;
    public void doSomething() { }
}
```

### Exercise 4: Find duplicate characters in a string
```java
public static Map<Character, Integer> findDuplicates(String str) {
    Map<Character, Integer> charCount = new LinkedHashMap<>();
    for (char c : str.toCharArray()) {
        charCount.merge(c, 1, Integer::sum);
    }
    charCount.entrySet().removeIf(e -> e.getValue() == 1);
    return charCount;
}
```

---

> **Next:** [02_Collections_Deep_Dive.md](02_Collections_Deep_Dive.md) →
