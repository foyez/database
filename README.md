# Complete Database Guide
**From Fundamentals to Production - Interview Ready**

> A comprehensive guide covering SQL, NoSQL, performance optimization, scaling strategies, and production best practices.

---

## 📚 Table of Contents

### [1. Fundamentals](./sections/01_fundamentals.md)
**Core database concepts, terminology, and decision-making**

- 1.1 What is a Database?
- 1.2 Database Types Overview
- 1.3 Database Terminology (SQL & NoSQL)
- 1.4 SQL Databases (Relational)
  - 1.4.1 Structure & Characteristics
  - 1.4.2 Popular SQL Databases
  - 1.4.3 When to Use SQL
- 1.5 NoSQL Databases (Non-Relational)
  - 1.5.1 Document-Oriented (MongoDB)
  - 1.5.2 Key-Value (Redis)
  - 1.5.3 Column-Family (Cassandra)
  - 1.5.4 Graph (Neo4j)
  - 1.5.5 Time-Series (InfluxDB)
- 1.6 SQL vs NoSQL Decision Guide
- 1.7 ACID Properties
  - 1.7.1 Atomicity
  - 1.7.2 Consistency
  - 1.7.3 Isolation
  - 1.7.4 Durability

---

### [2. SQL Essentials](./sections/02_sql_essentials.md)
**Core SQL concepts: relationships, joins, queries, and advanced topics**

- 2.1 Database Relationships
  - 2.1.1 One-to-One (1:1)
  - 2.1.2 One-to-Many (1:N)
  - 2.1.3 Many-to-Many (M:N)
  - 2.1.4 Foreign Keys
- 2.2 SQL JOINs
  - 2.2.1 INNER JOIN
  - 2.2.2 LEFT JOIN (LEFT OUTER JOIN)
  - 2.2.3 RIGHT JOIN (RIGHT OUTER JOIN)
  - 2.2.4 FULL OUTER JOIN
  - 2.2.5 CROSS JOIN
  - 2.2.6 Self JOIN
- 2.3 Aggregate Functions & GROUP BY
  - 2.3.1 COUNT, SUM, AVG, MIN, MAX
  - 2.3.2 GROUP BY Basics
  - 2.3.3 HAVING vs WHERE
  - 2.3.4 Multiple Aggregations
- 2.4 Advanced SQL Queries
  - 2.4.1 Subqueries (Nested Queries)
  - 2.4.2 Common Table Expressions (CTEs)
  - 2.4.3 Window Functions
  - 2.4.4 UNION, INTERSECT, EXCEPT
- 2.5 Transactions & Isolation Levels
  - 2.5.1 Transaction Basics
  - 2.5.2 Isolation Levels
  - 2.5.3 Locks and Deadlocks

---

### [3. NoSQL Deep Dive](./sections/03_nosql_deep_dive.md)
**MongoDB and Redis operations, patterns, and best practices**

- [3.1 MongoDB](./sections/03_01_mongodb.md)
  - 3.1.1 CRUD Operations
  - 3.1.2 Embedded vs Referenced Documents
  - 3.1.3 Aggregation Pipeline
  - 3.1.4 Schema Design Patterns
  - 3.1.5 Indexing in MongoDB
- [3.2 Redis](./sections/03_02_redis.md)
  - 3.2.1 Data Structures (String, Hash, List, Set, Sorted Set)
  - 3.2.2 Caching Patterns
  - 3.2.3 Pub/Sub Messaging
  - 3.2.4 Expiration & TTL
  - 3.2.5 Use Cases & Best Practices

---

### [4. Performance & Optimization](./sections/04_performance.md)
**Indexing strategies, query optimization, and caching**

- 4.1 Indexing Strategies
  - 4.1.1 SQL Indexes (B-Tree, Hash, GIN, Partial, Composite)
  - 4.1.2 NoSQL Indexes (MongoDB)
  - 4.1.3 Index Selection & Maintenance
- 4.2 Query Optimization
  - 4.2.1 EXPLAIN & Query Plans
  - 4.2.2 SQL Query Optimization
  - 4.2.3 MongoDB Query Optimization
- 4.3 Caching Strategies
  - 4.3.1 Application-Level Caching (Redis)
  - 4.3.2 Cache Invalidation
  - 4.3.3 Caching Patterns
  - 4.3.4 Database-Level Caching
  - 4.3.5 Materialized Views

---

### [5. Scaling & High Availability](./sections/05_scaling.md)
**Replication, sharding, and distributed systems**

- 5.1 Vertical vs Horizontal Scaling
- 5.2 Replication (Read Scaling)
  - 5.2.1 SQL Replication (PostgreSQL/MySQL)
  - 5.2.2 MongoDB Replica Sets
  - 5.2.3 Replication Lag & Consistency
- 5.3 Sharding (Write Scaling)
  - 5.3.1 Sharding Strategies
  - 5.3.2 MongoDB Sharding
  - 5.3.3 Challenges & Solutions
- 5.4 Connection Pooling
- 5.5 Load Balancing
- 5.6 CAP Theorem

---

### [6. Production Best Practices](./sections/06_production.md)
**Security, monitoring, backup, and testing**

- 6.1 Database Security
  - 6.1.1 Principle of Least Privilege
  - 6.1.2 SQL Injection Prevention
  - 6.1.3 Password Security
  - 6.1.4 Encryption (At Rest & In Transit)
  - 6.1.5 Environment Variables & Secrets
- 6.2 Monitoring & Observability
  - 6.2.1 Key Metrics
  - 6.2.2 Monitoring Tools
  - 6.2.3 Alert Configuration
