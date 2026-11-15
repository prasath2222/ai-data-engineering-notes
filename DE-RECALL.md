## Topic 0: What Is Data Engineering?

<details>
<summary><b>1. What does a Data Engineer actually do?</b></summary>

A Data Engineer builds automated pipelines (the "plumbing") that collect raw, messy data from apps and files, clean it up, and store it reliably so Data Analysts and AI models can use it.
</details>

## Topic 1: Data Roles Compared

<details>
<summary><b>1. How does a Data Engineer differ from a Data Analyst and an AI Engineer?</b></summary>

- **Data Engineer:** Builds and maintains the data pipelines and storage systems (delivers clean water).
- **Data Analyst:** Queries clean data to create dashboards, charts, and business reports (uses the water for daily reports).
- **AI / ML Engineer:** Uses clean data to train machine learning models, build embeddings, and power AI applications (uses the water to run AI engines).
</details>

## Topic 2: Core Pipeline Steps (ETL)

<details>
<summary><b>1. What are the 3 core steps of every data pipeline (ETL)?</b></summary>

1. **Extract:** Pull raw data from sources (databases, APIs, CSVs, PDFs).
2. **Transform:** Clean, filter, deduplicate, and standardize the raw data.
3. **Load:** Save the clean data into a target storage system (Data Warehouse or Vector Database).
</details>



## Topic 3: The 3 Types of Data

<details>
<summary><b>1. What is Structured Data?</b></summary>

Data that fits neatly into fixed **rows and columns** (like an Excel spreadsheet or a SQL database table), where every record follows the exact same format.
- *Examples:* Bank transactions, user signups, product prices (`user_id | name | amount`).
</details>

<details>
<summary><b>2. What is Semi-Structured Data?</b></summary>

Data that does **not** fit into rigid rows and columns because different records can have different fields, so it uses `"key": "value"` pairs (most commonly **JSON**).
- *Examples:* API responses, app event logs, order receipts (`{"user": "Arun", "items": ["dosa", "coffee"]}`).
</details>

<details>
<summary><b>3. What is Unstructured Data?</b></summary>

Raw human content with **no rows, no columns, and no keys**.
- *Examples:* Company PDF manuals, scanned invoices, customer support chats, images, and audio files.
- *(In AI Engineering, most of your work is converting messy Unstructured and Semi-Structured data into clean Structured tables and search indexes.)*
</details>

## Topic 4: Live App Database vs. Analytics Database (OLTP vs. OLAP)

<details>
<summary><b>1. Why do companies use two separate databases instead of just one?</b></summary>

1. **Live App Database (OLTP):** Runs the real-time app for users (likes, orders, messages). It needs to stay super fast for small, instant actions.
2. **Analytics Database / Data Warehouse (OLAP):** A **separate copy** of the data where analysts and AI models run heavy math on millions of rows without slowing down or crashing the live app.
</details>

<details>
<summary><b>2. What is the Data Engineer's role between these two databases?</b></summary>

The Data Engineer builds the automated pipeline that **copies** new data from the **Live App Database**, cleans it up, and loads it into the **Analytics Database**.
</details>

## Topic 5: How Data Moves — Batch vs. Streaming

<details>
<summary><b>1. What is Batch Processing?</b></summary>

Collecting data over time and moving it all together in **scheduled chunks** (e.g., every hour or once every night at 2:00 AM).
- **Used for:** 90% of data work — daily reports, updating AI search documents, and building training datasets. Simple, cheap, and reliable.
</details>

<details>
<summary><b>2. What is Streaming (Real-Time) Processing?</b></summary>

Moving each piece of data **the exact second it happens**, continuously 24/7, one event at a time.
- **Used for:** Urgent situations where waiting even 1 minute is too late — like blocking credit card fraud immediately or live delivery map tracking.
</details>

## Topic 6: ETL vs. ELT (Where Does the Cleaning Happen?)

<details>
<summary><b>1. What is the difference between ETL and ELT?</b></summary>

