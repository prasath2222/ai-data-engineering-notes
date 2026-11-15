# Module 4: Indexes & Query Performance

## 1. What is an Index? (The Big Notebook + A-to-Z Shortcut Book Analogy)

Imagine you run a shop and write down every customer in a **500-page notebook** in the random order they walk in:
- Line 1: Arun
- Line 2: Priya
- Line 3: Karthik
- *(...thousands of random names later...)*
- Line 8,420: **Rahul**

### Without an Index: Full Table Scan ("Reading Every Single Line")
- When Rahul calls and asks for his bill, the database has no idea where he is because the rows are stored in random order.
- It starts at **Line 1**, checks it, moves to **Line 2**, **Line 3**, and reads every single row one by one until it hits **Line 8,420** (or checks all **10,000,000 rows** if the name isn't there).
- This is called a **Full Table Scan**—and on big tables, it takes several seconds and slows down your entire app.

### With an Index: Index Scan ("Using an A-to-Z Shortcut Book")
When you create an index (`CREATE INDEX idx_name ON Customers(name);`), the database builds a **hidden, sorted shortcut book** separate from your main table:
- It copies **only the indexed column (`name`) in strict A-to-Z sorted order**, plus the exact **row address (line number)** where that customer lives in the main table:
  - **A** — Arun $\rightarrow$ *Line 1*
  - **K** — Karthik $\rightarrow$ *Line 3*
  - **P** — Priya $\rightarrow$ *Line 2*
  - **R** — **Rahul $\rightarrow$ *Line 8,420***
- Now, when Rahul calls:
  1. The database jumps straight to the **"R"** section in the sorted shortcut book (takes 3–4 tiny jumps instead of reading 10 million rows).
  2. It reads **`Rahul -> Line 8,420`**.
  3. It jumps straight to **Line 8,420** in the main table and grabs his full details in **under 1 millisecond**.

---

## 2. The Golden Trade-Off (Why We Don't Index Every Column)
- **Searching (`SELECT`) becomes 1,000x faster** because the database uses the sorted shortcut book instead of reading every row.
- **Adding or Editing (`INSERT`, `UPDATE`, `DELETE`) becomes slightly slower** and uses extra hard drive space—because every time a new customer walks in, the database must write them into the **main table** AND update every **shortcut book (index)** attached to that table.
- **Simple Rule:**
  - **DO index** columns you frequently search (`WHERE`), join (`ON`), or sort (`ORDER BY`)—like `phone`, `email`, or `customer_id`.
  - **DO NOT index** columns you rarely search (like `notes` or `address`) or tables that get thousands of writes per second with almost no searches.
  - *(Bonus: Databases **automatically** create an index for every **Primary Key** and **`UNIQUE`** column—you don't have to create those yourself.)*

    ## 3. B-Tree Index vs. Hash Index (Two Ways to Organize the Shortcut Book)

### A. B-Tree Index — "The Sorted A-to-Z Dictionary" (Default)
Almost every index you create in PostgreSQL or MySQL is a **B-Tree Index** by default. It keeps your shortcut book in **strict sorted order** (`1, 2, 3...` or `A, B, C...`).
- **What it is great for:**
  - **Exact searches:** `WHERE price = 50` (jumps straight to 50).
  - **Range searches:** `WHERE price > 50` or `WHERE order_date BETWEEN '2026-01-01' AND '2026-01-31'` (jumps to the start of the range, then reads straight down because everything is already sorted).
  - **Sorting:** `ORDER BY price` (already sorted, so zero extra sorting work).

### B. Hash Index — "The Coat-Check Locker Ticket" (Exact Match Only)
A **Hash Index** does **not** sort anything. Instead, it passes your value through a math formula (a hash function) that turns it into a direct locker number:
- `"arun@gmail.com"` $\rightarrow$ **Locker #42**
- `"priya@gmail.com"` $\rightarrow$ **Locker #918**
- **What it is great for:** **ONLY exact `=` matches** (`WHERE email = 'arun@gmail.com'`). It jumps straight to Locker #42 in 1 single step.
- **What it FAILS at:** **Ranges (`>`, `<`, `BETWEEN`) and sorting (`ORDER BY`).**
  - Because `"arun1@gmail.com"` might sit in Locker #42 while `"arun2@gmail.com"` sits in Locker #918, values are scattered in random lockers—so the database cannot scan a range from A to Z.
- **Rule of Thumb:** Stick to **B-Tree** 99% of the time. Only use a Hash Index when a huge text column (like a long URL or session token) is **only ever** searched with `=`.

### Quick Comparison: B-Tree vs. Hash
| Feature | B-Tree Index (Default) | Hash Index |
| :--- | :--- | :--- |
| **How it organizes data** | Strict sorted order (`A -> Z`, `1 -> 100`) | Random locker numbers via math formula |
| **Exact match (`=`)** | Very Fast | Slightly Faster |
| **Range queries (`>`, `<`, `BETWEEN`)** | **Very Fast** | **Impossible** (Reads every row instead) |
| **Sorting (`ORDER BY`)** | **Very Fast** | **Impossible** |

---

## 4. Composite Index — "A Phonebook Sorted by 2 Columns"

When you frequently search using **two columns together**, you can create one **Composite Index** covering both columns:
```sql
CREATE INDEX idx_city_name ON Customers(city, name);
```

### How It Looks Inside
Think of a printed phonebook sorted by **City first (1st column)**, and **Name second (2nd column)**:
- **Chennai** — Arun
- **Chennai** — Karthik
- **Chennai** — Priya
- **Coimbatore** — Arun
- **Coimbatore** — Meera

### The Golden "Left-to-Right" Rule
Because the shortcut book is sorted starting from the **leftmost column (`city`)**, look at which queries use the index and which fail:
1. **`WHERE city = 'Chennai' AND name = 'Priya'` (Uses 1st + 2nd column):** **WORKS!** Jumps straight to Chennai, then straight to Priya.
2. **`WHERE city = 'Chennai'` (Uses ONLY 1st column):** **WORKS!** All Chennai rows are grouped together in one place.
3. **`WHERE name = 'Arun'` (Uses ONLY 2nd column — skips `city`):** **FAILS (Full Table Scan)!**
   - Because `"Arun"` is scattered inside Chennai, inside Coimbatore, and inside Madurai, the database cannot jump straight to `"Arun"` and must read every single row.
- **Rule of Thumb:** Always put the column you filter by most often on the **left (1st position)** when creating a Composite Index.
  
## 5. `EXPLAIN` — Checking If Your Index Actually Worked

When a query feels slow, put the word **`EXPLAIN`** right in front of your `SELECT` query:
```sql
EXPLAIN SELECT * FROM Customers WHERE name = 'Rahul';
```
Instead of returning the customer's data, the database gives you a **report card** showing how it searched the table:
- **`Seq Scan` (Sequential Scan — Bad):** The database **ignored your shortcut book** and read every single row from Line 1 to the end.
- **`Index Scan` (Good):** The database **used your A-to-Z shortcut book** and jumped straight to Rahul in milliseconds.

---

## 6. Common Mistakes That Silently Break Your Index

Even when an index exists, the database will throw away the A-to-Z shortcut book and read every single row if you write your `WHERE` condition the wrong way:

### Mistake 1: Putting `%` at the Front of a `LIKE` Search
Think of a normal **A-to-Z dictionary**:
- **Works (`WHERE name LIKE 'Rah%'` — starts with `"Rah"`):**
  - Every name starting with `"Rah"` is grouped together in the **"R"** section. The database flips straight to `"R"` and uses the index.
- **Breaks (`WHERE name LIKE '%hul'` — ends with `"hul"`):**
  - A name ending in `"hul"` could start with **A** (*Anshul*), **M** (*Mehul*), or **R** (*Rahul*). Because they are scattered across every letter of the alphabet, the database cannot use the A-to-Z tabs and must read **every single row**.

### Mistake 2: Doing Math or Running Functions on the Indexed Column
Your shortcut book stores the **original column values** (e.g., monthly `salary` = `50000`), not calculated numbers:
- **Breaks (`WHERE salary * 12 > 600000`):** The shortcut book does not contain `salary * 12`, so the database ignores the index and calculates `salary * 12` for every single row one by one.
- **Works (`WHERE salary > 50000`):** Keep the column clean and alone on the left side, and put all math on the right side so the database can jump straight to `50000` in the shortcut book.

---

## Module 4 Quick Memory Table
| Concept | Simple Analogy | Key Takeaway |
| :--- | :--- | :--- |
| **Full Table Scan (`Seq Scan`)** | Reading a 500-page notebook line by line | Slow on big tables; checks every row. |
| **Index Scan** | Using an A-to-Z shortcut book | Fast; jumps straight to the row address. |
| **B-Tree Index** | Sorted A-to-Z dictionary (Default) | Works for both exact (`=`) and ranges (`>`, `<`, `BETWEEN`). |
| **Hash Index** | Coat-check locker ticket | Works **only** for exact (`=`); fails on ranges and sorting. |
| **Composite Index `(city, name)`** | Phonebook sorted by City first, Name second | Must use the **leftmost** column (`city`) first to work. |
| **Index Trade-Off** | Updating two books instead of one | Speeds up `SELECT`, slightly slows down `INSERT`/`UPDATE`/`DELETE`. |
