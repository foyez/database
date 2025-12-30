# Database

## Database Fundamentals

### What is a Database?

**Definition:** A database is an organized, structured collection of data stored electronically in a computer system, designed for efficient storage, retrieval, manipulation, and management of information.

#### Real-Life Analogy: Library System

Think of a **library**:
- **Database** = The entire library building and its organization system
- **Tables/Collections** = Different sections (Fiction, Non-fiction, Reference, Magazines)
- **Rows/Documents** = Individual books or items
- **Columns/Fields** = Book attributes (Title, Author, ISBN, Year, Genre)
- **Primary Key** = Unique ID (ISBN number)
- **Schema** = The cataloging system (Dewey Decimal Classification)
- **Query** = Asking the librarian "Find me all books by author X"
- **Index** = The card catalog for quick lookups
- **Relationships** = "Books written by this author", "Books in this series"

### Why Do We Need Databases?

**Problem: Application without Database**
```
User Request → Server (stores everything in memory) → Response

❌ Server crashes = All data lost forever
❌ Multiple servers = Different data on each (inconsistent)
❌ Can't handle millions of records efficiently
❌ No data persistence (data disappears when server stops)
❌ No concurrent access control (users overwrite each other)
❌ No backup/recovery mechanism
```

**Solution: Application with Database**
```
User Request → Stateless Server → Database → Response

✅ Data persists even if server crashes
✅ Multiple servers can share same database (consistent)
✅ Optimized for large-scale data storage (billions of records)
✅ ACID guarantees for data integrity
✅ Concurrent access with locks and transactions
✅ Backup and recovery mechanisms
✅ Indexing for fast searches
✅ Security and access control
```

### Types of Databases Overview

```
Databases
├── SQL (Relational)
│   ├── PostgreSQL
│   ├── MySQL
│   ├── Oracle
│   ├── Microsoft SQL Server
│   └── SQLite
│
└── NoSQL (Non-Relational)
    ├── Document-Oriented
    │   ├── MongoDB
    │   ├── CouchDB
    │   └── Firebase
    │
    ├── Key-Value
    │   ├── Redis
    │   ├── Memcached
    │   └── DynamoDB
    │
    ├── Column-Family
    │   ├── Cassandra
    │   ├── HBase
    │   └── ScyllaDB
    │
    ├── Graph
    │   ├── Neo4j
    │   ├── ArangoDB
    │   └── Amazon Neptune
    │
    └── Time-Series
        ├── InfluxDB
        ├── TimescaleDB
        └── Prometheus
```

---

## Database Terminology

### Core Concepts Across All Databases

#### 1. **Database**
The top-level container that holds all related data.

**SQL:**
```sql
-- Create database
CREATE DATABASE ecommerce_db;

-- Use database
USE ecommerce_db;
```

**MongoDB:**
```javascript
// Create/switch to database (implicit)
use blog_db;

// Or using driver
const db = client.db('blog_db');
```

**Redis:**
```bash
# Redis has 16 databases (0-15)
SELECT 0  # Use database 0
SELECT 1  # Use database 1
```

---

#### 2. **Schema**

**Definition:** The structure, organization, and constraints that define how data is stored.

**SQL Schema (Fixed/Rigid):**
```sql
-- Explicit schema definition
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  age INTEGER CHECK (age >= 0),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Every row MUST follow this structure
-- Can't add a field without ALTER TABLE
```

**MongoDB Schema (Flexible/Dynamic):**
```javascript
// No explicit schema required (schema-less)
// Each document can have different fields

// Document 1
{
  _id: ObjectId("..."),
  name: "Foyez",
  email: "foyez@example.com",
  age: 25
}

// Document 2 - different fields!
{
  _id: ObjectId("..."),
  name: "Alice",
  email: "alice@example.com",
  phoneNumber: "123-456-7890",
  address: {
    city: "Dhaka",
    country: "Bangladesh"
  }
}

// Both valid in same collection!
```

**Schema Validation (Optional in MongoDB):**
```javascript
// You CAN enforce schema if needed
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "email"],
      properties: {
        name: { bsonType: "string", maxLength: 100 },
        email: { bsonType: "string", pattern: "^.+@.+$" },
        age: { bsonType: "int", minimum: 0 }
      }
    }
  }
});
```

---

### SQL Terminology

| SQL Term | Definition | Example |
|----------|------------|---------|
| **Database** | Container for related tables | `ecommerce_db` |
| **Table** | Collection of related data organized in rows and columns | `users`, `products`, `orders` |
| **Row** (Record/Tuple) | Single entry in a table | User: `{id: 1, name: "Foyez", email: "foyez@example.com"}` |
| **Column** (Field/Attribute) | Property or attribute of data | `id`, `name`, `email`, `age` |
| **Primary Key** | Unique identifier for each row | `id` (must be unique, not null) |
| **Foreign Key** | Reference to primary key in another table | `user_id` in orders table → `id` in users table |
| **Schema** | Structure defining tables, columns, types, constraints | Fixed structure, must be defined upfront |
| **Index** | Data structure for faster lookups | `CREATE INDEX idx_email ON users(email)` |
| **View** | Virtual table based on query | `CREATE VIEW active_users AS SELECT * FROM users WHERE active = true` |
| **Constraint** | Rules for data integrity | `NOT NULL`, `UNIQUE`, `CHECK`, `FOREIGN KEY` |
| **Transaction** | Group of operations (all or nothing) | `BEGIN; UPDATE...; UPDATE...; COMMIT;` |
| **Query** | Request to retrieve or modify data | `SELECT * FROM users WHERE age > 18` |

**SQL Example:**
```sql
-- Table structure (schema)
CREATE TABLE users (
  id SERIAL PRIMARY KEY,           -- Primary Key
  name VARCHAR(100) NOT NULL,      -- Column with constraint
  email VARCHAR(255) UNIQUE,       -- Unique constraint
  age INTEGER CHECK (age >= 0),    -- Check constraint
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),  -- Foreign Key
  total DECIMAL(10, 2) NOT NULL,
  status VARCHAR(20) DEFAULT 'pending'
);

-- Insert row
INSERT INTO users (name, email, age) 
VALUES ('Foyez', 'foyez@example.com', 25);

-- Query (retrieve data)
SELECT name, email FROM users WHERE age > 18;
```

---

### NoSQL Terminology

#### MongoDB (Document-Oriented) Terminology

| MongoDB Term | SQL Equivalent | Definition | Example |
|--------------|----------------|------------|---------|
| **Database** | Database | Container for collections | `blog_db` |
| **Collection** | Table | Group of related documents | `users`, `posts`, `comments` |
| **Document** | Row/Record | Single JSON-like entry (BSON) | `{_id: ObjectId("..."), name: "Foyez", email: "..."}` |
| **Field** | Column | Property in a document | `name`, `email`, `age` |
| **_id** | Primary Key | Unique identifier (auto-generated ObjectId) | `ObjectId("507f1f77bcf86cd799439011")` |
| **Embedded Document** | JOIN (denormalized) | Nested object within document | `address: {city: "Dhaka", country: "BD"}` |
| **Reference** | Foreign Key | ObjectId pointing to another document | `userId: ObjectId("...")` |
| **Index** | Index | Data structure for faster lookups | `db.users.createIndex({email: 1})` |
| **Aggregation Pipeline** | Complex Query/JOIN | Multi-stage data processing | `db.collection.aggregate([...])` |

**MongoDB Example:**
```javascript
// Collection (like SQL table)
db.users

// Document (like SQL row, but flexible)
{
  _id: ObjectId("507f1f77bcf86cd799439011"),  // Primary key (auto-generated)
  name: "Foyez Ahmed",                        // Field
  email: "foyez@example.com",                 // Field
  age: 25,                                    // Field
  
  // Embedded document (nested object)
  address: {
    street: "123 Main St",
    city: "Cumilla",
    country: "Bangladesh"
  },
  
  // Array field
  phoneNumbers: [
    { type: "home", number: "01711-123456" },
    { type: "work", number: "01811-654321" }
  ],
  
  // Reference (like foreign key)
  favoritePostId: ObjectId("post_id_here"),
  
  created_at: ISODate("2024-01-15T10:30:00Z")
}

// Query
db.users.find({ age: { $gt: 18 } });

// Insert
db.users.insertOne({
  name: "Alice",
  email: "alice@example.com",
  age: 28
});
```

---

#### Redis (Key-Value) Terminology

| Redis Term | SQL Equivalent | Definition | Example |
|------------|----------------|------------|---------|
| **Database** | Database | Numbered namespace (0-15) | `SELECT 0` |
| **Key** | Primary Key | Unique identifier | `"user:1:name"`, `"session:abc123"` |
| **Value** | Row/Data | Data associated with key | `"Foyez Ahmed"`, `{"id":1,"name":"Foyez"}` |
| **String** | VARCHAR | Simple text value | `SET name "Foyez"` |
| **Hash** | Row | Field-value pairs | `HSET user:1 name "Foyez" email "foyez@example.com"` |
| **List** | Array | Ordered collection | `LPUSH mylist "item1" "item2"` |
| **Set** | Set | Unordered unique values | `SADD myset "value1" "value2"` |
| **Sorted Set** | Sorted Set | Set with scores | `ZADD leaderboard 100 "player1"` |
| **TTL** | Expiration | Time-to-live (auto-delete) | `EXPIRE key 300` (5 minutes) |

**Redis Example:**
```bash
# String (simple key-value)
SET user:1:name "Foyez Ahmed"
GET user:1:name
# Returns: "Foyez Ahmed"

# Hash (like a row with columns)
HSET user:1 name "Foyez" email "foyez@example.com" age 25
HGETALL user:1
# Returns: 
# 1) "name"
# 2) "Foyez"
# 3) "email"
# 4) "foyez@example.com"
# 5) "age"
# 6) "25"

# List (ordered collection)
LPUSH cart:user:1 "product:101" "product:102"
LRANGE cart:user:1 0 -1
# Returns: ["product:102", "product:101"]

# Set (unique values)
SADD likes:post:1 "user:1" "user:2" "user:3"
SMEMBERS likes:post:1
# Returns: ["user:1", "user:2", "user:3"]

# Sorted Set (leaderboard)
ZADD leaderboard 100 "player1" 95 "player2" 110 "player3"
ZRANGE leaderboard 0 -1 WITHSCORES
# Returns: ["player2", "95", "player1", "100", "player3", "110"]

# TTL (expiration)
SET session:abc123 "user_id:1" EX 3600  # Expires in 1 hour
TTL session:abc123  # Check remaining time
```

---

#### Cassandra (Column-Family) Terminology

| Cassandra Term | SQL Equivalent | Definition | Example |
|----------------|----------------|------------|---------|
| **Keyspace** | Database | Top-level container | `ecommerce_keyspace` |
| **Table** | Table | Collection of rows | `users`, `orders` |
| **Row** | Row | Single entry identified by partition key | `{id: 1, name: "Foyez", ...}` |
| **Column** | Column | Key-value pair | `name: "Foyez"` |
| **Partition Key** | Primary Key | Determines data distribution | `user_id` |
| **Clustering Key** | Sort Key | Determines sort order within partition | `timestamp` |
| **Wide Row** | Denormalized Row | Row with many dynamic columns | Time-series data |

