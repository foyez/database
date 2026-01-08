# Chapter 3: NoSQL Deep Dive
**Redis Operations, Patterns, and Best Practices**

---

## Table of Contents
- [3.2 Redis](#32-redis)
  - [3.2.1 Data Structures](#321-data-structures-string-hash-list-set-sorted-set)
  - [3.2.2 Caching Patterns](#322-caching-patterns)
  - [3.2.3 Pub/Sub Messaging](#323-pubsub-messaging)
  - [3.2.4 Expiration & TTL](#324-expiration--ttl)
  - [3.2.5 Use Cases & Best Practices](#325-use-cases--best-practices)

---

## 3.2 Redis

### What is Redis?

**Redis** (Remote Dictionary Server) is an in-memory key-value NoSQL database known for its blazing-fast performance and rich data structures.

**Key Features:**
- In-memory storage (< 1ms latency)
- Multiple data structures (String, Hash, List, Set, Sorted Set)
- Persistence options (RDB snapshots, AOF logging)
- Pub/Sub messaging
- TTL (expiration) support
- Atomic operations
- Lua scripting

**When to Use Redis:**
- Caching
- Session storage
- Real-time analytics
- Leaderboards
- Rate limiting
- Message queues
- Real-time features

---

### 3.2.1 Data Structures (String, Hash, List, Set, Sorted Set)

Redis supports five main data structures, each optimized for specific use cases.

---

#### 1. String (Simple Key-Value)

**Most basic data type:** Stores text or binary data up to 512MB.

**Commands:**

```bash
# SET - Set key to value
SET user:1:name "Alice Johnson"
# OK

# GET - Get value by key
GET user:1:name
# "Alice Johnson"

# MSET - Set multiple keys
MSET user:1:email "alice@example.com" user:1:age "28"
# OK

# MGET - Get multiple keys
MGET user:1:name user:1:email user:1:age
# 1) "Alice Johnson"
# 2) "alice@example.com"
# 3) "28"

# INCR - Increment number
SET page:views 100
INCR page:views
# 101

# INCRBY - Increment by amount
INCRBY page:views 10
# 111

# DECR, DECRBY - Decrement
DECR page:views
# 110

# APPEND - Append to string
SET greeting "Hello"
APPEND greeting " World"
# 11 (new length)
GET greeting
# "Hello World"

# SETEX - Set with expiration (seconds)
SETEX session:abc123 3600 "user-data"
# OK (expires in 1 hour)

# SETNX - Set only if not exists
SETNX user:1:name "Bob"
# 0 (failed, key exists)

# EXISTS - Check if key exists
EXISTS user:1:name
# 1 (exists)

# DEL - Delete key
DEL user:1:age
# 1 (deleted)
```

**Use Cases:**
- Simple caching
- Counters (page views, likes)
- Session tokens
- Feature flags

---

#### 2. Hash (Field-Value Pairs)

**Think of it as:** A mini key-value store within a key (like a JavaScript object).

**Commands:**

```bash
# HSET - Set field in hash
HSET user:1 name "Alice Johnson"
# 1 (field created)

# HGET - Get field from hash
HGET user:1 name
# "Alice Johnson"

# HMSET - Set multiple fields
HMSET user:1 email "alice@example.com" age 28 city "New York"
# OK

# HGETALL - Get all fields and values
HGETALL user:1
# 1) "name"
# 2) "Alice Johnson"
# 3) "email"
# 4) "alice@example.com"
# 5) "age"
# 6) "28"
# 7) "city"
# 8) "New York"

# HMGET - Get multiple fields
HMGET user:1 name email
# 1) "Alice Johnson"
# 2) "alice@example.com"

# HDEL - Delete field
HDEL user:1 city
# 1 (deleted)

# HEXISTS - Check if field exists
HEXISTS user:1 name
# 1 (exists)

# HINCRBY - Increment field value
HSET product:1 views 100
HINCRBY product:1 views 1
# 101

# HKEYS - Get all field names
HKEYS user:1
# 1) "name"
# 2) "email"
# 3) "age"

# HVALS - Get all values
HVALS user:1
# 1) "Alice Johnson"
# 2) "alice@example.com"
# 3) "28"

# HLEN - Count fields
HLEN user:1
# 3
```

**Use Cases:**
- User profiles
- Product details
- Configuration settings
- Shopping carts

**Real-World Example: User Session**

```bash
# Store user session
HMSET session:abc123 \
  userId "507f1f77bcf86cd799439011" \
  username "alice" \
  email "alice@example.com" \
  loginTime "2024-01-15T10:00:00Z" \
  lastActivity "2024-01-15T10:30:00Z"

# Update last activity
HSET session:abc123 lastActivity "2024-01-15T10:35:00Z"

# Get session data
HGETALL session:abc123

# Set expiration on entire session
EXPIRE session:abc123 3600  # 1 hour
```

---

#### 3. List (Ordered Collection)

**Think of it as:** A linked list - can add/remove from both ends efficiently.

**Commands:**

```bash
# LPUSH - Add to left (head)
LPUSH queue:tasks "task1"
# 1

# RPUSH - Add to right (tail)
RPUSH queue:tasks "task2" "task3"
# 3 (total length)

# LRANGE - Get range of elements
LRANGE queue:tasks 0 -1  # All elements
# 1) "task1"
# 2) "task2"
# 3) "task3"

LRANGE queue:tasks 0 1   # First 2 elements
# 1) "task1"
# 2) "task2"

# LPOP - Remove from left
LPOP queue:tasks
# "task1"

# RPOP - Remove from right
RPOP queue:tasks
# "task3"

# LLEN - Get length
LLEN queue:tasks
# 1 (only "task2" left)

# LINDEX - Get element by index
RPUSH mylist "a" "b" "c"
LINDEX mylist 1
# "b"

# LSET - Set element by index
LSET mylist 1 "B"
# OK

# LTRIM - Keep only specified range
RPUSH numbers 1 2 3 4 5
LTRIM numbers 0 2  # Keep first 3
LRANGE numbers 0 -1
# 1) "1"
# 2) "2"
# 3) "3"

# BLPOP - Blocking pop (wait for element)
BLPOP queue:tasks 10  # Wait up to 10 seconds
```

**Use Cases:**
- Message queues
- Activity feeds
- Recent items list
- Notifications

**Real-World Example: Activity Feed**

```bash
# Add activities to user's feed
LPUSH feed:user:1 "Alice liked your post"
LPUSH feed:user:1 "Bob commented on your photo"
LPUSH feed:user:1 "Carol started following you"

# Get latest 10 activities
LRANGE feed:user:1 0 9

# Keep only last 100 activities
LTRIM feed:user:1 0 99

# Count unread
LLEN feed:user:1
```

---

#### 4. Set (Unique Unordered Collection)

**Think of it as:** A collection of unique elements (no duplicates).

**Commands:**

```bash
# SADD - Add members to set
SADD tags:post:1 "mongodb" "nosql" "database"
# 3 (members added)

# SADD duplicate (ignored)
SADD tags:post:1 "mongodb"
# 0 (already exists)

# SMEMBERS - Get all members
SMEMBERS tags:post:1
# 1) "mongodb"
# 2) "nosql"
# 3) "database"

# SISMEMBER - Check membership
SISMEMBER tags:post:1 "mongodb"
# 1 (exists)

# SREM - Remove member
SREM tags:post:1 "database"
# 1 (removed)

# SCARD - Count members
SCARD tags:post:1
# 2

# SPOP - Remove random member
SPOP tags:post:1
# "nosql" (random)

# Set operations
SADD set1 "a" "b" "c"
SADD set2 "b" "c" "d"

# SINTER - Intersection (common elements)
SINTER set1 set2
# 1) "b"
# 2) "c"

# SUNION - Union (all elements)
SUNION set1 set2
# 1) "a"
# 2) "b"
# 3) "c"
# 4) "d"

# SDIFF - Difference (in set1 but not set2)
SDIFF set1 set2
# 1) "a"

# SINTERSTORE - Store intersection in new set
SINTERSTORE common set1 set2
# 2 (stored 2 elements)
```

**Use Cases:**
- Tags/categories
- Online users
- Unique visitors
- Followers/following
- Vote tracking

**Real-World Example: Unique Visitors**

```bash
# Track unique visitors per day
SADD visitors:2024-01-15 "user:1" "user:2" "user:3"
SADD visitors:2024-01-16 "user:2" "user:3" "user:4"

# Count unique visitors today
SCARD visitors:2024-01-15
# 3

# Users who visited both days
SINTER visitors:2024-01-15 visitors:2024-01-16
# 1) "user:2"
# 2) "user:3"

# Total unique visitors (either day)
SUNION visitors:2024-01-15 visitors:2024-01-16
# 1) "user:1"
# 2) "user:2"
# 3) "user:3"
# 4) "user:4"
```

---

#### 5. Sorted Set (Sorted Unique Collection)

**Think of it as:** A set where each member has a score for sorting.

**Commands:**

```bash
# ZADD - Add member with score
ZADD leaderboard 1000 "alice" 850 "bob" 950 "carol"
# 3 (members added)

# ZRANGE - Get range by rank (ascending)
ZRANGE leaderboard 0 -1
# 1) "bob"    (850)
# 2) "carol"  (950)
# 3) "alice"  (1000)

# ZRANGE with scores
ZRANGE leaderboard 0 -1 WITHSCORES
# 1) "bob"
# 2) "850"
# 3) "carol"
# 4) "950"
# 5) "alice"
# 6) "1000"

# ZREVRANGE - Get range (descending)
ZREVRANGE leaderboard 0 2 WITHSCORES
# 1) "alice"
# 2) "1000"
# 3) "carol"
# 4) "950"
# 5) "bob"
# 6) "850"

# ZRANK - Get member rank (0-indexed)
ZRANK leaderboard "bob"
# 0 (lowest score)

# ZREVRANK - Get reverse rank
ZREVRANK leaderboard "alice"
# 0 (highest score)

# ZSCORE - Get member score
ZSCORE leaderboard "alice"
# "1000"

# ZINCRBY - Increment score
ZINCRBY leaderboard 50 "bob"
# "900"

# ZCOUNT - Count members in score range
ZCOUNT leaderboard 900 1000
# 2 (bob and alice)

# ZRANGEBYSCORE - Get members by score range
ZRANGEBYSCORE leaderboard 900 1000 WITHSCORES
# 1) "bob"
# 2) "900"
# 3) "carol"
# 4) "950"
# 5) "alice"
# 6) "1000"

# ZREM - Remove member
ZREM leaderboard "bob"
# 1 (removed)

# ZCARD - Count members
ZCARD leaderboard
# 2
```

**Use Cases:**
- Leaderboards
- Priority queues
- Rate limiting (timestamp as score)
- Time-series data
- Trending posts

**Real-World Example: Gaming Leaderboard**

```bash
# Add player scores
ZADD game:leaderboard 15000 "player1" 12000 "player2" 18000 "player3"

# Get top 10 players
ZREVRANGE game:leaderboard 0 9 WITHSCORES

# Get player rank
ZREVRANK game:leaderboard "player1"
# 1 (2nd place)

# Update score after game
ZINCRBY game:leaderboard 500 "player1"
# "15500"

# Get players within score range
ZRANGEBYSCORE game:leaderboard 15000 20000 WITHSCORES

# Get player's neighbors (above and below)
ZREVRANK game:leaderboard "player1"  # Get rank
# Then get rank-1 and rank+1
```

---

### Practice Questions: Redis Data Structures

#### Fill in the Blanks

1. Redis ________ stores simple key-value pairs.
2. A Redis ________ is like a JavaScript object with field-value pairs.
3. Redis ________ maintains an ordered collection where you can add/remove from both ends.
4. Redis ________ stores unique elements without duplicates.
5. Redis ________ ________ stores unique elements with scores for sorting.

<details>
<summary><strong>View Answers</strong></summary>

1. String
2. Hash
3. List
4. Set
5. Sorted Set

</details>

---

#### True/False

1. Redis Strings can store up to 512MB of data.
2. Adding a duplicate element to a Set creates a second copy.
3. Lists in Redis can be used as message queues.
4. Sorted Sets automatically sort members by their scores.
5. Hashes can store nested objects directly.

<details>
<summary><strong>View Answers</strong></summary>

1. TRUE
2. FALSE (Sets store unique elements)
3. TRUE
4. TRUE
5. FALSE (Hashes store flat field-value pairs; serialize nested objects as JSON strings)

</details>

---

## 3.2.2 Caching Patterns

**Caching** is the most common use case for Redis. It stores frequently accessed data in memory to reduce database load and improve response times.

---

#### Pattern 1: Cache-Aside (Lazy Loading)

**Most common pattern:** Application checks cache first, loads from DB on miss, then updates cache.

```javascript
// Pseudocode
async function getUser(userId) {
  // 1. Try cache first
  const cached = await redis.get(`user:${userId}`);
  
  if (cached) {
    return JSON.parse(cached);  // Cache hit!
  }
  
  // 2. Cache miss - load from database
  const user = await db.users.findOne({ _id: userId });
  
  // 3. Update cache for next time
  await redis.setex(
    `user:${userId}`,
    3600,  // TTL: 1 hour
    JSON.stringify(user)
  );
  
  return user;
}
```

**Advantages:**
- ✅ Only caches requested data
- ✅ Simple to implement
- ✅ Resilient (cache failure doesn't break app)

**Disadvantages:**
- ❌ Cache miss penalty (extra delay on first request)
- ❌ Stale data possible (if DB updated externally)

---

#### Pattern 2: Write-Through Cache

**Pattern:** Update cache and database simultaneously on write.

```javascript
async function updateUser(userId, updates) {
  // 1. Update database
  await db.users.updateOne({ _id: userId }, { $set: updates });
  
  // 2. Update cache immediately
  const user = await db.users.findOne({ _id: userId });
  await redis.setex(
    `user:${userId}`,
    3600,
    JSON.stringify(user)
  );
  
  return user;
}
```

**Advantages:**
- ✅ Cache always up-to-date
- ✅ No stale data

**Disadvantages:**
- ❌ Write latency (two operations)
- ❌ Caches data even if not accessed

---

#### Pattern 3: Write-Behind Cache (Write-Back)

**Pattern:** Update cache immediately, update database asynchronously.

```javascript
async function updateUser(userId, updates) {
  // 1. Update cache immediately
  const cachedUser = JSON.parse(await redis.get(`user:${userId}`));
  const updatedUser = { ...cachedUser, ...updates };
  
  await redis.setex(
    `user:${userId}`,
    3600,
    JSON.stringify(updatedUser)
  );
  
  // 2. Queue database update (async)
  await queue.add('updateUser', { userId, updates });
  
  return updatedUser;
}

// Background worker processes queue
async function processUserUpdate(job) {
  await db.users.updateOne(
    { _id: job.userId },
    { $set: job.updates }
  );
}
```

**Advantages:**
- ✅ Fast writes (no wait for DB)
- ✅ Batch database updates

**Disadvantages:**
- ❌ Complex implementation
- ❌ Risk of data loss (cache failure before DB write)

---

#### Pattern 4: Refresh-Ahead

**Pattern:** Proactively refresh cache before expiration.

```javascript
async function getUser(userId) {
  const cached = await redis.get(`user:${userId}`);
  const ttl = await redis.ttl(`user:${userId}`);
  
  if (cached) {
    // Refresh if TTL < 10% of original (e.g., < 6 minutes of 1 hour)
    if (ttl < 360) {
      // Refresh asynchronously
      refreshUserCache(userId);  // Don't await
    }
    
    return JSON.parse(cached);
  }
  
  // Cache miss - load and cache
  return await loadAndCacheUser(userId);
}

async function refreshUserCache(userId) {
  const user = await db.users.findOne({ _id: userId });
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));
}
```

**Advantages:**
- ✅ No cache miss penalty for frequently accessed data
- ✅ Always fast responses

**Disadvantages:**
- ❌ More complex
- ❌ May refresh unused data

---

#### Cache Invalidation Strategies

**Problem:** How to keep cache in sync with database?

**Strategy 1: TTL (Time-To-Live)**

```javascript
// Expire after fixed time
await redis.setex('user:1', 3600, userData);  // 1 hour

// Good for: Slowly changing data
```

**Strategy 2: Explicit Invalidation**

```javascript
async function updateUser(userId, updates) {
  await db.users.updateOne({ _id: userId }, { $set: updates });
  
  // Invalidate cache
  await redis.del(`user:${userId}`);
}

// Good for: Critical data consistency
```

**Strategy 3: Event-Driven Invalidation**

```javascript
// Listen for database changes
db.users.watch().on('change', async (change) => {
  if (change.operationType === 'update') {
    await redis.del(`user:${change.documentKey._id}`);
  }
});

// Good for: Real-time updates
```

---

### Real-World Caching Example: E-Commerce Product Cache

```javascript
// Product cache with multiple strategies
class ProductCache {
  
  // Cache-aside for product details
  async getProduct(productId) {
    const cacheKey = `product:${productId}`;
    
    // Try cache
    const cached = await redis.hgetall(cacheKey);
    if (Object.keys(cached).length > 0) {
      return cached;
    }
    
    // Load from DB
    const product = await db.products.findOne({ _id: productId });
    
    // Cache as hash
    await redis.hmset(cacheKey, product);
    await redis.expire(cacheKey, 7200);  // 2 hours
    
    return product;
  }
  
  // Write-through for inventory updates (critical)
  async updateInventory(productId, quantity) {
    // Update DB
    await db.products.updateOne(
      { _id: productId },
      { $inc: { stock: -quantity } }
    );
    
    // Update cache immediately
    await redis.hincrby(`product:${productId}`, 'stock', -quantity);
    
    // Also invalidate product listings cache
    await redis.del('products:list:*');
  }
  
  // Cache product listings with pagination
  async getProducts(page = 1, limit = 20) {
    const cacheKey = `products:list:${page}:${limit}`;
    
    const cached = await redis.get(cacheKey);
    if (cached) {
      return JSON.parse(cached);
    }
    
    const products = await db.products
      .find({ status: 'active' })
      .skip((page - 1) * limit)
      .limit(limit)
      .toArray();
    
    // Cache for 5 minutes
    await redis.setex(cacheKey, 300, JSON.stringify(products));
    
    return products;
  }
  
  // Invalidate all product caches
  async invalidateAll() {
    const keys = await redis.keys('product:*');
    if (keys.length > 0) {
      await redis.del(...keys);
    }
  }
}
```

---

### Practice Questions: Caching Patterns

#### Multiple Choice

**Q1: Which caching pattern checks cache first, then loads from DB on miss?**

A) Write-Through
B) Write-Behind
C) Cache-Aside
D) Refresh-Ahead

<details>
<summary><strong>View Answer</strong></summary>

**Answer: C - Cache-Aside**

Explanation: Cache-Aside (Lazy Loading) is the most common pattern where the application checks the cache first, and loads from the database only on cache miss.

</details>

---

**Q2: What is the main advantage of Write-Behind caching?**

A) Cache always up-to-date
B) Fast writes (no wait for DB)
C) Simple to implement
D) No risk of data loss

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - Fast writes (no wait for DB)**

