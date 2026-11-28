# SQL JOINs: An In-Depth Guide

Covers: `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`, `SELF JOIN`, `CROSS JOIN`

---

## 0. What is a JOIN?

A relational database stores data in multiple tables to avoid duplication (normalization). A **JOIN** combines rows from two (or more) tables into a single result set, based on a related column, usually a **primary key / foreign key** pair.

Conceptually, every join does this:

1. Take the **left table** and the **right table**.
2. Compare rows using a **join condition** (the `ON` clause).
3. Produce output rows according to the join type, which decides what happens to rows that **match** and rows that **don't match**.

### Generic syntax

```sql
SELECT   columns
FROM     table_a
<JOIN TYPE> table_b
    ON   table_a.col = table_b.col;
```

Line by line:

| Line | Meaning |
|---|---|
| `SELECT columns` | Which columns appear in the result. Can come from either table. |
| `FROM table_a` | The **left** table (the one written first). |
| `<JOIN TYPE> table_b` | The **right** table and how to join it. |
| `ON table_a.col = table_b.col` | The **join condition**: decides which rows pair up. |

---

## 1. Sample Data (used throughout)

### `departments`

| dept_id | dept_name   |
|---------|-------------|
| 1       | Engineering |
| 2       | Sales       |
| 3       | HR          |
| 4       | Finance     |

### `employees`

| emp_id | emp_name | dept_id | manager_id |
|--------|----------|---------|------------|
| 101    | Asha     | 1       | NULL       |
| 102    | Ravi     | 1       | 101        |
| 103    | Meera    | 2       | 101        |
| 104    | Karan    | NULL    | 102        |
| 105    | Divya    | 2       | 103        |

### Things to notice (these make the joins interesting)

- **Karan** has `dept_id = NULL`: an employee with **no department**.
- **HR (3)** and **Finance (4)** have **no employees**: departments with **no matching employee**.
- **Asha** has `manager_id = NULL`: she is the top of the hierarchy.

### Setup script

```sql
CREATE TABLE departments (
    dept_id   INT PRIMARY KEY,
    dept_name VARCHAR(50) NOT NULL
);

CREATE TABLE employees (
    emp_id     INT PRIMARY KEY,
    emp_name   VARCHAR(50) NOT NULL,
    dept_id    INT NULL,
    manager_id INT NULL,
    FOREIGN KEY (dept_id)    REFERENCES departments(dept_id),
    FOREIGN KEY (manager_id) REFERENCES employees(emp_id)
);

INSERT INTO departments VALUES (1,'Engineering'),(2,'Sales'),(3,'HR'),(4,'Finance');

INSERT INTO employees VALUES
 (101,'Asha',  1,    NULL),
 (102,'Ravi',  1,    101),
 (103,'Meera', 2,    101),
 (104,'Karan', NULL, 102),
 (105,'Divya', 2,    103);
```

---

## 2. INNER JOIN

### Definition
Returns **only rows that have a match in both tables**. Rows with no match on either side are discarded.

`INNER` is optional: `JOIN` alone means `INNER JOIN`.

### Visual

```
   employees        departments
  ┌─────────┐      ┌─────────┐
  │    ┌────┼──────┼───┐     │
  │    │////│██████│///│     │      ██ = rows returned
  │    └────┼──────┼───┘     │
  └─────────┘      └─────────┘
          only the overlap
```

### Query

```sql
SELECT  e.emp_name,
        d.dept_name
FROM    employees   e
INNER JOIN departments d
        ON e.dept_id = d.dept_id;
```

### Line-by-line breakdown

| Line | What happens |
|---|---|
| `SELECT e.emp_name, d.dept_name` | Pick one column from each table. The `e.` / `d.` prefixes are **aliases** that say which table the column comes from. |
| `FROM employees e` | `employees` is the left table; `e` is its alias. |
| `INNER JOIN departments d` | Bring in `departments` (alias `d`) and keep only matching pairs. |
| `ON e.dept_id = d.dept_id` | A pair matches when the employee's `dept_id` equals the department's `dept_id`. |

### How the database evaluates it (row by row)

| Employee | dept_id | Match in `departments`? | In result? |
|---|---|---|---|
| Asha  | 1    | Yes → Engineering | ✅ |
| Ravi  | 1    | Yes → Engineering | ✅ |
| Meera | 2    | Yes → Sales       | ✅ |
| Karan | NULL | **No** (NULL equals nothing) | ❌ |
| Divya | 2    | Yes → Sales       | ✅ |

HR and Finance are never matched by any employee, so they are also dropped.

### Result (4 rows)

