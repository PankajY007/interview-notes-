# 09 — SQL & PostgreSQL (Interview Ready)

> **Level:** 3-4 Years Experience | **Priority:** 🔴 Critical  
> Covers: Joins, Subqueries, Window Functions, Indexing, Query Optimization, Transactions, PostgreSQL-Specific

---

## 📌 Table of Contents

1. [SQL Fundamentals](#1-sql-fundamentals)
2. [JOINs (Deep Dive)](#2-joins)
3. [Subqueries & CTEs](#3-subqueries--ctes)
4. [Window Functions](#4-window-functions)
5. [Indexing & Query Optimization](#5-indexing--query-optimization)
6. [Transactions & ACID](#6-transactions--acid)
7. [Normalization](#7-normalization)
8. [PostgreSQL-Specific Features](#8-postgresql-specific)
9. [Interview Questions & Answers](#9-interview-questions--answers)
10. [Practice Queries](#10-practice-queries)

---

## 1. SQL Fundamentals

### SQL Execution Order
```
FROM → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT
(Not the same as the writing order!)
```

### DDL, DML, DCL, TCL
| Type | Commands | Purpose |
|------|----------|---------|
| DDL | CREATE, ALTER, DROP, TRUNCATE | Structure |
| DML | SELECT, INSERT, UPDATE, DELETE | Data |
| DCL | GRANT, REVOKE | Permissions |
| TCL | COMMIT, ROLLBACK, SAVEPOINT | Transactions |

### DELETE vs TRUNCATE vs DROP
| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| WHERE clause | ✅ Yes | ❌ No | ❌ No |
| Rollback | ✅ Yes | ❌ No (usually) | ❌ No |
| Triggers fired | ✅ Yes | ❌ No | ❌ No |
| Speed | Slow (row by row) | Fast | Fast |
| Auto-increment | Preserved | Reset | N/A |
| Structure | Preserved | Preserved | Removed |

---

## 2. JOINs

### Sample Tables
```sql
-- employees
| id | name    | dept_id |
|----|---------|---------|
| 1  | Alice   | 10      |
| 2  | Bob     | 20      |
| 3  | Charlie | 10      |
| 4  | Diana   | NULL    |

-- departments
| id | name       |
|----|------------|
| 10 | Engineering|
| 20 | Marketing  |
| 30 | HR         |
```

### INNER JOIN — only matching rows
```sql
SELECT e.name, d.name AS department
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;

-- Result: Alice-Engineering, Bob-Marketing, Charlie-Engineering
-- Diana excluded (NULL dept_id), HR excluded (no employees)
```

### LEFT JOIN — all from left + matching from right
```sql
SELECT e.name, d.name AS department
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;

-- Result: includes Diana with NULL department
```

### RIGHT JOIN — all from right + matching from left
```sql
SELECT e.name, d.name AS department
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.id;

-- Result: includes HR with NULL employee
```

### FULL OUTER JOIN — all from both sides
```sql
SELECT e.name, d.name AS department
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.id;

-- Result: Diana (NULL dept) AND HR (NULL employee) both included
```

### CROSS JOIN — Cartesian product
```sql
SELECT e.name, d.name FROM employees e CROSS JOIN departments d;
-- 4 employees × 3 departments = 12 rows
```

### SELF JOIN
```sql
-- Find employees and their managers
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### ⚠️ Pitfall: JOIN vs WHERE for filtering
```sql
-- These are DIFFERENT for LEFT JOIN:
-- Filter in ON — preserves LEFT table rows
SELECT * FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id AND d.name = 'Engineering';
-- Diana still appears (LEFT preserved)

-- Filter in WHERE — filters AFTER join
SELECT * FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id
WHERE d.name = 'Engineering';
-- Diana excluded (WHERE filters null rows)
```

---

## 3. Subqueries & CTEs

### Subqueries
```sql
-- Scalar subquery (returns single value)
SELECT name, salary,
       (SELECT AVG(salary) FROM employees) AS avg_salary
FROM employees;

-- IN subquery
SELECT name FROM employees
WHERE dept_id IN (SELECT id FROM departments WHERE location = 'NYC');

-- EXISTS (more efficient than IN for large datasets)
SELECT name FROM employees e
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.employee_id = e.id);

-- Correlated subquery (references outer query)
SELECT name, salary,
       (SELECT COUNT(*) FROM employees e2 WHERE e2.salary > e1.salary) AS rank
FROM employees e1;
```

### Common Table Expressions (CTEs) — More Readable
```sql
WITH high_earners AS (
    SELECT name, salary, department
    FROM employees
    WHERE salary > 80000
),
dept_stats AS (
    SELECT department, COUNT(*) as count, AVG(salary) as avg_sal
    FROM employees
    GROUP BY department
)
SELECT h.name, h.salary, d.avg_sal
FROM high_earners h
JOIN dept_stats d ON h.department = d.department;
```

### Recursive CTE — Hierarchical Data
```sql
-- Employee → Manager hierarchy
WITH RECURSIVE org_chart AS (
    -- Base case: top-level managers (no manager)
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case
    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level;
```

---

## 4. Window Functions

> 🔴 **Heavily asked in interviews. Master these!**

### Syntax
```sql
function_name() OVER (
    PARTITION BY column       -- group rows (optional)
    ORDER BY column           -- order within partition
    ROWS/RANGE frame_clause   -- window frame (optional)
)
```

### ROW_NUMBER, RANK, DENSE_RANK
```sql
SELECT name, department, salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as row_num,
    RANK()       OVER (PARTITION BY department ORDER BY salary DESC) as rank,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dense_rank
FROM employees;

-- If two employees tie at salary 80000:
-- ROW_NUMBER: 1, 2 (always unique)
-- RANK:       1, 1, 3 (skips numbers after tie)
-- DENSE_RANK: 1, 1, 2 (no gaps)
```

### LAG, LEAD — Access adjacent rows
```sql
SELECT name, salary,
    LAG(salary, 1, 0) OVER (ORDER BY salary) AS prev_salary,
    LEAD(salary, 1, 0) OVER (ORDER BY salary) AS next_salary,
    salary - LAG(salary, 1, 0) OVER (ORDER BY salary) AS diff
FROM employees;
```

### Running Totals & Aggregates
```sql
SELECT date, amount,
    SUM(amount) OVER (ORDER BY date) AS running_total,
    AVG(amount) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg_3day,
    SUM(amount) OVER (PARTITION BY department ORDER BY date) AS dept_running_total
FROM transactions;
```

### NTILE — Divide into groups
```sql
SELECT name, salary,
    NTILE(4) OVER (ORDER BY salary DESC) AS quartile
FROM employees;
-- Divides into 4 equal groups (Q1=highest, Q4=lowest)
```

### Common Interview Pattern: Second/Nth Highest Salary
```sql
-- Method 1: DENSE_RANK
SELECT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rnk
    FROM employees
) ranked
WHERE rnk = 2;

-- Method 2: OFFSET
SELECT DISTINCT salary FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- Method 3: Subquery
SELECT MAX(salary) FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

---

## 5. Indexing & Query Optimization

### Index Types
```sql
-- B-Tree (default) — best for equality and range queries
CREATE INDEX idx_employee_name ON employees(name);

-- Composite index — order matters!
CREATE INDEX idx_dept_salary ON employees(department, salary);
-- Useful for: WHERE department = 'IT' AND salary > 50000
-- Useful for: WHERE department = 'IT' (leftmost prefix)
-- NOT useful for: WHERE salary > 50000 (skips first column)

-- Unique index
CREATE UNIQUE INDEX idx_employee_email ON employees(email);

-- Partial index (PostgreSQL)
CREATE INDEX idx_active_employees ON employees(name) WHERE active = true;

-- GIN index (for full-text search, JSONB, arrays)
CREATE INDEX idx_tags ON products USING GIN(tags);
```

### EXPLAIN ANALYZE
```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE name = 'Alice';
-- Shows: Seq Scan vs Index Scan, actual time, rows examined

-- Key terms:
-- Seq Scan — full table scan (bad for large tables)
-- Index Scan — uses index (good)
-- Index Only Scan — covered by index (best)
-- Nested Loop — small result sets
-- Hash Join — medium datasets
-- Merge Join — sorted large datasets
```

### Query Optimization Tips
```sql
-- ❌ BAD: Function on indexed column prevents index usage
SELECT * FROM employees WHERE UPPER(name) = 'ALICE';
-- ✅ GOOD: Index on expression or avoid function
CREATE INDEX idx_name_upper ON employees(UPPER(name));

-- ❌ BAD: SELECT *
SELECT * FROM employees; -- fetches all columns including BLOBs
-- ✅ GOOD: Select only needed columns
SELECT id, name, email FROM employees;

-- ❌ BAD: NOT IN with subquery
SELECT * FROM employees WHERE id NOT IN (SELECT employee_id FROM orders);
-- ✅ GOOD: NOT EXISTS or LEFT JOIN IS NULL
SELECT e.* FROM employees e
LEFT JOIN orders o ON e.id = o.employee_id
WHERE o.id IS NULL;

-- ❌ BAD: LIKE '%text%' (can't use index)
-- ✅ OK: LIKE 'text%' (can use index)

-- Use LIMIT for pagination
SELECT * FROM employees ORDER BY id LIMIT 20 OFFSET 40;
-- Better: Keyset pagination for large datasets
SELECT * FROM employees WHERE id > 40 ORDER BY id LIMIT 20;
```

---

## 6. Transactions & ACID

| Property | Meaning | Example |
|----------|---------|---------|
| **Atomicity** | All or nothing | Transfer: debit + credit both succeed or both fail |
| **Consistency** | DB rules maintained | Balance can't go negative (constraints) |
| **Isolation** | Concurrent txns don't interfere | Two transfers don't see half-done state |
| **Durability** | Committed data survives crash | Written to disk/WAL |

### Isolation Level Problems
| Problem | Description |
|---------|-------------|
| **Dirty Read** | Read uncommitted data from another txn |
| **Non-Repeatable Read** | Same query returns different data within one txn |
| **Phantom Read** | New rows appear in repeated query |

### PostgreSQL Default: READ COMMITTED
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- If anything fails:
ROLLBACK;
-- If everything succeeds:
COMMIT;

-- Savepoints
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SAVEPOINT sp1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- Oops, wrong account:
ROLLBACK TO SAVEPOINT sp1;
UPDATE accounts SET balance = balance + 100 WHERE id = 3;
COMMIT;
```

---

## 7. Normalization

| Form | Rule | Example Fix |
|------|------|-------------|
| **1NF** | No repeating groups, atomic values | Split "John,Jane" into separate rows |
| **2NF** | 1NF + no partial dependency on composite key | Move non-key-dependent columns to new table |
| **3NF** | 2NF + no transitive dependency | If A→B→C, move B→C to separate table |
| **BCNF** | Every determinant is a candidate key | Stricter 3NF |

### When to Denormalize?
- Read-heavy systems (reporting, analytics)
- When JOINs are too expensive
- Caching layers / materialized views
- NoSQL databases

---

## 8. PostgreSQL-Specific

```sql
-- JSONB support
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    data JSONB
);
INSERT INTO products (data) VALUES ('{"name": "Laptop", "specs": {"ram": 16}}');
SELECT data->>'name' FROM products;                  -- "Laptop"
SELECT data->'specs'->>'ram' FROM products;           -- "16"
SELECT * FROM products WHERE data @> '{"name": "Laptop"}'; -- containment

-- Arrays
CREATE TABLE users (id SERIAL, tags TEXT[]);
INSERT INTO users (tags) VALUES (ARRAY['java', 'spring']);
SELECT * FROM users WHERE 'java' = ANY(tags);

-- UPSERT (INSERT ON CONFLICT)
INSERT INTO employees (email, name, salary)
VALUES ('john@email.com', 'John', 50000)
ON CONFLICT (email) DO UPDATE SET salary = EXCLUDED.salary;

-- Materialized View (cached query result)
CREATE MATERIALIZED VIEW dept_summary AS
SELECT department, COUNT(*), AVG(salary) FROM employees GROUP BY department;
REFRESH MATERIALIZED VIEW dept_summary; -- manually refresh

-- LATERAL JOIN (correlated subquery in FROM)
SELECT d.name, e.top_earner
FROM departments d,
LATERAL (
    SELECT name AS top_earner FROM employees 
    WHERE dept_id = d.id ORDER BY salary DESC LIMIT 1
) e;
```

---

## 9. Interview Questions & Answers

### Q1: Write a query to find the second highest salary.
```sql
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 1;
-- OR
SELECT MAX(salary) FROM employees WHERE salary < (SELECT MAX(salary) FROM employees);
```

### Q2: Find duplicate records.
```sql
SELECT email, COUNT(*) FROM employees GROUP BY email HAVING COUNT(*) > 1;
```

### Q3: Delete duplicate rows keeping the one with lowest ID.
```sql
DELETE FROM employees WHERE id NOT IN (
    SELECT MIN(id) FROM employees GROUP BY email
);
```

### Q4: Find employees with salary greater than their department average.
```sql
SELECT e.name, e.salary, e.department
FROM employees e
JOIN (SELECT department, AVG(salary) AS avg_sal FROM employees GROUP BY department) d
ON e.department = d.department
WHERE e.salary > d.avg_sal;
-- OR with window function:
SELECT name, salary, department FROM (
    SELECT *, AVG(salary) OVER (PARTITION BY department) AS dept_avg
    FROM employees
) t WHERE salary > dept_avg;
```

### Q5: What is the difference between WHERE and HAVING?
**A:** `WHERE` filters rows BEFORE grouping. `HAVING` filters groups AFTER `GROUP BY`. You can't use aggregate functions in `WHERE` but you can in `HAVING`.

### Q6: What is a deadlock in SQL? How do you prevent it?
**A:** Two transactions waiting for each other's locks. Prevention: consistent lock ordering, use shorter transactions, avoid long-held locks, use `SELECT ... FOR UPDATE NOWAIT`.

---

## 10. Practice Queries

### Get top 3 earners per department
```sql
SELECT * FROM (
    SELECT name, department, salary,
           ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employees
) ranked WHERE rn <= 3;
```

### Year-over-year growth
```sql
WITH yearly AS (
    SELECT EXTRACT(YEAR FROM order_date) AS year,
           SUM(amount) AS total
    FROM orders
    GROUP BY EXTRACT(YEAR FROM order_date)
)
SELECT year, total,
       LAG(total) OVER (ORDER BY year) AS prev_year,
       ROUND(((total - LAG(total) OVER (ORDER BY year)) / 
              LAG(total) OVER (ORDER BY year)) * 100, 2) AS growth_pct
FROM yearly;
```

### Find gaps in sequential IDs
```sql
SELECT id + 1 AS gap_start,
       LEAD(id) OVER (ORDER BY id) - 1 AS gap_end
FROM employees
WHERE LEAD(id) OVER (ORDER BY id) - id > 1;
```

---

> **Next:** [10_Git_Version_Control.md](10_Git_Version_Control.md) →
