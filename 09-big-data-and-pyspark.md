# Module 9: Big Data Processing & PySpark

## 1. Why Do We Need Apache Spark / PySpark?

When your dataset is small (under 1–2 GB), Python's **Pandas** library works great on a single laptop because the entire file fits inside your laptop's RAM.

### The Problem: Single-Machine Crash (`OutOfMemoryError`)
- Suppose your laptop has **16 GB of RAM**, and your pipeline needs to clean **200 GB of messy JSON logs or Parquet files** sitting in cloud storage (AWS S3 / Google Cloud Storage).
- Pandas tries to load all **200 GB into your 16 GB RAM at once** $\rightarrow$ your laptop runs out of memory, freezes, and crashes (`OutOfMemoryError`).

### The Solution: Distributed Computing ("20 Workers Counting 200 Boxes at Once")
Instead of buying one impossibly expensive supercomputer with 1,000 GB of RAM, **Apache Spark** (and its Python API, **PySpark**) links **multiple normal computers (a Cluster)** together so they act like one single engine:
1. Spark chops your 200 GB file into **200 smaller chunks (partitions)** of ~1 GB each.
2. It hands 10 chunks to each of the 20 computers in the cluster.
3. All 20 computers process their chunks **in parallel at the exact same time** inside their own RAM, finishing in **30 seconds** instead of crashing.

### Quick Comparison: Pandas vs. PySpark
| Feature | Pandas (Single Computer) | PySpark (Cluster of Computers) |
| :--- | :--- | :--- |
| **Where it runs** | Inside **1 machine's RAM** | Spread across **many machines (Nodes)** in parallel |
| **Best Data Size** | Small to medium (under ~2 GB) | Huge Big Data (10 GB to Terabytes / Petabytes) |
| **Execution Style** | **Eager** (runs every line immediately) | **Lazy** (plans everything first, runs only on an Action) |
| **Crash Risk on Huge Files** | High (`OutOfMemoryError`) | Low (splits data into chunks & spills safely to disk if needed) |

---

## 2. Spark Architecture: Driver vs. Executors ("The Site Manager & The Workers")

Whenever you run a PySpark job on a cluster, two roles divide the work:

### A. The Driver ("The Construction Site Manager")
- **What it is:** The main computer running your Python script (`main.py`).
- **What it does:**
  - Reads your PySpark code and builds the step-by-step recipe plan (DAG).
  - Splits the big job into hundreds of small tasks.
  - Assigns tasks to the Executors and monitors if any worker crashes so it can re-assign that chunk.
- *(Note: The Driver does **not** load or crunch the heavy 200 GB dataset itself.)*

### B. The Executors ("The Construction Workers")
- **What they are:** Separate worker computers in the cluster with high RAM and CPU cores.
- **What they do:**
  - Each Executor grabs its assigned data chunks (partitions) directly from cloud storage.
  - Filters, joins, and crunches the numbers in parallel inside its own RAM.
  - Sends only the final small summary (or writes finished Parquet files to S3) back when finished.

---

## 3. Lazy Evaluation — "Writing the Whole Shopping List Before Walking Into the Store"

**Lazy Evaluation** is PySpark's smartest performance trick—and one of the most common Data Engineering interview questions.

In PySpark, every command falls into one of **two categories**:

### 1. Transformations ("Writing Steps on a Recipe Card" — Lazy)
- **Examples:** `.filter()`, `.select()`, `.withColumn()`, `.groupBy()`, `.join()`.
- **What Spark does:** **Nothing yet!** When you write `df.filter(df.city == "Chennai")`, Spark does *not* read the 200 GB file yet. Instead, it simply writes that step down on a logical recipe plan (called a **DAG**).

### 2. Actions ("Going to the Cash Register" — Executes Everything Now)
- **Examples:** `.show()`, `.count()`, `.collect()`, `.write.parquet()`.
- **What Spark does:** The moment Spark sees an Action, its built-in optimizer (**Catalyst Optimizer**) reads your **entire shopping list from start to finish**, rearranges it into the fastest possible route, and executes everything in one single pass.

### Why Lazy Evaluation Saves Huge Time & Money
```python
# Step 1 & 2: Transformations (Lazy — Spark just takes notes, takes 0.001 seconds)
df = spark.read.parquet("s3://company-bucket/sales/")
chennai_df = df.filter(df.city == "Chennai").select("order_id", "amount")

# Step 3: Action (Triggers the cluster to actually read and process the files!)
chennai_df.show(10)
```
- **Without Lazy Evaluation:** Spark would load all 200 GB and all 50 columns into RAM first, and only filter for `"Chennai"` and 2 columns later.
- **With Lazy Evaluation:** Because Spark waited until `.show(10)`, it sees your `.filter()` and `.select()` ahead of time—so it **skips 48 unneeded columns** and **skips non-Chennai Parquet partitions** right while reading from S3!

---

## 4. Shuffles — "The Most Expensive Operation in Spark"

Not all transformations are equal. Spark groups them into two speeds:

1. **Narrow Transformations (Fast — No Data Movement Between Workers):**
   - **Examples:** `.filter()`, `.select()`, `.withColumn()`.
   - Each worker computer can filter its own 10 chunks independently without talking to any other worker.
2. **Wide Transformations / Shuffles (Slow — Workers Must Trade Data Over the Network):**
   - **Examples:** `.groupBy()`, `.join()`, `.orderBy()`, `.distinct()`.
   - To calculate `SUM(amount)` per `city`, Worker #1 and Worker #2 must send all `"Chennai"` rows across the network to the same worker. Moving gigabytes of data across the network between machines is called a **Shuffle**—and minimizing shuffles is the #1 way Data Engineers tune slow Spark jobs.

---

