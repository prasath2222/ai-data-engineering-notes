# Third Normal Form (3NF) – Complete Notes

## 1. What is 3NF?

3NF stands for Third Normal Form.

A table is in 3NF when:

1. It is already in First Normal Form (1NF).
2. It is already in Second Normal Form (2NF).
3. It satisfies the 3NF rule (no transitive dependency that violates 3NF).

The main purpose of 3NF is to ensure that non-key attributes do not depend transitively on candidate keys.

### In simple words

Imagine a table containing student information and department information.

If we want to find a department name, we can use Dept_ID.

We don't need Student_ID directly to determine the department name.

This creates a dependency through another non-key attribute.

3NF helps us remove that problem by separating the information into logically related tables.

---

## 2. First Understand Why We Need 3NF

Imagine a college maintains student and department information in one table.

### Student Table

| Student_ID | Student_Name | Dept_ID | Dept_Name |
|---|---|---|---|
| 101 | Arun | D01 | Computer Science |
| 102 | Ravi | D02 | Mechanical |
| 103 | Kumar | D01 | Computer Science |
| 104 | Bala | D03 | Electrical |

Understand what each row represents.

- Student 101 belongs to Computer Science.
- Student 102 belongs to Mechanical.
- Student 103 belongs to Computer Science.
- Student 104 belongs to Electrical.

**One row represents one student's information along with their department information.**

This is important because we are storing both student information and the department to which each student belongs.

---

## 3. Understand the Primary Key

Before applying 3NF, we must understand how to identify one particular student record.

Look at the table:

| Student_ID | Student_Name | Dept_ID | Dept_Name |
|---|---|---|---|
| 101 | Arun | D01 | Computer Science |
| 102 | Ravi | D02 | Mechanical |
| 103 | Kumar | D01 | Computer Science |
| 104 | Bala | D03 | Electrical |

### A. Can Student_ID alone identify one student record?

Suppose we search:

`Student_ID = 101`

We get:

| Student_ID | Student_Name | Dept_ID | Dept_Name |
|---|---|---|---|
| 101 | Arun | D01 | Computer Science |

We get exactly one row.

Why?

Because every student has a unique Student_ID.

Therefore, Student_ID uniquely identifies one student record.

**Student_ID is our Primary Key.**

```sql
PRIMARY KEY (Student_ID)
```

### B. Can Dept_ID alone identify one student record?

Suppose we search:

`Dept_ID = D01`

We get:

| Student_ID | Student_Name | Dept_ID | Dept_Name |
|---|---|---|---|
| 101 | Arun | D01 | Computer Science |
| 103 | Kumar | D01 | Computer Science |

We get two rows.

Why?

Because multiple students can belong to the same department.

Therefore, Dept_ID cannot uniquely identify one student record.

**Important distinction:**

- Student_ID identifies one particular student.
- Dept_ID identifies a department.
- Dept_ID does not identify one particular student.

### C. What is our Primary Key?

`Student_ID`

We don't need to combine Student_ID and Dept_ID because Student_ID alone uniquely identifies each row.

---

## 4. Understand the Actual Problem: Transitive Dependency

Now we have established that our primary key is:

`Student_ID`

Look at the table:

| Student_ID | Student_Name | Dept_ID | Dept_Name |
|---|---|---|---|
| 101 | Arun | D01 | Computer Science |
| 102 | Ravi | D02 | Mechanical |
| 103 | Kumar | D01 | Computer Science |
| 104 | Bala | D03 | Electrical |

Ask yourself these questions.

### A. What do we need to find Student_Name?

Suppose I give you:

`Student_ID = 101`

Can you determine the student's name?

Yes. It is Arun.

Therefore:

`Student_ID → Student_Name`

Student_Name depends on Student_ID.

### B. What do we need to find Dept_ID?

Suppose I give you:

`Student_ID = 101`

Can you determine which department Arun belongs to?

Yes. It is D01.

Therefore:

`Student_ID → Dept_ID`

Dept_ID depends on Student_ID.

### C. What do we need to find Dept_Name?

Now suppose I give you:

`Dept_ID = D01`

Can you determine the department name?

