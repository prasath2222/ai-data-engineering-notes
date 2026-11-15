# Module 5: SQL Joins & Aggregations

## 1. Why Do We Need `JOIN`s?
When we split data into separate tables so names and phone numbers don't repeat (like **`Customers`** and **`Orders`**), we need a way to stitch them back together whenever we want to see **"Who bought what?"**.

A **`JOIN`** glues two tables side-by-side by matching a shared ID column (like `customer_id`).

---

## 2. Sample Tables Used in All Examples Below

**Left Table: `Customers`**
| customer_id | name | city |
| :--- | :--- | :--- |
| **1** | Arun | Chennai |
| **2** | Priya | Coimbatore |
| **3** | Karthik | Madurai |

**Right Table: `Orders`**
| order_id | customer_id | item | amount |
| :--- | :--- | :--- | :--- |
| **101** | **1** | Laptop | 50,000 |
| **102** | **1** | Mouse | 500 |
| **103** | **2** | Keyboard | 1,500 |
| **104** | **99** | Monitor | 12,000 |

*(Notice two special cases here: **Karthik (`id = 3`)** has **0 orders**, and **Order `104`** belongs to **`customer_id = 99`**, who was deleted from `Customers`.)*

---

## 3. The 4 Types of SQL Joins

### A. `INNER JOIN` — "Strict Match Only"
- **Plain-English Rule:** Keep **only** rows where the `customer_id` exists in **BOTH** tables. Anyone without a match gets dropped.
- **Query:**
```sql
SELECT Customers.name, Orders.item, Orders.amount
FROM Customers
INNER JOIN Orders
  ON Customers.customer_id = Orders.customer_id;
```
- **Result:**
| name | item | amount |
| :--- | :--- | :--- |
| Arun | Laptop | 50,000 |
| Arun | Mouse | 500 |
| Priya | Keyboard | 1,500 |
- **What got dropped?** Karthik (no orders) and Order `104` (no matching customer).

---

### B. `LEFT JOIN` (Most Common) — "Keep Everyone on the Left"
- **Plain-English Rule:** Keep **100% of the rows from the Left table (`Customers`)**, even if they have zero matches in the Right table (`Orders`). If an order is missing, SQL fills the empty boxes with **`NULL`**.
- **Query:**
```sql
SELECT Customers.name, Orders.item, Orders.amount
FROM Customers
LEFT JOIN Orders
  ON Customers.customer_id = Orders.customer_id;
```
- **Result:**
| name | item | amount |
| :--- | :--- | :--- |
| Arun | Laptop | 50,000 |
| Arun | Mouse | 500 |
| Priya | Keyboard | 1,500 |
| **Karthik** | **`NULL`** | **`NULL`** |
- **Data Engineering Trick (Finding Missing Data):** Add `WHERE Orders.order_id IS NULL` at the bottom of a `LEFT JOIN` to instantly find **customers who signed up but never bought anything** (here, Karthik!).

---

### C. `RIGHT JOIN` — "Keep Everything on the Right"
- **Plain-English Rule:** The exact opposite of `LEFT JOIN`—keeps **100% of the rows from the Right table (`Orders`)**, even if there is no matching customer on the Left.
- **Query:**
```sql
SELECT Customers.name, Orders.item, Orders.amount
FROM Customers
RIGHT JOIN Orders
  ON Customers.customer_id = Orders.customer_id;
```
- **Result:**
| name | item | amount |
| :--- | :--- | :--- |
| Arun | Laptop | 50,000 |
| Arun | Mouse | 500 |
| Priya | Keyboard | 1,500 |
| **`NULL`** | Monitor | 12,000 |
- **Pro Tip:** In real jobs, engineers rarely write `RIGHT JOIN`—they simply flip the table order and use `LEFT JOIN` (`FROM Orders LEFT JOIN Customers`) because it is much easier to read from left to right.

---

### D. `FULL OUTER JOIN` — "Keep Everything from Both Sides"
- **Plain-English Rule:** Keeps **100% of rows from BOTH tables**. Whenever a match is missing on either side, SQL fills the gap with **`NULL`**.
- **Query:**
```sql
SELECT Customers.name, Orders.item, Orders.amount
FROM Customers
FULL OUTER JOIN Orders
  ON Customers.customer_id = Orders.customer_id;
```
- **Result:**
| name | item | amount |
| :--- | :--- | :--- |
| Arun | Laptop | 50,000 |
| Arun | Mouse | 500 |
| Priya | Keyboard | 1,500 |
| **Karthik** | **`NULL`** | **`NULL`** |
| **`NULL`** | Monitor | 12,000 |
- **When it's used:** Data auditing—comparing two systems (e.g., Stripe payments vs. your database orders) to spot missing records on **either** side at once.

---

