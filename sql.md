# SQL Notes -- 27 November 2026

## Topics covered today

1.  `SELECT`
2.  `DISTINCT`
3.  `WHERE`
4.  `AND`, `OR`, `NOT`
5.  `ORDER BY`
6.  `LIMIT` and `OFFSET`
7.  Aggregate functions: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`
8.  `GROUP BY`
9.  `HAVING`
10. String functions
11. Date functions

------------------------------------------------------------------------

# 1. Example table

We will use this `employees` table in the examples.

    id name      age department     salary city        joining_date
  ---- ------- ----- ------------ -------- ----------- --------------
     1 Rahul      25 IT              50000 Delhi       2023-01-15
     2 Amit       28 HR              40000 Mumbai      2022-06-10
     3 Priya      24 IT              60000 Bangalore   2023-03-20
     4 Sneha      30 Finance         55000 Delhi       2021-08-12
     5 Karan      26 IT              45000 Mumbai      2024-02-05
     6 Neha       28 HR              40000 Delhi       2022-11-18
     7 Arjun      32 Finance         75000 Bangalore   2020-05-25
     8 Pooja      23 IT              35000 Delhi       2024-07-01

A row represents one employee. A column represents one attribute of an
employee.

------------------------------------------------------------------------

# 2. SELECT

`SELECT` retrieves data from a table.

### Syntax

``` sql
SELECT column1, column2
FROM table_name;
```

### Select one column

``` sql
SELECT name
FROM employees;
```

### Select multiple columns

``` sql
SELECT name, age, salary
FROM employees;
```

### Select every column

``` sql
SELECT *
FROM employees;
```

`*` means all columns. In application code, prefer selecting only the
columns you need to avoid transferring unnecessary data and coupling the
query to the entire table structure.

### Calculated columns and aliases

``` sql
SELECT
    name,
    salary,
    salary * 12 AS annual_salary
FROM employees;
```

`AS` assigns an alias (a temporary output name) to a selected
expression.

**Use cases:** displaying employee details, retrieving products,
fetching orders, and supplying data to backend APIs.

------------------------------------------------------------------------

# 3. DISTINCT

`DISTINCT` removes duplicate result rows.

### Unique departments

``` sql
SELECT DISTINCT department
FROM employees;
```

### Unique combinations of multiple columns

``` sql
SELECT DISTINCT department, city
FROM employees;
```

When multiple columns are selected, SQL removes duplicate combinations
of those columns. It does not independently deduplicate each column.

### Count unique values

``` sql
SELECT COUNT(DISTINCT city)
FROM employees;
```

There are three unique cities in the example: Delhi, Mumbai, and
Bangalore.

### DISTINCT vs UNIQUE

-   `DISTINCT` removes duplicates from a query's result.
-   `UNIQUE` is a database constraint that prevents duplicate values
    from being stored in a column or combination of columns.

------------------------------------------------------------------------

# 4. WHERE

`WHERE` filters individual rows using a condition.

### Syntax

``` sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

### Examples

``` sql
-- Employees with salary greater than 50000
SELECT *
FROM employees
WHERE salary > 50000;

-- Employees in the IT department
SELECT *
FROM employees
WHERE department = 'IT';

-- Employees aged at least 28
SELECT name, age
FROM employees
WHERE age >= 28;

-- Employees earning exactly 40000
SELECT *
FROM employees
WHERE salary = 40000;
```

### Comparison operators

  Operator   Meaning                    Example
  ---------- -------------------------- -------------------
  `=`        Equal to                   `age = 25`
  `!=`       Not equal to               `age != 25`
  `<>`       Not equal to               `age <> 25`
  `>`        Greater than               `salary > 50000`
  `<`        Less than                  `age < 30`
  `>=`       Greater than or equal to   `age >= 25`
  `<=`       Less than or equal to      `salary <= 50000`

`!=` and `<>` are equivalent in many commonly used SQL databases.

### BETWEEN

Checks whether a value falls within an inclusive range.

``` sql
SELECT *
FROM employees
WHERE salary BETWEEN 40000 AND 60000;
```

Both boundary values, 40000 and 60000, are included.

### IN

Checks whether a value matches any item in a list.