Explanation: Write-Behind caching updates the cache immediately and queues database updates asynchronously, making writes fast but introducing complexity and data loss risk.

</details>

---

# Chapter 3: NoSQL Deep Dive - PART 5 (FINAL)
**Redis: Pub/Sub, TTL, Use Cases, Best Practices & Chapter Summary**

---

## 3.2.3 Pub/Sub Messaging

**Redis Pub/Sub** enables message broadcasting between publishers and subscribers in real-time.

**How it works:**
```
Publisher → Channel → Subscribers
```

---

#### Basic Pub/Sub Commands

**Subscribe to Channel:**

```bash
# Terminal 1: Subscribe to channel
SUBSCRIBE news
# Waiting for messages...

# Terminal 2: Publish message
PUBLISH news "Breaking: MongoDB 7.0 released!"
# (integer) 1  (1 subscriber received)

# Terminal 1 receives:
# 1) "message"
# 2) "news"
# 3) "Breaking: MongoDB 7.0 released!"
```

**Pattern Subscription:**

```bash
# Subscribe to pattern
PSUBSCRIBE news:*  # All channels starting with "news:"

# Publish to different channels
PUBLISH news:tech "New tech article"
PUBLISH news:sports "Game results"
PUBLISH news:weather "Sunny today"

# Subscriber receives all three
```

