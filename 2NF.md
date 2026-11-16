
# Second Normal Form (2NF) – Complete Notes

## 1. What is 2NF?

2NF stands for Second Normal Form.

A table is in 2NF when:

1. It is already in First Normal Form (1NF).
2. It has no partial dependency.

The main purpose of 2NF is to ensure that non-key attributes depend on the entire candidate key, not just a part of a composite candidate key.

### In simple words

Imagine a table whose primary key consists of two columns.

If another column needs only ONE of those two columns to determine its value, we have a partial dependency.

2NF helps us remove that problem by separating the information into logically related tables.

---

## 2. First Understand Why We Need 2NF

Imagine a college maintains student enrollment information in one table.

### Student_Course Table

| Student_ID | Course_ID | Student_Name | Course_Name | Marks |
|---|---|---|---|---|
| 101 | C01 | Arun | Java | 85 |
| 101 | C02 | Arun | Python | 90 |
| 102 | C01 | Ravi | Java | 75 |
| 103 | C01 | Kumar | Java | 80 |

Understand what each row represents.

- Student 101 enrolled in Java.
- Student 101 enrolled in Python.
- Student 102 enrolled in Java.
- Student 103 enrolled in Java.

**One row represents one student's enrollment in one course.**

This is important because we are not storing only student information or only course information. We are storing the relationship between students and courses.

---

## 3. Understand the Primary Key

Before applying 2NF, we must understand how to identify one particular enrollment record.

Look at the table:

| Student_ID | Course_ID | Student_Name | Course_Name | Marks |
|---|---|---|---|---|
| 101 | C01 | Arun | Java | 85 |
| 101 | C02 | Arun | Python | 90 |
| 102 | C01 | Ravi | Java | 75 |
| 103 | C01 | Kumar | Java | 80 |

### A. Can Student_ID alone identify one enrollment row?

Suppose we search:

`Student_ID = 101`

We get:

| Student_ID | Course_ID | Student_Name | Course_Name | Marks |
|---|---|---|---|---|
| 101 | C01 | Arun | Java | 85 |
| 101 | C02 | Arun | Python | 90 |

We get two rows.

Why?

Because the same student can enrol in multiple courses.

Therefore, Student_ID alone cannot uniquely identify one enrollment record.

**Important distinction:**

- Student_ID can identify the student.
- But Student_ID cannot identify one particular enrollment when a student has multiple enrollments.

These are two different things.

### B. Can Course_ID alone identify one enrollment row?

Suppose we search:

`Course_ID = C01`

We get:

| Student_ID | Course_ID | Student_Name | Course_Name | Marks |
|---|---|---|---|---|
| 101 | C01 | Arun | Java | 85 |
| 102 | C01 | Ravi | Java | 75 |
| 103 | C01 | Kumar | Java | 80 |

We get three rows.

Why?

Because multiple students can enrol in the same course.

Therefore, Course_ID alone cannot uniquely identify one enrollment record.

### C. What happens when we combine both?

Suppose we search:

`Student_ID = 101 AND Course_ID = C02`

We get:

| Student_ID | Course_ID | Student_Name | Course_Name | Marks |
|---|---|---|---|---|
| 101 | C02 | Arun | Python | 90 |

Now we get exactly one enrollment record.

Therefore:

`Student_ID + Course_ID`

Together identify one enrollment record.

This is called a **Composite Primary Key**.

### Important

A composite primary key is ONE primary key made using two or more columns.

It is NOT two separate primary keys.

```sql
PRIMARY KEY (Student_ID, Course_ID)
```

### Remember the difference

| What do we want to identify? | Required column |
|---|---|
| A student | Student_ID |
| A course | Course_ID |
| One particular enrollment | Student_ID + Course_ID |

**Why do we combine columns?**

We combine columns when neither column alone uniquely identifies a record, but their combination does.

We do not combine columns merely to extract information. SQL queries can retrieve whichever columns and rows we need.

---

## 4. Understand the Actual Problem: Partial Dependency

Now we have established that our composite primary key is:

`(Student_ID, Course_ID)`

Look at the other columns:

| Student_ID | Course_ID | Student_Name | Course_Name | Marks |
|---|---|---|---|---|
| 101 | C01 | Arun | Java | 85 |
| 101 | C02 | Arun | Python | 90 |
| 102 | C01 | Ravi | Java | 75 |

Ask yourself these questions.

### A. What do we need to find Student_Name?

Suppose I give you:

`Student_ID = 101`

Can you determine the student's name?

Yes. It is Arun.

Do you need Course_ID to find the student's name?

No.

Therefore:

`Student_ID → Student_Name`

Student_Name depends only on Student_ID.

### B. What do we need to find Course_Name?

Suppose I give you:

`Course_ID = C01`

Can you determine the course name?

Yes. It is Java.

Do you need Student_ID to find the course name?

No.

Therefore:

`Course_ID → Course_Name`

Course_Name depends only on Course_ID.

### C. What do we need to find Marks?

Suppose I give you:

`Student_ID = 101`

Can you determine the marks?

No.

Why?

Because Arun has different marks in different courses.