---

#### Neo4j (Graph) Terminology

| Neo4j Term | SQL Equivalent | Definition | Example |
|------------|----------------|------------|---------|
| **Node** | Row | Entity in the graph | `(user:Person {name: "Foyez"})` |
| **Relationship** | Foreign Key/JOIN | Connection between nodes | `(user)-[:FOLLOWS]->(other_user)` |
| **Property** | Column | Attribute of node/relationship | `name`, `email`, `since` |
| **Label** | Table | Category/type of node | `:Person`, `:Product`, `:Post` |
| **Cypher** | SQL | Query language | `MATCH (u:User) WHERE u.age > 18 RETURN u` |

**Neo4j Example:**
```cypher
// Node (like SQL row)
(user:Person {name: "Foyez", email: "foyez@example.com"})

// Relationship (like SQL foreign key, but first-class)
(alice:Person)-[:FOLLOWS]->(bob:Person)
(alice)-[:LIKES]->(post:Post)
(alice)-[:COMMENTED {text: "Great post!"}]->(post)

// Query
MATCH (user:Person)-[:FOLLOWS]->(friend:Person)
WHERE user.name = "Foyez"
RETURN friend.name
```

---

### Terminology Comparison Table

| Concept | SQL | MongoDB | Redis | Cassandra | Neo4j |
|---------|-----|---------|-------|-----------|-------|
| **Container** | Database | Database | Database (0-15) | Keyspace | Database |
| **Data Group** | Table | Collection | N/A | Table/Column Family | Label |
| **Single Entry** | Row | Document | Key-Value | Row | Node |
| **Property** | Column | Field | N/A | Column | Property |
| **Unique ID** | Primary Key | _id (ObjectId) | Key | Partition Key | Node ID |
| **Link** | Foreign Key | Reference | N/A | N/A | Relationship |
| **Nested Data** | JOIN | Embedded Doc | Hash | N/A | N/A |
| **Query Language** | SQL | MQL/Aggregation | Commands | CQL | Cypher |
| **Structure** | Fixed Schema | Flexible | Key-Value | Schema | Graph |

---

## SQL Databases (Relational)

### Definition

**SQL (Structured Query Language) Databases**, also called **Relational Databases**, store data in structured tables with predefined schemas, where relationships between data are established through foreign keys.

### Key Characteristics

1. **Fixed Schema** - Structure must be defined before inserting data
2. **ACID Compliant** - Guarantees data integrity
3. **Relationships** - Tables linked via foreign keys
4. **SQL Language** - Standardized query language
5. **Vertical Scaling** - Scale by increasing server power
6. **Strong Consistency** - Immediate consistency across all reads

### Structure Example

```sql
-- Tables with fixed columns
users table:
+----+------------+----------------------+----------+
| id | name       | email                | city     |
+----+------------+----------------------+----------+
| 1  | Foyez      | foyez@example.com    | Cumilla  |
| 2  | Alice      | alice@example.com    | Dhaka    |
+----+------------+----------------------+----------+

orders table:
+----+---------+--------+-----------+
| id | user_id | total  | status    |
+----+---------+--------+-----------+
| 1  | 1       | 150.00 | completed |
| 2  | 1       |  75.50 | pending   |
| 3  | 2       | 200.00 | completed |
+----+---------+--------+-----------+

-- Relationships through foreign keys
orders.user_id → users.id
```

### Popular SQL Databases

#### 1. PostgreSQL
- **Type:** Open-source, object-relational
- **Best for:** Complex applications, data integrity
- **Features:** JSONB support, advanced queries, extensions
- **Use cases:** E-commerce, financial systems, analytics

#### 2. MySQL
- **Type:** Open-source, relational
- **Best for:** Web applications, read-heavy workloads
- **Features:** Fast reads, easy setup, large community
- **Use cases:** WordPress, content management, web apps

#### 3. SQLite
- **Type:** Embedded, serverless
- **Best for:** Mobile apps, small applications
- **Features:** Zero configuration, single file
- **Use cases:** Mobile apps, desktop apps, prototyping

#### 4. Oracle Database
- **Type:** Commercial, enterprise
- **Best for:** Large enterprises, mission-critical
- **Features:** High performance, scalability, security
- **Use cases:** Banking, enterprise resource planning

#### 5. Microsoft SQL Server
- **Type:** Commercial, enterprise
- **Best for:** .NET applications, Windows ecosystem
- **Features:** Integration with Microsoft tools
- **Use cases:** Enterprise applications, business intelligence

### When to Use SQL Databases

✅ **Use SQL When:**
- Complex relationships between data entities
- ACID transactions required (banking, payments)
- Data integrity is critical
- Need complex queries (JOINs, aggregations)
- Well-defined, stable schema
- Strong consistency required
- Regulatory compliance needed

**Examples:**
- Banking systems
- E-commerce platforms (orders, payments)
- Healthcare systems
- ERP/CRM systems
- Financial applications

---

## NoSQL Databases (Non-Relational)

### Definition

**NoSQL (Not Only SQL)** databases are non-relational databases designed for specific data models and have flexible schemas for building modern applications. They excel in distributed data stores with high scalability and performance requirements.

### Why "NoSQL"?

The term doesn't mean "No SQL" but rather "Not Only SQL" - many NoSQL databases support SQL-like query languages while offering flexibility beyond traditional relational models.

### Key Characteristics

1. **Flexible Schema** - Structure can evolve over time
2. **Horizontal Scaling** - Scale by adding more servers
3. **High Performance** - Optimized for specific use cases
4. **Eventually Consistent** - Prioritizes availability over immediate consistency
5. **Distributed Architecture** - Built for cloud and distributed systems
6. **Specialized** - Different types for different needs

---

### Types of NoSQL Databases

### 1. Document-Oriented Databases

**Definition:** Store data as JSON-like documents. Each document can have different structure.

**Popular:** MongoDB, CouchDB, Firebase Firestore

**Structure:**
```javascript
// MongoDB Collection
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  name: "Foyez Ahmed",
  email: "foyez@example.com",
  age: 25,
  address: {                    // Nested object
    city: "Cumilla",
    country: "Bangladesh"
  },
  hobbies: ["coding", "reading"],  // Array
  // Can add any field!
  favoriteColor: "blue"
}
```

**✅ Best For:**
- Content management systems
- User profiles with varying attributes
- Product catalogs (different products, different attributes)
- Hierarchical data
- Rapid prototyping

**✅ Pros:**
- Flexible schema (add fields anytime)
- Natural for JavaScript/JSON applications
- Embedded documents (no JOINs needed)
- Horizontal scaling
- Fast reads

**❌ Cons:**
- No built-in joins (need application-level)
- Data duplication
- Limited ACID transactions (improving)
- Complex queries harder than SQL

**Example Use Case - E-Commerce Product Catalog:**
```javascript
// Different products, different attributes
// Electronics
{
  _id: 1,
  type: "electronics",
  name: "Laptop",
  price: 1299.99,
  specs: {
    cpu: "Intel i7",
    ram: "16GB",
    storage: "512GB SSD"
  }
}

// Clothing
{
  _id: 2,
  type: "clothing",
  name: "T-Shirt",
  price: 29.99,
  sizes: ["S", "M", "L", "XL"],
  colors: ["red", "blue", "black"],
  material: "cotton"
}

// No fixed schema needed!
```

---

### 2. Key-Value Databases

**Definition:** Simplest NoSQL model. Each item stored as key-value pair. Think of it as a giant hash map.

**Popular:** Redis, Memcached, DynamoDB, Riak

**Structure:**
```bash
Key                    Value
-------------------------------------------
"user:1:name"       -> "Foyez Ahmed"
"session:abc123"    -> '{"userId":1,"expires":1234567890}'
"cart:user:1"       -> '["product:101","product:102"]'
"cache:trending"    -> '[{"id":1,"name":"Laptop"},{...}]'
```

**Data Types in Redis:**

```bash
# 1. String (simple value)
SET user:1:name "Foyez Ahmed"
GET user:1:name
# Returns: "Foyez Ahmed"

# 2. Hash (like a row with columns)
HSET user:1 name "Foyez" email "foyez@example.com" age "25"
HGET user:1 name
# Returns: "Foyez"
HGETALL user:1
# Returns: {"name": "Foyez", "email": "foyez@example.com", "age": "25"}

# 3. List (ordered collection, can have duplicates)
LPUSH cart:user:1 "product:101"
LPUSH cart:user:1 "product:102"
LRANGE cart:user:1 0 -1
# Returns: ["product:102", "product:101"]

# 4. Set (unordered, unique values)
SADD likes:post:1 "user:1"
SADD likes:post:1 "user:2"
SADD likes:post:1 "user:1"  # Duplicate ignored
SMEMBERS likes:post:1
# Returns: ["user:1", "user:2"]

# 5. Sorted Set (set with scores, ordered)
ZADD leaderboard 100 "player1"
ZADD leaderboard 95 "player2"
ZADD leaderboard 110 "player3"
ZRANGE leaderboard 0 -1 WITHSCORES
# Returns: ["player2", "95", "player1", "100", "player3", "110"]

# 6. Bitmaps
SETBIT user:visited:2024-12-29 1 1  # User 1 visited
SETBIT user:visited:2024-12-29 2 1  # User 2 visited
BITCOUNT user:visited:2024-12-29
# Returns: 2 (2 users visited)

# 7. HyperLogLog (cardinality estimation)
PFADD unique:visitors "user1" "user2" "user1"
PFCOUNT unique:visitors
# Returns: 2 (approximate unique count)

# 8. Streams (append-only log)
XADD events:clicks * user "user1" product "laptop"
XREAD STREAMS events:clicks 0
```

**✅ Best For:**
- Caching (most common use case)
- Session storage
- Real-time features (counters, leaderboards)
- Rate limiting
- Pub/Sub messaging
- Shopping carts

**✅ Pros:**
- **Extremely fast** (in-memory, < 1ms latency)
- Simple API (GET, SET, DELETE)
- TTL support (auto-expiration)
- Rich data structures
- Pub/Sub messaging

**❌ Cons:**
- No complex queries (only get by key)
- No relationships
- Limited by RAM (memory-based)
- No ad-hoc queries

**Example Use Cases:**

**Caching:**
```javascript
// Check cache first
const cached = await redis.get('product:101');
if (cached) {
  return JSON.parse(cached);  // Fast! ~1ms
}

// Cache miss - query database
const product = await db.query('SELECT * FROM products WHERE id = 101');
// Takes ~50ms

// Store in cache for 5 minutes
await redis.setex('product:101', 300, JSON.stringify(product));

return product;
```

**Session Storage:**
```javascript
// Create session
await redis.setex(
  `session:${sessionId}`,
  3600,  // Expire in 1 hour
  JSON.stringify({ userId: 1, role: 'admin' })
);

// Retrieve session
const session = await redis.get(`session:${sessionId}`);
```

