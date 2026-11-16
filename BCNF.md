# BCNF (Boyce-Codd Normal Form) – Detailed Notes

## 1. What is BCNF?

**BCNF (Boyce-Codd Normal Form)** is a database normalization rule that is stricter than 3NF.

In simple words:

> Whenever one attribute or a combination of attributes determines another attribute, the determining attribute(s) must be a super key.

### Remember the main rule:

For every non-trivial functional dependency:

```text
X → Y
```

**X must be a super key.**

This is the most important rule in BCNF.

---

## 2. Understand BCNF Using an Example

Suppose we have a table:

### Student_Course Table

| Student | Course | Instructor |
|---|---|---|
| Arun | Java | Ravi |
| Kumar | Python | Bala |
| Arun | Python | Bala |
| Ravi | Java | Ravi |

Assume these business rules:

1. A student can take multiple courses.
2. A course can have multiple students.
3. Each instructor teaches only one course.
4. A course can have multiple instructors.

Our table stores which instructor teaches which course to each student.

### Understand the columns

- `Student` = Student's name.
- `Course` = Course taken by the student.
- `Instructor` = Instructor teaching that course to the student.

---

## 3. Understand the Functional Dependencies

A functional dependency means that one attribute or a combination of attributes determines another attribute.

In our example, we have two important functional dependencies.

### Dependency 1: Student + Course → Instructor

```text
(Student, Course) → Instructor
```

What does this mean?

If we know the student and the course, we can identify the instructor assigned to that student for that course.

Example:

```text
Arun + Java → Ravi
Arun + Python → Bala
```

Therefore:

```text
(Student, Course) → Instructor
```

### Dependency 2: Instructor → Course

```text
Instructor → Course
```

What does this mean?

Each instructor teaches only one course.

Therefore, if we know the instructor, we can identify the course that instructor teaches.

Example:

```text
Ravi → Java
Bala → Python
```

We cannot have Ravi teaching Java in one row and Python in another row because our assumption says each instructor teaches only one course.

---

## 4. Find the Candidate Keys

This is important because we need to identify candidate keys before checking BCNF.

Remember:

**A candidate key is a minimal combination of attributes that uniquely identifies every row.**

Our functional dependencies are:

```text
(Student, Course) → Instructor

Instructor → Course
```

Let's find the candidate keys.

### Candidate Key 1: Student + Course

We know:

```text
(Student, Course) → Instructor
```

Therefore, Student + Course determines all three attributes:

```text
Student + Course → Instructor
```

So:

```text
(Student, Course)
```

is a candidate key.

Why?

Because it uniquely identifies every row, and neither Student nor Course alone is sufficient under our assumptions.

### Candidate Key 2: Student + Instructor

We also know:

```text
Instructor → Course
```

If we know Student + Instructor, we can determine Course.

Step by step:

```text
Student + Instructor
          |
          ↓
       Course
```

Therefore, Student + Instructor determines all three attributes.

So:

```text
(Student, Instructor)
```

is also a candidate key.

### Final candidate keys

```text
(Student, Course)

(Student, Instructor)
```

**Important:** A table can have more than one candidate key. The primary key is the candidate key chosen to identify rows.

---

## 5. Identify Prime and Non-Prime Attributes

Remember:

- **Prime attribute:** An attribute that belongs to at least one candidate key.
- **Non-prime attribute:** An attribute that does not belong to any candidate key.

Our candidate keys are:

```text
(Student, Course)

(Student, Instructor)
```

Let's check each attribute.

| Attribute | Belongs to a candidate key? | Type |
|---|---|---|
| Student | Yes | Prime |
| Course | Yes | Prime |
| Instructor | Yes | Prime |

All three attributes are prime attributes.

Why?

- Student belongs to both candidate keys.
- Course belongs to the first candidate key.
- Instructor belongs to the second candidate key.

Therefore, this table has no non-prime attributes.

---

## 6. Check Whether the Table Satisfies BCNF

Now we apply the BCNF rule.

Remember:

> For every non-trivial functional dependency, the left-hand side must be a super key.