| Student_ID | Course_ID | Marks |
|---|---|---|
| 101 | C01 | 85 |
| 101 | C02 | 90 |

Student_ID alone is not enough.

Suppose I give you:

`Course_ID = C01`

Can you determine the marks?

No.

Why?

Because different students get different marks in Java.

| Student_ID | Course_ID | Marks |
|---|---|---|
| 101 | C01 | 85 |
| 102 | C01 | 75 |

Therefore, we need BOTH Student_ID and Course_ID to determine Marks.

`(Student_ID, Course_ID) → Marks`

### Summary of Dependencies

| Attribute | What determines its value? |
|---|---|
| Student_Name | Student_ID |
| Course_Name | Course_ID |
| Marks | Student_ID + Course_ID |

Notice the difference.

- Student_Name depends only on Student_ID.
- Course_Name depends only on Course_ID.
- Marks depends on the complete composite key.

This is the central concept of 2NF.

---

## 5. What Exactly Is Partial Dependency?

Partial dependency occurs when a non-prime attribute depends on only a proper subset of a candidate key.

In our example:

Complete candidate key:

`(Student_ID, Course_ID)`

But:

```text
Student_ID → Student_Name
```

Student_Name depends on only one part of the composite key.

Similarly:

```text
Course_ID → Course_Name
```

Course_Name also depends on only one part of the composite key.

These are partial dependencies.

### Visual Understanding

```text
Complete Composite Key
(Student_ID, Course_ID)
          |
          |
          +--------------------> Marks
          |
          |
          +-- Student_ID ------> Student_Name
          |
          +-- Course_ID -------> Course_Name
```

The first two dependencies use only part of the composite key.

That is why the original table violates 2NF.

**Remember:**

Partial dependency is possible when a candidate key contains multiple columns.

If all candidate keys are single-column keys, partial dependency cannot exist.

---

## 6. How Do We Convert the Table Into 2NF?

We separate the information based on what determines it.

### Original Table

| Student_ID | Course_ID | Student_Name | Course_Name | Marks |
|---|---|---|---|---|
| 101 | C01 | Arun | Java | 85 |
| 101 | C02 | Arun | Python | 90 |
| 102 | C01 | Ravi | Java | 75 |
| 103 | C01 | Kumar | Java | 80 |

We identified these dependencies:

```text
Student_ID → Student_Name

Course_ID → Course_Name

(Student_ID, Course_ID) → Marks
```

Now separate them.

### Table 1: Student

| Student_ID (PK) | Student_Name |
|---|---|
| 101 | Arun |
| 102 | Ravi |
| 103 | Kumar |

Why is Student_Name here?

Because Student_ID alone determines Student_Name.

We only need to store each student's name once.

### Table 2: Course

| Course_ID (PK) | Course_Name |
|---|---|
| C01 | Java |
| C02 | Python |

Why is Course_Name here?

Because Course_ID alone determines Course_Name.

We only need to store each course name once.

### Table 3: Student_Course

| Student_ID (PK, FK) | Course_ID (PK, FK) | Marks |
|---|---|---|
| 101 | C01 | 85 |
| 101 | C02 | 90 |
| 102 | C01 | 75 |
| 103 | C01 | 80 |

Why do we keep both Student_ID and Course_ID here?

Because this table stores enrollment records.

One student can enrol in multiple courses.

One course can have multiple students.

Therefore, both columns together identify one particular enrollment record.

The combination remains the composite primary key.

Marks also stays here because marks belong to a particular student's performance in a particular course.

---

## 7. Understand PK and FK in the Three Tables

### Student Table

```text
Student_ID = Primary Key
```

Student_ID uniquely identifies each student.

### Course Table

```text
Course_ID = Primary Key
```

Course_ID uniquely identifies each course.

### Student_Course Table

```text
Student_ID = Primary Key + Foreign Key

Course_ID = Primary Key + Foreign Key
```

Together:

```text
PRIMARY KEY (Student_ID, Course_ID)
```

Individually, both columns are foreign keys referencing their respective parent tables.

Together, they form ONE composite primary key.

### Relationship

```text
STUDENT
-----------------
Student_ID (PK)
Student_Name
       |
       |
       | Student_ID (FK)
       |
       v
STUDENT_COURSE
-----------------
Student_ID (PK, FK)
Course_ID  (PK, FK)
Marks
       ^
       |
       | Course_ID (FK)
       |
       |
COURSE
-----------------
Course_ID (PK)
Course_Name
```

The Student_Course table connects the Student and Course tables.

---

## 8. Why Does This Separation Matter?

Let's understand what problems we avoid.

### A. Update Anomaly

Suppose Arun's name changes.

In the original table, we may need to update multiple rows.

After separation, we update only the Student table.

### B. Insertion Anomaly

Suppose the college introduces a new course called C++.

No student has enrolled yet.

In the original enrollment table, inserting course information without student enrollment is difficult.

After separation, we can insert C++ directly into the Course table.

### C. Deletion Anomaly

Suppose the only student enrolled in Python leaves.

If we delete that enrollment row, we might accidentally lose the only information about Python.

