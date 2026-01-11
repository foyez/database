# Chapter 3: NoSQL Deep Dive
**MongoDB Operations, Patterns, and Best Practices**

**Navigation:** [← SQL Essentials: Relationships, JOINs, and Aggregations](02_sql_essentials.md) | [README](../README.md) | [Next: NoSQL Deep Dive: Redis →](03_01_redis.md)

---

## Table of Contents
- [3.1 MongoDB](#31-mongodb)
  - [3.1.1 CRUD Operations](#311-crud-operations)
  - [3.1.2 Embedded vs Referenced Documents](#312-embedded-vs-referenced-documents)
  - [3.1.3 Aggregation Pipeline](#313-aggregation-pipeline)
  - [3.1.4 Schema Design Patterns](#314-schema-design-patterns)
  - [3.1.5 Indexing in MongoDB](#315-indexing-in-mongodb)

---

## 3.1 MongoDB

### What is MongoDB?

**MongoDB** is a document-oriented NoSQL database that stores data in flexible, JSON-like documents (BSON format). It's designed for scalability, high performance, and developer productivity.

**Key Features:**
- Schema-less (flexible documents)
- Horizontal scaling (sharding)
- Rich query language
- Aggregation framework
- Geospatial queries
- Full-text search

**When to Use MongoDB:**
- Flexible, evolving schemas
- Hierarchical data structures
- High write throughput
- Rapid prototyping
- Content management systems
- Real-time analytics

---

### 3.1.1 CRUD Operations

CRUD stands for **Create, Read, Update, Delete** - the four basic database operations.

---

#### Create (Insert) Operations

**Insert One Document:**

```javascript
// Insert single document
db.users.insertOne({
  name: "Alice Johnson",
  email: "alice@example.com",
  age: 28,
  city: "New York",
  interests: ["reading", "coding", "travel"],
  createdAt: new Date()
});

// Result:
{
  acknowledged: true,
  insertedId: ObjectId("507f1f77bcf86cd799439011")
}
```

**Insert Many Documents:**

```javascript
// Insert multiple documents
db.products.insertMany([
  {
    name: "Laptop",
    category: "Electronics",
    price: 999.99,
    stock: 50,
    specs: {
      cpu: "Intel i7",
      ram: "16GB",
      storage: "512GB SSD"
    }
  },
  {
    name: "Wireless Mouse",
    category: "Electronics",
    price: 29.99,
    stock: 200
  },
  {
    name: "Desk Chair",
    category: "Furniture",
    price: 199.99,
    stock: 30
  }
]);

// Result:
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId("..."),
    '1': ObjectId("..."),
    '2': ObjectId("...")
  }
}
```

---

#### Read (Query) Operations

**Find All Documents:**

```javascript
// Get all users
db.users.find();

// Get all with pretty formatting
db.users.find().pretty();
```

**Find with Filter:**

```javascript
// Find users from New York
db.users.find({ city: "New York" });

// Find users older than 25
db.users.find({ age: { $gt: 25 } });

// Find products under $100
db.products.find({ price: { $lt: 100 } });

// Find by multiple conditions (AND)
db.users.find({ 
  city: "New York", 
  age: { $gte: 25 } 
});

// Find by OR condition
db.users.find({
  $or: [
    { city: "New York" },
    { city: "Los Angeles" }
  ]
});
```

**Query Operators:**

```javascript
// Comparison operators
db.products.find({ price: { $eq: 99.99 } });    // Equal
db.products.find({ price: { $ne: 99.99 } });    // Not equal
db.products.find({ price: { $gt: 100 } });      // Greater than
db.products.find({ price: { $gte: 100 } });     // Greater than or equal
db.products.find({ price: { $lt: 100 } });      // Less than
db.products.find({ price: { $lte: 100 } });     // Less than or equal
db.products.find({ category: { $in: ["Electronics", "Furniture"] } });
db.products.find({ category: { $nin: ["Clothing"] } });

// Logical operators
db.products.find({
  $and: [
    { category: "Electronics" },
    { price: { $lt: 1000 } }
  ]
});

db.users.find({
  $or: [
    { age: { $lt: 25 } },
    { age: { $gt: 60 } }
  ]
});

db.products.find({ 
  price: { $not: { $gt: 1000 } } 
});

// Element operators
db.users.find({ phone: { $exists: true } });    // Has phone field
db.users.find({ phone: { $exists: false } });   // No phone field
db.users.find({ name: { $type: "string" } });   // Field type check

// Array operators
db.users.find({ interests: "coding" });         // Array contains
db.users.find({ interests: { $all: ["coding", "reading"] } });
db.users.find({ interests: { $size: 3 } });     // Array size
```

**Find One Document:**

```javascript
// Find first matching document
db.users.findOne({ email: "alice@example.com" });

// Find by ID
db.users.findOne({ _id: ObjectId("507f1f77bcf86cd799439011") });
```

**Projection (Select Specific Fields):**

```javascript
// Return only name and email (exclude _id)
db.users.find(
  { city: "New York" },
  { name: 1, email: 1, _id: 0 }
);

// Exclude specific fields
db.users.find(
  {},
  { password: 0, ssn: 0 }
);
```

**Sorting, Limiting, and Skipping:**

```javascript
// Sort by age ascending
db.users.find().sort({ age: 1 });

// Sort by age descending
db.users.find().sort({ age: -1 });

// Multiple sort fields
db.products.find().sort({ category: 1, price: -1 });

// Limit results
db.users.find().limit(10);

// Skip first 10 results (pagination)
db.users.find().skip(10).limit(10);

// Combine: Get page 2 of products (10 per page)
db.products
  .find({ category: "Electronics" })
  .sort({ price: 1 })
  .skip(10)
  .limit(10);

// Count documents
db.users.countDocuments({ city: "New York" });
```

---

#### Update Operations

**Update One Document:**

```javascript
// Update single document
db.users.updateOne(
  { email: "alice@example.com" },  // Filter
  { 
    $set: { 
      age: 29,
      city: "San Francisco" 
    } 
  }
);

// Result:
{
  acknowledged: true,
  matchedCount: 1,
  modifiedCount: 1
}
```

**Update Many Documents:**

```javascript
// Update all users from New York
db.users.updateMany(
  { city: "New York" },
  { $set: { timezone: "EST" } }
);

// Increase all product prices by 10%
db.products.updateMany(
  { category: "Electronics" },
  { $mul: { price: 1.1 } }
);
```

**Update Operators:**

```javascript
// $set - Set field value
db.users.updateOne(
  { _id: ObjectId("...") },
  { $set: { status: "active" } }
);

// $unset - Remove field
db.users.updateOne(
  { _id: ObjectId("...") },
  { $unset: { tempField: "" } }
);

// $inc - Increment numeric value
db.products.updateOne(
  { name: "Laptop" },
  { $inc: { stock: -1 } }  // Decrease by 1
);

// $mul - Multiply value
db.products.updateOne(
  { name: "Laptop" },
  { $mul: { price: 1.15 } }  // Increase by 15%
);

// $min - Update if new value is less than current
db.products.updateOne(
  { name: "Laptop" },
  { $min: { price: 899.99 } }  // Set to 899.99 if current is higher
);

// $max - Update if new value is greater than current
db.products.updateOne(
  { name: "Laptop" },
  { $max: { price: 999.99 } }  // Set to 999.99 if current is lower
);

// $rename - Rename field
db.users.updateMany(
  {},
  { $rename: { "phone": "phoneNumber" } }
);

// $currentDate - Set to current date
db.users.updateOne(
  { email: "alice@example.com" },
  { $currentDate: { lastModified: true } }
);
```

**Array Update Operators:**

```javascript
// $push - Add element to array
db.users.updateOne(
  { email: "alice@example.com" },
  { $push: { interests: "photography" } }
);

// $push with $each (add multiple)
db.users.updateOne(
  { email: "alice@example.com" },
  { 
    $push: { 
      interests: { 
        $each: ["gaming", "music"] 
      } 
    } 
  }
);

// $pull - Remove element from array
db.users.updateOne(
  { email: "alice@example.com" },
  { $pull: { interests: "gaming" } }
);

// $addToSet - Add only if not exists (unique)
db.users.updateOne(
  { email: "alice@example.com" },
  { $addToSet: { interests: "coding" } }  // Won't duplicate
);

// $pop - Remove first (-1) or last (1) element
db.users.updateOne(
  { email: "alice@example.com" },
  { $pop: { interests: 1 } }  // Remove last
);
```

**Replace One Document:**

```javascript
// Replace entire document (except _id)
db.users.replaceOne(
  { email: "alice@example.com" },
  {
    name: "Alice Johnson",
    email: "alice@example.com",
    age: 29,
    status: "premium"
    // All other fields removed!
  }
);
```

**Upsert (Update or Insert):**

```javascript
// Update if exists, insert if not
db.users.updateOne(
  { email: "bob@example.com" },
  { 
    $set: { 
      name: "Bob Smith",
      age: 35 
    } 
  },
  { upsert: true }  // Create if doesn't exist
);
```

---

#### Delete Operations

**Delete One Document:**

```javascript
// Delete single document
db.users.deleteOne({ email: "alice@example.com" });

// Result:
{
  acknowledged: true,
  deletedCount: 1
}
```

**Delete Many Documents:**

```javascript
// Delete all inactive users
db.users.deleteMany({ status: "inactive" });

// Delete all products out of stock
db.products.deleteMany({ stock: 0 });

// Delete all documents (dangerous!)
db.users.deleteMany({});
```

---

### Real-World CRUD Example: Blog Platform

```javascript
// 1. CREATE: Add new blog post
db.posts.insertOne({
  title: "Getting Started with MongoDB",
  author: {
    id: ObjectId("507f1f77bcf86cd799439011"),
    name: "Alice Johnson"
  },
  content: "MongoDB is a powerful NoSQL database...",
  tags: ["mongodb", "nosql", "database"],
  views: 0,
  likes: [],
  comments: [],
  status: "published",
  createdAt: new Date(),
  updatedAt: new Date()
});

// 2. READ: Get all published posts with author name
db.posts.find(
  { status: "published" },
  { title: 1, "author.name": 1, createdAt: 1, views: 1 }
).sort({ createdAt: -1 }).limit(10);

// 3. UPDATE: Increment views when post is read
db.posts.updateOne(
  { _id: ObjectId("...") },
  { 
    $inc: { views: 1 },
    $currentDate: { lastViewed: true }
  }
);

// 4. UPDATE: User likes a post
db.posts.updateOne(
  { _id: ObjectId("...") },
  { 
    $addToSet: { likes: ObjectId("user-id") }  // Add user ID to likes
  }
);

// 5. UPDATE: Add comment to post
db.posts.updateOne(
  { _id: ObjectId("...") },
  {
    $push: {
      comments: {
        userId: ObjectId("user-id"),
        userName: "Bob Smith",
        text: "Great article!",
        createdAt: new Date()
      }
    }
  }
);

// 6. READ: Find posts by tag
db.posts.find({ 
  tags: "mongodb",
  status: "published" 
}).sort({ views: -1 });

// 7. UPDATE: Update post content
db.posts.updateOne(
  { _id: ObjectId("...") },
  {
    $set: {
      content: "Updated content...",
      updatedAt: new Date()
    }
  }
);

// 8. DELETE: Remove post
db.posts.deleteOne({ _id: ObjectId("...") });
```

---

### Practice Questions: CRUD Operations

#### Fill in the Blanks

1. The method ________ inserts a single document into a collection.
2. To find documents where age is greater than 25, use the ________ operator.
3. The ________ operator adds an element to an array only if it doesn't exist.
4. Use ________ to update a document if it exists, or insert it if it doesn't.
5. The method ________ removes all documents matching the filter.

<details>
<summary><strong>View Answers</strong></summary>

1. insertOne()
2. $gt
3. $addToSet
4. upsert: true
5. deleteMany()

</details>

---

#### True/False

1. MongoDB automatically creates an _id field if not provided.
2. The find() method returns a cursor, not an array.
3. updateOne() with $set replaces the entire document.
4. The $push operator can add multiple elements with $each.
5. You can sort by multiple fields in MongoDB.

<details>
<summary><strong>View Answers</strong></summary>

1. TRUE
2. TRUE
3. FALSE (only updates specified fields)
4. TRUE
5. TRUE

</details>

---

#### Coding Challenges

**Challenge 1: E-Commerce Product Search**

Write a query to find all electronics products under $500, in stock, sorted by price ascending.

<details>
<summary><strong>View Solution</strong></summary>

```javascript
db.products.find({
  category: "Electronics",
  price: { $lt: 500 },
  stock: { $gt: 0 }
}).sort({ price: 1 });
```

</details>

---

**Challenge 2: User Activity Update**

When a user logs in, increment their login count and update their last login timestamp.

<details>
<summary><strong>View Solution</strong></summary>

```javascript
db.users.updateOne(
  { _id: ObjectId("user-id") },
  {
    $inc: { loginCount: 1 },
    $currentDate: { lastLogin: true }
  }
);
```

</details>

---

**Challenge 3: Pagination**

Get page 3 of products (10 products per page) sorted by price.

<details>
<summary><strong>View Solution</strong></summary>

```javascript
db.products
  .find()
  .sort({ price: 1 })
  .skip(20)  // Skip first 2 pages (2 * 10)
  .limit(10);
```

</details>

---

## 3.1.2 Embedded vs Referenced Documents

**Two ways to model relationships in MongoDB:**

1. **Embedded Documents** - Nest related data inside parent document
2. **Referenced Documents** - Store references (IDs) to related documents

---

#### Embedded Documents (Denormalization)

**Definition:** Store related data within the same document.

**When to Use:**
- One-to-One relationships
- One-to-Few relationships
- Data accessed together frequently
- Read performance is critical

**Example: Blog Post with Comments**

```javascript
// Embedded approach: Comments inside post
{
  _id: ObjectId("post-id"),
  title: "Introduction to MongoDB",
  content: "MongoDB is a NoSQL database...",
  author: {
    id: ObjectId("author-id"),
    name: "Alice Johnson",
    email: "alice@example.com"
  },
  comments: [
    {
      id: ObjectId("comment-1"),
      userId: ObjectId("user-1"),
      userName: "Bob Smith",
      text: "Great article!",
      createdAt: ISODate("2024-01-15T10:00:00Z")
    },
    {
      id: ObjectId("comment-2"),
      userId: ObjectId("user-2"),
      userName: "Carol White",
      text: "Very informative.",
      createdAt: ISODate("2024-01-15T11:30:00Z")
    }
  ],
  createdAt: ISODate("2024-01-15T09:00:00Z")
}

// Query: Get post with all comments (single read)
db.posts.findOne({ _id: ObjectId("post-id") });

// Update: Add new comment
db.posts.updateOne(
  { _id: ObjectId("post-id") },
  {
    $push: {
      comments: {
        id: ObjectId(),
        userId: ObjectId("user-3"),
        userName: "David Lee",
        text: "Thanks for sharing!",
        createdAt: new Date()
      }
    }
  }
);
```

**Advantages:**
- ✅ Single query to get all data
- ✅ Better read performance
- ✅ Atomic updates (all in one operation)
- ✅ No JOINs needed

**Disadvantages:**
- ❌ Document size limit (16MB)
- ❌ Data duplication
- ❌ Harder to query nested data
- ❌ Unbounded growth (arrays can get huge)

---

#### Referenced Documents (Normalization)

**Definition:** Store references (ObjectIds) to related documents in separate collections.

**When to Use:**
- One-to-Many (large scale)
- Many-to-Many relationships
- Data updated independently
- Large subdocuments
- Unbounded growth

**Example: Blog Post with Referenced Comments**

```javascript
// Posts collection
{
  _id: ObjectId("post-id"),
  title: "Introduction to MongoDB",
  content: "MongoDB is a NoSQL database...",
  authorId: ObjectId("author-id"),  // Reference to user
  createdAt: ISODate("2024-01-15T09:00:00Z")
}

// Comments collection (separate)
{
  _id: ObjectId("comment-1"),
  postId: ObjectId("post-id"),  // Reference to post
  userId: ObjectId("user-1"),   // Reference to user
  text: "Great article!",
  createdAt: ISODate("2024-01-15T10:00:00Z")
}

{
  _id: ObjectId("comment-2"),
  postId: ObjectId("post-id"),
  userId: ObjectId("user-2"),
  text: "Very informative.",
  createdAt: ISODate("2024-01-15T11:30:00Z")
}

// Query: Get post with comments (requires aggregation)
db.posts.aggregate([
  { $match: { _id: ObjectId("post-id") } },
  {
    $lookup: {
      from: "comments",
      localField: "_id",
      foreignField: "postId",
      as: "comments"
    }
  },
  {
    $lookup: {
      from: "users",
      localField: "authorId",
      foreignField: "_id",
      as: "author"
    }
  },
  { $unwind: "$author" }
]);

// Insert: Add new comment (separate operation)
db.comments.insertOne({
  postId: ObjectId("post-id"),
  userId: ObjectId("user-3"),
  text: "Thanks for sharing!",
  createdAt: new Date()
});
```

**Advantages:**
- ✅ No document size limit
- ✅ No data duplication
- ✅ Easier to update related data
- ✅ Better for unbounded relationships

**Disadvantages:**
- ❌ Multiple queries (or complex aggregations)
- ❌ Slower reads
- ❌ No atomic updates across collections
- ❌ Need to maintain referential integrity manually

---

#### Decision Guide: Embedded vs Referenced

| Factor | Embedded | Referenced |
|--------|----------|------------|
| **Relationship Size** | One-to-One, One-to-Few | One-to-Many, Many-to-Many |
| **Read Pattern** | Frequently accessed together | Accessed independently |
| **Update Frequency** | Rarely updated | Frequently updated |
| **Data Size** | Small subdocuments | Large subdocuments |
| **Growth** | Bounded (known limit) | Unbounded (unlimited) |
| **Consistency** | Atomic updates | Eventual consistency |

---

#### Hybrid Approach

**Best Practice:** Combine both strategies!

```javascript
// E-Commerce Order: Mix of embedded and referenced
{
  _id: ObjectId("order-id"),
  orderNumber: "ORD-2024-001",
  
  // EMBEDDED: User snapshot (data at time of order)
  customer: {
    id: ObjectId("user-id"),  // Reference for lookups
    name: "Alice Johnson",
    email: "alice@example.com",
    shippingAddress: {
      street: "123 Main St",
      city: "New York",
      zip: "10001"
    }
  },
  
  // EMBEDDED: Order items (snapshot with prices at order time)
  items: [
    {
      productId: ObjectId("product-1"),  // Reference for current info
      name: "Laptop",
      quantity: 1,
      priceAtOrder: 999.99,  // Historical price
      sku: "LAP-001"
    },
    {
      productId: ObjectId("product-2"),
      name: "Mouse",
      quantity: 2,
      priceAtOrder: 29.99,
      sku: "MOU-001"
    }
  ],
  
  total: 1059.97,
  status: "shipped",
  
  // REFERENCED: Payment details (separate for security)
  paymentId: ObjectId("payment-id"),
  
  createdAt: ISODate("2024-01-15T10:00:00Z"),
  updatedAt: ISODate("2024-01-16T14:30:00Z")
}
```

**Why Hybrid?**
- Customer data embedded: Historical record (what if user changes address?)
- Product references: Can look up current product info
- Payment referenced: Security (separate sensitive data)

---

### Practice Questions: Embedded vs Referenced

#### Multiple Choice

**Q1: When should you use embedded documents?**

A) Unbounded relationships
B) Data accessed independently
C) Data accessed together frequently
D) Very large subdocuments

<details>
<summary><strong>View Answer</strong></summary>

**Answer: C - Data accessed together frequently**

Explanation: Embedded documents are ideal when related data is always accessed together, providing better read performance with a single query.

</details>

---

**Q2: What is the main disadvantage of embedding?**

A) Slower reads
B) Document size limit (16MB)
C) Requires JOINs
D) No atomic updates

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - Document size limit (16MB)**

