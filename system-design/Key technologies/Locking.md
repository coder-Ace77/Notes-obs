
---

In our system we may have shared resources which can only be accessed by one client at a time. An example might be a shared counter (like inventory units) or an interface to a physical device (drawbridge up!). Locking is the process of ensuring that only one client can access a shared resource at a time.

Locks happen at every scale of computer systems: there are locks in your operating system kernel, locks in your applications, locks in the database, and even distributed locks - hence they are a common topic in system design interviews. Locks are important for enforcing the correctness of our system but can be disastrous for performance.

A race condition, if you remember, is a situation where multiple clients are trying to access the same resource at the same time. This can lead to data corruption, lost updates, and other bad things.In most system design interviews, you'll be forced to contend with locks when you consider race conditions.

There's three things to worry about when employing locks:

**Granularity of the lock**:

We want locks to be as fine-grained as possible. This means that we want to lock as little as possible to ensure that we're not blocking other clients from accessing the system.

**Duration of the lock**:

We want locks to be held for as short a time as possible. This means that we want to lock only for the duration of the critical section. For example, if we're updating a user's profile, we want to lock only for the duration of the update and not for the entire request.

**Whether we can bypass the lock**: 

In many cases, we can avoid locking by employing an "optimistic" concurrency control strategy, especially if the work to be done is either read-only or can be retried. In an optimistic strategy we're going to assume that we can do the work without locking and then check to see if we were right

Optimistic locking is a **concurrency control strategy** used in databases and applications to prevent **lost updates** when multiple transactions read and modify the same data at the same time — but **without using database-level locks** during most of the operation.

It works on the **assumption that conflicts are rare**, and it’s better to handle them _after they occur_ rather than block other users _before they happen_.

Let’s take a simple example:

1. **Read Phase:**  
    Transaction A reads a record (say, `user_balance = 100` with `version = 3`).
2. **Modify Locally:**  
    Transaction A makes some change in memory (e.g., sets `user_balance = 150`).
3. **Check Before Write (Validation Phase):**  
    Before updating, Transaction A checks if the record still has the same version (`version = 3`) in the database.
4. **Commit or Retry:**
    - If yes → No one else changed it → update succeeds and `version` increments to `4`.
    - If no → Someone else already updated the record → transaction fails → retry or abort.

Optimistic locking use cases:

1. Writes are rare and reads are frequent. The cost of **retrying failed transactions** is acceptable.
2. Systems emphasize **high concurrency** and **low contention**, e.g.:
	- E-commerce product catalog updates
	- User profile changes
	- Configuration edits by different admins
	- Inventory systems with mostly read operations

It should not be used where

- There are **frequent concurrent updates** to the same data → too many conflicts and retries.
- Transactions are **long-running** (increases likelihood that data changes in the meantime).
- The system must provide **strict serializability** (e.g., financial ledgers, booking systems).
- Retrying failed updates is **expensive or complex** (e.g., multi-step workflows).

### Optimisitc vs pessimistic locking

**Optimistic locking** assumes that most transactions won’t conflict, so it allows multiple transactions to read and modify data simultaneously without locking it upfront. Before committing, each transaction checks whether the data it read has changed (using a version number or timestamp). If the data is unchanged, the update succeeds; if it has been modified by another transaction, the update fails, and the transaction must retry. This approach minimizes locking overhead and improves performance in low-contention environments like read-heavy systems, but it can lead to retries and rollbacks if conflicts do occur.

**Pessimistic locking**, on the other hand, assumes that conflicts are likely, so it locks the data as soon as a transaction reads or modifies it, preventing others from accessing it until the lock is released. This ensures safety and consistency because no two transactions can change the same data concurrently, but it can reduce system throughput and increase waiting time. Pessimistic locking is preferred in high-contention or write-heavy systems where the cost of retries (in optimistic locking) would be higher than the cost of blocking.