**Rate Limiting:**
```javascript
// Allow 100 requests per minute per user
const key = `rate_limit:${userId}:${currentMinute}`;
const requests = await redis.incr(key);

if (requests === 1) {
  // First request - set expiration
  await redis.expire(key, 60);
}

if (requests > 100) {
  throw new Error('Rate limit exceeded');
}
```

**Leaderboard:**
```javascript
// Add score
await redis.zadd('game:leaderboard', score, userId);

// Get top 10
const top10 = await redis.zrange('game:leaderboard', 0, 9, 'WITHSCORES');

// Get user rank
const rank = await redis.zrevrank('game:leaderboard', userId);
```

---

### 3. Column-Family Databases

**Definition:** Store data in columns rather than rows. Optimized for queries on large datasets.

**Popular:** Cassandra, HBase, ScyllaDB

**Structure:**
```
Row Key: user:1
  ├── Column Family: info
  │   ├── name: "Foyez Ahmed"
  │   ├── email: "foyez@example.com"
  │   └── age: 25
  │
  └── Column Family: activity
      ├── last_login: "2024-12-29T10:30:00Z"
      ├── total_posts: 42
      └── total_comments: 156

Row Key: user:2
  ├── Column Family: info
  │   ├── name: "Alice"
  │   └── email: "alice@example.com"
  │
  └── Column Family: activity
      ├── last_login: "2024-12-28T15:20:00Z"
      └── total_posts: 28
```

**✅ Best For:**
- Write-heavy workloads
- Time-series data
- IoT sensor data
- Large-scale analytics
- Event logging

**✅ Pros:**
- **Massive scale** (petabytes of data)
- **High availability** (no single point of failure)
- **Fast writes** (optimized for writes)
- Handles time-series data excellently
- Linear scalability

**❌ Cons:**
- Complex to set up and manage
- No joins
- Eventually consistent
- Limited query flexibility
- Steep learning curve

**Example Use Case - IoT Sensor Data:**
```javascript
// Cassandra
CREATE TABLE sensor_data (
  sensor_id text,
  timestamp timestamp,
  temperature decimal,
  humidity decimal,
  PRIMARY KEY (sensor_id, timestamp)
);

// Partition by sensor_id, sort by timestamp
// Optimized for: "Get all readings for sensor X between dates Y and Z"
SELECT * FROM sensor_data 
WHERE sensor_id = 'sensor_123' 
AND timestamp >= '2024-12-01' 
AND timestamp < '2024-12-31';
```

---

### 4. Graph Databases

**Definition:** Store data as nodes (entities) and edges (relationships). Optimized for traversing connections.

**Popular:** Neo4j, ArangoDB, Amazon Neptune

**Structure:**
```
(Foyez:Person)
  |
  |--[:FOLLOWS]-->(Alice:Person)
  |--[:FOLLOWS]-->(Bob:Person)
  |
  |--[:WROTE]-->(Post1:BlogPost)
  |             |
  |             |--[:TAGGED]-->(MongoDB:Tag)
  |             |--[:TAGGED]-->(Database:Tag)
  |
  |--[:LIKES]-->(Post2:BlogPost)
                |
                |--[:WRITTEN_BY]-->(Alice:Person)
```

**✅ Best For:**
- Social networks (followers, friends)
- Recommendation engines
- Fraud detection (connection patterns)
- Knowledge graphs
- Network analysis

**✅ Pros:**
- **Natural relationship modeling**
- **Fast graph traversals** (find friends of friends)
- Flexible schema
- Powerful query language (Cypher)

**❌ Cons:**
- Specialized use case
- Different query language (not SQL)
- Scalability challenges
- Not for general-purpose data

**Example Use Case - Social Network:**

```cypher
// Create nodes and relationships
CREATE (foyez:Person {name: "Foyez", email: "foyez@example.com"})
CREATE (alice:Person {name: "Alice", email: "alice@example.com"})
CREATE (bob:Person {name: "Bob", email: "bob@example.com"})

CREATE (foyez)-[:FOLLOWS]->(alice)
CREATE (foyez)-[:FOLLOWS]->(bob)
CREATE (alice)-[:FOLLOWS]->(bob)

// Query: Find who Foyez follows
MATCH (foyez:Person {name: "Foyez"})-[:FOLLOWS]->(person)
RETURN person.name

// Query: Find friends of friends (2nd degree connections)
MATCH (foyez:Person {name: "Foyez"})-[:FOLLOWS]->()-[:FOLLOWS]->(fof)
WHERE NOT (foyez)-[:FOLLOWS]->(fof)  // Exclude existing friends
RETURN fof.name

// Query: Recommend products based on friends' purchases
MATCH (me:Person {name: "Foyez"})-[:FOLLOWS]->(friend)-[:PURCHASED]->(product)
WHERE NOT (me)-[:PURCHASED]->(product)
RETURN product.name, COUNT(*) as friend_purchases
ORDER BY friend_purchases DESC
LIMIT 10
```

---

### 5. Time-Series Databases

**Definition:** Optimized for time-stamped data. Handles massive volumes of sequential data.

**Popular:** InfluxDB, TimescaleDB (PostgreSQL extension), Prometheus

**Structure:**
```
Measurement: cpu_usage
  Tags: host=server1, region=us-east
  Fields: value=75.5
  Timestamp: 2024-12-29T10:30:00Z

Measurement: cpu_usage
  Tags: host=server1, region=us-east
  Fields: value=78.2
  Timestamp: 2024-12-29T10:31:00Z
```

**✅ Best For:**
- IoT sensor data
- Application metrics
- Server monitoring
- Financial tick data
- Log aggregation

**✅ Pros:**
- Optimized for time-based queries
- Efficient storage (compression)
- Built-in downsampling
- Fast aggregations over time ranges

**❌ Cons:**
- Specialized for time-series only
- Not for general-purpose data
- Limited query types

**Example Use Case - Application Monitoring:**
```javascript
// InfluxDB
// Write point
influx.writePoint({
  measurement: 'api_response_time',
  tags: { endpoint: '/api/users', method: 'GET' },
  fields: { value: 42.5 },
  timestamp: new Date()
});

// Query: Average response time over last hour
SELECT MEAN(value) 
FROM api_response_time 
WHERE time > now() - 1h 
GROUP BY time(1m)
```

---

### NoSQL Comparison Table

| Type | Example | Best For | Query Complexity | Scalability | Consistency |
|------|---------|----------|------------------|-------------|-------------|
| **Document** | MongoDB | CMS, catalogs, user profiles | Medium | High | Eventual |
| **Key-Value** | Redis | Caching, sessions, real-time | Very Low | Very High | Eventual |
| **Column-Family** | Cassandra | Time-series, IoT, logs | Low | Very High | Eventual |
| **Graph** | Neo4j | Social networks, recommendations | High | Medium | Strong |
| **Time-Series** | InfluxDB | Metrics, monitoring, sensors | Medium | High | Eventual |

---

### When to Use NoSQL Databases

✅ **Use NoSQL When:**
- Flexible/evolving schema
- Horizontal scaling needed (millions of users)
- High write throughput
- Hierarchical/nested data
- Eventual consistency acceptable
- Specific use case (caching, time-series, graphs)

**Examples:**
- Social media platforms (MongoDB)
- Real-time analytics (Cassandra)
- Session management (Redis)
- Recommendation engines (Neo4j)
- IoT applications (InfluxDB)

---

## SQL vs NoSQL Decision Guide

### The Big Picture

```
Choose SQL if:                     Choose NoSQL if:
├── Complex relationships          ├── Flexible schema
├── ACID transactions critical     ├── Horizontal scaling needed
├── Well-defined schema            ├── High write throughput
├── Complex queries/reporting      ├── Hierarchical data
├── Data integrity critical        ├── Eventual consistency OK
└── Regulatory compliance          └── Specialized use case
```

### SQL Databases (Relational)

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

---

#### Type 2: Key-Value Store (Redis, DynamoDB)

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

---

#### Type 3: Column-Family (Cassandra, HBase)

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

---

#### Type 4: Graph Database (Neo4j, ArangoDB)

**✅ Pros:**
- **Relationship queries** (social networks, recommendations)
- **Fast traversals** (find connections quickly)
- **Natural modeling** (follows how we think about relationships)

**❌ Cons:**
- Specialized use case
- Different query language (Cypher)
- Not for general-purpose data

---

### Decision Matrix

| Factor | SQL | NoSQL |
|--------|-----|-------|
| **Data Structure** | Fixed schema, structured | Flexible schema, semi-structured |
| **Relationships** | Complex (JOINs, foreign keys) | Simple or embedded |
| **Transactions** | Full ACID support | Limited (improving) |
| **Consistency** | Strong consistency | Eventual consistency |
| **Scalability** | Vertical (scale up) | Horizontal (scale out) |
| **Queries** | Complex SQL queries | Simple lookups, aggregations |
| **Use Case** | Banking, ERP, e-commerce | Social media, IoT, caching |
| **Learning Curve** | Moderate (SQL is standard) | Varies by type |
| **Data Integrity** | Enforced by constraints | Application-level |

### Real-World Decision Examples

#### Example 1: Banking Application
**Requirements:** Handle transactions, account balances, transfers
**Decision:** **SQL (PostgreSQL)**
**Why:**
- ACID transactions critical (money involved!)
- Strong consistency needed
- Complex relationships (accounts, transactions, users)
- Regulatory compliance required

#### Example 2: Social Media Platform
**Requirements:** User profiles, posts, comments, likes
**Decision:** **Hybrid (PostgreSQL + MongoDB + Redis + Neo4j)**
**Why:**
- PostgreSQL: User accounts, authentication (need ACID)
- MongoDB: Posts, comments (flexible content)
- Redis: Feed cache, sessions, trending (speed)
- Neo4j: Friend recommendations (graph relationships)

#### Example 3: E-Commerce Platform
**Requirements:** Products, orders, payments, inventory
**Decision:** **SQL (PostgreSQL) + Redis + Elasticsearch**
**Why:**
- PostgreSQL: Orders, payments, inventory (ACID critical)
- Redis: Shopping cart, sessions (temporary, fast)
- Elasticsearch: Product search (full-text search)

#### Example 4: IoT Sensor Platform
**Requirements:** Millions of sensor readings per second
**Decision:** **Cassandra or InfluxDB**
**Why:**
- High write throughput needed
- Time-series data
- No complex relationships
- Eventual consistency OK

#### Example 5: Content Management System
**Requirements:** Articles, images, flexible content types
**Decision:** **MongoDB**
**Why:**
- Flexible schema (different content types)
- Nested content (comments, media)
- No complex transactions
- Easy to evolve structure

---

### Hybrid Approach (Most Common in Production)

Most modern applications use **BOTH** SQL and NoSQL!

