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

## Indexing Strategies

### What is an Index?

**Real-Life Analogy:** A book's index at the back. Instead of reading all 500 pages to find "PostgreSQL", you look it up in the index and jump to page 147.

**Definition:** An index is a data structure that improves the speed of data retrieval operations on a database table at the cost of additional writes and storage space.

### Without vs With Index

```sql
-- PostgreSQL Example

-- ❌ WITHOUT Index: Sequential Scan (Slow)
SELECT * FROM users WHERE email = 'foyez@example.com';
-- Scans ALL 1,000,000 rows one by one
-- Time: O(n) = ~2000ms

-- ✅ WITH Index: Index Scan (Fast)
CREATE INDEX idx_users_email ON users(email);

SELECT * FROM users WHERE email = 'foyez@example.com';
-- Uses index to jump directly to the row
-- Time: O(log n) = ~50ms
-- 40x faster!
```

```javascript
// MongoDB Example

// ❌ WITHOUT Index: Collection Scan
db.users.find({ email: "foyez@example.com" });
// Scans entire collection
// Time: ~2000ms for 1M documents

// ✅ WITH Index: Index Scan
db.users.createIndex({ email: 1 });

db.users.find({ email: "foyez@example.com" });
// Uses index
// Time: ~50ms
// 40x faster!
```

**Performance Comparison:**
```
1,000,000 rows:
- Sequential scan: ~2000ms
- Index scan: ~50ms
- 40x faster!
```

---

## SQL Indexing (PostgreSQL/MySQL)

### Types of SQL Indexes

#### 1. B-Tree Index (Default, Most Common)

**Best for:** Equality, range queries, sorting

```sql
-- Create B-Tree index
CREATE INDEX idx_products_price ON products(price);

-- Queries that use this index:
SELECT * FROM products WHERE price = 99.99;          -- Exact match
SELECT * FROM products WHERE price > 50;             -- Range
SELECT * FROM products WHERE price BETWEEN 50 AND 150;
SELECT * FROM products ORDER BY price;               -- Sorting
SELECT MIN(price), MAX(price) FROM products;         -- Min/Max
```

**Structure (Balanced Tree):**
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
- Default index type
- Self-balancing

---

#### 2. Hash Index

**Best for:** Exact equality matches only

```sql
-- Create hash index (PostgreSQL)
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
- Faster than B-Tree for equality, but limited

---

#### 3. GIN (Generalized Inverted Index)

**Best for:** Full-text search, arrays, JSON

```sql
-- For array columns
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(200),
  tags VARCHAR[]  -- Array
);

CREATE INDEX idx_products_tags ON products USING GIN (tags);

-- Query: Find products with specific tags
SELECT * FROM products WHERE tags @> ARRAY['electronics', 'laptop'];
-- Very fast with GIN index!

-- For JSONB columns
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  preferences JSONB
);

CREATE INDEX idx_users_preferences ON users USING GIN (preferences);

-- Query: Find users with specific JSON property
SELECT * FROM users 
WHERE preferences @> '{"notifications": {"email": true}}';

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
- Larger index size than B-Tree
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
SELECT * FROM products 
WHERE name = 'Laptop' AND is_active = true;

-- ❌ Does NOT use index (is_active = false not in index)
SELECT * FROM products 
WHERE name = 'Laptop' AND is_active = false;

-- More examples
CREATE INDEX idx_high_value_orders ON orders(user_id, created_at)
WHERE total > 1000;  -- Only index large orders

CREATE INDEX idx_recent_posts ON posts(created_at)
WHERE created_at > NOW() - INTERVAL '30 days';  -- Only recent

CREATE INDEX idx_pending_orders ON orders(created_at DESC)
WHERE status = 'pending';  -- Only pending orders
```

**Benefits:**
- Smaller index size (faster)
- Reduced maintenance cost
- Targets specific query patterns

---

#### 5. Composite Index (Multiple Columns)

**Best for:** Queries filtering on multiple columns

```sql
-- Create composite index
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- ✅ Uses index (both columns, in order)
SELECT * FROM orders 
WHERE user_id = 1 AND status = 'pending';

-- ✅ Uses index (first column only - leftmost prefix)
SELECT * FROM orders WHERE user_id = 1;

-- ⚠️ Does NOT use index efficiently (second column without first)
SELECT * FROM orders WHERE status = 'pending';
-- Will do full scan or index-only scan (slow)

-- Column Order Rule: Most selective column FIRST
-- ❌ BAD: status has only 5 values (low selectivity)
CREATE INDEX idx_orders_bad ON orders(status, user_id);

-- ✅ GOOD: user_id has millions of unique values (high selectivity)
CREATE INDEX idx_orders_good ON orders(user_id, status);

-- Check selectivity:
SELECT 
  COUNT(DISTINCT user_id) as user_cardinality,  -- High = selective
  COUNT(DISTINCT status) as status_cardinality   -- Low = not selective
FROM orders;
-- Results: 1,000,000 vs 5 → user_id is more selective
```

**Leftmost Prefix Rule:**
```sql
-- Index on (a, b, c)
CREATE INDEX idx_abc ON table(a, b, c);

-- These queries can use the index:
WHERE a = 1                    -- ✅ Uses index on 'a'
WHERE a = 1 AND b = 2          -- ✅ Uses index on 'a, b'
WHERE a = 1 AND b = 2 AND c = 3 -- ✅ Uses full index

-- These queries CANNOT use the index efficiently:
WHERE b = 2                    -- ❌ Skips 'a'
WHERE c = 3                    -- ❌ Skips 'a' and 'b'
WHERE b = 2 AND c = 3          -- ❌ Skips 'a'
```

---

#### 6. Covering Index (Index-Only Scan)

**Definition:** Index contains ALL columns needed by query

```sql
-- Query needs: email, name, city
SELECT name, city FROM users WHERE email = 'foyez@example.com';

-- ❌ Regular index: Index lookup + table lookup (2 steps)
CREATE INDEX idx_users_email ON users(email);

-- ✅ Covering index: All data in index (1 step!)
CREATE INDEX idx_users_email_covering ON users(email) 
INCLUDE (name, city);

-- PostgreSQL also supports:
CREATE INDEX idx_users_covering ON users(email, name, city);

-- Check if query uses covering index:
EXPLAIN SELECT name, city FROM users WHERE email = 'foyez@example.com';
-- Look for "Index Only Scan"
```

**Benefits:**
- Faster queries (no table lookup needed)
- Especially good for read-heavy workloads
- Can turn 2-step query into 1-step

**Trade-offs:**
- Larger index size
- Slower writes (more data to update in index)

---

### When to Create SQL Indexes

#### ✅ CREATE Index When:

```sql
-- 1. Column used in WHERE clause frequently
SELECT * FROM products WHERE category_id = 1;
-- → CREATE INDEX idx_products_category ON products(category_id);

-- 2. Column used in JOIN conditions
SELECT * FROM orders o JOIN users u ON o.user_id = u.id;
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

### SQL Index Maintenance

```sql
-- 1. Check index usage
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

-- 2. Drop unused index
DROP INDEX idx_users_unused_column;

-- 3. Rebuild index (fix fragmentation)
REINDEX INDEX idx_users_email;
REINDEX TABLE users;  -- Rebuild all indexes on table

-- 4. Analyze table (update statistics for query planner)
ANALYZE users;

-- 5. Check index size
SELECT 
  indexname,
  pg_size_pretty(pg_relation_size(indexrelid)) as size
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC;

-- 6. Find duplicate indexes
SELECT 
  array_agg(indexname) as indexes,
  tablename,
  array_agg(indexdef) as definitions
FROM pg_indexes
WHERE schemaname = 'public'
GROUP BY tablename, indexdef
HAVING COUNT(*) > 1;
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

## NoSQL Indexing (MongoDB)

### Types of MongoDB Indexes

#### 1. Single Field Index

```javascript
// Create single field index
db.users.createIndex({ email: 1 });  // 1 = ascending, -1 = descending

// Query uses index
db.users.find({ email: "foyez@example.com" });

// Check if index used
db.users.find({ email: "foyez@example.com" }).explain("executionStats");
// Look for "stage": "IXSCAN" (index scan)
```

---

#### 2. Compound Index

```javascript
// Create compound index
db.posts.createIndex({ "author.id": 1, status: 1, publishedAt: -1 });

// ✅ Uses index (all fields, in order)
db.posts.find({ "author.id": ObjectId("..."), status: "published" })
  .sort({ publishedAt: -1 });

// ✅ Uses index (leftmost prefix)
db.posts.find({ "author.id": ObjectId("...") });

// ⚠️ Does NOT use index efficiently (skips first field)
db.posts.find({ status: "published" });

// Column order matters (same as SQL)
// Most selective field first
db.posts.createIndex({ userId: 1, status: 1 });  // ✅ Good
// vs
db.posts.createIndex({ status: 1, userId: 1 });  // ❌ Bad
```

---

#### 3. Multikey Index (Arrays)

```javascript
// Automatically created for array fields
db.posts.createIndex({ tags: 1 });

// Query: Find posts with specific tag
db.posts.find({ tags: "mongodb" });
// Index automatically handles array queries!

// Query: Find posts with any of multiple tags
db.posts.find({ tags: { $in: ["mongodb", "database"] } });

// Query: Find posts with all tags
db.posts.find({ tags: { $all: ["mongodb", "nosql"] } });
```

**Important:** Only ONE array field can be indexed in compound index
```javascript
// ❌ INVALID: Both fields are arrays
db.posts.createIndex({ tags: 1, categories: 1 });
// ERROR: Cannot index parallel arrays

// ✅ VALID: Only one array field
db.posts.createIndex({ tags: 1, status: 1 });
```

---

#### 4. Text Index (Full-Text Search)

```javascript
// Create text index
db.posts.createIndex({
  title: "text",
  content: "text",
  tags: "text"
});

// Only ONE text index per collection
// But can include multiple fields

// Search query
db.posts.find({
  $text: { $search: "mongodb tutorial guide" }
});

// Search with relevance score
db.posts.find(
  { $text: { $search: "mongodb tutorial" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } });

// Search with phrases
db.posts.find({
  $text: { $search: "\"complete guide\"" }  // Exact phrase
});

// Search with exclusion
db.posts.find({
  $text: { $search: "mongodb -mysql" }  // mongodb but not mysql
});
```

**Text Index Weights:**
```javascript
// Give more weight to title than content
db.posts.createIndex(
  {
    title: "text",
    content: "text"
  },
  {
    weights: {
      title: 10,      // 10x importance
      content: 1
    },
    name: "posts_text_index"
  }
);
```

---

#### 5. Geospatial Index (2dsphere)

```javascript
// Create geospatial index
db.restaurants.createIndex({ location: "2dsphere" });

// Location stored as GeoJSON
{
  name: "Restaurant Name",
  location: {
    type: "Point",
    coordinates: [longitude, latitude]  // [90.4125, 23.8103] for Bangladesh
  }
}

// Query: Find restaurants near a point
db.restaurants.find({
  location: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [90.4125, 23.8103]
      },
      $maxDistance: 5000  // 5km radius
    }
  }
});

// Query: Find within polygon
db.restaurants.find({
  location: {
    $geoWithin: {
      $geometry: {
        type: "Polygon",
        coordinates: [[
          [90.0, 23.0],
          [91.0, 23.0],
          [91.0, 24.0],
          [90.0, 24.0],
          [90.0, 23.0]
        ]]
      }
    }
  }
});
```

---

#### 6. Partial Index (Sparse Index)

```javascript
// Only index documents where field exists
db.posts.createIndex(
  { publishedAt: 1 },
  { sparse: true }  // Only index if publishedAt exists
);

// Only index documents matching condition
db.posts.createIndex(
  { publishedAt: -1 },
  {
    partialFilterExpression: {
      status: "published",
      "stats.views": { $gte: 1000 }
    }
  }
);
// Only indexes published posts with 1000+ views

// Benefits: Smaller index, faster maintenance
```

---

#### 7. TTL Index (Time-To-Live)

```javascript
// Auto-delete documents after time
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 3600 }  // Delete after 1 hour
);

// Document
{
  sessionId: "abc123",
  userId: 1,
  createdAt: new Date()  // Must be Date type
}

// MongoDB automatically deletes this document 1 hour after createdAt

// Use cases:
// - Session storage
// - Temporary cache
// - Event logs
// - Rate limiting counters
```

---

#### 8. Unique Index

```javascript
// Ensure unique values
db.users.createIndex({ email: 1 }, { unique: true });

// Attempt to insert duplicate fails
db.users.insertOne({ email: "existing@example.com" });
// ERROR: E11000 duplicate key error

// Compound unique index
db.followers.createIndex(
  { followerId: 1, followingId: 1 },
  { unique: true }
);
// Ensures user can't follow same person twice
```

---

### MongoDB Index Management

```javascript
// 1. List all indexes
db.posts.getIndexes();

// 2. Drop index
db.posts.dropIndex("idx_name");
db.posts.dropIndex({ tags: 1 });  // By specification

// 3. Drop all indexes (except _id)
db.posts.dropIndexes();

// 4. Rebuild indexes
db.posts.reIndex();

// 5. Check index usage
db.posts.aggregate([{ $indexStats: {} }]);

// 6. Check index size
db.posts.stats().indexSizes;

// 7. Hide index (test before dropping)
db.posts.hideIndex("idx_tags");
// Queries won't use it, but it's not deleted
db.posts.unhideIndex("idx_tags");

// 8. Analyze query performance
db.posts.find({ tags: "mongodb" }).explain("executionStats");

// Look for:
// - "stage": "IXSCAN" (good - using index)
// - "stage": "COLLSCAN" (bad - full collection scan)
// - "executionTimeMillis": execution time
// - "totalDocsExamined": documents scanned
// - "nReturned": documents returned
```

---

### When to Create MongoDB Indexes

```javascript
// ✅ CREATE Index When:

