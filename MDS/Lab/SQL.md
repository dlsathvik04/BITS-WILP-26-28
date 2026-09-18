# Advanced SQL Quick Syntax Reference

A concise, syntax-first study guide covering Aggregations, Grouping, Join types, Nested Subqueries, Correlated Subqueries, and Common Table Expressions (CTEs).

---

## 1. Aggregations & Grouping

Aggregate functions compute a single result from a set of input values.

### 1.1 Core Aggregate Functions

```sql
SELECT 
    COUNT(*) AS total_rows,
    COUNT(DISTINCT department_id) AS distinct_depts,
    SUM(salary) AS total_payroll,
    AVG(salary) AS avg_salary,
    MIN(salary) AS lowest_salary,
    MAX(salary) AS highest_salary
FROM employees;
```

### 1.2 `GROUP BY` and `HAVING`

* **`WHERE`** filters individual rows **before** aggregation.
* **`HAVING`** filters aggregated groups **after** `GROUP BY`.

```sql
-- Find departments with more than 5 employees and average salary > 60,000
SELECT 
    department_id,
    COUNT(emp_id) AS head_count,
    ROUND(AVG(salary), 2) AS avg_salary
FROM employees
WHERE status = 'ACTIVE'                -- Row filter before aggregation
GROUP BY department_id
HAVING COUNT(emp_id) > 5               -- Group filter after aggregation
   AND AVG(salary) > 60000;
```

---

## 2. Joins (Relational Operations)

Used to combine columns from two or more tables based on a related column.

### 2.1 Quick Join Summary Matrix

| Join Type | Returns |
| :--- | :--- |
| **`INNER JOIN`** | Only rows with matching values in **both** tables |
| **`LEFT JOIN`** | All rows from Left table + matching from Right (or `NULL`) |
| **`RIGHT JOIN`** | All rows from Right table + matching from Left (or `NULL`) |
| **`FULL OUTER JOIN`** | All rows when there is a match in **either** table |
| **`CROSS JOIN`** | Cartesian product (every row combined with every row) |
| **`SELF JOIN`** | A table joined with itself (requires aliasing) |

---

### 2.2 Inner Join

```sql
SELECT 
    e.emp_name,
    d.dept_name
FROM employees e
INNER JOIN departments d 
    ON e.department_id = d.dept_id;
```

### 2.3 Left Outer Join & Finding Missing Records

```sql
-- Return all employees, including those without assigned departments
SELECT 
    e.emp_name,
    d.dept_name
FROM employees e
LEFT JOIN departments d 
    ON e.department_id = d.dept_id;

-- Anti-Join: Find employees that have NO department assigned
SELECT e.emp_name
FROM employees e
LEFT JOIN departments d 
    ON e.department_id = d.dept_id
WHERE d.dept_id IS NULL;
```

### 2.4 Full Outer Join

```sql
SELECT 
    e.emp_name,
    d.dept_name
FROM employees e
FULL OUTER JOIN departments d 
    ON e.department_id = d.dept_id;
```

### 2.5 Self Join

Useful for hierarchical structures (e.g., employee-to-manager relationships):

```sql
SELECT 
    e.emp_name AS employee,
    m.emp_name AS manager
FROM employees e
LEFT JOIN employees m 
    ON e.manager_id = m.emp_id;
```

### 2.6 Cross Join (Cartesian Product)

```sql
-- Combines every size with every color
SELECT p.size, c.color_name
FROM product_sizes p
CROSS JOIN colors c;
```

---

## 3. Nested Queries (Subqueries)

A query nested inside a `SELECT`, `FROM`, or `WHERE` clause.

### 3.1 Scalar Subquery (Single Value)

Returns exactly one row and one column:

```sql
-- Find employees earning more than the company-wide average
SELECT emp_name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary) 
    FROM employees
);
```

### 3.2 Row-Set Subquery (`IN` / `NOT IN`)

Evaluates membership against a list returned by a subquery:

```sql
-- Find departments that have at least one active high-earner (> 100k)
SELECT dept_name
FROM departments
WHERE dept_id IN (
    SELECT DISTINCT department_id
    FROM employees
    WHERE salary > 100000
);
```

### 3.3 Derived Tables (Subquery in `FROM`)

Every derived table **must** have an alias:

```sql
-- Calculate the average departmental total payroll
SELECT AVG(dept_payroll) AS avg_department_spend
FROM (
    SELECT department_id, SUM(salary) AS dept_payroll
    FROM employees
    GROUP BY department_id
) AS dept_summary;
```

---

## 4. Correlated Subqueries & Existence

A correlated subquery references columns from the outer query. It is evaluated once for **each candidate row** processed by the outer query.

### 4.1 Correlated Comparison

```sql
-- Find employees who earn more than the average of THEIR OWN department
SELECT e.emp_name, e.department_id, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(sub.salary)
    FROM employees sub
    WHERE sub.department_id = e.department_id
);
```

### 4.2 `EXISTS` vs `NOT EXISTS`

`EXISTS` tests for the presence of rows. It halts execution as soon as the first matching row is found (short-circuit boolean check):

```sql
-- Find departments that currently employ at least one worker
SELECT d.dept_name
FROM departments d
WHERE EXISTS (
    SELECT 1 
    FROM employees e 
    WHERE e.department_id = d.dept_id
);

-- Find departments with NO assigned workers
SELECT d.dept_name
FROM departments d
WHERE NOT EXISTS (
    SELECT 1 
    FROM employees e 
    WHERE e.department_id = d.dept_id
);
```

---

## 5. Multiple Value Subquery Operators (`ANY` / `ALL`)

* **`> ALL (subquery)`**: Greater than the maximum value from the subquery.
* **`> ANY (subquery)`**: Greater than at least one value (i.e., greater than the minimum).

```sql
-- Earns more than ALL employees in Department 10 (higher than the max of dept 10)
SELECT emp_name, salary
FROM employees
WHERE salary > ALL (
    SELECT salary 
    FROM employees 
    WHERE department_id = 10
);

-- Earns more than ANY employee in Department 10 (higher than the lowest earner of dept 10)
SELECT emp_name, salary
FROM employees
WHERE salary > ANY (
    SELECT salary 
    FROM employees 
    WHERE department_id = 10
);
```

---

## 6. Common Table Expressions (CTEs with `WITH`)

CTEs create temporary named result sets that make complex nested queries easier to read and modularize.

### 6.1 Basic CTE Syntax

```sql
WITH DeptAvg AS (
    SELECT 
        department_id,
        AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department_id
)
SELECT 
    e.emp_name,
    e.salary,
    d.avg_sal
FROM employees e
JOIN DeptAvg d 
    ON e.department_id = d.department_id
WHERE e.salary > d.avg_sal;
```

### 6.2 Multiple Chained CTEs

```sql
WITH ActiveEmployees AS (
    SELECT emp_id, department_id, salary
    FROM employees
    WHERE status = 'ACTIVE'
),
DeptSalaries AS (
    SELECT 
        department_id,
        SUM(salary) AS total_spend,
        COUNT(emp_id) AS total_staff
    FROM ActiveEmployees
    GROUP BY department_id
)
SELECT 
    d.dept_name,
    ds.total_spend,
    ds.total_staff
FROM departments d
JOIN DeptSalaries ds 
    ON d.dept_id = ds.department_id
WHERE ds.total_spend > 250000;
```

---

## 7. SQL Execution Order (Mental Model)

SQL queries are written top-to-bottom, but executed in this logical order:

```
1. FROM / JOIN     -> Identify data sources & row combinations
2. WHERE           -> Filter raw individual rows
3. GROUP BY        -> Aggregate rows into groups
4. HAVING          -> Filter aggregated groups
5. SELECT          -> Pick columns & compute scalar expressions
6. DISTINCT        -> Eliminate duplicate output rows
7. ORDER BY        -> Sort final output
8. LIMIT / OFFSET  -> Restrict number of output rows
```