## Module 9 Quick Memory Table
| Concept | Simple Analogy | Key Takeaway |
| :--- | :--- | :--- |
| **Cluster / Distributed Computing** | 20 workers counting 200 boxes in parallel | Splits huge files across many machines so RAM never overflows. |
| **Driver** | Construction site manager | Plans tasks, assigns work to Executors, tracks progress. |
| **Executors** | Construction workers | Read data partitions and crunch numbers in parallel RAM. |
| **Transformation** | Writing on a shopping list (`.filter`, `.select`) | **Lazy**—records steps in a plan without touching data yet. |
| **Action** | Walking to the cash register (`.show`, `.write`) | Triggers the optimizer and runs the whole plan at once. |
| **Shuffle** | Workers mailing boxes to each other across the room | Network data transfer during `.groupBy()` or `.join()`; expensive! |
## 5. Broadcast Join — "Giving Every Worker a Photocopy of the Small Lookup Book"

Suppose your pipeline needs to join two tables:
- **`Fact_Sales` (Huge):** **200 GB** of orders spread across 20 Executor computers.
- **`Dim_Product` (Tiny):** **5 MB** lookup table with 2,000 product names.

### A. Normal Join (Slow — Shuffles 200 GB Across the Network)
- By default, Spark shuffles **both** tables across the network so matching `product_id` rows land on the same worker machine.
- Moving 200 GB of sales rows over the network wastes minutes and slows down your cluster.

### B. Broadcast Join (Blazing Fast — Zero Shuffle!)
- Because `Dim_Product` is tiny (5 MB), you tell the Driver to **mail a full photocopy of `Dim_Product` directly into every Executor's RAM**:
```python
from pyspark.sql.functions import broadcast

# Sends the small table to every worker's RAM so the 200 GB table never moves!
joined_df = fact_sales.join(broadcast(dim_product), on="product_id", how="inner")
```
- **Why it is 10x faster:** Every worker already has the full product list in its own pocket, so **zero rows of the 200 GB `Fact_Sales` table ever move across the network**.
- **Rule of Thumb:** Use `broadcast()` whenever joining a giant Fact table with a small Dimension/lookup table (typically under **10 MB to 100 MB**).

---

## 6. Data Skew — "One Cashier Gets 10,000 Customers While 19 Cashiers Stand Idle"

### What is Data Skew?
Suppose you run `.groupBy("city")` across 20 worker computers:
- **Worker 1** gets `"Chennai"` $\rightarrow$ **80 million rows** (takes **45 minutes** to finish!).
- **Workers 2–20** get small towns $\rightarrow$ **5,000 rows each** (finish in **2 seconds** and sit completely idle!).

Your Spark progress bar gets **stuck at 99% complete** (`199/200 tasks finished`) waiting for that single overloaded worker—and sometimes Worker 1 runs out of RAM and crashes (`OutOfMemoryError`). That uneven pile-up of rows on one partition is called **Data Skew** (most often caused by one dominant key or millions of `NULL` values).

### How Data Engineers Fix Data Skew
1. **Salting ("Splitting One Giant Line Into 10 Shorter Lines"):**
   - Attach a random number (`0` to `9`) to the overloaded key (`Chennai_0`, `Chennai_1`, ..., `Chennai_9`) so those 80 million rows split evenly across **10 workers instead of 1**.
   - Sum each salted bucket in parallel first, then remove the salt and add the 10 subtotals together.
2. **Adaptive Query Execution (AQE):**
   - Modern Spark (Spark 3.0+) includes **AQE** (`spark.sql.adaptive.enabled = true`), which automatically detects oversized partitions at runtime and splits them across idle workers for you.

---

## 7. Complete Real-World PySpark ETL Script

Here is what a clean, production-style daily PySpark pipeline looks like from start to finish:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, sum as _sum, broadcast

# 1. Start the Spark Engine
spark = SparkSession.builder.appName("DailyCategorySalesETL").getOrCreate()

# 2. EXTRACT: Read yesterday's raw Parquet sales & small product lookup table
sales_df = spark.read.parquet("s3://company-bucket/raw/sales/sale_date=2026-11-15/")
products_df = spark.read.parquet("s3://company-bucket/raw/products/")

# 3. TRANSFORM: Filter bad rows, Broadcast Join product info, and aggregate revenue
clean_summary_df = (
    sales_df
    .filter(col("amount") > 0)                                    # Drop cancelled or ₹0 orders
    .join(broadcast(products_df), on="product_id", how="inner")   # Fast Broadcast Join (no shuffle!)
    .groupBy("category", "city")
    .agg(_sum("amount").alias("total_revenue"))
)

# 4. LOAD: Write finished Parquet output (overwrite mode guarantees Idempotency!)
(
    clean_summary_df
    .write
    .mode("overwrite")
    .parquet("s3://company-bucket/warehouse/daily_category_sales/sale_date=2026-11-15/")
)
```

---

## Module 9 Complete Quick Memory Table
| Concept | Simple Analogy | Key Takeaway |
| :--- | :--- | :--- |
| **Driver & Executors** | Construction manager & workers | Driver plans tasks; Executors read partitions and crunch data in parallel. |
| **Lazy Evaluation** | Writing the whole shopping list first | Transformations build a plan; Actions (`.show`, `.write`) trigger execution. |
| **Shuffle** | Workers mailing boxes across the network | Happens during `.groupBy()` and normal `.join()`; expensive! |
| **Broadcast Join** | Photocopying a small book for every worker | Copies a tiny lookup table to all Executors so the huge table never shuffles. |
| **Data Skew** | 1 cashier with 10,000 people, 19 idle | Uneven key distribution stalls Spark at 99%; fixed with **Salting** or **AQE**. |
