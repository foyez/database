# Database

A database is an organized collection of structured data stored electronically, designed for efficient access, management, and updates.

## Real-Life Analogy
Think of a **library**:
- **Database** = The entire library building
- **Tables** = Different sections (Fiction, Non-fiction, Reference)
- **Rows** = Individual books
- **Columns** = Book attributes (Title, Author, ISBN)
- **Schema** = The organization system (Dewey Decimal)
- **Query** = Asking the librarian to find books
- **Index** = The card catalog for quick lookups

## Why Do We Need Databases?

**Without Database:**
```
User Request → Server (stores everything in memory) → Response

❌ Server crashes = All data lost
❌ Multiple servers = Different data on each
❌ Can't handle millions of records efficiently
❌ No data persistence
```

**With Database:**
```
User Request → Stateless Server → Database → Response

✅ Data persists even if server crashes
✅ Multiple servers can share same database
✅ Optimized for large-scale data storage
✅ ACID guarantees for data integrity
```

## Core Database Concepts

### 1. Query
**Definition:** A command sent to the database to retrieve, insert, update, or delete data.

```sql
-- SQL Query Examples

-- Retrieve data (SELECT)
SELECT * FROM users WHERE age > 18;

-- Add data (INSERT)
INSERT INTO users (name, email, city) 
VALUES ('Foyez', 'foyez@example.com', 'Cumilla');

-- Modify data (UPDATE)
UPDATE users 
SET city = 'Dhaka' 
WHERE id = 1;

-- Remove data (DELETE)
DELETE FROM users 
WHERE inactive = true;

-- Aggregate data
SELECT city, COUNT(*) as user_count 
FROM users 
GROUP BY city 
ORDER BY user_count DESC;
```

### 2. Schema
**Definition:** The structure/blueprint that defines how data is organized.

**Real-Life Analogy:** If the database is an Excel spreadsheet, the schema is the column headers and data types.

```javascript
// Example: JSON Object (Data Instance)
{
  "name": "Foyez",
  "city": "Cumilla",
  "village": "Surikara",
  "age": 25,
  "email": "foyez@example.com"
}

// Schema (Structure/Rules)
{
  name: String (required, max 100 chars),
  city: String (required, max 50 chars),
  village: String (optional, max 50 chars),
  age: Integer (required, min 0, max 150),
  email: String (required, unique, valid email format)
}
```

**Schema in SQL:**
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  city VARCHAR(50) NOT NULL,
  village VARCHAR(50),
  age INTEGER CHECK (age >= 0 AND age <= 150),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Schema in MongoDB (NoSQL):**
```javascript
// MongoDB uses flexible schema (schema-less)
// But you can enforce with validation
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      required: ["name", "email", "city"],
      properties: {
        name: { type: "string", maxLength: 100 },
        email: { type: "string", pattern: "^.+@.+$" },
        city: { type: "string" },
        village: { type: "string" },
        age: { type: "integer", minimum: 0, maximum: 150 }
      }
    }
  }
});
```

---

## Step-by-Step: Building a Database System

### Real-World Project: E-Commerce Platform

Let's build a complete database from scratch for an online shopping website.

---

### Step 1: Requirements Analysis 📋

**Questions to Ask:**
1. What data needs to be stored?
2. What relationships exist between data entities?
3. How will data be queried?
4. What's the expected scale (users, transactions)?
5. What are consistency requirements?
6. What are performance requirements?

**Our E-Commerce Requirements:**
```
Business Requirements:
- Store user accounts with authentication
- Manage product catalog with categories
- Handle orders with multiple products
- Track inventory levels
- Process payments
- Store product reviews and ratings
- Maintain shopping cart

Technical Requirements:
- Expected: 1M users, 100K products
- Peak: 10K orders per day
- Payments must be atomic (ACID critical!)
- Product search must be fast
- Order history must be accurate
- 99.9% uptime required
```

---

### Step 2: Choose Database Type 🎯

#### Decision Matrix for E-Commerce

| Requirement | SQL (PostgreSQL) | NoSQL (MongoDB) | Winner |
|-------------|-----------------|-----------------|---------|
| **ACID Transactions** (payments) | ✅ Full support | ⚠️ Limited | **SQL** |
| **Complex Relationships** (users ↔ orders ↔ products) | ✅ JOINs, foreign keys | ⚠️ Manual references | **SQL** |
| **Data Consistency** | ✅ Strong | ⚠️ Eventual | **SQL** |
| **Fixed Schema** | ✅ Enforced | ⚠️ Flexible | **SQL** |
| **Complex Queries** | ✅ Advanced SQL | ⚠️ Aggregation pipeline | **SQL** |
| **Horizontal Scaling** | ⚠️ Complex | ✅ Built-in sharding | NoSQL |
| **Fast Simple Reads** | ✅ Good | ✅ Excellent | Tie |

**Decision for E-Commerce:**
- **Primary Database**: PostgreSQL (ACID, relationships, transactions)
- **Caching Layer**: Redis (shopping cart, sessions, trending products)
- **Search Engine**: Elasticsearch (product search, autocomplete)

#### Why PostgreSQL over MySQL?

| Feature | PostgreSQL | MySQL |
|---------|-----------|-------|
| **ACID Compliance** | ✅ Always ACID | ⚠️ Depends on storage engine (InnoDB yes, MyISAM no) |
| **Complex Queries** | ✅ CTEs, window functions, recursive queries | ⚠️ Basic queries |
| **JSON Support** | ✅ Native JSONB (binary, indexed) | ⚠️ Basic JSON (slower) |
| **Concurrency** | ✅ MVCC (Multi-Version Concurrency Control) | ⚠️ Table-level locking can block |
| **Data Types** | ✅ Arrays, custom types, ranges, hstore | ⚠️ Limited types |
| **Full-Text Search** | ✅ Built-in (tsvector) | ⚠️ Basic FULLTEXT |
| **Extensibility** | ✅ Extensions (PostGIS, pgcrypto) | ⚠️ Limited plugins |
| **Standards** | ✅ SQL standard compliant | ⚠️ Some non-standard syntax |
| **Performance** | ✅ Complex queries, writes | ✅ Simple reads, replication |
| **Open Source** | ✅ Truly open (PostgreSQL License) | ⚠️ Oracle-owned (licensing concerns) |
| **Learning Curve** | ⚠️ Steeper | ✅ Gentler |
| **Community** | ✅ Strong, growing | ✅ Larger, mature |

**Use PostgreSQL when:**
- Financial transactions (banking, e-commerce)
- Complex queries and reporting
- Data integrity is critical
- Need advanced features (JSON, arrays, GIS)
- Strong consistency required

**Use MySQL when:**
- Simple CRUD applications
- Read-heavy workloads
- Easier learning curve needed
- Existing ecosystem (WordPress, Drupal)
- Simpler deployment requirements

**Our Choice: PostgreSQL** because we need:
- ✅ Strong ACID for payments
- ✅ Complex queries for reports/analytics
- ✅ JSONB for flexible product attributes
- ✅ Advanced features for future growth

---

### Step 3: Design ER Diagram 🎨

**Entity-Relationship Diagram** visualizes how data entities relate.

#### Identify Entities and Attributes

**Entities (Tables):**
1. **Users** - Customer accounts
2. **Products** - Items for sale
3. **Categories** - Product groupings
4. **Orders** - Purchase records
5. **Order_Items** - Junction table (order ↔ products)
6. **Reviews** - Product reviews
7. **Cart_Items** - Shopping cart

#### Define Relationships

```
┌─────────────────┐
│     USERS       │
├─────────────────┤
│ id (PK)         │──────┐
│ name            │      │
│ email           │      │ One user has
│ password_hash   │      │ many orders
│ phone           │      │
│ address         │      │
│ created_at      │      │
└─────────────────┘      │
                         │
                         ▼
              ┌──────────────────┐
              │     ORDERS       │
              ├──────────────────┤
              │ id (PK)          │
              │ user_id (FK)     │◄────┐
              │ total            │     │
              │ status           │     │ One order has
              │ payment_status   │     │ many items
              │ shipping_address │     │
              │ created_at       │     │
              └──────────────────┘     │
                                       │
                                       ▼
                            ┌──────────────────┐
                            │  ORDER_ITEMS     │
                            ├──────────────────┤
                            │ id (PK)          │
                            │ order_id (FK)    │
                            │ product_id (FK)  │◄────┐
                            │ quantity         │     │
                            │ price_at_order   │     │
                            └──────────────────┘     │
                                                     │
┌─────────────────┐                                 │
│   CATEGORIES    │                                 │
├─────────────────┤                                 │
│ id (PK)         │──────┐                         │
│ name            │      │                         │
│ description     │      │ One category            │
└─────────────────┘      │ has many                │
                         │ products                │
                         ▼                         │
              ┌──────────────────┐                │
              │    PRODUCTS      │                │
              ├──────────────────┤                │
              │ id (PK)          │────────────────┘
              │ name             │
              │ description      │         One product has
              │ price            │         many reviews
              │ stock            │              │
              │ category_id (FK) │              │
              │ image_url        │              │
              │ is_active        │              │
              │ created_at       │              │
              └──────────────────┘              │
                                                │
                                                ▼
                                     ┌──────────────────┐
                                     │     REVIEWS      │
                                     ├──────────────────┤
                                     │ id (PK)          │
                                     │ product_id (FK)  │
                                     │ user_id (FK)     │
                                     │ rating (1-5)     │
                                     │ comment          │
                                     │ created_at       │
                                     └──────────────────┘

┌─────────────────┐
│   CART_ITEMS    │
├─────────────────┤
│ id (PK)         │
│ user_id (FK)    │
│ product_id (FK) │
│ quantity        │
│ created_at      │
└─────────────────┘
```

#### Relationship Types

**1. One-to-Many (1:N)**
- One user → Many orders
- One product → Many reviews
- One category → Many products

**2. Many-to-Many (M:N)**
- Many orders ↔ Many products
- Requires junction table: `order_items`

**3. One-to-One (1:1)**
- Less common, example: User ↔ UserProfile

#### Cardinality Notation

```
Crow's Foot Notation:
│        = One (exactly 1)
│<       = Many (0 or more)
│|       = One (exactly 1, mandatory)
○|       = Zero or one (optional)

Example:
User ──│< Orders
One user has many orders

Orders >│──│< Products
Many orders to many products (via order_items)
```

---

### Step 4: Write Database Schema 📝

#### Create Tables with Constraints

```sql
-- ============================================
-- Step 4.1: Create Users Table
-- ============================================
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  phone VARCHAR(20),
  address TEXT,
  city VARCHAR(50),
  postal_code VARCHAR(10),
  country VARCHAR(50) DEFAULT 'Bangladesh',
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for faster lookups
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_city ON users(city);
CREATE INDEX idx_users_active ON users(is_active) WHERE is_active = true;

-- ============================================
-- Step 4.2: Create Categories Table
-- ============================================
CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) UNIQUE NOT NULL,
  description TEXT,
  parent_id INTEGER REFERENCES categories(id), -- For subcategories
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_categories_parent ON categories(parent_id);

-- ============================================
-- Step 4.3: Create Products Table
-- ============================================
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(200) NOT NULL,
  slug VARCHAR(200) UNIQUE NOT NULL, -- URL-friendly name
  description TEXT,
  price DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
  cost_price DECIMAL(10, 2) CHECK (cost_price >= 0), -- For profit calculations
  stock INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
  category_id INTEGER REFERENCES categories(id) ON DELETE SET NULL,
  image_url VARCHAR(500),
  is_active BOOLEAN DEFAULT true,
  featured BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  -- Ensure meaningful data
  CONSTRAINT valid_price CHECK (price >= cost_price)
);

-- Indexes for common queries
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_active ON products(is_active) WHERE is_active = true;
CREATE INDEX idx_products_featured ON products(featured) WHERE featured = true;
CREATE INDEX idx_products_name ON products USING gin(to_tsvector('english', name));
CREATE INDEX idx_products_slug ON products(slug);

-- ============================================
-- Step 4.4: Create Orders Table
-- ============================================
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  
  -- Pricing
  subtotal DECIMAL(10, 2) NOT NULL CHECK (subtotal >= 0),
  tax DECIMAL(10, 2) NOT NULL DEFAULT 0 CHECK (tax >= 0),
  shipping_cost DECIMAL(10, 2) NOT NULL DEFAULT 0 CHECK (shipping_cost >= 0),
  discount DECIMAL(10, 2) NOT NULL DEFAULT 0 CHECK (discount >= 0),
  total DECIMAL(10, 2) NOT NULL CHECK (total >= 0),
  
  -- Status tracking
  status VARCHAR(20) NOT NULL DEFAULT 'pending',
  payment_status VARCHAR(20) NOT NULL DEFAULT 'pending',
  payment_method VARCHAR(50),
  
  -- Shipping info
  shipping_name VARCHAR(100) NOT NULL,
  shipping_address TEXT NOT NULL,
  shipping_city VARCHAR(50) NOT NULL,
  shipping_postal_code VARCHAR(10),
  shipping_country VARCHAR(50) NOT NULL,
  shipping_phone VARCHAR(20),
  
  -- Tracking
  tracking_number VARCHAR(100),
  shipped_at TIMESTAMP,
  delivered_at TIMESTAMP,
  
  -- Timestamps
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  -- Constraints
  CONSTRAINT valid_order_status 
    CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled', 'refunded')),
  CONSTRAINT valid_payment_status 
    CHECK (payment_status IN ('pending', 'completed', 'failed', 'refunded')),
  CONSTRAINT valid_total 
    CHECK (total = subtotal + tax + shipping_cost - discount)
);

-- Indexes
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- ============================================
-- Step 4.5: Create Order Items (Junction Table)
-- ============================================
CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id INTEGER NOT NULL REFERENCES products(id) ON DELETE RESTRICT,
  
  -- Snapshot product info at time of purchase
  product_name VARCHAR(200) NOT NULL,
  product_price DECIMAL(10, 2) NOT NULL CHECK (product_price >= 0),
  
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  subtotal DECIMAL(10, 2) NOT NULL CHECK (subtotal >= 0),
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  -- Constraints
  UNIQUE(order_id, product_id), -- Can't add same product twice
  CONSTRAINT valid_subtotal CHECK (subtotal = product_price * quantity)
);

CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_order_items_product ON order_items(product_id);

-- ============================================
-- Step 4.6: Create Reviews Table
-- ============================================
CREATE TABLE reviews (
  id SERIAL PRIMARY KEY,
  product_id INTEGER NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  order_id INTEGER REFERENCES orders(id) ON DELETE SET NULL, -- Verified purchase
  
  rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
  title VARCHAR(200),
  comment TEXT,
  
  is_verified BOOLEAN DEFAULT false, -- Verified purchase
  is_approved BOOLEAN DEFAULT false, -- Moderation
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  -- One review per user per product
  UNIQUE(product_id, user_id)
);

CREATE INDEX idx_reviews_product ON reviews(product_id);
CREATE INDEX idx_reviews_user ON reviews(user_id);
CREATE INDEX idx_reviews_rating ON reviews(rating);
CREATE INDEX idx_reviews_approved ON reviews(is_approved) WHERE is_approved = true;

-- ============================================
-- Step 4.7: Create Cart Items Table
-- ============================================
CREATE TABLE cart_items (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  product_id INTEGER NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  quantity INTEGER NOT NULL DEFAULT 1 CHECK (quantity > 0),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(user_id, product_id)
);

CREATE INDEX idx_cart_user ON cart_items(user_id);
CREATE INDEX idx_cart_product ON cart_items(product_id);
```

#### Understanding Key Constraints

```sql
-- PRIMARY KEY: Unique identifier, auto-incrementing
id SERIAL PRIMARY KEY
-- Ensures every row has unique ID

-- FOREIGN KEY: Links to another table
user_id INTEGER REFERENCES users(id)
-- Ensures order.user_id exists in users.id

-- UNIQUE: No duplicate values
email VARCHAR(255) UNIQUE
-- No two users can have same email

-- NOT NULL: Value required
name VARCHAR(100) NOT NULL
-- Must provide a name

-- CHECK: Custom validation
price DECIMAL(10, 2) CHECK (price >= 0)
-- Price cannot be negative

-- DEFAULT: Value if not provided
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
-- Auto-sets to current time

-- ON DELETE CASCADE: Delete related records
REFERENCES orders(id) ON DELETE CASCADE
-- If order deleted, delete order_items too

-- ON DELETE RESTRICT: Prevent deletion
REFERENCES products(id) ON DELETE RESTRICT
-- Can't delete product if in orders

-- ON DELETE SET NULL: Clear reference
REFERENCES categories(id) ON DELETE SET NULL
-- If category deleted, set product.category_id = NULL
```