Explanation: MongoDB has a 16MB document size limit. Embedding large arrays can exceed this limit, requiring referenced documents instead.

</details>

---

## 3.1.3 Aggregation Pipeline

**Definition:** A framework for data processing that transforms documents through a sequence of stages.

**Think of it as:** A data processing pipeline where each stage transforms the data and passes it to the next stage.

```
Input Collection → Stage 1 → Stage 2 → Stage 3 → Output
```

---

#### Basic Aggregation Example

```javascript
// Calculate total sales by category
db.products.aggregate([
  // Stage 1: Filter active products
  { $match: { status: "active" } },
  
  // Stage 2: Group by category
  {
    $group: {
      _id: "$category",
      totalSales: { $sum: "$sold" },
      avgPrice: { $avg: "$price" },
      count: { $sum: 1 }
    }
  },
  
  // Stage 3: Sort by total sales
  { $sort: { totalSales: -1 } },
  
  // Stage 4: Limit to top 5
  { $limit: 5 }
]);

// Result:
[
  { _id: "Electronics", totalSales: 5000, avgPrice: 299.99, count: 50 },
  { _id: "Furniture", totalSales: 3000, avgPrice: 399.99, count: 30 },
  { _id: "Clothing", totalSales: 2000, avgPrice: 49.99, count: 100 }
]
```