Our dependencies:

```text
(Student, Course) → Instructor

Instructor → Course
```

We must check both dependencies separately.

### A. Check (Student, Course) → Instructor

Question:

Is Student + Course a super key?

Yes.

Because Student + Course uniquely identifies every row.

Therefore:

```text
(Student, Course) → Instructor
```

**Satisfies BCNF.**

### B. Check Instructor → Course

Question:

Is Instructor alone a super key?

No.

Why?

Because the same instructor can teach multiple students.

For example:

| Student | Course | Instructor |
|---|---|---|
| Arun | Java | Ravi |
| Ravi | Java | Ravi |

Here, Instructor = Ravi appears in multiple rows.

Therefore, Instructor alone cannot uniquely identify every row.

So:

```text
Instructor → Course
```

**Violates BCNF.**

### Final result

| Functional dependency | Is the left side a super key? | BCNF |
|---|---|---|
| (Student, Course) → Instructor | Yes | Satisfied |
| Instructor → Course | No | Violated |

Even though the first dependency satisfies BCNF, the second dependency violates it.

Therefore, the entire table is **not in BCNF**.

---

## 7. But Wait! Doesn't This Table Already Satisfy 3NF?

Yes!

This is the most important difference between 3NF and BCNF.

Let's remember the formal 3NF rule.

For every non-trivial functional dependency:

```text
X → Y
```

At least one of these conditions must be true:

1. X is a super key.
2. Y is a prime attribute.

Now check:

```text
Instructor → Course
```

Condition 1:

Is Instructor a super key?

**No.**

Condition 2:

Is Course a prime attribute?

**Yes.**

Course belongs to the candidate key `(Student, Course)`.

Therefore, this dependency satisfies 3NF.

But it violates BCNF because Instructor is not a super key.

### Why is this important?

3NF allows an exception:

The right-hand side can be a prime attribute even when the left-hand side is not a super key.

BCNF does not allow this exception.

---

## 8. How Do We Convert the Table into BCNF?

We split the original table into two tables.

Why?

Because this dependency violates BCNF:

```text
Instructor → Course
```

We need a separate table where Instructor can uniquely identify the course.

### Table 1: Instructor_Course

| Instructor (PK) | Course |
|---|---|
| Ravi | Java |
| Bala | Python |

Functional dependency:

```text
Instructor → Course
```

Here:

- Instructor = Primary key.
- Course = Non-prime attribute.

Why is Instructor the primary key?

Because each instructor teaches only one course, according to our assumption.

Therefore, Instructor uniquely identifies each row in this table.

### Table 2: Student_Instructor

| Student (PK part) | Instructor (PK part, FK) |
|---|---|
| Arun | Ravi |
| Kumar | Bala |
| Arun | Bala |
| Ravi | Ravi |

Candidate key:

```text
(Student, Instructor)
```

Here:

- Student + Instructor = Composite primary key.
- Instructor = Foreign key referencing Instructor_Course.

This table tells us which instructor is associated with each student.

### Final structure

```text
INSTRUCTOR_COURSE

Instructor (PK)
Course
```

```text
STUDENT_INSTRUCTOR

Student
Instructor (FK)

Composite PK: Student + Instructor
```

---

## 9. Why Do We Need Two Tables?

Look at the original table:

| Student | Course | Instructor |
|---|---|---|
| Arun | Java | Ravi |
| Kumar | Python | Bala |
| Arun | Python | Bala |
| Ravi | Java | Ravi |

Notice that Ravi teaches Java.

If Ravi teaches Java to 100 students, the same instructor-course information is repeated 100 times.

This creates unnecessary repetition.

After splitting:

### Instructor_Course

| Instructor | Course |
|---|---|
| Ravi | Java |
| Bala | Python |

We store the instructor's course only once.

### Student_Instructor

| Student | Instructor |
|---|---|
| Arun | Ravi |
| Kumar | Bala |
| Arun | Bala |
| Ravi | Ravi |

We only store which instructor is associated with each student.

---

