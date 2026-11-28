# SQL Basics and Functions: A Beginner-Friendly Deep Dive

This guide explains each topic in simple language, with small examples you can trace by hand.
Syntax is **MySQL**-style. Where PostgreSQL differs, it is noted.

**Topics**

- Part 1: SQL Basics: `SELECT`, `DISTINCT`, `WHERE`, `AND / OR / NOT`, `ORDER BY`, `LIMIT`
- Part 2: SQL Functions: `COUNT / SUM / AVG / MIN / MAX`, `GROUP BY`, `HAVING`, String functions, Date functions
- Part 3: How SQL runs a query (execution order), common mistakes, practice questions, cheat sheet

---

## 0. The sample table used in every example

Think of a table like an Excel sheet. **Columns** are the headings, and each **row** is one record.

**Table: `employees`**

| id | name  | city      | dept  | salary | join_date  | email             |
|----|-------|-----------|-------|--------|------------|-------------------|
| 1  | Asha  | Bengaluru | IT    | 60000  | 2021-03-15 | asha@example.com  |
| 2  | Ravi  | Mumbai    | IT    | 40000  | 2022-07-01 | ravi@example.com  |
| 3  | Meena | Delhi     | HR    | 50000  | 2020-01-10 | meena@example.com |
| 4  | John  | Mumbai    | HR    | 30000  | 2023-05-20 | john@example.com  |
| 5  | Kiran | Bengaluru | Sales | 70000  | 2019-11-05 | kiran@example.com |
| 6  | Sara  | Delhi     | Sales | 45000  | 2022-02-14 | NULL              |
| 7  | Dev   | Bengaluru | IT    | 40000  | 2024-01-08 | dev@example.com   |

Notes:
- `NULL` means "no value / unknown". It is **not** zero and **not** an empty string. Sara has no email.
- There are 7 rows, 3 departments (IT, HR, Sales) and 3 cities (Bengaluru, Mumbai, Delhi).

To create it yourself:

```sql
CREATE TABLE employees (
    id        INT PRIMARY KEY,
    name      VARCHAR(50),
    city      VARCHAR(50),
    dept      VARCHAR(50),
    salary    INT,
    join_date DATE,
    email     VARCHAR(100)
);

INSERT INTO employees VALUES
(1, 'Asha',  'Bengaluru', 'IT',    60000, '2021-03-15', 'asha@example.com'),
(2, 'Ravi',  'Mumbai',    'IT',    40000, '2022-07-01', 'ravi@example.com'),
(3, 'Meena', 'Delhi',     'HR',    50000, '2020-01-10', 'meena@example.com'),
(4, 'John',  'Mumbai',    'HR',    30000, '2023-05-20', 'john@example.com'),
(5, 'Kiran', 'Bengaluru', 'Sales', 70000, '2019-11-05', 'kiran@example.com'),
(6, 'Sara',  'Delhi',     'Sales', 45000, '2022-02-14', NULL),
(7, 'Dev',   'Bengaluru', 'IT',    40000, '2024-01-08', 'dev@example.com');
```

---

# PART 1: SQL BASICS

---

## 1. SELECT: "Show me these columns"

`SELECT` chooses **which columns** to show. `FROM` says **which table** to read.

```sql
SELECT name, salary
FROM employees;
```

**Line by line**
- `SELECT name, salary` means show only these two columns.
- `FROM employees` means read them from the `employees` table.
- The `;` ends the statement.

**Result (all 7 rows, 2 columns)**

| name  | salary |
|-------|--------|
| Asha  | 60000  |
| Ravi  | 40000  |
| Meena | 50000  |
| John  | 30000  |
| Kiran | 70000  |
| Sara  | 45000  |
| Dev   | 40000  |

### Select all columns with `*`

```sql
SELECT * FROM employees;
```

`*` means "every column". It is fine for exploring, but in real applications list the columns you need. It is faster and doesn't break when the table changes.

### Alias with `AS`: rename a column in the output

```sql
SELECT name AS employee_name, salary * 12 AS yearly_salary
FROM employees;
```

- `AS employee_name` gives the output column a friendlier name.
- `salary * 12` is a calculation done on every row.
- The alias changes only the output, not the table.

| employee_name | yearly_salary |
|---------------|---------------|
| Asha          | 720000        |
| Ravi          | 480000        |
| ...           | ...           |

### SELECT without a table

```sql
SELECT 2 + 3;          -- 5
SELECT NOW();          -- current date and time
```

Handy for testing a function.

---

## 2. DISTINCT: "Remove duplicates"