- **ETL (Extract -> Transform -> Load):** Pull raw data, clean it **first**, and save only the clean result into your storage/warehouse.
- **ELT (Extract -> Load -> Transform):** Pull raw data, save the untouched raw copy into your storage/warehouse **first**, and clean it inside there later.
</details>

<details>
<summary><b>2. Why do modern AI and Data Engineering teams prefer ELT?</b></summary>

Because **ELT keeps the original raw data untouched**. If an AI engineer or analyst needs a different field or wants to re-clean last month's data in a new way later, the original raw files are still safely stored and ready to reuse.
</details>

## Topic 7: Data Lake vs. Data Warehouse (Where Data Lives)

<details>
<summary><b>1. What is a Data Lake?</b></summary>

A cheap, massive storage system that holds **raw, untouched files** in any format—CSVs, JSON logs, PDFs, images, and audio.
- **Analogy:** A giant storage room where boxes are stacked straight off the truck before unpacking.
- **Examples:** Amazon S3, Google Cloud Storage (GCS).
- **Why AI teams use it:** Cheaply stores millions of raw PDFs, text files, and logs before processing them for AI models.
</details>

<details>
<summary><b>2. What is a Data Warehouse?</b></summary>

A fast analytical system that holds **only cleaned, structured data** neatly organized into SQL tables (rows and columns).
- **Analogy:** Supermarket shelves where every item is unpacked, labeled, and arranged for fast lookup.
- **Examples:** Google BigQuery, Snowflake.
- **Why Analysts use it:** Dashboards and business reports run super fast because the data is already cleaned and structured.
</details>

## Topic 8: What Happens Inside "Cleaning" (Data Quality & Validation)

<details>
<summary><b>1. What are the 4 main problems Data Engineers fix when cleaning raw data?</b></summary>

1. **Duplicate Rows (Deduplication):** Network glitches often save the exact same event twice. Keeping only one unique record per ID prevents double-counting revenue or users.
2. **Missing Values (NULLs):** Blank fields can crash calculations or AI pipelines. Fix by filling safe defaults (like `'Unknown'`) or dropping rows missing critical IDs.
3. **Wrong Formats & Data Types:** Converting messy dates (`"15-11-2026"`, `"Nov 15, 2026"`) and text numbers (`"₹500"`) into one strict standard format (`2026-11-15` and pure number `500`).
4. **Impossible / Corrupt Values (Data Validation):** Blocking broken values (like `age = -5` or negative prices) using strict validation rules (**Data Contracts**) before bad rows enter the database.
</details>

<details>
<summary><b>2. What is a Data Contract?</b></summary>

A strict set of rules placed at the entrance of a pipeline that checks every incoming record's data type and valid range (e.g., `age > 0`, `email` is not blank) so corrupted data is rejected automatically instead of breaking downstream dashboards or AI models.
</details>

## Topic 9: Orchestration (Running Pipelines Automatically)

<details>
<summary><b>1. What is Orchestration in Data Engineering?</b></summary>

**Orchestration** is the automated system (like **Prefect**, **Apache Airflow**, or **Cron**) that schedules, runs, and monitors data pipelines automatically without human clicks.
</details>

<details>
<summary><b>2. What are the 4 main jobs of an Orchestrator?</b></summary>

1. **Scheduling:** Runs pipelines automatically on a timer (e.g., every hour or at `2:00 AM` every night).
2. **Dependencies:** Enforces the exact order of steps (Step 2 waits until Step 1 succeeds; Step 3 is blocked if Step 2 fails).
3. **Auto-Retries:** Automatically waits and retries failed steps (e.g., if a network blip happens) before marking the job as failed.
4. **Alerts:** Sends an immediate notification (Slack/Email) if a pipeline breaks so the engineer can fix it.
</details>

## Topic 10: Safe Re-Runs & Loading Only New Data (Idempotency & Incremental Loads)

<details>
<summary><b>1. What is the difference between a Full Load and an Incremental Load?</b></summary>

- **Full Load:** Copying the **entire history** of a table from scratch every time the pipeline runs (slow, expensive, wasteful for large data).
- **Incremental Load:** Saving a bookmark (**Watermark**, like `last_updated_at`) and copying **only the new or updated rows** created since the last successful run (fast, cheap, standard in production).
</details>