- 6.3 Backup & Recovery
  - 6.3.1 Backup Strategies
  - 6.3.2 Recovery Scenarios
  - 6.3.3 Best Practices
- 6.4 Database Testing
  - 6.4.1 Unit Testing
  - 6.4.2 Integration Testing
  - 6.4.3 Load Testing
- 6.5 Database Migrations

---

### 7. Real-World Projects
**Complete implementations with SQL and NoSQL**

- 7.1 [E-Commerce Platform (PostgreSQL)](sections/07_projects/01-e-commerce-platform.md)
  - 7.1.1 Requirements Analysis
  - 7.1.2 Schema Design
  - 7.1.3 Common Queries
  - 7.1.4 Transaction Examples
- [7.2 Blog Platform (MongoDB)](sections/07_projects/02-blog-platform.md)
  - 7.2.1 Document Model Design
  - 7.2.2 CRUD Operations
  - 7.2.3 Aggregation Queries
- [7.3 Real-Time Gaming (Redis)](sections/07_projects/03-real-time-gaming.md)
  - 7.3.1 Leaderboard Implementation
  - 7.3.2 Session Management
  - 7.3.3 Rate Limiting
  - 7.3.4 Pub/Sub Chat

---

### [8. Interview Preparation](./sections/08_interview_prep.md)
**Common questions, memory tricks, and quick reference**

- 8.1 Memory Tips & Mnemonics
- 8.2 Top 20 Interview Questions & Answers
- 8.3 System Design Questions
- 8.4 Quick Reference Cheatsheet

---

## 🎯 Learning Path

### Beginner (Weeks 1-2)
1. Read **Fundamentals** (Chapter 1)
2. Study **SQL Essentials** basics (Chapter 2.1-2.3)
3. Complete **E-Commerce Project** (Chapter 7.1)
4. Practice questions in each section

### Intermediate (Weeks 3-4)
1. Deep dive into **NoSQL** (Chapter 3)
2. Study **Performance & Optimization** (Chapter 4)
3. Complete **Blog Platform** and **Gaming** projects (Chapter 7.2-7.3)
4. Review **Interview Questions** (Chapter 8.2)

### Advanced (Weeks 5-6)
1. Master **Scaling & High Availability** (Chapter 5)
2. Learn **Production Best Practices** (Chapter 6)
3. Practice **System Design** questions (Chapter 8.3)
4. Review all memory tricks and quick references

---

## 📊 Practice Questions

Each chapter includes:
- ✅ **Fill-in-the-Blanks** - Test vocabulary and concepts
- ✅ **True/False** - Verify understanding
- ✅ **Multiple Choice** - Decision-making scenarios
- ✅ **Coding Challenges** - Real SQL/NoSQL queries
- ✅ **Answers** - Detailed explanations provided

---

## 🚀 Quick Reference

### SQL vs NoSQL Decision
```
Use SQL when:
✅ ACID transactions critical (banking, payments)
✅ Complex relationships (orders ↔ users ↔ products)
✅ Fixed schema
✅ Complex queries with JOINs

Use NoSQL when:
✅ Flexible schema (product catalogs)
✅ Horizontal scaling needed (millions of users)
✅ Simple queries
✅ High write throughput
```

### ACID Memory Trick: "A Car Is Durable"
- **A**tomicity - All or nothing
- **C**onsistency - Rules followed
- **I**solation - No interference
- **D**urability - Survives crashes

### JOIN Types Memory: "ILRF CrossSelf"
- **I**NNER - Only matches
- **L**EFT - All left + matches
- **R**IGHT - All right + matches
- **F**ULL - Everything
- **Cross** - All combinations
- **Self** - Table with itself

---

## 💡 Tips for Interview Success

1. **Understand, Don't Memorize**
   - Know WHY, not just WHAT
   - Explain with real-world examples

2. **Practice Writing Queries**
   - Set up local PostgreSQL and MongoDB
   - Run all examples in this guide

3. **Draw Diagrams**
   - ER diagrams for relationships
   - System architecture for scaling

4. **Use Memory Tricks**
   - ACID, CAP, JOINs mnemonics
   - Create your own if helpful

5. **Stay Current**
   - Database versions evolve
   - New features added regularly

---

## 📖 How to Use This Guide

### For Learning
1. Read sections in order
2. Try all code examples
3. Complete practice questions
4. Review answers carefully

### For Interview Prep
1. Read **Memory Tips** (Chapter 8.1)
2. Practice **Top Questions** (Chapter 8.2)
3. Review **Quick Reference** sections
4. Do mock interviews

### For Reference
1. Use Table of Contents to find topics
2. Each section is self-contained
3. Copy-paste code examples directly
4. Adapt to your use case

---

## 🤝 Contributing

Found an error? Have a suggestion? Topics to add:
- Additional NoSQL databases (Cassandra, Neo4j deep dives)
- More real-world projects
- Video tutorials
- Practice problem sets

---

## 📝 License & Usage

Feel free to:
- ✅ Use for learning and reference
- ✅ Share with others
- ✅ Adapt examples to your projects
- ✅ Use in interviews

---

## 🎓 What Makes This Guide Special

1. **Real-World Examples** - No foo/bar, actual e-commerce scenarios
2. **Interview-Ready** - Questions and answers included
3. **Production-Tested** - Patterns used in real systems
4. **Complete Coverage** - From basics to advanced
5. **Practice Questions** - Verify understanding throughout
6. **Memory Tricks** - Easy to remember frameworks
7. **Visual Learning** - Diagrams and tables
8. **Code Examples** - Copy-paste ready

---

**Let's master databases together!** 🚀

Start with [Chapter 1: Fundamentals](./sections/01_fundamentals.md)