``` sql
SELECT *
FROM employees
WHERE city IN ('Delhi', 'Mumbai');
```

### LIKE

Matches text patterns.

  Pattern     Meaning
  ----------- -------------------------
  `'A%'`      Starts with A
  `'%a'`      Ends with a
  `'%h%'`     Contains h
  `'_____'`   Exactly five characters

``` sql
-- Names starting with A
SELECT *
FROM employees
WHERE name LIKE 'A%';
```

`%` means zero or more characters. `_` means exactly one character.

### NULL checks

Use `IS NULL` and `IS NOT NULL`, not `= NULL`.

``` sql
SELECT *
FROM employees
WHERE salary IS NULL;

SELECT *
FROM employees
WHERE salary IS NOT NULL;
```

`NULL` represents missing or unknown information. Comparisons with NULL
generally produce UNKNOWN rather than TRUE or FALSE.

------------------------------------------------------------------------

# 5. AND, OR, and NOT

These are logical operators used to combine or reverse conditions.

## AND

Returns a row only when every condition is TRUE.

``` sql
SELECT *
FROM employees
WHERE department = 'IT'
  AND salary > 45000;
```

Both conditions must be satisfied.

  Condition A   Condition B   A AND B
  ------------- ------------- ---------
  TRUE          TRUE          TRUE
  TRUE          FALSE         FALSE
  FALSE         TRUE          FALSE
  FALSE         FALSE         FALSE

## OR

Returns a row when at least one condition is TRUE.

``` sql
SELECT *
FROM employees
WHERE department = 'IT'
   OR department = 'HR';
```

  Condition A   Condition B   A OR B
  ------------- ------------- --------
  TRUE          TRUE          TRUE
  TRUE          FALSE         TRUE
  FALSE         TRUE          TRUE
  FALSE         FALSE         FALSE

## NOT

Reverses a condition.

``` sql
-- Employees not in IT
SELECT *
FROM employees
WHERE NOT department = 'IT';

-- Equivalent for non-NULL department values
SELECT *
FROM employees
WHERE department <> 'IT';

-- Exclude IT and HR
SELECT *
FROM employees
WHERE department NOT IN ('IT', 'HR');
```

  Condition   NOT Condition
  ----------- ---------------
  TRUE        FALSE
  FALSE       TRUE

SQL uses three-valued logic: TRUE, FALSE, and UNKNOWN. For example,
`NOT (salary = 50000)` does not include rows where salary is NULL.

## Combining logical operators

``` sql
SELECT *
FROM employees
WHERE (department = 'IT' OR department = 'HR')
  AND salary > 40000
  AND city <> 'Delhi';
```

Parentheses make the intended grouping explicit.

### Operator precedence

SQL's logical precedence is:

1.  `NOT`
2.  `AND`
3.  `OR`

For example:

``` sql
WHERE department = 'IT'
   OR department = 'HR'
  AND salary > 45000;
```

This is interpreted as:

``` sql
WHERE department = 'IT'
   OR (department = 'HR' AND salary > 45000);
```

It is not equivalent to:

``` sql
WHERE (department = 'IT' OR department = 'HR')
  AND salary > 45000;
```

**Best practice:** Use parentheses whenever combining `AND` and `OR`.

------------------------------------------------------------------------

# 6. ORDER BY

`ORDER BY` sorts the result rows.

### Ascending order

``` sql
SELECT name, salary
FROM employees
ORDER BY salary ASC;
```

`ASC` means ascending (smallest to largest for numbers). It is the
default order.

### Descending order

``` sql
SELECT name, salary
FROM employees
ORDER BY salary DESC;
```

`DESC` means descending (largest to smallest for numbers).

### Sort by multiple columns

``` sql
SELECT name, department, salary
FROM employees
ORDER BY department ASC, salary DESC;
```

This sorts departments alphabetically and sorts employees within each
department by salary from highest to lowest.

A column used for sorting does not always need to appear in the SELECT
list:

``` sql
SELECT name, department
FROM employees
ORDER BY salary DESC;
```

`ORDER BY` does not change the stored table. If multiple rows have equal
sort values, their relative order is not guaranteed unless another
tie-breaker column is included.

------------------------------------------------------------------------

# 7. LIMIT and OFFSET

