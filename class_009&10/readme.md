# SQL Server — Session 9 & 10




## Setup: Create the database 

```sql
CREATE DATABASE StudentManagement;
GO
USE StudentManagement;
GO

CREATE TABLE Courses (
    CourseID INT PRIMARY KEY,
    CourseName VARCHAR(50) NOT NULL
);

CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    StudentName VARCHAR(50) NOT NULL,
    City VARCHAR(30) NOT NULL,
    Marks INT NOT NULL,
    CourseID INT NULL,
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
);

INSERT INTO Courses (CourseID, CourseName) VALUES
(1, 'Flutter'),
(2, 'Web Development'),
(3, 'AI');

INSERT INTO Students (StudentID, StudentName, City, Marks, CourseID) VALUES
(1, 'Ali',    'Karachi', 85, 1),
(2, 'Sara',   'Lahore',  90, 2),
(3, 'Ahmed',  'Karachi', 65, 1),
(4, 'Hassan', 'Lahore',  40, NULL),
(5, 'Zain',   'Karachi', 75, 2);
```

**Important detail:** Hassan has **no assigned course**. The **AI** course has **no students**. We will use these examples when we reach JOINs.

**Quick check:**
```sql
SELECT * FROM Students;
SELECT * FROM Courses;
```

## 1. Aggregate Functions — A quick class summary

**Principal asks:** “How many students do we have? What are the highest, lowest, and average marks?”

Aggregate functions calculate a value from multiple records.

| Function | Meaning |
|---|---|
| `COUNT(*)` | Count rows |
| `SUM(Marks)` | Add all marks |
| `AVG(Marks)` | Calculate average marks |
| `MAX(Marks)` | Highest marks |
| `MIN(Marks)` | Lowest marks |

```sql
SELECT
    COUNT(*) AS TotalStudents,
    SUM(Marks) AS TotalMarks,
    AVG(Marks) AS AverageMarks,
    MAX(Marks) AS HighestMarks,
    MIN(Marks) AS LowestMarks
FROM Students;
```

**Expected result:** TotalStudents = **5**, TotalMarks = **355**, AverageMarks = **71**, HighestMarks = **90**, LowestMarks = **40**.

**Remember:** `AVG` on this `INT` column returns an integer in SQL Server. To see decimals, use `AVG(CAST(Marks AS DECIMAL(10,2)))`.

**Ask students:** Which function would you use to find the total number of enrolled students?

---

## 2. GROUP BY — A city-wise report

**Next question:** “The overall average is useful. But what is the average of **each city**?”

`GROUP BY` puts rows with the same value together, so we can calculate a result for each group.

```sql
SELECT City, COUNT(*) AS TotalStudents
FROM Students
GROUP BY City;
```

**Expected results:** Karachi → **3** students; Lahore → **2** students.

Now show the city-wise averages:

```sql
SELECT City, AVG(Marks) AS AverageMarks
FROM Students
GROUP BY City;
```

**Expected results:** Karachi → **75**; Lahore → **65**.

### Add WHERE: filter students before grouping

**Principal asks:** “Count students from each city, but consider only students scoring at least 60.”

```sql
SELECT City, COUNT(*) AS TotalStudents
FROM Students
WHERE Marks >= 60
GROUP BY City;
```

**Expected:** Karachi → **3**; Lahore → **1**.

### Add HAVING: filter groups after grouping

**Principal asks:** “Show cities with an average above 70.”

```sql
SELECT City, AVG(Marks) AS AverageMarks
FROM Students
GROUP BY City
HAVING AVG(Marks) > 70;
```

**Expected:** Karachi → **75**.

**Simple difference:**
- `WHERE` filters **individual rows**, before grouping.
- `HAVING` filters **groups**, after grouping.

---

## 3. Subqueries — Use one query inside another

 “Which students scored **more than the class average**?”

First find the average:

```sql
SELECT AVG(Marks) AS AverageMarks
FROM Students;
```

**Result:** 71.

Now use that result inside another query:

```sql
SELECT StudentName, Marks
FROM Students
WHERE Marks > (
    SELECT AVG(Marks)
    FROM Students
);
```

**Expected students:** Ali (**85**), Sara (**90**), Zain (**75**).

**How it works:**
1. The **inner query** calculates the class average (71).
2. The **outer query** finds students whose marks are above 71.

### One more example: Subquery with IN

**Question:** “Show students enrolled in Flutter or Web Development.”

```sql
SELECT StudentName, CourseID
FROM Students
WHERE CourseID IN (
    SELECT CourseID
    FROM Courses
    WHERE CourseName IN ('Flutter', 'Web Development')
);
```