// 1. Field used in find() queries
db.users.find({ city: "Dhaka" });
// → db.users.createIndex({ city: 1 });

// 2. Field used in sort()
db.posts.find().sort({ publishedAt: -1 });
// → db.posts.createIndex({ publishedAt: -1 });

// 3. High cardinality field
db.users.find({ email: "..." });
// → db.users.createIndex({ email: 1 });

// 4. Embedded document fields
db.posts.find({ "author.id": ObjectId("...") });
// → db.posts.createIndex({ "author.id": 1 });

// 5. Array fields
db.posts.find({ tags: "mongodb" });
// → db.posts.createIndex({ tags: 1 });

// ❌ DON'T CREATE Index When:

// 1. Small collection (< 1000 docs)
// 2. Field has low cardinality
// 3. Write-heavy collection
// 4. Field rarely queried
// 5. Result set is large portion of collection
```

---

## Query Optimization

### SQL Query Optimization

#### 1. Use EXPLAIN ANALYZE

**The #1 tool for optimization**

```sql
-- PostgreSQL: Show execution plan
EXPLAIN 
SELECT * FROM orders WHERE user_id = 1 AND status = 'pending';

-- Show ACTUAL execution time
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 1 AND status = 'pending';
```

**Reading EXPLAIN Output:**

```
❌ Before optimization:
Seq Scan on orders  (cost=0.00..180.00 rows=10 width=100) 
  (actual time=0.031..2.456 rows=5 loops=1)
  Filter: ((user_id = 1) AND ((status)::text = 'pending'::text))
  Rows Removed by Filter: 995
Planning Time: 0.123 ms
Execution Time: 2.489 ms

❌ Problem: Sequential Scan (scanning all rows) = slow!
✅ Solution: Create index on (user_id, status)

✅ After creating index:
Index Scan using idx_orders_user_status on orders  
  (cost=0.29..8.31 rows=5 width=100) 
  (actual time=0.021..0.034 rows=5 loops=1)
  Index Cond: ((user_id = 1) AND ((status)::text = 'pending'::text))
Planning Time: 0.089 ms
Execution Time: 0.056 ms

✅ Result: 44x faster! (2.489ms → 0.056ms)
```

---

#### 2. SELECT Only Needed Columns

```sql
-- ❌ BAD: Retrieves all columns
SELECT * FROM products;
-- If table has 20 columns, wastes bandwidth

-- ✅ GOOD: Only needed columns
SELECT id, name, price FROM products;

-- Real impact:
-- Table: 20 columns, 1MB per row
-- SELECT *: 1000 rows = 1GB transferred
-- SELECT id, name, price: 1000 rows = 100MB
-- 10x less data!
```

---

#### 3. Use LIMIT for Pagination

```sql
-- ❌ BAD: Retrieves all rows
SELECT * FROM products ORDER BY created_at DESC;
-- Retrieves 100,000 products, shows 20

-- ✅ GOOD: Database only returns needed rows
SELECT * FROM products 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 0;  -- Page 1

SELECT * FROM products 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 20;  -- Page 2

-- ✅ BETTER: Keyset pagination (faster for large offsets)
-- Instead of OFFSET, use WHERE clause
SELECT * FROM products 
WHERE created_at < '2024-01-15 10:30:00'
ORDER BY created_at DESC 
LIMIT 20;
```

---

#### 4. Avoid SELECT DISTINCT When Possible

```sql
-- ❌ SLOW: DISTINCT sorts entire result set
SELECT DISTINCT city FROM users;

-- ✅ FASTER: GROUP BY (usually faster)
SELECT city FROM users GROUP BY city;

-- ✅ BEST: Fix data model to avoid duplicates
-- Or use EXISTS instead
```

---

#### 5. Use EXISTS Instead of IN for Large Subqueries

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

#### 6. Avoid Functions on Indexed Columns

```sql
-- ❌ BAD: Can't use index
SELECT * FROM users WHERE LOWER(email) = 'foyez@example.com';
-- Function on column prevents index usage

-- ✅ GOOD: Store data in consistent case
SELECT * FROM users WHERE email = 'foyez@example.com';

-- ✅ Alternative: Functional index
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
-- Now function queries can use index
SELECT * FROM users WHERE LOWER(email) = 'foyez@example.com';
```

---

#### 7. Use JOIN Instead of Subqueries

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

#### 8. Batch Operations

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
-- 1 round trip

-- ✅ BETTER: Use COPY for bulk inserts (PostgreSQL)
COPY products(name, price) FROM '/path/to/products.csv' WITH CSV;
-- Can be 100x faster for large datasets
```

---

#### 9. Use WHERE Before JOIN (Let Optimizer Handle It)

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

### MongoDB Query Optimization

#### 1. Use explain()

```javascript
// Check if query uses index
db.posts.find({ tags: "mongodb" }).explain("executionStats");

// Look for:
{
  "executionStats": {
    "executionTimeMillis": 15,        // ✅ Low is good
    "totalDocsExamined": 100,         // Documents scanned
    "nReturned": 100,                 // Documents returned
    "executionStages": {
      "stage": "IXSCAN"               // ✅ Using index (good)
      // "stage": "COLLSCAN"          // ❌ Collection scan (bad)
    }
  }
}

// Ideal: totalDocsExamined ≈ nReturned
// If totalDocsExamined >> nReturned, query is inefficient
```

---

#### 2. Project Only Needed Fields

```javascript
// ❌ BAD: Returns all fields
db.posts.find({ status: "published" });

// ✅ GOOD: Only needed fields
db.posts.find(
  { status: "published" },
  { title: 1, slug: 1, publishedAt: 1, _id: 0 }
);
// 1 = include, 0 = exclude
```

---

#### 3. Use Covered Queries

```javascript
// Index contains all fields in query + projection
db.posts.createIndex({ status: 1, publishedAt: 1, title: 1 });

db.posts.find(
  { status: "published" },
  { title: 1, publishedAt: 1, _id: 0 }
).explain("executionStats");

// Look for "stage": "IXSCAN" with totalDocsExamined = 0
// This means query answered entirely from index!
```

---

#### 4. Limit Results

```javascript
// ✅ Always use limit for pagination
db.posts.find({ status: "published" })
  .sort({ publishedAt: -1 })
  .limit(20)
  .skip(0);

// ⚠️ SKIP is slow for large offsets
// Better: Use range queries
db.posts.find({
  status: "published",
  publishedAt: { $lt: lastSeenDate }
})
.sort({ publishedAt: -1 })
.limit(20);
```

---

#### 5. Avoid Large Documents

```javascript
// ❌ BAD: Embedding grows huge
{
  _id: ObjectId("post_id"),
  title: "Post",
  comments: [/* 10,000 comments */]  // 16MB limit!
}

// ✅ GOOD: Reference in separate collection
// posts collection
{
  _id: ObjectId("post_id"),
  title: "Post",
  stats: { commentsCount: 10000 }
}

// comments collection
db.comments.find({ postId: ObjectId("post_id") })
  .limit(20);
```

---

#### 6. Use Aggregation Efficiently

```javascript
// ✅ Put $match early in pipeline (uses index)
db.posts.aggregate([
  { $match: { status: "published" } },  // First! Uses index
  { $sort: { publishedAt: -1 } },
  { $limit: 10 },
  { $lookup: { ... } },  // Expensive operations later
  { $project: { ... } }
]);

// ❌ BAD: $match after expensive operations
db.posts.aggregate([
  { $lookup: { ... } },  // Processes all documents
  { $match: { status: "published" } }  // Too late!
]);
```

---

#### 7. Avoid $where and $expr When Possible

```javascript
// ❌ SLOW: JavaScript execution, can't use index
db.products.find({
  $where: "this.price > this.cost * 1.5"
});

// ✅ FAST: Use query operators
db.products.find({
  $expr: {
    $gt: ["$price", { $multiply: ["$cost", 1.5] }]
  }
});

// ✅ BETTER: Pre-calculate and store
db.products.updateMany({}, [{
  $set: { marginPercent: { 
    $divide: [
      { $subtract: ["$price", "$cost"] }, 
      "$cost" 
    ]
  }}
}]);

db.products.find({ marginPercent: { $gt: 0.5 } });
// Can use index on marginPercent!
```

---

#### 8. Batch Operations

```javascript
// ❌ BAD: Multiple operations
for (let product of products) {
  await db.products.insertOne(product);
}
// N round trips

// ✅ GOOD: Batch insert
await db.products.insertMany(products);
// 1 round trip

// Bulk write operations
const bulkOps = [
  {
    updateOne: {
      filter: { _id: 1 },
      update: { $inc: { "stats.views": 1 } }
    }
  },
  {
    insertOne: {
      document: { name: "New Product" }
    }
  }
];

await db.products.bulkWrite(bulkOps);
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

Benefits:
+ Reduces database load
+ Handles traffic spikes
+ Improves user experience
+ Saves money (fewer DB resources)
```

---

### 1. Application-Level Caching (Redis)

```javascript
const Redis = require('ioredis');
const redis = new Redis();

// Without caching
async function getProduct(id) {
  const product = await db.query('SELECT * FROM products WHERE id = ?', [id]);
  return product;
}
// Every call hits database: ~50ms

// With caching
async function getProductCached(id) {
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

// First call: Cache MISS (~50ms)
// Subsequent calls: Cache HIT (~1ms)
// After 5 minutes: Cache expires, MISS again
```

---

### 2. Cache Invalidation

**"There are only two hard things in Computer Science: cache invalidation and naming things." - Phil Karlton**

```javascript
// Strategy 1: Time-based (TTL)
await redis.setex('products:trending', 300, data);  // Expires in 5 minutes

// Strategy 2: Event-based (invalidate on change)
async function updateProduct(id, data) {
  // Update database
  await db.query('UPDATE products SET ... WHERE id = ?', [id, data]);
  
  // Invalidate cache
  await redis.del(`product:${id}`);
  await redis.del('products:list');
  await redis.del('products:trending');
}

// Strategy 3: Write-through (update cache and DB together)
async function updateProductWriteThrough(id, data) {
  // 1. Update database
  await db.query('UPDATE products SET ... WHERE id = ?', [id, data]);
  
  // 2. Update cache
  const updated = await db.query('SELECT * FROM products WHERE id = ?', [id]);
  await redis.set(`product:${id}`, JSON.stringify(updated));
}

// Strategy 4: Cache-aside with versioning
async function getProductVersioned(id) {
  const version = await redis.get(`product:${id}:version`) || 0;
  const cacheKey = `product:${id}:v${version}`;
  
  let product = await redis.get(cacheKey);
  if (product) return JSON.parse(product);
  
  product = await db.query('SELECT * FROM products WHERE id = ?', [id]);
  await redis.set(cacheKey, JSON.stringify(product));
  return product;
}

// When updating, increment version
async function updateProductVersioned(id, data) {
  await db.query('UPDATE products SET ... WHERE id = ?', [id, data]);
  await redis.incr(`product:${id}:version`);  // New version
  // Old cache automatically becomes stale
}
```

---

### 3. Caching Patterns

#### Pattern 1: Cache-Aside (Lazy Loading)

```javascript
// Application checks cache first
async function getCacheAside(key) {
  // 1. Try cache
  let data = await redis.get(key);
  if (data) return JSON.parse(data);
  
  // 2. Cache miss - load from DB
  data = await db.query('...');
  
  // 3. Populate cache
  await redis.setex(key, 3600, JSON.stringify(data));
  
  return data;
}

// ✅ Pros: Only caches requested data
// ❌ Cons: First request is slow (cache miss)
```

---

#### Pattern 2: Write-Through

```javascript
// Application writes to cache and DB
async function setWriteThrough(key, data) {
  // 1. Write to cache
  await redis.set(key, JSON.stringify(data));
  
  // 2. Write to database
  await db.query('INSERT INTO ...', [data]);
}

// ✅ Pros: Cache always up-to-date
// ❌ Cons: Slower writes (2 operations)
```

---

#### Pattern 3: Write-Behind (Write-Back)

```javascript
// Application writes to cache only
// Background process syncs to DB
async function setWriteBehind(key, data) {
  // 1. Write to cache immediately
  await redis.set(key, JSON.stringify(data));
  
  // 2. Add to write queue
  await redis.lpush('write_queue', JSON.stringify({ key, data }));
}

// Background worker syncs to DB
async function syncWorker() {
  while (true) {
    const item = await redis.brpop('write_queue', 0);
    const { key, data } = JSON.parse(item[1]);
    
    await db.query('INSERT INTO ...', [data]);
  }
}

// ✅ Pros: Very fast writes
// ❌ Cons: Risk of data loss if cache fails before sync
```

---

#### Pattern 4: Refresh-Ahead

```javascript
// Proactively refresh cache before expiration
async function getRefreshAhead(key) {
  let data = await redis.get(key);
  
  if (data) {
    const ttl = await redis.ttl(key);
    
    // If TTL < 60 seconds, refresh in background
    if (ttl < 60) {
      refreshCache(key);  // Async, non-blocking
    }
    
    return JSON.parse(data);
  }
  
  // Cache miss - load synchronously
  data = await loadAndCache(key);
  return data;
}

async function refreshCache(key) {
  const data = await db.query('...');
  await redis.setex(key, 3600, JSON.stringify(data));
}

// ✅ Pros: Users rarely experience cache misses
// ❌ Cons: More complex, may refresh unused data
```

---

### 4. Database-Level Caching

#### PostgreSQL Shared Buffers

```sql
-- PostgreSQL caches frequently accessed pages in RAM

-- Check current setting
SHOW shared_buffers;  -- Default: 128MB

-- Increase for better performance (25% of total RAM recommended)
ALTER SYSTEM SET shared_buffers = '4GB';
-- Restart required

-- Check cache hit rate
SELECT 
  sum(heap_blks_read) as heap_read,
  sum(heap_blks_hit) as heap_hit,
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as cache_hit_ratio
FROM pg_statio_user_tables;
-- Aim for > 0.99 (99% cache hit rate)
```

