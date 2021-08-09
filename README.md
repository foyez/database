# Complete Database

## Database (DB)

A **database** is a collection of information that is organized so that it can be easily accessed, managed and updated. It is a place to save your application's state so that it can be retrieved later. This allows you to make your servers stateless since your database will be storing all the information.

## Query

A query is a command we send to a database to get information out of the database or to add/update/delete something in the database. It can also aggregate information from a database into some sort of overview.

## Schema

If a database is a table in Microsoft Excel, then a schema is the columns. It's a rigid structure used to model data.

If I had a JSON object of user that looked like `{ "name": "Foyez", "city": "Cumilla", "village": "Surikara" }` then the schema would be name, city, and village. It's the shape of the data.

## Types of databases

1. Relational databases (RDBMS or SQL)
2. Document-based databases (NoSQL)
3. Graph database
4. Key-value store

## ACID (Atomicity, Consistency, Isolation & Durability)

One should think about these four factors when thinking about writing queries. ACID is safe but slow.

**1. Atomicity:** Does this query happen all at once? Or is it broken up into multiple actions? Sometimes this is a very important question. If you're doing financial transactions this can be paramount. Imagine if you have two queries: one to subtract money from one person's account and one to add that amount to another person's account for a bank transfer. What if the power went out in between the two queries? Money would be lost. This is an unacceptable trade-off in this case. You would need this to be atomic. That is, this transaction cannot be divided.

**2. Consistency:** If I have five servers running with one running as the primary server (sometimes called master but we prefer leader or primary) and the primary server crashes, what happens? Again, using money, what if a query had been written to the primary but not yet to the secondaries? This could be a disaster and people could lose money. In this case, we need our servers to be consistent.

**3. Isolation:** Isolation says that we can have a multi-threaded database but it needs to work the same if a query was ran in parallel or if it was ran sequentially. Otherwise it fails the isolation test.

**4. Durability:** Durability says that if a server crashes that we can restore it to the state that it was in previously. Some databases only run in memory so if a server crashes, that data is gone. However this is slow; waiting for a DB to write to disk before finishing a query adds a lot of time.

## Transactions

A transaction is like an envelope of transactions that get to happen all at the same time with a guarantee that they will all get ran at once or none at all. If they are run, it guarantees that no other query will happen between them.
