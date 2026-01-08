# Chapter 3: NoSQL Deep Dive
**MongoDB and Redis Operations, Patterns, and Best Practices**

---

## Table of Contents
- [3.1 MongoDB](./sections/03_01_mongodb.md)
- [3.2 Redis](./sections/03_02_redis.md)

---

## Chapter Summary

### MongoDB

**3.1.1 CRUD Operations:**
- Create: insertOne(), insertMany()
- Read: find(), findOne() with operators ($gt, $lt, $in, $exists)
- Update: updateOne(), updateMany() with $set, $inc, $push, $addToSet
- Delete: deleteOne(), deleteMany()
- Real-world blog platform example

**3.1.2 Embedded vs Referenced:**
- **Embedded**: Data together, faster reads, 16MB limit
- **Referenced**: Normalized, flexible, requires $lookup
- **Decision**: Use hybrid approach for real-world applications
- Orders example: embed customer snapshot, reference payment

**3.1.3 Aggregation Pipeline:**
- Stages: $match, $group, $project, $sort, $limit, $skip, $lookup, $unwind, $addFields
- Accumulators: $sum, $avg, $min, $max, $first, $last, $push
- Real-world: E-commerce sales reports, user analytics
- Pipeline processes data through sequential transformations

**3.1.4 Schema Design Patterns:**
- **Attribute Pattern**: Sparse fields → key-value array
- **Bucket Pattern**: Time-series → group readings into buckets
- **Computed Pattern**: Expensive queries → pre-calculate stats
- **Subset Pattern**: Large arrays → recent + full data split
- **Extended Reference**: Frequent lookups → duplicate common fields

**3.1.5 Indexing:**
- Types: Single, Compound, Unique, Sparse, TTL, Text, Multikey
- **ESR Rule**: Equality, Sort, Range (optimal compound index order)
- Covered queries: All data from index (no document fetch)
- Use explain() to analyze: IXSCAN (good) vs COLLSCAN (slow)
- Index selectivity matters (unique email > binary gender)

---

### Redis

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

### Key Takeaways

**MongoDB Best For:**
- ✅ Flexible schemas
- ✅ Complex queries and aggregations
- ✅ Hierarchical/nested data
- ✅ Document storage
- ✅ Strong query language

**Redis Best For:**
- ✅ Ultra-fast access (< 1ms)
- ✅ Caching layer
- ✅ Real-time features
- ✅ Simple data structures
- ✅ Atomic operations

**Combined Architecture:**
```
User Request
    ↓
Redis (Cache) ← Fast reads
    ↓ (miss)
MongoDB (Database) ← Complex queries
    ↓
Cache result in Redis
    ↓
Return to user
```

**Performance Tips:**
- MongoDB: Use indexes (ESR rule), aggregation pipeline
- Redis: Pipeline commands, use appropriate data types
- Both: Connection pooling, monitoring

**Schema Design:**
- MongoDB: Embed frequently accessed together, reference for large/unbounded
- Redis: Hierarchical key naming (user:1:profile), Hashes for objects

**Production Considerations:**
- MongoDB: Sharding for scale, replica sets for HA
- Redis: Persistence (RDB + AOF), maxmemory policies
- Both: Monitoring, backups, security

---

## Comprehensive Practice Challenge

### Social Media Platform Backend

Build a simplified social media backend using both MongoDB and Redis.

**Requirements:**

**MongoDB Collections:**
```javascript
// Users
{
  _id: ObjectId,
  username: String,
  email: String,
  bio: String,
  followers: [ObjectId],
  following: [ObjectId],
  stats: { posts: Number, followers: Number, following: Number }
}

// Posts
{
  _id: ObjectId,
  userId: ObjectId,
  content: String,
  imageUrl: String,
  likes: [ObjectId],
  comments: [{ userId, text, createdAt }],
  createdAt: Date
}

// Indexes
db.users.createIndex({ username: 1 }, { unique: true });
db.posts.createIndex({ userId: 1, createdAt: -1 });
```

**Redis Caching:**
```javascript
// User profiles (Hash, 1 hour TTL)
HMSET user:123 username "alice" bio "Developer" followers 500

// Online users (Set)
SADD users:online "user:123" "user:456"

// Post likes counter (String)
INCR post:789:likes

// Trending posts (Sorted Set, score = engagement)
ZADD trending:posts 1523 "post:789"

// User sessions (Hash, 30 min TTL)
HMSET session:abc123 userId "123" loginAt "..."
EXPIRE session:abc123 1800
```

**Aggregation Example:**
```javascript
// User feed: Posts from followed users
db.posts.aggregate([
  { $match: { userId: { $in: user.following } } },
  { $lookup: { from: "users", localField: "userId", foreignField: "_id", as: "author" } },
  { $unwind: "$author" },
  { $sort: { createdAt: -1 } },
  { $limit: 20 }
]);
```

---

**Next Chapter:** [Performance & Optimization →](./sections/04_performance.md)