After separation, deleting an enrollment does not delete the course information.

---

## 9. Important Rules to Remember

| Rule | Explanation |
|---|---|
| 1NF is compulsory | A table must already satisfy 1NF before it can satisfy 2NF. |
| No partial dependency | A non-prime attribute must not depend on only part of a candidate key. |
| Composite keys matter | Partial dependency is possible when a candidate key contains multiple attributes. |
| Single-column candidate keys | A table with only single-column candidate keys automatically satisfies 2NF if it is in 1NF. |
| Keep the relationship | When splitting tables, preserve the necessary keys and relationships. |
| Move attributes based on dependencies | Do not blindly split tables just because they contain multiple columns. |

### Important Technical Note

2NF is checked against ALL candidate keys, not only the declared primary key.

A non-prime attribute is an attribute that does not belong to any candidate key.

---

## 10. SQL Example

Create the Student table:

```sql
CREATE TABLE Student (
    Student_ID INT PRIMARY KEY,
    Student_Name VARCHAR(100)
);
```

Create the Course table:

```sql
CREATE TABLE Course (
    Course_ID VARCHAR(10) PRIMARY KEY,
    Course_Name VARCHAR(100)
);
```

Create the Student_Course table:

```sql
CREATE TABLE Student_Course (
    Student_ID INT,
    Course_ID VARCHAR(10),
    Marks INT,

    PRIMARY KEY (Student_ID, Course_ID),

    FOREIGN KEY (Student_ID)
        REFERENCES Student(Student_ID),

    FOREIGN KEY (Course_ID)
        REFERENCES Course(Course_ID)
);
```

Insert student details:

```sql
INSERT INTO Student
VALUES
(101, 'Arun'),
(102, 'Ravi'),
(103, 'Kumar');
```

Insert course details:

```sql
INSERT INTO Course
VALUES
('C01', 'Java'),
('C02', 'Python');
```

Insert enrollment details:

```sql
INSERT INTO Student_Course
VALUES
(101, 'C01', 85),
(101, 'C02', 90),
(102, 'C01', 75),
(103, 'C01', 80);
```

### Retrieve a student's courses and marks

```sql
SELECT
    S.Student_Name,
    C.Course_Name,
    SC.Marks
FROM Student_Course SC
JOIN Student S
    ON SC.Student_ID = S.Student_ID
JOIN Course C
    ON SC.Course_ID = C.Course_ID
WHERE S.Student_ID = 101;
```

Result:

| Student_Name | Course_Name | Marks |
|---|---|---|
| Arun | Java | 85 |
| Arun | Python | 90 |

Notice that the information is stored in separate tables, but we can combine it whenever we need it using JOIN.

**Separating tables does not mean losing the ability to retrieve the complete information.**

---

## 11. Common Confusions

### Confusion 1: Does a composite primary key mean two primary keys?

No.

It is ONE primary key consisting of multiple columns.

```sql
PRIMARY KEY (Student_ID, Course_ID)
```

### Confusion 2: Why can't Student_ID alone be the primary key in Student_Course?

Because one student can have multiple enrollment records.

Student_ID = 101 appears more than once.

### Confusion 3: Why can't Course_ID alone be the primary key in Student_Course?

Because multiple students can enrol in the same course.

Course_ID = C01 appears more than once.

### Confusion 4: Are we combining columns just to extract information?

No.

SQL can retrieve information using any appropriate WHERE condition.

We combine columns as a composite key because their combination uniquely identifies one record.

### Confusion 5: Does 2NF mean every table must have three tables?

No.

The number of tables depends on the data and its dependencies.

Sometimes no decomposition is necessary.

Sometimes multiple tables are needed.

2NF is about removing partial dependencies, not creating a fixed number of tables.

---

## 12. Final Summary

### Remember This Example

Original table:

```text
Student_ID
Course_ID
Student_Name
Course_Name
Marks
```

Composite candidate key:

```text
(Student_ID, Course_ID)
```

Dependencies:

```text
Student_ID → Student_Name

Course_ID → Course_Name

(Student_ID, Course_ID) → Marks
```

Problem:

```text
Student_Name depends only on Student_ID.

Course_Name depends only on Course_ID.
```

These are partial dependencies.

Solution:

```text
STUDENT
Student_ID (PK)
Student_Name


COURSE
Course_ID (PK)
Course_Name


STUDENT_COURSE
Student_ID (PK, FK)
Course_ID (PK, FK)
Marks
```

### The One-Line Definition

**2NF ensures that every non-prime attribute depends on the whole candidate key, not just a part of a composite candidate key.**

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| 2NF | 1NF + no partial dependency |
| Partial dependency | An attribute depends on only part of a composite candidate key |
| Composite primary key | One key made from two or more columns |
| Student_ID → Student_Name | Student name depends on student ID |
| Course_ID → Course_Name | Course name depends on course ID |
| Student_ID + Course_ID → Marks | Marks depend on the particular student-course combination |
| Decomposition | Separating tables based on their dependencies |
| JOIN | Reconnecting separated tables when we need combined information |

**End of 2NF Notes.**