```
┌─────────────────── Application ───────────────────┐
│                                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────┐ │
│  │ PostgreSQL   │  │    Redis     │  │ MongoDB │ │
│  │              │  │              │  │         │ │
│  │ • Users      │  │ • Sessions   │  │ • Logs  │ │
│  │ • Orders     │  │ • Cart       │  │ • Events│ │
│  │ • Payments   │  │ • Cache      │  │ • Metrics│ │
│  │              │  │ • Trending   │  │         │ │
│  │              │  │              │  │         │ │
│  │ ACID         │  │ Speed        │  │ Flexible│ │
│  │ Consistent   │  │ Temporary    │  │ Scale   │ │
│  └──────────────┘  └──────────────┘  └─────────┘ │
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │          Elasticsearch                       │ │
│  │          • Product search                    │ │
│  │          • Full-text search                  │ │
│  └──────────────────────────────────────────────┘ │
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │          Neo4j (Optional)                    │ │
│  │          • Friend recommendations            │ │
│  │          • Social graph                      │ │
│  └──────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────┘
```

**Why Hybrid?**
- Use the **right tool for the right job**
- SQL for critical data (orders, payments)
- Redis for speed (cache, sessions)
- MongoDB for flexibility (logs, events)
- Elasticsearch for search

**Example: E-Commerce Platform**
- **PostgreSQL**: Orders, users, payments (need ACID)
- **Redis**: Shopping cart, sessions, product rankings
- **MongoDB**: User activity logs, product reviews
- **Elasticsearch**: Product search, filters, autocomplete

---

## Quick Reference - SQL vs NoSQL

```
SQL (SAFE):
S - Strong consistency
A - Advanced queries (JOINs)
F - Foreign keys (relationships)
E - Enterprise (banking, compliance)

NoSQL (RASH):
R - Rapid development
A - Aggregations (horizontal scaling)
S - Simple queries
H - Huge scale (millions of users)
```

---

## Step-by-Step: E-Commerce (PostgreSQL)

### Project: Complete E-Commerce Platform with SQL

Let's build a production-ready e-commerce database from scratch using PostgreSQL.

---

### Step 1: Requirements Analysis 📋

**Questions to Ask:**
1. What data needs to be stored?
2. What relationships exist between data entities?
3. How will data be queried?
4. What's the expected scale (users, transactions)?
5. What are consistency requirements?
6. What are performance requirements?

**Business Requirements:**
- User accounts with authentication
- Product catalog with categories
- Shopping cart functionality
- Order processing with payment tracking
- Product reviews and ratings
- Inventory management
- Order history

**Technical Requirements:**
- Expected: 1M users, 100K products
- Peak: 10K orders per day
- **ACID transactions critical** (payments!)
- Complex relationships (users ↔ orders ↔ products)
- Order history must be accurate
- Inventory must stay consistent
- 99.9% uptime required

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
✅ ACID transactions (payments need atomicity)
✅ Complex relationships (foreign keys, JOINs)
✅ Strong consistency (inventory accuracy)
✅ Advanced features (JSONB for flexible attributes)
✅ Proven reliability

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
┌──────────────┐
│    USERS     │
├──────────────┤
│ id (PK)      │──────┐
│ name         │      │
│ email        │      │ 1:N
│ password_hash│      │ One user → Many orders
│ address      │      │
│ created_at   │      │
└──────────────┘      │
                      │
                      ▼
           ┌──────────────────┐
           │     ORDERS       │
           ├──────────────────┤
           │ id (PK)          │
           │ user_id (FK)     │◄────┐
           │ total            │     │
           │ status           │     │ 1:N
           │ payment_status   │     │ One order → Many items
           │ shipping_address │     │
           │ created_at       │     │
           └──────────────────┘     │
                                    │
                                    ▼
                         ┌──────────────────┐
                         │  ORDER_ITEMS     │
                         │  (Junction)      │
                         ├──────────────────┤
                         │ id (PK)          │
                         │ order_id (FK)    │
                         │ product_id (FK)  │◄────┐
                         │ quantity         │     │
                         │ price_at_order   │     │
                         └──────────────────┘     │
                                                  │
┌──────────────┐                                 │
│  CATEGORIES  │                                 │
├──────────────┤                                 │
│ id (PK)      │──────┐                         │
│ name         │      │                         │
│ description  │      │ 1:N                     │
└──────────────┘      │ One category →          │
                      │ Many products           │
                      ▼                         │
           ┌──────────────────┐                │
           │    PRODUCTS      │────────────────┘
           ├──────────────────┤
           │ id (PK)          │
           │ name             │         1:N
           │ description      │         One product →
           │ price            │         Many reviews
           │ stock            │              │
           │ category_id (FK) │              │
           │ attributes (JSONB)│             │
           │ created_at       │              │
           └──────────────────┘              │
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

### Step 3: Create Database Schema 📝

```sql
-- ============================================
-- Create Database
-- ============================================
CREATE DATABASE ecommerce_db;
\c ecommerce_db;

-- ============================================
-- Users Table
-- ============================================
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  phone VARCHAR(20),
  
  -- Address fields
  street_address TEXT,
  city VARCHAR(50),
  state VARCHAR(50),
  postal_code VARCHAR(10),
  country VARCHAR(50) DEFAULT 'Bangladesh',
  
  -- Account status
  is_active BOOLEAN DEFAULT true,
  is_verified BOOLEAN DEFAULT false,
  
  -- Timestamps
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  -- Constraints
  CONSTRAINT valid_email CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$')
);

-- Indexes for fast lookups
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_city ON users(city);
CREATE INDEX idx_users_active ON users(is_active) WHERE is_active = true;

-- ============================================
-- Categories Table
-- ============================================
CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) UNIQUE NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  description TEXT,
  parent_id INTEGER REFERENCES categories(id) ON DELETE SET NULL, -- For subcategories
  image_url VARCHAR(500),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_categories_parent ON categories(parent_id);
CREATE INDEX idx_categories_slug ON categories(slug);

-- ============================================
-- Products Table
-- ============================================
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(200) NOT NULL,
  slug VARCHAR(200) UNIQUE NOT NULL, -- URL-friendly name
  description TEXT,
  
  -- Pricing
  price DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
  cost_price DECIMAL(10, 2) CHECK (cost_price >= 0), -- For profit calculations
  discount_percent DECIMAL(5, 2) DEFAULT 0 CHECK (discount_percent >= 0 AND discount_percent <= 100),
  
  -- Inventory
  stock INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
  low_stock_threshold INTEGER DEFAULT 10,
  
  -- Category
  category_id INTEGER REFERENCES categories(id) ON DELETE SET NULL,
  
  -- Media
  image_url VARCHAR(500),
  images TEXT[],  -- Array of image URLs
  
  -- Flexible attributes (JSONB)
  attributes JSONB DEFAULT '{}',
  
  -- Status
  is_active BOOLEAN DEFAULT true,
  is_featured BOOLEAN DEFAULT false,
  
  -- Timestamps
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  -- Constraints
  CONSTRAINT valid_price CHECK (price >= COALESCE(cost_price, 0))
);

-- Indexes
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_active ON products(is_active) WHERE is_active = true;
CREATE INDEX idx_products_slug ON products(slug);
CREATE INDEX idx_products_price ON products(price);
CREATE INDEX idx_products_stock ON products(stock);
-- Full-text search index
CREATE INDEX idx_products_search ON products USING gin(to_tsvector('english', name || ' ' || COALESCE(description, '')));
-- JSONB index for attributes
CREATE INDEX idx_products_attributes ON products USING gin(attributes);

-- ============================================
-- Orders Table
-- ============================================
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  order_number VARCHAR(50) UNIQUE NOT NULL,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  
  -- Pricing breakdown
  subtotal DECIMAL(10, 2) NOT NULL CHECK (subtotal >= 0),
  tax DECIMAL(10, 2) NOT NULL DEFAULT 0 CHECK (tax >= 0),
  shipping_cost DECIMAL(10, 2) NOT NULL DEFAULT 0 CHECK (shipping_cost >= 0),
  discount DECIMAL(10, 2) NOT NULL DEFAULT 0 CHECK (discount >= 0),
  total DECIMAL(10, 2) NOT NULL CHECK (total >= 0),
  
  -- Status tracking
  status VARCHAR(20) NOT NULL DEFAULT 'pending',
  payment_status VARCHAR(20) NOT NULL DEFAULT 'pending',
  payment_method VARCHAR(50),
  
  -- Shipping information
  shipping_name VARCHAR(100) NOT NULL,
  shipping_email VARCHAR(255),
  shipping_phone VARCHAR(20),
  shipping_address TEXT NOT NULL,
  shipping_city VARCHAR(50) NOT NULL,
  shipping_state VARCHAR(50),
  shipping_postal_code VARCHAR(10),
  shipping_country VARCHAR(50) NOT NULL,
  
  -- Tracking
  tracking_number VARCHAR(100),
  carrier VARCHAR(50),
  
  -- Notes
  customer_notes TEXT,
  admin_notes TEXT,
  
  -- Timestamps
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  paid_at TIMESTAMP,
  shipped_at TIMESTAMP,
  delivered_at TIMESTAMP,
  
  -- Constraints
  CONSTRAINT valid_order_status CHECK (status IN (
    'pending', 'processing', 'confirmed', 
    'shipped', 'delivered', 'cancelled', 'refunded'
  )),
  CONSTRAINT valid_payment_status CHECK (payment_status IN (
    'pending', 'completed', 'failed', 'refunded'
  )),
  CONSTRAINT valid_total CHECK (total = subtotal + tax + shipping_cost - discount)
);

-- Indexes
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_payment_status ON orders(payment_status);
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
CREATE INDEX idx_orders_number ON orders(order_number);

-- ============================================
-- Order Items (Junction Table)
-- ============================================
CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id INTEGER NOT NULL REFERENCES products(id) ON DELETE RESTRICT,
  
  -- Snapshot product data at time of purchase
  product_name VARCHAR(200) NOT NULL,
  product_sku VARCHAR(100),
  product_price DECIMAL(10, 2) NOT NULL CHECK (product_price >= 0),
  
  -- Order specifics
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  subtotal DECIMAL(10, 2) NOT NULL CHECK (subtotal >= 0),
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  -- Constraints
  UNIQUE(order_id, product_id),
  CONSTRAINT valid_subtotal CHECK (subtotal = product_price * quantity)
);

CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_order_items_product ON order_items(product_id);

-- ============================================
-- Reviews Table
-- ============================================
CREATE TABLE reviews (
  id SERIAL PRIMARY KEY,
  product_id INTEGER NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  order_id INTEGER REFERENCES orders(id) ON DELETE SET NULL,
  
  -- Review content
  rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
  title VARCHAR(200),
  comment TEXT,
  
  -- Media
  images TEXT[],
  
  -- Verification
  is_verified_purchase BOOLEAN DEFAULT false,
  is_approved BOOLEAN DEFAULT false,
  
  -- Helpful votes
  helpful_count INTEGER DEFAULT 0,
  
  -- Timestamps
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
-- Shopping Cart Table
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

### Step 4: Sample Data & Common Queries 🔍

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

#### More queries for apis

```sql
-- ============================================
-- CREATE: Insert new records
-- ============================================