`LIMIT` restricts how many rows a query returns. It is supported by
MySQL and PostgreSQL, among other databases.

### Return the first three rows

``` sql
SELECT *
FROM employees
LIMIT 3;
```

Without `ORDER BY`, the database does not guarantee which rows will be
returned.

### Top three highest-paid employees

``` sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 3;
```

Always combine `LIMIT` with a meaningful `ORDER BY` when you need a
predictable top-N result.

### OFFSET

`OFFSET` skips rows before returning the requested number.

``` sql
SELECT *
FROM employees
ORDER BY id
LIMIT 3 OFFSET 2;
```

This skips the first two rows and returns the next three.

### Pagination

For 10 rows per page:

    Page   LIMIT   OFFSET
  ------ ------- --------
       1      10        0
       2      10       10
       3      10       20
       4      10       30

Formula:

``` text
offset = (page - 1) * limit
```

Example backend calculation:

``` java
int page = 2;
int limit = 10;

int offset = (page - 1) * limit;
```

Query:

``` sql
SELECT *
FROM employees
ORDER BY id
LIMIT 10 OFFSET 10;
```

**Performance note:** Large OFFSET values can be expensive because the
database may scan and skip many rows. Cursor/keyset pagination, such as
`WHERE id > last_seen_id ORDER BY id LIMIT 10`, can be more efficient
for large datasets.

------------------------------------------------------------------------

# 8. Aggregate Functions

Aggregate functions calculate a value from multiple rows and return one
result per group. Without `GROUP BY`, they generally produce one result
for the whole filtered set.

  Function    Purpose
  ----------- --------------------------------
  `COUNT()`   Counts rows or non-NULL values
  `SUM()`     Calculates a numeric total
  `AVG()`     Calculates a numeric average
  `MIN()`     Finds the smallest value
  `MAX()`     Finds the largest value

## 8.1 COUNT()

Counts rows or non-NULL values.

``` sql
-- Count every row
SELECT COUNT(*)
FROM employees;

-- Count non-NULL IDs
SELECT COUNT(id)
FROM employees;

-- Count non-NULL salaries
SELECT COUNT(salary)
FROM employees;

-- Count unique cities
SELECT COUNT(DISTINCT city)
FROM employees;
```

Difference:

-   `COUNT(*)` counts all rows, including rows containing NULL in
    individual columns.
-   `COUNT(column)` counts only rows where that column is not NULL.
-   `COUNT(DISTINCT column)` counts unique non-NULL values.

## 8.2 SUM()

Adds numeric values. NULL values are ignored.

``` sql
SELECT SUM(salary) AS total_salary
FROM employees;
```

Total salary in the example: `400000`.

Filter before calculating:

``` sql
SELECT SUM(salary) AS it_total_salary
FROM employees
WHERE department = 'IT';
```

IT department total: `190000`.

## 8.3 AVG()

Calculates the average of non-NULL numeric values.

Formula:

``` text
Average = Sum of non-NULL values / Count of non-NULL values
```

``` sql
SELECT AVG(salary) AS average_salary
FROM employees;
```

Average salary in the example: `50000`.

``` sql
SELECT AVG(salary)
FROM employees
WHERE department = 'IT';
```

IT average salary: `47500`.

## 8.4 MIN()

Returns the smallest non-NULL value.

``` sql
SELECT MIN(salary)
FROM employees;
```

Lowest salary: `35000`.

It also works with dates:

``` sql
SELECT MIN(joining_date)
FROM employees;
```

This returns the earliest joining date.

## 8.5 MAX()

Returns the largest non-NULL value.

``` sql
SELECT MAX(salary)
FROM employees;
```

Highest salary: `75000`.

``` sql
SELECT MAX(joining_date)
FROM employees;
```

This returns the latest joining date.

## 8.6 Combine aggregate functions

``` sql
SELECT
    COUNT(*) AS total_employees,
    SUM(salary) AS total_salary,
    AVG(salary) AS average_salary,
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary
FROM employees;
```

Expected results:

  -----------------------------------------------------------------------------------
    total_employees   total_salary   average_salary   minimum_salary   maximum_salary
  ----------------- -------------- ---------------- ---------------- ----------------
                  8         400000            50000            35000            75000

  -----------------------------------------------------------------------------------

