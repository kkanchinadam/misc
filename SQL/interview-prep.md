# SQL — Interview Prep

---

## 1. Core SQL Statements

**Q: What are the key DML and DDL statements?**

**DML (Data Manipulation Language)** — works with data:
- **`SELECT`** — retrieve rows: `SELECT name, age FROM users WHERE age > 25 ORDER BY name;`
- **`INSERT`** — add rows: `INSERT INTO users (name, age) VALUES ('Alice', 30);`
- **`UPDATE`** — modify rows: `UPDATE users SET age = 31 WHERE name = 'Alice';`
- **`DELETE`** — remove rows: `DELETE FROM users WHERE age < 18;`

**DDL (Data Definition Language)** — works with structure:
- **`CREATE TABLE`** — define a new table.
- **`ALTER TABLE`** — add, modify, or drop columns/constraints.
- **`DROP TABLE`** — permanently delete a table and all its data.
- **`TRUNCATE TABLE`** — delete all rows fast (no WHERE, no rollback in most DBs). Faster than DELETE for clearing a table.

**TCL (Transaction Control Language)**:
- **`COMMIT`** — persist the current transaction.
- **`ROLLBACK`** — undo changes since the last commit.
- **`SAVEPOINT`** — set a named checkpoint within a transaction.

---

**Q: What is the difference between DELETE, TRUNCATE, and DROP?**

| | `DELETE` | `TRUNCATE` | `DROP` |
|---|---|---|---|
| Removes | Selected rows (or all if no WHERE) | All rows | Entire table (structure + data) |
| WHERE clause | Yes | No | No |
| Rollback | Yes (transactional) | No (in most DBs) | No |
| Triggers | Fires row-level triggers | Does not fire triggers | Does not fire triggers |
| Speed | Slower (logged per row) | Very fast | Instant |
| Use when | Selectively removing rows | Clearing a table but keeping structure | Getting rid of the table entirely |

---

## 2. Filtering, Sorting & Aggregation

**Q: How do WHERE, GROUP BY, HAVING, and ORDER BY work together?**

SQL query execution order (logical, not written order):
1. `FROM` / `JOIN` — determine the source rows
2. `WHERE` — filter individual rows (before grouping)
3. `GROUP BY` — group remaining rows
4. `HAVING` — filter groups (after grouping)
5. `SELECT` — compute output columns
6. `ORDER BY` — sort the result
7. `LIMIT` / `OFFSET` — paginate

```sql
SELECT dept_id, COUNT(*) AS headcount, AVG(salary) AS avg_salary
FROM employees
WHERE status = 'active'           -- filter rows first
GROUP BY dept_id                  -- group
HAVING COUNT(*) > 5               -- filter groups
ORDER BY avg_salary DESC          -- sort output
LIMIT 10;                         -- paginate
```

**Key rule:** `WHERE` cannot reference aggregate functions (`COUNT`, `AVG`, etc.) — use `HAVING` for that.

---

**Q: What are aggregate functions?**

| Function | Description |
|---|---|
| `COUNT(*)` | Count all rows in the group |
| `COUNT(col)` | Count non-NULL values in col |
| `SUM(col)` | Sum of values |
| `AVG(col)` | Average of values |
| `MAX(col)` | Maximum value |
| `MIN(col)` | Minimum value |

```sql
SELECT dept_id,
       COUNT(*) AS total,
       MAX(salary) AS top_salary,
       MIN(salary) AS bottom_salary
FROM employees
GROUP BY dept_id;
```

---

## 3. JOINs

**Q: What are the types of JOINs and how do they differ?**

Using tables:
- `employees (id, name, dept_id)`
- `departments (id, dept_name)`

| JOIN Type | Returns |
|---|---|
| **INNER JOIN** | Only rows with a match in both tables. No match → excluded entirely. |
| **LEFT JOIN** (LEFT OUTER) | All rows from the left table. Non-matching right rows → NULL. |
| **RIGHT JOIN** (RIGHT OUTER) | All rows from the right table. Non-matching left rows → NULL. |
| **FULL OUTER JOIN** | All rows from both tables. NULL on whichever side has no match. |
| **CROSS JOIN** | Cartesian product — every combination of left and right rows. |
| **SELF JOIN** | A table joined with itself using aliases. Good for hierarchical data. |

```sql
-- INNER JOIN: employees who have a department
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;

-- LEFT JOIN: all employees, dept_name is NULL if unassigned
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;

-- FULL OUTER JOIN: all employees and all departments, NULLs where no match
SELECT e.name, d.dept_name
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.id;

-- SELF JOIN: find each employee and their manager
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;

-- CROSS JOIN: pair every employee with every project
SELECT e.name, p.project_name
FROM employees e
CROSS JOIN projects p;
```

**Quick memory rule:** LEFT JOIN = everything from left + matches from right. Think of it as "keep left, attach right where possible."

---

## 4. Subqueries & CTEs

**Q: What is a subquery?**

A subquery is a `SELECT` nested inside another SQL statement. It can appear in `WHERE`, `FROM`, or `SELECT` clauses.