-- Users: Batch insert (multiple rows)
INSERT INTO users (name, email, password_hash, city, country) VALUES
('Foyez Ahmed', 'foyez@example.com', '$2b$10$...', 'Cumilla', 'Bangladesh'),
('Alice Rahman', 'alice@example.com', '$2b$10$...', 'Dhaka', 'Bangladesh'),
('Bob Khan', 'bob@example.com', '$2b$10$...', 'Chittagong', 'Bangladesh');

-- Categories
INSERT INTO categories (name, slug, description) VALUES
('Electronics', 'electronics', 'Electronic devices and accessories'),
('Computers', 'computers', 'Laptops, desktops, and accessories'),
('Phones', 'phones', 'Mobile phones and accessories'),
('Clothing', 'clothing', 'Men and women clothing');

-- Products
INSERT INTO products (name, slug, description, price, cost_price, stock, category_id, attributes) VALUES
(
  'Gaming Laptop', 
  'gaming-laptop-rtx-4060',
  '15.6" FHD Display, Intel Core i7, 16GB RAM, RTX 4060',
  1299.99,
  950.00,
  25,
  2,
  '{"cpu": "Intel i7-12700H", "ram": "16GB DDR5", "storage": "512GB NVMe SSD", "gpu": "RTX 4060"}'::jsonb
),
(
  'Wireless Mouse',
  'wireless-mouse-logitech',
  'Logitech Wireless Mouse with USB Receiver',
  29.99,
  15.00,
  150,
  2,
  '{"brand": "Logitech", "connectivity": "wireless", "dpi": "1600"}'::jsonb
),
(
  'Mechanical Keyboard',
  'mechanical-keyboard-rgb',
  'RGB Mechanical Gaming Keyboard',
  89.99,
  45.00,
  75,
  2,
  '{"switches": "Cherry MX Red", "backlight": "RGB", "layout": "full-size"}'::jsonb
);

-- ============================================
-- Common Queries
-- ============================================

-- User Registration
INSERT INTO users (name, email, password_hash, city)
VALUES ('New User', 'newuser@example.com', '$2b$10$...', 'Dhaka')
RETURNING id, name, email, created_at;

-- ============================================
-- READ: Retrieve records
-- ============================================

-- User Login
SELECT id, name, email, password_hash, is_active
FROM users
WHERE email = 'foyez@example.com';

-- Get all active products with category
SELECT 
  p.id,
  p.name,
  p.slug,
  p.price,
  p.stock,
  p.image_url,
  c.name as category_name,
  c.slug as category_slug
FROM products p
LEFT JOIN categories c ON p.category_id = c.id
WHERE p.is_active = true
ORDER BY p.created_at DESC
LIMIT 20;

-- Search products by name
SELECT id, name, slug, price, stock
FROM products
WHERE to_tsvector('english', name || ' ' || description) @@ to_tsquery('english', 'gaming & laptop')
  AND is_active = true
ORDER BY ts_rank(to_tsvector('english', name), to_tsquery('english', 'gaming & laptop')) DESC;

-- Get product details with average rating
SELECT 
  p.*,
  c.name as category_name,
  COALESCE(AVG(r.rating), 0) as avg_rating,
  COUNT(r.id) as review_count
FROM products p
LEFT JOIN categories c ON p.category_id = c.id
LEFT JOIN reviews r ON p.id = r.product_id AND r.is_approved = true
WHERE p.slug = 'gaming-laptop-rtx-4060'
GROUP BY p.id, c.name;

-- Add item to cart
INSERT INTO cart_items (user_id, product_id, quantity)
VALUES (1, 1, 1)
ON CONFLICT (user_id, product_id) 
DO UPDATE SET 
  quantity = cart_items.quantity + 1,
  updated_at = CURRENT_TIMESTAMP;

-- Get user's cart with product details
SELECT 
  ci.id as cart_item_id,
  p.id as product_id,
  p.name,
  p.slug,
  p.price,
  p.image_url,
  p.stock,
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

-- Calculate cart totals
SELECT 
  COUNT(*) as item_count,
  SUM(ci.quantity) as total_quantity,
  SUM(p.price * ci.quantity) as subtotal,
  SUM(p.price * ci.quantity) * 0.08 as tax,
  10.00 as shipping,
  SUM(p.price * ci.quantity) * 1.08 + 10 as total
FROM cart_items ci
JOIN products p ON ci.product_id = p.id
WHERE ci.user_id = 1
  AND p.is_active = true;

-- Create order (Transaction - ACID in action!)
BEGIN;

-- Generate order number
SELECT 'ORD-' || TO_CHAR(CURRENT_TIMESTAMP, 'YYYYMMDD') || '-' || LPAD(nextval('orders_id_seq')::text, 6, '0') INTO @order_number;

-- Create order
INSERT INTO orders (
  order_number, user_id, subtotal, tax, shipping_cost, total,
  status, payment_status, payment_method,
  shipping_name, shipping_address, shipping_city, shipping_country
)
SELECT 
  @order_number,
  1,
  SUM(p.price * ci.quantity) as subtotal,
  SUM(p.price * ci.quantity) * 0.08 as tax,
  10.00 as shipping,
  SUM(p.price * ci.quantity) * 1.08 + 10 as total,
  'pending',
  'pending',
  'credit_card',
  u.name,
  u.street_address,
  u.city,
  u.country
FROM cart_items ci
JOIN products p ON ci.product_id = p.id
JOIN users u ON ci.user_id = u.id
WHERE ci.user_id = 1
GROUP BY u.name, u.street_address, u.city, u.country
RETURNING id INTO @order_id;

-- Copy cart items to order items
INSERT INTO order_items (order_id, product_id, product_name, product_price, quantity, subtotal)
SELECT 
  @order_id,
  p.id,
  p.name,
  p.price,
  ci.quantity,
  p.price * ci.quantity
FROM cart_items ci
JOIN products p ON ci.product_id = p.id
WHERE ci.user_id = 1;

-- Update product stock
UPDATE products p
SET 
  stock = stock - ci.quantity,
  updated_at = CURRENT_TIMESTAMP
FROM cart_items ci
WHERE p.id = ci.product_id
  AND ci.user_id = 1
  AND p.stock >= ci.quantity;

-- Clear user's cart
DELETE FROM cart_items WHERE user_id = 1;

COMMIT;

-- Get user's order history
SELECT 
  o.id,
  o.order_number,
  o.total,
  o.status,
  o.payment_status,
  o.created_at,
  COUNT(oi.id) as item_count
FROM orders o
LEFT JOIN order_items oi ON o.id = oi.order_id
WHERE o.user_id = 1
GROUP BY o.id
ORDER BY o.created_at DESC;

-- Get order details with items
SELECT 
  o.*,
  json_agg(
    json_build_object(
      'product_name', oi.product_name,
      'quantity', oi.quantity,
      'price', oi.product_price,
      'subtotal', oi.subtotal
    )
  ) as items
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
WHERE o.id = 1
GROUP BY o.id;

-- Top selling products (last 30 days)
SELECT 
  p.id,
  p.name,
  p.slug,
  p.price,
  SUM(oi.quantity) as total_sold,
  SUM(oi.subtotal) as total_revenue,
  COUNT(DISTINCT o.id) as order_count
FROM products p
JOIN order_items oi ON p.id = oi.product_id
JOIN orders o ON oi.order_id = o.id
WHERE o.status IN ('delivered', 'shipped')
  AND o.created_at >= NOW() - INTERVAL '30 days'
GROUP BY p.id
ORDER BY total_sold DESC
LIMIT 10;

-- Products with reviews and ratings
SELECT 
  p.id,
  p.name,
  p.price,
  COUNT(r.id) as review_count,
  AVG(r.rating) as avg_rating,
  COUNT(CASE WHEN r.rating = 5 THEN 1 END) as five_star,
  COUNT(CASE WHEN r.rating = 4 THEN 1 END) as four_star,
  COUNT(CASE WHEN r.rating = 3 THEN 1 END) as three_star,
  COUNT(CASE WHEN r.rating = 2 THEN 1 END) as two_star,
  COUNT(CASE WHEN r.rating = 1 THEN 1 END) as one_star
FROM products p
LEFT JOIN reviews r ON p.id = r.product_id AND r.is_approved = true
WHERE p.id = 1
GROUP BY p.id;

-- Low stock alert
SELECT 
  p.id,
  p.name,
  p.stock,
  p.low_stock_threshold,
  c.name as category
FROM products p
LEFT JOIN categories c ON p.category_id = c.id
WHERE p.is_active = true
  AND p.stock <= p.low_stock_threshold
ORDER BY p.stock ASC;

-- Revenue report by date
SELECT 
  DATE(created_at) as date,
  COUNT(*) as order_count,
  SUM(total) as total_revenue,
  AVG(total) as avg_order_value
FROM orders
WHERE payment_status = 'completed'
  AND created_at >= NOW() - INTERVAL '30 days'
GROUP BY DATE(created_at)
ORDER BY date DESC;
```

---

## Step-by-Step: Blog Platform (MongoDB)

### Project: Complete Blog Platform with MongoDB

Let's build a flexible blog platform using MongoDB's document-oriented approach.

---

### Step 1: Requirements Analysis 📋

**Business Requirements:**
- Users can write and publish blog posts
- Support for drafts and published posts
- Comments on posts (with threading/replies)
- User profiles with followers
- Posts can have multiple categories and tags
- Track post views and engagement

**Technical Requirements:**
- Expected: 100K users, 1M posts
- Need fast reads (blog posts and comments)
- **Flexible schema** (different post types: text, video, gallery)
- Real-time comments
- Search posts by tags/categories
- No complex financial transactions

**Why MongoDB?**
✅ Flexible schema (different content types)
✅ Fast reads (denormalized embedded data)
✅ Nested comments naturally
✅ Easy to evolve structure
✅ Horizontal scaling built-in

---

### Step 2: Design Document Model 🎨

**Key Design Decisions:**

1. **Users Collection** - Basic user info
2. **Posts Collection** - Embed author info for fast reads
3. **Comments Collection** - Separate (can grow large)
4. **Followers Collection** - Many-to-many relationship
5. **Categories Collection** - Reference in posts

---

### Step 3: Create Collections & Documents 📝

```javascript
// ============================================
// Connect to MongoDB
// ============================================
const { MongoClient, ObjectId } = require('mongodb');

const client = new MongoClient('mongodb://localhost:27017');
await client.connect();
const db = client.db('blog_platform');

// ============================================
// Collection 1: Users
// ============================================
const usersSchema = {
  _id: ObjectId("user_id"),
  username: "foyez_ahmed",  // Unique
  email: "foyez@example.com",  // Unique
  passwordHash: "$2b$10$...",
  
  // Profile (embedded for fast access)
  profile: {
    firstName: "Foyez",
    lastName: "Ahmed",
    bio: "Full-stack developer from Bangladesh",
    avatar: "https://cdn.example.com/avatars/foyez.jpg",
    location: {
      city: "Cumilla",
      country: "Bangladesh"
    },
    socialLinks: {
      twitter: "https://twitter.com/foyez",
      github: "https://github.com/foyez",
      website: "https://foyez.dev"
    }
  },
  
  // Statistics (updated frequently)
  stats: {
    postsCount: 42,
    followersCount: 1250,
    followingCount: 340,
    totalViews: 50000
  },
  
  // Settings (embedded)
  settings: {
    emailNotifications: true,
    publicProfile: true,
    theme: "dark",
    language: "en"
  },
  
  // Roles
  roles: ["author", "moderator"],
  
  // Timestamps
  createdAt: new Date("2023-01-15T10:30:00Z"),
  updatedAt: new Date("2024-12-29T14:20:00Z"),
  lastLoginAt: new Date("2024-12-29T09:15:00Z")
};

