# Chapter 2: SQL Essentials
**Relationships, JOINs, Aggregations, and Advanced Queries**

**Navigation:** [← Introduction](01_fundamentals.md) | [README](../README.md) | [Next: NoSQL Deep Dive: MongoDB →](03_01_mongodb.md)

---

## Table of Contents
- [2.1 Database Relationships](#21-database-relationships)
  - [2.1.1 One-to-One (1:1)](#211-one-to-one-11)
  - [2.1.2 One-to-Many (1:N)](#212-one-to-many-1n)
  - [2.1.3 Many-to-Many (M:N)](#213-many-to-many-mn)
  - [2.1.4 Foreign Keys](#214-foreign-keys)
- [2.2 SQL JOINs](#22-sql-joins)
  - [2.2.1 INNER JOIN](#221-inner-join)
  - [2.2.2 LEFT JOIN (LEFT OUTER JOIN)](#222-left-join-left-outer-join)
  - [2.2.3 RIGHT JOIN (RIGHT OUTER JOIN)](#223-right-join-right-outer-join)
  - [2.2.4 FULL OUTER JOIN](#224-full-outer-join)
  - [2.2.5 CROSS JOIN](#225-cross-join)
  - [2.2.6 Self JOIN](#226-self-join)
  - [2.2.7 Multiple JOINs](#227-multiple-joins)
  - [2.2.8 JOIN Performance](#228-join-performance)
- [2.3 Aggregate Functions & GROUP BY](#23-aggregate-functions--group-by)
  - [2.3.1 COUNT, SUM, AVG, MIN, MAX](#231-count-sum-avg-min-max)
  - [2.3.2 GROUP BY Basics](#232-group-by-basics)
  - [2.3.3 HAVING vs WHERE](#233-having-vs-where)
  - [2.3.4 Multiple Aggregations](#234-multiple-aggregations)
- [2.4 Advanced SQL Queries](#24-advanced-sql-queries)
  - [2.4.1 Subqueries (Nested Queries)](#241-subqueries-nested-queries)
  - [2.4.2 Common Table Expressions (CTEs)](#242-common-table-expressions-ctes)
  - [2.4.3 Window Functions](#243-window-functions)
  - [2.4.4 UNION, INTERSECT, EXCEPT](#244-union-intersect-except)
- [2.5 Transactions & Isolation Levels](#25-transactions--isolation-levels)
  - [2.5.1 Transaction Basics](#251-transaction-basics)
  - [2.5.2 Isolation Levels](#252-isolation-levels)
  - [2.5.3 Locks and Deadlocks](#253-locks-and-deadlocks)

---

## 2.1 Database Relationships

### What are Relationships?

**Definition:** Connections between tables that link related data together in a meaningful way.

**Real-world analogy:** 
```
Library System:
- Authors write books (relationship)
- Books belong to categories (relationship)
- Members borrow books (relationship)
```

---

### Why Relationships Matter

```
❌ Without relationships (all in one table):
orders
─────────────────────────────────────────────
order_id | user_name | user_email      | product_name | product_price
1        | John      | john@email.com  | iPhone       | 999
2        | John      | john@email.com  | AirPods      | 199
3        | Sarah     | sarah@email.com | iPhone       | 999

Problems:
- Data duplication (John's email repeated)
- Update anomaly (if John changes email, update multiple rows)
- Storage waste
- Inconsistency risk

✅ With relationships (normalized):
users
──────────────────────────
user_id | name  | email
1       | John  | john@email.com
2       | Sarah | sarah@email.com

products
───────────────────────────
product_id | name    | price
1          | iPhone  | 999
2          | AirPods | 199

orders
─────────────────────────────
order_id | user_id | product_id
1        | 1       | 1
2        | 1       | 2
3        | 2       | 1

Benefits:
- No duplication
- Update once (John's email in one place)
- Storage efficient
- Data integrity
```

---

### 2.1.1 One-to-One (1:1)

**Definition:** One record in Table A relates to exactly one record in Table B.

**Real-world examples:**
- User ↔ User Profile (extended info)
- Employee ↔ Parking Spot
- Country ↔ Capital City
- Person ↔ Passport

**Visual representation:**
```
users                    user_profiles
┌─────┬──────┐          ┌─────┬─────────┬─────────┐
│ id  │ name │    1:1   │ id  │ user_id │ bio     │
├─────┼──────┤ ◄──────► ├─────┼─────────┼─────────┤
│  1  │ John │          │  1  │    1    │ ...     │
│  2  │ Sara │          │  2  │    2    │ ...     │
└─────┴──────┘          └─────┴─────────┴─────────┘
```

**SQL Implementation:**

```sql
-- Primary table
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Related table (1:1)
CREATE TABLE user_profiles (
  id SERIAL PRIMARY KEY,
  user_id INTEGER UNIQUE NOT NULL,  -- UNIQUE enforces 1:1
  bio TEXT,
  avatar_url VARCHAR(500),
  date_of_birth DATE,
  phone VARCHAR(20),
  
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Insert data
INSERT INTO users (username, email) 
VALUES ('john_doe', 'john@example.com');

INSERT INTO user_profiles (user_id, bio, phone)
VALUES (1, 'Software developer from NYC', '555-1234');

-- Query with JOIN
SELECT 
  u.username,
  u.email,
  up.bio,
  up.phone
FROM users u
LEFT JOIN user_profiles up ON u.id = up.user_id;
```

**Result:**
| username | email                                       | bio                         | phone    |
| -------- | ------------------------------------------- | --------------------------- | -------- |
| john_doe | [john@example.com](mailto:john@example.com) | Software developer from NYC | 555-1234 |

**When to use 1:1:**
- Separate frequently accessed data from rarely accessed data
- Split large table for performance
- Different security requirements
- Optional extended information

---

### 2.1.2 One-to-Many (1:N)

**Definition:** One record in Table A can relate to many records in Table B, but each record in Table B relates to only one record in Table A.

**Most common relationship type!**

**Real-world examples:**
- Customer → Orders (one customer, many orders)
- Author → Books (one author, many books)
- Blog Post → Comments (one post, many comments)
- Department → Employees (one department, many employees)

**Visual representation:**
```
customers              orders
┌─────┬──────┐        ┌─────┬──────────────┬───────┐
│ id  │ name │   1:N  │ id  │ customer_id  │ total │
├─────┼──────┤ ◄───── ├─────┼──────────────┼───────┤
│  1  │ John │        │  1  │      1       │ 150   │
│  2  │ Sara │        │  2  │      1       │  75   │
└─────┴──────┘        │  3  │      2       │ 200   │
                      └─────┴──────────────┴───────┘
```

**SQL Implementation:**

```sql
-- Parent table (one side)
CREATE TABLE customers (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Child table (many side)
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL,  -- No UNIQUE (allows many)
  total DECIMAL(10, 2) NOT NULL,
  status VARCHAR(20) DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  FOREIGN KEY (customer_id) REFERENCES customers(id) ON DELETE RESTRICT
);

-- Insert data
INSERT INTO customers (name, email) VALUES
('John Doe', 'john@example.com'),
('Sarah Smith', 'sarah@example.com');

INSERT INTO orders (customer_id, total, status) VALUES
(1, 150.00, 'completed'),
(1, 75.50, 'pending'),
(2, 200.00, 'completed');

-- Query: Get customer with their orders
SELECT 
  c.name,
  c.email,
  o.id AS order_id,
  o.total,
  o.status,
  o.created_at
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
ORDER BY c.name, o.created_at DESC;

-- Query: Count orders per customer
SELECT 
  c.name,
  COUNT(o.id) AS order_count,
  COALESCE(SUM(o.total), 0) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name
ORDER BY total_spent DESC;
```

**Result:**

*Query 1:*
| name        | email                                         | order_id | total  | status    | created_at          |
| ----------- | --------------------------------------------- | -------- | ------ | --------- | ------------------- |
| John Doe    | [john@example.com](mailto:john@example.com)   | 2        | 75.50  | pending   | 2026-01-08 10:15:00 |
| John Doe    | [john@example.com](mailto:john@example.com)   | 1        | 150.00 | completed | 2026-01-07 18:30:00 |
| Sarah Smith | [sarah@example.com](mailto:sarah@example.com) | 3        | 200.00 | completed | 2026-01-08 09:45:00 |

---

*Query 2:*
| name        | order_count | total_spent |
| ----------- | ----------- | ----------- |
| John Doe    | 2           | 225.50      |
| Sarah Smith | 1           | 200.00      |

**Key points:**
- Foreign key in "many" side (child table)
- No UNIQUE constraint on foreign key
- Use LEFT JOIN to include customers with no orders

---

### 2.1.3 Many-to-Many (M:N)

**Definition:** Many records in Table A can relate to many records in Table B, and vice versa.

**Requires a junction table (also called bridge table or associative table)!**

**Real-world examples:**
- Students ↔ Courses (students take many courses, courses have many students)
- Products ↔ Tags (products have many tags, tags apply to many products)
- Actors ↔ Movies (actors in many movies, movies have many actors)
- Authors ↔ Books (for co-authorship)

**Visual representation:**
```
students              enrollments (junction)      courses
┌─────┬───────┐      ┌─────┬────────────┬───────────┐      ┌─────┬─────────┐
│ id  │ name  │  M:N │ id  │ student_id │ course_id │  M:N │ id  │ name    │
├─────┼───────┤ ◄─── ├─────┼────────────┼───────────┤ ───► ├─────┼─────────┤
│  1  │ Alice │      │  1  │     1      │     1     │      │  1  │ Math    │
│  2  │ Bob   │      │  2  │     1      │     2     │      │  2  │ Physics │
└─────┴───────┘      │  3  │     2      │     1     │      │  3  │ CS      │
                     │  4  │     2      │     3     │      └─────┴─────────┘
                     └─────┴────────────┴───────────┘

Alice takes: Math, Physics
Bob takes: Math, CS
Math has: Alice, Bob
```

**SQL Implementation:**

```sql
-- First table
CREATE TABLE students (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL
);

-- Second table
CREATE TABLE courses (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  code VARCHAR(20) UNIQUE NOT NULL,
  credits INTEGER NOT NULL
);

-- Junction table (many-to-many)
CREATE TABLE enrollments (
  id SERIAL PRIMARY KEY,
  student_id INTEGER NOT NULL,
  course_id INTEGER NOT NULL,
  enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  grade VARCHAR(2),
  
  -- Foreign keys to both tables
  FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
  FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE,
  
  -- Prevent duplicate enrollments
  UNIQUE (student_id, course_id)
);

-- Insert data
INSERT INTO students (name, email) VALUES
('Alice Johnson', 'alice@example.com'),
('Bob Smith', 'bob@example.com'),
('Carol White', 'carol@example.com');

INSERT INTO courses (name, code, credits) VALUES
('Mathematics', 'MATH101', 3),
('Physics', 'PHYS101', 4),
('Computer Science', 'CS101', 3);

-- Enroll students in courses
INSERT INTO enrollments (student_id, course_id) VALUES
(1, 1),  -- Alice → Math
(1, 2),  -- Alice → Physics
(2, 1),  -- Bob → Math
(2, 3),  -- Bob → CS
(3, 2),  -- Carol → Physics
(3, 3);  -- Carol → CS

-- Query: Get all courses for a student
SELECT 
  s.name AS student_name,
  c.name AS course_name,
  c.code,
  c.credits
FROM students s
JOIN enrollments e ON s.id = e.student_id
JOIN courses c ON e.course_id = c.id
WHERE s.id = 1
ORDER BY c.name;

-- Query: Get all students in a course
SELECT 
  c.name AS course_name,
  s.name AS student_name,
  s.email
FROM courses c
JOIN enrollments e ON c.id = e.course_id
JOIN students s ON e.student_id = s.id
WHERE c.code = 'MATH101'
ORDER BY s.name;

-- Query: Students with course count
SELECT 
  s.name,
  COUNT(e.id) AS courses_enrolled,
  SUM(c.credits) AS total_credits
FROM students s
LEFT JOIN enrollments e ON s.id = e.student_id
LEFT JOIN courses c ON e.course_id = c.id
GROUP BY s.id, s.name
ORDER BY courses_enrolled DESC;
```

**Result:**

*Query 1:*
| student_name  | course_name | code    | credits |
| ------------- | ----------- | ------- | ------- |
| Alice Johnson | Mathematics | MATH101 | 3       |
| Alice Johnson | Physics     | PHYS101 | 4       |

---

*Query 2:*
| course_name | student_name  | email                                         |
| ----------- | ------------- | --------------------------------------------- |
| Mathematics | Alice Johnson | [alice@example.com](mailto:alice@example.com) |
| Mathematics | Bob Smith     | [bob@example.com](mailto:bob@example.com)     |

---

*Query 3:*
| name          | courses_enrolled | total_credits |
| ------------- | ---------------- | ------------- |
| Alice Johnson | 2                | 7             |
| Bob Smith     | 2                | 6             |
| Carol White   | 2                | 7             |


**Junction table requirements:**
- Foreign keys to both tables
- UNIQUE constraint on combination (student_id, course_id)
- Can include additional data (enrollment date, grade, etc.)

---

### 2.1.4 Foreign Keys

**Definition:** A column (or columns) that creates a link between two tables by referencing the primary key of another table.

**Purpose:**
- Maintain referential integrity
- Enforce relationships
- Prevent orphaned records
- Enable JOINs

---

#### Foreign Key Syntax

```sql
-- Basic foreign key
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL,
  
  FOREIGN KEY (customer_id) REFERENCES customers(id)
);

-- Alternative syntax (inline)
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL REFERENCES customers(id)
);

-- Named constraint (better for debugging)
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL,
  
  CONSTRAINT fk_orders_customer 
  FOREIGN KEY (customer_id) REFERENCES customers(id)
);
```

---

#### Referential Actions

**ON DELETE:** What happens when parent record is deleted?

**1. CASCADE** - Delete child records too
```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL,
  
  FOREIGN KEY (customer_id) REFERENCES customers(id) 
  ON DELETE CASCADE
);

-- Example:
DELETE FROM customers WHERE id = 1;
-- Also deletes: All orders for customer 1
```

**2. SET NULL** - Set foreign key to NULL
```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER,  -- Nullable
  
  FOREIGN KEY (customer_id) REFERENCES customers(id) 
  ON DELETE SET NULL
);

-- Example:
DELETE FROM customers WHERE id = 1;
-- Result: Orders remain but customer_id becomes NULL
```

**3. RESTRICT** - Prevent deletion if child records exist
```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL,
  
  FOREIGN KEY (customer_id) REFERENCES customers(id) 
  ON DELETE RESTRICT
);

-- Example:
DELETE FROM customers WHERE id = 1;
-- ERROR: Cannot delete customer with existing orders
```

**4. NO ACTION** - Similar to RESTRICT (check at end of transaction)
```sql
FOREIGN KEY (customer_id) REFERENCES customers(id) 
ON DELETE NO ACTION
```

---

#### ON UPDATE

**What happens when parent primary key is updated?**

```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL,
  
  FOREIGN KEY (customer_id) REFERENCES customers(id) 
  ON DELETE CASCADE
  ON UPDATE CASCADE  -- Update foreign key when parent key changes
);
```

---

#### Complete E-Commerce Example

```sql
-- Customers table (parent)
CREATE TABLE customers (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL
);

-- Orders table (child of customers, parent of order_items)
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER NOT NULL,
  total DECIMAL(10, 2) NOT NULL,
  status VARCHAR(20) DEFAULT 'pending',
  
  CONSTRAINT fk_orders_customer 
  FOREIGN KEY (customer_id) REFERENCES customers(id) 
  ON DELETE RESTRICT  -- Can't delete customer with orders
);

-- Products table (independent)
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(200) NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  stock INTEGER NOT NULL DEFAULT 0
);

-- Order items (child of orders and products)
CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER NOT NULL,
  product_id INTEGER NOT NULL,
  quantity INTEGER NOT NULL,
  price_at_order DECIMAL(10, 2) NOT NULL,  -- Snapshot of price
  
  CONSTRAINT fk_order_items_order 
  FOREIGN KEY (order_id) REFERENCES orders(id) 
  ON DELETE CASCADE,  -- Delete items when order deleted
  
  CONSTRAINT fk_order_items_product 
  FOREIGN KEY (product_id) REFERENCES products(id) 
  ON DELETE RESTRICT,  -- Can't delete product in orders
  
  UNIQUE (order_id, product_id)  -- One product per order
);

-- Test referential integrity
INSERT INTO customers (name, email) VALUES ('John', 'john@example.com');
INSERT INTO orders (customer_id, total) VALUES (1, 100.00);

-- ❌ This fails: foreign key violation
INSERT INTO orders (customer_id, total) VALUES (999, 50.00);
-- ERROR: customer 999 doesn't exist

-- ❌ This fails: can't delete customer with orders
DELETE FROM customers WHERE id = 1;
-- ERROR: orders reference this customer

-- ✅ This works: delete order first, then customer
DELETE FROM orders WHERE customer_id = 1;
DELETE FROM customers WHERE id = 1;
```

---

### Practice Questions: Database Relationships

#### Fill in the Blanks

1. A _________ relationship means one record relates to many records.
2. A _________ table is required to implement a many-to-many relationship.
3. The _________ key in the child table references the primary key in the parent table.
4. ON DELETE _________ will delete child records when parent is deleted.
5. A _________ constraint on a foreign key enforces a 1:1 relationship.

<details>
<summary><strong>View Answers</strong></summary>

1. one-to-many (1:N)
2. junction / bridge / associative
3. foreign
4. CASCADE
5. UNIQUE

</details>

---

#### True/False

1. One-to-many is the most common relationship type in databases.
2. A many-to-many relationship can be implemented without a junction table.
3. ON DELETE RESTRICT prevents deletion of parent records with children.
4. Foreign keys automatically create indexes in all databases.
5. A junction table in M:N relationships always requires exactly two foreign keys.

<details>
<summary><strong>View Answers</strong></summary>

1. TRUE
2. FALSE (requires junction table)
3. TRUE
4. FALSE (depends on database; PostgreSQL doesn't auto-index FKs)
5. TRUE (at minimum - can have additional columns)

</details>

---

#### Multiple Choice

**Q1: Which relationship type is represented by this scenario?**
"A blog post can have many comments, but each comment belongs to only one post."

A) One-to-One
B) One-to-Many
C) Many-to-Many
D) No relationship

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - One-to-Many**

Explanation: One post (parent) has many comments (children), but each comment has only one post. This is a classic 1:N relationship.

</details>

---

**Q2: What happens with ON DELETE CASCADE?**

```sql
FOREIGN KEY (customer_id) REFERENCES customers(id) 
ON DELETE CASCADE
```

A) Prevents deleting the parent record
B) Sets foreign key to NULL in child records
C) Deletes child records automatically
D) Does nothing

<details>
<summary><strong>View Answer</strong></summary>

**Answer: C - Deletes child records automatically**

Explanation: CASCADE means when you delete the parent (customer), all child records (orders) are automatically deleted too.

</details>

---

## 2.2 SQL JOINs

### What is a JOIN?

**Definition:** A JOIN combines rows from two or more tables based on a related column between them.

**Why JOINs?**
- Retrieve data from multiple tables in a single query
- Avoid data duplication (normalization)
- Maintain data integrity
- Enable complex queries

---

### Sample Data for Examples

```sql
-- Employees table
CREATE TABLE employees (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  department_id INTEGER,
  salary DECIMAL(10, 2)
);

INSERT INTO employees VALUES
(1, 'Alice', 1, 60000),
(2, 'Bob', 2, 75000),
(3, 'Carol', 1, 55000),
(4, 'David', NULL, 50000);  -- No department

-- Departments table
CREATE TABLE departments (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100)
);

INSERT INTO departments VALUES
(1, 'Engineering'),
(2, 'Marketing'),
(3, 'Finance');  -- No employees
```

---

### 2.2.1 INNER JOIN

**Definition:** Returns only rows where there's a match in BOTH tables.

**Visual representation (Venn diagram):**
```
┌─────────┐     ┌─────────┐
│    A    │     │    B    │
│    ╔════╧═════╧════╗    │
│    ║   MATCH       ║    │
│    ╚════╤═════╤════╝    │
│         │     │         │
└─────────┘     └─────────┘
   Only the overlapping part
```

**Syntax:**
```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.column = table2.column;
```

**Example:**
```sql
SELECT 
  e.name AS employee_name,
  d.name AS department_name,
  e.salary
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

**Result:**
```
employee_name | department_name | salary
-------------+-----------------+--------
Alice        | Engineering     | 60000
Bob          | Marketing       | 75000
Carol        | Engineering     | 55000

-- Note: David (no department) and Finance (no employees) excluded!
```

**When to use:**
- When you only want records that exist in BOTH tables
- Most common type of JOIN
- Default JOIN (can omit INNER keyword)

**Key points:**
- Excludes unmatched rows from both tables
- NULL values in join column won't match
- Can use multiple conditions: `ON t1.a = t2.a AND t1.b = t2.b`

---

### 2.2.2 LEFT JOIN (LEFT OUTER JOIN)

**Definition:** Returns ALL rows from the left table, and matched rows from the right table. NULL for non-matches from right.

**Visual representation:**
```
┌─────────┐     ┌─────────┐
│   ALL   │     │    B    │
│  ╔══════╧═════╧════╗    │
│  ║   MATCH         ║    │
│  ╚══════╤═════╤════╝    │
│         │     │         │
└─────────┘     └─────────┘
   All of left + matching right
```

**Syntax:**
```sql
SELECT columns
FROM table1
LEFT JOIN table2 ON table1.column = table2.column;
```

**Example:**
```sql
SELECT 
  e.name AS employee_name,
  d.name AS department_name,
  e.salary
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

**Result:**
```
employee_name | department_name | salary
-------------+-----------------+--------
Alice        | Engineering     | 60000
Bob          | Marketing       | 75000
Carol        | Engineering     | 55000
David        | NULL            | 50000  ← Included even though no department!

-- All employees included, even those without departments
```

**Common use case: Find records WITHOUT matches**
```sql
-- Find employees without a department
SELECT 
  e.name,
  e.salary
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;

-- Result:
-- name  | salary
-- ------+--------
-- David | 50000
```

**When to use:**
- When you want ALL records from the left table
- Finding records without matches (WHERE right.id IS NULL)
- Master-detail reports (show all masters even without details)

---

### 2.2.3 RIGHT JOIN (RIGHT OUTER JOIN)

**Definition:** Returns ALL rows from the right table, and matched rows from the left table. NULL for non-matches from left.

**Visual representation:**
```
┌─────────┐     ┌─────────┐
│    A    │     │   ALL   │
│    ╔════╧═════╧══════╗  │
│    ║   MATCH         ║  │
│    ╚════╤═════╤══════╝  │
│         │     │         │
└─────────┘     └─────────┘
   All of right + matching left
```

**Syntax:**
```sql
SELECT columns
FROM table1
RIGHT JOIN table2 ON table1.column = table2.column;
```

**Example:**
```sql
SELECT 
  e.name AS employee_name,
  d.name AS department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

**Result:**
```
employee_name | department_name
-------------+-----------------
Alice        | Engineering
Carol        | Engineering
Bob          | Marketing
NULL         | Finance         ← No employees in Finance!

-- All departments included, even those without employees
```

**Find departments without employees:**
```sql
SELECT 
  d.name AS department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id
WHERE e.id IS NULL;

-- Result:
-- department_name
-- ----------------
-- Finance
```

**Note:** Most developers prefer LEFT JOIN with tables flipped:
```sql
-- These are equivalent:
SELECT * FROM employees e RIGHT JOIN departments d ON e.department_id = d.id;
SELECT * FROM departments d LEFT JOIN employees e ON d.id = e.department_id;
```

---

### 2.2.4 FULL OUTER JOIN

**Definition:** Returns ALL rows from BOTH tables. NULL where no match.

**Visual representation:**
```
┌─────────┐     ┌─────────┐
│   ALL   │     │   ALL   │
│  ╔══════╧═════╧══════╗  │
│  ║     MATCH         ║  │
│  ╚══════╤═════╤══════╝  │
│         │     │         │
└─────────┘     └─────────┘
   Everything from both tables
```

**Syntax:**
```sql
SELECT columns
FROM table1
FULL OUTER JOIN table2 ON table1.column = table2.column;
```

**Example:**
```sql
SELECT 
  e.name AS employee_name,
  d.name AS department_name,
  e.salary
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;
```

**Result:**
```
employee_name | department_name | salary
-------------+-----------------+--------
Alice        | Engineering     | 60000
Carol        | Engineering     | 55000
Bob          | Marketing       | 75000
David        | NULL            | 50000  ← No department
NULL         | Finance         | NULL   ← No employees

-- All employees AND all departments included!
```

**Find all mismatches:**
```sql
-- Find employees without departments OR departments without employees
SELECT 
  e.name AS employee_name,
  d.name AS department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id
WHERE e.id IS NULL OR d.id IS NULL;

-- Result:
-- employee_name | department_name
-- -------------+-----------------
-- David        | NULL            (no department)
-- NULL         | Finance         (no employees)
```

**When to use:**
- When you want EVERYTHING from both tables
- Finding all mismatches
- Data reconciliation
- Rare in typical applications

---

### 2.2.5 CROSS JOIN

**Definition:** Returns the Cartesian product (all possible combinations) of two tables.

**Visual representation:**
```
Table A (2 rows) × Table B (3 rows) = 6 rows

A: [1, 2]
B: [X, Y, Z]

Result: [(1,X), (1,Y), (1,Z), (2,X), (2,Y), (2,Z)]
```

**Syntax:**
```sql
SELECT columns
FROM table1
CROSS JOIN table2;

-- Or without JOIN keyword:
SELECT columns FROM table1, table2;
```

**Example:**
```sql
-- Sample data
CREATE TABLE sizes (
  id SERIAL PRIMARY KEY,
  name VARCHAR(10)
);

CREATE TABLE colors (
  id SERIAL PRIMARY KEY,
  name VARCHAR(20)
);

INSERT INTO sizes (name) VALUES ('S'), ('M'), ('L');
INSERT INTO colors (name) VALUES ('Red'), ('Blue'), ('Green');

-- Cross join: All size-color combinations
SELECT 
  s.name AS size,
  c.name AS color
FROM sizes s
CROSS JOIN colors c;
```

**Result:**
```
size | color
-----+-------
S    | Red
S    | Blue
S    | Green
M    | Red
M    | Blue
M    | Green
L    | Red
L    | Blue
L    | Green

-- 3 sizes × 3 colors = 9 combinations
```

**Real-world use cases:**

**1. Generate product variants**
```sql
-- Create all possible T-shirt variants
SELECT 
  p.name AS product,
  s.name AS size,
  c.name AS color,
  p.base_price AS price
FROM products p
CROSS JOIN sizes s
CROSS JOIN colors c
WHERE p.name = 'T-Shirt';
```

**2. Generate time slots**
```sql
-- Generate appointment slots
SELECT 
  d.date,
  t.time_slot
FROM dates d
CROSS JOIN time_slots t
WHERE d.date BETWEEN '2024-01-01' AND '2024-01-07';
```

**3. Test data generation**
```sql
-- Generate test combinations
SELECT 
  u.name AS user,
  p.name AS product
FROM test_users u
CROSS JOIN test_products p;
```

**⚠️ Warning:** 
- Can generate HUGE result sets (1000 × 1000 = 1,000,000 rows!)
- Usually unintentional if you forget WHERE clause
- Use with caution on large tables

---

### 2.2.6 Self JOIN

**Definition:** A table joined with itself. Useful for hierarchical or comparative data.

**Use cases:**
- Organization charts (employees and managers)
- Finding relationships within same table
- Comparing rows within a table

**Example 1: Employee-Manager Hierarchy**

```sql
-- Employees table with manager reference
CREATE TABLE employees (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  manager_id INTEGER,  -- References employees(id)
  FOREIGN KEY (manager_id) REFERENCES employees(id)
);

INSERT INTO employees VALUES
(1, 'Alice', NULL),      -- CEO, no manager
(2, 'Bob', 1),           -- Reports to Alice
(3, 'Carol', 1),         -- Reports to Alice
(4, 'David', 2),         -- Reports to Bob
(5, 'Emma', 2);          -- Reports to Bob

-- Find each employee with their manager
SELECT 
  e.name AS employee,
  m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

**Result:**
```
employee | manager
---------+---------
Alice    | NULL     (CEO, no manager)
Bob      | Alice
Carol    | Alice
David    | Bob
Emma     | Bob
```

**Example 2: Find employees in same department**

```sql
SELECT 
  e1.name AS employee1,
  e2.name AS employee2,
  e1.department_id
FROM employees e1
JOIN employees e2 ON e1.department_id = e2.department_id
WHERE e1.id < e2.id  -- Avoid duplicates and self-matches
ORDER BY e1.department_id, e1.name;
```

**Example 3: Find pairs of employees with similar salaries**

```sql
SELECT 
  e1.name AS employee1,
  e1.salary AS salary1,
  e2.name AS employee2,
  e2.salary AS salary2,
  ABS(e1.salary - e2.salary) AS salary_diff
FROM employees e1
JOIN employees e2 ON e1.id < e2.id
WHERE ABS(e1.salary - e2.salary) < 5000
ORDER BY salary_diff;
```

---

### JOIN Comparison Table

| JOIN Type | Left Table | Right Table | Use Case |
|-----------|-----------|-------------|----------|
| **INNER** | Matched only | Matched only | Only records in both |
| **LEFT** | ALL | Matched only | All from left + matches |
| **RIGHT** | Matched only | ALL | All from right + matches |
| **FULL OUTER** | ALL | ALL | Everything from both |
| **CROSS** | ALL | ALL | All combinations |
| **SELF** | Same table | Same table | Hierarchies, comparisons |

---

### 2.2.7 Multiple JOINs

**Definition:** Join more than 2 tables.

**Example: E-commerce order details**

```sql
SELECT 
    c.name AS customer_name,
    o.order_id,
    o.order_date,
    p.product_name,
    oi.quantity,
    oi.price,
    (oi.quantity * oi.price) AS item_total
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
WHERE c.email = 'john@email.com'
ORDER BY o.order_date DESC, p.product_name;

Result:
customer_name | order_id | order_date | product_name | quantity | price  | item_total
──────────────┼──────────┼────────────┼──────────────┼──────────┼────────┼───────────
John Doe      | 3        | 2024-01-15 | iPhone       | 1        | 999.00 | 999.00
John Doe      | 3        | 2024-01-15 | AirPods      | 2        | 199.00 | 398.00
John Doe      | 2        | 2024-01-10 | iPad         | 1        | 599.00 | 599.00
```

---

### 2.2.8 JOIN Performance

#### 1. Index Foreign Keys
```sql
-- Always index columns used in JOINs
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);

-- Check index usage
EXPLAIN ANALYZE
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

#### 2. Filter Before JOIN
```sql
-- ❌ Bad: JOIN then filter
SELECT c.name, o.order_id
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date > '2024-01-01';

-- ✅ Better: Filter first (if possible)
SELECT c.name, o.order_id
FROM customers c
INNER JOIN (
    SELECT * FROM orders WHERE order_date > '2024-01-01'
) o ON c.customer_id = o.customer_id;
```

#### 3. Choose Appropriate JOIN Type
```sql
-- If you only need matching rows, use INNER JOIN
-- Don't use LEFT JOIN if you'll filter out NULLs later

-- ❌ Bad
SELECT c.name, o.order_id
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NOT NULL;  -- Filtering NULLs = waste

-- ✅ Good
SELECT c.name, o.order_id
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;
```

#### 4. Limit Result Set Size
```sql
-- Add LIMIT for large queries
SELECT *
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
ORDER BY o.order_date DESC
LIMIT 100;
```

---

### Practice Questions: SQL JOINs

#### Fill in the Blanks

1. _________ JOIN returns only rows where there's a match in both tables.
2. _________ JOIN returns all rows from the left table and matched rows from the right.
3. A _________ JOIN produces the Cartesian product of two tables.
4. To find employees without managers, use a _________ JOIN with WHERE manager_id IS NULL.
5. When a table joins with itself, it's called a _________ JOIN.

<details>
<summary><strong>View Answers</strong></summary>

1. INNER
2. LEFT
3. CROSS
4. LEFT / SELF
5. SELF

</details>

---

#### True/False

1. INNER JOIN and JOIN are the same thing.
2. LEFT JOIN always returns more rows than INNER JOIN.
3. CROSS JOIN requires an ON clause.
4. FULL OUTER JOIN is supported by all databases.
5. Self JOINs can only be used with employee-manager relationships.
6. Foreign keys automatically create indexes.

<details>
<summary><strong>View Answers</strong></summary>

1. TRUE (INNER is default)
2. FALSE (same count if all rows match)
3. FALSE (no ON clause needed)
4. FALSE (MySQL doesn't support FULL OUTER JOIN)
5. FALSE (any hierarchical or comparative data)
6. FALSE (you must create indexes manually in most databases)

</details>

---

#### Coding Challenges

**Challenge 1: Basic JOIN**

Given tables `customers` and `orders`, write a query to show customer names with their order totals. Include only customers who have placed orders.

<details>
<summary><strong>View Solution</strong></summary>

```sql
SELECT 
  c.name,
  o.id AS order_id,
  o.total
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id
ORDER BY c.name, o.id;
```

</details>

---

**Challenge 2: Find Records Without Matches**

Find all customers who have NOT placed any orders.

<details>
<summary><strong>View Solution</strong></summary>

```sql
SELECT 
  c.id,
  c.name,
  c.email
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;
```

</details>

---

**Challenge 3: Multiple JOINs**

Show order details including customer name, product name, and quantity.

Tables: `customers`, `orders`, `order_items`, `products`

<details>
<summary><strong>View Solution</strong></summary>

```sql
SELECT 
  c.name AS customer_name,
  o.id AS order_id,
  p.name AS product_name,
  oi.quantity,
  oi.price AS unit_price,
  (oi.quantity * oi.price) AS line_total
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
ORDER BY o.id, p.name;
```

</details>

---

**Challenge 4: Self JOIN - Organizational Chart**

Show employees with their managers and manager's managers (2 levels).

<details>
<summary><strong>View Solution</strong></summary>

```sql
SELECT 
  e.name AS employee,
  m.name AS manager,
  gm.name AS grand_manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id
LEFT JOIN employees gm ON m.manager_id = gm.id
ORDER BY e.name;
```

</details>

---

## 2.3 Aggregate Functions & GROUP BY

### Overview

Aggregate functions perform calculations on sets of rows and return a single value.

**Common aggregate functions:**
- `COUNT()` - Count rows
- `SUM()` - Sum values
- `AVG()` - Average value
- `MIN()` - Minimum value
- `MAX()` - Maximum value

---

### 2.3.1 COUNT, SUM, AVG, MIN, MAX

```sql
-- Sample data
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER,
  product_name VARCHAR(100),
  quantity INTEGER,
  price DECIMAL(10, 2),
  order_date DATE
);

INSERT INTO orders VALUES
(1, 1, 'Laptop', 1, 1000.00, '2024-01-15'),
(2, 1, 'Mouse', 2, 25.00, '2024-01-15'),
(3, 2, 'Keyboard', 1, 75.00, '2024-01-16'),
(4, 2, 'Monitor', 2, 300.00, '2024-01-16'),
(5, 3, 'Laptop', 1, 1000.00, '2024-01-17');

-- COUNT: Count all orders
SELECT COUNT(*) AS total_orders FROM orders;
-- Result: 5

-- COUNT with condition
SELECT COUNT(*) AS laptop_orders 
FROM orders 
WHERE product_name = 'Laptop';
-- Result: 2

-- COUNT DISTINCT: Count unique users
SELECT COUNT(DISTINCT user_id) AS unique_customers 
FROM orders;
-- Result: 3

-- SUM: Total revenue
SELECT SUM(quantity * price) AS total_revenue 
FROM orders;
-- Result: 2725.00

-- AVG: Average order value
SELECT AVG(quantity * price) AS avg_order_value 
FROM orders;
-- Result: 545.00

-- MIN and MAX: Price range
SELECT 
  MIN(price) AS min_price,
  MAX(price) AS max_price
FROM orders;
-- Result: min_price=25.00, max_price=1000.00
```

---

### 2.3.2 GROUP BY Basics

**Definition:** `GROUP BY` groups rows with same values into summary rows.

```sql
-- Group by user_id: Orders per customer
SELECT 
  user_id,
  COUNT(*) AS order_count,
  SUM(quantity * price) AS total_spent
FROM orders
GROUP BY user_id;

-- Result:
-- user_id | order_count | total_spent
-- --------+-------------+-------------
--    1    |      2      |   1050.00
--    2    |      2      |    675.00
--    3    |      1      |   1000.00

-- Group by product: Most popular products
SELECT 
  product_name,
  COUNT(*) AS times_ordered,
  SUM(quantity) AS total_quantity,
  AVG(price) AS avg_price
FROM orders
GROUP BY product_name
ORDER BY times_ordered DESC;

-- Group by date: Daily sales
SELECT 
  order_date,
  COUNT(*) AS orders_count,
  SUM(quantity * price) AS daily_revenue
FROM orders
GROUP BY order_date
ORDER BY order_date;
```

**Key Rules:**
1. Every column in SELECT must be either:
   - In GROUP BY clause, OR
   - Inside an aggregate function

```sql
-- ❌ WRONG: user_id not in GROUP BY or aggregate
SELECT user_id, product_name, COUNT(*) 
FROM orders 
GROUP BY product_name;
-- ERROR: column "user_id" must appear in GROUP BY

-- ✅ CORRECT: All non-aggregated columns in GROUP BY
SELECT user_id, product_name, COUNT(*) 
FROM orders 
GROUP BY user_id, product_name;
```

---

### 2.3.3 HAVING vs WHERE

**WHERE:** Filters rows BEFORE grouping
**HAVING:** Filters groups AFTER grouping

```sql
-- Sample data
CREATE TABLE sales (
  id SERIAL PRIMARY KEY,
  salesperson VARCHAR(100),
  product VARCHAR(100),
  amount DECIMAL(10, 2),
  sale_date DATE
);

INSERT INTO sales VALUES
(1, 'Alice', 'Laptop', 1000.00, '2024-01-15'),
(2, 'Alice', 'Mouse', 25.00, '2024-01-15'),
(3, 'Bob', 'Keyboard', 75.00, '2024-01-16'),
(4, 'Bob', 'Monitor', 300.00, '2024-01-16'),
(5, 'Carol', 'Laptop', 1000.00, '2024-01-17'),
(6, 'Carol', 'Mouse', 25.00, '2024-01-17');

-- WHERE: Filter before grouping
SELECT 
  salesperson,
  COUNT(*) AS sales_count,
  SUM(amount) AS total_sales
FROM sales
WHERE amount > 50  -- Filters rows BEFORE grouping
GROUP BY salesperson;

-- Result:
-- salesperson | sales_count | total_sales
-- ------------+-------------+-------------
-- Alice       |      1      |   1000.00
-- Bob         |      2      |    375.00
-- Carol       |      1      |   1000.00

-- HAVING: Filter after grouping
SELECT 
  salesperson,
  COUNT(*) AS sales_count,
  SUM(amount) AS total_sales
FROM sales
GROUP BY salesperson
HAVING SUM(amount) > 500;  -- Filters groups AFTER aggregation

-- Result:
-- salesperson | sales_count | total_sales
-- ------------+-------------+-------------
-- Alice       |      2      |   1025.00
-- Carol       |      2      |   1025.00

-- Combining WHERE and HAVING
SELECT 
  salesperson,
  COUNT(*) AS high_value_sales,
  SUM(amount) AS total_high_value
FROM sales
WHERE amount > 100  -- First: Keep only sales > $100
GROUP BY salesperson
HAVING COUNT(*) > 1;  -- Then: Keep only salespeople with 2+ such sales

-- Result:
-- salesperson | high_value_sales | total_high_value
-- ------------+------------------+------------------
-- Bob         |        2         |     375.00
```

**Execution order:**
```
1. FROM - Get data from tables
2. WHERE - Filter rows
3. GROUP BY - Group rows
4. HAVING - Filter groups
5. SELECT - Select columns and apply aggregates
6. ORDER BY - Sort results
7. LIMIT - Limit rows returned
```

---

### 2.3.4 Multiple Aggregations

**Real-world example: E-commerce analytics**

```sql
-- Order statistics per customer
SELECT 
  user_id,
  COUNT(*) AS total_orders,
  COUNT(DISTINCT product_name) AS unique_products,
  SUM(quantity) AS total_items,
  SUM(quantity * price) AS total_spent,
  AVG(quantity * price) AS avg_order_value,
  MIN(order_date) AS first_order,
  MAX(order_date) AS last_order
FROM orders
GROUP BY user_id
HAVING SUM(quantity * price) > 500
ORDER BY total_spent DESC;

-- Product performance
SELECT 
  product_name,
  COUNT(*) AS times_ordered,
  SUM(quantity) AS units_sold,
  SUM(quantity * price) AS revenue,
  AVG(price) AS avg_price,
  MIN(price) AS min_price,
  MAX(price) AS max_price
FROM orders
GROUP BY product_name
ORDER BY revenue DESC;

-- GROUP BY with JOINs
SELECT 
  c.name,
  COUNT(DISTINCT o.id) AS order_count,
  SUM(o.total) AS lifetime_value,
  AVG(o.total) AS avg_order_value
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name
ORDER BY lifetime_value DESC;
```

---

### Practice Questions: Aggregate Functions & GROUP BY

#### Fill in the Blanks

1. The ________ function counts the number of rows in a result set.
2. The ________ function calculates the average of numeric values.
3. ________ filters rows before grouping, while ________ filters groups after aggregation.
4. Every column in SELECT must be either in ________ clause or inside an ________ function.
5. To count only unique values, use ________ keyword before the column name.

<details>
<summary><strong>View Answers</strong></summary>

1. COUNT() / COUNT(*)
2. AVG()
3. WHERE, HAVING
4. GROUP BY, aggregate
5. DISTINCT / COUNT(DISTINCT column)

</details>

---

#### Coding Challenges

**Challenge 1: Basic Aggregation**

Find total products, average price, and inventory value from `products` table.

<details>
<summary><strong>View Solution</strong></summary>

```sql
SELECT 
  COUNT(*) AS total_products,
  AVG(price) AS avg_price,
  MAX(price) AS max_price,
  SUM(price * stock) AS inventory_value
FROM products;
```

</details>

---

**Challenge 2: GROUP BY with HAVING**

Find categories with average price > $50, at least 10 products, and inventory value > $10,000.

<details>
<summary><strong>View Solution</strong></summary>

```sql
SELECT 
  category,
  COUNT(*) AS product_count,
  AVG(price) AS avg_price,
  SUM(price * stock) AS inventory_value
FROM products
GROUP BY category
HAVING AVG(price) > 50
  AND COUNT(*) >= 10
  AND SUM(price * stock) > 10000
ORDER BY inventory_value DESC;
```

</details>

---

## 2.4 Advanced SQL Queries

### 2.4.1 Subqueries (Nested Queries)

**Definition:** A query inside another query.

#### Types of Subqueries

**1. Scalar Subquery (Returns single value)**

```sql
-- Find products more expensive than average
SELECT name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

**2. Column Subquery (Returns single column)**

```sql
-- Find users who placed orders
SELECT name, email
FROM users
WHERE id IN (SELECT DISTINCT user_id FROM orders);
```

**3. Correlated Subquery**

```sql
-- Find products more expensive than category average
SELECT p.name, p.price, p.category_id
FROM products p
WHERE p.price > (
  SELECT AVG(price)
  FROM products
  WHERE category_id = p.category_id
);
```

**EXISTS vs IN**

```sql
-- EXISTS (faster for large datasets)
SELECT name, email
FROM users u
WHERE EXISTS (
  SELECT 1 
  FROM orders o 
  WHERE o.user_id = u.id
);

-- NOT EXISTS (find users without orders)
SELECT name, email
FROM users u
WHERE NOT EXISTS (
  SELECT 1 
  FROM orders o 
  WHERE o.user_id = u.id
);
```

---

### 2.4.2 Common Table Expressions (CTEs)

**Definition:** Named temporary result sets (WITH clause).

```sql
-- Calculate user lifetime value
WITH user_stats AS (
  SELECT 
    user_id,
    COUNT(*) as order_count,
    SUM(total) as lifetime_value
  FROM orders
  GROUP BY user_id
)
SELECT 
  u.name,
  s.order_count,
  s.lifetime_value
FROM users u
JOIN user_stats s ON u.id = s.user_id
WHERE s.lifetime_value > 1000;

-- Multiple CTEs
WITH top_products AS (
  SELECT product_id, SUM(quantity) as total_sold
  FROM order_items
  GROUP BY product_id
  ORDER BY total_sold DESC
  LIMIT 10
),
top_customers AS (
  SELECT user_id, SUM(total) as total_spent
  FROM orders
  GROUP BY user_id
  ORDER BY total_spent DESC
  LIMIT 10
)
SELECT * FROM top_products, top_customers;

-- Recursive CTE (organizational hierarchy)
WITH RECURSIVE employee_hierarchy AS (
  SELECT id, name, manager_id, 0 as level
  FROM employees
  WHERE manager_id IS NULL
  
  UNION ALL
  
  SELECT e.id, e.name, e.manager_id, eh.level + 1
  FROM employees e
  JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM employee_hierarchy ORDER BY level;
```

---

### 2.4.3 Window Functions

**Definition:** Perform calculations across rows related to current row WITHOUT grouping.

```sql
-- ROW_NUMBER, RANK, DENSE_RANK
SELECT 
  salesperson,
  amount,
  ROW_NUMBER() OVER (ORDER BY amount DESC) as row_num,
  RANK() OVER (ORDER BY amount DESC) as rank,
  DENSE_RANK() OVER (ORDER BY amount DESC) as dense_rank
FROM sales;

-- PARTITION BY (rank within groups)
SELECT 
  category,
  name,
  price,
  RANK() OVER (PARTITION BY category ORDER BY price DESC) as price_rank
FROM products;

-- LAG and LEAD (previous/next row)
SELECT 
  month,
  revenue,
  LAG(revenue) OVER (ORDER BY month) as prev_month,
  revenue - LAG(revenue) OVER (ORDER BY month) as change
FROM monthly_sales;

-- Running total
SELECT 
  order_date,
  total,
  SUM(total) OVER (ORDER BY order_date) as running_total
FROM orders;
```

---

### 2.4.4 UNION, INTERSECT, EXCEPT

**UNION (Remove duplicates)**

```sql
SELECT email FROM customers
UNION
SELECT email FROM employees;
```

**UNION ALL (Keep duplicates, faster)**

```sql
SELECT * FROM orders_2023
UNION ALL
SELECT * FROM orders_2024;
```

**INTERSECT (Common rows)**

```sql
SELECT email FROM customers
INTERSECT
SELECT email FROM employees;
```

**EXCEPT (Rows in first but not second)**

```sql
SELECT id FROM users
EXCEPT
SELECT DISTINCT user_id FROM orders;
```

---

### Practice Questions: Advanced SQL

#### Coding Challenge: Top 3 per Category

Write a query to find the top 3 most expensive products in each category.

<details>
<summary><strong>View Solution</strong></summary>

```sql
SELECT category, name, price
FROM (
  SELECT 
    category,
    name,
    price,
    RANK() OVER (PARTITION BY category ORDER BY price DESC) as price_rank
  FROM products
) ranked
WHERE price_rank <= 3
ORDER BY category, price_rank;
```

</details>

---

## 2.5 Transactions & Isolation Levels

### 2.5.1 Transaction Basics

**Definition:** A sequence of operations performed as a single logical unit of work.

```sql
-- Basic transaction
BEGIN TRANSACTION;

  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;  -- Make permanent

-- Or ROLLBACK to undo
ROLLBACK;
```

---

### 2.5.2 Isolation Levels

**1. READ UNCOMMITTED** - Allows dirty reads (almost never use)

**2. READ COMMITTED** - Default, prevents dirty reads

**3. REPEATABLE READ** - Prevents non-repeatable reads

**4. SERIALIZABLE** - Most strict, prevents all anomalies

```sql
-- Set isolation level
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

  SELECT * FROM products WHERE id = 1 FOR UPDATE;
  UPDATE products SET stock = stock - 1 WHERE id = 1;

COMMIT;
```

---

### 2.5.3 Locks and Deadlocks

**Types of Locks:**

```sql
-- Shared lock (read)
SELECT * FROM products WHERE id = 1 FOR SHARE;

-- Exclusive lock (write)
SELECT * FROM products WHERE id = 1 FOR UPDATE;
```

**Preventing Deadlocks:**

1. Lock in same order
2. Keep transactions short
3. Use lower isolation levels when possible

---

## Chapter Summary

**2.1 Database Relationships**
- One-to-One, One-to-Many, Many-to-Many
- Foreign keys with CASCADE, RESTRICT, SET NULL

**2.2 SQL JOINs**
- INNER, LEFT, RIGHT, FULL OUTER, CROSS, Self
- Visual Venn diagrams for each type

**2.3 Aggregates & GROUP BY**
- COUNT, SUM, AVG, MIN, MAX
- WHERE vs HAVING

**2.4 Advanced Queries**
- Subqueries, CTEs, Window Functions
- Set operations

**2.5 Transactions**
- ACID properties
- Four isolation levels
- Deadlock prevention

---

**Navigation:** [← Introduction](01_fundamentals.md) | [README](../README.md) | [Next: NoSQL Deep Dive: MongoDB →](03_01_mongodb.md)