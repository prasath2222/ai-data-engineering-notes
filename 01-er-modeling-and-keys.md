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
    
# Module 1: ER Modeling, Keys, Relationships & Constraints

## 1. The Blueprint (ER Model & Building Blocks)
- **ER Model (Entity-Relationship Model):** The blueprint you plan *before* creating a database—deciding what separate tables you need and how they link together.
- **Why Separate Tables? (Single Source of Truth):** Instead of dumping everything into one giant Excel sheet (which causes typos, wasted storage, and update nightmares), each fact lives in **one single place**. Update a price once in `Products`, and every linked order automatically reflects it.
- **Core Terms:**
  - **Entity = Table:** One separate list for one type of thing (`Customers`, `Products`, `Orders`).
  - **Attribute = Column:** One detail about that thing (`name`, `phone`, `price`).
  - **Record / Tuple = Row:** One single entry (one customer or one order).

---

## 2. Keys (How Tables Identify & Link Rows)

### Primary Key & Surrogate Key
- **Primary Key (PK):** One column chosen to uniquely identify every row in a table (like an Aadhaar number or Roll Number).
  - **2 Strict Rules:** Must be **Unique** (no duplicates) and **Not Null** (never blank).
- **Surrogate Key:** A system-generated ID number (`1, 2, 3...` or `UUID`) with no real-world meaning.
  - **Why not use `phone` or `email` as Primary Key?** People change phone numbers/emails, and walk-in customers might leave them blank. A system ID (`customer_id`) **never changes**.

### Foreign Key (Linking Tables)
- **Foreign Key (FK):** A Primary Key borrowed from another table and placed in a connecting table to link rows together.
- **Referential Integrity:** The safety rule enforced by a Foreign Key—the database blocks you from saving an order with a `customer_id` or `product_id` that does not exist.

### Other Key Names (For Interviews)
- **Candidate Key:** Any column that *could* qualify to be the Primary Key (`customer_id`, `phone`, `email`).
- **Alternate Key:** The unique Candidate Keys you *didn't* pick as the Primary Key (`phone`, `email`).
- **Composite Key:** When **2 columns team up together** to act as one Primary Key (e.g., `(order_id, product_id)` inside an order items table).

---

## 3. Relationships (Cardinality)

**Cardinality** defines how many rows in Table A connect to rows in Table B:

### A. One-to-One (`1:1`)
- **Rule:** 1 row in Table A links to **at most 1** row in Table B, and vice versa.
- **Example:** 1 `Branch` has 1 `Manager`, and 1 `Manager` can only run 1 `Branch`.
- **How to build it:** Put the Foreign Key (`manager_id`) in `Branches` and add a **`UNIQUE`** lock so the same manager cannot be entered twice.

### B. One-to-Many (`1:N`)
- **Rule:** 1 parent row links to **many** child rows, while each child row belongs to **only 1** parent.
- **Example:** 1 `Customer` places **many** `Orders`; each `Order` belongs to **only 1** `Customer`.
- **Golden Rule:** Always put the **Foreign Key** on the **"Many"** side (`Orders` gets `customer_id`).

### C. Many-to-Many (`M:N`) — The 3-Table Bridge Pattern
- **Rule:** Many rows in Table A connect to many rows in Table B.
- **Example:** Many `Customers` buy many `Products` (or 1 `Order` contains many `Products`, and 1 `Product` appears in many `Orders`).
- **How to build it:** Keep both parent tables pure, and create a **3rd Connecting (Bridge) Table** in the middle holding **both Primary Keys** as Foreign Keys:

**1. `Customers` (People Only)**
| customer_id (PK) | name | phone |
| :--- | :--- | :--- |
| 1 | Arun | 9876543210 |
| 2 | Priya | 9123456789 |

**2. `Products` (Menu Only)**
| product_id (PK) | item | price |
| :--- | :--- | :--- |
| 10 | Masala Tea | 20 |
| 20 | Biscuit | 10 |

**3. `Orders` (Connecting Table in the Middle)**
| order_id (PK) | customer_id (FK) | product_id (FK) | qty |
| :--- | :--- | :--- | :--- |
| 101 | 1 *(Arun)* | 10 *(Masala Tea)* | 2 |
| 102 | 2 *(Priya)* | 10 *(Masala Tea)* | 1 |

---

## 4. Constraints (Door Locks Against Bad Data)
Rules attached to columns so the database automatically rejects invalid input **before** saving it:
- **`NOT NULL`:** Blocks blank cells (e.g., product `name` cannot be empty).
- **`UNIQUE`:** Blocks duplicate values (e.g., two accounts cannot share the same `email`).
- **`CHECK`:** Enforces a condition (e.g., `CHECK (price > 0)` rejects negative prices like `-50`).
- **`DEFAULT`:** Auto-fills a preset value if left blank (e.g., today's date/time).

---

## 5. Metadata & Data Independence
- **Metadata (Data Dictionary):** "Data about data"—the database's hidden system catalog (`INFORMATION_SCHEMA`) that tracks all table names, column types, and rules. Automated data pipelines inspect this to detect schema changes.
- **Data Independence:**
  - *Logical:* Adding new columns or views does not break existing queries.
  - *Physical:* Adding indexes or upgrading storage hardware speeds up queries without changing any SQL code.
    