If an aggregate function receives no non-NULL values, `SUM`, `AVG`,
`MIN`, and `MAX` generally return NULL. `COUNT` returns zero.

------------------------------------------------------------------------

# 9. GROUP BY

`GROUP BY` groups rows that have the same value or combination of
values. It is commonly used with aggregate functions.

## 9.1 Count employees in each department

``` sql
SELECT
    department,
    COUNT(*) AS total_employees
FROM employees
GROUP BY department;
```

Expected result:

  department     total_employees
  ------------ -----------------
  IT                           4
  HR                           2
  Finance                      2

Conceptually, SQL creates a group for IT, a group for HR, and a group
for Finance, then counts the rows in each group.

## 9.2 Total salary by department

``` sql
SELECT
    department,
    SUM(salary) AS total_salary
FROM employees
GROUP BY department;
```

  department     total_salary
  ------------ --------------
  IT                   190000
  HR                    80000
  Finance              130000

## 9.3 Average salary by department

``` sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department;
```

  department     average_salary
  ------------ ----------------
  IT                      47500
  HR                      40000
  Finance                 65000

## 9.4 Minimum and maximum salary by department

``` sql
SELECT
    department,
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary
FROM employees
GROUP BY department;
```

## 9.5 Group by multiple columns

``` sql
SELECT
    department,
    city,
    COUNT(*) AS total_employees
FROM employees
GROUP BY department, city;
```

This creates a group for each unique department-city combination, such
as IT + Delhi or HR + Mumbai.

## 9.6 WHERE with GROUP BY

`WHERE` filters rows before they are grouped.

``` sql
SELECT
    department,
    COUNT(*) AS total_employees
FROM employees
WHERE salary > 40000
GROUP BY department;
```

Only employees with salary greater than 40000 participate in grouping.

### Important GROUP BY rule

Every selected column that is not inside an aggregate function should
generally be included in the `GROUP BY` clause, subject to
database-specific functional-dependency rules.

Correct:

``` sql
SELECT department, AVG(salary)
FROM employees
GROUP BY department;
```

Generally invalid in standard SQL:

``` sql
SELECT department, name, AVG(salary)
FROM employees
GROUP BY department;
```

A department can have several names, so SQL cannot choose one employee
name for the group without an additional rule.

------------------------------------------------------------------------

# 10. HAVING

`HAVING` filters groups, usually after `GROUP BY`.

-   `WHERE` filters individual rows.
-   `HAVING` filters groups.

### Syntax

``` sql
SELECT column_name, AGGREGATE_FUNCTION(column_name)
FROM table_name
GROUP BY column_name
HAVING aggregate_condition;
```

## 10.1 Departments with average salary above 45000

``` sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 45000;
```

Result: IT and Finance.

## 10.2 Departments with more than two employees

``` sql
SELECT
    department,
    COUNT(*) AS total_employees
FROM employees
GROUP BY department
HAVING COUNT(*) > 2;
```

Result: IT, with four employees.

## 10.3 Departments with total salary above 100000

``` sql
SELECT
    department,
    SUM(salary) AS total_salary
FROM employees
GROUP BY department
HAVING SUM(salary) > 100000;
```

Result: IT and Finance.

## 10.4 Combine WHERE, GROUP BY, and HAVING

``` sql
SELECT
    department,
    COUNT(*) AS total_employees,
    AVG(salary) AS average_salary
FROM employees
WHERE salary > 35000
GROUP BY department
HAVING COUNT(*) > 1;
```

Process:

1.  `WHERE` removes employees earning 35000 or less.
2.  `GROUP BY` groups the remaining rows by department.
3.  `HAVING` keeps groups with more than one qualifying employee.
4.  `SELECT` returns the requested results.

Expected result:

  department     total_employees   average_salary
  ------------ ----------------- ----------------
  IT                           3         51666.67
  HR                           2            40000
  Finance                      2            65000

### WHERE vs HAVING

  -----------------------------------------------------------------------
  WHERE                               HAVING
  ----------------------------------- -----------------------------------
  Filters individual rows             Filters groups

  Applied before grouping             Applied after grouping

  Commonly filters ordinary column    Commonly filters aggregate results
  values                              

  Can be used without GROUP BY        Can also be used without GROUP BY
                                      in many databases, as a condition
                                      on the aggregate result
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 11. String Functions

