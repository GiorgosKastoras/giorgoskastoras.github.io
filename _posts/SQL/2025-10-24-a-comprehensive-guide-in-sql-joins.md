---
layout: post
title: A Comprehensive Guide in SQL Joins
date: 2025-10-24
permalink: /a-comprehensive-guide-in-sql-joins/
---

Join is an operation that combines rows from two or more tables based on related columns.

**The three pillars:**
- **Grain** — What does one row represent in each table?
- **Key(s)** — Which columns uniquely identify a row (primary key) and connect tables (foreign key)?
- **Cardinality** — one-to-one (1-1), one-to-many (1-*), many-to-many (*-*).


## Type of Joins

### JOIN (INNER)
Returns records that have matching values in both tables. If a row in one table doesn’t have a match in the other, it’s excluded.

**Example tables (before):**

**Customers**

| Customer_ID | Customer_Name |
| ---: | --- |
| 1 | Alice |
| 2 | Bob |
| 3 | Carol |

**Orders**

| Order_ID | Customer_ID | Amount |
| ---: | ---: | ---: |
| 101 | 1 | 50 |
| 102 | 1 | 30 |
| 103 | 4 | 20 |

**Query**
```sql
SELECT o.Order_ID, c.Customer_ID, c.Customer_Name, o.Amount
FROM Orders o
JOIN Customers c ON c.Customer_ID = o.Customer_ID;
````

**Result (after):**

| Order_ID | Customer_ID | Customer_Name | Amount |
| -------: | ----------: | ------------- | -----: |
|      101 |           1 | Alice         |     50 |
|      102 |           1 | Alice         |     30 |

---

### LEFT JOIN

Returns all records from the left table and the matched records from the right table. If there is no match, NULLs are returned for right columns.

**Query**

```sql
SELECT c.Customer_ID, c.Customer_Name, o.Order_ID, o.Amount
FROM Customers c
LEFT JOIN Orders o ON o.Customer_ID = c.Customer_ID;
```

**Result (after):**

| Customer_ID | Customer_Name | Order_ID | Amount |
| ----------: | ------------- | -------: | -----: |
|           1 | Alice         |      101 |     50 |
|           1 | Alice         |      102 |     30 |
|           2 | Bob           |     NULL |   NULL |
|           3 | Carol         |     NULL |   NULL |

---

### RIGHT JOIN

Returns all records from the right table and the matched records from the left table.

**Query**

```sql
SELECT c.Customer_ID, c.Customer_Name, o.Order_ID, o.Amount
FROM Customers c
RIGHT JOIN Orders o ON o.Customer_ID = c.Customer_ID;
```

**Result (after):**

| Customer_ID | Customer_Name | Order_ID | Amount |
| ----------: | ------------- | -------: | -----: |
|           1 | Alice         |      101 |     50 |
|           1 | Alice         |      102 |     30 |
|        NULL | NULL          |      103 |     20 |

---

### FULL OUTER JOIN (FULL JOIN)

Returns all records when there is a match in either left or right table.

**Query**

```sql
SELECT c.Customer_ID, c.Customer_Name, o.Order_ID, o.Amount
FROM Customers c
FULL OUTER JOIN Orders o ON o.Customer_ID = c.Customer_ID;
```

**Result (after):**

| Customer_ID | Customer_Name | Order_ID | Amount |
| ----------: | ------------- | -------: | -----: |
|           1 | Alice         |      101 |     50 |
|           1 | Alice         |      102 |     30 |
|           2 | Bob           |     NULL |   NULL |
|           3 | Carol         |     NULL |   NULL |
|        NULL | NULL          |      103 |     20 |

## Other Types of Joins

### SELF JOIN

Merge a table with itself to compare rows or express hierarchies (e.g., manager → employee).

**Employees**

| emp_id | name  | manager_id |
| -----: | ----- | ---------: |
|      1 | Alice |       NULL |
|      2 | Bob   |          1 |
|      3 | Carol |          1 |
|      4 | Dan   |          2 |

**Query**

```sql
SELECT e.emp_id, e.name AS employee, m.name AS manager
FROM Employees e
JOIN Employees m ON m.emp_id = e.manager_id;
```

**Result (after):**

| emp_id | employee | manager |
| -----: | -------- | ------- |
|      2 | Bob      | Alice   |
|      3 | Carol    | Alice   |
|      4 | Dan      | Bob     |

---

### CROSS JOIN

Produces all possible pairings (Cartesian product). No join condition required.

**Students**

| student |
| ------- |
| Anna    |
| Ben     |

**Subjects**

| subject |
| ------- |
| Math    |
| English |

**Query**

```sql
SELECT s.student, t.subject
FROM Students s
CROSS JOIN Subjects t;
```

**Result (after):**

| student | subject |
| ------- | ------- |
| Anna    | Math    |
| Anna    | English |
| Ben     | Math    |
| Ben     | English |

---

### SEMI JOIN

Returns left-table rows that have a match in the right table; only left columns are returned.

**Query**

```sql
SELECT c.Customer_ID, c.Customer_Name
FROM Customers c
WHERE EXISTS (
  SELECT 1
  FROM Orders o
  WHERE c.Customer_ID = o.Customer_ID
);
```

**Result (after):**

| Customer_ID | Customer_Name |
| ----------: | ------------- |
|           1 | Alice         |

---

### ANTI JOIN

Returns left-table rows that have **no** match in the right table.

**Query**

```sql
SELECT c.Customer_ID, c.Customer_Name
FROM Customers c
LEFT JOIN Orders o ON o.Customer_ID = c.Customer_ID
WHERE o.Customer_ID IS NULL;
```

**Result (after):**

| Customer_ID | Customer_Name |
| ----------: | ------------- |
|           2 | Bob           |
|           3 | Carol         |

## Join Mindset

1) **Start with the end in mind (declare the grain).**
   Example: one row per `(store_id, sku, date)`.

2) **Pick a driving table (and be explicit why).**
   Use LEFT when data quality isn’t guaranteed; INNER when referential integrity is trusted.

3) **Make keys & cardinality explicit.**

   * fact→dim = many→one
   * fact→fact = many↔many → pre-aggregate the many side.

4) **Choose the right pattern.**
   Enrich: LEFT/INNER · Presence: `EXISTS` (SEMI) · Absence: `LEFT … WHERE right IS NULL` or `NOT EXISTS` (ANTI) · Reconciliation: FULL · Combinations: CROSS (intentional).

5) **Put filters in the right place (ON vs WHERE).**

```sql
-- GOOD: keeps unmatched promos as NULL (true LEFT)
LEFT JOIN promo pr
  ON pr.sku = s.sku
 AND s.sale_dt BETWEEN pr.start_dt AND pr.end_dt
