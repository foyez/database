# Database Schema & Keys

## 2.1 Primary Keys

### 2.1.1 Definition and Purpose

A **primary key** is a column (or set of columns) that uniquely identifies each row in a table. Think of it like a Social Security Number for database records—no two rows can have the same primary key value.

**Key Characteristics:**
- **Uniqueness**: No duplicate values allowed
- **NOT NULL**: Every row must have a primary key value
- **Immutability**: Should never change once set
- **Single per table**: Each table can have only one primary key

**Real-World Example:**
```sql
-- Amazon Orders Table
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10, 2),
    status VARCHAR(20)
);
```

In Amazon's system, each order needs a unique identifier. The `order_id` serves as the primary key, ensuring you can always distinguish between your coffee maker purchase and your laptop purchase, even if they were ordered at the same time.

**Why Primary Keys Matter:**
- **Fast lookups**: Databases automatically index primary keys
- **Relationships**: Other tables reference this key to create relationships
- **Data integrity**: Prevents duplicate records
- **Query optimization**: Database engines optimize queries using primary keys

### 2.1.2 Natural vs Surrogate Keys

**Natural Key**: A key that has business meaning and exists naturally in the data.

**Examples:**
```sql
-- Using email as natural key (NOT RECOMMENDED)
CREATE TABLE users (
    email VARCHAR(255) PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    created_at TIMESTAMP
);

-- Using ISBN for books (GOOD use case)
CREATE TABLE books (
    isbn VARCHAR(13) PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255),
    price DECIMAL(8, 2)
);

-- Using SSN (AVOID - privacy concerns)
CREATE TABLE employees (
    ssn CHAR(11) PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50)
);
```

**Problems with Natural Keys:**
- **Email example**: What if user wants to change their email? Updating a primary key affects all related tables
- **SSN example**: Privacy regulations make this problematic
- **Business rules change**: What seemed permanent might not be

**Surrogate Key**: An artificial key with no business meaning, typically an auto-incrementing integer.

```sql
-- Stripe-style payment processing
CREATE TABLE payments (
    payment_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Surrogate key
    payment_intent_id VARCHAR(50) UNIQUE NOT NULL, -- Natural identifier for API
    customer_email VARCHAR(255) NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    currency CHAR(3) DEFAULT 'USD',
    status VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Why Stripe uses both:**
- `payment_id`: Internal database optimization (surrogate)
- `payment_intent_id`: External API identifier (natural-like, but still surrogate)
- `customer_email`: Business data that can change

**Best Practices:**
- Use surrogate keys for primary keys (AUTO_INCREMENT integers or UUIDs)
- Keep natural identifiers as UNIQUE constraints
- Never expose internal IDs in URLs if security matters

### 2.1.3 Composite Primary Keys

A **composite primary key** uses multiple columns together to uniquely identify a row. Each column alone might not be unique, but the combination always is.

**Real-World Example: Netflix Viewing History**
```sql
-- Netflix tracks what you watch
CREATE TABLE viewing_history (
    user_id BIGINT NOT NULL,
    content_id BIGINT NOT NULL,
    watch_date DATE NOT NULL,
    watch_time_seconds INT DEFAULT 0,
    completed BOOLEAN DEFAULT FALSE,
    PRIMARY KEY (user_id, content_id, watch_date)
);
```

**Why composite here?**
- Same user can watch different content: `(user_id, content_id)` not unique
- Same user can rewatch same content on different days: need `watch_date`
- Together `(user_id, content_id, watch_date)` uniquely identifies each viewing session

**E-commerce: Order Line Items**
```sql
-- Amazon order details
CREATE TABLE order_items (
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    discount_amount DECIMAL(10, 2) DEFAULT 0,
    PRIMARY KEY (order_id, product_id)
);
```

Each product appears once per order. The combination of `order_id` and `product_id` is unique.

**Class Enrollment System:**
```sql
-- University course enrollment
CREATE TABLE enrollments (
    student_id INT NOT NULL,
    course_id VARCHAR(10) NOT NULL,
    semester VARCHAR(10) NOT NULL,
    grade CHAR(2),
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id, semester)
);

