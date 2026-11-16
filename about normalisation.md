# DBMS – Database Normalization (From Zero)

Before learning **1NF**, **2NF** and **3NF**, we need to understand **WHY** normalization exists in the first place.

I'll explain using a real-world college database.

---

## 1. What is Database Normalization?

**Database normalization** is the process of organizing data into properly structured tables to:

* Reduce duplicate data.
* Avoid data inconsistency.
* Prevent insertion problems.
* Prevent update problems.
* Prevent deletion problems.

**In simple words:**

> Normalization means organizing database tables so that data is stored efficiently without unnecessary duplication or problems.

---

## 2. Why do we need Normalization?

Imagine a college maintains student and course information in one table.

### `Student_Course` Table

| Student_ID | Student_Name | Course_ID | Course_Name | Instructor |
| :--- | :--- | :--- | :--- | :--- |
| 101 | Arun | C01 | Java | Kumar |
| 101 | Arun | C02 | Python | Ravi |
| 102 | Ravi | C01 | Java | Kumar |
| 103 | Kumar | C01 | Java | Kumar |

At first glance, this table looks fine.

**But look carefully:**

* Arun's name is repeated.
* Java is repeated.
* Kumar (instructor) is repeated.

This creates unnecessary duplication.

Let's understand what problems this causes.

---

## 3. Three Major Problems Without Normalization

These are called **Database Anomalies**.

### A. Update Anomaly

Suppose the Java instructor changes from **Kumar** to **Suresh**.

Java appears in three rows:

| Student_ID | Course_Name | Instructor |
| :--- | :--- | :--- |
| 101 | Java | Kumar |
| 102 | Java | Kumar |
| 103 | Java | Kumar |

We need to update all three rows.

What happens if we update only two rows?

| Student_ID | Course_Name | Instructor |
| :--- | :--- | :--- |
| 101 | Java | Suresh |
| 102 | Java | Suresh |
| 103 | Java | Kumar |

Now the same course has two different instructors in our database.

* This is an **Update Anomaly**.
* The data becomes inconsistent.

### B. Insertion Anomaly

Imagine the college introduces a new course called **C++**.

But no student has enrolled in it yet.

Our existing table requires student information for each row.

How do we insert C++ without a student?

We may be forced to:

* Insert `NULL` values for student information.
* Wait until a student enrols.

This is an **Insertion Anomaly**.

### C. Deletion Anomaly

Imagine **Ravi** is the only student enrolled in **Python**.

We delete Ravi's enrollment because he leaves the college.

But what if his row also contains the only information about Python?

Deleting Ravi's row could accidentally delete the information about the Python course.

This is a **Deletion Anomaly**.

---

## 4. How does Normalization solve these problems?

Instead of keeping everything in one table, we divide the data into logically related tables.

For example:

### Table 1: `Student`

| Student_ID | Student_Name |
| :--- | :--- |
| 101 | Arun |
| 102 | Ravi |
| 103 | Kumar |

### Table 2: `Course`

| Course_ID | Course_Name | Instructor |
| :--- | :--- | :--- |
| C01 | Java | Kumar |
| C02 | Python | Ravi |

### Table 3: `Student_Course`

| Student_ID | Course_ID |
| :--- | :--- |
| 101 | C01 |
| 101 | C02 |
| 102 | C01 |
| 103 | C01 |

**Now:**

* Student information is stored separately.
* Course information is stored separately.
* Student-course relationships are stored separately.

We avoid repeating unnecessary information.

**Better database structure.**

---

## 5. Different Normal Forms

Normalization has multiple stages:

| Normal Form | Main Rule |
| :--- | :--- |
| **1NF** | Each cell contains one atomic value |
| **2NF** | No partial dependency |
| **3NF** | No transitive dependency |
| **BCNF** | Every determinant must be a super key |

We'll learn each one step by step.

### Our Learning Roadmap

* **1NF – First Normal Form**
  * Remove multiple values from a single cell.
  * *Starting here*
* **2NF – Second Normal Form**
  * Remove partial dependencies.
* **3NF – Third Normal Form**
  * Remove transitive dependencies.
* **BCNF**
  * Handle stronger functional dependency rules.

---

## 6. Check Your Understanding

Three quick questions:

**1. Which anomaly happens when we change data in one row but forget to update other repeated rows?**
* [ ] Insertion Anomaly
* [ ] Update Anomaly
* [ ] Deletion Anomaly

**2. Which anomaly happens when we cannot add a new course because no student has enrolled yet?**
* [ ] Insertion Anomaly
* [ ] Update Anomaly
* [ ] Deletion Anomaly

**3. Which anomaly happens when deleting a student accidentally removes course information?**
* [ ] Insertion Anomaly
* [ ] Update Anomaly
* [ ] Deletion Anomaly

---

**Next lesson:** **1NF (First Normal Form)** — we'll take an incorrect table, identify exactly what's wrong, and convert it into 1NF step by step.