## 10. What Problems Does BCNF Prevent?

### A. Update Anomaly

Suppose Ravi changes from teaching Java to another course.

In the original table, we would need to update every row containing Ravi.

If we forget one row, inconsistent information appears.

After BCNF, we update Ravi's course in only one place: Instructor_Course.

### B. Insertion Anomaly

Suppose a new instructor joins:

```text
Instructor = Kumar
Course = Java
```

But no student has been assigned to Kumar yet.

In the original table, storing the instructor-course relationship independently is difficult.

After BCNF, we can insert Kumar directly into Instructor_Course.

### C. Deletion Anomaly

Suppose Ravi is teaching Java to only one student.

If we delete that student's row from the original table, we also lose the information that Ravi teaches Java.

After BCNF, the instructor-course information is stored separately.

Deleting a student-instructor relationship does not delete the instructor's course information.

---

## 11. Difference Between 3NF and BCNF

| 3NF | BCNF |
|---|---|
| Less strict than BCNF. | Stricter than 3NF. |
| The left side must be a super key OR the right side must be a prime attribute. | The left side must always be a super key. |
| Allows certain dependencies where the determinant is not a super key. | Does not allow those dependencies. |
| Some 3NF tables may violate BCNF. | Every BCNF table satisfies 3NF. |

### Easy example to remember

Our dependency:

```text
Instructor → Course
```

For 3NF:

- Instructor is not a super key.
- Course is a prime attribute.

Therefore, it satisfies 3NF.

For BCNF:

- Instructor is not a super key.

Therefore, it violates BCNF.

---

## 12. Formal BCNF Rule

For every non-trivial functional dependency:

```text
X → Y
```

The determinant X must be a super key.

**Non-trivial dependency:** A dependency where the right-hand side contains at least one attribute that is not already part of the left-hand side.

For example:

```text
Instructor → Course
```

This is non-trivial because Course is not part of Instructor.

Now check:

```text
Instructor → Course
```

Is Instructor a super key in the original table?

No.

Therefore, the original table violates BCNF.

---

## 13. Important Things to Remember

1. BCNF is stricter than 3NF.
2. Every BCNF table satisfies 3NF.
3. A table can satisfy 3NF but violate BCNF.
4. A super key uniquely identifies every row.
5. A candidate key is a minimal super key.
6. A prime attribute belongs to at least one candidate key.
7. BCNF does not have the prime-attribute exception that 3NF has.
8. A table can have multiple candidate keys.
9. When a functional dependency violates BCNF, we may decompose the table.
10. After decomposition, foreign keys can preserve relationships between tables.

---

## 14. Final Revision

### Original table

```text
Student | Course | Instructor
```

### Functional dependencies

```text
(Student, Course) → Instructor

Instructor → Course
```

### Candidate keys

```text
(Student, Course)

(Student, Instructor)
```

### Prime attributes

```text
Student
Course
Instructor
```

All three attributes are prime.

### BCNF violation

```text
Instructor → Course
```

Reason:

```text
Instructor is NOT a super key
```

### Why it still satisfies 3NF

```text
Course is a prime attribute
```

### Solution

Split into:

```text
INSTRUCTOR_COURSE

Instructor (PK)
Course
```

```text
STUDENT_INSTRUCTOR

Student
Instructor (FK)

Composite PK: Student + Instructor
```

---

## 15. Test Your Understanding

Try answering these questions without looking at the notes.

1. What is BCNF?
2. What is the main difference between 3NF and BCNF?
3. What are the two functional dependencies in our example?
4. Why is `(Student, Course)` a candidate key?
5. Why is `(Student, Instructor)` also a candidate key?
6. Why are all three attributes prime?
7. Why does `Instructor → Course` violate BCNF?
8. Why does the same dependency satisfy 3NF?
9. Why do we split the original table?
10. Why is Instructor a primary key in the Instructor_Course table?
11. Why is Instructor a foreign key in the Student_Instructor table?

**If you can explain why `Instructor → Course` satisfies 3NF but violates BCNF, you have understood the central idea of BCNF.**
