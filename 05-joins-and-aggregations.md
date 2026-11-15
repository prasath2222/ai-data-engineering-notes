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
