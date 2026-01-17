# Chapter 5: Scaling & High Availability
**Replication, Sharding, and Distributed Systems**

---

## Table of Contents
- [5.1 Vertical vs Horizontal Scaling](#51-vertical-vs-horizontal-scaling)
- [5.2 Replication (Read Scaling)](#52-replication-read-scaling)
  - [5.2.1 SQL Replication (PostgreSQL/MySQL)](#521-sql-replication-postgresqlmysql)
  - [5.2.2 MongoDB Replica Sets](#522-mongodb-replica-sets)
  - [5.2.3 Replication Lag & Consistency](#523-replication-lag--consistency)
- [5.3 Sharding (Write Scaling)](#53-sharding-write-scaling)
  - [5.3.1 Sharding Strategies](#531-sharding-strategies)
  - [5.3.2 MongoDB Sharding](#532-mongodb-sharding)
  - [5.3.3 Challenges & Solutions](#533-challenges--solutions)
- [5.4 Connection Pooling](#54-connection-pooling)
- [5.5 Load Balancing](#55-load-balancing)
- [5.6 CAP Theorem](#56-cap-theorem)

---

## 5.1 Vertical vs Horizontal Scaling

### The Scaling Problem

**Scenario:** Your application is growing!

```
Day 1:   100 users  → 1 server handles it fine
Day 30:  10,000 users  → Server struggling
Day 90:  100,000 users  → Server crashing
Day 180: 1,000,000 users  → Need to scale!
```

**Two Options:**

---

### Vertical Scaling (Scale Up)

**Definition:** Add more resources to existing server.

**Example:**
```
Before: 4 CPU cores, 8GB RAM, 100GB SSD
After:  32 CPU cores, 256GB RAM, 2TB SSD
```

**Analogy:** Upgrade your bicycle to a motorcycle.

**Advantages:**
- ✅ Simple (no code changes)
- ✅ No data distribution complexity
- ✅ ACID transactions work normally
- ✅ Consistent performance

**Disadvantages:**
- ❌ Physical limits (can't add infinite RAM)
- ❌ Expensive (exponential cost increase)
- ❌ Single point of failure
- ❌ Downtime during upgrades

**When to Use:**
- Small to medium applications
- Budget for powerful hardware
- Need strong consistency
- Simple architecture preferred

---

### Horizontal Scaling (Scale Out)

**Definition:** Add more servers to distribute load.

**Example:**
```
Before: 1 server (4 cores, 8GB RAM)
After:  10 servers (4 cores, 8GB RAM each)
```

**Analogy:** Instead of one big truck, use ten small trucks.

**Advantages:**
- ✅ Nearly unlimited scaling
- ✅ Cost-effective (commodity hardware)
- ✅ High availability (server failure OK)
- ✅ Linear cost scaling

**Disadvantages:**
- ❌ Complex architecture
- ❌ Data distribution challenges
- ❌ Potential consistency issues
- ❌ Requires code changes

**When to Use:**
- Large-scale applications
- Need high availability
- Unpredictable growth
- Cost-conscious

---

### Comparison Table

| Feature | Vertical Scaling | Horizontal Scaling |
|---------|-----------------|-------------------|
| **Hardware** | One powerful server | Many commodity servers |
| **Cost** | Expensive (exponential) | Cheaper (linear) |
| **Complexity** | Simple | Complex |
| **Limit** | Hardware ceiling | Nearly unlimited |
| **Availability** | Single point of failure | Redundant (HA) |
| **Data** | Centralized | Distributed |
| **Consistency** | Strong | Eventual (often) |
| **Implementation** | Hardware upgrade | Architecture redesign |

---

### Real-World Example: E-Commerce Growth

**Phase 1: Startup (Vertical)**
```
Users: 1,000
Server: 4 cores, 16GB RAM, PostgreSQL
Cost: $100/month
Strategy: Vertical scaling
```

**Phase 2: Growing (Vertical + Caching)**
```
Users: 50,000
Server: 16 cores, 64GB RAM, PostgreSQL + Redis
Cost: $500/month
Strategy: Still vertical, added cache
```

**Phase 3: Popular (Horizontal - Replication)**
```
Users: 500,000
Servers: 
  - 1 Primary DB (writes)
  - 3 Read Replicas (reads)
  - 2 Redis servers
Cost: $2,000/month
Strategy: Horizontal (read scaling)
```

**Phase 4: Large Scale (Horizontal - Sharding)**
```
Users: 5,000,000
Servers:
  - 4 Shards (distributed writes)
  - 12 Replicas (3 per shard)
  - 5 Redis servers (cache cluster)
Cost: $10,000/month
Strategy: Full horizontal (read + write scaling)
```

---

### Hybrid Approach (Best Practice)

**Combination:** Vertical + Horizontal

```
Start: Vertical scaling (simpler)
  ↓
Optimize: Add caching, indexes
  ↓
Grow: Add read replicas (horizontal reads)
  ↓
Scale: Add sharding (horizontal writes)
```

**Benefits:**
- ✅ Start simple (vertical)
- ✅ Scale when needed (horizontal)
- ✅ Maximize single server before distributing
- ✅ Gradual complexity increase

---

### Practice Questions: Scaling Basics

#### Fill in the Blanks

1. ________ scaling adds more resources to an existing server.
2. ________ scaling adds more servers to distribute load.
3. Vertical scaling has a ________ limit based on hardware capabilities.
4. Horizontal scaling provides ________ ________ through redundancy.
5. A hybrid approach starts with ________ scaling and adds ________ scaling as needed.

<details>
<summary><strong>View Answers</strong></summary>

1. Vertical
2. Horizontal
3. physical / hardware
4. high availability
5. vertical, horizontal

</details>

---

#### True/False

1. Vertical scaling is cheaper than horizontal scaling at large scale.
2. Horizontal scaling requires changes to application architecture.
3. A single powerful server has no single point of failure.
4. Horizontal scaling can handle nearly unlimited growth.
5. Vertical scaling is simpler to implement than horizontal scaling.

<details>
<summary><strong>View Answers</strong></summary>

1. FALSE (vertical becomes exponentially expensive)
2. TRUE
3. FALSE (single server is a single point of failure)
4. TRUE
5. TRUE

</details>

---

## 5.2 Replication (Read Scaling)

### What is Replication?

**Definition:** Maintaining copies of data across multiple servers.

**Basic Architecture:**
```
        ┌─────────────┐
        │   Primary   │ ← Writes
        │  (Master)   │
        └──────┬──────┘
               │
      ┌────────┴────────┐
      ▼                 ▼
┌──────────┐      ┌──────────┐
│ Replica 1│      │ Replica 2│ ← Reads
│(Secondary)│      │(Secondary)│
└──────────┘      └──────────┘
```

**How it Works:**
1. Writes go to Primary
2. Primary replicates changes to Replicas
3. Reads distributed across Replicas
4. If Primary fails, Replica promoted

---

### Why Replication?

**1. Read Scaling**
```
Without Replication:
- 1 server handles 1000 req/sec
- All reads + writes on same server

With Replication (1 Primary + 3 Replicas):
- Primary: 200 writes/sec
- Replica 1: 300 reads/sec
- Replica 2: 300 reads/sec
- Replica 3: 300 reads/sec
- Total: 1100 req/sec (more capacity!)
```

**2. High Availability**
```
Primary Fails:
- Replica promoted to Primary
- No downtime!
- Data preserved
```

**3. Geographic Distribution**
```
Primary: US East
Replica 1: US West (faster for west coast)
Replica 2: EU (faster for Europe)
Replica 3: Asia (faster for Asia)
```

**4. Backups**
```
Replica can be used for:
- Backups (snapshot without affecting primary)
- Analytics (heavy queries don't slow primary)
- Development/testing
```

---

## 5.2.1 SQL Replication (PostgreSQL/MySQL)

### PostgreSQL Replication

#### Types of PostgreSQL Replication

**1. Streaming Replication (Physical)**

**Setup:**

```bash
# postgresql.conf (Primary)
wal_level = replica
max_wal_senders = 3
wal_keep_size = 64  # Keep 64MB of WAL files

# Create replication user
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'password';

# pg_hba.conf (Primary)
host replication replicator replica-ip/32 md5
```

```bash
# Create replica (on replica server)
pg_basebackup -h primary-ip -D /var/lib/postgresql/data -U replicator -P

# standby.signal (Replica)
touch /var/lib/postgresql/data/standby.signal

# postgresql.conf (Replica)
primary_conninfo = 'host=primary-ip port=5432 user=replicator password=password'
hot_standby = on
```

**How it Works:**
```
1. Primary writes to WAL (Write-Ahead Log)
2. WAL streamed to Replicas
3. Replicas apply WAL changes
4. Replicas serve read-only queries
```

---

**2. Logical Replication (Table-Level)**

**Setup:**

```sql
-- Primary: Enable logical replication
ALTER SYSTEM SET wal_level = logical;
-- Restart PostgreSQL

-- Create publication (Primary)
CREATE PUBLICATION my_publication FOR TABLE users, orders;

-- Or publish all tables
CREATE PUBLICATION all_tables FOR ALL TABLES;

-- Replica: Create subscription
CREATE SUBSCRIPTION my_subscription
  CONNECTION 'host=primary-ip dbname=mydb user=replicator password=password'
  PUBLICATION my_publication;
```

**Advantages:**
- ✅ Selective replication (specific tables)
- ✅ Different PostgreSQL versions
- ✅ Can filter rows (WHERE clause)
- ✅ Merge from multiple sources

**Use Cases:**
- Replicate subset of tables
- Aggregate data from multiple DBs
- Migrate between versions

---

#### Read/Write Split in Application

```javascript
const { Pool } = require('pg');

// Primary connection (writes)
const primaryPool = new Pool({
  host: 'primary-db.example.com',
  database: 'mydb',
  user: 'app_user',
  password: 'password',
  max: 20
});

// Replica connections (reads)
const replicaPools = [
  new Pool({
    host: 'replica1-db.example.com',
    database: 'mydb',
    user: 'app_user',
    password: 'password',
    max: 20
  }),
  new Pool({
    host: 'replica2-db.example.com',
    database: 'mydb',
    user: 'app_user',
    password: 'password',
    max: 20
  })
];

// Round-robin load balancing for reads
let replicaIndex = 0;

function getReplicaPool() {
  const pool = replicaPools[replicaIndex];
  replicaIndex = (replicaIndex + 1) % replicaPools.length;
  return pool;
}

// Write query (goes to primary)
async function createUser(name, email) {
  const result = await primaryPool.query(
    'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
    [name, email]
  );
  return result.rows[0];
}

// Read query (goes to replica)
async function getUser(id) {
  const replica = getReplicaPool();
  const result = await replica.query(
    'SELECT * FROM users WHERE id = $1',
    [id]
  );
  return result.rows[0];
}

// Heavy analytics (goes to specific replica)
async function getAnalytics() {
  const result = await replicaPools[0].query(`
    SELECT 
      DATE(created_at) as date,
      COUNT(*) as user_count,
      SUM(total) as revenue
    FROM orders
    WHERE created_at >= NOW() - INTERVAL '30 days'
    GROUP BY DATE(created_at)
    ORDER BY date
  `);
  return result.rows;
}
```

---

### MySQL Replication

#### Master-Slave Replication

**Setup:**

```sql
-- Master (my.cnf)
[mysqld]
server-id = 1
log-bin = mysql-bin
binlog-do-db = mydb

-- Create replication user
CREATE USER 'replicator'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'%';

-- Get master status
SHOW MASTER STATUS;
-- Note: File and Position
```

```sql
-- Slave (my.cnf)
[mysqld]
server-id = 2
relay-log = mysql-relay-bin
read-only = 1

-- Configure replication
CHANGE MASTER TO
  MASTER_HOST='master-ip',
  MASTER_USER='replicator',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=154;

-- Start replication
START SLAVE;

-- Check status
SHOW SLAVE STATUS\G
```

---

#### MySQL Group Replication (Multi-Master)

**Setup:**

```sql
-- All nodes (my.cnf)
[mysqld]
server-id = 1  # Unique per server
gtid-mode = ON
enforce-gtid-consistency = ON
binlog-checksum = NONE
log-slave-updates = ON
log-bin = binlog
binlog-format = ROW

plugin-load-add = group_replication.so
group_replication_group_name = "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot = OFF
group_replication_local_address = "node1:33061"
group_replication_group_seeds = "node1:33061,node2:33061,node3:33061"

-- Bootstrap group (first node only)
SET GLOBAL group_replication_bootstrap_group=ON;
START GROUP_REPLICATION;
SET GLOBAL group_replication_bootstrap_group=OFF;

-- Join group (other nodes)
START GROUP_REPLICATION;
```

**Advantages:**
- ✅ Multi-master (writes to any node)
- ✅ Automatic failover
- ✅ Conflict detection

---

### Monitoring Replication

**PostgreSQL:**

```sql
-- On Primary: Check replication status
SELECT 
  client_addr,
  state,
  sent_lsn,
  write_lsn,
  flush_lsn,
  replay_lsn,
  sync_state
FROM pg_stat_replication;

-- On Replica: Check replication lag
SELECT 
  now() - pg_last_xact_replay_timestamp() AS replication_lag;
```

**MySQL:**

```sql
-- Check slave status
SHOW SLAVE STATUS\G

-- Key metrics:
-- Seconds_Behind_Master: Replication lag
-- Slave_IO_Running: Should be "Yes"
-- Slave_SQL_Running: Should be "Yes"
```

---

### Practice Questions: SQL Replication

#### Fill in the Blanks

1. PostgreSQL ________ replication streams the Write-Ahead Log to replicas.
2. ________ replication allows selective table-level replication in PostgreSQL.
3. In a read/write split, ________ queries go to the primary and ________ queries go to replicas.
4. MySQL uses the ________ to track changes for replication.
5. Replication ________ is the delay between primary and replica.

<details>
<summary><strong>View Answers</strong></summary>

1. streaming (or physical)
2. Logical
3. write, read
4. binary log (binlog)
5. lag

</details>

---

#### Coding Challenge: Implement Read/Write Split

Create a database wrapper that automatically routes reads to replicas and writes to primary.

<details>
<summary><strong>View Solution</strong></summary>

```javascript
class DatabasePool {
  constructor(primaryConfig, replicaConfigs) {
    this.primary = new Pool(primaryConfig);
    this.replicas = replicaConfigs.map(config => new Pool(config));
    this.replicaIndex = 0;
  }
  
  // Get next replica (round-robin)
  getReplicaPool() {
    const pool = this.replicas[this.replicaIndex];
    this.replicaIndex = (this.replicaIndex + 1) % this.replicas.length;
    return pool;
  }
  
  // Execute query with automatic routing
  async query(sql, params = [], options = {}) {
    const isWrite = /^\s*(INSERT|UPDATE|DELETE|CREATE|ALTER|DROP)/i.test(sql);
    const forceWrite = options.usePrimary || false;
    
    const pool = (isWrite || forceWrite) ? this.primary : this.getReplicaPool();
    
    try {
      const result = await pool.query(sql, params);
      return result;
    } catch (error) {
      console.error('Query failed:', error);
      throw error;
    }
  }
  
  // Explicit write
  async write(sql, params = []) {
    return await this.primary.query(sql, params);
  }
  
  // Explicit read
  async read(sql, params = []) {
    const replica = this.getReplicaPool();
    return await replica.query(sql, params);
  }
  
  // Transaction (always on primary)
  async transaction(callback) {
    const client = await this.primary.connect();
    
    try {
      await client.query('BEGIN');
      const result = await callback(client);
      await client.query('COMMIT');
      return result;
    } catch (error) {
      await client.query('ROLLBACK');
      throw error;
    } finally {
      client.release();
    }
  }
  
  // Close all connections
  async close() {
    await this.primary.end();
    await Promise.all(this.replicas.map(pool => pool.end()));
  }
}

// Usage
const db = new DatabasePool(
  { host: 'primary-db.com', /* ... */ },
  [
    { host: 'replica1-db.com', /* ... */ },
    { host: 'replica2-db.com', /* ... */ }
  ]
);

// Automatic routing
await db.query('SELECT * FROM users WHERE id = $1', [123]);  // → Replica
await db.query('INSERT INTO users (name) VALUES ($1)', ['Alice']);  // → Primary

// Force primary for critical reads (avoid replication lag)
await db.query('SELECT * FROM users WHERE id = $1', [123], { usePrimary: true });

// Explicit
await db.read('SELECT * FROM products LIMIT 10');
await db.write('UPDATE products SET stock = stock - 1 WHERE id = $1', [456]);

// Transaction
await db.transaction(async (client) => {
  await client.query('UPDATE accounts SET balance = balance - 100 WHERE id = $1', [1]);
  await client.query('UPDATE accounts SET balance = balance + 100 WHERE id = $1', [2]);
});
```

</details>

---

## 5.2.2 MongoDB Replica Sets

### What is a Replica Set?

**Definition:** A group of MongoDB instances that maintain the same data set.

**Architecture:**
```
     ┌─────────────┐
     │   PRIMARY   │ ← All writes
     └──────┬──────┘
            │
    ┌───────┴───────┐
    ▼               ▼
┌──────────┐   ┌──────────┐
│SECONDARY │   │SECONDARY │ ← Reads (optional)
└──────────┘   └──────────┘
```

**Minimum Configuration:** 3 members
- 1 Primary (accepts writes)
- 2 Secondaries (replicate data)

---

### Setting Up a Replica Set

#### Step 1: Start MongoDB Instances

```bash
# Start 3 MongoDB instances (different ports)

# Member 1 (will become primary)
mongod --replSet rs0 --port 27017 --dbpath /data/rs0-1 --bind_ip localhost

# Member 2
mongod --replSet rs0 --port 27018 --dbpath /data/rs0-2 --bind_ip localhost

# Member 3
mongod --replSet rs0 --port 27019 --dbpath /data/rs0-3 --bind_ip localhost
```

---

#### Step 2: Initialize Replica Set

```javascript
// Connect to first instance
mongosh --port 27017

// Initialize replica set
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "localhost:27017" },
    { _id: 1, host: "localhost:27018" },
    { _id: 2, host: "localhost:27019" }
  ]
});

// Check status
rs.status();

// Output shows:
// - One PRIMARY
// - Two SECONDARY
// - Replication state: HEALTHY
```

---

#### Step 3: Verify Replication

```javascript
// Connect to primary
use testdb;

// Insert data
db.users.insertOne({ name: "Alice", email: "alice@example.com" });

// Connect to secondary
mongosh --port 27018

use testdb;

// Enable reading from secondary
rs.secondaryOk();  // or db.getMongo().setReadPref('secondary')

// Verify data replicated
db.users.find();
// Should show Alice!
```

---

### Replica Set Configuration

#### Priority (Election Weight)

```javascript
// Set priorities (higher = more likely to become primary)
cfg = rs.conf();

cfg.members[0].priority = 2;  // Preferred primary
cfg.members[1].priority = 1;  // Standard
cfg.members[2].priority = 0;  // Never primary (passive)

rs.reconfig(cfg);
```

**Use Cases:**
- Data center preference (higher priority in main DC)
- Hardware differences (powerful server = higher priority)
- Geographic considerations

---

#### Hidden Members (Analytics)

```javascript
// Hidden member: Replicates but invisible to clients
cfg = rs.conf();

cfg.members[2].hidden = true;
cfg.members[2].priority = 0;  // Must be 0

rs.reconfig(cfg);
```

**Use Cases:**
- Dedicated analytics server
- Backup server
- Reporting without affecting production

---

#### Delayed Members (Point-in-Time Recovery)

```javascript
// Delayed member: Replicates with time lag
cfg = rs.conf();

cfg.members[2].slaveDelay = 3600;  // 1 hour delay
cfg.members[2].priority = 0;
cfg.members[2].hidden = true;

rs.reconfig(cfg);
```

**Use Case:** Recover from accidental data deletion
```
10:00 - Accidentally delete important collection
10:30 - Notice the mistake
10:45 - Recover from delayed member (still has data from 9:45)
```

---

#### Arbiter (Tie Breaker)

```javascript
// Arbiter: Participates in elections but holds no data
rs.addArb("localhost:27020");
```

**Use Case:** Even number of members
```
2 Data Members + 1 Arbiter = 3 voting members
Cheaper than 3 full data members
```

**Configuration:**
```
┌─────────┐     ┌─────────┐
│ PRIMARY │     │SECONDARY│ ← Data members
└─────────┘     └─────────┘
        \         /
         \       /
          \     /
        ┌─────────┐
        │ ARBITER │ ← No data, votes only
        └─────────┘
```

---

### Automatic Failover

**How Failover Works:**

```
Normal State:
PRIMARY ←─ Writes
  │
  ├─→ SECONDARY 1
  └─→ SECONDARY 2

Primary Fails:
✗ PRIMARY (offline)
  │
  ├─→ SECONDARY 1 ← Election starts
  └─→ SECONDARY 2 ← Election starts

After Election (< 10 seconds):
NEW PRIMARY ←─ Writes
  │
  ├─→ ✗ Old Primary (offline)
  └─→ SECONDARY

Old Primary Recovers:
PRIMARY
  │
  ├─→ SECONDARY (old primary rejoins)
  └─→ SECONDARY
```

**Election Process:**
1. Primary fails (heartbeat missed)
2. Secondaries detect failure (10 seconds)
3. Election begins
4. Member with highest priority wins
5. New primary elected (5-10 seconds total)
6. Clients automatically reconnect

---

### Application Connection String

```javascript
// Single server (no failover)
mongodb://localhost:27017/mydb

// Replica set (automatic failover)
mongodb://localhost:27017,localhost:27018,localhost:27019/mydb?replicaSet=rs0

// With read preference
mongodb://localhost:27017,localhost:27018,localhost:27019/mydb?replicaSet=rs0&readPreference=secondaryPreferred
```

**Node.js Example:**

```javascript
const { MongoClient } = require('mongodb');

const client = new MongoClient(
  'mongodb://localhost:27017,localhost:27018,localhost:27019/mydb?replicaSet=rs0',
  {
    // Automatic retry on failover
    retryWrites: true,
    retryReads: true,
    
    // Read preference
    readPreference: 'secondaryPreferred'
  }
);

await client.connect();

// Writes go to primary
await client.db('mydb').collection('users').insertOne({ 
  name: 'Alice' 
});

// Reads can go to secondaries
const users = await client.db('mydb').collection('users').find().toArray();
```

---

### Read Preferences

**Options:**

1. **primary** (default)
```javascript
db.users.find().readPref('primary');
// Reads from primary only
// Strong consistency, but no read scaling
```

2. **primaryPreferred**
```javascript
db.users.find().readPref('primaryPreferred');
// Primary if available, secondary if primary down
// Good for critical reads during failover
```

3. **secondary**
```javascript
db.users.find().readPref('secondary');
// Reads from secondary only
// Maximum read scaling, but possible stale data
```

4. **secondaryPreferred** (recommended)
```javascript
db.users.find().readPref('secondaryPreferred');
// Secondary if available, primary as fallback
// Good balance of scaling + availability
```

5. **nearest**
```javascript
db.users.find().readPref('nearest');
// Reads from member with lowest network latency
// Best for geographically distributed replica sets
```

---

### Write Concerns

**Control write acknowledgment:**

```javascript
// w: 1 (default) - Acknowledge from primary only
db.users.insertOne(
  { name: 'Alice' },
  { writeConcern: { w: 1 } }
);
// Fast, but data may not be replicated yet

// w: 2 - Acknowledge from primary + 1 secondary
db.users.insertOne(
  { name: 'Bob' },
  { writeConcern: { w: 2 } }
);
// Slower, but data on 2 servers before acknowledgment

// w: 'majority' - Acknowledge from majority of members
db.users.insertOne(
  { name: 'Carol' },
  { writeConcern: { w: 'majority' } }
);
// Safest, data on majority before acknowledgment

// j: true - Wait for journal commit
db.users.insertOne(
  { name: 'Dave' },
  { writeConcern: { w: 'majority', j: true } }
);
// Survives server crash (data on disk)
```

**Trade-offs:**

| Write Concern | Speed | Durability | Use Case |
|--------------|-------|------------|----------|
| `w: 1` | Fastest | Low | Logs, metrics |
| `w: 2` | Medium | Medium | Regular data |
| `w: 'majority'` | Slow | High | Critical data |
| `w: 'majority', j: true` | Slowest | Highest | Financial transactions |

---

### Monitoring Replica Sets

```javascript
// Replica set status
rs.status();

// Key metrics:
// - stateStr: PRIMARY, SECONDARY, ARBITER
// - health: 1 (healthy), 0 (down)
// - optime: Last operation timestamp
// - optimeDate: Last operation date
// - syncingTo: Which member syncing from

// Replication lag
rs.printReplicationInfo();
// Shows:
// - configured oplog size
// - log length start to end
// - oplog first event time
// - oplog last event time
// - how long oplog covers

// Secondary lag
rs.printSlaveReplicationInfo();
// Shows:
// - source (which member)
// - syncedTo: timestamp
// - seconds behind master
```

---

### Practice Questions: MongoDB Replica Sets

#### Fill in the Blanks

1. A MongoDB replica set requires a minimum of ________ members.
2. The ________ member accepts all write operations in a replica set.
3. An ________ participates in elections but holds no data.
4. ________ members replicate data with a time delay for point-in-time recovery.
5. Write concern ________ waits for acknowledgment from the majority of replica set members.

<details>
<summary><strong>View Answers</strong></summary>

1. three (3)
2. primary
3. arbiter
4. Delayed
5. 'majority'

</details>

---

#### Coding Challenge: Configure Replica Set for Multi-Region

Set up a replica set with:
- 2 members in US (one primary-preferred)
- 1 member in EU (for EU reads)
- 1 delayed member (1 hour) for recovery

<details>
<summary><strong>View Solution</strong></summary>

```javascript
// Initialize replica set
rs.initiate({
  _id: "global-rs",
  members: [
    {
      _id: 0,
      host: "us-east-1.example.com:27017",
      priority: 2,  // Preferred primary
      tags: { region: "us", dc: "east" }
    },
    {
      _id: 1,
      host: "us-west-1.example.com:27017",
      priority: 1,
      tags: { region: "us", dc: "west" }
    },
    {
      _id: 2,
      host: "eu-central-1.example.com:27017",
      priority: 1,
      tags: { region: "eu", dc: "central" }
    },
    {
      _id: 3,
      host: "us-east-1-delayed.example.com:27017",
      priority: 0,
      hidden: true,
      slaveDelay: 3600,  // 1 hour delay
      tags: { usage: "recovery" }
    }
  ],
  settings: {
    // Ensure writes replicate to both US and EU
    getLastErrorModes: {
      multiRegion: { region: 2 }
    }
  }
});

// Application connection strings:

// US users - prefer US servers
const usConnectionString = 
  'mongodb://us-east-1.example.com:27017,us-west-1.example.com:27017,eu-central-1.example.com:27017/mydb?' +
  'replicaSet=global-rs&' +
  'readPreference=nearest&' +
  'readPreferenceTags=region:us';

// EU users - prefer EU servers
const euConnectionString = 
  'mongodb://us-east-1.example.com:27017,us-west-1.example.com:27017,eu-central-1.example.com:27017/mydb?' +
  'replicaSet=global-rs&' +
  'readPreference=nearest&' +
  'readPreferenceTags=region:eu';

// Critical writes - ensure multi-region replication
await db.collection('orders').insertOne(
  { /* order data */ },
  { 
    writeConcern: { 
      w: 'multiRegion',  // Custom write concern
      j: true,
      wtimeout: 5000
    } 
  }
);
```

</details>

---

## 5.2.3 Replication Lag & Consistency

### What is Replication Lag?

**Definition:** Time delay between primary write and replica availability.

```
Timeline:
10:00:00.000 - Write on Primary
10:00:00.050 - Replicated to Secondary (50ms lag)
10:00:00.100 - Available on Secondary (100ms total)
```

**Causes:**
1. Network latency (geographic distance)
2. Secondary overload (heavy reads/analytics)
3. Large transactions
4. Hardware differences (slower secondary)
5. Network congestion

---

### Measuring Replication Lag

**PostgreSQL:**

```sql
-- On Primary
SELECT 
  client_addr,
  state,
  sent_lsn,
  replay_lsn,
  (sent_lsn - replay_lsn) AS lag_bytes,
  EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp())) AS lag_seconds
FROM pg_stat_replication;

-- On Replica
SELECT 
  now() - pg_last_xact_replay_timestamp() AS replication_lag,
  pg_is_in_recovery() AS is_replica;
```

**MongoDB:**

```javascript
// Check replication lag
rs.printSlaveReplicationInfo();

// Output:
// source: localhost:27017
// syncedTo: Thu Jan 15 2024 10:00:05 GMT-0500 (EST)
// 0 secs (0 hrs) behind the primary

// Programmatic check
const status = rs.status();
const primary = status.members.find(m => m.stateStr === 'PRIMARY');
const secondaries = status.members.filter(m => m.stateStr === 'SECONDARY');

secondaries.forEach(secondary => {
  const lag = (primary.optimeDate - secondary.optimeDate) / 1000;
  console.log(`${secondary.name}: ${lag} seconds behind`);
});
```

**MySQL:**

```sql
SHOW SLAVE STATUS\G

-- Key metrics:
-- Seconds_Behind_Master: Replication lag in seconds
-- Slave_IO_Running: Should be "Yes"
-- Slave_SQL_Running: Should be "Yes"
```

---

### Consistency Models

#### 1. Strong Consistency

**Definition:** Reads always return the latest write.

```javascript
// Read from primary only
db.users.find().readPref('primary');

// Or use majority write + read concern
await db.users.insertOne(
  { name: 'Alice' },
  { writeConcern: { w: 'majority' } }
);

const user = await db.users.findOne(
  { name: 'Alice' },
  { readConcern: { level: 'majority' }, readPreference: 'primary' }
);
```

**Pros:**
- ✅ Always see latest data
- ✅ No stale reads
- ✅ Simpler application logic

**Cons:**
- ❌ No read scaling
- ❌ Higher latency
- ❌ Primary becomes bottleneck

**Use Cases:**
- Financial transactions
- Inventory management
- User authentication

---

#### 2. Eventual Consistency

**Definition:** Reads may return stale data, but eventually consistent.

```javascript
// Read from secondaries
db.users.find().readPref('secondary');

// Write to primary
await db.users.insertOne({ name: 'Bob' });

// Immediate read from secondary might not see Bob yet
// But will see it after replication lag (< 100ms typically)
```

**Pros:**
- ✅ Read scaling (distribute reads)
- ✅ Lower latency (read from nearest)
- ✅ Higher throughput

**Cons:**
- ❌ Possible stale reads
- ❌ Complex application logic
- ❌ User confusion (just wrote data, can't see it)

**Use Cases:**
- Social media feeds
- Product catalogs
- Analytics dashboards

---

#### 3. Read-After-Write Consistency

**Definition:** User always sees their own writes.

**Implementation:**

```javascript
class ConsistentDB {
  constructor(client) {
    this.client = client;
    this.db = client.db('mydb');
  }
  
  // Write to primary
  async write(collection, doc) {
    const result = await this.db.collection(collection).insertOne(
      doc,
      { writeConcern: { w: 'majority' } }
    );
    
    // Store user's last write timestamp
    return {
      _id: result.insertedId,
      writtenAt: new Date()
    };
  }
  
  // Read with consistency guarantee
  async read(collection, query, userId, lastWriteTime) {
    // If user wrote recently (< 5 seconds), read from primary
    if (lastWriteTime && (Date.now() - lastWriteTime) < 5000) {
      return await this.db.collection(collection)
        .findOne(query, { readPreference: 'primary' });
    }
    
    // Otherwise, read from secondary
    return await this.db.collection(collection)
      .findOne(query, { readPreference: 'secondaryPreferred' });
  }
}

// Usage
const db = new ConsistentDB(mongoClient);

// User writes
const result = await db.write('posts', {
  userId: '123',
  title: 'My Post',
  content: 'Hello World'
});

// User reads immediately (goes to primary)
const post = await db.read(
  'posts',
  { _id: result._id },
  '123',
  result.writtenAt
);
// Guaranteed to see their own write

// User reads after 5 seconds (goes to secondary)
setTimeout(async () => {
  const post = await db.read(
    'posts',
    { _id: result._id },
    '123',
    result.writtenAt
  );
  // May use secondary (replication likely complete)
}, 6000);
```

---

### Handling Replication Lag

#### Strategy 1: Acceptable Staleness Window

```javascript
// Accept data up to 5 seconds old
async function getProduct(productId) {
  const cacheKey = `product:${productId}`;
  
  // Check cache (Redis)
  let product = await redis.get(cacheKey);
  if (product) return JSON.parse(product);
  
  // Read from secondary (may be stale)
  product = await db.products.findOne(
    { _id: productId },
    { readPreference: 'secondaryPreferred' }
  );
  
  // Cache for 5 seconds
  await redis.setex(cacheKey, 5, JSON.stringify(product));
  
  return product;
}
```

---

#### Strategy 2: Critical Path to Primary

```javascript
async function placeOrder(userId, items) {
  // Check inventory (read from primary - critical)
  const products = await db.products.find(
    { _id: { $in: items.map(i => i.productId) } },
    { readPreference: 'primary' }  // Must be current
  ).toArray();
  
  // Verify stock
  for (const item of items) {
    const product = products.find(p => p._id.equals(item.productId));
    if (product.stock < item.quantity) {
      throw new Error(`Insufficient stock for ${product.name}`);
    }
  }
  
  // Create order (write to primary)
  const order = await db.orders.insertOne(
    { userId, items, createdAt: new Date() },
    { writeConcern: { w: 'majority' } }
  );
  
  // Update inventory (write to primary)
  for (const item of items) {
    await db.products.updateOne(
      { _id: item.productId },
      { $inc: { stock: -item.quantity } },
      { writeConcern: { w: 'majority' } }
    );
  }
  
  return order;
}
```

---

#### Strategy 3: Version Vectors / Timestamps

```javascript
// Track when data was written
async function updateUser(userId, updates) {
  const result = await db.users.findOneAndUpdate(
    { _id: userId },
    { 
      $set: { 
        ...updates,
        updatedAt: new Date(),
        version: new Date().getTime()
      } 
    },
    { 
      returnDocument: 'after',
      writeConcern: { w: 'majority' }
    }
  );
  
  return result.value;
}

// Read with version check
async function getUser(userId, minVersion = 0) {
  let retries = 0;
  const maxRetries = 3;
  
  while (retries < maxRetries) {
    const user = await db.users.findOne(
      { _id: userId },
      { readPreference: 'secondaryPreferred' }
    );
    
    // Check if version is recent enough
    if (user && user.version >= minVersion) {
      return user;
    }
    
    // Version too old, retry with primary
    if (retries === maxRetries - 1) {
      return await db.users.findOne(
        { _id: userId },
        { readPreference: 'primary' }
      );
    }
    
    // Wait for replication
    await new Promise(resolve => setTimeout(resolve, 100));
    retries++;
  }
}
```

---

### Practice Questions: Replication Lag & Consistency

#### Multiple Choice

**Q1: Which consistency model guarantees users see their own writes?**

A) Strong consistency
B) Eventual consistency
C) Read-after-write consistency
D) Causal consistency

<details>
<summary><strong>View Answer</strong></summary>

**Answer: C - Read-after-write consistency**

Explanation: Read-after-write consistency ensures that a user always sees their own writes, even if other users might see stale data. This is implemented by routing a user's reads to the primary shortly after they write.

</details>

---

**Q2: What is the main cause of replication lag?**

A) Too many writes
B) Network latency and distance
C) Large database size
D) Too many indexes

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - Network latency and distance**

Explanation: Replication lag is primarily caused by the time it takes for data to travel from the primary to replicas over the network, especially in geographically distributed systems.

</details>

---

## 5.3 Sharding (Write Scaling)

### What is Sharding?

**Definition:** Partitioning data across multiple servers (shards).

**Problem Sharding Solves:**
```
Single Database:
- 100M users
- 10 TB data
- 10,000 writes/sec
- One server can't handle it! 💥
```

**Solution:**
```
Shard 1: Users A-M (5 TB, 5,000 writes/sec)
Shard 2: Users N-Z (5 TB, 5,000 writes/sec)

Total capacity doubled!
```

---

### Replication vs Sharding

| Feature | Replication | Sharding |
|---------|------------|----------|
| **Purpose** | Read scaling, HA | Write scaling |
| **Data** | Full copy on each server | Subset on each server |
| **Writes** | All to primary | Distributed across shards |
| **Reads** | Distributed across replicas | Routed to correct shard |
| **Complexity** | Low | High |
| **Use When** | Read-heavy | Write-heavy, huge data |

**Often Combined:**
```
Shard 1: Primary + 2 Replicas
Shard 2: Primary + 2 Replicas
Shard 3: Primary + 2 Replicas

= Write scaling (sharding) + Read scaling (replication)
```

---

## 5.3.1 Sharding Strategies

### 1. Range-Based Sharding

**Definition:** Partition by value ranges.

**Example: Users by Last Name**
```
Shard 1: A-G
Shard 2: H-N
Shard 3: O-Z
```

**Implementation:**

```javascript
function getShardForUser(lastName) {
  const firstLetter = lastName[0].toUpperCase();
  
  if (firstLetter >= 'A' && firstLetter <= 'G') return 'shard1';
  if (firstLetter >= 'H' && firstLetter <= 'N') return 'shard2';
  if (firstLetter >= 'O' && firstLetter <= 'Z') return 'shard3';
}

// Insert user
const shard = getShardForUser('Smith');  // shard3
await shards[shard].users.insertOne({ lastName: 'Smith', /* ... */ });

// Query user
const shard = getShardForUser('Johnson');  // shard2
const user = await shards[shard].users.findOne({ lastName: 'Johnson' });
```

**Advantages:**
- ✅ Simple to understand
- ✅ Range queries efficient (all data on one shard)
- ✅ Easy to add/remove ranges

**Disadvantages:**
- ❌ **Hotspots:** Uneven data distribution
  ```
  Shard 1 (A-G): 10M users
  Shard 2 (H-N): 15M users
  Shard 3 (O-Z): 5M users ← Underutilized
  ```
- ❌ **Popular ranges:** Shard 2 overloaded (many last names start with M, S)

**Use Cases:**
- Time-series data (partition by date)
- Geographic data (partition by region)
- When range queries are common

---

### 2. Hash-Based Sharding

**Definition:** Partition by hash of shard key.

**Example: Users by ID**
```javascript
function getShardForUserId(userId) {
  const hash = simpleHash(userId);
  const shardCount = 3;
  return `shard${(hash % shardCount) + 1}`;
}

function simpleHash(str) {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    hash = ((hash << 5) - hash) + str.charCodeAt(i);
    hash = hash & hash; // Convert to 32-bit integer
  }
  return Math.abs(hash);
}

// Examples:
getShardForUserId('user-123');  // shard2
getShardForUserId('user-456');  // shard1
getShardForUserId('user-789');  // shard3
```

**Advantages:**
- ✅ **Even distribution:** No hotspots
- ✅ Predictable performance
- ✅ Simple algorithm

**Disadvantages:**
- ❌ **Range queries slow:** Must query all shards
  ```javascript
  // Get users created in January
  // Must query ALL shards and merge results
  const results = await Promise.all(
    shards.map(shard => 
      shard.users.find({ createdAt: { $gte: jan1, $lt: feb1 } })
    )
  );
  ```
- ❌ **Resharding hard:** Changing shard count rehashes everything

**Use Cases:**
- Evenly distributed writes
- Point lookups (by ID)
- When range queries are rare

---

### 3. Geographic/Directory-Based Sharding

**Definition:** Partition by location or tenant.

**Example: Multi-Tenant SaaS**
```
Shard 1: Company A, Company B
Shard 2: Company C, Company D
Shard 3: Company E, Company F
```

**Implementation:**

```javascript
// Tenant mapping table
const tenantMapping = {
  'company-a': 'shard1',
  'company-b': 'shard1',
  'company-c': 'shard2',
  'company-d': 'shard2',
  'company-e': 'shard3',
  'company-f': 'shard3'
};

function getShardForTenant(tenantId) {
  return tenantMapping[tenantId];
}

// Insert data
const shard = getShardForTenant('company-a');  // shard1
await shards[shard].orders.insertOne({
  tenantId: 'company-a',
  orderId: 'order-123',
  /* ... */
});

// Query data
const shard = getShardForTenant('company-c');  // shard2
const orders = await shards[shard].orders.find({ 
  tenantId: 'company-c' 
}).toArray();
```

**Advantages:**
- ✅ **Data locality:** Related data on same shard
- ✅ **Tenant isolation:** Each tenant on specific shard
- ✅ **Easy compliance:** Region-specific data storage
- ✅ **Flexible:** Can move tenants between shards

**Disadvantages:**
- ❌ **Uneven distribution:** Large tenants vs small tenants
- ❌ **Mapping overhead:** Need to maintain directory
- ❌ **Cross-tenant queries:** Must query multiple shards

**Use Cases:**
- Multi-tenant applications
- Geographic data isolation (GDPR)
- Enterprise customers with dedicated shards

---

### 4. Consistent Hashing

**Definition:** Hash-based with minimal data movement on resharding.

**How it Works:**

```
Hash Ring (0-360°):
         0°
         │
    Shard3  Shard1
         │
   270° ─┼─ 90°
         │
    Shard2
         │
        180°

User hash determines position on ring:
- user-123: hash=45° → Shard1 (next clockwise)
- user-456: hash=120° → Shard2 (next clockwise)
- user-789: hash=300° → Shard3 (next clockwise)
```

**Adding Shard:**
```
Before: 3 shards
After: 4 shards

Only ~25% of data moves (to new shard)
Not 100% like simple hash!
```

**Implementation:**

```javascript
class ConsistentHash {
  constructor(shards, virtualNodes = 150) {
    this.ring = [];
    this.shards = {};
    
    // Create virtual nodes for better distribution
    for (const shard of shards) {
      for (let i = 0; i < virtualNodes; i++) {
        const hash = this.hash(`${shard}-${i}`);
        this.ring.push({ hash, shard });
        this.shards[shard] = true;
      }
    }
    
    // Sort ring by hash
    this.ring.sort((a, b) => a.hash - b.hash);
  }
  
  hash(key) {
    // Simple hash function (use better one in production)
    let hash = 0;
    for (let i = 0; i < key.length; i++) {
      hash = ((hash << 5) - hash) + key.charCodeAt(i);
      hash = hash & hash;
    }
    return Math.abs(hash);
  }
  
  getShard(key) {
    if (this.ring.length === 0) return null;
    
    const hash = this.hash(key);
    
    // Binary search for next shard (clockwise)
    let left = 0;
    let right = this.ring.length - 1;
    
    while (left < right) {
      const mid = Math.floor((left + right) / 2);
      if (this.ring[mid].hash < hash) {
        left = mid + 1;
      } else {
        right = mid;
      }
    }
    
    return this.ring[left].shard;
  }
  
  addShard(shard, virtualNodes = 150) {
    for (let i = 0; i < virtualNodes; i++) {
      const hash = this.hash(`${shard}-${i}`);
      this.ring.push({ hash, shard });
    }
    this.shards[shard] = true;
    this.ring.sort((a, b) => a.hash - b.hash);
  }
  
  removeShard(shard) {
    this.ring = this.ring.filter(node => node.shard !== shard);
    delete this.shards[shard];
  }
}

// Usage
const ch = new ConsistentHash(['shard1', 'shard2', 'shard3']);

console.log(ch.getShard('user-123'));  // shard2
console.log(ch.getShard('user-456'));  // shard1
console.log(ch.getShard('user-789'));  // shard3

// Add new shard - only ~25% of data moves
ch.addShard('shard4');
```

**Advantages:**
- ✅ **Minimal data movement:** Only affected data moves
- ✅ **Elastic scaling:** Easy to add/remove shards
- ✅ **Even distribution:** Virtual nodes balance load

**Disadvantages:**
- ❌ **Complex implementation**
- ❌ **Range queries still slow**

**Use Cases:**
- Distributed caches (Redis, Memcached)
- Dynamically scaling systems
- Cloud-native applications

---

### Choosing a Sharding Strategy

```
Decision Tree:

Need range queries?
├─ YES → Range-based sharding
│         (e.g., time-series, geographic)
└─ NO
   │
   Multi-tenant application?
   ├─ YES → Directory-based sharding
   │         (tenant isolation, data locality)
   └─ NO
      │
      Need dynamic scaling?
      ├─ YES → Consistent hashing
      │         (elastic, minimal data movement)
      └─ NO → Hash-based sharding
                (simple, even distribution)
```

---

### Practice Questions: Sharding Strategies

#### Fill in the Blanks

1. ________ sharding partitions data by value ranges.
2. ________ sharding uses a hash function to distribute data evenly.
3. ________ ________ minimizes data movement when adding or removing shards.
4. ________ -based sharding is ideal for multi-tenant applications.
5. The main disadvantage of hash-based sharding is that ________ queries must hit all shards.

<details>
<summary><strong>View Answers</strong></summary>

1. Range-based
2. Hash-based
3. Consistent hashing
4. Directory (or Geographic)
5. range

</details>

---

#### Scenario Question

**Q: You're building a social media platform with 100M users. You need to:**
- Store user profiles
- Support queries like "find all users in California"
- Handle 50,000 profile updates/sec
- Scale to 500M users

**Which sharding strategy should you use and why?**

<details>
<summary><strong>View Answer</strong></summary>

**Best Choice: Hybrid Approach**

**Primary Sharding: Hash-based on user_id**
```javascript
// Even distribution of writes
shard = hash(user_id) % num_shards
```

**Reasons:**
- ✅ Handles 50,000 writes/sec evenly distributed
- ✅ Point lookups by user_id are fast
- ✅ Can scale to 500M users by adding shards

**Secondary Index: Geographic**
```javascript
// Separate geo index for location queries
geo_index: {
  'California': ['shard1', 'shard2', 'shard3'],  // Users in CA across shards
  'New York': ['shard1', 'shard2', 'shard3'],
  // ...
}
```

**For geographic queries:**
1. Lookup which shards have users in California (from geo index)
2. Query those shards in parallel
3. Merge results

**Trade-off:** Geographic queries slower but acceptable for less frequent use case.

**Alternative:** If geographic queries are critical, use **range-based sharding by region** (but accept uneven distribution).

</details>

---

## 5.3.2 MongoDB Sharding

### MongoDB Sharding Architecture

```
              ┌─────────────┐
              │   mongos    │ ← Query Router
              │  (Router)   │
              └──────┬──────┘
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
  ┌────────┐    ┌────────┐    ┌────────┐
  │Shard 1 │    │Shard 2 │    │Shard 3 │
  │Replica │    │Replica │    │Replica │
  │  Set   │    │  Set   │    │  Set   │
  └────────┘    └────────┘    └────────┘
       │
       │        ┌──────────────┐
       └────────│Config Servers│ ← Metadata
                │ (Replica Set)│
                └──────────────┘
```

**Components:**

1. **mongos (Router):** Routes queries to appropriate shards
2. **Config Servers:** Store metadata (which data on which shard)
3. **Shards:** Replica sets holding data

---

### Setting Up MongoDB Sharding

#### Step 1: Start Config Servers (Replica Set)

```bash
# Start 3 config servers
mongod --configsvr --replSet configRS --port 27019 --dbpath /data/config1
mongod --configsvr --replSet configRS --port 27020 --dbpath /data/config2
mongod --configsvr --replSet configRS --port 27021 --dbpath /data/config3

# Initialize config replica set
mongosh --port 27019
rs.initiate({
  _id: "configRS",
  configsvr: true,
  members: [
    { _id: 0, host: "localhost:27019" },
    { _id: 1, host: "localhost:27020" },
    { _id: 2, host: "localhost:27021" }
  ]
});
```

---

#### Step 2: Start Shards (Replica Sets)

```bash
# Shard 1
mongod --shardsvr --replSet shard1RS --port 27017 --dbpath /data/shard1-1
mongod --shardsvr --replSet shard1RS --port 27018 --dbpath /data/shard1-2

mongosh --port 27017
rs.initiate({
  _id: "shard1RS",
  members: [
    { _id: 0, host: "localhost:27017" },
    { _id: 1, host: "localhost:27018" }
  ]
});

# Shard 2 (similar setup on ports 27022-27023)
# Shard 3 (similar setup on ports 27024-27025)
```

---

#### Step 3: Start mongos (Router)

```bash
mongos --configdb configRS/localhost:27019,localhost:27020,localhost:27021 --port 27100
```

---

#### Step 4: Add Shards

```javascript
// Connect to mongos
mongosh --port 27100

// Add shards
sh.addShard("shard1RS/localhost:27017,localhost:27018");
sh.addShard("shard2RS/localhost:27022,localhost:27023");
sh.addShard("shard3RS/localhost:27024,localhost:27025");

// Verify
sh.status();
```

---

#### Step 5: Enable Sharding on Database

```javascript
// Enable sharding on database
sh.enableSharding("mydb");

// Create shard key index
db.users.createIndex({ userId: "hashed" });

// Shard collection (hash-based)
sh.shardCollection("mydb.users", { userId: "hashed" });

// Or shard with range-based
db.orders.createIndex({ customerId: 1 });
sh.shardCollection("mydb.orders", { customerId: 1 });
```

---

### Shard Key Selection

**Critical Decision:** Shard key determines data distribution.

**Good Shard Key Properties:**

1. **High Cardinality:** Many distinct values
   ```javascript
   ✅ Good: userId (millions of values)
   ❌ Bad: gender (2-3 values)
   ❌ Bad: status (5-10 values)
   ```

2. **Even Distribution:** No hotspots
   ```javascript
   ✅ Good: hash(userId)
   ❌ Bad: timestamp (all new writes to one shard)
   ```

3. **Non-Monotonic:** Not always increasing
   ```javascript
   ✅ Good: hash(userId)
   ❌ Bad: auto-increment ID (newest always to last shard)
   ```

4. **Frequently Queried:** Used in most queries
   ```javascript
   ✅ Good: If you query by userId, shard on userId
   ❌ Bad: Shard on one field, query by another
   ```

---

### Shard Key Examples

**Example 1: E-Commerce Orders**

```javascript
// ❌ Bad: Order date (monotonic, all new orders to one shard)
sh.shardCollection("mydb.orders", { orderDate: 1 });

// ❌ Bad: Auto-increment ID (monotonic)
sh.shardCollection("mydb.orders", { _id: 1 });

// ✅ Good: Customer ID (high cardinality, even distribution)
sh.shardCollection("mydb.orders", { customerId: "hashed" });

// ✅ Best: Compound key (customerId + orderDate)
sh.shardCollection("mydb.orders", { customerId: 1, orderDate: 1 });
// Allows: Query by customer (single shard) + range queries on date
```

---

**Example 2: Social Media Posts**

```javascript
// ❌ Bad: Post ID (monotonic if auto-increment)
sh.shardCollection("mydb.posts", { _id: 1 });

// ✅ Good: User ID (hash-based)
sh.shardCollection("mydb.posts", { userId: "hashed" });

// Queries:
// Single user's posts → One shard
// All posts (feed) → All shards (but rare query)
```

---

**Example 3: IoT Sensor Data**

```javascript
// Time-series data
// ❌ Bad: Timestamp only (all new data to one shard)
sh.shardCollection("mydb.sensors", { timestamp: 1 });

// ✅ Good: Sensor ID + Timestamp
sh.shardCollection("mydb.sensors", { sensorId: 1, timestamp: 1 });

// Queries:
// Single sensor's data → One shard + efficient range query
// All sensors at time T → All shards (parallel query)
```

---

### Querying Sharded Collections

**Targeted Query (Single Shard):**

```javascript
// Query includes shard key
db.users.find({ userId: "user-123" });
// mongos routes to ONE shard (fast!)
```

**Broadcast Query (All Shards):**

```javascript
// Query doesn't include shard key
db.users.find({ email: "alice@example.com" });
// mongos queries ALL shards, merges results (slow)
```

**Covered Query (Index + Shard Key):**

```javascript
// Create index on email
db.users.createIndex({ email: 1 });

// Query with email but no userId
db.users.find({ email: "alice@example.com" });
// Still broadcast, but can use index on each shard
```

---

### Chunk Management

**What are Chunks?**
- MongoDB splits data into chunks (default 64MB)
- Each chunk contains a range of shard key values
- Chunks distributed across shards

```
Shard 1: Chunks [A-M]
Shard 2: Chunks [N-Z]

Chunk A-D: 64MB
Chunk E-H: 64MB
Chunk I-M: 64MB
...
```

**Balancer:**
- Background process
- Moves chunks between shards to balance load
- Runs automatically (or manually triggered)

```javascript
// Check balancer status
sh.getBalancerState();

// Enable/disable balancer
sh.setBalancerState(true);
sh.setBalancerState(false);

// Manually trigger balancing
sh.startBalancer();
```

---

### Monitoring Sharded Cluster

```javascript
// Cluster status
sh.status();

// Shard distribution
db.users.getShardDistribution();

// Output:
// Shard shard1RS at shard1RS/localhost:27017,localhost:27018
//  data: 150MB docs: 500000 chunks: 5
//  estimated data per chunk: 30MB
//  estimated docs per chunk: 100000
//
// Shard shard2RS at shard2RS/localhost:27022,localhost:27023
//  data: 155MB docs: 520000 chunks: 5
//
// Totals
//  data: 305MB docs: 1020000 chunks: 10

// Check for jumbo chunks (can't be moved)
db.chunks.find({ jumbo: true });
```

---

### Practice Questions: MongoDB Sharding

#### Fill in the Blanks

1. ________ routes queries to the appropriate shards in MongoDB.
2. ________ servers store metadata about which data is on which shard.
3. A good shard key has high ________, even distribution, and is frequently queried.
4. MongoDB splits data into ________, which are distributed across shards.
5. The ________ automatically moves chunks between shards to balance load.

<details>
<summary><strong>View Answers</strong></summary>

1. mongos
2. Config
3. cardinality
4. chunks
5. balancer

</details>

---

#### Coding Challenge: Design Shard Key

**Scenario:** Chat application with:
- 10M users
- 1B messages
- Queries:
  - Get conversation between two users (90% of queries)
  - Get all messages from user (5% of queries)
  - Search all messages (5% of queries)

**Design the optimal shard key and explain your choice.**

<details>
<summary><strong>View Solution</strong></summary>

**Shard Key: Compound (senderId, receiverId) - Hashed**

```javascript
// Create index
db.messages.createIndex({ senderId: 1, receiverId: 1 });

// Shard collection with hashed compound key
sh.shardCollection("chatdb.messages", { 
  senderId: "hashed", 
  receiverId: 1 
});
```

**Rationale:**

**Primary Use Case (90%): Conversation between two users**
```javascript
// Targeted query (single shard)
db.messages.find({
  senderId: "user-123",
  receiverId: "user-456"
}).sort({ timestamp: -1 });

// All messages in conversation on ONE shard
```

**Alternative Considered:**
```javascript
// ❌ Hash(senderId) only
// Problem: Conversation split across multiple shards
// (user A's sent messages on shard 1, user B's sent messages on shard 2)

// ❌ Hash(conversationId)
// Problem: Requires generating conversationId from two userIds
// and maintaining consistency
```

**Trade-offs:**

✅ **Pros:**
- 90% of queries hit single shard (fast!)
- Even distribution (hash on senderId)
- High cardinality (millions of user pairs)

❌ **Cons:**
- "All messages from user" query broadcasts to all shards (5% of queries)
  ```javascript
  // Broadcast query
  db.messages.find({ senderId: "user-123" });
  // Hits all shards
  ```
- Search all messages broadcasts to all shards (5% of queries, acceptable)

**Optimization for "All messages from user" query:**
- Create secondary index on senderId on each shard
- Use aggregation pipeline to parallelize

```javascript
db.messages.createIndex({ senderId: 1, timestamp: -1 });

// Still broadcast, but indexed
db.messages.find({ senderId: "user-123" })
  .sort({ timestamp: -1 })
  .limit(100);
```

**Final Answer:** Compound hash on (senderId, receiverId) optimizes for the 90% use case while accepting broadcast for 10% of queries.

</details>

---

## 5.3.3 Challenges & Solutions

### Challenge 1: Joins Across Shards

**Problem:** Data needed for join is on different shards.

```javascript
// Users sharded by userId
// Orders sharded by orderId

// This query needs data from multiple shards
db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user"
    }
  }
]);
// Extremely slow - mongos must coordinate across shards
```

**Solutions:**

**Solution 1: Denormalize Data**
```javascript
// Store user info in order (no join needed)
{
  _id: ObjectId("order-1"),
  orderId: "ORD-001",
  userId: ObjectId("user-123"),
  
  // Denormalized user data
  userName: "Alice Johnson",
  userEmail: "alice@example.com",
  
  items: [...],
  total: 99.99
}

// Query is fast (single shard, no join)
db.orders.find({ orderId: "ORD-001" });
```

**Solution 2: Co-locate Related Data (Same Shard Key)**
```javascript
// Shard both collections by userId
sh.shardCollection("mydb.users", { userId: 1 });
sh.shardCollection("mydb.orders", { userId: 1 });

// Now user and their orders are on SAME shard
db.orders.aggregate([
  { $match: { userId: ObjectId("user-123") } },
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user"
    }
  }
]);
// Fast - all data on one shard!
```

**Solution 3: Application-Level Join**
```javascript
// Query shards separately, join in application
async function getOrdersWithUsers(orderIds) {
  // Step 1: Get orders
  const orders = await db.orders.find({ 
    _id: { $in: orderIds } 
  }).toArray();
  
  // Step 2: Get unique user IDs
  const userIds = [...new Set(orders.map(o => o.userId))];
  
  // Step 3: Get users
  const users = await db.users.find({ 
    _id: { $in: userIds } 
  }).toArray();
  
  // Step 4: Join in application
  const userMap = new Map(users.map(u => [u._id.toString(), u]));
  
  return orders.map(order => ({
    ...order,
    user: userMap.get(order.userId.toString())
  }));
}
```

---

### Challenge 2: Distributed Transactions

**Problem:** Transaction spans multiple shards.

```javascript
// Transfer money between accounts on different shards
// Account A on shard1, Account B on shard2

// ❌ This won't work atomically across shards (in older MongoDB)
session.startTransaction();
db.accounts.updateOne({ _id: accountA }, { $inc: { balance: -100 } });
db.accounts.updateOne({ _id: accountB }, { $inc: { balance: 100 } });
session.commitTransaction();
```

**Solutions:**

**Solution 1: MongoDB Multi-Document Transactions (4.0+)**
```javascript
// MongoDB 4.0+ supports multi-shard transactions
const session = client.startSession();

try {
  await session.withTransaction(async () => {
    // Deduct from account A (shard1)
    await db.accounts.updateOne(
      { _id: accountA },
      { $inc: { balance: -100 } },
      { session }
    );
    
    // Add to account B (shard2)
    await db.accounts.updateOne(
      { _id: accountB },
      { $inc: { balance: 100 } },
      { session }
    );
  });
  
  console.log('Transaction successful');
} catch (error) {
  console.log('Transaction aborted');
} finally {
  await session.endSession();
}
```

**Limitations:**
- ❌ Performance impact (distributed coordination)
- ❌ Higher latency than single-shard transactions
- ⚠️ Use sparingly

---

**Solution 2: Two-Phase Commit (2PC) Pattern**

```javascript
// Manual implementation for eventual consistency
async function transferMoney(fromAccount, toAccount, amount) {
  const transactionId = new ObjectId();
  
  // Phase 1: Create pending transaction
  await db.transactions.insertOne({
    _id: transactionId,
    from: fromAccount,
    to: toAccount,
    amount: amount,
    state: 'pending',
    createdAt: new Date()
  });
  
  try {
    // Phase 2: Deduct from source
    await db.accounts.updateOne(
      { _id: fromAccount, balance: { $gte: amount } },
      { 
        $inc: { balance: -amount },
        $push: { 
          transactions: { 
            id: transactionId, 
            amount: -amount, 
            state: 'pending' 
          } 
        }
      }
    );
    
    // Phase 3: Add to destination
    await db.accounts.updateOne(
      { _id: toAccount },
      { 
        $inc: { balance: amount },
        $push: { 
          transactions: { 
            id: transactionId, 
            amount: amount, 
            state: 'pending' 
          } 
        }
      }
    );
    
    // Phase 4: Mark transaction complete
    await db.transactions.updateOne(
      { _id: transactionId },
      { 
        $set: { 
          state: 'committed',
          completedAt: new Date()
        } 
      }
    );
    
    return { success: true, transactionId };
    
  } catch (error) {
    // Rollback: Mark as failed and compensate
    await db.transactions.updateOne(
      { _id: transactionId },
      { $set: { state: 'failed', error: error.message } }
    );
    
    // Compensation logic here
    return { success: false, error: error.message };
  }
}

// Background job: Clean up orphaned transactions
async function cleanupOrphanedTransactions() {
  const staleTransactions = await db.transactions.find({
    state: 'pending',
    createdAt: { $lt: new Date(Date.now() - 5 * 60 * 1000) }  // 5 min old
  }).toArray();
  
  for (const txn of staleTransactions) {
    // Rollback logic
    await rollbackTransaction(txn._id);
  }
}
```

---

**Solution 3: Saga Pattern (Eventual Consistency)**

```javascript
// Saga: Sequence of local transactions with compensation
class TransferSaga {
  async execute(fromAccount, toAccount, amount) {
    const sagaId = new ObjectId();
    const steps = [];
    
    try {
      // Step 1: Reserve amount from source
      await this.reserveAmount(fromAccount, amount);
      steps.push({ action: 'reserve', account: fromAccount });
      
      // Step 2: Transfer to destination
      await this.addAmount(toAccount, amount);
      steps.push({ action: 'add', account: toAccount });
      
      // Step 3: Confirm reservation
      await this.confirmReservation(fromAccount, amount);
      steps.push({ action: 'confirm', account: fromAccount });
      
      return { success: true, sagaId };
      
    } catch (error) {
      // Compensate in reverse order
      for (let i = steps.length - 1; i >= 0; i--) {
        await this.compensate(steps[i]);
      }
      
      return { success: false, error: error.message };
    }
  }
  
  async reserveAmount(accountId, amount) {
    await db.accounts.updateOne(
      { _id: accountId, balance: { $gte: amount } },
      { 
        $inc: { reserved: amount },
        $set: { updatedAt: new Date() }
      }
    );
  }
  
  async addAmount(accountId, amount) {
    await db.accounts.updateOne(
      { _id: accountId },
      { $inc: { balance: amount } }
    );
  }
  
  async confirmReservation(accountId, amount) {
    await db.accounts.updateOne(
      { _id: accountId },
      { 
        $inc: { reserved: -amount, balance: -amount }
      }
    );
  }
  
  async compensate(step) {
    switch (step.action) {
      case 'reserve':
        await db.accounts.updateOne(
          { _id: step.account },
          { $inc: { reserved: -step.amount } }
        );
        break;
      case 'add':
        await db.accounts.updateOne(
          { _id: step.account },
          { $inc: { balance: -step.amount } }
        );
        break;
    }
  }
}
```

---

### Challenge 3: Hotspots (Unbalanced Load)

**Problem:** One shard receives disproportionate traffic.

```javascript
// Celebrity user with millions of followers
// All their posts on ONE shard (sharded by userId)

// Shard distribution:
// Shard 1: 100,000 req/sec (celebrity user) 💥
// Shard 2: 1,000 req/sec
// Shard 3: 1,000 req/sec
```

**Solutions:**

**Solution 1: Hash Shard Key**
```javascript
// Instead of range-based (userId), use hash
sh.shardCollection("mydb.posts", { userId: "hashed" });

// Celebrity's posts distributed across shards
// More even load
```

**Solution 2: Compound Shard Key with High Cardinality**
```javascript
// Add timestamp to shard key
sh.shardCollection("mydb.posts", { 
  userId: 1, 
  createdAt: 1 
});

// Celebrity's posts split by time across shards
```

**Solution 3: Application-Level Caching**
```javascript
// Cache celebrity content aggressively
async function getPost(postId, userId) {
  const cacheKey = `post:${postId}`;
  
  // Check if user is celebrity (high follower count)
  const isCelebrity = await isCelebrityUser(userId);
  
  if (isCelebrity) {
    // Cache for 1 hour (longer TTL)
    const cached = await redis.get(cacheKey);
    if (cached) return JSON.parse(cached);
    
    const post = await db.posts.findOne({ _id: postId });
    await redis.setex(cacheKey, 3600, JSON.stringify(post));
    return post;
  } else {
    // Regular cache (5 minutes)
    return await getCachedPost(postId, 300);
  }
}
```

**Solution 4: Read Replicas for Hot Shards**
```javascript
// Add more replicas to hot shard
// Shard 1 (hot): 1 primary + 5 secondaries
// Shard 2: 1 primary + 2 secondaries
// Shard 3: 1 primary + 2 secondaries

// Distribute reads across more replicas for hot shard
```

---

### Challenge 4: Resharding (Changing Shard Key)

**Problem:** Original shard key no longer optimal.

**Solution (MongoDB 5.0+): Reshard Collection**

```javascript
// Change shard key (MongoDB 5.0+)
db.adminCommand({
  reshardCollection: "mydb.users",
  key: { newField: "hashed" }
});

// MongoDB handles data migration automatically
```

**Solution (Manual Migration):**

```javascript
// 1. Create new collection with new shard key
sh.shardCollection("mydb.users_new", { newField: "hashed" });

// 2. Copy data (in batches)
const batchSize = 1000;
let skip = 0;

while (true) {
  const batch = await db.users.find()
    .skip(skip)
    .limit(batchSize)
    .toArray();
  
  if (batch.length === 0) break;
  
  await db.users_new.insertMany(batch);
  skip += batchSize;
}

// 3. Switch application to new collection
// 4. Drop old collection
db.users.drop();

// 5. Rename new collection
db.users_new.renameCollection("users");
```

---

## 5.4 Connection Pooling

### Why Connection Pooling?

**Without Pooling:**
```javascript
// Every request creates new connection
app.get('/users', async (req, res) => {
  const client = new MongoClient(url);
  await client.connect();  // Expensive! (~100ms)
  
  const users = await client.db().collection('users').find().toArray();
  
  await client.close();
  res.json(users);
});

// 100 requests = 100 connections = 10 seconds overhead!
```

**With Pooling:**
```javascript
// Reuse existing connections
const pool = new MongoClient(url, { maxPoolSize: 50 });
await pool.connect();  // Once at startup

app.get('/users', async (req, res) => {
  // Reuse connection from pool (~1ms)
  const users = await pool.db().collection('users').find().toArray();
  res.json(users);
});

// 100 requests = 100ms overhead (100x faster!)
```

---

### PostgreSQL Connection Pooling

**Using pg-pool:**

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  host: 'localhost',
  database: 'mydb',
  user: 'postgres',
  password: 'password',
  
  // Pool configuration
  max: 20,              // Max connections
  min: 5,               // Min connections (kept open)
  idleTimeoutMillis: 30000,  // Close idle after 30s
  connectionTimeoutMillis: 2000,  // Wait 2s for connection
});

// Query (automatically uses pool)
const result = await pool.query('SELECT * FROM users WHERE id = $1', [123]);

// Explicit client checkout (for transactions)
const client = await pool.connect();
try {
  await client.query('BEGIN');
  await client.query('INSERT INTO users ...');
  await client.query('UPDATE accounts ...');
  await client.query('COMMIT');
} catch (error) {
  await client.query('ROLLBACK');
  throw error;
} finally {
  client.release();  // Return to pool
}
```

---

### MongoDB Connection Pooling

```javascript
const { MongoClient } = require('mongodb');

const client = new MongoClient(url, {
  maxPoolSize: 50,          // Max connections
  minPoolSize: 10,          // Min connections
  maxIdleTimeMS: 30000,     // Close idle after 30s
  waitQueueTimeoutMS: 5000, // Wait 5s for available connection
  
  // Monitoring
  monitorCommands: true
});

await client.connect();

// Pool events
client.on('connectionPoolCreated', (event) => {
  console.log('Pool created:', event);
});

client.on('connectionCheckedOut', (event) => {
  console.log('Connection checked out');
});

client.on('connectionCheckedIn', (event) => {
  console.log('Connection returned to pool');
});

// Query (uses pool automatically)
const users = await client.db('mydb').collection('users').find().toArray();
```

---

### Connection Pool Sizing

**Formula:**
```
Pool Size = (Core Count × 2) + Effective Spindle Count

Examples:
- 4 cores, SSD → (4 × 2) + 1 = 9 connections
- 8 cores, SSD → (8 × 2) + 1 = 17 connections
- 16 cores, RAID → (16 × 2) + 10 = 42 connections
```

**Guidelines:**
- **Too small:** Requests wait for connections (slow)
- **Too large:** Database overload, memory waste
- **Start conservative:** 10-20 connections
- **Monitor and adjust:** Based on load

---

### External Connection Poolers

**PgBouncer (PostgreSQL):**

```ini
# pgbouncer.ini
[databases]
mydb = host=localhost dbname=mydb

[pgbouncer]
listen_addr = 0.0.0.0
listen_port = 6432
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt

# Pool modes:
pool_mode = transaction  # Connection per transaction (recommended)
# pool_mode = session    # Connection per session (default)
# pool_mode = statement  # Connection per statement (rare)

max_client_conn = 1000   # Max client connections
default_pool_size = 25   # Connections per database
```

**Benefits:**
- ✅ Handles 1000s of client connections
- ✅ Actual DB connections: Only 25
- ✅ Reduces database load
- ✅ Connection reuse across clients

---

## 5.5 Load Balancing

### What is Load Balancing?

**Definition:** Distribute traffic across multiple servers.

```
           ┌──────────────┐
Clients ───│ Load Balancer│
           └──────┬───────┘
                  │
      ┌───────────┼───────────┐
      ▼           ▼           ▼
  ┌──────┐    ┌──────┐    ┌──────┐
  │Server│    │Server│    │Server│
  │  1   │    │  2   │    │  3   │
  └──────┘    └──────┘    └──────┘
```

---

### Load Balancing Algorithms

#### 1. Round Robin

```javascript
class RoundRobinBalancer {
  constructor(servers) {
    this.servers = servers;
    this.current = 0;
  }
  
  getNextServer() {
    const server = this.servers[this.current];
    this.current = (this.current + 1) % this.servers.length;
    return server;
  }
}

// Usage
const lb = new RoundRobinBalancer([
  'server1.example.com',
  'server2.example.com',
  'server3.example.com'
]);

console.log(lb.getNextServer());  // server1
console.log(lb.getNextServer());  // server2
console.log(lb.getNextServer());  // server3
console.log(lb.getNextServer());  // server1 (cycles)
```

**Pros:** ✅ Simple, even distribution
**Cons:** ❌ Ignores server load

---

#### 2. Least Connections

```javascript
class LeastConnectionsBalancer {
  constructor(servers) {
    this.servers = servers.map(server => ({
      url: server,
      connections: 0
    }));
  }
  
  getNextServer() {
    // Find server with fewest connections
    const server = this.servers.reduce((min, curr) => 
      curr.connections < min.connections ? curr : min
    );
    
    server.connections++;
    return server.url;
  }
  
  releaseConnection(serverUrl) {
    const server = this.servers.find(s => s.url === serverUrl);
    if (server) server.connections--;
  }
}

// Usage
const lb = new LeastConnectionsBalancer([...servers]);
const server = lb.getNextServer();
// ... use connection
lb.releaseConnection(server);
```

**Pros:** ✅ Accounts for server load
**Cons:** ❌ More complex

---

#### 3. Weighted Round Robin

```javascript
class WeightedRoundRobinBalancer {
  constructor(servers) {
    // servers: [{ url, weight }, ...]
    this.servers = [];
    
    // Expand based on weights
    servers.forEach(server => {
      for (let i = 0; i < server.weight; i++) {
        this.servers.push(server.url);
      }
    });
    
    this.current = 0;
  }
  
  getNextServer() {
    const server = this.servers[this.current];
    this.current = (this.current + 1) % this.servers.length;
    return server;
  }
}

// Usage: More powerful server gets more traffic
const lb = new WeightedRoundRobinBalancer([
  { url: 'server1.example.com', weight: 3 },  // 3x traffic
  { url: 'server2.example.com', weight: 2 },  // 2x traffic
  { url: 'server3.example.com', weight: 1 }   // 1x traffic
]);
```

**Pros:** ✅ Accounts for server capacity
**Cons:** ❌ Static weights

---

#### 4. IP Hash (Session Affinity)

```javascript
class IPHashBalancer {
  constructor(servers) {
    this.servers = servers;
  }
  
  hash(ip) {
    let hash = 0;
    for (let i = 0; i < ip.length; i++) {
      hash = ((hash << 5) - hash) + ip.charCodeAt(i);
      hash = hash & hash;
    }
    return Math.abs(hash);
  }
  
  getServerForIP(clientIP) {
    const hash = this.hash(clientIP);
    const index = hash % this.servers.length;
    return this.servers[index];
  }
}

// Usage: Same client always goes to same server
const lb = new IPHashBalancer([...servers]);
console.log(lb.getServerForIP('192.168.1.100'));  // server2
console.log(lb.getServerForIP('192.168.1.100'));  // server2 (always)
console.log(lb.getServerForIP('192.168.1.101'));  // server1
```

**Pros:** ✅ Session persistence (sticky sessions)
**Cons:** ❌ Uneven distribution

---

### Database Load Balancing

**HAProxy Configuration (PostgreSQL):**

```
# haproxy.cfg
global
    maxconn 4096

defaults
    mode tcp
    timeout connect 5s
    timeout client 50s
    timeout server 50s

# PostgreSQL primary (writes)
frontend postgres_primary
    bind *:5432
    default_backend postgres_primary_backend

backend postgres_primary_backend
    server primary primary-db.example.com:5432 check

# PostgreSQL replicas (reads)
frontend postgres_replicas
    bind *:5433
    default_backend postgres_replicas_backend

backend postgres_replicas_backend
    balance roundrobin
    server replica1 replica1-db.example.com:5432 check
    server replica2 replica2-db.example.com:5432 check
    server replica3 replica3-db.example.com:5432 check
```

**Application Code:**

```javascript
const primaryPool = new Pool({
  host: 'haproxy.example.com',
  port: 5432  // Primary (writes)
});

const replicaPool = new Pool({
  host: 'haproxy.example.com',
  port: 5433  // Replicas (reads)
});

// Write
await primaryPool.query('INSERT INTO users ...');

// Read (distributed across replicas by HAProxy)
await replicaPool.query('SELECT * FROM users WHERE ...');
```

---

## 5.6 CAP Theorem

### The CAP Theorem

**Definition:** In a distributed system, you can only have 2 of 3:

- **C**onsistency: All nodes see same data
- **A**vailability: System always responds
- **P**artition Tolerance: Works despite network failures

```
     Consistency
         /\
        /  \
       /    \
      /  CA  \
     /________\
    /    |     \
   / CP  |  AP  \
  /______|_______\
Partition     Availability
Tolerance
```

---

### Understanding with Examples

#### CA (Consistency + Availability)

**Example:** Traditional single-server database

```
Client → [Single Database] → Always consistent, always available

BUT: No partition tolerance
- If server fails → System down
- Not distributed → No P
```

**Real Systems:**
- PostgreSQL (single instance)
- MySQL (single instance)

**Trade-off:** ❌ No fault tolerance

---

#### CP (Consistency + Partition Tolerance)

**Example:** MongoDB with majority write concern

```
Client → [Primary] ← Replicates → [Secondary 1]
                                  [Secondary 2]

Network partition occurs:
Primary isolated from secondaries

Result:
- Primary refuses writes (can't reach majority) ← Unavailable
- Ensures consistency (no split brain)
- Partition tolerant
```

**Real Systems:**
- HBase
- MongoDB (w: 'majority')
- Redis Cluster
- Zookeeper

**Trade-off:** ❌ May become unavailable during partitions

---

#### AP (Availability + Partition Tolerance)

**Example:** Cassandra

```
Client → [Node 1] ← Eventually → [Node 2]
         [Node 2]    consistent  [Node 3]

Network partition occurs:
Nodes 1-2 isolated from Node 3

Result:
- All nodes still accept writes ← Available
- Data eventually consistent ← May be temporarily inconsistent
- Partition tolerant
```

**Real Systems:**
- Cassandra
- DynamoDB
- Riak
- CouchDB

**Trade-off:** ❌ Temporary inconsistency

---

### CAP Theorem Decision Guide

```
Choose based on requirements:

Need strong consistency? (Financial, Inventory)
├─ Single datacenter → CA (PostgreSQL)
└─ Multi-datacenter → CP (MongoDB w: majority)

Need high availability? (Social media, Analytics)
└─ AP (Cassandra, DynamoDB)

Typical choices:
- Banking → CP (consistency critical)
- E-commerce → CP for orders, AP for catalog
- Social media → AP (eventual consistency OK)
- Analytics → AP (availability + eventual consistency)
```

---

### Real-World Example: E-Commerce

```javascript
// Product Catalog (AP)
// - Cassandra (availability + partition tolerance)
// - Eventual consistency OK (price updates can lag slightly)
const catalog = cassandraClient.query(
  'SELECT * FROM products WHERE category = ?',
  ['electronics']
);

// Shopping Cart (CP)
// - MongoDB with majority write concern
// - Consistency critical (can't lose cart items)
const cart = await mongoClient.db('shop').collection('carts').findOne(
  { userId: 'user-123' },
  { readConcern: { level: 'majority' } }
);

// Order Processing (CP)
// - PostgreSQL (ACID transactions)
// - Strong consistency required
await pgPool.query('BEGIN');
await pgPool.query('INSERT INTO orders ...');
await pgPool.query('UPDATE inventory ...');
await pgPool.query('COMMIT');

// Analytics (AP)
// - Elasticsearch
// - Eventual consistency acceptable
const analytics = await esClient.search({
  index: 'orders',
  body: {
    query: { range: { date: { gte: '2024-01-01' } } },
    aggs: { total_revenue: { sum: { field: 'amount' } } }
  }
});
```

---

### Practice Questions: CAP Theorem

#### Multiple Choice

**Q1: Which system prioritizes availability over consistency?**

A) PostgreSQL single instance
B) MongoDB with w: 'majority'
C) Cassandra
D) Single Redis server

<details>
<summary><strong>View Answer</strong></summary>

**Answer: C - Cassandra**

Explanation: Cassandra is an AP system that prioritizes availability and partition tolerance, accepting eventual consistency.

</details>

---

**Q2: A banking system requires strong consistency and must work across multiple datacenters. Which CAP combination should it choose?**

A) CA
B) CP
C) AP
D) All three

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - CP**

Explanation: Banking requires strong consistency (C) and must handle network partitions across datacenters (P). It can tolerate temporary unavailability during partitions to ensure consistency.

</details>

---

## Chapter Summary

### Key Concepts Covered

#### 5.1 Vertical vs Horizontal Scaling

**Vertical (Scale Up):**
- Add resources to single server
- Simple but limited
- Expensive at scale
- Single point of failure

**Horizontal (Scale Out):**
- Add more servers
- Complex but unlimited
- Cost-effective
- High availability

**Best Practice:** Hybrid approach

---

#### 5.2 Replication (Read Scaling)

**SQL Replication:**
- PostgreSQL: Streaming (physical), Logical (table-level)
- MySQL: Master-slave, Group replication
- Read/write split for scaling

**MongoDB Replica Sets:**
- Minimum 3 members
- Automatic failover (<10 seconds)
- 5 read preferences
- Write concerns (w: 1, w: 'majority')

**Replication Lag:**
- Causes: Network latency, load, distance
- Consistency models: Strong, Eventual, Read-after-write
- Solutions: Staleness windows, primary routing, versions

---

#### 5.3 Sharding (Write Scaling)

**Sharding Strategies:**
1. **Range-based:** Partition by ranges (hotspots possible)
2. **Hash-based:** Even distribution (range queries slow)
3. **Geographic/Directory:** Data locality (uneven distribution)
4. **Consistent hashing:** Minimal data movement (complex)

**MongoDB Sharding:**
- Components: mongos, config servers, shards
- Shard key selection: High cardinality, even distribution, non-monotonic
- Targeted vs broadcast queries
- Chunk balancing

**Challenges:**
- Cross-shard joins → Denormalize or co-locate
- Distributed transactions → Use MongoDB 4.0+, 2PC, or Saga
- Hotspots → Hash keys, caching, more replicas
- Resharding → MongoDB 5.0+ or manual migration

---

#### 5.4 Connection Pooling

**Benefits:**
- Reuse connections (100x faster)
- Reduce overhead
- Control resource usage

**Configuration:**
- Size: (Cores × 2) + Spindles
- Monitoring: Checkout/checkin events
- External poolers: PgBouncer (PostgreSQL)

---

#### 5.5 Load Balancing

**Algorithms:**
1. Round Robin (simple, even)
2. Least Connections (accounts for load)
3. Weighted (server capacity)
4. IP Hash (sticky sessions)

**Database LB:**
- HAProxy for PostgreSQL
- Separate ports for primary/replicas
- Application-aware routing

---

#### 5.6 CAP Theorem

**Trade-offs:**
- **CA:** Single server (PostgreSQL, MySQL)
- **CP:** Consistency + Partition tolerance (MongoDB, HBase)
- **AP:** Availability + Partition tolerance (Cassandra, DynamoDB)

**Decision:**
- Strong consistency → CP
- High availability → AP
- Single datacenter → CA

---

### Scaling Progression

```
Stage 1: Single Server (Vertical)
- 1 database server
- Optimize queries, add indexes, cache

Stage 2: Read Scaling (Horizontal Reads)
- 1 primary + N replicas
- Read/write split
- Connection pooling

Stage 3: Geographic Distribution
- Multi-region replicas
- Load balancing
- CDN for static content

Stage 4: Write Scaling (Sharding)
- Multiple shards
- Each shard is replica set
- mongos routing

Stage 5: Full Distribution
- Sharding + Replication
- Multi-region shards
- Advanced caching
- Microservices
```

---

### Performance Metrics

| Technique | Read Scaling | Write Scaling | Availability |
|-----------|--------------|---------------|--------------|
| **Vertical Scaling** | ✅ 2-4x | ✅ 2-4x | ❌ Single point |
| **Replication** | ✅ Nx (N replicas) | ❌ No improvement | ✅ Automatic failover |
| **Sharding** | ✅ Nx (N shards) | ✅ Nx (N shards) | ✅ Shard-level fault tolerance |
| **Caching** | ✅ 10-100x | ❌ No improvement | ✅ Reduces DB load |
| **Connection Pooling** | ✅ 100x (overhead) | ✅ 100x (overhead) | ✅ Efficient resources |

---

### Best Practices

**Scaling:**
1. ✅ Start simple (vertical + caching)
2. ✅ Add replication before sharding
3. ✅ Monitor and measure before scaling
4. ✅ Use hybrid approach
5. ❌ Don't premature optimize

**Replication:**
1. ✅ Use read replicas for read-heavy workloads
2. ✅ Monitor replication lag
3. ✅ Use appropriate consistency model
4. ✅ Automate failover
5. ❌ Don't ignore lag in critical paths

**Sharding:**
1. ✅ Choose shard key carefully (hard to change)
2. ✅ High cardinality, even distribution
3. ✅ Denormalize for co-location
4. ✅ Monitor shard balance
5. ❌ Don't shard too early (adds complexity)

**Connection Management:**
1. ✅ Use connection pooling
2. ✅ Size pools appropriately
3. ✅ Monitor pool saturation
4. ✅ Consider external poolers (PgBouncer)
5. ❌ Don't create connections per request

---

**Next Chapter:** [Production Best Practices →](06_production.md)