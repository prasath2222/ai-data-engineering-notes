# Module 2: Normalization (1NF $\rightarrow$ 2NF $\rightarrow$ 3NF)

## What is Normalization?
- **Normalization:** A 3-step cleanup checklist (`1NF` $\rightarrow$ `2NF` $\rightarrow$ `3NF`) used to chop one giant, messy Excel-style table into clean, separate tables so data never repeats.
- **Why We Do It (Single Source of Truth):**
  - Update a price or phone number in **one single row** instead of editing 10,000 rows.
  - Prevent spelling typos from ruining sales reports.
  - Save storage space and cloud costs.

---

## Step 1: First Normal Form (`1NF`) — "One Value Per Box"
- **The Rule:** Every single cell (box) in a table must hold **only ONE value**—never a comma-separated list or multiple items crammed together.
- **Extra Rule for Addresses/Names:** Split multi-part details into separate columns (e.g., split `address` into `street`, `city`, and `pincode` so you can easily filter by city).

### Before `1NF` (Messy — Multiple items in one box)
| bill_no | customer | phone | items |
| :--- | :--- | :--- | :--- |
| 101 | Arun | 9876543210 | **Masala Tea, Biscuit** |
| 102 | Priya | 9123456789 | Ginger Tea |

### After `1NF` (Clean — Each item gets its own row)
| bill_no | customer | phone | item |
| :--- | :--- | :--- | :--- |
| 101 | Arun | 9876543210 | **Masala Tea** |
| 101 | Arun | 9876543210 | **Biscuit** |
| 102 | Priya | 9123456789 | Ginger Tea |

## Step 2: Second Normal Form (`2NF`) — "No Half-Key Repeats"

