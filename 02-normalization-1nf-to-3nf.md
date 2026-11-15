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

