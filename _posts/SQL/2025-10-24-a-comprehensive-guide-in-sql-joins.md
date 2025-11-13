---
layout: post
title: A Comprehensive Guide in SQL Joins
date: 2025-10-24
permalink: /a-comprehensive-guide-in-sql-joins/
---

Join is an operation performed on the relations or tables that combine rows from two or more relations or tables based on the related columns between them.

The three pillars:

1. **Grain:** What does one row represent in each table?  
2. **Key(s):** Which columns uniquely identify a row (primary key) and connect tables (foreign key)?  
3. **Cardinality:** one to one (1-1), one to many (1 - *), many to many (* - *).

---

## Type of Joins

**JOIN (INNER):** Returns records that have matching values in both tables.  
— Note: If a row in one table doesn’t have a corresponding row in another, it’s excluded from the final result.

**Example tables (before):**

**Customers**

| Customer_ID | Customer_Name |
|---:|---|
| 1 | Alice |
| 2 | Bob |
| 3 | Carol |

**Orders**

| Order_ID | Customer_ID | Amount |
|---:|---:|---:|
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

**LEFT JOIN:** Returns all records from the left table, and the matched records from the right table.
— Note: If there is no match, NULL values are returned for columns from the right table.

**Example tables (before):** *(same as above)*

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

**RIGHT JOIN:** Returns all records from the right table, and the matched records from the left table.

**Example tables (before):** *(same as above)*

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

**FULL OUTER JOIN (FULL JOIN):** Returns all records when there is a match in either left or right table.

**Example tables (before):** *(same as above)*

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

---

## Other types of Joins

### SELF JOIN

SELF JOIN in SQL is an approach to merge a table with itself, comparing rows inside the same table or arranging the data in the table.

There are some important situations where you should use SELF JOIN in SQL:

* You need to compare rows in a table (e.g., comparing the sales for a product across two different months).
* You have hierarchical data in a single table, such as an employee reporting structure.
* There is a parent–child relationship within the same dataset.

**Example tables (before):**

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

Produces all possible pairings by combining each row from one table with each row from another.
Unlike other joins, it doesn't need any matching requirements. A CROSS JOIN, for instance, will list every student and every subject in a database, including all combinations.

**Key points to remember:**

* No condition required: it combines every row from both tables automatically.
* Cartesian Product: it creates all possible combinations of rows. So, if a table has *m* rows and the other table has *n* rows, the results will have *m × n* rows.

**Example tables (before):**

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

Semi-join is a type of join that is applied to relations to join them based on the related columns. When semi-join is applied, it returns the rows from one table for which there are matching records in another related table.

**Characteristics:**

* A Semi-join returns rows from the left table for which there are corresponding matching rows in the right table.
* Unlike regular joins which include the matching rows from both tables, a semi-join only includes columns from the left table in the result.

In a nutshell: **Keep left rows that have a match, don’t return right columns.**

**Example (before):** *(Customers & Orders as above)*

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

Returns the rows from one table for which there are no matching records in another related table.

**Example (before):** *(Customers & Orders as above)*

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

---

## Join Mindset

### 1) Start with the end in mind (declare the grain)

Write a one-liner before you query:

* Target grain: “one row per (store_id, sku, date)”
* Required columns/measures: e.g., `units, net_amount, category, region`

Everything you do should preserve or intentionally change this grain.

---

### 2) Pick a driving table (and be explicit why)

* If you must not lose dates/SKUs, use a calendar/assortment scaffold as the left table.
* If the task is about realized facts, `fact_sales` is usually the driver.
* Use `LEFT JOIN` from the driver when data quality isn’t guaranteed; use `INNER` when referential integrity is trusted.

---

### 3) Make keys & cardinality explicit (avoid surprises)

* State the join keys and the expected relationship:

  * fact → dim = many → one
  * fact → fact (e.g., sales ↔ promotions) = many ↔ many (danger — pre-aggregate!)

* If it’s not 1:1 or many:1, decide before joining:

  * Pre-aggregate the many side to the driver’s keys, or
  * Accept duplication and aggregate **after** (rarely ideal).

---

