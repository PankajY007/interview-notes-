# 02 — Collections Deep Dive (Interview Ready)

> **Level:** 3-4 Years Experience | **Priority:** 🔴 Critical  
> Covers: List, Set, Map, Queue, HashMap Internals, Concurrent Collections

---

## 📌 Table of Contents

1. [Collections Framework Overview](#1-collections-framework-overview)
2. [List Interface](#2-list-interface)
3. [Set Interface](#3-set-interface) — HashSet internals, EnumSet
4. [Map Interface](#4-map-interface) — LinkedHashMap, NavigableMap, WeakHashMap, EnumMap, IdentityHashMap
5. [HashMap Internal Working (MOST ASKED)](#5-hashmap-internal-working)
6. [Queue & Deque](#6-queue--deque) — poll vs remove, peek vs element
7. [Concurrent Collections](#7-concurrent-collections) — Iterator `modCount` internals
8. [Comparable vs Comparator](#8-comparable-vs-comparator)
9. [Collections Utility Class](#9-collections-utility-class) — Immutable collections comparison
10. [Interview Questions & Answers](#10-interview-questions--answers) — 18 Q&As
11. [Mini Coding Exercises](#11-mini-coding-exercises)

---

## 1. Collections Framework Overview

```
                        Iterable
                           │
                       Collection
                      /    |     \
                   List   Set   Queue
                  / | \    |  \     \
        ArrayList  |  \   |   \   PriorityQueue
        LinkedList |   \  |    \  ArrayDeque
        Vector  CopyOn  HashSet LinkedHashSet
         │     WriteList    |       TreeSet
        Stack              HashMap (via Map)
                           LinkedHashMap
                           TreeMap
                           ConcurrentHashMap
```

### Key Interfaces

| Interface | Ordered? | Duplicates? | Null? | Key Feature |
|-----------|----------|-------------|-------|-------------|
| `List` | ✅ Insertion order | ✅ Yes | ✅ Yes | Index-based access |
| `Set` | ❌ (LinkedHashSet ✅) | ❌ No | ✅ (1 null in HashSet) | Uniqueness |
| `Map` | ❌ (LinkedHashMap ✅) | Keys: ❌, Values: ✅ | 1 null key (HashMap) | Key-Value pairs |
| `Queue` | ✅ FIFO | ✅ Yes | ❌ (PQ) | FIFO processing |

---

## 2. List Interface

### ArrayList vs LinkedList vs Vector

| Feature | ArrayList | LinkedList | Vector |
|---------|-----------|------------|--------|
| Underlying DS | Dynamic Array | Doubly Linked List | Dynamic Array |
| Get (index) | O(1) | O(n) | O(1) |
| Add (end) | O(1) amortized | O(1) | O(1) |
| Add (middle) | O(n) | O(1) if at node | O(n) |
| Remove | O(n) | O(1) if at node | O(n) |
| Thread-safe | ❌ | ❌ | ✅ (synchronized) |
| Memory | Less (contiguous) | More (node overhead) | Same as ArrayList |

### ArrayList Internal Working

```java
// Initial capacity = 10 (or 0 if empty constructor in modern JDK)
// Growth formula: newCapacity = oldCapacity + (oldCapacity >> 1) = 1.5x
// Uses Arrays.copyOf() — copies to new larger array

ArrayList<String> list = new ArrayList<>(); // capacity = 0
list.add("A"); // capacity grows to 10 (default)
// When 11th element added → capacity grows to 15 (10 * 1.5)
// When 16th element added → capacity grows to 22 (15 * 1.5)
```

**⚠️ Production Tip:** If you know the size upfront, set initial capacity:
```java
// BAD — multiple resize operations
List<String> names = new ArrayList<>();

// GOOD — single allocation
List<String> names = new ArrayList<>(1000);
```

### CopyOnWriteArrayList (Thread-safe)
```java
// Creates a new copy of array on every write operation
// Best for: read-heavy, write-rare scenarios (e.g., event listeners)
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.add("A"); // copies entire array
// Iterators see snapshot — no ConcurrentModificationException
```

---

## 3. Set Interface

### HashSet
```java
// Internally uses HashMap (value = dummy PRESENT object)
// No order guarantee
// Allows ONE null
// O(1) for add, remove, contains
HashSet<String> set = new HashSet<>();
set.add("Java");
set.add("Java"); // ignored — duplicate
set.add(null);   // allowed
```

### LinkedHashSet
```java
// Maintains INSERTION order
// Uses LinkedHashMap internally
// Slightly slower than HashSet due to linked list overhead
LinkedHashSet<String> set = new LinkedHashSet<>();
set.add("C");
set.add("A");
set.add("B");
// Iteration order: C, A, B (insertion order)
```

### TreeSet
```java
// Sorted order (natural or custom Comparator)
// Uses Red-Black Tree (TreeMap internally)
// O(log n) for add, remove, contains
// Does NOT allow null (throws NPE in compareTo)
TreeSet<Integer> set = new TreeSet<>();
set.add(30);
set.add(10);
set.add(20);
// Iteration order: 10, 20, 30 (sorted)

// Useful methods
set.first();        // 10
set.last();         // 30
set.headSet(20);    // [10] — elements < 20
set.tailSet(20);    // [20, 30] — elements >= 20
set.subSet(10, 30); // [10, 20] — elements >= 10 and < 30
```

### HashSet Internal Working

```java
// HashSet is backed by a HashMap internally!
public class HashSet<E> {
    private transient HashMap<E, Object> map;
    private static final Object PRESENT = new Object(); // dummy value

    public boolean add(E e) {
        return map.put(e, PRESENT) == null; // key = element, value = PRESENT
    }

    public boolean contains(Object o) {
        return map.containsKey(o);
    }

    public boolean remove(Object o) {
        return map.remove(o) == PRESENT;
    }
}
```

**Key insight:** When you add to a HashSet, it calls `HashMap.put(element, DUMMY_OBJECT)`. Uniqueness is guaranteed because HashMap doesn't allow duplicate keys.

### EnumSet (Fastest Set)

```java
enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

// Uses bit-vector internally — blazing fast (no hashing!)
EnumSet<Day> weekdays = EnumSet.range(Day.MON, Day.FRI);
EnumSet<Day> weekend = EnumSet.of(Day.SAT, Day.SUN);
EnumSet<Day> allDays = EnumSet.allOf(Day.class);
EnumSet<Day> noDays = EnumSet.noneOf(Day.class);
EnumSet<Day> notWeekend = EnumSet.complementOf(weekend); // MON-FRI

// Why use EnumSet?
// ✅ Internally uses a single long (bit-vector) for enums ≤ 64 values
// ✅ add/remove/contains are O(1) bitwise operations
// ✅ Much faster than HashSet<MyEnum>
// ❌ Only works with enum types
```

---

## 4. Map Interface

### Map Implementations Comparison

| Feature | HashMap | LinkedHashMap | TreeMap | Hashtable | ConcurrentHashMap |
|---------|---------|---------------|---------|-----------|-------------------|
| Order | No | Insertion order | Sorted (keys) | No | No |
| Null Key | 1 allowed | 1 allowed | ❌ No | ❌ No | ❌ No |
| Null Value | ✅ Multiple | ✅ Multiple | ❌ No | ❌ No | ❌ No |
| Thread-safe | ❌ | ❌ | ❌ | ✅ (slow) | ✅ (fast) |
| Performance | O(1) | O(1) | O(log n) | O(1) | O(1) |

### LinkedHashMap — Access Order Mode (LRU Cache!)
```java
// LRU Cache using LinkedHashMap
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // true = access order
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}

LRUCache<String, String> cache = new LRUCache<>(3);
cache.put("a", "1");
cache.put("b", "2");
cache.put("c", "3");
cache.get("a");       // "a" moves to end (most recently used)
cache.put("d", "4");  // "b" evicted (least recently used)
```

### NavigableMap / NavigableSet (TreeMap methods)

```java
TreeMap<Integer, String> map = new TreeMap<>();
map.put(10, "A");
map.put(20, "B");
map.put(30, "C");
map.put(40, "D");

map.floorKey(25);     // 20 — greatest key ≤ 25
map.ceilingKey(25);   // 30 — smallest key ≥ 25
map.lowerKey(20);     // 10 — greatest key < 20 (strictly less)
map.higherKey(20);    // 30 — smallest key > 20 (strictly greater)

map.firstEntry();     // 10=A
map.lastEntry();      // 40=D
map.pollFirstEntry(); // removes and returns 10=A

map.headMap(30);      // {10=A, 20=B}        — keys < 30
map.tailMap(30);      // {30=C, 40=D}        — keys ≥ 30
map.subMap(10, 30);   // {10=A, 20=B}        — keys [10, 30)

map.descendingMap();  // {40=D, 30=C, 20=B, 10=A}
```

**Real-world use:** Finding the nearest price tier, scheduling (find next event ≥ current time), range queries.

### WeakHashMap (Keys Eligible for GC)

```java
// Keys are wrapped in WeakReference — GC can reclaim them!
WeakHashMap<Object, String> cache = new WeakHashMap<>();

Object key = new Object();
cache.put(key, "data");
System.out.println(cache.size()); // 1

key = null;        // no strong reference to key anymore
System.gc();       // GC reclaims the key
System.out.println(cache.size()); // 0 (entry automatically removed!)
```

**Use case:** Metadata caches, ClassLoader caches — when you want entries to disappear automatically when keys are no longer referenced elsewhere.

**⚠️ Pitfall:** String literals as keys will NEVER be garbage collected (they live in the String Pool). Use `new String("key")` for testing.

### IdentityHashMap (Uses `==` Instead of `equals()`)

```java
// Normal HashMap: uses equals() and hashCode()
// IdentityHashMap: uses == (reference equality) and System.identityHashCode()

IdentityHashMap<String, Integer> map = new IdentityHashMap<>();
String s1 = new String("hello");
String s2 = new String("hello");

map.put(s1, 1);
map.put(s2, 2);
System.out.println(map.size()); // 2! (different objects, even though equals() is true)

// Normal HashMap would have size 1 (same equals/hashCode)
```

**Use case:** Object graph traversal (tracking visited objects by identity), serialization frameworks.

### EnumMap (Fastest Map for Enum Keys)

```java
enum Status { PENDING, ACTIVE, CLOSED }

// Internally uses a plain array indexed by enum ordinal — O(1) everything
EnumMap<Status, List<Order>> ordersByStatus = new EnumMap<>(Status.class);
ordersByStatus.put(Status.PENDING, new ArrayList<>());
ordersByStatus.put(Status.ACTIVE, new ArrayList<>());

// Why EnumMap over HashMap<Status, ...>?
// ✅ Fixed-size array internally (no hashing, no collisions)
// ✅ Maintains natural enum order
// ✅ ~2x faster than HashMap for enum keys
// ✅ Less memory usage
```

---

## 5. HashMap Internal Working

> 🔴 **This is the #1 most asked Collections topic in Java interviews. Know it inside-out.**

### 5.1 Data Structure

```
HashMap internally uses: Array of Node<K,V> (called "table" or "buckets")

Index:  [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]  ...  [15]
         |         |
        null    Node("A",1) → Node("X",5) → null   (Linked List / Tree)
                  ↑ collision at same bucket
```

### 5.2 Node Structure

```java
static class Node<K, V> implements Map.Entry<K, V> {
    final int hash;    // cached hash
    final K key;
    V value;
    Node<K, V> next;   // linked list pointer
}
```

### 5.3 How `put(key, value)` Works (Step by Step)

```
Step 1: Calculate hash
        hash = key.hashCode()
        hash = hash ^ (hash >>> 16)  // spread bits (perturbation function)

Step 2: Calculate bucket index
        index = hash & (capacity - 1)  // equivalent to hash % capacity
        // Works because capacity is always power of 2

Step 3: Check bucket
        if (bucket is empty)
            → place new Node directly

        if (bucket has nodes)
            → traverse linked list / tree
            → if key.equals(existingKey) → REPLACE value
            → if no match → ADD to end of list

Step 4: Check if treeification needed
        if (linked list length >= 8 AND table size >= 64)
            → convert linked list to Red-Black Tree (O(n) → O(log n))
        if (tree size <= 6)
            → convert back to linked list (untreeify)

Step 5: Check load factor
        if (size > capacity * loadFactor)  // default: 16 * 0.75 = 12
            → RESIZE (double capacity, rehash all entries)
```

### 5.4 How `get(key)` Works

```
Step 1: Calculate hash → bucket index
Step 2: Go to bucket[index]
Step 3: If bucket is empty → return null
Step 4: Compare first node:
        - hash matches AND (key == node.key OR key.equals(node.key))
        - If match → return value
Step 5: If linked list → traverse and compare each node
        If tree → search tree (O(log n))
Step 6: No match found → return null
```

### 5.5 Key Constants

```java
static final int DEFAULT_INITIAL_CAPACITY = 16;     // must be power of 2
static final int MAXIMUM_CAPACITY = 1 << 30;         // 2^30
static final float DEFAULT_LOAD_FACTOR = 0.75f;      // time-space tradeoff
static final int TREEIFY_THRESHOLD = 8;               // list → tree
static final int UNTREEIFY_THRESHOLD = 6;             // tree → list
static final int MIN_TREEIFY_CAPACITY = 64;           // min table size for treeify
```

### 5.6 Resizing (Rehashing)

```
When size > capacity * loadFactor:
1. New array = 2 * old capacity
2. Rehash ALL existing entries
3. Each entry goes to either:
   - Same index (bit is 0)
   - index + oldCapacity (bit is 1)
   
⚠️ Resizing is O(n) — expensive!
Tip: Set initial capacity = (expected entries / 0.75) + 1
```

### 5.7 What Makes a Good HashMap Key?

```java
// ✅ GOOD keys: Immutable objects with proper equals/hashCode
String, Integer, Long, enum types

// ❌ BAD key: Mutable object
class MutableKey {
    int id;
    void setId(int id) { this.id = id; }
    // If you change 'id' after putting in map → lost forever!
}

// Rule: hashCode must be CONSISTENT — same key, same hash, always
```

### 5.8 Visual Example

```java
HashMap<String, Integer> map = new HashMap<>();
map.put("apple", 1);
map.put("banana", 2);
map.put("cherry", 3);

// Let's say:
// "apple".hashCode()  → bucket index 3
// "banana".hashCode() → bucket index 7
// "cherry".hashCode() → bucket index 3 (COLLISION with "apple")

// Bucket array:
// [0] null
// [1] null
// [2] null
// [3] Node("apple",1) → Node("cherry",3) → null  ← collision chain
// [4] null
// [5] null
// [6] null
// [7] Node("banana",2) → null
// ...
```

---

## 6. Queue & Deque

### PriorityQueue
```java
// Min-Heap by default
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(30);
minHeap.offer(10);
minHeap.offer(20);
minHeap.poll(); // 10 (smallest first)

// Max-Heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.offer(30);
maxHeap.offer(10);
maxHeap.poll(); // 30 (largest first)

// Custom object
PriorityQueue<Task> taskQueue = new PriorityQueue<>(
    Comparator.comparingInt(Task::getPriority)
);
```

### Queue Methods: `poll` vs `remove`, `peek` vs `element`

| Operation | Throws Exception | Returns null/false |
|-----------|-----------------|--------------------|
| Insert | `add(e)` — throws `IllegalStateException` if full | `offer(e)` — returns `false` if full |
| Remove | `remove()` — throws `NoSuchElementException` if empty | `poll()` — returns `null` if empty |
| Examine | `element()` — throws `NoSuchElementException` if empty | `peek()` — returns `null` if empty |

**Rule of thumb:** Use `offer/poll/peek` for capacity-constrained or empty-possible queues. Use `add/remove/element` when empty/full is unexpected (you want it to fail loudly).

### ArrayDeque (Stack & Queue replacement)
```java
// Better than Stack class and LinkedList for stack/queue usage
Deque<String> stack = new ArrayDeque<>();
stack.push("A");  // addFirst
stack.push("B");
stack.pop();      // "B" — removeFirst

Deque<String> queue = new ArrayDeque<>();
queue.offer("A"); // addLast
queue.offer("B");
queue.poll();     // "A" — removeFirst
```

**⚠️ Why not use `Stack` class?**
- `Stack` extends `Vector` (synchronized, slow)
- `Stack` allows random access (breaks stack abstraction)
- Use `ArrayDeque` instead — faster, cleaner API

---

## 7. Concurrent Collections

### ConcurrentHashMap

```java
// Segment-based locking (Java 7) → Node-level CAS + synchronized (Java 8+)
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Thread-safe compound operations
map.putIfAbsent("key", 1);
map.computeIfAbsent("key", k -> expensiveComputation(k));
map.merge("key", 1, Integer::sum); // atomic increment

// ❌ Still NOT atomic
if (!map.containsKey("key")) {  // check-then-act race condition!
    map.put("key", value);
}
// ✅ Use putIfAbsent instead
```

### ConcurrentHashMap vs Hashtable vs SynchronizedMap

| Feature | ConcurrentHashMap | Hashtable | SynchronizedMap |
|---------|-------------------|-----------|-----------------|
| Lock granularity | Bucket-level | Entire map | Entire map |
| Read locks | Lock-free (volatile) | Full lock | Full lock |
| Null key/value | ❌ Neither | ❌ Neither | Depends on wrapped map |
| Performance | Best | Worst | Medium |
| Iterator | Weakly consistent | Fail-fast | Fail-fast |

### ConcurrentModificationException
```java
// ❌ WILL throw ConcurrentModificationException
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
for (String s : list) {
    if (s.equals("B")) {
        list.remove(s); // modifying during iteration!
    }
}

// ✅ Fix 1: Use Iterator.remove()
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("B")) {
        it.remove();
    }
}

// ✅ Fix 2: Use removeIf (Java 8+)
list.removeIf(s -> s.equals("B"));

// ✅ Fix 3: Use CopyOnWriteArrayList (thread-safe iteration)
```

### How `modCount` Works (Fail-Fast Internals)

```java
// Inside ArrayList (simplified)
public class ArrayList<E> {
    protected transient int modCount = 0; // modification counter

    public boolean add(E e) {
        modCount++;    // incremented on every structural change
        // ... add element
    }

    public E remove(int index) {
        modCount++;    // incremented
        // ... remove element
    }

    private class Itr implements Iterator<E> {
        int expectedModCount = modCount; // snapshot at iterator creation

        public E next() {
            checkForComodification();  // check EVERY call
            // ... return next element
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)  // list was modified externally!
                throw new ConcurrentModificationException();
        }
    }
}
```

**Why `Iterator.remove()` doesn't throw:** It updates BOTH `modCount` and `expectedModCount` in sync, so the check passes.

---

## 8. Comparable vs Comparator

```java
// Comparable — natural ordering (implemented BY the class)
public class Employee implements Comparable<Employee> {
    private int id;
    private String name;
    private double salary;

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.id, other.id); // natural order by ID
    }
}

// Comparator — external/custom ordering
// Sort by salary descending
Comparator<Employee> bySalaryDesc = (e1, e2) -> Double.compare(e2.getSalary(), e1.getSalary());

// Chained comparators (Java 8+)
Comparator<Employee> complex = Comparator
    .comparing(Employee::getDepartment)
    .thenComparing(Employee::getSalary, Comparator.reverseOrder())
    .thenComparing(Employee::getName);

employees.sort(complex);

// Null-safe comparator
Comparator<Employee> nullSafe = Comparator.nullsLast(
    Comparator.comparing(Employee::getName)
);
```

---

## 9. Collections Utility Class

```java
// Unmodifiable collections
List<String> immutable = Collections.unmodifiableList(list);
// immutable.add("X"); // UnsupportedOperationException

// Java 9+ — factory methods (truly immutable)
List<String> list = List.of("A", "B", "C");
Set<String> set = Set.of("A", "B");
Map<String, Integer> map = Map.of("A", 1, "B", 2);

// Synchronized wrapper
List<String> syncList = Collections.synchronizedList(new ArrayList<>());

// Singleton, empty collections
List<String> empty = Collections.emptyList();
List<String> single = Collections.singletonList("A");

// Sorting
Collections.sort(list);                    // natural order
Collections.sort(list, comparator);        // custom order
list.sort(Comparator.reverseOrder());      // Java 8+

// Frequency, disjoint
Collections.frequency(list, "A");          // count of "A"
Collections.disjoint(list1, list2);        // true if no common elements

// Binary search (list must be sorted!)
int index = Collections.binarySearch(sortedList, "target");
```

### Immutable Collections: `unmodifiableList` vs `List.of` vs `List.copyOf`

```java
List<String> original = new ArrayList<>(List.of("A", "B", "C"));

// 1. Collections.unmodifiableList — VIEW over original (changes reflect!)
List<String> view = Collections.unmodifiableList(original);
original.add("D");
System.out.println(view.size()); // 4! ← changes reflected
// view.add("E");               // ❌ UnsupportedOperationException

// 2. List.of — truly immutable, no nulls allowed (Java 9+)
List<String> immutable = List.of("A", "B", "C");
// immutable.add("D");          // ❌ UnsupportedOperationException
// List.of("A", null);          // ❌ NullPointerException

// 3. List.copyOf — immutable COPY, no nulls (Java 10+)
List<String> copy = List.copyOf(original);
original.add("E");
System.out.println(copy.size()); // 4 ← NOT affected (independent copy)
```

| Method | Immutable? | Reflects changes? | Allows null? | Since |
|--------|-----------|-------------------|-------------|-------|
| `Collections.unmodifiableList()` | View only | ✅ Yes | ✅ Yes | Java 2 |
| `List.of()` | ✅ Truly | N/A (no backing list) | ❌ No | Java 9 |
| `List.copyOf()` | ✅ Truly | ❌ No (independent copy) | ❌ No | Java 10 |

---

## 10. Interview Questions & Answers

### Q1: How does HashMap handle collisions?
**A:** When two keys hash to the same bucket index:
1. **Java 7:** Linked list at the bucket (new entry at head — O(n) worst case)
2. **Java 8+:** Linked list initially → converts to **Red-Black Tree** when chain length ≥ 8 (and table size ≥ 64). This reduces worst-case from O(n) to O(log n). When tree shrinks to ≤ 6 nodes, it converts back to linked list.

---

### Q2: Why is the initial capacity of HashMap always a power of 2?
**A:** Because `index = hash & (capacity - 1)` is used instead of `hash % capacity`. Bitwise AND is faster than modulo. This only works correctly when capacity is a power of 2 (all lower bits are 1 in capacity-1).

---

### Q3: What is the significance of load factor 0.75?
**A:** It's a **tradeoff between time and space:**
- **Lower (0.5):** More space wasted, fewer collisions, faster lookups
- **Higher (1.0):** Less space, more collisions, slower lookups
- **0.75** provides ~3-4% collision rate in a well-distributed hash — optimal balance per statistical analysis.

---

### Q4: Difference between `fail-fast` and `fail-safe` iterators?
**A:**
| Feature | Fail-Fast | Fail-Safe |
|---------|-----------|-----------|
| Behavior | Throws `ConcurrentModificationException` | No exception |
| Examples | `ArrayList`, `HashMap` | `ConcurrentHashMap`, `CopyOnWriteArrayList` |
| How | Checks `modCount` on each next() | Works on clone/snapshot |
| Performance | No extra overhead | Memory overhead (copy) |

---

### Q5: When to use LinkedHashMap over HashMap?
**A:**
1. When you need **insertion order** preserved
2. When implementing **LRU Cache** (access-order mode with `removeEldestEntry`)
3. When iterating order matters (e.g., building JSON response with predictable key order)

---

### Q6: What happens if we put a duplicate key in HashMap?
**A:** The **old value is replaced** with the new value. The `put()` method returns the previous value (or null if no previous mapping). The key itself is NOT replaced (same key object remains).

---

### Q7: Can we use a custom object as HashMap key?
**A:** Yes, but the object MUST:
1. Override `equals()` and `hashCode()` correctly
2. Follow the contract: equal objects must have equal hashcodes
3. Ideally be **immutable** — if the object mutates after being used as a key, the hash changes and the entry becomes unreachable (memory leak)

---

### Q8: Difference between HashMap and TreeMap?
**A:**
- **HashMap:** O(1) get/put, unordered, allows 1 null key, uses hashing
- **TreeMap:** O(log n) get/put, **sorted by keys**, NO null keys, uses Red-Black Tree
- Use TreeMap when you need range queries (`headMap`, `tailMap`, `subMap`) or sorted iteration

---

### Q9: How does ConcurrentHashMap achieve thread-safety without locking entire map?
**A:** In Java 8+, it uses:
1. **CAS (Compare-And-Swap)** for writes to empty buckets — lock-free
2. **synchronized** on the first node of a bucket for collision chains — fine-grained locking
3. **volatile reads** for get operations — no locking needed
4. This allows multiple threads to read/write different buckets simultaneously

---

### Q10: What is the difference between `Iterator` and `ListIterator`?
**A:**
| Feature | Iterator | ListIterator |
|---------|----------|-------------|
| Direction | Forward only | Forward AND Backward |
| Works on | Any Collection | List only |
| Add element | ❌ | ✅ `add()` |
| Set element | ❌ | ✅ `set()` |
| Get index | ❌ | ✅ `nextIndex()`, `previousIndex()` |

---

### Q11: `HashMap` vs `ConcurrentHashMap` — internal locking difference?
**A:**
- **HashMap:** No synchronization. Not thread-safe. Concurrent access can cause infinite loop (Java 7 resize) or data corruption.
- **ConcurrentHashMap (Java 8+):** Uses **CAS for empty buckets** (lock-free), **synchronized on the first node** of a non-empty bucket. Only the affected bucket is locked — other buckets can be accessed concurrently.
- **Key difference:** HashMap locks nothing; ConcurrentHashMap locks at the bucket level (not the entire map).

---

### Q12: Why doesn't `ConcurrentHashMap` allow null keys or values?
**A:** Because of **ambiguity in concurrent context.** If `map.get(key)` returns `null`, you can't tell if:
1. The key maps to `null` (exists with null value), OR
2. The key doesn't exist

In single-threaded HashMap, you'd call `containsKey()` to check — but in a concurrent map, another thread could modify it between `get()` and `containsKey()`. So null is banned to avoid this race condition.

---

### Q13: When should you actually use `LinkedList` over `ArrayList`?
**A:** Almost **never** in practice. ArrayList is faster for:
- Random access: O(1) vs O(n)
- Iteration: better cache locality (contiguous memory)
- Even add/remove in the middle: array copy is CPU-cache-friendly

LinkedList only wins when:
- Frequent add/remove at **both ends** (but `ArrayDeque` is better for this)
- You already have a reference to the node (rare in Java's API)

**Bottom line:** Default to `ArrayList`. Use `ArrayDeque` for stack/queue. `LinkedList` is rarely the right choice.

---

### Q14: How does `TreeMap` maintain sorted order?
**A:** TreeMap uses a **Red-Black Tree** (self-balancing BST). Properties:
1. Every node is either RED or BLACK
2. Root is always BLACK
3. No two adjacent RED nodes
4. Every path from root to null has the same number of BLACK nodes

This guarantees O(log n) height, so `get`, `put`, `remove` are all O(log n). Keys must implement `Comparable` or a `Comparator` must be provided.

---

### Q15: `poll()` vs `remove()`, `peek()` vs `element()` — when to use which?
**A:**
- `poll()` / `peek()` — return `null` on empty queue. Use when empty is a **normal possibility** (e.g., processing a work queue).
- `remove()` / `element()` — throw `NoSuchElementException` on empty. Use when empty is **unexpected/bug** (you want to fail loudly).
- Same logic applies to `offer()` vs `add()` for insertion.

---

### Q16: What is `WeakHashMap`? Give a real-world use case.
**A:** A Map where keys are held via **WeakReferences**. When the key has no strong references elsewhere, the GC can reclaim it, and the entry is automatically removed from the map.

**Use cases:**
1. **Caches** — cached data disappears when the key object is no longer needed
2. **Listener registries** — avoid memory leaks when listeners aren't deregistered
3. **ClassLoader metadata** — JVM uses it internally to track class metadata

⚠️ String literals as keys will never be GC'd (they're in the String Pool permanently).

---

### Q17: What are the different ways to make a collection thread-safe?
**A:**
1. **`Collections.synchronizedList/Map/Set()`** — wraps with synchronized methods (entire collection locked)
2. **`ConcurrentHashMap` / `CopyOnWriteArrayList`** — purpose-built concurrent collections (better performance)
3. **`Vector` / `Hashtable`** — legacy synchronized collections (avoid in new code)
4. **External synchronization** — `synchronized(collection) { ... }` (manual, error-prone)
5. **Immutable collections** — `List.of()`, `Collections.unmodifiableList()` — no writes = no races

**Best practice:** Use `java.util.concurrent` classes (`ConcurrentHashMap`, `CopyOnWriteArrayList`, `BlockingQueue`) over synchronized wrappers.

---

### Q18: `Iterable` vs `Iterator` — what's the difference?
**A:**
| | `Iterable<T>` | `Iterator<T>` |
|---|---|---|
| Package | `java.lang` | `java.util` |
| Method | `iterator()` — returns an `Iterator` | `hasNext()`, `next()`, `remove()` |
| Purpose | "I can be iterated" — a contract | "I am an iterator" — does the work |
| Reusable? | ✅ Can create multiple iterators | ❌ One-time traversal |
| For-each | Enables `for(T item : collection)` | Cannot be used in for-each directly |

```java
// Iterable makes for-each loop possible
public class NumberRange implements Iterable<Integer> {
    private final int start, end;

    public NumberRange(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    public Iterator<Integer> iterator() {
        return new Iterator<>() {
            int current = start;
            public boolean hasNext() { return current <= end; }
            public Integer next() { return current++; }
        };
    }
}

// Now you can do:
for (int n : new NumberRange(1, 5)) {
    System.out.println(n); // 1, 2, 3, 4, 5
}
```

---

## 11. Mini Coding Exercises

### Exercise 1: Find first non-repeating character in a string
```java
public static Character firstNonRepeating(String str) {
    // LinkedHashMap maintains insertion order
    Map<Character, Integer> charCount = new LinkedHashMap<>();
    for (char c : str.toCharArray()) {
        charCount.merge(c, 1, Integer::sum);
    }
    return charCount.entrySet().stream()
        .filter(e -> e.getValue() == 1)
        .map(Map.Entry::getKey)
        .findFirst()
        .orElse(null);
}
// firstNonRepeating("aabbcde") → 'c'
```

### Exercise 2: Find duplicate elements in a list
```java
public static <T> Set<T> findDuplicates(List<T> list) {
    Set<T> seen = new HashSet<>();
    return list.stream()
        .filter(e -> !seen.add(e)) // add returns false if already present
        .collect(Collectors.toSet());
}
```

### Exercise 3: Sort a Map by values
```java
public static <K, V extends Comparable<V>> Map<K, V> sortByValue(Map<K, V> map) {
    return map.entrySet().stream()
        .sorted(Map.Entry.comparingByValue())
        .collect(Collectors.toMap(
            Map.Entry::getKey,
            Map.Entry::getValue,
            (e1, e2) -> e1,
            LinkedHashMap::new // preserve sorted order
        ));
}
```

### Exercise 4: Group employees by department
```java
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// Count per department
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment, Collectors.counting()));

// Average salary per department
Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));
```

---

> **Next:** [03_Multithreading_Concurrency.md](03_Multithreading_Concurrency.md) →
