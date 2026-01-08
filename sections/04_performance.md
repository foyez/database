# Chapter 4: Performance & Optimization
**Indexing Strategies, Query Optimization, and Caching**

---

## Table of Contents
- [4.1 Indexing Strategies](#41-indexing-strategies)
  - [4.1.1 SQL Indexes (B-Tree, Hash, GIN, Partial, Composite)](#411-sql-indexes-b-tree-hash-gin-partial-composite)
  - [4.1.2 NoSQL Indexes (MongoDB)](#412-nosql-indexes-mongodb)
  - [4.1.3 Index Selection & Maintenance](#413-index-selection--maintenance)
- [4.2 Query Optimization](#42-query-optimization)
  - [4.2.1 EXPLAIN & Query Plans](#421-explain--query-plans)
  - [4.2.2 SQL Query Optimization](#422-sql-query-optimization)
  - [4.2.3 MongoDB Query Optimization](#423-mongodb-query-optimization)
- [4.3 Caching Strategies](#43-caching-strategies)
  - [4.3.1 Application-Level Caching (Redis)](#431-application-level-caching-redis)
  - [4.3.2 Cache Invalidation](#432-cache-invalidation)
  - [4.3.3 Caching Patterns](#433-caching-patterns)
  - [4.3.4 Database-Level Caching](#434-database-level-caching)
  - [4.3.5 Materialized Views](#435-materialized-views)

---

## 4.1 Indexing Strategies

### Why Indexing Matters

**Without Index:**
```
Query: Find user with email "alice@example.com"
Database: Scans ALL 1,000,000 rows sequentially
Time: 5 seconds ❌
```

**With Index:**
```
Query: Find user with email "alice@example.com"
Database: Uses index to jump directly to row
Time: 5 milliseconds ✅
```

**Performance Improvement:** 1000x faster!

---

### Index Fundamentals

**What is an Index?**
An index is a data structure that improves data retrieval speed at the cost of additional writes and storage.

**Book Analogy:**
```
Without Index: Read entire book to find "MongoDB" → Slow
With Index: Look up "MongoDB" in index → Jump to page 342 → Fast
```

**Trade-offs:**
- ✅ **Faster reads** (queries)
- ❌ **Slower writes** (inserts, updates, deletes)
- ❌ **More storage** (index takes disk space)
- ❌ **Memory usage** (indexes loaded in RAM)

---

### 4.1.1 SQL Indexes (B-Tree, Hash, GIN, Partial, Composite)

PostgreSQL supports multiple index types, each optimized for different use cases.

---

#### 1. B-Tree Index (Default & Most Common)

**Structure:** Balanced tree that keeps data sorted.

**When to Use:**
- Equality comparisons (`=`)
- Range queries (`<`, `>`, `BETWEEN`)
- Pattern matching (`LIKE 'prefix%'`)
- Sorting (`ORDER BY`)

**Create B-Tree Index:**

```sql
-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Automatically used for queries like:
SELECT * FROM users WHERE email = 'alice@example.com';
SELECT * FROM users WHERE email > 'a@example.com';
SELECT * FROM users ORDER BY email;
```

**How B-Tree Works:**

```
B-Tree Structure (simplified):
                [M]
              /     \
          [D]         [T]
         /   \       /   \
      [A-C] [E-L] [N-S] [U-Z]
       
Search for "H":
1. Start at root: H < M, go left
2. D < H < T, go to [E-L] leaf
3. Find "H" in leaf node
Steps: 3 (vs scanning all records)
```

**Real-World Example: E-Commerce**

```sql
-- Products table
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  category VARCHAR(100),
  price DECIMAL(10,2),
  stock INTEGER,
  created_at TIMESTAMP
);

-- Create indexes for common queries
CREATE INDEX idx_products_category ON products(category);
CREATE INDEX idx_products_price ON products(price);
CREATE INDEX idx_products_created_at ON products(created_at);

-- Queries that use indexes:
-- 1. Filter by category
SELECT * FROM products WHERE category = 'Electronics';
-- Uses: idx_products_category

-- 2. Find products under $100
SELECT * FROM products WHERE price < 100;
-- Uses: idx_products_price

-- 3. Recent products
SELECT * FROM products ORDER BY created_at DESC LIMIT 10;
-- Uses: idx_products_created_at
```

---

#### 2. Hash Index

**Structure:** Hash table for fast equality lookups.

**When to Use:**
- ONLY equality comparisons (`=`)
- NOT for range queries
- NOT for sorting

**Create Hash Index:**

```sql
-- Hash index (PostgreSQL)
CREATE INDEX idx_users_username_hash ON users USING HASH (username);

-- ✅ Uses hash index
SELECT * FROM users WHERE username = 'alice';

-- ❌ Cannot use hash index
SELECT * FROM users WHERE username > 'alice';  -- Range query
SELECT * FROM users ORDER BY username;         -- Sorting
```

**Performance Comparison:**

```sql
-- B-Tree vs Hash for equality
-- Hash: O(1) average - slightly faster
-- B-Tree: O(log n) - very close in practice

-- For most cases, B-Tree is better (more versatile)
```

**When to Choose Hash:**
- Very large tables (100M+ rows)
- ONLY equality queries
- No need for range/sort
- Rare in practice

---

#### 3. GIN Index (Generalized Inverted Index)

**Purpose:** For indexing composite values (arrays, JSON, full-text search).

**When to Use:**
- Array columns (`@>`, `&&` operators)
- JSONB columns
- Full-text search
- Multiple values per row

**Create GIN Index:**

```sql
-- Array column
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255),
  tags TEXT[]  -- Array of tags
);

-- GIN index on array
CREATE INDEX idx_posts_tags ON posts USING GIN (tags);

-- ✅ Uses GIN index
SELECT * FROM posts WHERE tags @> ARRAY['postgresql'];  -- Contains
SELECT * FROM posts WHERE tags && ARRAY['sql', 'database'];  -- Overlap

-- JSONB column
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  data JSONB
);

-- GIN index on JSONB
CREATE INDEX idx_users_data ON users USING GIN (data);

-- ✅ Uses GIN index
SELECT * FROM users WHERE data @> '{"city": "New York"}';
SELECT * FROM users WHERE data ? 'email';  -- Has key
```

**Real-World Example: Blog Platform**

```sql
CREATE TABLE articles (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255),
  content TEXT,
  tags TEXT[],
  metadata JSONB,
  created_at TIMESTAMP
);

-- GIN indexes
CREATE INDEX idx_articles_tags ON articles USING GIN (tags);
CREATE INDEX idx_articles_metadata ON articles USING GIN (metadata);

-- Full-text search GIN
CREATE INDEX idx_articles_search ON articles 
  USING GIN (to_tsvector('english', title || ' ' || content));

-- Queries:
-- 1. Find articles with specific tag
SELECT * FROM articles WHERE tags @> ARRAY['postgresql'];

-- 2. Find articles with metadata
SELECT * FROM articles WHERE metadata @> '{"featured": true}';

-- 3. Full-text search
SELECT * FROM articles 
WHERE to_tsvector('english', title || ' ' || content) @@ to_tsquery('database & optimization');
```

---

#### 4. GiST Index (Generalized Search Tree)

**Purpose:** For geometric data types and full-text search.

**When to Use:**
- Geometric data (points, polygons)
- Range types
- Full-text search (alternative to GIN)

```sql
-- Geographic data
CREATE TABLE locations (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  coordinates POINT
);

-- GiST index for geometric queries
CREATE INDEX idx_locations_coords ON locations USING GIST (coordinates);

-- Find nearby locations
SELECT * FROM locations 
WHERE coordinates <-> POINT(40.7128, -74.0060) < 10;  -- Within 10 units
```

---

#### 5. Partial Index

**Purpose:** Index only a subset of rows that match a condition.

**When to Use:**
- Frequently query specific subset
- Reduce index size
- Improve write performance

**Create Partial Index:**

```sql
-- Index only active users
CREATE INDEX idx_users_active_email ON users(email) 
WHERE status = 'active';

-- Index only expensive products
CREATE INDEX idx_products_expensive_price ON products(price)
WHERE price > 1000;

-- Index only recent orders
CREATE INDEX idx_orders_recent ON orders(created_at)
WHERE created_at > NOW() - INTERVAL '30 days';

-- ✅ Uses partial index
SELECT * FROM users WHERE email = 'alice@example.com' AND status = 'active';

-- ❌ Cannot use partial index (status not 'active')
SELECT * FROM users WHERE email = 'alice@example.com' AND status = 'inactive';
```

**Benefits:**

```sql
-- Without partial index
CREATE INDEX idx_users_email ON users(email);
-- Size: 100MB (indexes all 1M users)

-- With partial index (only 10% active)
CREATE INDEX idx_users_active_email ON users(email) WHERE status = 'active';
-- Size: 10MB (indexes only 100K active users)
-- Faster writes (90% of writes don't update index)
```

**Real-World Example: Order System**

```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER,
  total DECIMAL(10,2),
  status VARCHAR(50),
  created_at TIMESTAMP
);

-- Partial indexes for common queries
CREATE INDEX idx_orders_pending ON orders(customer_id, created_at)
WHERE status = 'pending';

CREATE INDEX idx_orders_processing ON orders(customer_id, created_at)
WHERE status = 'processing';

-- Most queries filter by status
SELECT * FROM orders 
WHERE customer_id = 123 AND status = 'pending'
ORDER BY created_at DESC;
-- Uses: idx_orders_pending (small, fast)
```

---

#### 6. Composite Index (Multi-Column)

**Purpose:** Index multiple columns together.

**Column Order Matters!**

```sql
-- Composite index
CREATE INDEX idx_users_city_age ON users(city, age);

-- ✅ Uses index (prefix match)
SELECT * FROM users WHERE city = 'New York';
SELECT * FROM users WHERE city = 'New York' AND age > 25;
SELECT * FROM users WHERE city = 'New York' AND age = 30;

-- ❌ Cannot use index efficiently (no prefix)
SELECT * FROM users WHERE age > 25;  -- Missing city (first column)
```

**Index Column Order Rule:**

**ESR (Equality, Sort, Range):**
1. **Equality** columns first (`=`)
2. **Sort** columns next (`ORDER BY`)
3. **Range** columns last (`<`, `>`, `BETWEEN`)

```sql
-- Query with equality, sort, and range
SELECT * FROM products
WHERE category = 'Electronics'    -- Equality
  AND price < 1000                -- Range
ORDER BY name;                    -- Sort

-- ✅ Optimal index (E-S-R order)
CREATE INDEX idx_products_opt ON products(
  category,   -- E: Equality
  name,       -- S: Sort
  price       -- R: Range
);

-- ❌ Suboptimal index orders
CREATE INDEX idx_bad_1 ON products(price, category, name);  -- Wrong order
CREATE INDEX idx_bad_2 ON products(name, category, price);  -- Wrong order
```

**Composite Index Examples:**

```sql
-- E-commerce: Category + Price
CREATE INDEX idx_products_cat_price ON products(category, price);

-- ✅ Uses index
SELECT * FROM products WHERE category = 'Electronics' AND price < 500;
SELECT * FROM products WHERE category = 'Electronics' ORDER BY price;

-- User search: Last name + First name
CREATE INDEX idx_users_name ON users(last_name, first_name);

-- ✅ Uses index
SELECT * FROM users WHERE last_name = 'Smith';
SELECT * FROM users WHERE last_name = 'Smith' AND first_name = 'John';

-- ❌ Cannot use index efficiently
SELECT * FROM users WHERE first_name = 'John';  -- Missing last_name prefix
```

**Covering Index (Include Columns):**

```sql
-- Include non-indexed columns for covering queries
CREATE INDEX idx_products_category_inc ON products(category) 
INCLUDE (name, price);

-- Covering query: All data from index (no table access needed)
SELECT name, price FROM products WHERE category = 'Electronics';
-- PostgreSQL doesn't even touch the table!
```

---

### Practice Questions: SQL Indexes

#### Fill in the Blanks

1. The ________ index type is the default in PostgreSQL and works for equality, range, and sorting.
2. ________ indexes are best for indexing array and JSONB columns.
3. A ________ index only indexes rows that match a specific condition.
4. The ________ rule states optimal column order: Equality, Sort, Range.
5. ________ indexes use hash tables and only support equality comparisons.

<details>
<summary><strong>View Answers</strong></summary>

1. B-Tree
2. GIN
3. partial
4. ESR
5. Hash

</details>

---

#### True/False

1. Hash indexes can be used for range queries.
2. Composite index on (A, B) can efficiently query B alone.
3. GIN indexes are good for full-text search.
4. Partial indexes can reduce index size and improve write performance.
5. More indexes always mean better query performance.

<details>
<summary><strong>View Answers</strong></summary>

1. FALSE (Hash indexes only support equality)
2. FALSE (needs A or A+B prefix)
3. TRUE
4. TRUE
5. FALSE (too many indexes slow down writes and use memory)

</details>

---

#### Coding Challenges

**Challenge 1: Create Optimal Index**

Given this query, create the optimal index:

```sql
SELECT id, name, price 
FROM products 
WHERE category = 'Electronics' 
  AND stock > 0 
  AND price BETWEEN 100 AND 500
ORDER BY price;
```

<details>
<summary><strong>View Solution</strong></summary>

```sql
-- ESR Rule: Equality, Sort, Range
CREATE INDEX idx_products_optimal ON products(
  category,   -- Equality
  price,      -- Sort (also used in range, but sort takes priority)
  stock       -- Range
) INCLUDE (id, name);  -- Covering index

-- Alternative without INCLUDE:
CREATE INDEX idx_products_optimal ON products(
  category,
  price,
  stock
);
-- Note: 'price' handles both ORDER BY and range filter
```

**Explanation:**
- `category = 'Electronics'` → Equality (first)
- `ORDER BY price` → Sort (second)
- `price BETWEEN` → Range (handled by sort column)
- `stock > 0` → Range (last)
- `INCLUDE (id, name)` → Covering index (no table access needed)

</details>

---

**Challenge 2: Partial Index Decision**

You have an orders table where 95% of orders are 'completed' and only 5% are 'pending' or 'processing'. Most queries filter for non-completed orders. Should you use a partial index? If yes, create it.

<details>
<summary><strong>View Solution</strong></summary>

```sql
-- ✅ Yes! Use partial index for non-completed orders (5%)
CREATE INDEX idx_orders_active ON orders(customer_id, created_at)
WHERE status IN ('pending', 'processing');

-- Benefits:
-- 1. Index size: 95% smaller (only 5% of rows)
-- 2. Faster writes: 95% of order updates don't touch index
-- 3. Faster queries: Smaller index = faster scans

-- Queries that benefit:
SELECT * FROM orders 
WHERE customer_id = 123 
  AND status = 'pending'
ORDER BY created_at DESC;
```

**Why not index completed orders?**
- They're rarely queried (archive queries)
- Would create a huge index (95% of data)
- Slows down all writes

</details>

---

**Challenge 3: GIN vs B-Tree**

You have a products table with a `tags` TEXT[] column. When should you use GIN vs B-Tree?

<details>
<summary><strong>View Solution</strong></summary>

```sql
-- ✅ Use GIN for array containment queries
CREATE INDEX idx_products_tags_gin ON products USING GIN (tags);

-- Queries that need GIN:
SELECT * FROM products WHERE tags @> ARRAY['electronics'];  -- Contains
SELECT * FROM products WHERE tags && ARRAY['sale', 'new'];  -- Overlaps
SELECT * FROM products WHERE tags @> ARRAY['laptop', 'gaming'];

-- ❌ B-Tree doesn't work well for these queries
CREATE INDEX idx_products_tags_btree ON products(tags);
-- Can only check exact array equality (rarely useful)
```

**Decision:**
- **GIN**: Array containment, overlap, JSON queries → Use GIN
- **B-Tree**: Single value comparison, sorting → Use B-Tree

</details>

---

## 4.1.2 NoSQL Indexes (MongoDB)

MongoDB indexing is similar to SQL but with some unique considerations for document-oriented data.

---

### MongoDB Index Types

#### 1. Single Field Index

**Most basic index type.**

```javascript
// Create ascending index
db.users.createIndex({ email: 1 });

// Create descending index
db.users.createIndex({ age: -1 });

// Queries that use the index
db.users.find({ email: "alice@example.com" });
db.users.find({ age: { $gt: 25 } });
db.users.find().sort({ email: 1 });
```

**Direction (1 vs -1):**
```javascript
// For single field, direction doesn't matter much
db.users.createIndex({ email: 1 });   // Same as...
db.users.createIndex({ email: -1 });  // ...for single field queries

// Direction matters for compound indexes and sorting
```

---

#### 2. Compound Index

**Index on multiple fields - ORDER MATTERS!**

```javascript
// Create compound index
db.products.createIndex({ category: 1, price: -1 });

// ✅ Uses index (prefix match)
db.products.find({ category: "Electronics" });
db.products.find({ category: "Electronics", price: { $lt: 1000 } });
db.products.find({ category: "Electronics" }).sort({ price: -1 });

// ❌ Cannot use index efficiently
db.products.find({ price: { $lt: 1000 } });  // Missing 'category' prefix
```

**ESR Rule (Same as SQL):**

```javascript
// Query with equality, sort, and range
db.orders.find({
  status: "completed",     // Equality
  userId: ObjectId("..."), // Equality
  total: { $gte: 100 }     // Range
}).sort({ createdAt: -1 }); // Sort

// ✅ Optimal index (E-S-R)
db.orders.createIndex({
  status: 1,      // Equality
  userId: 1,      // Equality
  createdAt: -1,  // Sort
  total: 1        // Range
});
```

---

#### 3. Multikey Index (Arrays)

**Automatically created when indexing array fields.**

```javascript
// Document with array
{
  _id: ObjectId("..."),
  title: "MongoDB Guide",
  tags: ["mongodb", "nosql", "database"]
}

// Create index on array field (multikey index)
db.posts.createIndex({ tags: 1 });

// ✅ Index is used for array queries
db.posts.find({ tags: "mongodb" });
db.posts.find({ tags: { $in: ["mongodb", "nosql"] } });
db.posts.find({ tags: { $all: ["mongodb", "database"] } });

// Each array value is indexed separately
// Document with 3 tags creates 3 index entries
```

**Important Multikey Limitation:**

```javascript
// ❌ Cannot create compound index with multiple array fields
db.posts.createIndex({ tags: 1, categories: 1 });
// Error: cannot index parallel arrays

// ✅ Can index one array + one non-array
db.posts.createIndex({ tags: 1, createdAt: 1 });  // OK
```

---

#### 4. Text Index (Full-Text Search)

**For searching text content.**

```javascript
// Create text index on multiple fields
db.articles.createIndex({
  title: "text",
  content: "text",
  tags: "text"
});

// Search for text
db.articles.find({
  $text: { $search: "mongodb database" }
});

// Search with score (relevance)
db.articles.find(
  { $text: { $search: "mongodb performance" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } });

// Text search operators
db.articles.find({ $text: { $search: "mongodb -sql" } });  // Exclude "sql"
db.articles.find({ $text: { $search: "\"exact phrase\"" } });  // Exact match
```

**Limitations:**
- Only one text index per collection
- Cannot use with other sort fields
- Language-specific (default: English)

---

#### 5. Geospatial Index (2dsphere)

**For location-based queries.**

```javascript
// Document with GeoJSON location
{
  _id: ObjectId("..."),
  name: "Central Park",
  location: {
    type: "Point",
    coordinates: [-73.9654, 40.7829]  // [longitude, latitude]
  }
}

// Create 2dsphere index
db.places.createIndex({ location: "2dsphere" });

// Find places near coordinates
db.places.find({
  location: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [-73.9712, 40.7831]
      },
      $maxDistance: 1000  // meters
    }
  }
});

// Find places within polygon
db.places.find({
  location: {
    $geoWithin: {
      $geometry: {
        type: "Polygon",
        coordinates: [[
          [-74.0, 40.7],
          [-73.9, 40.7],
          [-73.9, 40.8],
          [-74.0, 40.8],
          [-74.0, 40.7]
        ]]
      }
    }
  }
});
```

---

#### 6. Unique Index

**Ensures field values are unique.**

```javascript
// Create unique index
db.users.createIndex({ email: 1 }, { unique: true });

// ✅ First insert succeeds
db.users.insertOne({ email: "alice@example.com", name: "Alice" });

// ❌ Duplicate insert fails
db.users.insertOne({ email: "alice@example.com", name: "Alice2" });
// Error: E11000 duplicate key error

// Unique compound index
db.users.createIndex(
  { email: 1, username: 1 }, 
  { unique: true }
);
// Combination of email + username must be unique
```

---

#### 7. Partial Index

**Index only documents matching a filter.**

```javascript
// Index only active users
db.users.createIndex(
  { email: 1 },
  { 
    partialFilterExpression: { 
      status: "active" 
    } 
  }
);

// ✅ Uses partial index
db.users.find({ email: "alice@example.com", status: "active" });

// ❌ Cannot use partial index
db.users.find({ email: "alice@example.com" });  // Missing status filter
db.users.find({ email: "alice@example.com", status: "inactive" });
```

**Real-World Example: E-Commerce**

```javascript
// Most queries filter for in-stock products
db.products.createIndex(
  { category: 1, price: 1 },
  { 
    partialFilterExpression: { 
      stock: { $gt: 0 } 
    } 
  }
);

// Benefits:
// 1. Smaller index (only in-stock products)
// 2. Faster writes (out-of-stock updates don't touch index)

// Query:
db.products.find({
  category: "Electronics",
  price: { $lt: 1000 },
  stock: { $gt: 0 }
}).sort({ price: 1 });
```

---

#### 8. TTL Index (Time-To-Live)

**Automatically delete documents after a time period.**

```javascript
// Auto-delete sessions after 24 hours
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 86400 }  // 24 hours
);

// Document is deleted when:
// createdAt + expireAfterSeconds < current time

// Use cases:
// - Session storage
// - Temporary data
// - Log files
// - Cache entries
```

---

#### 9. Sparse Index

**Index only documents that have the indexed field.**

```javascript
// Sparse index (only documents with phoneNumber)
db.users.createIndex(
  { phoneNumber: 1 },
  { sparse: true }
);

// Documents:
{ _id: 1, name: "Alice", phoneNumber: "555-1234" }  // Indexed
{ _id: 2, name: "Bob" }                             // NOT indexed (no phoneNumber)
{ _id: 3, name: "Carol", phoneNumber: "555-5678" }  // Indexed

// ✅ Index is used
db.users.find({ phoneNumber: "555-1234" });

// Note: Sparse indexes skip documents without the field
// May return incomplete results for some queries
```

---

#### 10. Case-Insensitive Index (Collation)

**For case-insensitive queries.**

```javascript
// Create index with collation
db.users.createIndex(
  { username: 1 },
  { 
    collation: { 
      locale: "en", 
      strength: 2  // Case-insensitive
    } 
  }
);

// Query must use same collation
db.users.find(
  { username: "ALICE" }
).collation({ locale: "en", strength: 2 });

// Finds: "alice", "Alice", "ALICE", "AlIcE"
```

---

### MongoDB Index Management

**List Indexes:**

```javascript
// List all indexes
db.users.getIndexes();

// Result:
[
  { v: 2, key: { _id: 1 }, name: "_id_" },  // Default
  { v: 2, key: { email: 1 }, name: "email_1" },
  { v: 2, key: { city: 1, age: 1 }, name: "city_1_age_1" }
]
```

**Create Index:**

```javascript
// Basic
db.collection.createIndex({ field: 1 });

// With options
db.collection.createIndex(
  { field: 1 },
  {
    unique: true,
    sparse: true,
    name: "custom_index_name",
    background: true  // Don't block writes (deprecated in 4.2+)
  }
);
```

**Drop Index:**

```javascript
// Drop by name
db.users.dropIndex("email_1");

// Drop by keys
db.users.dropIndex({ email: 1 });

// Drop all indexes (except _id)
db.users.dropIndexes();
```

**Index Statistics:**

```javascript
// Get index usage stats
db.users.aggregate([{ $indexStats: {} }]);

// Result shows:
// - Index name
// - Operations using index
// - Last accessed time
```

---

### Practice Questions: MongoDB Indexes

#### Fill in the Blanks

1. A ________ index is automatically created when indexing an array field.
2. MongoDB allows only ________ text index per collection.
3. The ________ index type automatically deletes documents after a specified time.
4. Use ________ indexes to index only documents matching a filter expression.
5. A ________ index only includes documents that have the indexed field.

<details>
<summary><strong>View Answers</strong></summary>

1. multikey
2. one
3. TTL
4. partial
5. sparse

</details>

---

#### Coding Challenges

**Challenge 1: Optimal Compound Index**

Given this query, create the optimal MongoDB index:

```javascript
db.orders.find({
  customerId: ObjectId("..."),
  status: "completed",
  total: { $gte: 100 }
}).sort({ createdAt: -1 });
```

<details>
<summary><strong>View Solution</strong></summary>

```javascript
// ESR Rule: Equality, Sort, Range
db.orders.createIndex({
  customerId: 1,   // Equality
  status: 1,       // Equality
  createdAt: -1,   // Sort
  total: 1         // Range
});
```

</details>

---

**Challenge 2: TTL Index for Cache**

Create a cache collection that automatically deletes entries after 1 hour.

<details>
<summary><strong>View Solution</strong></summary>

```javascript
// Create collection
db.createCollection("cache");

// Create TTL index
db.cache.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 3600 }  // 1 hour
);

// Insert cache entry
db.cache.insertOne({
  key: "user:123",
  value: { name: "Alice", email: "alice@example.com" },
  createdAt: new Date()
});

// Automatically deleted after 1 hour
```

</details>

---

## 4.1.3 Index Selection & Maintenance

### How to Choose the Right Index

#### Step 1: Analyze Query Patterns

**Identify Common Queries:**

```sql
-- PostgreSQL: Top 10 slow queries
SELECT 
  query,
  calls,
  total_time,
  mean_time,
  rows
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;
```

```javascript
// MongoDB: Enable profiling
db.setProfilingLevel(2);  // Log all queries

// View slow queries
db.system.profile.find({
  millis: { $gt: 100 }  // Queries > 100ms
}).sort({ ts: -1 }).limit(10);
```

---

#### Step 2: Identify Index Candidates

**Questions to Ask:**

1. **Which columns are in WHERE clauses?**
   ```sql
   SELECT * FROM users WHERE email = '...' AND status = 'active';
   -- Candidates: email, status
   ```

2. **Which columns are in ORDER BY?**
   ```sql
   SELECT * FROM posts ORDER BY created_at DESC;
   -- Candidate: created_at
   ```

3. **Which columns are in JOIN conditions?**
   ```sql
   SELECT * FROM orders o JOIN users u ON o.user_id = u.id;
   -- Candidates: orders.user_id, users.id
   ```

4. **What's the query selectivity?**
   ```sql
   -- High selectivity (good for index)
   SELECT * FROM users WHERE email = 'alice@example.com';  -- Returns 1 row
   
   -- Low selectivity (bad for index)
   SELECT * FROM users WHERE gender = 'M';  -- Returns 50% of rows
   ```

---

#### Step 3: Apply Index Selection Rules

**Rule 1: Index High-Selectivity Columns**

```sql
-- ✅ Good: Unique or near-unique values
CREATE INDEX idx_users_email ON users(email);        -- Unique
CREATE INDEX idx_orders_id ON orders(order_number);  -- Unique

-- ❌ Bad: Low selectivity
CREATE INDEX idx_users_gender ON users(gender);      -- Only 2-3 values
CREATE INDEX idx_posts_published ON posts(published); -- Boolean (2 values)

-- Exception: Use partial index for low-selectivity
CREATE INDEX idx_posts_unpublished ON posts(created_at)
WHERE published = false;  -- Only unpublished posts (small subset)
```

**Rule 2: Index Foreign Keys**

```sql
-- Always index foreign key columns
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);

-- Speeds up:
-- 1. JOINs
-- 2. Referential integrity checks
-- 3. CASCADE deletes
```

**Rule 3: Use Covering Indexes**

```sql
-- Query needs: category, name, price
SELECT name, price 
FROM products 
WHERE category = 'Electronics';

-- ✅ Covering index (includes all needed columns)
CREATE INDEX idx_products_covering ON products(category) 
INCLUDE (name, price);

-- PostgreSQL never touches the table!
```

**Rule 4: Avoid Over-Indexing**

```sql
-- ❌ Too many indexes (slow writes)
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_phone ON users(phone);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_created_at ON users(created_at);
CREATE INDEX idx_users_email_username ON users(email, username);
CREATE INDEX idx_users_username_email ON users(username, email);
-- 7 indexes on one table! Writes will be SLOW

-- ✅ Consolidate to essential indexes
CREATE INDEX idx_users_email ON users(email);  -- Unique, frequently queried
CREATE INDEX idx_users_username ON users(username);  -- Unique
CREATE INDEX idx_users_created_at ON users(created_at);  -- For sorting
-- 3 indexes - much better
```

---

### Index Maintenance

#### 1. Monitor Index Usage

**PostgreSQL:**

```sql
-- Find unused indexes
SELECT 
  schemaname,
  tablename,
  indexname,
  idx_scan,
  idx_tup_read,
  idx_tup_fetch
FROM pg_stat_user_indexes
WHERE idx_scan = 0  -- Never used
  AND indexrelname NOT LIKE 'pk_%'  -- Exclude primary keys
ORDER BY pg_relation_size(indexrelid) DESC;

-- Drop unused indexes
DROP INDEX IF EXISTS idx_unused_index;
```

**MongoDB:**

```javascript
// Index usage statistics
db.collection.aggregate([{ $indexStats: {} }]).pretty();

// Find unused indexes (never accessed)
db.collection.aggregate([
  { $indexStats: {} },
  { 
    $match: { 
      "accesses.ops": 0 
    } 
  }
]);
```

---

#### 2. Rebuild Fragmented Indexes

**PostgreSQL:**

```sql
-- Check index bloat
SELECT 
  schemaname,
  tablename,
  indexname,
  pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
  idx_scan,
  idx_tup_read
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC;

-- Rebuild index (removes bloat)
REINDEX INDEX CONCURRENTLY idx_users_email;

-- Rebuild all indexes on table
REINDEX TABLE CONCURRENTLY users;
```

**MongoDB:**

```javascript
// Rebuild all indexes
db.collection.reIndex();

// MongoDB 4.4+: Background rebuild
db.collection.createIndex(
  { field: 1 },
  { background: true }  // Deprecated, but non-blocking
);
```

---

#### 3. Update Statistics

**PostgreSQL:**

```sql
-- Update table statistics (helps query planner)
ANALYZE users;

-- Update all tables
ANALYZE;

-- Auto-vacuum (runs automatically, but can trigger manually)
VACUUM ANALYZE users;
```

**MongoDB:**

```javascript
// Update collection statistics
db.collection.stats();

// Compact collection (reclaim space)
db.runCommand({ compact: 'collection' });
```

---

### Index Anti-Patterns

**Anti-Pattern 1: Indexing Low-Selectivity Columns**

```sql
-- ❌ Bad: Gender has only 2-3 values
CREATE INDEX idx_users_gender ON users(gender);

-- Database scans 50% of table anyway
-- Index overhead not worth it

-- ✅ Better: Use partial index if needed
CREATE INDEX idx_users_male ON users(created_at)
WHERE gender = 'M' AND status = 'active';
```

**Anti-Pattern 2: Duplicate Indexes**

```sql
-- ❌ Redundant indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_email_status ON users(email, status);

-- Second index covers first (has email as prefix)

-- ✅ Drop redundant index
DROP INDEX idx_users_email;
-- Keep only idx_users_email_status
```

**Anti-Pattern 3: Wrong Column Order**

```sql
-- Query filters by category, sorts by price
SELECT * FROM products 
WHERE category = 'Electronics'
ORDER BY price;

-- ❌ Wrong order
CREATE INDEX idx_products_wrong ON products(price, category);

-- ✅ Correct order (E-S-R)
CREATE INDEX idx_products_correct ON products(category, price);
```

**Anti-Pattern 4: Function-Based Queries Without Function Index**

```sql
-- Query uses function
SELECT * FROM users WHERE LOWER(email) = 'alice@example.com';

-- ❌ Regular index won't be used
CREATE INDEX idx_users_email ON users(email);

-- ✅ Function-based index
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
```

---

### Practice Questions: Index Selection & Maintenance

#### Multiple Choice

**Q1: Which column is the BEST candidate for indexing?**

A) Gender (2 values: M/F)
B) Published (Boolean: true/false)
C) Email (Unique for each user)
D) Department (10 values for 100K employees)

<details>
<summary><strong>View Answer</strong></summary>

**Answer: C - Email (Unique for each user)**

Explanation: High selectivity (unique values) means the index can quickly narrow down to 1 row. Low selectivity (gender, boolean) means index scans many rows anyway, making the index inefficient.

</details>

---

**Q2: You have indexes on (A) and (A, B). Which should you drop?**

A) Drop index on (A)
B) Drop index on (A, B)
C) Keep both
D) Drop both

<details>
<summary><strong>View Answer</strong></summary>

**Answer: A - Drop index on (A)**

Explanation: Index on (A, B) can handle queries on just A (prefix match), making the single-column index redundant. Always keep the composite index.

</details>

---

## 4.2 Query Optimization

### Why Query Optimization Matters

**Before Optimization:**
```sql
SELECT * FROM orders WHERE EXTRACT(YEAR FROM created_at) = 2024;
-- Execution time: 8 seconds
-- Rows scanned: 10,000,000
```

**After Optimization:**
```sql
SELECT * FROM orders 
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
-- Execution time: 0.05 seconds
-- Rows scanned: 50,000
-- Performance: 160x faster! 🚀
```

---

## 4.2.1 EXPLAIN & Query Plans

### Understanding EXPLAIN

**EXPLAIN** shows how the database will execute a query.

---

### PostgreSQL EXPLAIN

#### Basic EXPLAIN

```sql
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';
```

**Output:**
```
Seq Scan on users  (cost=0.00..1808.00 rows=1 width=68)
  Filter: ((email)::text = 'alice@example.com'::text)
```

**Key Metrics:**
- **Seq Scan** = Sequential scan (reads every row) ❌
- **cost** = Estimated cost (0.00 startup, 1808.00 total)
- **rows** = Estimated rows returned
- **width** = Average row size in bytes

---

#### EXPLAIN ANALYZE (Actual Execution)

```sql
EXPLAIN ANALYZE 
SELECT * FROM users WHERE email = 'alice@example.com';
```

**Output:**
```
Seq Scan on users  (cost=0.00..1808.00 rows=1 width=68) 
                   (actual time=0.015..45.238 rows=1 loops=1)
  Filter: ((email)::text = 'alice@example.com'::text)
  Rows Removed by Filter: 999999
Planning Time: 0.123 ms
Execution Time: 45.267 ms
```

**Additional Info:**
- **actual time** = Real execution time (start..end)
- **rows** = Actual rows returned
- **Rows Removed by Filter** = Rows scanned but filtered out
- **Planning Time** = Time to create query plan
- **Execution Time** = Actual query execution time

---

#### EXPLAIN with Index

```sql
-- Create index
CREATE INDEX idx_users_email ON users(email);

-- Re-run EXPLAIN
EXPLAIN ANALYZE 
SELECT * FROM users WHERE email = 'alice@example.com';
```

**Output:**
```
Index Scan using idx_users_email on users  
  (cost=0.42..8.44 rows=1 width=68) 
  (actual time=0.025..0.027 rows=1 loops=1)
  Index Cond: ((email)::text = 'alice@example.com'::text)
Planning Time: 0.234 ms
Execution Time: 0.052 ms
```

**Improvement:**
- **Seq Scan** → **Index Scan** ✅
- **45.267 ms** → **0.052 ms** (870x faster!)
- **Rows Removed: 999999** → **0** (only scanned 1 row)

---

#### Understanding Execution Nodes

**1. Seq Scan (Sequential Scan)**
```sql
EXPLAIN SELECT * FROM users WHERE age > 25;
-- Seq Scan on users (bad if table is large)
```
- Reads every row in table
- ❌ Slow for large tables
- ✅ OK for small tables or no index

**2. Index Scan**
```sql
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';
-- Index Scan using idx_users_email (good!)
```
- Uses index to find rows
- ✅ Fast and efficient

**3. Index Only Scan**
```sql
-- Covering index
CREATE INDEX idx_users_email_inc ON users(email) INCLUDE (name);

EXPLAIN SELECT name FROM users WHERE email = 'alice@example.com';
-- Index Only Scan using idx_users_email_inc (best!)
```
- All data from index (no table access)
- ✅ Fastest possible

**4. Bitmap Index Scan**
```sql
EXPLAIN SELECT * FROM users WHERE age > 25 AND age < 30;
-- Bitmap Index Scan on idx_users_age
```
- Builds bitmap of matching rows
- ✅ Good for medium selectivity

**5. Nested Loop**
```sql
EXPLAIN SELECT * FROM orders o JOIN users u ON o.user_id = u.id;
-- Nested Loop
```
- For each row in outer table, lookup in inner table
- ✅ Good for small datasets
- ❌ Slow for large datasets

**6. Hash Join**
```sql
EXPLAIN SELECT * FROM orders o JOIN users u ON o.user_id = u.id;
-- Hash Join
```
- Builds hash table of one side, probes with other
- ✅ Good for medium-large datasets
- Memory intensive

**7. Merge Join**
```sql
EXPLAIN SELECT * FROM orders o JOIN users u ON o.user_id = u.id;
-- Merge Join
```
- Both sides sorted, then merged
- ✅ Good for large pre-sorted datasets

---

### MongoDB EXPLAIN

#### Basic Explain

```javascript
db.users.find({ email: "alice@example.com" }).explain("executionStats");
```

**Output:**
```json
{
  "executionStats": {
    "executionSuccess": true,
    "nReturned": 1,
    "executionTimeMillis": 156,
    "totalKeysExamined": 0,
    "totalDocsExamined": 1000000,
    "executionStages": {
      "stage": "COLLSCAN",
      "nReturned": 1,
      "executionTimeMillisEstimate": 142,
      "works": 1000002,
      "advanced": 1,
      "needTime": 1000000,
      "needYield": 0,
      "direction": "forward",
      "docsExamined": 1000000
    }
  }
}
```

**Key Metrics:**
- **COLLSCAN** = Collection scan (no index) ❌
- **executionTimeMillis** = Query execution time
- **totalDocsExamined** = Documents scanned
- **nReturned** = Documents returned

---

#### Explain with Index

```javascript
// Create index
db.users.createIndex({ email: 1 });

// Re-run explain
db.users.find({ email: "alice@example.com" }).explain("executionStats");
```

**Output:**
```json
{
  "executionStats": {
    "executionSuccess": true,
    "nReturned": 1,
    "executionTimeMillis": 2,
    "totalKeysExamined": 1,
    "totalDocsExamined": 1,
    "executionStages": {
      "stage": "FETCH",
      "nReturned": 1,
      "executionTimeMillisEstimate": 0,
      "inputStage": {
        "stage": "IXSCAN",
        "keyPattern": { "email": 1 },
        "indexName": "email_1",
        "isMultiKey": false,
        "direction": "forward",
        "indexBounds": {
          "email": [
            "[\"alice@example.com\", \"alice@example.com\"]"
          ]
        }
      }
    }
  }
}
```

**Improvement:**
- **COLLSCAN** → **IXSCAN** (Index Scan) ✅
- **156 ms** → **2 ms** (78x faster!)
- **totalDocsExamined: 1000000** → **1**

---

#### MongoDB Execution Stages

**1. COLLSCAN (Collection Scan)**
```javascript
db.users.find({ age: { $gt: 25 } }).explain("executionStats");
// stage: "COLLSCAN"
```
- No index used
- ❌ Scans every document

**2. IXSCAN (Index Scan)**
```javascript
db.users.find({ email: "alice@example.com" }).explain("executionStats");
// stage: "IXSCAN"
```
- Uses index
- ✅ Fast lookup

**3. FETCH**
```javascript
// IXSCAN → FETCH
// Uses index, then fetches full document
```
- Index scan + document retrieval
- ✅ Good (using index)

**4. SORT**
```javascript
db.users.find({}).sort({ age: 1 }).explain("executionStats");
// stage: "SORT"
```
- In-memory sort (no index)
- ❌ Slow for large results
- ✅ Better: Create index for sorting

**5. COVERED (Covered Query)**
```javascript
// All fields in index
db.users.find(
  { email: "alice@example.com" },
  { _id: 0, email: 1, name: 1 }
).hint({ email: 1, name: 1 }).explain("executionStats");
// No FETCH stage - all data from index
```
- All data from index
- ✅ Fastest possible

---

### Explain Modes Comparison

| Feature | PostgreSQL | MongoDB |
|---------|-----------|---------|
| **Show Plan** | `EXPLAIN` | `.explain()` |
| **Execute & Show** | `EXPLAIN ANALYZE` | `.explain("executionStats")` |
| **Verbose** | `EXPLAIN (VERBOSE, ...)` | `.explain("allPlansExecution")` |
| **Index Scan** | Index Scan | IXSCAN |
| **Table Scan** | Seq Scan | COLLSCAN |
| **Best** | Index Only Scan | Covered Query |

---

### Practice Questions: EXPLAIN

#### Fill in the Blanks

1. In PostgreSQL, ________ scans every row in the table without using an index.
2. MongoDB's ________ stage indicates a collection scan (no index).
3. Use ________ in PostgreSQL to see actual execution times, not just estimates.
4. An ________ ________ ________ in PostgreSQL gets all data from the index without accessing the table.
5. In MongoDB explain output, ________ shows how many documents were examined.

<details>
<summary><strong>View Answers</strong></summary>

1. Seq Scan (Sequential Scan)
2. COLLSCAN
3. EXPLAIN ANALYZE
4. Index Only Scan
5. totalDocsExamined

</details>

---

#### Coding Challenge: Diagnose Slow Query

**Scenario:** This query is slow. Use EXPLAIN to diagnose and fix.

```sql
-- PostgreSQL
SELECT * FROM orders 
WHERE EXTRACT(YEAR FROM created_at) = 2024
ORDER BY total DESC
LIMIT 10;

-- Current execution: 8 seconds
```

<details>
<summary><strong>View Solution</strong></summary>

**Step 1: Run EXPLAIN**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders 
WHERE EXTRACT(YEAR FROM created_at) = 2024
ORDER BY total DESC
LIMIT 10;

-- Output shows:
-- Seq Scan (no index)
-- Sort (in-memory sort)
```

**Step 2: Identify Problems**
1. ❌ `EXTRACT(YEAR FROM created_at)` prevents index usage (function on column)
2. ❌ No index on `total` for sorting
3. ❌ Scanning all rows

**Step 3: Fix Query**
```sql
-- Rewrite to allow index usage
SELECT * FROM orders 
WHERE created_at >= '2024-01-01' 
  AND created_at < '2025-01-01'
ORDER BY total DESC
LIMIT 10;

-- Create index
CREATE INDEX idx_orders_2024 ON orders(created_at, total DESC)
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```

**Step 4: Verify Improvement**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders 
WHERE created_at >= '2024-01-01' 
  AND created_at < '2025-01-01'
ORDER BY total DESC
LIMIT 10;

-- Output shows:
-- Index Scan using idx_orders_2024
-- No sort needed (index provides order)
-- Execution: 0.05 seconds (160x faster!)
```

</details>

---

## 4.2.2 SQL Query Optimization

### Optimization Techniques

---

#### 1. Avoid Functions on Indexed Columns

**❌ Bad: Function prevents index usage**
```sql
-- Cannot use index on created_at
SELECT * FROM orders 
WHERE EXTRACT(YEAR FROM created_at) = 2024;

-- Cannot use index on email
SELECT * FROM users 
WHERE LOWER(email) = 'alice@example.com';

-- Cannot use index on price
SELECT * FROM products 
WHERE price * 1.1 > 100;  -- Price with tax
```

**✅ Good: Rewrite to use index**
```sql
-- Use range instead
SELECT * FROM orders 
WHERE created_at >= '2024-01-01' 
  AND created_at < '2025-01-01';

-- Store email in lowercase or use function index
SELECT * FROM users WHERE email = 'alice@example.com';
-- Or: CREATE INDEX ON users(LOWER(email));

-- Calculate on constant side
SELECT * FROM products 
WHERE price > 100 / 1.1;  -- 90.91
```

---

#### 2. Use Appropriate JOIN Order

**❌ Bad: Large table first**
```sql
-- 10M orders JOIN 1K users
SELECT * FROM orders o
JOIN users u ON o.user_id = u.id
WHERE u.status = 'active';

-- PostgreSQL scans 10M orders first
```

**✅ Good: Filter first, then join**
```sql
-- Filter users first (1K → 100 active)
SELECT * FROM users u
JOIN orders o ON o.user_id = u.id
WHERE u.status = 'active';

-- Or use subquery
SELECT * FROM orders o
WHERE user_id IN (
  SELECT id FROM users WHERE status = 'active'
);
```

---

#### 3. Use EXISTS Instead of IN for Large Subqueries

**❌ Bad: IN with large subquery**
```sql
-- Subquery returns 100K IDs (builds large array in memory)
SELECT * FROM orders 
WHERE user_id IN (
  SELECT id FROM users WHERE city = 'New York'
);
```

**✅ Good: EXISTS (stops at first match)**
```sql
-- Stops as soon as match is found
SELECT * FROM orders o
WHERE EXISTS (
  SELECT 1 FROM users u 
  WHERE u.id = o.user_id AND u.city = 'New York'
);
```

**Performance Comparison:**
```
IN:     Check if user_id is in array of 100K IDs
EXISTS: For each order, check if matching user exists (stops at first)
```

---

#### 4. Avoid SELECT * (Specify Columns)

**❌ Bad: Retrieves unnecessary data**
```sql
SELECT * FROM users WHERE email = 'alice@example.com';
-- Returns all 20 columns (500 bytes)
```

**✅ Good: Only needed columns**
```sql
SELECT id, name, email FROM users WHERE email = 'alice@example.com';
-- Returns only 3 columns (100 bytes)
-- 5x less data transferred
-- Can use covering index
```

---

#### 5. Use LIMIT for Pagination

**❌ Bad: Retrieve all rows**
```sql
-- Application fetches 1M rows, displays 10
SELECT * FROM products ORDER BY created_at DESC;
```

**✅ Good: LIMIT + OFFSET**
```sql
-- Fetch only needed rows
SELECT * FROM products 
ORDER BY created_at DESC 
LIMIT 10 OFFSET 0;

-- Page 2
SELECT * FROM products 
ORDER BY created_at DESC 
LIMIT 10 OFFSET 10;
```

**Even Better: Cursor-based pagination**
```sql
-- Page 1
SELECT * FROM products 
WHERE id > 0
ORDER BY id 
LIMIT 10;
-- Last ID: 10

-- Page 2 (use last ID from previous page)
SELECT * FROM products 
WHERE id > 10
ORDER BY id 
LIMIT 10;
-- Faster than OFFSET (doesn't skip rows)
```

---

#### 6. Optimize JOINs

**Use Proper Indexes on JOIN Columns:**

```sql
-- Create indexes on foreign keys
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_users_id ON users(id);  -- Usually already exists (PK)

-- Fast JOIN
SELECT o.*, u.name 
FROM orders o
JOIN users u ON o.user_id = u.id;
```

**Choose RIGHT JOIN Type:**

```sql
-- INNER JOIN (most common)
-- Only matching rows from both tables
SELECT * FROM orders o
INNER JOIN users u ON o.user_id = u.id;

-- LEFT JOIN
-- All orders + matching users (NULL if no user)
SELECT * FROM orders o
LEFT JOIN users u ON o.user_id = u.id;

-- Use INNER JOIN when possible (faster)
```

---

#### 7. Batch Operations

**❌ Bad: Individual inserts**
```sql
-- 1000 separate queries
INSERT INTO users (name, email) VALUES ('User1', 'user1@example.com');
INSERT INTO users (name, email) VALUES ('User2', 'user2@example.com');
-- ... 998 more
-- Time: 10 seconds
```

**✅ Good: Batch insert**
```sql
-- Single query with multiple values
INSERT INTO users (name, email) VALUES
  ('User1', 'user1@example.com'),
  ('User2', 'user2@example.com'),
  -- ... 998 more
  ('User1000', 'user1000@example.com');
-- Time: 0.5 seconds (20x faster!)
```

---

#### 8. Use Appropriate Data Types

**❌ Bad: Oversized types**
```sql
CREATE TABLE users (
  id BIGINT,           -- Max 2^63 (usually INT is enough)
  age VARCHAR(10),     -- String for number
  is_active VARCHAR(5) -- String for boolean
);
```

**✅ Good: Appropriate types**
```sql
CREATE TABLE users (
  id INTEGER,          -- Max 2^31 (enough for most cases)
  age SMALLINT,        -- Max 32767 (perfect for age)
  is_active BOOLEAN    -- True/false
);
-- Smaller storage, faster queries, better indexes
```

---

#### 9. Avoid DISTINCT When Possible

**❌ Bad: Expensive DISTINCT**
```sql
-- Removes duplicates (expensive sort/hash)
SELECT DISTINCT user_id FROM orders;
```

**✅ Good: GROUP BY (if you need aggregation anyway)**
```sql
SELECT user_id FROM orders GROUP BY user_id;
```

**✅ Even Better: Fix data model**
```sql
-- If you need distinct user_ids, maybe you need a different query
-- Or ensure unique constraint at insertion time
```

---

#### 10. Use UNION ALL Instead of UNION

**❌ Bad: UNION removes duplicates**
```sql
SELECT id FROM active_users
UNION
SELECT id FROM inactive_users;
-- Removes duplicates (expensive)
```

**✅ Good: UNION ALL (if duplicates are OK)**
```sql
SELECT id FROM active_users
UNION ALL
SELECT id FROM inactive_users;
-- No duplicate check (faster)
```

---

### Real-World Optimization Example

**Before Optimization:**
```sql
-- Slow query (8 seconds)
SELECT 
  u.*,
  COUNT(o.id) as order_count,
  SUM(o.total) as total_spent
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE EXTRACT(YEAR FROM o.created_at) = 2024
  AND LOWER(u.email) LIKE '%@gmail.com'
GROUP BY u.id
ORDER BY total_spent DESC;
```

**Problems:**
1. ❌ `EXTRACT(YEAR FROM ...)` prevents index usage
2. ❌ `LOWER(u.email)` prevents index usage
3. ❌ `LIKE '%@gmail.com'` cannot use index (leading wildcard)
4. ❌ `SELECT u.*` retrieves all columns
5. ❌ Expensive GROUP BY

**After Optimization:**
```sql
-- Create indexes
CREATE INDEX idx_orders_created_user ON orders(created_at, user_id, total)
WHERE created_at >= '2024-01-01';

CREATE INDEX idx_users_email_pattern ON users(email)
WHERE email LIKE '%@gmail.com';  -- Partial index

-- Optimized query (0.3 seconds)
SELECT 
  u.id,
  u.name,
  u.email,
  COUNT(o.id) as order_count,
  SUM(o.total) as total_spent
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE o.created_at >= '2024-01-01' 
  AND o.created_at < '2025-01-01'
  AND u.email LIKE '%@gmail.com'
GROUP BY u.id, u.name, u.email
ORDER BY total_spent DESC
LIMIT 100;  -- Added limit
```

**Improvements:**
- ✅ Use date range instead of EXTRACT
- ✅ Removed LOWER (store emails lowercase)
- ✅ Added LIMIT (don't need all results)
- ✅ Specified columns instead of `u.*`
- ✅ Changed LEFT JOIN to JOIN (faster)
- ✅ **Result: 27x faster!**

---

### Practice Questions: SQL Query Optimization

#### True/False

1. Using functions on indexed columns (like UPPER, EXTRACT) allows the index to be used.
2. UNION ALL is faster than UNION because it doesn't check for duplicates.
3. SELECT * is always slower than selecting specific columns.
4. EXISTS is generally faster than IN for large subqueries.
5. More indexes always improve query performance.

<details>
<summary><strong>View Answers</strong></summary>

1. FALSE (functions prevent index usage)
2. TRUE
3. TRUE (retrieves unnecessary data, can't use covering indexes)
4. TRUE (EXISTS stops at first match)
5. FALSE (too many indexes slow down writes)

</details>

---

#### Coding Challenge: Optimize This Query

```sql
SELECT * FROM products 
WHERE UPPER(name) LIKE '%LAPTOP%'
  AND price * 1.2 > 1000
ORDER BY created_at DESC;
```

<details>
<summary><strong>View Solution</strong></summary>

**Problems:**
1. ❌ `UPPER(name)` prevents index usage
2. ❌ `LIKE '%LAPTOP%'` cannot use B-Tree index (leading wildcard)
3. ❌ `price * 1.2 > 1000` prevents index usage
4. ❌ `SELECT *` retrieves all columns

**Optimized:**
```sql
-- 1. Store name in uppercase or use function index
CREATE INDEX idx_products_name_upper ON products(UPPER(name));

-- 2. Use full-text search instead of LIKE
CREATE INDEX idx_products_name_fts ON products USING GIN(to_tsvector('english', name));

-- 3. Calculate on constant side
-- price * 1.2 > 1000  →  price > 1000 / 1.2  →  price > 833.33

-- 4. Create index for sorting
CREATE INDEX idx_products_created ON products(created_at DESC);

-- Optimized query
SELECT id, name, price, created_at 
FROM products 
WHERE to_tsvector('english', name) @@ to_tsquery('laptop')
  AND price > 833.33
ORDER BY created_at DESC;
```

**Alternative (if full-text search not needed):**
```sql
-- Store name in uppercase
UPDATE products SET name = UPPER(name);

-- Query
SELECT id, name, price, created_at 
FROM products 
WHERE name LIKE '%LAPTOP%'
  AND price > 833.33
ORDER BY created_at DESC;

-- Create trigram index for LIKE
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_products_name_trgm ON products USING GIN(name gin_trgm_ops);
```

</details>

---

## 4.2.3 MongoDB Query Optimization

### MongoDB Optimization Techniques

---

#### 1. Use Projection to Limit Fields

**❌ Bad: Retrieve all fields**
```javascript
db.users.find({ email: "alice@example.com" });
// Returns entire document (all 20 fields)
```

**✅ Good: Project only needed fields**
```javascript
db.users.find(
  { email: "alice@example.com" },
  { name: 1, email: 1, _id: 0 }
);
// Returns only name and email
// Less network transfer, faster response
```

---

#### 2. Use Covered Queries

**❌ Bad: Index + document fetch**
```javascript
db.users.createIndex({ email: 1 });

db.users.find(
  { email: "alice@example.com" },
  { name: 1, email: 1 }
);
// IXSCAN → FETCH (reads index + document)
```

**✅ Good: Covered query (all data from index)**
```javascript
// Compound index with all needed fields
db.users.createIndex({ email: 1, name: 1 });

db.users.find(
  { email: "alice@example.com" },
  { name: 1, email: 1, _id: 0 }  // Must exclude _id
);
// IXSCAN only (no FETCH needed)
// Faster - never touches documents
```

---

#### 3. Avoid Negation Operators

**❌ Bad: Negation prevents index usage**
```javascript
db.products.find({ 
  category: { $ne: "Electronics" } 
});
// Cannot use index efficiently (scans all non-Electronics)

db.orders.find({ 
  status: { $nin: ["cancelled", "refunded"] } 
});
// Scans many documents
```

**✅ Good: Use positive queries**
```javascript
// If you know the values
db.products.find({ 
  category: { $in: ["Furniture", "Clothing", "Books"] } 
});

db.orders.find({ 
  status: { $in: ["pending", "processing", "completed"] } 
});
// Uses index on status
```

---

#### 4. Use $elemMatch for Array Queries

**❌ Bad: Implicit AND on array**
```javascript
// Documents:
{ _id: 1, scores: [80, 90, 70] }
{ _id: 2, scores: [60, 85, 95] }

// Intended: Find docs with score > 80 AND score < 90
db.students.find({ 
  scores: { $gt: 80, $lt: 90 } 
});
// Returns BOTH documents!
// 80 < 90 (doc 1), 85 is between 80-90 (doc 2)
```

**✅ Good: Use $elemMatch**
```javascript
db.students.find({ 
  scores: { 
    $elemMatch: { $gt: 80, $lt: 90 } 
  } 
});
// Returns only doc 2
// Single array element must satisfy BOTH conditions
```

---

#### 5. Optimize Sort with Indexes

**❌ Bad: In-memory sort**
```javascript
db.products.find({ category: "Electronics" })
  .sort({ price: 1 });
// COLLSCAN or IXSCAN → SORT (in-memory)
// Limited to 32MB of data
```

**✅ Good: Index supports sort**
```javascript
// Create compound index
db.products.createIndex({ category: 1, price: 1 });

db.products.find({ category: "Electronics" })
  .sort({ price: 1 });
// IXSCAN (no SORT stage needed)
// Data already sorted in index
```

---

#### 6. Use Aggregation Pipeline Wisely

**❌ Bad: Filter late in pipeline**
```javascript
db.orders.aggregate([
  { $lookup: { from: "users", localField: "userId", foreignField: "_id", as: "user" } },
  { $unwind: "$user" },
  { $match: { status: "completed" } },  // Filter AFTER expensive join
  { $group: { _id: "$userId", total: { $sum: "$total" } } }
]);
```

**✅ Good: Filter early (reduce data)**
```javascript
db.orders.aggregate([
  { $match: { status: "completed" } },  // Filter FIRST
  { $lookup: { from: "users", localField: "userId", foreignField: "_id", as: "user" } },
  { $unwind: "$user" },
  { $group: { _id: "$userId", total: { $sum: "$total" } } }
]);
// Processes fewer documents through expensive stages
```

**Pipeline Optimization Rules:**
1. **$match** early (filter first)
2. **$project** early (reduce field size)
3. **$limit** after sort (reduce sorted data)
4. **$lookup** late (expensive)
5. Use indexes in **$match** and **$sort**

---

#### 7. Limit Array Growth

**❌ Bad: Unbounded array**
```javascript
// Document grows indefinitely
{
  _id: ObjectId("..."),
  title: "Popular Post",
  comments: [
    // ... 10,000 comments
  ]
}
// Document exceeds 16MB limit!
// Slow to retrieve entire document
```

**✅ Good: Cap array size or use references**
```javascript
// Option 1: Cap array with $slice
db.posts.updateOne(
  { _id: postId },
  { 
    $push: { 
      comments: { 
        $each: [newComment],
        $slice: -100  // Keep only last 100
      } 
    } 
  }
);

// Option 2: Separate collection (better for large arrays)
// posts collection
{ _id: ObjectId("post-1"), title: "Popular Post" }

// comments collection (referenced)
{ _id: ObjectId("..."), postId: ObjectId("post-1"), text: "Great!" }
{ _id: ObjectId("..."), postId: ObjectId("post-1"), text: "Thanks!" }
```

---

#### 8. Use Bulk Operations

**❌ Bad: Individual operations**
```javascript
// 1000 separate queries
for (let i = 0; i < 1000; i++) {
  db.users.insertOne({ name: `User${i}`, email: `user${i}@example.com` });
}
// Time: 10 seconds
```

**✅ Good: Bulk operations**
```javascript
// Single bulk operation
const bulk = db.users.initializeUnorderedBulkOp();

for (let i = 0; i < 1000; i++) {
  bulk.insert({ name: `User${i}`, email: `user${i}@example.com` });
}

bulk.execute();
// Time: 0.5 seconds (20x faster!)
```

---

#### 9. Use Hint to Force Index

**When to Use:**
- Query planner chooses wrong index
- Testing different indexes

```javascript
// MongoDB chooses index automatically
db.products.find({ category: "Electronics", price: { $lt: 1000 } });
// Might use idx_category OR idx_price

// Force specific index
db.products.find({ 
  category: "Electronics", 
  price: { $lt: 1000 } 
}).hint({ category: 1, price: 1 });

// Force collection scan (for testing)
db.products.find({ ... }).hint({ $natural: 1 });
```

---

#### 10. Avoid Large Skip Values

**❌ Bad: Large SKIP**
```javascript
// Get page 1000 (skip 10,000 documents)
db.products.find()
  .sort({ _id: 1 })
  .skip(10000)
  .limit(10);
// Scans and discards 10,000 documents
// Very slow!
```

**✅ Good: Range-based pagination**
```javascript
// Page 1
db.products.find()
  .sort({ _id: 1 })
  .limit(10);
// Last _id: ObjectId("...")

// Page 2 (use last _id from previous page)
db.products.find({ 
  _id: { $gt: ObjectId("last-id-from-page-1") } 
})
  .sort({ _id: 1 })
  .limit(10);
// No skip needed - much faster!
```

---

### Real-World MongoDB Optimization Example

**Before Optimization:**
```javascript
// Slow aggregation (12 seconds)
db.orders.aggregate([
  {
    $lookup: {
      from: "products",
      localField: "items.productId",
      foreignField: "_id",
      as: "productDetails"
    }
  },
  {
    $match: {
      createdAt: { 
        $gte: ISODate("2024-01-01"),
        $lt: ISODate("2024-02-01")
      },
      status: "completed"
    }
  },
  {
    $group: {
      _id: "$userId",
      totalOrders: { $sum: 1 },
      totalSpent: { $sum: "$total" }
    }
  },
  { $sort: { totalSpent: -1 } }
]);
```

**Problems:**
1. ❌ $lookup before $match (processes all orders)
2. ❌ No index on createdAt and status
3. ❌ No limit (processes all results)

**After Optimization:**
```javascript
// Create indexes
db.orders.createIndex({ createdAt: 1, status: 1, userId: 1 });

// Optimized aggregation (0.8 seconds)
db.orders.aggregate([
  // 1. Filter FIRST (use index)
  {
    $match: {
      createdAt: { 
        $gte: ISODate("2024-01-01"),
        $lt: ISODate("2024-02-01")
      },
      status: "completed"
    }
  },
  // 2. Group (reduced data)
  {
    $group: {
      _id: "$userId",
      totalOrders: { $sum: 1 },
      totalSpent: { $sum: "$total" },
      productIds: { $push: "$items.productId" }  // Collect product IDs
    }
  },
  // 3. Sort
  { $sort: { totalSpent: -1 } },
  // 4. Limit (don't need all results)
  { $limit: 100 },
  // 5. Lookup LAST (only for top 100)
  {
    $lookup: {
      from: "products",
      localField: "productIds",
      foreignField: "_id",
      as: "productDetails"
    }
  }
]);

// Result: 15x faster!
```

---

### Practice Questions: MongoDB Query Optimization

#### Fill in the Blanks

1. A ________ query gets all data from the index without fetching documents.
2. Use ________ to force MongoDB to use a specific index.
3. The ________ operator ensures all conditions apply to a single array element.
4. In aggregation pipelines, place ________ stages early to reduce data processed.
5. Use ________ operations to batch multiple inserts/updates into a single request.

<details>
<summary><strong>View Answers</strong></summary>

1. covered
2. hint()
3. $elemMatch
4. $match
5. bulk

</details>

---

#### Coding Challenge: Optimize This Aggregation

```javascript
db.posts.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "authorId",
      foreignField: "_id",
      as: "author"
    }
  },
  { $unwind: "$author" },
  {
    $lookup: {
      from: "comments",
      localField: "_id",
      foreignField: "postId",
      as: "comments"
    }
  },
  {
    $match: {
      "author.status": "active",
      createdAt: { $gte: ISODate("2024-01-01") }
    }
  },
  {
    $project: {
      title: 1,
      "author.name": 1,
      commentCount: { $size: "$comments" }
    }
  },
  { $sort: { commentCount: -1 } },
  { $limit: 10 }
]);
```

<details>
<summary><strong>View Solution</strong></summary>

**Problems:**
1. ❌ Two expensive $lookup operations before filtering
2. ❌ $match after $lookup (processes all documents)
3. ❌ No indexes

**Optimized:**
```javascript
// Create indexes
db.posts.createIndex({ createdAt: 1, authorId: 1 });
db.comments.createIndex({ postId: 1 });

db.posts.aggregate([
  // 1. Filter posts FIRST
  {
    $match: {
      createdAt: { $gte: ISODate("2024-01-01") }
    }
  },
  
  // 2. Lookup author
  {
    $lookup: {
      from: "users",
      localField: "authorId",
      foreignField: "_id",
      as: "author"
    }
  },
  { $unwind: "$author" },
  
  // 3. Filter by author status EARLY
  {
    $match: {
      "author.status": "active"
    }
  },
  
  // 4. Project early (reduce field size)
  {
    $project: {
      title: 1,
      "author.name": 1,
      _id: 1
    }
  },
  
  // 5. Lookup comments (with pipeline for count only)
  {
    $lookup: {
      from: "comments",
      let: { postId: "$_id" },
      pipeline: [
        { $match: { $expr: { $eq: ["$postId", "$$postId"] } } },
        { $count: "count" }
      ],
      as: "commentStats"
    }
  },
  
  // 6. Add comment count
  {
    $addFields: {
      commentCount: { 
        $ifNull: [{ $arrayElemAt: ["$commentStats.count", 0] }, 0] 
      }
    }
  },
  
  // 7. Sort and limit
  { $sort: { commentCount: -1 } },
  { $limit: 10 },
  
  // 8. Final projection
  {
    $project: {
      title: 1,
      "author.name": 1,
      commentCount: 1
    }
  }
]);
```

**Improvements:**
- ✅ Filter posts early with index
- ✅ Filter authors before expensive operations
- ✅ Use $lookup pipeline to count comments (don't fetch all)
- ✅ Project early to reduce data size
- ✅ Result: ~10x faster

</details>

---

## 4.3 Caching Strategies

### Why Caching?

**Without Cache:**
```
User Request → Database Query → Response
Every request hits database (slow, expensive)
```

**With Cache:**
```
User Request → Check Cache → Cache Hit? → Return from Cache (fast!)
                           → Cache Miss? → Query DB → Cache Result → Return
```

**Benefits:**
- ✅ **Faster response** (10-100x faster)
- ✅ **Reduced database load** (fewer queries)
- ✅ **Lower costs** (less compute/bandwidth)
- ✅ **Better scalability** (handle more users)

---

## 4.3.1 Application-Level Caching (Redis)

### Redis as a Cache

**Setup:**
```javascript
const redis = require('redis');
const client = redis.createClient({
  host: 'localhost',
  port: 6379
});
```

---

### Basic Caching Pattern

```javascript
async function getUser(userId) {
  const cacheKey = `user:${userId}`;
  
  // 1. Check cache
  const cached = await redis.get(cacheKey);
  if (cached) {
    console.log('Cache hit!');
    return JSON.parse(cached);
  }
  
  // 2. Cache miss - query database
  console.log('Cache miss - querying database');
  const user = await db.users.findOne({ _id: userId });
  
  // 3. Store in cache (1 hour TTL)
  await redis.setex(cacheKey, 3600, JSON.stringify(user));
  
  return user;
}
```

**Performance:**
```
First request:  Cache miss  → 50ms (database query)
Second request: Cache hit   → 1ms (Redis)
Improvement: 50x faster!
```

---

### Caching Strategies

#### 1. Cache-Aside (Lazy Loading)

**Pattern:** Application manages cache manually.

```javascript
// Read
async function getProduct(productId) {
  const cacheKey = `product:${productId}`;
  
  // Try cache
  let product = await redis.get(cacheKey);
  if (product) return JSON.parse(product);
  
  // Query DB
  product = await db.products.findOne({ _id: productId });
  
  // Update cache
  await redis.setex(cacheKey, 3600, JSON.stringify(product));
  
  return product;
}

// Write
async function updateProduct(productId, updates) {
  // Update database
  await db.products.updateOne({ _id: productId }, { $set: updates });
  
  // Invalidate cache
  await redis.del(`product:${productId}`);
}
```

**Pros:**
- ✅ Only caches requested data
- ✅ Cache failure doesn't break app
- ✅ Simple to implement

**Cons:**
- ❌ Cache miss penalty (first request slow)
- ❌ Potential for stale data

---

#### 2. Write-Through Cache

**Pattern:** Update cache and database together.

```javascript
async function updateProduct(productId, updates) {
  // 1. Update database
  await db.products.updateOne({ _id: productId }, { $set: updates });
  
  // 2. Update cache immediately
  const product = await db.products.findOne({ _id: productId });
  await redis.setex(`product:${productId}`, 3600, JSON.stringify(product));
}
```

**Pros:**
- ✅ Cache always up-to-date
- ✅ No stale data

**Cons:**
- ❌ Write latency (two operations)
- ❌ Caches data even if never read

---

#### 3. Write-Behind Cache (Write-Back)

**Pattern:** Update cache immediately, database asynchronously.

```javascript
const updateQueue = [];

async function updateProduct(productId, updates) {
  // 1. Update cache immediately
  const cacheKey = `product:${productId}`;
  const product = await redis.get(cacheKey);
  const updated = { ...JSON.parse(product), ...updates };
  await redis.setex(cacheKey, 3600, JSON.stringify(updated));
  
  // 2. Queue database update
  updateQueue.push({ productId, updates });
  
  return updated;
}

// Background worker
setInterval(async () => {
  while (updateQueue.length > 0) {
    const { productId, updates } = updateQueue.shift();
    await db.products.updateOne({ _id: productId }, { $set: updates });
  }
}, 1000);  // Flush every second
```

**Pros:**
- ✅ Fast writes (no wait for DB)
- ✅ Can batch updates

**Cons:**
- ❌ Risk of data loss (cache failure before DB write)
- ❌ Complex implementation

---

### Cache Key Design

**Good Key Patterns:**

```javascript
// Hierarchical naming
user:123:profile
user:123:settings
user:123:sessions:abc

product:456:details
product:456:reviews:page:1

// Include version for breaking changes
user:123:profile:v2

// Include query parameters
products:category:electronics:page:1:limit:20
```

**Bad Key Patterns:**

```javascript
// ❌ No structure
u123
p456data

// ❌ Too generic (collisions)
data:123
```

---

### Cache TTL Strategy

```javascript
// Short TTL for frequently changing data
await redis.setex('cart:user:123', 300, data);  // 5 minutes

// Medium TTL for semi-static data
await redis.setex('product:456', 3600, data);  // 1 hour

// Long TTL for static data
await redis.setex('category:list', 86400, data);  // 24 hours

// No expiration for permanent data
await redis.set('config:api:key', data);  // No TTL
```

---

### Multi-Level Caching

**Architecture:**
```
Request
  ↓
Application Memory Cache (L1) ← Fastest (0.01ms)
  ↓ (miss)
Redis Cache (L2) ← Fast (1ms)
  ↓ (miss)
Database (L3) ← Slow (50ms)
```

**Implementation:**

```javascript
const NodeCache = require('node-cache');
const memoryCache = new NodeCache({ stdTTL: 60 });

async function getUser(userId) {
  const cacheKey = `user:${userId}`;
  
  // L1: Memory cache
  let user = memoryCache.get(cacheKey);
  if (user) {
    console.log('L1 hit');
    return user;
  }
  
  // L2: Redis cache
  user = await redis.get(cacheKey);
  if (user) {
    console.log('L2 hit');
    user = JSON.parse(user);
    memoryCache.set(cacheKey, user);  // Populate L1
    return user;
  }
  
  // L3: Database
  console.log('Database query');
  user = await db.users.findOne({ _id: userId });
  
  // Populate caches
  memoryCache.set(cacheKey, user);  // L1
  await redis.setex(cacheKey, 3600, JSON.stringify(user));  // L2
  
  return user;
}
```

---

### Caching Full Pages (HTTP Cache)

```javascript
const express = require('express');
const app = express();

// Cache middleware
async function cacheMiddleware(req, res, next) {
  const cacheKey = `page:${req.url}`;
  
  const cached = await redis.get(cacheKey);
  if (cached) {
    console.log('Serving from cache');
    return res.send(cached);
  }
  
  // Store original send function
  const originalSend = res.send;
  
  // Override send to cache response
  res.send = function(body) {
    redis.setex(cacheKey, 300, body);  // Cache 5 minutes
    originalSend.call(this, body);
  };
  
  next();
}

// Apply to routes
app.get('/products', cacheMiddleware, async (req, res) => {
  const products = await db.products.find().limit(20);
  res.json(products);
});
```

---

## 4.3.2 Cache Invalidation

### The Two Hard Things in Computer Science

> "There are only two hard things in Computer Science: cache invalidation and naming things."
> — Phil Karlton

**The Problem:** How to keep cache in sync with database?

---

### Invalidation Strategies

#### 1. TTL-Based Invalidation (Time-To-Live)

**Pattern:** Cache expires after fixed time.

```javascript
// Set TTL when caching
async function cacheProduct(productId, product) {
  await redis.setex(
    `product:${productId}`,
    3600,  // 1 hour TTL
    JSON.stringify(product)
  );
}

// After 1 hour, cache automatically expires
// Next request fetches fresh data from database
```

**Pros:**
- ✅ Simple to implement
- ✅ Automatic cleanup
- ✅ Prevents unbounded cache growth

**Cons:**
- ❌ Stale data possible (until TTL expires)
- ❌ Cache stampede (many requests hit DB when cache expires)

**When to Use:**
- Data changes infrequently
- Eventual consistency is acceptable
- Example: Product catalog, categories

---

#### 2. Explicit Invalidation (On Write)

**Pattern:** Delete/update cache when database changes.

```javascript
// Update product in database and invalidate cache
async function updateProduct(productId, updates) {
  // 1. Update database
  await db.products.updateOne(
    { _id: productId },
    { $set: updates }
  );
  
  // 2. Invalidate cache
  await redis.del(`product:${productId}`);
  
  // Next read will fetch fresh data
}

// Or update cache immediately (write-through)
async function updateProductWriteThrough(productId, updates) {
  // 1. Update database
  await db.products.updateOne(
    { _id: productId },
    { $set: updates }
  );
  
  // 2. Update cache
  const product = await db.products.findOne({ _id: productId });
  await redis.setex(
    `product:${productId}`,
    3600,
    JSON.stringify(product)
  );
}
```

**Pros:**
- ✅ Cache always fresh
- ✅ No stale data

**Cons:**
- ❌ Must invalidate on every write
- ❌ Complex with multiple update paths
- ❌ Cache invalidation logic in application code

**When to Use:**
- Strong consistency required
- Controlled write paths
- Example: User profiles, settings

---

#### 3. Event-Driven Invalidation

**Pattern:** Database triggers invalidation events.

```javascript
// MongoDB Change Streams
const changeStream = db.products.watch();

changeStream.on('change', async (change) => {
  if (change.operationType === 'update' || change.operationType === 'replace') {
    const productId = change.documentKey._id;
    
    // Invalidate cache
    await redis.del(`product:${productId}`);
    
    console.log(`Cache invalidated for product ${productId}`);
  }
});

// PostgreSQL NOTIFY/LISTEN
// In PostgreSQL trigger:
CREATE OR REPLACE FUNCTION notify_product_change()
RETURNS TRIGGER AS $$
BEGIN
  PERFORM pg_notify('product_changed', NEW.id::text);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER product_update_trigger
AFTER UPDATE ON products
FOR EACH ROW
EXECUTE FUNCTION notify_product_change();

// In Node.js:
const { Client } = require('pg');
const client = new Client();

client.on('notification', async (msg) => {
  if (msg.channel === 'product_changed') {
    const productId = msg.payload;
    await redis.del(`product:${productId}`);
  }
});

await client.query('LISTEN product_changed');
```

**Pros:**
- ✅ Decoupled from application logic
- ✅ Handles external updates (admin panels, scripts)
- ✅ Centralized invalidation

**Cons:**
- ❌ Complex setup
- ❌ Requires database support
- ❌ Additional infrastructure

**When to Use:**
- Multiple applications updating database
- External data updates
- Microservices architecture

---

#### 4. Tag-Based Invalidation

**Pattern:** Group related cache entries with tags.

```javascript
// Cache with tags
async function cacheProductWithTags(productId, product) {
  const cacheKey = `product:${productId}`;
  
  // Store product
  await redis.setex(cacheKey, 3600, JSON.stringify(product));
  
  // Add to tag sets
  await redis.sadd(`tag:category:${product.category}`, cacheKey);
  await redis.sadd(`tag:brand:${product.brand}`, cacheKey);
}

// Invalidate all products in category
async function invalidateCategory(category) {
  const keys = await redis.smembers(`tag:category:${category}`);
  
  if (keys.length > 0) {
    await redis.del(...keys);
    await redis.del(`tag:category:${category}`);
  }
  
  console.log(`Invalidated ${keys.length} products in category ${category}`);
}

// Invalidate all products from brand
async function invalidateBrand(brand) {
  const keys = await redis.smembers(`tag:brand:${brand}`);
  
  if (keys.length > 0) {
    await redis.del(...keys);
    await redis.del(`tag:brand:${brand}`);
  }
}
```

**Pros:**
- ✅ Bulk invalidation
- ✅ Flexible grouping
- ✅ Granular control

**Cons:**
- ❌ Additional storage (tags)
- ❌ Must maintain tag sets

**When to Use:**
- Related data changes together
- Bulk invalidation needed
- Example: All products in category, all user's sessions

---

### Cache Stampede Prevention

**Problem:** When cache expires, many requests hit database simultaneously.

```
Cache expires at 10:00:00
↓
1000 simultaneous requests at 10:00:01
↓
All miss cache → All query database
↓
Database overload! 💥
```

**Solution 1: Probabilistic Early Expiration**

```javascript
async function getCachedWithEarlyExpiration(key, ttl, fetchFn) {
  const cached = await redis.get(key);
  
  if (cached) {
    const data = JSON.parse(cached);
    const age = Date.now() - data.cachedAt;
    
    // Probabilistic refresh: higher chance as expiration approaches
    // Refresh probability = age / ttl
    const refreshProb = age / (ttl * 1000);
    
    if (Math.random() < refreshProb) {
      // Refresh asynchronously (don't wait)
      fetchFn().then(freshData => {
        redis.setex(key, ttl, JSON.stringify({
          value: freshData,
          cachedAt: Date.now()
        }));
      });
    }
    
    return data.value;
  }
  
  // Cache miss
  const freshData = await fetchFn();
  await redis.setex(key, ttl, JSON.stringify({
    value: freshData,
    cachedAt: Date.now()
  }));
  
  return freshData;
}
```

**Solution 2: Lock-Based Cache Refresh**

```javascript
async function getCachedWithLock(key, ttl, fetchFn) {
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);
  
  const lockKey = `lock:${key}`;
  const lockAcquired = await redis.set(lockKey, '1', 'NX', 'EX', 10);
  
  if (lockAcquired) {
    // This request refreshes cache
    try {
      const data = await fetchFn();
      await redis.setex(key, ttl, JSON.stringify(data));
      return data;
    } finally {
      await redis.del(lockKey);
    }
  } else {
    // Another request is refreshing, wait briefly
    await new Promise(resolve => setTimeout(resolve, 100));
    
    const refreshed = await redis.get(key);
    if (refreshed) return JSON.parse(refreshed);
    
    // Still no cache, fetch anyway
    return await fetchFn();
  }
}
```

---

## 4.3.3 Caching Patterns

### Pattern 1: Cache-Aside with Read-Through

```javascript
class CacheAsideService {
  constructor(redis, db) {
    this.redis = redis;
    this.db = db;
  }
  
  async get(collection, id) {
    const cacheKey = `${collection}:${id}`;
    
    // Read from cache
    const cached = await this.redis.get(cacheKey);
    if (cached) return JSON.parse(cached);
    
    // Read from database
    const data = await this.db[collection].findOne({ _id: id });
    
    // Write to cache
    if (data) {
      await this.redis.setex(cacheKey, 3600, JSON.stringify(data));
    }
    
    return data;
  }
  
  async update(collection, id, updates) {
    // Update database
    await this.db[collection].updateOne({ _id: id }, { $set: updates });
    
    // Invalidate cache
    await this.redis.del(`${collection}:${id}`);
  }
}
```

---

### Pattern 2: Query Result Caching

```javascript
async function getCachedQuery(queryKey, queryFn, ttl = 3600) {
  const cacheKey = `query:${queryKey}`;
  
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);
  
  const result = await queryFn();
  await redis.setex(cacheKey, ttl, JSON.stringify(result));
  
  return result;
}

// Usage
const topProducts = await getCachedQuery(
  'products:top:electronics',
  async () => {
    return await db.products
      .find({ category: 'Electronics' })
      .sort({ sales: -1 })
      .limit(10);
  },
  1800  // 30 minutes
);
```

---

### Pattern 3: Fragment Caching (Partial Page)

```javascript
// Cache individual page components
async function renderProductPage(productId) {
  // Cache product details separately
  const product = await getCached(
    `product:${productId}`,
    () => db.products.findOne({ _id: productId }),
    3600
  );
  
  // Cache reviews separately (shorter TTL)
  const reviews = await getCached(
    `reviews:${productId}`,
    () => db.reviews.find({ productId }).limit(5),
    300
  );
  
  // Cache related products separately
  const related = await getCached(
    `related:${productId}`,
    () => db.products.find({ category: product.category }).limit(4),
    1800
  );
  
  return {
    product,
    reviews,
    related
  };
}
```

---

### Pattern 4: Memoization (Function Result Caching)

```javascript
// Memoize expensive calculations
function memoize(fn, ttl = 3600) {
  return async function(...args) {
    const cacheKey = `memoize:${fn.name}:${JSON.stringify(args)}`;
    
    const cached = await redis.get(cacheKey);
    if (cached) return JSON.parse(cached);
    
    const result = await fn(...args);
    await redis.setex(cacheKey, ttl, JSON.stringify(result));
    
    return result;
  };
}

// Usage
const calculateShipping = memoize(async (weight, distance, zone) => {
  // Expensive calculation
  await new Promise(resolve => setTimeout(resolve, 1000));
  return (weight * 0.5) + (distance * 0.1) + (zone * 2);
});

// First call: 1 second
await calculateShipping(10, 500, 3);

// Second call with same args: < 1ms
await calculateShipping(10, 500, 3);
```

---

## 4.3.4 Database-Level Caching

### PostgreSQL Query Cache

**Shared Buffers:**
```sql
-- View buffer cache statistics
SELECT 
  schemaname,
  tablename,
  heap_blks_read,     -- Disk reads
  heap_blks_hit,      -- Cache hits
  ROUND(
    heap_blks_hit::numeric / 
    NULLIF(heap_blks_hit + heap_blks_read, 0) * 100, 
    2
  ) AS cache_hit_ratio
FROM pg_stattuple_approx('users');

-- Configure buffer size (postgresql.conf)
shared_buffers = 4GB  -- 25% of RAM recommended
```

**Query Plan Cache:**
```sql
-- Prepared statements are cached
PREPARE get_user (INTEGER) AS
  SELECT * FROM users WHERE id = $1;

-- Executes with cached plan
EXECUTE get_user(123);
```

---

### MongoDB Query Cache

**WiredTiger Cache:**
```javascript
// View cache statistics
db.serverStatus().wiredTiger.cache;

// Configure cache size (mongod.conf)
// wiredTigerCacheSizeGB: 4  // 50% of RAM - MongoDB overhead
```

**Query Result Cache:**
```javascript
// MongoDB doesn't cache query results by default
// Use application-level caching (Redis)

// But indexes are cached in memory
db.collection.stats().indexSizes;  // Memory used by indexes
```

---

## 4.3.5 Materialized Views

### What is a Materialized View?

**Regular View:** Virtual table (query executed each time)
```sql
CREATE VIEW active_users AS
  SELECT * FROM users WHERE status = 'active';

-- Query runs every time
SELECT * FROM active_users;
```

**Materialized View:** Stored results (pre-computed)
```sql
CREATE MATERIALIZED VIEW active_users_mv AS
  SELECT * FROM users WHERE status = 'active';

-- Query uses stored results (fast!)
SELECT * FROM active_users_mv;
```

---

### PostgreSQL Materialized Views

**Create Materialized View:**
```sql
-- Complex aggregation
CREATE MATERIALIZED VIEW product_stats AS
SELECT 
  p.id,
  p.name,
  p.category,
  COUNT(o.id) AS total_orders,
  SUM(oi.quantity) AS total_quantity_sold,
  SUM(oi.price * oi.quantity) AS total_revenue,
  AVG(r.rating) AS avg_rating
FROM products p
LEFT JOIN order_items oi ON oi.product_id = p.id
LEFT JOIN orders o ON o.id = oi.order_id
LEFT JOIN reviews r ON r.product_id = p.id
GROUP BY p.id, p.name, p.category;

-- Create index on materialized view
CREATE INDEX idx_product_stats_category ON product_stats(category);
```

**Refresh Materialized View:**
```sql
-- Full refresh (locks table)
REFRESH MATERIALIZED VIEW product_stats;

-- Concurrent refresh (no locks, requires unique index)
CREATE UNIQUE INDEX idx_product_stats_id ON product_stats(id);
REFRESH MATERIALIZED VIEW CONCURRENTLY product_stats;
```

**Query Materialized View:**
```sql
-- Fast query (pre-computed data)
SELECT * FROM product_stats 
WHERE category = 'Electronics'
ORDER BY total_revenue DESC
LIMIT 10;

-- Execution time: 5ms (vs 5000ms without materialized view)
```

---

### Refresh Strategies

**Strategy 1: Scheduled Refresh**
```sql
-- Cron job (every hour)
0 * * * * psql -c "REFRESH MATERIALIZED VIEW CONCURRENTLY product_stats;"
```

**Strategy 2: Trigger-Based Refresh**
```sql
-- Refresh when source data changes
CREATE OR REPLACE FUNCTION refresh_product_stats()
RETURNS TRIGGER AS $$
BEGIN
  REFRESH MATERIALIZED VIEW CONCURRENTLY product_stats;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER product_stats_refresh
AFTER INSERT OR UPDATE OR DELETE ON orders
FOR EACH STATEMENT
EXECUTE FUNCTION refresh_product_stats();
```

**Strategy 3: On-Demand Refresh (Application)**
```javascript
// Refresh after bulk operations
async function processBulkOrders(orders) {
  await db.query('BEGIN');
  
  for (const order of orders) {
    await db.query('INSERT INTO orders ...', [order]);
  }
  
  await db.query('COMMIT');
  
  // Refresh materialized view
  await db.query('REFRESH MATERIALIZED VIEW CONCURRENTLY product_stats');
}
```

---

### MongoDB Equivalent: Pre-Aggregated Collections

**Pattern: Maintain aggregated collection**

```javascript
// Aggregated stats collection
{
  _id: ObjectId("product-id"),
  productId: ObjectId("product-id"),
  totalOrders: 150,
  totalQuantitySold: 300,
  totalRevenue: 45000,
  avgRating: 4.5,
  lastUpdated: ISODate("2024-01-15T10:00:00Z")
}

// Update stats when order placed
async function createOrder(order) {
  // Insert order
  await db.orders.insertOne(order);
  
  // Update product stats
  for (const item of order.items) {
    await db.productStats.updateOne(
      { productId: item.productId },
      {
        $inc: {
          totalOrders: 1,
          totalQuantitySold: item.quantity,
          totalRevenue: item.price * item.quantity
        },
        $set: { lastUpdated: new Date() }
      },
      { upsert: true }
    );
  }
}

// Query is fast (pre-aggregated)
const stats = await db.productStats.find({ 
  totalRevenue: { $gte: 10000 } 
}).sort({ totalRevenue: -1 }).limit(10);
```

---

### Practice Questions: Caching

#### Fill in the Blanks

1. ________ invalidation uses time-to-live to automatically expire cache entries.
2. The ________ ________ problem occurs when many requests hit the database simultaneously after cache expiration.
3. A ________ ________ is a stored result of a query that can be refreshed periodically.
4. ________ -based invalidation groups related cache entries with tags for bulk deletion.
5. In PostgreSQL, ________ ________ stores frequently accessed data in RAM.

<details>
<summary><strong>View Answers</strong></summary>

1. TTL
2. cache stampede
3. materialized view
4. Tag
5. shared buffers

</details>

---

#### Multiple Choice

**Q1: Which invalidation strategy provides the strongest consistency?**

A) TTL-based
B) Explicit invalidation on write
C) Probabilistic early expiration
D) Tag-based

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - Explicit invalidation on write**

Explanation: Explicit invalidation ensures cache is updated/removed immediately when data changes, providing strong consistency at the cost of additional complexity.

</details>

---

**Q2: When should you use a materialized view?**

A) For simple single-table queries
B) For complex aggregations queried frequently
C) For data that changes every second
D) For small tables with few rows

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - For complex aggregations queried frequently**

Explanation: Materialized views are ideal for expensive queries (joins, aggregations) that are run frequently but the underlying data changes infrequently.

</details>

---

## Chapter Summary

### Key Concepts Covered

#### 4.1 Indexing Strategies

**SQL Indexes:**
- **B-Tree** (default): Equality, range, sorting
- **Hash**: Equality only, rare in practice
- **GIN**: Arrays, JSON, full-text search
- **Partial**: Index subset (95% smaller)
- **Composite**: ESR rule (Equality, Sort, Range)

**MongoDB Indexes:**
- 10 types: Single, Compound, Multikey, Text, Geospatial, Unique, Partial, TTL, Sparse, Collation
- Same ESR rule applies
- Use hint() to force specific index

**Index Selection:**
- High selectivity (unique values)
- Foreign keys (always index)
- Covering indexes (all data from index)
- Avoid over-indexing (slows writes)

**Index Maintenance:**
- Monitor usage (drop unused)
- Rebuild fragmented indexes
- Update statistics (ANALYZE, VACUUM)

---

#### 4.2 Query Optimization

**EXPLAIN Analysis:**
- PostgreSQL: `EXPLAIN ANALYZE`
- MongoDB: `.explain("executionStats")`
- Look for: Index scans (good), Collection/Seq scans (bad)

**SQL Optimization (10 Techniques):**
1. Avoid functions on indexed columns
2. Appropriate JOIN order
3. EXISTS vs IN
4. Avoid SELECT *
5. Use LIMIT
6. Optimize JOINs
7. Batch operations
8. Appropriate data types
9. Avoid DISTINCT
10. UNION ALL vs UNION

**MongoDB Optimization (10 Techniques):**
1. Use projection
2. Covered queries
3. Avoid negation ($ne, $nin)
4. Use $elemMatch
5. Index-supported sorting
6. Pipeline optimization ($match early)
7. Limit array growth
8. Bulk operations
9. Use hint()
10. Range-based pagination

---

#### 4.3 Caching Strategies

**Application-Level Caching:**
- **Cache-Aside**: Check cache → DB on miss → Update cache
- **Write-Through**: Update cache + DB together
- **Write-Behind**: Update cache → Async DB update

**Cache Invalidation:**
- **TTL**: Auto-expire (simple, eventual consistency)
- **Explicit**: On write (strong consistency)
- **Event-Driven**: Database triggers (decoupled)
- **Tag-Based**: Bulk invalidation (flexible)

**Cache Patterns:**
- Query result caching
- Fragment caching
- Memoization
- Multi-level caching (L1/L2/L3)

**Database-Level Caching:**
- PostgreSQL shared buffers
- MongoDB WiredTiger cache
- Query plan caching

**Materialized Views:**
- Pre-computed query results
- Refresh strategies: Scheduled, trigger-based, on-demand
- MongoDB equivalent: Pre-aggregated collections

---

### Performance Improvements Summary

| Technique | Improvement | Use Case |
|-----------|-------------|----------|
| **Add B-Tree Index** | 100-1000x | Equality, range queries |
| **Covering Index** | 2-5x | SELECT specific columns |
| **Query Optimization** | 10-100x | Remove functions, proper JOINs |
| **Redis Cache** | 50-100x | Frequently read data |
| **Materialized View** | 100-1000x | Complex aggregations |
| **Batch Operations** | 10-20x | Multiple inserts/updates |
| **Pagination** | 10-100x | Large result sets |

---

### Decision Matrix

**When to Use Each Strategy:**

```
Fast Reads, Infrequent Updates
└─→ Materialized View
    └─→ Refresh: Scheduled or trigger-based

Frequently Read, Frequently Updated
└─→ Application Cache (Redis)
    └─→ TTL: Short (5-15 min)
    └─→ Invalidation: Explicit on write

Complex Query, Many JOINs
└─→ Index Optimization First
    └─→ If still slow: Materialized View
    └─→ If read-heavy: Add Redis Cache

Simple Query, Slow
└─→ Check EXPLAIN
    └─→ Add appropriate index
    └─→ Optimize query (remove functions)

Write-Heavy Workload
└─→ Minimize indexes (only essential)
    └─→ Batch operations
    └─→ Avoid write-through cache
```

---

### Best Practices

**Indexing:**
1. ✅ Index foreign keys
2. ✅ Use ESR rule for compound indexes
3. ✅ Monitor and drop unused indexes
4. ✅ Use partial indexes for subsets
5. ❌ Don't over-index (slows writes)

**Query Optimization:**
1. ✅ Use EXPLAIN to diagnose
2. ✅ Avoid functions on indexed columns
3. ✅ Specify columns (not SELECT *)
4. ✅ Filter early in aggregations
5. ❌ Don't use OFFSET for deep pagination

**Caching:**
1. ✅ Cache frequently read data
2. ✅ Use appropriate TTL
3. ✅ Invalidate on writes
4. ✅ Prevent cache stampede
5. ❌ Don't cache frequently changing data

**Materialized Views:**
1. ✅ Use for complex aggregations
2. ✅ Refresh appropriately (not too often)
3. ✅ Index materialized views
4. ✅ Use CONCURRENTLY to avoid locks
5. ❌ Don't use for rapidly changing data

---

### Common Anti-Patterns

**Indexing:**
- ❌ Indexing low-selectivity columns (gender, boolean)
- ❌ Duplicate indexes (A) and (A, B)
- ❌ Wrong column order in compound index
- ❌ Function-based queries without function index

**Queries:**
- ❌ `SELECT *` when only need few columns
- ❌ `EXTRACT(YEAR FROM date)` instead of date range
- ❌ `UPPER(column) = 'VALUE'` without function index
- ❌ Large OFFSET values

**Caching:**
- ❌ No expiration (unbounded growth)
- ❌ Caching everything (memory waste)
- ❌ No invalidation strategy (stale data)
- ❌ Ignoring cache stampede

---

### Real-World Performance Case Study

**Scenario:** E-commerce product search

**Before Optimization:**
```sql
SELECT * FROM products 
WHERE UPPER(name) LIKE '%LAPTOP%'
  AND price * 1.2 > 1000
ORDER BY created_at DESC;

-- Execution: 8 seconds
-- Rows scanned: 1,000,000
```

**After Optimization:**
```sql
-- 1. Create indexes
CREATE INDEX idx_products_name_trgm ON products USING GIN(name gin_trgm_ops);
CREATE INDEX idx_products_created ON products(created_at DESC);

-- 2. Optimize query
SELECT id, name, price, created_at 
FROM products 
WHERE name ILIKE '%laptop%'  -- Case-insensitive, uses GIN
  AND price > 833.33  -- Moved calculation to constant side
ORDER BY created_at DESC
LIMIT 20;

-- 3. Add Redis cache
const cacheKey = `products:search:laptop:page:1`;
let results = await redis.get(cacheKey);

if (!results) {
  results = await db.query(optimizedQuery);
  await redis.setex(cacheKey, 300, JSON.stringify(results));
}

-- Results:
-- Database query: 8s → 0.05s (160x faster)
-- With Redis cache: 0.05s → 0.001s (8000x faster overall!)
```

---

**Next Chapter:** [Scaling & High Availability →](./sections/05_scaling.md)