---

#### MongoDB WiredTiger Cache

```javascript
// MongoDB caches documents and indexes in RAM

// Check cache statistics
db.serverStatus().wiredTiger.cache

// Configure cache size (mongod.conf)
storage:
  wiredTiger:
    engineConfig:
      cacheSizeGB: 4  // 50% of RAM minus 1GB recommended
```

---

### 5. Materialized Views (Pre-computed Results)

#### SQL Materialized View

```sql
-- Expensive query: Product statistics
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

-- Refresh periodically (cron job)
REFRESH MATERIALIZED VIEW product_stats;

-- Or concurrent refresh (doesn't lock readers)
REFRESH MATERIALIZED VIEW CONCURRENTLY product_stats;
```

---

### Cache Best Practices

```javascript
// 1. Set appropriate TTL based on data volatility
const ttls = {
  static: 86400,      // 24 hours (privacy policy)
  semiStatic: 3600,   // 1 hour (product info)
  dynamic: 300,       // 5 minutes (stock levels)
  realtime: 60        // 1 minute (trending)
};

// 2. Use cache prefixes for organization
const keys = {
  user: (id) => `user:${id}`,
  product: (id) => `product:${id}`,
  cart: (userId) => `cart:${userId}`,
  session: (sessionId) => `session:${sessionId}`
};

// 3. Monitor cache hit rate
let hits = 0, misses = 0;

async function getCached(key) {
  const data = await redis.get(key);
  
  if (data) {
    hits++;
    return JSON.parse(data);
  }
  
  misses++;
  // Load from DB...
}

setInterval(() => {
  const hitRate = hits / (hits + misses);
  console.log(`Cache hit rate: ${(hitRate * 100).toFixed(2)}%`);
  // Aim for > 90%
}, 60000);

// 4. Handle cache stampede (thundering herd)
async function getProductSafe(id) {
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
    // Other requests wait and retry
    await sleep(50);
    return getProductSafe(id);
  }
}

// 5. Cache hot data (frequently accessed)
// ✅ Product details, user profiles, trending lists
// ❌ Admin logs, old data
// Cache warming (preload popular data)
async function warmCache() {
  // Load top 100 products into cache
  const products = await db.query('SELECT * FROM products ORDER BY sales DESC LIMIT 100');
  
  for (const product of products) {
    await redis.setex(`product:${product.id}`, 3600, JSON.stringify(product));
  }
}

// Run on startup
warmCache();
```

---

## Database Scaling Strategies

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
- **Simple** - No code changes required
- **No synchronization** - Single source of truth
- **Easier maintenance** - One server to manage
- **Consistent** - No replication lag
- **Transactions** - ACID works perfectly

**❌ Cons:**
- **Expensive** - Enterprise hardware costs scale non-linearly
- **Limited ceiling** - Physical limits (CPU, RAM, disk)
- **Single point of failure** - Server down = everything down
- **Downtime** - Upgrades require restart
- **Diminishing returns** - 2x cost ≠ 2x performance

**When to Use:**
- Small to medium applications
- Budget/timeline constraints
- Simple architecture preferred
- Strong consistency required
- < 10,000 concurrent users

---

#### Horizontal Scaling (Scale Out) ➡️

**Definition:** Add more servers (distributed system)

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
- **Unlimited scaling** - Add servers as needed
- **Cheaper** - Commodity hardware
- **No downtime** - Rolling updates
- **Fault tolerant** - One fails, others work
- **Geographic distribution** - Servers in multiple regions

**❌ Cons:**
- **Complex** - Data synchronization required
- **Application changes** - Code must handle distributed data
- **More infrastructure** - Load balancers, monitoring, etc.
- **CAP theorem** - Must choose between consistency and availability
- **Network latency** - Inter-server communication overhead

**When to Use:**
- Large applications (millions of users)
- Need high availability (99.99%+)
- Expect rapid growth
- Geographic distribution needed
- Read-heavy workloads

---

## Replication (Read Scaling)

### SQL Replication (PostgreSQL/MySQL)

**Problem:** 90% of operations are reads, but only 1 server handles them.

**Solution:** Create read-only copies (replicas)

```
                    ┌─────────────┐
                    │   PRIMARY   │ (Master)
                    │   WRITES    │
                    └──────┬──────┘
                           │
                   Async Replication
                    (WAL Streaming)
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼─────┐      ┌────▼─────┐      ┌────▼─────┐
    │ Replica1 │      │ Replica2 │      │ Replica3 │
    │ (READS)  │      │ (READS)  │      │ (READS)  │
    └──────────┘      └──────────┘      └──────────┘
      US East           EU West            Asia
```

#### How It Works

**1. Write-Ahead Log (WAL) Streaming**
```
Primary:
1. Client writes data
2. Write to WAL first
3. Apply to database
4. Stream WAL to replicas

Replicas:
1. Receive WAL stream
2. Apply changes to local database
3. Serve read queries
```

#### PostgreSQL Replication Setup

```sql
-- ==========================================
-- PRIMARY SERVER Configuration
-- ==========================================

-- postgresql.conf
wal_level = replica                    # Enable replication
max_wal_senders = 3                    # Number of replicas
wal_keep_size = 1GB                    # Keep WAL for replicas
hot_standby = on                       # Allow reads on standby
archive_mode = on                      # Enable WAL archiving
archive_command = 'cp %p /archive/%f'  # Archive command

-- Create replication user
CREATE USER replicator REPLICATION LOGIN ENCRYPTED PASSWORD 'password';

-- pg_hba.conf (allow replication connections)
host replication replicator 10.0.0.0/8 md5

-- ==========================================
-- REPLICA SERVER Setup
-- ==========================================

-- Stop PostgreSQL on replica
systemctl stop postgresql

-- Remove existing data
rm -rf /var/lib/postgresql/14/main/*

-- Create base backup from primary
pg_basebackup -h primary.db.example.com \
  -D /var/lib/postgresql/14/main \
  -U replicator \
  -P \
  -X stream

-- Create standby.signal file
touch /var/lib/postgresql/14/main/standby.signal

-- postgresql.auto.conf (or recovery.conf for older versions)
primary_conninfo = 'host=primary.db.example.com port=5432 user=replicator password=password'
primary_slot_name = 'replica1_slot'

-- Start replica
systemctl start postgresql

-- ==========================================
-- Verify Replication
-- ==========================================

-- On primary: Check replication status
SELECT * FROM pg_stat_replication;

-- On replica: Check if receiving
SELECT pg_is_in_recovery();  -- Should return true
```

#### Application Implementation

```javascript
const { Pool } = require('pg');

// Primary database (writes)
const primary = new Pool({
  host: 'primary.db.example.com',
  database: 'mydb',
  user: 'app_user',
  password: 'password',
  max: 20
});

// Read replicas
const replicas = [
  new Pool({ host: 'replica1.db.example.com', database: 'mydb', user: 'app_user', password: 'password' }),
  new Pool({ host: 'replica2.db.example.com', database: 'mydb', user: 'app_user', password: 'password' }),
  new Pool({ host: 'replica3.db.example.com', database: 'mydb', user: 'app_user', password: 'password' })
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
  return await replica.query(
    'SELECT * FROM users ORDER BY created_at DESC LIMIT 20'
  );
}

// Read from primary if consistency critical
async function getUserBalance(userId) {
  // Financial data - must be from primary!
  return await primary.query(
    'SELECT balance FROM accounts WHERE user_id = $1',
    [userId]
  );
}

// Read-after-write pattern
async function createAndGetUser(userData) {
  // 1. Write to primary
  const result = await primary.query(
    'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING id',
    [userData.name, userData.email]
  );
  
  const userId = result.rows[0].id;
  
  // 2. Read from primary (ensure we see our write)
  const user = await primary.query(
    'SELECT * FROM users WHERE id = $1',
    [userId]
  );
  
  return user.rows[0];
}
```

---

### MongoDB Replication (Replica Sets)

```
                    ┌─────────────┐
                    │   PRIMARY   │
                    │   WRITES    │
                    └──────┬──────┘
                           │
                      Oplog Sync
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼─────┐     ┌────▼─────┐     ┌────▼─────┐
    │Secondary1│     │Secondary2│     │Secondary3│
    │  READS   │     │  READS   │     │  READS   │
    └──────────┘     └──────────┘     └──────────┘
```

#### MongoDB Replica Set Setup

```javascript
// ==========================================
// Initialize Replica Set
// ==========================================

// Connect to first node
mongo mongodb://localhost:27017

// Initialize replica set
rs.initiate({
  _id: "myReplicaSet",
  members: [
    { _id: 0, host: "mongo1.example.com:27017", priority: 2 },  // Primary (higher priority)
    { _id: 1, host: "mongo2.example.com:27017", priority: 1 },  // Secondary
    { _id: 2, host: "mongo3.example.com:27017", priority: 1 }   // Secondary
  ]
});

// Check status
rs.status();

// Check which is primary
rs.isMaster();

// ==========================================
// Application Configuration
// ==========================================

const { MongoClient } = require('mongodb');

const client = new MongoClient('mongodb://mongo1.example.com:27017,mongo2.example.com:27017,mongo3.example.com:27017/mydb?replicaSet=myReplicaSet', {
  readPreference: 'secondaryPreferred',  // Read from secondaries if available
  w: 'majority'  // Write to majority before acknowledging
});

await client.connect();
const db = client.db('mydb');

// Write (always goes to primary)
await db.collection('users').insertOne({
  name: "Foyez",
  email: "foyez@example.com"
});

// Read (uses read preference)
const users = await db.collection('users').find().toArray();
// Reads from secondary if available

// Force read from primary
const user = await db.collection('users')
  .find({ email: "foyez@example.com" })
  .readPref('primary')
  .toArray();

// ==========================================
// Read Preferences
// ==========================================

// primary: All reads from primary (strong consistency)
{ readPreference: 'primary' }

// primaryPreferred: Primary if available, else secondary
{ readPreference: 'primaryPreferred' }

// secondary: Only from secondaries (eventual consistency)
{ readPreference: 'secondary' }

// secondaryPreferred: Secondary if available, else primary (recommended)
{ readPreference: 'secondaryPreferred' }

// nearest: Lowest latency node
{ readPreference: 'nearest' }
```

---

### Replication Lag

**Problem:** Replicas are slightly behind primary

```
Timeline:
00:00:00  Primary: INSERT user (id=1)
00:00:00  Replica: [not yet replicated]
00:00:01  User queries replica: User not found!
00:00:02  Replica: [replication complete]
00:00:02  User queries replica: User found!
```

**Solutions:**

```javascript
// Solution 1: Read from primary after write
async function createUserConsistent(userData) {
  // Write to primary
  await primary.query('INSERT INTO users ...');
  
  // Read from primary (ensures consistency)
  return await primary.query('SELECT * FROM users WHERE ...');
}

// Solution 2: Wait for replication (MongoDB)
async function createUserWithWriteConcern(userData) {
  await db.collection('users').insertOne(
    userData,
    { writeConcern: { w: 'majority', wtimeout: 5000 } }
  );
  // Waits until majority of replicas have the write
}

// Solution 3: Use session/cookie to track writes
async function createUserWithTracking(userData) {
  const result = await primary.query('INSERT INTO users ... RETURNING id');
  const userId = result.rows[0].id;
  
  // Store in session
  req.session.justCreatedUser = { id: userId, timestamp: Date.now() };
  
  return userId;
}

async function getUserSmart(userId, req) {
  // If just created (< 5 seconds ago), read from primary
  if (req.session.justCreatedUser?.id === userId &&
      Date.now() - req.session.justCreatedUser.timestamp < 5000) {
    return await primary.query('SELECT * FROM users WHERE id = $1', [userId]);
  }
  
  // Otherwise, read from replica
  const replica = getReadReplica();
  return await replica.query('SELECT * FROM users WHERE id = $1', [userId]);
}
```

---

### Benefits & Challenges

**✅ Benefits:**
- **Distribute read load** across multiple servers
- **Geographic distribution** (lower latency for users)
- **High availability** (promote replica if primary fails)
- **Backup** (replicas can be used for backups)
- **Analytics** (run expensive queries on replica)

**❌ Challenges:**
- **Replication lag** (eventual consistency)
- **Complexity** (application must route queries)
- **Consistency** (stale reads possible)
- **Failover** (automatic or manual promotion)
- **Cost** (multiple servers)

---

## Sharding (Write Scaling)

**Problem:** Single database can't handle millions of writes per second.

**Solution:** Split data across multiple databases (shards)

### Horizontal Sharding

#### By User ID (Hash-Based)

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Shard 0    │    │   Shard 1    │    │   Shard 2    │
│ Users 0-333K │    │Users 333K-666K│   │Users 666K-1M │
├──────────────┤    ├──────────────┤    ├──────────────┤
│ id: 150000   │    │ id: 450000   │    │ id: 750000   │
│ name: Alice  │    │ name: Bob    │    │ name: Carol  │
│ email: ...   │    │ email: ...   │    │ email: ...   │
└──────────────┘    └──────────────┘    └──────────────┘
```

**Shard Selection Logic:**
```javascript
function getShardForUser(userId) {
  return userId % 3;  // 3 shards (0, 1, 2)
}

const shards = [
  new Pool({ host: 'shard0.db.example.com', ... }),
  new Pool({ host: 'shard1.db.example.com', ... }),
  new Pool({ host: 'shard2.db.example.com', ... })
];

// Write to appropriate shard
async function createOrder(userId, orderData) {
  const shardIndex = getShardForUser(userId);
  const shard = shards[shardIndex];
  
  return await shard.query(
    'INSERT INTO orders (user_id, total) VALUES ($1, $2) RETURNING *',
    [userId, orderData.total]
  );
}