String functions manipulate text values.

Examples include converting text to uppercase or lowercase, finding
string length, joining strings, extracting text, removing spaces, and
replacing text.

> **Database compatibility:** String function names and behavior can
> differ between MySQL, PostgreSQL, SQL Server, and Oracle. Examples
> below use common syntax; differences are mentioned where relevant.

## 11.1 UPPER()

Converts text to uppercase.

``` sql
SELECT UPPER('hello world');
```

Result:

``` text
HELLO WORLD
```

With a table:

``` sql
SELECT UPPER(name)
FROM employees;
```

## 11.2 LOWER()

Converts text to lowercase.

``` sql
SELECT LOWER('HELLO WORLD');
```

Result:

``` text
hello world
```

Case-normalized comparison:

``` sql
SELECT *
FROM employees
WHERE LOWER(name) = 'rahul';
```

For large tables, applying a function to a column in a filter may
prevent ordinary index usage. Consider a case-insensitive collation or
functional index where appropriate.

## 11.3 LENGTH()

Returns the length of a string.

``` sql
SELECT LENGTH('Rahul');
```

Result: `5`.

Database differences:

-   MySQL `LENGTH()` counts bytes.
-   PostgreSQL `LENGTH()` counts characters.
-   SQL Server commonly uses `LEN()`.

## 11.4 CHAR_LENGTH()

Returns the number of characters in a string in MySQL and PostgreSQL.

``` sql
SELECT CHAR_LENGTH('Hello');
```

Result: `5`.

This differs from byte length for multibyte characters.

## 11.5 CONCAT()

Combines strings.

``` sql
SELECT CONCAT('Satendra', ' ', 'Kumar');
```

Result:

``` text
Satendra Kumar
```

Example with columns:

``` sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM employees;
```

NULL behavior differs by database. For example, MySQL `CONCAT()` returns
NULL if an argument is NULL, whereas PostgreSQL `concat()` treats NULL
arguments as empty strings.

## 11.6 SUBSTRING()

Extracts part of a string.

Common syntax:

``` sql
SUBSTRING(string, start_position, length)
```

Examples:

``` sql
SELECT SUBSTRING('Satendra', 1, 4);
-- Sate

SELECT SUBSTRING('Hello World', 7, 5);
-- World
```

Positions are one-based in these common implementations.

### Extract part of an email

MySQL:

``` sql
SELECT SUBSTRING_INDEX('rahul@gmail.com', '@', 1);
-- rahul
```

PostgreSQL:

``` sql
SELECT SPLIT_PART('rahul@gmail.com', '@', 1);
-- rahul
```

## 11.7 TRIM()

Removes spaces from both ends of a string.

``` sql
SELECT TRIM('   Hello World   ');
-- Hello World
```

## 11.8 LTRIM() and RTRIM()

``` sql
SELECT LTRIM('   Hello');
-- Hello

SELECT RTRIM('Hello   ');
-- Hello
```

`LTRIM()` removes leading spaces; `RTRIM()` removes trailing spaces.

## 11.9 REPLACE()

Replaces occurrences of one substring with another.

``` sql
SELECT REPLACE('Hello World', 'World', 'SQL');
-- Hello SQL
```

This changes the query output, not the stored table value.

## 11.10 LEFT() and RIGHT()

Extract characters from the beginning or end of a string.

``` sql
SELECT LEFT('Satendra', 4);
-- Sate

SELECT RIGHT('Satendra', 3);
-- dra
```

These functions are available in MySQL and some other databases; check
the target database for equivalent syntax.

## 11.11 REVERSE()

Reverses a string in databases that support this function, such as MySQL
and SQL Server.

``` sql
SELECT REVERSE('HELLO');
-- OLLEH
```

## 11.12 POSITION() and INSTR()

Find the position of a substring.

PostgreSQL:

``` sql
SELECT POSITION('@' IN 'rahul@gmail.com');
-- 6
```

MySQL:

``` sql
SELECT INSTR('rahul@gmail.com', '@');
-- 6
```