// Create collection with validation
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["username", "email", "passwordHash"],
      properties: {
        username: {
          bsonType: "string",
          minLength: 3,
          maxLength: 30,
          pattern: "^[a-zA-Z0-9_]+$"
        },
        email: {
          bsonType: "string",
          pattern: "^.+@.+\..+$"
        },
        passwordHash: { bsonType: "string" }
      }
    }
  }
});

// Create indexes
db.users.createIndex({ email: 1 }, { unique: true });
db.users.createIndex({ username: 1 }, { unique: true });
db.users.createIndex({ "profile.location.city": 1 });

// ============================================
// Collection 2: Posts
// ============================================
const postSchema = {
  _id: ObjectId("post_id"),
  
  // Author (embedded for fast reads - denormalized)
  author: {
    id: ObjectId("user_id"),
    username: "foyez_ahmed",
    avatar: "https://cdn.example.com/avatars/foyez.jpg"
  },
  
  // Content
  title: "Complete MongoDB Guide",
  slug: "complete-mongodb-guide",  // URL-friendly, unique
  excerpt: "Learn MongoDB from basics to advanced concepts...",
  content: "Full markdown content here...",
  coverImage: "https://cdn.example.com/posts/mongodb-guide.jpg",
  
  // Content type (flexible!)
  type: "article",  // article, video, gallery, podcast
  
  // Type-specific data (only for video posts)
  videoData: {
    url: "https://youtube.com/watch?v=...",
    duration: 1800,  // seconds
    thumbnail: "https://..."
  },
  
  // Metadata
  status: "published",  // draft, published, archived, deleted
  visibility: "public",  // public, private, unlisted
  
  // Categories and tags
  categories: ["Database", "NoSQL"],
  tags: ["mongodb", "nosql", "database", "tutorial", "guide"],
  
  // Statistics (updated frequently)
  stats: {
    views: 5420,
    likes: 342,
    commentsCount: 28,
    sharesCount: 15,
    readingTime: 8,  // minutes
    bookmarksCount: 67
  },
  
  // SEO
  seo: {
    metaTitle: "Complete MongoDB Guide - Learn NoSQL Database",
    metaDescription: "Comprehensive guide to MongoDB covering basics to advanced concepts...",
    keywords: ["mongodb", "nosql", "database", "tutorial"],
    ogImage: "https://cdn.example.com/og/mongodb-guide.jpg"
  },
  
  // Recent comments (cached for quick display)
  recentComments: [
    {
      id: ObjectId("comment_id_1"),
      author: {
        username: "alice",
        avatar: "https://..."
      },
      text: "Great post! Very helpful.",
      createdAt: new Date("2024-12-28T16:45:00Z")
    }
    // Keep only 5 most recent
  ],
  
  // Timestamps
  publishedAt: new Date("2024-01-20T12:00:00Z"),
  createdAt: new Date("2024-01-15T10:30:00Z"),
  updatedAt: new Date("2024-12-29T14:20:00Z")
};

// Create indexes
db.posts.createIndex({ "author.id": 1, status: 1 });
db.posts.createIndex({ slug: 1 }, { unique: true });
db.posts.createIndex({ tags: 1 });
db.posts.createIndex({ categories: 1 });
db.posts.createIndex({ publishedAt: -1 });
db.posts.createIndex({ status: 1, publishedAt: -1 });
// Full-text search
db.posts.createIndex({
  title: "text",
  content: "text",
  tags: "text"
});

// ============================================
// Collection 3: Comments
// ============================================
const commentSchema = {
  _id: ObjectId("comment_id"),
  
  // Post reference
  postId: ObjectId("post_id"),
  postTitle: "Complete MongoDB Guide",  // Denormalized for display
  
  // Author (embedded)
  author: {
    id: ObjectId("user_id"),
    username: "alice",
    avatar: "https://cdn.example.com/avatars/alice.jpg"
  },
  
  // Content
  text: "This is a great tutorial! Thanks for sharing.",
  
  // Threading (for replies)
  parentId: null,  // null for top-level, ObjectId for replies
  depth: 0,        // 0 for top-level, 1 for direct replies, 2 for nested
  path: "/comment_id",  // For efficient querying of entire thread
  
  // Moderation
  status: "approved",  // pending, approved, rejected, spam
  
  // Engagement
  likesCount: 15,
  repliesCount: 3,
  
  // Timestamps
  createdAt: new Date("2024-12-28T16:45:00Z"),
  updatedAt: new Date("2024-12-28T16:45:00Z"),
  editedAt: null
};

// Create indexes
db.comments.createIndex({ postId: 1, createdAt: -1 });
db.comments.createIndex({ "author.id": 1 });
db.comments.createIndex({ parentId: 1 });
db.comments.createIndex({ path: 1 });
db.comments.createIndex({ status: 1 });

// ============================================
// Collection 4: Followers (Many-to-Many)
// ============================================
const followerSchema = {
  _id: ObjectId("follow_id"),
  followerId: ObjectId("user_id_1"),   // Alice follows
  followingId: ObjectId("user_id_2"),  // Bob
  
  // Denormalized for display (optional)
  follower: {
    username: "alice",
    avatar: "https://..."
  },
  following: {
    username: "bob",
    avatar: "https://..."
  },
  
  createdAt: new Date("2024-06-15T08:20:00Z")
};

// Create compound indexes (both directions)
db.followers.createIndex({ followerId: 1, followingId: 1 }, { unique: true });
db.followers.createIndex({ followingId: 1, followerId: 1 });

// ============================================
// Collection 5: Categories
// ============================================
const categorySchema = {
  _id: ObjectId("category_id"),
  name: "Database",
  slug: "database",
  description: "Everything about databases - SQL, NoSQL, and more",
  icon: "https://cdn.example.com/icons/database.svg",
  color: "#3B82F6",
  
  // Statistics
  postsCount: 156,
  followersCount: 1200,
  
  // Hierarchy (for subcategories)
  parentId: null,  // null for top-level, ObjectId for subcategory
  
  // SEO
  metaDescription: "Learn about databases...",
  
  createdAt: new Date("2023-01-01T00:00:00Z")
};

db.categories.createIndex({ slug: 1 }, { unique: true });
db.categories.createIndex({ parentId: 1 });
```

---

### Step 4: Common Operations 🔍

```javascript
// ============================================
// USER OPERATIONS
// ============================================

// 1. Create user (registration)
const newUser = await db.users.insertOne({
  username: "foyez_ahmed",
  email: "foyez@example.com",
  passwordHash: "$2b$10$...",
  profile: {
    firstName: "Foyez",
    lastName: "Ahmed",
    bio: "",
    avatar: "",
    location: { city: "", country: "Bangladesh" }
  },
  stats: {
    postsCount: 0,
    followersCount: 0,
    followingCount: 0,
    totalViews: 0
  },
  settings: {
    emailNotifications: true,
    publicProfile: true,
    theme: "light"
  },
  roles: ["author"],
  createdAt: new Date(),
  updatedAt: new Date()
});

// 2. Find user by email (login)
const user = await db.users.findOne({ email: "foyez@example.com" });

// 3. Update user profile
await db.users.updateOne(
  { _id: ObjectId("user_id") },
  {
    $set: {
      "profile.bio": "Full-stack developer specializing in Node.js and React",
      "profile.location": { city: "Cumilla", country: "Bangladesh" },
      updatedAt: new Date()
    }
  }
);

// 4. Increment user stats (atomic operation)
await db.users.updateOne(
  { _id: ObjectId("user_id") },
  {
    $inc: { "stats.postsCount": 1, "stats.totalViews": 100 }
  }
);

// 5. Get user profile with recent posts
const userProfile = await db.users.aggregate([
  { $match: { username: "foyez_ahmed" } },
  {
    $lookup: {
      from: "posts",
      let: { userId: "$_id" },
      pipeline: [
        {
          $match: {
            $expr: { $eq: ["$author.id", "$$userId"] },
            status: "published"
          }
        },
        { $sort: { publishedAt: -1 } },
        { $limit: 5 },
        { $project: { title: 1, slug: 1, excerpt: 1, publishedAt: 1, "stats.views": 1 } }
      ],
      as: "recentPosts"
    }
  }
]).toArray();

// ============================================
// POST OPERATIONS
// ============================================

// 1. Create new post
const newPost = await db.posts.insertOne({
  author: {
    id: ObjectId("user_id"),
    username: "foyez_ahmed",
    avatar: "https://cdn.example.com/avatars/foyez.jpg"
  },
  title: "Complete MongoDB Guide",
  slug: "complete-mongodb-guide",
  excerpt: "Learn MongoDB from basics to advanced...",
  content: "Full post content in markdown...",
  coverImage: "https://cdn.example.com/posts/cover.jpg",
  type: "article",
  status: "published",
  visibility: "public",
  categories: ["Database", "NoSQL"],
  tags: ["mongodb", "nosql", "database", "tutorial"],
  stats: {
    views: 0,
    likes: 0,
    commentsCount: 0,
    sharesCount: 0,
    readingTime: 8
  },
  seo: {
    metaTitle: "Complete MongoDB Guide",
    metaDescription: "Learn MongoDB...",
    keywords: ["mongodb", "nosql"]
  },
  recentComments: [],
  publishedAt: new Date(),
  createdAt: new Date(),
  updatedAt: new Date()
});

// 2. Get post by slug (with author info already embedded!)
const post = await db.posts.findOne({ slug: "complete-mongodb-guide" });
// No JOIN needed! Author info already there.

// 3. Get user's published posts
const userPosts = await db.posts.find({
  "author.id": ObjectId("user_id"),
  status: "published"
}).sort({ publishedAt: -1 }).limit(10).toArray();

// 4. Update post
await db.posts.updateOne(
  { _id: ObjectId("post_id") },
  {
    $set: {
      title: "Updated Title",
      content: "Updated content...",
      updatedAt: new Date()
    }
  }
);

// 5. Increment view count (atomic)
await db.posts.updateOne(
  { _id: ObjectId("post_id") },
  { $inc: { "stats.views": 1 } }
);

// 6. Add tag to post
await db.posts.updateOne(
  { _id: ObjectId("post_id") },
  { $addToSet: { tags: "beginner-friendly" } }  // Only adds if not exists
);

// 7. Remove tag from post
await db.posts.updateOne(
  { _id: ObjectId("post_id") },
  { $pull: { tags: "old-tag" } }
);

// 8. Search posts by tags (array query)
const taggedPosts = await db.posts.find({
  tags: { $in: ["mongodb", "database"] },
  status: "published"
}).sort({ publishedAt: -1 }).toArray();