| emp_name | dept_name   |
|----------|-------------|
| Asha     | Engineering |
| Ravi     | Engineering |
| Meera    | Sales       |
| Divya    | Sales       |

### When to use
- You only care about records that have a relationship on **both** sides.
- Example: "List employees together with their department", where employees without a department aren't interesting.

---

## 3. LEFT JOIN (LEFT OUTER JOIN)

### Definition
Returns **all rows from the left table**, plus matching rows from the right table. If there's no match, the right-side columns are filled with `NULL`.

`OUTER` is optional: `LEFT JOIN` = `LEFT OUTER JOIN`.

### Visual

```
   employees        departments
  ┌─────────┐      ┌─────────┐
  │████┌────┼──────┼───┐     │
  │████│████│██████│///│     │      ██ = rows returned
  │████└────┼──────┼───┘     │
  └─────────┘      └─────────┘
  all of left + overlap
```

### Query

```sql
SELECT  e.emp_name,
        d.dept_name
FROM    employees   e
LEFT JOIN departments d
        ON e.dept_id = d.dept_id;
```

### Line-by-line breakdown

| Line | What happens |
|---|---|
| `FROM employees e` | **Left table.** Every row of it is guaranteed to appear in the output. |
| `LEFT JOIN departments d` | Right table is optional; if no match exists, its columns become `NULL`. |
| `ON e.dept_id = d.dept_id` | Same matching rule as before. |

### Row-by-row evaluation

| Employee | dept_id | Match? | Output |
|---|---|---|---|
| Asha  | 1    | Yes | Asha, Engineering |
| Ravi  | 1    | Yes | Ravi, Engineering |
| Meera | 2    | Yes | Meera, Sales |
| Karan | NULL | **No** | Karan, **NULL** ← preserved because he's on the left |
| Divya | 2    | Yes | Divya, Sales |

### Result (5 rows)

| emp_name | dept_name   |
|----------|-------------|
| Asha     | Engineering |
| Ravi     | Engineering |
| Meera    | Sales       |
| Karan    | NULL        |
| Divya    | Sales       |

### Pattern: finding rows with NO match (anti-join)

```sql
SELECT  e.emp_name
FROM    employees e
LEFT JOIN departments d
        ON e.dept_id = d.dept_id
WHERE   d.dept_id IS NULL;
```

| Line | Meaning |
|---|---|
| `LEFT JOIN ...` | Keep all employees, with `NULL`s where no department matched. |
| `WHERE d.dept_id IS NULL` | Keep **only** the rows where the join failed to find a department. |

Result: `Karan`: the employee without a department.

> Use `IS NULL`, never `= NULL`. Comparing anything to `NULL` with `=` yields *unknown*, not true.

### When to use
- "Show **all** customers, even those with no orders."
- "Find records in A that have **no** counterpart in B" (the anti-join above).

---

## 4. RIGHT JOIN (RIGHT OUTER JOIN)

### Definition
The mirror image of `LEFT JOIN`: returns **all rows from the right table**, plus matching rows from the left. Unmatched left-side columns become `NULL`.

### Visual

```
   employees        departments
  ┌─────────┐      ┌─────────┐
  │    ┌────┼──────┼───┐█████│
  │    │████│██████│███│█████│      ██ = rows returned
  │    └────┼──────┼───┘█████│
  └─────────┘      └─────────┘
  overlap + all of right
```

### Query

```sql
SELECT  e.emp_name,
        d.dept_name
FROM    employees   e
RIGHT JOIN departments d
        ON e.dept_id = d.dept_id;
```

### Line-by-line breakdown

| Line | What happens |
|---|---|
| `FROM employees e` | Left table. Now the *optional* side. |
| `RIGHT JOIN departments d` | Right table. Every department is guaranteed to appear. |
| `ON ...` | Matching rule. |

### Row-by-row evaluation (driven by the right table)

| Department | Employees with that dept_id | Output |
|---|---|---|
| Engineering (1) | Asha, Ravi | Asha, Engineering / Ravi, Engineering |
| Sales (2)       | Meera, Divya | Meera, Sales / Divya, Sales |
| HR (3)          | none | **NULL**, HR |
| Finance (4)     | none | **NULL**, Finance |

### Result (6 rows)

| emp_name | dept_name   |
|----------|-------------|
| Asha     | Engineering |
| Ravi     | Engineering |
| Meera    | Sales       |
| Divya    | Sales       |
| NULL     | HR          |
| NULL     | Finance     |

Karan is **missing**: he has no department, and Karan is on the left (the optional side).

