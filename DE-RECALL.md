## Topic 0: What Is Data Engineering?

<details>
<summary><b>1. What does a Data Engineer actually do?</b></summary>

A **Data Engineer** builds automated programs (called **data pipelines**) that collect messy, scattered raw data from apps, APIs, and files, clean it up (removing duplicates, fixing missing values, standardizing dates), and store it in a fast, organized database so dashboards and AI models can use it reliably.
</details>

<details>
<summary><b>2. How does a Data Engineer differ from a Data Analyst and an AI Engineer?</b></summary>

Think of a house's water system:
- **Data Engineer (The Plumber):** Builds the pipes that pull dirty raw data in, clean it, and store clean data in the tank (database).
- **Data Analyst:** Uses that clean data to build charts and answer business questions (e.g., *"How much revenue did we make yesterday?"*).
- **AI / ML Engineer:** Uses that clean data to train models and build AI apps (e.g., *"Build a chatbot that answers questions from company PDFs"*).
</details>

<details>
<summary><b>3. What are the 3 core steps of every data pipeline (ETL)?</b></summary>

1. **Extract:** Pull raw data from apps, APIs, databases, or files.
2. **Transform:** Clean the data (remove duplicates, fix missing values, convert formats).
3. **Load:** Save the clean data into a fast database or storage system so others can query it.
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