-- Valid data:
-- (12345, 'CS101', '2024-FALL', 'A')  ✓
-- (12345, 'CS101', '2025-SPRING', 'B') ✓ Same student, same course, different semester
-- (12345, 'CS102', '2024-FALL', 'A')  ✓ Same student, different course
```

**When to Use Composite Keys:**
- Junction tables (many-to-many relationships)
- Time-series data with natural grouping
- When the combination genuinely represents a unique business concept

**When to Avoid:**
- More than 3 columns gets unwieldy
- Any column might need to change
- Foreign keys become complex

**Alternative Approach:**
```sql
-- Same Netflix example with surrogate key
CREATE TABLE viewing_history (
    viewing_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Easier to reference
    user_id BIGINT NOT NULL,
    content_id BIGINT NOT NULL,
    watch_date DATE NOT NULL,
    watch_time_seconds INT DEFAULT 0,
    completed BOOLEAN DEFAULT FALSE,
    UNIQUE KEY unique_viewing (user_id, content_id, watch_date)  -- Still enforce uniqueness
);
```

This approach gives you:
- Simple single-column primary key
- Easy foreign key references
- Same uniqueness guarantee via UNIQUE constraint

### 2.1.4 AUTO_INCREMENT/SERIAL

**AUTO_INCREMENT** (MySQL) and **SERIAL** (PostgreSQL) automatically generate unique sequential numbers for new rows.

**MySQL Syntax:**
```sql
-- GitHub-style repository system
CREATE TABLE repositories (
    repo_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    owner_id BIGINT NOT NULL,
    repo_name VARCHAR(100) NOT NULL,
    description TEXT,
    is_private BOOLEAN DEFAULT FALSE,
    star_count INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Inserting without specifying repo_id
INSERT INTO repositories (owner_id, repo_name, description, is_private)
VALUES (1001, 'awesome-project', 'A cool project', FALSE);
-- repo_id automatically becomes 1

INSERT INTO repositories (owner_id, repo_name, description, is_private)
VALUES (1001, 'another-project', 'Another project', TRUE);
-- repo_id automatically becomes 2
```

**PostgreSQL Syntax:**
```sql
-- Same table in PostgreSQL
CREATE TABLE repositories (
    repo_id SERIAL PRIMARY KEY,  -- Equivalent to AUTO_INCREMENT
    owner_id BIGINT NOT NULL,
    repo_name VARCHAR(100) NOT NULL,
    description TEXT,
    is_private BOOLEAN DEFAULT FALSE,
    star_count INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Modern PostgreSQL alternative
CREATE TABLE repositories (
    repo_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    owner_id BIGINT NOT NULL,
    repo_name VARCHAR(100) NOT NULL
);
```

**How It Works:**
```sql
-- Starting fresh
INSERT INTO repositories (owner_id, repo_name) VALUES (100, 'repo-1');  -- Gets ID 1
INSERT INTO repositories (owner_id, repo_name) VALUES (100, 'repo-2');  -- Gets ID 2
INSERT INTO repositories (owner_id, repo_name) VALUES (100, 'repo-3');  -- Gets ID 3

DELETE FROM repositories WHERE repo_id = 2;  -- Delete middle row

INSERT INTO repositories (owner_id, repo_name) VALUES (100, 'repo-4');  -- Gets ID 4, NOT 2!
-- AUTO_INCREMENT never reuses deleted IDs by default
```

**Getting the Last Inserted ID:**

MySQL:
```sql
INSERT INTO repositories (owner_id, repo_name) VALUES (100, 'new-repo');
SELECT LAST_INSERT_ID();  -- Returns the repo_id that was just created
```

PostgreSQL:
```sql
INSERT INTO repositories (owner_id, repo_name) 
VALUES (100, 'new-repo') 
RETURNING repo_id;  -- More powerful, can return any column
```

**Real-World Example: Twitter-like Social Media**
```sql
-- Tweet storage
CREATE TABLE tweets (
    tweet_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    content VARCHAR(280) NOT NULL,
    reply_to_tweet_id BIGINT NULL,  -- For threading
    retweet_count INT DEFAULT 0,
    like_count INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user_tweets (user_id, created_at),
    FOREIGN KEY (reply_to_tweet_id) REFERENCES tweets(tweet_id)
);

-- Application code (pseudo-code)
tweet_id = db.execute(
    "INSERT INTO tweets (user_id, content) VALUES (?, ?)",
    user_id, tweet_content
).last_insert_id

return f"https://twitter.com/user/status/{tweet_id}"
```

**Gotchas and Best Practices:**

**1. Gaps in Sequence:**
```sql
-- This is normal and expected
INSERT INTO repositories (owner_id, repo_name) VALUES (100, 'repo-1');  -- ID: 1
BEGIN TRANSACTION;
    INSERT INTO repositories (owner_id, repo_name) VALUES (100, 'repo-2');  -- ID: 2
ROLLBACK;  -- Transaction rolled back, but ID 2 is "burned"
INSERT INTO repositories (owner_id, repo_name) VALUES (100, 'repo-3');  -- ID: 3 (not 2!)
```

**2. Exhausting the Range:**
```sql
-- INT range: -2,147,483,648 to 2,147,483,647
-- BIGINT range: -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807

-- Instagram/Twitter scale: Use BIGINT
CREATE TABLE posts (
    post_id BIGINT AUTO_INCREMENT PRIMARY KEY  -- Billions of posts
);

-- Small lookup table: INT is fine
CREATE TABLE post_types (
    type_id INT AUTO_INCREMENT PRIMARY KEY  -- Maybe 10-20 types total
);
```

**3. Resetting AUTO_INCREMENT:**
```sql
-- Dangerous! Only do on empty tables in development
ALTER TABLE repositories AUTO_INCREMENT = 1;

-- Check current value
SELECT AUTO_INCREMENT 
FROM information_schema.TABLES 
WHERE TABLE_NAME = 'repositories';
```

**4. Multi-Master Replication:**
```sql
-- Server 1 generates: 1, 3, 5, 7, 9...
-- Server 2 generates: 2, 4, 6, 8, 10...
SET @@auto_increment_increment = 2;
SET @@auto_increment_offset = 1;  -- Server 1
SET @@auto_increment_offset = 2;  -- Server 2
```

**UUIDs as Alternative:**
```sql
-- Distributed systems often prefer UUIDs
CREATE TABLE sessions (
    session_id CHAR(36) PRIMARY KEY DEFAULT (UUID()),
    user_id BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO sessions (user_id) VALUES (1001);
-- session_id: '550e8400-e29b-41d4-a716-446655440000'
```

**UUID vs AUTO_INCREMENT:**
- **AUTO_INCREMENT**: Sequential, smaller storage, better performance, predictable
- **UUID**: Globally unique, works across distributed systems, unpredictable (more secure), larger storage (36 bytes vs 8 bytes)

---

### Section 2.1 Practice Questions

<details>
<summary><strong>Question 1: Fill in the Blanks</strong></summary>

Complete the following statements:

a) A primary key must be both _______ and _______.

b) A _______ key has business meaning, while a _______ key is artificially generated.

c) In MySQL, _______ automatically generates sequential numbers, while PostgreSQL uses _______.

d) When using composite primary keys, the combination of columns must be _______, but individual columns can contain _______ values.

e) After a transaction rollback, AUTO_INCREMENT values are _______, creating _______ in the sequence.

</details>

<details>
<summary><strong>Question 2: True or False</strong></summary>

a) A table can have multiple primary keys.

b) Primary keys are automatically indexed by the database.

c) AUTO_INCREMENT always reuses deleted ID values to prevent gaps.

d) Natural keys like email addresses are always better choices for primary keys than surrogate keys.

e) Composite primary keys can only have exactly two columns.

f) SERIAL in PostgreSQL and AUTO_INCREMENT in MySQL serve the same purpose.

g) Once set, a primary key value should never be updated.

</details>

<details>
<summary><strong>Question 3: Multiple Choice</strong></summary>

**a) Which is the BEST primary key choice for a users table?**
- A) `email VARCHAR(255)`
- B) `user_id BIGINT AUTO_INCREMENT`
- C) `username VARCHAR(50)`
- D) `ssn CHAR(11)`

**b) What happens when you insert a row without specifying an AUTO_INCREMENT column?**
- A) The insert fails
- B) The value is set to NULL
- C) The database automatically generates the next sequential value
- D) The value is set to 0

**c) For a table tracking student course enrollments where students can retake courses, which primary key is most appropriate?**
- A) `student_id` alone
- B) `course_id` alone
- C) `(student_id, course_id)`
- D) `(student_id, course_id, semester)`

**d) Which scenario is BEST suited for a composite primary key?**
- A) User profile table
- B) Many-to-many junction table
- C) Product catalog
- D) Single transaction log

**e) What is the maximum value for BIGINT in MySQL?**
- A) 2,147,483,647
- B) 4,294,967,295
- C) 9,223,372,036,854,775,807
- D) 18,446,744,073,709,551,615

</details>

<details>
<summary><strong>Question 4: Code Analysis</strong></summary>

**What's wrong with this table design?**

```sql
CREATE TABLE products (
    product_name VARCHAR(100) PRIMARY KEY,
    sku VARCHAR(50) UNIQUE,
    price DECIMAL(10, 2),
    category VARCHAR(50)
);
```

List at least three issues with using `product_name` as the primary key.

</details>

<details>
<summary><strong>Question 5: Practical Scenario</strong></summary>

You're designing a table to track employee shift assignments. Each employee can work multiple shifts, and each shift can have multiple employees. The same employee might work the same shift on different dates.

**Required fields:**
- Employee identifier
- Shift type (morning, afternoon, night)
- Date worked
- Hours worked

Design the table with an appropriate primary key strategy. Explain your choice.

</details>

---

### Answers to Section 2.1 Practice Questions

<details>
<summary><strong>Answer 1: Fill in the Blanks</strong></summary>

a) A primary key must be both **UNIQUE** and **NOT NULL**.

b) A **NATURAL** key has business meaning, while a **SURROGATE** key is artificially generated.

c) In MySQL, **AUTO_INCREMENT** automatically generates sequential numbers, while PostgreSQL uses **SERIAL** (or GENERATED ALWAYS AS IDENTITY).

d) When using composite primary keys, the combination of columns must be **UNIQUE**, but individual columns can contain **DUPLICATE** values.

e) After a transaction rollback, AUTO_INCREMENT values are **NOT REUSED**, creating **GAPS** in the sequence.

</details>

<details>
<summary><strong>Answer 2: True or False</strong></summary>

a) **FALSE** - A table can have only ONE primary key (though it can be composite).

b) **TRUE** - Databases automatically create an index on the primary key for performance.

c) **FALSE** - AUTO_INCREMENT does NOT reuse deleted values; gaps are permanent.

d) **FALSE** - Surrogate keys are generally preferred because natural keys can change.

e) **FALSE** - Composite primary keys can have any number of columns (though 2-3 is most common).

f) **TRUE** - Both serve the same purpose of auto-generating unique sequential values.

g) **TRUE** - Primary keys should be immutable; changing them affects all foreign key relationships.

</details>

<details>
<summary><strong>Answer 3: Multiple Choice</strong></summary>

**a) B** - `user_id BIGINT AUTO_INCREMENT`
- Emails can change, usernames can change, SSNs have privacy concerns
- Surrogate integer keys are stable and performant

**b) C** - The database automatically generates the next sequential value
- This is the whole purpose of AUTO_INCREMENT/SERIAL

**c) D** - `(student_id, course_id, semester)`
- Students can retake courses, so need semester to differentiate
- Example: Student 123 takes CS101 in Fall 2024 and Spring 2025

**d) B** - Many-to-many junction table
- Junction tables naturally benefit from composite keys
- Example: (user_id, role_id) in user_roles table

**e) C** - 9,223,372,036,854,775,807
- BIGINT is 8 bytes = 2^63 - 1 for signed values
- Unsigned BIGINT would be answer D

</details>

<details>
<summary><strong>Answer 4: Code Analysis</strong></summary>

**Issues with using product_name as primary key:**

1. **Not Immutable**: Product names can change (typo fixes, rebranding, translations)
   - Changing a primary key requires updating all foreign key references

2. **Not Guaranteed Unique**: Two different products could have the same name
   - "Wireless Mouse" could refer to multiple products across brands

3. **Performance**: VARCHAR primary keys are slower than integer keys
   - More storage space (100 bytes vs 8 bytes)
   - Slower comparisons and joins

4. **Size**: Larger keys mean larger indexes and foreign keys
   - Every table referencing products would store 100-byte strings

**Better design:**
```sql
CREATE TABLE products (
    product_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    sku VARCHAR(50) UNIQUE NOT NULL,  -- SKU is the natural identifier
    price DECIMAL(10, 2),
    category VARCHAR(50)
);
```

</details>

<details>
<summary><strong>Answer 5: Practical Scenario</strong></summary>

**Option 1: Composite Primary Key**
```sql
CREATE TABLE shift_assignments (
    employee_id BIGINT NOT NULL,
    shift_type VARCHAR(20) NOT NULL,
    work_date DATE NOT NULL,
    hours_worked DECIMAL(4, 2) NOT NULL,
    PRIMARY KEY (employee_id, shift_type, work_date),
    FOREIGN KEY (employee_id) REFERENCES employees(employee_id)
);
```

**Reasoning:**
- `(employee_id, shift_type, work_date)` uniquely identifies each shift worked
- Employee 101 can work morning shift on 2024-01-15
- Same employee can work night shift on 2024-01-15 (different shift_type)
- Same employee can work morning shift on 2024-01-16 (different date)

**Option 2: Surrogate Key (Often Better)**
```sql
CREATE TABLE shift_assignments (
    assignment_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    employee_id BIGINT NOT NULL,
    shift_type VARCHAR(20) NOT NULL,
    work_date DATE NOT NULL,
    hours_worked DECIMAL(4, 2) NOT NULL,
    UNIQUE KEY unique_assignment (employee_id, shift_type, work_date),
    FOREIGN KEY (employee_id) REFERENCES employees(employee_id)
);
```

**Why Option 2 is preferred:**
- Easier to reference in other tables (payroll, attendance)
- Simpler foreign keys (single column vs three)
- Can add additional metadata (created_at, modified_at) easily
- Still enforces uniqueness via UNIQUE constraint

**Real-world parallel:** Shift scheduling systems like When I Work or Deputy use surrogate keys but enforce business rules via unique constraints.

</details>

---

## 2.2 Foreign Keys

### 2.2.1 Definition and Purpose

A **foreign key** is a column (or set of columns) in one table that references the primary key of another table. It creates a link between two tables, establishing a parent-child relationship.

**Real-World Analogy:**
Think of a library system. Each book (child) must belong to a specific branch (parent). The book's record contains a reference to which branch owns it. You can't add a book assigned to a non-existent branch.

**Basic Example: E-commerce System**
```sql
-- Parent table: Customers
CREATE TABLE customers (
    customer_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Child table: Orders (references customers)
CREATE TABLE orders (
    order_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10, 2),
    status VARCHAR(20),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

**What This Enforces:**
```sql
-- Valid: Customer 1 exists
INSERT INTO customers (email, first_name) VALUES ('alice@example.com', 'Alice');
INSERT INTO orders (customer_id, total_amount) VALUES (1, 99.99);  -- ✓ Works

-- Invalid: Customer 999 doesn't exist
INSERT INTO orders (customer_id, total_amount) VALUES (999, 49.99);
-- ERROR: Cannot add or update a child row: a foreign key constraint fails
```

**Purpose of Foreign Keys:**

1. **Data Integrity**: Prevents orphaned records
```sql
-- Without foreign key: Orphaned order (customer doesn't exist)
-- With foreign key: Database prevents this automatically
```

2. **Enforces Relationships**: Documents the database structure
```sql
-- Looking at the schema tells you: "Orders belong to customers"
SHOW CREATE TABLE orders;
```

3. **Cascading Actions**: Automatically handle related data (more on this in 2.2.3)

**Real-World Example: Spotify Database**
```sql
-- Artists table
CREATE TABLE artists (
    artist_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    artist_name VARCHAR(255) NOT NULL,
    biography TEXT,
    monthly_listeners BIGINT DEFAULT 0
);

-- Albums table
CREATE TABLE albums (
    album_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    artist_id BIGINT NOT NULL,
    album_title VARCHAR(255) NOT NULL,
    release_date DATE,
    genre VARCHAR(50),
    FOREIGN KEY (artist_id) REFERENCES artists(artist_id)
);

-- Tracks table
CREATE TABLE tracks (
    track_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    album_id BIGINT NOT NULL,
    track_title VARCHAR(255) NOT NULL,
    duration_seconds INT,
    track_number INT,
    FOREIGN KEY (album_id) REFERENCES albums(album_id)
);
```

**Relationship Chain:**
```
Artists (1) ──→ Albums (Many)
Albums (1) ──→ Tracks (Many)

Artist: "The Beatles"
  ├── Album: "Abbey Road"
  │   ├── Track: "Come Together"
  │   ├── Track: "Something"
  │   └── Track: "Here Comes the Sun"
  └── Album: "Let It Be"
      ├── Track: "Let It Be"
      └── Track: "Get Back"
```

### 2.2.2 Referential Integrity

**Referential integrity** means that relationships between tables remain consistent. If a foreign key says a record exists, that record MUST exist.

**The Four Rules of Referential Integrity:**

**1. Insert Rule: Can't add a child without a parent**
```sql
-- Airbnb-style booking system
CREATE TABLE properties (
    property_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    host_id BIGINT NOT NULL,
    title VARCHAR(255),
    city VARCHAR(100)
);

CREATE TABLE bookings (
    booking_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    property_id BIGINT NOT NULL,
    guest_id BIGINT NOT NULL,
    check_in DATE,
    check_out DATE,
    FOREIGN KEY (property_id) REFERENCES properties(property_id)
);

-- This fails: property_id 999 doesn't exist
INSERT INTO bookings (property_id, guest_id, check_in, check_out)
VALUES (999, 501, '2024-06-01', '2024-06-07');
-- ERROR: Cannot add child row

-- Must create property first
INSERT INTO properties (host_id, title, city) 
VALUES (101, 'Cozy Downtown Apartment', 'Seattle');
-- property_id 1 created

-- Now booking works
INSERT INTO bookings (property_id, guest_id, check_in, check_out)
VALUES (1, 501, '2024-06-01', '2024-06-07');  -- ✓ Success
```

**2. Update Rule: Can't update foreign key to point to non-existent parent**
```sql
-- Existing booking points to property 1
UPDATE bookings SET property_id = 999 WHERE booking_id = 1;
-- ERROR: Cannot update child row
```

**3. Delete Rule: Can't delete parent if children exist (by default)**
```sql
-- Property 1 has bookings
DELETE FROM properties WHERE property_id = 1;
-- ERROR: Cannot delete parent row: foreign key constraint fails

-- Must delete children first
DELETE FROM bookings WHERE property_id = 1;
DELETE FROM properties WHERE property_id = 1;  -- ✓ Now works
```

**4. Update Parent Key Rule: Can't change parent's primary key if children reference it**
```sql
-- This is why primary keys should be immutable
UPDATE properties SET property_id = 2000 WHERE property_id = 1;
-- ERROR or CASCADE (depending on setup)
```

**Real-World Example: Banking System**
```sql
-- Accounts table
CREATE TABLE accounts (
    account_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    account_number VARCHAR(20) UNIQUE NOT NULL,
    account_type VARCHAR(20),
    balance DECIMAL(15, 2) DEFAULT 0.00,
    status VARCHAR(20) DEFAULT 'ACTIVE'
);

-- Transactions table
CREATE TABLE transactions (
    transaction_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    account_id BIGINT NOT NULL,
    transaction_type VARCHAR(20) NOT NULL,
    amount DECIMAL(15, 2) NOT NULL,
    balance_after DECIMAL(15, 2),
    transaction_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    description VARCHAR(255),
    FOREIGN KEY (account_id) REFERENCES accounts(account_id)
);
```

**What Referential Integrity Prevents:**
```sql
-- Scenario 1: Orphaned transactions
-- WITHOUT foreign key: You could delete an account but transactions remain
-- Result: Transactions point to non-existent account (data corruption)

-- WITH foreign key: Database prevents deletion
DELETE FROM accounts WHERE account_id = 12345;
-- ERROR: Cannot delete - transactions exist

-- Scenario 2: Invalid account references
-- WITHOUT foreign key: Transaction could reference account 99999 (doesn't exist)
INSERT INTO transactions (account_id, transaction_type, amount)
VALUES (99999, 'DEPOSIT', 1000.00);  -- Would succeed without FK

-- WITH foreign key: Prevented
-- ERROR: Cannot insert - account 99999 doesn't exist
```

**Checking Referential Integrity:**
```sql
-- Find orphaned records (if foreign keys weren't enforced)
SELECT t.* 
FROM transactions t
LEFT JOIN accounts a ON t.account_id = a.account_id
WHERE a.account_id IS NULL;
-- Should return 0 rows if integrity is maintained
```

**Benefits in Production:**
- **Data accuracy**: No broken relationships
- **Application simplicity**: Don't need to validate relationships in code
- **Documentation**: Schema shows relationships clearly
- **Query safety**: JOIN operations always work correctly

**Trade-offs:**
- **Performance overhead**: Database checks every insert/update/delete
- **Lock contention**: Parent table locks when children modified
- **Bulk operations**: Can be slower due to validation

Some high-scale systems (Twitter, Instagram) occasionally disable foreign keys for performance, but enforce integrity in application code instead. This is an advanced optimization with risks.

### 2.2.3 CASCADE, SET NULL, RESTRICT

When you delete or update a parent record, foreign keys let you specify what happens to child records through **referential actions**.

**The Four Main Actions:**

1. **CASCADE**: Automatically propagate changes to children
2. **SET NULL**: Set foreign key to NULL in children
3. **RESTRICT**: Prevent the operation (default in most databases)
4. **NO ACTION**: Similar to RESTRICT (slight differences in timing)

**RESTRICT vs NO ACTION:** 
- **RESTRICT**: Checks immediately
- **NO ACTION**: Checks at end of statement (allows temporary violations)
- In practice, they're nearly identical; use RESTRICT

---

**1. ON DELETE CASCADE**

When parent deleted, automatically delete all children.

**Real-World Example: Social Media Posts**
```sql
-- Users table
CREATE TABLE users (
    user_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);

-- Posts table
CREATE TABLE posts (
    post_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) 
        REFERENCES users(user_id)
        ON DELETE CASCADE  -- Delete user → delete all their posts
);

-- Comments on posts
CREATE TABLE comments (
    comment_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    post_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (post_id) 
        REFERENCES posts(post_id)
        ON DELETE CASCADE,  -- Delete post → delete all comments
    FOREIGN KEY (user_id)
        REFERENCES users(user_id)
        ON DELETE CASCADE   -- Delete user → delete all their comments
);
```

**What Happens:**
```sql
-- Create user
INSERT INTO users (username, email) VALUES ('alice', 'alice@example.com');
-- user_id = 1

-- Alice creates posts
INSERT INTO posts (user_id, content) VALUES (1, 'Hello world!');
INSERT INTO posts (user_id, content) VALUES (1, 'My second post');
-- post_ids = 1, 2

-- Someone comments on Alice's first post
INSERT INTO comments (post_id, user_id, content) 
VALUES (1, 2, 'Nice post!');

-- Delete Alice's account
DELETE FROM users WHERE user_id = 1;

-- RESULT: Cascade happens automatically
-- 1. All of Alice's posts (IDs 1, 2) are deleted
-- 2. All comments on Alice's posts are deleted
-- 3. All of Alice's comments on other posts are deleted
```

**Cascade Chain:**
```
DELETE users (user_id = 1)
  ├── CASCADE → DELETE posts (user_id = 1) [2 rows]
  │     └── CASCADE → DELETE comments (post_id IN (1,2)) [1 row]
  └── CASCADE → DELETE comments (user_id = 1) [0 rows]
```

**When to Use CASCADE:**
- Truly dependent data (posts belong to user, can't exist without owner)
- Cleanup is desired (deleting project should delete all tasks)
- Parent-child lifetime is identical

---

**2. ON DELETE SET NULL**

When parent deleted, set foreign key to NULL in children (orphan them).

**Real-World Example: Employee Manager Relationship**
```sql
-- Employees table (self-referencing)
CREATE TABLE employees (
    employee_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    manager_id BIGINT NULL,  -- Must allow NULL
    department VARCHAR(50),
    FOREIGN KEY (manager_id) 
        REFERENCES employees(employee_id)
        ON DELETE SET NULL  -- Manager leaves → employees remain, manager_id becomes NULL
);
```

**Scenario:**
```sql
-- Create organizational structure
INSERT INTO employees (first_name, last_name, manager_id, department)
VALUES 
    ('Alice', 'Johnson', NULL, 'Engineering'),      -- employee_id = 1 (CEO)
    ('Bob', 'Smith', 1, 'Engineering'),             -- employee_id = 2 (reports to Alice)
    ('Charlie', 'Brown', 2, 'Engineering'),         -- employee_id = 3 (reports to Bob)
    ('David', 'Wilson', 2, 'Engineering');          -- employee_id = 4 (reports to Bob)

-- Current hierarchy:
-- Alice (CEO)
--   └── Bob (Manager)
--         ├── Charlie
--         └── David

-- Bob leaves the company
DELETE FROM employees WHERE employee_id = 2;

-- RESULT:
-- Charlie and David remain employed
-- Their manager_id is now NULL (need new manager assigned)

SELECT employee_id, first_name, manager_id FROM employees;
-- 1 | Alice   | NULL
-- 3 | Charlie | NULL  ← Was 2, now NULL
-- 4 | David   | NULL  ← Was 2, now NULL
```

**E-commerce Example: Optional Coupon Usage**
```sql
CREATE TABLE coupons (
    coupon_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    coupon_code VARCHAR(20) UNIQUE,
    discount_percent DECIMAL(5, 2),
    expires_at TIMESTAMP
);

CREATE TABLE orders (
    order_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    coupon_id BIGINT NULL,  -- Optional coupon
    total_amount DECIMAL(10, 2),
    FOREIGN KEY (coupon_id) 
        REFERENCES coupons(coupon_id)
        ON DELETE SET NULL  -- Coupon deleted → order remains, just no coupon reference
);
```

**Scenario:**
```sql
-- Create limited-time coupon
INSERT INTO coupons (coupon_code, discount_percent, expires_at)
VALUES ('SUMMER20', 20, '2024-08-31');
-- coupon_id = 1

-- Customer uses coupon
INSERT INTO orders (customer_id, coupon_id, total_amount)
VALUES (101, 1, 80.00);  -- Original $100, 20% off = $80

-- Coupon expires and is deleted from system
DELETE FROM coupons WHERE coupon_id = 1;

-- Order remains, but coupon_id is NULL
SELECT * FROM orders WHERE order_id = 1;
-- order_id: 1, customer_id: 101, coupon_id: NULL, total_amount: 80.00
-- The order keeps its discounted price, but coupon reference is gone
```

**When to Use SET NULL:**
- Optional relationships (order doesn't require a coupon)
- Soft dependencies (employee can exist without manager temporarily)
- Historical data preservation (keep order even if product discontinued)

---

**3. ON DELETE RESTRICT (Default)**

Prevent parent deletion if children exist. You must manually clean up first.

**Real-World Example: Banking Accounts**
```sql
CREATE TABLE accounts (
    account_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    account_number VARCHAR(20) UNIQUE,
    balance DECIMAL(15, 2)
);

CREATE TABLE transactions (
    transaction_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    account_id BIGINT NOT NULL,
    amount DECIMAL(15, 2),
    transaction_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (account_id) 
        REFERENCES accounts(account_id)
        ON DELETE RESTRICT  -- Can't delete account with transaction history
);
```

**Why This Makes Sense:**
```sql
-- Account has $10,000 balance and transaction history
INSERT INTO accounts (customer_id, account_number, balance)
VALUES (501, '1234567890', 10000.00);

INSERT INTO transactions (account_id, amount)
VALUES (1, 500.00), (1, -200.00), (1, 1000.00);

-- Try to delete account
DELETE FROM accounts WHERE account_id = 1;
-- ERROR: Cannot delete: foreign key constraint fails

-- Financial regulations require keeping transaction history
-- You CANNOT delete accounts with transactions
-- Instead, mark as closed:
ALTER TABLE accounts ADD COLUMN status VARCHAR(20) DEFAULT 'ACTIVE';
UPDATE accounts SET status = 'CLOSED' WHERE account_id = 1;
```

**Product Catalog Example:**
```sql
CREATE TABLE products (
    product_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(255),
    sku VARCHAR(50) UNIQUE,
    price DECIMAL(10, 2)
);

CREATE TABLE order_items (
    order_item_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INT,
    unit_price DECIMAL(10, 2),
    FOREIGN KEY (product_id)
        REFERENCES products(product_id)
        ON DELETE RESTRICT  -- Can't delete product if ever ordered
);
```

**Business Logic:**
```sql
-- Product has been ordered
INSERT INTO products (product_name, sku, price)
VALUES ('Laptop', 'LAP-001', 999.99);

INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (1, 1, 2, 999.99);

-- Try to delete product
DELETE FROM products WHERE product_id = 1;
-- ERROR: Cannot delete

-- Solution: Soft delete (mark as discontinued)
ALTER TABLE products ADD COLUMN is_active BOOLEAN DEFAULT TRUE;
UPDATE products SET is_active = FALSE WHERE product_id = 1;
-- Product appears "deleted" to customers but history preserved
```

**When to Use RESTRICT:**
- Financial data (transactions, payments, invoices)
- Audit trails (activity logs, access logs)
- Legal compliance (need to preserve history)
- Any data that must never be deleted

---

**4. ON UPDATE CASCADE**

When parent's primary key updated, automatically update all child foreign keys.

**Note:** This is rare because primary keys should never change! But for completeness:

```sql
CREATE TABLE departments (
    dept_id VARCHAR(10) PRIMARY KEY,  -- Using code as PK (not ideal)
    dept_name VARCHAR(100)
);

CREATE TABLE employees (
    employee_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    dept_id VARCHAR(10),
    first_name VARCHAR(50),
    FOREIGN KEY (dept_id)
        REFERENCES departments(dept_id)
        ON UPDATE CASCADE  -- Dept code changes → update all employees
);
```

**Example:**
```sql
INSERT INTO departments VALUES ('ENG', 'Engineering');
INSERT INTO employees (dept_id, first_name) VALUES ('ENG', 'Alice'), ('ENG', 'Bob');

-- Company restructures: Engineering becomes Technology
UPDATE departments SET dept_id = 'TECH' WHERE dept_id = 'ENG';

-- CASCADE automatically updates employees
SELECT * FROM employees;
-- Both employees now have dept_id = 'TECH'
```

**Best Practice:** Don't do this. Use surrogate integer keys that never change:
```sql
CREATE TABLE departments (
    dept_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Never changes
    dept_code VARCHAR(10),  -- Can change freely
    dept_name VARCHAR(100)
);
```

---

**Combining Actions:**
```sql
-- GitHub-style repository system
CREATE TABLE repositories (
    repo_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    owner_id BIGINT NOT NULL,
    repo_name VARCHAR(100)
);

CREATE TABLE issues (
    issue_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    repo_id BIGINT NOT NULL,
    author_id BIGINT NULL,  -- User who created the issue
    title VARCHAR(255),
    status VARCHAR(20),
    FOREIGN KEY (repo_id) 
        REFERENCES repositories(repo_id)
        ON DELETE CASCADE,      -- Delete repo → delete all issues
    FOREIGN KEY (author_id)
        REFERENCES users(user_id)
        ON DELETE SET NULL      -- Delete user → preserve issues, author becomes NULL
);
```

**Decision Matrix:**

| Scenario | Action | Reason |
|----------|--------|---------|
| Blog posts when user deleted | CASCADE | Posts belong to user, meaningless without author |
| Order items when product deleted | RESTRICT | Need historical record of what was ordered |
| Employee manager when manager quits | SET NULL | Employee remains, needs new manager later |
| Comments when post deleted | CASCADE | Comments belong to post, meaningless without it |
| Payment method when customer deleted | CASCADE | Payment info belongs to customer |
| Transactions when account closed | RESTRICT | Financial audit trail required |

### 2.2.4 Self-Referencing Foreign Keys

A **self-referencing foreign key** is when a table has a foreign key that points to its own primary key. This creates a hierarchical or recursive relationship within the same table.

**Real-World Example: Employee Organizational Chart**
```sql
CREATE TABLE employees (
    employee_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(255) UNIQUE,
    job_title VARCHAR(100),
    manager_id BIGINT NULL,  -- References same table!
    hire_date DATE,
    salary DECIMAL(10, 2),
    FOREIGN KEY (manager_id) 
        REFERENCES employees(employee_id)
        ON DELETE SET NULL  -- Manager leaves → employee remains, needs reassignment
);
```

**Building an Organization:**
```sql
-- CEO (no manager)
INSERT INTO employees (first_name, last_name, job_title, manager_id, salary)
VALUES ('Sarah', 'CEO', 'Chief Executive Officer', NULL, 250000);
-- employee_id = 1

-- Direct reports to CEO
INSERT INTO employees (first_name, last_name, job_title, manager_id, salary)
VALUES 
    ('John', 'Smith', 'CTO', 1, 180000),           -- employee_id = 2
    ('Emma', 'Davis', 'CFO', 1, 170000),           -- employee_id = 3
    ('Michael', 'Brown', 'VP Sales', 1, 160000);   -- employee_id = 4

-- Engineering team under CTO
INSERT INTO employees (first_name, last_name, job_title, manager_id, salary)
VALUES 
    ('Alice', 'Johnson', 'Engineering Manager', 2, 140000),  -- employee_id = 5
    ('Bob', 'Wilson', 'Senior Engineer', 5, 120000),         -- employee_id = 6
    ('Charlie', 'Lee', 'Engineer', 5, 95000);                -- employee_id = 7

-- Finance team under CFO
INSERT INTO employees (first_name, last_name, job_title, manager_id, salary)
VALUES 
    ('Diana', 'Martinez', 'Finance Manager', 3, 130000),     -- employee_id = 8
    ('Ethan', 'Taylor', 'Accountant', 8, 80000);             -- employee_id = 9
```

**Resulting Hierarchy:**
```
Sarah (CEO, employee_id=1)
├── John (CTO, employee_id=2, manager_id=1)
│   └── Alice (Eng Manager, employee_id=5, manager_id=2)
│       ├── Bob (Sr Engineer, employee_id=6, manager_id=5)
│       └── Charlie (Engineer, employee_id=7, manager_id=5)
├── Emma (CFO, employee_id=3, manager_id=1)
│   └── Diana (Finance Mgr, employee_id=8, manager_id=3)
│       └── Ethan (Accountant, employee_id=9, manager_id=8)
└── Michael (VP Sales, employee_id=4, manager_id=1)
```

**Querying the Hierarchy:**

**Find direct reports:**
```sql
-- Who reports directly to the CTO (employee_id = 2)?
SELECT employee_id, first_name, last_name, job_title
FROM employees
WHERE manager_id = 2;

-- Result: Alice (Engineering Manager)
```

**Find an employee's manager:**
```sql
-- Who is Bob's manager?
SELECT m.employee_id, m.first_name, m.last_name, m.job_title
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id
WHERE e.employee_id = 6;

-- Result: Alice Johnson (Engineering Manager)
```

**Find entire management chain (recursive query):**
```sql
-- PostgreSQL/MySQL 8.0+ (WITH RECURSIVE)
WITH RECURSIVE management_chain AS (
    -- Base case: start with Charlie (employee_id = 7)
    SELECT employee_id, first_name, last_name, manager_id, 0 AS level
    FROM employees
    WHERE employee_id = 7
    
    UNION ALL
    
    -- Recursive case: get manager of current employee
    SELECT e.employee_id, e.first_name, e.last_name, e.manager_id, mc.level + 1
    FROM employees e
    JOIN management_chain mc ON e.employee_id = mc.manager_id
)
SELECT level, first_name, last_name FROM management_chain ORDER BY level;

-- Result:
-- Level 0: Charlie (Engineer)
-- Level 1: Alice (Engineering Manager)
-- Level 2: John (CTO)
-- Level 3: Sarah (CEO)
```

**Find all subordinates (everyone below a manager):**
```sql
-- Everyone who reports to CTO (direct or indirect)
WITH RECURSIVE subordinates AS (
    -- Base case: CTO
    SELECT employee_id, first_name, last_name, manager_id, 0 AS level
    FROM employees
    WHERE employee_id = 2
    
    UNION ALL
    
    -- Recursive: everyone who reports to someone in the result
    SELECT e.employee_id, e.first_name, e.last_name, e.manager_id, s.level + 1
    FROM employees e
    JOIN subordinates s ON e.manager_id = s.employee_id
)
SELECT level, first_name, last_name FROM subordinates ORDER BY level;

-- Result:
-- Level 0: John (CTO himself)
-- Level 1: Alice (direct report)
-- Level 2: Bob, Charlie (indirect reports)
```

---

**Real-World Example 2: Comment Threading (Reddit/HackerNews)**
```sql
CREATE TABLE comments (
    comment_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    post_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    parent_comment_id BIGINT NULL,  -- Self-reference for replies
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    upvotes INT DEFAULT 0,
    FOREIGN KEY (parent_comment_id) 
        REFERENCES comments(comment_id)
        ON DELETE CASCADE,  -- Delete parent comment → delete all replies
    FOREIGN KEY (post_id) REFERENCES posts(post_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

**Creating a Comment Thread:**
```sql
-- Top-level comment
INSERT INTO comments (post_id, user_id, parent_comment_id, content)
VALUES (1, 101, NULL, 'Great article!');
-- comment_id = 1

-- Reply to comment 1
INSERT INTO comments (post_id, user_id, parent_comment_id, content)
VALUES (1, 102, 1, 'I agree, very insightful.');
-- comment_id = 2

-- Another reply to comment 1
INSERT INTO comments (post_id, user_id, parent_comment_id, content)
VALUES (1, 103, 1, 'I have a different perspective...');
-- comment_id = 3

-- Reply to comment 2 (nested reply)
INSERT INTO comments (post_id, user_id, parent_comment_id, content)
VALUES (1, 104, 2, 'Could you elaborate?');
-- comment_id = 4
```

**Comment Tree:**
```
Comment 1 (parent_comment_id = NULL)
├── Comment 2 (parent_comment_id = 1)
│   └── Comment 4 (parent_comment_id = 2)
└── Comment 3 (parent_comment_id = 1)
```

**Get All Replies to a Comment:**
```sql
-- All replies to comment 1 (flat)
SELECT comment_id, content, user_id
FROM comments
WHERE parent_comment_id = 1;

-- Results: comments 2 and 3
```

**Get Entire Thread (Recursive):**
```sql
WITH RECURSIVE comment_thread AS (
    -- Start with top-level comment
    SELECT comment_id, parent_comment_id, content, 0 AS depth
    FROM comments
    WHERE comment_id = 1
    
    UNION ALL
    
    -- Get all replies recursively
    SELECT c.comment_id, c.parent_comment_id, c.content, ct.depth + 1
    FROM comments c
    JOIN comment_thread ct ON c.parent_comment_id = ct.comment_id
)
SELECT CONCAT(REPEAT('  ', depth), content) AS threaded_comment
FROM comment_thread
ORDER BY comment_id;

-- Result:
-- Great article!
--   I agree, very insightful.
--     Could you elaborate?
--   I have a different perspective...
```

---

**Real-World Example 3: File System / Folder Structure**
```sql
CREATE TABLE folders (
    folder_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    folder_name VARCHAR(255) NOT NULL,
    parent_folder_id BIGINT NULL,  -- Self-reference
    created_by BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (parent_folder_id) 
        REFERENCES folders(folder_id)
        ON DELETE CASCADE  -- Delete folder → delete all subfolders
);
```

**Google Drive-like Structure:**
```sql
-- Root folders (no parent)
INSERT INTO folders (folder_name, parent_folder_id, created_by)
VALUES ('My Drive', NULL, 1);  -- folder_id = 1

INSERT INTO folders (folder_name, parent_folder_id, created_by)
VALUES 
    ('Work', 1, 1),      -- folder_id = 2
    ('Personal', 1, 1);  -- folder_id = 3

-- Work subfolders
INSERT INTO folders (folder_name, parent_folder_id, created_by)
VALUES 
    ('Projects', 2, 1),   -- folder_id = 4
    ('Reports', 2, 1);    -- folder_id = 5

-- Projects subfolders
INSERT INTO folders (folder_name, parent_folder_id, created_by)
VALUES 
    ('Q1-2024', 4, 1),    -- folder_id = 6
    ('Q2-2024', 4, 1);    -- folder_id = 7
```

**Structure:**
```
My Drive (1)
├── Work (2)
│   ├── Projects (4)
│   │   ├── Q1-2024 (6)
│   │   └── Q2-2024 (7)
│   └── Reports (5)
└── Personal (3)
```

**Get Full Path to a Folder:**
```sql
WITH RECURSIVE folder_path AS (
    -- Start with target folder (Q1-2024)
    SELECT folder_id, folder_name, parent_folder_id, 0 AS level
    FROM folders
    WHERE folder_id = 6
    
    UNION ALL
    
    -- Traverse up to root
    SELECT f.folder_id, f.folder_name, f.parent_folder_id, fp.level + 1
    FROM folders f
    JOIN folder_path fp ON f.folder_id = fp.parent_folder_id
)
SELECT GROUP_CONCAT(folder_name ORDER BY level DESC SEPARATOR '/') AS full_path
FROM folder_path;

-- Result: My Drive/Work/Projects/Q1-2024
```

---

**Common Pitfalls with Self-Referencing Keys:**

**1. Circular References (Impossible with proper FK):**
```sql
-- Can't happen due to foreign key constraint
UPDATE employees SET manager_id = 7 WHERE employee_id = 5;
-- ERROR if employee 7 has manager_id = 5 (circular)
```

**2. Deleting Without Considering Children:**
```sql
-- With ON DELETE SET NULL
DELETE FROM employees WHERE employee_id = 5;
-- Alice deleted → Bob and Charlie now have manager_id = NULL (need reassignment)

-- With ON DELETE RESTRICT (better for org charts)
-- Would prevent deletion if anyone reports to Alice
```

**3. Infinite Recursion in Queries:**
```sql
-- Accidentally creating infinite loop (bad data)
UPDATE employees SET manager_id = employee_id WHERE employee_id = 1;
-- Should be prevented by application logic or CHECK constraint

-- Protection: add depth limit
WITH RECURSIVE subordinates AS (
    ...
)
SELECT * FROM subordinates WHERE level < 10;  -- Prevent infinite recursion
```

**Best Practices:**
- Use `NULL` for top-level/root entries (CEO, root folder, top comment)
- Always use `ON DELETE SET NULL` or `ON DELETE RESTRICT` (never CASCADE for managers)
- Add CHECK constraint to prevent self-reference: `CHECK (employee_id != manager_id)`
- Limit recursion depth in queries
- Consider materialized path or nested sets for very deep hierarchies (performance)

---

### Section 2.2 Practice Questions

<details>
<summary><strong>Question 1: Fill in the Blanks</strong></summary>

a) A foreign key creates a _______ relationship between two tables, where one table is the _______ and the other is the _______.

b) The four referential actions are _______, _______, _______, and _______.

c) Using `ON DELETE CASCADE` means when a parent is deleted, all _______ are automatically _______.

d) In a self-referencing foreign key, the foreign key column references the _______ of the same table.

e) `ON DELETE RESTRICT` is appropriate for _______ data where you need to preserve _______ trails.

</details>

<details>
<summary><strong>Question 2: True or False</strong></summary>

a) You can insert a child record that references a non-existent parent if the foreign key is optional (NULL allowed).

b) CASCADE should always be used instead of RESTRICT for maximum flexibility.

c) A self-referencing foreign key allows a table to have hierarchical relationships within itself.

d) Foreign keys automatically create indexes on both the child and parent tables.

e) ON DELETE SET NULL requires the foreign key column to allow NULL values.

f) You can have multiple foreign keys in a single table referencing different tables.

g) With ON DELETE RESTRICT, you must manually delete all child records before deleting the parent.

</details>

<details>
<summary><strong>Question 3: Multiple Choice</strong></summary>

**a) Which referential action is best for a blog where posts should be deleted when a user account is deleted?**
- A) ON DELETE RESTRICT
- B) ON DELETE SET NULL
- C) ON DELETE CASCADE
- D) ON DELETE NO ACTION

**b) For a banking transactions table, which referential action is most appropriate?**
- A) ON DELETE CASCADE
- B) ON DELETE SET NULL
- C) ON DELETE RESTRICT
- D) Either CASCADE or SET NULL

**c) In an employee-manager relationship, when a manager quits, what should happen to their direct reports?**
- A) CASCADE (delete employees)
- B) SET NULL (manager becomes NULL)
- C) RESTRICT (prevent manager from quitting)
- D) NO ACTION

**d) What happens if you try to insert a child record with a foreign key value that doesn't exist in the parent table?**
- A) The record is inserted with NULL
- B) The database creates the missing parent record
- C) The database throws a foreign key constraint error
- D) The record is inserted but marked as invalid

**e) In a comment threading system with parent_comment_id self-reference, what's the best ON DELETE action?**
- A) RESTRICT (preserve all comments forever)
- B) CASCADE (deleting a comment deletes all replies)
- C) SET NULL (make all replies orphans)
- D) NO ACTION

</details>

<details>
<summary><strong>Question 4: Schema Design Challenge</strong></summary>

Design a database schema for a project management system with the following requirements:

1. **Projects** - can have multiple tasks
2. **Tasks** - belong to one project, can have subtasks
3. **Users** - can be assigned to multiple tasks
4. **Comments** - can be on tasks, can be replies to other comments

For each foreign key, specify:
- Which referential action to use (CASCADE, SET NULL, or RESTRICT)
- Why you chose that action

</details>

<details>
<summary><strong>Question 5: Query Challenge</strong></summary>

Given this employee table with self-referencing manager_id:

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(50),
    manager_id INT,
    FOREIGN KEY (manager_id) REFERENCES employees(employee_id)
);

-- Data:
-- 1, 'Alice', NULL     (CEO)
-- 2, 'Bob', 1          (Reports to Alice)
-- 3, 'Charlie', 1      (Reports to Alice)
-- 4, 'David', 2        (Reports to Bob)
-- 5, 'Eve', 2          (Reports to Bob)
-- 6, 'Frank', 4        (Reports to David)
```

Write queries to:
a) Find all direct reports of Bob (employee_id = 2)
b) Find Bob's manager
c) Find all employees at the same level as David (same distance from CEO)

</details>

---

### Answers to Section 2.2 Practice Questions

<details>
<summary><strong>Answer 1: Fill in the Blanks</strong></summary>

a) A foreign key creates a **PARENT-CHILD** (or **ONE-TO-MANY**) relationship between two tables, where one table is the **PARENT** and the other is the **CHILD**.

b) The four referential actions are **CASCADE**, **SET NULL**, **RESTRICT**, and **NO ACTION**.

c) Using `ON DELETE CASCADE` means when a parent is deleted, all **CHILD RECORDS** are automatically **DELETED**.