### 1. The 2 Strict Rules for `2NF`
1. The table must already pass **`1NF`** (one single value per cell, no comma-separated lists).
2. **No Partial Dependencies:** Every regular column must depend on the **ENTIRE** Primary Key—never just **one half** of a Composite Primary Key.
   - *(Shortcut rule: If a table has a single-column Primary Key like `customer_id`, it automatically passes `2NF` once it's in `1NF`! Partial dependencies only happen when 2+ columns team up as a Composite Primary Key.)*

### 2. Why It Matters
If `(employee_id, project_id)` is your Primary Key, keeping columns that only care about `employee_id` (like `emp_name`) or only care about `project_id` (like `project_name`) forces you to repeat those names and budgets on every single assignment row.

---

### 3. Before `2NF` (Messy — Both halves of the key cause repeats)
*Composite Primary Key = `(employee_id, project_id)`*

| employee_id | project_id | emp_name | emp_role | project_name | project_budget | hours_worked |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **E1** | **P10** | **Karthik** | **ML Engineer** | **Chatbot AI** | **50,000** | 40 |
| **E1** | **P20** | **Karthik** | **ML Engineer** | Vision App | 80,000 | 25 |
| **E2** | **P10** | Meera | Data Engineer | **Chatbot AI** | **50,000** | 35 |

- `emp_name` & `emp_role` depend **only on `employee_id`** (half the key $\rightarrow$ repeats Karthik).
- `project_name` & `project_budget` depend **only on `project_id`** (half the key $\rightarrow$ repeats Chatbot AI).
- `hours_worked` depends on **both `(employee_id + project_id)`** together (the whole key $\rightarrow$ stays in the middle table).

---

### 4. After `2NF` (Clean — Split into 3 Tables)

**1. `Employees`** *(Depends only on `employee_id`)*
| employee_id (PK) | emp_name | emp_role |
| :--- | :--- | :--- |
| E1 | Karthik | ML Engineer |
| E2 | Meera | Data Engineer |

**2. `Projects`** *(Depends only on `project_id`)*
| project_id (PK) | project_name | project_budget |
| :--- | :--- | :--- |
| P10 | Chatbot AI | 50,000 |
| P20 | Vision App | 80,000 |

**3. `Employee_Projects`** *(Depends on both `employee_id` + `project_id`)*
| employee_id (PK/FK) | project_id (PK/FK) | hours_worked |
| :--- | :--- | :--- |
| E1 | P10 | 40 |
| E1 | P20 | 25 |
| E2 | P10 | 35 |

## Step 3: Third Normal Form (`3NF`) — "No Chain Dependencies"

### 1. The 2 Strict Rules for `3NF`
1. The table must already pass **`1NF`** (one value per cell) and **`2NF`** (no half-key repeats).
2. **No Transitive (Chain) Dependencies:** Every regular column must depend **directly and only** on the Primary Key—never on another regular column (`Primary Key -> Column A -> Column B` is not allowed).

### 2. Why It Matters
Even with a single Primary Key (`student_id`), keeping `hostel_warden` and `warden_phone` inside the `Students` table causes repeats—because the warden belongs to the **`hostel_code`**, not to the individual student. If Mr. Kumar changes his phone number, you would have to update hundreds of student rows.

---

### 3. Before `3NF` (Messy — Chain dependency: `student_id -> hostel_code -> warden`)
*Primary Key = `student_id`*

| student_id (PK) | student_name | hostel_code | hostel_warden | warden_phone |
| :--- | :--- | :--- | :--- | :--- |
| **101** | Rahul | H1 | **Mr. Kumar** | **9876500001** |
| **102** | Divya | H2 | Mrs. Latha | 9876500002 |
| **103** | Sanjay | H1 | **Mr. Kumar** | **9876500001** |

- `student_name` and `hostel_code` depend **directly on `student_id`**.
- `hostel_warden` and `warden_phone` depend **on `hostel_code`**, not `student_id`.

---

### 4. After `3NF` (Clean — Split into `Students` and `Hostels`)

**1. `Students`** *(Depends only on `student_id`)*
| student_id (PK) | student_name | hostel_code (FK) |
| :--- | :--- | :--- |
| 101 | Rahul | H1 |
| 102 | Divya | H2 |
| 103 | Sanjay | H1 |

**2. `Hostels`** *(Depends only on `hostel_code`)*
| hostel_code (PK) | hostel_warden | warden_phone |
| :--- | :--- | :--- |
| H1 | Mr. Kumar | 9876500001 |
| H2 | Mrs. Latha | 9876500002 |

---

## Module 2 Summary Cheat Sheet
- **`1NF` (One Value Per Box):** No comma-separated lists inside a single cell.
- **`2NF` (No Half-Key Repeats):** Every column must depend on the **entire** Primary Key, not just half of a Composite Key.
- **`3NF` (No Chain Dependencies):** Every column must depend **only** on the Primary Key, never on another regular column.
- **OLTP vs. OLAP Rule (Data Engineering):**
  - **App Databases (OLTP — PostgreSQL, MySQL):** Keep tables in **`3NF`** so updates are clean and data never goes out of sync.
  - **Data Warehouses (OLAP — BigQuery, Snowflake, DuckDB):** Often **Denormalize** (merge tables back into Star Schemas) so analytical reports on millions of rows run fast without heavy joins.
    
# Second Normal Form (2NF) — Simple Step-by-Step Guide

## 1. The Core Idea (Why We Combine Two Keys)
In a database, one main ID (like `order_id`) often connects to many items:
* In Order `101`, a customer buys **2 items**: Tea (`P1`) and Vada (`P2`).
* Because there are 2 items, `order_id = 101` must appear **2 times** in the `Order_Items` table.
* **The Problem:** A Primary Key can never repeat. If `order_id` is the only Primary Key, the database blocks the second item.
* **The Solution:** Combine **`order_id` + `product_id`** together as a **2-Part Primary Key**:
  * Row 1: `101 + P1` (Unique)
  * Row 2: `101 + P2` (Unique)
  * Now `order_id` can repeat safely, and you can still use `order_id = 101` to look up every product inside that order.

---

## 2. The 2NF Rule
Whenever a table uses a **2-Part Primary Key** (like `order_id + product_id`), follow one strict rule:

> **Every other column in that table must need BOTH keys together—not just 1 key.**

*(Note: If a table only has a **1-column Primary Key**, it already passes 2NF. You only check 2NF when 2 keys are combined.)*

---

## 3. Example: What Breaks 2NF?

Look at this `Order_Items` table where **`order_id` + `product_id`** is the 2-Part Primary Key:

| order_id (Key 1) | product_id (Key 2) | quantity | product_name | product_price |
| :--- | :--- | :--- | :--- | :--- |
| **101** | **P1** | 2 | Masala Tea | 20 |
| **101** | **P2** | 1 | Medu Vada | 15 |
| **102** | **P1** | 5 | Masala Tea | 20 |

Check every normal column against **Key 1 (`order_id`)** and **Key 2 (`product_id`)**:

1. **`quantity` (Pass — Needs BOTH keys):**
   * Does `quantity` (`2`) need `order_id` (`101`)? **Yes** (to know which order).
   * Does `quantity` (`2`) need `product_id` (`P1`)? **Yes** (to know which product).
   * **Result:** Keep `quantity` in this table.

2. **`product_name` and `product_price` (Fail — Needs ONLY 1 key):**
   * Does `"Masala Tea"` or `20` care about `order_id` (`101` or `102`)? **No.** `P1` is always `"Masala Tea"` and costs `20` in every order.
   * These two columns only belong to **`product_id` (Key 2)**.
   * **Result:** This **breaks 2NF**.

### Why is this bad?
* `"Masala Tea"` and `20` repeat in every single order where someone buys `P1`.
* If the price of Tea changes from `20` to `25`, you must find and edit **thousands of order rows** instead of just 1 row.

---

## 4. How to Fix 2NF (Split and Link)

Move the columns that only need **Key 2 (`product_id`)** into their own separate `Products` table, and link the tables using `product_id`.

### Table 1: `Orders` (1-Part Primary Key: `order_id`)
Stores the main order details once.
| order_id (PK) | customer_id | order_date |
| :--- | :--- | :--- |
| **101** | 1 | 2026-11-16 |
| **102** | 2 | 2026-11-16 |

### Table 2: `Order_Items` (2-Part Primary Key: `order_id + product_id`)
Stores only columns that need **both** `order_id` and `product_id`.
| order_id (PK, FK) | product_id (PK, FK) | quantity |
| :--- | :--- | :--- |
| **101** | **P1** | 2 |
| **101** | **P2** | 1 |
| **102** | **P1** | 5 |

### Table 3: `Products` (1-Part Primary Key: `product_id`)
Stores product details once. If the price changes, you only edit **1 row** here.
| product_id (PK) | product_name | product_price |
| :--- | :--- | :--- |
| **P1** | Masala Tea | 20 |
| **P2** | Medu Vada | 15 |

---

## 5. Quick 3-Step Check for 2NF
1. **Check the Primary Key:** Does it have **2 columns combined**? (If no, 2NF is already done).
2. **Test every other column:** Ask *"Does this column need BOTH IDs, or only ONE ID?"*
3. **Move single-ID columns:** If a column only needs **1 ID**, move it to its own table with that single ID as its Primary Key.
   
# Third Normal Form (3NF) & BCNF — Simple Step-by-Step Guide

## 1. Third Normal Form (3NF) — "No Middleman Column"

### The Core Rule
1. The table must already pass **1NF** and **2NF**.
2. Every normal column in a table must describe **only the Primary Key**.
3. One normal column **cannot** describe another normal column (no middleman chain: `Key -> Column A -> Column B`).

### Example: What Breaks 3NF? (The Bad Table)
Look at this `Employees` table where **`emp_id`** is the Primary Key:

| emp_id (PK) | emp_name | dept_id | dept_city |
| :--- | :--- | :--- | :--- |
| E1 | Arun | D10 | Chennai |
| E2 | Priya | D10 | Chennai |
| E3 | Karthik | D20 | Madurai |

Check every column against **`emp_id` (Primary Key)**:
* **`emp_name` (`Arun`):** Describes Employee `E1`. (**Good**)
* **`dept_id` (`D10`):** Shows which department Employee `E1` works in. (**Good — this is the link**)
* **`dept_city` (`Chennai`):** Describes **Department `D10`**, NOT Employee `E1`. (**Breaks 3NF**)
  * Here, `dept_id` is acting like a middleman: `emp_id` -> `dept_id` -> `dept_city`.

### Why is This Bad? (The 3 Problems)
1. **Repeat Problem:** `"Chennai"` repeats for every employee in `D10`.
2. **Edit Problem:** If Department `D10` moves from Chennai to Coimbatore, you must edit 500 employee rows instead of 1 row.
3. **Delete Problem:** Karthik (`E3`) is the only worker in `D20`. If Karthik quits and you delete his row, you also lose the fact that `D20` is in `Madurai`.

### How to Fix 3NF (Split and Link)
Keep **only the link ID (`dept_id`)** in `Employees`, and move **`dept_id` + `dept_city`** into its own `Departments` table:

**Table 1: `Employees` (Primary Key = `emp_id`)**
| emp_id (PK) | emp_name | dept_id (Link / FK) |
| :--- | :--- | :--- |
| E1 | Arun | D10 |
| E2 | Priya | D10 |
| E3 | Karthik | D20 |

**Table 2: `Departments` (Primary Key = `dept_id`)**
| dept_id (PK) | dept_city |
| :--- | :--- |
| D10 | Chennai |
| D20 | Madurai |

---

## 2. Important Design Rule: 2 Tables vs. 3 Tables
Both designs below follow **2NF and 3NF**. Choose based on your business rule:

### Case A: Use 2 Tables (1 Employee -> 1 Department)
If each employee works in **only 1 department**, put `dept_id` inside `Employees` as a Foreign Key link:
* **`Employees`:** `emp_id (PK)`, `emp_name`, `dept_id (FK)`
* **`Departments`:** `dept_id (PK)`, `dept_city`

### Case B: Use 3 Tables (1 Employee -> Many Departments)
If one employee can work in **many departments** (and one department has many employees), create a **3rd link table** using a 2-part Primary Key:

**Table 1: `Employees`**
| emp_id (PK) | emp_name |
| :--- | :--- |
| E1 | Arun |
| E2 | Priya |

**Table 2: `Departments`**
| dept_id (PK) | dept_city |
| :--- | :--- |
| D10 | Chennai |
| D20 | Madurai |

**Table 3: `Employee_Departments` (Link Table — 2-Part Primary Key)**
| emp_id (PK, FK) | dept_id (PK, FK) |
| :--- | :--- |
| E1 | D10 |
| E1 | D20 |
| E2 | D10 |

---

## 3. BCNF (Boyce-Codd Normal Form) — "Strict 3NF"

BCNF is a slightly stricter version of 3NF. Most tables that pass 3NF already pass BCNF.

### The Core Rule
> **If Column A decides Column B, then Column A MUST be a Primary Key in that table.**
> A normal column cannot decide any other column—not even part of the Primary Key.

### Example: What Breaks BCNF?
Suppose a college has two rules:
1. One student can take many subjects.
2. **Each teacher teaches only 1 subject** (Ravi only teaches SQL; Meena only teaches Python).

Look at this table where **`student_id + subject`** is the 2-Part Primary Key:

| student_id (Key 1) | subject (Key 2) | teacher (Normal Column) |
| :--- | :--- | :--- |
| S1 | SQL | Ravi |
| S2 | SQL | Ravi |
| S1 | Python | Meena |

* **What is wrong?**
  * Because each teacher teaches only 1 subject, **`teacher` decides `subject`** (`Ravi` -> `SQL`).
  * `teacher` is a **normal column**, NOT a Primary Key, yet it decides `subject` (Key 2). This **breaks BCNF**.
  * `"Ravi"` + `"SQL"` repeats for every student. If S1 and S2 drop the class, you lose the record that Ravi teaches SQL.

### How to Fix BCNF
Split the table so **`teacher`** becomes the Primary Key of its own table:

**Table 1: `Student_Teachers` (2-Part Primary Key: `student_id + teacher`)**
| student_id (PK) | teacher (PK, FK) |
| :--- | :--- |
| S1 | Ravi |
| S2 | Ravi |
| S1 | Meena |

**Table 2: `Teachers` (Primary Key: `teacher`)**
| teacher (PK) | subject |
| :--- | :--- |
| Ravi | SQL |
| Meena | Python |

---

## 4. Complete Normalization Cheat Sheet (1NF to BCNF)

| Normal Form | What the Bad Table Looks Like | Simple Rule to Fix It |
| :--- | :--- | :--- |
| **1NF** | A single box holds a list of values (`"Tea, Vada"`). | Put **only 1 value** in each box (create separate rows). |
| **2NF** | A **2-part Primary Key** (`order_id + product_id`), and a column (`product_name`) only needs **1 key**. | Move that column to its own table with that **1 key** as Primary Key. |
| **3NF** | A **1-part Primary Key** (`emp_id`), and a normal column (`dept_city`) describes **another normal column** (`dept_id`). | Keep `dept_id` as the link, and move `dept_id` + `dept_city` to a new table. |
| **BCNF** | A **normal column** (`teacher`) decides **part of the Primary Key** (`subject`). | Make `teacher` the Primary Key of its own table. |