### String function revision

  Function          Purpose
  ----------------- --------------------------------
  `UPPER()`         Convert to uppercase
  `LOWER()`         Convert to lowercase
  `LENGTH()`        Return length (bytes in MySQL)
  `CHAR_LENGTH()`   Count characters
  `CONCAT()`        Join strings
  `SUBSTRING()`     Extract a substring
  `TRIM()`          Remove surrounding spaces
  `LTRIM()`         Remove leading spaces
  `RTRIM()`         Remove trailing spaces
  `REPLACE()`       Replace text
  `LEFT()`          Extract from the left
  `RIGHT()`         Extract from the right
  `REVERSE()`       Reverse text
  `POSITION()`      Find substring position

------------------------------------------------------------------------

# 12. Date Functions

Date functions work with dates, times, and timestamps.

Common data types:

  -----------------------------------------------------------------------
  Type                    Stores                  Example
  ----------------------- ----------------------- -----------------------
  `DATE`                  Date only               `2026-11-27`

  `TIME`                  Time only               `14:30:45`

  `DATETIME`              Date and time (MySQL)   `2026-11-27 14:30:45`

  `TIMESTAMP`             Timestamp; timezone     `2026-11-27 14:30:45`
                          behavior depends on     
                          database                
  -----------------------------------------------------------------------

## 12.1 CURRENT_DATE

Returns the current date according to the database session's date/time
context.

``` sql
SELECT CURRENT_DATE;
```

Example output:

``` text
2026-11-27
```

Find employees who joined today if `joining_date` is a DATE column:

``` sql
SELECT *
FROM employees
WHERE joining_date = CURRENT_DATE;
```

## 12.2 CURRENT_TIME

Returns the current time.

``` sql
SELECT CURRENT_TIME;
```

## 12.3 NOW()

Returns the current date and time in MySQL and PostgreSQL.

``` sql
SELECT NOW();
```

Example output:

``` text
2026-11-27 12:30:45
```

Example insertion in MySQL or PostgreSQL:

``` sql
INSERT INTO orders (customer_id, created_at)
VALUES (101, NOW());
```

SQL Server commonly uses `GETDATE()` or `SYSDATETIME()` instead.

## 12.4 EXTRACT()

Extracts a component from a date or timestamp.

Syntax:

``` sql
EXTRACT(field FROM date_value)
```

Examples:

``` sql
SELECT EXTRACT(YEAR FROM DATE '2026-11-27');
-- 2026

SELECT EXTRACT(MONTH FROM DATE '2026-11-27');
-- 11

SELECT EXTRACT(DAY FROM DATE '2026-11-27');
-- 27
```

Extract joining year for every employee:

``` sql
SELECT
    name,
    EXTRACT(YEAR FROM joining_date) AS joining_year
FROM employees;
```

## 12.5 YEAR(), MONTH(), DAY()

These are common MySQL functions.

``` sql
SELECT YEAR(joining_date)
FROM employees;

SELECT MONTH(joining_date)
FROM employees;

SELECT DAY(joining_date)
FROM employees;
```

PostgreSQL commonly uses `EXTRACT()` for these operations.

## 12.6 DATE_ADD()

Adds an interval to a date. This is MySQL syntax.

``` sql
SELECT DATE_ADD('2026-11-27', INTERVAL 7 DAY);
-- 2026-12-04

SELECT DATE_ADD('2026-11-27', INTERVAL 2 MONTH);
-- 2027-01-27

SELECT DATE_ADD('2026-11-27', INTERVAL 1 YEAR);
-- 2027-11-27
```

PostgreSQL equivalent:

``` sql
SELECT DATE '2026-11-27' + INTERVAL '7 days';
```

**Use cases:** subscription expiry, payment due dates, trial periods,
and reminder dates.

## 12.7 DATE_SUB()

Subtracts an interval. This is MySQL syntax.

``` sql
SELECT DATE_SUB('2026-11-27', INTERVAL 7 DAY);
-- 2026-11-20
```

PostgreSQL equivalent:

``` sql
SELECT DATE '2026-11-27' - INTERVAL '7 days';
```

## 12.8 DATEDIFF()

Calculates the difference between dates in days.

MySQL:

``` sql
SELECT DATEDIFF('2026-11-27', '2026-11-20');
-- 7
```

