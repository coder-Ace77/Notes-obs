
---

**Database** is far more than a simple electronic filing cabinet; it is a systematic, organized collection of data designed for rapid search and retrieval. The **need** for databases arises from the limitations of traditional file systems, specifically the requirements for **concurrency** (multiple people accessing data at once), **integrity** (ensuring data remains accurate), and **scalability** (handling petabytes of data without losing speed).

A **Database Management System (DBMS)** is the software layer that sits between the data and the user. It acts as a gatekeeper, handling tasks like storage, security, and transaction management. Without a DBMS, developers would have to manually write code to handle how data is physically written to a disk every single time they want to save a record.

#### Key Components of a DBMS:

- **Query Processor:** Interprets user commands (like SQL) and optimizes them for performance.
- **Storage Manager:** Controls how data is physically stored and retrieved from the hardware.
- **Transaction Manager:** Ensures that database operations are reliable and follow **ACID** properties.

The architecture of a database system dictates how the client (user), the server, and the business logic interact.

Data abstraction is the process of hiding complex storage details from users to simplify their interaction with the system. This is typically visualized through the

**Three-Schema Architecture**:

**Physical (Internal) Level:** The lowest level. It describes **how** the data is actually stored (e.g., on SSDs, using B-trees or hashing). This is managed by system programmers.**Physical (Internal) Level:** The lowest level. It describes **how** the data is actually stored (e.g., on SSDs, using B-trees or hashing). This is managed by system programmers.

**Logical (Conceptual) Level:** The middle level. It describes **what** data is stored and the relationships between them (e.g., tables, fields, and constraints). This is where Database Administrators (DBAs) work.

**View (External) Level:** The highest level. It describes only the **part** of the database relevant to a specific user. For example, a student can see their grades, but not the salaries of the professors