// Read from appropriate shard
async function getUserOrders(userId) {
  const shardIndex = getShardForUser(userId);
  const shard = shards[shardIndex];
  
  return await shard.query(
    'SELECT * FROM orders WHERE user_id = $1 ORDER BY created_at DESC',
    [userId]
  );
}
```

---

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

```javascript
function getShardByRegion(country) {
  const regionMap = {
    'BD': 'asia',
    'IN': 'asia',
    'CN': 'asia',
    'DE': 'eu',
    'FR': 'eu',
    'UK': 'eu',
    'US': 'us',
    'CA': 'us'
  };
  
  return regionMap[country] || 'us';  // Default to US
}

const regionShards = {
  asia: new Pool({ host: 'asia.db.example.com', ... }),
  eu: new Pool({ host: 'eu.db.example.com', ... }),
  us: new Pool({ host: 'us.db.example.com', ... })
};

async function createUser(userData) {
  const region = getShardByRegion(userData.country);
  const shard = regionShards[region];
  
  return await shard.query('INSERT INTO users ...', [userData]);
}
```

---

### MongoDB Sharding

MongoDB has **built-in sharding** that's automatic and transparent.

```
                    ┌─────────────────┐
                    │  Query Router   │
                    │    (mongos)     │
                    └────────┬────────┘
                             │
                  ┌──────────┼──────────┐
                  │          │          │
            ┌─────▼────┐ ┌──▼─────┐ ┌──▼─────┐
            │ Shard 0  │ │Shard 1 │ │Shard 2 │
            │ Asia     │ │ EU     │ │ US     │
            │ Replica  │ │Replica │ │Replica │
            │ Set      │ │Set     │ │Set     │
            └──────────┘ └────────┘ └────────┘
```

#### MongoDB Sharding Setup

```javascript
// ==========================================
// Enable Sharding
// ==========================================

// Connect to mongos (query router)
mongo mongodb://mongos.example.com:27017

// Enable sharding on database
sh.enableSharding("mydb");

// Shard collection by user_id
sh.shardCollection("mydb.orders", { user_id: 1 });

// MongoDB automatically distributes data:
// Shard 0: user_id 0 - 333333
// Shard 1: user_id 333334 - 666666
// Shard 2: user_id 666667 - 999999

// ==========================================
// Application Code (No Changes!)
// ==========================================

const client = new MongoClient('mongodb://mongos.example.com:27017/mydb');
await client.connect();
const db = client.db('mydb');

// MongoDB routes automatically
await db.collection('orders').insertOne({
  user_id: 450000,
  total: 150.00
});
// Automatically routed to Shard 1!

await db.collection('orders').find({ user_id: 450000 }).toArray();
// Automatically routed to Shard 1!

// ==========================================
// Shard Key Selection
// ==========================================

// ✅ Good shard keys:
// - High cardinality (many unique values)
// - Even distribution
// - Query pattern aligned (frequently used in queries)

sh.shardCollection("mydb.posts", { author_id: 1 });          // ✅ Good
sh.shardCollection("mydb.users", { email: "hashed" });       // ✅ Good (hashed)
sh.shardCollection("mydb.events", { timestamp: 1, user_id: 1 });  // ✅ Good (compound)

// ❌ Bad shard keys:
sh.shardCollection("mydb.users", { country: 1 });            // ❌ Low cardinality
sh.shardCollection("mydb.logs", { status: 1 });              // ❌ Few values
sh.shardCollection("mydb.orders", { created_at: 1 });        // ❌ Monotonic (hotspot)
```

---

### Benefits:
- Distribute writes across multiple databases
- Each shard smaller and faster
- Geographic locality (lower latency for users)
- Unlimited scaling potential

### Sharding Challenges

#### Challenge 1: Cross-Shard Queries

```javascript
// ❌ Can't JOIN across shards
// Must query each shard and merge in application
async function getTopUsers() {
  const results = await Promise.all([
    shards[0].query('SELECT * FROM users ORDER BY score DESC LIMIT 10'),
    shards[1].query('SELECT * FROM users ORDER BY score DESC LIMIT 10'),
    shards[2].query('SELECT * FROM users ORDER BY score DESC LIMIT 10')
  ]);
  
  // Merge and sort in application
  const allUsers = [...results[0], ...results[1], ...results[2]];
  return allUsers.sort((a, b) => b.score - a.score).slice(0, 10);
}

// MongoDB scatter-gather (automatic)
db.users.find().sort({ score: -1 }).limit(10);
// Mongos queries all shards and merges results
```

---

#### Challenge 2: Distributed Transactions

```javascript
// ❌ Can't have ACID transaction across shards (SQL)
// Must use saga pattern or 2-phase commit

// MongoDB supports multi-shard transactions (since v4.2)
const session = client.startSession();

try {
  await session.startTransaction();
  
  // Update in Shard 0
  await db.collection('users').updateOne(
    { _id: user1Id },
    { $inc: { balance: -100 } },
    { session }
  );
  
  // Update in Shard 1 (different shard)
  await db.collection('users').updateOne(
    { _id: user2Id },
    { $inc: { balance: 100 } },
    { session }
  );
  
  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
} finally {
  session.endSession();
}
```

---

#### Challenge 3: Rebalancing

```javascript
// Adding new shard changes hash function
// Old: userId % 3
// New: userId % 4

// Problem: user_id 6 was in shard 0 (6 % 3 = 0)
//          user_id 6 now in shard 2 (6 % 4 = 2)
// Must migrate data!

// Solution: Consistent hashing
function consistentHash(userId, numShards) {
  const hash = crypto.createHash('md5')
    .update(userId.toString())
    .digest('hex');
  const hashInt = parseInt(hash.substring(0, 8), 16);
  return hashInt % numShards;
}

// Minimizes data movement when adding shards

// MongoDB handles rebalancing automatically
sh.addShard("mongodb://new-shard:27017");
// MongoDB migrates data automatically (balancer)
```

---

#### Challenge 4: Hotspots

```javascript
// Problem: Celebrity user creates millions of records
// Their shard becomes overloaded

// Solution 1: Compound shard key
sh.shardCollection("mydb.posts", { author_id: 1, created_at: 1 });
// Distributes posts across time ranges

// Solution 2: Hash shard key
sh.shardCollection("mydb.users", { user_id: "hashed" });
// Even distribution regardless of user_id pattern

// Solution 3: Further shard hot keys
if (isCelebrityUser(userId)) {
  return getCelebrityShard(userId);  // Dedicated shards for celebrities
}
```

---

## Connection Pooling

**Problem:** Opening new database connection is expensive (~100ms)

**Solution:** Reuse connections from a pool

### SQL Connection Pooling

```javascript
const { Pool } = require('pg');

// ❌ Without pooling (slow)
async function getUserBad(id) {
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
  
  // Pool configuration
  max: 20,                    // Maximum connections
  min: 5,                     // Minimum idle connections
  idleTimeoutMillis: 30000,   // Close idle connections after 30s
  connectionTimeoutMillis: 2000  // Wait 2s for connection
});

async function getUser(id) {
  const client = await pool.connect();  // < 1ms (reuses existing)
  
  try {
    const result = await client.query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows[0];
  } finally {
    client.release();  // Return to pool (don't close!)
  }
}

// 100x faster! (~1ms vs ~100ms)

// Monitor pool
pool.on('connect', () => {
  console.log('New client connected to pool');
});

pool.on('acquire', () => {
  console.log('Client acquired from pool');
});

pool.on('remove', () => {
  console.log('Client removed from pool');
});

pool.on('error', (err, client) => {
  console.error('Unexpected error on idle client', err);
});

// Check pool stats
setInterval(() => {
  console.log({
    total: pool.totalCount,      // Total connections
    idle: pool.idleCount,         // Idle connections
    waiting: pool.waitingCount    // Waiting requests
  });
}, 10000);

// Alert if pool exhausted
if (pool.waitingCount > 0) {
  console.warn('Connection pool exhausted! Increase max or optimize queries.');
}
```

---

### MongoDB Connection Pooling

```javascript
const { MongoClient } = require('mongodb');

const client = new MongoClient('mongodb://localhost:27017', {
  maxPoolSize: 50,        // Maximum connections
  minPoolSize: 10,        // Minimum connections
  maxIdleTimeMS: 30000,   // Close idle after 30s
  waitQueueTimeoutMS: 5000  // Wait 5s for connection
});

await client.connect();
const db = client.db('mydb');

// Connection automatically managed
const users = await db.collection('users').find().toArray();
// Uses connection from pool

// Monitor connection pool
client.on('connectionPoolCreated', (event) => {
  console.log('Pool created', event);
});

client.on('connectionCreated', (event) => {
  console.log('Connection created', event.connectionId);
});

client.on('connectionReady', (event) => {
  console.log('Connection ready', event.connectionId);
});

client.on('connectionClosed', (event) => {
  console.log('Connection closed', event.connectionId);
});

client.on('connectionCheckOutStarted', (event) => {
  console.log('Connection checkout started');
});

client.on('connectionCheckOutFailed', (event) => {
  console.error('Connection checkout failed', event.reason);
});
```

---

### Redis Connection Pooling

```javascript
const Redis = require('ioredis');

// Create pool-like behavior with cluster
const cluster = new Redis.Cluster([
  { host: 'redis1.example.com', port: 6379 },
  { host: 'redis2.example.com', port: 6379 },
  { host: 'redis3.example.com', port: 6379 }
], {
  redisOptions: {
    maxRetriesPerRequest: 3,
    connectTimeout: 10000
  }
});

// Or use single connection with reconnection
const redis = new Redis({
  host: 'localhost',
  port: 6379,
  retryStrategy: (times) => {
    const delay = Math.min(times * 50, 2000);
    return delay;
  },
  maxRetriesPerRequest: 3
});

// Connection maintained automatically
await redis.set('key', 'value');
await redis.get('key');
```

---

## Advanced Performance Concepts

### 1. Query Result Streaming

```javascript
// ❌ BAD: Load all results into memory
const users = await pool.query('SELECT * FROM users');
// If 1M users, loads all into memory (~1GB)

// ✅ GOOD: Stream results
const { Readable } = require('stream');

async function* streamUsers() {
  const client = await pool.connect();
  
  try {
    const cursor = client.query(
      new Cursor('SELECT * FROM users')
    );
    
    let rows;
    while ((rows = await cursor.read(100))) {
      for (const row of rows) {
        yield row;
      }
    }
  } finally {
    client.release();
  }
}

// Process in chunks
for await (const user of streamUsers()) {
  await processUser(user);
}
```

---

### 2. Prepared Statements

```javascript
// ✅ Prepared statements (faster, safer)
const preparedQuery = {
  name: 'get-user',
  text: 'SELECT * FROM users WHERE id = $1',
  // Query plan cached by database
};

// First execution: Parse + plan + execute
await pool.query(preparedQuery, [1]);

// Subsequent executions: Use cached plan (faster!)
await pool.query(preparedQuery, [2]);
await pool.query(preparedQuery, [3]);

// MongoDB: Queries are automatically cached
```

---

### 3. Bulk Operations

```javascript
// SQL: Batch inserts
const users = [...];  // 1000 users

// ❌ BAD: 1000 queries
for (const user of users) {
  await pool.query('INSERT INTO users (name, email) VALUES ($1, $2)', [user.name, user.email]);
}
// 1000 round trips

// ✅ GOOD: Single batch query
const values = users.map(u => `('${u.name}', '${u.email}')`).join(',');
await pool.query(`INSERT INTO users (name, email) VALUES ${values}`);
// 1 round trip

// ✅ BEST: Use COPY for huge datasets
const copyStream = client.query(copyFrom('COPY users (name, email) FROM STDIN CSV'));
copyStream.write('Alice,alice@example.com\n');
copyStream.write('Bob,bob@example.com\n');
copyStream.end();
// 100x faster for millions of rows

// MongoDB: Bulk operations
await db.collection('users').insertMany(users, { ordered: false });
// Parallel inserts
```

---

### 4. Pagination Optimization

```sql
-- ❌ SLOW: Large OFFSET
SELECT * FROM posts 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 10000;
-- Database still scans 10,020 rows

-- ✅ FAST: Keyset pagination
SELECT * FROM posts 
WHERE created_at < '2024-01-15 10:30:00'
ORDER BY created_at DESC 
LIMIT 20;
-- Uses index efficiently

-- Implementation
function paginatePosts(lastSeenDate = null, limit = 20) {
  if (lastSeenDate) {
    return pool.query(
      'SELECT * FROM posts WHERE created_at < $1 ORDER BY created_at DESC LIMIT $2',
      [lastSeenDate, limit]
    );
  } else {
    return pool.query(
      'SELECT * FROM posts ORDER BY created_at DESC LIMIT $1',
      [limit]
    );
  }
}
```

---

### 5. Database Partitioning

```sql
-- Split large tables by date range
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

-- Queries automatically use correct partition
SELECT * FROM orders WHERE created_at >= '2024-01-15';
-- Only scans orders_2024_01 (faster!)

-- Benefits:
-- 1. Smaller indexes (per partition)
-- 2. Faster queries (scan only relevant partitions)
-- 3. Easy to drop old data (DROP TABLE orders_2023_12)
-- 4. Parallel query execution
```

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

#### SQL (PostgreSQL)

```sql
-- ❌ BAD: Give admin access to application
GRANT ALL PRIVILEGES ON DATABASE mydb TO app_user;
-- Application can DROP tables, CREATE users, DELETE everything!

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

**Role-Based Access Control (RBAC)**

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

#### MongoDB

```javascript
// Create users with specific roles
db.createUser({
  user: "app_user",
  pwd: "secure_password",
  roles: [
    { role: "readWrite", db: "mydb" },  // Read/write access
    { role: "read", db: "analytics" }   // Read-only on analytics
  ]
});

// Read-only user
db.createUser({
  user: "readonly_user",
  pwd: "password",
  roles: [{ role: "read", db: "mydb" }]
});

// Custom role
db.createRole({
  role: "orderManager",
  privileges: [
    {
      resource: { db: "mydb", collection: "orders" },
      actions: ["find", "insert", "update"]
    },
    {
      resource: { db: "mydb", collection: "products" },
      actions: ["find"]  // Read-only on products
    }
  ],
  roles: []
});

// Grant custom role
db.grantRolesToUser("app_user", ["orderManager"]);
```