## Quick Memory Cheat Sheet (Joins)
| Join Type | What It Keeps | Missing Values Become |
| :--- | :--- | :--- |
| **`INNER JOIN`** | Only rows that match in **both** tables. | Dropped completely |
| **`LEFT JOIN`** | **All** rows from Left table + matching rows from Right. | `NULL` on the Right |
| **`RIGHT JOIN`** | **All** rows from Right table + matching rows from Left. | `NULL` on the Left |
| **`FULL OUTER JOIN`** | **All** rows from **both** Left and Right tables. | `NULL` on whichever side is missing |
## 4. SQL Aggregations (Crunching Many Rows Into One Summary)

An **Aggregation** squashes hundreds or millions of individual rows down into a clean summary report—like looking at 500 daily shop receipts and asking *"How much total money did we make today?"*

### Sample `Orders` Table Used Below
| order_id | customer_name | item | amount |
| :--- | :--- | :--- | :--- |
| **101** | Arun | Laptop | 50,000 |
| **102** | Arun | Mouse | 500 |
| **103** | Priya | Keyboard | 1,500 |
| **104** | Priya | Monitor | 12,000 |
| **105** | Karthik | Cable | 300 |

---

### A. The 5 Basic Summary Functions (Whole Table)
When you want **one single total** across the entire table:
- **`COUNT(*)`** — Counts how many rows exist (`5` orders). *(Note: `COUNT(column_name)` ignores `NULL` rows, while `COUNT(*)` counts every row).*
- **`SUM(amount)`** — Adds all numbers together (`64,300`).
- **`AVG(amount)`** — Calculates the average (`12,860`).
- **`MAX(amount)`** — Finds the highest value (`50,000`).
- **`MIN(amount)`** — Finds the lowest value (`300`).

```sql
SELECT
  COUNT(*) AS total_orders,
  SUM(amount) AS total_revenue,
  AVG(amount) AS avg_order_value
FROM Orders;
```
- **Result:** `total_orders = 5`, `total_revenue = 64,300`, `avg_order_value = 12,860`.

---

### B. `GROUP BY` — "Sorting Rows into Buckets First"
When you want separate totals **per customer**, **per city**, or **per day**, `GROUP BY` sorts the rows into separate buckets before running `COUNT` or `SUM`:
- **Arun's Bucket:** Laptop (`50,000`) + Mouse (`500`) $\rightarrow$ **2 orders, `50,500` total**
- **Priya's Bucket:** Keyboard (`1,500`) + Monitor (`12,000`) $\rightarrow$ **2 orders, `13,500` total**
- **Karthik's Bucket:** Cable (`300`) $\rightarrow$ **1 order, `300` total**

```sql
SELECT
  customer_name,
  COUNT(*) AS order_count,
  SUM(amount) AS total_spent
FROM Orders
GROUP BY customer_name;
```
- **Result:**
| customer_name | order_count | total_spent |
| :--- | :--- | :--- |
| Arun | 2 | 50,500 |
| Priya | 2 | 13,500 |
| Karthik | 1 | 300 |
- **Golden Rule of `GROUP BY`:** Every column in your `SELECT` line must either be **inside a summary function** (`COUNT`, `SUM`, `AVG`) OR **listed in `GROUP BY`**.

---

### C. `WHERE` vs. `HAVING` — "Filter Before Buckets vs. Filter After Buckets"
Both filter data, but they run at **different times** in the query:

1. **`WHERE` (Runs BEFORE `GROUP BY`):** Throws out individual rows **before** putting anything into buckets.
   ```sql
   -- Only count expensive items over 1,000
   SELECT customer_name, SUM(amount) AS total_spent
   FROM Orders
   WHERE amount > 1000
   GROUP BY customer_name;
   ```
   - **What happens:** Arun's `500` Mouse and Karthik's `300` Cable are thrown away first. Only Laptop (`50,000`), Keyboard (`1,500`), and Monitor (`12,000`) go into buckets.
   - **Result:** Arun = `50,000`, Priya = `13,500` (Karthik is gone completely).

2. **`HAVING` (Runs AFTER `GROUP BY`):** Creates all buckets and adds up the totals **first**, then throws away finished buckets that fail the condition.
   ```sql
   -- Find VIP customers whose TOTAL spending is over 10,000
   SELECT customer_name, SUM(amount) AS total_spent
   FROM Orders
   GROUP BY customer_name
   HAVING SUM(amount) > 10000;
   ```
   - **What happens:** SQL adds up every bucket first (`Arun = 50,500`, `Priya = 13,500`, `Karthik = 300`), and **then** `HAVING` drops Karthik because his total is below `10,000`.
   - **Why `WHERE SUM(amount) > 10000` causes an error:** Because `WHERE` runs *before* `GROUP BY`, `SUM(amount)` hasn't been calculated yet!

---

## Module 5 Quick Memory Table
| Keyword | Plain-English Job | When It Runs |
| :--- | :--- | :--- |
| **`GROUP BY`** | Splits rows into separate buckets (e.g., per customer). | After `WHERE` |
| **`WHERE`** | Filters **individual rows** before buckets are created. | Before `GROUP BY` |
| **`HAVING`** | Filters **finished bucket totals** (`SUM`, `COUNT`) after grouping. | After `GROUP BY` |





