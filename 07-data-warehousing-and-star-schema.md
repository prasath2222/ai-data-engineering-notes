
# Module 7: Data Warehousing — OLTP vs. OLAP & Star Schema

## 1. Why Companies Need Two Different Databases (OLTP vs. OLAP)

Imagine running a busy supermarket:
- At the **front counter**, cashiers scan barcodes one item at a time (`INSERT` / `UPDATE`) so customers can pay and leave in 5 seconds.
- In the **back office**, the manager wants to scan **3 years of receipts** (`SELECT SUM(...)`) to see which city bought the most laptops every November.

If the manager runs that giant 3-year report on the **exact same database** the cashiers are using, the database gets overloaded, the checkout machines freeze, and live customers cannot pay.

To prevent this, companies separate their data into **two different types of systems**:

### A. OLTP — Online Transaction Processing ("The Shop Cash Register")
- **Plain-English Job:** Runs the live website or mobile app day-to-day (signing up users, booking tickets, taking payments).
- **Popular Tools:** PostgreSQL, MySQL, SQL Server.
- **How It Works:**
  - Handles **thousands of tiny, fast reads/writes per second** on 1–2 rows at a time.
  - Uses **strictly split tables (Normalization / 3NF)** so customer names and addresses never duplicate.
  - Optimized for **speed of writing** and **ACID safety**.

### B. OLAP — Online Analytical Processing ("The Manager's Analytics Room")
- **Plain-English Job:** Stores historical data (often copied every night from OLTP) so analysts and dashboards can ask big business questions.
- **Popular Tools:** Google BigQuery, Snowflake, DuckDB, ClickHouse, Amazon Redshift.
- **How It Works:**
  - Rarely updates single rows—instead, it **reads millions or billions of rows at once** (`SUM`, `AVG`, `GROUP BY`).
  - Merges tables together (denormalized) so reports need fewer `JOIN`s and finish in seconds instead of minutes.

### Quick Comparison: OLTP vs. OLAP
| Feature | OLTP (App Database) | OLAP (Data Warehouse) |
| :--- | :--- | :--- |
| **Main Users** | App users, customers, cashiers | Data Analysts, Data Engineers, BI Dashboards |
| **Main Work** | Tiny, fast `INSERT`, `UPDATE`, `DELETE` | Giant `SELECT` queries (`SUM`, `GROUP BY`) |
| **Data Scope** | Current live data (today's orders) | Years of historical data |
| **Table Style** | Many small, strictly split tables (`3NF`) | Fewer, wider tables (**Star Schema**) |
| **Examples** | PostgreSQL, MySQL | Snowflake, BigQuery, DuckDB, ClickHouse |

---

## 2. Star Schema — "One Central Receipt Table + Small Lookup Tables"

Inside an OLAP Data Warehouse, Data Engineers reshape messy app tables into a **Star Schema**. When you draw it on paper, **one giant table sits in the center**, surrounded by smaller tables pointing into it—forming the shape of a star.

A Star Schema has only **two types of tables**:

### A. Fact Table — "What Happened?" (The Numbers & Measurements)
- **What it stores:** Every business event or transaction (e.g., `Fact_Sales`), containing mostly **IDs and numbers** you can add up (`SUM`, `COUNT`).
- **Shape:** **Very tall** (millions or billions of rows) and **narrow** (mostly numbers).
- **Example (`Fact_Sales`):**
| sale_id | customer_id | product_id | date_id | quantity | revenue |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1001 | 1 | 50 | 20261115 | 2 | 100,000 |
| 1002 | 2 | 51 | 20261115 | 1 | 500 |

### B. Dimension Tables — "Who, What, Where, When?" (The Descriptive Labels)
- **What they store:** The human-readable descriptions used to filter (`WHERE`) and bucket (`GROUP BY`) your numbers.
- **Shape:** **Short** (hundreds or thousands of rows) and **wide** (lots of text columns).
- **Examples:**
  - **`Dim_Customer` (Who?):** `customer_id`, `name`, `city`, `state`, `signup_date`
  - **`Dim_Product` (What?):** `product_id`, `product_name`, `brand`, `category`
  - **`Dim_Date` (When?):** `date_id`, `full_date`, `day_of_week`, `month`, `quarter`, `year`

---

## 3. Why Data Engineers Love Star Schema
In a live OLTP database, getting *"Total Laptop revenue by City"* might require **5 or 6 complicated `JOIN`s** across `Orders -> Order_Items -> Products -> Categories -> Customers -> Addresses`.

In a **Star Schema**, every Dimension connects directly to the central Fact table in **one single hop**:
```sql
SELECT
  c.city,
  p.category,
  SUM(f.revenue) AS total_revenue
FROM Fact_Sales f
JOIN Dim_Customer c ON f.customer_id = c.customer_id
JOIN Dim_Product  p ON f.product_id  = p.product_id
WHERE p.category = 'Laptops'
GROUP BY c.city, p.category;
```
- **Simple to write:** Analysts never get lost in a maze of tables.
- **Blazing fast:** Modern Data Warehouses are specifically engineered to join one giant Fact table with small Dimension tables in milliseconds.
