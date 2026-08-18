
---

Psql is a client application for the postgres database we can use psql to connect to any postgres db running anywhere in the world.

**`psql`** is the **PostgreSQL interactive terminal** — a command-line tool to interact with a PostgreSQL database.

psql is command line client program. 
Options:

```bash
psql -h <host> -p <port> -U <username> -d <database>
```

-h : hostname basically specify the ip address - default is localhost
-p: specify the port number , by default 5432 if ommited. 
-d: database

| Command            | Description                                      |
| ------------------ | ------------------------------------------------ |
| `\l` or `\list`    | List all databases                               |
| `\c <database>`    | Connect to a specific database                   |
| `\dt`              | List tables in the current database              |
| `\d <table>`       | Describe table structure                         |
| `\q`               | Quit psql                                        |
| `\?`               | Show help on psql commands                       |
| `\h <SQL-command>` | Help on specific SQL command (e.g., `\h SELECT`) |
#### Basics of creating:

```sql
CREATE DATABASE testdb;
CREATE USER testuser WITH ENCRYPTED PASSWORD 'password123';
GRANT ALL PRIVILEGES ON DATABASE testdb TO testuser;
```

we can also run a script 

```bash
psql -U testuser -d testdb -f path/to/script.sql
```

After connection to db we get connected through specified shell and specified db. Then we can run any sql query.


---

## DDL 

```psql
-- Create a new database
CREATE DATABASE mydb;

-- Delete a database
DROP DATABASE mydb;
```

```psql
\c mydb --connect to a db
```

Creating and dropping tables

```psql
-- Create a table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Drop a table
DROP TABLE users;
```

Alter table

```psql
-- Add a new column
ALTER TABLE users ADD COLUMN age INT;

-- Drop a column
ALTER TABLE users DROP COLUMN age;

-- Rename a column
ALTER TABLE users RENAME COLUMN name TO full_name;

-- Rename the table
ALTER TABLE users RENAME TO customers;

```

Indexing 

```psql
-- Create an index on a column
CREATE INDEX idx_users_email ON users(email);

-- Drop an index
DROP INDEX idx_users_email;

```

Create and drop all schema

```psql
-- Create a schema
CREATE SCHEMA my_schema;

-- Drop a schema (and all objects in it)
DROP SCHEMA my_schema CASCADE;

-- Use a schema in table creation
CREATE TABLE my_schema.products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);
```


---

## DML:

Insert into is the general syntax to insert table. 

```psql
INSERT INTO table_name (column1, column2, ...) 
VALUES (value1, value2, ...);
```

Columns with `DEFAULT` or `SERIAL` get their values automatically.

Inserting multiple rows at once

```psql
INSERT INTO products (name, description, category)
VALUES 
('product1', 'desc1', 'category1'),
('product2', 'desc2', 'category2'),
('product3', 'desc3', 'category1');
```

We can also insert generated series wise data

```psql
INSERT INTO products (name, description, category)
SELECT 'product' || gs, 'desc' || gs, 'category' || (1 + (random()*99)::int)
FROM generate_series(1, 1000000) AS gs;
```

So here gerenate_series create a column with name row. select uses that to generate the rows which are then inserted to products.








