# DBMS – Complete Notes on Keys

**Beginner-friendly notes | Conceptual explanations | Examples | SQL | Exam revision**

---

## Table of Contents
1. [Understanding Rows and Columns](#1-understanding-rows-and-columns)
2. [Primary Key](#2-primary-key-pk)
3. [Candidate Key](#3-candidate-key)
4. [Alternate Key](#4-alternate-key)
5. [Super Key](#5-super-key)
6. [Foreign Key](#6-foreign-key-fk)
7. [Composite Key](#7-composite-key)
8. [Unique Key](#8-unique-key)
9. [Relationship Between Keys](#9-relationship-between-keys)
10. [SQL Examples](#10-sql-examples)
11. [Common Mistakes](#11-common-mistakes)
12. [Practice Questions](#12-practice-questions)
13. [Final Revision](#13-final-revision)
14. [Connection to 2NF](#14-connection-to-2nf)

---

## 1. Understanding Rows and Columns

Before learning keys, understand two basic concepts.

Consider this **Student** table:

| Student_ID | Email | Name | Age |
| :--- | :--- | :--- | :--- |
| 101 | arun@gmail.com | Arun | 20 |
| 102 | ravi@gmail.com | Ravi | 21 |
| 103 | kumar@gmail.com | Kumar | 22 |

* **Row:** Complete information about one student.
* **Column:** A particular type of information, such as `Student_ID`, `Email`, `Name` or `Age`.

Our main objective is to **identify each row uniquely**.

For example, if we want to find Arun's complete information, we should be able to identify his row without confusion.

This is where **database keys** come into the picture.

---

## 2. Primary Key (PK)

### What is a Primary Key?
A **Primary Key** is a column or combination of columns selected to uniquely identify every row in a table.

### Example

| Student_ID (PK) | Name | Age |
| :--- | :--- | :--- |
| 101 | Arun | 20 |
| 102 | Ravi | 21 |
| 103 | Kumar | 22 |

Here, `Student_ID` is the Primary Key.

**Why?**
* `101` identifies Arun.
* `102` identifies Ravi.
* `103` identifies Kumar.

Every student has a different `Student_ID`.

### Primary Key Rules
* Cannot contain duplicate values.
* Cannot contain `NULL` values.
* A table can have **only ONE** Primary Key constraint.
* That one Primary Key can consist of **multiple columns**.

> **Important:** One Primary Key does not mean one column. It can contain two or more columns.

**Example:**
```sql
PRIMARY KEY (Student_ID, Course_ID)
```

This is still **ONE Primary Key**, even though it contains two columns.

---

## 3. Candidate Key

### What is a Candidate Key?
A **Candidate Key** is a **minimal** column or combination of columns that can uniquely identify a row.

The important word is **minimal**.

**Minimal** means we cannot remove any column from that key without losing its ability to uniquely identify rows.

### Example

| Student_ID | Email | Name |
| :--- | :--- | :--- |
| 101 | arun@gmail.com | Arun |
| 102 | ravi@gmail.com | Ravi |
| 103 | kumar@gmail.com | Kumar |

Assume `Student_ID` and `Email` are both unique and non-NULL.

Now think:
* Can `Student_ID` identify Arun? **YES.**
* Can `Email` identify Arun? **YES.**

Both independently identify the same student's row.

Therefore:
* `Student_ID` = Candidate Key.
* `Email` = Candidate Key.

We have **TWO Candidate Keys**.

**Why are they called Candidate Keys?**
Because both are eligible to be selected as the Primary Key.

We have multiple possible choices, but we select one.

For example:
* `Student_ID` is selected as Primary Key.
* `Email` is not selected.

This brings us to **Alternate Key**.

---

## 4. Alternate Key

### What is an Alternate Key?
An **Alternate Key** is a Candidate Key that was **NOT selected** as the Primary Key.

### Example
Our Candidate Keys are:
* `Student_ID`
* `Email`

We select `Student_ID` as the Primary Key.

Now:

| Key | Status |
| :--- | :--- |
| Student_ID | Primary Key |
| Email | Alternate Key |

**Remember:**
* **Candidate Key** = All possible minimal unique identifiers.
* **Primary Key** = The selected Candidate Key.
* **Alternate Key** = The remaining Candidate Keys.

### Visual Understanding
```text
      CANDIDATE KEYS
          /    \
         /      \
 Student_ID    Email
     |           |
PRIMARY KEY  ALTERNATE KEY
 (Selected)  (Not selected)
```

> **IMPORTANT:** Candidate Keys are not a separate set of columns that must be added to the table. They are possible ways of uniquely identifying rows.

---

## 5. Super Key

### What is a Super Key?
A **Super Key** is **any set of one or more columns** that can uniquely identify a row.

Now you might ask:
*If Primary Key and Candidate Key already identify rows uniquely, why do we need Super Key?*

Excellent question!

Super Key explains **ALL possible combinations of columns** that can uniquely identify a row, including combinations containing unnecessary columns.

### Example

| Student_ID | Email | Name |
| :--- | :--- | :--- |
| 101 | arun@gmail.com | Arun |
| 102 | ravi@gmail.com | Ravi |
| 103 | kumar@gmail.com | Kumar |

Assume `Student_ID` and `Email` are unique.

**Possible Super Keys:**

| Super Key | Why is it a Super Key? |
| :--- | :--- |
| `Student_ID` | Already unique |
| `Email` | Already unique |
| `Student_ID + Name` | `Student_ID` is already unique |
| `Email + Name` | `Email` is already unique |
| `Student_ID + Email` | Both together are unique |
| `Student_ID + Email + Name` | Still uniquely identifies each row |

**But why combine `Student_ID + Name`?**
We **DON'T** need to do this in real life.

`Student_ID` alone is enough.

We use this combination to understand the mathematical definition of Super Key.

Even after adding unnecessary columns, the combination still uniquely identifies a row.

That is why it is a Super Key.

### Super Key vs Candidate Key

| Super Key | Candidate Key |
| :--- | :--- |
| Uniquely identifies a row | Uniquely identifies a row |
| May contain unnecessary columns | Cannot contain unnecessary columns |
| Does not have to be minimal | Must be minimal |
| Includes Candidate Keys | Is a minimal Super Key |

### Most Important Relationship
**Every Candidate Key is a Super Key, but every Super Key is NOT a Candidate Key.**

Composite: Student_ID + Course_ID uses two columns.
Super key: Student_ID + Course_ID uniquely identifies a row.

**Example:**
* `Student_ID` = Candidate Key and Super Key.
* `Student_ID + Name` = Super Key, but NOT Candidate Key.

### Remember
```text
        SUPER KEY
            |
            |  Remove unnecessary columns
            v
      CANDIDATE KEY
            |
            |  Select one
            v
       PRIMARY KEY
```

---

## 6. Foreign Key (FK)

### What is a Foreign Key?
A **Foreign Key** is a column or combination of columns in one table that references a key in another table.

In simple words:
**Foreign Key connects two tables.**

### Example
We have two tables.

**Table 1: `Student`**

| Student_ID (PK) | Name |
| :--- | :--- |
| 101 | Arun |
| 102 | Ravi |
| 103 | Kumar |

**Table 2: `Marks`**

| Mark_ID (PK) | Student_ID (FK) | Marks |
| :--- | :--- | :--- |
| 1 | 101 | 90 |
| 2 | 102 | 85 |
| 3 | 103 | 95 |

Notice `Student_ID` appears in both tables.

* In `Student` table: `Student_ID` = Primary Key
* In `Marks` table: `Student_ID` = Foreign Key

**Why?**
Because `Student_ID` in the `Marks` table refers to `Student_ID` in the `Student` table.

For example:
* `Marks` table says `Student_ID = 101`.
* We look at the `Student` table and find `Student_ID` `101`.
* We discover that the student is Arun.
* Therefore, Arun scored 90 marks.

### Foreign Key Rules
* A Foreign Key can contain duplicate values.
* It can sometimes contain `NULL`, if allowed by database constraints.
* It prevents references to non-existent parent records when referential integrity is enforced.
* It can reference a Primary Key or another eligible unique key.

### Important Difference

| Primary Key | Foreign Key |
| :--- | :--- |
| Identifies a row in its own table | References a key in another or the same table |
| Cannot contain `NULL` | Can sometimes contain `NULL` |
| Cannot contain duplicates | Can contain duplicates |

---

## 7. Composite Key

### What is a Composite Key?
A **Composite Key** is a key made up of **TWO OR MORE columns** that work together to uniquely identify a row.

### Example
Imagine students can enrol in multiple courses.

| Student_ID | Course_ID | Marks |
| :--- | :--- | :--- |
| 101 | C01 | 90 |
| 101 | C02 | 85 |
| 102 | C01 | 95 |
| 102 | C02 | 88 |

Now, check the columns.

* Can `Student_ID` alone uniquely identify a row? **NO.** Because `Student_ID` `101` appears twice.
* Can `Course_ID` alone uniquely identify a row? **NO.** Because `C01` appears twice.
* But can `Student_ID + Course_ID` identify a row? **YES.**

For example:
* `101 + C01` = Arun's Java enrolment.
* `101 + C02` = Arun's Python enrolment.

The combination uniquely identifies each enrolment.

Therefore:
```sql
PRIMARY KEY (Student_ID, Course_ID)
```

This is a **Composite Primary Key**.

### Important: Does Two Columns Mean Two Primary Keys?
**NO!**

There is **ONE Primary Key** containing **TWO columns**.

```text
PRIMARY KEY
   |
   |------ Student_ID
   |
   |------ Course_ID
```

This is called a **Composite Primary Key**.

### Composite Key vs Single-Column Key

| Single-Column Key | Composite Key |
| :--- | :--- |
| Uses one column | Uses two or more columns |
| Example: `Student_ID` | Example: `Student_ID + Course_ID` |

> **Important:** A composite key can also be a candidate key, super key, or primary key depending on its role.

---

## 8. Unique Key

### What is a Unique Key?
A **UNIQUE constraint** ensures that values in a column or combination of columns do not repeat.

### Example

| Student_ID (PK) | Email (UNIQUE) | Name |
| :--- | :--- | :--- |
| 101 | arun@gmail.com | Arun |
| 102 | ravi@gmail.com | Ravi |
| 103 | kumar@gmail.com | Kumar |

Here:
* `Student_ID` is Primary Key.
* `Email` has a `UNIQUE` constraint.

**Why?**
We don't want two different students to register using the same email address.

### Primary Key vs Unique Key

| Primary Key | Unique Key |
| :--- | :--- |
| Uniquely identifies a row | Prevents duplicate values |
| Cannot contain `NULL` | `NULL` handling depends on DBMS |
| Only one Primary Key constraint per table | Multiple `UNIQUE` constraints can exist |
| Main row identifier | Enforces uniqueness on additional columns |

> **IMPORTANT:** In SQL, `UNIQUE` is a constraint. It can also be applied to multiple columns together. Different database systems handle `NULL` values in `UNIQUE` constraints differently.

---

## 9. Relationship Between Keys

Understand these relationships carefully.

### Relationship 1: Super Key and Candidate Key
* Every **Candidate Key** is a **Super Key**.
* But every **Super Key** is **NOT** a **Candidate Key**.

**Why?**
Because a Super Key can contain unnecessary columns, whereas a Candidate Key must be minimal.

### Relationship 2: Candidate Key, Primary Key and Alternate Key
* **Candidate Keys** are all possible minimal unique identifiers.
* We select one as the **Primary Key**.
* The remaining ones become **Alternate Keys**.

```text
      CANDIDATE KEYS
          /    \
         /      \
 Student_ID    Email
     |           |
PRIMARY KEY  ALTERNATE KEY
```

### Relationship 3: Primary Key and Foreign Key
* A **Primary Key** identifies a row in one table.
* A **Foreign Key** references that key from another table.

```text
STUDENT TABLE
-----------------
Student_ID (PK)
Name
    |
    | Reference
    v
MARKS TABLE
-----------------
Mark_ID (PK)
Student_ID (FK)
Marks
```

### Relationship 4: Composite Key and Primary Key
A **Composite Key** can be a **Primary Key**.

**Example:**
```sql
PRIMARY KEY (Student_ID, Course_ID)
```

It is **ONE Primary Key** with **TWO columns**.

---

## 10. SQL Examples

Let's see how these keys are used in actual SQL.

### Student Table
```sql
CREATE TABLE Student (
    Student_ID INT PRIMARY KEY,
    Email VARCHAR(100) UNIQUE,
    Name VARCHAR(50)
);
```

Here:
* `Student_ID` = Primary Key.
* `Email` = `UNIQUE` constraint.
* If `Email` is non-NULL and unique, it can also be a Candidate Key.

### Marks Table
```sql
CREATE TABLE Marks (
    Mark_ID INT PRIMARY KEY,
    Student_ID INT,
    Marks INT,
    FOREIGN KEY (Student_ID) REFERENCES Student(Student_ID)
);
```

Here:
* `Mark_ID` = Primary Key.
* `Student_ID` = Foreign Key.

### Composite Primary Key
```sql
CREATE TABLE Student_Course (
    Student_ID INT,
    Course_ID INT,
    PRIMARY KEY (Student_ID, Course_ID)
);
```

Here:
* `Student_ID` and `Course_ID` together form **ONE Composite Primary Key**.

---

## 11. Common Mistakes

| Wrong Understanding | Correct Understanding |
| :--- | :--- |
| Primary Key and Candidate Key identify different things | Both identify rows uniquely |
| Candidate Key identifies a column | Candidate Key identifies a row |
| Two columns in a Primary Key mean two Primary Keys | They form one Composite Primary Key |
| Super Key is always a combination of multiple columns | A single unique column can also be a Super Key |
| We must combine `Student_ID` and `Name` in real life | It is only an example of an unnecessary-column Super Key |
| Foreign Key must always be unique | Foreign Keys can contain duplicates |
| Every table must have exactly one Primary Key column | A Primary Key can consist of multiple columns |

---

## 12. Practice Questions

Try answering these before checking your understanding.

1. **Question 1:** A `Student` table has `Student_ID` and `Email`. Both independently identify students uniquely. How many Candidate Keys are there?
2. **Question 2:** `Student_ID` is selected as the Primary Key. What happens to `Email`?
3. **Question 3:** `Student_ID` alone uniquely identifies a row. Is `Student_ID + Name` also a Super Key?
4. **Question 4:** A `Marks` table contains `Student_ID`, which references `Student_ID` in the `Student` table. What type of key is `Student_ID` in the `Marks` table?
5. **Question 5:** A table has `Student_ID` and `Course_ID`. Neither column alone uniquely identifies a row, but their combination does. What is this called?
6. **Question 6:** Can a table have two separate Primary Key constraints?
7. **Question 7:** Can a table have multiple `UNIQUE` constraints?

---

## 13. Final Revision

| Key | Definition |
| :--- | :--- |
| **Primary Key** | Selected key that uniquely identifies each row |
| **Candidate Key** | Minimal possible key that uniquely identifies each row |
| **Alternate Key** | Candidate Key not selected as Primary Key |
| **Super Key** | Any set of columns that uniquely identifies a row |
| **Foreign Key** | References a key in another table |
| **Composite Key** | Key made using two or more columns |
| **Unique Key** | Constraint that prevents duplicate values |

---

## 14. Connection to 2NF

Now that you understand keys, remember these two concepts especially:
* **Primary Key**
* **Composite Primary Key**

**Why?**
Because **2NF** is concerned with **Partial Dependency**, which can occur when a candidate key has multiple attributes.

**Example:**
```sql
PRIMARY KEY (Student_ID, Course_ID)
```

Suppose:
```text
Student_ID -> Student_Name
Course_ID  -> Course_Name
```

* `Student_Name` depends only on `Student_ID`, not on the entire composite key.
* `Course_Name` depends only on `Course_ID`, not on the entire composite key.

These are **partial dependencies**.

That is the problem we will learn to remove using **Second Normal Form (2NF)**.
# Key Attributes and Non-Key Attributes in DBMS

## 1. What is an Attribute?

An attribute is simply a column in a database table.

Example:

| Student_ID | Course_ID | Student_Name | Course_Name | Marks |
|---|---|---|---|---|
| 101 | C01 | Arun | Java | 85 |
| 101 | C02 | Arun | Python | 90 |
| 102 | C01 | Ravi | Java | 75 |

Here, the attributes are:
- Student_ID
- Course_ID
- Student_Name
- Course_Name
- Marks

## 2. What is a Key Attribute?

A **key attribute** is an attribute that is part of at least one candidate key.

In simple words:

If a column is used as part of a candidate key that uniquely identifies a row, it is a key attribute.

In our example, assume:

- Student_ID + Course_ID together uniquely identify each enrollment.
- Neither Student_ID nor Course_ID alone uniquely identifies an enrollment.

Therefore, the candidate key is:

`(Student_ID, Course_ID)`

Both columns are key attributes:

- Student_ID
- Course_ID

## 3. What is a Non-Key Attribute?

A **non-key attribute** is an attribute that does not belong to any candidate key.

In our example:

| Attribute | Classification |
|---|---|
| Student_ID | Key Attribute |
| Course_ID | Key Attribute |
| Student_Name | Non-Key Attribute |
| Course_Name | Non-Key Attribute |
| Marks | Non-Key Attribute |

Why?

Because Student_Name, Course_Name, and Marks are not part of any candidate key in this example.

## 4. Important: Non-Key Does NOT Mean Unimportant

The word "non-key" does not mean that the column is useless or unimportant.

For example:

- Marks is important because it stores a student's result.
- Student_Name is important because it tells us the student's name.
- Course_Name is important because it tells us the course name.

They are called non-key attributes only because they are not part of a candidate key.

## 5. Prime Attribute vs Non-Prime Attribute

In DBMS, the more technically precise terms are:

### Prime Attribute
An attribute that belongs to at least one candidate key.

### Non-Prime Attribute
An attribute that does not belong to any candidate key.

For our example:

| Attribute | Classification |
|---|---|
| Student_ID | Prime Attribute |
| Course_ID | Prime Attribute |
| Student_Name | Non-Prime Attribute |
| Course_Name | Non-Prime Attribute |
| Marks | Non-Prime Attribute |

**Remember:**

- Prime Attribute = Part of at least one candidate key.
- Non-Prime Attribute = Not part of any candidate key.

## 6. Why Are These Concepts Important?

These concepts are important when learning database normalization.

For example, in 2NF:

A table must:
1. Be in 1NF.
2. Have no partial dependency of a non-prime attribute on any candidate key.

Consider these functional dependencies:

`Student_ID → Student_Name`

`Course_ID → Course_Name`

`(Student_ID, Course_ID) → Marks`

Here:

- Student_Name depends only on Student_ID.
- Course_Name depends only on Course_ID.
- Marks depends on the complete combination of Student_ID and Course_ID.

This helps us identify partial dependencies and understand why decomposition is needed.

---

## Quick Revision

| Term | Meaning |
|---|---|
| Key Attribute | Attribute that belongs to at least one candidate key |
| Non-Key Attribute | Informal term for an attribute that is not part of any candidate key |
| Prime Attribute | Attribute that belongs to at least one candidate key |
| Non-Prime Attribute | Attribute that does not belong to any candidate key |

**Most important point:**

A column can be important to the database without being a key attribute.
---

*End of DBMS Keys Notes.*
