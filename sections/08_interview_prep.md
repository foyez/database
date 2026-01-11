# Chapter 8: Interview Preparation
**Common Questions, Memory Tricks, and Quick Reference**

---

## Table of Contents
- [8.1 Memory Tips & Mnemonics](#81-memory-tips--mnemonics)
- [8.2 Top 20 Interview Questions & Answers](#82-top-20-interview-questions--answers)
- [8.3 System Design Questions](#83-system-design-questions)
- [8.4 Quick Reference Cheatsheet](#84-quick-reference-cheatsheet)

---

## 8.1 Memory Tips & Mnemonics

### ACID Properties

**Mnemonic: "A Car Is Durable"**

- **A**tomicity - All or nothing
- **C**onsistency - Valid state to valid state
- **I**solation - Transactions don't interfere
- **D**urability - Changes persist after commit

**Interview Answer:**
> "ACID ensures reliable transactions. Atomicity means the transaction either completes fully or not at all. Consistency ensures the database moves from one valid state to another. Isolation prevents concurrent transactions from interfering with each other. Durability guarantees that committed changes survive system failures."

---

### CAP Theorem

**Mnemonic: "Can't Always Party"**

- **C**onsistency - All nodes see same data
- **A**vailability - System always responds
- **P**artition Tolerance - Works despite network failures

**Pick 2 of 3:**
- **CA** - Single node (PostgreSQL, MySQL)
- **CP** - Consistent + Partition tolerant (MongoDB, HBase)
- **AP** - Available + Partition tolerant (Cassandra, DynamoDB)

**Interview Answer:**
> "The CAP theorem states you can only guarantee 2 of 3 properties in a distributed system. For financial systems, I'd choose CP (consistency + partition tolerance) like MongoDB with majority writes. For social media feeds, I'd choose AP (availability + partition tolerance) like Cassandra, accepting eventual consistency."

---

### Normalization Levels

**Mnemonic: "Every Boy Truly Needs Dessert"**

- **1NF** - **E**liminate repeating groups (atomic values)
- **2NF** - **B**uild on 1NF + remove partial dependencies
- **3NF** - **T**ake 2NF + remove transitive dependencies
- **BCNF** - **N**ormalized + every determinant is a candidate key
- **4NF** - **D**ecompose multi-valued dependencies

**Quick Memory:**
```
1NF: No repeating groups
2NF: No partial dependencies (non-key depends on part of key)
3NF: No transitive dependencies (non-key depends on non-key)
```

---

### SQL JOIN Types

**Mnemonic: "I Left Right For Cross"**

- **INNER** - Intersection only
- **LEFT** - All from left + matches from right
- **RIGHT** - All from right + matches from left
- **FULL** - All from both tables
- **CROSS** - Cartesian product

**Visual Memory:**
```
INNER:  ●●   (overlap only)
LEFT:   ●●○  (left complete)
RIGHT:  ○●●  (right complete)
FULL:   ●●●  (everything)
```

---

### Index Types Memory

**Mnemonic: "B-Trees Grow Fast, Hash Stays Put"**

- **B-Tree** - Range queries, sorted
- **Hash** - Exact matches only
- **GiST** - Geographic/full-text
- **GIN** - Array/JSONB searches

**When to Use:**
```
Range queries (>, <, BETWEEN) → B-Tree
Equality only (=) → Hash
Arrays/JSON → GIN
Geographic data → GiST
```

---

### MongoDB vs PostgreSQL Decision

**Mnemonic: "Flexible Mongo, Strict Postgres"**

**Choose MongoDB when:**
- **F**lexible schema (requirements evolving)
- **R**apid development (prototyping)
- **E**mbedded documents (hierarchical data)
- **S**caling horizontally (easy sharding)
- **H**uge writes (high write throughput)

**Choose PostgreSQL when:**
- **S**trong consistency (ACID critical)
- **T**ransactions (multi-table operations)
- **R**elationships (complex joins)
- **I**ntegrity (referential constraints)
- **C**ompliance (financial, healthcare)
- **T**ested (mature, 30+ years)

---

### Replication vs Sharding

**Mnemonic: "Read Replicates, Write Shards"**

**Replication:**
- **R**ead scaling (distribute reads)
- **R**edundancy (high availability)
- **R**ecovery (backups, failover)

**Sharding:**
- **W**rite scaling (distribute writes)
- **W**ide data (split horizontally)
- **W**orkload distribution (parallel processing)

---

### Transaction Isolation Levels

**Mnemonic: "Read Until Serialized Reality"**

1. **Read Uncommitted** - Dirty reads possible
2. **Read Committed** - See only committed data
3. **Repeatable Read** - Same query, same result
4. **Serializable** - Full isolation

**Problems Prevented:**
```
Read Committed:   Dirty reads ❌
Repeatable Read:  Non-repeatable reads ❌
Serializable:     Phantom reads ❌
```

---

### Query Optimization Steps

**Mnemonic: "EXPLAIN Before Index Join Cache"**

1. **E**XPLAIN - Analyze query plan
2. **I**ndex - Add missing indexes
3. **J**oin - Optimize join order
4. **C**ache - Add caching layer

**Quick Checklist:**
```
☐ Run EXPLAIN ANALYZE
☐ Check index usage
☐ Review WHERE clauses
☐ Optimize JOINs
☐ Add indexes if needed
☐ Consider caching
```

---

## 8.2 Top 20 Interview Questions & Answers

### Question 1: Explain ACID properties with a real example

**Answer:**
"ACID ensures reliable database transactions. Let me explain with a bank transfer:

**Atomicity**: When transferring $100 from Account A to Account B, both the debit and credit must succeed. If the credit fails, the debit is rolled back. It's all or nothing.

**Consistency**: Before the transfer, total money is $1000. After the transfer, it's still $1000. The database maintains valid constraints like non-negative balances.

**Isolation**: If two transfers happen simultaneously, they don't interfere. Each transaction sees a consistent snapshot of the data.

**Durability**: Once the transaction commits and you see the confirmation, the money transfer persists even if the system crashes immediately after.

In code:
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 'A';
UPDATE accounts SET balance = balance + 100 WHERE id = 'B';
COMMIT;
```

If anything fails, ROLLBACK undoes all changes."

---

### Question 2: What's the difference between SQL and NoSQL?

**Answer:**
"The key differences are:

**Schema:**
- SQL: Fixed schema, must define tables/columns upfront
- NoSQL: Flexible schema, can add fields anytime

**Data Model:**
- SQL: Relational tables with foreign keys
- NoSQL: Document, key-value, column-family, or graph

**Scaling:**
- SQL: Vertical scaling (bigger servers)
- NoSQL: Horizontal scaling (more servers)

**Transactions:**
- SQL: Full ACID transactions
- NoSQL: Eventual consistency (most), some support ACID

**Use Cases:**
- SQL: Financial systems, e-commerce, where data integrity is critical
- NoSQL: Social media, real-time analytics, where flexibility and scale matter

Example decision: I'd use PostgreSQL for an e-commerce platform needing strong consistency for payments, but MongoDB for a blogging platform where schema flexibility and read performance are priorities."

---

### Question 3: Explain database normalization and when to denormalize

**Answer:**
"Normalization reduces data redundancy by organizing data into related tables.

**1NF - Atomic values:**
```sql
-- ❌ Bad (repeating groups)
users: id, name, emails ('a@x.com,b@x.com')

-- ✅ Good (atomic)
users: id, name
emails: id, user_id, email
```

**2NF - No partial dependencies:**
```sql
-- ❌ Bad (product_name depends on product_id only)
order_items: order_id, product_id, product_name, quantity

-- ✅ Good (separate tables)
order_items: order_id, product_id, quantity
products: product_id, product_name
```

**3NF - No transitive dependencies:**
```sql
-- ❌ Bad (city depends on zip_code, not user_id)
users: id, name, zip_code, city

-- ✅ Good
users: id, name, zip_code
zip_codes: zip_code, city
```

**When to Denormalize:**
- Read-heavy systems (avoid JOINs)
- Data warehouses (faster aggregations)
- Historical records (preserve data at point in time)

Example: In e-commerce, I'd denormalize order_items to store product_name and price at purchase time, so historical orders aren't affected by product updates."

---

### Question 4: How do you optimize a slow query?

**Answer:**
"I follow a systematic approach:

**1. Measure First:**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders 
WHERE user_id = 123 AND created_at > '2024-01-01';
```

**2. Check Index Usage:**
- Sequential Scan → Missing index
- Index Scan → Good
- Bitmap Heap Scan → Index helpful but not perfect

**3. Add Appropriate Indexes:**
```sql
CREATE INDEX idx_orders_user_created 
ON orders(user_id, created_at);
```

**4. Optimize Query:**
```sql
-- ❌ Bad (SELECT *)
SELECT * FROM orders WHERE user_id = 123;

-- ✅ Good (specific columns)
SELECT id, total, status FROM orders WHERE user_id = 123;
```

**5. Consider Other Optimizations:**
- Add caching for frequently accessed data
- Use materialized views for complex aggregations
- Partition large tables
- Review JOIN order

**Example:** In a production system, I optimized a dashboard query from 5 seconds to 50ms by adding a composite index on (user_id, created_at) and eliminating unnecessary JOINs by denormalizing user names into the orders table."

---

### Question 5: Explain indexing and different index types

**Answer:**
"Indexes are data structures that improve query performance by allowing fast lookups without scanning the entire table.

**B-Tree Index (Default):**
- Best for: Range queries, sorting, inequalities
- Use case: `WHERE created_at > '2024-01-01'`
```sql
CREATE INDEX idx_created ON orders(created_at);
```

**Hash Index:**
- Best for: Exact equality matches
- Use case: `WHERE id = 123`
- Limitation: No range queries

**GIN (Generalized Inverted Index):**
- Best for: Arrays, JSONB, full-text search
```sql
CREATE INDEX idx_tags ON posts USING GIN(tags);
SELECT * FROM posts WHERE tags @> ARRAY['mongodb'];
```

**Partial Index:**
- Index subset of rows
```sql
CREATE INDEX idx_active_users 
ON users(email) WHERE is_active = true;
```

**Composite Index:**
- Multiple columns
```sql
CREATE INDEX idx_user_status 
ON orders(user_id, status);
```

**Trade-offs:**
- ✅ Faster reads
- ❌ Slower writes (index must be updated)
- ❌ Extra storage

**Rule of thumb:** Index columns used in WHERE, JOIN, ORDER BY clauses."

---

### Question 6: What is database sharding and when would you use it?

**Answer:**
"Sharding is horizontal partitioning where data is distributed across multiple database servers.

**How it Works:**
```
Users with ID 1-1000000   → Shard 1
Users with ID 1000001-2000000 → Shard 2
Users with ID 2000001-3000000 → Shard 3
```

**Sharding Strategies:**

**1. Range-Based:**
```javascript
function getShard(userId) {
  if (userId <= 1000000) return 'shard1';
  if (userId <= 2000000) return 'shard2';
  return 'shard3';
}
```

**2. Hash-Based:**
```javascript
function getShard(userId) {
  const shardId = userId % 3;
  return `shard${shardId}`;
}
```

**When to Use:**
- Database exceeds single server capacity
- Need to scale writes (replication only helps reads)
- Geographic distribution required

**Challenges:**
- Cross-shard JOINs are slow
- Distributed transactions are complex
- Resharding is difficult

**Example:** At a previous company, we sharded user data by user_id hash when we hit 100M users. This allowed us to scale writes linearly by adding more shards."

---

### Question 7: Explain the difference between clustered and non-clustered indexes

**Answer:**
"**Clustered Index:**
- Physical order of data matches index order
- Table data IS the index
- Only ONE per table
- Typically the primary key

```sql
-- PostgreSQL (implicit)
CREATE TABLE users (
  id SERIAL PRIMARY KEY,  -- Clustered index
  email VARCHAR
);
```

**Non-Clustered Index:**
- Separate structure from table data
- Points to actual data rows
- Can have MANY per table

```sql
CREATE INDEX idx_email ON users(email);  -- Non-clustered
```

**Visual Difference:**
```
Clustered Index (Phone Book):
- Pages physically sorted by name
- Fast range scans on indexed column

Non-Clustered Index (Book Index):
- Separate section pointing to pages
- Extra lookup required
```

**Performance Impact:**
- Clustered: Faster for range scans on PK
- Non-Clustered: Faster for specific lookups on other columns

**Example:** If you frequently query `SELECT * FROM users WHERE created_at > X`, making created_at the clustered index would be faster than the default id clustering."

---

### Question 8: What is a deadlock and how do you prevent it?

**Answer:**
"A deadlock occurs when two transactions wait for each other to release locks, creating a circular dependency.

**Example:**
```
Transaction 1:
  LOCK(Account A)
  Wait for LOCK(Account B)

Transaction 2:
  LOCK(Account B)
  Wait for LOCK(Account A)

→ Deadlock! Both waiting forever.
```

**Prevention Strategies:**

**1. Lock Ordering:**
```sql
-- ✅ Always lock accounts in ID order
BEGIN;
LOCK TABLE accounts WHERE id IN (123, 456) ORDER BY id;
UPDATE accounts SET balance = balance - 100 WHERE id = 123;
UPDATE accounts SET balance = balance + 100 WHERE id = 456;
COMMIT;
```

**2. Lock Timeout:**
```sql
SET lock_timeout = '2s';
BEGIN;
-- If lock not acquired in 2s, rollback
```

**3. Minimize Lock Duration:**
```sql
-- ❌ Bad (long transaction)
BEGIN;
-- Complex calculations
UPDATE accounts...
COMMIT;

-- ✅ Good (short transaction)
-- Do calculations first
BEGIN;
UPDATE accounts...
COMMIT;
```

**4. Use Appropriate Isolation Levels:**
```sql
-- Read Committed instead of Serializable
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

**Detection:**
```sql
-- PostgreSQL
SELECT * FROM pg_stat_activity 
WHERE wait_event_type = 'Lock';
```

Most databases automatically detect and resolve deadlocks by killing one transaction."

---

### Question 9: Explain database replication and its types

**Answer:**
"Replication is copying data from a primary database to one or more replica databases.

**Master-Slave (Primary-Replica):**
```
Primary (Writes) → Replica 1 (Reads)
                 → Replica 2 (Reads)
                 → Replica 3 (Reads)
```

**Benefits:**
- Read scaling (distribute reads across replicas)
- High availability (automatic failover)
- Geographic distribution (low latency)

**Types:**

**1. Synchronous Replication:**
```sql
-- Transaction waits for replica confirmation
BEGIN;
UPDATE accounts SET balance = balance - 100;
COMMIT;  -- Waits for replica write
```
- ✅ No data loss
- ❌ Higher latency

**2. Asynchronous Replication:**
```sql
COMMIT;  -- Returns immediately
-- Replica catches up later
```
- ✅ Low latency
- ❌ Possible data loss if primary fails

**3. Semi-Synchronous:**
- Wait for at least one replica
- Balance of both approaches

**Configuration (PostgreSQL):**
```sql
-- postgresql.conf
wal_level = replica
max_wal_senders = 3
```

**Application Pattern:**
```javascript
// Write to primary
await primaryDB.query('INSERT INTO users...');

// Read from replica
const users = await replicaDB.query('SELECT * FROM users');
```

**Challenge: Replication Lag**
```javascript
// Handle read-after-write
await primaryDB.insert(user);
// Read from primary for consistency
const inserted = await primaryDB.findOne(user.id);
```

I've used master-slave replication to scale read-heavy applications by distributing reads across 5 replicas, reducing primary load by 80%."

---

### Question 10: What is the N+1 query problem and how do you solve it?

**Answer:**
"The N+1 problem occurs when you execute 1 query to fetch N records, then N additional queries to fetch related data.

**Example Problem:**
```javascript
// 1 query for users
const users = await db.query('SELECT * FROM users LIMIT 10');

// 10 queries for posts (N+1!)
for (const user of users) {
  const posts = await db.query(
    'SELECT * FROM posts WHERE user_id = ?', 
    [user.id]
  );
  user.posts = posts;
}
// Total: 11 queries
```

**Solution 1: JOIN**
```sql
-- Single query
SELECT 
  u.id, u.name, u.email,
  p.id AS post_id, p.title, p.content
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
WHERE u.id IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
```

**Solution 2: IN Query**
```javascript
// 1 query for users
const users = await db.query('SELECT * FROM users LIMIT 10');
const userIds = users.map(u => u.id);

// 1 query for all posts
const posts = await db.query(
  'SELECT * FROM posts WHERE user_id IN (?)',
  [userIds]
);

// Group in application
const postsByUser = posts.reduce((acc, post) => {
  acc[post.user_id] = acc[post.user_id] || [];
  acc[post.user_id].push(post);
  return acc;
}, {});

users.forEach(user => {
  user.posts = postsByUser[user.id] || [];
});
// Total: 2 queries
```

**Solution 3: ORM Eager Loading**
```javascript
// Sequelize
const users = await User.findAll({
  include: [{ model: Post }]
});

// Mongoose
const users = await User.find().populate('posts');
```

**Detection:**
```javascript
// Log queries in development
db.on('query', (sql) => console.log(sql));
```

I've optimized APIs from 500ms to 50ms by fixing N+1 queries using eager loading."

---

### Question 11: Explain MVCC (Multi-Version Concurrency Control)

**Answer:**
"MVCC allows multiple transactions to access the same data simultaneously without locking by maintaining multiple versions of data.

**How It Works:**
```
Time: T1
User A reads row (version 1): balance = 100

Time: T2
User B updates row (creates version 2): balance = 50
User A still sees version 1: balance = 100

Time: T3
User A's transaction completes
User B commits
Future reads see version 2: balance = 50
```

**Benefits:**
- ✅ Readers don't block writers
- ✅ Writers don't block readers
- ✅ High concurrency

**Implementation (PostgreSQL):**
```sql
BEGIN;
SELECT * FROM accounts WHERE id = 1;
-- Sees snapshot at transaction start

-- Meanwhile, another transaction updates
-- This transaction still sees old version

COMMIT;
```

**Hidden Columns:**
```sql
-- PostgreSQL tracks versions internally
xmin: Transaction ID that created version
xmax: Transaction ID that deleted version
```

**Garbage Collection:**
```sql
-- VACUUM removes old versions
VACUUM accounts;

-- Auto-vacuum runs automatically
```

**Trade-offs:**
- ✅ Better concurrency than locking
- ❌ Extra storage for versions
- ❌ Periodic vacuuming needed

This is why PostgreSQL can handle high read concurrency without performance degradation."

---

### Question 12: What's the difference between DELETE, TRUNCATE, and DROP?

**Answer:**

**DELETE:**
```sql
DELETE FROM users WHERE created_at < '2020-01-01';
```
- Removes specific rows
- Can use WHERE clause
- Transactional (can rollback)
- Triggers fire
- Slow for large datasets
- Can be logged

**TRUNCATE:**
```sql
TRUNCATE TABLE users;
```
- Removes ALL rows
- No WHERE clause
- Fast (deallocates data pages)
- Transactional in PostgreSQL, not in MySQL
- Triggers don't fire
- Resets auto-increment

**DROP:**
```sql
DROP TABLE users;
```
- Removes entire table structure
- Data + schema gone
- Cannot rollback (DDL)
- Very fast
- Irreversible

**Comparison Table:**

| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| Speed | Slow | Fast | Fastest |
| WHERE | ✅ | ❌ | ❌ |
| Rollback | ✅ | Depends | ❌ |
| Triggers | ✅ | ❌ | ❌ |
| Structure | Keeps | Keeps | Removes |
| Auto-increment | Keeps | Resets | Removes |

**Use Cases:**
```sql
-- Clean up old data
DELETE FROM logs WHERE created_at < NOW() - INTERVAL '90 days';

-- Clear test data
TRUNCATE TABLE test_users;

-- Remove deprecated table
DROP TABLE old_feature_table;
```"

---

### Question 13: Explain database connection pooling

**Answer:**
"Connection pooling reuses database connections instead of creating new ones for each request.

**Without Pooling:**
```javascript
// Every request creates new connection
app.get('/users', async (req, res) => {
  const conn = await createConnection();  // ~100ms
  const users = await conn.query('SELECT * FROM users');
  await conn.close();
  res.json(users);
});
// 1000 requests = 1000 connections = 100 seconds overhead!
```

**With Pooling:**
```javascript
// Pool created once at startup
const pool = new Pool({
  host: 'localhost',
  database: 'mydb',
  max: 20,      // Max connections
  min: 5,       // Min idle connections
  idleTimeoutMillis: 30000
});

app.get('/users', async (req, res) => {
  const users = await pool.query('SELECT * FROM users');
  // Connection automatically returned to pool
  res.json(users);
});
// 1000 requests = 0 seconds overhead!
```

**How It Works:**
```
Request 1 → Pool → Connection A (in use)
Request 2 → Pool → Connection B (in use)
Request 3 → Pool → Wait (all busy)
Request 1 completes → Connection A returned
Request 3 → Pool → Connection A (reused)
```

**Sizing Formula:**
```
Pool Size = (Core Count × 2) + Effective Spindle Count

Example:
- 4 cores, SSD → (4 × 2) + 1 = 9 connections
```

**Configuration:**
```javascript
const pool = new Pool({
  max: 20,                    // Max connections
  min: 5,                     // Min idle
  idleTimeoutMillis: 30000,   // Close idle after 30s
  connectionTimeoutMillis: 2000  // Wait 2s for connection
});

// Monitor
pool.on('error', (err) => {
  console.error('Pool error:', err);
});
```

**Best Practices:**
- Don't create pools per request
- Size based on workload testing
- Monitor pool saturation
- Use external pooler (PgBouncer) for thousands of clients

I've used connection pooling to support 10,000 concurrent users with just 20 database connections."

---

### Question 14: What are materialized views and when would you use them?

**Answer:**
"Materialized views store query results physically, unlike regular views which execute the query each time.

**Regular View:**
```sql
CREATE VIEW user_stats AS
SELECT 
  user_id,
  COUNT(*) AS post_count,
  SUM(views) AS total_views
FROM posts
GROUP BY user_id;

-- Query executes every time
SELECT * FROM user_stats WHERE user_id = 123;
```

**Materialized View:**
```sql
CREATE MATERIALIZED VIEW user_stats AS
SELECT 
  user_id,
  COUNT(*) AS post_count,
  SUM(views) AS total_views
FROM posts
GROUP BY user_id;

CREATE INDEX idx_user_stats_user ON user_stats(user_id);

-- Fast query (pre-computed)
SELECT * FROM user_stats WHERE user_id = 123;
```

**Refresh Strategies:**

**1. Manual:**
```sql
REFRESH MATERIALIZED VIEW user_stats;
```

**2. Scheduled (with cron/scheduler):**
```sql
-- Every hour
REFRESH MATERIALIZED VIEW user_stats;
```

**3. Concurrent (non-blocking):**
```sql
-- PostgreSQL
REFRESH MATERIALIZED VIEW CONCURRENTLY user_stats;
```

**When to Use:**
- Complex, expensive aggregations
- Dashboard queries
- Reports run frequently
- Data doesn't need to be real-time

**Trade-offs:**
- ✅ Much faster queries
- ✅ Reduced load on base tables
- ❌ Stale data (until refresh)
- ❌ Extra storage
- ❌ Refresh overhead

**Example:**
```sql
-- Analytics dashboard
CREATE MATERIALIZED VIEW daily_revenue AS
SELECT 
  DATE(created_at) AS date,
  SUM(total) AS revenue,
  COUNT(*) AS order_count,
  AVG(total) AS avg_order_value
FROM orders
WHERE status = 'completed'
GROUP BY DATE(created_at);

-- Refresh nightly
-- Dashboard queries: 5 seconds → 50ms
```

I've used materialized views to optimize a complex analytics dashboard from 30 seconds to under 1 second by pre-computing aggregations."

---

### Question 15: Explain database partitioning

**Answer:**
"Partitioning divides a large table into smaller, manageable pieces while maintaining a single logical table.

**Range Partitioning:**
```sql
-- PostgreSQL
CREATE TABLE orders (
  id SERIAL,
  user_id INTEGER,
  created_at DATE,
  total DECIMAL
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2023 
  PARTITION OF orders 
  FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

CREATE TABLE orders_2024 
  PARTITION OF orders 
  FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

**Query Automatically Routes:**
```sql
-- Only scans orders_2024 partition
SELECT * FROM orders 
WHERE created_at >= '2024-01-01';
```

**List Partitioning:**
```sql
CREATE TABLE users (
  id SERIAL,
  country VARCHAR(2),
  name VARCHAR(100)
) PARTITION BY LIST (country);

CREATE TABLE users_us PARTITION OF users FOR VALUES IN ('US');
CREATE TABLE users_uk PARTITION OF users FOR VALUES IN ('UK');
CREATE TABLE users_other PARTITION OF users DEFAULT;
```

**Hash Partitioning:**
```sql
CREATE TABLE measurements (
  id SERIAL,
  sensor_id INTEGER,
  value DECIMAL
) PARTITION BY HASH (sensor_id);

CREATE TABLE measurements_0 
  PARTITION OF measurements 
  FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE measurements_1 
  PARTITION OF measurements 
  FOR VALUES WITH (MODULUS 4, REMAINDER 1);
-- etc.
```

**Benefits:**
- ✅ Improved query performance (partition pruning)
- ✅ Easier maintenance (drop old partitions)
- ✅ Better indexing (smaller indexes per partition)

**Maintenance:**
```sql
-- Drop old data (fast)
DROP TABLE orders_2020;

-- vs deleting (slow)
DELETE FROM orders WHERE created_at < '2021-01-01';
```

**When to Use:**
- Tables > 100GB
- Time-series data
- Archival strategies
- Queries filter on partition key

I've used monthly partitioning for a logs table with 500M rows, improving query performance by 10x and making archival instant."

---

### Question 16: How do you handle database migrations in production?

**Answer:**
"Safe production migrations require careful planning to avoid downtime and data loss.

**Best Practices:**

**1. Backward Compatible Changes:**
```sql
-- ❌ Bad (breaking change)
ALTER TABLE users RENAME COLUMN name TO full_name;

-- ✅ Good (gradual migration)
-- Step 1: Add new column
ALTER TABLE users ADD COLUMN full_name VARCHAR(255);

-- Step 2: Backfill data
UPDATE users SET full_name = name WHERE full_name IS NULL;

-- Step 3: Deploy code using both columns
-- Step 4: Drop old column (after verification)
ALTER TABLE users DROP COLUMN name;
```

**2. Lock-Free Migrations:**
```sql
-- ❌ Bad (locks table)
ALTER TABLE users ADD COLUMN email VARCHAR(255) NOT NULL DEFAULT '';

-- ✅ Good (no lock)
ALTER TABLE users ADD COLUMN email VARCHAR(255);
-- Backfill in batches
UPDATE users SET email = '' WHERE email IS NULL LIMIT 1000;
-- After backfill
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
```

**3. Zero-Downtime Index Creation:**
```sql
-- PostgreSQL
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
```

**4. Migration Tools:**
```javascript
// Flyway, Liquibase, or custom
// migrations/001_add_email.sql
ALTER TABLE users ADD COLUMN email VARCHAR(255);

// migrations/002_backfill_email.sql
UPDATE users SET email = 'unknown@example.com' WHERE email IS NULL;
```

**5. Rollback Plan:**
```sql
-- migration_up.sql
ALTER TABLE users ADD COLUMN email VARCHAR(255);

-- migration_down.sql
ALTER TABLE users DROP COLUMN email;
```

**6. Testing Strategy:**
```bash
# Test on production copy
pg_dump production > backup.sql
createdb test_migration
psql test_migration < backup.sql
# Run migration on test_migration
# Verify results
```

**Production Checklist:**
```
☐ Test on production copy
☐ Have rollback plan
☐ Run during low-traffic period
☐ Monitor query performance
☐ Backup before migration
☐ Gradual rollout (blue-green deployment)
```

I once led a zero-downtime migration for a 500GB table by adding the new column, backfilling in batches over 2 weeks, then switching the application code."

---

### Question 17: Explain eventual consistency vs strong consistency

**Answer:**
"**Strong Consistency:**
All clients see the same data at the same time, immediately after any write.

```
Client A writes: balance = 100
Client B reads: balance = 100 (immediately)
```

Examples: PostgreSQL, MySQL
Use case: Financial transactions, inventory

**Eventual Consistency:**
Clients may temporarily see different data, but eventually all converge to the same state.

```
Client A writes: balance = 100 (to Node 1)
Client B reads: balance = 50 (from Node 2, not yet replicated)
... few milliseconds later ...
Client B reads: balance = 100 (replicated)
```

Examples: Cassandra, DynamoDB
Use case: Social media feeds, product catalogs

**Trade-offs:**

| Aspect | Strong | Eventual |
|--------|--------|----------|
| Latency | Higher | Lower |
| Availability | Lower | Higher |
| Scalability | Harder | Easier |
| Use Case | Banking | Social Media |

**Real Example:**
```javascript
// Strong Consistency (PostgreSQL)
await db.createPost(post);
const fetched = await db.getPost(post.id);
// Guaranteed to see the post

// Eventual Consistency (DynamoDB)
await db.createPost(post);
const fetched = await db.getPost(post.id);
// Might not see it yet (stale read)
```

**Handling Eventual Consistency:**
```javascript
// Read-after-write consistency
async function createAndVerify(userId, post) {
  await db.create(post);
  
  // Read from primary (not replica)
  const created = await db.getPrimary(post.id);
  
  return created;
}
```

Choose based on requirements:
- Money, inventory → Strong consistency
- Likes, views, feeds → Eventual consistency acceptable"

---

### Question 18: What is database denormalization and when should you use it?

**Answer:**
"Denormalization is intentionally introducing redundancy to optimize read performance.

**Normalized (3NF):**
```sql
-- 3 tables, 2 JOINs required
SELECT o.id, o.total, u.name, u.email
FROM orders o
JOIN users u ON o.user_id = u.id;
```

**Denormalized:**
```sql
-- 1 table, no JOINs
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER,
  user_name VARCHAR(100),    -- Denormalized
  user_email VARCHAR(255),   -- Denormalized
  total DECIMAL
);

SELECT id, total, user_name, user_email FROM orders;
```

**When to Denormalize:**

**1. Read-Heavy Systems:**
```sql
-- Dashboard needs order count per user (millions of orders)
-- ❌ Normalized: COUNT(*) query on every request
SELECT user_id, COUNT(*) FROM orders GROUP BY user_id;

-- ✅ Denormalized: Pre-computed
ALTER TABLE users ADD COLUMN order_count INTEGER DEFAULT 0;
-- Update on order creation
```

**2. Historical Records:**
```sql
-- Order items should preserve product price at purchase time
CREATE TABLE order_items (
  order_id INTEGER,
  product_id INTEGER,
  product_name VARCHAR(255),  -- Denormalized
  price DECIMAL,              -- Denormalized (price at time of order)
  quantity INTEGER
);
```

**3. Analytics/Reporting:**
```sql
-- Denormalized fact table for fast aggregations
CREATE TABLE sales_fact (
  date DATE,
  product_id INTEGER,
  product_name VARCHAR,        -- Denormalized
  category_name VARCHAR,       -- Denormalized
  customer_country VARCHAR,    -- Denormalized
  revenue DECIMAL
);
```

**Maintaining Consistency:**
```javascript
// Update denormalized data on changes
async function updateUserName(userId, newName) {
  await db.query('UPDATE users SET name = $1 WHERE id = $2', 
    [newName, userId]);
  
  // Update denormalized copies
  await db.query('UPDATE orders SET user_name = $1 WHERE user_id = $2',
    [newName, userId]);
}
```

**Trade-offs:**
- ✅ Faster reads (no JOINs)
- ✅ Simpler queries
- ❌ Slower writes (update multiple places)
- ❌ Data inconsistency risk
- ❌ More storage

I've denormalized user data in an orders table to reduce dashboard query time from 5 seconds to 100ms by eliminating JOINs on 50M rows."

---

### Question 19: Explain database backups and recovery strategies

**Answer:**
"A solid backup strategy follows the 3-2-1 rule: 3 copies, 2 media types, 1 off-site.

**Backup Types:**

**1. Full Backup:**
```bash
# PostgreSQL
pg_dump -F c mydb > backup_full.dump

# MongoDB
mongodump --out=/backups/full
```
- Complete database copy
- Slowest, largest
- Use: Weekly/monthly

**2. Incremental Backup:**
```bash
# PostgreSQL WAL archiving
archive_command = 'cp %p /backups/wal/%f'
```
- Only changes since last backup
- Fastest, smallest
- Use: Hourly/continuously

**3. Differential Backup:**
- Changes since last full backup
- Medium speed/size
- Use: Daily

**Recovery Scenarios:**

**1. Point-in-Time Recovery (PITR):**
```bash
# Restore to 5 minutes before disaster
pg_basebackup -D /data/postgres
# recovery.conf
restore_command = 'cp /backups/wal/%f %p'
recovery_target_time = '2024-01-15 14:55:00'
```

**2. Complete Restore:**
```bash
pg_restore -d mydb backup_full.dump
```

**Backup Schedule Example:**
```
Continuous: WAL archiving (PITR)
Hourly: Incremental
Daily: Full backup (retained 30 days)
Weekly: Full backup (retained 12 weeks)
Monthly: Full backup (retained 12 months)
```

**Verification:**
```bash
# Test restore monthly
pg_restore --list backup.dump  # Verify
createdb test_restore
pg_restore -d test_restore backup.dump
# Run test queries
```

**Disaster Recovery:**
```bash
# RTO (Recovery Time Objective): 1 hour
# RPO (Recovery Point Objective): 5 minutes

# Process:
1. Detect failure (1 min)
2. Spin up new server (10 min)
3. Restore base backup (20 min)
4. Apply WAL logs (25 min)
5. Verify and switch (4 min)
# Total: 60 minutes
```

**Production Setup:**
```yaml
# Automated backups
schedule:
  - cron: "0 2 * * *"  # Daily at 2 AM
  - upload: s3://backups/
  - retention: 30 days local, 1 year S3
  - verify: weekly restore test
```

I've designed a backup system with 5-minute RPO using continuous WAL archiving and 1-hour RTO with automated failover to replicas."

---

### Question 20: How would you design a database for high availability?

**Answer:**
"High availability requires eliminating single points of failure and enabling automatic recovery.

**Architecture:**
```
               ┌─────────────┐
               │ Load Balancer│
               └──────┬───────┘
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
   ┌────────┐   ┌────────┐   ┌────────┐
   │Primary │   │Replica1│   │Replica2│
   │(Writes)│──→│(Reads) │   │(Reads) │
   └────────┘   └────────┘   └────────┘
        │
        └──→ Automatic Failover
```

**Components:**

**1. Replication (Multi-Region):**
```
Region 1 (Primary):
  - Primary database
  - 2 replicas (same region)

Region 2 (DR):
  - Async replica (different region)
```

**2. Automatic Failover:**
```javascript
// Health check
setInterval(async () => {
  try {
    await primary.ping();
  } catch (error) {
    // Promote replica
    await promoteReplica(replica1);
    await updateDNS(replica1);
  }
}, 5000);
```

**3. Connection Pooling:**
```javascript
const pool = new Pool({
  host: 'db-cluster.example.com',  // DNS that updates on failover
  max: 20,
  connectionTimeoutMillis: 2000
});
```

**4. Monitoring:**
```yaml
alerts:
  - name: database_down
    condition: pg_up == 0
    duration: 1m
    action: page_oncall
  
  - name: replication_lag
    condition: replication_lag_seconds > 60
    action: notify_team
```

**RTO/RPO Targets:**
```
RTO (Recovery Time Objective): < 5 minutes
RPO (Recovery Point Objective): < 1 minute

Achieved via:
- Synchronous replication (RPO < 1 min)
- Automatic failover (RTO < 5 min)
```

**Disaster Recovery Plan:**
```markdown
1. Primary Failure:
   - Automatic failover to replica (2 min)
   - Update DNS (1 min)
   - Verify integrity (2 min)

2. Region Failure:
   - Manual failover to DR region (15 min)
   - Acceptable data loss: < 5 min

3. Data Corruption:
   - Restore from backup (1 hour)
   - Point-in-time recovery
```

**Testing:**
```bash
# Chaos engineering - monthly drills
# 1. Kill primary database
docker kill primary-db

# 2. Verify automatic failover
# 3. Measure actual RTO
# 4. Verify data integrity
```

**Cost Optimization:**
```
Primary: High-performance SSD
Replicas (same region): SSD
DR Replica: Standard disk (lower cost)
```

I've designed a 99.99% uptime system (< 1 hour downtime/year) using multi-region replication with automatic failover achieving 3-minute RTO and 30-second RPO."

---

## 8.3 System Design Questions

### Question 1: Design Instagram's Database

**Requirements:**
- 500M users
- 100M daily active users
- 95M photos uploaded per day
- Users can follow others
- Photo feed (posts from people you follow)
- Likes and comments

**Solution:**

**1. Database Choice:**
- User data, relationships: **PostgreSQL** (ACID, relationships)
- Photo metadata, feed: **Cassandra** (write-heavy, scalable)
- Photo storage: **S3** (object storage)
- Cache: **Redis** (feed cache)

**2. Schema Design:**

**PostgreSQL (Users & Relationships):**
```sql
-- Users
CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  username VARCHAR(50) UNIQUE,
  email VARCHAR(255) UNIQUE,
  password_hash VARCHAR(255),
  created_at TIMESTAMP
);

-- Follows
CREATE TABLE follows (
  follower_id BIGINT REFERENCES users(id),
  following_id BIGINT REFERENCES users(id),
  created_at TIMESTAMP,
  PRIMARY KEY (follower_id, following_id)
);

CREATE INDEX idx_follows_follower ON follows(follower_id);
CREATE INDEX idx_follows_following ON follows(following_id);
```

**Cassandra (Photos & Feed):**
```cql
-- Photos (partition by user_id)
CREATE TABLE photos (
  photo_id UUID,
  user_id BIGINT,
  image_url TEXT,
  caption TEXT,
  created_at TIMESTAMP,
  PRIMARY KEY (user_id, created_at, photo_id)
) WITH CLUSTERING ORDER BY (created_at DESC);

-- Feed (denormalized for fast reads)
CREATE TABLE user_feed (
  user_id BIGINT,
  photo_id UUID,
  posted_by_user_id BIGINT,
  image_url TEXT,
  created_at TIMESTAMP,
  PRIMARY KEY (user_id, created_at, photo_id)
) WITH CLUSTERING ORDER BY (created_at DESC);

-- Likes (counter column)
CREATE TABLE photo_likes (
  photo_id UUID PRIMARY KEY,
  like_count COUNTER
);
```

**3. Feed Generation:**

**Option A: Pull Model (Read-Time)**
```javascript
// When user opens app
async function getUserFeed(userId) {
  // Get users they follow
  const following = await postgres.query(
    'SELECT following_id FROM follows WHERE follower_id = $1',
    [userId]
  );
  
  // Get recent photos from each
  const photos = await cassandra.execute(
    'SELECT * FROM photos WHERE user_id IN ? LIMIT 50',
    [following.map(f => f.following_id)]
  );
  
  return photos.sort((a, b) => b.created_at - a.created_at);
}
```
- ✅ Easy to implement
- ❌ Slow for users following many people

**Option B: Push Model (Write-Time) - Better**
```javascript
// When photo is uploaded
async function uploadPhoto(userId, photo) {
  const photoId = uuid();
  
  // Store photo
  await cassandra.execute(
    'INSERT INTO photos (photo_id, user_id, image_url, created_at) VALUES (?, ?, ?, ?)',
    [photoId, userId, photo.url, new Date()]
  );
  
  // Get followers
  const followers = await postgres.query(
    'SELECT follower_id FROM follows WHERE following_id = $1',
    [userId]
  );
  
  // Push to each follower's feed (fanout)
  await Promise.all(followers.map(follower =>
    cassandra.execute(
      'INSERT INTO user_feed (user_id, photo_id, posted_by_user_id, image_url, created_at) VALUES (?, ?, ?, ?, ?)',
      [follower.follower_id, photoId, userId, photo.url, new Date()]
    )
  ));
}

// Reading feed is fast
async function getFeed(userId) {
  return await cassandra.execute(
    'SELECT * FROM user_feed WHERE user_id = ? LIMIT 50',
    [userId]
  );
}
```
- ✅ Very fast reads
- ❌ High write amplification for celebrities

**Hybrid Approach (Used by Instagram):**
```javascript
// Regular users: Push model
// Celebrities (> 1M followers): Pull model

async function uploadPhoto(userId, photo) {
  const followerCount = await getFollowerCount(userId);
  
  if (followerCount < 1000000) {
    // Push to feeds
    await pushToFollowers(userId, photo);
  } else {
    // Just store photo, pull at read time
    await storePhoto(userId, photo);
  }
}
```

**4. Sharding Strategy:**
```
Users: Shard by user_id hash (even distribution)
Photos: Shard by user_id (co-locate user's photos)
Feeds: Shard by user_id (co-locate user's feed)
```

**5. Caching:**
```javascript
// Redis cache for hot feeds
async function getFeed(userId) {
  // Check cache
  const cached = await redis.get(`feed:${userId}`);
  if (cached) return JSON.parse(cached);
  
  // Fetch from Cassandra
  const feed = await cassandra.execute(
    'SELECT * FROM user_feed WHERE user_id = ? LIMIT 50',
    [userId]
  );
  
  // Cache for 5 minutes
  await redis.setex(`feed:${userId}`, 300, JSON.stringify(feed));
  
  return feed;
}
```

**Scale Numbers:**
- 95M photos/day = 1,100 photos/second
- With average 100 followers = 110,000 feed writes/second
- Cassandra can handle 100K+ writes/second per node
- Need ~10-20 Cassandra nodes

---

### Question 2: Design URL Shortener (like bit.ly)

**Requirements:**
- Generate short URLs (e.g., bit.ly/abc123)
- Redirect short URL to original URL
- Track click analytics
- 100M URLs created per month
- 10B redirects per month

**Solution:**

**1. Database Choice:**
- URLs: **PostgreSQL** or **Redis** (simple key-value)
- Analytics: **ClickHouse** or **Cassandra** (time-series)

**2. Schema:**

**PostgreSQL:**
```sql
CREATE TABLE urls (
  id BIGSERIAL PRIMARY KEY,
  short_code VARCHAR(10) UNIQUE NOT NULL,
  original_url TEXT NOT NULL,
  user_id BIGINT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  expires_at TIMESTAMP,
  click_count BIGINT DEFAULT 0
);

CREATE INDEX idx_urls_short_code ON urls(short_code);
CREATE INDEX idx_urls_user_id ON urls(user_id);

CREATE TABLE clicks (
  id BIGSERIAL PRIMARY KEY,
  short_code VARCHAR(10) NOT NULL,
  clicked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  ip_address INET,
  user_agent TEXT,
  referrer TEXT,
  country VARCHAR(2)
);

CREATE INDEX idx_clicks_short_code ON clicks(short_code);
CREATE INDEX idx_clicks_clicked_at ON clicks(clicked_at);
```

**3. Short Code Generation:**

**Option A: Hash-based**
```javascript
const crypto = require('crypto');

function generateShortCode(url) {
  const hash = crypto.createHash('md5').update(url).digest('base64');
  return hash.substring(0, 7);  // e.g., "aBc12Xy"
}
```
- ✅ Deterministic (same URL → same code)
- ❌ Collisions possible

**Option B: Counter-based (Better)**
```javascript
// Base62 encoding (a-z, A-Z, 0-9)
const BASE62 = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';

function encodeBase62(num) {
  if (num === 0) return BASE62[0];
  
  let result = '';
  while (num > 0) {
    result = BASE62[num % 62] + result;
    num = Math.floor(num / 62);
  }
  return result;
}

async function createShortURL(originalUrl, userId) {
  // Use database sequence
  const result = await db.query(`
    INSERT INTO urls (original_url, user_id, short_code)
    VALUES ($1, $2, '')
    RETURNING id
  `, [originalUrl, userId]);
  
  const id = result.rows[0].id;
  const shortCode = encodeBase62(id);
  
  await db.query(
    'UPDATE urls SET short_code = $1 WHERE id = $2',
    [shortCode, id]
  );
  
  return shortCode;
}
```
- ✅ No collisions (unique ID)
- ✅ 7 characters = 62^7 = 3.5 trillion URLs

**4. Redirect Flow:**
```javascript
async function redirect(shortCode) {
  // Check Redis cache
  let url = await redis.get(`url:${shortCode}`);
  
  if (!url) {
    // Fetch from database
    const result = await db.query(
      'SELECT original_url FROM urls WHERE short_code = $1',
      [shortCode]
    );
    
    if (result.rows.length === 0) {
      throw new Error('URL not found');
    }
    
    url = result.rows[0].original_url;
    
    // Cache for 24 hours
    await redis.setex(`url:${shortCode}`, 86400, url);
  }
  
  // Async: Increment counter & log click
  recordClick(shortCode, req);
  
  return url;
}

async function recordClick(shortCode, req) {
  // Batch writes for performance
  await queue.add({
    shortCode,
    clickedAt: new Date(),
    ip: req.ip,
    userAgent: req.headers['user-agent'],
    referrer: req.headers['referer']
  });
}
```

**5. Analytics:**
```javascript
// Get click stats
async function getStats(shortCode, days = 7) {
  const stats = await db.query(`
    SELECT 
      DATE(clicked_at) AS date,
      COUNT(*) AS clicks,
      COUNT(DISTINCT ip_address) AS unique_visitors
    FROM clicks
    WHERE short_code = $1
      AND clicked_at > NOW() - INTERVAL '${days} days'
    GROUP BY DATE(clicked_at)
    ORDER BY date
  `, [shortCode]);
  
  return stats.rows;
}
```

**6. Scale Estimates:**
```
100M URLs/month = 38 writes/second
10B redirects/month = 3,850 redirects/second

Storage:
- 100M URLs × 1KB = 100GB/month = 1.2TB/year
- Clicks: 10B × 200 bytes = 2TB/month

Solution:
- PostgreSQL primary + 5 read replicas
- Redis cache (95% hit rate)
- Partition clicks table by month
```

**7. High Availability:**
```
Primary DB → Replicas (5x)
Redis Cluster (3 nodes)
Load Balancer
Multi-region deployment
```

---

### Question 3: Design Twitter's Timeline

**Requirements:**
- 300M users
- 200M daily active users
- 500M tweets per day
- Home timeline (tweets from people you follow)
- Real-time updates

**Solution:**

**1. Database Architecture:**
```
Users & Follows → PostgreSQL (relationships)
Tweets → Cassandra (write-heavy, time-series)
Timelines → Redis (cache) + Cassandra (persistent)
Real-time → Redis Pub/Sub
```

**2. Schema:**

**PostgreSQL:**
```sql
CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  username VARCHAR(50) UNIQUE,
  created_at TIMESTAMP
);

CREATE TABLE follows (
  follower_id BIGINT,
  following_id BIGINT,
  created_at TIMESTAMP,
  PRIMARY KEY (follower_id, following_id)
);
```

**Cassandra:**
```cql
-- Tweets
CREATE TABLE tweets (
  tweet_id TIMEUUID PRIMARY KEY,
  user_id BIGINT,
  content TEXT,
  created_at TIMESTAMP
);

CREATE INDEX ON tweets(user_id);

-- Home timeline (denormalized)
CREATE TABLE home_timeline (
  user_id BIGINT,
  tweet_id TIMEUUID,
  tweet_user_id BIGINT,
  content TEXT,
  created_at TIMESTAMP,
  PRIMARY KEY (user_id, created_at, tweet_id)
) WITH CLUSTERING ORDER BY (created_at DESC);
```

**3. Timeline Generation (Hybrid Push/Pull):**

```javascript
// Post tweet
async function postTweet(userId, content) {
  const tweetId = uuidv1();  // Time-based UUID
  
  // Store tweet
  await cassandra.execute(
    'INSERT INTO tweets (tweet_id, user_id, content, created_at) VALUES (?, ?, ?, ?)',
    [tweetId, userId, content, new Date()]
  );
  
  // Fanout strategy based on follower count
  const followerCount = await getFollowerCount(userId);
  
  if (followerCount < 10000) {
    // PUSH: Fanout to followers
    await fanoutToFollowers(userId, tweetId, content);
  } else {
    // PULL: Celebrity, don't fanout
    // Followers will fetch at read time
  }
  
  // Notify real-time via Redis Pub/Sub
  await redis.publish(`timeline:${userId}`, JSON.stringify({
    type: 'new_tweet',
    tweetId,
    userId,
    content
  }));
}

async function fanoutToFollowers(userId, tweetId, content) {
  // Get followers in batches
  const followers = await getFollowers(userId);
  
  // Write to each follower's timeline
  const batch = followers.map(followerId =>
    cassandra.execute(
      'INSERT INTO home_timeline (user_id, tweet_id, tweet_user_id, content, created_at) VALUES (?, ?, ?, ?, ?)',
      [followerId, tweetId, userId, content, new Date()]
    )
  );
  
  await Promise.all(batch);
}

// Get timeline
async function getHomeTimeline(userId, limit = 50) {
  // Check Redis cache
  const cached = await redis.lrange(`timeline:${userId}`, 0, limit - 1);
  if (cached.length >= limit) {
    return cached.map(JSON.parse);
  }
  
  // Fetch from Cassandra
  let timeline = await cassandra.execute(
    'SELECT * FROM home_timeline WHERE user_id = ? LIMIT ?',
    [userId, limit]
  );
  
  // For celebrities user follows, fetch their recent tweets (pull)
  const celebs = await getCelebritiesFollowing(userId);
  const celebTweets = await Promise.all(
    celebs.map(celebId =>
      cassandra.execute(
        'SELECT * FROM tweets WHERE user_id = ? LIMIT 20',
        [celebId]
      )
    )
  );
  
  // Merge and sort
  timeline = mergeSortByTime([...timeline, ...celebTweets.flat()]);
  
  // Cache in Redis (list)
  if (timeline.length > 0) {
    await redis.del(`timeline:${userId}`);
    await redis.lpush(
      `timeline:${userId}`,
      ...timeline.map(JSON.stringify)
    );
    await redis.expire(`timeline:${userId}`, 600);  // 10 min TTL
  }
  
  return timeline;
}
```

**4. Real-time Updates:**

```javascript
// WebSocket connection
io.on('connection', (socket) => {
  const userId = socket.userId;
  
  // Subscribe to user's timeline updates
  const subscriber = redis.duplicate();
  subscriber.subscribe(`timeline:${userId}`);
  
  subscriber.on('message', (channel, message) => {
    socket.emit('new_tweet', JSON.parse(message));
  });
  
  socket.on('disconnect', () => {
    subscriber.unsubscribe();
    subscriber.quit();
  });
});
```

**5. Scale Numbers:**

```
500M tweets/day = 5,787 tweets/second

Fanout write amplification:
- Average 200 followers per user
- 5,787 tweets/sec × 200 = 1.15M timeline writes/sec

Cassandra nodes needed:
- 100K writes/sec per node
- Need 12 nodes minimum
- Use 20 nodes for headroom

Read load:
- 200M users × 10 timeline views/day = 2B reads/day
- 23,148 reads/second
- Redis cache hit rate: 90%
- Actual DB reads: 2,315/sec (easy)
```

**6. Optimizations:**

```javascript
// Pre-compute timelines for active users
async function precomputeTimelines() {
  const activeUsers = await getActiveUsers();  // Last 24h
  
  for (const userId of activeUsers) {
    await getHomeTimeline(userId);  // Warms cache
  }
}

// Run every 10 minutes
setInterval(precomputeTimelines, 600000);
```

---

## 8.4 Quick Reference Cheatsheet

### SQL Quick Reference

**SELECT Queries:**
```sql
-- Basic
SELECT name, email FROM users WHERE age > 18;

-- Aggregation
SELECT category, COUNT(*), AVG(price) 
FROM products 
GROUP BY category 
HAVING AVG(price) > 100;

-- Joins
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- Subquery
SELECT * FROM products
WHERE price > (SELECT AVG(price) FROM products);

-- Window functions
SELECT name, salary,
  RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank
FROM employees;
```

**INSERT/UPDATE/DELETE:**
```sql
-- Insert
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');

-- Update
UPDATE users SET email = 'new@example.com' WHERE id = 1;

-- Delete
DELETE FROM users WHERE created_at < '2020-01-01';

-- Upsert
INSERT INTO users (id, name, email) VALUES (1, 'Alice', 'alice@example.com')
ON CONFLICT (id) DO UPDATE SET email = EXCLUDED.email;
```

**Indexes:**
```sql
-- Create
CREATE INDEX idx_users_email ON users(email);

-- Composite
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Partial
CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;

-- Unique
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Drop
DROP INDEX idx_users_email;
```

**Transactions:**
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- or
ROLLBACK;
```

---

### MongoDB Quick Reference

**CRUD:**
```javascript
// Create
await db.collection('users').insertOne({
  name: 'Alice',
  email: 'alice@example.com'
});

// Read
await db.collection('users').findOne({ email: 'alice@example.com' });
await db.collection('users').find({ age: { $gt: 18 } }).toArray();

// Update
await db.collection('users').updateOne(
  { email: 'alice@example.com' },
  { $set: { age: 30 } }
);

// Delete
await db.collection('users').deleteOne({ email: 'alice@example.com' });
```

**Aggregation:**
```javascript
await db.collection('orders').aggregate([
  { $match: { status: 'completed' } },
  { $group: {
      _id: '$userId',
      totalSpent: { $sum: '$total' },
      orderCount: { $sum: 1 }
    }
  },
  { $sort: { totalSpent: -1 } },
  { $limit: 10 }
]).toArray();
```

**Indexes:**
```javascript
// Create
await db.collection('users').createIndex({ email: 1 }, { unique: true });

// Compound
await db.collection('orders').createIndex({ userId: 1, createdAt: -1 });

// Text search
await db.collection('posts').createIndex({ title: 'text', content: 'text' });

// List
await db.collection('users').indexes();
```

---

### Redis Quick Reference

**Strings:**
```javascript
await redis.set('user:1:name', 'Alice');
await redis.get('user:1:name');
await redis.incr('counter');
await redis.setex('session:123', 3600, 'data');  // Expires in 1h
```

**Hashes:**
```javascript
await redis.hSet('user:1', 'name', 'Alice');
await redis.hSet('user:1', 'email', 'alice@example.com');
await redis.hGetAll('user:1');
```

**Lists:**
```javascript
await redis.lPush('queue', 'job1');
await redis.rPush('queue', 'job2');
await redis.lPop('queue');
await redis.lRange('list', 0, -1);  // Get all
```

**Sets:**
```javascript
await redis.sAdd('tags', 'mongodb', 'redis');
await redis.sMembers('tags');
await redis.sIsMember('tags', 'mongodb');
```

**Sorted Sets:**
```javascript
await redis.zAdd('leaderboard', { score: 100, value: 'player1' });
await redis.zRangeWithScores('leaderboard', 0, 9, { REV: true });  // Top 10
await redis.zIncrBy('leaderboard', 10, 'player1');
```

**Pub/Sub:**
```javascript
// Subscribe
await subscriber.subscribe('channel', (message) => {
  console.log(message);
});

// Publish
await redis.publish('channel', 'Hello!');
```

---

### Performance Cheatsheet

**Optimization Checklist:**
```
☐ Add indexes on WHERE/JOIN columns
☐ Use EXPLAIN ANALYZE
☐ Avoid SELECT *
☐ Use connection pooling
☐ Cache frequently accessed data
☐ Denormalize for read-heavy loads
☐ Partition large tables
☐ Use prepared statements
☐ Batch operations
☐ Monitor slow queries
```

**Index Decision Tree:**
```
Need range queries? → B-Tree
Exact equality only? → Hash
Arrays/JSON? → GIN
Full-text search? → GIN or text index
Geographic data? → GiST
```

**Scaling Decision Tree:**
```
Read-heavy? → Add read replicas
Write-heavy? → Shard database
Need consistency? → PostgreSQL + replication
Need flexibility? → MongoDB
Need speed? → Redis cache
```

---

### Common Pitfalls

**❌ Don't:**
```sql
-- SELECT *
SELECT * FROM users;  -- Use specific columns

-- No indexes
WHERE email = 'alice@example.com';  -- Add index on email

-- N+1 queries
for (user in users) {
  getOrders(user.id);  -- Use JOIN or IN query
}

-- String concatenation
"SELECT * FROM users WHERE id = " + userId;  -- SQL injection!

-- No connection pooling
createConnection() on every request  -- Reuse connections
```

**✅ Do:**
```sql
-- Specific columns
SELECT id, name, email FROM users;

-- Proper indexes
CREATE INDEX idx_users_email ON users(email);

-- Single query
SELECT u.*, o.* FROM users u LEFT JOIN orders o ON u.id = o.user_id;

-- Parameterized queries
SELECT * FROM users WHERE id = $1;

-- Connection pooling
const pool = new Pool({ max: 20 });
```

---