---

### Step 5: Write Common Queries 🔍

#### Basic CRUD Operations

```sql
-- ============================================
-- CREATE: Insert new records
-- ============================================

-- Insert user
INSERT INTO users (name, email, password_hash, city)
VALUES ('Foyez Ahmed', 'foyez@example.com', '$2b$10$hashed...', 'Cumilla')
RETURNING id, name, email, created_at;

-- Insert product
INSERT INTO products (name, slug, description, price, stock, category_id)
VALUES (
  'Gaming Laptop',
  'gaming-laptop-rtx-4060',
  '15.6" FHD, Intel i7, 16GB RAM, RTX 4060',
  1299.99,
  50,
  1
)
RETURNING *;

-- Batch insert (multiple rows)
INSERT INTO products (name, slug, price, stock, category_id)
VALUES 
  ('Wireless Mouse', 'wireless-mouse', 29.99, 100, 2),
  ('Mechanical Keyboard', 'mechanical-keyboard', 89.99, 75, 2),
  ('USB-C Hub', 'usb-c-hub', 39.99, 200, 3);

-- ============================================
-- READ: Retrieve records
-- ============================================

-- Get user by email
SELECT id, name, email, city, created_at
FROM users
WHERE email = 'foyez@example.com';

-- Get active products with category
SELECT 
  p.id,
  p.name,
  p.price,
  p.stock,
  c.name as category_name
FROM products p
LEFT JOIN categories c ON p.category_id = c.id
WHERE p.is_active = true
ORDER BY p.created_at DESC
LIMIT 20;

-- Search products by name
SELECT id, name, price, stock
FROM products
WHERE to_tsvector('english', name) @@ to_tsquery('english', 'laptop & gaming')
  AND is_active = true;

-- ============================================
-- UPDATE: Modify records
-- ============================================

-- Update user profile
UPDATE users
SET 
  city = 'Dhaka',
  address = '123 Main Street',
  updated_at = CURRENT_TIMESTAMP
WHERE id = 1
RETURNING *;

-- Decrease product stock after order
UPDATE products
SET 
  stock = stock - 2,
  updated_at = CURRENT_TIMESTAMP
WHERE id = 101 AND stock >= 2
RETURNING id, name, stock;

-- Update order status
UPDATE orders
SET 
  status = 'shipped',
  tracking_number = 'TRACK123456',
  shipped_at = CURRENT_TIMESTAMP,
  updated_at = CURRENT_TIMESTAMP
WHERE id = 1
RETURNING *;

-- ============================================
-- DELETE: Remove records
-- ============================================

-- Delete cart item
DELETE FROM cart_items
WHERE user_id = 1 AND product_id = 101;

-- Soft delete (preferred for users/orders)
UPDATE users
SET is_active = false, updated_at = CURRENT_TIMESTAMP
WHERE id = 1;

-- Hard delete (cleanup old carts)
DELETE FROM cart_items
WHERE updated_at < NOW() - INTERVAL '30 days';
```

#### Complex Queries with JOINs

```sql
-- ============================================
-- Query 1: Get user's order history with products
-- ============================================
SELECT 
  o.id as order_id,
  o.created_at as order_date,
  o.status,
  o.total,
  p.name as product_name,
  oi.quantity,
  oi.product_price,
  (oi.quantity * oi.product_price) as item_total
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE o.user_id = 1
ORDER BY o.created_at DESC, p.name;

-- ============================================
-- Query 2: Get products with average ratings
-- ============================================
SELECT 
  p.id,
  p.name,
  p.price,
  p.stock,
  COALESCE(AVG(r.rating), 0) as avg_rating,
  COUNT(r.id) as review_count,
  c.name as category_name
FROM products p
LEFT JOIN reviews r ON p.id = r.product_id AND r.is_approved = true
LEFT JOIN categories c ON p.category_id = c.id
WHERE p.is_active = true
GROUP BY p.id, p.name, p.price, p.stock, c.name
HAVING AVG(r.rating) >= 4.0 OR AVG(r.rating) IS NULL
ORDER BY avg_rating DESC NULLS LAST, review_count DESC
LIMIT 10;

-- ============================================
-- Query 3: Top 10 best-selling products
-- ============================================
SELECT 
  p.id,
  p.name,
  p.price,
  p.stock,
  SUM(oi.quantity) as total_sold,
  SUM(oi.subtotal) as total_revenue,
  COUNT(DISTINCT o.id) as order_count
FROM products p
JOIN order_items oi ON p.id = oi.product_id
JOIN orders o ON oi.order_id = o.id
WHERE o.status IN ('delivered', 'shipped')
  AND o.created_at >= NOW() - INTERVAL '30 days'
GROUP BY p.id, p.name, p.price, p.stock
ORDER BY total_sold DESC
LIMIT 10;

-- ============================================
-- Query 4: User lifetime value and stats
-- ============================================
SELECT 
  u.id,
  u.name,
  u.email,
  COUNT(DISTINCT o.id) as total_orders,
  SUM(o.total) as lifetime_value,
  AVG(o.total) as average_order_value,
  MAX(o.created_at) as last_order_date,
  COUNT(DISTINCT r.id) as reviews_written
FROM users u
LEFT JOIN orders o ON u.id = o.user_id AND o.status != 'cancelled'
LEFT JOIN reviews r ON u.id = r.user_id
WHERE u.is_active = true
GROUP BY u.id, u.name, u.email
HAVING COUNT(DISTINCT o.id) > 0
ORDER BY lifetime_value DESC
LIMIT 100;

-- ============================================
-- Query 5: Low stock alert
-- ============================================
SELECT 
  p.id,
  p.name,
  p.stock,
  p.price,
  c.name as category,
  COUNT(DISTINCT oi.id) as orders_last_30_days
FROM products p
LEFT JOIN categories c ON p.category_id = c.id
LEFT JOIN order_items oi ON p.id = oi.product_id
LEFT JOIN orders o ON oi.order_id = o.id 
  AND o.created_at >= NOW() - INTERVAL '30 days'
  AND o.status != 'cancelled'
WHERE p.is_active = true
  AND p.stock < 10  -- Low stock threshold
GROUP BY p.id, p.name, p.stock, p.price, c.name
ORDER BY orders_last_30_days DESC, p.stock ASC;

-- ============================================
-- Query 6: Get user's shopping cart with totals
-- ============================================
SELECT 
  ci.id as cart_item_id,
  p.id as product_id,
  p.name as product_name,
  p.price,
  p.stock,
  p.image_url,
  ci.quantity,
  (p.price * ci.quantity) as item_total,
  CASE 
    WHEN p.stock >= ci.quantity THEN 'in_stock'
    WHEN p.stock > 0 THEN 'limited_stock'
    ELSE 'out_of_stock'
  END as availability
FROM cart_items ci
JOIN products p ON ci.product_id = p.id
WHERE ci.user_id = 1
  AND p.is_active = true
ORDER BY ci.created_at;

-- Cart totals
SELECT 
  COUNT(*) as item_count,
  SUM(ci.quantity) as total_quantity,
  SUM(p.price * ci.quantity) as subtotal
FROM cart_items ci
JOIN products p ON ci.product_id = p.id
WHERE ci.user_id = 1
  AND p.is_active = true;
```

#### Transaction Example (ACID in Action!)

```sql
-- ============================================
-- Transaction: Place an Order
-- This MUST be atomic - all or nothing!
-- ============================================

BEGIN TRANSACTION;

-- Step 1: Validate cart has items
SELECT COUNT(*) FROM cart_items WHERE user_id = 1;
-- If count = 0, ROLLBACK

-- Step 2: Check product availability
SELECT 
  ci.product_id,
  ci.quantity,
  p.stock
FROM cart_items ci
JOIN products p ON ci.product_id = p.id
WHERE ci.user_id = 1
  AND p.stock < ci.quantity;
-- If any rows returned, insufficient stock - ROLLBACK

-- Step 3: Calculate order totals
SELECT 
  SUM(p.price * ci.quantity) as subtotal
FROM cart_items ci
JOIN products p ON ci.product_id = p.id
WHERE ci.user_id = 1
INTO @subtotal;

SET @tax = @subtotal * 0.08;  -- 8% tax
SET @shipping = 10.00;         -- Flat shipping
SET @total = @subtotal + @tax + @shipping;

-- Step 4: Create order
INSERT INTO orders (
  user_id, subtotal, tax, shipping_cost, total,
  status, payment_status,
  shipping_name, shipping_address, shipping_city, shipping_country
)
VALUES (
  1, @subtotal, @tax, @shipping, @total,
  'pending', 'pending',
  'Foyez Ahmed', '123 Main St', 'Cumilla', 'Bangladesh'
)
RETURNING id INTO @order_id;

-- Step 5: Copy cart items to order items
INSERT INTO order_items (
  order_id, product_id, product_name, product_price, quantity, subtotal
)
SELECT 
  @order_id,
  p.id,
  p.name,
  p.price,
  ci.quantity,
  (p.price * ci.quantity)
FROM cart_items ci
JOIN products p ON ci.product_id = p.id
WHERE ci.user_id = 1;

-- Step 6: Update product stock (CRITICAL!)
UPDATE products p
SET 
  stock = stock - ci.quantity,
  updated_at = CURRENT_TIMESTAMP
FROM cart_items ci
WHERE p.id = ci.product_id
  AND ci.user_id = 1
  AND p.stock >= ci.quantity;  -- Double-check availability

-- Step 7: Clear user's cart
DELETE FROM cart_items WHERE user_id = 1;

-- If we made it here without errors, commit!
COMMIT;

-- If ANY error occurred, everything rolls back automatically
-- Money is never deducted without creating order
-- Stock is never decreased without order
-- Cart is never cleared without order
-- This is ATOMICITY in action!

EXCEPTION WHEN OTHERS THEN
  ROLLBACK;
  RAISE;
END;
```

---

## SQL vs NoSQL Deep Dive

### The Big Picture

```
SQL (Relational)              NoSQL (Non-Relational)
┌──────────────┐             ┌──────────────────────┐
│   Tables     │             │   Collections        │
│   with       │             │   or                 │
│   Fixed      │             │   Key-Value          │
│   Schema     │             │   or Graph           │
└──────────────┘             └──────────────────────┘
      │                               │
      │                               │
      ▼                               ▼
 Relationships                  Flexible Schema
 via JOINs                      Document-based
 ACID                           Eventually Consistent
```

### SQL Databases (Relational)

**Examples:** PostgreSQL, MySQL, Oracle, SQL Server, SQLite

#### Structure

```sql
-- Fixed schema: Tables with columns and types
users table:
+----+-------+-------------------+----------+
| id | name  | email             | city     |
+----+-------+-------------------+----------+
| 1  | Alice | alice@example.com | Dhaka    |
| 2  | Bob   | bob@example.com   | Cumilla  |
+----+-------+-------------------+----------+

orders table:
+----+---------+--------+----------+
| id | user_id | total  | status   |
+----+---------+--------+----------+
| 1  | 1       | 150.00 | shipped  |
| 2  | 1       |  75.50 | pending  |
| 3  | 2       | 200.00 | delivered|
+----+---------+--------+----------+

-- Relationships via foreign keys
-- orders.user_id → users.id
```

#### ✅ Strengths

**1. ACID Guarantees**
```sql
-- Perfect for financial transactions
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
-- Both happen or neither (atomicity)
```

**2. Complex Queries and JOINs**
```sql
-- Easy to combine data from multiple tables
SELECT 
  u.name,
  COUNT(o.id) as order_count,
  SUM(o.total) as total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name
HAVING SUM(o.total) > 500
ORDER BY total_spent DESC;
```

**3. Data Integrity**
```sql
-- Constraints enforce rules
CREATE TABLE products (
  price DECIMAL(10,2) CHECK (price > 0),  -- Can't be negative
  stock INTEGER CHECK (stock >= 0),        -- Can't be negative
  category_id INTEGER REFERENCES categories(id)  -- Must exist
);
```

**4. Mature Ecosystem**
- Decades of tools and documentation
- Skilled developers widely available
- Proven at scale (banks, airlines, governments)

#### ❌ Weaknesses

**1. Rigid Schema**
```sql
-- Adding a column requires migration
ALTER TABLE products ADD COLUMN weight DECIMAL(10,2);
-- Can be slow on large tables
-- Downtime might be required
```

**2. Vertical Scaling Challenges**
```
Single powerful server
- More expensive as you scale up
- Physical limits (CPU, RAM, disk)
- Single point of failure
```

**3. Performance at Massive Scale**
```sql
-- JOINs become slow with huge tables
SELECT * FROM orders o
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
JOIN users u ON o.user_id = u.id
-- Billions of rows = slow queries
```

#### When to Use SQL

✅ **Banking & Finance** - ACID transactions critical
✅ **E-commerce** - Complex relationships (users, orders, products)
✅ **ERP/CRM Systems** - Data integrity, complex reporting
✅ **Healthcare** - Regulatory compliance, data consistency
✅ **Any system** where data loss is unacceptable

---

### NoSQL Databases (Non-Relational)

NoSQL is not "one thing" - it's multiple types of databases.

#### Type 1: Document-Oriented (MongoDB, CouchDB)

**Structure:** JSON-like documents

```javascript
// MongoDB: Flexible nested documents
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  name: "Alice",
  email: "alice@example.com",
  city: "Dhaka",
  
  // Embed related data (no JOINs needed!)
  orders: [
    {
      id: 1,
      total: 150.00,
      status: "shipped",
      items: [
        { product_id: 101, name: "Laptop", quantity: 1, price: 1000 },
        { product_id: 102, name: "Mouse", quantity: 2, price: 25 }
      ]
    },
    {
      id: 2,
      total: 75.50,
      status: "pending",
      items: [...]
    }
  ],
  
  // Flexible schema - can have different fields
  preferences: {
    newsletter: true,
    notifications: false
  },
  
  // Easy to add new fields
  loyaltyPoints: 1250
}
```

**✅ Pros:**
- Flexible schema (add fields anytime)
- Fast reads (no JOINs, data embedded)
- Horizontal scaling (sharding built-in)
- Natural for web apps (JSON)
- Good for hierarchical data

**❌ Cons:**
- Data duplication (same product in many orders)
- No ACID across documents (limited transactions)
- Complex queries harder than SQL
- Can grow large (embedded data)

**When to use:**
- Content management systems
- User profiles with varying attributes
- Product catalogs (different products have different attributes)
- Rapid prototyping (schema changes frequently)

**MongoDB Example:**

```javascript
// Insert (flexible schema)
db.users.insertOne({
  name: "Foyez",
  email: "foyez@example.com",
  city: "Cumilla",
  // Can add any fields!
  favoriteColor: "blue",
  hobbies: ["coding", "reading"]
});

// Query
db.users.find({ 
  city: "Cumilla",
  "orders.total": { $gt: 100 }
});

// Update
db.users.updateOne(
  { email: "foyez@example.com" },
  { $set: { city: "Dhaka" } }
);

// Aggregation (like SQL GROUP BY)
db.orders.aggregate([
  { $match: { status: "delivered" } },
  { $group: { 
    _id: "$user_id", 
    total: { $sum: "$total" } 
  }},
  { $sort: { total: -1 } },
  { $limit: 10 }
]);
```

---

#### Type 2: Key-Value Store (Redis, DynamoDB)

**Structure:** Simple key → value mapping

```javascript
// Redis: Ultra-fast, in-memory
SET user:1:name "Alice"
SET user:1:email "alice@example.com"
SET session:abc123 "user_id:1|expires:1234567890"
SET cache:popular_products "[{id:1,name:'Laptop'},{id:2,name:'Mouse'}]"

GET user:1:name  // "Alice"
GET session:abc123  // "user_id:1|expires:1234567890"

// TTL (time-to-live) - auto-expiration
SETEX cache:products:trending 300 "[...]"  // Expires in 5 minutes

// Data structures
LPUSH cart:user:1 "product:101"  // List
SADD likes:post:1 "user:123"     // Set
HINCRBY stats:products:101 views 1  // Hash
```