`DISTINCT` shows each value only once.

```sql
SELECT dept FROM employees;
```

Result: `IT, IT, HR, HR, Sales, Sales, IT` (7 rows, with duplicates).

```sql
SELECT DISTINCT dept FROM employees;
```

| dept  |
|-------|
| IT    |
| HR    |
| Sales |

**Line by line**
- SQL reads all 7 `dept` values.
- It keeps one copy of each different value.
- So you get 3 rows.

### DISTINCT on more than one column

```sql
SELECT DISTINCT dept, city FROM employees;
```

`DISTINCT` applies to the **combination** of columns, not each column separately.

| dept  | city      |
|-------|-----------|
| IT    | Bengaluru |
| IT    | Mumbai    |
| HR    | Delhi     |
| HR    | Mumbai    |
| Sales | Bengaluru |
| Sales | Delhi     |

That is 6 rows. `(IT, Bengaluru)` appears for both Asha and Dev, but is shown once.

### Count distinct values

```sql
SELECT COUNT(DISTINCT city) FROM employees;   -- 3
```

---

## 3. WHERE: "Only the rows that match"

`SELECT` picks columns. `WHERE` picks **rows**.

```sql
SELECT name, salary
FROM employees
WHERE salary > 45000;
```

**Line by line**
- SQL goes through each row and checks `salary > 45000`.
- If true, the row stays. If false, it is dropped.

| name  | salary |
|-------|--------|
| Asha  | 60000  |
| Meena | 50000  |
| Kiran | 70000  |

Sara (45000) is **not** included, because `>` means strictly greater.

### Comparison operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | equal | `dept = 'IT'` |
| `<>` or `!=` | not equal | `city <> 'Delhi'` |
| `>` `<` | greater / less | `salary > 50000` |
| `>=` `<=` | greater or equal / less or equal | `salary >= 50000` |
| `BETWEEN a AND b` | in a range, **inclusive** | `salary BETWEEN 40000 AND 50000` |
| `IN (...)` | matches any value in a list | `city IN ('Delhi', 'Mumbai')` |
| `LIKE` | pattern match | `name LIKE 'A%'` |
| `IS NULL` / `IS NOT NULL` | check for missing value | `email IS NULL` |

Remember: text values go in **single quotes** (`'IT'`), numbers don't.

### BETWEEN

```sql
SELECT name, salary FROM employees
WHERE salary BETWEEN 40000 AND 50000;
```

Result: Ravi (40000), Meena (50000), Sara (45000), Dev (40000). Both ends are included.

### IN

```sql
SELECT name, city FROM employees
WHERE city IN ('Delhi', 'Mumbai');
```

Result: Ravi, Meena, John, Sara. `IN` is a short way to write `city = 'Delhi' OR city = 'Mumbai'`.

### LIKE: pattern matching

Two special symbols:
- `%` means any number of characters (including none)
- `_` means exactly one character

| Pattern | Matches | Result on our table |
|---------|---------|---------------------|
| `'A%'` | starts with A | Asha |
| `'%a'` | ends with a | Asha, Meena, Sara |
| `'%an%'` | contains "an" | Kiran |
| `'_ohn'` | any 1 letter, then "ohn" | John |

```sql
SELECT name FROM employees WHERE name LIKE '%a';
```

### NULL: the special case

```sql
SELECT name FROM employees WHERE email = NULL;     -- WRONG: returns nothing
SELECT name FROM employees WHERE email IS NULL;    -- RIGHT: returns Sara
```

Any comparison with `NULL` using `=`, `<>` or `>` gives "unknown", and `WHERE` only keeps rows where the condition is true. Always use `IS NULL` or `IS NOT NULL`.

---

## 4. AND, OR, NOT: combining conditions

### AND: all conditions must be true

```sql
SELECT name, city, dept FROM employees
WHERE city = 'Bengaluru' AND dept = 'IT';
```

| name | city      | dept |
|------|-----------|------|
| Asha | Bengaluru | IT   |
| Dev  | Bengaluru | IT   |

Kiran is in Bengaluru but in Sales, so the second condition fails.

### OR: at least one condition must be true

```sql
SELECT name, dept FROM employees
WHERE dept = 'HR' OR dept = 'Sales';
```

Result: Meena, John, Kiran, Sara.

### NOT: flip a condition

```sql
SELECT name, city FROM employees
WHERE NOT city = 'Mumbai';
```

Result: Asha, Meena, Kiran, Sara, Dev. It is the same as `city <> 'Mumbai'`.