```sql
-- Subquery in WHERE: employees earning more than the average
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Subquery in FROM (derived table): per-dept averages
SELECT dept_id, avg_sal
FROM (
    SELECT dept_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY dept_id
) AS dept_averages
WHERE avg_sal > 70000;

-- Correlated subquery: references outer query — runs once per row
SELECT name
FROM employees e
WHERE salary = (
    SELECT MAX(salary) FROM employees WHERE dept_id = e.dept_id
);
```

A **correlated subquery** references the outer query and re-executes for each outer row — can be slow. Often rewritable as a JOIN or window function.

---

**Q: What is a CTE (Common Table Expression)?**

A CTE is a named temporary result set defined with `WITH`, valid only for the duration of the query. It makes complex queries more readable and avoids repeated subexpressions.

```sql
WITH dept_stats AS (
    SELECT dept_id, AVG(salary) AS avg_sal, COUNT(*) AS headcount
    FROM employees
    GROUP BY dept_id
),
large_depts AS (
    SELECT dept_id FROM dept_stats WHERE headcount > 10
)
SELECT e.name, e.salary, ds.avg_sal
FROM employees e
JOIN dept_stats ds ON e.dept_id = ds.dept_id
WHERE e.dept_id IN (SELECT dept_id FROM large_depts);
```

**Recursive CTEs** — used for hierarchical data (org charts, category trees):

```sql
WITH RECURSIVE org_chart AS (
    -- anchor: top-level employees (no manager)
    SELECT id, name, manager_id, 1 AS level
    FROM employees WHERE manager_id IS NULL

    UNION ALL

    -- recursive: employees whose manager is in the current result
    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level;
```

---

## 5. Window Functions

**Q: What are window functions and why are they useful?**

Window functions perform calculations **across a set of rows related to the current row**, without collapsing the result into groups (unlike `GROUP BY`). The `OVER()` clause defines the window.

```sql
-- ROW_NUMBER: rank employees within each department by salary
SELECT name, dept_id, salary,
       ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rank
FROM employees;

-- Running total of sales
SELECT order_date, amount,
       SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- LAG/LEAD: compare with previous/next row
SELECT order_date, amount,
       LAG(amount, 1) OVER (ORDER BY order_date) AS prev_amount,
       amount - LAG(amount, 1) OVER (ORDER BY order_date) AS change
FROM orders;
```

**Key window functions:**

| Function | Purpose |
|---|---|
| `ROW_NUMBER()` | Unique sequential number per row within partition |
| `RANK()` | Rank with gaps on ties (1, 1, 3) |
| `DENSE_RANK()` | Rank without gaps on ties (1, 1, 2) |
| `NTILE(n)` | Divide rows into n buckets |
| `SUM/AVG/COUNT` | Cumulative or moving aggregates |
| `LAG(col, n)` | Value from n rows before current row |
| `LEAD(col, n)` | Value from n rows after current row |
| `FIRST_VALUE / LAST_VALUE` | First/last value in the window frame |

**`PARTITION BY`** divides rows into groups (like GROUP BY, but keeps all rows).  
**`ORDER BY`** within OVER defines the ordering within each partition.

---

## 6. Indexes

**Q: What is an index and why does it matter?**

An index is a data structure (typically a B-tree) that the database maintains alongside a table to allow fast lookup by one or more columns — avoiding a full table scan.

```sql
CREATE INDEX idx_employees_dept ON employees(dept_id);
CREATE UNIQUE INDEX idx_users_email ON users(email);   -- enforces uniqueness too
```

**When to index:**
- Columns frequently used in `WHERE`, `JOIN ON`, `ORDER BY`.
- Foreign key columns.
- High-cardinality columns (many distinct values) benefit most.

**Trade-offs:**
- Reads become faster; writes (INSERT/UPDATE/DELETE) become slightly slower because indexes must be updated.
- Indexes consume disk space.
- Too many indexes hurt write-heavy tables more than they help.

**Composite index** — index on multiple columns. Order matters: `(dept_id, salary)` helps queries filtering by `dept_id` alone or `dept_id + salary`, but NOT by `salary` alone.

---

**Q: What is the difference between a clustered and non-clustered index?**

- **Clustered index** — determines the physical storage order of rows. A table can have only one. In most databases, the primary key is the clustered index by default. Lookups by the clustered key are very fast because the data is right there.
- **Non-clustered index** — a separate structure that stores index values + pointers (row IDs) back to the actual data. A table can have many. Slightly slower than clustered because of the extra pointer lookup.

---

## 7. Constraints & Keys

**Q: What are the common constraints in SQL?**

| Constraint | Purpose |
|---|---|
| `PRIMARY KEY` | Uniquely identifies each row. Implies NOT NULL + UNIQUE. One per table. |
| `FOREIGN KEY` | Enforces referential integrity — value must exist in referenced table. |
| `UNIQUE` | All values in the column(s) must be distinct. |
| `NOT NULL` | Column cannot hold NULL. |
| `CHECK` | Value must satisfy a boolean expression: `CHECK (age >= 0)`. |
| `DEFAULT` | Provides a default value if none is supplied on INSERT. |