**✅ Pros:**
- **Extremely fast** (in-memory, typically < 1ms)
- Simple API (GET, SET, DELETE)
- TTL support (auto-expiring data)
- Data structures (lists, sets, hashes)
- Pub/Sub messaging

**❌ Cons:**
- No complex queries (only get by key)
- No relationships (can't JOIN)
- Limited by RAM (memory-based)
- No persistent storage by default

**When to use:**
- **Caching** (most common use case)
- **Session storage** (user login sessions)
- **Real-time features** (leaderboards, counters)
- **Rate limiting** (API throttling)
- **Pub/Sub messaging** (chat, notifications)

---

#### Type 3: Column-Family (Cassandra, HBase)

**Structure:** Wide-column store

```
Row Key: user:1
  | Column: name          | Value: Alice              |
  | Column: email         | Value: alice@example.com  |
  | Column: order:1:total | Value: 150.00            |
  | Column: order:1:date  | Value: 2024-01-15        |
  | Column: order:2:total | Value: 75.50             |
  | Column: order:2:date  | Value: 2024-01-20        |
```

**✅ Pros:**
- **Massive scale** (petabytes of data)
- **High availability** (no single point of failure)
- **Fast writes** (optimized for write-heavy workloads)
- **Time-series data** (perfect for logs, IoT sensors)

**❌ Cons:**
- Complex to learn
- No JOINs
- Eventually consistent (not immediate)
- Difficult to change data model

**When to use:**
- IoT sensor data (billions of data points)
- Log aggregation (server logs, application logs)
- Time-series analytics
- Need 99.99% uptime

---

#### Type 4: Graph Database (Neo4j, ArangoDB)

**Structure:** Nodes and relationships

```cypher
// Neo4j: Social network
(Alice:User)-[:FOLLOWS]->(Bob:User)
(Alice)-[:LIKES]->(Post1:Post)
(Bob)-[:WROTE]->(Post1)
(Post1)-[:TAGGED]->(Technology:Tag)

// Find Alice's friends of friends
MATCH (alice:User {name: "Alice"})-[:FOLLOWS]->()-[:FOLLOWS]->(fof)
WHERE NOT (alice)-[:FOLLOWS]->(fof)
RETURN fof.name

// Recommendation: Products bought by similar users
MATCH (me:User)-[:PURCHASED]->(p:Product)<-[:PURCHASED]-(other:User)
MATCH (other)-[:PURCHASED]->(recommendation:Product)
WHERE NOT (me)-[:PURCHASED]->(recommendation)
RETURN recommendation, COUNT(*) as score
ORDER BY score DESC
LIMIT 10
```

**✅ Pros:**
- **Relationship queries** (social networks, recommendations)
- **Fast traversals** (find connections quickly)
- **Natural modeling** (follows how we think about relationships)

**❌ Cons:**
- Specialized use case
- Different query language (Cypher)
- Not for general-purpose data

**When to use:**
- Social networks (followers, friends)
- Recommendation engines
- Fraud detection (connection patterns)
- Knowledge graphs

---

### SQL vs NoSQL: Decision Matrix

| Factor | Choose SQL | Choose NoSQL |
|--------|-----------|--------------|
| **Data Structure** | Well-defined, structured | Flexible, evolving |
| **Relationships** | Complex (many JOINs) | Simple or embedded |
| **Transactions** | Critical (banking, e-commerce) | Not critical |
| **Consistency** | Must be immediate | Eventual consistency OK |
| **Scale** | < 1TB, vertical scaling OK | > 1TB, need horizontal scaling |
| **Queries** | Complex reporting, aggregations | Simple lookups by ID |
| **Development Speed** | Stable requirements | Rapid prototyping |
| **Team Expertise** | SQL knowledge available | NoSQL experience available |

### Real-World Hybrid Architecture

Most modern applications use **BOTH**!

```
┌─────────────────── Application ───────────────────┐
│                                                    │
│  ┌─────────────┐  ┌─────────────┐  ┌───────────┐ │
│  │ PostgreSQL  │  │    Redis    │  │  MongoDB  │ │
│  │             │  │             │  │           │ │
│  │ • Users     │  │ • Sessions  │  │ • Logs    │ │
│  │ • Orders    │  │ • Cart      │  │ • Events  │ │
│  │ • Products  │  │ • Cache     │  │ • Metrics │ │
│  │ • Payments  │  │ • Trending  │  │           │ │
│  │             │  │             │  │           │ │
│  │ ACID        │  │ Speed       │  │ Flexible  │ │
│  └─────────────┘  └─────────────┘  └───────────┘ │
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │          Elasticsearch                       │ │
│  │          • Product search                    │ │
│  │          • Autocomplete                      │ │
│  └──────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────┘
```

**Example: E-Commerce Platform**
- **PostgreSQL**: Orders, users, payments (need ACID)
- **Redis**: Shopping cart, sessions, product rankings
- **MongoDB**: User activity logs, product reviews
- **Elasticsearch**: Product search, filters, autocomplete

---

## ACID Properties

### Easy Memory Method: "**A Car Is Durable**"

- **A** - **A**tomicity (All or nothing)
- **C** - **C**onsistency (Rules always followed)
- **I** - **I**solation (No interference)
- **D** - **D**urability (Survives crashes)

---

### 1. Atomicity ⚛️
**"All or Nothing - No Half-Done Work"**

#### Real-Life Analogy
**ATM Withdrawal**:
1. Check balance ($500)
2. Deduct $100 from account
3. Dispense $100 cash

❌ **What if power goes out at step 2?**
- Money deducted but cash not dispensed
- You lose $100!

✅ **With Atomicity:**
- Either ALL steps complete, or NONE
- If power fails, transaction rolls back
- Your $500 stays intact

#### Code Example

```sql
-- ❌ WITHOUT Atomicity (DANGEROUS!)
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- ⚡ POWER FAILURE HERE!
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- Result: $100 disappeared into the void!

-- ✅ WITH Atomicity (SAFE!)
BEGIN TRANSACTION;
  
  -- Step 1: Deduct from sender
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  
  -- Step 2: Add to receiver
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
  
  -- ⚡ If power fails anywhere above, EVERYTHING rolls back!

COMMIT;
-- Only NOW is the transfer permanent
```

**E-Commerce Example:**

```sql
BEGIN TRANSACTION;

  -- 1. Create order
  INSERT INTO orders (user_id, total) VALUES (1, 150.00);
  
  -- 2. Add order items
  INSERT INTO order_items (order_id, product_id, quantity)
  VALUES (1, 101, 2);
  
  -- 3. Decrease stock
  UPDATE products SET stock = stock - 2 WHERE id = 101;
  
  -- 4. Clear cart
  DELETE FROM cart_items WHERE user_id = 1;
  
  -- If ANY step fails, EVERYTHING rolls back!

COMMIT;
```

**Key Points:**
- Transaction = smallest indivisible unit
- Either complete success or complete failure
- No partial updates
- Critical for financial systems

---

### 2. Consistency 🎯
**"Data Always Follows the Rules"**

#### Real-Life Analogy
**Chess Rules**: A pawn can't suddenly move like a queen. The game enforces rules. Similarly, database enforces: "balance can't be negative" or "email must be unique."

#### Code Example

```sql
-- Database enforces consistency through constraints

CREATE TABLE accounts (
  id SERIAL PRIMARY KEY,
  balance DECIMAL(10, 2) NOT NULL CHECK (balance >= 0),  -- Rule 1
  email VARCHAR(255) UNIQUE NOT NULL,                    -- Rule 2
  status VARCHAR(20) CHECK (status IN ('active', 'closed'))  -- Rule 3
);

-- ❌ This violates Rule 1 (negative balance)
UPDATE accounts SET balance = -50 WHERE id = 1;
-- ERROR: Check constraint "accounts_balance_check" violated

-- ❌ This violates Rule 2 (duplicate email)
INSERT INTO accounts (email) VALUES ('existing@email.com');
-- ERROR: Duplicate key value violates unique constraint

-- ❌ This violates Rule 3 (invalid status)
UPDATE accounts SET status = 'frozen' WHERE id = 1;
-- ERROR: Check constraint "accounts_status_check" violated

-- ✅ This follows all rules
UPDATE accounts SET balance = 100 WHERE id = 1;
-- SUCCESS
```

**Complex Consistency Example:**

```sql
-- Rule: Order total must match sum of items

CREATE OR REPLACE FUNCTION check_order_total()
RETURNS TRIGGER AS $$
DECLARE
  calculated_total DECIMAL(10, 2);
BEGIN
  -- Calculate actual total from order items
  SELECT SUM(quantity * price) INTO calculated_total
  FROM order_items
  WHERE order_id = NEW.id;
  
  -- Verify it matches the order total
  IF NEW.total != calculated_total THEN
    RAISE EXCEPTION 'Order total (%) does not match items total (%)',
      NEW.total, calculated_total;
  END IF;
  
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER validate_order_total
  BEFORE INSERT OR UPDATE ON orders
  FOR EACH ROW
  EXECUTE FUNCTION check_order_total();

-- ❌ This will fail (total doesn't match items)
INSERT INTO orders (id, user_id, total) VALUES (1, 1, 100.00);
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (1, 101, 2, 50.00);  -- Total should be 100.00
-- But if we update order total incorrectly:
UPDATE orders SET total = 150.00 WHERE id = 1;
-- ERROR: Order total (150) does not match items total (100)
```

**Key Points:**
- Database moves from one valid state to another
- Constraints enforce business rules
- Invalid data is rejected
- Maintains data integrity

---

### 3. Isolation 🔒
**"Transactions Don't Interfere with Each Other"**

#### Real-Life Analogy
**Movie Theater Seats**: Two people try to book seat A5 simultaneously. Only one succeeds. They don't see each other's half-booked seats or cause double-booking.

#### The Problem: Race Conditions

```sql
-- ❌ WITHOUT proper isolation:
-- User A and User B both try to buy the last item (stock = 1)

-- Time 1: User A checks stock
SELECT stock FROM products WHERE id = 101;
-- Returns: 1 (in stock!)

-- Time 2: User B checks stock (before A completes purchase)
SELECT stock FROM products WHERE id = 101;
-- Returns: 1 (in stock!) 🚨 UH OH!

-- Time 3: User A buys
UPDATE products SET stock = stock - 1 WHERE id = 101;
-- Stock = 0

-- Time 4: User B buys
UPDATE products SET stock = stock - 1 WHERE id = 101;
-- Stock = -1 🚨 OVERSOLD!
```

#### Solution: Proper Isolation

```sql
-- ✅ WITH proper isolation (FOR UPDATE lock)

-- User A's transaction
BEGIN TRANSACTION;
  
  -- Lock the row (other transactions must wait)
  SELECT stock FROM products WHERE id = 101 FOR UPDATE;
  -- Returns: 1
  
  -- Check if in stock
  IF stock >= 1 THEN
    UPDATE products SET stock = stock - 1 WHERE id = 101;
  ELSE
    RAISE EXCEPTION 'Out of stock';
  END IF;

COMMIT;
-- Now User B can proceed

-- User B's transaction
BEGIN TRANSACTION;
  
  SELECT stock FROM products WHERE id = 101 FOR UPDATE;
  -- Returns: 0 (User A already bought it)
  
  IF stock >= 1 THEN
    -- Won't execute because stock = 0
  ELSE
    RAISE EXCEPTION 'Out of stock';  -- ✅ Correct!
  END IF;

COMMIT;
```

#### Isolation Levels

| Level | Dirty Read | Non-repeatable Read | Phantom Read | Performance |
|-------|-----------|---------------------|--------------|-------------|
| **Read Uncommitted** | ❌ Possible | ❌ Possible | ❌ Possible | ⚡⚡⚡ Fastest |
| **Read Committed** (default) | ✅ Prevented | ❌ Possible | ❌ Possible | ⚡⚡ Fast |
| **Repeatable Read** | ✅ Prevented | ✅ Prevented | ❌ Possible | ⚡ Slower |
| **Serializable** | ✅ Prevented | ✅ Prevented | ✅ Prevented | 🐌 Slowest |

**Dirty Read Example:**
```sql
-- Transaction A
BEGIN;
UPDATE products SET price = 10 WHERE id = 1;
-- NOT committed yet!

-- Transaction B (if Read Uncommitted allowed)
SELECT price FROM products WHERE id = 1;
-- Sees price = 10 (uncommitted data!)

-- Transaction A
ROLLBACK;  -- Oops! B saw temporary data!
```

**Non-repeatable Read Example:**
```sql
-- Transaction A
BEGIN;
SELECT balance FROM accounts WHERE id = 1;
-- Returns: 100

-- Transaction B (commits between A's reads)
UPDATE accounts SET balance = 200 WHERE id = 1;
COMMIT;

-- Transaction A (same query, different result!)
SELECT balance FROM accounts WHERE id = 1;
-- Returns: 200 (changed!)
```

**Key Points:**
- Higher isolation = More consistent but slower
- Use SERIALIZABLE for financial transactions
- Use READ COMMITTED for general queries
- Locks prevent concurrent access

---

### 4. Durability 💾
**"Once Committed, Data Survives Forever"**

#### Real-Life Analogy
**Signing a Contract**: Once you sign with permanent marker, it can't be erased, even if the building burns down (because there are copies in safe locations).

#### Code Example

```sql
-- ✅ Committed transactions are permanent
BEGIN;
  INSERT INTO orders (user_id, total, status)
  VALUES (1, 100.00, 'pending');
COMMIT;
-- Data is now written to disk PERMANENTLY

-- ⚡ Even if server crashes RIGHT NOW, order is safe!

-- ❌ Uncommitted transactions are lost
BEGIN;
  INSERT INTO orders (user_id, total) VALUES (1, 100);
-- ⚡ Server crashes here - LOST! Never committed!
```

#### How Databases Ensure Durability

**1. Write-Ahead Logging (WAL)**
```
Transaction Process:
1. Write to WAL (log file) FIRST  ← Fast, sequential write
2. Mark as committed in WAL
3. Eventually update actual database files  ← Slower, random write
4. If crash occurs, replay WAL on restart

Example (PostgreSQL):
- Transaction: INSERT INTO users ...
- Step 1: Write to pg_wal/000001234  ← Immediate
- Step 2: COMMIT recorded in WAL
- Step 3: Update actual data files  ← Background process
- If crash: Replay WAL entries to recover
```

**2. Replication (Multiple Copies)**
```
Primary Server (Dhaka) ─┬→ Replica 1 (Chittagong)
                        ├→ Replica 2 (Sylhet)
                        └→ Replica 3 (Khulna)

If Dhaka datacenter destroyed, data still safe!
```

**3. Backups**
```
Backup Strategy:
- Full backup: Daily at 2 AM
- Incremental: Every 6 hours
- Transaction logs: Continuous
- Retention: 30 days daily, 12 months monthly
- Location: Different geographic regions
```

**Configuration Example (PostgreSQL):**

```sql
-- postgresql.conf

-- WAL settings
wal_level = replica
fsync = on  -- Force writes to disk (slower but durable)
synchronous_commit = on  -- Wait for WAL before returning

-- Without these, committed transactions might be lost!
```

**Key Points:**
- Committed data survives power failure, crash, or disaster
- Achieved through WAL, replication, backups
- Trade-off: Waiting for disk writes makes COMMIT slower
- Essential for databases (would you trust a bank that loses transactions?)

---

### ACID Summary

| Property | Question | Real-Life Example | Database Example |
|----------|----------|------------------|------------------|
| **Atomicity** | All or nothing? | ATM: Deduct money AND dispense cash | Bank transfer: Both accounts updated or neither |
| **Consistency** | Rules followed? | Chess: Pieces move by rules | Balance can't be negative |
| **Isolation** | No interference? | Theater: Two people can't book same seat | Two users can't buy last item twice |
| **Durability** | Survives crashes? | Signed contract survives fire | Committed order survives power failure |

**ACID Trade-offs:**
- ✅ Safety: Data integrity guaranteed
- ❌ Speed: Slower than "eventual consistency"
- ✅ Simplicity: Easier to reason about
- ❌ Scalability: Harder to scale horizontally

**When ACID is Critical:**
- Banking and finance
- E-commerce payments
- Healthcare records
- Anything involving money or legal data

**When Eventual Consistency is OK:**
- Social media likes/follows
- Product view counts
- Analytics and logs
- Recommendation systems

---

## Database Selection Guide

### Step-by-Step Selection Process

#### Step 1: Analyze Requirements

Ask these questions:

**1. Data Structure**
- Fixed schema or evolving?
- Simple or complex relationships?
- Hierarchical or flat?

**2. Scale**
- How much data? (MB, GB, TB, PB)
- How many users? (hundreds, thousands, millions)
- Growth rate? (stable, doubling yearly)

**3. Operations**
- Read-heavy or write-heavy?
- Reads per second?
- Writes per second?

**4. Consistency**
- Must be immediate (strong) or eventual OK?
- ACID transactions required?

**5. Queries**
- Simple lookups by ID?
- Complex JOINs and aggregations?
- Full-text search?

**6. Budget & Team**
- Cloud or self-hosted?
- Team expertise?
- Operational complexity acceptable?

---

#### Step 2: Decision Tree

```
START: Choose Your Database
│
├─ Need ACID transactions? (banking, payments)
│  ├─ YES → SQL
│  │   ├─ Complex queries → PostgreSQL
│  │   └─ Simple queries → MySQL
│  └─ NO → Continue
│
├─ Data structure changing frequently?
│  ├─ YES → NoSQL (MongoDB)
│  └─ NO → Continue
│
├─ Need to scale to millions of users?
│  ├─ YES → NoSQL
│  │   ├─ Document storage → MongoDB
│  │   ├─ Time-series/logs → Cassandra
│  │   └─ Caching → Redis
│  └─ NO → SQL (PostgreSQL)
│
├─ Complex relationships and JOINs?
│  ├─ YES → PostgreSQL
│  └─ NO → Continue
│
├─ Need caching or sessions?
│  ├─ YES → Redis
│  └─ NO → Continue
│
├─ Graph relationships? (social network)
│  ├─ YES → Neo4j
│  └─ NO → Continue
│
└─ Not sure?
   └─ Start with PostgreSQL
      (Can always add NoSQL later)
```

---

#### Step 3: Common Use Cases

| Use Case | Best Database(s) | Why |
|----------|-----------------|-----|
| **E-commerce** | PostgreSQL + Redis | ACID (orders/payments) + Fast cache (cart) |
| **Social Media** | PostgreSQL + Neo4j + Redis | Users (SQL) + Relationships (Graph) + Sessions (Redis) |
| **Blog/CMS** | MongoDB or PostgreSQL | Flexible content (Mongo) OR Structured (Postgres) |
| **Analytics** | Cassandra or ClickHouse | Write-heavy, time-series data |
| **Real-time Gaming** | Redis + Cassandra | Low latency (Redis) + Event storage (Cassandra) |
| **IoT Sensors** | TimescaleDB or InfluxDB | Optimized for time-series |
| **Search Engine** | Elasticsearch | Full-text search, autocomplete |
| **Banking** | PostgreSQL or Oracle | ACID, strong consistency, proven reliability |

---

### Why PostgreSQL vs MySQL vs MongoDB?

#### PostgreSQL vs MySQL

**Choose PostgreSQL when:**
```
✅ Complex applications (e-commerce, CRM, ERP)
✅ Need advanced features (JSON, arrays, full-text search)
✅ Strong consistency critical (financial data)
✅ Complex queries and analytics
✅ Future-proof (active development)
```

**Choose MySQL when:**
```
✅ Simple CRUD operations
✅ Read-heavy workloads
✅ Easier learning curve
✅ Existing ecosystem (WordPress, etc.)
✅ Fast replication setup
```

#### PostgreSQL vs MongoDB

**Choose PostgreSQL when:**
```
✅ Financial transactions (banking, payments)
✅ Complex relationships (orders ↔ users ↔ products)
✅ ACID transactions required
✅ Team knows SQL
✅ Data integrity critical
```

**Choose MongoDB when:**
```
✅ Rapid prototyping (schema changes frequently)
✅ Horizontal scaling required
✅ Document-oriented data (catalogs, CMS)
✅ Flexible schema (each record different)
✅ High-volume simple queries
```

---

### Decision Matrix

| Factor | PostgreSQL | MySQL | MongoDB | Redis |
|--------|-----------|-------|---------|-------|
| **ACID** | ✅ Full | ✅ Full (InnoDB) | ⚠️ Limited | ❌ No |
| **Schema** | 🔒 Fixed | 🔒 Fixed | 🔓 Flexible | 🔓 Flexible |
| **Scaling** | ⬆️ Vertical | ⬆️ Vertical | ➡️ Horizontal | ➡️ Horizontal |
| **Complex Queries** | ✅ Excellent | ✅ Good | ⚠️ Limited | ❌ No |
| **Speed** | ⚡⚡ Fast | ⚡⚡⚡ Faster reads | ⚡⚡ Fast | ⚡⚡⚡ Fastest |
| **Learning Curve** | ⚠️ Steep | ✅ Gentle | ✅ Moderate | ✅ Easy |
| **Use Case** | General purpose | Web apps | CMS, catalogs | Caching |

---

### Real-World Architecture Example

**E-Commerce Platform:**

```
┌────────────── Application ─────────────┐
│                                        │
│  ┌──────────────┐  ┌──────────────┐    │
│  │ PostgreSQL   │  │    Redis     │    │
│  │              │  │              │    │
│  │ • Users      │  │ • Sessions   │    │
│  │ • Orders     │  │ • Cart       │    │
│  │ • Payments   │  │ • Cache      │    │
│  │ • Products   │  │ • Trending   │    │
│  │              │  │              │    │
│  │ Strong       │  │ Fast         │    │
│  │ Consistency  │  │ Temporary    │    │
│  └──────────────┘  └──────────────┘    │
│                                        │
│  ┌──────────────────────────────────┐  │
│  │      Elasticsearch               │  │
│  │      • Product search            │  │
│  │      • Filters & facets          │  │
│  │      • Autocomplete              │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

**Why this architecture:**
- **PostgreSQL**: Orders and payments need ACID
- **Redis**: Shopping cart is temporary, needs speed
- **Elasticsearch**: Product search needs full-text capabilities

---

## Indexing Strategies

### What is an Index?

**Real-Life Analogy:** A book's index at the back. Instead of reading all 500 pages to find "PostgreSQL", you look it up in the index and jump to page 147.

### Without vs With Index

```sql
-- ❌ WITHOUT Index: Sequential Scan
SELECT * FROM users WHERE email = 'foyez@example.com';
-- Database scans ALL 1,000,000 rows one by one
-- Time: O(n) = SLOW! (could take seconds)

-- ✅ WITH Index: Index Scan
CREATE INDEX idx_users_email ON users(email);

SELECT * FROM users WHERE email = 'foyez@example.com';
-- Database uses index to jump directly to the row
-- Time: O(log n) = FAST! (milliseconds)
```

**Performance Comparison:**
```
1,000,000 rows:
- Sequential scan: ~2000ms
- Index scan: ~50ms
- 40x faster!
```

---

### Types of Indexes

#### 1. B-Tree Index (Default, Most Common)

**Best for:** Equality and range queries, sorting

```sql
-- Create B-Tree index
CREATE INDEX idx_products_price ON products(price);

-- Queries that use this index:
SELECT * FROM products WHERE price = 99.99;       -- Exact match
SELECT * FROM products WHERE price > 50;          -- Range
SELECT * FROM products WHERE price BETWEEN 50 AND 150;
SELECT * FROM products ORDER BY price;            -- Sorting
SELECT MIN(price), MAX(price) FROM products;      -- Min/Max
```

**Structure (Simplified):**
```
        [50]
       /    \
    [25]    [75]
    /  \    /  \
 [10][40][60][90]
```

**Characteristics:**
- Balanced tree structure
- O(log n) lookup time
- Works for <, <=, =, >=, >
- Default index type in most databases

---

#### 2. Hash Index

**Best for:** Exact equality matches only

```sql
-- Create hash index
CREATE INDEX idx_users_email_hash ON users USING HASH (email);

-- ✅ Uses index (exact match)
SELECT * FROM users WHERE email = 'foyez@example.com';

-- ❌ Does NOT use index (pattern matching)
SELECT * FROM users WHERE email LIKE '%example.com';

-- ❌ Does NOT use index (range)
SELECT * FROM users WHERE email > 'a@example.com';
```

**Characteristics:**
- O(1) lookup for exact matches
- Doesn't support range queries
- Doesn't support sorting
- Faster than B-Tree for equality, but limited use cases

---

#### 3. GIN (Generalized Inverted Index)

**Best for:** Arrays, JSON, full-text search

```sql
-- For array columns
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(200),
  tags VARCHAR[] -- Array of tags
);