---

#### Aggregation Stages

**1. $match - Filter Documents**

```javascript
// Filter products by category
db.products.aggregate([
  { $match: { category: "Electronics", price: { $lt: 500 } } }
]);

// Multiple conditions
db.orders.aggregate([
  { 
    $match: { 
      status: "completed",
      createdAt: { $gte: new Date("2024-01-01") }
    } 
  }
]);
```

---

**2. $group - Group and Aggregate**

```javascript
// Group by category, calculate stats
db.products.aggregate([
  {
    $group: {
      _id: "$category",               // Group by category
      totalProducts: { $sum: 1 },     // Count products
      avgPrice: { $avg: "$price" },   // Average price
      maxPrice: { $max: "$price" },   // Max price
      minPrice: { $min: "$price" },   // Min price
      totalRevenue: { $sum: { $multiply: ["$price", "$sold"] } }
    }
  }
]);

// Group by multiple fields
db.orders.aggregate([
  {
    $group: {
      _id: {
        year: { $year: "$createdAt" },
        month: { $month: "$createdAt" }
      },
      orderCount: { $sum: 1 },
      revenue: { $sum: "$total" }
    }
  }
]);

// Accumulator operators in $group:
// $sum, $avg, $min, $max, $first, $last, $push, $addToSet
```