// 9. Full-text search
const searchResults = await db.posts.find({
  $text: { $search: "mongodb tutorial guide" },
  status: "published"
}).sort({
  score: { $meta: "textScore" }
}).toArray();

// 10. Get posts by category
const categoryPosts = await db.posts.find({
  categories: "Database",
  status: "published"
}).sort({ publishedAt: -1 }).limit(20).toArray();

// ============================================
// COMMENT OPERATIONS
// ============================================

// 1. Add comment to post
const comment = {
  postId: ObjectId("post_id"),
  postTitle: "Complete MongoDB Guide",
  author: {
    id: ObjectId("user_id"),
    username: "alice",
    avatar: "https://cdn.example.com/avatars/alice.jpg"
  },
  text: "Great post! Very helpful. Thanks for sharing!",
  parentId: null,  // Top-level comment
  depth: 0,
  path: null,  // Will be updated with _id
  status: "approved",
  likesCount: 0,
  repliesCount: 0,
  createdAt: new Date(),
  updatedAt: new Date()
};

const insertedComment = await db.comments.insertOne(comment);

// Update path with actual _id
await db.comments.updateOne(
  { _id: insertedComment.insertedId },
  { $set: { path: `/${insertedComment.insertedId}` } }
);

// Update post's comment count and add to recent comments
await db.posts.updateOne(
  { _id: ObjectId("post_id") },
  {
    $inc: { "stats.commentsCount": 1 },
    $push: {
      recentComments: {
        $each: [{
          id: insertedComment.insertedId,
          author: comment.author,
          text: comment.text,
          createdAt: comment.createdAt
        }],
        $slice: -5  // Keep only 5 most recent
      }
    }
  }
);

// 2. Reply to comment
const reply = {
  postId: ObjectId("post_id"),
  postTitle: "Complete MongoDB Guide",
  author: {
    id: ObjectId("author_user_id"),
    username: "foyez_ahmed",
    avatar: "https://..."
  },
  text: "Thanks! Glad you found it helpful.",
  parentId: ObjectId("parent_comment_id"),
  depth: 1,
  path: null,  // Will be updated
  status: "approved",
  likesCount: 0,
  repliesCount: 0,
  createdAt: new Date(),
  updatedAt: new Date()
};

const insertedReply = await db.comments.insertOne(reply);

// Get parent's path and append
const parentComment = await db.comments.findOne({ _id: reply.parentId });
const newPath = `${parentComment.path}/${insertedReply.insertedId}`;

await db.comments.updateOne(
  { _id: insertedReply.insertedId },
  { $set: { path: newPath } }
);

// Increment parent's reply count
await db.comments.updateOne(
  { _id: reply.parentId },
  { $inc: { repliesCount: 1 } }
);

// 3. Get comments for post (paginated)
const comments = await db.comments.find({
  postId: ObjectId("post_id"),
  parentId: null,  // Top-level only
  status: "approved"
}).sort({ createdAt: -1 }).limit(20).skip(0).toArray();

// 4. Get replies for a comment
const replies = await db.comments.find({
  parentId: ObjectId("parent_comment_id"),
  status: "approved"
}).sort({ createdAt: 1 }).toArray();

// 5. Get entire comment thread
const thread = await db.comments.find({
  path: new RegExp(`^/${parentCommentId}`),  // All descendants
  status: "approved"
}).sort({ path: 1 }).toArray();

// ============================================
// FOLLOW OPERATIONS
// ============================================

// 1. Follow a user
await db.followers.insertOne({
  followerId: ObjectId("alice_id"),
  followingId: ObjectId("bob_id"),
  follower: {
    username: "alice",
    avatar: "https://..."
  },
  following: {
    username: "bob",
    avatar: "https://..."
  },
  createdAt: new Date()
});

// Update both users' stats
await db.users.updateOne(
  { _id: ObjectId("alice_id") },
  { $inc: { "stats.followingCount": 1 } }
);

await db.users.updateOne(
  { _id: ObjectId("bob_id") },
  { $inc: { "stats.followersCount": 1 } }
);

// 2. Unfollow a user
await db.followers.deleteOne({
  followerId: ObjectId("alice_id"),
  followingId: ObjectId("bob_id")
});

// Update stats (decrement)
await db.users.updateOne(
  { _id: ObjectId("alice_id") },
  { $inc: { "stats.followingCount": -1 } }
);

await db.users.updateOne(
  { _id: ObjectId("bob_id") },
  { $inc: { "stats.followersCount": -1 } }
);

// 3. Get user's followers
const followers = await db.followers.find({
  followingId: ObjectId("bob_id")
}).sort({ createdAt: -1 }).toArray();

// 4. Get users that someone follows
const following = await db.followers.find({
  followerId: ObjectId("alice_id")
}).sort({ createdAt: -1 }).toArray();

// 5. Check if Alice follows Bob
const isFollowing = await db.followers.findOne({
  followerId: ObjectId("alice_id"),
  followingId: ObjectId("bob_id")
});
// Returns document if following, null if not

// ============================================
// AGGREGATION PIPELINE EXAMPLES
// ============================================

// 1. Top 10 most viewed posts
const topPosts = await db.posts.aggregate([
  { $match: { status: "published" } },
  { $sort: { "stats.views": -1 } },
  { $limit: 10 },
  {
    $project: {
      title: 1,
      slug: 1,
      "author.username": 1,
      "stats.views": 1,
      "stats.likes": 1,
      publishedAt: 1
    }
  }
]).toArray();

// 2. Posts grouped by category with counts
const postsByCategory = await db.posts.aggregate([
  { $match: { status: "published" } },
  { $unwind: "$categories" },  // Separate by category
  {
    $group: {
      _id: "$categories",
      count: { $sum: 1 },
      totalViews: { $sum: "$stats.views" },
      avgViews: { $avg: "$stats.views" }
    }
  },
  { $sort: { count: -1 } }
]).toArray();

// 3. User feed (posts from followed users)
const feed = await db.followers.aggregate([
  // Get users that current user follows
  { $match: { followerId: ObjectId("current_user_id") } },
  
  // Lookup their posts
  {
    $lookup: {
      from: "posts",
      localField: "followingId",
      foreignField: "author.id",
      as: "posts"
    }
  },
  
  // Unwind posts array
  { $unwind: "$posts" },
  
  // Only published posts
  { $match: { "posts.status": "published" } },
  
  // Sort by publish date
  { $sort: { "posts.publishedAt": -1 } },
  
  // Limit to 20 posts
  { $limit: 20 },
  
  // Format output
  { $replaceRoot: { newRoot: "$posts" } }
]).toArray();

// 4. Trending posts (last 7 days, most views)
const trending = await db.posts.aggregate([
  {
    $match: {
      status: "published",
      publishedAt: {
        $gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)
      }
    }
  },
  { $sort: { "stats.views": -1 } },
  { $limit: 10 }
]).toArray();

// 5. Author leaderboard (by total views)
const leaderboard = await db.posts.aggregate([
  { $match: { status: "published" } },
  {
    $group: {
      _id: "$author.id",
      username: { $first: "$author.username" },
      avatar: { $first: "$author.avatar" },
      postsCount: { $sum: 1 },
      totalViews: { $sum: "$stats.views" },
      totalLikes: { $sum: "$stats.likes" },
      avgViews: { $avg: "$stats.views" }
    }
  },
  { $sort: { totalViews: -1 } },
  { $limit: 10 }
]).toArray();
```

---

## Step-by-Step: Real-Time App (Redis)

### Project: Real-Time Gaming Leaderboard & Chat

Let's build a real-time gaming application using Redis for speed.

---

### Step 1: Requirements Analysis 📋

**Business Requirements:**
- Real-time leaderboard updates
- Session management for logged-in users
- Rate limiting for API requests
- Real-time chat/messaging
- Cache game state and user profiles
- Track active users online

**Technical Requirements:**
- **< 1ms latency** for leaderboard
- Handle 10K+ concurrent users
- Session expiration (30 min inactivity)
- Rate limit: 100 req/min per user
- Real-time pub/sub for chat
- Cache frequently accessed data

**Why Redis?**
✅ Extremely fast (in-memory, < 1ms)
✅ Perfect for temporary data (sessions, cache)
✅ Sorted sets for leaderboards
✅ Pub/Sub for real-time messaging
✅ TTL for auto-expiration

---

### Step 2: Redis Data Structures 🎨

```bash
# 1. SESSION STORAGE (String with TTL)
Key: session:{sessionId}
Value: {"userId": 1, "username": "foyez", "role": "player"}
TTL: 1800 seconds (30 minutes)

# 2. USER CACHE (Hash)
Key: user:{userId}
Fields: {
  "username": "foyez",
  "level": "42",
  "coins": "10000",
  "avatar": "https://..."
}

# 3. LEADERBOARD (Sorted Set)
Key: leaderboard:global
Members: {
  "user:1" => 10000,  # score
  "user:2" => 9500,
  "user:3" => 9200
}

# 4. RATE LIMITING (String with TTL)
Key: rate_limit:{userId}:{minute}
Value: request_count
TTL: 60 seconds

# 5. ONLINE USERS (Set)
Key: online:users
Members: ["user:1", "user:2", "user:3"]

# 6. CHAT MESSAGES (List)
Key: chat:room:{roomId}
Values: [
  '{"user":"foyez","msg":"Hello!","time":1234567890}',
  '{"user":"alice","msg":"Hi there!","time":1234567891}'
]

# 7. USER INVENTORY (Set)
Key: inventory:{userId}
Members: ["item:sword", "item:shield", "item:potion"]
```

---

### Step 3: Implementation Examples 🔍

```javascript
const Redis = require('ioredis');
const redis = new Redis();

// ============================================
// SESSION MANAGEMENT
// ============================================

// 1. Create session (login)
async function createSession(userId, username, role) {
  const sessionId = generateSessionId();  // UUID
  
  const sessionData = JSON.stringify({
    userId,
    username,
    role,
    createdAt: Date.now()
  });
  
  // Store with 30 min expiration
  await redis.setex(`session:${sessionId}`, 1800, sessionData);
  
  return sessionId;
}

// 2. Get session
async function getSession(sessionId) {
  const sessionData = await redis.get(`session:${sessionId}`);
  
  if (!sessionData) return null;
  
  // Refresh expiration on access
  await redis.expire(`session:${sessionId}`, 1800);
  
  return JSON.parse(sessionData);
}

// 3. Delete session (logout)
async function deleteSession(sessionId) {
  await redis.del(`session:${sessionId}`);
}

// ============================================
// USER CACHING
// ============================================

// 1. Cache user data
async function cacheUser(userId, userData) {
  await redis.hset(`user:${userId}`, {
    username: userData.username,
    level: userData.level.toString(),
    coins: userData.coins.toString(),
    avatar: userData.avatar
  });
  
  // Set expiration (1 hour)
  await redis.expire(`user:${userId}`, 3600);
}

// 2. Get cached user
async function getCachedUser(userId) {
  const userData = await redis.hgetall(`user:${userId}`);
  
  if (!Object.keys(userData).length) return null;
  
  return {
    username: userData.username,
    level: parseInt(userData.level),
    coins: parseInt(userData.coins),
    avatar: userData.avatar
  };
}