### RIGHT JOIN ≡ LEFT JOIN with tables swapped

```sql
-- These two are identical:
FROM employees e RIGHT JOIN departments d ON e.dept_id = d.dept_id
FROM departments d LEFT JOIN employees e ON e.dept_id = d.dept_id
```

Most teams standardize on `LEFT JOIN` only, because reading queries left-to-right ("start with the main table, optionally attach others") is easier. `RIGHT JOIN` is rare in practice.

### When to use
- "Show **all** departments, even the empty ones."

---

## 5. FULL OUTER JOIN

### Definition
Returns **all rows from both tables**. Where rows match, they're combined; where they don't, the missing side is `NULL`. It's effectively `LEFT JOIN` ∪ `RIGHT JOIN`.

### Visual

```
   employees        departments
  ┌─────────┐      ┌─────────┐
  │████┌────┼──────┼───┐█████│
  │████│████│██████│███│█████│      ██ = rows returned
  │████└────┼──────┼───┘█████│
  └─────────┘      └─────────┘
        everything
```

### Query

```sql
SELECT  e.emp_name,
        d.dept_name
FROM    employees   e
FULL OUTER JOIN departments d
        ON e.dept_id = d.dept_id;
```

### Line-by-line breakdown

| Line | What happens |
|---|---|
| `FULL OUTER JOIN departments d` | Neither side is optional-only; unmatched rows from **both** tables are kept. |
| `ON ...` | Matching rule. |

### Result (7 rows)

| emp_name | dept_name   | Why it's here |
|----------|-------------|---------------|
| Asha     | Engineering | matched |
| Ravi     | Engineering | matched |
| Meera    | Sales       | matched |
| Divya    | Sales       | matched |
| Karan    | NULL        | left-only (no department) |
| NULL     | HR          | right-only (no employees) |
| NULL     | Finance     | right-only (no employees) |

### Database support: important!

| Database | `FULL OUTER JOIN` supported? |
|---|---|
| PostgreSQL, SQL Server, Oracle, SQLite (3.39+) | ✅ Yes |
| **MySQL / MariaDB** | ❌ **No**, must emulate |

### Emulating FULL OUTER JOIN in MySQL

```sql
SELECT e.emp_name, d.dept_name
FROM   employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id

UNION

SELECT e.emp_name, d.dept_name
FROM   employees e
RIGHT JOIN departments d ON e.dept_id = d.dept_id;
```

| Part | Meaning |
|---|---|
| First `SELECT` (LEFT JOIN) | All employees + their departments (includes Karan). |
| `UNION` | Combines both result sets **and removes duplicates** (the matched rows appear in both queries). |
| Second `SELECT` (RIGHT JOIN) | All departments + their employees (includes HR, Finance). |

Use `UNION ALL` plus an anti-join filter on the second query if you need to avoid the cost of de-duplication.

### When to use
- Reconciliation / comparison: "Which records exist in A only, B only, or both?"
- Data migration audits.

---

## 6. SELF JOIN

### Definition
A **table joined to itself**. It is not a separate keyword: you use any join type (`INNER`, `LEFT`, ...) but reference the same table twice with **different aliases**.

Typical use: **hierarchies**, such as employee → manager, category → parent category, or comparing rows within the same table.

### Query: employee and their manager

```sql
SELECT  e.emp_name  AS employee,
        m.emp_name  AS manager
FROM    employees e
LEFT JOIN employees m
        ON e.manager_id = m.emp_id;
```

### Line-by-line breakdown

| Line | What happens |
|---|---|
| `FROM employees e` | First "copy" of the table: plays the role of the **employee**. |
| `LEFT JOIN employees m` | Second "copy" of the **same** table: plays the role of the **manager**. Aliases `e` and `m` are *mandatory* so the database can tell them apart. |
| `ON e.manager_id = m.emp_id` | An employee's `manager_id` points to the manager's `emp_id`. |
| `LEFT` (not `INNER`) | Keeps employees with no manager (Asha, the top boss). `INNER` would drop her. |

### Row-by-row evaluation

| e.emp_name | e.manager_id | Matching `m.emp_id` | m.emp_name |
|---|---|---|---|
| Asha  | NULL | none | NULL |
| Ravi  | 101  | 101  | Asha |
| Meera | 101  | 101  | Asha |
| Karan | 102  | 102  | Ravi |
| Divya | 103  | 103  | Meera |

### Result

| employee | manager |
|----------|---------|
| Asha     | NULL    |
| Ravi     | Asha    |
| Meera    | Asha    |
| Karan    | Ravi    |
| Divya    | Meera   |

### Another self join: pairs in the same department

