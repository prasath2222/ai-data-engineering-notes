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