---

**3. $project - Shape Output**

```javascript
// Select and reshape fields
db.users.aggregate([
  {
    $project: {
      _id: 0,                    // Exclude _id
      name: 1,                   // Include name
      email: 1,                  // Include email
      age: 1,                    // Include age
      isAdult: { $gte: ["$age", 18] }  // Computed field
    }
  }
]);

// Create computed fields
db.products.aggregate([
  {
    $project: {
      name: 1,
      price: 1,
      discountedPrice: { $multiply: ["$price", 0.9] },  // 10% off
      priceWithTax: { $multiply: ["$price", 1.1] },     // 10% tax
      inStock: { $gt: ["$stock", 0] }                   // Boolean
    }
  }
]);

// String operations
db.users.aggregate([
  {
    $project: {
      fullName: { $concat: ["$firstName", " ", "$lastName"] },
      upperEmail: { $toUpper: "$email" },
      emailDomain: { 
        $arrayElemAt: [{ $split: ["$email", "@"] }, 1] 
      }
    }
  }
]);
```

---

**4. $sort - Sort Results**

```javascript
// Sort by single field
db.products.aggregate([
  { $sort: { price: -1 } }  // Descending
]);

// Sort by multiple fields
db.orders.aggregate([
  { $sort: { status: 1, createdAt: -1 } }
]);
```

