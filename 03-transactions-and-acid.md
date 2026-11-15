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

## 3. Locks (How the Database Makes People Wait Their Turn)

### The Real-World Problem
Imagine there is **only 1 last movie ticket** left in your `Shows` table (`seats_left = 1`).
At the exact same millisecond, **Arun** and **Priya** both click **"Book Now"**.

If the database lets both of them edit that row at the exact same time, both see `seats_left = 1`, both subtract `1`, and you just sold the same seat to two people.

To stop this, the database puts a **Lock** on a row while it is being used—just like locking a fitting room door while you change clothes.

---

### The 2 Types of Locks

#### 1. Shared Lock (Read Lock — "Looking Through the Window")
- **When it happens:** Whenever someone runs a `SELECT` query (just reading data, like checking how many seats are left).
- **How it works:** 100 people can look at `seats_left = 1` at the exact same time. Nobody is changing the number yet, so everyone is allowed to read together.
- **What it blocks:** Nobody can edit (`UPDATE` or `DELETE`) that row while people are actively reading it.

#### 2. Exclusive Lock (Write Lock — "Door Locked, Editing Now")
- **When it happens:** The instant someone runs `UPDATE`, `INSERT`, or `DELETE` (like Arun clicking "Pay & Book").
- **How it works Step-by-Step:**
  1. Arun's transaction reaches the `seats_left = 1` row first and slaps an **Exclusive Lock** on it.
  2. Priya's transaction arrives 1 millisecond later. Because Arun holds the Exclusive Lock, the database **pauses Priya's query** and makes her wait in line.
  3. Arun finishes paying and hits **`COMMIT`** (`seats_left` is now `0`). The Exclusive Lock opens.
  4. Now Priya's paused query wakes up, checks `seats_left`, sees **`0`**, and politely tells her *"Sold Out."*

---

## 4. Deadlocks ("Two People Stuck in a Narrow Doorway")

Locks keep data safe, **until two transactions lock each other out in a circle** and freeze forever. That freeze is called a **Deadlock**.

### Step-by-Step How a Deadlock Happens
Imagine **Arun** sends ₹500 to **Priya**, and at the exact same millisecond, **Priya** sends ₹500 to **Arun**:

1. **Millisecond 1:**
   - **Arun's Transaction** locks **Arun's row** (so it can subtract ₹500).
   - **Priya's Transaction** locks **Priya's row** (so it can subtract ₹500).
2. **Millisecond 2:**
   - **Arun's Transaction** now tries to lock **Priya's row** (to add ₹500) $\rightarrow$ **BLOCKED!** Priya already locked it, so Arun waits.
   - **Priya's Transaction** now tries to lock **Arun's row** (to add ₹500) $\rightarrow$ **BLOCKED!** Arun already locked it, so Priya waits.
3. **Millisecond 3 (The Freeze):**
   - Arun will not unlock his row until Priya finishes.
   - Priya will not unlock her row until Arun finishes.
   - **Result:** Both wait forever. Neither can move forward.

---

### How Databases Fix Deadlocks Automatically
- **The Referee (Deadlock Detector):** Every few milliseconds, the database scans for circular waits where Transaction A waits on B, and B waits on A.
- **Picking a "Victim":** As soon as it spots the circle, the database picks one transaction as the **victim**, forces it to **`ROLLBACK`** (cancels it completely and unlocks its row), and lets the other transaction finish immediately.
- **What Your App Sees:** The cancelled transaction gets an error (`Deadlock detected, please retry`), and your app automatically runs it again 10 milliseconds later when the row is free.

### How Programmers Prevent Deadlocks
- **Always Lock in the Same Order:** Instead of Arun locking Arun first and Priya locking Priya first, programmers write a rule: *"Always lock the smaller `account_id` first."*
  - If `Arun = ID 1` and `Priya = ID 2`, **both** transactions try to lock **ID 1 first**.
  - One gets ID 1, the other waits cleanly outside the door, and a Deadlock never happens.

---

## Quick Memory Table (Locks & Deadlocks)
| Concept | Simple Analogy | What It Does |
| :--- | :--- | :--- |
| **Shared Lock** | Looking through a shop window | Many users can read (`SELECT`) a row at the same time. |
| **Exclusive Lock** | Locking a fitting room door | Only **1 user** can edit (`UPDATE`/`DELETE`) a row; others must wait. |
| **Deadlock** | Two people stuck in a narrow doorway | Transaction A holds Row 1 and wants Row 2; Transaction B holds Row 2 and wants Row 1. |
| **Deadlock Victim** | Referee pulling one person back | Database kills (`ROLLBACK`) one transaction so the other can finish. |