**Unsubscribe:**

```bash
UNSUBSCRIBE news
PUNSUBSCRIBE news:*
```

---

#### Real-World Example: Chat Application

```javascript
// Server setup
const redis = require('redis');
const publisher = redis.createClient();
const subscriber = redis.createClient();

// Subscribe to chat room
subscriber.subscribe('chat:room:1');

subscriber.on('message', (channel, message) => {
  const data = JSON.parse(message);
  console.log(`[${data.user}]: ${data.text}`);
  
  // Broadcast to all connected WebSocket clients
  io.to('room:1').emit('message', data);
});

// User sends message
app.post('/api/chat/send', async (req, res) => {
  const message = {
    user: req.user.name,
    text: req.body.text,
    timestamp: Date.now()
  };
  
  // Publish to channel
  await publisher.publish(
    'chat:room:1',
    JSON.stringify(message)
  );
  
  res.json({ success: true });
});
```

---

#### Real-World Example: Real-Time Notifications

```javascript
// Notification service
class NotificationService {
  constructor() {
    this.publisher = redis.createClient();
  }
  
  // Send notification to specific user
  async notify(userId, notification) {
    await this.publisher.publish(
      `user:${userId}:notifications`,
      JSON.stringify(notification)
    );
  }
  
  // Send notification to all users
  async notifyAll(notification) {
    await this.publisher.publish(
      'notifications:global',
      JSON.stringify(notification)
    );
  }
  
  // Send notification by user role
  async notifyByRole(role, notification) {
    await this.publisher.publish(
      `notifications:role:${role}`,
      JSON.stringify(notification)
    );
  }
}

// User connection (WebSocket server)
const subscriber = redis.createClient();

socket.on('connect', () => {
  const userId = socket.userId;
  
  // Subscribe to user-specific channel
  subscriber.subscribe(`user:${userId}:notifications`);
  
  // Subscribe to global notifications
  subscriber.subscribe('notifications:global');
  
  // Subscribe to role-specific
  subscriber.subscribe(`notifications:role:${socket.userRole}`);
});

subscriber.on('message', (channel, message) => {
  const notification = JSON.parse(message);
  
  // Send to WebSocket client
  io.to(socket.id).emit('notification', notification);
});
```

