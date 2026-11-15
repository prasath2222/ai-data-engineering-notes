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



## Topic 2: How Tables Connect (Foreign Keys & Relationships)

<details>
<summary><b>1. Why don't we put all app data into one giant table?</b></summary>

Putting everything in one table repeats the same information over and over (e.g., repeating a user's `name`, `email`, and `city` on every single order). Splitting data into separate linked tables saves storage and prevents update bugs when a user changes their details.
</details>

<details>
<summary><b>2. What is a Foreign Key, and what two mistakes does it prevent?</b></summary>

A **Foreign Key (FK)** is a column in a **Child table** (like `orders.user_id`) that points to the **Primary Key** in a **Parent table** (like `users.user_id`). It protects data integrity in two ways:
1. **Blocks Fake Records:** You cannot add an order for `user_id = 999` if user 999 does not exist in `users`.
2. **Blocks Orphan Records:** You cannot delete a user from `users` while they still have linked orders in `orders`.
</details>

<details>
<summary><b>3. What are the 3 types of table relationships?</b></summary>

1. **One-to-Many (Most Common):** One parent row links to many child rows (e.g., one user has many orders).
2. **One-to-One:** One row links to at most one row in another table (e.g., one user has one profile settings row).
3. **Many-to-Many:** Multiple rows on both sides link together (e.g., students and courses), solved by creating a middle **bridge table** (`enrollments`) holding both IDs.
</details>

<details>
<summary><b>4. Example: How do chat_sessions and messages connect in an AI app?</b></summary>

- **Parent Table:** `chat_sessions` (Primary Key: `session_id`)
- **Child Table:** `messages` (Primary Key: `message_id`, Foreign Key: `session_id`)
- **Relationship:** **One-to-Many** — one chat session holds many prompt/reply messages.
</details>