---

### 2. SQL Injection Prevention

**SQL Injection:** Attacker manipulates SQL queries through user input.

```javascript
// ❌ DANGEROUS: SQL Injection vulnerable
async function getUserBad(userId) {
  const query = `SELECT * FROM users WHERE id = ${userId}`;
  return await db.query(query);
}

// Attack 1:
// Input: "1 OR 1=1 --"
// Query becomes: SELECT * FROM users WHERE id = 1 OR 1=1 --
// Returns ALL users!

// Attack 2:
// Input: "1; DROP TABLE users; --"
// Query becomes: SELECT * FROM users WHERE id = 1; DROP TABLE users; --
// DELETES ENTIRE TABLE!

// Attack 3:
// Input: "1 UNION SELECT username, password, NULL FROM admin_users--"
// Exposes admin passwords!

// ✅ SAFE: Parameterized queries (PostgreSQL)
async function getUserSafe(userId) {
  return await db.query(
    'SELECT * FROM users WHERE id = $1',
    [userId]  // Parameters are escaped automatically
  );
}

// ✅ SAFE: Prepared statements (MySQL)
const sql = mysql.format('SELECT * FROM users WHERE id = ?', [userId]);

// ✅ SAFE: Use ORM (Prisma)
const user = await prisma.user.findUnique({
  where: { id: userId }
});

// ✅ SAFE: Input validation
function validateUserId(userId) {
  const id = parseInt(userId, 10);
  if (isNaN(id) || id < 1 || id > Number.MAX_SAFE_INTEGER) {
    throw new Error('Invalid user ID');
  }
  return id;
}

const safeUserId = validateUserId(req.params.id);
```

**NoSQL Injection (MongoDB):**

```javascript
// ❌ VULNERABLE: NoSQL injection
async function loginBad(username, password) {
  return await db.collection('users').findOne({
    username: username,
    password: password
  });
}

// Attack:
// Input: username = "admin", password = { $ne: "" }
// Query becomes: { username: "admin", password: { $ne: "" } }
// Matches admin with ANY password!

// ✅ SAFE: Validate input types
async function loginSafe(username, password) {
  if (typeof username !== 'string' || typeof password !== 'string') {
    throw new Error('Invalid input types');
  }
  
  return await db.collection('users').findOne({
    username: username,
    password: hashPassword(password)  // Never store plain passwords!
  });
}

// ✅ SAFE: Sanitize input
const sanitize = require('mongo-sanitize');

async function loginSanitized(username, password) {
  return await db.collection('users').findOne({
    username: sanitize(username),
    password: hashPassword(sanitize(password))
  });
}
```

---

### 3. Password Security

```javascript
const bcrypt = require('bcrypt');
const crypto = require('crypto');

// ❌ NEVER store plain passwords
async function registerUserBad(email, password) {
  await db.query(
    'INSERT INTO users (email, password) VALUES ($1, $2)',
    [email, password]  // DANGER! Plain text!
  );
}

// ✅ Always hash passwords with salt
async function registerUser(email, password) {
  const saltRounds = 12;  // Cost factor (higher = more secure, slower)
  const hashedPassword = await bcrypt.hash(password, saltRounds);
  
  await db.query(
    'INSERT INTO users (email, password_hash) VALUES ($1, $2)',
    [email, hashedPassword]
  );
}

// Verify password
async function loginUser(email, password) {
  const user = await db.query(
    'SELECT id, password_hash FROM users WHERE email = $1',
    [email]
  );
  
  if (!user || !user.rows[0]) {
    return null;
  }
  
  const isValid = await bcrypt.compare(password, user.rows[0].password_hash);
  return isValid ? user.rows[0] : null;
}

// Generate secure random tokens
function generateToken() {
  return crypto.randomBytes(32).toString('hex');
}

// Generate secure session ID
function generateSessionId() {
  return crypto.randomBytes(16).toString('hex');
}

// Password strength validation
function validatePasswordStrength(password) {
  if (password.length < 8) {
    throw new Error('Password must be at least 8 characters');
  }
  
  if (!/[A-Z]/.test(password)) {
    throw new Error('Password must contain uppercase letter');
  }
  
  if (!/[a-z]/.test(password)) {
    throw new Error('Password must contain lowercase letter');
  }
  
  if (!/[0-9]/.test(password)) {
    throw new Error('Password must contain number');
  }
  
  if (!/[^A-Za-z0-9]/.test(password)) {
    throw new Error('Password must contain special character');
  }
  
  return true;
}
```

---

### 4. Encryption

#### Encryption at Rest (SQL)

```sql
-- PostgreSQL: Encrypt sensitive columns
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

-- Use environment variable for key (don't hardcode!)
SELECT pgp_sym_encrypt(sensitive_data, current_setting('app.encryption_key'));
```

#### Encryption in Transit

```javascript
// PostgreSQL: SSL/TLS connection
const { Pool } = require('pg');

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

// MongoDB: TLS connection
const { MongoClient } = require('mongodb');

const client = new MongoClient('mongodb://localhost:27017', {
  tls: true,
  tlsCAFile: '/path/to/ca.pem',
  tlsCertificateKeyFile: '/path/to/client.pem'
});
```

#### MongoDB Field-Level Encryption

```javascript
// Client-Side Field Level Encryption (CSFLE)
const { ClientEncryption } = require('mongodb-client-encryption');

const encryption = new ClientEncryption(client, {
  keyVaultNamespace: 'encryption.__keyVault',
  kmsProviders: {
    local: {
      key: Buffer.from(masterKey, 'base64')
    }
  }
});

// Encrypt before insert
const encryptedSSN = await encryption.encrypt('123-45-6789', {
  algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic',
  keyId: dataKeyId
});

await db.collection('users').insertOne({
  name: "Foyez",
  ssn: encryptedSSN  // Encrypted value
});

// Decrypt after query
const user = await db.collection('users').findOne({ name: "Foyez" });
const decryptedSSN = await encryption.decrypt(user.ssn);
```

---

### 5. Environment Variables & Secrets Management

```javascript
// ❌ NEVER hardcode credentials
const pool = new Pool({
  host: 'localhost',
  user: 'postgres',
  password: 'supersecret123'  // DANGER!
});

// ✅ Use environment variables
require('dotenv').config();

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
DB_PASSWORD=super_secure_password_here_with_special_chars!@#$
DB_SSL=true

// .gitignore
.env
.env.local
.env.production
.env.*.local

// ✅ BETTER: Use secrets manager (AWS Secrets Manager, Azure Key Vault)
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager({ region: 'us-east-1' });

async function getDbCredentials() {
  const secret = await secretsManager.getSecretValue({
    SecretId: 'prod/database/credentials'
  }).promise();
  
  return JSON.parse(secret.SecretString);
}

const dbCreds = await getDbCredentials();
const pool = new Pool(dbCreds);
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

## Monitoring & Observability

### Key Metrics to Monitor

#### 1. Query Performance (SQL)

```sql
-- PostgreSQL: Enable pg_stat_statements
CREATE EXTENSION pg_stat_statements;

-- View slow queries
SELECT 
  query,
  calls,
  total_exec_time,
  mean_exec_time,
  max_exec_time,
  stddev_exec_time,
  rows
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

-- Reset statistics
SELECT pg_stat_statements_reset();
```

#### 2. Query Performance (MongoDB)

```javascript
// Enable profiling
db.setProfilingLevel(2);  // Profile all operations
// 0 = off, 1 = slow ops only, 2 = all ops

// View slow queries
db.system.profile.find({
  millis: { $gt: 100 }  // Slower than 100ms
}).sort({ ts: -1 }).limit(10);

// Profiling with threshold
db.setProfilingLevel(1, { slowms: 100 });  // Only ops > 100ms

// Check current profiling level
db.getProfilingStatus();

// View profiling data
db.system.profile.find().sort({ ts: -1 }).pretty();

// Disable profiling
db.setProfilingLevel(0);
```

---

#### 3. Connection Pool Metrics

```javascript
// PostgreSQL
setInterval(() => {
  console.log({
    total: pool.totalCount,
    idle: pool.idleCount,
    waiting: pool.waitingCount,
    utilization: ((pool.totalCount - pool.idleCount) / pool.totalCount * 100).toFixed(2) + '%'
  });
  
  // Alert if utilization > 80%
  if ((pool.totalCount - pool.idleCount) / pool.totalCount > 0.8) {
    console.warn('⚠️  Connection pool utilization > 80%!');
  }
  
  // Alert if requests waiting
  if (pool.waitingCount > 0) {
    console.error('🚨 Connection pool exhausted! ' + pool.waitingCount + ' requests waiting');
  }
}, 10000);

// MongoDB: Monitor connection pool
client.on('connectionPoolCreated', (event) => {
  console.log('✅ Connection pool created:', event.options.maxPoolSize);
});

client.on('connectionCheckOutStarted', (event) => {
  console.log('🔄 Connection checkout started');
});

client.on('connectionCheckOutFailed', (event) => {
  console.error('❌ Connection checkout failed:', event.reason);
});
```

---

#### 4. Cache Hit Ratio

```sql
-- PostgreSQL: Should be > 99%
SELECT 
  sum(heap_blks_read) as heap_read,
  sum(heap_blks_hit) as heap_hit,
  (sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read))) as cache_hit_ratio
FROM pg_statio_user_tables;

-- If < 0.99, increase shared_buffers
ALTER SYSTEM SET shared_buffers = '4GB';  -- 25% of RAM
-- Restart required
```

```javascript
// Redis: Monitor hit rate
let hits = 0, misses = 0;

setInterval(async () => {
  const info = await redis.info('stats');
  const stats = info.split('\n').reduce((acc, line) => {
    const [key, value] = line.split(':');
    if (key && value) acc[key.trim()] = value.trim();
    return acc;
  }, {});
  
  const hitRate = parseInt(stats.keyspace_hits) / 
    (parseInt(stats.keyspace_hits) + parseInt(stats.keyspace_misses));
  
  console.log(`Cache hit rate: ${(hitRate * 100).toFixed(2)}%`);
  // Aim for > 90%
}, 60000);
```

---

#### 5. Replication Lag

```sql
-- PostgreSQL: Check replication lag (on primary)
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

```javascript
// MongoDB: Check replication lag
rs.printReplicationInfo();
rs.printSlaveReplicationInfo();

// Programmatically
db.adminCommand({ replSetGetStatus: 1 });
```

---

#### 6. Disk Usage

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

#### 1. Prometheus + Grafana

```yaml
# docker-compose.yml
version: '3'
services:
  postgres_exporter:
    image: prometheuscommunity/postgres-exporter
    environment:
      DATA_SOURCE_NAME: "postgresql://monitoring_user:password@postgres:5432/mydb?sslmode=disable"
    ports:
      - "9187:9187"
  
  mongodb_exporter:
    image: percona/mongodb_exporter
    command:
      - '--mongodb.uri=mongodb://mongodb:27017'
    ports:
      - "9216:9216"
  
  redis_exporter:
    image: oliver006/redis_exporter
    environment:
      REDIS_ADDR: "redis:6379"
    ports:
      - "9121:9121"
  
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
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
```

#### 2. Application Performance Monitoring

```javascript
// Example: Custom metrics collection
const metrics = {
  queryCount: 0,
  queryTime: 0,
  errors: 0
};

async function monitoredQuery(query, params) {
  const start = Date.now();
  metrics.queryCount++;
  
  try {
    const result = await pool.query(query, params);
    metrics.queryTime += Date.now() - start;
    return result;
  } catch (error) {
    metrics.errors++;
    throw error;
  }
}

// Expose metrics endpoint
app.get('/metrics', (req, res) => {
  res.json({
    queries: {
      total: metrics.queryCount,
      avgTime: metrics.queryTime / metrics.queryCount,
      errors: metrics.errors,
      errorRate: (metrics.errors / metrics.queryCount * 100).toFixed(2) + '%'
    },
    pool: {
      total: pool.totalCount,
      idle: pool.idleCount,
      waiting: pool.waitingCount
    }
  });
});
```

#### 3. pgBadger (PostgreSQL Log Analyzer)

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

---

## Backup & Recovery

### SQL Backup Strategies

#### 1. Logical Backups (SQL Dump)

**PostgreSQL (pg_dump):**

```bash
# Full database backup
pg_dump -U postgres -d mydb -F c -f /backups/mydb_$(date +%Y%m%d).dump

# Options:
# -F c: Custom format (compressed, allows partial restore)
# -F p: Plain SQL format (readable, can edit)
# -F t: Tar format

# Backup specific tables
pg_dump -U postgres -d mydb -t users -t orders -f tables_backup.sql

# Backup schema only (no data)
pg_dump -U postgres -d mydb --schema-only -f schema.sql

# Backup data only (no schema)
pg_dump -U postgres -d mydb --data-only -f data.sql

# Backup with inserts (compatible with any database)
pg_dump -U postgres -d mydb --inserts -f mydb_inserts.sql

# Restore
pg_restore -U postgres -d mydb /backups/mydb_20241229.dump

# Restore specific table
pg_restore -U postgres -d mydb -t users /backups/mydb_20241229.dump

# Restore to different database
createdb mydb_restored
pg_restore -U postgres -d mydb_restored /backups/mydb_20241229.dump
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
# PostgreSQL: Binary copy of entire cluster
pg_basebackup -D /backups/basebackup \
  -F tar \
  -z \
  -P \
  -U replication_user \
  -h primary.db.example.com

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

```bash
# postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /archive/%f'  # Or use s3 sync

# Point-in-time recovery (PITR)
# Restore to specific timestamp
pg_restore --target-time='2024-12-29 10:30:00' /backups/basebackup
```

---

### MongoDB Backup Strategies

```bash
# 1. mongodump (logical backup)
mongodump --uri="mongodb://localhost:27017/mydb" \
  --out=/backups/mongodb_$(date +%Y%m%d)