---

## 3.2.4 Expiration & TTL

**TTL (Time-To-Live)** automatically expires keys after a specified duration.

---

#### Setting Expiration

```bash
# Set key with expiration (seconds)
SETEX session:abc123 3600 "user-data"  # Expires in 1 hour

# Set expiration on existing key
SET mykey "value"
EXPIRE mykey 300  # Expires in 5 minutes

# Set expiration in milliseconds
PEXPIRE mykey 5000  # 5 seconds

# Set expiration at specific Unix timestamp
EXPIREAT mykey 1705334400  # Jan 15, 2024 12:00:00 GMT

# Set expiration only if key has no expiration
SET mykey "value"
EXPIRE mykey 3600 NX  # Only if no expiration set

# Set expiration only if key already has expiration
EXPIRE mykey 7200 XX  # Only if expiration exists

# Set expiration and return remaining TTL
EXPIRE mykey 1800 GET  # Set 30 min, return old TTL
```

---

#### Checking TTL

```bash
# Get remaining time in seconds
TTL mykey
# 295 (4 minutes 55 seconds left)

# Get remaining time in milliseconds
PTTL mykey
# 295000

# Key without expiration
TTL persistent-key
# -1 (no expiration)

# Key doesn't exist
TTL nonexistent
# -2 (doesn't exist)
```