Calculate calendar days since each employee joined:

``` sql
SELECT
    name,
    DATEDIFF(CURRENT_DATE, joining_date) AS days_since_joining
FROM employees;
```

This counts calendar days, not working days. It does not automatically
exclude weekends or holidays.

PostgreSQL:

``` sql
SELECT CURRENT_DATE - joining_date
FROM employees;
```

## 12.9 DATE_FORMAT()

Formats a date as a string. This is MySQL syntax.

``` sql
SELECT DATE_FORMAT('2026-11-27', '%d-%m-%Y');
-- 27-11-2026
```

Common MySQL format specifiers:

  Symbol   Meaning                Example
  -------- ---------------------- ----------
  `%Y`     Four-digit year        2026
  `%y`     Two-digit year         26
  `%m`     Month number           11
  `%M`     Full month name        November
  `%d`     Day of month           27
  `%H`     Hour, 24-hour format   14
  `%i`     Minutes                30
  `%s`     Seconds                45

Example:

``` sql
SELECT DATE_FORMAT(NOW(), '%d %M %Y');
```

PostgreSQL equivalent uses `TO_CHAR()`.

## 12.10 TO_CHAR()

Formats date/time values as strings in PostgreSQL and Oracle.

``` sql
SELECT TO_CHAR(
    DATE '2026-11-27',
    'DD Mon YYYY'
);
-- 27 Nov 2026
```

Format a time in PostgreSQL:

``` sql
SELECT TO_CHAR(
    CURRENT_TIMESTAMP,
    'HH24:MI:SS'
);
```

## 12.11 LAST_DAY()

Returns the last day of a month in MySQL and Oracle.

``` sql
SELECT LAST_DAY('2026-11-27');
-- 2026-11-30
```

Use cases include monthly billing, reporting, and calculating month-end
dates.

## 12.12 Filter a date range

Find employees who joined during 2023:

``` sql
SELECT *
FROM employees
WHERE joining_date BETWEEN '2023-01-01' AND '2023-12-31';
```

`BETWEEN` includes both boundary values.

### Important: Filter timestamp ranges safely

If `created_at` contains both date and time, this can unintentionally
exclude records later on November 30:

``` sql
-- Potentially problematic for timestamp columns
WHERE created_at BETWEEN '2026-11-01' AND '2026-11-30';
```

Prefer a half-open range:

``` sql
SELECT *
FROM orders
WHERE created_at >= '2026-11-01'
  AND created_at < '2026-12-01';
```

This includes every timestamp in November and is generally friendly to
indexes.

### Date function revision

  Function          Purpose                  Common database syntax
  ----------------- ------------------------ ------------------------
  `CURRENT_DATE`    Current date             MySQL, PostgreSQL
  `CURRENT_TIME`    Current time             MySQL, PostgreSQL
  `NOW()`           Current timestamp        MySQL, PostgreSQL
  `EXTRACT()`       Extract date component   MySQL, PostgreSQL
  `YEAR()`          Extract year             MySQL
  `MONTH()`         Extract month            MySQL
  `DAY()`           Extract day              MySQL
  `DATE_ADD()`      Add interval             MySQL
  `DATE_SUB()`      Subtract interval        MySQL
  `DATEDIFF()`      Difference in days       MySQL
  `DATE_FORMAT()`   Format date              MySQL
  `TO_CHAR()`       Format date/time         PostgreSQL, Oracle
  `LAST_DAY()`      Last day of month        MySQL, Oracle

------------------------------------------------------------------------

# 13. Logical SQL query execution order

For the clauses covered in these notes, the simplified logical
processing order is:

1.  `FROM` -- identify source tables.
2.  `WHERE` -- filter individual rows.
3.  `GROUP BY` -- form groups.
4.  `HAVING` -- filter groups.
5.  `SELECT` -- calculate and select output expressions.
6.  `DISTINCT` -- remove duplicate result rows.
7.  `ORDER BY` -- sort results.
8.  `LIMIT` / `OFFSET` -- restrict returned rows.

Actual database engines may use a different physical execution plan for
optimization.

### Why does execution order matter?

Consider:

``` sql
SELECT salary * 12 AS annual_salary
FROM employees
WHERE annual_salary > 500000;
```