CREATE INDEX idx_products_tags ON products USING GIN (tags);

-- Query: Find products with specific tags
SELECT * FROM products WHERE tags @> ARRAY['electronics', 'laptop'];
-- Very fast with GIN index!

-- For JSON columns
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  preferences JSONB
);

CREATE INDEX idx_users_preferences ON users USING GIN (preferences);

-- Query: Find users with specific JSON property
SELECT * FROM users WHERE preferences @> '{"notifications": true}';

-- For full-text search
ALTER TABLE products ADD COLUMN search_vector tsvector;

CREATE INDEX idx_products_search ON products USING GIN (search_vector);

UPDATE products 
SET search_vector = to_tsvector('english', name || ' ' || description);

-- Query: Full-text search
SELECT * FROM products 
WHERE search_vector @@ to_tsquery('english', 'gaming & laptop');
```

**Characteristics:**
- Perfect for contains operators (@>, @@)
- Larger index size
- Slower inserts/updates
- Essential for JSON, arrays, full-text

---

#### 4. Partial Index

**Best for:** Indexing subset of data

```sql
-- Only index active products
CREATE INDEX idx_active_products ON products(name) 
WHERE is_active = true;

-- Saves space! If 90% products are inactive, index is 10x smaller

-- ✅ Uses index
SELECT * FROM products WHERE name = 'Laptop' AND is_active = true;

-- ❌ Does NOT use index (is_active = false not in index)
SELECT * FROM products WHERE name = 'Laptop' AND is_active = false;

-- More examples
CREATE INDEX idx_high_value_orders ON orders(user_id, created_at)
WHERE total > 1000;  -- Only index large orders

CREATE INDEX idx_recent_posts ON posts(created_at)
WHERE created_at > NOW() - INTERVAL '30 days';  -- Only recent posts
```

**Benefits:**
- Smaller index size
- Faster index maintenance
- Targets specific query patterns

---

#### 5. Composite Index (Multiple Columns)

**Best for:** Queries filtering on multiple columns

```sql
-- Create composite index
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- ✅ Uses index (both columns, in order)
SELECT * FROM orders WHERE user_id = 1 AND status = 'pending';

-- ✅ Uses index (first column only)
SELECT * FROM orders WHERE user_id = 1;

-- ❌ Does NOT use index efficiently (second column without first)
SELECT * FROM orders WHERE status = 'pending';
-- Will do full scan or index-only scan (slow)

-- ⚠️ Column order matters!
-- Better index for status-only queries:
CREATE INDEX idx_orders_status_user ON orders(status, user_id);
```

**Column Order Rule:**
```sql
-- Rule: Most selective column FIRST

-- ❌ BAD: status has only 5 values (pending, processing, shipped, delivered, cancelled)
CREATE INDEX idx_orders_status_user ON orders(status, user_id);

-- ✅ GOOD: user_id has millions of unique values (more selective)
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- How to check selectivity:
SELECT 
  COUNT(DISTINCT user_id) as user_cardinality,  -- High = selective
  COUNT(DISTINCT status) as status_cardinality   -- Low = not selective
FROM orders;
-- Results: 1,000,000 vs 5 → user_id is more selective
```

---

#### 6. Covering Index

**Definition:** Index contains ALL columns needed by query

```sql
-- Query needs: email, name, city
SELECT name, city FROM users WHERE email = 'foyez@example.com';

-- ❌ Regular index: Index lookup + table lookup (2 steps)
CREATE INDEX idx_users_email ON users(email);

-- ✅ Covering index: All data in index (1 step!)
CREATE INDEX idx_users_email_covering ON users(email) 
INCLUDE (name, city);

-- Or in PostgreSQL:
CREATE INDEX idx_users_email_covering ON users(email, name, city);
```

**Benefits:**
- Faster queries (no table lookup)
- Especially good for read-heavy workloads

**Trade-offs:**
- Larger index size
- Slower writes (more to update)

---

### When to Create Indexes

#### ✅ CREATE Index When:

```sql
-- 1. Column used in WHERE clause frequently
SELECT * FROM products WHERE category_id = 1;
-- → CREATE INDEX idx_products_category ON products(category_id);

-- 2. Column used in JOIN conditions
SELECT * FROM orders o
JOIN users u ON o.user_id = u.id;
-- → CREATE INDEX idx_orders_user_id ON orders(user_id);

-- 3. Column used in ORDER BY frequently
SELECT * FROM products ORDER BY created_at DESC LIMIT 20;
-- → CREATE INDEX idx_products_created_at ON products(created_at DESC);

-- 4. High cardinality column (many unique values)
SELECT * FROM users WHERE email = '...';
-- → CREATE INDEX idx_users_email ON users(email);

-- 5. Large table (1000+ rows)
-- Small tables don't benefit from indexes
```

#### ❌ DON'T CREATE Index When:

```sql
-- 1. Small table (< 1000 rows)
-- Sequential scan faster than index lookup overhead

-- 2. Low cardinality column (few unique values)
-- Example: gender (2 values), status (5 values)
SELECT * FROM users WHERE gender = 'M';
-- Sequential scan of half the table is faster

-- 3. Column rarely used in queries
-- Index maintenance cost outweighs benefits

-- 4. Write-heavy table
-- Every INSERT/UPDATE/DELETE must update indexes
-- Can significantly slow down writes

-- 5. Column with many NULLs
-- Index efficiency decreases
```

---

### Index Maintenance

```sql
-- Check index usage
SELECT 
  schemaname,
  tablename,
  indexname,
  idx_scan,  -- How many times index used
  idx_tup_read,
  idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC;
-- idx_scan = 0 means index never used → DROP it!

-- Drop unused index
DROP INDEX idx_users_unused_column;

-- Rebuild index (fix fragmentation)
REINDEX INDEX idx_users_email;

-- Analyze table (update statistics for query planner)
ANALYZE users;

-- Check index size
SELECT 
  indexname,
  pg_size_pretty(pg_relation_size(indexrelid)) as size
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC;
```

---

### Indexing Strategy for E-Commerce

```sql
-- Users table
CREATE INDEX idx_users_email ON users(email);  -- Login
CREATE INDEX idx_users_city ON users(city);    -- Filtering

-- Products table
CREATE INDEX idx_products_category ON products(category_id);  -- Category pages
CREATE INDEX idx_products_price ON products(price);           -- Price sorting
CREATE INDEX idx_products_active ON products(is_active) WHERE is_active = true;
CREATE INDEX idx_products_name_gin ON products USING gin(to_tsvector('english', name));  -- Search

-- Orders table
CREATE INDEX idx_orders_user_id ON orders(user_id);  -- User's orders
CREATE INDEX idx_orders_status ON orders(status);    -- Admin panel
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);  -- Recent orders
CREATE INDEX idx_orders_user_status ON orders(user_id, status);  -- User's pending orders

-- Order items
CREATE INDEX idx_order_items_order ON order_items(order_id);    -- Order details
CREATE INDEX idx_order_items_product ON order_items(product_id); -- Product sales stats

-- Reviews
CREATE INDEX idx_reviews_product ON reviews(product_id);  -- Product reviews
CREATE INDEX idx_reviews_approved ON reviews(is_approved) WHERE is_approved = true;
```

---

## Query Optimization

### Use EXPLAIN ANALYZE

**The #1 tool for optimization**

```sql
-- Show query execution plan
EXPLAIN 
SELECT * FROM orders WHERE user_id = 1 AND status = 'pending';

-- Show ACTUAL execution time and row counts
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 1 AND status = 'pending';
```

**Reading EXPLAIN output:**

```
Seq Scan on orders  (cost=0.00..180.00 rows=10 width=100) (actual time=0.031..2.456 rows=5 loops=1)
  Filter: ((user_id = 1) AND ((status)::text = 'pending'::text))
  Rows Removed by Filter: 995
Planning Time: 0.123 ms
Execution Time: 2.489 ms

❌ Problem: Sequential Scan = slow!
✅ Solution: Create index on (user_id, status)
```

**After adding index:**

```
Index Scan using idx_orders_user_status on orders  (cost=0.29..8.31 rows=5 width=100) (actual time=0.021..0.034 rows=5 loops=1)
  Index Cond: ((user_id = 1) AND ((status)::text = 'pending'::text))
