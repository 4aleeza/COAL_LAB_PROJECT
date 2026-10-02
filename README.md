# 🗄️ FAST Mini SQL Database — x86 Assembly

A **mini database management system implemented entirely in x86 Assembly Language** using the **Irvine32 library**.

The project recreates several fundamental database and SQL operations at a low level without using an actual database engine. Student records are stored manually in memory using arrays, while Assembly procedures implement operations similar to SQL's `INSERT`, `SELECT`, `UPDATE`, `DELETE`, `COUNT`, `MIN`, `MAX`, and `LIKE`.

---

## 📌 Project Overview

The **FAST Mini Database** stores student information and allows the user to manipulate records through a menu-driven console interface.

Each student record contains:

| Field | Description |
|---|---|
| First Name | Student's first name |
| Last Name | Student's last name |
| NU-ID | Unique student identifier |
| Attendance | Attendance percentage |
| Average Marks | Average marks percentage |

The system supports up to **50 student records**.

---

## ✨ Features

- ➕ Add new student records
- 🔍 Search for a student using NU-ID
- 📋 Display all student records
- ✏️ Update existing records
- 🗑️ Delete records using NU-ID
- 🔢 Count total stored students
- 🧹 Clear all records
- 🔎 Search students by name
- 🔤 `LIKE`-style name searching
- 📉 Find minimum attendance
- 📈 Find maximum attendance
- 📉 Find minimum marks
- 📈 Find maximum marks
- 💾 Manual memory-based record storage
- ⚙️ Implemented entirely in x86 Assembly

---

# 🧠 SQL Concepts Implemented

Although the project does not use an actual SQL server, its Assembly procedures simulate common SQL operations.

| SQL Concept | Assembly Implementation |
|---|---|
| `INSERT` | Add Record |
| `SELECT ... WHERE ID =` | Display by ID |
| `SELECT *` | Display All |
| `UPDATE ... WHERE ID =` | Update Record |
| `DELETE ... WHERE ID =` | Delete by ID |
| `COUNT(*)` | Count Total Students |
| `LIKE 'A%'` | Search Starts With |
| `LIKE '%A'` | Search Ends With |
| `MIN(attendance)` | Minimum Attendance |
| `MAX(attendance)` | Maximum Attendance |
| `MIN(marks)` | Minimum Marks |
| `MAX(marks)` | Maximum Marks |

For example, the Assembly operation:

```text
Search first names beginning with 'A'
```

is conceptually similar to:

```sql
SELECT *
FROM students
WHERE first_name LIKE 'A%';
```

---

# 🏗️ Record Structure

Instead of SQL tables, records are stored using parallel arrays.

Conceptually, the database behaves like:

```text
Students
┌────────────┬───────────┬────────────┬────────────┬─────────┐
│ First Name │ Last Name │ NU-ID      │ Attendance │ Marks   │
├────────────┼───────────┼────────────┼────────────┼─────────┤
│ Aleeza     │ Khan      │ 24K-3053   │ 92         │ 88      │
│ Ali        │ Ahmed     │ 24K-1001   │ 85         │ 79      │
│ Sara       │ Hassan    │ 24K-2042   │ 96         │ 91      │
└────────────┴───────────┴────────────┴────────────┴─────────┘
```

Internally, however, the data is represented through Assembly arrays:

```asm
first_name  BYTE MAX * STRLEN DUP(0)
last_name   BYTE MAX * STRLEN DUP(0)
id_array    BYTE MAX * STRLEN DUP(0)
avg         BYTE MAX DUP(0)
attendance  BYTE MAX DUP(0)

countRec    DWORD 0
```

`countRec` keeps track of how many records are currently stored.

---

# 📊 Database Capacity

The program uses the following configuration:

```asm
MAX     = 50
STRLEN  = 20
```

Therefore:

- Maximum records: **50**
- String field size: **20 bytes**
- Attendance: stored as `BYTE`
- Average marks: stored as `BYTE`

---

# 🖥️ Main Menu

The program provides a console-based interface similar to:

```text
=== FAST Mini Database ===

1. Add Record
2. Display by ID
3. Display All
4. Delete by ID
5. Update Record
6. Count Total Students
7. Clear All Records
8. Exit
9. Search Name (LIKE)

11. Minimum Attendance
12. Maximum Attendance
13. Minimum Marks
14. Maximum Marks

Enter choice:
```

The selected option is read using Irvine32's:

```asm
call ReadInt
```

and Assembly comparisons and conditional jumps determine which procedure is executed.

---

# ➕ Adding Records — `INSERT`

