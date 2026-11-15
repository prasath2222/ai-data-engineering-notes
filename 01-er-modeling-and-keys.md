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
  3. 