Yes. It is Computer Science.

Do you need Student_ID to find the department name?

No.

Therefore:

`Dept_ID → Dept_Name`

Dept_Name depends on Dept_ID.

### D. Understand the Important Difference

Look at these dependencies:

```text
Student_ID → Student_Name

Student_ID → Dept_ID

Dept_ID → Dept_Name
```

Now connect them:

```text
Student_ID → Dept_ID → Dept_Name
```

What does this mean?

1. Student_ID determines Dept_ID.
2. Dept_ID determines Dept_Name.
3. Therefore, Student_ID indirectly determines Dept_Name through Dept_ID.

This is called **Transitive Dependency**.

### Summary of Dependencies

| Attribute | What determines its value? |
|---|---|
| Student_Name | Student_ID |
| Dept_ID | Student_ID |
| Dept_Name | Dept_ID |

Notice the difference.

- Student_Name depends directly on Student_ID.
- Dept_ID depends directly on Student_ID.
- Dept_Name depends on Dept_ID, which is another non-key attribute.

This is the central concept of 3NF.

---

## 5. What Exactly Is Transitive Dependency?

Transitive dependency occurs when a non-prime attribute depends transitively on a candidate key through another attribute.

In our example:

Complete candidate key:

`Student_ID`

But:

```text
Student_ID → Dept_ID
```

And:

```text
Dept_ID → Dept_Name
```

Therefore:

```text
Student_ID → Dept_ID → Dept_Name
```

Dept_Name depends on Dept_ID instead of directly depending on Student_ID.

This creates transitive dependency.

### Visual Understanding

```text
          Student_ID
          (Primary Key)
               |
       +-------+-------+
       |               |
       v               v
  Student_Name       Dept_ID
                       |
                       |
                       v
                   Dept_Name
```

Notice:

- Student_Name depends directly on Student_ID.
- Dept_ID depends directly on Student_ID.
- Dept_Name depends on Dept_ID.

That last dependency creates the problem.

**Remember:**

Transitive dependency can exist even when a table has a single-column candidate key.

Unlike partial dependency, it does not require a composite key.

---

## 6. How Do We Convert the Table Into 3NF?

We separate the information based on what determines it.

### Original Table

| Student_ID | Student_Name | Dept_ID | Dept_Name |
|---|---|---|---|
| 101 | Arun | D01 | Computer Science |
| 102 | Ravi | D02 | Mechanical |
| 103 | Kumar | D01 | Computer Science |
| 104 | Bala | D03 | Electrical |

We identified these dependencies:

```text
Student_ID → Student_Name

Student_ID → Dept_ID

Dept_ID → Dept_Name
```

Now separate them.

### Table 1: Student

| Student_ID (PK) | Student_Name | Dept_ID (FK) |
|---|---|---|
| 101 | Arun | D01 |
| 102 | Ravi | D02 |
| 103 | Kumar | D01 |
| 104 | Bala | D03 |

Why is Dept_ID here?

Because we still need to know which department each student belongs to.

Dept_ID is now a Foreign Key referencing the Department table.

### Table 2: Department

| Dept_ID (PK) | Dept_Name |
|---|---|
| D01 | Computer Science |
| D02 | Mechanical |
| D03 | Electrical |

Why is Dept_Name here?

Because Dept_ID alone determines Dept_Name.

We only need to store each department name once.

### Important

We did not remove Dept_ID from the Student table.

We moved Dept_Name into the Department table and kept Dept_ID in both tables to maintain their relationship.

---

## 7. Understand PK and FK in the Two Tables

### Student Table

```text
Student_ID = Primary Key

Dept_ID = Foreign Key
```

Student_ID uniquely identifies each student.

Dept_ID references the department to which the student belongs.

### Department Table

```text
Dept_ID = Primary Key
```

Dept_ID uniquely identifies each department.

### Relationship

```text
STUDENT
-------------------------
Student_ID (PK)
Student_Name
Dept_ID (FK)
          |
          |
          v
DEPARTMENT
-------------------------
Dept_ID (PK)
Dept_Name
```

The Student table connects to the Department table using Dept_ID.