d) In a self-referencing foreign key, the foreign key column references the **PRIMARY KEY** of the same table.

e) `ON DELETE RESTRICT` is appropriate for **FINANCIAL/AUDIT** data where you need to preserve **HISTORICAL** trails.

</details>

<details>
<summary><strong>Answer 2: True or False</strong></summary>

a) **FALSE** - You can insert a child with NULL foreign key, but if you specify a value, it must exist in the parent table.

b) **FALSE** - CASCADE is dangerous for data you need to preserve. Use RESTRICT for financial data, audit logs, etc.

c) **TRUE** - Self-referencing foreign keys create hierarchies like org charts, folder structures, and comment threads.

d) **FALSE** - Foreign keys create an index on the CHILD table's foreign key column, but not on the parent (it already has an index via PRIMARY KEY).

e) **TRUE** - ON DELETE SET NULL can't work if the foreign key column is NOT NULL.

f) **TRUE** - A table can have multiple foreign keys referencing different parent tables (e.g., order_items references both orders and products).

g) **TRUE** - RESTRICT prevents parent deletion if children exist. You must delete children first or use SET NULL/CASCADE.

</details>

<details>
<summary><strong>Answer 3: Multiple Choice</strong></summary>

**a) C - ON DELETE CASCADE**
- User's posts have no meaning without the user
- When user account deleted, their content should go too
- Common pattern for social media platforms

