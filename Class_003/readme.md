# DATABASE NORMALIZATION AND RELATIONSHIPS IN SQL SERVER

# 1. What is Database Normalization?

Database normalization is a way of organizing data in a database so that the same information is not unnecessarily stored again and again. Instead of keeping all information in one large table, we divide the data into smaller, meaningful tables and connect those tables when needed. This makes the database easier to maintain, update, and manage.

For example, instead of storing student, course, and instructor information repeatedly in one table, we can keep students in one table, courses in another table, instructors in another table, and enrollment information in a separate table.

The main idea is:

```text
Store each type of information in the right place.
```

---

# 2. Why Do We Need Normalization?

Suppose we create a college database.

We need to store:

- Students
- Courses
- Instructors
- Enrollments

A simple table might look like this:

| Student_ID | Student_Name | Course | Instructor |
|---|---|---|---|
| 1 | Ali | SQL | Kamran |
| 1 | Ali | Python | Ahmed |
| 2 | Sara | SQL | Kamran |
| 3 | Hamza | SQL | Kamran |

This table creates several problems.

### Duplicate Data

`Ali` is stored multiple times.

`SQL` is stored multiple times.

`Kamran` is stored multiple times.

If 500 students take SQL, the instructor's name may be stored 500 times.

### Update Problem

Suppose Kamran's name needs to be changed.

We may need to update many rows.

If one row is missed, the database can contain different values for the same instructor.

### Insert Problem

Suppose we add a new course:

```text
Java
```

but no student has enrolled yet.

Where should we store the course?

Our table is designed around students and their enrollments, so inserting a course without a student becomes difficult.

### Delete Problem

Suppose a course currently has only one student.

If we delete that student's enrollment, we may also lose the only record containing information about that course.

These problems happen because different types of information are mixed together.

Normalization helps us solve these problems.

---

# 3. Types of Normalization

Normalization is divided into different normal forms.

In this lecture, we will cover the three most commonly used forms:

```text
1NF → First Normal Form

2NF → Second Normal Form

3NF → Third Normal Form
```

Each normal form solves a different type of problem.

```text
1NF → Fix multiple values

2NF → Fix partial dependency

3NF → Fix transitive dependency
```

---

# 4. First Normal Form — 1NF

## Definition

First Normal Form means that each column should contain atomic values, meaning one value in each cell. A column should not contain multiple values together, and repeating groups should be removed from the table.

In simple words:

> **One cell should contain one value.**

## Rule

```text
1NF = One Cell → One Value
```

---

# Problem Before 1NF

Suppose we create this table:

| Student_ID | Student_Name | Courses |
|---|---|---|
| 1 | Ali | SQL, Python |
| 2 | Sara | SQL, Web |
| 3 | Hamza | Database, Java |

The problem is in the `Courses` column.

For Ali:

```text
SQL, Python
```

Two values are stored in one cell.

For Sara:

```text
SQL, Web
```

Again, two values are stored in one cell.

This makes the data difficult to search, update, and manage properly.

For example, if we want to find students who are taking Python, we have to search inside a text value containing multiple courses.

This table is not properly organized according to 1NF.

---

# Solution — Convert the Table into 1NF

Instead of storing multiple courses in one cell, we create separate rows.

### Before

| Student_ID | Student_Name | Courses |
|---|---|---|
| 1 | Ali | SQL, Python |
| 2 | Sara | SQL, Web |

### After

| Student_ID | Student_Name | Course |
|---|---|---|
| 1 | Ali | SQL |
| 1 | Ali | Python |
| 2 | Sara | SQL |
| 2 | Sara | Web |

Now every cell contains only one value.

```text
SQL, Python
```

became:

```text
SQL
Python
```

---

# 1NF Practical Example in SQL Server

```sql
CREATE TABLE StudentCourses_1NF
(
    Student_ID INT,
    Student_Name VARCHAR(100),
    Course VARCHAR(100)
);
```

Insert the data:

```sql
INSERT INTO StudentCourses_1NF
VALUES
(1, 'Ali', 'SQL'),
(1, 'Ali', 'Python'),
(2, 'Sara', 'SQL'),
(2, 'Sara', 'Web');
```

Check the data:

```sql
SELECT * 
FROM StudentCourses_1NF;
```

Now every row contains one course value.

That is the basic idea of **1NF**.

---

# 5. Second Normal Form — 2NF