### 4) Choose the right join *pattern* for the question

* Enrich: `LEFT JOIN` (or `JOIN` if safe).
* Presence test: SEMI (`WHERE EXISTS(...)`).
* Absence test: ANTI (`LEFT JOIN ... WHERE right IS NULL` or `WHERE NOT EXISTS(...)`).
* Reconciliation: `FULL OUTER JOIN`.
* Combinations: `CROSS JOIN` (only when intentional).

> Rule of thumb: If the question is “Does a match exist?” → use `EXISTS`.
> If the question is “Bring me attributes/measures” → use `JOIN` (pre-agg when needed).

---

### 5) Put filters in the right place (ON vs WHERE)

* Filters on the right table that shouldn’t kill the LEFT behavior belong in the `ON` clause.
* Filters on the left/driver can live in `WHERE`.
* Any right-table condition in `WHERE` turns a LEFT into an INNER.

```sql
-- GOOD: keeps unmatched promos as NULL (true LEFT)
LEFT JOIN promo pr
  ON pr.sku = s.sku
 AND s.sale_dt BETWEEN pr.start_dt AND pr.end_dt
-- BAD: WHERE pr.start_dt ...   -- converts to INNER
```

---

### 6) Aggregate at the correct moment

* If joining to a **many-side** just to count/sum, aggregate **before** the join.

```sql
WITH pr_by_sale AS (
  SELECT sale_id, COUNT(*) AS promos
  FROM fact_promotions pr
  JOIN fact_sales s
    ON pr.sku = s.sku
   AND s.sale_dt BETWEEN pr.start_dt AND pr.end_dt
  GROUP BY sale_id
)
SELECT s.*, COALESCE(p.promos,0)
FROM fact_sales s
LEFT JOIN pr_by_sale p USING (sale_id);
```

---

### 7) Treat NULLs deliberately

* `NULL = NULL` is unknown → never matches.
* Use `COALESCE` only when the replacement is semantically correct (e.g., `COALESCE(units,0)` for sums).
* Remember: in a LEFT JOIN **no-match**, every `right.*` becomes **NULL in the result** (even if not NULL in the base table).

---

### 8) Build with scaffolds for robustness

For “all stores × all days × all listed SKUs,” generate the frame first (calendar × store × assortment), then left-join facts. This converts “missing data” into **explicit zeros/NULLs** you can reason about.

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

---

### 11) Performance habits (especially BigQuery-like engines)

* Filter early, project few columns.
* Pre-aggregate the many side before joins.
* Partition/cluster big facts by date and by common join keys (`store_id, sku`).
* Prefer `EXISTS` over `JOIN + DISTINCT` for presence tests (less shuffle).
* Avoid `SELECT *` in production; be intentional.

---

### 12) Reusable mini-templates

**Presence (SEMI):**

```sql
SELECT p.sku
FROM dim_product p
WHERE EXISTS (
  SELECT 1 FROM fact_sales s
  WHERE s.sku = p.sku
    AND s.sale_dt >= CURRENT_DATE - INTERVAL 30 DAY
);
```

**Absence (ANTI):**

```sql
SELECT p.sku
FROM dim_product p
LEFT JOIN fact_sales s
  ON s.sku = p.sku
 AND s.sale_dt >= CURRENT_DATE - INTERVAL 30 DAY
WHERE s.sku IS NULL;
```

**SCD2 time-bounded:**

```sql
SELECT s.sale_id, d.category
FROM fact_sales s
JOIN dim_product_scd2 d
  ON d.sku = s.sku
 AND s.sale_dt >= d.effective_start_dt
 AND s.sale_dt <  d.effective_end_dt;
```

**Many-to-many guard (pre-aggregate):**

```sql
WITH sales_30 AS (
  SELECT sku, store_id, SUM(units) units_30
  FROM fact_sales
  WHERE sale_dt >= CURRENT_DATE - INTERVAL 30 DAY
  GROUP BY 1,2
)
SELECT f.store_id, f.sku, s.units_30
FROM frame f
LEFT JOIN sales_30 s USING (store_id, sku);
```

---

### 12) Mental checklist

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