### What is the Relationship?

One department can have multiple students.

For example:

| Dept_ID | Student_ID |
|---|---|
| D01 | 101 |
| D01 | 103 |

This is a **One-to-Many (1:M) relationship**.

- One department.
- Many students.

---

## 8. Why Does This Separation Matter?

Let's understand what problems we avoid.

### A. Update Anomaly

Suppose the Computer Science department changes its name to Computer Engineering.

In the original table, we need to update multiple student records.

| Student_ID | Dept_ID | Dept_Name |
|---|---|---|
| 101 | D01 | Computer Science |
| 103 | D01 | Computer Science |

After separation, we update only the Department table.

| Dept_ID | Dept_Name |
|---|---|
| D01 | Computer Engineering |

We don't need to update every student's record.

### B. Insertion Anomaly

Suppose the college introduces a new department called Civil Engineering.

But no students have joined that department yet.

In the original table, inserting department information without a student is difficult.

After separation, we can insert Civil Engineering directly into the Department table.

### C. Deletion Anomaly

Suppose the only student belonging to the Electrical department leaves the college.

If we delete that student's row, we might accidentally lose the only information about the Electrical department.

After separation, deleting a student does not delete the department information.

---

## 9. Important Rules to Remember

| Rule | Explanation |
|---|---|
| 2NF is compulsory | A table must already satisfy 2NF before it can satisfy 3NF. |
| No transitive dependency violating 3NF | Non-prime attributes must not depend transitively on candidate keys. |
| Non-key attributes | They should not depend on other non-key attributes in a way that violates 3NF. |
| Single-column keys | Transitive dependency can exist even with a single-column candidate key. |
| Keep the relationship | Preserve necessary primary keys and foreign keys when splitting tables. |
| Move attributes based on dependencies | Do not blindly split tables just because they contain multiple columns. |

### Important Technical Note

The formal definition of 3NF is:

For every non-trivial functional dependency:

`X → A`

At least one of these must be true:

1. X is a superkey.
2. A is a prime attribute (belongs to at least one candidate key).

For our example:

`Dept_ID → Dept_Name`

violates 3NF because:

- Dept_ID is not a superkey of the original Student table.
- Dept_Name is not a prime attribute.

---

## 10. SQL Example

Create the Department table:

```sql
CREATE TABLE Department (
    Dept_ID VARCHAR(10) PRIMARY KEY,
    Dept_Name VARCHAR(100)
);
```

Create the Student table:

```sql
CREATE TABLE Student (
    Student_ID INT PRIMARY KEY,
    Student_Name VARCHAR(100),
    Dept_ID VARCHAR(10),

    FOREIGN KEY (Dept_ID)
        REFERENCES Department(Dept_ID)
);
```

Insert department details:

```sql
INSERT INTO Department
VALUES
('D01', 'Computer Science'),
('D02', 'Mechanical'),
('D03', 'Electrical');
```

Insert student details:

```sql
INSERT INTO Student
VALUES
(101, 'Arun', 'D01'),
(102, 'Ravi', 'D02'),
(103, 'Kumar', 'D01'),
(104, 'Bala', 'D03');
```

### Retrieve Students Along With Their Department Names

```sql
SELECT
    S.Student_ID,
    S.Student_Name,
    D.Dept_Name
FROM Student S
JOIN Department D
    ON S.Dept_ID = D.Dept_ID;
```

Result:

| Student_ID | Student_Name | Dept_Name |
|---|---|---|
| 101 | Arun | Computer Science |
| 102 | Ravi | Mechanical |
| 103 | Kumar | Computer Science |
| 104 | Bala | Electrical |

Notice that the information is stored in separate tables, but we can combine it whenever we need using JOIN.

**Separating tables does not mean losing the ability to retrieve complete information.**

---

## 11. Common Confusions

### Confusion 1: Why can't we keep Dept_Name in the Student table?

Because multiple students can belong to the same department.

We would need to repeat the same department name in multiple rows.

Instead, we store the department name once in the Department table.

### Confusion 2: Why do we keep Dept_ID in both tables?

Because we need to connect the Student and Department tables.