```sql
SELECT  a.emp_name AS emp1,
        b.emp_name AS emp2,
        a.dept_id
FROM    employees a
JOIN    employees b
        ON  a.dept_id = b.dept_id
        AND a.emp_id  < b.emp_id;
```

| Condition | Purpose |
|---|---|
| `a.dept_id = b.dept_id` | Same department. |
| `a.emp_id < b.emp_id` | Avoids pairing a person with **themselves** and avoids **duplicate mirrored pairs** (Asha-Ravi and Ravi-Asha). |

Result: `(Asha, Ravi, 1)` and `(Meera, Divya, 2)`.

> Hierarchies deeper than one level (whole chain of command) need a **recursive CTE** (`WITH RECURSIVE`), since each self join only walks one level.

---

## 7. CROSS JOIN

### Definition
Returns the **Cartesian product**: every row of the left table paired with every row of the right table. There is **no join condition**.

Row count = `rows(A) × rows(B)`.

### Query

```sql
SELECT  e.emp_name,
        d.dept_name
FROM    employees   e
CROSS JOIN departments d;
```

### Line-by-line breakdown

| Line | What happens |
|---|---|
| `CROSS JOIN departments d` | Pair every employee with every department. |
| *(no `ON`)* | Not allowed/needed: no matching is performed. |

### Result size

`5 employees × 4 departments = 20 rows` (first few shown):

| emp_name | dept_name   |
|----------|-------------|
| Asha     | Engineering |
| Asha     | Sales       |
| Asha     | HR          |
| Asha     | Finance     |
| Ravi     | Engineering |
| Ravi     | Sales       |
| ...      | ...         |

### Equivalent older syntax (avoid)

```sql
SELECT e.emp_name, d.dept_name
FROM   employees e, departments d;      -- comma join = CROSS JOIN
```

⚠️ **Danger:** forgetting the `WHERE`/`ON` in a comma join silently produces a Cartesian product. The explicit `CROSS JOIN` keyword makes the intent clear.

### Realistic use case: generate all combinations

```sql
-- Table sizes(size): S, M, L      Table colors(color): Red, Blue
SELECT s.size, c.color
FROM   sizes s
CROSS JOIN colors c;
-- 3 × 2 = 6 rows: S-Red, S-Blue, M-Red, M-Blue, L-Red, L-Blue
```

Other uses: building a calendar grid (dates × stores), test data generation, product variant matrices.

### Performance warning
Row counts multiply fast: 100,000 × 100,000 = **10 billion rows**. Use only when you genuinely need all combinations.

---

## 8. Side-by-Side Comparison

| Join | Left-only rows | Matched rows | Right-only rows | Row count (our data) |
|---|:---:|:---:|:---:|:---:|
| `INNER JOIN` | ❌ | ✅ | ❌ | 4 |
| `LEFT JOIN`  | ✅ | ✅ | ❌ | 5 |
| `RIGHT JOIN` | ❌ | ✅ | ✅ | 6 |
| `FULL OUTER JOIN` | ✅ | ✅ | ✅ | 7 |
| `CROSS JOIN` | n/a (every combination) | n/a | n/a | 20 |
| `SELF JOIN` | depends on the join type used (table joined to itself) | | | 5 (with LEFT) |

### Quick decision guide

| You want... | Use |
|---|---|
| Only rows that relate on both sides | `INNER JOIN` |
| Everything from A, plus B if available | `LEFT JOIN` |
| Everything from B, plus A if available | `RIGHT JOIN` (or swap and use `LEFT`) |
| Everything from both, matched where possible | `FULL OUTER JOIN` |
| Rows related to other rows in the same table | `SELF JOIN` |
| Every possible combination | `CROSS JOIN` |
| Rows in A with **no** match in B | `LEFT JOIN ... WHERE b.key IS NULL` |

---

## 9. Critical Concepts & Gotchas

### 9.1 `ON` vs `WHERE` with outer joins

This is the most common source of bugs.

```sql
-- Version A: filter in ON
SELECT e.emp_name, d.dept_name
FROM   employees e
LEFT JOIN departments d
       ON e.dept_id = d.dept_id AND d.dept_name = 'Sales';

-- Version B: filter in WHERE
SELECT e.emp_name, d.dept_name
FROM   employees e
LEFT JOIN departments d
       ON e.dept_id = d.dept_id
WHERE  d.dept_name = 'Sales';
```

