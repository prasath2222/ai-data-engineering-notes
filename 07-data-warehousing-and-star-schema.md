
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
## 4. Row Storage vs. Columnar Storage ("Reading a Full Receipt vs. Reading One Column")

Suppose your `Sales` table has **50 columns** (`sale_id`, `customer_name`, `address`, `phone`, ..., `revenue`) and **100 million rows**, and you run:
```sql
SELECT SUM(revenue) FROM Sales;
```
Notice you only need **1 column (`revenue`)** out of 50. How the database saves those rows on the hard drive decides whether this query takes **2 minutes** or **1 second**:

### A. Row-Oriented Storage (Used by OLTP: PostgreSQL, MySQL, CSV)
- **How it saves data on disk:** Writes one complete row after another:
  `[101, Arun, Chennai, ..., 50000]` $\rightarrow$ `[102, Priya, Coimbatore, ..., 1500]`
- **Why it is great for apps (OLTP):** When Arun opens his profile page, the database grabs his entire row (`name + city + phone + email`) in **one single read**.
- **Why it is slow for analytics (OLAP):** To calculate `SUM(revenue)`, the database must read **all 50 columns** of all 100 million rows off the hard drive—wasting 98% of its effort reading addresses and phone numbers it doesn't even use.

### B. Columnar Storage (Used by OLAP: Parquet, BigQuery, Snowflake, DuckDB)
- **How it saves data on disk:** Groups **each column together** in its own separate block:
  - `sale_id` block: `[101, 102, 103, ...]`
  - `city` block: `[Chennai, Coimbatore, Madurai, ...]`
  - `revenue` block: `[50000, 1500, 300, ...]`
- **Why it is 100x faster for analytics:**
  1. **Skips 49 columns completely:** To run `SELECT SUM(revenue)`, the warehouse reads **only the `revenue` block** and never touches `name`, `address`, or `phone`.
  2. **Extreme compression:** Because every value inside one column block has the exact same data type (for example, `city` repeats `"Chennai"` 2 million times), columnar engines compress file sizes down by **70% to 90%**.

---

## 5. Apache Parquet — "The King of Data Engineering File Formats"

When Data Engineers save large datasets in cloud storage (AWS S3, Google Cloud Storage, Azure Blob), they almost never use `.csv` or `.json`—they convert the data into **`.parquet`** files:

| Feature | CSV / JSON | Apache Parquet (`.parquet`) |
| :--- | :--- | :--- |
| **Storage Layout** | Row-by-row (plain text) | **Columnar** (binary) |
| **File Size** | Huge (e.g., 10 GB) | **5x to 10x smaller** (e.g., 1.2 GB) via compression |
| **Query Speed** | Slow (must read every character) | **Blazing fast** (reads only needed columns) |
| **Schema / Data Types** | Forgets types (`"123"` looks like text) | Stores exact types (`INT`, `DATE`, `FLOAT`) inside the file |

---

## 6. Partitioning — "Sorting Receipts Into Separate Daily Folders"

Even with Parquet files, imagine you have **5 years of sales (5 billion rows)**, and your dashboard only asks for **yesterday's sales**:
```sql
SELECT SUM(revenue) FROM Sales WHERE sale_date = '2026-11-14';
```

**Partitioning** splits one giant table into **separate physical folders** based on a column you filter by every day (most commonly `sale_date` or `country`):
```text
/sales_data/
  ├── sale_date=2026-11-12/part-01.parquet
  ├── sale_date=2026-11-13/part-01.parquet
  └── sale_date=2026-11-14/part-01.parquet
```
- **Partition Pruning (Skipping Folders):** When you include `WHERE sale_date = '2026-11-14'`, the query engine completely ignores the other 1,824 daily folders and opens **only that single folder**—cutting query time and cloud data-scan costs by up to **99%**.
- **Rule of Thumb:** Partition by a low-to-medium cardinality column used in almost every `WHERE` filter (like `order_date` by day/month). **Never** partition by a unique ID like `user_id` (that would create millions of tiny folders and crash performance).

---

## Module 7 Quick Memory Table
| Concept | Simple Analogy | Key Takeaway |
| :--- | :--- | :--- |
| **OLTP** | Shop cash register (PostgreSQL, MySQL) | Fast single-row writes for live apps. |
| **OLAP** | Manager's analytics office (BigQuery, Snowflake) | Fast multi-million-row reads for reports. |
| **Fact Table** | The receipt numbers (`revenue`, `quantity`, IDs) | Tall, narrow table holding events and measurements. |
| **Dimension Table** | Descriptive labels (`customer`, `product`, `date`) | Short, wide lookup tables joined in 1 hop. |
| **Columnar / Parquet** | Storing each column in its own compressed block | Reads only requested columns; 10x smaller than CSV. |
| **Partitioning** | Putting each day's files into its own folder | Skips irrelevant folders (**Partition Pruning**). |