In many databases this fails because the SELECT alias is not available
to the WHERE clause.

Use the expression directly:

``` sql
SELECT salary * 12 AS annual_salary
FROM employees
WHERE salary * 12 > 500000;
```

------------------------------------------------------------------------

# 14. Combined real-world query

Requirement:

Find the top three highest-paid employees who:

1.  Work in IT or HR.
2.  Earn more than 40000.
3.  Do not live in Delhi.
4.  Display name, department, city, and salary.
5.  Sort by salary from highest to lowest.

``` sql
SELECT DISTINCT
    name,
    department,
    city,
    salary
FROM employees
WHERE (department = 'IT' OR department = 'HR')
  AND salary > 40000
  AND city <> 'Delhi'
ORDER BY salary DESC
LIMIT 3;
```

This combines selection, duplicate removal, filtering, logical
operators, sorting, and limiting.

------------------------------------------------------------------------

# 15. Interview revision questions

1.  What is the difference between `SELECT *` and selecting specific
    columns?
2.  What is the difference between `DISTINCT` and `UNIQUE`?
3.  What is the difference between `COUNT(*)` and `COUNT(column)`?
4.  Does `AVG()` consider NULL values?
5.  What is the difference between `AND` and `OR`?
6.  What is the logical precedence of `NOT`, `AND`, and `OR`?
7.  What is the difference between `ASC` and `DESC`?
8.  What is the difference between `LIMIT` and `OFFSET`?
9.  What is the purpose of `GROUP BY`?
10. Can we group by multiple columns?
11. What is the difference between `WHERE` and `HAVING`?
12. Can we use aggregate functions directly in `WHERE`?
13. What is the difference between `LENGTH()` and `CHAR_LENGTH()`?
14. What is the difference between `CONCAT()` and `SUBSTRING()`?
15. What is the difference between `DATE` and `TIMESTAMP`?
16. How do you calculate the difference between two dates?
17. How can you filter all records from a particular month?
18. Why is a half-open timestamp range useful?

------------------------------------------------------------------------

# 16. Practice exercises

Use the `employees` table from the beginning of this document.

1.  Display the name and salary of all employees.
2.  Find all unique departments.
3.  Find employees whose age is greater than 25.
4.  Find IT employees with salary greater than 45000.
5.  Find employees who live in Delhi or Mumbai.
6.  Find employees who do not belong to HR.
7.  Sort employees by salary from highest to lowest.
8.  Find the top two highest-paid employees.
9.  Count the total number of employees.
10. Calculate the total salary of all employees.
11. Find the average salary of employees.
12. Find the highest and lowest salaries.
13. Count employees in each department.
14. Calculate the average salary of each department.
15. Find departments with more than two employees.
16. Find departments whose total salary exceeds 100000.
17. Display all employee names in uppercase.
18. Display employee names and their lengths.
19. Extract the year in which each employee joined.
20. Find employees who joined during 2023.
21. Find the latest joining date.
22. Count the number of unique cities.

## Suggested learning sequence after today's topics

1.  SQL JOINs: INNER, LEFT, RIGHT, FULL OUTER, SELF JOIN.
2.  Subqueries: nested queries, correlated subqueries, `EXISTS`, `IN`.
3.  Constraints: PRIMARY KEY, FOREIGN KEY, UNIQUE, NOT NULL, CHECK,
    DEFAULT.
4.  CTEs: `WITH` clause.
5.  Window functions: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`.
6.  Indexing and query optimization.
7.  Database normalization and transactions.

------------------------------------------------------------------------

## Final revision

  Topic              Main purpose
  ------------------ --------------------------------
  SELECT             Retrieve data
  DISTINCT           Remove duplicate results
  WHERE              Filter rows
  AND                Require all conditions
  OR                 Require at least one condition
  NOT                Reverse a condition
  ORDER BY           Sort results
  LIMIT              Restrict result count
  OFFSET             Skip rows
  COUNT              Count rows or non-NULL values
  SUM                Calculate total
  AVG                Calculate average
  MIN                Find minimum
  MAX                Find maximum
  GROUP BY           Group similar rows
  HAVING             Filter groups
  String functions   Manipulate text
  Date functions     Work with dates and times
