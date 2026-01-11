# Chapter 7: Real-World Projects
**Complete Implementations with NoSQL**

---

## Table of Contents
- [7.2 Blog Platform (MongoDB)](#72-blog-platform-mongodb)
  - [7.2.1 Document Model Design](#721-document-model-design)
  - [7.2.2 CRUD Operations](#722-crud-operations)
  - [7.2.3 Aggregation Queries](#723-aggregation-queries)
  - [Practice Questions](#practice-questions)

---

## 7.2 Blog Platform (MongoDB)

### Project Overview

**Features:**
- User accounts with profiles
- Create, edit, delete posts
- Categories and tags
- Comments and replies
- Like/bookmark posts
- User followers
- Draft posts
- Rich text content

**Why MongoDB?**
- ✅ Flexible schema (posts can have varying structures)
- ✅ Embedded documents (comments within posts)
- ✅ Fast reads for content delivery
- ✅ Easy to scale horizontally
- ✅ Good for evolving requirements

---

## 7.2.1 Document Model Design

### Design Decisions

**Embed vs Reference:**

```javascript
// ❌ Reference everything (too many queries)
{
  _id: ObjectId("post-1"),
  title: "My Post",
  authorId: ObjectId("user-1"),     // Reference
  categoryId: ObjectId("cat-1"),    // Reference
  commentIds: [...]                 // Reference
}

// ✅ Hybrid approach (balance)
{
  _id: ObjectId("post-1"),
  title: "My Post",
  
  // Embedded author info (read frequently, changes rarely)
  author: {
    id: ObjectId("user-1"),
    name: "Alice",
    avatar: "url"
  },
  
  // Embedded category (small, stable)
  category: {
    id: ObjectId("cat-1"),
    name: "Technology",
    slug: "technology"
  },
  
  // Embedded comments (bounded - limit to 100)
  comments: [
    {
      id: ObjectId("comment-1"),
      userId: ObjectId("user-2"),
      userName: "Bob",
      text: "Great post!",
      createdAt: ISODate("2024-01-15")
    }
    // ... max 100 comments embedded
  ],
  
  // Tags array (small list)
  tags: ["javascript", "mongodb", "nodejs"]
}
```

---

### Complete Schema Design

**Users Collection:**

```javascript
{
  _id: ObjectId(),
  email: "alice@example.com",
  username: "alice_dev",
  passwordHash: "bcrypt_hash_here",
  
  profile: {
    firstName: "Alice",
    lastName: "Johnson",
    bio: "Software engineer passionate about databases",
    avatar: "https://example.com/avatars/alice.jpg",
    website: "https://alice.dev",
    location: "San Francisco, CA"
  },
  
  stats: {
    postsCount: 42,
    followersCount: 1250,
    followingCount: 380,
    totalLikes: 3420
  },
  
  social: {
    twitter: "alice_dev",
    github: "alice",
    linkedin: "alicejohnson"
  },
  
  settings: {
    emailNotifications: true,
    publicProfile: true,
    theme: "dark"
  },
  
  createdAt: ISODate("2023-06-15T10:30:00Z"),
  updatedAt: ISODate("2024-01-15T14:20:00Z"),
  lastLoginAt: ISODate("2024-01-15T14:20:00Z"),
  isActive: true,
  emailVerified: true
}
```

**Indexes:**
```javascript
db.users.createIndex({ email: 1 }, { unique: true });
db.users.createIndex({ username: 1 }, { unique: true });
db.users.createIndex({ "stats.followersCount": -1 });
db.users.createIndex({ createdAt: -1 });
```

---

**Posts Collection:**

```javascript
{
  _id: ObjectId(),
  title: "Getting Started with MongoDB",
  slug: "getting-started-with-mongodb",
  
  // Rich text content
  content: {
    markdown: "# Introduction\n\nMongoDB is...",
    html: "<h1>Introduction</h1><p>MongoDB is...</p>",
    excerpt: "MongoDB is a NoSQL database...",
    readingTime: 5  // minutes
  },
  
  // Author info (embedded for fast reads)
  author: {
    id: ObjectId("user-1"),
    username: "alice_dev",
    name: "Alice Johnson",
    avatar: "https://example.com/avatars/alice.jpg"
  },
  
  // Category (embedded - small, stable)
  category: {
    id: ObjectId("cat-1"),
    name: "Database",
    slug: "database"
  },
  
  // Tags (array)
  tags: ["mongodb", "nosql", "databases", "tutorial"],
  
  // Cover image
  coverImage: {
    url: "https://example.com/images/mongodb-cover.jpg",
    alt: "MongoDB logo",
    width: 1200,
    height: 630
  },
  
  // Engagement stats
  stats: {
    views: 3450,
    likes: 234,
    comments: 42,
    bookmarks: 67,
    shares: 18
  },
  
  // Comments (embedded, limited to 100 recent)
  comments: [
    {
      _id: ObjectId(),
      userId: ObjectId("user-2"),
      username: "bob_coder",
      userAvatar: "https://example.com/avatars/bob.jpg",
      text: "Great tutorial! Very helpful.",
      likes: 5,
      
      // Nested replies (1 level only)
      replies: [
        {
          _id: ObjectId(),
          userId: ObjectId("user-1"),
          username: "alice_dev",
          text: "Thanks Bob!",
          createdAt: ISODate("2024-01-16T10:00:00Z")
        }
      ],
      
      createdAt: ISODate("2024-01-16T09:30:00Z"),
      updatedAt: ISODate("2024-01-16T09:30:00Z")
    }
  ],
  
  // Publishing info
  status: "published",  // draft, published, archived
  publishedAt: ISODate("2024-01-15T10:00:00Z"),
  createdAt: ISODate("2024-01-14T15:00:00Z"),
  updatedAt: ISODate("2024-01-15T10:00:00Z"),
  
  // SEO
  seo: {
    metaTitle: "Getting Started with MongoDB - Complete Guide",
    metaDescription: "Learn MongoDB from scratch with this comprehensive guide",
    keywords: ["mongodb", "nosql", "database", "tutorial"]
  },
  
  // Flags
  isFeatured: false,
  allowComments: true,
  isPublic: true
}
```

**Indexes:**
```javascript
db.posts.createIndex({ slug: 1 }, { unique: true });
db.posts.createIndex({ "author.id": 1, status: 1, publishedAt: -1 });
db.posts.createIndex({ status: 1, publishedAt: -1 });
db.posts.createIndex({ "category.slug": 1, publishedAt: -1 });
db.posts.createIndex({ tags: 1, publishedAt: -1 });
db.posts.createIndex({ "stats.views": -1 });
db.posts.createIndex({ "stats.likes": -1 });

// Text search index
db.posts.createIndex({
  title: "text",
  "content.excerpt": "text",
  tags: "text"
}, {
  weights: {
    title: 10,
    tags: 5,
    "content.excerpt": 1
  }
});
```

---

**Categories Collection:**

```javascript
{
  _id: ObjectId(),
  name: "Database",
  slug: "database",
  description: "Articles about databases and data management",
  icon: "database-icon.svg",
  color: "#3B82F6",
  
  stats: {
    postsCount: 156,
    followersCount: 4200
  },
  
  createdAt: ISODate("2023-01-01T00:00:00Z"),
  isActive: true
}
```

**Indexes:**
```javascript
db.categories.createIndex({ slug: 1 }, { unique: true });
db.categories.createIndex({ "stats.postsCount": -1 });
```

---

**Follows Collection (User Relationships):**

```javascript
{
  _id: ObjectId(),
  followerId: ObjectId("user-2"),    // Bob follows Alice
  followingId: ObjectId("user-1"),
  
  // Denormalized for fast queries
  follower: {
    username: "bob_coder",
    name: "Bob Smith",
    avatar: "https://example.com/avatars/bob.jpg"
  },
  
  following: {
    username: "alice_dev",
    name: "Alice Johnson",
    avatar: "https://example.com/avatars/alice.jpg"
  },
  
  createdAt: ISODate("2024-01-10T12:00:00Z")
}
```

**Indexes:**
```javascript
db.follows.createIndex({ followerId: 1, followingId: 1 }, { unique: true });
db.follows.createIndex({ followerId: 1, createdAt: -1 });
db.follows.createIndex({ followingId: 1, createdAt: -1 });
```

---

**Likes Collection:**

```javascript
{
  _id: ObjectId(),
  userId: ObjectId("user-2"),
  postId: ObjectId("post-1"),
  createdAt: ISODate("2024-01-16T14:00:00Z")
}
```

**Indexes:**
```javascript
db.likes.createIndex({ userId: 1, postId: 1 }, { unique: true });
db.likes.createIndex({ postId: 1, createdAt: -1 });
db.likes.createIndex({ userId: 1, createdAt: -1 });
```

---

**Bookmarks Collection:**

```javascript
{
  _id: ObjectId(),
  userId: ObjectId("user-2"),
  postId: ObjectId("post-1"),
  
  // Denormalized post info
  post: {
    title: "Getting Started with MongoDB",
    slug: "getting-started-with-mongodb",
    excerpt: "MongoDB is a NoSQL database...",
    coverImage: "https://example.com/images/mongodb-cover.jpg",
    author: {
      username: "alice_dev",
      name: "Alice Johnson"
    }
  },
  
  createdAt: ISODate("2024-01-16T15:00:00Z")
}
```

**Indexes:**
```javascript
db.bookmarks.createIndex({ userId: 1, postId: 1 }, { unique: true });
db.bookmarks.createIndex({ userId: 1, createdAt: -1 });
```

---

# Chapter 7: Real-World Projects - PART 3 (FINAL)
**Blog CRUD & Aggregations, Real-Time Gaming with Redis**

---

## 7.2.2 CRUD Operations

### User Operations

```javascript
const { MongoClient, ObjectId } = require('mongodb');

const client = new MongoClient('mongodb://localhost:27017');
const db = client.db('blog_platform');

// Register user
async function registerUser(userData) {
  const bcrypt = require('bcrypt');
  
  // Check if email/username exists
  const existing = await db.collection('users').findOne({
    $or: [
      { email: userData.email },
      { username: userData.username }
    ]
  });
  
  if (existing) {
    throw new Error('Email or username already exists');
  }
  
  // Hash password
  const passwordHash = await bcrypt.hash(userData.password, 12);
  
  const user = {
    email: userData.email,
    username: userData.username,
    passwordHash: passwordHash,
    profile: {
      firstName: userData.firstName,
      lastName: userData.lastName,
      bio: '',
      avatar: 'default-avatar.jpg'
    },
    stats: {
      postsCount: 0,
      followersCount: 0,
      followingCount: 0,
      totalLikes: 0
    },
    settings: {
      emailNotifications: true,
      publicProfile: true,
      theme: 'light'
    },
    createdAt: new Date(),
    updatedAt: new Date(),
    isActive: true,
    emailVerified: false
  };
  
  const result = await db.collection('users').insertOne(user);
  
  // Don't return password hash
  delete user.passwordHash;
  user._id = result.insertedId;
  
  return user;
}

// Update user profile
async function updateUserProfile(userId, updates) {
  const result = await db.collection('users').findOneAndUpdate(
    { _id: new ObjectId(userId) },
    {
      $set: {
        'profile.bio': updates.bio,
        'profile.website': updates.website,
        'profile.location': updates.location,
        'social.twitter': updates.twitter,
        'social.github': updates.github,
        updatedAt: new Date()
      }
    },
    { returnDocument: 'after' }
  );
  
  return result.value;
}

// Get user profile
async function getUserProfile(username) {
  const user = await db.collection('users').findOne(
    { username: username },
    { projection: { passwordHash: 0 } }  // Exclude password
  );
  
  if (!user) {
    throw new Error('User not found');
  }
  
  return user;
}
```

---

### Post Operations

**Create Post:**

```javascript
async function createPost(userId, postData) {
  const session = client.startSession();
  
  try {
    await session.withTransaction(async () => {
      // Get user info
      const user = await db.collection('users').findOne(
        { _id: new ObjectId(userId) },
        { projection: { username: 1, 'profile.firstName': 1, 'profile.lastName': 1, 'profile.avatar': 1 } }
      );
      
      if (!user) {
        throw new Error('User not found');
      }
      
      // Get category
      const category = await db.collection('categories').findOne(
        { slug: postData.categorySlug },
        { projection: { name: 1, slug: 1 } }
      );
      
      if (!category) {
        throw new Error('Category not found');
      }
      
      // Calculate reading time (simple: ~200 words/minute)
      const wordCount = postData.content.split(/\s+/).length;
      const readingTime = Math.ceil(wordCount / 200);
      
      // Generate slug from title
      const slug = postData.title
        .toLowerCase()
        .replace(/[^a-z0-9]+/g, '-')
        .replace(/^-|-$/g, '');
      
      const post = {
        title: postData.title,
        slug: slug,
        content: {
          markdown: postData.content,
          html: postData.html,
          excerpt: postData.excerpt || postData.content.substring(0, 200) + '...',
          readingTime: readingTime
        },
        author: {
          id: user._id,
          username: user.username,
          name: `${user.profile.firstName} ${user.profile.lastName}`,
          avatar: user.profile.avatar
        },
        category: {
          id: category._id,
          name: category.name,
          slug: category.slug
        },
        tags: postData.tags || [],
        coverImage: postData.coverImage || null,
        stats: {
          views: 0,
          likes: 0,
          comments: 0,
          bookmarks: 0,
          shares: 0
        },
        comments: [],
        status: postData.status || 'draft',
        publishedAt: postData.status === 'published' ? new Date() : null,
        createdAt: new Date(),
        updatedAt: new Date(),
        seo: {
          metaTitle: postData.title,
          metaDescription: postData.excerpt,
          keywords: postData.tags || []
        },
        allowComments: true,
        isPublic: true
      };
      
      const result = await db.collection('posts').insertOne(post);
      
      // Update user post count
      if (post.status === 'published') {
        await db.collection('users').updateOne(
          { _id: user._id },
          { $inc: { 'stats.postsCount': 1 } }
        );
        
        // Update category post count
        await db.collection('categories').updateOne(
          { _id: category._id },
          { $inc: { 'stats.postsCount': 1 } }
        );
      }
      
      post._id = result.insertedId;
      return post;
    });
    
  } finally {
    await session.endSession();
  }
}
```

---

**Get Posts (with pagination and filters):**

```javascript
async function getPosts(filters = {}, page = 1, limit = 20) {
  const query = { status: 'published' };
  
  // Filter by category
  if (filters.category) {
    query['category.slug'] = filters.category;
  }
  
  // Filter by tag
  if (filters.tag) {
    query.tags = filters.tag;
  }
  
  // Filter by author
  if (filters.author) {
    query['author.username'] = filters.author;
  }
  
  // Text search
  if (filters.search) {
    query.$text = { $search: filters.search };
  }
  
  // Sort options
  let sort = {};
  switch (filters.sort) {
    case 'popular':
      sort = { 'stats.views': -1 };
      break;
    case 'likes':
      sort = { 'stats.likes': -1 };
      break;
    case 'oldest':
      sort = { publishedAt: 1 };
      break;
    default:
      sort = { publishedAt: -1 };  // newest
  }
  
  const skip = (page - 1) * limit;
  
  const [posts, totalCount] = await Promise.all([
    db.collection('posts')
      .find(query)
      .sort(sort)
      .skip(skip)
      .limit(limit)
      .project({
        title: 1,
        slug: 1,
        'content.excerpt': 1,
        'content.readingTime': 1,
        author: 1,
        category: 1,
        tags: 1,
        coverImage: 1,
        stats: 1,
        publishedAt: 1
      })
      .toArray(),
    
    db.collection('posts').countDocuments(query)
  ]);
  
  return {
    posts,
    pagination: {
      page,
      limit,
      totalPages: Math.ceil(totalCount / limit),
      totalCount
    }
  };
}
```

---

**Update Post:**

```javascript
async function updatePost(postId, userId, updates) {
  // Verify ownership
  const post = await db.collection('posts').findOne({
    _id: new ObjectId(postId),
    'author.id': new ObjectId(userId)
  });
  
  if (!post) {
    throw new Error('Post not found or unauthorized');
  }
  
  const updateDoc = {
    $set: {
      title: updates.title,
      'content.markdown': updates.content,
      'content.html': updates.html,
      'content.excerpt': updates.excerpt,
      tags: updates.tags,
      updatedAt: new Date()
    }
  };
  
  // If publishing draft
  if (post.status === 'draft' && updates.status === 'published') {
    updateDoc.$set.status = 'published';
    updateDoc.$set.publishedAt = new Date();
  }
  
  const result = await db.collection('posts').findOneAndUpdate(
    { _id: new ObjectId(postId) },
    updateDoc,
    { returnDocument: 'after' }
  );
  
  return result.value;
}
```

---

**Delete Post:**

```javascript
async function deletePost(postId, userId) {
  const session = client.startSession();
  
  try {
    await session.withTransaction(async () => {
      const post = await db.collection('posts').findOne({
        _id: new ObjectId(postId),
        'author.id': new ObjectId(userId)
      });
      
      if (!post) {
        throw new Error('Post not found or unauthorized');
      }
      
      // Delete post
      await db.collection('posts').deleteOne({ _id: new ObjectId(postId) });
      
      // Delete associated likes
      await db.collection('likes').deleteMany({ postId: new ObjectId(postId) });
      
      // Delete associated bookmarks
      await db.collection('bookmarks').deleteMany({ postId: new ObjectId(postId) });
      
      // Update user stats
      if (post.status === 'published') {
        await db.collection('users').updateOne(
          { _id: new ObjectId(userId) },
          { 
            $inc: { 
              'stats.postsCount': -1,
              'stats.totalLikes': -post.stats.likes
            } 
          }
        );
        
        // Update category stats
        await db.collection('categories').updateOne(
          { _id: post.category.id },
          { $inc: { 'stats.postsCount': -1 } }
        );
      }
    });
  } finally {
    await session.endSession();
  }
}
```

---

### Comment Operations

```javascript
// Add comment
async function addComment(postId, userId, commentText) {
  const user = await db.collection('users').findOne(
    { _id: new ObjectId(userId) },
    { projection: { username: 1, 'profile.avatar': 1 } }
  );
  
  const comment = {
    _id: new ObjectId(),
    userId: user._id,
    username: user.username,
    userAvatar: user.profile.avatar,
    text: commentText,
    likes: 0,
    replies: [],
    createdAt: new Date(),
    updatedAt: new Date()
  };
  
  const result = await db.collection('posts').findOneAndUpdate(
    { _id: new ObjectId(postId) },
    {
      $push: { comments: { $each: [comment], $position: 0, $slice: 100 } },
      $inc: { 'stats.comments': 1 },
      $set: { updatedAt: new Date() }
    },
    { returnDocument: 'after' }
  );
  
  return comment;
}

// Add reply to comment
async function addReply(postId, commentId, userId, replyText) {
  const user = await db.collection('users').findOne(
    { _id: new ObjectId(userId) },
    { projection: { username: 1 } }
  );
  
  const reply = {
    _id: new ObjectId(),
    userId: user._id,
    username: user.username,
    text: replyText,
    createdAt: new Date()
  };
  
  await db.collection('posts').updateOne(
    { 
      _id: new ObjectId(postId),
      'comments._id': new ObjectId(commentId)
    },
    {
      $push: { 'comments.$.replies': reply },
      $set: { updatedAt: new Date() }
    }
  );
  
  return reply;
}

// Delete comment
async function deleteComment(postId, commentId, userId) {
  await db.collection('posts').updateOne(
    { _id: new ObjectId(postId) },
    {
      $pull: { 
        comments: { 
          _id: new ObjectId(commentId),
          userId: new ObjectId(userId)
        } 
      },
      $inc: { 'stats.comments': -1 }
    }
  );
}
```

---

### Social Operations

```javascript
// Like post
async function likePost(postId, userId) {
  const session = client.startSession();
  
  try {
    await session.withTransaction(async () => {
      // Check if already liked
      const existingLike = await db.collection('likes').findOne({
        userId: new ObjectId(userId),
        postId: new ObjectId(postId)
      });
      
      if (existingLike) {
        throw new Error('Post already liked');
      }
      
      // Create like
      await db.collection('likes').insertOne({
        userId: new ObjectId(userId),
        postId: new ObjectId(postId),
        createdAt: new Date()
      });
      
      // Increment post likes
      await db.collection('posts').updateOne(
        { _id: new ObjectId(postId) },
        { $inc: { 'stats.likes': 1 } }
      );
      
      // Increment author total likes
      const post = await db.collection('posts').findOne(
        { _id: new ObjectId(postId) },
        { projection: { 'author.id': 1 } }
      );
      
      await db.collection('users').updateOne(
        { _id: post.author.id },
        { $inc: { 'stats.totalLikes': 1 } }
      );
    });
  } finally {
    await session.endSession();
  }
}

// Unlike post
async function unlikePost(postId, userId) {
  const session = client.startSession();
  
  try {
    await session.withTransaction(async () => {
      const result = await db.collection('likes').deleteOne({
        userId: new ObjectId(userId),
        postId: new ObjectId(postId)
      });
      
      if (result.deletedCount === 0) {
        throw new Error('Like not found');
      }
      
      // Decrement counts
      await db.collection('posts').updateOne(
        { _id: new ObjectId(postId) },
        { $inc: { 'stats.likes': -1 } }
      );
      
      const post = await db.collection('posts').findOne(
        { _id: new ObjectId(postId) },
        { projection: { 'author.id': 1 } }
      );
      
      await db.collection('users').updateOne(
        { _id: post.author.id },
        { $inc: { 'stats.totalLikes': -1 } }
      );
    });
  } finally {
    await session.endSession();
  }
}

// Follow user
async function followUser(followerId, followingId) {
  const session = client.startSession();
  
  try {
    await session.withTransaction(async () => {
      // Get user info
      const [follower, following] = await Promise.all([
        db.collection('users').findOne(
          { _id: new ObjectId(followerId) },
          { projection: { username: 1, 'profile.firstName': 1, 'profile.lastName': 1, 'profile.avatar': 1 } }
        ),
        db.collection('users').findOne(
          { _id: new ObjectId(followingId) },
          { projection: { username: 1, 'profile.firstName': 1, 'profile.lastName': 1, 'profile.avatar': 1 } }
        )
      ]);
      
      // Create follow relationship
      await db.collection('follows').insertOne({
        followerId: new ObjectId(followerId),
        followingId: new ObjectId(followingId),
        follower: {
          username: follower.username,
          name: `${follower.profile.firstName} ${follower.profile.lastName}`,
          avatar: follower.profile.avatar
        },
        following: {
          username: following.username,
          name: `${following.profile.firstName} ${following.profile.lastName}`,
          avatar: following.profile.avatar
        },
        createdAt: new Date()
      });
      
      // Update counts
      await Promise.all([
        db.collection('users').updateOne(
          { _id: new ObjectId(followerId) },
          { $inc: { 'stats.followingCount': 1 } }
        ),
        db.collection('users').updateOne(
          { _id: new ObjectId(followingId) },
          { $inc: { 'stats.followersCount': 1 } }
        )
      ]);
    });
  } finally {
    await session.endSession();
  }
}
```

---

## 7.2.3 Aggregation Queries

### Popular Posts

```javascript
// Get trending posts (most views in last 7 days)
async function getTrendingPosts(limit = 10) {
  const sevenDaysAgo = new Date();
  sevenDaysAgo.setDate(sevenDaysAgo.getDate() - 7);
  
  const posts = await db.collection('posts').aggregate([
    {
      $match: {
        status: 'published',
        publishedAt: { $gte: sevenDaysAgo }
      }
    },
    {
      $addFields: {
        trendingScore: {
          $add: [
            { $multiply: ['$stats.views', 1] },
            { $multiply: ['$stats.likes', 5] },
            { $multiply: ['$stats.comments', 3] },
            { $multiply: ['$stats.shares', 10] }
          ]
        }
      }
    },
    {
      $sort: { trendingScore: -1 }
    },
    {
      $limit: limit
    },
    {
      $project: {
        title: 1,
        slug: 1,
        'content.excerpt': 1,
        author: 1,
        category: 1,
        coverImage: 1,
        stats: 1,
        publishedAt: 1,
        trendingScore: 1
      }
    }
  ]).toArray();
  
  return posts;
}
```

---

### User Feed (Following)

```javascript
// Get posts from users you follow
async function getUserFeed(userId, page = 1, limit = 20) {
  const skip = (page - 1) * limit;
  
  const feed = await db.collection('follows').aggregate([
    // Get users this user follows
    {
      $match: { followerId: new ObjectId(userId) }
    },
    {
      $lookup: {
        from: 'posts',
        let: { followingId: '$followingId' },
        pipeline: [
          {
            $match: {
              $expr: { $eq: ['$author.id', '$$followingId'] },
              status: 'published'
            }
          },
          { $sort: { publishedAt: -1 } }
        ],
        as: 'posts'
      }
    },
    {
      $unwind: '$posts'
    },
    {
      $replaceRoot: { newRoot: '$posts' }
    },
    {
      $sort: { publishedAt: -1 }
    },
    {
      $skip: skip
    },
    {
      $limit: limit
    },
    {
      $project: {
        title: 1,
        slug: 1,
        'content.excerpt': 1,
        'content.readingTime': 1,
        author: 1,
        category: 1,
        tags: 1,
        coverImage: 1,
        stats: 1,
        publishedAt: 1
      }
    }
  ]).toArray();
  
  return feed;
}
```

---

### Category Statistics

```javascript
// Get detailed category stats
async function getCategoryStats() {
  const stats = await db.collection('posts').aggregate([
    {
      $match: { status: 'published' }
    },
    {
      $group: {
        _id: '$category.slug',
        categoryName: { $first: '$category.name' },
        postsCount: { $sum: 1 },
        totalViews: { $sum: '$stats.views' },
        totalLikes: { $sum: '$stats.likes' },
        avgViews: { $avg: '$stats.views' },
        avgLikes: { $avg: '$stats.likes' }
      }
    },
    {
      $sort: { postsCount: -1 }
    }
  ]).toArray();
  
  return stats;
}
```

---

### Top Authors

```javascript
// Get top authors by engagement
async function getTopAuthors(limit = 10) {
  const authors = await db.collection('posts').aggregate([
    {
      $match: { status: 'published' }
    },
    {
      $group: {
        _id: '$author.id',
        username: { $first: '$author.username' },
        name: { $first: '$author.name' },
        avatar: { $first: '$author.avatar' },
        postsCount: { $sum: 1 },
        totalViews: { $sum: '$stats.views' },
        totalLikes: { $sum: '$stats.likes' },
        totalComments: { $sum: '$stats.comments' }
      }
    },
    {
      $addFields: {
        engagementScore: {
          $add: [
            { $multiply: ['$totalViews', 1] },
            { $multiply: ['$totalLikes', 5] },
            { $multiply: ['$totalComments', 3] }
          ]
        }
      }
    },
    {
      $sort: { engagementScore: -1 }
    },
    {
      $limit: limit
    }
  ]).toArray();
  
  return authors;
}
```

---

### Tag Cloud

```javascript
// Get popular tags with counts
async function getPopularTags(limit = 50) {
  const tags = await db.collection('posts').aggregate([
    {
      $match: { status: 'published' }
    },
    {
      $unwind: '$tags'
    },
    {
      $group: {
        _id: '$tags',
        count: { $sum: 1 }
      }
    },
    {
      $sort: { count: -1 }
    },
    {
      $limit: limit
    },
    {
      $project: {
        _id: 0,
        tag: '$_id',
        count: 1
      }
    }
  ]).toArray();
  
  return tags;
}
```

---

### User Activity Summary

```javascript
// Get user's complete activity stats
async function getUserActivitySummary(userId) {
  const summary = await db.collection('posts').aggregate([
    {
      $facet: {
        // Posts by this user
        userPosts: [
          {
            $match: {
              'author.id': new ObjectId(userId),
              status: 'published'
            }
          },
          {
            $group: {
              _id: null,
              totalPosts: { $sum: 1 },
              totalViews: { $sum: '$stats.views' },
              totalLikes: { $sum: '$stats.likes' },
              totalComments: { $sum: '$stats.comments' }
            }
          }
        ],
        
        // Posts by category
        postsByCategory: [
          {
            $match: {
              'author.id': new ObjectId(userId),
              status: 'published'
            }
          },
          {
            $group: {
              _id: '$category.name',
              count: { $sum: 1 }
            }
          },
          {
            $sort: { count: -1 }
          }
        ],
        
        // Most used tags
        topTags: [
          {
            $match: {
              'author.id': new ObjectId(userId),
              status: 'published'
            }
          },
          {
            $unwind: '$tags'
          },
          {
            $group: {
              _id: '$tags',
              count: { $sum: 1 }
            }
          },
          {
            $sort: { count: -1 }
          },
          {
            $limit: 10
          }
        ],
        
        // Publishing timeline
        publishingTimeline: [
          {
            $match: {
              'author.id': new ObjectId(userId),
              status: 'published'
            }
          },
          {
            $group: {
              _id: {
                year: { $year: '$publishedAt' },
                month: { $month: '$publishedAt' }
              },
              count: { $sum: 1 }
            }
          },
          {
            $sort: { '_id.year': -1, '_id.month': -1 }
          },
          {
            $limit: 12
          }
        ]
      }
    }
  ]).toArray();
  
  return summary[0];
}
```

---

### Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

#### Fill in the Blanks

1. In MongoDB, ________ documents (like author info in posts) are used for data read frequently but changed rarely.
2. The ________ index enables full-text search across title, tags, and content.
3. Comments are embedded with a limit of ________ to prevent unbounded document growth.
4. The ________ collection uses a compound unique index on followerId and followingId to prevent duplicate follows.
5. ________ allows applying different importance weights to fields in text search.
6. MongoDB ________ provide a powerful way to process and transform documents across multiple stages.
7. The $________ stage in aggregation is used to deconstruct an array field into separate documents.
8. The $________ operator in updates can be used to push elements to an array with position and slice controls.
9. ________ in MongoDB allow multiple operations to execute atomically across documents.
10. The ________ option in findOneAndUpdate returns the modified document instead of the original.

<details>
<summary><strong>View Answers</strong></summary>

1. Embedded
2. text
3. 100
4. follows
5. Weights (or weights parameter)
6. aggregations (or aggregation pipelines)
7. unwind
8. push
9. Transactions (or sessions)
10. returnDocument: 'after'

</details>

</details>

---
