# Library_System_Managemant

# 📚 Library System Management — SQL Project

A relational database project that models a multi-branch library system and solves 18 real-world tasks — from basic CRUD operations to advanced queries, CTAS, and stored procedures — using pure SQL.

---

## 📌 Project Overview

This project designs and queries a library management database covering branches, employees, members, books, and the full issue/return lifecycle. It progresses through four levels of complexity:

1. **Database Setup** – schema design with 6 related tables and foreign keys
2. **CRUD Operations** – inserting, updating, and deleting records
3. **CTAS (Create Table As Select)** – generating derived summary tables
4. **Data Analysis & Advanced SQL** – joins, aggregations, subqueries, and stored procedures for real business logic

---

## 🗂️ Database Schema (ERD)

The system is built around 6 tables:

| Table            | Purpose                                              |
|-------------------|-------------------------------------------------------|
| `branch`          | Library branch details and assigned manager           |
| `employees`       | Staff records, linked to a branch                     |
| `members`         | Library members and registration dates                |
| `books`           | Book catalog — title, category, price, status, author |
| `issued_status`   | Records of books issued to members                    |
| `return_status`   | Records of books returned                              |

**Relationships:**
- `employees.branch_id` → `branch.branch_id`
- `issued_status.issued_member_id` → `members.member_id`
- `issued_status.issued_emp_id` → `employees.emp_id`
- `issued_status.issued_book_isbn` → `books.isbn`
- `return_status.return_book_isbn` → `books.isbn`

---

## 🎯 Objectives

- Design a normalized library database with proper foreign key relationships
- Perform CRUD operations to simulate day-to-day library operations
- Use CTAS to build summary/reporting tables from query results
- Answer business questions using joins, aggregation, and window-style logic
- Implement stored procedures to automate book issue/return status updates

---

## ❓ Tasks Solved

**CRUD Operations**
1. Add a new book record
2. Update an existing member's address
3. Delete a record from the issued status table
4. Retrieve all books issued by a specific employee
5. List members who have issued more than one book

**CTAS**

6. Create a summary table of each book with its total issue count

**Data Analysis & Findings**

7. Retrieve all books in a specific category
8. Find total rental income by category
9. List members who registered in the last 180 days
10. List employees with their branch manager's name and branch details
11. Create a table of books with rental price above a threshold
12. Retrieve the list of books not yet returned

**Advanced SQL**

13. Identify members with overdue books (30-day return period)
14. Update book status to "available" when a book is returned
15. Generate a branch performance report (books issued, returned, revenue)
16. CTAS: create a table of active members (issued a book in the last 2 months)
17. Find the top employees by number of book issues processed
18. Stored procedure to issue a book — checks availability and updates status automatically

---

## 🧠 SQL Code

### Schema

```sql
-- Library System Management SQL Project

CREATE TABLE branch
(
    branch_id VARCHAR(10) PRIMARY KEY,
    manager_id VARCHAR(10),
    branch_address VARCHAR(30),
    contact_no VARCHAR(15)
);

CREATE TABLE employees
(
    emp_id VARCHAR(10) PRIMARY KEY,
    emp_name VARCHAR(30),
    position VARCHAR(30),
    salary DECIMAL(10,2),
    branch_id VARCHAR(10),
    FOREIGN KEY (branch_id) REFERENCES branch(branch_id)
);

CREATE TABLE members
(
    member_id VARCHAR(10) PRIMARY KEY,
    member_name VARCHAR(30),
    member_address VARCHAR(30),
    reg_date DATE
);

CREATE TABLE books
(
    isbn VARCHAR(50) PRIMARY KEY,
    book_title VARCHAR(80),
    category VARCHAR(30),
    rental_price DECIMAL(10,2),
    status VARCHAR(10),
    author VARCHAR(30),
    publisher VARCHAR(30)
);

CREATE TABLE issued_status
(
    issued_id VARCHAR(10) PRIMARY KEY,
    issued_member_id VARCHAR(30),
    issued_book_name VARCHAR(80),
    issued_date DATE,
    issued_book_isbn VARCHAR(50),
    issued_emp_id VARCHAR(10),
    FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
    FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
    FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn)
);

CREATE TABLE return_status
(
    return_id VARCHAR(10) PRIMARY KEY,
    issued_id VARCHAR(30),
    return_book_name VARCHAR(80),
    return_date DATE,
    return_book_isbn VARCHAR(50),
    FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);
```

### CRUD & CTAS Tasks

```sql
-- Task 1: Add a new book record
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES ('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');

-- Task 2: Update an existing member's address
UPDATE members
SET member_address = '125 Main St'
WHERE member_id = 'C101';

-- Task 3: Delete a record from issued_status
DELETE FROM issued_status
WHERE issued_id = 'IS121';

-- Task 4: Retrieve all books issued by a specific employee
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101';

-- Task 5: Members who have issued more than one book
SELECT ist.issued_emp_id, e.emp_name
FROM issued_status as ist
JOIN employees as e ON e.emp_id = ist.issued_emp_id
GROUP BY 1, 2
HAVING COUNT(ist.issued_id) > 1;

-- Task 6: CTAS - book issue counts
CREATE TABLE book_cnts AS
SELECT b.isbn, b.book_title, COUNT(ist.issued_id) as no_issued
FROM books as b
JOIN issued_status as ist ON ist.issued_book_isbn = b.isbn
GROUP BY 1, 2;

-- Task 7: All books in a specific category
SELECT * FROM books
WHERE category = 'Classic';

-- Task 8: Total rental income by category
SELECT b.category, SUM(b.rental_price), COUNT(*)
FROM books as b
JOIN issued_status as ist ON ist.issued_book_isbn = b.isbn
GROUP BY 1;

-- Task 9: Members registered in the last 180 days
SELECT * FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';

-- Task 10: Employees with their branch manager's name
SELECT e1.*, b.manager_id, e2.emp_name as manager
FROM employees as e1
JOIN branch as b ON b.branch_id = e1.branch_id
JOIN employees as e2 ON b.manager_id = e2.emp_id;

-- Task 11: Books above a rental price threshold
CREATE TABLE books_price_greater_than_seven AS
SELECT * FROM books
WHERE rental_price > 7;

-- Task 12: Books not yet returned
SELECT DISTINCT ist.issued_book_name
FROM issued_status as ist
LEFT JOIN return_status as rs ON ist.issued_id = rs.issued_id
WHERE rs.return_id IS NULL;
```

