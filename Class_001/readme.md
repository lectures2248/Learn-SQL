# Introduction to Database

> "A database is a digital collection of related data, organized in a way so you can easily store, access, manage, and update it."

**Example:**
A school register or Excel sheet where you have all student names, roll numbers, marks, and attendance in one place – that's a basic form of a database.

---

# Evolution of Databases

## 1. Introduction to File System

> "A file system is how your computer stores and organizes data in files and folders."

**Example:**
Teacher store many student records saved as separate Word files on their laptop.
Each file is like a student's data (name, marks, attendance).
They put them in a folder named "Class_A".

**Drawbacks:**
- Difficult to search data quickly.
- If the number of files increases, it becomes hard to manage.
- No security – anyone can open the files.
- No relationship – you can't connect one student's file with another's attendance file.
- Duplication of Data.

## 2. DBMS – Database Management System

> "A DBMS is software that helps us create, store, modify, and manage databases."

**Example:**
Like a librarian who manages all books in a library. DBMS is like a librarian for data.

**Drawbacks:**
- Data Redundancy (Same data repeated).
- No strong relationship between data tables.
- Not very good for complex queries or large data.
- DBMS systems usually don't provide automatic, reliable backup & recovery.

## 3. RDBMS – Relational Database Management System

> "An RDBMS is an advanced DBMS where data is stored in tables and relations are made between those tables using keys."

**Why RDBMS?**
- Data is well-structured.
- Tables can be connected using primary and foreign keys.
- More secure, faster, and supports multiple users.

**Example:**
A student's data is in one table, and their marks are in another table – but both are linked by roll number.

---

# SQL – Structured Query Language

> "SQL is the language used to communicate with the database."

You use SQL to:
- Insert data
- Retrieve data
- Update data
- Delete data

---

# Categories of SQL Commands

## DDL – Data Definition Language
DDL commands are used to define and structure the database, tables, and other database objects like views, indexes, and schemas.

## DML – Data Manipulation Language
DML commands are used to manipulate the data inside the tables. These commands let you add, change, or remove the actual data.

## DCL – Data Control Language
DCL commands are used to control access to the database. They help in giving or restricting permissions for different users.

## TCL – Transaction Control Language
TCL commands are used to manage transactions. A transaction is a set of operations that must be completed all together.

**Example:**
Think of a bank transaction:
You transfer money from one account to another.
If something goes wrong (like the network breaks), you want to undo the entire transaction so that no money is lost or mistakenly transferred.

---

# Step-by-Step Installation of SQL Server

**STEP 1: Download SQL Server**
- Visit https://www.microsoft.com/en-us/sql-server/sql-server-downloads
- Under Developer Edition, click Download now.

**STEP 2: Run the Installer**
- Open the downloaded file: `SQLServer2022-SSEI-Dev.exe`
- Select Basic installation type.

**STEP 3: Download SSMS**
- Visit: https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms
- Click Download SQL Server Management Studio (SSMS).

**STEP 4: Install SSMS**
- Run the installer: `SSMS-Setup-ENU.exe`
- Click Install.

**STEP 5: Open SSMS and Connect to SQL Server**
- Open SQL Server Management Studio from the Start menu.
- In the Connect to Server window:

| Field | Value |
|---|---|
| Server Type | Database Engine |
| Server Name | localhost or .\SQLEXPRESS or MSSQLSERVER |
| Authentication | Windows Authentication |

- Click Connect.

---

# Working with SQL Queries

To work in SQL we need these queries to execute, which further divide in 4 groups:

- **DDL** – Data Definition Language
- **DML** – Data Manipulation Language
- **DCL** – Data Control Language
- **TCL** – Transaction Control Language

---

# Now We Are Starting with DML Commands

Since we are starting, we have to use one DDL command (`CREATE`) for our working.

## STEP 1: Create Database and Table

**Create the database:**
```sql
CREATE DATABASE SchoolDB;
GO
```

**Select that database:**
```sql
USE SchoolDB;
GO
```

**Create a sample table:**
```sql
CREATE TABLE Students
(
   StudentID   INT IDENTITY(1,1) PRIMARY KEY,
   FirstName   VARCHAR(50),
   LastName    VARCHAR(50),
   Age         INT,
   City        VARCHAR(50)
);
```

## STEP 2: INSERT – Add new data

The INSERT command adds new rows into the table.

**1) Insert one record:**
```sql
INSERT INTO Students (FirstName, LastName, Age, City)
VALUES ('Ali', 'Khan', 20, 'Karachi');
```

**Check table:**
```sql
SELECT * FROM Students;
```

**2) Insert multiple records at once:**
```sql
INSERT INTO Students (FirstName, LastName, Age, City)
VALUES
('Aisha', 'Ahmed', 22, 'Lahore'),
('Bilal', 'Shah', 19, 'Islamabad'),
('Sara', 'Iqbal', 23, 'Multan');
```

## STEP 3: SELECT – View or read data

SELECT is used to fetch or view data.

**1) View all columns:**
```sql
SELECT * FROM Students;
```

**2) View specific columns:**
```sql
SELECT FirstName, City FROM Students;
```

**3) Use WHERE condition:**
```sql
SELECT * FROM Students
WHERE City = 'Lahore';
```

**4) Use comparison operators:**
```sql
SELECT * FROM Students
WHERE Age > 20;
```

**5) Use ORDER BY (to sort data):**
```sql
SELECT * FROM Students
ORDER BY Age DESC;

SELECT * FROM Students
ORDER BY Age ASC;
```

## STEP 4: DELETE – Remove data

DELETE removes rows from the table.

**Example 1: Delete one student**
```sql
DELETE FROM Students
WHERE StudentID = 3;
```

**Example 2: Delete all rows from the table**
```sql
DELETE FROM Students;

SELECT * FROM Students;
```

## STEP 5: UPDATE – Modify existing data

**Update a single row**

Suppose we want to change the city of student "Ali" to "Hyderabad".
```sql
UPDATE Students
SET City = 'Hyderabad'
WHERE Name = 'Bilal';

SELECT * FROM Students;
```

**Update multiple columns at once**

You can change more than one column together.
```sql
UPDATE Students
SET City = 'Karachi',
    Age = 25
WHERE Name = 'Sara';

SELECT * FROM Students;
```

**Update multiple rows**

If you give a condition that matches many rows, they all will be updated.
```sql
UPDATE Students
SET City = 'Islamabad'
WHERE City = 'Lahore' OR City = 'Hyderabad';
```