<details>
<summary><b>2. What does Idempotency mean in a data pipeline?</b></summary>

**Idempotency** means you can run the exact same pipeline **once, twice, or 10 times** for the same day/batch and get the **exact same clean result** with zero duplicate rows.
- **How it is achieved:** Instead of blindly inserting rows, pipelines use **UPSERT** (`UPDATE` if the ID already exists, `INSERT` if it is new) or delete-and-replace that specific date's partition before loading.
</details>

## Topic 11: How Data Engineering Powers AI & LLMs (RAG Pipelines)

<details>
<summary><b>1. What is a RAG (Retrieval-Augmented Generation) Pipeline?</b></summary>

A specialized data pipeline that takes a company's private documents (PDFs, manuals, support chats), cleans and indexes them, and lets an AI model search and retrieve the exact relevant facts in milliseconds to answer user questions accurately.
</details>

<details>
<summary><b>2. What are the 3 core steps in an AI Data Pipeline?</b></summary>

1. **Extract & Clean Text (Parsing):** Pulling raw files from a Data Lake and stripping out messy headers, page numbers, and broken formatting so only clean text remains.
2. **Chunking:** Slicing long documents into small, bite-sized paragraphs (300–500 words with slight overlap) so the AI only reads the exact relevant sections instead of processing a massive file every time.
3. **Embeddings & Vector Database:** Converting each text chunk into a list of numbers (**Embedding**) that captures its *meaning*, and storing them in a **Vector Database** (like **pgvector**, **Qdrant**, or **Pinecone**) for instant semantic search.
</details>

## Topic 12: Putting It All Together (End-to-End Architecture)

<details>
<summary><b>1. How do all 11 Data Engineering building blocks connect in a real company?</b></summary>

1. **Live App Sources (OLTP & Data Shapes):** User orders live in an **OLTP App Database** (Structured), click events arrive as **JSON** (Semi-Structured), and manuals/docs arrive as **PDFs** (Unstructured).
2. **Scheduled Pickup (Orchestration, Batch & Incremental):** An **Orchestrator** (Prefect / Airflow) wakes up on a **Batch** schedule (e.g., `2:00 AM` nightly) and uses a **Watermark (Incremental Load)** to pull *only* new records since the last run.
3. **Raw Storage (ELT & Data Lake):** Following **ELT**, raw files land untouched in a **Data Lake** (S3 / GCS) first so original data is never lost.
4. **Cleaning & Safety (Quality, Contracts & Idempotency):** Pipelines enforce **Data Contracts**, remove duplicates, handle `NULL`s, standardize formats, and use **UPSERT** so re-runs are **Idempotent** (zero duplicates).
5. **Serving Layer (Data Warehouse & Vector DB):**
   - Clean structured tables load into a **Data Warehouse (OLAP)** for fast analytics and ML without slowing down the live app.
   - Clean text documents are **Chunked**, converted into **Embeddings**, and stored in a **Vector Database** to power **RAG** AI chatbots.
</details>

## Topic 13: Data Modeling — Fact vs. Dimension Tables (Star Schema)

<details>
<summary><b>1. What is the difference between a Fact Table and a Dimension Table?</b></summary>

- **Fact Table (The Events & Numbers):** Stores measurable events or actions that happen over time (things you count or sum), containing mostly IDs and numbers (`order_id`, `user_id`, `quantity`, `total_amount`). *Example:* `fact_orders`.
- **Dimension Table (The Who, What, Where):** Stores descriptive background details about the people, products, or places involved in those events. *Examples:* `dim_users`, `dim_products`.
</details>

<details>
<summary><b>2. Why do Data Engineers use Star Schema instead of one giant table?</b></summary>

Splitting events (**Fact**) from descriptive profiles (**Dimension**) avoids repeating the same text thousands of times, saves storage, and makes updates clean—if a user changes their city, you update **1 row** in `dim_users` instead of thousands of rows in `fact_orders`.

</details>

## Topic 14: Core SQL Patterns (`GROUP BY`, Window Functions, `JOIN`)