Other forms: `NOT IN (...)`, `NOT LIKE '...'`, `NOT BETWEEN ...`, `IS NOT NULL`.

### The order of evaluation: AND before OR

This causes many bugs. `AND` is processed **before** `OR`, like multiplication before addition.

```sql
SELECT name, dept, salary FROM employees
WHERE dept = 'IT' OR dept = 'HR' AND salary > 45000;
```

SQL reads it as:

```
dept = 'IT'  OR  (dept = 'HR' AND salary > 45000)
```

So you get all IT employees (Asha, Ravi, Dev), plus only the HR employees with salary above 45000 (Meena). John is dropped.

If you meant "(IT or HR) and salary above 45000", use **brackets**:

```sql
WHERE (dept = 'IT' OR dept = 'HR') AND salary > 45000;
```

Result: Asha (60000) and Meena (50000).

**Tip:** when mixing `AND` and `OR`, always add brackets, even if they aren't strictly needed.

---

## 5. ORDER BY: sort the result

By default, rows come back in no guaranteed order. `ORDER BY` sorts them.

```sql
SELECT name, salary FROM employees
ORDER BY salary DESC;
```

- `ASC` means ascending, smallest first. This is the **default**.
- `DESC` means descending, biggest first.

| name  | salary |
|-------|--------|
| Kiran | 70000  |
| Asha  | 60000  |
| Meena | 50000  |
| Sara  | 45000  |
| Ravi  | 40000  |
| Dev   | 40000  |
| John  | 30000  |

Ravi and Dev both earn 40000. Their relative order here is **not guaranteed**. Fix that with a second sort column.

### Sort by more than one column

```sql
SELECT name, salary FROM employees
ORDER BY salary DESC, name ASC;
```

**How it works:** sort by `salary` first (highest first). If two rows have the same salary, sort those by `name` A to Z.

| name  | salary |
|-------|--------|
| Kiran | 70000  |
| Asha  | 60000  |
| Meena | 50000  |
| Sara  | 45000  |
| Dev   | 40000  |
| Ravi  | 40000  |
| John  | 30000  |

Now Dev comes before Ravi every time.

### Other ways to sort

```sql
ORDER BY dept, salary DESC;   -- by dept A-Z, then salary high to low inside each dept
ORDER BY 2;                   -- sort by the 2nd column in the SELECT list (works, but unclear)
ORDER BY salary * 12 DESC;    -- sort by a calculation
```

Text sorts alphabetically, dates sort by time, and numbers sort by value. `NULL` sorts first in `ASC` in MySQL (and last in PostgreSQL).

---

## 6. LIMIT: "Only give me N rows"

```sql
SELECT name, salary FROM employees
ORDER BY salary DESC, name ASC
LIMIT 3;
```

Result: Kiran, Asha, Meena. This is the **top 3 highest-paid employees**.

> **Always use `ORDER BY` with `LIMIT`.** Without it, "the first 3 rows" can be any 3 rows.

### OFFSET: skip rows (pagination)

```sql
SELECT name, salary FROM employees
ORDER BY salary DESC, name ASC
LIMIT 3 OFFSET 3;
```

**How it works:** skip the first 3 rows, then return the next 3. Result: Sara, Dev, Ravi.

Pagination pattern for a web page (page size 10):

```sql
-- page 1: LIMIT 10 OFFSET 0
-- page 2: LIMIT 10 OFFSET 10
-- page n: LIMIT 10 OFFSET (n-1)*10
```

MySQL also allows `LIMIT 3, 3`, which means `LIMIT offset, count`. The order is reversed compared to `OFFSET`, so it is confusing. Prefer the `OFFSET` form.

### Useful patterns

```sql
-- highest salary employee
SELECT name, salary FROM employees ORDER BY salary DESC LIMIT 1;

-- 2nd highest salary value
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 1;
```

The second query returns 60000. `DISTINCT` is there so tied salaries don't count twice.

> SQL Server uses `SELECT TOP 3 ...` and Oracle uses `FETCH FIRST 3 ROWS ONLY` instead of `LIMIT`.

---

# PART 2: SQL FUNCTIONS

A **function** takes some input and returns a value. There are two kinds:

- **Aggregate functions** take many rows and return **one** value (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`).
- **Scalar functions** work on **each row** and return one value per row (string and date functions).

---

## 7. Aggregate functions: COUNT, SUM, AVG, MIN, MAX

| Function | What it does |
|----------|--------------|
| `COUNT(*)` | counts rows |
| `COUNT(col)` | counts rows where `col` is **not NULL** |
| `SUM(col)` | adds up the values |
| `AVG(col)` | average of the values |
| `MIN(col)` | smallest value |
| `MAX(col)` | largest value |

```sql
SELECT COUNT(*)        AS total_employees,
       SUM(salary)     AS total_salary,
       AVG(salary)     AS avg_salary,
       MIN(salary)     AS lowest,
       MAX(salary)     AS highest
FROM employees;
```

| total_employees | total_salary | avg_salary  | lowest | highest |
|-----------------|--------------|-------------|--------|---------|
| 7               | 335000       | 47857.1429  | 30000  | 70000   |

**How each value is calculated**
- `COUNT(*)`: 7 rows.
- `SUM`: 60000 + 40000 + 50000 + 30000 + 70000 + 45000 + 40000 = 335000.
- `AVG`: 335000 / 7 = 47857.14...
- `MIN` and `MAX`: the smallest and largest salary.

Round it: `ROUND(AVG(salary), 2)` gives `47857.14`.

### COUNT(*) vs COUNT(column) vs COUNT(DISTINCT column)

```sql
SELECT COUNT(*)              AS all_rows,       -- 7
       COUNT(email)          AS with_email,     -- 6  (Sara's NULL is skipped)
       COUNT(DISTINCT dept)  AS unique_depts    -- 3
FROM employees;
```

### Important rules

1. **Aggregates ignore NULLs** (except `COUNT(*)`). `AVG` divides by the number of non-NULL values, not total rows.
2. An aggregate can't be used in `WHERE`. Use `HAVING` instead (section 9).
3. `SUM` and `AVG` are for numbers. `MIN` and `MAX` also work on text and dates:

```sql
SELECT MIN(name), MAX(name) FROM employees;          -- Asha, Sara (alphabetical)
SELECT MIN(join_date), MAX(join_date) FROM employees; -- 2019-11-05, 2024-01-08
```

4. If the table (or `WHERE` result) has no rows: `COUNT` returns `0`, while `SUM`, `AVG`, `MIN`, `MAX` return `NULL`.

### Aggregates with WHERE

```sql
SELECT COUNT(*), AVG(salary)
FROM employees
WHERE dept = 'IT';
```

`WHERE` filters the rows **first**, then the aggregate runs on what is left: 3 employees, average (60000+40000+40000)/3 = 46666.67.

---

## 8. GROUP BY: one result per group

Aggregates on their own give one answer for the whole table. `GROUP BY` splits the rows into groups and gives one answer **per group**.

```sql
SELECT dept, COUNT(*) AS employees, AVG(salary) AS avg_salary
FROM employees
GROUP BY dept;
```

**What SQL does, step by step**

1. Looks at every row and puts it into a bucket by `dept`.

```
IT    bucket: Asha 60000, Ravi 40000, Dev 40000
HR    bucket: Meena 50000, John 30000
Sales bucket: Kiran 70000, Sara 45000
```

2. Runs `COUNT` and `AVG` inside each bucket.
3. Returns one row per bucket.

| dept  | employees | avg_salary |
|-------|-----------|------------|
| IT    | 3         | 46666.67   |
| HR    | 2         | 40000.00   |
| Sales | 2         | 57500.00   |

### The golden rule of GROUP BY

> Every column in `SELECT` must either be in the `GROUP BY` or be inside an aggregate function.

```sql
SELECT dept, name, COUNT(*) FROM employees GROUP BY dept;
-- ERROR (or random name): which name should an IT group show, Asha, Ravi or Dev?
```

### Group by more than one column

```sql
SELECT city, dept, COUNT(*) AS employees
FROM employees
GROUP BY city, dept;
```

Groups are unique `(city, dept)` pairs. `(Bengaluru, IT)` has 2 people (Asha and Dev).

### Group by a calculation

```sql
SELECT YEAR(join_date) AS join_year, COUNT(*) AS hires
FROM employees
GROUP BY YEAR(join_date)
ORDER BY join_year;
```

| join_year | hires |
|-----------|-------|
| 2019      | 1     |
| 2020      | 1     |
| 2021      | 1     |
| 2022      | 2     |
| 2023      | 1     |
| 2024      | 1     |

### Combine with WHERE and ORDER BY

```sql
SELECT city, SUM(salary) AS total_salary
FROM employees
WHERE dept <> 'HR'
GROUP BY city
ORDER BY total_salary DESC;
```

1. `WHERE` removes HR rows first.
2. `GROUP BY` makes one group per city.
3. `SUM` adds the salaries inside each group.
4. `ORDER BY` sorts the groups.

| city      | total_salary |
|-----------|--------------|
| Bengaluru | 170000       |
| Delhi     | 45000        |
| Mumbai    | 40000        |

(Bengaluru is Asha 60000 + Kiran 70000 + Dev 40000.)

---

## 9. HAVING: filter the groups

`WHERE` filters **rows before** grouping. `HAVING` filters **groups after** grouping.

Question: which departments have more than 2 employees?

```sql
SELECT dept, COUNT(*) AS employees
FROM employees
GROUP BY dept
HAVING COUNT(*) > 2;
```

| dept | employees |
|------|-----------|
| IT   | 3         |

HR and Sales have only 2 each, so those groups are removed.

**Why not `WHERE COUNT(*) > 2`?** Because when `WHERE` runs, the groups don't exist yet, so there is nothing to count. This is the single biggest difference between the two.

### WHERE and HAVING together

```sql
SELECT dept, AVG(salary) AS avg_salary
FROM employees
WHERE city <> 'Mumbai'         -- filter rows first
GROUP BY dept
HAVING AVG(salary) > 45000;    -- then filter groups
```

**Trace**
1. `WHERE` removes Ravi and John (Mumbai). Rows left: Asha, Meena, Kiran, Sara, Dev.
2. `GROUP BY dept`: IT has Asha 60000 and Dev 40000, HR has Meena 50000, Sales has Kiran 70000 and Sara 45000.
3. Averages: IT = 50000, HR = 50000, Sales = 57500.
4. `HAVING > 45000`: all three pass.

| dept  | avg_salary |
|-------|------------|
| IT    | 50000.00   |
| HR    | 50000.00   |
| Sales | 57500.00   |

### WHERE vs HAVING at a glance

| | WHERE | HAVING |
|---|-------|--------|
| Filters | individual rows | groups |
| Runs | before `GROUP BY` | after `GROUP BY` |
| Can use aggregates? | no | yes |
| Without `GROUP BY` | works | rarely used |

**Performance tip:** filter as much as you can in `WHERE`, because it reduces the rows before grouping. Use `HAVING` only for conditions that need an aggregate.

---

## 10. String functions

String functions work on text, one row at a time.

| Function | What it does | Example | Result |
|----------|--------------|---------|--------|
| `UPPER(s)` | all capitals | `UPPER('asha')` | `ASHA` |
| `LOWER(s)` | all small letters | `LOWER('ASHA')` | `asha` |
| `LENGTH(s)` | length in bytes (see note) | `LENGTH('Asha')` | `4` |
| `CHAR_LENGTH(s)` | length in characters | `CHAR_LENGTH('Asha')` | `4` |
| `CONCAT(a, b, ...)` | join strings | `CONCAT('Asha', ' - ', 'IT')` | `Asha - IT` |
| `SUBSTRING(s, start, len)` | cut part of a string (start at 1) | `SUBSTRING('Bengaluru', 1, 3)` | `Ben` |
| `LEFT(s, n)` / `RIGHT(s, n)` | first / last n characters | `LEFT('Bengaluru', 3)` | `Ben` |
| `TRIM(s)` | remove spaces at both ends | `TRIM('  hi  ')` | `hi` |
| `REPLACE(s, from, to)` | replace text | `REPLACE('2024-01-08', '-', '/')` | `2024/01/08` |
| `INSTR(s, sub)` | position of text (0 if not found) | `INSTR('asha@example.com', '@')` | `5` |
| `REVERSE(s)` | reverse the string | `REVERSE('abc')` | `cba` |
| `COALESCE(a, b)` | first non-NULL value | `COALESCE(NULL, 'N/A')` | `N/A` |

> `LENGTH` counts **bytes** in MySQL, so non-English characters can give a bigger number. Use `CHAR_LENGTH` for the real character count. In PostgreSQL, `LENGTH` counts characters.

### Using them in queries

**Make a label**

```sql
SELECT CONCAT(name, ' (', dept, ')') AS employee_label
FROM employees;
```

Result: `Asha (IT)`, `Ravi (IT)`, `Meena (HR)`, and so on.

**Capital first letter, small rest** (clean messy data)

```sql
SELECT CONCAT(UPPER(LEFT(name, 1)), LOWER(SUBSTRING(name, 2))) AS clean_name
FROM employees;
```

**Line by line**
- `LEFT(name, 1)` takes the first letter.
- `UPPER(...)` makes it a capital.
- `SUBSTRING(name, 2)` takes everything from the 2nd character onward.
- `LOWER(...)` makes that part small.
- `CONCAT` joins the two pieces. So `'aSHA'` becomes `'Asha'`.

**Get the username from an email**

```sql
SELECT email, SUBSTRING(email, 1, INSTR(email, '@') - 1) AS username
FROM employees
WHERE email IS NOT NULL;
```

**Trace for `asha@example.com`**
- `INSTR(email, '@')` gives `5`, the position of `@`.
- `5 - 1 = 4` characters before it.
- `SUBSTRING(email, 1, 4)` gives `asha`.

**Replace a NULL with a friendly value**

```sql
SELECT name, COALESCE(email, 'no email') AS email
FROM employees;
```

Sara's row shows `no email` instead of `NULL`.

**Case-insensitive search**

```sql
SELECT name FROM employees WHERE LOWER(name) = 'asha';
```

(MySQL comparisons are often case-insensitive already, depending on the collation. PostgreSQL is case-sensitive, so `LOWER` or `ILIKE` helps there.)

### String function notes

- Positions in SQL start at **1**, not 0 (unlike Java).
- String concatenation: MySQL uses `CONCAT(...)`. PostgreSQL and Oracle also support `||` (`'a' || 'b'`). In MySQL, `||` means OR by default.
- If any argument to `CONCAT` is `NULL`, the whole result is `NULL` in MySQL. Wrap nullable columns with `COALESCE(col, '')`.
- Using a function on a column inside `WHERE` (like `LOWER(name) = 'asha'`) can stop the database from using an index on that column.

---

## 11. Date functions

Date functions work with `DATE`, `DATETIME` and `TIMESTAMP` values. Dates are written as `'YYYY-MM-DD'`.

### Getting the current date and time

| Function (MySQL) | Returns | PostgreSQL |
|------------------|---------|------------|
| `CURDATE()` | today's date | `CURRENT_DATE` |
| `NOW()` | date and time now | `NOW()` |
| `CURTIME()` | current time | `CURRENT_TIME` |

### Pulling out parts of a date

| Function | Example | Result |
|----------|---------|--------|
| `YEAR(d)` | `YEAR('2021-03-15')` | `2021` |
| `MONTH(d)` | `MONTH('2021-03-15')` | `3` |
| `DAY(d)` | `DAY('2021-03-15')` | `15` |
| `DAYNAME(d)` | `DAYNAME('2024-01-08')` | `Monday` |
| `MONTHNAME(d)` | `MONTHNAME('2021-03-15')` | `March` |
| `QUARTER(d)` | `QUARTER('2021-03-15')` | `1` |

PostgreSQL uses one function for all: `EXTRACT(YEAR FROM join_date)`. MySQL supports `EXTRACT` too.

```sql
SELECT name, join_date, YEAR(join_date) AS join_year
FROM employees
WHERE YEAR(join_date) = 2022;
```

Result: Ravi (2022-07-01) and Sara (2022-02-14).

> For big tables, this version is faster because it can use an index on `join_date`:
> ```sql
> WHERE join_date >= '2022-01-01' AND join_date < '2023-01-01'
> ```

### Date arithmetic

```sql
SELECT DATE_ADD('2024-01-08', INTERVAL 30 DAY);     -- 2024-02-07
SELECT DATE_SUB('2024-01-08', INTERVAL 1 MONTH);    -- 2023-12-08
SELECT DATEDIFF('2024-01-08', '2024-01-01');        -- 7  (days between)
SELECT TIMESTAMPDIFF(YEAR, '2021-03-15', '2024-01-08');  -- 2 (full years)
```

**Line by line**
- `DATE_ADD(date, INTERVAL n UNIT)` adds time. Units: `DAY`, `MONTH`, `YEAR`, `HOUR`, and so on.
- `DATE_SUB` subtracts time.
- `DATEDIFF(later, earlier)` gives the number of **days**. The order matters, and a wrong order gives a negative number.
- `TIMESTAMPDIFF(unit, start, end)` gives whole units, here full years. Between 2021-03-15 and 2024-01-08 only 2 full years have passed.

In PostgreSQL: `date + INTERVAL '30 days'`, `date1 - date2` (gives days) and `AGE(date1, date2)`.

### Formatting a date: `DATE_FORMAT`

```sql
SELECT DATE_FORMAT('2021-03-15', '%d-%m-%Y');   -- 15-03-2021
SELECT DATE_FORMAT('2021-03-15', '%d %M %Y');   -- 15 March 2021
```

| Code | Meaning | Example |
|------|---------|---------|
| `%Y` | 4-digit year | 2021 |
| `%y` | 2-digit year | 21 |
| `%m` | month number | 03 |
| `%M` | month name | March |
| `%d` | day of month | 15 |
| `%W` | weekday name | Monday |
| `%H:%i:%s` | hour:minute:second (24 hour) | 14:30:05 |

PostgreSQL uses `TO_CHAR(date, 'DD-MM-YYYY')`.

### Turning text into a date: `STR_TO_DATE`

```sql
SELECT STR_TO_DATE('15/03/2021', '%d/%m/%Y');   -- 2021-03-15
```

Use this when your data arrives in a non-standard format (Indian-style `dd/mm/yyyy` is common). PostgreSQL uses `TO_DATE('15/03/2021', 'DD/MM/YYYY')`.

### Real examples

**Each employee's experience in years**

```sql
SELECT name,
       join_date,
       TIMESTAMPDIFF(YEAR, join_date, CURDATE()) AS years_in_company
FROM employees
ORDER BY join_date;
```

**Employees who joined in the last 2 years**

```sql
SELECT name, join_date
FROM employees
WHERE join_date >= DATE_SUB(CURDATE(), INTERVAL 2 YEAR);
```

`CURDATE()` is today. Subtracting 2 years gives the cut-off date, and `>=` keeps everyone who joined after it.

**Hires per year, newest first**

```sql
SELECT YEAR(join_date) AS join_year, COUNT(*) AS hires
FROM employees
GROUP BY YEAR(join_date)
ORDER BY join_year DESC;
```

### Date function notes

- A `DATE` has no time part. `DATETIME` and `TIMESTAMP` do. When comparing a `DATETIME` against a date like `'2024-01-08'`, the date is read as `'2024-01-08 00:00:00'`, so rows later that day are missed. Use a range (`>= day AND < next day`).
- Always store dates in `DATE` columns, not text, so sorting and date math work properly.
- In your Java code, a SQL `DATE` maps to `LocalDate` and a `DATETIME` to `LocalDateTime`.

---

# PART 3: PUTTING IT TOGETHER

---

## 12. How SQL runs your query: the execution order

You **write** a query in this order:

```
SELECT -> FROM -> WHERE -> GROUP BY -> HAVING -> ORDER BY -> LIMIT
```

SQL **runs** it in this order:

| Step | Clause | What happens |
|------|--------|--------------|
| 1 | `FROM` | pick the table |
| 2 | `WHERE` | filter rows |
| 3 | `GROUP BY` | make groups |
| 4 | `HAVING` | filter groups |
| 5 | `SELECT` | choose columns and compute expressions, aliases created |
| 6 | `DISTINCT` | remove duplicates |
| 7 | `ORDER BY` | sort |
| 8 | `LIMIT` | cut the number of rows |

This explains common questions:

- **Why can't `WHERE` use a `SELECT` alias?** `WHERE` runs at step 2, but aliases are created at step 5.
- **Why can `ORDER BY` use an alias?** It runs at step 7, after the alias exists.
- **Why can't `WHERE` use `COUNT(*)`?** Groups aren't made until step 3.
- **Why does `HAVING` work with aggregates?** It runs after the groups exist.

(MySQL is lenient and lets `GROUP BY` and `HAVING` use aliases, but standard SQL and PostgreSQL don't.)

### A full query traced

```sql
SELECT dept, COUNT(*) AS employees, AVG(salary) AS avg_salary
FROM employees
WHERE city IN ('Bengaluru', 'Mumbai')
GROUP BY dept
HAVING COUNT(*) >= 2
ORDER BY avg_salary DESC
LIMIT 2;
```

1. **FROM** `employees`: 7 rows.
2. **WHERE**: keep Bengaluru and Mumbai. Rows left: Asha, Ravi, John, Kiran, Dev (5 rows).
3. **GROUP BY dept**: IT (Asha 60000, Ravi 40000, Dev 40000), HR (John 30000), Sales (Kiran 70000).
4. **HAVING COUNT(\*) >= 2**: IT has 3 (kept). HR has 1 and Sales has 1 (removed).
5. **SELECT**: IT, 3, average 46666.67.
6. **ORDER BY** and **LIMIT 2**: only one group left.

| dept | employees | avg_salary |
|------|-----------|------------|
| IT   | 3         | 46666.67   |

---

## 13. Common mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| `WHERE email = NULL` | always returns nothing | `WHERE email IS NULL` |
| `WHERE COUNT(*) > 2` | aggregate in `WHERE` | use `HAVING COUNT(*) > 2` |
| `SELECT dept, name ... GROUP BY dept` | `name` isn't grouped or aggregated | group by it or aggregate it |
| `A OR B AND C` | `AND` runs first | add brackets |
| `LIMIT` without `ORDER BY` | random rows | always sort first |
| `LIKE '%a'` for case-insensitive search on a big table | slow | consider an index or full-text search |
| `AVG` on a column with NULLs | ignores NULLs, may surprise you | use `AVG(COALESCE(col, 0))` if NULL should count as 0 |
| `'IT'` written as `"IT"` | double quotes can mean a column name | use single quotes for text |
| comparing `DATETIME` to a plain date | misses the rest of that day | use a range |
| `COUNT(col)` instead of `COUNT(*)` | skips NULL rows | use `COUNT(*)` to count rows |

---

## 14. Practice questions

Try each one before looking at the answer.

1. Show names and cities of all employees in Delhi.
2. List the distinct cities.
3. Show the 3 lowest-paid employees.
4. How many employees are there in each city?
5. Which departments have an average salary above 45000?
6. Show employees whose name starts with `K` or ends with `a`.
7. Show total salary paid per department, highest first.
8. Show employees who joined after 1 Jan 2022, newest first.
9. Show each employee's email username in capitals (skip NULL emails).
10. How many employees joined in each year?

<details>
<summary>Answers</summary>

```sql
-- 1
SELECT name, city FROM employees WHERE city = 'Delhi';

-- 2
SELECT DISTINCT city FROM employees;

-- 3
SELECT name, salary FROM employees ORDER BY salary ASC, name ASC LIMIT 3;

-- 4
SELECT city, COUNT(*) AS employees FROM employees GROUP BY city;

-- 5
SELECT dept, AVG(salary) AS avg_salary FROM employees
GROUP BY dept HAVING AVG(salary) > 45000;

-- 6
SELECT name FROM employees WHERE name LIKE 'K%' OR name LIKE '%a';

-- 7
SELECT dept, SUM(salary) AS total_salary FROM employees
GROUP BY dept ORDER BY total_salary DESC;

-- 8
SELECT name, join_date FROM employees
WHERE join_date > '2022-01-01' ORDER BY join_date DESC;

-- 9
SELECT UPPER(SUBSTRING(email, 1, INSTR(email, '@') - 1)) AS username
FROM employees WHERE email IS NOT NULL;

-- 10
SELECT YEAR(join_date) AS join_year, COUNT(*) AS hires
FROM employees GROUP BY YEAR(join_date) ORDER BY join_year;
```

Expected results for a few:
- Q3: John (30000), Dev (40000), Ravi (40000). Dev and Ravi tie on salary, so the second sort column (`name ASC`) puts Dev before Ravi.
- Q5: IT (46666.67) and Sales (57500.00).
- Q6: Kiran (starts with K), Asha, Meena, Sara (end with a).
- Q8: Dev (2024-01-08), John (2023-05-20), Ravi (2022-07-01), Sara (2022-02-14).

</details>

---

## 15. Quick cheat sheet

```sql
SELECT DISTINCT col1, col2, AGG(col3) AS alias   -- what to show
FROM table_name                                   -- where from
WHERE row_condition                               -- filter rows (no aggregates)
GROUP BY col1, col2                               -- make groups
HAVING group_condition                            -- filter groups (aggregates OK)
ORDER BY col1 DESC, col2 ASC                      -- sort
LIMIT n OFFSET m;                                 -- take n rows after skipping m
```

| Need | Use |
|------|-----|
| remove duplicates | `DISTINCT` |
| missing values | `IS NULL`, `COALESCE` |
| one result per group | `GROUP BY` + aggregate |
| filter groups | `HAVING` |
| top N | `ORDER BY ... LIMIT N` |
| text cleanup | `TRIM`, `UPPER`, `LOWER`, `REPLACE`, `SUBSTRING` |
| join text | `CONCAT` |
| date parts | `YEAR`, `MONTH`, `DAY` |
| date difference | `DATEDIFF`, `TIMESTAMPDIFF` |
| date format | `DATE_FORMAT`, `STR_TO_DATE` |

**Remember**
1. `WHERE` filters rows. `HAVING` filters groups.
2. Aggregates ignore `NULL` (except `COUNT(*)`).
3. `AND` runs before `OR`. Use brackets.
4. Write order is `SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT`, but the run order starts at `FROM`.
5. Always pair `LIMIT` with `ORDER BY`.
