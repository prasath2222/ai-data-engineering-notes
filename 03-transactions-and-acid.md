# Module 3: Transactions, ACID, Locks & Deadlocks

## 1. What is a Transaction?
- **Transaction:** A group of two or more database operations bundled together so the database treats them as **one single unit of work**.
- **Core Commands:**
  - **`BEGIN`:** Starts the transaction (puts changes in a temporary holding area).
  - **`COMMIT`:** Permanently saves all changes once every step succeeds.
  - **`ROLLBACK`:** Cancels and rewinds all changes back to the starting point if any step fails.

---

## 2. ACID Properties (The 4 Safety Guarantees)

### A — Atomicity ("All or Nothing")
- **The Rule:** Either **every single step** inside a transaction succeeds, or **none of them happen**. There is no halfway state.
- **Real-World Example (UPI Transfer of ₹500 from Arun to Priya):**
  1. `UPDATE Accounts SET balance = balance - 500 WHERE user = 'Arun';`
  2. `UPDATE Accounts SET balance = balance + 500 WHERE user = 'Priya';`
- **What happens if the server crashes after Step 1?**
  - Without Atomicity: Arun loses ₹500, Priya gets ₹0, and ₹500 vanishes.
  - With Atomicity: Because Step 2 never finished, the database automatically runs a **`ROLLBACK`** when it restarts—restoring Arun's ₹500.
  - 