```sql
CREATE TABLE orders (
    id         INT PRIMARY KEY,
    user_id    INT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    total      DECIMAL(10,2) CHECK (total >= 0),
    status     VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW()
);
```

**`ON DELETE CASCADE`** — automatically delete child rows when the parent is deleted. Alternative: `SET NULL`, `RESTRICT`, `NO ACTION`.

---

**Q: What is the difference between a primary key and a foreign key?**

- **Primary key** — uniquely identifies a row within its own table. Cannot be NULL. Each table has at most one.
- **Foreign key** — a column in one table that references the primary key of another table. Enforces referential integrity — you can't insert a foreign key value that doesn't exist in the parent table (and can't delete a parent row that has dependent children, unless CASCADE is set).

---

## 8. Normalization

**Q: What is normalization and why does it matter?**

Normalization is the process of organizing a database to reduce **data redundancy** and improve **data integrity** by decomposing tables into smaller, well-structured ones.

**Normal Forms (the key ones):**

- **1NF (First Normal Form)** — each column holds atomic (indivisible) values; no repeating groups. Each row is unique.
- **2NF (Second Normal Form)** — 1NF + every non-key column is fully dependent on the entire primary key (no partial dependencies). Relevant when the primary key is composite.
- **3NF (Third Normal Form)** — 2NF + no transitive dependencies (non-key column depending on another non-key column).

**Example of a 3NF violation:**

| order_id | customer_id | customer_city |
|---|---|---|
| 1 | 42 | New York |

`customer_city` depends on `customer_id`, not on `order_id`. Fix: move `customer_city` to a `customers` table.

**Denormalization** — intentionally introducing redundancy for read performance (e.g., caching a `customer_city` in the orders table to avoid a join). Common in analytics/reporting databases.

---

## 9. Transactions & ACID

**Q: What is a transaction and what are ACID properties?**

A **transaction** is a unit of work that either completes fully or not at all.

- **Atomicity** — all operations in the transaction succeed, or none do. If any step fails, the entire transaction is rolled back.
- **Consistency** — the database moves from one valid state to another. Constraints are never violated.
- **Isolation** — concurrent transactions don't interfere with each other. Each transaction sees a consistent snapshot.
- **Durability** — once committed, data persists even if the system crashes.

```sql
BEGIN;
    UPDATE accounts SET balance = balance - 500 WHERE id = 1;
    UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT;   -- both updates succeed together
-- If any statement fails: ROLLBACK undoes everything
```

---

**Q: What are transaction isolation levels?**

Isolation levels control the trade-off between consistency and concurrency. Lower isolation = more performance but more anomalies allowed.

| Isolation Level | Dirty Read | Non-repeatable Read | Phantom Read |
|---|---|---|---|
| `READ UNCOMMITTED` | Possible | Possible | Possible |
| `READ COMMITTED` (common default) | Prevented | Possible | Possible |
| `REPEATABLE READ` | Prevented | Prevented | Possible |
| `SERIALIZABLE` | Prevented | Prevented | Prevented |

- **Dirty read** — reading uncommitted changes from another transaction.
- **Non-repeatable read** — reading the same row twice within a transaction and getting different values.
- **Phantom read** — a re-executed query returns different rows (another transaction inserted/deleted).

Most production databases default to `READ COMMITTED`. Use `SERIALIZABLE` when strict consistency is critical (financial ledgers).

---

## 10. Performance & Query Optimization

**Q: How do you approach slow queries?**

1. **`EXPLAIN` / `EXPLAIN ANALYZE`** — shows the query execution plan. Look for sequential scans on large tables, nested loop joins with many rows, and missing indexes.

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 42 AND status = 'pending';
```

2. **Add indexes** on columns in WHERE/JOIN/ORDER BY that have high cardinality and are queried frequently.
3. **Avoid `SELECT *`** — fetch only the columns you need.
4. **Avoid functions on indexed columns in WHERE** — `WHERE YEAR(created_at) = 2024` prevents index use. Use `WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01'` instead.
5. **Avoid `OR` on different columns** — can prevent index use. Consider `UNION` instead.
6. **Pagination with `LIMIT/OFFSET`** — large offsets are slow (DB scans and discards). Use keyset pagination (`WHERE id > last_seen_id`) for large datasets.
7. **Rewrite correlated subqueries** as JOINs or CTEs where possible.

---

**Q: What is the N+1 query problem?**

The N+1 problem occurs when you run 1 query to fetch a list of N items, then run N additional queries to fetch related data for each item — totaling N+1 queries instead of 1 or 2.

```sql
-- Bad: 1 query for orders + 1 per order to fetch user name = N+1 queries
SELECT * FROM orders;                          -- 1 query, returns 100 rows
SELECT name FROM users WHERE id = ?;           -- repeated 100 times

-- Good: 1 JOIN instead
SELECT o.id, o.total, u.name
FROM orders o
JOIN users u ON o.user_id = u.id;
```

In ORMs (like Spring Data JPA), this often surfaces when lazily loading relationships in a loop. Fix with `JOIN FETCH` or batch fetching.