### Advanced SQL & Stored Procedures

```sql
-- Task 13: Members with overdue books (30-day period)
SELECT
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    ist.issued_date,
    CURRENT_DATE - ist.issued_date as over_dues_days
FROM issued_status as ist
JOIN members as m ON m.member_id = ist.issued_member_id
JOIN books as bk ON bk.isbn = ist.issued_book_isbn
LEFT JOIN return_status as rs ON rs.issued_id = ist.issued_id
WHERE rs.return_date IS NULL
    AND (CURRENT_DATE - ist.issued_date) > 30
ORDER BY 1;

-- Task 14: Stored procedure - update book status on return
CREATE OR REPLACE PROCEDURE add_return_records(
    p_return_id VARCHAR(10),
    p_issued_id VARCHAR(10),
    p_book_quality VARCHAR(10)
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_isbn VARCHAR(50);
    v_book_name VARCHAR(80);
BEGIN
    INSERT INTO return_status(return_id, issued_id, return_date, book_quality)
    VALUES (p_return_id, p_issued_id, CURRENT_DATE, p_book_quality);

    SELECT issued_book_isbn, issued_book_name
    INTO v_isbn, v_book_name
    FROM issued_status
    WHERE issued_id = p_issued_id;

    UPDATE books
    SET status = 'yes'
    WHERE isbn = v_isbn;

    RAISE NOTICE 'Thank you for returning the book: %', v_book_name;
END;
$$;

-- Task 15: Branch performance report
CREATE TABLE branch_reports AS
SELECT
    b.branch_id,
    b.manager_id,
    COUNT(ist.issued_id) as number_book_issued,
    COUNT(rs.return_id) as number_of_book_return,
    SUM(bk.rental_price) as total_revenue
FROM issued_status as ist
JOIN employees as e ON e.emp_id = ist.issued_emp_id
JOIN branch as b ON e.branch_id = b.branch_id
LEFT JOIN return_status as rs ON rs.issued_id = ist.issued_id
JOIN books as bk ON ist.issued_book_isbn = bk.isbn
GROUP BY 1, 2;

-- Task 16: CTAS - active members (issued a book in the last 2 months)
CREATE TABLE active_members AS
SELECT * FROM members
WHERE member_id IN (
    SELECT DISTINCT issued_member_id
    FROM issued_status
    WHERE issued_date >= CURRENT_DATE - INTERVAL '2 month'
);

-- Task 17: Employees with the most book issues processed
SELECT
    e.emp_name,
    b.*,
    COUNT(ist.issued_id) as no_book_issued
FROM issued_status as ist
JOIN employees as e ON e.emp_id = ist.issued_emp_id
JOIN branch as b ON e.branch_id = b.branch_id
GROUP BY 1, 2;

-- Task 18: Stored procedure - issue a book (checks availability first)
CREATE OR REPLACE PROCEDURE issue_book(
    p_issued_id VARCHAR(10),
    p_issued_member_id VARCHAR(30),
    p_issued_book_isbn VARCHAR(30),
    p_issued_emp_id VARCHAR(10)
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_status VARCHAR(10);
BEGIN
    SELECT status INTO v_status
    FROM books
    WHERE isbn = p_issued_book_isbn;

    IF v_status = 'yes' THEN
        INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
        VALUES (p_issued_id, p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

        UPDATE books
        SET status = 'no'
        WHERE isbn = p_issued_book_isbn;

        RAISE NOTICE 'Book records added successfully for book isbn: %', p_issued_book_isbn;
    ELSE
        RAISE NOTICE 'Sorry, the book you requested is unavailable, isbn: %', p_issued_book_isbn;
    END IF;
END;
$$;

-- Example call
CALL issue_book('IS155', 'C108', '978-0-553-29698-2', 'E104');
```

---

## 🚀 How to Use

1. Clone this repository
   ```bash
   git clone https://github.com/Attharva10/Library_System_Managemant.git
   ```
2. Run `Tables.sql` to create the database schema
3. Run `insert_queries.sql` and `insert_queries2.sql` to populate sample data
4. Run `Solution1.sql` and `Solution2.sql` to execute the CRUD, CTAS, and advanced analysis queries

---

## 🛠️ Tech Stack

- **SQL** (PostgreSQL — joins, subqueries, CTAS, stored procedures, `plpgsql`)

---

## 👤 Author

**Atharva**
- GitHub: [@Attharva10](https://github.com/Attharva10)
- LinkedIn: [www.linkedin.com/in/atharvaumate]

---

⭐ If you found this project useful, consider giving it a star!