**b) C - ON DELETE RESTRICT**
- Financial regulations require preserving transaction history
- Even if an account is closed, transactions must remain
- Prevents accidental data loss of audit trails

**c) B - SET NULL (manager becomes NULL)**
- Employees still work for the company
- They need to be reassigned to a new manager
- Deleting employees when manager quits makes no sense

**d) C - The database throws a foreign key constraint error**
- Referential integrity prevents insertion of orphaned records
- Parent must exist before child can reference it

**e) B - CASCADE (deleting a comment deletes all replies)**
- Replies have no context without parent comment
- Similar to how Reddit/HN handle deleted comments
- Alternative: SET NULL with special UI for "deleted" comments

</details>

<details>
<summary><strong>Answer 4: Schema Design Challenge</strong></summary>

```sql
-- Projects table
CREATE TABLE projects (
    project_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    project_name VARCHAR(255) NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Users table
CREATE TABLE users (
    user_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);

-- Tasks table (with self-reference for subtasks)
CREATE TABLE tasks (
    task_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    project_id BIGINT NOT NULL,
    parent_task_id BIGINT NULL,  -- Self-reference for subtasks
    title VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(20) DEFAULT 'TODO',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (project_id) 
        REFERENCES projects(project_id)
        ON DELETE CASCADE,
        -- WHY: Tasks belong to projects. Delete project → delete all tasks
    
    FOREIGN KEY (parent_task_id)
        REFERENCES tasks(task_id)
        ON DELETE CASCADE
        -- WHY: Subtasks belong to parent tasks. Delete parent → delete subtasks
);

-- Task assignments (many-to-many: users ↔ tasks)
CREATE TABLE task_assignments (
    assignment_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    task_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE KEY unique_assignment (task_id, user_id),
    
    FOREIGN KEY (task_id)
        REFERENCES tasks(task_id)
        ON DELETE CASCADE,
        -- WHY: Assignment meaningless without task. Delete task → delete assignment
    
    FOREIGN KEY (user_id)
        REFERENCES users(user_id)
        ON DELETE CASCADE
        -- WHY: If user account deleted, remove their assignments too
        -- ALTERNATIVE: ON DELETE SET NULL if you want to preserve history
);

-- Comments (with self-reference for threading)
CREATE TABLE comments (
    comment_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    task_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    parent_comment_id BIGINT NULL,  -- Self-reference for replies
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (task_id)
        REFERENCES tasks(task_id)
        ON DELETE CASCADE,
        -- WHY: Comments belong to tasks. Delete task → delete comments
    
    FOREIGN KEY (user_id)
        REFERENCES users(user_id)
        ON DELETE SET NULL,
        -- WHY: Preserve comment history even if user deleted
        -- Show as "Deleted User" in UI
    
    FOREIGN KEY (parent_comment_id)
        REFERENCES comments(comment_id)
        ON DELETE CASCADE
        -- WHY: Reply threads. Delete parent comment → delete all replies
        -- ALTERNATIVE: SET NULL + special UI for "[deleted]" parent
);
```