- Dept_ID in Student is a Foreign Key.
- Dept_ID in Department is a Primary Key.

### Confusion 3: Is Dept_ID a Primary Key in the Student table?

No.

Dept_ID is a Foreign Key in the Student table.

It is not unique there because multiple students can belong to the same department.

### Confusion 4: Does transitive dependency mean indirect dependency?

Yes.

In our example:

```text
Student_ID → Dept_ID

Dept_ID → Dept_Name
```

Student_ID indirectly determines Dept_Name through Dept_ID.

### Confusion 5: Is 3NF the same as 2NF?

No.

| 2NF | 3NF |
|---|---|
| Removes partial dependency | Removes transitive dependency that violates 3NF |
| Focuses on dependency on part of a candidate key | Focuses on dependencies through other attributes |
| Requires 1NF | Requires 2NF |

### Confusion 6: Does every 3NF table need two tables?

No.

The number of tables depends on the dependencies.

Sometimes no decomposition is necessary.

Sometimes multiple tables are needed.

3NF is about satisfying the 3NF dependency rule, not creating a fixed number of tables.

---

## 12. Final Summary

### Remember This Example

Original table:

```text
Student_ID
Student_Name
Dept_ID
Dept_Name
```

Primary key:

```text
Student_ID
```

Dependencies:

```text
Student_ID → Student_Name

Student_ID → Dept_ID

Dept_ID → Dept_Name
```

Problem:

```text
Dept_ID → Dept_Name
```

Dept_Name depends on another non-key attribute, Dept_ID.

This creates a transitive dependency that violates 3NF.

Solution:

```text
STUDENT
-----------------
Student_ID (PK)
Student_Name
Dept_ID (FK)


DEPARTMENT
-----------------
Dept_ID (PK)
Dept_Name
```

### The One-Line Definition

**3NF ensures that every non-prime attribute satisfies the 3NF dependency rule and does not depend transitively on a candidate key.**

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| 3NF | 2NF + no transitive dependency violating 3NF |
| Transitive dependency | Indirect dependency through another attribute |
| Student_ID → Dept_ID | Student ID determines department ID |
| Dept_ID → Dept_Name | Department ID determines department name |
| Student_ID → Dept_Name | Indirect dependency through Dept_ID |
| Decomposition | Separating Student and Department information |
| Primary Key | Student_ID in Student; Dept_ID in Department |
| Foreign Key | Dept_ID in Student |
| JOIN | Reconnecting separated tables when we need combined information |
# Third Normal Form (3NF) – Understanding Transitive Dependency

## 1. What is 3NF?

**Third Normal Form (3NF)** is a database normalization rule that removes certain **transitive dependencies**.

In simple words:

> A non-prime attribute should not depend on another non-prime attribute when that dependency causes a violation of 3NF.

Instead, non-prime attributes should depend on a candidate key, the whole candidate key, and nothing but the candidate key, subject to the formal 3NF rule.

### First, remember:

- **Prime attribute:** An attribute that is part of at least one candidate key.
- **Non-prime attribute:** An attribute that is not part of any candidate key.
- **Candidate key:** A minimal set of attributes that uniquely identifies a row.
- **Super key:** Any set of attributes that uniquely identifies a row.

---

## 2. Let's Understand 3NF Using One Example

Suppose we have a Student table:

| Student_ID | Student_Name | Dept_ID | Dept_Name |
|---|---|---|---|
| 101 | Arun | D01 | Computer Science |
| 102 | Ravi | D02 | Mechanical |
| 103 | Kumar | D01 | Computer Science |
| 104 | Bala | D03 | Electrical |

Our primary key is:

`Student_ID`

Why?

Because each student has a unique Student_ID.

For example:

- 101 identifies Arun.
- 102 identifies Ravi.
- 103 identifies Kumar.
- 104 identifies Bala.

Therefore, `Student_ID` uniquely identifies every row.

---

## 3. Understand Which Attribute Depends on Which Attribute

Look at the table carefully.

| Attribute | Depends on |
|---|---|
| Student_Name | Student_ID |
| Dept_ID | Student_ID |
| Dept_Name | Dept_ID |

