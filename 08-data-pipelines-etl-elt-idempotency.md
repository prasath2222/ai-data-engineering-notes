# Module 8: Data Pipelines — ETL vs. ELT, Batch vs. Streaming & Idempotency

## 1. What is a Data Pipeline?
A **Data Pipeline** is the automated plumbing that collects messy raw data from where it is born (your live app database, payment APIs, mobile logs), cleans it up, and delivers it into your Data Warehouse (Star Schema tables) so analysts and dashboards can use it every morning.

Every pipeline performs three core jobs:
- **E — Extract:** Copy raw data out of source systems (PostgreSQL, Stripe API, CSV/JSON logs).
- **T — Transform:** Clean and reshape messy rows (fix date formats, remove duplicates, join tables into Fact and Dimension tables).
- **L — Load:** Save the data into your Data Warehouse (BigQuery, Snowflake, DuckDB).

---

## 2. ETL vs. ELT ("Clean Before Storing vs. Store Raw First, Clean Inside")

The difference between **ETL** and **ELT** is simply **when** and **where** you clean the data:

### A. ETL (Extract $\rightarrow$ Transform $\rightarrow$ Load) — "Wash Vegetables Before Putting Them in the Fridge"
- **How it works:**
  1. Pull raw data out of your app database.
  2. Clean, filter, and transform it on a **separate Python or Spark server** *before* loading.
  3. Load **only the finished, clean tables** into the Data Warehouse.
- **When it is still used:**
  - When strict privacy rules require stripping sensitive data (like passwords, credit card numbers, or Aadhaar/SSN IDs) *before* data ever enters the warehouse.
  - Older on-premise warehouses where storage and computing power were very expensive.

### B. ELT (Extract $\rightarrow$ Load $\rightarrow$ Transform) — "Throw Everything in the Fridge First, Cook Inside" (Modern Standard)
- **How it works:**
  1. Pull raw data out of your app database and **immediately load 100% of it as-is** into a "Raw Zone" inside your cloud Data Warehouse.
  2. Use **SQL (often managed by a tool called `dbt`)** right inside the Data Warehouse to clean and build your Star Schema tables.
- **Why modern Data Engineers prefer ELT:**
  - **Raw data is never lost:** If the business changes a formula 6 months later, you still have the original untouched raw data sitting in the warehouse—you just edit your SQL query and re-run it instead of re-downloading from the live app.
  - **Massive speed:** Cloud warehouses (BigQuery, Snowflake) have huge built-in computing engines designed to crunch SQL transformations fast.
  - **Faster setup:** You start saving raw data on Day 1 and build cleaning rules over time.

### Quick Comparison: ETL vs. ELT
| Feature | ETL (Traditional) | ELT (Modern Cloud Standard) |
| :--- | :--- | :--- |
| **Order of Steps** | Extract $\rightarrow$ **Transform** $\rightarrow$ Load | Extract $\rightarrow$ **Load** $\rightarrow$ Transform |
| **Where Cleaning Happens** | Separate external server (Python / Spark) | **Inside the Data Warehouse** (using SQL / `dbt`) |
| **Raw Data Saved?** | Usually discarded after cleaning | **Saved permanently** in a raw staging area |
| **Best For** | Stripping sensitive PII before storage | Fast cloud analytics & flexible rule changes |

---

## 3. Batch vs. Streaming ("The Scheduled Bus vs. The Non-Stop Conveyor Belt")

How often does your pipeline run?

### A. Batch Processing — "The Scheduled Bus"
- **How it works:** Waits until a set time (every 1 hour, or every night at 2:00 AM), collects all new orders from the past period, and moves them together in **one big batch**.
- **Real-World Examples:** Daily sales dashboards, monthly payroll reports, weekly inventory summaries.
- **Usage:** Covers roughly **85% of real-world Data Engineering pipelines** because it is simpler, cheaper, and easier to debug.

### B. Streaming Processing — "The Non-Stop Conveyor Belt"
- **How it works:** Processes every single event **one by one within milliseconds** as soon as the user clicks or swipes.
- **Popular Tools:** Apache Kafka, Apache Flink, AWS Kinesis.
- **Real-World Examples:** Credit card fraud detection (blocking a stolen card in 0.2 seconds), live Uber driver maps, live cricket/sports scoreboards.