Planning Time: 0.089 ms
Execution Time: 0.056 ms

✅ 44x faster! (2.489ms → 0.056ms)
```

---

### Query Optimization Techniques

#### 1. SELECT Only Needed Columns

```sql
-- ❌ BAD: Retrieves all columns (wastes bandwidth and memory)
SELECT * FROM products;

-- ✅ GOOD: Only needed columns
SELECT id, name, price FROM products;

-- Real impact:
-- Table: 10 columns, 1MB per row
-- SELECT *: 1000 rows = 1GB transferred
-- SELECT id, name, price: 1000 rows = 100MB transferred
-- 10x less data!
```

---

#### 2. Use LIMIT for Pagination

```sql
-- ❌ BAD: Retrieves all rows, paginate in application
SELECT * FROM products ORDER BY created_at DESC;
-- Retrieves 100,000 products, application shows 20

-- ✅ GOOD: Database only returns needed rows
SELECT * FROM products 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 0;  -- Page 1

SELECT * FROM products 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 20;  -- Page 2

-- Even better: Keyset pagination (faster for large offsets)
SELECT * FROM products 
WHERE created_at < '2024-01-15 10:30:00'
ORDER BY created_at DESC 
LIMIT 20;
```

---

#### 3. Avoid SELECT DISTINCT

```sql
-- ❌ SLOW: DISTINCT is expensive
SELECT DISTINCT city FROM users;
-- Sorts entire result set to find uniques

-- ✅ FASTER: GROUP BY (usually faster)
SELECT city FROM users GROUP BY city;

-- ✅ BEST: Fix data model to avoid duplicates
```

---

#### 4. Use EXISTS Instead of IN

```sql
-- ❌ SLOW: IN with large subquery
SELECT * FROM products
WHERE id IN (SELECT product_id FROM order_items);
-- Subquery returns all IDs, then does lookup

-- ✅ FAST: EXISTS (stops at first match)
SELECT * FROM products p
WHERE EXISTS (
  SELECT 1 FROM order_items oi 
  WHERE oi.product_id = p.id
);
-- Stops checking once found
```

---

#### 5. Avoid Functions on Indexed Columns

```sql
-- ❌ BAD: Can't use index
SELECT * FROM users WHERE LOWER(email) = 'foyez@example.com';
-- Function on column prevents index usage

-- ✅ GOOD: Query data as stored
SELECT * FROM users WHERE email = 'foyez@example.com';

-- ✅ Alternative: Functional index
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
-- Now function queries can use index
```

---

#### 6. Use JOIN Instead of Subqueries

```sql
-- ❌ SLOW: Correlated subquery
SELECT 
  u.name,
  (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) as order_count
FROM users u;
-- Subquery runs for EACH user (N queries)

-- ✅ FAST: JOIN with GROUP BY
SELECT 
  u.name,
  COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;
-- Single query with JOIN
```

---

#### 7. Batch Operations

```sql
-- ❌ BAD: Multiple queries (N round trips)
INSERT INTO products (name, price) VALUES ('Product 1', 10);
INSERT INTO products (name, price) VALUES ('Product 2', 20);
INSERT INTO products (name, price) VALUES ('Product 3', 30);
-- 3 round trips to database

-- ✅ GOOD: Single batch query (1 round trip)
INSERT INTO products (name, price) VALUES 
  ('Product 1', 10),
  ('Product 2', 20),
  ('Product 3', 30);
-- 1 round trip to database

-- Even better: Use COPY for bulk inserts
COPY products(name, price) FROM '/path/to/products.csv' WITH CSV;
-- Can be 100x faster for large datasets
```

---

#### 8. Use WHERE Before JOIN (Let Optimizer Handle It)

```sql
-- Modern query optimizers handle this, but be aware:

-- Conceptually slower (JOINs all, then filters)
SELECT u.name, o.total
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.city = 'Dhaka';

-- Conceptually faster (filters first, then JOINs)
SELECT u.name, o.total
FROM (SELECT * FROM users WHERE city = 'Dhaka') u
JOIN orders o ON u.id = o.user_id;

-- But in practice, modern optimizers do the same thing!
-- Use EXPLAIN to verify
```

---

### Real-World Optimization Example

**Before Optimization:**

```sql
SELECT 
  p.*,
  (SELECT AVG(rating) FROM reviews WHERE product_id = p.id) as avg_rating,
  (SELECT COUNT(*) FROM reviews WHERE product_id = p.id) as review_count
FROM products p
WHERE p.category_id = 1
ORDER BY (SELECT AVG(rating) FROM reviews WHERE product_id = p.id) DESC;

-- Execution Time: 5234ms (5.2 seconds!)
-- Problems:
-- 1. Correlated subqueries (run for each product)
-- 2. No indexes
-- 3. Sorting by subquery result
```

**After Optimization:**

```sql
-- Step 1: Create indexes
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_reviews_product ON reviews(product_id);

-- Step 2: Rewrite with JOIN
SELECT 
  p.*,
  COALESCE(AVG(r.rating), 0) as avg_rating,
  COUNT(r.id) as review_count
FROM products p
LEFT JOIN reviews r ON p.id = r.product_id
WHERE p.category_id = 1
GROUP BY p.id
ORDER BY AVG(r.rating) DESC NULLS LAST;

-- Execution Time: 47ms
-- 111x faster! (5234ms → 47ms)
```

---

## Caching Strategies

### Why Cache?

```
Database Query: 50ms
Cache Hit: 1ms
50x faster!

+ Reduces database load
+ Handles traffic spikes
+ Improves user experience
```

---

### 1. Application-Level Caching (Redis)

```typescript
import Redis from 'ioredis';

const redis = new Redis();

// Without caching
async function getProduct(id: number) {
  const product = await db.query('SELECT * FROM products WHERE id = ?', [id]);
  return product;
}
// Every call hits database

// With caching
async function getProductCached(id: number) {
  const cacheKey = `product:${id}`;
  
  // 1. Try cache first
  let product = await redis.get(cacheKey);
  
  if (product) {
    console.log('Cache HIT!');
    return JSON.parse(product);  // Fast! ~1ms
  }
  
  console.log('Cache MISS');
  
  // 2. Cache miss - query database
  product = await db.query('SELECT * FROM products WHERE id = ?', [id]);
  
  // 3. Store in cache for 5 minutes
  await redis.setex(cacheKey, 300, JSON.stringify(product));
  
  return product;
}

// First call: Cache MISS (50ms)
// Subsequent calls: Cache HIT (1ms)
// After 5 minutes: Cache expires, MISS again
```

---

### 2. Cache Invalidation

**"There are only two hard things in Computer Science: cache invalidation and naming things." - Phil Karlton**

```typescript
// Strategy 1: Time-based (TTL)
await redis.setex('products:trending', 300, data);  // Expires in 5 minutes

// Strategy 2: Event-based (invalidate on change)
async function updateProduct(id: number, data: any) {
  // Update database
  await db.query('UPDATE products SET ... WHERE id = ?', [id, data]);
  
  // Invalidate cache
  await redis.del(`product:${id}`);
  await redis.del('products:list');
  await redis.del('products:trending');
}

// Strategy 3: Write-through (update cache and DB together)
async function updateProductWriteThrough(id: number, data: any) {
  // 1. Update database
  await db.query('UPDATE products SET ... WHERE id = ?', [id, data]);
  
  // 2. Update cache
  const updated = await db.query('SELECT * FROM products WHERE id = ?', [id]);
  await redis.set(`product:${id}`, JSON.stringify(updated));
}

// Strategy 4: Cache-aside with versioning
async function getProductVersioned(id: number) {
  const version = await redis.get(`product:${id}:version`);
  const cacheKey = `product:${id}:v${version}`;
  
  let product = await redis.get(cacheKey);
  if (product) return JSON.parse(product);
  
  product = await db.query('SELECT * FROM products WHERE id = ?', [id]);
  await redis.set(cacheKey, JSON.stringify(product));
  return product;
}

// When updating, increment version
async function updateProductVersioned(id: number, data: any) {
  await db.query('UPDATE products SET ... WHERE id = ?', [id, data]);
  await redis.incr(`product:${id}:version`);  // New version
  // Old cache automatically becomes stale
}
```

---

### 3. Database-Level Caching

```sql
-- PostgreSQL shared_buffers (RAM cache for frequently accessed pages)
SHOW shared_buffers;  -- Default: 128MB

-- Increase for better performance (25% of total RAM)
ALTER SYSTEM SET shared_buffers = '4GB';
-- Restart required

-- Check cache hit rate
SELECT 
  sum(heap_blks_read) as heap_read,
  sum(heap_blks_hit)  as heap_hit,
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as cache_hit_ratio
FROM pg_statio_user_tables;
-- Aim for > 0.99 (99% cache hit rate)
```

---

### 4. Materialized Views (Pre-computed Results)

```sql
-- Expensive query: Product statistics
SELECT 
  p.id,
  p.name,
  p.price,
  COUNT(r.id) as review_count,
  AVG(r.rating) as avg_rating,
  SUM(oi.quantity) as total_sold
FROM products p
LEFT JOIN reviews r ON p.id = r.product_id
LEFT JOIN order_items oi ON p.id = oi.product_id
GROUP BY p.id, p.name, p.price;
-- Takes 2 seconds to run!

-- Create materialized view (stores results)
CREATE MATERIALIZED VIEW product_stats AS
SELECT 
  p.id,
  p.name,
  p.price,
  COUNT(r.id) as review_count,
  AVG(r.rating) as avg_rating,
  SUM(oi.quantity) as total_sold
FROM products p
LEFT JOIN reviews r ON p.id = r.product_id
LEFT JOIN order_items oi ON p.id = oi.product_id
GROUP BY p.id, p.name, p.price;

-- Now query is instant!
SELECT * FROM product_stats WHERE avg_rating >= 4;
-- Takes 10ms instead of 2000ms!

-- Refresh periodically (nightly cron job)
REFRESH MATERIALIZED VIEW product_stats;