---

**5. $limit and $skip - Pagination**

```javascript
// Get top 10 most expensive products
db.products.aggregate([
  { $sort: { price: -1 } },
  { $limit: 10 }
]);

// Pagination: Page 2 (10 per page)
db.products.aggregate([
  { $sort: { price: -1 } },
  { $skip: 10 },
  { $limit: 10 }
]);
```

---

**6. $lookup - Join Collections**

```javascript
// Join orders with users
db.orders.aggregate([
  {
    $lookup: {
      from: "users",              // Collection to join
      localField: "userId",       // Field in orders
      foreignField: "_id",        // Field in users
      as: "customer"              // Output array name
    }
  },
  { $unwind: "$customer" }        // Convert array to object
]);

// Multiple lookups
db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "customer"
    }
  },
  {
    $lookup: {
      from: "products",
      localField: "productId",
      foreignField: "_id",
      as: "product"
    }
  },
  { $unwind: "$customer" },
  { $unwind: "$product" }
]);
```

---

**7. $unwind - Deconstruct Arrays**

```javascript
// Sample document
{
  _id: 1,
  name: "Product A",
  tags: ["electronics", "gadgets", "popular"]
}

// Unwind tags array
db.products.aggregate([
  { $unwind: "$tags" }
]);

// Result (one document per tag):
{ _id: 1, name: "Product A", tags: "electronics" }
{ _id: 1, name: "Product A", tags: "gadgets" }
{ _id: 1, name: "Product A", tags: "popular" }

// Use case: Count occurrences of each tag
db.products.aggregate([
  { $unwind: "$tags" },
  { $group: { _id: "$tags", count: { $sum: 1 } } },
  { $sort: { count: -1 } }
]);
```

---

**8. $addFields - Add New Fields**

```javascript
// Add computed fields without removing existing ones
db.products.aggregate([
  {
    $addFields: {
      totalValue: { $multiply: ["$price", "$stock"] },
      discountPrice: { $multiply: ["$price", 0.9] },
      hasDiscount: { $gt: ["$discount", 0] }
    }
  }
]);
```

---

#### Real-World Aggregation Examples

**Example 1: E-Commerce Sales Report**

```javascript
// Monthly sales report with top products
db.orders.aggregate([
  // Stage 1: Filter completed orders from 2024
  {
    $match: {
      status: "completed",
      createdAt: { $gte: new Date("2024-01-01") }
    }
  },
  
  // Stage 2: Unwind order items
  { $unwind: "$items" },
  
  // Stage 3: Group by month and product
  {
    $group: {
      _id: {
        year: { $year: "$createdAt" },
        month: { $month: "$createdAt" },
        productId: "$items.productId"
      },
      productName: { $first: "$items.name" },
      totalQuantity: { $sum: "$items.quantity" },
      totalRevenue: { $sum: { $multiply: ["$items.quantity", "$items.price"] } }
    }
  },
  
  // Stage 4: Sort by revenue
  { $sort: { totalRevenue: -1 } },
  
  // Stage 5: Group by month (get top products per month)
  {
    $group: {
      _id: { year: "$_id.year", month: "$_id.month" },
      topProducts: {
        $push: {
          name: "$productName",
          quantity: "$totalQuantity",
          revenue: "$totalRevenue"
        }
      },
      monthlyRevenue: { $sum: "$totalRevenue" }
    }
  },
  
  // Stage 6: Limit top 5 products per month
  {
    $project: {
      topProducts: { $slice: ["$topProducts", 5] },
      monthlyRevenue: 1
    }
  },
  
  // Stage 7: Sort by date
  { $sort: { "_id.year": 1, "_id.month": 1 } }
]);
```

---

**Example 2: User Analytics Dashboard**

```javascript
// User engagement metrics
db.users.aggregate([
  // Join with orders
  {
    $lookup: {
      from: "orders",
      localField: "_id",
      foreignField: "userId",
      as: "orders"
    }
  },
  
  // Join with reviews
  {
    $lookup: {
      from: "reviews",
      localField: "_id",
      foreignField: "userId",
      as: "reviews"
    }
  },
  
  // Add computed metrics
  {
    $addFields: {
      orderCount: { $size: "$orders" },
      reviewCount: { $size: "$reviews" },
      totalSpent: { $sum: "$orders.total" },
      avgOrderValue: { $avg: "$orders.total" },
      lastOrderDate: { $max: "$orders.createdAt" }
    }
  },
  
  // Classify users
  {
    $addFields: {
      userSegment: {
        $switch: {
          branches: [
            { case: { $gte: ["$totalSpent", 1000] }, then: "VIP" },
            { case: { $gte: ["$totalSpent", 500] }, then: "Premium" },
            { case: { $gte: ["$orderCount", 5] }, then: "Regular" }
          ],
          default: "New"
        }
      }
    }
  },
  
  // Project final shape
  {
    $project: {
      name: 1,
      email: 1,
      orderCount: 1,
      totalSpent: 1,
      avgOrderValue: 1,
      userSegment: 1,
      lastOrderDate: 1
    }
  },
  
  // Sort by total spent
  { $sort: { totalSpent: -1 } }
]);
```

---

### Practice Questions: Aggregation Pipeline

#### Fill in the Blanks

1. The ________ stage filters documents in an aggregation pipeline.
2. Use ________ to join two collections in MongoDB aggregation.
3. The ________ stage deconstructs an array field into separate documents.
4. ________ is used to reshape documents and select specific fields.
5. The ________ accumulator calculates the average value in $group.

<details>
<summary><strong>View Answers</strong></summary>

1. $match
2. $lookup
3. $unwind
4. $project
5. $avg

</details>

---

#### Coding Challenges

**Challenge 1: Top Customers**

Find the top 5 customers by total spending from the orders collection.

<details>
<summary><strong>View Solution</strong></summary>