---

#### Removing Expiration

```bash
# Remove expiration (make persistent)
PERSIST mykey
# 1 (expiration removed)

# Check expiration removed
TTL mykey
# -1 (no expiration)
```

---

#### Real-World Example: Session Management

```javascript
// Session store with automatic cleanup
class SessionStore {
  
  // Create session with 30-minute expiration
  async create(userId, data) {
    const sessionId = uuid.v4();
    const sessionKey = `session:${sessionId}`;
    
    await redis.hmset(sessionKey, {
      userId,
      createdAt: Date.now(),
      ...data
    });
    
    // Auto-expire after 30 minutes
    await redis.expire(sessionKey, 1800);
    
    return sessionId;
  }
  
  // Get session and refresh expiration
  async get(sessionId) {
    const sessionKey = `session:${sessionId}`;
    
    const session = await redis.hgetall(sessionKey);
    
    if (Object.keys(session).length === 0) {
      return null;  // Session expired or doesn't exist
    }
    
    // Refresh expiration (sliding window)
    await redis.expire(sessionKey, 1800);
    
    return session;
  }
  
  // Update session
  async update(sessionId, data) {
    const sessionKey = `session:${sessionId}`;
    
    await redis.hmset(sessionKey, data);
    
    // Refresh expiration
    await redis.expire(sessionKey, 1800);
  }
  
  // Delete session
  async destroy(sessionId) {
    await redis.del(`session:${sessionId}`);
  }
  
  // Get remaining time
  async getExpiration(sessionId) {
    const ttl = await redis.ttl(`session:${sessionId}`);
    
    if (ttl === -2) return null;  // Doesn't exist
    if (ttl === -1) return Infinity;  // No expiration
    
    return ttl;  // Seconds remaining
  }
}
```

---

#### Real-World Example: Rate Limiting