Let's understand each dependency.

### A. Student_ID → Student_Name

If I give you Student_ID = 101, you can find the student's name.

`101 → Arun`

Therefore:

`Student_ID → Student_Name`

### B. Student_ID → Dept_ID

If I give you Student_ID = 101, you can find the student's department ID.

`101 → D01`

Therefore:

`Student_ID → Dept_ID`

### C. Dept_ID → Dept_Name

If I give you Dept_ID = D01, you can find the department name.

`D01 → Computer Science`

Therefore:

`Dept_ID → Dept_Name`

**Important:** Notice that Dept_ID determines Dept_Name. We do not need Student_ID to determine the department name once we already know Dept_ID.

---

## 4. Understand the Transitive Dependency

Now combine the dependencies:

```text
Student_ID → Dept_ID
Dept_ID    → Dept_Name
```

Therefore:

```text
Student_ID → Dept_ID → Dept_Name
```

This is called a **transitive dependency**.

### What does that actually mean?

Suppose I ask:

**What is Arun's department name?**

You find the answer through these steps:

**Step 1: Find Student_ID**

```text
Student_ID = 101
```

This identifies Arun.

**Step 2: Find Dept_ID**

```text
Student_ID = 101
        ↓
Dept_ID = D01
```

This tells us which department Arun belongs to.

**Step 3: Find Dept_Name**

```text
Dept_ID = D01
        ↓
Dept_Name = Computer Science
```

Now we know Arun's department name.

The complete dependency is:

```text
Student_ID → Dept_ID → Dept_Name
```

We reached Dept_Name through Dept_ID.

That is the basic idea of transitive dependency.

---

## 5. Why Is This a Problem?

Look at these dependencies again:

```text
Student_ID → Student_Name

Student_ID → Dept_ID

Dept_ID → Dept_Name
```

Our candidate key is:

```text
Student_ID
```

The non-prime attributes are:

```text
Student_Name
Dept_ID
Dept_Name
```

Why are they non-prime?

Because none of these attributes is part of a candidate key in this example.

Now look carefully:

- Student_Name depends on Student_ID.
- Dept_ID depends on Student_ID.
- Dept_Name depends on Dept_ID.

The problem is:

`Dept_Name` depends on `Dept_ID`, which is a non-prime attribute in the original Student table.

And `Dept_ID` itself depends on the candidate key, Student_ID.

Therefore:

```text
Student_ID → Dept_ID → Dept_Name
```

This is the transitive dependency that violates 3NF.

### Important clarification

3NF is NOT about a non-prime attribute depending on multiple primary keys.

We have only one candidate key in this example:

`Student_ID`

The issue is that a non-prime attribute (`Dept_Name`) depends on another non-prime attribute (`Dept_ID`), rather than being directly determined by a candidate key.

---

## 6. The Most Important Question: Why Is Dept_ID Not a Candidate Key?

You might wonder:

"Wait! Isn't Dept_ID a primary key in the Department table?"

Yes, it is. But we must check which table we are talking about.

In the original Student table:

| Student_ID | Student_Name | Dept_ID | Dept_Name |
|---|---|---|---|
| 101 | Arun | D01 | Computer Science |
| 102 | Ravi | D02 | Mechanical |
| 103 | Kumar | D01 | Computer Science |
| 104 | Bala | D03 | Electrical |

Look at Dept_ID:

```text
D01
D02
D01
D03
```

D01 appears twice.

Therefore, Dept_ID cannot uniquely identify every row in the original Student table.

So, in the original Student table:

- Student_ID = Candidate key and primary key.
- Dept_ID = Non-prime attribute.
- Dept_ID = Not a candidate key.
- Dept_ID = Not a super key.

**Remember:** Whether an attribute is a candidate key or a foreign key depends on the particular table and its constraints.

---

## 7. How Do We Convert the Table into 3NF?

We separate the department information from the student information.

Why?

Because department details should depend on the department ID, not be unnecessarily repeated for every student.

We create two tables.

### A. Student Table

| Student_ID (PK) | Student_Name | Dept_ID (FK) |
|---|---|---|
| 101 | Arun | D01 |
| 102 | Ravi | D02 |
| 103 | Kumar | D01 |
| 104 | Bala | D03 |