## Definition

Second Normal Form means that the table must already be in 1NF, and every non-key column must depend on the complete primary key, not just a part of it. This problem mainly occurs when a table has a composite key, meaning a key made from two or more columns.

In simple words:

> **A non-key column should depend on the complete key.**

## Rule

```text
2NF = Remove Partial Dependency
```

---

# Problem Before 2NF

Suppose our table is already in 1NF:

| Student_ID | Course_ID | Student_Name | Course_Name |
|---|---|---|---|
| 1 | 101 | Ali | SQL |
| 1 | 102 | Ali | Python |
| 2 | 101 | Sara | SQL |
| 3 | 103 | Hamza | Database |

Suppose:

```text
Student_ID + Course_ID
```

together identify one enrollment.

So our composite key is:

```text
Student_ID + Course_ID
```

Now look at `Student_Name`.

If:

```text
Student_ID = 1
```

we already know:

```text
Student_Name = Ali
```

We do not need `Course_ID`.

Therefore:

```text
Student_ID → Student_Name
```

Now look at `Course_Name`.

If:

```text
Course_ID = 101
```

we already know:

```text
Course_Name = SQL
```

We do not need `Student_ID`.

Therefore:

```text
Course_ID → Course_Name
```

The problem is that:

```text
Student_Name
```

depends only on part of the composite key:

```text
Student_ID
```

and:

```text
Course_Name
```

depends only on:

```text
Course_ID
```

They do not depend on the complete key:

```text
Student_ID + Course_ID
```

This is called **Partial Dependency**.

---

# Solution — Convert the Table into 2NF

We separate student information, course information, and enrollment information.

### Students

| Student_ID | Student_Name |
|---|---|
| 1 | Ali |
| 2 | Sara |
| 3 | Hamza |

### Courses

| Course_ID | Course_Name |
|---|---|
| 101 | SQL |
| 102 | Python |
| 103 | Database |

### Enrollments

| Student_ID | Course_ID |
|---|---|
| 1 | 101 |
| 1 | 102 |
| 2 | 101 |
| 3 | 103 |

Now:

```text
Student information
        ↓
Students table

Course information
        ↓
Courses table

Enrollment information
        ↓
Enrollments table
```

The partial dependency has been removed.

---

# 2NF Practical Example in SQL Server

Create the Students table:

```sql
CREATE TABLE Students
(
    Student_ID INT PRIMARY KEY,
    Student_Name VARCHAR(100)
);
```

Create the Courses table:

```sql
CREATE TABLE Courses
(
    Course_ID INT PRIMARY KEY,
    Course_Name VARCHAR(100)
);
```

At this point, we have separate tables.

But now we have an important problem:

```text
How do we connect these tables?
```

This is where **Foreign Keys** become necessary.

---


# 7. Foreign Key

A Foreign Key is a column in one table that refers to the Primary Key of another table. It is used to create a connection between related tables and helps maintain valid relationships between the data.

---

# Foreign Key Syntax

The general syntax is:

```sql
FOREIGN KEY (column_name)
REFERENCES ParentTable(primary_key_column)
```
---

# 2NF SQL Example

```sql
CREATE TABLE Students
(
    Student_ID INT PRIMARY KEY,
    Student_Name VARCHAR(100)
);
```

```sql
CREATE TABLE Courses
(
    Course_ID INT PRIMARY KEY,
    Course_Name VARCHAR(100)
);
```

Now create the Enrollment table:

```sql
CREATE TABLE Enrollments
(
    Student_ID INT,
    Course_ID INT,

    PRIMARY KEY (Student_ID, Course_ID),

    FOREIGN KEY (Student_ID)
        REFERENCES Students(Student_ID),

    FOREIGN KEY (Course_ID)
        REFERENCES Courses(Course_ID)
);
```

Notice that the `Enrollments` table contains two Foreign Keys:

```text
Student_ID → Students table

Course_ID → Courses table
```

The combination:

```text
Student_ID + Course_ID
```

is the Primary Key of the `Enrollments` table.

---

# Insert Data

Insert students:

```sql
INSERT INTO Students
VALUES
(1, 'Ali'),
(2, 'Sara'),
(3, 'Hamza');
```

Insert courses:

```sql
INSERT INTO Courses
VALUES
(101, 'SQL'),
(102, 'Python'),
(103, 'Database');
```

Insert enrollments:

```sql
INSERT INTO Enrollments
VALUES
(1, 101),
(1, 102),
(2, 101),
(3, 103);
```

Now the tables are connected.

---

# 8. Third Normal Form — 3NF

## Definition

Third Normal Form means that the table must already be in 2NF, and a non-key column should not depend on another non-key column. Every non-key column should depend directly on the key of the table.

In simple words:

> **Non-key information should not depend on another non-key information.**

## Rule

```text
3NF = Remove Transitive Dependency
```

---

# Problem Before 3NF

Now look at our Courses table:

| Course_ID | Course_Name | Instructor_ID | Instructor_Name |
|---|---|---|---|
| 101 | SQL | 1 | Kamran |
| 102 | Python | 1 | Kamran |
| 103 | Web | 2 | Amna |

The Primary Key is:

```text
Course_ID
```

We know:

```text
Course_ID → Instructor_ID
```

For example:

```text
101 → 1
```

But we also know:

```text
Instructor_ID → Instructor_Name
```

For example:

```text
1 → Kamran
2 → Amna
```

So the dependency becomes:

```text
Course_ID
    ↓
Instructor_ID
    ↓
Instructor_Name
```

The problem is that `Instructor_Name` depends on `Instructor_ID`, not directly on `Course_ID`.

`Instructor_ID` is a non-key column in the Courses table.

This is called **Transitive Dependency**.

---

# Another Problem — Repeated Instructor Data

Look at:

| Course_ID | Course_Name | Instructor_ID | Instructor_Name |
|---|---|---|---|
| 101 | SQL | 1 | Kamran |
| 102 | Python | 1 | Kamran |
| 103 | Web | 2 | Amna |

`Kamran` is repeated.

If Kamran's name changes, we have to update multiple course records.

This is exactly the type of problem 3NF helps us solve.

---

# Solution — Convert the Table into 3NF

Separate instructor information into its own table.

### Courses

| Course_ID | Course_Name | Instructor_ID |
|---|---|---|
| 101 | SQL | 1 |
| 102 | Python | 1 |
| 103 | Web | 2 |

### Instructors

| Instructor_ID | Instructor_Name |
|---|---|
| 1 | Kamran |
| 2 | Amna |

Now:

```text
Courses
    ↓
Instructor_ID
    ↓
Instructors
```

The instructor's name is stored only once.

If the instructor's name changes, we update one record.

---

# 3NF Practical Example in SQL Server

Create the Instructors table:

```sql
CREATE TABLE Instructors
(
    Instructor_ID INT PRIMARY KEY,
    Instructor_Name VARCHAR(100)
);
```

Now modify the Courses design:

```sql
CREATE TABLE Courses
(
    Course_ID INT PRIMARY KEY,
    Course_Name VARCHAR(100),
    Instructor_ID INT,

    FOREIGN KEY (Instructor_ID)
        REFERENCES Instructors(Instructor_ID)
);
```

Now insert instructors:

```sql
INSERT INTO Instructors
VALUES
(1, 'Kamran'),
(2, 'Amna');
```

Insert courses:

```sql
INSERT INTO Courses
VALUES
(101, 'SQL', 1),
(102, 'Python', 1),
(103, 'Web', 2);
```

Now the instructor's name is not repeated inside the Courses table.

---

# 9. Complete Normalized Database

After applying 1NF, 2NF, and 3NF, our database can look like this:

## Students

```sql
CREATE TABLE Students
(
    Student_ID INT PRIMARY KEY,
    Student_Name VARCHAR(100)
);
```

## Instructors

```sql
CREATE TABLE Instructors
(
    Instructor_ID INT PRIMARY KEY,
    Instructor_Name VARCHAR(100)
);
```

## Courses

```sql
CREATE TABLE Courses
(
    Course_ID INT PRIMARY KEY,
    Course_Name VARCHAR(100),
    Instructor_ID INT,

    FOREIGN KEY (Instructor_ID)
        REFERENCES Instructors(Instructor_ID)
);
```

## Enrollments

```sql
CREATE TABLE Enrollments
(
    Student_ID INT,
    Course_ID INT,

    PRIMARY KEY (Student_ID, Course_ID),

    FOREIGN KEY (Student_ID)
        REFERENCES Students(Student_ID),

    FOREIGN KEY (Course_ID)
        REFERENCES Courses(Course_ID)
);
```

