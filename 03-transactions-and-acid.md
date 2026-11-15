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
    
### C — Consistency ("Follow the Rules Before & After")
- **The Rule:** Every transaction must take the database from one valid state to another valid state—never violating table constraints (`CHECK`, `NOT NULL`, `FOREIGN KEY`) or business rules.
- **Example:** If `Accounts` has `CHECK (balance >= 0)` and Arun only has ₹300, trying to transfer ₹500 fails the rule and triggers an automatic **`ROLLBACK`** so his balance never becomes `-200`.

### I — Isolation ("Don't Step on Each Other's Toes")
- **The Rule:** Concurrent transactions running at the exact same time cannot see each other's half-finished (uncommitted) work—each behaves as if it is running alone.
- **Example:** If Arun has ₹500 and triggers two ₹500 transfers at the exact same millisecond, Isolation ensures one finishes (`COMMIT`) before the other reads the balance—preventing double-spending.

### D — Durability ("Once Committed, It's Permanent")
- **The Rule:** Once a transaction hits **`COMMIT`**, its changes are permanently saved to non-volatile disk (using a **Write-Ahead Log / WAL**) and survive power outages or server crashes.
- **Example:** Once a payment confirmation is sent, a power cut 1 millisecond later will not erase the transaction.

---

## ACID Quick Cheat Sheet
| Letter | Name | Core Promise | Real-World Failure Prevented |
| :--- | :--- | :--- | :--- |
| **A** | **Atomicity** | All steps succeed, or all undo (`ROLLBACK`). | Money deducted from sender, never reaches receiver. |
| **C** | **Consistency** | All constraints (`CHECK`, `FK`) stay valid. | Account balance dropping to `-200`. |
| **I** | **Isolation** | Parallel transactions don't corrupt each other. | Double-spending ₹500 twice at the exact same second. |
| **D** | **Durability** | Committed data survives crashes (WAL). | Confirmed payment vanishing after a power cut. |

## 3. Locks (How Databases Enforce Isolation)
When multiple users access the same row simultaneously, databases use **Locks** to prevent race conditions and overwritten data:
- **Shared Lock (Read Lock):** Triggered during `SELECT` queries. Multiple users can read the same row at the same time, but nobody can modify it until readers finish.
- **Exclusive Lock (Write Lock):** Triggered during `INSERT`, `UPDATE`, or `DELETE`. Only **one** transaction can hold an Exclusive Lock on a row at a time—all other transactions must wait until `COMMIT` or `ROLLBACK` releases the lock.

---

## 4. Deadlocks ("Stuck in a Narrow Doorway")
- **What is a Deadlock?** When two transactions hold locks that the other needs, causing both to wait forever:
  1. **Transaction A** locks Row 1, then waits for Row 2.
  2. **Transaction B** locks Row 2, then waits for Row 1.
- **How Databases Resolve It:** The database's **Deadlock Detector** spots the circular wait within milliseconds, kills one transaction (the **"victim"**) via **`ROLLBACK`**, frees its lock so the other finishes, and returns a deadlock error so the application can retry.
- **How Engineers Prevent It:** Always access and lock rows/tables in a **consistent order** (e.g., always lock lower IDs before higher IDs) and keep transactions short.
  
