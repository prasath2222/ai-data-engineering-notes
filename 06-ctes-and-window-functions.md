
# Module 6: CTEs (`WITH`) & Window Functions

## 1. CTE (`WITH`) — "A Temporary Whiteboard Scratchpad"

### Why Do We Need CTEs?
When a SQL query needs two steps (for example: first add up every customer's total spending, then filter only VIP customers), putting a `SELECT` inside another `SELECT` creates messy, hard-to-read spaghetti code.

A **CTE (Common Table Expression)** uses the keyword **`WITH`** to let you write your query in clean, top-to-bottom steps—just like writing temporary numbers on a whiteboard and wiping it clean when you finish.

### Step-by-Step Example
```sql
-- Step 1: Write a temporary scratchpad table called "CustomerTotals"
WITH CustomerTotals AS (
  SELECT
    customer_name,
    SUM(amount) AS total_spent
  FROM Orders
  GROUP BY customer_name
)

-- Step 2: Query that scratchpad table just like a normal table
SELECT
  customer_name,
  total_spent
FROM CustomerTotals
WHERE total_spent > 10000;
```
- **How long does `CustomerTotals` exist?** Only while this single query is running—it disappears automatically right after, without taking up any database storage.
- **Chaining Multiple Steps:** You can create multiple scratchpad tables in one query separated by commas (`WITH Step1 AS (...), Step2 AS (...) SELECT ...`).

---

## 2. Window Functions — "Doing Math Without Squashing Rows"

### The Key Difference from `GROUP BY`
- **`GROUP BY`** **squashes** 10 rows down into 3 summary rows—you lose the individual row details.
- **Window Functions** **keep every single original row visible** and simply add a **new calculated column** next to each row (such as a leaderboard rank or yesterday's sales number).
- **How to spot a Window Function:** It always uses the keyword **`OVER (...)`**.

### Sample `DailySales` Table Used Below
| sale_date | person | amount |
| :--- | :--- | :--- |
| May 1 | Arun | 1,000 |
| May 2 | Arun | 1,500 |
| May 1 | Priya | 1,500 |
| May 2 | Priya | 2,000 |

---

### A. `ROW_NUMBER()` vs. `RANK()` — "Leaderboard Positions"
Suppose we rank all 4 sales from highest to lowest (`ORDER BY amount DESC`). Notice two sales are tied at **`1,500`**:

```sql
SELECT
  person,
  amount,
  ROW_NUMBER() OVER (ORDER BY amount DESC) AS row_num,
  RANK()       OVER (ORDER BY amount DESC) AS rank_num
FROM DailySales;
```

**Result:**
| person | amount | row_num (`ROW_NUMBER`) | rank_num (`RANK`) |
| :--- | :--- | :--- | :--- |
| Priya | 2,000 | **1** | **1** |
| Arun | 1,500 | **2** | **2** |
| Priya | 1,500 | **3** | **2** *(Tie!)* |
| Arun | 1,000 | **4** | **4** *(Skips 3!)* |

- **`ROW_NUMBER()`:** Forces strict `1, 2, 3, 4` numbering with **no ties**—even when two rows have the exact same amount (`1,500`), one gets `2` and the other gets `3`.
  - **Real-World Data Engineering Trick:** Used with a CTE (`WHERE row_num = 1`) to **delete duplicate records** from messy tables!
- **`RANK()`:** Gives tied rows the **exact same rank** (`2` and `2`), and then **skips** the next number (`4`) just like Olympic medals.
- *(Bonus: `DENSE_RANK()` also gives tied rows `2` and `2`, but does **not** skip `3`—it goes `1, 2, 2, 3`.)*

---

### B. `LAG()` & `LEAD()` — "Looking at Previous or Next Rows"
When you want to compare **today's row** with **yesterday's row** for each person separately, you combine **`LAG()`** with **`PARTITION BY`**:
- **`PARTITION BY person`:** Divides the rows into separate mini-windows per person (so Priya's May 1 sale never accidentally compares against Arun's May 2 sale!).
- **`ORDER BY sale_date`:** Sorts each person's rows in calendar order.
- **`LAG(amount)`:** Grabs the `amount` from **1 row above** (yesterday).
- **`LEAD(amount)`:** Grabs the `amount` from **1 row below** (tomorrow).

```sql
SELECT
  person,
  sale_date,
  amount AS today_sale,
  LAG(amount) OVER (PARTITION BY person ORDER BY sale_date) AS yesterday_sale
FROM DailySales;
```

**Result:**
| person | sale_date | today_sale | yesterday_sale (`LAG`) |
| :--- | :--- | :--- | :--- |
| Arun | May 1 | 1,000 | **`NULL`** *(First day — no previous row)* |
| Arun | May 2 | 1,500 | **1,000** *(Arun's May 1 sale)* |
| Priya | May 1 | 1,500 | **`NULL`** *(Window resets for Priya!)* |
| Priya | May 2 | 2,000 | **1,500** *(Priya's May 1 sale)* |

- **Why Data Engineers Love `LAG()`:** You can calculate daily growth in one line: `amount - LAG(amount) OVER (PARTITION BY person ORDER BY sale_date) AS daily_growth`.

---

## Module 6 Quick Memory Table
| Tool / Keyword | Plain-English Meaning | Common Real-World Use |
| :--- | :--- | :--- |
| **`WITH` (CTE)** | Temporary whiteboard table that exists for 1 query. | Breaking complex multi-step queries into readable blocks. |
| **`OVER (...)`** | Marks a Window Function (keeps all rows unsquashed). | Adding ranks, running totals, or previous-row values. |
| **`PARTITION BY`** | Splits rows into separate mini-windows before calculating. | Resetting ranks or `LAG()` per customer/department. |
| **`ROW_NUMBER()`** | Strict `1, 2, 3, 4` numbering (never ties). | Removing duplicate rows (`WHERE row_num = 1`). |
| **`RANK()`** | Tied rows share a rank, next number skips (`1, 2, 2, 4`). | Building scoreboards and top-N leaderboards. |
| **`LAG()` / `LEAD()`** | Peeks at the previous (`LAG`) or next (`LEAD`) row. | Day-over-day or month-over-month growth comparisons. |