<details>
<summary><b>1. Base Example Table (Small Tea Shop Sales)</b></summary>

Imagine starting with these **3 individual sales rows**:

| Order | Customer ID | Item | Price |
| :--- | :--- | :--- | :--- |
| 1 | C1 | Tea | ₹20 |
| 2 | C2 | Coffee | ₹30 |
| 3 | C1 | Tea | ₹20 |
</details>

<details>
<summary><b>2. GROUP BY (Squashes Rows into a Short Summary)</b></summary>

**`GROUP BY`** collapses many individual rows down into summary totals. The original individual rows disappear.

- **Example (`GROUP BY Item`):** Squashes the 3 orders down into **2 summary rows**:

| Item | Total Money |
| :--- | :--- |
| **Tea** | ₹40 |
| **Coffee** | ₹30 |
</details>

<details>
<summary><b>3. Window Function (Keeps All Rows, Adds a New Column)</b></summary>

A **Window Function (`OVER (...)`)** calculates totals, rankings, or row numbers **without deleting or squashing any rows**—it simply adds a new calculated column on the right.

- **Example (Running Total):** Keeps all 3 orders visible and tracks the cash box total after each sale:

| Order | Item | Price | **Running Total (New Column)** |
| :--- | :--- | :--- | :--- |
| 1 | Tea | ₹20 | **₹20** |
| 2 | Coffee | ₹30 | **₹50** *(20 + 30)* |
| 3 | Tea | ₹20 | **₹70** *(50 + 20)* |

- **Data Engineering Use:** Used with `ROW_NUMBER()` to number duplicate rows (`1, 2, 3...`) and keep only `#1`.
</details>

<details>
<summary><b>4. JOIN (Glues Info from a Second Table)</b></summary>

**`JOIN`** matches an ID column between two separate tables and attaches missing details side-by-side.

- **Table A (Orders):** `Order 1 | C1 | Tea`
- **Table B (Customer Names):** `C1 | Arun`
- **Result after `JOIN` on Customer ID:**

| Order | Customer ID | **Name (from Table B)** | Item |
| :--- | :--- | :--- | :--- |
| 1 | C1 | **Arun** | Tea |

</details>

## Topic 15: File Formats for Big Data (CSV vs. JSON vs. Parquet)

<details>
<summary><b>1. How do CSV and JSON store data on disk?</b></summary>

**Row-by-Row Storage:** They store one complete row after another (`[1, Tea, 20] -> [2, Coffee, 30]`).
- **The Problem:** If a table has 50 columns and you only need to add up 1 column (`Price`), the computer is still forced to read all 50 columns—making queries slow and expensive.
</details>

<details>
<summary><b>2. What is Apache Parquet and why do Data & AI Engineers prefer it?</b></summary>

**Column-by-Column Storage:** Parquet groups each column together separately on disk (e.g., all `Orders` together, all `Prices` together).
1. **10x–100x Faster Queries:** Reads only the exact columns your query asks for and completely skips the rest.
2. **Up to 80% Smaller Size:** Compresses identical data types sitting next to each other tightly, cutting cloud storage bills.
3. **Preserves Data Types:** Unlike CSV (all plain text), Parquet remembers whether each column is a number, date, or string.

</details>

## Topic 16: Partitioning & Indexing (How Huge Tables Stay Fast)

<details>
<summary><b>1. What is Partitioning?</b></summary>

**Partitioning** splits a massive table into separate physical folders based on a column you filter by often (usually `date` or `country`), like `/orders/date=2026-11-15/`.
- **Analogy:** Sorting 5 years of paper receipts into separate folders labeled by date instead of dumping them into one giant box.
- **Why it is fast (Partition Pruning):** When you query a specific date, the database completely skips all other date folders and reads **only that single folder**.
- **Where it is used:** Data Lakes and Data Warehouses (**OLAP**).
</details>

<details>
<summary><b>2. What is Indexing?</b></summary>

