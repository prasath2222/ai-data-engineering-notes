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
- 