-- Or concurrent refresh (doesn't lock readers)
REFRESH MATERIALIZED VIEW CONCURRENTLY product_stats;
```

---

### Caching Best Practices

```typescript
// 1. Cache hot data (frequently accessed)
// ✅ Product details, user profiles, trending lists
// ❌ Admin logs, old data

// 2. Set appropriate TTL
// Frequently changing: 1-5 minutes
// Rarely changing: 1-24 hours
// Static: 7 days

// 3. Cache at multiple levels
Browser Cache (static assets)
    ↓
CDN Cache (images, CSS, JS)
    ↓
Application Cache (Redis - API responses)
    ↓
Database Cache (PostgreSQL shared_buffers)
    ↓
Disk

// 4. Monitor cache hit rate
const hits = await redis.get('cache:hits') || 0;
const misses = await redis.get('cache:misses') || 0;
const hitRate = hits / (hits + misses);
console.log(`Cache hit rate: ${(hitRate * 100).toFixed(2)}%`);
// Aim for > 90%

// 5. Handle cache stampede (thundering herd)
// Problem: Cache expires, 1000 requests hit DB simultaneously
async function getProductSafe(id: number) {
  const cacheKey = `product:${id}`;
  const lockKey = `lock:product:${id}`;
  
  let product = await redis.get(cacheKey);
  if (product) return JSON.parse(product);
  
  // Try to acquire lock
  const lockAcquired = await redis.set(lockKey, '1', 'EX', 10, 'NX');
  
  if (lockAcquired) {
    // This request refreshes cache
    product = await db.query('SELECT * FROM products WHERE id = ?', [id]);
    await redis.setex(cacheKey, 300, JSON.stringify(product));
    await redis.del(lockKey);
    return product;
  } else {
    // Other requests wait for cache to be refreshed
    await sleep(50);
    return getProductSafe(id);  // Retry
  }
}
```

---

## Scaling Databases

### Vertical vs Horizontal Scaling

#### Vertical Scaling (Scale Up) ⬆️

**Definition:** Add more power to a single server

```
Before:              After:
┌─────────┐         ┌──────────┐
│ 4 CPU   │    →    │ 16 CPU   │
│ 16GB RAM│         │ 128GB RAM│
│ 1TB SSD │         │ 10TB SSD │
└─────────┘         └──────────┘
  $200/mo            $2000/mo
```

**✅ Pros:**
- Simple (no code changes)
- No data synchronization
- Easier to maintain
- Single source of truth

**❌ Cons:**
- Expensive (enterprise hardware)
- Limited ceiling (physical limits)
- Single point of failure
- Downtime during upgrades

**When to Use:**
- Small to medium applications
- Budget/timeline constraints
- Simple architecture preferred

---

#### Horizontal Scaling (Scale Out) ➡️

**Definition:** Add more servers

```
Before:              After:
┌─────────┐         ┌─────────┐  ┌─────────┐  ┌─────────┐
│ Server  │    →    │ Server1 │  │ Server2 │  │ Server3 │
│         │         │         │  │         │  │         │
└─────────┘         └─────────┘  └─────────┘  └─────────┘
  $200/mo             $200/mo      $200/mo      $200/mo
                      = $600/mo for 3x capacity
```

**✅ Pros:**
- Unlimited scaling potential
- Cheaper (commodity hardware)
- No downtime (rolling upgrades)
- Fault tolerant (one fails, others work)

**❌ Cons:**
- Complex (data synchronization)
- Application changes needed
- More infrastructure to manage
- CAP theorem limitations

**When to Use:**
- Large applications (millions of users)
- Need high availability
- Expect rapid growth

---

### Read Replicas (Read Scaling) 📖

**Problem:** 90% of operations are reads, but only 1 server handles them.

**Solution:** Create read-only copies (replicas)

```
                    ┌─────────────┐
                    │   PRIMARY   │ (Master)
                    │ (WRITES)    │
                    └──────┬──────┘
                           │
                Async Replication
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼─────┐     ┌────▼─────┐     ┌────▼─────┐
    │ Replica1 │     │ Replica2 │     │ Replica3 │
    │ (READS)  │     │ (READS)  │     │ (READS)  │
    └──────────┘     └──────────┘     └──────────┘
      US East          EU West           Asia
```

**Implementation:**

```typescript
import { Pool } from 'pg';

// Primary database (writes)
const primary = new Pool({
  host: 'primary.db.example.com',
  database: 'mydb',
  user: 'app_user',
  password: 'password',
  max: 20
});

// Read replicas (reads)
const replicas = [
  new Pool({ host: 'replica1.db.example.com', ... }),
  new Pool({ host: 'replica2.db.example.com', ... }),
  new Pool({ host: 'replica3.db.example.com', ... })
];

let replicaIndex = 0;

function getReadReplica() {
  // Round-robin load balancing
  const replica = replicas[replicaIndex];
  replicaIndex = (replicaIndex + 1) % replicas.length;
  return replica;
}

// Write to primary
async function createUser(userData) {
  return await primary.query(
    'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
    [userData.name, userData.email]
  );
}

// Read from replica
async function getUsers() {
  const replica = getReadReplica();
  return await replica.query('SELECT * FROM users ORDER BY created_at DESC LIMIT 20');
}

// Read from primary if consistency critical
async function getUserBalance(userId) {
  // Financial data - must be from primary!
  return await primary.query('SELECT balance FROM accounts WHERE user_id = $1', [userId]);
}
```

**PostgreSQL Replication Setup:**

```sql
-- Primary server: postgresql.conf
wal_level = replica
max_wal_senders = 3
wal_keep_size = 1GB

-- Create replication user
CREATE USER replicator REPLICATION LOGIN ENCRYPTED PASSWORD 'password';

-- Replica server: Create replica
pg_basebackup -h primary.db.example.com -D /var/lib/postgresql/14/main -U replicator -P

-- Replica server: recovery.conf (PostgreSQL 12+: postgresql.auto.conf)
primary_conninfo = 'host=primary.db.example.com port=5432 user=replicator password=password'
```

**Benefits:**
- Distribute read load across servers
- Geographic distribution (lower latency)
- Can promote replica to primary if primary fails

**Trade-offs:**
- **Replication Lag**: Replica might be few seconds behind
- **Eventual Consistency**: Recent writes might not appear immediately

**Replication Lag Example:**

```typescript
// User updates profile
await primary.query('UPDATE users SET name = $1 WHERE id = $2', ['New Name', 1]);

// Immediately read from replica
const replica = getReadReplica();
const user = await replica.query('SELECT name FROM users WHERE id = $1', [1]);
// Might still see "Old Name" for a few seconds until replication catches up!

// Solution: Read from primary after write if consistency critical
const user = await primary.query('SELECT name FROM users WHERE id = $1', [1]);
```

---

### Sharding (Write Scaling) ✂️

**Problem:** Single database can't handle millions of writes per second.

**Solution:** Split data across multiple databases (shards)

#### Horizontal Sharding (by User ID)

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Shard 0    │    │   Shard 1    │    │   Shard 2    │
│ Users 0-1M   │    │ Users 1M-2M  │    │ Users 2M-3M  │
├──────────────┤    ├──────────────┤    ├──────────────┤
│ id: 500000   │    │ id: 1500000  │    │ id: 2500000  │
│ name: Alice  │    │ name: Bob    │    │ name: Carol  │
└──────────────┘    └──────────────┘    └──────────────┘
```

**Implementation:**

```typescript
// Shard selection logic
function getShardForUser(userId: number): Pool {
  const shardIndex = userId % 3;  // 3 shards (0, 1, 2)
  return shards[shardIndex];
}

const shards = [
  new Pool({ host: 'shard0.db.example.com', ... }),
  new Pool({ host: 'shard1.db.example.com', ... }),
  new Pool({ host: 'shard2.db.example.com', ... })
];

// Write to appropriate shard
async function createOrder(userId: number, orderData: any) {
  const shard = getShardForUser(userId);
  return await shard.query(
    'INSERT INTO orders (user_id, total) VALUES ($1, $2) RETURNING *',
    [userId, orderData.total]
  );
}

// Read from appropriate shard
async function getUserOrders(userId: number) {
  const shard = getShardForUser(userId);
  return await shard.query(
    'SELECT * FROM orders WHERE user_id = $1 ORDER BY created_at DESC',
    [userId]
  );
}
```

#### Geographic Sharding

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Asia Shard  │    │   EU Shard   │    │   US Shard   │
│ (Singapore)  │    │  (Frankfurt) │    │ (Virginia)   │
├──────────────┤    ├──────────────┤    ├──────────────┤
│ Users from   │    │ Users from   │    │ Users from   │
│ Bangladesh,  │    │ Germany,     │    │ USA,         │
│ India, China │    │ France, UK   │    │ Canada       │
└──────────────┘    └──────────────┘    └──────────────┘
```

**Benefits:**
- Distribute writes across multiple databases
- Each shard smaller and faster
- Geographic locality (lower latency for users)
- Unlimited scaling potential

**Challenges:**

**1. Cross-Shard Queries:**

```typescript
// ❌ Can't JOIN across shards
// This query spans all shards - must aggregate in application
async function getTopUsers() {
  const shard0Users = await shards[0].query('SELECT * FROM users ORDER BY orders_count DESC LIMIT 10');
  const shard1Users = await shards[1].query('SELECT * FROM users ORDER BY orders_count DESC LIMIT 10');
  const shard2Users = await shards[2].query('SELECT * FROM users ORDER BY orders_count DESC LIMIT 10');
  
  // Merge and sort in application
  const allUsers = [...shard0Users, ...shard1Users, ...shard2Users];
  return allUsers.sort((a, b) => b.orders_count - a.orders_count).slice(0, 10);
}
```

**2. Rebalancing (Hard to Move Data):**

```typescript
// Adding new shard requires re-distributing data
// Old: 3 shards (0, 1, 2)
// New: 4 shards (0, 1, 2, 3)
// Problem: user_id % 3 !== user_id % 4
// Must migrate data!

async function rebalance() {
  // This is complex and requires downtime or careful coordination
  // Better: Use consistent hashing to minimize data movement
}
```

**3. Hotspots (Uneven Load):**

```typescript
// Problem: Celebrity user creates millions of orders
// Their shard becomes overloaded

// Solution 1: Shard by multiple keys
function getShardForOrder(userId: number, orderId: number): Pool {
  const shardIndex = (userId + orderId) % 3;
  return shards[shardIndex];
}

// Solution 2: Further shard hot users
if (isCelebrityUser(userId)) {
  return getCelebrityShard(userId);
}
```

---

### Connection Pooling 🏊

**Problem:** Opening new database connection is expensive (~100ms)

**Solution:** Reuse connections from a pool

```typescript
import { Pool } from 'pg';

// ❌ Without pooling (slow)
async function getUser(id: number) {
  const client = await createNewConnection();  // 100ms!
  const user = await client.query('SELECT * FROM users WHERE id = $1', [id]);
  await client.end();
  return user;
}
// Every request: 100ms connection overhead

// ✅ With pooling (fast)
const pool = new Pool({
  host: 'localhost',
  database: 'mydb',
  user: 'app_user',
  password: 'password',
  
  max: 20,  // Maximum pool size
  min: 5,   // Minimum idle connections
  idleTimeoutMillis: 30000,  // Close idle connections after 30s
  connectionTimeoutMillis: 2000  // Wait 2s for connection
});

async function getUser(id: number) {
  const client = await pool.connect();  // < 1ms (reuses existing)
  const user = await client.query('SELECT * FROM users WHERE id = $1', [id]);
  client.release();  // Return to pool
  return user;
}

// Connection reused: ~1ms overhead
// 100x faster!

// Monitor pool
pool.on('connect', () => {
  console.log('New client connected');
});

pool.on('acquire', () => {
  console.log('Client acquired from pool');
});

pool.on('error', (err) => {
  console.error('Pool error:', err);
});

// Check pool stats
console.log('Total clients:', pool.totalCount);
console.log('Idle clients:', pool.idleCount);
console.log('Waiting clients:', pool.waitingCount);
```

**Best Practices:**
- Set `max` to expected concurrent connections
- Set `min` to keep connections warm
- Monitor pool exhaustion (all connections busy)
- Use separate pools for read replicas

---

## Advanced Database Concepts

### 1. Transactions & Isolation Levels

```sql
-- Set isolation level
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

  SELECT * FROM accounts WHERE id = 1 FOR UPDATE;  -- Lock row
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;

-- Isolation levels (strictest to loosest)
-- SERIALIZABLE: Safest, slowest
-- REPEATABLE READ: Good balance
-- READ COMMITTED: Default (PostgreSQL)
-- READ UNCOMMITTED: Fastest, least safe
```

---

### 2. Database Locks

```sql
-- Row-level lock (locks specific rows)
SELECT * FROM products WHERE id = 1 FOR UPDATE;

-- Table-level lock (locks entire table)
LOCK TABLE products IN EXCLUSIVE MODE;

-- Advisory lock (application-level coordination)
SELECT pg_advisory_lock(123);
-- Critical section
SELECT pg_advisory_unlock(123);
```

---

### 3. Full-Text Search

```sql
-- Add tsvector column
ALTER TABLE products ADD COLUMN search_vector tsvector;

-- Create GIN index
CREATE INDEX idx_products_search ON products USING GIN(search_vector);

-- Update search vector
UPDATE products 
SET search_vector = to_tsvector('english', name || ' ' || description);

-- Search query
SELECT * FROM products
WHERE search_vector @@ to_tsquery('english', 'gaming & laptop')
ORDER BY ts_rank(search_vector, to_tsquery('english', 'gaming & laptop')) DESC;

-- With highlighting
SELECT 
  name,
  ts_headline('english', description, to_tsquery('english', 'gaming & laptop')) as highlighted
FROM products
WHERE search_vector @@ to_tsquery('english', 'gaming & laptop');
```

---

### 4. Partitioning (Split Large Tables)

```sql
-- Partition by range (date)
CREATE TABLE orders (
  id BIGSERIAL,
  user_id INTEGER,
  total DECIMAL(10, 2),
  created_at TIMESTAMP
) PARTITION BY RANGE (created_at);

-- Create monthly partitions
CREATE TABLE orders_2024_01 PARTITION OF orders
  FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE orders_2024_02 PARTITION OF orders
  FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

CREATE TABLE orders_2024_03 PARTITION OF orders
  FOR VALUES FROM ('2024-03-01') TO ('2024-04-01');

-- Queries automatically use correct partition
SELECT * FROM orders WHERE created_at >= '2024-01-15';
-- Only scans orders_2024_01 (faster!)

-- Benefits:
-- 1. Smaller indexes (per partition)
-- 2. Faster queries (scan only relevant partitions)
-- 3. Easy to drop old data (DROP TABLE orders_2023_01)
```

---

### 5. Database Triggers

```sql
-- Auto-update timestamp
CREATE OR REPLACE FUNCTION update_timestamp()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = CURRENT_TIMESTAMP;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_users_timestamp
  BEFORE UPDATE ON users
  FOR EACH ROW
  EXECUTE FUNCTION update_timestamp();

-- Audit trail
CREATE TABLE audit_log (
  id SERIAL PRIMARY KEY,
  table_name VARCHAR(50),
  action VARCHAR(10),
  old_data JSONB,
  new_data JSONB,
  changed_by VARCHAR(100),
  changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE FUNCTION audit_changes()
RETURNS TRIGGER AS $$
BEGIN
  IF TG_OP = 'INSERT' THEN
    INSERT INTO audit_log (table_name, action, new_data, changed_by)
    VALUES (TG_TABLE_NAME, 'INSERT', row_to_json(NEW), current_user);
    RETURN NEW;
  ELSIF TG_OP = 'UPDATE' THEN
    INSERT INTO audit_log (table_name, action, old_data, new_data, changed_by)
    VALUES (TG_TABLE_NAME, 'UPDATE', row_to_json(OLD), row_to_json(NEW), current_user);
    RETURN NEW;
  ELSIF TG_OP = 'DELETE' THEN
    INSERT INTO audit_log (table_name, action, old_data, changed_by)
    VALUES (TG_TABLE_NAME, 'DELETE', row_to_json(OLD), current_user);
    RETURN OLD;
  END IF;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER audit_users
  AFTER INSERT OR UPDATE OR DELETE ON users
  FOR EACH ROW
  EXECUTE FUNCTION audit_changes();
```

---

## Database Security

### 1. Principle of Least Privilege

**Rule:** Give each user/application only the permissions they need.

```sql
-- ❌ BAD: Give admin access to application
GRANT ALL PRIVILEGES ON DATABASE mydb TO app_user;
-- Application can DROP tables, CREATE users, etc.!

-- ✅ GOOD: Only necessary permissions
CREATE USER app_user WITH PASSWORD 'secure_password';

-- Read-only access to most tables
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_user;

-- Write access only to specific tables
GRANT INSERT, UPDATE, DELETE ON orders, order_items, cart_items TO app_user;

-- No access to sensitive tables
REVOKE ALL ON users_sensitive_data, admin_logs FROM app_user;

-- No schema changes
REVOKE CREATE ON SCHEMA public FROM app_user;
```

**Role-Based Access:**

```sql
-- Create roles for different access levels
CREATE ROLE readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;

CREATE ROLE app_writer;
GRANT SELECT, INSERT, UPDATE, DELETE ON orders, products TO app_writer;

CREATE ROLE admin;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO admin;

-- Assign roles to users
CREATE USER app_user1 WITH PASSWORD '...';
GRANT app_writer TO app_user1;

CREATE USER reporting_user WITH PASSWORD '...';
GRANT readonly TO reporting_user;
```

---

### 2. Password Security

```typescript
import bcrypt from 'bcrypt';
import crypto from 'crypto';

// ❌ NEVER store plain passwords
async function registerUserBad(email: string, password: string) {
  await db.query(
    'INSERT INTO users (email, password) VALUES ($1, $2)',
    [email, password]  // DANGER!
  );
}

// ✅ Always hash passwords
async function registerUser(email: string, password: string) {
  const salt = await bcrypt.genSalt(10);  // Cost factor: 10
  const hashedPassword = await bcrypt.hash(password, salt);
  
  await db.query(
    'INSERT INTO users (email, password_hash) VALUES ($1, $2)',
    [email, hashedPassword]
  );
}

// Verify password
async function loginUser(email: string, password: string) {
  const user = await db.query(
    'SELECT id, password_hash FROM users WHERE email = $1',
    [email]
  );
  
  if (!user) return null;
  
  const isValid = await bcrypt.compare(password, user.password_hash);
  return isValid ? user : null;
}

// Generate secure random tokens
function generateToken(): string {
  return crypto.randomBytes(32).toString('hex');
}
```

---

### 3. SQL Injection Prevention

**SQL Injection:** Attacker manipulates SQL queries through user input.

```typescript
// ❌ DANGEROUS: SQL Injection vulnerable
async function getUserBad(userId: string) {
  const query = `SELECT * FROM users WHERE id = ${userId}`;
  return await db.query(query);
}

// Attack:
// Input: "1 OR 1=1 --"
// Query becomes: SELECT * FROM users WHERE id = 1 OR 1=1 --
// Returns ALL users!

// Attack 2:
// Input: "1; DROP TABLE users; --"
// Query becomes: SELECT * FROM users WHERE id = 1; DROP TABLE users; --
// DELETES ENTIRE TABLE!

// ✅ SAFE: Parameterized queries
async function getUserSafe(userId: string) {
  return await db.query(
    'SELECT * FROM users WHERE id = $1',
    [userId]  // Parameters are escaped automatically
  );
}

// ✅ SAFE: Use ORM
const user = await prisma.user.findUnique({
  where: { id: userId }
});

// ✅ SAFE: Input validation
function validateUserId(userId: string): number {
  const id = parseInt(userId, 10);
  if (isNaN(id) || id < 1) {
    throw new Error('Invalid user ID');
  }
  return id;
}
```

---

### 4. Encryption

#### Encryption at Rest

```sql
-- PostgreSQL: Enable encryption for sensitive columns
CREATE EXTENSION pgcrypto;

-- Encrypt data before storing
INSERT INTO users (ssn, credit_card) 
VALUES (
  pgp_sym_encrypt('123-45-6789', 'encryption_key'),
  pgp_sym_encrypt('4111-1111-1111-1111', 'encryption_key')
);

-- Decrypt when reading
SELECT 
  id,
  name,
  pgp_sym_decrypt(ssn::bytea, 'encryption_key') as ssn,
  pgp_sym_decrypt(credit_card::bytea, 'encryption_key') as credit_card
FROM users
WHERE id = 1;
```

#### Encryption in Transit

```typescript
// ✅ Use SSL/TLS for database connections
import { Pool } from 'pg';

const pool = new Pool({
  host: 'database.example.com',
  port: 5432,
  database: 'mydb',
  user: 'app_user',
  password: 'password',
  
  ssl: {
    rejectUnauthorized: true,  // Verify server certificate
    ca: fs.readFileSync('/path/to/ca-cert.pem').toString(),
    key: fs.readFileSync('/path/to/client-key.pem').toString(),
    cert: fs.readFileSync('/path/to/client-cert.pem').toString()
  }
});
```

---

### 5. Environment Variables & Secrets

```typescript
// ❌ NEVER hardcode credentials
const pool = new Pool({
  host: 'localhost',
  user: 'postgres',
  password: 'supersecret123'  // DANGER!
});

// ✅ Use environment variables
import dotenv from 'dotenv';
dotenv.config();

const pool = new Pool({
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT || '5432'),
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  ssl: process.env.DB_SSL === 'true'
});

// .env file (NEVER commit to git!)
DB_HOST=database.example.com
DB_PORT=5432
DB_NAME=production_db
DB_USER=app_user
DB_PASSWORD=super_secure_password_here
DB_SSL=true

// .gitignore
.env
.env.local
.env.production
```

---

### 6. Row-Level Security (PostgreSQL)

```sql
-- Enable row-level security
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their own data
CREATE POLICY user_isolation ON users
  FOR ALL
  USING (id = current_setting('app.current_user')::INTEGER);

-- Set current user in application
SET app.current_user = '123';

-- This query only returns user 123's data
SELECT * FROM users;
-- Automatically filtered: WHERE id = 123

-- Multi-tenant example
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON orders
  FOR ALL
  USING (tenant_id = current_setting('app.current_tenant')::INTEGER);

-- Application sets tenant
SET app.current_tenant = '456';

-- Queries automatically filtered by tenant
SELECT * FROM orders;  -- Only tenant 456's orders
```

---

## Backup & Recovery

### Backup Strategies

#### 1. Logical Backups (SQL Dump)

**PostgreSQL:**

```bash
# Full database backup
pg_dump -U postgres -d mydb -F c -f /backups/mydb_$(date +%Y%m%d).dump

# Options:
# -F c: Custom format (compressed, allows partial restore)
# -F p: Plain SQL format (readable, can edit)
# -F t: Tar format

# Backup specific tables
pg_dump -U postgres -d mydb -t users -t orders -f tables_backup.sql

# Backup with inserts (compatible with any database)
pg_dump -U postgres -d mydb --inserts -f mydb_inserts.sql

# Restore
pg_restore -U postgres -d mydb /backups/mydb_20241229.dump

# Restore specific table
pg_restore -U postgres -d mydb -t users /backups/mydb_20241229.dump
```

**MySQL:**

```bash
# Full database backup
mysqldump -u root -p mydb > /backups/mydb_$(date +%Y%m%d).sql

# All databases
mysqldump -u root -p --all-databases > /backups/all_dbs_$(date +%Y%m%d).sql

# Specific tables
mysqldump -u root -p mydb users orders > /backups/tables.sql

# Restore
mysql -u root -p mydb < /backups/mydb_20241229.sql
```

---

#### 2. Physical Backups

```bash
# PostgreSQL: Base backup (binary copy)
pg_basebackup -D /backups/basebackup -F tar -z -P -U replication_user

# Options:
# -D: Destination directory
# -F tar: Tar format
# -z: Compress
# -P: Show progress

# MySQL: Physical backup
mysqlbackup --backup-dir=/backups/physical --backup-image=backup.mbi --compress backup-to-image

# Restore
mysqlbackup --backup-dir=/backups/physical --backup-image=backup.mbi copy-back-and-apply-log
```

---

#### 3. Continuous Archiving (WAL)

```sql
-- postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /archive/%f'  -- Or use s3 sync

# Point-in-time recovery (PITR)
# Restore to specific timestamp
pg_restore --target-time='2024-12-29 10:30:00' /backups/basebackup
```

---

### Backup Best Practices

```bash
# 1. Automate backups (cron job)
# /etc/cron.d/postgres-backup
0 2 * * * postgres pg_dump -U postgres -d mydb -F c -f /backups/mydb_$(date +\%Y\%m\%d).dump

# 2. Test restores regularly (monthly)
pg_restore -U postgres -d mydb_test /backups/mydb_20241229.dump

# 3. Store backups off-site
aws s3 sync /backups/ s3://my-db-backups/

# 4. Encrypt backups
pg_dump -U postgres -d mydb | gpg --encrypt --recipient admin@example.com > backup.dump.gpg

# 5. Retention policy
# Keep: Daily for 7 days, Weekly for 4 weeks, Monthly for 12 months
find /backups -name "*.dump" -mtime +7 -delete  # Delete older than 7 days

# 6. Monitor backup success
#!/bin/bash
BACKUP_FILE="/backups/mydb_$(date +%Y%m%d).dump"
pg_dump -U postgres -d mydb -F c -f $BACKUP_FILE

if [ $? -eq 0 ] && [ -f $BACKUP_FILE ]; then
  echo "Backup successful: $BACKUP_FILE"
  # Send success notification
else
  echo "Backup failed!"
  # Send alert
  mail -s "Backup Failed" admin@example.com
fi
```

---

### Recovery Scenarios

#### Scenario 1: Accidental DELETE

```sql
-- Oops! Deleted all users
DELETE FROM users;  -- No WHERE clause!

-- Solution 1: Restore from backup
pg_restore -U postgres -d mydb -t users /backups/mydb_latest.dump

-- Solution 2: Point-in-time recovery (if using WAL)
-- Restore to 5 minutes before mistake
pg_restore --target-time='2024-12-29 14:55:00' /backups/basebackup

-- Prevention: Always use transactions
BEGIN;
DELETE FROM users WHERE city = 'Test';
SELECT COUNT(*) FROM users;  -- Verify count before commit
ROLLBACK;  -- Undo if wrong
-- or
COMMIT;  -- Apply if correct
```

---

#### Scenario 2: Corrupted Database

```bash
# PostgreSQL: Check for corruption
postgres -D /var/lib/postgresql/data --single -P disable_system_indexes

# If corruption detected, restore from backup
systemctl stop postgresql
rm -rf /var/lib/postgresql/data/*
pg_basebackup -D /var/lib/postgresql/data -U replication_user
systemctl start postgresql
```

---

#### Scenario 3: Complete Server Loss

```bash
# Recovery steps:
# 1. Provision new server
# 2. Install PostgreSQL
# 3. Restore from off-site backup
aws s3 cp s3://my-db-backups/mydb_20241229.dump /tmp/
pg_restore -U postgres -d mydb /tmp/mydb_20241229.dump

# 4. Point-in-time recovery (if needed)
# 5. Update application connection strings
# 6. Test thoroughly before going live
```

---

## Monitoring & Observability

### Key Metrics to Monitor

#### 1. Query Performance

```sql
-- Enable pg_stat_statements
CREATE EXTENSION pg_stat_statements;

-- View slow queries
SELECT 
  query,
  calls,
  total_exec_time,
  mean_exec_time,
  max_exec_time,
  stddev_exec_time
FROM pg_stat_statements
WHERE mean_exec_time > 100  -- Slower than 100ms
ORDER BY mean_exec_time DESC
LIMIT 20;

-- Find queries causing most load
SELECT 
  query,
  calls,
  total_exec_time,
  (total_exec_time / sum(total_exec_time) OVER ()) * 100 AS percent_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

---

#### 2. Connection Pool Metrics

```typescript
// Monitor pool health
setInterval(() => {
  console.log({
    total: pool.totalCount,
    idle: pool.idleCount,
    waiting: pool.waitingCount,
    utilization: (pool.totalCount - pool.idleCount) / pool.totalCount * 100
  });
  
  // Alert if utilization > 80%
  if ((pool.totalCount - pool.idleCount) / pool.totalCount > 0.8) {
    console.warn('Connection pool utilization > 80%!');
  }
}, 10000);
```

---

#### 3. Cache Hit Ratio

```sql
-- PostgreSQL cache hit ratio (should be > 99%)
SELECT 
  sum(heap_blks_read) as heap_read,
  sum(heap_blks_hit) as heap_hit,
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as cache_hit_ratio
FROM pg_statio_user_tables;

-- If < 0.99, increase shared_buffers
ALTER SYSTEM SET shared_buffers = '4GB';  -- 25% of RAM
```

---

#### 4. Replication Lag

```sql
-- Check replication lag (on primary)
SELECT 
  client_addr,
  state,
  sync_state,
  replay_lag,
  write_lag,
  flush_lag
FROM pg_stat_replication;

-- Alert if lag > 10 seconds
```

---

#### 5. Disk Usage

```sql
-- Database size
SELECT 
  pg_database.datname,
  pg_size_pretty(pg_database_size(pg_database.datname)) AS size
FROM pg_database
ORDER BY pg_database_size(pg_database.datname) DESC;

-- Table sizes
SELECT 
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size,
  pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS table_size,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - pg_relation_size(schemaname||'.'||tablename)) AS index_size
FROM pg_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 20;
```

---

### Monitoring Tools

**1. Prometheus + Grafana**

```yaml
# docker-compose.yml
services:
  postgres_exporter:
    image: prometheuscommunity/postgres-exporter
    environment:
      DATA_SOURCE_NAME: "postgresql://monitoring_user:password@postgres:5432/mydb?sslmode=disable"
    ports:
      - "9187:9187"
  
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
  
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
```

**2. pgBadger (PostgreSQL Log Analyzer)**

```bash
# Enable logging in postgresql.conf
log_min_duration_statement = 100  # Log queries > 100ms
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on

# Generate report
pgbadger /var/log/postgresql/postgresql-*.log -o report.html

# Open report.html in browser
```

---

## Database Testing

### 1. Unit Testing (Mock Database)

```typescript
import { jest } from '@jest/globals';

// Mock database
const mockDb = {
  query: jest.fn()
};

describe('UserService', () => {
  it('should create user', async () => {
    // Arrange
    mockDb.query.mockResolvedValue({ 
      rows: [{ id: 1, name: 'Test User', email: 'test@example.com' }] 
    });
    
    const userService = new UserService(mockDb);
    
    // Act
    const user = await userService.createUser({ 
      name: 'Test User', 
      email: 'test@example.com' 
    });
    
    // Assert
    expect(mockDb.query).toHaveBeenCalledWith(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
      ['Test User', 'test@example.com']
    );
    expect(user.id).toBe(1);
    expect(user.name).toBe('Test User');
  });
});
```

---

### 2. Integration Testing (Test Database)

```typescript
import { Pool } from 'pg';

describe('Database Integration', () => {
  let pool: Pool;
  
  beforeAll(async () => {
    // Create test database
    pool = new Pool({
      host: 'localhost',
      database: 'mydb_test',
      user: 'test_user',
      password: 'test_password'
    });
    
    // Run migrations
    await runMigrations(pool);
  });
  
  afterAll(async () => {
    await pool.end();
  });
  
  beforeEach(async () => {
    // Clear data before each test
    await pool.query('TRUNCATE users, orders, products CASCADE');
  });
  
  it('should create and retrieve user', async () => {
    // Insert
    const insertResult = await pool.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
      ['Test User', 'test@example.com']
    );
    
    const userId = insertResult.rows[0].id;
    
    // Retrieve
    const selectResult = await pool.query(
      'SELECT * FROM users WHERE id = $1',
      [userId]
    );
    
    expect(selectResult.rows[0].name).toBe('Test User');
    expect(selectResult.rows[0].email).toBe('test@example.com');
  });
  
  it('should enforce unique email constraint', async () => {
    await pool.query(
      'INSERT INTO users (name, email) VALUES ($1, $2)',
      ['User 1', 'duplicate@example.com']
    );
    
    // Try to insert duplicate email
    await expect(
      pool.query(
        'INSERT INTO users (name, email) VALUES ($1, $2)',
        ['User 2', 'duplicate@example.com']
      )
    ).rejects.toThrow(/unique constraint/);
  });
});
```

---

### 3. Load Testing

```bash
# pgbench (PostgreSQL)
# Initialize test data
pgbench -i -s 100 testdb  # Scale factor 100

# Run load test
pgbench -c 10 -j 2 -t 1000 testdb
# -c 10: 10 concurrent clients
# -j 2: 2 threads
# -t 1000: 1000 transactions per client

# Results:
# transaction type: <builtin: TPC-B (sort of)>
# number of transactions actually processed: 10000
# latency average = 15.844 ms
# tps = 631.135591 (including connections establishing)
```

---

## Memory Tips & Tricks

### Remember ACID: "**A Car Is Durable**"

- **A** - **A**tomicity (All or nothing - bank transfer)
- **C** - **C**onsistency (Rules followed - no negative balance)
- **I** - **I**solation (No interference - can't buy last item twice)
- **D** - **D**urability (Survives crashes - committed data permanent)

### Remember Index Types: "**B**asically **H**ashing **G**ives **P**erformance"

- **B** - **B**-Tree (default, range queries)
- **H** - **H**ash (exact matches only)
- **G** - **G**IN (arrays, JSON, full-text)
- **P** - **P**artial (subset of data)

### Remember Scaling: "**V**ery **H**igh **S**hould **R**eplicate"

- **V** - **V**ertical (scale up - more power to one server)
- **H** - **H**orizontal (scale out - more servers)
- **S** - **S**harding (write scaling - split data)
- **R** - **R**eplicas (read scaling - copy data)

### Query Optimization: "**ISELW**ind" (I Select Wind)

- **I** - **I**ndexes (create on frequently queried columns)
- **S** - **S**elect specific columns (not SELECT *)
- **E** - **E**XPLAIN ANALYZE (understand query plan)
- **L** - **L**IMIT (paginate results)
- **W** - **W**HERE before JOIN (filter early)

### SQL vs NoSQL: "**RASH**" vs "**SAFE**"

**NoSQL (RASH):**
- **R** - **R**apid development (flexible schema)
- **A** - **A**ggregations (horizontal scaling)
- **S** - **S**imple queries (key-value lookups)
- **H** - **H**uge scale (millions of users)

**SQL (SAFE):**
- **S** - **S**trong consistency (ACID)
- **A** - **A**dvanced queries (JOINs, aggregations)
- **F** - **F**oreign keys (relationships)
- **E** - **E**nterprise (banking, healthcare)

---

## Interview Questions & Answers

### Q1: Explain database normalization

**Answer:**
"Normalization is organizing data to reduce redundancy and improve integrity.

**First Normal Form (1NF): Atomic values**
```
❌ orders table with items as comma-separated string:
id | customer | items
1  | Alice    | Laptop $1000, Mouse $20

✅ Atomic values:
orders:
id | customer
1  | Alice

order_items:
id | order_id | item   | price
1  | 1        | Laptop | 1000
2  | 1        | Mouse  | 20
```

**Second Normal Form (2NF): Remove partial dependencies**
```
✅ Separate customers:
customers:
id | name
1  | Alice

orders:
id | customer_id
1  | 1
```

**Third Normal Form (3NF): Remove transitive dependencies**
```
✅ Products table:
products:
id  | name   | price
101 | Laptop | 1000
102 | Mouse  | 20

order_items:
id | order_id | product_id | quantity
1  | 1        | 101        | 1
2  | 1        | 102        | 2
```

**Benefits:**
- No data redundancy
- Easy to update (change price once)
- Data integrity (consistent prices)

**Trade-off:**
- More JOINs (slower queries)

**When to denormalize:**
- Read-heavy systems
- Need performance
- Data doesn't change often

Example: Store `product_price` in `order_items` to capture price at time of order."

---

### Q2: What are indexes and how do they work?

**Answer:**
"An index is like a book's index - instead of scanning every page, you look up the term and jump to the page.

**Without Index:**
```sql
SELECT * FROM users WHERE email = 'foyez@example.com';
-- Scans all 1M rows sequentially
-- O(n) = 2000ms
```

**With Index:**
```sql
CREATE INDEX idx_users_email ON users(email);
-- B-Tree structure: Binary search
-- O(log n) = 50ms
-- 40x faster!
```

**How it works:**
```
B-Tree Index on email:
        [m@example.com]
       /              \
  [d@example.com]   [t@example.com]
   /        \          /        \
[a@...]  [f@...]   [p@...]   [z@...]
          ↓
    Row pointer to data
```

**Types:**
1. **B-Tree** (default): Range queries, sorting
2. **Hash**: Exact matches only
3. **GIN**: Arrays, JSON, full-text search
4. **Partial**: Subset of data (WHERE clause)

**When to create:**
- Column in WHERE, JOIN, ORDER BY
- High cardinality (many unique values)
- Large table (1000+ rows)

**When NOT to create:**
- Small tables
- Low cardinality (boolean, gender)
- Write-heavy tables (indexes slow writes)

**Trade-offs:**
- ✅ Fast reads
- ❌ Slow writes (must update index)
- ❌ Storage space"

---

### Q3: Explain the CAP theorem

**Answer:**
"CAP theorem states you can only have 2 out of 3 in a distributed database:

- **C**onsistency
- **A**vailability
- **P**artition Tolerance

**Consistency:** All nodes see same data at same time
**Availability:** System always responds (even if some nodes down)
**Partition Tolerance:** System works even if network splits

**The trade-off:**

**CA (Consistency + Availability):**
- Single-server PostgreSQL
- Problem: Not partition-tolerant (if server down, everything down)

**CP (Consistency + Partition tolerance):**
- MongoDB (strong consistency mode)
- Trade-off: Some nodes may reject writes during partition
- Example: Banking (need consistency over availability)

**AP (Availability + Partition tolerance):**
- Cassandra, DynamoDB
- Trade-off: Eventual consistency (stale data for seconds)
- Example: Social media (availability matters more)

**Real-world:**

Banking app:
- Orders/Payments: CP (PostgreSQL) - consistency critical
- Product Catalog: AP (MongoDB) - eventual consistency OK
- Shopping Cart: AP (Redis) - availability critical

**Key insight:** In distributed systems, network partitions WILL happen, so choose between consistency and availability."

---

### Q4: What's the difference between clustered and non-clustered indexes?

**Answer:**
"Clustered and non-clustered indexes differ in how data is physically stored.

**Clustered Index:**
- **Physical order**: Rows stored in index order on disk
- **One per table**: Table can only be sorted one way
- **Usually**: Primary key is clustered index
- **Faster**: Range queries read sequential disk blocks

```
Clustered on user_id:
Disk Storage (physically ordered):
Row 1: id=1, name=Alice
Row 2: id=2, name=Bob
Row 3: id=3, name=Carol

SELECT * FROM users WHERE id BETWEEN 2 AND 4;
-- Reads consecutive disk blocks (fast!)
```

**Non-Clustered Index:**
- **Logical order**: Separate structure pointing to data
- **Multiple allowed**: Many per table
- **Two-step lookup**: Index → Find location → Read row

```
Non-clustered on email:
Index:               Actual Data (unordered):
alice@... → Row 3    Row 1: dave@example.com
bob@... → Row 1      Row 2: carol@example.com
carol@... → Row 2    Row 3: alice@example.com
dave@... → Row 1

SELECT * FROM users WHERE email = 'alice@example.com';
-- 1. Look up in index (finds Row 3)
-- 2. Jump to Row 3 on disk
```

**Comparison:**

| Feature | Clustered | Non-clustered |
|---------|-----------|---------------|
| Per table | 1 | Multiple |
| Physical order | Yes | No |
| Storage | Table itself | Separate |
| Speed | Faster (direct) | Slower (two-step) |
| Range queries | Very fast | Slower |
| Space | None extra | Extra space |

**Best practices:**
- Clustered: Use for primary key or frequently ranged column
- Non-clustered: Create for WHERE, JOIN, ORDER BY columns"

---

### Q5: How would you optimize a slow database query?

**Answer:**
"I follow a systematic approach:

**Step 1: Use EXPLAIN ANALYZE**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 1 AND status = 'pending';

-- Output shows:
-- Seq Scan (sequential scan - BAD!)
-- Execution Time: 2000ms
```

**Step 2: Identify the problem**
- Sequential scan? → Need index
- Too many rows? → Add WHERE or LIMIT
- Complex JOINs? → Simplify or use covering index

**Step 3: Create indexes**
```sql
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Now shows:
-- Index Scan
-- Execution Time: 50ms (40x faster!)
```

**Other optimization techniques:**

**1. SELECT specific columns**
```sql
-- ❌ SELECT *
-- ✅ SELECT id, total, created_at
```

**2. Use LIMIT for pagination**
```sql
SELECT * FROM products LIMIT 20 OFFSET 0;
```

**3. Avoid functions on indexed columns**
```sql
-- ❌ WHERE LOWER(email) = '...'
-- ✅ WHERE email = '...'
```

**4. Use EXISTS instead of IN**
```sql
-- ❌ WHERE id IN (SELECT ...)
-- ✅ WHERE EXISTS (SELECT 1 ...)
```

**5. Cache results**
```typescript
// Redis cache for expensive queries
const cached = await redis.get('top_products');
if (cached) return JSON.parse(cached);

const products = await db.query(...);
await redis.setex('top_products', 300, JSON.stringify(products));
```

**Real example:**
Before: 5000ms (sequential scan, SELECT *, correlated subqueries)
After: 47ms (indexes, specific columns, JOINs)
**106x faster!**"

---

### Q6: Explain database replication and when to use it

**Answer:**
"Database replication creates copies (replicas) of the database for read scaling and redundancy.

**Structure:**
```
           Primary (Master)
           Writes Only
                │
        ┌───────┼───────┐
        │       │       │
    Replica1 Replica2 Replica3
     Reads    Reads    Reads
```

**How it works:**
1. Application writes to primary
2. Primary logs changes (WAL)
3. Replicas fetch and apply changes
4. Application reads from replicas

**Types:**

**1. Asynchronous (most common):**
- Primary doesn't wait for replicas
- Fastest writes
- Possible replication lag (seconds)

**2. Synchronous:**
- Primary waits for at least one replica
- Slower writes
- No replication lag

**Implementation:**
```typescript
// Write to primary
await primaryDB.query('INSERT INTO users ...');

// Read from replica (round-robin)
const replica = selectReplica();
const users = await replica.query('SELECT * FROM users');

// Read from primary if consistency critical
const balance = await primaryDB.query('SELECT balance FROM accounts WHERE id = ?');
```

**When to use:**
- Read-heavy workloads (90% reads)
- Need high availability (primary fails → promote replica)
- Geographic distribution (lower latency)

**Trade-offs:**
- **Replication lag**: Recent writes might not appear on replica
- **Complexity**: Application must route queries correctly
- **Cost**: More servers

**Example:** E-commerce
- Product browsing: Replicas (eventual consistency OK)
- Order placement: Primary (consistency critical)
- Analytics: Dedicated replica (don't affect production)"

---

### Q7: What's database sharding and when would you use it?

**Answer:**
"Sharding splits data across multiple databases to scale writes.

**Structure:**
```
Shard 0: Users 0-1M
Shard 1: Users 1M-2M
Shard 2: Users 2M-3M
```

**Implementation:**
```typescript
function getShardForUser(userId: number): Database {
  return shards[userId % 3];  // Modulo sharding
}

// Write to appropriate shard
const shard = getShardForUser(userId);
await shard.query('INSERT INTO orders ...');
```

**Sharding strategies:**

**1. Range-based:**
```
Shard 0: IDs 1-1M
Shard 1: IDs 1M-2M
```
Pro: Simple
Con: Uneven load if new users concentrated

**2. Hash-based:**
```
shard = userId % numShards
```
Pro: Even distribution
Con: Hard to add shards

**3. Geographic:**
```
Asia Shard: Asian users
EU Shard: European users
US Shard: American users
```
Pro: Low latency for users
Con: Complex routing

**When to use:**
- Write-heavy workloads (millions of writes/sec)
- Single database hitting limits
- Need horizontal scaling

**Challenges:**

**1. Cross-shard queries:**
```typescript
// Can't JOIN across shards
// Must query each shard and merge in application
const results = await Promise.all([
  shard0.query('SELECT ...'),
  shard1.query('SELECT ...'),
  shard2.query('SELECT ...')
]);
const merged = mergeAndSort(results);
```

**2. Distributed transactions:**
```
// Can't have ACID transaction across shards
// Must use saga pattern or 2-phase commit
```

**3. Rebalancing:**
```
// Adding 4th shard changes hash
// userId % 3 !== userId % 4
// Must migrate data (complex!)
```

**Real-world example:**
Instagram shards by user_id - each user's photos on same shard (no cross-shard queries needed for feed)."

---

### Q8: How do you handle database migrations?

**Answer:**
"Database migrations are version-controlled schema changes.

**Tools:** Flyway, Liquibase, Prisma Migrate, Rails migrations

**Best practices:**

**1. Version control**
```sql
-- V1__create_users_table.sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);

-- V2__add_email_to_users.sql
ALTER TABLE users ADD COLUMN email VARCHAR(255) UNIQUE;

-- V3__create_orders_table.sql
CREATE TABLE orders (...);
```

**2. Never modify existing migrations**
```
❌ Edit V1__create_users_table.sql
✅ Create V4__modify_users_table.sql
```

**3. Make migrations reversible**
```sql
-- Up
ALTER TABLE users ADD COLUMN city VARCHAR(50);

-- Down (for rollback)
ALTER TABLE users DROP COLUMN city;
```

**4. Test migrations on copy of production**
```bash
# 1. Backup production
pg_dump production > backup.sql

# 2. Restore to staging
pg_restore backup.sql staging

# 3. Run migration on staging
flyway migrate

# 4. Test thoroughly
# 5. If OK, run on production
```

**5. Handle large tables carefully**
```sql
-- ❌ BAD: Locks table for hours
ALTER TABLE large_table ADD COLUMN new_column VARCHAR(100);

-- ✅ GOOD: Use ALTER TABLE ... NOT VALID (PostgreSQL)
ALTER TABLE large_table ADD COLUMN new_column VARCHAR(100) DEFAULT 'default' NOT VALID;
-- Fast, doesn't validate existing rows

-- Then validate in background
ALTER TABLE large_table VALIDATE CONSTRAINT constraint_name;
```

**6. Backwards compatibility**
```
Deploy sequence:
1. Add new column (default value)
2. Deploy code using both old and new columns
3. Migrate data from old to new
4. Deploy code using only new column
5. Drop old column

This allows rollback at any point!
```

**7. Zero-downtime migrations**
```sql
-- Add nullable column first
ALTER TABLE users ADD COLUMN city VARCHAR(50);

-- Deploy code to populate it
UPDATE users SET city = 'Unknown' WHERE city IS NULL;

-- Later: Make it NOT NULL
ALTER TABLE users ALTER COLUMN city SET NOT NULL;
```

**Real example:**
```bash
# Flyway
flyway migrate  # Run all pending migrations
flyway info     # Show migration status
flyway validate # Check migration consistency
flyway repair   # Fix checksum issues
```"

---

### Q9: What are transactions and isolation levels?

**Answer:**
"Transactions group operations that must succeed or fail together.

**ACID properties:**
```sql
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
-- Both happen or neither (atomicity)
```

**Isolation levels** control how transactions interact:

**1. Read Uncommitted (lowest):**
- Can see uncommitted changes from other transactions (dirty reads)
- Fastest but least safe
- Rarely used

**2. Read Committed (default in PostgreSQL):**
- Only sees committed changes
- Good balance of safety and performance
```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

**3. Repeatable Read:**
- Same query returns same results throughout transaction
- Prevents non-repeatable reads
```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
  SELECT balance FROM accounts WHERE id = 1;  -- Returns 100
  -- Another transaction updates balance to 200
  SELECT balance FROM accounts WHERE id = 1;  -- Still returns 100!
COMMIT;
```

**4. Serializable (highest):**
- Transactions execute as if serial (one at a time)
- Safest but slowest
- Use for financial transactions
```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
  -- Critical financial operation
  SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
```

**Problems prevented:**

| Isolation Level | Dirty Read | Non-repeatable Read | Phantom Read |
|----------------|-----------|---------------------|--------------|
| Read Uncommitted | ❌ | ❌ | ❌ |
| Read Committed | ✅ | ❌ | ❌ |
| Repeatable Read | ✅ | ✅ | ❌ |
| Serializable | ✅ | ✅ | ✅ |

**When to use:**
- **Read Committed**: Most applications (default)
- **Repeatable Read**: Reports that need consistency
- **Serializable**: Financial transactions, critical data

**Trade-off:** Higher isolation = More consistent but slower"

---

### Q10: How would you design a database for a social media platform?

**Answer:**
"I'll walk through the complete design process:

**Step 1: Requirements**
- Users can post, follow, like, comment
- Need personalized feed
- Millions of users, billions of posts
- Real-time updates preferred

**Step 2: Choose databases**
- **Primary**: PostgreSQL (users, relationships, ACID)
- **Cache**: Redis (feed cache, trending)
- **Graph**: Neo4j (friend recommendations)
- **Search**: Elasticsearch (search users/posts)

**Step 3: Schema design**
```sql
CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  bio TEXT,
  follower_count INTEGER DEFAULT 0,
  following_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE followers (
  id BIGSERIAL PRIMARY KEY,
  follower_id BIGINT REFERENCES users(id),
  following_id BIGINT REFERENCES users(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(follower_id, following_id)
);

CREATE TABLE posts (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT REFERENCES users(id),
  content TEXT NOT NULL,
  image_url VARCHAR(500),
  like_count INTEGER DEFAULT 0,
  comment_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE likes (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT REFERENCES users(id),
  post_id BIGINT REFERENCES posts(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(user_id, post_id)
);

CREATE TABLE comments (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT REFERENCES users(id),
  post_id BIGINT REFERENCES posts(id),
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_followers_follower ON followers(follower_id);
CREATE INDEX idx_followers_following ON followers(following_id);
CREATE INDEX idx_posts_user ON posts(user_id);
CREATE INDEX idx_posts_created ON posts(created_at DESC);
CREATE INDEX idx_likes_post ON likes(post_id);
CREATE INDEX idx_comments_post ON comments(post_id);
```

**Step 4: Scaling strategy**

**1. Read replicas for posts/users**
```
Primary → Replica1, Replica2, Replica3
```

**2. Shard users by ID**
```
Shard 0: Users 0-10M
Shard 1: Users 10M-20M
Shard 2: Users 20M-30M
```

**3. Cache feeds in Redis**
```typescript
// Pre-compute and cache feeds
async function getUserFeed(userId: number) {
  const cacheKey = `feed:${userId}`;
  let feed = await redis.get(cacheKey);
  
  if (!feed) {
    // Generate feed from followed users
    feed = await db.query(`
      SELECT p.*, u.username 
      FROM posts p
      JOIN users u ON p.user_id = u.id
      WHERE p.user_id IN (
        SELECT following_id FROM followers WHERE follower_id = ?
      )
      ORDER BY p.created_at DESC
      LIMIT 50
    `, [userId]);
    
    await redis.setex(cacheKey, 300, JSON.stringify(feed));
  }
  
  return JSON.parse(feed);
}
```

**4. Denormalize counts**
```sql
-- Store counts on posts (avoid COUNT queries)
UPDATE posts SET like_count = like_count + 1 WHERE id = 123;
-- Instead of: SELECT COUNT(*) FROM likes WHERE post_id = 123
```

**Step 5: Optimizations**
- CDN for images
- Materialized views for trending posts
- Async processing for notifications
- Elasticsearch for search

This design handles millions of users while maintaining performance!"

---

## Quick Reference

```
ACID: "A Car Is Durable"
- Atomicity: All or nothing
- Consistency: Rules followed
- Isolation: No interference
- Durability: Survives crashes

SQL vs NoSQL:
- SQL: ACID, relationships, complex queries
- NoSQL: Scale, flexibility, speed

Database Choice:
- Transactions? → PostgreSQL
- Flexible schema? → MongoDB
- Caching? → Redis
- Graph? → Neo4j
- Time-series? → Cassandra/TimescaleDB
```

```
Security:
- Principle of least privilege
- Parameterized queries (prevent SQL injection)
- Hash passwords (bcrypt)
- Encrypt sensitive data
- Use SSL/TLS
- Environment variables for secrets

Backup:
- Automate daily backups
- Test restores monthly
- Store off-site (S3, etc.)
- Keep multiple versions
- Point-in-time recovery (WAL)

Monitoring:
- Query performance (pg_stat_statements)
- Connection pool health
- Cache hit ratio (> 99%)
- Replication lag
- Disk usage

Testing:
- Unit tests (mock database)
- Integration tests (test database)
- Load tests (pgbench)
```

---









## Database (DB)

A **database** is a collection of information that is organized so that it can be easily accessed, managed and updated. It is a place to save your application's state so that it can be retrieved later. This allows you to make your servers stateless since your database will be storing all the information.

## Query

A query is a command we send to a database to get information out of the database or to add/update/delete something in the database. It can also aggregate information from a database into some sort of overview.

## Schema

If a database is a table in Microsoft Excel, then a schema is the columns. It's a rigid structure used to model data.

If I had a JSON object of user that looked like `{ "name": "Foyez", "city": "Cumilla", "village": "Surikara" }` then the schema would be name, city, and village. It's the shape of the data.

## Types of databases

1. Relational databases (RDBMS or SQL)
2. Document-based databases (NoSQL)
3. Graph database
4. Key-value store

## ACID (Atomicity, Consistency, Isolation & Durability)

One should think about these four factors when thinking about writing queries. ACID is safe but slow.

**1. Atomicity:** Does this query happen all at once? Or is it broken up into multiple actions? Sometimes this is a very important question. If you're doing financial transactions this can be paramount. Imagine if you have two queries: one to subtract money from one person's account and one to add that amount to another person's account for a bank transfer. What if the power went out in between the two queries? Money would be lost. This is an unacceptable trade-off in this case. You would need this to be atomic. That is, this transaction cannot be divided.

**2. Consistency:** If I have five servers running with one running as the primary server (sometimes called master but we prefer leader or primary) and the primary server crashes, what happens? Again, using money, what if a query had been written to the primary but not yet to the secondaries? This could be a disaster and people could lose money. In this case, we need our servers to be consistent.

**3. Isolation:** Isolation says that we can have a multi-threaded database but it needs to work the same if a query was ran in parallel or if it was ran sequentially. Otherwise it fails the isolation test.

**4. Durability:** Durability says that if a server crashes that we can restore it to the state that it was in previously. Some databases only run in memory so if a server crashes, that data is gone. However this is slow; waiting for a DB to write to disk before finishing a query adds a lot of time.

## Transactions

A transaction is like an envelope of transactions that get to happen all at the same time with a guarantee that they will all get ran at once or none at all. If they are run, it guarantees that no other query will happen between them.