**Decision Summary:**

| Relationship | Action | Reasoning |
|--------------|--------|-----------|
| project → tasks | CASCADE | Tasks can't exist without project |
| task → subtasks | CASCADE | Subtasks are part of parent task |
| task → assignments | CASCADE | No point keeping assignments for deleted tasks |
| user → assignments | CASCADE | Remove deleted user's assignments |
| task → comments | CASCADE | Comments belong to task context |
| user → comments | SET NULL | Preserve comment history, show as "Deleted User" |
| comment → replies | CASCADE | Replies need parent for context |

</details>

<details>
<summary><strong>Answer 5: Query Challenge</strong></summary>

**a) Find all direct reports of Bob (employee_id = 2):**
```sql
SELECT employee_id, name
FROM employees
WHERE manager_id = 2;

-- Result:
-- 4, 'David'
-- 5, 'Eve'
```

**b) Find Bob's manager:**
```sql
SELECT m.employee_id, m.name
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id
WHERE e.employee_id = 2;

-- Result:
-- 1, 'Alice'

-- Alternative (simpler):
SELECT manager_id
FROM employees
WHERE employee_id = 2;
-- Returns: 1 (then look up separately)
```

**c) Find all employees at the same level as David (2 levels below CEO):**
```sql
WITH RECURSIVE employee_levels AS (
    -- Base: CEO is level 0
    SELECT employee_id, name, manager_id, 0 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: add one level each iteration
    SELECT e.employee_id, e.name, e.manager_id, el.level + 1
    FROM employees e
    JOIN employee_levels el ON e.manager_id = el.employee_id
)
SELECT employee_id, name, level
FROM employee_levels
WHERE level = (
    SELECT level FROM employee_levels WHERE employee_id = 4
);

-- Result:
-- 4, 'David', 2
-- 5, 'Eve', 2
-- (Both are at level 2: CEO → Manager → Employee)

-- Simpler non-recursive approach (only works for fixed depth):
SELECT e1.employee_id, e1.name
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.employee_id
JOIN employees e3 ON e2.manager_id = e3.employee_id
WHERE e3.manager_id IS NULL;  -- e3 is CEO
-- This finds all level-2 employees (2 hops from CEO)
```

</details>

---

## 2.3 Other Key Types

### 2.3.1 Unique Keys

A **UNIQUE constraint** ensures that all values in a column (or combination of columns) are distinct across all rows. Unlike primary keys, a table can have multiple unique constraints, and unique columns can contain NULL values (though typically only one NULL per column in most databases).

**Primary Key vs Unique Key:**

| Feature | Primary Key | Unique Key |
|---------|-------------|------------|
| **Number per table** | Exactly one | Multiple allowed |
| **NULL values** | Not allowed | Allowed (usually one NULL) |
| **Purpose** | Unique identifier for row | Prevent duplicate business values |
| **Automatic index** | Yes | Yes |
| **Foreign key target** | Yes (default) | Yes (less common) |

**Real-World Example: User Registration System**
```sql
CREATE TABLE users (
    user_id BIGINT AUTO_INCREMENT PRIMARY KEY,     -- Surrogate key
    username VARCHAR(50) UNIQUE NOT NULL,           -- Natural identifier 1
    email VARCHAR(255) UNIQUE NOT NULL,             -- Natural identifier 2
    phone_number VARCHAR(20) UNIQUE,                -- Optional unique field
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Why This Design:**
- `user_id`: Internal database ID (primary key) - never exposed to users, never changes
- `username`: User-facing identifier - must be unique for login
- `email`: Another way to identify user - must be unique, used for password reset
- `phone_number`: Optional, but if provided must be unique

**What UNIQUE Prevents:**
```sql
-- First user
INSERT INTO users (username, email, password_hash)
VALUES ('alice', 'alice@example.com', 'hash123');
-- ✓ Success

-- Try to register with same username
INSERT INTO users (username, email, password_hash)
VALUES ('alice', 'different@example.com', 'hash456');
-- ERROR: Duplicate entry 'alice' for key 'username'

-- Try to register with same email
INSERT INTO users (username, email, password_hash)
VALUES ('alice2', 'alice@example.com', 'hash789');
-- ERROR: Duplicate entry 'alice@example.com' for key 'email'
```

---

**E-commerce Example: Product SKUs**
```sql
CREATE TABLE products (
    product_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    sku VARCHAR(50) UNIQUE NOT NULL,  -- Stock Keeping Unit
    upc VARCHAR(12) UNIQUE,            -- Universal Product Code (barcode)
    product_name VARCHAR(255) NOT NULL,
    price DECIMAL(10, 2),
    stock_quantity INT DEFAULT 0
);
```

**Real-World Scenario:**
```sql
-- Amazon's system
INSERT INTO products (sku, upc, product_name, price)
VALUES ('AMZN-LAPTOP-001', '012345678901', 'Dell XPS 13', 999.99);

-- Try to add another product with same SKU
INSERT INTO products (sku, upc, product_name, price)
VALUES ('AMZN-LAPTOP-001', '987654321098', 'HP Spectre', 1099.99);
-- ERROR: Prevents inventory confusion

-- UPC can be NULL for custom/handmade products
INSERT INTO products (sku, upc, product_name, price)
VALUES ('AMZN-CRAFT-042', NULL, 'Handmade Scarf', 29.99);
-- ✓ Success
```

---

**Composite Unique Constraints**

Multiple columns together must be unique, but individual columns can have duplicates.

**Real-World Example: Course Enrollment**
```sql
CREATE TABLE course_offerings (
    offering_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    course_code VARCHAR(10) NOT NULL,      -- CS101
    semester VARCHAR(10) NOT NULL,         -- FALL-2024
    section VARCHAR(5) NOT NULL,           -- A, B, C
    instructor_id BIGINT NOT NULL,
    max_students INT,
    UNIQUE KEY unique_offering (course_code, semester, section)
);
```

**Why Composite Unique:**
```sql
-- Valid: Same course, different semesters
INSERT INTO course_offerings (course_code, semester, section, instructor_id, max_students)
VALUES ('CS101', 'FALL-2024', 'A', 101, 30);

INSERT INTO course_offerings (course_code, semester, section, instructor_id, max_students)
VALUES ('CS101', 'SPRING-2025', 'A', 101, 30);
-- ✓ Different semester, allowed

-- Valid: Same course + semester, different section
INSERT INTO course_offerings (course_code, semester, section, instructor_id, max_students)
VALUES ('CS101', 'FALL-2024', 'B', 102, 25);
-- ✓ Different section, allowed

-- Invalid: Exact same course + semester + section
INSERT INTO course_offerings (course_code, semester, section, instructor_id, max_students)
VALUES ('CS101', 'FALL-2024', 'A', 103, 35);
-- ERROR: Can't have two "CS101 Fall-2024 Section A" offerings
```

---

**Flight Booking Example:**
```sql
CREATE TABLE flights (
    flight_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    flight_number VARCHAR(10) NOT NULL,    -- AA100
    departure_date DATE NOT NULL,          -- 2024-06-15
    origin VARCHAR(3) NOT NULL,            -- JFK
    destination VARCHAR(3) NOT NULL,       -- LAX
    departure_time TIME,
    UNIQUE KEY unique_flight (flight_number, departure_date)
);
```

**Business Logic:**
```sql
-- AA100 on June 15
INSERT INTO flights (flight_number, departure_date, origin, destination, departure_time)
VALUES ('AA100', '2024-06-15', 'JFK', 'LAX', '08:00:00');

-- AA100 on June 16 (different day, same flight number)
INSERT INTO flights (flight_number, departure_date, origin, destination, departure_time)
VALUES ('AA100', '2024-06-16', 'JFK', 'LAX', '08:00:00');
-- ✓ Allowed: Flight numbers repeat daily

-- Try to create duplicate flight
INSERT INTO flights (flight_number, departure_date, origin, destination, departure_time)
VALUES ('AA100', '2024-06-15', 'JFK', 'SFO', '10:00:00');
-- ERROR: AA100 on June 15 already exists (even with different destination)
```

---

**NULL Behavior in Unique Constraints:**

**MySQL Quirk:**
```sql
CREATE TABLE contacts (
    contact_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    alternate_email VARCHAR(255) UNIQUE  -- Optional
);

-- Multiple NULLs are allowed
INSERT INTO contacts (name, alternate_email) VALUES ('Alice', NULL);
INSERT INTO contacts (name, alternate_email) VALUES ('Bob', NULL);
INSERT INTO contacts (name, alternate_email) VALUES ('Charlie', NULL);
-- ✓ All succeed: NULL != NULL in unique constraint logic

-- But actual values must be unique
INSERT INTO contacts (name, alternate_email) VALUES ('David', 'david@example.com');
INSERT INTO contacts (name, alternate_email) VALUES ('Eve', 'david@example.com');
-- ERROR: Duplicate entry 'david@example.com'
```

**PostgreSQL/SQL Server:** Some databases have stricter NULL handling - check documentation.

---

**When to Use UNIQUE Constraints:**

**1. Natural Identifiers (username, email, SSN):**
```sql
CREATE TABLE employees (
    employee_id BIGINT PRIMARY KEY,
    ssn CHAR(11) UNIQUE,           -- Social Security Number
    employee_number VARCHAR(10) UNIQUE,  -- Company ID
    email VARCHAR(255) UNIQUE
);
```

**2. Preventing Duplicate Business Logic:**
```sql
-- Stripe-style payment intents
CREATE TABLE payment_intents (
    intent_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    idempotency_key VARCHAR(255) UNIQUE,  -- Prevents duplicate charges
    customer_id BIGINT NOT NULL,
    amount DECIMAL(10, 2),
    currency CHAR(3),
    status VARCHAR(20)
);

