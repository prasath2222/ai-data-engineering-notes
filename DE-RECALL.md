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