```javascript
db.orders.aggregate([
  {
    $group: {
      _id: "$userId",
      totalSpent: { $sum: "$total" },
      orderCount: { $sum: 1 }
    }
  },
  { $sort: { totalSpent: -1 } },
  { $limit: 5 },
  {
    $lookup: {
      from: "users",
      localField: "_id",
      foreignField: "_id",
      as: "user"
    }
  },
  { $unwind: "$user" },
  {
    $project: {
      userName: "$user.name",
      email: "$user.email",
      totalSpent: 1,
      orderCount: 1
    }
  }
]);
```

</details>

---

**Challenge 2: Product Tag Analysis**

Count how many products use each tag, sorted by frequency.

<details>
<summary><strong>View Solution</strong></summary>

```javascript
db.products.aggregate([
  { $unwind: "$tags" },
  {
    $group: {
      _id: "$tags",
      count: { $sum: 1 },
      products: { $push: "$name" }
    }
  },
  { $sort: { count: -1 } }
]);
```

</details>

---

## 3.1.4 Schema Design Patterns

MongoDB schema design patterns are reusable solutions to common data modeling problems.

---

#### Pattern 1: Attribute Pattern

**Problem:** Documents have many similar fields that are sparsely populated.

**Solution:** Store attributes in an array of key-value pairs.

```javascript
// ❌ Bad: Many sparse fields
{
  _id: ObjectId("product-1"),
  name: "Laptop",
  cpu: "Intel i7",
  ram: "16GB",
  storage: "512GB SSD",
  screenSize: "15.6 inch"
  // Shirt product would have: size, color, material (different attributes!)
}

// ✅ Good: Attribute pattern
{
  _id: ObjectId("product-1"),
  name: "Laptop",
  category: "Electronics",
  attributes: [
    { key: "cpu", value: "Intel i7" },
    { key: "ram", value: "16GB" },
    { key: "storage", value: "512GB SSD" },
    { key: "screenSize", value: "15.6 inch" }
  ]
}

{
  _id: ObjectId("product-2"),
  name: "T-Shirt",
  category: "Clothing",
  attributes: [
    { key: "size", value: "L" },
    { key: "color", value: "Blue" },
    { key: "material", value: "Cotton" }
  ]
}

// Query by attribute
db.products.find({
  "attributes.key": "cpu",
  "attributes.value": "Intel i7"
});

// Create index on attributes
db.products.createIndex({ "attributes.key": 1, "attributes.value": 1 });
```

**Use Cases:**
- Products with varying specifications
- User profiles with custom fields
- IoT sensor data with different metrics

---

#### Pattern 2: Bucket Pattern

**Problem:** Time-series data with high write volume.

**Solution:** Group multiple data points into buckets.

```javascript
// ❌ Bad: One document per reading
{
  _id: ObjectId("..."),
  sensorId: "sensor-1",
  temperature: 72.5,
  timestamp: ISODate("2024-01-15T10:00:00Z")
}
{
  _id: ObjectId("..."),
  sensorId: "sensor-1",
  temperature: 72.7,
  timestamp: ISODate("2024-01-15T10:01:00Z")
}
// Thousands of documents per hour!

// ✅ Good: Bucket pattern (hourly buckets)
{
  _id: ObjectId("..."),
  sensorId: "sensor-1",
  date: ISODate("2024-01-15T10:00:00Z"),  // Bucket start
  readings: [
    { temperature: 72.5, timestamp: ISODate("2024-01-15T10:00:00Z") },
    { temperature: 72.7, timestamp: ISODate("2024-01-15T10:01:00Z") },
    { temperature: 72.9, timestamp: ISODate("2024-01-15T10:02:00Z") },
    // ... up to 60 readings (one per minute)
  ],
  readingCount: 3,
  avgTemperature: 72.7,
  minTemperature: 72.5,
  maxTemperature: 72.9
}

// Update: Add new reading to bucket
db.sensorData.updateOne(
  {
    sensorId: "sensor-1",
    date: ISODate("2024-01-15T10:00:00Z"),
    readingCount: { $lt: 60 }  // Don't exceed bucket size
  },
  {
    $push: { 
      readings: { 
        temperature: 73.0, 
        timestamp: new Date() 
      } 
    },
    $inc: { readingCount: 1 },
    $max: { maxTemperature: 73.0 },
    $min: { minTemperature: 72.5 }
  },
  { upsert: true }
);
```

**Benefits:**
- ✅ Reduces document count (60x fewer documents)
- ✅ Better query performance
- ✅ Pre-computed statistics

**Use Cases:**
- IoT sensor data
- Application metrics
- Financial tick data

---

## 3.1.4 Schema Design Patterns (Continued)

#### Pattern 3: Computed Pattern

**Problem:** Expensive computations performed repeatedly.

**Solution:** Pre-compute and store results.

```javascript
// ❌ Bad: Compute on every query
db.orders.aggregate([
  {
    $group: {
      _id: "$userId",
      totalOrders: { $sum: 1 },
      totalSpent: { $sum: "$total" },
      avgOrderValue: { $avg: "$total" }
    }
  }
]);
// Slow for large collections!

// ✅ Good: Pre-compute and store
{
  _id: ObjectId("user-1"),
  name: "Alice Johnson",
  email: "alice@example.com",
  
  // Computed statistics
  stats: {
    totalOrders: 25,
    totalSpent: 5432.50,
    avgOrderValue: 217.30,
    lastOrderDate: ISODate("2024-01-15T10:00:00Z"),
    favoriteCategories: ["Electronics", "Books"]
  },
  
  updatedAt: ISODate("2024-01-15T10:00:00Z")
}

// Update computed stats when order placed
db.users.updateOne(
  { _id: ObjectId("user-1") },
  {
    $inc: {
      "stats.totalOrders": 1,
      "stats.totalSpent": 99.99
    },
    $currentDate: { "stats.lastOrderDate": true }
  }
);

// Periodically recalculate for accuracy
```

**Use Cases:**
- User statistics
- Product ratings and review counts
- Dashboard metrics

---

#### Pattern 4: Subset Pattern

**Problem:** Documents contain large arrays that are rarely fully accessed.

**Solution:** Store frequently accessed subset in main document, full data elsewhere.