**Expected:** Ali, Sara, Ahmed, and Zain.

**Mini practice:** Find students scoring below the class average.

---

## 4. JOINs — Combine students and courses

**Next question:** “We can see CourseID in the Students table, but the principal wants to see the **CourseName**.”

The data is in two tables:

- `Students.CourseID` points to `Courses.CourseID`.
- `Students` is our **left** table; `Courses` is our **right** table in the examples below.

### LEFT JOIN — Keep all students

**Question:** “Show every student, even if their course is not assigned.”

```sql
SELECT s.StudentName, c.CourseName
FROM Students AS s
LEFT JOIN Courses AS c
    ON s.CourseID = c.CourseID;
```

**Key result:** Hassan appears with `NULL` as `CourseName`.

**Rule:** Keep **all rows from the left table**; add matching information from the right. If there is no match, right-side columns are `NULL`.

### RIGHT JOIN — Keep all courses

**Question:** “Show every course, even if nobody has enrolled.”

```sql
SELECT s.StudentName, c.CourseName
FROM Students AS s
RIGHT JOIN Courses AS c
    ON s.CourseID = c.CourseID;
```

**Key result:** AI appears with `NULL` as `StudentName`. Hassan is not included.

**Rule:** Keep **all rows from the right table**; add matching information from the left.

### FULL OUTER JOIN — Keep everything

**Question:** “Show all students and all courses, even when there is no match.”

```sql
SELECT s.StudentName, c.CourseName
FROM Students AS s
FULL OUTER JOIN Courses AS c
    ON s.CourseID = c.CourseID;
```

**Key results:** Hassan appears without a course, **and** AI appears without a student.

**Easy memory trick:**

| Join | Always keeps |
|---|---|
| `LEFT JOIN` | All records from the **left** table |
| `RIGHT JOIN` | All records from the **right** table |
| `FULL OUTER JOIN` | All records from **both** tables |

### Combine JOIN + GROUP BY

**Question:** “How many students are in each course, including empty courses?”

```sql
SELECT c.CourseName, COUNT(s.StudentID) AS TotalStudents
FROM Courses AS c
LEFT JOIN Students AS s
    ON c.CourseID = s.CourseID
GROUP BY c.CourseID, c.CourseName
ORDER BY c.CourseID;
```

| Course | TotalStudents |
|---|---:|
| Flutter | 2 |
| Web Development | 2 |
| AI | 0 |

**Why `COUNT(s.StudentID)` and not `COUNT(*)`?** Because a course with no matching student still gets one `NULL`-filled joined row; `COUNT(*)` would count that row as 1. `COUNT(s.StudentID)` counts only non-NULL student IDs.

---

## 5. Set Operators — Combine or compare two result lists

**Next question:** “We have two student lists: students from Karachi, and students scoring above 70. How do we combine or compare them?”

For each operator, use these **same two SELECT statements**. That makes the difference easy to see.

- **List A:** Karachi → Ali, Ahmed, Zain.
- **List B:** Marks above 70 → Ali, Sara, Zain.

### UNION — Both lists, no duplicates

```sql
SELECT StudentID, StudentName FROM Students WHERE City = 'Karachi'
UNION
SELECT StudentID, StudentName FROM Students WHERE Marks > 70;
```

**Expected names:** Ali, Ahmed, Zain, Sara (**4 distinct students**).

### UNION ALL — Both lists, duplicates allowed

```sql
SELECT StudentID, StudentName FROM Students WHERE City = 'Karachi'
UNION ALL
SELECT StudentID, StudentName FROM Students WHERE Marks > 70;
```

**Expected:** **6 rows**. Ali and Zain appear twice because they belong to both lists.

### INTERSECT — Only students present in both lists

```sql
SELECT StudentID, StudentName FROM Students WHERE City = 'Karachi'
INTERSECT
SELECT StudentID, StudentName FROM Students WHERE Marks > 70;
```

**Expected:** Ali and Zain.

### EXCEPT — In the first list, but not the second

```sql
SELECT StudentID, StudentName FROM Students WHERE City = 'Karachi'
EXCEPT
SELECT StudentID, StudentName FROM Students WHERE Marks > 70;
```

**Expected:** Ahmed.

| Operator | Think of it as |
|---|---|
| `UNION` | A + B, remove duplicate rows |
| `UNION ALL` | A + B, keep duplicate rows |
| `INTERSECT` | Only what A and B share |
| `EXCEPT` | What A has but B does not |

**Important:** Both `SELECT` statements must return the **same number of columns**, in the same order, with compatible data types. Without `ORDER BY`, row order is not guaranteed.

---
