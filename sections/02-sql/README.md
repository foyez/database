## Table of Contents

## **1. Database Fundamentals**
   * **1.1 Introduction to Databases**
      * 1.1.1 What is a Database?
      * 1.1.2 DBMS vs RDBMS
      * 1.1.3 SQL vs NoSQL
   * **1.2 Database Design Concepts**
      * 1.2.1 Entity–Relationship (ER) Diagrams
      * 1.2.2 Entities, Attributes, and Relationships
      * 1.2.3 Cardinality and Participation Constraints
   * **1.3 Database Relationships**
      * 1.3.1 One-to-One (1:1)
      * 1.3.2 One-to-Many (1:N)
      * 1.3.3 Many-to-Many (M:N)
      * 1.3.4 Junction/Bridge Tables

## **2. Database Schema & Keys**
   * **2.1 Primary Keys**
      * 2.1.1 Definition and Purpose
      * 2.1.2 Natural vs Surrogate Keys
      * 2.1.3 Composite Primary Keys
      * 2.1.4 AUTO_INCREMENT/SERIAL
   * **2.2 Foreign Keys**
      * 2.2.1 Definition and Purpose
      * 2.2.2 Referential Integrity
      * 2.2.3 CASCADE, SET NULL, RESTRICT
      * 2.2.4 Self-Referencing Foreign Keys
   * **2.3 Other Key Types**
      * 2.3.1 Unique Keys
      * 2.3.2 Candidate Keys
      * 2.3.3 Alternate Keys
      * 2.3.4 Composite Keys

## **3. Normalization & Data Integrity**
   * **3.1 Database Normalization**
      * 3.1.1 Why Normalize?
      * 3.1.2 First Normal Form (1NF)
      * 3.1.3 Second Normal Form (2NF)
      * 3.1.4 Third Normal Form (3NF)
      * 3.1.5 Boyce-Codd Normal Form (BCNF)
      * 3.1.6 Fourth & Fifth Normal Forms (4NF, 5NF)
      * 3.1.7 Functional Dependencies
   * **3.2 Denormalization**
      * 3.2.1 When and Why to Denormalize
      * 3.2.2 Trade-offs: Performance vs Redundancy
      * 3.2.3 Common Denormalization Techniques
   * **3.3 Data Integrity Constraints**
      * 3.3.1 NOT NULL
      * 3.3.2 UNIQUE
      * 3.3.3 CHECK Constraints
      * 3.3.4 DEFAULT Values

## **4. Basic SQL Operations**
   * **4.1 Data Definition Language (DDL)**
      * 4.1.1 CREATE (Database, Table, Schema)
      * 4.1.2 ALTER (Modify Structure)
      * 4.1.3 DROP & TRUNCATE
      * 4.1.4 RENAME
   * **4.2 Data Manipulation Language (DML)**
      * 4.2.1 INSERT
      * 4.2.2 UPDATE
      * 4.2.3 DELETE
      * 4.2.4 MERGE/UPSERT
   * **4.3 Data Query Language (DQL)**
      * 4.3.1 SELECT Basics
      * 4.3.2 WHERE Clause
      * 4.3.3 ORDER BY
      * 4.3.4 LIMIT/OFFSET (Pagination)
      * 4.3.5 DISTINCT
   * **4.4 Data Control Language (DCL)**
      * 4.4.1 GRANT
      * 4.4.2 REVOKE
      * 4.4.3 User Permissions

## **5. SQL JOINs**
   * **5.1 INNER JOIN**
   * **5.2 LEFT JOIN (LEFT OUTER JOIN)**
   * **5.3 RIGHT JOIN (RIGHT OUTER JOIN)**
   * **5.4 FULL OUTER JOIN**
   * **5.5 CROSS JOIN**
   * **5.6 Self JOIN**
   * **5.7 Multiple JOINs**
   * **5.8 JOIN Performance & Optimization**

## **6. Aggregate Functions & Grouping**
   * **6.1 Aggregate Functions**
      * 6.1.1 COUNT, SUM, AVG, MIN, MAX
      * 6.1.2 String Aggregation (GROUP_CONCAT, STRING_AGG)
   * **6.2 GROUP BY**
      * 6.2.1 GROUP BY Basics
      * 6.2.2 Multiple Column Grouping
      * 6.2.3 ROLLUP, CUBE, GROUPING SETS
   * **6.3 HAVING vs WHERE**
   * **6.4 Multiple Aggregations**

## **7. Advanced SQL Queries**
   * **7.1 Subqueries**
      * 7.1.1 Scalar Subqueries
      * 7.1.2 Row Subqueries
      * 7.1.3 Table Subqueries
      * 7.1.4 Correlated Subqueries
      * 7.1.5 EXISTS and NOT EXISTS
   * **7.2 Common Table Expressions (CTEs)**
      * 7.2.1 Basic CTEs
      * 7.2.2 Recursive CTEs
      * 7.2.3 Multiple CTEs
   * **7.3 Set Operations**
      * 7.3.1 UNION & UNION ALL
      * 7.3.2 INTERSECT
      * 7.3.3 EXCEPT/MINUS
   * **7.4 Window Functions**
      * 7.4.1 ROW_NUMBER, RANK, DENSE_RANK
      * 7.4.2 PARTITION BY
      * 7.4.3 LEAD, LAG
      * 7.4.4 Running Totals (Cumulative SUM)
      * 7.4.5 NTILE, PERCENT_RANK
   * **7.5 CASE Expressions**
      * 7.5.1 Simple CASE
      * 7.5.2 Searched CASE
      * 7.5.3 Conditional Aggregation