```javascript
// ❌ Bad: All reviews in product document
{
  _id: ObjectId("product-1"),
  name: "Laptop",
  price: 999.99,
  reviews: [
    // Thousands of reviews...
    { userId: ObjectId("..."), rating: 5, text: "Great!" },
    // ... 10,000 more reviews
  ]
}
// 16MB document limit!

// ✅ Good: Subset pattern
// Products collection: Most recent reviews
{
  _id: ObjectId("product-1"),
  name: "Laptop",
  price: 999.99,
  
  // Only recent reviews (last 10)
  recentReviews: [
    { userId: ObjectId("..."), rating: 5, text: "Great!", date: ISODate("...") },
    // ... 9 more recent reviews
  ],
  
  // Summary statistics
  reviewStats: {
    count: 10523,
    avgRating: 4.6,
    ratingDistribution: { 5: 7200, 4: 2100, 3: 800, 2: 300, 1: 123 }
  }
}

// Reviews collection: All reviews
{
  _id: ObjectId("review-1"),
  productId: ObjectId("product-1"),
  userId: ObjectId("user-1"),
  rating: 5,
  text: "Great product!",
  helpful: 45,
  date: ISODate("2024-01-15T10:00:00Z")
}

// Query: Get product with recent reviews (fast)
db.products.findOne({ _id: ObjectId("product-1") });

// Query: Get all reviews (when needed)
db.reviews.find({ productId: ObjectId("product-1") });
```

**Use Cases:**
- Product reviews
- Social media feeds
- Comment threads

---

#### Pattern 5: Extended Reference Pattern

**Problem:** Frequently JOIN-ing documents from different collections.

**Solution:** Duplicate frequently accessed fields.

```javascript
// ❌ Bad: Always need lookup for user info
// Orders collection
{
  _id: ObjectId("order-1"),
  userId: ObjectId("user-1"),  // Only reference
  total: 999.99,
  status: "shipped"
}

// Must lookup user for every order display
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

// ✅ Good: Duplicate frequently used fields
{
  _id: ObjectId("order-1"),
  userId: ObjectId("user-1"),  // Reference for full data
  
  // Duplicate frequently accessed fields
  userName: "Alice Johnson",
  userEmail: "alice@example.com",
  
  total: 999.99,
  status: "shipped"
}

// No lookup needed for order list!
db.orders.find({ status: "shipped" });
```

**Trade-off:**
- ✅ Faster reads (no lookups)
- ❌ Data duplication
- ❌ Must update duplicated data when source changes

**Use Cases:**
- Order history with user info
- Blog posts with author name
- Comments with user avatars

---

### Practice Questions: Schema Design Patterns

#### Multiple Choice

**Q1: When should you use the Bucket Pattern?**

A) For user profiles with custom fields
B) For time-series data with high write volume
C) For documents with large arrays
D) For expensive computations

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - For time-series data with high write volume**

Explanation: The Bucket Pattern groups multiple data points (like sensor readings) into single documents, reducing document count and improving performance for time-series data.

</details>

---

**Q2: What problem does the Subset Pattern solve?**

A) Sparse fields across documents
B) Documents exceeding 16MB limit due to large arrays
C) Expensive repeated computations
D) Slow lookups across collections

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - Documents exceeding 16MB limit due to large arrays**

Explanation: The Subset Pattern stores frequently accessed data (like recent reviews) in the main document and moves the full dataset to a separate collection to avoid document size limits.

</details>

---

## 3.1.5 Indexing in MongoDB

**Definition:** Indexes are data structures that improve query performance by allowing MongoDB to quickly locate documents without scanning every document.

**Analogy:** Like a book index - instead of reading every page to find "MongoDB," you look at the index and jump to page 342.

---

#### Why Indexes Matter

```javascript
// ❌ Without index: Collection scan (slow)
// MongoDB must check every document
db.users.find({ email: "alice@example.com" });
// Examines: 1,000,000 documents
// Time: 5 seconds

// ✅ With index: Index scan (fast)
db.users.createIndex({ email: 1 });
db.users.find({ email: "alice@example.com" });
// Examines: 1 document
// Time: 5 milliseconds
```

---

#### Index Types

**1. Single Field Index**

```javascript
// Create ascending index on email
db.users.createIndex({ email: 1 });

// Create descending index on age
db.users.createIndex({ age: -1 });

// Query uses index automatically
db.users.find({ email: "alice@example.com" });
```

---

**2. Compound Index (Multiple Fields)**

```javascript
// Create compound index
db.products.createIndex({ category: 1, price: -1 });

// ✅ Uses index (both fields)
db.products.find({ category: "Electronics", price: { $lt: 1000 } });

// ✅ Uses index (category only - prefix)
db.products.find({ category: "Electronics" });

// ❌ Doesn't use index efficiently (price only - not prefix)
db.products.find({ price: { $lt: 1000 } });

// Index order matters!
// Can use: category, (category + price)
// Cannot use efficiently: price alone
```

**Rule:** Compound index can be used for queries on:
- All fields (left to right)
- Prefix fields (starting from left)

---

**3. Unique Index**

```javascript
// Ensure email is unique
db.users.createIndex({ email: 1 }, { unique: true });

// ❌ Duplicate insert fails
db.users.insertOne({ name: "Bob", email: "alice@example.com" });
// Error: E11000 duplicate key error
```

---

**4. Sparse Index**

```javascript
// Index only documents that have the field
db.users.createIndex({ phoneNumber: 1 }, { sparse: true });

// Only indexes users with phoneNumber field
// Users without phoneNumber field are not in index
```

---

**5. TTL Index (Time-To-Live)**

```javascript
// Auto-delete documents after 24 hours
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 86400 }  // 24 hours
);

// MongoDB automatically removes expired documents
```

---

**6. Text Index (Full-Text Search)**

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

// Text search with score
db.articles.find(
  { $text: { $search: "mongodb aggregation" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } });
```

---

**7. Multikey Index (Arrays)**

```javascript
// Index array field
db.products.createIndex({ tags: 1 });

// Queries on array elements use index
db.products.find({ tags: "electronics" });
db.products.find({ tags: { $in: ["electronics", "gadgets"] } });
```

---

#### Index Management

**List Indexes:**

```javascript
// List all indexes on collection
db.users.getIndexes();