---

## 4. Idempotency — "Pressing the Elevator Button 10 Times"

**Idempotency** is the #1 reliability rule in Data Engineering.

### Plain-English Meaning
A pipeline is **Idempotent** if running it **1 time, 2 times, or 10 times** for the exact same day produces the **exact same final result**—with **zero duplicate rows**.

### Why It Matters in Real Life
Pipelines crash all the time (internet drops, server restarts, API timeouts). When your pipeline crashes halfway through loading yesterday's sales (`2026-11-14`) and you hit **"Retry"**:

#### 1. Non-Idempotent Pipeline (Broken — Creates Duplicate Money!)
```sql
-- BAD: Blindly appends rows every time you run it
INSERT INTO Fact_Sales
SELECT * FROM Raw_Sales WHERE sale_date = '2026-11-14';
```
- **What goes wrong:** Yesterday had ₹5,00,000 in sales. The script inserted ₹3,00,000 before crashing. When you re-run it, it inserts all ₹5,00,000 again—now your dashboard shows **₹8,00,000** for yesterday!

#### 2. Idempotent Pipeline (Safe — Delete/Overwrite Partition First)
```sql
-- GOOD: Wipe yesterday's slice first, then insert fresh rows
DELETE FROM Fact_Sales WHERE sale_date = '2026-11-14';

INSERT INTO Fact_Sales
SELECT * FROM Raw_Sales WHERE sale_date = '2026-11-14';
```
*(Or use `MERGE` / `UPSERT` to update existing IDs instead of inserting duplicates.)*
- **What happens:** Even if the script crashes 5 times and you re-run it 5 times, yesterday's slice is cleanly wiped and rewritten—so yesterday's total is always **₹5,00,000**.

---

## Module 8 Quick Memory Table
| Concept | Simple Analogy | Key Takeaway |
| :--- | :--- | :--- |
| **ETL** | Wash vegetables before putting in the fridge | Clean data on a separate server *before* loading into warehouse. |
| **ELT** | Put raw groceries in fridge first, cook inside | Load raw data into warehouse *first*, transform with SQL (`dbt`). |
| **Batch** | Scheduled school bus (hourly / nightly) | Moves chunks of data on a schedule; powers ~85% of analytics. |
| **Streaming** | Non-stop factory conveyor belt (milliseconds) | Real-time event processing (Kafka) for fraud alerts & live maps. |
| **Idempotency** | Pressing an elevator button 10 times | Re-running a failed pipeline produces the exact same data with **zero duplicates**. |
## 5. Pipeline Orchestration (Airflow & DAGs) — "The Kitchen Head Chef"

In a real company, you don't just run 1 isolated script—you have **50+ scripts** that depend on each other in a strict sequence. For example, you cannot build the **Daily Sales Dashboard** (`Step 3`) until **Clean Orders** (`Step 2`) finishes, and `Step 2` cannot start until **Download Raw Data** (`Step 1`) succeeds.

### A. What is an Orchestrator?
- **Popular Tools:** Apache Airflow (industry standard), Prefect, Dagster.
- **Plain-English Analogy:** An Orchestrator is the **Head Chef** in a busy restaurant kitchen. It does not chop the vegetables or wash the dishes itself—instead, it watches the clock, tells each worker when to start, stops the next step if an earlier step fails, automatically retries temporary glitches, and sends a Slack/email alert if something breaks at 3:00 AM.

### B. What is a DAG (Directed Acyclic Graph)? — "The Recipe Flowchart"
Inside Airflow, you define your pipeline's order of tasks as a **DAG**:
- **Directed:** Arrows point strictly forward (`Task A -> Task B -> Task C`).
- **Acyclic (No Loops):** Arrows can **never circle backward** (`Task C` cannot trigger `Task A`, otherwise the pipeline would run in an infinite loop forever!).
- **Graph:** Individual tasks (boxes) connected by dependencies (arrows).