-- Client retries same request with same idempotency_key
-- Database prevents duplicate charge automatically
```

**3. Many-to-Many Junction Tables:**
```sql
CREATE TABLE student_courses (
    enrollment_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    student_id BIGINT NOT NULL,
    course_id BIGINT NOT NULL,
    semester VARCHAR(10),
    UNIQUE KEY unique_enrollment (student_id, course_id, semester),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
-- Prevents enrolling same student in same course twice in same semester
```

---

**Checking for Violations Before Insert:**
```sql
-- Application-level check
SELECT COUNT(*) FROM users WHERE username = 'alice';
-- If 0, proceed with insert

-- But this has a race condition! Better to let database enforce:
INSERT INTO users (username, email, password_hash)
VALUES ('alice', 'alice@example.com', 'hash')
ON DUPLICATE KEY UPDATE user_id = user_id;  -- MySQL: no-op if duplicate

-- Or catch the exception in application code
try:
    db.insert_user('alice', 'alice@example.com', 'hash')
except DuplicateKeyError:
    return "Username already taken"
```

---

**Finding Duplicate Data (Before Adding Constraint):**
```sql
-- Identify duplicates before adding UNIQUE constraint
SELECT email, COUNT(*) as count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Result: Shows emails that appear more than once
-- Clean these up before adding UNIQUE constraint
```

**Adding UNIQUE to Existing Table:**
```sql
-- This will fail if duplicates exist
ALTER TABLE users ADD UNIQUE KEY unique_email (email);

-- Must clean data first:
-- 1. Find duplicates
-- 2. Decide which to keep
-- 3. Delete or update duplicates
-- 4. Then add constraint
```

### 2.3.2 Candidate Keys

A **candidate key** is any column (or set of columns) that could serve as a primary key. Every candidate key must be:
- **Unique**: No duplicate values
- **NOT NULL**: Every row must have a value
- **Minimal**: No unnecessary columns in composite keys

One candidate key is chosen as the **primary key**; the others become **alternate keys**.

**Real-World Example: Employee Database**
```sql
CREATE TABLE employees (
    employee_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Chosen as primary key
    ssn CHAR(11) UNIQUE NOT NULL,      -- Candidate key 1
    employee_number VARCHAR(10) UNIQUE NOT NULL,    -- Candidate key 2
    email VARCHAR(255) UNIQUE NOT NULL,             -- Candidate key 3
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    hire_date DATE
);
```

**Candidate Keys Identified:**
1. `employee_id` - Surrogate key ✓ (CHOSEN as primary key)
2. `ssn` - Natural key, uniquely identifies person ✓
3. `employee_number` - Company-assigned unique ID ✓
4. `email` - Unique company email ✓

**Why `employee_id` was chosen:**
- **Immutable**: Never changes (SSN, email, employee_number might)
- **Simple**: Single integer column (vs 11-character SSN)
- **No privacy concerns**: Not sensitive data
- **Performance**: Integer joins are faster
- **Database-generated**: No manual assignment needed

---

**Superkey vs Candidate Key:**

**Superkey**: Any combination that uniquely identifies a row (can have extra columns)
```sql
-- All are superkeys:
(employee_id)                    -- ✓ Minimal
(ssn)                           -- ✓ Minimal
(employee_id, first_name)       -- ✗ Not minimal (first_name is redundant)
(ssn, email)                    -- ✗ Not minimal (both individually unique)
(first_name, last_name, hire_date) -- Might be unique, but not guaranteed
```

**Candidate Key**: Minimal superkey (no unnecessary columns)
```sql
-- Candidate keys (minimal):
(employee_id)
(ssn)
(employee_number)
(email)
```

---

**Real-World Example: University Student Records**
```sql
CREATE TABLE students (
    student_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    university_id VARCHAR(10) UNIQUE NOT NULL,  -- Student ID assigned by school
    national_id VARCHAR(20) UNIQUE NOT NULL,    -- Government ID
    passport_number VARCHAR(20) UNIQUE,         -- NULL for domestic students
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    date_of_birth DATE
);
```

**Candidate Keys Analysis:**

| Column | Unique? | NOT NULL? | Candidate Key? | Why/Why Not |
|--------|---------|-----------|----------------|-------------|
| `student_id` | ✓ | ✓ | ✓ | Chosen as PK |
| `university_id` | ✓ | ✓ | ✓ | School-assigned unique ID |
| `national_id` | ✓ | ✓ | ✓ | Government ID unique per person |
| `passport_number` | ✓ | ✗ | ✗ | Can be NULL (domestic students) |
| `email` | ✓ | ✓ | ✓ | Unique school email |
| `(first_name, last_name, date_of_birth)` | Maybe | ✓ | ✗ | Not guaranteed unique |

**Valid Candidate Keys:**
1. `student_id` (primary key)
2. `university_id`
3. `national_id`
4. `email`

**Not Candidate Keys:**
- `passport_number` - Can be NULL
- `(first_name, last_name)` - Not guaranteed unique

---

**Composite Candidate Keys**

Sometimes a single column isn't unique, but a combination is.

**Real-World Example: Flight Seats**
```sql
CREATE TABLE flight_seats (
    flight_id BIGINT NOT NULL,
    seat_number VARCHAR(5) NOT NULL,  -- "12A", "15F"
    passenger_id BIGINT,
    seat_class VARCHAR(20),
    PRIMARY KEY (flight_id, seat_number),  -- Composite primary key (also a candidate key)
    FOREIGN KEY (flight_id) REFERENCES flights(flight_id),
    FOREIGN KEY (passenger_id) REFERENCES passengers(passenger_id)
);
```

**Candidate Keys:**
1. `(flight_id, seat_number)` - Composite candidate key (chosen as PK)
   - `flight_id` alone: Not unique (many seats per flight)
   - `seat_number` alone: Not unique (seat "12A" exists on every flight)
   - Together: Unique (only one "12A" on flight AA100 on June 15)

**Alternative: Surrogate Key**
```sql
CREATE TABLE flight_seats (
    seat_assignment_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Surrogate key
    flight_id BIGINT NOT NULL,
    seat_number VARCHAR(5) NOT NULL,
    passenger_id BIGINT,
    UNIQUE KEY unique_seat (flight_id, seat_number),  -- Composite unique (candidate key)
    FOREIGN KEY (flight_id) REFERENCES flights(flight_id)
);
```

Now there are TWO candidate keys:
1. `seat_assignment_id` (chosen as PK)
2. `(flight_id, seat_number)` (alternate key via UNIQUE constraint)

---

**Real-World Example: Hotel Room Bookings**
```sql
CREATE TABLE room_bookings (
    booking_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Surrogate candidate key
    hotel_id INT NOT NULL,
    room_number VARCHAR(10) NOT NULL,
    check_in_date DATE NOT NULL,
    check_out_date DATE NOT NULL,
    guest_id BIGINT NOT NULL,
    UNIQUE KEY unique_room_date (hotel_id, room_number, check_in_date),  -- Composite candidate key
    FOREIGN KEY (hotel_id) REFERENCES hotels(hotel_id),
    FOREIGN KEY (guest_id) REFERENCES guests(guest_id)
);
```

**Candidate Keys:**
1. `booking_id` (primary key - easier for references)
2. `(hotel_id, room_number, check_in_date)` - Room can't be double-booked on same date

**Why composite is candidate:**
- Room 305 at Hotel Marriott on 2024-06-15 can only have one booking
- But room 305 on 2024-06-16 can have different booking
- So all three columns together uniquely identify a booking

---

**Choosing the Best Candidate Key as Primary Key:**

**Criteria:**
1. **Stability**: Never changes (emails change, SSNs don't, but privacy concerns)
2. **Simplicity**: Single column preferred over composite
3. **Performance**: Smaller data types (INT/BIGINT better than VARCHAR)
4. **Privacy**: Non-sensitive (don't use SSN even if unique)
5. **Meaningless**: Surrogate keys prevent business logic coupling

**Decision Tree:**
```
Is there a single, stable, non-sensitive natural key?
├─ YES: Consider using it as PK (e.g., ISBN for books)
└─ NO: Use surrogate key (AUTO_INCREMENT) as PK
    └─ Keep natural keys as UNIQUE constraints
```

**Examples:**

**Good Natural Key (Used as PK):**
```sql
CREATE TABLE books (
    isbn VARCHAR(13) PRIMARY KEY,  -- ISBN never changes, globally unique
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255)
);
```

**Bad Natural Key (Use Surrogate Instead):**
```sql
-- DON'T DO THIS
CREATE TABLE customers (
    email VARCHAR(255) PRIMARY KEY,  -- Email can change!
    first_name VARCHAR(50),
    last_name VARCHAR(50)
);

-- DO THIS INSTEAD
CREATE TABLE customers (
    customer_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Stable
    email VARCHAR(255) UNIQUE NOT NULL,             -- Keep uniqueness
    first_name VARCHAR(50),
    last_name VARCHAR(50)
);
```

---

**Finding Candidate Keys in Existing Data:**

```sql
-- Check if column is unique (potential candidate key)
SELECT email, COUNT(*) as count
FROM customers
GROUP BY email
HAVING COUNT(*) > 1;
-- If returns 0 rows, email is unique → candidate key

-- Check for NULLs (disqualifies from being candidate key)
SELECT COUNT(*) FROM customers WHERE email IS NULL;
-- If > 0, can't be candidate key (must be NOT NULL)

