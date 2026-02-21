# 04 — Java 8+ Features (Interview Ready)

> **Level:** 3-4 Years Experience | **Priority:** 🔴 Critical  
> Covers: Lambda, Functional Interfaces, Streams, Optional, Method References, Records, Sealed Classes

---

## 📌 Table of Contents

1. [Functional Interfaces](#1-functional-interfaces) — Built-in, Custom, `@FunctionalInterface`
2. [Lambda Expressions](#2-lambda-expressions) — Syntax, Variable Capture
3. [Method References](#3-method-references)
4. [Stream API (Deep Dive)](#4-stream-api-deep-dive) — Intermediate, Terminal, Collectors, `reduce()`, Primitive Streams, Parallel
5. [Optional](#5-optional) — Do’s and Don’ts, Anti-patterns
6. [New Date/Time API (java.time)](#6-new-datetime-api)
7. [Java 9-21 Key Features](#7-java-9-21-key-features) — Records, Sealed Classes, Pattern Matching, Virtual Threads
8. [Interview Questions & Answers](#8-interview-questions--answers) — 14 Q&As

---

## 1. Functional Interfaces

A functional interface has **exactly one abstract method** (SAM — Single Abstract Method).

### Built-in Functional Interfaces

| Interface | Method | Input → Output | Use Case |
|-----------|--------|----------------|----------|
| `Predicate<T>` | `test(T)` | T → boolean | Filtering |
| `Function<T,R>` | `apply(T)` | T → R | Transforming |
| `Consumer<T>` | `accept(T)` | T → void | Side effects (print, save) |
| `Supplier<T>` | `get()` | () → T | Factory, lazy loading |
| `BiFunction<T,U,R>` | `apply(T,U)` | (T,U) → R | Two-arg transformation |
| `BiPredicate<T,U>` | `test(T,U)` | (T,U) → boolean | Two-arg filtering |
| `UnaryOperator<T>` | `apply(T)` | T → T | Same type transform |
| `BinaryOperator<T>` | `apply(T,T)` | (T,T) → T | Reduce operations |

```java
// Predicate — filtering
Predicate<String> isLong = s -> s.length() > 5;
Predicate<String> startsWithA = s -> s.startsWith("A");
Predicate<String> combined = isLong.and(startsWithA); // chaining

// Function — transformation
Function<String, Integer> toLength = String::length;
Function<String, String> toUpper = String::toUpperCase;
Function<String, Integer> pipeline = toUpper.andThen(toLength); // chain

// Consumer — side effects
Consumer<String> print = System.out::println;
Consumer<String> log = s -> logger.info("Processing: {}", s);
Consumer<String> both = print.andThen(log); // chain consumers

// Supplier — lazy creation
Supplier<List<String>> listFactory = ArrayList::new;
Supplier<LocalDateTime> now = LocalDateTime::now;
```

### Custom Functional Interface
```java
@FunctionalInterface
public interface Converter<F, T> {
    T convert(F from);
    
    // Can have default and static methods
    default Converter<F, T> andThen(Converter<T, ?> after) {
        return (F f) -> after.convert(this.convert(f));
    }
}

Converter<String, Integer> parseInt = Integer::valueOf;
Converter<Integer, String> intToString = String::valueOf;
```

### Why `@FunctionalInterface` Matters

```java
@FunctionalInterface
public interface Converter<F, T> {
    T convert(F from);
    // Adding a second abstract method → COMPILE ERROR because of @FunctionalInterface
    // void anotherMethod(); // ❌
}

public interface NotMarked {
    void doSomething();
    // Without @FunctionalInterface, someone could add another abstract method
    // breaking all existing lambdas — no compiler protection!
}
```

**Rule:** A functional interface CAN have:
- ✅ Exactly 1 abstract method
- ✅ Multiple `default` methods
- ✅ Multiple `static` methods
- ✅ Methods from `Object` class (`equals`, `hashCode`, `toString`)

The `@FunctionalInterface` annotation is optional but recommended — it's like `@Override`, a compile-time safety net.

---

## 2. Lambda Expressions

```java
// Syntax: (parameters) -> expression OR (parameters) -> { statements }
Runnable r = () -> System.out.println("Hello");
Comparator<String> comp = (a, b) -> a.compareTo(b);
Function<Integer, Integer> square = x -> x * x;

// Multi-line
Comparator<Employee> byName = (e1, e2) -> {
    String name1 = e1.getName().toLowerCase();
    String name2 = e2.getName().toLowerCase();
    return name1.compareTo(name2);
};
```

### Lambda Variable Capture
```java
int x = 10;
// x = 20; // ❌ Cannot modify — must be effectively final
Runnable r = () -> System.out.println(x); // captures x

// ⚠️ Lambda captures reference to variable, not value for objects
List<String> names = new ArrayList<>();
Runnable r = () -> names.add("Java"); // ✅ modifying object OK
// names = new ArrayList<>(); // ❌ can't reassign reference
```

---

## 3. Method References

| Type | Syntax | Lambda Equivalent |
|------|--------|-------------------|
| Static method | `Class::staticMethod` | `x -> Class.staticMethod(x)` |
| Instance method (bound) | `obj::method` | `x -> obj.method(x)` |
| Instance method (unbound) | `Class::method` | `(obj, x) -> obj.method(x)` |
| Constructor | `Class::new` | `x -> new Class(x)` |

```java
// Static
Function<String, Integer> parse = Integer::parseInt;

// Bound instance
String str = "Hello";
Supplier<Integer> len = str::length;

// Unbound instance
Function<String, String> upper = String::toUpperCase;

// Constructor
Supplier<ArrayList<String>> factory = ArrayList::new;
Function<String, Employee> empFactory = Employee::new;
```

---

## 4. Stream API (Deep Dive)

### Stream Pipeline

```
Source → Intermediate Operations (lazy) → Terminal Operation (triggers execution)
```

### Creating Streams
```java
// From collection
List<String> names = List.of("Alice", "Bob", "Charlie");
Stream<String> stream = names.stream();
Stream<String> parallel = names.parallelStream();

// From values
Stream<String> of = Stream.of("A", "B", "C");

// From array
int[] arr = {1, 2, 3};
IntStream intStream = Arrays.stream(arr);

// Generate/Iterate
Stream<Double> randoms = Stream.generate(Math::random).limit(10);
Stream<Integer> evens = Stream.iterate(0, n -> n + 2).limit(10);

// Java 9: iterate with predicate
Stream<Integer> range = Stream.iterate(0, n -> n < 100, n -> n + 2);
```

### Intermediate Operations (Lazy — don't execute until terminal op)

```java
List<Employee> employees = getEmployees();

// filter — keep elements matching predicate
employees.stream()
    .filter(e -> e.getSalary() > 50000)

// map — transform each element
    .map(Employee::getName)

// flatMap — flatten nested collections
List<List<String>> nested = List.of(List.of("A", "B"), List.of("C"));
nested.stream().flatMap(Collection::stream); // ["A", "B", "C"]

// distinct — remove duplicates (uses equals/hashCode)
    .distinct()

// sorted — sort (natural or custom)
    .sorted(Comparator.comparing(Employee::getSalary).reversed())

// peek — debug (side effect without consuming)
    .peek(e -> log.info("Processing: {}", e))

// limit / skip
    .skip(5)
    .limit(10)
```

### Terminal Operations (Trigger execution)

```java
// collect — gather results
List<String> names = stream.collect(Collectors.toList());
Set<String> unique = stream.collect(Collectors.toSet());
Map<Long, Employee> map = stream.collect(Collectors.toMap(Employee::getId, e -> e));

// forEach — iterate
stream.forEach(System.out::println);

// reduce — combine all elements
int sum = IntStream.of(1, 2, 3, 4, 5).reduce(0, Integer::sum);
Optional<Integer> max = stream.reduce(Integer::max);

// count, min, max
long count = stream.count();
Optional<Employee> youngest = stream.min(Comparator.comparingInt(Employee::getAge));

// findFirst, findAny
Optional<String> first = stream.filter(s -> s.startsWith("A")).findFirst();

// anyMatch, allMatch, noneMatch
boolean hasAdmin = stream.anyMatch(e -> e.getRole().equals("ADMIN"));
```

### Collectors — Advanced

```java
// groupingBy
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// groupingBy with downstream collector
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment, Collectors.counting()));

Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));

Map<String, Optional<Employee>> highestPaidByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.maxBy(Comparator.comparingDouble(Employee::getSalary))
    ));

// partitioningBy (boolean split)
Map<Boolean, List<Employee>> partition = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getSalary() > 50000));
List<Employee> highPaid = partition.get(true);
List<Employee> lowPaid = partition.get(false);

// joining
String csv = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.joining(", ", "[", "]"));
// Output: [Alice, Bob, Charlie]

// toUnmodifiableList (Java 10+)
List<String> immutable = stream.collect(Collectors.toUnmodifiableList());

// teeing (Java 12+) — two collectors at once
var result = employees.stream().collect(Collectors.teeing(
    Collectors.counting(),
    Collectors.averagingDouble(Employee::getSalary),
    (count, avgSalary) -> "Count: " + count + ", Avg: " + avgSalary
));
```

### Parallel Streams

```java
// Use for CPU-intensive tasks with large data
List<Integer> result = numbers.parallelStream()
    .filter(n -> isPrime(n))
    .collect(Collectors.toList());

// ⚠️ Pitfalls:
// 1. NOT faster for small datasets (overhead > benefit)
// 2. NOT for I/O-bound tasks (thread pool is shared ForkJoinPool)
// 3. Must be stateless & non-interfering
// 4. Order may not be preserved (use forEachOrdered if needed)
// 5. Shared mutable state → race conditions
```

### `flatMap` — Real-World Examples

```java
// Example 1: Orders → all order items
List<Order> orders = getOrders();
List<Item> allItems = orders.stream()
    .flatMap(order -> order.getItems().stream()) // flatten nested lists
    .collect(Collectors.toList());

// Example 2: Split sentences into words
List<String> sentences = List.of("Hello World", "Java Streams");
List<String> words = sentences.stream()
    .flatMap(s -> Arrays.stream(s.split(" ")))
    .collect(Collectors.toList());
// ["Hello", "World", "Java", "Streams"]

// Example 3: Optional + flatMap (avoid Optional<Optional<T>>)
Optional<User> user = findUser(id);
Optional<Address> address = user.flatMap(User::getAddress); // NOT map!
// map would give Optional<Optional<Address>> — flatMap flattens it
```

### Short-Circuiting Operations

Some stream operations DON'T process all elements:

```java
// Short-circuiting INTERMEDIATE ops:
.limit(n)       // stops after n elements

// Short-circuiting TERMINAL ops:
.findFirst()    // stops at first match
.findAny()      // stops at any match (useful in parallel)
.anyMatch()     // stops when first true found
.allMatch()     // stops when first false found
.noneMatch()    // stops when first true found

// Example: efficient early termination
boolean hasExpensive = products.stream()
    .peek(p -> System.out.println("Checking: " + p.getName()))
    .anyMatch(p -> p.getPrice() > 1000);
// Only checks elements until it finds one > 1000, then STOPS
```

### `reduce()` Deep Dive

```java
// 1. reduce(identity, accumulator)
int sum = List.of(1, 2, 3, 4, 5).stream()
    .reduce(0, Integer::sum); // 0 + 1 + 2 + 3 + 4 + 5 = 15

// 2. reduce(accumulator) — returns Optional (no identity)
Optional<Integer> max = List.of(1, 2, 3).stream()
    .reduce(Integer::max); // Optional[3]

// 3. reduce(identity, accumulator, combiner) — for PARALLEL streams
int total = List.of(1, 2, 3, 4, 5).parallelStream()
    .reduce(
        0,                    // identity
        Integer::sum,         // accumulator (per-thread partial sum)
        Integer::sum          // combiner (merge partial results)
    );
// Without combiner, parallel stream can't merge results from different threads!

// Real-world: concatenate names
String allNames = employees.stream()
    .map(Employee::getName)
    .reduce("", (a, b) -> a + ", " + b);
// Better: use Collectors.joining(", ") instead
```

**Identity rules:** `reduce(identity, op)` — identity must satisfy `op(identity, x) == x`.
- Sum: identity = `0` (because 0 + x = x)
- Product: identity = `1` (because 1 * x = x)
- String concat: identity = `""` (because "" + x = x)

### Primitive Streams (`IntStream`, `LongStream`, `DoubleStream`)

```java
// Avoid boxing overhead for numeric operations
IntStream intStream = IntStream.of(1, 2, 3, 4, 5);
IntStream range = IntStream.range(1, 10);        // 1 to 9
IntStream rangeClosed = IntStream.rangeClosed(1, 10); // 1 to 10

// Convert to primitive stream
int totalSalary = employees.stream()
    .mapToInt(Employee::getSalary)   // Stream<Employee> → IntStream
    .sum();                          // direct sum (no boxing)

// summaryStatistics — get everything in one pass!
IntSummaryStatistics stats = employees.stream()
    .mapToInt(Employee::getAge)
    .summaryStatistics();

stats.getMin();     // youngest
stats.getMax();     // oldest
stats.getAverage(); // average age
stats.getSum();     // total age
stats.getCount();   // number of employees

// Boxing back: IntStream → Stream<Integer>
Stream<Integer> boxed = IntStream.of(1, 2, 3).boxed();
```

### `Collectors.toMap()` Pitfalls

```java
// ❌ Duplicate key → throws IllegalStateException!
List<Employee> emps = List.of(
    new Employee(1, "HR", "Alice"),
    new Employee(2, "HR", "Bob")  // same department!
);

Map<String, Employee> byDept = emps.stream()
    .collect(Collectors.toMap(Employee::getDepartment, e -> e));
// 💥 IllegalStateException: Duplicate key HR

// ✅ Fix: provide merge function (3rd argument)
Map<String, Employee> byDept = emps.stream()
    .collect(Collectors.toMap(
        Employee::getDepartment,
        e -> e,
        (existing, replacement) -> existing  // keep first on conflict
    ));

// ✅ With specific Map type (4th argument)
Map<String, Employee> sorted = emps.stream()
    .collect(Collectors.toMap(
        Employee::getName,
        e -> e,
        (e1, e2) -> e1,
        TreeMap::new            // sorted by key
    ));
```

---

## 5. Optional

```java
// Creating Optional
Optional<String> empty = Optional.empty();
Optional<String> present = Optional.of("Hello");   // NPE if null!
Optional<String> nullable = Optional.ofNullable(name); // safe

// Using Optional — DO
Optional<User> user = userRepository.findById(id);

String name = user
    .map(User::getName)
    .filter(n -> n.length() > 3)
    .orElse("Unknown");

String name = user
    .map(User::getName)
    .orElseThrow(() -> new UserNotFoundException(id));

user.ifPresent(u -> log.info("Found: {}", u));

// Java 9+
user.ifPresentOrElse(
    u -> log.info("Found: {}", u),
    () -> log.warn("User not found")
);

Optional<User> fallback = user.or(() -> findInCache(id)); // Java 9+
Stream<User> stream = user.stream(); // Java 9+

// ⚠️ DON'T — Anti-patterns
// 1. Don't use Optional for fields
class User {
    Optional<String> name; // ❌ BAD — use null + getter returns Optional
}

// 2. Don't use as method parameter
void process(Optional<String> name) { } // ❌ BAD

// 3. Don't use isPresent() + get()
if (opt.isPresent()) {
    return opt.get(); // ❌ BAD — use orElse/map instead
}

// 4. Don't use for collections — return empty collection instead
Optional<List<String>> names; // ❌ BAD
List<String> names = Collections.emptyList(); // ✅ GOOD
```

---

## 6. New Date/Time API

```java
// Java 8+ (java.time package — replaces Date & Calendar)
LocalDate date = LocalDate.now();           // 2026-02-18
LocalTime time = LocalTime.now();           // 14:30:15.123
LocalDateTime dateTime = LocalDateTime.now();
ZonedDateTime zoned = ZonedDateTime.now(ZoneId.of("America/New_York"));
Instant instant = Instant.now();            // UTC timestamp

// Parsing & Formatting
LocalDate parsed = LocalDate.parse("2026-02-18");
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd-MM-yyyy");
String formatted = date.format(fmt); // "18-02-2026"

// Manipulation (immutable — returns new object)
LocalDate tomorrow = date.plusDays(1);
LocalDate lastMonth = date.minusMonths(1);
LocalDate nextMonday = date.with(TemporalAdjusters.next(DayOfWeek.MONDAY));

// Duration & Period
Duration duration = Duration.between(time1, time2);     // hours, minutes, seconds
Period period = Period.between(date1, date2);            // years, months, days
long days = ChronoUnit.DAYS.between(date1, date2);
```

---

## 7. Java 9-21 Key Features

### Java 9
```java
// Factory methods for collections
List<String> list = List.of("A", "B", "C");       // immutable
Set<Integer> set = Set.of(1, 2, 3);
Map<String, Integer> map = Map.of("A", 1, "B", 2);

// Optional improvements
optional.ifPresentOrElse(val -> use(val), () -> handleEmpty());
optional.or(() -> fallback());
optional.stream();

// Private methods in interfaces
interface MyInterface {
    default void method1() { helper(); }
    default void method2() { helper(); }
    private void helper() { /* shared logic */ }
}

// Modules (Project Jigsaw) — module-info.java
```

### Java 10
```java
// var — local variable type inference
var list = new ArrayList<String>();    // inferred as ArrayList<String>
var stream = list.stream();            // inferred as Stream<String>

// ⚠️ var rules:
// ✅ Local variables with initializer
// ❌ Not for fields, method params, return types
// ❌ Not for null: var x = null; // compile error
```

### Java 11
```java
// var in lambda parameters
list.forEach((var item) -> process(item));

// String methods
"  hello  ".strip();          // "hello" (Unicode-aware, unlike trim())
"  hello  ".stripLeading();   // "hello  "
"hello".repeat(3);            // "hellohellohello"
"hello".isBlank();            // false
"line1\nline2".lines();       // Stream of lines

// HttpClient (standard)
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com"))
    .build();
HttpResponse<String> response = client.send(request, BodyHandlers.ofString());
```

### Java 14
```java
// Switch expressions
String result = switch (day) {
    case MONDAY, FRIDAY -> "Work";
    case SATURDAY, SUNDAY -> "Rest";
    default -> "Unknown";
};

// Records (preview, stable in 16)
public record Employee(String name, int age, double salary) {
    // auto-generates: constructor, getters, equals, hashCode, toString
    
    // Compact constructor (validation)
    public Employee {
        if (age < 0) throw new IllegalArgumentException("Age can't be negative");
    }
}

Employee emp = new Employee("John", 30, 50000);
emp.name();   // "John" — no get prefix
emp.age();    // 30
```

### Java 15-16
```java
// Text blocks (multi-line strings)
String json = """
        {
            "name": "John",
            "age": 30
        }
        """;

// Pattern matching for instanceof
if (obj instanceof String s) {
    System.out.println(s.length()); // no explicit cast needed
}

// Sealed classes (Java 17 stable)
public sealed class Shape permits Circle, Rectangle, Triangle {
    // Only Circle, Rectangle, Triangle can extend Shape
}

public final class Circle extends Shape { }
public non-sealed class Rectangle extends Shape { } // open for extension
```

### `var` Best Practices

```java
// ✅ GOOD — type is obvious from the right side
var list = new ArrayList<String>();
var response = httpClient.send(request, BodyHandlers.ofString());
var entry = map.entrySet().iterator().next();

// ❌ BAD — type is NOT obvious
var result = service.process();     // what type is result?
var x = calculate();                // meaningless name + unclear type
var data = getData();               // too vague

// ❌ CANNOT use var for:
var x;                              // no initializer
var x = null;                       // can't infer type from null
var x = {1, 2, 3};                  // array initializer
public var field = 10;              // class fields
public var method() { }             // return types
public void method(var x) { }       // parameters
```

### Java 17+ (Pattern Matching Switch)

```java
// Java 17: Pattern matching for switch (preview, stable in 21)
static String describe(Object obj) {
    return switch (obj) {
        case Integer i -> "Integer: " + i;
        case String s  -> "String of length " + s.length();
        case null      -> "null value";
        case int[] arr -> "Array of length " + arr.length;
        default        -> "Unknown: " + obj.getClass();
    };
}

// Guarded patterns (Java 21)
static String categorize(Shape shape) {
    return switch (shape) {
        case Circle c when c.radius() > 10 -> "Large circle";
        case Circle c                       -> "Small circle";
        case Rectangle r                    -> "Rectangle";
        default                             -> "Unknown";
    };
}
```

### Java 21 Features

```java
// 1. Sequenced Collections — first/last access for ordered collections
List<String> list = new ArrayList<>(List.of("A", "B", "C"));
list.getFirst();                // "A" (replaces list.get(0))
list.getLast();                 // "C" (replaces list.get(list.size()-1))
list.reversed();                // reversed view ["C", "B", "A"]

SequencedMap<String, Integer> map = new LinkedHashMap<>();
map.firstEntry();               // first inserted entry
map.lastEntry();                // last inserted entry
map.pollFirstEntry();           // remove and return first

// 2. Record Patterns
record Point(int x, int y) {}
record Line(Point start, Point end) {}

static void printLine(Object obj) {
    if (obj instanceof Line(Point(var x1, var y1), Point(var x2, var y2))) {
        System.out.println("Line from (" + x1 + "," + y1 + ") to (" + x2 + "," + y2 + ")");
    }
}

// 3. Virtual Threads (covered in Multithreading file)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> callAPI());
}

// 4. Unnamed Variables (Java 22, preview in 21)
try {
    processFile();
} catch (IOException _) {       // don't need the exception variable
    log.error("File error");
}

map.forEach((_, value) -> process(value)); // don't need the key
```

---

## 8. Interview Questions & Answers

### Q1: What is the difference between `map()` and `flatMap()` in streams?
**A:** `map()` applies a function to each element and wraps the result (1:1 mapping). `flatMap()` applies a function that returns a stream, then flattens all streams into one (1:many mapping).
```java
// map: ["hello", "world"] → [5, 5] (lengths)
// flatMap: [["h","e","l","l","o"], ["w","o","r","l","d"]] → ["h","e","l","l","o","w","o","r","l","d"]
```

---

### Q2: Can you explain the difference between intermediate and terminal operations?
**A:** **Intermediate** operations (filter, map, sorted) are lazy — they return a new Stream and don't execute until a terminal operation is called. **Terminal** operations (collect, forEach, reduce) trigger the entire pipeline and produce a result or side effect. A stream can only have ONE terminal operation.

---

### Q3: What is a `Predicate`? How do you chain predicates?
**A:** `Predicate<T>` is a functional interface with `test(T) → boolean`. Chain with: `.and()`, `.or()`, `.negate()`.
```java
Predicate<Employee> isAdult = e -> e.getAge() >= 18;
Predicate<Employee> isHighPaid = e -> e.getSalary() > 50000;
Predicate<Employee> combined = isAdult.and(isHighPaid);
```

---

### Q4: What is the difference between `Stream.of()` and `Arrays.stream()`?
**A:** `Stream.of(array)` creates a stream of ONE element (the array itself). `Arrays.stream(array)` creates a stream of array elements. For primitive arrays, `Arrays.stream(int[])` returns `IntStream`, while `Stream.of(int[])` returns `Stream<int[]>`.

---

### Q5: What are records in Java? When to use them?
**A:** Records (Java 16+) are immutable data carriers. They auto-generate constructor, getters (no `get` prefix), `equals()`, `hashCode()`, `toString()`. Use for DTOs, value objects, API responses. Cannot extend other classes (but can implement interfaces). Fields are `final`.

---

### Q6: Explain sealed classes.
**A:** Sealed classes (Java 17) restrict which classes can extend them using the `permits` clause. Subclasses must be `final`, `sealed`, or `non-sealed`. Useful for modeling fixed hierarchies (like algebraic data types). Works great with pattern matching in switch.

---

### Q7: `orElse()` vs `orElseGet()` — what's the difference?
**A:**
```java
// orElse — ALWAYS evaluates the fallback (even if value is present)
String result = optional.orElse(expensiveDefaultComputation());
// expensiveDefaultComputation() runs EVEN if optional has a value!

// orElseGet — evaluates fallback ONLY if value is absent (lazy)
String result = optional.orElseGet(() -> expensiveDefaultComputation());
// Lambda only runs if optional is empty
```

**Rule:** Use `orElse()` for constants (`orElse("default")`). Use `orElseGet()` for computed values, DB calls, or expensive operations.

---

### Q8: Why are streams lazy? What's the benefit?
**A:** Intermediate operations are lazy — they don't execute until a terminal operation is invoked. Benefits:
1. **Efficiency:** Multiple operations fuse into a single pass over data
2. **Short-circuiting:** `findFirst()`, `anyMatch()` stop processing early
3. **Memory:** No intermediate collections created between operations
4. **Infinite streams:** `Stream.iterate()` works because only needed elements are computed

```java
// Without laziness, this would run forever:
Stream.iterate(1, n -> n + 1)  // infinite!
    .filter(n -> n % 2 == 0)
    .limit(5)                   // only takes 5 even numbers
    .forEach(System.out::println);
```

---

### Q9: How does parallel stream work internally?
**A:** Parallel streams use the **ForkJoinPool.commonPool()** (shared across the JVM, default threads = CPU cores - 1). The source is split into chunks using `Spliterator`, each chunk processed by a different thread, and results merged.

**⚠️ Cautions:**
1. Not faster for small datasets (splitting/merging overhead)
2. I/O-bound tasks block the shared pool — use a custom `ForkJoinPool` instead
3. `ArrayList` splits well (indexed), `LinkedList` splits poorly (sequential access)
4. Operations must be stateless, non-interfering, and associative

```java
// Custom ForkJoinPool for parallel stream (isolate from common pool)
ForkJoinPool customPool = new ForkJoinPool(4);
customPool.submit(() ->
    myList.parallelStream().forEach(item -> process(item))
).get();
```

---

### Q10: Explain `reduce()` — what is identity, accumulator, combiner?
**A:**
- **Identity:** Starting value. Must satisfy `op(identity, x) == x`. For sum: `0`. For product: `1`.
- **Accumulator:** Combines current result with next element. `(partialResult, element) -> newResult`.
- **Combiner:** Only used in **parallel streams** to merge results from different threads.

Without a combiner, parallel reduce can't merge partial results and will throw an error for type-changing reductions.

---

### Q11: Can a functional interface have multiple methods?
**A:** Yes, it can have multiple methods, but only **one abstract** method. It can have:
- Any number of `default` methods
- Any number of `static` methods
- Methods inherited from `Object` (`equals`, `hashCode`, `toString`)

`Comparator<T>` is a functional interface with `compare()` as its SAM, plus ~15 default/static methods like `reversed()`, `thenComparing()`, `naturalOrder()`.

---

### Q12: What happens with duplicate keys in `Collectors.toMap()`?
**A:** By default, it throws `IllegalStateException: Duplicate key`. To handle it, provide a merge function as the 3rd argument:
```java
.collect(Collectors.toMap(
    Employee::getDepartment,
    e -> e,
    (existing, replacement) -> existing  // keep existing on conflict
));
```
You can also provide a 4th argument to specify the Map type (`TreeMap::new`, `LinkedHashMap::new`).

---

### Q13: Difference between `Collection.stream()` and `Stream.of()`?
**A:**
- `collection.stream()` — streams over the elements of the collection
- `Stream.of(collection)` — creates a stream containing ONE element (the collection itself)
- `Stream.of("A", "B")` — streams over varargs elements

Same trap with arrays: `Stream.of(intArray)` gives `Stream<int[]>` (one element). Use `Arrays.stream(intArray)` for `IntStream`.

---

### Q14: What Java 9-21 features do you use in real projects?
**A:** Common production-ready features:
- **`var`** (Java 10) — cleaner local variable declarations
- **`List.of()`, `Map.of()`** (Java 9) — immutable collection factories
- **Records** (Java 16) — DTOs, API responses, event objects
- **Text blocks** (Java 15) — multi-line strings for JSON, SQL, HTML
- **Switch expressions** (Java 14) — cleaner branching with arrow syntax
- **Pattern matching `instanceof`** (Java 16) — eliminate explicit casts
- **Sealed classes** (Java 17) — domain modeling with exhaustive switch
- **Virtual threads** (Java 21) — I/O-bound microservices (Spring Boot 3.2+)
- **Sequenced Collections** (Java 21) — `getFirst()`, `getLast()`, `reversed()`

---

> **Next:** [05_Spring_Boot.md](05_Spring_Boot.md) →
