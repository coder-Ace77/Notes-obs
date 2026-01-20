
---

Java database connectivity is java api to interact with realtional databases. 
Java provides the JDBC API, which includes classes like `DriverManager` that help manage database drivers. JDBC driver manager are used to manager the `many` drivers and then a driver knows how to talk to database using api. 

The **JDBC Driver** is what knows how to communicate with the specific **database** (e.g., MySQL, Oracle) using the database's native protocol (often via sockets).

- The **DriverManager** helps **select the correct driver** for the JDBC URL you use (like `jdbc:mysql://...`).


> [!NOTE] Diver
> > **A piece of software that allows a higher-level program (like an operating system or application) to communicate with a specific hardware device or external system.**

### Type 1:

It is JDBC ODBC bridge driver. 


> [!NOTE] ODBC
> **ODBC** stands for **Open Database Connectivity**.
It is a **standard API (Application Programming Interface)** for accessing **relational databases**. ODBC allows applications to communicate with a wide variety of database management systems (**DBMS**) — like MySQL, SQL Server, Oracle, etc. — **without being tied to a specific one**.
When an app wants to interact with a database, it sends SQL queries to the ODBC **Driver**, which then translates those commands into DB-specific operations.

```bash
Java App → JDBC API → JDBC-ODBC Bridge → ODBC Driver → Database
```

Requires **ODBC drivers** installed on the client machine.

### Type 2:

Converts JDBC calls into native database calls using database specified native libraries.

```bash
Java App → JDBC API → Native API (via JNI) → Database
```

### Type 3:

It is network protocol driver where JDBC calls to a **middleware server** (written in Java), which then communicates with the database.

```bash
Java App → JDBC API → Middleware Server → Database
```

This is slow as we need one middleware server which slows it down.
### Type 4:

- **Pure Java driver** that converts JDBC calls directly into the **database-specific protocol**.

```bash
Java App → JDBC API → Type 4 Driver → Database
```

- **No native libraries**, fully **cross-platform**.
- **Best performance** and **easiest deployment**.
- Ideal for **cloud, microservices, Docker, etc.**

#### JDBC format to connect:

JDBC url is a string which specifies:
Which db to connect to , where it is located host or post , which db name to use 

```bash
jdbc:<subprotocol>:<subname>
```

subprotocol: the **database type or driver name** (e.g., mysql, postgresql, oracle).
subname: database-specific info like host, port, database name, and params.

eg:

```bash
jdbc:postgresql://<host>:<port>/<database>
```

### Tutorial on how to connect:

First we need to add the dependency to pom

```xml
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.33</version> <!-- check for latest version -->
</dependency>
```

code to connect and query

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class JdbcExample {
    public static void main(String[] args) {
        // JDBC URL, username, password
        String url = "jdbc:mysql://localhost:3306/testdb?useSSL=false&serverTimezone=UTC";
        String user = "root";
        String password = "your_password";

        try {
            // 1. Establish the connection
            Connection conn = DriverManager.getConnection(url, user, password);

            // 2. Create a statement object to send SQL
            Statement stmt = conn.createStatement();

            // 3. Execute a query and get the result set
            ResultSet rs = stmt.executeQuery("SELECT id, name, email FROM users");

            // 4. Iterate over the result set and print results
            while (rs.next()) {
                int id = rs.getInt("id");
                String name = rs.getString("name");
                String email = rs.getString("email");
                System.out.printf("User: %d, %s, %s%n", id, name, email);
            }

            // 5. Close resources (important!)
            rs.close();
            stmt.close();
            conn.close();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### Statements : 

Statements in JDBC are used to send SQL queries or commands to database. There are three kinds of statements: 

| Statement Type        | Purpose                                                | Use Case                                                           |
| --------------------- | ------------------------------------------------------ | ------------------------------------------------------------------ |
| **Statement**         | For executing simple SQL statements without parameters | Simple queries or updates without input variables                  |
| **PreparedStatement** | For executing precompiled SQL with parameters          | Queries or updates with input parameters; safer and more efficient |
| **CallableStatement** | For executing stored procedures in the database        | Calling stored procedures or functions                             |
##### Statement (Basic)

- Created via `Connection.createStatement()`.
- Used to execute simple, static SQL statements.
- Vulnerable to **SQL Injection** if user input is concatenated.

```bash
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM users");
```

#### Prepared statments:

Created via `Connection.prepareStatement(sql)`.Supports **parameterized queries** using `?` placeholders.Precompiled by the database for better performance.Protects against **SQL Injection*. Allows setting input parameters dynamically.

```java
String sql = "SELECT * FROM users WHERE id = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setInt(1, 5); // set parameter 1 to value 5

ResultSet rs = pstmt.executeQuery();
```

#### Callable statements:

They are used to execute stored flows. Used to execute **stored procedures** or database functions.

```java
CallableStatement cstmt = conn.prepareCall("{call getUserById(?)}");
cstmt.setInt(1, 5);
ResultSet rs = cstmt.executeQuery();
```

### Result set:

A **ResultSet** is a **Java object** that represents the data returned by a **SQL query** executed against a database.

- When you run a **SELECT** query using JDBC, the database sends back the matching rows.
- The JDBC driver wraps those rows into a **ResultSet** object.
- You can then **iterate over** this ResultSet to read the data row-by-row.

Result set object has internal cursor pointing to current row. 

```java
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT id, name FROM users");

while (rs.next()) { // moves cursor to next row, returns false when no more rows
    int id = rs.getInt("id");          // get data by column name
    String name = rs.getString("name");
    System.out.println("User: " + id + ", " + name);
}

rs.close();
stmt.close();
```

|SQL Command|JDBC Method|Returns|
|---|---|---|
|`SELECT`|`executeQuery()`|`ResultSet`|
|`INSERT`, `UPDATE`, `DELETE`|`executeUpdate()`|`int` (number of rows affected)|
|Any SQL (unknown type)|`execute()`|`boolean` (true if result is a `ResultSet`)|
```java
boolean hasResultSet = stmt.execute("UPDATE users SET name = 'Alice' WHERE id = 1");

if (hasResultSet) {
    ResultSet rs = stmt.getResultSet();
    // Not applicable in this case
} else {
    int updateCount = stmt.getUpdateCount();
    System.out.println("Rows updated: " + updateCount);
}
```

execute returns a boolean and result is ResultSet rs. 