// 3. Update user coins (increment)
async function addCoins(userId, amount) {
  await redis.hincrby(`user:${userId}`, 'coins', amount);
}

// ============================================
// LEADERBOARD
// ============================================

// 1. Add/Update player score
async function updateScore(userId, score) {
  await redis.zadd('leaderboard:global', score, `user:${userId}`);
}

// 2. Get top 10 players
async function getTopPlayers(limit = 10) {
  const results = await redis.zrevrange(
    'leaderboard:global',
    0,
    limit - 1,
    'WITHSCORES'
  );
  
  // Parse results: [user:1, score1, user:2, score2, ...]
  const leaderboard = [];
  for (let i = 0; i < results.length; i += 2) {
    const userId = results[i].split(':')[1];
    const score = parseInt(results[i + 1]);
    leaderboard.push({ userId, score, rank: (i / 2) + 1 });
  }
  
  return leaderboard;
}

// 3. Get player rank
async function getPlayerRank(userId) {
  const rank = await redis.zrevrank('leaderboard:global', `user:${userId}`);
  
  if (rank === null) return null;
  
  const score = await redis.zscore('leaderboard:global', `user:${userId}`);
  
  return {
    rank: rank + 1,  // 0-indexed, so add 1
    score: parseInt(score)
  };
}

// 4. Get players near user's rank
async function getNearbyPlayers(userId, range = 5) {
  const rank = await redis.zrevrank('leaderboard:global', `user:${userId}`);
  
  if (rank === null) return [];
  
  const start = Math.max(0, rank - range);
  const end = rank + range;
  
  return await redis.zrevrange(
    'leaderboard:global',
    start,
    end,
    'WITHSCORES'
  );
}

// 5. Increment player score
async function incrementScore(userId, points) {
  const newScore = await redis.zincrby('leaderboard:global', points, `user:${userId}`);
  return parseInt(newScore);
}

// ============================================
// RATE LIMITING
// ============================================

// 1. Check and increment rate limit
async function checkRateLimit(userId, maxRequests = 100) {
  const currentMinute = Math.floor(Date.now() / 60000);
  const key = `rate_limit:${userId}:${currentMinute}`;
  
  // Increment counter
  const requests = await redis.incr(key);
  
  // Set expiration on first request
  if (requests === 1) {
    await redis.expire(key, 60);
  }
  
  // Check if limit exceeded
  if (requests > maxRequests) {
    const ttl = await redis.ttl(key);
    throw new Error(`Rate limit exceeded. Try again in ${ttl} seconds.`);
  }
  
  return {
    requests,
    limit: maxRequests,
    remaining: maxRequests - requests,
    resetIn: await redis.ttl(key)
  };
}

// ============================================
// ONLINE USERS TRACKING
// ============================================

// 1. Mark user as online
async function setUserOnline(userId) {
  await redis.sadd('online:users', `user:${userId}`);
  
  // Set expiration key to auto-remove if not refreshed
  await redis.setex(`online:${userId}`, 300, '1');  // 5 min
}

// 2. Mark user as offline
async function setUserOffline(userId) {
  await redis.srem('online:users', `user:${userId}`);
  await redis.del(`online:${userId}`);
}

// 3. Get online users count
async function getOnlineCount() {
  return await redis.scard('online:users');
}

// 4. Check if user is online
async function isUserOnline(userId) {
  return await redis.sismember('online:users', `user:${userId}`);
}

// 5. Get all online users
async function getOnlineUsers() {
  return await redis.smembers('online:users');
}

// ============================================
// REAL-TIME CHAT (Pub/Sub)
// ============================================

// 1. Subscribe to chat room
const subscriber = new Redis();
subscriber.subscribe('chat:room:1', (err, count) => {
  console.log(`Subscribed to ${count} channels`);
});

subscriber.on('message', (channel, message) => {
  const msg = JSON.parse(message);
  console.log(`[${msg.username}]: ${msg.text}`);
  // Send to WebSocket clients
});

// 2. Publish message to room
async function sendMessage(roomId, userId, username, text) {
  const message = JSON.stringify({
    userId,
    username,
    text,
    timestamp: Date.now()
  });
  
  // Publish to subscribers
  await redis.publish(`chat:room:${roomId}`, message);
  
  // Store in list (last 100 messages)
  await redis.lpush(`chat:history:${roomId}`, message);
  await redis.ltrim(`chat:history:${roomId}`, 0, 99);
}

// 3. Get chat history
async function getChatHistory(roomId, limit = 50) {
  const messages = await redis.lrange(
    `chat:history:${roomId}`,
    0,
    limit - 1
  );
  
  return messages.map(msg => JSON.parse(msg)).reverse();
}

// ============================================
// GAME STATE CACHING
// ============================================

// 1. Cache game state
async function cacheGameState(gameId, state) {
  await redis.setex(
    `game:${gameId}:state`,
    3600,  // 1 hour
    JSON.stringify(state)
  );
}

// 2. Get game state
async function getGameState(gameId) {
  const state = await redis.get(`game:${gameId}:state`);
  return state ? JSON.parse(state) : null;
}

// 3. Update specific game field
async function updateGameField(gameId, field, value) {
  const state = await getGameState(gameId);
  if (!state) return null;
  
  state[field] = value;
  await cacheGameState(gameId, state);
  
  return state;
}

// ============================================
// INVENTORY MANAGEMENT
// ============================================

// 1. Add item to inventory
async function addItemToInventory(userId, itemId) {
  await redis.sadd(`inventory:${userId}`, `item:${itemId}`);
}

// 2. Remove item from inventory
async function removeItemFromInventory(userId, itemId) {
  await redis.srem(`inventory:${userId}`, `item:${itemId}`);
}

// 3. Get user inventory
async function getUserInventory(userId) {
  const items = await redis.smembers(`inventory:${userId}`);
  return items.map(item => item.split(':')[1]);
}

// 4. Check if user has item
async function hasItem(userId, itemId) {
  return await redis.sismember(`inventory:${userId}`, `item:${itemId}`);
}

// ============================================
// ATOMIC OPERATIONS (Transactions)
// ============================================

// Transfer coins between users (atomic)
async function transferCoins(fromUserId, toUserId, amount) {
  const multi = redis.multi();
  
  multi.hincrby(`user:${fromUserId}`, 'coins', -amount);
  multi.hincrby(`user:${toUserId}`, 'coins', amount);
  
  const results = await multi.exec();
  
  // Check if first operation resulted in negative balance
  const newBalance = results[0][1];
  if (newBalance < 0) {
    // Rollback
    await redis.hincrby(`user:${fromUserId}`, 'coins', amount);
    await redis.hincrby(`user:${toUserId}`, 'coins', -amount);
    throw new Error('Insufficient funds');
  }
  
  return true;
}

// ============================================
// PERFORMANCE EXAMPLE
// ============================================

// Compare: Redis vs Database
async function performanceTest() {
  console.time('Redis lookup');
  const cached = await redis.get('user:1');
  console.timeEnd('Redis lookup');
  // ~1ms
  
  console.time('Database lookup');
  const dbResult = await db.query('SELECT * FROM users WHERE id = 1');
  console.timeEnd('Database lookup');
  // ~50ms
  
  // Redis is 50x faster!
}
```

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

#### Real-Life Analogy: ATM Withdrawal
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

#### SQL Example

```sql
-- ❌ WITHOUT Transaction (DANGEROUS!)
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- ⚡ POWER FAILURE HERE!
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- Result: $100 disappeared!

-- ✅ WITH Transaction (SAFE!)
BEGIN TRANSACTION;
  
  -- Step 1: Deduct from sender
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  
  -- Step 2: Add to receiver
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
  
  -- ⚡ If power fails anywhere above, EVERYTHING rolls back!

COMMIT;
-- Only NOW is the transfer permanent
```

#### MongoDB Example

```javascript
// MongoDB Transactions (ACID compliant since v4.0)
const session = client.startSession();

try {
  await session.startTransaction();
  
  // Deduct from sender
  await db.collection('accounts').updateOne(
    { _id: ObjectId("sender_id") },
    { $inc: { balance: -100 } },
    { session }
  );
  
  // Add to receiver
  await db.collection('accounts').updateOne(
    { _id: ObjectId("receiver_id") },
    { $inc: { balance: 100 } },
    { session }
  );
  
  await session.commitTransaction();
  console.log("Transfer successful");
  
} catch (error) {
  await session.abortTransaction();
  console.error("Transfer failed, rolled back");
} finally {
  session.endSession();
}
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

#### SQL Example

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

-- ✅ This follows all rules
UPDATE accounts SET balance = 100 WHERE id = 1;
-- SUCCESS
```

#### MongoDB Example

```javascript
// Schema validation in MongoDB
db.createCollection("accounts", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["balance", "email"],
      properties: {
        balance: {
          bsonType: "double",
          minimum: 0  // Can't be negative
        },
        email: {
          bsonType: "string",
          pattern: "^.+@.+\..+$"  // Must be valid email
        },
        status: {
          enum: ["active", "closed"]  // Must be one of these
        }
      }
    }
  }
});

// ❌ Violates balance rule
await db.accounts.insertOne({ balance: -50, email: "user@example.com" });
// ERROR: Document failed validation

// ✅ Follows all rules
await db.accounts.insertOne({ balance: 100, email: "user@example.com" });
// SUCCESS
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
  END IF;

COMMIT;
-- Now User B can proceed

-- User B's transaction (must wait for A to finish)
BEGIN TRANSACTION;
  
  SELECT stock FROM products WHERE id = 101 FOR UPDATE;
  -- Returns: 0 (User A already bought it)
  
  IF stock >= 1 THEN
    -- Won't execute
  ELSE
    RAISE EXCEPTION 'Out of stock';  -- ✅ Correct!
  END IF;

COMMIT;
```

#### MongoDB Example

```javascript
// Optimistic concurrency control
async function buyProduct(productId, userId) {
  let success = false;
  let retries = 0;
  
  while (!success && retries < 3) {
    // Read current stock
    const product = await db.products.findOne({ _id: productId });
    
    if (product.stock < 1) {
      throw new Error('Out of stock');
    }
    
    // Try to update only if stock hasn't changed
    const result = await db.products.updateOne(
      {
        _id: productId,
        stock: product.stock  // Only update if stock still same
      },
      {
        $inc: { stock: -1 }
      }
    );
    
    if (result.modifiedCount === 1) {
      success = true;
    } else {
      // Someone else bought it, retry
      retries++;
      await new Promise(resolve => setTimeout(resolve, 100));
    }
  }
  
  if (!success) {
    throw new Error('Failed to purchase after retries');
  }
}
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

#### SQL Example

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

PostgreSQL Example:
- Transaction: INSERT INTO users ...
- Step 1: Write to pg_wal/000001234  ← Immediate
- Step 2: COMMIT recorded in WAL
- Step 3: Update actual data files  ← Background process
- If crash: Replay WAL entries to recover
```

**2. Replication**
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
- Essential for databases

---

### ACID Summary Table

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