```javascript
// Rate limiter using TTL
class RateLimiter {
  
  // Limit: 100 requests per hour per user
  async checkLimit(userId) {
    const key = `ratelimit:${userId}`;
    
    // Increment counter
    const count = await redis.incr(key);
    
    // Set expiration on first request (1 hour)
    if (count === 1) {
      await redis.expire(key, 3600);
    }
    
    // Check if over limit
    if (count > 100) {
      const ttl = await redis.ttl(key);
      throw new Error(`Rate limit exceeded. Try again in ${ttl} seconds.`);
    }
    
    return {
      allowed: true,
      remaining: 100 - count,
      resetIn: await redis.ttl(key)
    };
  }
  
  // Sliding window rate limiter (more accurate)
  async checkSlidingWindow(userId) {
    const key = `ratelimit:${userId}`;
    const now = Date.now();
    const windowStart = now - 3600000;  // 1 hour ago
    
    // Remove old requests
    await redis.zremrangebyscore(key, 0, windowStart);
    
    // Count requests in window
    const count = await redis.zcard(key);
    
    if (count >= 100) {
      throw new Error('Rate limit exceeded');
    }
    
    // Add current request
    await redis.zadd(key, now, `${now}-${Math.random()}`);
    
    // Set expiration
    await redis.expire(key, 3600);
    
    return {
      allowed: true,
      remaining: 100 - count - 1
    };
  }
}
```

---

## 3.2.5 Use Cases & Best Practices

---

#### Use Case 1: Shopping Cart

```javascript
class ShoppingCart {
  
  // Add item to cart
  async addItem(userId, product) {
    const cartKey = `cart:${userId}`;
    const itemKey = `item:${product.id}`;
    
    // Store item details as hash
    await redis.hset(
      cartKey,
      itemKey,
      JSON.stringify({
        id: product.id,
        name: product.name,
        price: product.price,
        quantity: 1
      })
    );
    
    // Set cart expiration (7 days)
    await redis.expire(cartKey, 604800);
    
    return await this.getCart(userId);
  }
  
  // Update item quantity
  async updateQuantity(userId, productId, quantity) {
    const cartKey = `cart:${userId}`;
    const itemKey = `item:${productId}`;
    
    const item = await redis.hget(cartKey, itemKey);
    
    if (!item) {
      throw new Error('Item not in cart');
    }
    
    const itemData = JSON.parse(item);
    itemData.quantity = quantity;
    
    await redis.hset(cartKey, itemKey, JSON.stringify(itemData));
    
    return await this.getCart(userId);
  }
  
  // Remove item
  async removeItem(userId, productId) {
    const cartKey = `cart:${userId}`;
    const itemKey = `item:${productId}`;
    
    await redis.hdel(cartKey, itemKey);
    
    return await this.getCart(userId);
  }
  
  // Get cart
  async getCart(userId) {
    const cartKey = `cart:${userId}`;
    const items = await redis.hgetall(cartKey);
    
    const cart = Object.values(items).map(item => JSON.parse(item));
    
    const total = cart.reduce((sum, item) => {
      return sum + (item.price * item.quantity);
    }, 0);
    
    return {
      items: cart,
      itemCount: cart.length,
      total
    };
  }
  
  // Clear cart
  async clear(userId) {
    await redis.del(`cart:${userId}`);
  }
}
```

---

#### Use Case 2: Real-Time Leaderboard

```javascript
class Leaderboard {
  
  // Update player score
  async updateScore(gameId, playerId, score) {
    const key = `leaderboard:${gameId}`;
    
    // Add/update score
    await redis.zadd(key, score, playerId);
    
    // Set expiration (30 days)
    await redis.expire(key, 2592000);
  }
  
  // Increment player score
  async incrementScore(gameId, playerId, points) {
    const key = `leaderboard:${gameId}`;
    
    await redis.zincrby(key, points, playerId);
  }
  
  // Get top N players
  async getTopPlayers(gameId, limit = 10) {
    const key = `leaderboard:${gameId}`;
    
    const players = await redis.zrevrange(
      key,
      0,
      limit - 1,
      'WITHSCORES'
    );
    
    // Format: [player1, score1, player2, score2, ...]
    const leaderboard = [];
    for (let i = 0; i < players.length; i += 2) {
      leaderboard.push({
        rank: Math.floor(i / 2) + 1,
        playerId: players[i],
        score: parseInt(players[i + 1])
      });
    }
    
    return leaderboard;
  }
  
  // Get player rank and score
  async getPlayerRank(gameId, playerId) {
    const key = `leaderboard:${gameId}`;
    
    const score = await redis.zscore(key, playerId);
    const rank = await redis.zrevrank(key, playerId);
    
    if (score === null) {
      return null;
    }
    
    return {
      playerId,
      score: parseInt(score),
      rank: rank + 1  // 1-indexed
    };
  }
  
  // Get players around a player (context)
  async getPlayersAround(gameId, playerId, range = 5) {
    const key = `leaderboard:${gameId}`;
    
    const rank = await redis.zrevrank(key, playerId);
    
    if (rank === null) {
      return null;
    }
    
    const start = Math.max(0, rank - range);
    const end = rank + range;
    
    const players = await redis.zrevrange(
      key,
      start,
      end,
      'WITHSCORES'
    );
    
    const leaderboard = [];
    for (let i = 0; i < players.length; i += 2) {
      leaderboard.push({
        rank: start + Math.floor(i / 2) + 1,
        playerId: players[i],
        score: parseInt(players[i + 1]),
        isCurrentPlayer: players[i] === playerId
      });
    }
    
    return leaderboard;
  }
  
  // Remove player
  async removePlayer(gameId, playerId) {
    const key = `leaderboard:${gameId}`;
    await redis.zrem(key, playerId);
  }
  
  // Get total players
  async getTotalPlayers(gameId) {
    const key = `leaderboard:${gameId}`;
    return await redis.zcard(key);
  }
}
```