Here:

- Student_ID = Primary key.
- Dept_ID = Foreign key.

### B. Department Table

| Dept_ID (PK) | Dept_Name |
|---|---|
| D01 | Computer Science |
| D02 | Mechanical |
| D03 | Electrical |

Here:

- Dept_ID = Primary key.
- Dept_Name = Non-prime attribute.

### Why do we keep Dept_ID in the Student table?

Because we still need to know which department each student belongs to.

For example:

```text
Arun → D01
Kumar → D01
```

Both students belong to the same department.

The foreign key connects the Student table with the Department table.

---

## 8. Understand Primary Key and Foreign Key Here

The same attribute can have different roles in different tables.

| Table | Attribute | Role |
|---|---|---|
| Student | Student_ID | Primary key |
| Student | Dept_ID | Foreign key |
| Department | Dept_ID | Primary key |
| Department | Dept_Name | Non-prime attribute |

### Why?

In the Student table:

`Dept_ID` contains repeated values, so it cannot uniquely identify students.

In the Department table:

`Dept_ID` contains unique values, so it can uniquely identify departments.

**Important:** A foreign key can contain duplicate values. A primary key cannot contain duplicate values.

---

## 9. What Problems Does 3NF Prevent?

Keeping department information in the original Student table creates unnecessary repetition.

### A. Update Anomaly

Suppose the department name changes:

```text
Computer Science → Computer Science and Engineering
```

In the original table, we must update every student row belonging to D01.

If we forget one row, the database contains inconsistent department names.

After 3NF, we update the department name in only one place: the Department table.

### B. Insertion Anomaly

Suppose we want to add a new department:

```text
D04 → Civil Engineering
```

But no student has joined that department yet.

In the original Student table, we cannot conveniently store this department without also having a student row.

After 3NF, we can add the department directly to the Department table.

### C. Deletion Anomaly

Suppose Kumar is the only student in a particular department.

If we delete Kumar's row from the original Student table, we may accidentally lose that department's information too.

After 3NF, department information is stored separately, so deleting a student does not delete the department.

---

## 10. How Do We Get the Department Name After Splitting the Tables?

You might ask:

"If Dept_Name is removed from the Student table, how do I find Arun's department name?"

We use a **JOIN**.

```sql
SELECT
    Student.Student_Name,
    Department.Dept_Name
FROM Student
JOIN Department
    ON Student.Dept_ID = Department.Dept_ID;
```

Result:

| Student_Name | Dept_Name |
|---|---|
| Arun | Computer Science |
| Ravi | Mechanical |
| Kumar | Computer Science |
| Bala | Electrical |

The foreign key connects the two tables.

So we can still retrieve the department name without storing it repeatedly in the Student table.

---

## 11. Difference Between 2NF and 3NF

| 2NF | 3NF |
|---|---|
| Removes partial dependencies. | Removes transitive dependencies that violate 3NF. |
| Concerned with a non-prime attribute depending on only part of a candidate key. | Concerned with a non-prime attribute depending on a determinant that is not a super key. |
| Relevant when a candidate key is composite. | Can apply even when the candidate key contains only one attribute. |
| Example: A student's name depends only on Student_ID, which is part of a composite key. | Example: Dept_Name depends on Dept_ID, a non-prime attribute in the original Student table. |

### Easy way to remember

**2NF:**

```text
Part of a composite candidate key
                  ↓
          Non-prime attribute
```

This is a partial dependency.

**3NF:**

```text
Candidate key
      ↓
 Non-prime attribute
      ↓
 Another non-prime attribute
```

This illustrates a transitive dependency.

---

## 12. Formal 3NF Rule

For every non-trivial functional dependency:

```text
X → A
```

At least one of these conditions must be true:

1. X is a super key.
2. A is a prime attribute.

In our original Student table:

```text
Dept_ID → Dept_Name
```

Check the conditions:

- Is Dept_ID a super key in the original Student table? **No.**
- Is Dept_Name a prime attribute? **No.**

Therefore, this dependency violates 3NF.