-- Composite key check
SELECT hotel_id, room_number, check_in_date, COUNT(*) as count
FROM room_bookings
GROUP BY hotel_id, room_number, check_in_date
HAVING COUNT(*) > 1;
-- If 0 rows, the combination is unique → candidate key
```

### 2.3.3 Alternate Keys

An **alternate key** is a candidate key that was NOT chosen as the primary key. It's implemented as a UNIQUE constraint in the database.

**Simple Definition:**
```
Candidate Keys - Primary Key = Alternate Keys
```

**Real-World Example: User Account System**
```sql
CREATE TABLE users (
    user_id BIGINT AUTO_INCREMENT PRIMARY KEY,     -- Primary key (chosen candidate)
    username VARCHAR(50) UNIQUE NOT NULL,           -- Alternate key 1
    email VARCHAR(255) UNIQUE NOT NULL,             -- Alternate key 2
    phone_number VARCHAR(20) UNIQUE,                -- Alternate key 3 (optional)
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Breakdown:**
- **Candidate keys**: user_id, username, email, phone_number (all uniquely identify a user)
- **Primary key**: user_id (chosen for internal use)
- **Alternate keys**: username, email, phone_number (implemented as UNIQUE constraints)

**Why Keep Alternate Keys:**
1. **Multiple Ways to Identify**: Users can log in with username OR email
2. **Business Rules**: Username must be unique even if not the primary key
3. **Data Integrity**: Prevent duplicate usernames/emails
4. **Natural Search**: Find user by email without knowing user_id

---

**Real-World Example: E-commerce Products**
```sql
CREATE TABLE products (
    product_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Internal database ID
    sku VARCHAR(50) UNIQUE NOT NULL,               -- Alternate key 1 (Stock Keeping Unit)
    upc VARCHAR(12) UNIQUE,                        -- Alternate key 2 (Barcode - optional)
    product_name VARCHAR(255) NOT NULL,
    price DECIMAL(10, 2),
    stock_quantity INT
);
```

**Different Ways to Identify a Product:**
```sql
-- By primary key (internal operations)
SELECT * FROM products WHERE product_id = 12345;

-- By SKU (warehouse operations, inventory system)
SELECT * FROM products WHERE sku = 'LAPTOP-DELL-XPS13-001';

-- By UPC (point-of-sale scanning)
SELECT * FROM products WHERE upc = '012345678901';
```

**Why Each Key Exists:**
- `product_id`: Database efficiency, foreign key relationships
- `sku`: Human-readable product identifier for staff
- `upc`: Standard barcode for retail scanning

---

**Banking Example:**
```sql
CREATE TABLE accounts (
    account_id BIGINT AUTO_INCREMENT PRIMARY KEY,           -- Internal ID
    account_number VARCHAR(20) UNIQUE NOT NULL,             -- Alternate key 1 (Customer-facing)
    iban VARCHAR(34) UNIQUE,                                -- Alternate key 2 (International)
    customer_id BIGINT NOT NULL,
    account_type VARCHAR(20),
    balance DECIMAL(15, 2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

**Usage:**
```sql
-- Internal database queries
SELECT balance FROM accounts WHERE account_id = 98765;

-- Customer service lookup
SELECT * FROM accounts WHERE account_number = '1234567890';

-- International wire transfer
SELECT * FROM accounts WHERE iban = 'DE89370400440532013000';
```

---

**Composite Alternate Keys**

**Real-World Example: University Course Sections**
```sql
CREATE TABLE course_sections (
    section_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Surrogate primary key
    course_code VARCHAR(10) NOT NULL,               -- CS101
    semester VARCHAR(10) NOT NULL,                  -- FALL-2024
    section_number VARCHAR(5) NOT NULL,             -- A, B, C
    instructor_id BIGINT NOT NULL,
    max_students INT,
    UNIQUE KEY alternate_key (course_code, semester, section_number),  -- Composite alternate key
    FOREIGN KEY (instructor_id) REFERENCES instructors(instructor_id)
);
```

**Why This Design:**
- **Primary key**: `section_id` - Simple integer for database efficiency
- **Alternate key**: `(course_code, semester, section_number)` - How humans identify sections

**Both work for lookups:**
```sql
-- Internal database join (fast integer comparison)
SELECT * FROM enrollments WHERE section_id = 42;

-- Natural human query
SELECT * FROM course_sections 
WHERE course_code = 'CS101' 
  AND semester = 'FALL-2024' 
  AND section_number = 'A';
```

---

**Foreign Keys Can Reference Alternate Keys**

While uncommon, foreign keys can reference alternate keys (UNIQUE constraints), not just primary keys.

```sql
CREATE TABLE orders (
    order_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    customer_email VARCHAR(255) NOT NULL,  -- References users.email (alternate key)
    order_date TIMESTAMP,
    FOREIGN KEY (customer_email) REFERENCES users(email)  -- References alternate key!
);
```

**Why You Might Do This:**
- External system integration (only has email, not user_id)
- API design (exposing email is acceptable, user_id is not)

**Why It's Usually Avoided:**
- **Performance**: String comparison slower than integer
- **Updates**: Changing email requires updating all foreign key references
- **Size**: Email foreign keys are 255 bytes vs 8 bytes for BIGINT

**Better Approach:**
```sql
CREATE TABLE orders (
    order_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    customer_id BIGINT NOT NULL,  -- Reference primary key
    order_date TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES users(user_id)  -- Integer FK
);
```

---

**Alternate Keys in Application Logic**

Alternate keys are crucial for business operations:

**1. Login System (Username or Email):**
```sql
-- User can log in with username
SELECT user_id, password_hash FROM users WHERE username = 'alice';

-- Or with email
SELECT user_id, password_hash FROM users WHERE email = 'alice@example.com';

-- Both work because both are alternate keys (UNIQUE)
```

**2. Idempotency in APIs (Stripe-style):**
```sql
CREATE TABLE payment_intents (
    intent_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    idempotency_key VARCHAR(255) UNIQUE NOT NULL,  -- Alternate key
    customer_id BIGINT NOT NULL,
    amount DECIMAL(10, 2),
    status VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Client retries request with same idempotency key
-- Prevents duplicate charges
INSERT INTO payment_intents (idempotency_key, customer_id, amount)
VALUES ('unique-key-123', 1001, 99.99);
-- First time: succeeds

-- Retry due to network issue
INSERT INTO payment_intents (idempotency_key, customer_id, amount)
VALUES ('unique-key-123', 1001, 99.99);
-- ERROR: Duplicate key (prevented duplicate charge!)

-- API returns existing payment_intent instead of creating new
SELECT * FROM payment_intents WHERE idempotency_key = 'unique-key-123';
```

**3. Natural Lookups:**
```sql
-- Support staff: "Find order by order number"
SELECT * FROM orders WHERE order_number = 'ORD-2024-00123';

-- Warehouse: "Find product by SKU"
SELECT * FROM products WHERE sku = 'WIDGET-RED-M';

-- Customer: "Track my package by tracking number"
SELECT * FROM shipments WHERE tracking_number = '1Z999AA10123456784';
```

---

**When to Use Alternate Keys:**

**DO use alternate keys when:**
1. Multiple natural identifiers exist (email AND username)
2. External systems reference different identifiers (SKU, UPC, GTIN)
3. Business processes rely on specific identifiers (account number, order number)
4. Preventing duplicates is critical (idempotency keys)

**DON'T create alternate keys when:**
1. Column values might change frequently (mailing address)
2. Values aren't truly unique in practice (phone numbers - shared landlines)
3. Column allows NULLs and multiple NULLs exist
4. Unnecessary constraints that limit flexibility

---

**Summary Table:**

| Concept | Definition | Implementation | Example |
|---------|------------|----------------|---------|
| **Candidate Key** |Column(s) that could be primary key | UNIQUE + NOT NULL | username, email, ssn |
| **Primary Key** | Chosen candidate key | PRIMARY KEY | user_id |
| **Alternate Key** | Non-chosen candidate keys | UNIQUE NOT NULL | username, email (if user_id is PK) |

**Relationship:**
```
All candidate keys are UNIQUE + NOT NULL
Primary key is one candidate key
Alternate keys are all other candidate keys
```

### 2.3.4 Composite Keys

A **composite key** is a key made up of two or more columns that together uniquely identify a row. No single column alone is unique, but the combination is.

**When to Use Composite Keys:**
1. **Junction/Bridge tables** (many-to-many relationships)
2. **Time-series data** (same entity across different time periods)
3. **Multi-tenant systems** (same ID across different tenants)
4. **Natural business constraints** (room + date, seat + flight)

---

**Real-World Example 1: Many-to-Many Relationships**

**Student-Course Enrollment:**
```sql
CREATE TABLE enrollments (
    student_id BIGINT NOT NULL,
    course_id BIGINT NOT NULL,
    semester VARCHAR(10) NOT NULL,
    grade CHAR(2),
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id, semester),  -- Composite primary key
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

**Why Composite:**
```sql
-- Student 1001 enrolls in CS101 for Fall 2024
INSERT INTO enrollments (student_id, course_id, semester, grade)
VALUES (1001, 101, 'FALL-2024', NULL);

-- Same student enrolls in different course (different course_id)
INSERT INTO enrollments (student_id, course_id, semester)
VALUES (1001, 102, 'FALL-2024');  -- ✓ Valid

-- Same student retakes same course in different semester (different semester)
INSERT INTO enrollments (student_id, course_id, semester)
VALUES (1001, 101, 'SPRING-2025');  -- ✓ Valid (retaking)

-- Try to enroll same student in same course in same semester
INSERT INTO enrollments (student_id, course_id, semester)
VALUES (1001, 101, 'FALL-2024');  
-- ERROR: Duplicate entry for key 'PRIMARY'
```

**Each Column Alone is NOT Unique:**
- `student_id = 1001`: Appears 3 times (multiple courses)
- `course_id = 101`: Appears 2 times (multiple students take it)
- `semester = 'FALL-2024'`: Appears 2 times (multiple enrollments)
- **Together**: `(1001, 101, 'FALL-2024')` appears only once ✓

---

**Real-World Example 2: Hotel Reservations**
```sql
CREATE TABLE room_bookings (
    hotel_id INT NOT NULL,
    room_number VARCHAR(10) NOT NULL,
    check_in_date DATE NOT NULL,
    guest_id BIGINT NOT NULL,
    check_out_date DATE NOT NULL,
    total_price DECIMAL(10, 2),
    PRIMARY KEY (hotel_id, room_number, check_in_date),  -- Composite key
    FOREIGN KEY (hotel_id) REFERENCES hotels(hotel_id),
    FOREIGN KEY (guest_id) REFERENCES guests(guest_id)
);
```

**Business Logic:**
```sql
-- Room 305 at Hotel Marriott for June 15
INSERT INTO room_bookings (hotel_id, room_number, check_in_date, guest_id, check_out_date, total_price)
VALUES (1, '305', '2024-06-15', 5001, '2024-06-18', 450.00);

-- Same room, different hotel (different hotel_id)
INSERT INTO room_bookings (hotel_id, room_number, check_in_date, guest_id, check_out_date, total_price)
VALUES (2, '305', '2024-06-15', 5002, '2024-06-17', 300.00);  -- ✓ Different hotel

-- Same room at same hotel, different date (different check_in_date)
INSERT INTO room_bookings (hotel_id, room_number, check_in_date, guest_id, check_out_date, total_price)
VALUES (1, '305', '2024-06-20', 5003, '2024-06-22', 300.00);  -- ✓ Different date

-- Try to double-book same room on same date
INSERT INTO room_bookings (hotel_id, room_number, check_in_date, guest_id, check_out_date, total_price)
VALUES (1, '305', '2024-06-15', 5004, '2024-06-19', 600.00);
-- ERROR: Duplicate entry - room already booked
```

**Why Three Columns Needed:**
- `hotel_id` alone: Not unique (many rooms per hotel)
- `room_number` alone: Not unique (room 305 exists at every hotel)
- `check_in_date` alone: Not unique (many rooms booked on same date)
- **Together**: Uniquely identifies a specific room reservation

---

**Real-World Example 3: Product Prices Over Time**
```sql
CREATE TABLE product_price_history (
    product_id BIGINT NOT NULL,
    effective_date DATE NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    currency CHAR(3) DEFAULT 'USD',
    PRIMARY KEY (product_id, effective_date),  -- Composite key
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

**Price Changes Over Time:**
```sql
-- Product 100 costs $50 starting Jan 1, 2024
INSERT INTO product_price_history (product_id, effective_date, price)
VALUES (100, '2024-01-01', 50.00);

-- Price increases to $55 on March 1, 2024
INSERT INTO product_price_history (product_id, effective_date, price)
VALUES (100, '2024-03-01', 55.00);

-- Sale price $45 on June 1, 2024
INSERT INTO product_price_history (product_id, effective_date, price)
VALUES (100, '2024-06-01', 45.00);

-- Query: What was the price on May 15, 2024?
SELECT price 
FROM product_price_history
WHERE product_id = 100 
  AND effective_date <= '2024-05-15'
ORDER BY effective_date DESC
LIMIT 1;
-- Result: $55 (price effective from March 1)
```

---

**Composite Keys in Junction Tables**

**Example: User Roles (Many-to-Many)**
```sql
CREATE TABLE user_roles (
    user_id BIGINT NOT NULL,
    role_id INT NOT NULL,
    assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    assigned_by BIGINT,
    PRIMARY KEY (user_id, role_id),  -- User can't have same role twice
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (role_id) REFERENCES roles(role_id)
);
```

**Usage:**
```sql
-- Grant admin role to user
INSERT INTO user_roles (user_id, role_id, assigned_by)
VALUES (1001, 1, 9999);  -- user 1001, role 1 (admin)

-- Grant editor role to same user
INSERT INTO user_roles (user_id, role_id)
VALUES (1001, 2);  -- ✓ Different role

-- Try to grant admin role again
INSERT INTO user_roles (user_id, role_id)
VALUES (1001, 1);
-- ERROR: Duplicate - user already has this role
```

---

**Composite Foreign Keys**

When a child table references a composite primary key in a parent table.

```sql
-- Parent: Order line items (composite PK)
CREATE TABLE order_items (
    order_id BIGINT NOT NULL,
    line_number INT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INT,
    unit_price DECIMAL(10, 2),
    PRIMARY KEY (order_id, line_number),
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

-- Child: Shipment tracking (must reference composite)
CREATE TABLE item_shipments (
    shipment_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    order_id BIGINT NOT NULL,
    line_number INT NOT NULL,
    tracking_number VARCHAR(50),
    shipped_date DATE,
    FOREIGN KEY (order_id, line_number) 
        REFERENCES order_items(order_id, line_number)  -- Composite FK!
);
```

**Why Composite FK:**
```sql
-- Order 1 has multiple line items
INSERT INTO order_items (order_id, line_number, product_id, quantity, unit_price)
VALUES 
    (1, 1, 501, 2, 29.99),   -- Line 1: 2x Product 501
    (1, 2, 502, 1, 49.99);   -- Line 2: 1x Product 502

-- Ship line 1 of order 1
INSERT INTO item_shipments (order_id, line_number, tracking_number)
VALUES (1, 1, '1Z999AA1234567890');  -- ✓ Valid

-- Try to ship non-existent line
INSERT INTO item_shipments (order_id, line_number, tracking_number)
VALUES (1, 99, '1Z999AA0987654321');
-- ERROR: Foreign key constraint fails (line 99 doesn't exist)
```

---

**Alternative: Surrogate Key + Composite Unique Constraint**

Many developers prefer a surrogate primary key even for junction tables:

```sql
-- Instead of composite PRIMARY KEY
CREATE TABLE enrollments (
    enrollment_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Surrogate key
    student_id BIGINT NOT NULL,
    course_id BIGINT NOT NULL,
    semester VARCHAR(10) NOT NULL,
    UNIQUE KEY unique_enrollment (student_id, course_id, semester),  -- Composite UNIQUE
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

**Advantages of Surrogate Approach:**
1. **Simpler foreign keys**: Other tables reference single `enrollment_id`
2. **Easier updates**: Composite keys are harder to update
3. **Framework compatibility**: ORMs prefer single-column PKs
4. **URL friendliness**: `/enrollments/12345` vs `/enrollments/1001/101/FALL-2024`

**Advantages of Composite Primary Key:**
1. **Storage efficiency**: No extra column needed
2. **Natural constraint**: Business rule is the primary key itself
3. **Semantic clarity**: Key directly represents the business concept

---

**Querying Composite Keys:**

**Find by exact composite:**
```sql
SELECT * FROM enrollments
WHERE student_id = 1001 
  AND course_id = 101 
  AND semester = 'FALL-2024';
```

**Find by partial composite (must include leftmost columns for index efficiency):**
```sql
-- Uses composite index efficiently
SELECT * FROM enrollments WHERE student_id = 1001;

-- Also efficient
SELECT * FROM enrollments 
WHERE student_id = 1001 AND course_id = 101;

-- Less efficient (doesn't use composite index well)
SELECT * FROM enrollments WHERE course_id = 101;
```

---

**Common Patterns:**

| Use Case | Composite Key Columns | Example |
|----------|----------------------|---------|
| Many-to-many | (entity1_id, entity2_id) | (user_id, role_id) |
| Time-series | (entity_id, timestamp/date) | (sensor_id, reading_time) |
| Multi-tenant | (tenant_id, local_id) | (company_id, employee_number) |
| Hierarchical | (parent_id, child_seq) | (order_id, line_number) |
| Versioning | (entity_id, version) | (document_id, version_number) |

---

### Section 2.3 Practice Questions

<details>
<summary><strong>Question 1: Fill in the Blanks</strong></summary>

1. A UNIQUE constraint ensures all values in a column are _______, but unlike a primary key, a table can have _______ unique constraints.
2. A _______ key is any column that could serve as a primary key, while an _______ key is a candidate key that was not chosen as the primary key.
3. Composite keys are made up of _______ or more columns that together _______ identify a row.
4. In most databases, UNIQUE constraints allow _______ NULL value(s), while primary keys allow _______ NULL values.
5. Alternate keys are implemented in SQL using _______ constraints.

</details>

<details>
<summary><strong>Question 2: True or False</strong></summary>

a) A table can have multiple primary keys.

b) UNIQUE constraints automatically create indexes on the column(s).

c) Candidate keys must be NOT NULL.

d) Composite keys are always better than surrogate keys.

e) Foreign keys can reference alternate keys (UNIQUE constraints), not just primary keys.