**Indexing** creates a sorted lookup map pointing directly to where each row lives on disk.
- **Analogy:** The alphabetical index at the back of a textbook that tells you *"Backpropagation — Page 412"* so you don't flip through every page from page 1.
- **Why it is fast:** Lets the database jump straight to **1 exact row** (like `user_id = 84920`) in 1 millisecond instead of scanning millions of rows.
- **Where it is used:** Live App Databases (**OLTP**).
</details>

<details>
<summary><b>3. Partitioning vs. Indexing at a Glance</b></summary>

| Feature | **Partitioning** | **Indexing** |
| :--- | :--- | :--- |
| **Best Analogy** | Separate folders by date | Index page at the back of a book |
| **Where It Lives** | Data Lakes & Warehouses (OLAP) | Live App Databases (OLTP) |
| **Best For** | Skipping millions of irrelevant rows (e.g., filtering by `date`) | Finding **1 exact row** instantly (e.g., `user_id = 84920`) |
</details>

## Topic 17: Open Table Formats (Apache Iceberg & Delta Lake — The "Lakehouse")

<details>
<summary><b>1. Why are plain Parquet files in a Data Lake not enough on their own?</b></summary>

Plain Parquet files are locked once written (like printed receipts), which creates **3 problems**:
1. **Cannot easily edit or delete 1 row:** Fixing a single wrong price requires manually rewriting huge files.
2. **Crashes leave half-finished data:** If a pipeline crashes after saving 1 out of 2 files, analysts see broken, incomplete numbers.
3. **No Undo button:** If a buggy script overwrites or deletes a file, the old version is lost.
</details>

<details>
<summary><b>2. How do Apache Iceberg and Delta Lake solve this?</b></summary>

They keep your cheap Parquet files in the Data Lake and add a smart **"Master Register Book" (Transaction Log)** on top—working just like **Google Docs Version History** to turn a Data Lake into a **Data Lakehouse**:

| Version | What Happened | Which File the Register Shows to Users |
| :--- | :--- | :--- |
| **Version 1** | Morning sales saved | `day1.parquet` *(contains a ₹300 typo)* |
| **Version 2** | You fix Coffee price to ₹30 | `day1_fixed.parquet` *(creates a clean file and updates the Register)* |
| **Version 3** | Tonight's script crashes halfway | **Still shows `day1_fixed.parquet`!** *(hides half-written files until 100% done)* |
</details>

<details>
<summary><b>3. What are the 3 Lakehouse superpowers?</b></summary>

1. **Crash Safety (ACID Transactions):** Half-written files stay completely invisible until a pipeline finishes 100% cleanly.
2. **Easy `UPDATE` & `DELETE`:** Lets you edit or delete single rows cleanly—the Register Book creates the updated file behind the scenes.
3. **Time Travel (Built-in Undo):** Lets you query past versions or instantly roll back (`Version 2 -> Version 1`) if a bad pipeline corrupts today's data.
</details>

## Topic 18: Data Transformation & `dbt` (Building Clean Tables in Stages)

<details>
<summary><b>1. What is dbt (Data Build Tool)?</b></summary>

**`dbt`** is the industry-standard tool for the **"T" (Transform)** in ELT. It lets Data Engineers organize messy SQL transformations inside a Data Warehouse into clean, tested, version-controlled stages instead of running hundreds of chaotic manual scripts.
</details>

<details>
<summary><b>2. What are the 3 dbt transformation layers?</b></summary>

| Layer | Kitchen Analogy | What Happens to the Data | Example Table Name |
| :--- | :--- | :--- | :--- |
| **1. Staging** | Washing & chopping vegetables | Renames ugly columns (`c_id` → `customer_id`), trims spaces, and fixes basic data types (`"₹20"` → `20`). No joins or heavy math yet. | `stg_orders` |
| **2. Intermediate** | Cooking the main dishes | Joins clean staging tables together and applies core business rules (e.g., calculating discounts). | `int_orders_with_names` |
| **3. Marts** | Plating the final meal | Builds the final **Fact & Dimension tables** (`fact_sales`, `dim_customers`) ready for dashboards and AI models. | `fact_sales` |
</details>

<details>
<summary><b>3. What are the 3 superpowers dbt gives Data Engineers?</b></summary>