That is why we separate the Department information.

---

## 13. Final Revision – Understand This Completely

Original table:

```text
Student_ID | Student_Name | Dept_ID | Dept_Name
```

Candidate key:

```text
Student_ID
```

Functional dependencies:

```text
Student_ID → Student_Name

Student_ID → Dept_ID

Dept_ID → Dept_Name
```

Transitive dependency:

```text
Student_ID → Dept_ID → Dept_Name
```

Problem:

```text
Dept_ID → Dept_Name
```

Dept_ID is not a super key in the original Student table, and Dept_Name is non-prime.

Solution:

```text
STUDENT TABLE

Student_ID (PK)
Student_Name
Dept_ID (FK)
```

```text
DEPARTMENT TABLE

Dept_ID (PK)
Dept_Name
```

---

## 14. Test Your Understanding

Try answering these questions without looking at the notes.

**Question 1:** In the original Student table, why is Dept_ID not a candidate key?

**Question 2:** Why can Dept_ID be a primary key in the Department table?

**Question 3:** Which dependency creates the 3NF violation?

**Question 4:** Why is Student_ID → Dept_ID → Dept_Name called a transitive dependency?

**Question 5:** After converting to 3NF, why do we still keep Dept_ID in the Student table?

**Question 6:** If I give you Dept_ID = D01, can you find Dept_Name without knowing Student_ID?

If you can explain all six answers in your own words, you understand the central idea behind 3NF.

# 3NF – Simple Notes (Transitive Dependency)

## 1. Main Idea

3NF removes **transitive dependency**.

In simple words:

A non-prime attribute should not depend on another non-prime attribute when that dependency violates 3NF.

## 2. Example – Original Student Table

| Student_ID (PK) | Student_Name | Dept_ID | Dept_Name |
|---|---|---|---|
| 101 | Arun | D01 | Computer Science |
| 102 | Ravi | D02 | Mechanical |
| 103 | Kumar | D01 | Computer Science |

## 3. Functional Dependencies

```text
Student_ID → Student_Name

Student_ID → Dept_ID

Dept_ID → Dept_Name
```

## 4. Understand the Problem

Suppose we want Arun's department name.

```text
Student_ID = 101
       ↓
Dept_ID = D01
       ↓
Dept_Name = Computer Science
```

Notice:

We use `Dept_ID` to find `Dept_Name`.

We don't need `Student_ID` to determine the department name once we know `Dept_ID`.

Therefore:

```text
Student_ID → Dept_ID → Dept_Name
```

This is called **transitive dependency**.

## 5. Why Is It a Problem?

- `Student_ID` is the primary key.
- `Dept_ID` is a non-prime attribute in the original table.
- `Dept_Name` is a non-prime attribute.
- `Dept_ID` determines `Dept_Name`.

This creates unnecessary repetition of department information in the Student table.

## 6. Solution – Convert into 3NF

Split the original table into two tables.

### Student Table

| Student_ID (PK) | Student_Name | Dept_ID (FK) |
|---|---|---|
| 101 | Arun | D01 |
| 102 | Ravi | D02 |
| 103 | Kumar | D01 |

### Department Table

| Dept_ID (PK) | Dept_Name |
|---|---|
| D01 | Computer Science |
| D02 | Mechanical |

## 7. Why Do We Keep Dept_ID in Both Tables?

- In the Student table, `Dept_ID` is a **foreign key**.
- In the Department table, `Dept_ID` is a **primary key**.

We keep Dept_ID in the Student table to identify which department each student belongs to.

We keep Dept_ID in the Department table to identify the department name.

## 8. Final Understanding

**Before 3NF:**

```text
Student_ID → Dept_ID → Dept_Name
```

**After 3NF:**

```text
STUDENT TABLE:

Student_ID → Student_Name
Student_ID → Dept_ID
```

```text
DEPARTMENT TABLE:

Dept_ID → Dept_Name
```

**Remember:**

3NF separates information when a non-prime attribute depends on another non-prime attribute in a way that violates 3NF.

The goal is to store department details in the Department table instead of repeating them for every student.
**End of 3NF Notes.**