```text
[Download Customers] ──┐
                       ├──> [Join & Build Fact_Sales] ──> [Update Dashboard]
[Download Orders] ─────┘
```
*(In this DAG, `Download Customers` and `Download Orders` run in parallel at the same time, and `Join & Build Fact_Sales` automatically waits until **both** succeed before starting.)*

---

## 6. Full Load vs. Incremental Load — "Re-Printing the Whole Book vs. Adding Today's Page"

Suppose your live `Orders` table has **100 million rows** from the last 5 years, and **10,000 new orders** arrived today. How should your pipeline copy data into the Data Warehouse tonight?

### A. Full Load ("Re-Writing the Entire Notebook Every Night")
- **How it works:** Completely wipes the destination table (`TRUNCATE TABLE`) and copies all **100,010,000 rows** from scratch every single night.
- **Pros:** Super simple to code; automatically captures deleted rows.
- **Cons:** Way too slow and expensive for huge tables.
- **When to use:** Small Dimension/lookup tables (under ~100,000 rows, like `Dim_Product` or `Dim_Store`) where re-copying the entire table takes 2 seconds.

### B. Incremental Load ("Only Copying Today's New Lines")
- **How it works:** Remembers the timestamp of the last successful run (called a **Watermark**, e.g., `2026-11-14 23:59:59`) and copies **only the 10,000 rows added or updated since yesterday**:
  ```sql
  SELECT * FROM Orders
  WHERE updated_at > '2026-11-14 23:59:59';
  ```
- **Pros:** Blazing fast and cheap—processes 10,000 rows instead of 100 million.
- **Cons:** Slightly more complex to manage retries and updates (`MERGE` / `UPSERT`).
- **When to use:** Large Fact tables (`Fact_Sales`, app clickstreams, payment logs) with millions or billions of rows.

### Quick Comparison: Full Load vs. Incremental Load
| Feature | Full Load | Incremental Load |
| :--- | :--- | :--- |
| **Plain-English Analogy** | Throwing out the notebook and rewriting every page | Keeping old pages and writing only today's new lines |
| **Rows Moved Each Night** | 100% of the table (old + new) | Only new/updated rows since the last run (**Watermark**) |
| **Speed & Cloud Cost** | Slow & expensive on big tables | **Fast & cheap** on big tables |
| **Best For** | Small lookup tables (`Dim_Product`, `Dim_City`) | Huge event/Fact tables (`Fact_Sales`, click logs) |

---

## 7. `dbt` (Data Build Tool) — "Software Engineering for SQL Transformations"

In modern **ELT** pipelines, raw data is dumped straight into the Data Warehouse first, and then transformed using SQL. When a company grows to **300+ SQL queries**, managing raw `.sql` scripts manually becomes chaotic.

**`dbt`** is the industry-standard tool that organizes and runs the **"T" (Transform)** step inside your Data Warehouse:
1. **Write Only `SELECT` Queries:** You write clean `SELECT` statements; `dbt` automatically wraps them into `CREATE TABLE`, `CREATE VIEW`, or incremental `MERGE` commands behind the scenes.
2. **Automatic Dependency Graph (`ref()`):** When you write `FROM {{ ref('clean_orders') }}`, `dbt` automatically knows `clean_orders` must finish *before* running your current model—building a lineage DAG automatically.
3. **Built-in Data Quality Tests:** With simple YAML rules (`unique`, `not_null`, `accepted_values`), `dbt` automatically tests your finished tables every run and stops bad or duplicate data from reaching executive dashboards.

---

## Module 8 Complete Quick Memory Table
| Tool / Concept | Simple Analogy | Key Takeaway |
| :--- | :--- | :--- |
| **Orchestrator (Airflow)** | Kitchen Head Chef | Schedules tasks, enforces order, retries failures, sends alerts. |
| **DAG** | One-way recipe flowchart | Tasks connected by forward-only arrows with **no loops**. |
| **Full Load** | Rewriting the whole notebook | Replaces 100% of a small table (`TRUNCATE` + `INSERT`). |
| **Incremental Load** | Writing only today's new page | Copies only new/changed rows using a `updated_at` watermark. |
| **`dbt`** | Software engineering for SQL | Manages SQL transformations (`ref()`), lineage, and data quality tests in ELT. |