1. **Automatic Run Order (Lineage):** Automatically detects dependencies between tables and runs them in the exact right order (`Staging` → `Intermediate` → `Marts`).
2. **Built-in Data Tests:** Runs automated checks (like `unique` and `not_null`) before publishing tables so broken numbers never reach dashboards.
3. **Git Version Control & Docs:** Stores every SQL model in GitHub for peer review and auto-generates a visual lineage map showing where every column came from.
</details>

## Topic 19: Change Data Capture — CDC (Streaming Live Database Edits)

<details>
<summary><b>1. What is the problem with checking a live database using timestamps?</b></summary>

Checking a live app database with `WHERE updated_at > last_run` has **2 blind spots**:
1. **Deleted rows vanish silently:** If a customer cancels an order and the row is erased from the live app table, a later timestamp query never sees it—leaving "ghost" cancelled orders inside your Data Warehouse.
2. **Slows down the live app:** Repeatedly scanning the live database puts extra load on real customers.
</details>

<details>
<summary><b>2. How does Change Data Capture (CDC) work?</b></summary>

Instead of interrupting the live database tables, **CDC** (using tools like **Debezium**) quietly reads the database's internal append-only log file (**Write-Ahead Log / Binlog**)—just like watching a **cash machine paper roll** where nothing can ever be erased:

| What Happens at the Tea Shop Counter | Line Printed on the Paper Roll | What CDC Does to the Data Warehouse |
| :--- | :--- | :--- |
| **10:00 AM:** Customer orders Tea | `NEW: Order #1, Tea, ₹20` | Adds `Order #1` |
| **10:02 AM:** Price fixed to ₹25 | `CHANGED: Order #1, ₹20 -> ₹25` | Updates price to `₹25` |
| **10:05 AM:** Customer cancels | `DELETED: Order #1` | Removes `Order #1` |
</details>

<details>
<summary><b>3. What are the 3 main benefits of CDC?</b></summary>

1. **Captures `DELETE`s Accurately:** Because every `DELETE` is recorded on the log roll, the Data Warehouse removes cancelled rows and stays 100% synced.
2. **Zero Extra Load on the Live App:** Reading the log file never slows down live customer traffic.
3. **Real-Time Updates:** Streams every `INSERT`, `UPDATE`, and `DELETE` into Kafka, Data Lakehouses, or Vector Databases within seconds.
</details>


## Topic 20: Slowly Changing Dimensions — SCD Type 1 & Type 2 (Tracking Profile Changes)

<details>
<summary><b>1. What is a Slowly Changing Dimension (SCD)?</b></summary>

Customer and product profiles in **Dimension Tables** (`dim_customers`) change occasionally over time—such as a customer moving cities or changing their subscription tier. **SCD Type 1** and **SCD Type 2** are the two standard ways Data Engineers handle those updates.
</details>

<details>
<summary><b>2. What is SCD Type 1 (Overwrite)?</b></summary>

**SCD Type 1** simply erases the old value and writes the new value in the same row—keeping **no history**.

| Customer ID | Name | City |
| :--- | :--- | :--- |
| C1 | Arun | **Coimbatore** *(Old city "Chennai" is erased)* |

- **Best For:** Fixing typos and spelling mistakes (e.g., `"Arnn"` → `"Arun"`) where nobody cares about the old wrong value.
</details>

<details>
<summary><b>3. What is SCD Type 2 (Add a New Row — Keep Full History)?</b></summary>

**SCD Type 2** never overwrites old data. It closes the old row with an `End Date` (`Is Current = No`) and inserts a **brand-new row** for the updated profile (`Is Current = Yes`):

| Row ID | Customer ID | Name | City | Start Date | End Date | Is Current? |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | C1 | Arun | **Chennai** | Jan 1 | May 31 | ❌ No |
| 2 | C1 | Arun | **Coimbatore** | Jun 1 | *(Open)* | ✅ Yes |

- **Best For:** Tracking real-world changes (city moves, plan upgrades) so past sales stay accurately linked to where the customer lived at that exact time.
</details>