The `add_record` procedure creates a new student record.

The user enters:

```text
First Name
Last Name
NU-ID
Attendance Percentage
Average Percentage
```

Conceptually, this behaves like:

```sql
INSERT INTO students
(first_name, last_name, nu_id, attendance, marks)
VALUES (...);
```

The program calculates the correct array position using:

```text
index × STRLEN
```

and copies string data into memory using:

```asm
rep movsb
```

After insertion:

```text
countRec = countRec + 1
```

---

# 🔍 Display by ID — `SELECT WHERE`

A student can be searched using their NU-ID.

Conceptually:

```sql
SELECT *
FROM students
WHERE nu_id = '24K-3053';
```

The program performs a manual character-by-character comparison between the entered ID and IDs stored inside `id_array`.

If a matching record is found, its information is displayed.

Otherwise:

```text
Data For The ID Not Found.
```

is shown.

---

# 📋 Display All — `SELECT *`

The **Display All** operation iterates through every stored record.

Equivalent SQL:

```sql
SELECT *
FROM students;
```

The program loops from:

```text
index = 0
```

to:

```text
index = countRec - 1
```

and calls the record-display procedure for each student.

---

# ✏️ Update Record — `UPDATE`

Records can be updated using their NU-ID.

Conceptually:

```sql
UPDATE students
SET first_name = ...,
    last_name = ...,
    attendance = ...,
    marks = ...
WHERE nu_id = ...;
```

The program first searches for the requested NU-ID.

Once the record is located, its:

- First name
- Last name
- Attendance
- Average marks

can be overwritten with new values.

---

# 🗑️ Delete Record — `DELETE`

Students can be deleted using their NU-ID.

Equivalent SQL:

```sql
DELETE FROM students
WHERE nu_id = '24K-3053';
```

Rather than shifting every record after the deleted student, the program uses a more efficient approach.

```text
Record to Delete
       ↓
Locate its Index
       ↓
Copy Last Record
       ↓
Overwrite Deleted Slot
       ↓
Decrease countRec
```

For example:

```text
Before:

[Ali] [Sara] [Ahmed] [Aleeza]
         ↑
       DELETE

After:

[Ali] [Aleeza] [Ahmed]
```

The last record replaces the deleted record, avoiding the need to shift the entire database.

---

# 🔎 LIKE-Style Searching

The project implements basic SQL `LIKE` behavior for first names.

## Starts With

The user enters a character and the program finds first names beginning with that character.

Example:

```text
Enter character: A
```

Conceptually:

```sql
SELECT *
FROM students
WHERE first_name LIKE 'A%';
```

Possible matches:

```text
Aleeza
Ali
Ahmed
Ayesha
```

---

## Ends With

The program can also search for names ending with a specific character.

For example:

```text
Enter character: a
```

Conceptually:

```sql
SELECT *
FROM students
WHERE first_name LIKE '%a';
```

The program manually scans each fixed-length string to locate its final non-null character before performing the comparison.

---

# 🔢 COUNT()

The project implements an equivalent of:

```sql
SELECT COUNT(*)
FROM students;
```

The `count_total` procedure reads:

```asm
countRec
```

and displays:

```text
Total students: X
```

Because `countRec` is maintained whenever records are inserted or deleted, the database does not need to scan all records simply to calculate the count.

---

# 📉 MIN()

Minimum values can be calculated for both attendance and marks.

Equivalent SQL operations:

```sql
SELECT MIN(attendance)
FROM students;
```

and:

```sql
SELECT MIN(marks)
FROM students;
```

The Assembly procedures iterate through the corresponding arrays and retain the smallest value encountered.

---

# 📈 MAX()

Maximum values are also supported.

Equivalent SQL:

```sql
SELECT MAX(attendance)
FROM students;
```

and:

```sql
SELECT MAX(marks)
FROM students;
```

The program scans the arrays and updates the current maximum whenever a larger value is encountered.

---

# 🧹 Clear Database

The program can remove all active records.

Instead of manually deleting every stored value, it performs:

```asm
mov countRec, 0
```

This logically resets the database because subsequent operations only consider records below `countRec`.

Conceptually, this behaves similarly to clearing the table:

```sql
DELETE FROM students;
```

---

# 🧩 Major Procedures

The project is divided into multiple Assembly procedures to keep individual database operations separated.

```text
main
│
├── add_record
│
├── display_record
│
├── display_all
│
├── delete_by_id
│
├── update_by_id
│
├── count_total
│
├── clear_all
│
├── search_menu
│   ├── search_starts_with
│   └── search_ends_with
│
├── show_record_at_index
│
├── find_min_att
├── find_max_att
├── find_min_marks
└── find_max_marks
```