---

#### Use Case 3: Distributed Locks

```javascript
class DistributedLock {
  
  // Acquire lock with timeout
  async acquire(resource, timeout = 10000) {
    const lockKey = `lock:${resource}`;
    const lockId = uuid.v4();
    const expiration = Math.ceil(timeout / 1000);
    
    // Try to acquire lock
    const acquired = await redis.set(
      lockKey,
      lockId,
      'NX',  // Only if not exists
      'EX',  // Expiration in seconds
      expiration
    );
    
    if (!acquired) {
      throw new Error('Failed to acquire lock');
    }
    
    return lockId;
  }
  
  // Release lock
  async release(resource, lockId) {
    const lockKey = `lock:${resource}`;
    
    // Lua script for atomic check-and-delete
    const script = `
      if redis.call("get", KEYS[1]) == ARGV[1] then
        return redis.call("del", KEYS[1])
      else
        return 0
      end
    `;
    
    const result = await redis.eval(script, 1, lockKey, lockId);
    
    return result === 1;
  }
  
  // Use lock with automatic release
  async withLock(resource, fn, timeout = 10000) {
    const lockId = await this.acquire(resource, timeout);
    
    try {
      return await fn();
    } finally {
      await this.release(resource, lockId);
    }
  }
}

// Usage
const lock = new DistributedLock();

// Prevent concurrent inventory updates
await lock.withLock('product:123:inventory', async () => {
  const product = await db.products.findOne({ _id: '123' });
  
  if (product.stock >= quantity) {
    await db.products.updateOne(
      { _id: '123' },
      { $inc: { stock: -quantity } }
    );
  }
});
```

---

#### Use Case 4: Job Queue

```javascript
class JobQueue {
  
  // Add job to queue
  async enqueue(queueName, job) {
    const jobId = uuid.v4();
    const jobData = JSON.stringify({
      id: jobId,
      ...job,
      enqueuedAt: Date.now()
    });
    
    // Add to list
    await redis.rpush(`queue:${queueName}`, jobData);
    
    return jobId;
  }
  
  // Process job (blocking)
  async dequeue(queueName, timeout = 0) {
    const result = await redis.blpop(`queue:${queueName}`, timeout);
    
    if (!result) {
      return null;
    }
    
    const [, jobData] = result;
    return JSON.parse(jobData);
  }
  
  // Priority queue using sorted sets
  async enqueuePriority(queueName, job, priority) {
    const jobId = uuid.v4();
    const jobData = JSON.stringify({
      id: jobId,
      ...job
    });
    
    // Higher priority = lower score (processed first)
    await redis.zadd(
      `queue:priority:${queueName}`,
      -priority,
      jobData
    );
    
    return jobId;
  }
  
  // Dequeue highest priority job
  async dequeuePriority(queueName) {
    // Get and remove highest priority (lowest score)
    const result = await redis.zpopmin(`queue:priority:${queueName}`);
    
    if (result.length === 0) {
      return null;
    }
    
    const [jobData] = result;
    return JSON.parse(jobData);
  }
  
  // Get queue length
  async getLength(queueName) {
    return await redis.llen(`queue:${queueName}`);
  }
  
  // Peek at next job without removing
  async peek(queueName) {
    const jobData = await redis.lindex(`queue:${queueName}`, 0);
    return jobData ? JSON.parse(jobData) : null;
  }
}

// Worker process
const queue = new JobQueue();

async function worker() {
  while (true) {
    const job = await queue.dequeue('email', 5);  // 5 second timeout
    
    if (job) {
      console.log(`Processing job ${job.id}`);
      
      try {
        await sendEmail(job.to, job.subject, job.body);
        console.log(`Job ${job.id} completed`);
      } catch (error) {
        console.error(`Job ${job.id} failed:`, error);
        // Re-queue or move to dead letter queue
      }
    }
  }
}
```

---

### Redis Best Practices

#### 1. Key Naming Conventions

```bash
# ✅ Good: Use colon-separated hierarchy
user:1:profile
user:1:sessions:abc123
product:123:reviews
cache:user:1:timeline

# ❌ Bad: Inconsistent or unclear
user_1_profile
u1p
userProfile123

# Pattern: object:id:attribute
# or: namespace:object:id:attribute
```

---

#### 2. Memory Optimization

```javascript
// ✅ Use Hashes for objects (more efficient)
await redis.hmset('user:1', {
  name: 'Alice',
  email: 'alice@example.com',
  age: '28'
});
// Memory: ~50 bytes

// ❌ Don't use separate keys for each field
await redis.set('user:1:name', 'Alice');
await redis.set('user:1:email', 'alice@example.com');
await redis.set('user:1:age', '28');
// Memory: ~200 bytes (4x more!)

// ✅ Set memory limits
CONFIG SET maxmemory 2gb
CONFIG SET maxmemory-policy allkeys-lru
```

---

#### 3. Connection Pooling

```javascript
// ✅ Good: Reuse connections
const redis = require('redis');
const client = redis.createClient({
  host: 'localhost',
  port: 6379,
  // Connection pool
  max_clients: 50,
  // Retry strategy
  retry_strategy: (options) => {
    if (options.total_retry_time > 1000 * 60 * 60) {
      return new Error('Retry time exhausted');
    }
    return Math.min(options.attempt * 100, 3000);
  }
});

// ❌ Bad: Create new connection for each operation
async function badExample() {
  const client = redis.createClient();
  await client.set('key', 'value');
  client.quit();  // Don't do this!
}
```