-- BAD: WHERE pr.start_dt ...  -- turns LEFT into INNER
```

6) **Aggregate at the correct moment (pre-agg many side).**

```sql
WITH pr_by_sale AS (
  SELECT sale_id, COUNT(*) AS promos
  FROM fact_promotions pr
  JOIN fact_sales s
    ON pr.sku = s.sku
   AND s.sale_dt BETWEEN pr.start_dt AND pr.end_dt
  GROUP BY sale_id
)
SELECT s.*, COALESCE(p.promos,0) AS promos
FROM fact_sales s
LEFT JOIN pr_by_sale p USING (sale_id);
```

7) **Treat NULLs deliberately.**
   `NULL = NULL` is unknown; use `COALESCE` only when correct.

8) **Scaffolds help robustness.**

```sql
WITH frame AS (
  SELECT c.dt, a.store_id, a.sku
  FROM dim_calendar c
  JOIN assortment a ON c.dt BETWEEN a.start_dt AND a.end_dt
  WHERE c.dt BETWEEN DATE '2025-09-01' AND DATE '2025-09-07'
)
SELECT f.dt, f.store_id, f.sku,
       COALESCE(SUM(s.units),0) AS units
FROM frame f
LEFT JOIN fact_sales s
  ON s.sale_dt = f.dt
 AND s.store_id = f.store_id
 AND s.sku = f.sku
GROUP BY 1,2,3;
```

### Performance habits (BigQuery-like engines)

* Filter early; select only needed columns.
* Pre-aggregate many sides before joins.
* Partition/cluster big facts by date and common keys.
* Prefer `EXISTS` to `JOIN + DISTINCT` for presence tests.
* Avoid `SELECT *` in production.

### Reusable mini-templates

**Presence (SEMI)**

```sql
SELECT p.sku
FROM dim_product p
WHERE EXISTS (
  SELECT 1 FROM fact_sales s
  WHERE s.sku = p.sku
    AND s.sale_dt >= CURRENT_DATE - INTERVAL 30 DAY
);
```

**Absence (ANTI)**

```sql
SELECT p.sku
FROM dim_product p
LEFT JOIN fact_sales s
  ON s.sku = p.sku
 AND s.sale_dt >= CURRENT_DATE - INTERVAL 30 DAY
WHERE s.sku IS NULL;
```

**SCD2 time-bounded**

```sql
SELECT s.sale_id, d.category
FROM fact_sales s
JOIN dim_product_scd2 d
  ON d.sku = s.sku
 AND s.sale_dt >= d.effective_start_dt
 AND s.sale_dt <  d.effective_end_dt;
```

**Many-to-many guard (pre-aggregate)**

```sql
WITH sales_30 AS (
  SELECT sku, store_id, SUM(units) AS units_30
  FROM fact_sales
  WHERE sale_dt >= CURRENT_DATE - INTERVAL 30 DAY
  GROUP BY 1,2
)
SELECT f.store_id, f.sku, s.units_30
FROM frame f
LEFT JOIN sales_30 s USING (store_id, sku);
```

**Mental checklist**

* What’s my **result grain**?
* What’s the **driver** table?
* What are the **keys** and the **cardinality**?
* Do I need **presence/absence** (`EXISTS` / `NOT EXISTS`) or **enrichment** (`JOIN`)?
* Are right-side filters in the **ON** clause?
* Do I need to **pre-aggregate** to avoid duplication?
* How do **NULLs** affect logic and totals?
* Quick **row-count & dup** checks after each step.
* Any **performance** wins (filter early, select few, partitions/cluster)?
* Does the output still match the declared grain?

