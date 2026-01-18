# Database Fundamentals

## 1.2 Database Relationships

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

### 1.2.1 One-to-One (1:1)

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

### 1.2.2 One-to-Many (1:N)

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

### 1.2.3 Many-to-Many (M:N)

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

### 1.2.4 Foreign Keys

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