| Version | Behavior | Rows |
|---|---|---|
| A (`ON`) | Filter is applied **while matching**. Employees outside Sales still appear, with `dept_name = NULL`. | **5** (all employees) |
| B (`WHERE`) | Filter is applied **after** the join, discarding every row where `dept_name` isn't Sales (including the `NULL` rows). This silently turns the `LEFT JOIN` into an `INNER JOIN`. | **2** (Meera, Divya) |

**Rule:** for outer joins, put conditions about the *optional* table in `ON`; put conditions about the *preserved* table in `WHERE`.

### 9.2 NULLs never match in a join

`NULL = NULL` is *unknown*, not true. Two rows with `NULL` join keys will **not** match each other. That's why Karan didn't match anything.

### 9.3 One-to-many multiplies rows

If one department has 3 employees, joining departments → employees yields 3 rows for that department. Aggregating afterward without care (e.g. `SUM` on a department-level column) will **double-count**. Use `GROUP BY` or aggregate in a subquery first.

### 9.4 Ambiguous column names

Both tables have `dept_id`. Writing `SELECT dept_id` raises an *ambiguous column* error. Always qualify: `e.dept_id` or `d.dept_id`.

### 9.5 `USING` and `NATURAL JOIN` shortcuts

```sql
-- USING: when the join column has the same name in both tables
SELECT emp_name, dept_name
FROM   employees
JOIN   departments USING (dept_id);
```

`USING (dept_id)` replaces `ON e.dept_id = d.dept_id` and returns the column **once**.
`NATURAL JOIN` joins on *all* same-named columns automatically; it is risky because adding a column later can silently change the query, so avoid it in production code.

### 9.6 Joining more than two tables

```sql
SELECT o.order_id, c.name, p.product_name
FROM   orders o
JOIN   customers c ON o.customer_id = c.customer_id
JOIN   products  p ON o.product_id  = p.product_id;
```

Joins are evaluated left to right logically (the optimizer may reorder them). Mixing `INNER` and `LEFT` joins in a chain needs care: an `INNER JOIN` placed after a `LEFT JOIN` on the same optional table can eliminate the preserved `NULL` rows.

### 9.7 Performance tips

- **Index the join columns** (foreign keys especially): `CREATE INDEX idx_emp_dept ON employees(dept_id);`
- Select only the columns you need; avoid `SELECT *` across joins.
- Filter early (`WHERE` on indexed columns) to shrink the data being joined.
- Use `EXPLAIN` / `EXPLAIN ANALYZE` to inspect the plan.
- Common physical algorithms the optimizer picks from: **Nested Loop Join** (small tables / indexed lookups), **Hash Join** (large, unsorted, equality joins), **Merge Join** (both inputs sorted on the key).

---

## 10. Common Interview Questions

1. **Difference between `INNER JOIN` and `LEFT JOIN`?**
   Inner returns only matches; left keeps every left-table row, filling `NULL` where there's no match.

2. **How do you find records in table A that don't exist in table B?**
   `LEFT JOIN` + `WHERE b.key IS NULL`, or `NOT EXISTS`, or `NOT IN` (careful: `NOT IN` misbehaves if the subquery returns `NULL`).

3. **How do you do a `FULL OUTER JOIN` in MySQL?**
   `LEFT JOIN UNION RIGHT JOIN`.

4. **What is a self join and when would you use it?**
   Joining a table to itself via aliases, for hierarchies like employee-manager.

5. **What does a `CROSS JOIN` return for tables of 5 and 4 rows?**
   20 rows (Cartesian product).

6. **Why can a `WHERE` clause turn a `LEFT JOIN` into an `INNER JOIN`?**
   Because filtering on the right table's columns removes the `NULL` rows that the outer join preserved. Move the condition to `ON`.

7. **What is the difference between `JOIN ... ON` and `JOIN ... USING`?**
   `USING` works only when the column names are identical and outputs that column once.

---

## 11. Cheat Sheet

```sql
-- INNER: matches only
SELECT * FROM A JOIN B ON A.id = B.a_id;

-- LEFT: all A, matching B
SELECT * FROM A LEFT JOIN B ON A.id = B.a_id;

-- RIGHT: all B, matching A
SELECT * FROM A RIGHT JOIN B ON A.id = B.a_id;

-- FULL OUTER: everything from both
SELECT * FROM A FULL OUTER JOIN B ON A.id = B.a_id;

-- SELF: table with itself
SELECT x.*, y.* FROM T x JOIN T y ON x.parent_id = y.id;

-- CROSS: all combinations
SELECT * FROM A CROSS JOIN B;

-- Anti-join: in A, not in B
SELECT A.* FROM A LEFT JOIN B ON A.id = B.a_id WHERE B.a_id IS NULL;
```