---

# 🔄 Program Flow

```text
                  ┌─────────────────┐
                  │  Start Program  │
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │  Display Menu   │
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │  Read Choice    │
                  └────────┬────────┘
                           │
          ┌────────────────┼─────────────────┐
          │                │                 │
          ▼                ▼                 ▼
      INSERT            SELECT            UPDATE
          │                │                 │
          └────────────────┼─────────────────┘
                           │
             ┌─────────────┼──────────────┐
             │             │              │
             ▼             ▼              ▼
           DELETE        SEARCH        AGGREGATE
                                      FUNCTIONS
                           │
                           ▼
                    Return to Menu
                           │
                           ▼
                         EXIT
```

---

# 🛠️ Technologies Used

- **x86 Assembly Language**
- **MASM**
- **Irvine32 Library**
- **Microsoft Visual Studio**
- Low-level memory manipulation
- Assembly procedures
- CPU registers
- Stack-based parameter passing

---

# ⚙️ Requirements

To compile and run the project, you need:

- Windows
- Microsoft Visual Studio
- MASM / Microsoft Macro Assembler
- Irvine32 library correctly configured

The source file begins with:

```asm
INCLUDE Irvine32.inc
```

so Irvine32 must be available to the assembler.

---

# 🚀 Running the Project

### 1. Configure Irvine32

Make sure the Irvine32 library has been properly installed and configured with Visual Studio.

### 2. Create an Assembly Project

Create or open a MASM-compatible project in Visual Studio.

### 3. Add the Source File

Add the project's `.asm` file to the project.

Example:

```text
FAST-Mini-Database/
│
├── main.asm
└── README.md
```

### 4. Build

Compile the Assembly source using MASM.

### 5. Run

Run the generated executable and use the numbered menu to interact with the mini database.

---

# 🧠 Assembly Concepts Demonstrated

This project demonstrates several important low-level programming concepts:

- Arrays
- Fixed-size strings
- Memory addressing
- Indexed addressing
- Registers
- Procedures
- Stack parameters
- Loops
- Conditional jumps
- String instructions
- `REP MOVSB`
- Manual string comparison
- Array traversal
- Input/output using Irvine32
- Low-level record management

---

# 🗃️ Database Concepts Demonstrated

From a database perspective, the project demonstrates:

- Records
- Fields
- CRUD operations
- Searching
- Unique identifier lookup
- SQL-like `LIKE` searching
- Aggregate operations
- `COUNT`
- `MIN`
- `MAX`
- Table-style data organization

---

# ⚠️ Limitations

This is an educational in-memory database rather than a full relational database management system.

Current limitations include:

- Maximum of 50 records
- Data is not permanently stored after the program exits
- No actual SQL query parser
- No disk-based database file
- No indexing
- No transactions
- No authentication
- No concurrency
- Fixed-length string storage
- LIKE-style search is limited compared with real SQL
- Records are stored using parallel arrays rather than relational tables

---

# 🔮 Possible Future Improvements

The project could be expanded with:

- File handling for persistent records
- Sorting by name, marks, or attendance
- Full substring searching
- Case-insensitive searching
- Additional aggregate functions such as `AVG()`
- Duplicate NU-ID validation
- Input validation for percentages
- More SQL-like query operations
- Better formatted record output
- Dynamic record capacity
- Additional student attributes

---

# 🎯 Project Purpose

The purpose of this project is to understand how familiar **database operations can be implemented manually at the Assembly level**.

Instead of relying on a database engine to perform operations such as:

```sql
INSERT
SELECT
UPDATE
DELETE
LIKE
COUNT()
MIN()
MAX()
```

the project implements the underlying searching, copying, comparison, iteration, and memory-management logic directly using x86 Assembly instructions.

This provides practical experience with both **database fundamentals** and **low-level computer architecture concepts**.

---

# 📌 Summary

**FAST Mini SQL Database** is a menu-driven student record management system built using **x86 Assembly and Irvine32**.

It recreates several fundamental SQL concepts through low-level Assembly procedures:

```text
Student Records
      │
      ├── INSERT
      ├── SELECT
      ├── UPDATE
      ├── DELETE
      ├── LIKE
      ├── COUNT()
      ├── MIN()
      └── MAX()
```

The project demonstrates how high-level database functionality ultimately translates into fundamental operations such as **memory access, loops, comparisons, copying, indexing, and conditional branching**.
