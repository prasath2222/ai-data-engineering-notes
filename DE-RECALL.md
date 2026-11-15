## Topic 1: Why Databases Exist (Files vs. Databases)

<details>
<summary><b>1. Why do apps use Databases instead of CSV or Excel files?</b></summary>

1. **Speed:** Searching millions of rows in a CSV freezes the system; databases use indexes to find data in milliseconds.
2. **Safety (Concurrency):** If two users update data at the exact same millisecond, plain files get corrupted. Databases handle simultaneous users safely.
3. **Strict Rules (Schema):** A database enforces data types (e.g., blocking text inside a `price` column) so bad data cannot enter.
</details>

<details>
<summary><b>2. What are Tables, Rows, Columns, and Primary Keys?</b></summary>

- **Table:** A structured grid storing one type of data (e.g., `users` or `orders`).
- **Column:** A specific field or category (e.g., `user_id`, `name`, `city`).
- **Row:** One complete single record (e.g., `101 | Arun | Chennai`).
- **Primary Key:** A special column (like `user_id`) that must be **100% unique** and **never blank (`NOT NULL`)** so every row can be identified without confusion.
</details>

<details>
<summary><b>3. Why is `user_id` a good Primary Key while `city` is a bad one?</b></summary>

- **`user_id` works** because every user gets a unique ID that never repeats and is never empty.
- **`city` fails** because thousands of users live in the same city (duplicates) and some users leave their city blank.
</details>