// Result:
[
  { v: 2, key: { _id: 1 }, name: "_id_" },  // Default index
  { v: 2, key: { email: 1 }, name: "email_1" },
  { v: 2, key: { city: 1, age: -1 }, name: "city_1_age_-1" }
]
```

**Drop Index:**

```javascript
// Drop index by name
db.users.dropIndex("email_1");

// Drop index by keys
db.users.dropIndex({ email: 1 });

// Drop all indexes (except _id)
db.users.dropIndexes();
```

**Index Statistics:**

```javascript
// Get index usage stats
db.users.aggregate([{ $indexStats: {} }]);
```

---

#### Index Best Practices

**1. ESR Rule (Equality, Sort, Range)**

```javascript
// Query with equality, sort, and range
db.users.find({
  status: "active",        // Equality
  age: { $gte: 25 }       // Range
}).sort({ lastName: 1 });  // Sort

// ✅ Optimal index: E-S-R order
db.users.createIndex({
  status: 1,      // Equality first
  lastName: 1,    // Sort second
  age: 1          // Range last
});
```

**2. Index Selectivity**

```javascript
// ✅ Good: Highly selective (email is unique)
db.users.createIndex({ email: 1 });

// ❌ Bad: Low selectivity (only 2 values)
db.users.createIndex({ gender: 1 });
// Better to scan than use index!

// ✅ Good: Compound index with selective fields
db.users.createIndex({ status: 1, email: 1 });
```

---

**3. Cover Queries with Indexes**

```javascript
// Create compound index
db.users.createIndex({ status: 1, name: 1, email: 1 });

// ✅ Covered query: All fields in index
db.users.find(
  { status: "active" },
  { _id: 0, name: 1, email: 1 }  // Only indexed fields
);
// MongoDB doesn't need to look at documents!
```

---

**4. Index Intersection**

```javascript
// Multiple single-field indexes
db.products.createIndex({ category: 1 });
db.products.createIndex({ price: 1 });

// MongoDB can use BOTH indexes
db.products.find({
  category: "Electronics",
  price: { $lt: 500 }
});

// But compound index is still better!
db.products.createIndex({ category: 1, price: 1 });
```

---

#### Analyzing Query Performance

**Use explain() to see execution plan:**

```javascript
// Explain query execution
db.users.find({ email: "alice@example.com" }).explain("executionStats");

// Key metrics:
{
  executionStats: {
    executionSuccess: true,
    nReturned: 1,              // Documents returned
    executionTimeMillis: 2,     // Time taken
    totalKeysExamined: 1,       // Index keys scanned
    totalDocsExamined: 1,       // Documents examined
    executionStages: {
      stage: "FETCH",
      inputStage: {
        stage: "IXSCAN",        // Index scan (good!)
        keyPattern: { email: 1 }
      }
    }
  }
}

// ❌ Bad execution plan (COLLSCAN)
{
  executionStages: {
    stage: "COLLSCAN",          // Collection scan (slow!)
    nReturned: 1,
    totalDocsExamined: 1000000  // Scanned all documents!
  }
}
```

---

**Execution Stages:**

- `IXSCAN` - Index scan (good ✅)
- `FETCH` - Fetch documents using index
- `COLLSCAN` - Collection scan (slow ❌)
- `SORT` - In-memory sort (slow if no index ❌)

---

### Practice Questions: Indexing

#### Fill in the Blanks

1. An ________ is a data structure that improves query performance.
2. The ________ rule states the optimal order for compound indexes: Equality, Sort, Range.
3. A ________ index automatically deletes documents after a specified time.
4. Use ________ to analyze query execution and see if indexes are used.
5. ________ ensures a field has unique values across documents.

<details>
<summary><strong>View Answers</strong></summary>

1. index
2. ESR
3. TTL
4. explain()
5. Unique index

</details>

---

#### True/False

1. MongoDB automatically creates an index on the _id field.
2. A compound index on (A, B) can efficiently query B alone.
3. Text indexes support full-text search on string fields.
4. More indexes always mean better performance.
5. Covered queries don't need to examine documents.

<details>
<summary><strong>View Answers</strong></summary>

1. TRUE
2. FALSE (must query A or A+B, not B alone)
3. TRUE
4. FALSE (indexes slow down writes and use memory)
5. TRUE (all data comes from index)

</details>

---

#### Coding Challenges

**Challenge 1: Optimal Index**

Given this query, create the optimal compound index:

```javascript
db.orders.find({
  status: "completed",
  userId: ObjectId("..."),
  total: { $gte: 100 }
}).sort({ createdAt: -1 });
```

<details>
<summary><strong>View Solution</strong></summary>

```javascript
// ESR Rule: Equality, Sort, Range
db.orders.createIndex({
  status: 1,      // Equality
  userId: 1,      // Equality
  createdAt: -1,  // Sort
  total: 1        // Range
});
```

</details>

---

**Challenge 2: Analyze Performance**

You have a slow query. Use explain() to diagnose and fix it.

```javascript
// Slow query
db.products.find({
  category: "Electronics",
  price: { $lt: 500 },
  stock: { $gt: 0 }
});
```

<details>
<summary><strong>View Solution</strong></summary>

```javascript
// Step 1: Analyze
db.products.find({
  category: "Electronics",
  price: { $lt: 500 },
  stock: { $gt: 0 }
}).explain("executionStats");

// If shows COLLSCAN, create index:

// Step 2: Create optimal index (ESR)
db.products.createIndex({
  category: 1,   // Equality
  stock: 1,      // Equality
  price: 1       // Range
});

// Step 3: Verify improvement
db.products.find({
  category: "Electronics",
  price: { $lt: 500 },
  stock: { $gt: 0 }
}).explain("executionStats");
// Should now show IXSCAN
```

</details>

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

**Navigation:** [← SQL Essentials: Relationships, JOINs, and Aggregations](02_sql_essentials.md) | [README](../README.md) | [Next: NoSQL Deep Dive: Redis →](03_01_redis.md)