---

#### 4. Pipeline Multiple Commands

```javascript
// ✅ Good: Use pipeline for multiple commands
const pipeline = redis.pipeline();

pipeline.set('key1', 'value1');
pipeline.set('key2', 'value2');
pipeline.set('key3', 'value3');
pipeline.incr('counter');

const results = await pipeline.exec();
// Single round-trip!

// ❌ Bad: Multiple round-trips
await redis.set('key1', 'value1');
await redis.set('key2', 'value2');
await redis.set('key3', 'value3');
await redis.incr('counter');
// 4 round-trips!
```

---

#### 5. Use Appropriate Data Types

```javascript
// ✅ Use Sorted Sets for leaderboards
await redis.zadd('leaderboard', 1000, 'player1');

// ❌ Don't use Strings
// Bad: Manual sorting required
await redis.set('leaderboard:player1', '1000');

// ✅ Use Hashes for objects
await redis.hmset('user:1', { name: 'Alice', age: 28 });

// ❌ Don't use JSON strings if you need field access
// Bad: Must parse entire object to get one field
await redis.set('user:1', JSON.stringify({ name: 'Alice', age: 28 }));

// ✅ Use Sets for unique collections
await redis.sadd('tags:post:1', 'mongodb', 'redis', 'nosql');

// ❌ Don't use Lists for uniqueness
// Bad: Duplicates possible
await redis.rpush('tags:post:1', 'mongodb', 'redis', 'mongodb');
```

---

#### 6. Monitor Performance

```bash
# Monitor commands in real-time
MONITOR

# Get server info
INFO stats
INFO memory
INFO replication

# Slow log
SLOWLOG GET 10

# Check memory usage of key
MEMORY USAGE user:1

# Get detailed server stats
INFO all
```

---

#### 7. Persistence Strategy

```javascript
// RDB: Point-in-time snapshots
// Good for: Disaster recovery, low overhead
CONFIG SET save "900 1"      // After 900s if 1 key changed
CONFIG SET save "300 10"     // After 300s if 10 keys changed
CONFIG SET save "60 10000"   // After 60s if 10000 keys changed

// AOF: Append-only file (every write logged)
// Good for: Durability, minimal data loss
CONFIG SET appendonly yes
CONFIG SET appendfsync everysec  // Sync every second

// Both: Maximum durability
// Use both RDB + AOF for best recovery options
```

---

### Practice Questions: Redis Use Cases & Best Practices

#### Fill in the Blanks

1. For shopping carts, use Redis ________ to store items with automatic expiration.
2. Redis ________ ________ are perfect for implementing real-time game leaderboards.
3. Use ________ to execute multiple Redis commands in a single round-trip.
4. The recommended key naming pattern is ________ separated hierarchy.
5. ________ persistence creates point-in-time snapshots, while ________ logs every write.

<details>
<summary><strong>View Answers</strong></summary>

1. Hashes
2. Sorted Sets
3. pipeline
4. colon (:)
5. RDB, AOF

</details>

---

#### True/False

1. Hashes are more memory-efficient than separate string keys for objects.
2. You should create a new Redis connection for every operation.
3. Pipeline reduces network round-trips for multiple commands.
4. Redis can automatically expire keys after a specified time.
5. Sorted Sets maintain insertion order like Lists.

<details>
<summary><strong>View Answers</strong></summary>

1. TRUE
2. FALSE (reuse connections with pooling)
3. TRUE
4. TRUE
5. FALSE (Sorted Sets order by score, not insertion)

</details>

---

## Redis Summary

**3.2.1 Data Structures:**
- **String**: Simple key-value, counters (INCR, SETEX)
- **Hash**: Objects with fields (HSET, HGETALL, HINCRBY)
- **List**: Ordered collection, queues (LPUSH, RPUSH, LPOP, LRANGE)
- **Set**: Unique elements, tags (SADD, SINTER, SUNION, SDIFF)
- **Sorted Set**: Ranked data, leaderboards (ZADD, ZRANGE, ZINCRBY)

**3.2.2 Caching Patterns:**
- **Cache-Aside**: Check cache → DB on miss → update cache (most common)
- **Write-Through**: Update cache + DB simultaneously (consistency)
- **Write-Behind**: Update cache → async DB update (performance)
- **Refresh-Ahead**: Proactive refresh before expiration
- Invalidation: TTL (time-based), explicit (on update), event-driven

**3.2.3 Pub/Sub:**
- Real-time messaging: PUBLISH, SUBSCRIBE, PSUBSCRIBE
- Use cases: Chat apps, notifications, microservices
- Pattern matching for dynamic channels

**3.2.4 TTL & Expiration:**
- Auto-expire keys: SETEX, EXPIRE, EXPIREAT
- Check TTL: TTL (seconds), PTTL (milliseconds)
- Use cases: Sessions (sliding window), rate limiting

**3.2.5 Use Cases:**
- **Shopping Cart**: Hash with TTL (7 days)
- **Leaderboard**: Sorted Set with ZINCRBY
- **Distributed Lock**: SET NX EX with Lua script release
- **Job Queue**: List (FIFO) or Sorted Set (priority)

---