Now our database is properly separated.

```text
Students
    |
    | Student_ID
    |
    ↓
Enrollments
    |
    | Course_ID
    |
    ↓
Courses
    |
    | Instructor_ID
    |
    ↓
Instructors
```

---

# 10. Normalization Summary

| Normal Form | Problem | Solution |
|---|---|---|
| 1NF | Multiple values in one cell | One value per cell |
| 2NF | Partial dependency | Remove partial dependency |
| 3NF | Transitive dependency | Remove transitive dependency |

Easy way to remember:

```text
1NF
One Cell = One Value

2NF
Remove Partial Dependency

3NF
Remove Transitive Dependency
```

---

# 11. What is a Relationship?

Once we divide our database into different tables, those tables need to be connected.

A relationship describes how records in one table are connected to records in another table.

For example:

```text
Students
    ↓
Enrollments
    ↓
Courses
```

The Foreign Key creates the actual connection between the tables.

---

# 12. Types of Relationships

There are three common types of relationships:

```text
1. One-to-One       1 : 1

2. One-to-Many      1 : N

3. Many-to-Many     M : N
```

---

# 13. One-to-One Relationship — 1:1

In a One-to-One relationship, one record in the first table is related to one record in the second table.

Example:

```text
Person → Passport
```

One person has one passport record.

### SQL Example

```sql
CREATE TABLE Person
(
    Person_ID INT PRIMARY KEY,
    Person_Name VARCHAR(100)
);
```

```sql
CREATE TABLE Passport
(
    Passport_ID INT PRIMARY KEY,
    Passport_Number VARCHAR(50) UNIQUE,
    Person_ID INT UNIQUE,

    FOREIGN KEY (Person_ID)
        REFERENCES Person(Person_ID)
);
```

The `UNIQUE` constraint on `Person_ID` ensures that the same person cannot be associated with multiple passport records in this table.

---

# 14. One-to-Many Relationship — 1:N

In a One-to-Many relationship, one record in the first table can be related to many records in the second table.

Example:

```text
Customer → Orders
```

One customer can have many orders.

But each order belongs to one customer.

### Customers

```sql
CREATE TABLE Customers
(
    Customer_ID INT PRIMARY KEY,
    Customer_Name VARCHAR(100)
);
```

### Orders

```sql
CREATE TABLE Orders
(
    Order_ID INT PRIMARY KEY,
    Order_Date DATE,
    Customer_ID INT,

    FOREIGN KEY (Customer_ID)
        REFERENCES Customers(Customer_ID)
);
```

Example:

```text
Customer 1
    ↓
Order 101
Order 102
Order 103
```

This is:

```text
1 Customer → Many Orders
```

---

# 15. Many-to-Many Relationship — M:N

In a Many-to-Many relationship, many records in one table can be related to many records in another table.

Example:

```text
Students ↔ Courses
```

One student can take many courses.

One course can have many students.

For example:

```text
Ali
 ├── SQL
 └── Python

Sara
 └── SQL

Hamza
 └── Database
```

This is a Many-to-Many relationship.

---

# How Do We Implement M:N?

We use a third table called a:

```text
Junction Table
```

or:

```text
Bridge Table
```

For example:

```text
Students
    ↓
StudentCourses
    ↓
Courses
```

The junction table contains the Foreign Keys of both tables.

```sql
CREATE TABLE StudentCourses
(
    Student_ID INT,
    Course_ID INT,

    PRIMARY KEY (Student_ID, Course_ID),

    FOREIGN KEY (Student_ID)
        REFERENCES Students(Student_ID),

    FOREIGN KEY (Course_ID)
        REFERENCES Courses(Course_ID)
);
```

This converts the Many-to-Many relationship into two One-to-Many relationships:

```text
Students
   1
   |
   N
StudentCourses
   N
   |
   1
Courses
```

---

# 16. Referential Integrity

A Foreign Key also helps protect the relationship between tables.

Suppose we have:

```text
Students

Student_ID
1
2
3
```

Now we try:

```sql
INSERT INTO Enrollments
VALUES (10, 101);
```

But Student `10` does not exist.

SQL Server will reject this operation because:

```text
Student_ID = 10
```

does not exist in the parent `Students` table.

This protection is called **Referential Integrity**.

## Definition

> **Referential integrity ensures that a Foreign Key value refers to an existing record in the related table.**

---