## **8. Indexes & Performance**
   * **8.1 Index Fundamentals**
      * 8.1.1 What are Indexes?
      * 8.1.2 B-Tree Indexes
      * 8.1.3 Hash Indexes
      * 8.1.4 Full-Text Indexes
      * 8.1.5 Spatial Indexes
   * **8.2 Creating and Managing Indexes**
      * 8.2.1 CREATE INDEX
      * 8.2.2 Unique Indexes
      * 8.2.3 Composite Indexes
      * 8.2.4 Partial Indexes
      * 8.2.5 Covering Indexes
   * **8.3 Query Optimization**
      * 8.3.1 EXPLAIN/EXPLAIN ANALYZE
      * 8.3.2 Query Execution Plans
      * 8.3.3 Index Selection Strategy
      * 8.3.4 Query Rewriting Techniques
   * **8.4 Performance Best Practices**
      * 8.4.1 Avoiding SELECT *
      * 8.4.2 N+1 Query Problem
      * 8.4.3 Batch Operations
      * 8.4.4 Connection Pooling

## **9. Transactions & Concurrency**
   * **9.1 Transaction Basics**
      * 9.1.1 ACID Properties
      * 9.1.2 BEGIN, COMMIT, ROLLBACK
      * 9.1.3 SAVEPOINT
      * 9.1.4 Autocommit Mode
   * **9.2 Isolation Levels**
      * 9.2.1 READ UNCOMMITTED
      * 9.2.2 READ COMMITTED
      * 9.2.3 REPEATABLE READ
      * 9.2.4 SERIALIZABLE
      * 9.2.5 Isolation Level Problems (Dirty Reads, Phantom Reads, etc.)
   * **9.3 Locks and Concurrency**
      * 9.3.1 Shared vs Exclusive Locks
      * 9.3.2 Row-Level vs Table-Level Locks
      * 9.3.3 Deadlocks
      * 9.3.4 Lock Timeout Configuration
      * 9.3.5 Optimistic vs Pessimistic Locking

## **10. Views, Stored Procedures & Triggers**
   * **10.1 Views**
      * 10.1.1 Creating Views
      * 10.1.2 Updatable Views
      * 10.1.3 Materialized Views
      * 10.1.4 View Performance Considerations
   * **10.2 Stored Procedures**
      * 10.2.1 Creating Procedures
      * 10.2.2 Parameters (IN, OUT, INOUT)
      * 10.2.3 Control Flow (IF, WHILE, LOOP)
      * 10.2.4 Error Handling
   * **10.3 Functions**
      * 10.3.1 User-Defined Functions (UDFs)
      * 10.3.2 Scalar vs Table-Valued Functions
   * **10.4 Triggers**
      * 10.4.1 BEFORE vs AFTER Triggers
      * 10.4.2 INSERT, UPDATE, DELETE Triggers
      * 10.4.3 Trigger Use Cases
      * 10.4.4 Trigger Performance Impact

## **11. Data Types & Functions**
   * **11.1 Data Types**
      * 11.1.1 Numeric Types (INT, DECIMAL, FLOAT)
      * 11.1.2 String Types (CHAR, VARCHAR, TEXT)
      * 11.1.3 Date & Time Types
      * 11.1.4 Boolean
      * 11.1.5 JSON/XML Types
      * 11.1.6 BLOB/Binary Types
   * **11.2 String Functions**
      * 11.2.1 CONCAT, LENGTH, SUBSTRING
      * 11.2.2 UPPER, LOWER, TRIM
      * 11.2.3 LIKE, REGEXP Pattern Matching
   * **11.3 Date & Time Functions**
      * 11.3.1 NOW, CURDATE, CURTIME
      * 11.3.2 DATE_ADD, DATE_SUB, DATEDIFF
      * 11.3.3 DATE_FORMAT, STR_TO_DATE
   * **11.4 Numeric Functions**
      * 11.4.1 ROUND, CEIL, FLOOR
      * 11.4.2 ABS, POWER, SQRT
      * 11.4.3 RAND, MOD
   * **11.5 Conditional & Null Functions**
      * 11.5.1 COALESCE, NULLIF
      * 11.5.2 IFNULL, NVL

## **12. Security & Best Practices**
   * **12.1 SQL Injection Prevention**
      * 12.1.1 Parameterized Queries
      * 12.1.2 Input Validation
      * 12.1.3 Escaping Special Characters
   * **12.2 User Management**
      * 12.2.1 Creating Users
      * 12.2.2 Role-Based Access Control (RBAC)
      * 12.2.3 Principle of Least Privilege
   * **12.3 Backup & Recovery**
      * 12.3.1 Backup Strategies
      * 12.3.2 Point-in-Time Recovery
      * 12.3.3 Replication
   * **12.4 Auditing & Logging**

## **13. Advanced Topics**
   * **13.1 Partitioning**
      * 13.1.1 Range Partitioning
      * 13.1.2 List Partitioning
      * 13.1.3 Hash Partitioning
   * **13.2 Sharding**
   * **13.3 Database Replication**
      * 13.3.1 Master-Slave Replication
      * 13.3.2 Master-Master Replication
   * **13.4 Database Migration**
      * 13.4.1 Schema Versioning
      * 13.4.2 Migration Tools
   * **13.5 Database Design Patterns**
      * 13.5.1 Temporal Tables
      * 13.5.2 Soft Deletes
      * 13.5.3 Audit Tables