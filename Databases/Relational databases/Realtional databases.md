
At its core, PostgreSQL stores data in tables (also called relations). Think of a table like a spreadsheet with rows and columns. Each column has a specific data type (like text, numbers, or dates), and each row represents one complete record.

Imagine we are creating a table

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Each user gets a unique id (that's what PRIMARY KEY means), and we ensure no two users can have the same username or email (that's what UNIQUE does).

But users aren't much fun by themselves. They need to be able to post content. Here's where the "relational" part of relational databases comes in. We can create a posts table that's connected to our users:

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

See REFERENCES users(id)? That's called a foreign key - it creates a relationship between posts and users. Every post must belong to a valid user, and PostgreSQL will enforce this for us. This is one of the key strengths of relational databases: they help maintain data integrity by enforcing these relationships.

There are three types of relations :

- one to one
- one to many
- many to many

Now, what if we want users to be able to like posts? This introduces a many-to-many relationship - one user can like many posts, and one post can be liked by many users. We handle this with what's called a join table:

```sql
CREATE TABLE likes (
    user_id INTEGER REFERENCES users(id),
    post_id INTEGER REFERENCES posts(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, post_id)
);
```

This structure, where we break data into separate tables and connect them through relationships, is called "normalization." It helps us:

- avoid duplicate data
- maintain data integrity
- make data model flexible

While normalization is generally good, sometimes we intentionally denormalize data for performance. For example, we might store a post's like count directly in the posts table even though we could calculate it from the likes table. This trade-off between data consistency and query performance is exactly the kind of thing you should discuss in your interview!

### ACID properties

One of PostgreSQL's greatest strengths is its strict adherence to ACID (Atomicity, Consistency, Isolation, and Durability) properties. If you've used databases like MongoDB or Cassandra, you're familiar with eventual consistency or relaxed transaction guarantees which are common trade-offs in NoSQL databases. PostgreSQL takes a different approach – it ensures that your data always follows all defined rules and constraints (like foreign keys, unique constraints, and custom checks), and that all transactions complete fully or not at all, even if it means sacrificing some performance.

#### Atomicity

```sql
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE account_id = 'savings';
  UPDATE accounts SET balance = balance + 100 WHERE account_id = 'checking';
COMMIT;
```

Atomicity guarantees that either both operations succeed or neither does. If the system crashes after deducting from savings but before adding to checking, PostgreSQL will roll back the entire transaction. Your money never disappears into thin air.

#### Consistency

Consistency ensures that transactions can only bring the database from one valid state to another.

```sql
CREATE TABLE accounts (
    account_id TEXT PRIMARY KEY,
    balance DECIMAL CHECK (balance >= 0),
    owner_id INTEGER REFERENCES users(id)
);
```

If a transaction would make your balance negative, PostgreSQL will reject the entire transaction.

Confusingly, consistency in ACID has a slightly different meaning than consistency in CAP Theorem. In ACID, consistency means that the database always follows all defined rules and constraints. In the CAP Theorem, consistency means that the database always returns the correct result, even if it means sacrificing availability or partition tolerance.

#### Isolation

_Isolation_ means that each transaction runs **as if it were the only transaction** executing on the system — even though many might actually run at the same time.

It prevents **interference** between concurrent transactions, so one transaction’s intermediate (uncommitted) data isn’t visible to another.

Without isolation, two transactions running together could **read or modify each other’s partial results**, leading to problems like:

- Incorrect totals
- Missing updates
- Inconsistent reads

In reality there are isolation levels which define when multiple transations run concurrently how much others are allowed to see each other transation.

When multiple transactions run **concurrently**, the database needs to decide how much they are allowed to “see” or “interfere” with each other.

**Higher isolation = safer, but slower.**
**Lower isolation = faster, but may cause inconsistent results.**

#### Read uncommited 

Transactions can read data that has not been commited by others. No locks on reads. Its lowest safety but fastest.
There can be many problems due to this 

1. Dirty read: A dirty read happens when **one transaction reads data that another transaction has written but not yet committed**.  If that other transaction rolls back (undoes) the change, the first transaction has read **invalid or “dirty” data**.

|Transaction A|Transaction B|
|---|---|
|BEGIN TRANSACTION;||
|UPDATE accounts SET balance = 200 WHERE id = 1;||
|_(not yet committed)_||
||BEGIN TRANSACTION;|
||SELECT balance FROM accounts WHERE id = 1; → **200**|
||-- B reads the uncommitted value|
|ROLLBACK;||
|-- balance goes back to 100||
||COMMIT;|
||-- B has used wrong data (200) that never really existed


2. Non repetable reads: A non-repeatable read occurs when **a transaction reads the same record twice and gets different values**, because another transaction **modified or deleted** that record in between the reads.

| Transaction A                                        | Transaction B                                   |
| ---------------------------------------------------- | ----------------------------------------------- |
|                                                      |                                                 |
| BEGIN TRANSACTION;                                   |                                                 |
| SELECT balance FROM accounts WHERE id = 1; → **100** |                                                 |
|                                                      | BEGIN TRANSACTION;                              |
|                                                      | UPDATE accounts SET balance = 200 WHERE id = 1; |
|                                                      | COMMIT;                                         |
| SELECT balance FROM accounts WHERE id = 1; → **200** |                                                 |
| COMMIT;                                              |                                                 |


3. Phantom reads: A phantom read occurs when **new rows appear (or existing rows disappear)** between two reads in the same transaction not because a row changed, but because **another transaction inserted or deleted rows** that match the query condition.

| Transaction A                                                          | Transaction B                         |
| ---------------------------------------------------------------------- | ------------------------------------- |
|                                                                        |                                       |
| BEGIN TRANSACTION;                                                     |                                       |
| SELECT * FROM accounts WHERE balance > 100; → returns **(id=2)**       |                                       |
|                                                                        | BEGIN TRANSACTION;                    |
|                                                                        | INSERT INTO accounts VALUES (3, 200); |
|                                                                        | COMMIT;                               |
| SELECT * FROM accounts WHERE balance > 100; → returns **(id=2, id=3)** |                                       |
| COMMIT;                                                                |                                       |

READ uncommited does not solve any of the issues.

#### Read commited

A transaction can **only read committed data** uncommitted updates from others are invisible. 

So it solves the problem of dirty reads. However while this transaction is running any other transation can update the row so It does not solves no repetable reads. Similary since other transaction can still insert or delete a row while this is going on so phantom read is still a problem. 

#### Repeatable read

Once you read a row, **no other transaction can modify that row** until you finish (depending on DB engine).You’ll always get the same values if you re-read the same rows.However, **new rows** that match your query condition can appear later  this is the **phantom read** problem.

#### Serializable

Transactions are executed **as if they were completely sequential** — one after another.  
The database ensures no other transaction affects your results, even indirectly.

Prevents: Dirty reads, non-repeatable reads, phantom reads. It is like doing each transaction one by one and this is achieved by **locks** or **MVCC snapshots**.

#### Durability

Once PostgreSQL says a transaction is committed, that data is guaranteed to have been written to disk and sync'd, protecting against crashes or power failures. This is achieved through Write-Ahead Logging (WAL):

1. Changes are first written to a log
2. The log is flushed to disk
3. Only then is the transaction considered committed