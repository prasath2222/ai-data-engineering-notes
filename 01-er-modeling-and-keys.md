# Module 1: ER Modeling, Cardinality & Keys

## Part 1: ER Model, Tables, Rows, Columns & Primary Keys

### 1. What is an ER Model?
- **ER Model (Entity-Relationship Model):** A rough blueprint you plan *before* building a database—deciding what separate lists (tables) you need and how they connect.
- **Entity = Table:** One separate list for one type of thing (e.g., `Customers`, `Products`, `Orders`).
- **Attribute = Column:** A specific detail you record about that thing (e.g., `name`, `phone`, `price`).
- **Record / Tuple = Row:** One single entry in the table (e.g., one customer).

**Example: `Customers` Table**

| customer_id | name | phone |
| :--- | :--- | :--- |
| 1 | Arun | 9876543210 |
| 2 | Priya | 9123456789 |
| 3 | Arun | 9988776655 |

---

### 2. Primary Key (Unique ID Card)
- **What it is:** One column chosen to uniquely identify every single row in a table (like an Aadhaar number or Roll Number).
- **Why we need it:** Two customers can have the exact same name (`Arun`), and phone numbers can change. A system-generated ID (`1, 2, 3...`) guarantees the database never confuses two rows.
- **2 Strict Rules of a Primary Key:**
  1. **Unique:** No two rows can ever have the same Primary Key.
  2. **Not Null:** It can never be left blank/empty.


  ## Part 2: Foreign Keys & One-to-Many (1:N) Relationships

### 1. What is a Foreign Key?
- **Foreign Key (FK):** A column in a child table that points to the **Primary Key** of a parent table to link the two tables together.
- **Why we use it:** Instead of repeating a customer's name and phone number on every order, we just store their `customer_id`.
- **Referential Integrity:** The safety rule enforced by a Foreign Key—the database blocks you from adding an order for a `customer_id` that does not exist in the `Customers` table (preventing fake/orphan records).

**Example: Linking `Customers` and `Orders`**

`Customers` (Parent Table)
| customer_id (PK) | name | phone |
| :--- | :--- | :--- |
| 1 | Arun | 9876543210 |
| 2 | Priya | 9123456789 |

`Orders` (Child Table)
| order_id (PK) | customer_id (FK) | item | amount |
| :--- | :--- | :--- | :--- |
| 101 | 1 | Masala Tea | 20 |
| 102 | 2 | Ginger Tea | 25 |
| 103 | 1 | Biscuit | 10 |

---

### 2. Cardinality & One-to-Many (1:N)
- **Cardinality:** Defines how many rows in one table can connect to rows in another table.
- **One-to-Many (`1:N`):** The most common relationship in databases. **One** parent row (`Customer`) can connect to **many** child rows (`Orders`), while each child row belongs to **only one** parent.
- **Golden Rule:** Always place the **Foreign Key** on the **"Many"** side (`Orders` gets `customer_id`).

  ## Part 3: One-to-One (1:1) & Many-to-Many (M:N) Relationships

### 1. One-to-One (1:1)
- **What it is:** 1 row in Table A connects to **at most 1** row in Table B (and vice versa).
- **Example:** `Customers` $\leftrightarrow$ `Customer_Profiles` (storing optional delivery address or loyalty card info separately from basic login/contact info).
- **How to build it:** Put the **Foreign Key** in the second table and mark it **`UNIQUE`** so no customer gets two profiles.

---

### 2. Many-to-Many (M:N) & Bridge Tables
- **What it is:** Many rows in Table A connect to many rows in Table B.
- **Example:** One `Order` can include **many** `Products` (Tea + Biscuit), and one `Product` (Masala Tea) appears in **many** `Orders`.
- **Golden Rule:** Databases cannot link `M:N` directly between two tables. You must create a **3rd Bridge (Junction) Table** in the middle (`Order_Items`) holding Foreign Keys from both parent tables.

**Example: Splitting `M:N` with `Order_Items`**

`Orders`
| order_id (PK) | customer_id (FK) | order_date |
| :--- | :--- | :--- |
| 101 | 1 | 2026-11-15 |

`Products`
| product_id (PK) | name | price |
| :--- | :--- | :--- |
| 10 | Masala Tea | 20 |
| 20 | Biscuit | 10 |

`Order_Items` (Bridge Table)
| order_id (FK) | product_id (FK) | quantity |
| :--- | :--- | :--- |
| 101 | 10 | 2 |
| 101 | 20 | 1 |

## Part 4: Other Key Types, Constraints & Metadata

### 1. Key Types Summary
- **Candidate Key:** Any column (or set of columns) that uniquely identifies a row and *could* be chosen as the Primary Key (`customer_id`, `email`, `phone`).
- **Alternate Key:** Any Candidate Key that was *not* chosen as the Primary Key (`email`, `phone`).
- **Surrogate Key:** A system-generated, meaningless ID (`1, 2, 3...` or `UUID`) used as the Primary Key instead of real-world values like phone numbers that might change.
- **Composite Key:** A Primary Key made by combining **2 or more columns together** (e.g., `(order_id, product_id)` in the `Order_Items` bridge table, where neither column is unique alone, but their pair is always unique).

---

### 2. Constraints (Guardrails Against Bad Data)
- **`NOT NULL`:** Column cannot be left empty.
- **`UNIQUE`:** No two rows can have the same value in this column.
- **`CHECK`:** Enforces a custom condition (e.g., `price > 0` or `quantity > 0`).
- **`DEFAULT`:** Automatically inserts a preset value if none is provided (e.g., current timestamp).

---

### 3. Metadata & Data Independence
- **Metadata (Data Dictionary):** "Data about data"—the database's internal catalog (`INFORMATION_SCHEMA`) storing table names, column types, and constraints. Automated data pipelines inspect this to detect schema changes.
- **Data Independence:**
  - *Logical:* Adding new columns or views doesn't break existing queries.
  - *Physical:* Adding indexes or upgrading storage hardware speeds up queries without changing any SQL code.
    
