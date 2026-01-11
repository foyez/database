# Chapter 7: Real-World Projects
**Complete Implementations with NoSQL**

---

## Table of Contents
- [7.3 Real-Time Gaming (Redis)](#73-real-time-gaming-redis)
  - [7.3.1 Leaderboard Implementation](#731-leaderboard-implementation)
  - [7.3.2 Session Management](#732-session-management)
  - [7.3.3 Rate Limiting](#733-rate-limiting)
  - [7.3.4 Pub/Sub Chat](#734-pubsub-chat)
  - [Practice Questions](#practice-questions)

---

## 7.3 Real-Time Gaming (Redis)

### Project Overview

**Features:**
- Real-time leaderboards
- Session management
- Rate limiting
- Pub/Sub chat system
- Player matchmaking
- Game state caching

**Why Redis?**
- ✅ In-memory (microsecond latency)
- ✅ Built-in data structures (sorted sets, hashes)
- ✅ Pub/Sub messaging
- ✅ TTL for automatic expiration
- ✅ Atomic operations

---

## 7.3.1 Leaderboard Implementation

### Global Leaderboard

```javascript
const redis = require('redis');
const client = redis.createClient({
  host: 'localhost',
  port: 6379
});

await client.connect();

// Add player score
async function updateScore(playerId, score) {
  // Use sorted set (ZADD)
  await client.zAdd('leaderboard:global', {
    score: score,
    value: playerId
  });
}

// Increment score
async function incrementScore(playerId, points) {
  await client.zIncrBy('leaderboard:global', points, playerId);
}

// Get top N players
async function getTopPlayers(limit = 10) {
  // ZREVRANGE with WITHSCORES (highest to lowest)
  const results = await client.zRangeWithScores(
    'leaderboard:global',
    0,
    limit - 1,
    { REV: true }
  );
  
  // Enrich with player data
  const players = await Promise.all(
    results.map(async (result, index) => {
      const playerData = await client.hGetAll(`player:${result.value}`);
      return {
        rank: index + 1,
        playerId: result.value,
        score: result.score,
        username: playerData.username,
        avatar: playerData.avatar
      };
    })
  );
  
  return players;
}

// Get player rank
async function getPlayerRank(playerId) {
  // ZREVRANK (0-based, so add 1)
  const rank = await client.zRevRank('leaderboard:global', playerId);
  
  if (rank === null) {
    return null;
  }
  
  const score = await client.zScore('leaderboard:global', playerId);
  
  return {
    rank: rank + 1,
    score: score
  };
}

// Get players around a specific player
async function getPlayersAround(playerId, range = 5) {
  const rank = await client.zRevRank('leaderboard:global', playerId);
  
  if (rank === null) {
    return [];
  }
  
  const start = Math.max(0, rank - range);
  const end = rank + range;
  
  const results = await client.zRangeWithScores(
    'leaderboard:global',
    start,
    end,
    { REV: true }
  );
  
  const players = await Promise.all(
    results.map(async (result, index) => {
      const playerData = await client.hGetAll(`player:${result.value}`);
      return {
        rank: start + index + 1,
        playerId: result.value,
        score: result.score,
        username: playerData.username,
        isCurrentPlayer: result.value === playerId
      };
    })
  );
  
  return players;
}
```

---

### Daily/Weekly Leaderboards

```javascript
// Add score to daily leaderboard
async function addDailyScore(playerId, score) {
  const today = new Date().toISOString().split('T')[0];  // YYYY-MM-DD
  const key = `leaderboard:daily:${today}`;
  
  await client.zAdd(key, {
    score: score,
    value: playerId
  });
  
  // Expire after 7 days
  await client.expire(key, 7 * 24 * 60 * 60);
}

// Get daily top players
async function getDailyTopPlayers(date, limit = 10) {
  const key = `leaderboard:daily:${date}`;
  
  const results = await client.zRangeWithScores(
    key,
    0,
    limit - 1,
    { REV: true }
  );
  
  return results.map((result, index) => ({
    rank: index + 1,
    playerId: result.value,
    score: result.score
  }));
}

// Weekly leaderboard (sum of daily scores)
async function getWeeklyScores(playerId) {
  const keys = [];
  const today = new Date();
  
  // Get last 7 days
  for (let i = 0; i < 7; i++) {
    const date = new Date(today);
    date.setDate(date.getDate() - i);
    const dateStr = date.toISOString().split('T')[0];
    keys.push(`leaderboard:daily:${dateStr}`);
  }
  
  // Use ZUNIONSTORE to combine
  const weeklyKey = `leaderboard:weekly:temp:${playerId}`;
  
  await client.zUnionStore(weeklyKey, keys);
  
  const score = await client.zScore(weeklyKey, playerId);
  const rank = await client.zRevRank(weeklyKey, playerId);
  
  // Clean up temp key
  await client.del(weeklyKey);
  
  return {
    score: score || 0,
    rank: rank !== null ? rank + 1 : null
  };
}
```

---

### Multi-Level Leaderboards

```javascript
// Store player in level-specific leaderboard
async function updateLevelScore(playerId, level, score) {
  const key = `leaderboard:level:${level}`;
  
  await client.zAdd(key, {
    score: score,
    value: playerId
  });
}

// Get top players for a level
async function getLevelTopPlayers(level, limit = 10) {
  const key = `leaderboard:level:${level}`;
  
  const results = await client.zRangeWithScores(
    key,
    0,
    limit - 1,
    { REV: true }
  );
  
  return results.map((result, index) => ({
    rank: index + 1,
    playerId: result.value,
    score: result.score
  }));
}
```

---

## 7.3.2 Session Management

### Store Session Data

```javascript
// Create session
async function createSession(userId, sessionData) {
  const sessionId = `session:${userId}:${Date.now()}`;
  
  const session = {
    userId: userId,
    username: sessionData.username,
    loginTime: Date.now(),
    ip: sessionData.ip,
    userAgent: sessionData.userAgent
  };
  
  // Store as hash
  await client.hSet(sessionId, session);
  
  // Set expiration (30 minutes)
  await client.expire(sessionId, 30 * 60);
  
  return sessionId;
}

// Get session
async function getSession(sessionId) {
  const session = await client.hGetAll(sessionId);
  
  if (Object.keys(session).length === 0) {
    return null;
  }
  
  // Refresh TTL
  await client.expire(sessionId, 30 * 60);
  
  return session;
}

// Update last activity
async function updateSessionActivity(sessionId) {
  await client.hSet(sessionId, 'lastActivity', Date.now());
  await client.expire(sessionId, 30 * 60);
}

// Delete session (logout)
async function deleteSession(sessionId) {
  await client.del(sessionId);
}

// Get all active sessions for user
async function getUserSessions(userId) {
  const pattern = `session:${userId}:*`;
  const keys = [];
  
  for await (const key of client.scanIterator({
    MATCH: pattern,
    COUNT: 100
  })) {
    keys.push(key);
  }
  
  const sessions = await Promise.all(
    keys.map(key => client.hGetAll(key))
  );
  
  return sessions;
}
```

---

### Active Players Tracking

```javascript
// Mark player as online
async function markPlayerOnline(playerId) {
  const key = 'players:online';
  
  // Add to set with current timestamp as score
  await client.zAdd(key, {
    score: Date.now(),
    value: playerId
  });
}

// Check if player is online
async function isPlayerOnline(playerId) {
  const score = await client.zScore('players:online', playerId);
  
  if (!score) {
    return false;
  }
  
  // Consider online if active in last 5 minutes
  const fiveMinutesAgo = Date.now() - (5 * 60 * 1000);
  return score > fiveMinutesAgo;
}

// Get online players count
async function getOnlineCount() {
  const fiveMinutesAgo = Date.now() - (5 * 60 * 1000);
  
  const count = await client.zCount(
    'players:online',
    fiveMinutesAgo,
    '+inf'
  );
  
  return count;
}

// Clean up inactive players
async function cleanupInactivePlayers() {
  const fiveMinutesAgo = Date.now() - (5 * 60 * 1000);
  
  await client.zRemRangeByScore(
    'players:online',
    '-inf',
    fiveMinutesAgo
  );
}

// Run cleanup periodically
setInterval(cleanupInactivePlayers, 60000);  // Every minute
```

---

## 7.3.3 Rate Limiting

### Token Bucket Algorithm

```javascript
// Check if action is allowed
async function checkRateLimit(userId, action, maxRequests, windowSeconds) {
  const key = `ratelimit:${action}:${userId}`;
  const now = Date.now();
  const windowStart = now - (windowSeconds * 1000);
  
  // Remove old entries
  await client.zRemRangeByScore(key, '-inf', windowStart);
  
  // Count recent requests
  const requestCount = await client.zCard(key);
  
  if (requestCount >= maxRequests) {
    const oldest = await client.zRange(key, 0, 0, { WITHSCORES: true });
    const retryAfter = Math.ceil((oldest[0].score + (windowSeconds * 1000) - now) / 1000);
    
    return {
      allowed: false,
      retryAfter: retryAfter
    };
  }
  
  // Add current request
  await client.zAdd(key, {
    score: now,
    value: `${now}:${Math.random()}`
  });
  
  // Set expiration
  await client.expire(key, windowSeconds);
  
  return {
    allowed: true,
    remaining: maxRequests - requestCount - 1
  };
}

// Usage example
async function sendMessage(userId, message) {
  // Limit: 10 messages per minute
  const rateLimit = await checkRateLimit(userId, 'chat', 10, 60);
  
  if (!rateLimit.allowed) {
    throw new Error(`Rate limit exceeded. Retry after ${rateLimit.retryAfter} seconds`);
  }
  
  // Send message
  await publishMessage(userId, message);
  
  return {
    success: true,
    remaining: rateLimit.remaining
  };
}
```

---

### Fixed Window Counter

```javascript
// Simpler rate limiting (fixed window)
async function simpleRateLimit(userId, action, maxRequests, windowSeconds) {
  const now = Math.floor(Date.now() / 1000);
  const windowStart = Math.floor(now / windowSeconds) * windowSeconds;
  const key = `ratelimit:${action}:${userId}:${windowStart}`;
  
  const count = await client.incr(key);
  
  if (count === 1) {
    await client.expire(key, windowSeconds);
  }
  
  if (count > maxRequests) {
    return {
      allowed: false,
      retryAfter: windowSeconds - (now % windowSeconds)
    };
  }
  
  return {
    allowed: true,
    remaining: maxRequests - count
  };
}
```

---

## 7.3.4 Pub/Sub Chat

### Channel-Based Chat

```javascript
// Subscribe to channel
async function subscribeToChannel(channelName, onMessage) {
  const subscriber = client.duplicate();
  await subscriber.connect();
  
  await subscriber.subscribe(channelName, (message) => {
    const data = JSON.parse(message);
    onMessage(data);
  });
  
  return subscriber;
}

// Publish message
async function publishMessage(channelName, userId, message) {
  const data = {
    userId: userId,
    message: message,
    timestamp: Date.now()
  };
  
  await client.publish(channelName, JSON.stringify(data));
  
  // Also store in chat history (last 100 messages)
  const historyKey = `chat:history:${channelName}`;
  
  await client.lPush(historyKey, JSON.stringify(data));
  await client.lTrim(historyKey, 0, 99);  // Keep last 100
}

// Get chat history
async function getChatHistory(channelName, limit = 50) {
  const historyKey = `chat:history:${channelName}`;
  const messages = await client.lRange(historyKey, 0, limit - 1);
  
  return messages.map(msg => JSON.parse(msg)).reverse();
}

// Example usage
const subscriber = await subscribeToChannel('game:lobby', (data) => {
  console.log(`[${new Date(data.timestamp).toISOString()}] User ${data.userId}: ${data.message}`);
});

await publishMessage('game:lobby', 'user123', 'Hello everyone!');
```

---

### Private Messaging

```javascript
// Create private channel between two users
function getPrivateChannelName(userId1, userId2) {
  const sorted = [userId1, userId2].sort();
  return `chat:private:${sorted[0]}:${sorted[1]}`;
}

// Send private message
async function sendPrivateMessage(fromUserId, toUserId, message) {
  const channelName = getPrivateChannelName(fromUserId, toUserId);
  
  const data = {
    from: fromUserId,
    to: toUserId,
    message: message,
    timestamp: Date.now()
  };
  
  // Publish
  await client.publish(channelName, JSON.stringify(data));
  
  // Store in history
  const historyKey = `chat:history:${channelName}`;
  await client.lPush(historyKey, JSON.stringify(data));
  await client.lTrim(historyKey, 0, 99);
  
  // Store unread count for recipient
  const unreadKey = `chat:unread:${toUserId}:${fromUserId}`;
  await client.incr(unreadKey);
}

// Mark messages as read
async function markAsRead(userId, fromUserId) {
  const unreadKey = `chat:unread:${userId}:${fromUserId}`;
  await client.del(unreadKey);
}

// Get unread count
async function getUnreadCount(userId) {
  const pattern = `chat:unread:${userId}:*`;
  let totalUnread = 0;
  
  for await (const key of client.scanIterator({
    MATCH: pattern,
    COUNT: 100
  })) {
    const count = await client.get(key);
    totalUnread += parseInt(count || 0);
  }
  
  return totalUnread;
}
```

---

### Presence Notifications

```javascript
// Notify user joined channel
async function notifyUserJoined(channelName, userId) {
  const data = {
    type: 'user_joined',
    userId: userId,
    timestamp: Date.now()
  };
  
  await client.publish(`${channelName}:presence`, JSON.stringify(data));
  
  // Add to online users set
  await client.sAdd(`channel:${channelName}:users`, userId);
}

// Notify user left channel
async function notifyUserLeft(channelName, userId) {
  const data = {
    type: 'user_left',
    userId: userId,
    timestamp: Date.now()
  };
  
  await client.publish(`${channelName}:presence`, JSON.stringify(data));
  
  // Remove from online users set
  await client.sRem(`channel:${channelName}:users`, userId);
}

// Get online users in channel
async function getChannelUsers(channelName) {
  const users = await client.sMembers(`channel:${channelName}:users`);
  return users;
}
```

---

### Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

#### Fill in the Blanks

1. Redis ________ ________ are perfect for leaderboards because they maintain sorted order automatically.
2. The ________ command increments a sorted set member's score atomically.
3. Redis ________ provides publish-subscribe messaging for real-time communication.
4. ________ ________ is a rate limiting algorithm that uses sorted sets to track request timestamps.
5. The ________ command removes members from a sorted set within a score range.

<details>
<summary><strong>View Answers</strong></summary>

1. sorted sets
2. ZINCRBY
3. Pub/Sub
4. Token bucket (or sliding window)
5. ZREMRANGEBYSCORE

</details>

</details>

---