f) In a composite key (A, B, C), each individual column must be unique.

g) All alternate keys are candidate keys, but not all candidate keys are alternate keys.

</details>

<details>
<summary><strong>Question 3: Multiple Choice</strong></summary>

**a) Which is NOT a requirement for a candidate key?**
- A) Must be unique
- B) Must be NOT NULL
- C) Must be a single column
- D) Must be minimal (no unnecessary columns)

**b) For a users table with user_id (PK), username (UNIQUE), and email (UNIQUE), how many alternate keys exist?**
- A) 0
- B) 1
- C) 2
- D) 3

**c) When should you use a composite primary key instead of a surrogate key?**
- A) Always - composite keys are more efficient
- B) When the combination naturally represents the business concept and simplicity isn't critical
- C) Never - surrogate keys are always better
- D) Only for tables with exactly two columns

**d) In a hotel booking system with composite key (hotel_id, room_number, check_in_date), which query uses the index most efficiently?**
- A) WHERE room_number = '305'
- B) WHERE check_in_date = '2024-06-15'
- C) WHERE hotel_id = 1
- D) WHERE check_in_date = '2024-06-15' AND room_number = '305'

**e) Which scenario is BEST suited for a composite UNIQUE constraint rather than a composite PRIMARY KEY?**
- A) Student enrollments (student_id, course_id, semester)
- B) When you want the benefits of both a surrogate key and natural uniqueness
- C) Product inventory (never appropriate)
- D) User login credentials

</details>

<details>
<summary><strong>Question 4: Schema Design</strong></summary>

You're designing a library system:
- Books can have multiple copies
- Each copy has a barcode and belongs to a specific branch
- Copies can be borrowed by patrons
- Same book can be at multiple branches

Design tables for:
1. **books** (ISBN, title, author)
2. **book_copies** (which copy at which branch)
3. **loans** (tracking who borrowed what)

For each table, identify:
- Primary key (composite or surrogate?)
- Any alternate keys
- Unique constraints
- Reasoning for your choices

</details>

<details>
<summary><strong>Question 5: Identify Keys</strong></summary>

Given this table:
```sql
CREATE TABLE flight_seats (
    seat_assignment_id BIGINT AUTO_INCREMENT,
    flight_id BIGINT NOT NULL,
    seat_number VARCHAR(5) NOT NULL,
    passenger_id BIGINT,
    boarding_zone INT,
    INDEX (flight_id, seat_number)
);
```

a) What's wrong with this schema from a key perspective?

b) List all potential candidate keys

c) Suggest the best primary key approach and explain why

d) Should any unique constraints be added?

</details>

---

### Answers to Section 2.3 Practice Questions

<details>
<summary><strong>Answer 1: Fill in the Blanks</strong></summary>

a) A UNIQUE constraint ensures all values in a column are **DISTINCT/UNIQUE**, but unlike a primary key, a table can have **MULTIPLE** unique constraints.

b) A **CANDIDATE** key is any column that could serve as a primary key, while an **ALTERNATE** key is a candidate key that was not chosen as the primary key.

c) Composite keys are made up of **TWO** or more columns that together **UNIQUELY** identify a row.

d) In most databases, UNIQUE constraints allow **ONE** NULL value(s), while primary keys allow **ZERO** NULL values.

e) Alternate keys are implemented in SQL using **UNIQUE** constraints (with NOT NULL).

</details>

<details>
<summary><strong>Answer 2: True or False</strong></summary>

a) **FALSE** - A table can have only ONE primary key (though it can be composite).

b) **TRUE** - Databases automatically create indexes on UNIQUE constraints for efficient lookup and enforcement.

c) **TRUE** - Candidate keys must be NOT NULL because they could potentially be chosen as the primary key.

d) **FALSE** - Each has trade-offs. Surrogate keys are simpler; composite keys can be more natural but more complex.

e) **TRUE** - Though uncommon, foreign keys can reference any UNIQUE NOT NULL column, not just primary keys.

f) **FALSE** - Individual columns in a composite key need NOT be unique. Only the combination must be unique.

g) **TRUE** - All alternate keys are candidate keys (they could have been PK). But the chosen PK is a candidate key that's NOT an alternate key.

</details>

<details>
<summary><strong>Answer 3: Multiple Choice</strong></summary>

**a) C - Must be a single column**
- Candidate keys can be composite (multiple columns)
- They must be unique, NOT NULL, and minimal, but can have multiple columns

**b) C - 2**
- Candidate keys: user_id, username, email (all unique + not null)
- Primary key: user_id (chosen)
- Alternate keys: username, email (2 total)

**c) B - When the combination naturally represents the business concept and simplicity isn't critical**
- Use composite PK for junction tables or when the combination IS the natural identifier
- Use surrogate when foreign key references would be complex or columns might change

**d) C - WHERE hotel_id = 1**
- Composite indexes work left-to-right: (hotel_id, room_number, check_in_date)
- Best performance when querying leftmost column(s)
- Partial match starting from the left uses the index efficiently

**e) B - When you want the benefits of both a surrogate key and natural uniqueness**
- Surrogate key as PK (simple, stable)
- Composite UNIQUE constraint for business rule enforcement
- Example: enrollment_id as PK, UNIQUE(student_id, course_id, semester)

</details>

<details>
<summary><strong>Answer 4: Schema Design</strong></summary>

```sql
-- 1. Books table
CREATE TABLE books (
    isbn VARCHAR(13) PRIMARY KEY,  -- Natural key (ISBN is globally unique, stable)
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255) NOT NULL,
    publisher VARCHAR(100),
    publish_year INT
);
```

**Reasoning:**
- **Primary key**: ISBN (natural key that never changes)
- **No alternate keys**: No other unique identifiers for books
- **Why ISBN as PK**: Globally recognized, stable, meaningful

---

```sql
-- 2. Book Copies table
CREATE TABLE book_copies (
    copy_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Surrogate key
    isbn VARCHAR(13) NOT NULL,                  -- Which book
    branch_id INT NOT NULL,                     -- Which branch
    barcode VARCHAR(20) UNIQUE NOT NULL,        -- Physical barcode (alternate key)
    condition VARCHAR(20) DEFAULT 'GOOD',
    acquisition_date DATE,
    UNIQUE KEY unique_copy (isbn, branch_id, barcode),  -- Belt-and-suspenders uniqueness
    FOREIGN KEY (isbn) REFERENCES books(isbn),
    FOREIGN KEY (branch_id) REFERENCES branches(branch_id)
);
```

**Reasoning:**
- **Primary key**: copy_id (surrogate) - easier for foreign keys in loans table
- **Alternate key**: barcode - each physical copy has unique barcode
- **Candidate keys**: 
  - copy_id (chosen as PK)
  - barcode (alternate key)
  - Potentially (isbn, branch_id, barcode) but barcode alone is sufficient
- **Why surrogate**: Loans table references copy_id (simple BIGINT) rather than composite

---

```sql
-- 3. Loans table
CREATE TABLE loans (
    loan_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Surrogate key
    copy_id BIGINT NOT NULL,                    -- Which copy
    patron_id BIGINT NOT NULL,                  -- Who borrowed
    checkout_date DATE NOT NULL,
    due_date DATE NOT NULL,
    return_date DATE NULL,
    UNIQUE KEY active_loan (copy_id, return_date),  -- Only one active loan per copy
    FOREIGN KEY (copy_id) REFERENCES book_copies(copy_id),
    FOREIGN KEY (patron_id) REFERENCES patrons(patron_id),
    CHECK (return_date IS NULL OR return_date >= checkout_date)
);
```

**Reasoning:**
- **Primary key**: loan_id (surrogate) - simple, stable
- **Unique constraint**: `(copy_id, return_date)` - prevents double-borrowing
  - Logic: If return_date IS NULL, copy is currently borrowed
  - Multiple NULLs allowed in UNIQUE constraint, so same copy can have many historical loans
  - But only ONE loan with return_date = NULL (active loan)

**Alternative Approach:**
```sql
-- Separate active loans table
CREATE TABLE active_loans (
    copy_id BIGINT PRIMARY KEY,  -- Copy can only have ONE active loan
    patron_id BIGINT NOT NULL,
    checkout_date DATE NOT NULL,
    due_date DATE NOT NULL,
    FOREIGN KEY (copy_id) REFERENCES book_copies(copy_id),
    FOREIGN KEY (patron_id) REFERENCES patrons(patron_id)
);

CREATE TABLE loan_history (
    loan_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    copy_id BIGINT NOT NULL,
    patron_id BIGINT NOT NULL,
    checkout_date DATE NOT NULL,
    due_date DATE NOT NULL,
    return_date DATE NOT NULL,
    FOREIGN KEY (copy_id) REFERENCES book_copies(copy_id),
    FOREIGN KEY (patron_id) REFERENCES patrons(patron_id)
);
```

</details>

<details>
<summary><strong>Answer 5: Identify Keys</strong></summary>

**a) What's wrong with this schema?**

1. **No primary key defined**: seat_assignment_id should be PRIMARY KEY
2. **No uniqueness constraint**: Can insert duplicate (flight_id, seat_number) combinations
3. **Allows double-booking**: Same seat can be assigned to multiple passengers
4. **Passenger_id allows NULL**: Can have unassigned seats, which might be fine, but needs thought

**b) Potential candidate keys:**

1. `seat_assignment_id` - Unique identifier (should be PK)
2. `(flight_id, seat_number)` - Each seat unique per flight (should be UNIQUE)
3. Possibly `passenger_id` if one passenger can only be on one flight at a time (unlikely)

**c) Best primary key approach:**

```sql
CREATE TABLE flight_seats (
    seat_assignment_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Surrogate PK
    flight_id BIGINT NOT NULL,
    seat_number VARCHAR(5) NOT NULL,
    passenger_id BIGINT NULL,  -- NULL until assigned
    boarding_zone INT,
    UNIQUE KEY unique_seat (flight_id, seat_number),  -- Composite alternate key
    INDEX idx_passenger (passenger_id),
    FOREIGN KEY (flight_id) REFERENCES flights(flight_id),
    FOREIGN KEY (passenger_id) REFERENCES passengers(passenger_id)
);
```

**Why this approach:**
- **Surrogate PK** (seat_assignment_id): Simple, stable, easy to reference
- **Composite UNIQUE** (flight_id, seat_number): Enforces business rule (no double-booking)
- **Passenger index**: Fast lookup of which seats a passenger has

**d) Unique constraints to add:**

YES - Add `UNIQUE KEY unique_seat (flight_id, seat_number)`

This prevents:
```sql
-- Without UNIQUE constraint, this would succeed (BAD!)
INSERT INTO flight_seats (flight_id, seat_number, passenger_id) 
VALUES (100, '12A', 501);

INSERT INTO flight_seats (flight_id, seat_number, passenger_id) 
VALUES (100, '12A', 502);  -- Same seat, different passenger!

-- With UNIQUE constraint, second insert fails (GOOD!)
-- ERROR: Duplicate entry for key 'unique_seat'
```

</details>