# Backup specific collections
mongodump --db=mydb --collection=users --out=/backups/

# Compressed backup
mongodump --archive=/backups/mydb.archive --gzip

# Restore
mongorestore --uri="mongodb://localhost:27017/mydb" \
  /backups/mongodb_20241229/

# Restore from archive
mongorestore --archive=/backups/mydb.archive --gzip

# 2. File system snapshot (physical backup)
# Requires replica set or sharded cluster
# Use MongoDB Cloud Backup or filesystem snapshots

# 3. Continuous backup (MongoDB Atlas)
# Automatic continuous backups with point-in-time restore
```

---

### Backup Best Practices

```bash
# 1. Automate backups (cron job)
# /etc/cron.d/postgres-backup
0 2 * * * postgres pg_dump -U postgres -d mydb -F c -f /backups/mydb_$(date +\%Y\%m\%d).dump

# 2. Test restores regularly (monthly)
createdb test_restore
pg_restore -U postgres -d test_restore /backups/mydb_latest.dump
# Verify data integrity
psql -U postgres -d test_restore -c "SELECT COUNT(*) FROM users;"
dropdb test_restore

# 3. Store backups off-site (S3, Google Cloud Storage)
aws s3 sync /backups/ s3://my-db-backups/ --delete

# 4. Encrypt backups
pg_dump -U postgres -d mydb | \
  gpg --encrypt --recipient admin@example.com > \
  /backups/mydb_encrypted.dump.gpg

# Decrypt
gpg --decrypt /backups/mydb_encrypted.dump.gpg | \
  pg_restore -U postgres -d mydb

# 5. Retention policy
# Keep: Daily for 7 days, Weekly for 4 weeks, Monthly for 12 months
find /backups -name "*.dump" -mtime +7 -delete  # Delete > 7 days

# 6. Monitor backup success
#!/bin/bash
BACKUP_FILE="/backups/mydb_$(date +%Y%m%d).dump"
pg_dump -U postgres -d mydb -F c -f $BACKUP_FILE

if [ $? -eq 0 ] && [ -f $BACKUP_FILE ]; then
  echo "✅ Backup successful: $BACKUP_FILE"
  SIZE=$(du -h $BACKUP_FILE | cut -f1)
  echo "Size: $SIZE"
else
  echo "❌ Backup failed!"
  mail -s "Backup Failed" admin@example.com <<< "Database backup failed at $(date)"
  exit 1
fi

# 7. Verify backup integrity
pg_restore --list $BACKUP_FILE > /dev/null
if [ $? -eq 0 ]; then
  echo "✅ Backup file is valid"
else
  echo "❌ Backup file is corrupted!"
  exit 1
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

## Database Testing

### 1. Unit Testing (Mock Database)

```javascript
const { jest } = require('@jest/globals');

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
  
  it('should handle database errors', async () => {
    // Arrange
    mockDb.query.mockRejectedValue(new Error('Connection failed'));
    const userService = new UserService(mockDb);
    
    // Act & Assert
    await expect(userService.createUser({ name: 'Test', email: 'test@example.com' }))
      .rejects.toThrow('Connection failed');
  });
});
```

---

### 2. Integration Testing (Test Database)

```javascript
const { Pool } = require('pg');

describe('Database Integration', () => {
  let pool;
  
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
  
  it('should handle transactions correctly', async () => {
    const client = await pool.connect();
    
    try {
      await client.query('BEGIN');
      
      await client.query(
        'INSERT INTO users (name, email) VALUES ($1, $2)',
        ['User 1', 'user1@example.com']
      );
      
      await client.query(
        'INSERT INTO users (name, email) VALUES ($1, $2)',
        ['User 2', 'user2@example.com']
      );
      
      await client.query('COMMIT');
      
      const result = await pool.query('SELECT COUNT(*) FROM users');
      expect(parseInt(result.rows[0].count)).toBe(2);
    } catch (error) {
      await client.query('ROLLBACK');
      throw error;
    } finally {
      client.release();
    }
  });
});
```

---

### 3. Load Testing

```bash
# PostgreSQL: pgbench
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

# Custom SQL script
# bench.sql:
# \set id random(1, 100000)
# SELECT * FROM users WHERE id = :id;

pgbench -c 10 -j 2 -t 1000 -f bench.sql testdb
```

---

## Memory Tips & Mnemonics

