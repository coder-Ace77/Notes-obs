
---

Cassandra supports three main features -

- Highly scalable
- Highly available
- Fault tolerant

## Basics of cassandra

### Keyspace

The top-level organizational unit in Cassandra, equivalent to a "database" in relational systems like Postgres or MySQL. A keyspace defines replication strategies (discussed later) for managing data redundancy and availability. It also owns any user-defined-types (UDTs) you might create.

### Table

Lives within a keyspace and organizes data into rows. Each table has a schema that defines its columns and primary key structure.

### Row

A single record in a table, identified by a primary key. Each row stores values across multiple columns.

### Column

The actual data storage unit. A column has a name, a type, and a value for that specific row. Not all columns need to be specified per row in a Cassandra table. Cassandra is a wide-column database so the specified columns can vary per row in a table, making Cassandra more flexible than something like a relational database, which requires an entry for every column per row (even if that entry is NULL). Additionally, every column has timestamp metadata associated with it, denoting when it was written. When a column has a write conflict between replicas, it is resolved via "last write wins".

At the most basic level, you can liken Cassandra's data structures to a large JSON.

```json
{
  "keyspace1": {
    "table1": {
      "row1": {
        "col1": 1,
        "col2": "2"
      },
      "row2": {
        "col1": 10,
        "col3": 3.0
      },
      "row3": {
        "col4": {
          "company": "Hello Interview",
          "city": "Seattle",
          "state": "WA"
        }
      }
    }
  }
}
```

Cassandra columns support a plethora of types, including user-defined types and JSON values. This makes Cassandra very flexible as a data store for both flat and nested data.