### 1. ACID: "**A Car Is Durable**"
- **A** - **A**tomicity (All or nothing - bank transfer)
- **C** - **C**onsistency (Rules followed - no negative balance)
- **I** - **I**solation (No interference - can't buy last item twice)
- **D** - **D**urability (Survives crashes - committed data permanent)

### 2. SQL Index Types: "**B**asically **H**ashing **G**ives **P**erformance"
- **B** - **B**-Tree (default, range queries)
- **H** - **H**ash (exact matches only)
- **G** - **G**IN (arrays, JSON, full-text)
- **P** - **P**artial (subset of data)

### 3. Scaling: "**V**ery **H**igh **S**hould **R**eplicate"
- **V** - **V**ertical (scale up - more power)
- **H** - **H**orizontal (scale out - more servers)
- **S** - **S**harding (write scaling - split data)
- **R** - **R**eplicas (read scaling - copy data)

### 4. Query Optimization: "**ISELW**ind" (I Select Wind)
- **I** - **I**ndexes (create on frequently queried columns)
- **S** - **S**elect specific columns (not SELECT *)
- **E** - **E**XPLAIN ANALYZE (understand query plan)
- **L** - **L**IMIT (paginate results)
- **W** - **W**HERE before JOIN (filter early)

### 5. SQL vs NoSQL: "**SAFE**" vs "**RASH**"

**SQL (SAFE):**
- **S** - **S**trong consistency (ACID)
- **A** - **A**dvanced queries (JOINs, aggregations)
- **F** - **F**oreign keys (relationships)
- **E** - **E**nterprise (banking, compliance)

**NoSQL (RASH):**
- **R** - **R**apid development (flexible schema)
- **A** - **A**ggregations (horizontal scaling)
- **S** - **S**imple queries (key-value lookups)
- **H** - **H**uge scale (millions of users)

### 6. CAP Theorem: "**CAP**tain's Choice"
You can only have **2 of 3**:
- **C** - **C**onsistency
- **A** - **A**vailability
- **P** - **P**artition tolerance

**CA** - Traditional SQL (single server)
**CP** - MongoDB (consistent but may be unavailable during partition)
**AP** - Cassandra (available but eventually consistent)

---

## Interview Questions & Answers

### Q1: Explain the difference between SQL and NoSQL databases. When would you use each?

**Answer:**

"SQL and NoSQL databases differ fundamentally in structure, scaling, and use cases.

**SQL (Relational):**
- **Structure:** Tables with fixed schema, rows and columns
- **Relationships:** Foreign keys, JOINs
- **Consistency:** ACID transactions
- **Scaling:** Vertical (scale up)
- **Query Language:** SQL (standardized)

**NoSQL (Non-Relational):**
- **Structure:** Flexible - documents, key-value, graphs, column-family
- **Relationships:** Embedded or referenced
- **Consistency:** Eventual consistency (BASE)
- **Scaling:** Horizontal (scale out)
- **Query Language:** Varies by type

**When to use SQL:**
```
✅ Banking systems (need ACID)
✅ E-commerce orders and payments
✅ Complex relationships (many JOINs)
✅ Fixed, well-defined schema
✅ Strong consistency required
✅ Regulatory compliance

Example: Order processing
- PostgreSQL ensures payment and inventory update atomically
- Can't have money deducted without creating order
```

**When to use NoSQL:**
```
✅ Flexible schema (product catalogs with varying attributes)
✅ High write throughput (IoT sensors, logs)
✅ Horizontal scaling needed (millions of users)
✅ Eventual consistency acceptable
✅ Hierarchical data (comments with nested replies)

Example: Blog platform
- MongoDB stores posts with embedded comments
- No JOINs needed for displaying post with comments
- Easy to add new fields without migration
```

**Real-world hybrid:**
```javascript
// E-commerce uses both
PostgreSQL: Orders, payments, inventory (ACID critical)
Redis: Shopping cart, sessions (speed)
MongoDB: Product reviews, user activity logs (flexible)
Elasticsearch: Product search (full-text)
```

**Decision criteria:**
1. Data structure: Fixed → SQL, Evolving → NoSQL
2. Transactions: Critical → SQL, Not critical → NoSQL
3. Scale: < 1TB → SQL, > 1TB → NoSQL (usually)
4. Relationships: Complex → SQL, Simple → NoSQL
5. Team expertise: SQL familiar → SQL, else evaluate"

---

### Q2: What is database normalization? Explain with an example.

**Answer:**

"Normalization organizes data to reduce redundancy and improve integrity.

**First Normal Form (1NF): Atomic values**
```sql
❌ Violates 1NF (comma-separated values)
orders:
┌────┬──────────┬─────────────────────────┐
│ id │ customer │ items                   │
├────┼──────────┼─────────────────────────┤
│ 1  │ Alice    │ Laptop $1000, Mouse $20 │
└────┴──────────┴─────────────────────────┘

✅ Follows 1NF (atomic values)
orders:
┌────┬──────────┐
│ id │ customer │
├────┼──────────┤
│ 1  │ Alice    │
└────┴──────────┘

order_items:
┌────┬──────────┬────────┬───────┐
│ id │ order_id │ item   │ price │
├────┼──────────┼────────┼───────┤
│ 1  │ 1        │ Laptop │ 1000  │
│ 2  │ 1        │ Mouse  │ 20    │
└────┴──────────┴────────┴───────┘
```

**Second Normal Form (2NF): Remove partial dependencies**
```sql
✅ 2NF: Separate customers
customers:
┌────┬───────┬────────────────────┐
│ id │ name  │ email              │
├────┼───────┼────────────────────┤
│ 1  │ Alice │ alice@example.com  │
└────┴───────┴────────────────────┘

orders:
┌────┬─────────────┬────────┐
│ id │ customer_id │ total  │
├────┼─────────────┼────────┤
│ 1  │ 1           │ 1020   │
└────┴─────────────┴────────┘
```

**Third Normal Form (3NF): Remove transitive dependencies**
```sql
✅ 3NF: Separate products
products:
┌────┬────────┬───────┐
│ id │ name   │ price │
├────┼────────┼───────┤
│101 │ Laptop │ 1000  │
│102 │ Mouse  │ 20    │
└────┴────────┴───────┘

order_items:
┌────┬──────────┬────────────┬──────────┐
│ id │ order_id │ product_id │ quantity │
├────┼──────────┼────────────┼──────────┤
│ 1  │ 1        │ 101        │ 1        │
│ 2  │ 1        │ 102        │ 2        │
└────┴──────────┴────────────┴──────────┘
```

**Benefits:**
- No data redundancy
- Easy updates (change price once in products table)
- Data integrity maintained

**Trade-off:**
- More JOINs required
- Can be slower for reads

**When to denormalize:**
- Read-heavy systems
- Need performance
- Data doesn't change often

```sql
-- For read-heavy systems
-- Store price at time of order
order_items:
┌────┬──────────┬────────────┬──────────┬────────────────┐
│ id │ order_id │ product_id │ quantity │ price_at_order │
├────┼──────────┼────────────┼──────────┼────────────────┤
│ 1  │ 1        │ 101        │ 1        │ 1000           │
└────┴──────────┴────────────┴──────────┴────────────────┘

-- Now if product price changes, order history stays correct
```

**Real example:**
```
E-commerce: Normalize user, product, order data
Analytics: Denormalize for reporting (star schema)
```"

---

### Q3: Explain database indexing. How do they work, What are the types and when to use them?

**Answer:**

"An index is a data structure that speeds up data retrieval at the cost of additional writes and storage.

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

```
Without index: O(n) - sequential scan of all rows
With index: O(log n) - binary search

1,000,000 rows:
- Sequential: ~2000ms
- Index: ~50ms
- 40x faster!
```

**Types of indexes:**
1. **B-Tree** (default): Range queries, sorting
2. **Hash**: Exact matches only
3. **GIN**: Arrays, JSON, full-text search
4. **Partial**: Subset of data (WHERE clause)

**1. B-Tree (default):**
```sql
CREATE INDEX idx_products_price ON products(price);

-- Use cases:
✅ Equality: WHERE price = 99.99
✅ Range: WHERE price > 50
✅ Sorting: ORDER BY price
```

**2. Hash:**
```sql
CREATE INDEX idx_users_email USING HASH (email);

-- Use cases:
✅ Exact match only: WHERE email = '...'
❌ Can't do: WHERE email LIKE '%example.com'
```

**3. GIN (Generalized Inverted):**
```sql
-- For arrays, JSON, full-text search
CREATE INDEX idx_products_tags USING GIN (tags);

-- Use cases:
✅ Arrays: WHERE tags @> ARRAY['electronics']
✅ JSON: WHERE attributes @> '{"color": "red"}'
✅ Full-text: WHERE search_vector @@ to_tsquery('laptop')
```

**4. Composite (multiple columns):**
```sql
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

✅ Uses index: WHERE user_id = 1 AND status = 'pending'
✅ Uses index: WHERE user_id = 1 (leftmost prefix)
❌ Doesn't use: WHERE status = 'pending' (skips first column)

-- Column order matters: Most selective first
```

**5. Partial:**
```sql
CREATE INDEX idx_active_products ON products(name) 
WHERE is_active = true;

-- Benefits: Smaller index, faster for specific queries
```

**6. Covering:**
```sql
CREATE INDEX idx_users_email_covering ON users(email) 
INCLUDE (name, city);

-- All data in index (no table lookup needed)
```

**When to create:**
```sql
✅ Column in WHERE clause frequently
✅ Column in JOIN conditions
✅ Column in ORDER BY
✅ High cardinality (many unique values)
✅ Large table (1000+ rows)
```

**When NOT to create:**
```sql
❌ Small table (< 1000 rows)
❌ Low cardinality (gender, status)
❌ Write-heavy table
❌ Column rarely queried
```

**MongoDB indexes:**
```javascript
// Single field
db.users.createIndex({ email: 1 });

// Compound
db.posts.createIndex({ author_id: 1, status: 1 });

// Text (full-text search)
db.posts.createIndex({ title: "text", content: "text" });

// Geospatial
db.restaurants.createIndex({ location: "2dsphere" });

// TTL (auto-delete)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 });
```

**Trade-offs:**
- ✅ Faster reads
- ❌ Slower writes (must update index)
- ❌ Storage space
- ❌ Maintenance overhead"

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

### Q5: What is the CAP theorem and how does it apply to databases?

**Answer:**

"CAP theorem states that in a distributed system, you can only guarantee 2 of 3:

**C - Consistency:** All nodes see same data at same time
**A - Availability:** System always responds (even if some nodes down)
**P - Partition Tolerance:** System works despite network failures

**Why can't have all 3:**
```
Scenario: Network partition splits database
┌─────────┐  ╳  ┌─────────┐
│ Node 1  │     │ Node 2  │
└─────────┘     └─────────┘

If you prioritize Consistency (C):
- Must reject writes until partition heals
- Sacrifices Availability (A)
- Result: CP system

If you prioritize Availability (A):
- Both nodes accept writes
- Data diverges temporarily
- Sacrifices Consistency (C)
- Result: AP system
```

**Database categorization:**

**CA (Consistency + Availability):**
```
Traditional SQL on single server
- PostgreSQL, MySQL (single instance)
- Problem: Not partition-tolerant (if server down, system down)
- Reality: Single-server isn't truly distributed, so P doesn't apply
```

**CP (Consistency + Partition Tolerance):**
```
MongoDB (with strong consistency)
- Guarantees consistent reads
- During partition: Reject writes to minority nodes
- Trade-off: Some nodes unavailable during split

HBase, Redis Sentinel
- Strong consistency
- May be unavailable during network issues
```

**AP (Availability + Partition Tolerance):**
```
Cassandra, DynamoDB, CouchDB
- Always available for reads/writes
- During partition: Accept writes on all nodes
- Trade-off: Eventual consistency (data may be stale)
- Resolves conflicts later

Example:
User A writes to Node 1: balance = $100
User B reads from Node 2: balance = $50 (stale!)
Eventually: Both nodes sync to $100
```

**Real-world decisions:**

**Banking (CP):**
```
Use PostgreSQL or MongoDB with strong consistency
- Can't show incorrect balance
- Better to be unavailable than inconsistent
- ACID transactions critical
```

**Social Media (AP):**
```
Use Cassandra or DynamoDB
- Like count can be slightly off
- Better to be available than consistent
- User can always post/read

Example: Facebook
- Post count might be 99 or 101 temporarily
- Eventually becomes 100
- Nobody cares about exact count at every moment
```

**E-commerce hybrid:**
```
CP: Orders, payments (must be consistent)
- PostgreSQL with synchronous replication
- Won't process order if can't guarantee consistency

AP: Product views, recommendations (can be stale)
- Cassandra for metrics
- View count being off by a few is fine
```

**Key insight:**
```
In distributed systems, network partitions WILL happen
So you must choose between C and A
Most modern systems: Choose per use case
```"

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

### Q7: Explain database transactions and isolation levels.

**Answer:**

"Transactions group operations that must succeed or fail together, with 4 isolation levels controlling how they interact.

**ACID Transaction example:**
```sql
BEGIN TRANSACTION;
  -- Atomicity: All or nothing
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
  
  -- If ANY error occurs, ROLLBACK everything
COMMIT;
```

**Isolation Levels (strictest to loosest):**

**1. Read Uncommitted:**
```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

Problem: Dirty reads
Transaction A: UPDATE accounts SET balance = 200 WHERE id = 1;
-- NOT committed yet
Transaction B: SELECT balance FROM accounts WHERE id = 1;
-- Returns: 200 (uncommitted!)
Transaction A: ROLLBACK;
-- B read data that never existed!

Use: Almost never (too dangerous)
```

**2. Read Committed (DEFAULT):**
```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

Prevents: Dirty reads
Still allows: Non-repeatable reads

Example:
BEGIN;
  SELECT balance FROM accounts WHERE id = 1;  -- Returns: 100
  -- Another transaction updates and commits: balance = 200
  SELECT balance FROM accounts WHERE id = 1;  -- Returns: 200 (changed!)
COMMIT;

Use: Most applications (good balance)
```

**3. Repeatable Read:**
```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

Prevents: Dirty reads, non-repeatable reads
Still allows: Phantom reads

Example:
BEGIN;
  SELECT balance FROM accounts WHERE id = 1;  -- Returns: 100
  -- Another transaction updates and commits: balance = 200
  SELECT balance FROM accounts WHERE id = 1;  -- Still returns: 100 (consistent!)
COMMIT;

Use: Financial reports, data consistency critical
```

**4. Serializable (STRICTEST):**
```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

Prevents: All anomalies
Acts as if: Transactions executed serially (one at a time)

Example: Booking last seat
Transaction A:
BEGIN;
  SELECT COUNT(*) FROM seats WHERE available = true;  -- Returns: 1
  -- Locks query range
  UPDATE seats SET available = false WHERE id = 1;
COMMIT;

Transaction B (concurrent):
BEGIN;
  SELECT COUNT(*) FROM seats WHERE available = true;  -- WAITS for A
COMMIT;

Use: Financial transactions, inventory management
```

**Comparison table:**

| Level | Dirty Read | Non-repeatable | Phantom | Speed |
|-------|-----------|----------------|---------|-------|
| Read Uncommitted | ❌ | ❌ | ❌ | ⚡⚡⚡ |
| Read Committed | ✅ | ❌ | ❌ | ⚡⚡ |
| Repeatable Read | ✅ | ✅ | ❌ | ⚡ |
| Serializable | ✅ | ✅ | ✅ | 🐌 |

**Real-world example:**

**E-commerce order:**
```sql
-- Use SERIALIZABLE for inventory
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
  
  -- Check stock
  SELECT stock FROM products WHERE id = 101 FOR UPDATE;
  -- Locks row, prevents others from reading
  
  IF stock >= 1 THEN
    -- Decrease stock
    UPDATE products SET stock = stock - 1 WHERE id = 101;
    
    -- Create order
    INSERT INTO orders (user_id, product_id) VALUES (1, 101);
  ELSE
    RAISE EXCEPTION 'Out of stock';
  END IF;
  
COMMIT;
```

**MongoDB transactions:**
```javascript
const session = client.startSession();

try {
  await session.startTransaction({
    readConcern: { level: 'snapshot' },       // Like Serializable
    writeConcern: { w: 'majority' },
    readPreference: 'primary'
  });
  
  // Transfer money
  await db.collection('accounts').updateOne(
    { _id: account1Id },
    { $inc: { balance: -100 } },
    { session }
  );
  
  await db.collection('accounts').updateOne(
    { _id: account2Id },
    { $inc: { balance: 100 } },
    { session }
  );
  
  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
  throw error;
} finally {
  session.endSession();
}
```

**Key points:**
- Higher isolation = More consistent but slower
- Choose based on use case
- Most apps: Read Committed
- Financial: Serializable"

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

### Q9: How would you design a database for a social media platform?

**Answer:**

"I'll walk through a complete design process:

**Step 1: Requirements**
```
Features:
- User profiles
- Posts (text, images, videos)
- Followers/Following
- Likes and comments
- Feed (posts from followed users)
- Real-time notifications

Scale:
- 100M users
- 1B posts
- 10B interactions (likes, comments)
- High read-to-write ratio (90:10)
```

**Step 2: Choose databases (hybrid approach)**
```
PostgreSQL: Users, authentication, financial data
MongoDB: Posts, comments (flexible content)
Redis: Feed cache, online users, trending
Neo4j: Friend recommendations (graph relationships)
Elasticsearch: Search users/posts
```

**Step 3: Schema design**

**PostgreSQL - Users:**
```sql
CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  full_name VARCHAR(100),
  bio TEXT,
  avatar_url VARCHAR(500),
  follower_count INTEGER DEFAULT 0,
  following_count INTEGER DEFAULT 0,
  verified BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);

CREATE TABLE follows (
  id BIGSERIAL PRIMARY KEY,
  follower_id BIGINT REFERENCES users(id),
  following_id BIGINT REFERENCES users(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(follower_id, following_id)
);

CREATE INDEX idx_follows_follower ON follows(follower_id);
CREATE INDEX idx_follows_following ON follows(following_id);
```

**MongoDB - Posts:**
```javascript
{
  _id: ObjectId(),
  author: {
    id: ObjectId("user_id"),
    username: "foyez",
    avatar: "https://..."
  },
  type: "text",  // text, image, video
  content: "Just learned about database design!",
  media: [
    {
      type: "image",
      url: "https://...",
      thumbnail: "https://..."
    }
  ],
  stats: {
    likes: 1250,
    comments: 48,
    shares: 23,
    views: 5420
  },
  hashtags: ["database", "learning"],
  mentions: ["@alice", "@bob"],
  visibility: "public",  // public, followers, private
  createdAt: ISODate(),
  updatedAt: ISODate()
}

// Indexes
db.posts.createIndex({ "author.id": 1, createdAt: -1 });
db.posts.createIndex({ hashtags: 1 });
db.posts.createIndex({ createdAt: -1 });
```

**MongoDB - Comments:**
```javascript
{
  _id: ObjectId(),
  postId: ObjectId("post_id"),
  author: {
    id: ObjectId("user_id"),
    username: "alice",
    avatar: "https://..."
  },
  content: "Great post!",
  parentId: null,  // For threaded comments
  likesCount: 15,
  createdAt: ISODate()
}

db.comments.createIndex({ postId: 1, createdAt: -1 });
```

**Redis - Feed cache:**
```javascript
// User's feed (list of post IDs)
Key: `feed:user:${userId}`
Value: [postId1, postId2, postId3, ...]
TTL: 300 seconds

// Trending posts
Key: `trending:posts`
Value: Sorted set with scores (engagement)
TTL: 60 seconds

// Online users
Key: `online:users`
Value: Set of user IDs
```

**Neo4j - Social graph:**
```cypher
// Nodes and relationships
(user:User)-[:FOLLOWS]->(other:User)
(user)-[:POSTED]->(post:Post)
(user)-[:LIKES]->(post)

// Friend suggestions
MATCH (me:User {id: userId})-[:FOLLOWS]->()-[:FOLLOWS]->(suggestion)
WHERE NOT (me)-[:FOLLOWS]->(suggestion) AND suggestion <> me
RETURN suggestion.username, COUNT(*) as mutual_friends
ORDER BY mutual_friends DESC
LIMIT 10
```

**Step 4: Common queries**

**Get user feed:**
```javascript
async function getUserFeed(userId, page = 1, limit = 20) {
  // 1. Check cache
  const cacheKey = `feed:${userId}:${page}`;
  let feed = await redis.get(cacheKey);
  
  if (feed) {
    return JSON.parse(feed);
  }
  
  // 2. Get followed users from PostgreSQL
  const following = await pg.query(
    'SELECT following_id FROM follows WHERE follower_id = $1',
    [userId]
  );
  
  const followingIds = following.rows.map(r => r.following_id);
  
  // 3. Get posts from MongoDB
  const posts = await mongo.collection('posts').find({
    'author.id': { $in: followingIds }
  })
  .sort({ createdAt: -1 })
  .skip((page - 1) * limit)
  .limit(limit)
  .toArray();
  
  // 4. Cache result
  await redis.setex(cacheKey, 300, JSON.stringify(posts));
  
  return posts;
}
```

**Step 5: Scaling strategy**

**Read scaling:**
```
- PostgreSQL: 1 primary + 3 read replicas
- MongoDB: Replica set (1 primary + 2 secondaries)
- Redis: Cluster mode (3 master + 3 replica)
```

**Write scaling:**
```
- MongoDB: Shard by user_id
- Shard 0: Users 0-33M
- Shard 1: Users 33M-66M
- Shard 2: Users 66M-100M
```

**Caching:**
```
- Feed cache (Redis): 5 minutes
- User profile cache: 1 hour
- Trending cache: 1 minute
- CDN: Images, videos (permanent)
```

**Step 6: Performance optimizations**

**Fan-out on write:**
```javascript
// When user posts, push to followers' feeds
async function createPost(userId, postData) {
  // 1. Save post to MongoDB
  const post = await mongo.collection('posts').insertOne(postData);
  
  // 2. Get followers
  const followers = await pg.query(
    'SELECT follower_id FROM follows WHERE following_id = $1',
    [userId]
  );
  
  // 3. Fan-out to followers' feeds (async)
  for (const follower of followers.rows) {
    await redis.lpush(`feed:${follower.follower_id}`, post.insertedId);
    await redis.ltrim(`feed:${follower.follower_id}`, 0, 999);  // Keep 1000 posts
  }
}
```

**This design handles:**
- 100M users
- High read throughput (90% reads → replicas)
- Real-time feeds (Redis cache)
- Complex relationships (Neo4j)
- Flexible content (MongoDB)
- Strong consistency where needed (PostgreSQL)"

---

### Q10: Explain sharding and when to use it.

**Answer:**

"Sharding splits data across multiple databases to scale writes horizontally.

**How it works:**
```
Single Database:          Sharded:
┌──────────────┐         ┌─────────┐ ┌─────────┐ ┌─────────┐
│ All data     │    →    │ Shard 0 │ │ Shard 1 │ │ Shard 2 │
│ 10M users    │         │ 0-3.3M  │ │3.3M-6.6M│ │6.6M-10M │
└──────────────┘         └─────────┘ └─────────┘ └─────────┘
  1 server                  3 servers
  Write limit               3x write capacity
```

**Sharding strategies:**

**1. Hash-based (modulo):**
```javascript
function getShard(userId) {
  return userId % 3;  // 3 shards
}

const shards = [
  new Pool({ host: 'shard0.db.example.com' }),
  new Pool({ host: 'shard1.db.example.com' }),
  new Pool({ host: 'shard2.db.example.com' })
];

async function createOrder(userId, orderData) {
  const shardIndex = getShard(userId);
  return await shards[shardIndex].query('INSERT INTO orders ...');
}

// ✅ Pros: Even distribution
// ❌ Cons: Hard to add shards (changes hash)
```

**2. Range-based:**
```javascript
function getShard(userId) {
  if (userId <= 3333333) return 0;
  if (userId <= 6666666) return 1;
  return 2;
}

// ✅ Pros: Easy to add shards
// ❌ Cons: Uneven if new users concentrated
```

**3. Geographic:**
```javascript
function getShard(country) {
  const regionMap = {
    'BD': 'asia',
    'US': 'americas',
    'DE': 'europe'
  };
  return shards[regionMap[country]];
}

// ✅ Pros: Low latency for users
// ❌ Cons: Uneven distribution by region
```

**4. Consistent hashing:**
```javascript
function consistentHash(userId) {
  const hash = crypto.createHash('md5')
    .update(userId.toString())
    .digest('hex');
  const hashInt = parseInt(hash.substring(0, 8), 16);
  return hashInt % shards.length;
}

// ✅ Pros: Minimal data movement when adding shards
```

**MongoDB automatic sharding:**
```javascript
// Enable sharding
sh.enableSharding("mydb");

// Shard collection
sh.shardCollection("mydb.orders", { user_id: 1 });

// MongoDB handles everything automatically!
// Application code doesn't change
db.orders.insertOne({ user_id: 1000000, total: 150 });
// Automatically routed to correct shard
```

**When to use sharding:**
```
✅ Write-heavy workloads (millions of writes/sec)
✅ Single database hitting limits
✅ Need horizontal scaling
✅ Data naturally partitionable (by user, region)

Examples:
- Social media (shard by user_id)
- Multi-tenant SaaS (shard by tenant_id)
- Time-series data (shard by date)
```

**Challenges:**

**1. Cross-shard queries:**
```javascript
// ❌ Can't JOIN across shards
async function getTopUsers() {
  // Must query each shard
  const results = await Promise.all([
    shards[0].query('SELECT * FROM users ORDER BY score DESC LIMIT 10'),
    shards[1].query('SELECT * FROM users ORDER BY score DESC LIMIT 10'),
    shards[2].query('SELECT * FROM users ORDER BY score DESC LIMIT 10')
  ]);
  
  // Merge and sort in application
  const allUsers = [...results[0], ...results[1], ...results[2]];
  return allUsers.sort((a, b) => b.score - a.score).slice(0, 10);
}
```

**2. Distributed transactions:**
```javascript
// Can't have ACID across shards (without 2PC)
// User A on Shard 0, User B on Shard 1

// ❌ Can't do atomically:
await shards[0].query('UPDATE accounts SET balance = balance - 100 WHERE id = A');
await shards[1].query('UPDATE accounts SET balance = balance + 100 WHERE id = B');

// Solution: Saga pattern or avoid cross-shard transactions
```

**3. Rebalancing:**
```javascript
// Adding 4th shard changes distribution
// userId % 3 !== userId % 4
// Must migrate data

// MongoDB handles this automatically
sh.addShard("mongodb://new-shard:27017");
// Balancer migrates data in background
```

**4. Hotspots:**
```javascript
// Celebrity user creates millions of posts
// Their shard becomes overloaded

// Solution: Compound shard key
sh.shardCollection("mydb.posts", { author_id: 1, created_at: 1 });
// Distributes across time ranges

// Or: Further shard hot users
if (isCelebrity(userId)) {
  return getCelebrityShard(userId);
}
```

**Best practices:**
```
1. Choose shard key carefully:
   - High cardinality
   - Even distribution
   - Query pattern aligned

2. Plan for rebalancing
3. Monitor shard distribution
4. Use consistent hashing
5. Consider compound shard keys
```"

---

### Q11: What is the difference between MongoDB's embedding and referencing?

**Answer:**

"Embedding and referencing are two ways to model relationships in MongoDB.

**Embedding (Denormalization):**
```javascript
// All data in one document
{
  _id: ObjectId("user_id"),
  name: "Foyez",
  email: "foyez@example.com",
  
  // Embedded addresses
  addresses: [
    { type: "home", street: "123 Main St", city: "Cumilla" },
    { type: "work", street: "456 Office Rd", city: "Dhaka" }
  ],
  
  // Embedded orders
  orders: [
    {
      id: ObjectId("order1"),
      total: 150.00,
      items: [
        { product: "Laptop", price: 1000, qty: 1 }
      ]
    }
  ]
}

// ✅ Single query gets everything
db.users.findOne({ _id: userId });
// No JOINs needed!

// ✅ Atomic updates
db.users.updateOne(
  { _id: userId },
  { $set: { "addresses.0.city": "Dhaka" } }
);
```

**Referencing (Normalization):**
```javascript
// Users collection
{
  _id: ObjectId("user_id"),
  name: "Foyez",
  email: "foyez@example.com"
}

// Addresses collection (separate)
{
  _id: ObjectId("address1"),
  userId: ObjectId("user_id"),  // Reference
  type: "home",
  street: "123 Main St",
  city: "Cumilla"
}

// Orders collection (separate)
{
  _id: ObjectId("order1"),
  userId: ObjectId("user_id"),  // Reference
  total: 150.00
}

// Need $lookup to get everything
db.users.aggregate([
  { $match: { _id: userId } },
  {
    $lookup: {
      from: "addresses",
      localField: "_id",
      foreignField: "userId",
      as: "addresses"
    }
  },
  {
    $lookup: {
      from: "orders",
      localField: "_id",
      foreignField: "userId",
      as: "orders"
    }
  }
]);

// ✅ No data duplication
// ✅ Easy to update (one place)
```

**Decision matrix:**

| Factor | Embed | Reference |
|--------|-------|-----------|
| **Relationship** | 1-to-1, 1-to-few | 1-to-many, many-to-many |
| **Data size** | Small (<100 items) | Large (1000+) |
| **Access pattern** | Always together | Sometimes separate |
| **Update frequency** | Rare | Frequent |
| **Data duplication** | OK | Avoid |

**When to embed:**
```javascript
// 1-to-1: User profile
{
  _id: ObjectId("user_id"),
  name: "Foyez",
  profile: {  // Always accessed with user
    bio: "...",
    avatar: "...",
    location: { city: "Cumilla" }
  }
}

// 1-to-few: Product with reviews (< 20 reviews)
{
  _id: ObjectId("product_id"),
  name: "Laptop",
  price: 1299,
  reviews: [  // Few reviews, always shown with product
    { rating: 5, comment: "Great!", user: "Alice" },
    { rating: 4, comment: "Good", user: "Bob" }
  ]
}

// Hierarchical: Comment threads
{
  _id: ObjectId("comment_id"),
  text: "Great post!",
  replies: [  // Nested comments
    {
      text: "Thanks!",
      replies: [
        { text: "Welcome!" }
      ]
    }
  ]
}
```

**When to reference:**
```javascript
// 1-to-many: User with thousands of orders
// users collection
{ _id: ObjectId("user_id"), name: "Foyez" }

// orders collection (separate)
{ _id: ObjectId("order1"), userId: ObjectId("user_id"), total: 150 }
{ _id: ObjectId("order2"), userId: ObjectId("user_id"), total: 200 }
// Can't embed thousands of orders!

// Many-to-many: Students and courses
// students collection
{ _id: ObjectId("student1"), name: "Alice", courseIds: [ObjectId("c1"), ObjectId("c2")] }

// courses collection
{ _id: ObjectId("c1"), name: "Math", studentIds: [ObjectId("student1"), ObjectId("student2")] }
```

**Hybrid approach (best practice):**
```javascript
// Blog posts with author info
{
  _id: ObjectId("post_id"),
  title: "MongoDB Guide",
  content: "...",
  
  // Reference for queries
  authorId: ObjectId("user_id"),
  
  // Embed for display (cached)
  author: {
    username: "foyez",
    avatar: "https://..."
  },
  
  // Reference comments (too many to embed)
  commentsCount: 150
}

// Comments in separate collection
db.comments.find({ postId: ObjectId("post_id") });

// ✅ Fast display (author embedded)
// ✅ Can query by authorId
// ✅ Comments separate (scalable)

// Trade-off: If author changes username, must update all posts
// But username changes are rare, so acceptable
```

**Real-world example - E-commerce:**
```javascript
// Order (snapshot of data at purchase time)
{
  _id: ObjectId("order_id"),
  userId: ObjectId("user_id"),
  
  // Embed items (snapshot)
  items: [
    {
      productId: ObjectId("product1"),
      name: "Laptop",  // Embed name and price
      price: 1000,     // Price at time of order
      qty: 1
    }
  ],
  
  total: 1000,
  createdAt: new Date()
}

// Why embed? Order is historical record
// Even if product price changes to $1200, order stays $1000
```

**Key principle:**
```
Embed: Data accessed together
Reference: Data accessed independently or large datasets
```"

---

### Q12: How do you optimize a slow database query?

**Answer:**

"I follow a systematic approach:

**Step 1: Identify the problem with EXPLAIN**
```sql
-- PostgreSQL
EXPLAIN ANALYZE
SELECT * FROM orders 
WHERE user_id = 1 AND status = 'pending';

-- Look for:
❌ Seq Scan (sequential scan - bad!)
✅ Index Scan (using index - good!)

-- Before optimization:
Seq Scan on orders  (actual time=0.031..2.456 rows=5)
  Filter: ((user_id = 1) AND (status = 'pending'))
Execution Time: 2.489 ms

-- After adding index:
Index Scan using idx_orders_user_status
  (actual time=0.021..0.034 rows=5)
Execution Time: 0.056 ms

-- 44x faster!
```

**Step 2: Create appropriate indexes**
```sql
-- Single column index
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Composite index (better for multiple conditions)
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Covering index (includes all needed columns)
CREATE INDEX idx_orders_covering ON orders(user_id, status) 
INCLUDE (total, created_at);
```

**Step 3: Rewrite query**

**❌ Before:**
```sql
SELECT 
  u.name,
  (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) as order_count
FROM users u;
-- Correlated subquery runs for EACH user (N queries)
```

**✅ After:**
```sql
SELECT 
  u.name,
  COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;
-- Single query with JOIN
```

**Step 4: SELECT only needed columns**
```sql
-- ❌ Bad
SELECT * FROM products;  -- Returns all 20 columns

-- ✅ Good
SELECT id, name, price FROM products;  -- Only needed columns
```

**Step 5: Use LIMIT for pagination**
```sql
-- ❌ Bad
SELECT * FROM products ORDER BY created_at DESC;
-- Returns all 100,000 products

-- ✅ Good
SELECT * FROM products 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 0;

-- ✅ Better (keyset pagination for large offsets)
SELECT * FROM products 
WHERE created_at < '2024-01-15 10:30:00'
ORDER BY created_at DESC 
LIMIT 20;
```

**Step 6: Avoid functions on indexed columns**
```sql
-- ❌ Can't use index
SELECT * FROM users WHERE LOWER(email) = 'foyez@example.com';

-- ✅ Use index
SELECT * FROM users WHERE email = 'foyez@example.com';

-- ✅ Or create functional index
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
```

**Step 7: Use EXISTS instead of IN**
```sql
-- ❌ Slow with large subquery
SELECT * FROM products
WHERE id IN (SELECT product_id FROM order_items);

-- ✅ Fast (stops at first match)
SELECT * FROM products p
WHERE EXISTS (
  SELECT 1 FROM order_items oi 
  WHERE oi.product_id = p.id
);
```

**Step 8: Cache results**
```javascript
async function getProductCached(id) {
  // 1. Check cache
  const cached = await redis.get(`product:${id}`);
  if (cached) return JSON.parse(cached);
  
  // 2. Query database
  const product = await db.query('SELECT * FROM products WHERE id = $1', [id]);
  
  // 3. Cache for 5 minutes
  await redis.setex(`product:${id}`, 300, JSON.stringify(product));
  
  return product;
}
```

**MongoDB optimization:**
```javascript
// 1. Use explain()
db.posts.find({ status: "published" }).explain("executionStats");

// Look for:
// - "stage": "IXSCAN" (good - using index)
// - "stage": "COLLSCAN" (bad - collection scan)

// 2. Create index
db.posts.createIndex({ status: 1, publishedAt: -1 });

// 3. Project only needed fields
db.posts.find(
  { status: "published" },
  { title: 1, slug: 1, _id: 0 }
);

// 4. Use aggregation efficiently
db.posts.aggregate([
  { $match: { status: "published" } },  // FIRST - uses index
  { $sort: { publishedAt: -1 } },
  { $limit: 10 }
  // Expensive operations after filtering
]);
```

**Real example:**

**Before (5000ms):**
```sql
SELECT 
  p.*,
  (SELECT AVG(rating) FROM reviews WHERE product_id = p.id) as avg_rating,
  (SELECT COUNT(*) FROM reviews WHERE product_id = p.id) as review_count
FROM products p
WHERE p.category_id = 1
ORDER BY (SELECT AVG(rating) FROM reviews WHERE product_id = p.id) DESC;

-- Problems:
-- 1. Correlated subqueries (N queries)
-- 2. No indexes
-- 3. Sorting by subquery
```

**After (50ms):**
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

-- Result: 100x faster!
```

**Monitoring:**
```sql
-- Check slow queries (PostgreSQL)
SELECT 
  query,
  calls,
  mean_exec_time,
  total_exec_time
FROM pg_stat_statements
WHERE mean_exec_time > 100
ORDER BY mean_exec_time DESC;
```"

---

## Quick Reference

```
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