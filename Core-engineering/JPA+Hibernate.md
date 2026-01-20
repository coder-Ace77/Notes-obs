
---

HIbernate is the default implementation of JPA.

### Entity: 
Entities in JPA are nothing but POJOs representing data that can be persisted in the database. An entity represents a table stored in a database. Every instance of an entity represents a row in the table. To do this we use @Entity notation

```java
@Entity
public class Student {
    // fields, getters and setters   
}
```

The entity name defaults to the name of the class.Because various JPA implementations will try subclassing our entity to provide their functionality, **entity classes must not be declared _final_.**

```java
@Entity(name="student")
public class Student{
    // fields, getters and setters   
}
```

@Id annotation is used to define the primary key. There are four genearation options  **The value can be _AUTO, TABLE, SEQUENCE,_ or _IDENTITY:_**

```java
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private String name;   
    // getters and setters
}
```

@Table notation is used to define table name

```java
@Entity
@Table(name="STUDENT")
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    @Column(name="STUDENT_NAME", length=50, nullable=false, unique=false)
    private String name;   
    // other fields, getters and setters
}
```

@Column can be used to define the details of column in the table.

```java
@Entity
@Table(name="STUDENT")
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    
    @Column(name="STUDENT_NAME", length=50, nullable=false, unique=false)
    private String name;   
    // other fields, getters and setters
}
```

@Transient annotation makes the field non persistent in db.

Reference: https://www.baeldung.com/learn-jpa-hibernate

#### Default values in Entity:

The first way to set a default column value is to **set it directly as an entity property value**:

```java
@Entity
public class User {
    @Id
    private Long id;
    private String firstName = "John Snow";
    private Integer age = 25;
    private Boolean locked = false;
}
```

There is one drawback to this solution.When we take a look at the SQL table definition, we won’t see any default value in it:

So, **if we override them with _null_, the entity will be saved without any error**:

To create **a default value directly in the SQL table definition**, we can use the _@Column_ annotation and set its _columnDefinition_ parameter:

```java
@Entity
public class User {
    @Id
    Long id;

    @Column(columnDefinition = "varchar(255) default 'John Snow'")
    private String name;

    @Column(columnDefinition = "integer default 25")
    private Integer age;

    @Column(columnDefinition = "boolean default false")
    private Boolean locked;
}
```

We can also use @ColumnDefault

```java
@Entity
@Table(name="users_entity")
public class UserEntity {
    @Id
    private Long id;

    @ColumnDefault("John Snow")
    private String name;

    @ColumnDefault("25")
    private Integer age;

    @ColumnDefault("false")
    private Boolean locked;
}
```

Mapping the Entity to table

```java
@Entity
@Table(name = "ARTICLES")
public class Article {
    // ...
}
```

@Size

For validations, we’ll use _@Size_**,** a bean validation annotation. We’ll use the property _middleName_ annotated with _@Size_ to validate its value between the attributes _min_ and _max:_

```java
public class User {
    // ...
    @Size(min = 3, max = 15)
    private String middleName;
    // ...
}
```

_@Length_ is the Hibernate-specific version of _@Size._ We’ll enforce the range for _lastName_ using _@Length_:

```java
@Entity
public class User {
    // ...
    @Length(min = 3, max = 15)
    private String lastName;
    // ...
}
```

_@Column_ to **indicate specific characteristics of the physical database column.**

```java
@Entity
public class User {

    @Column(length = 3)
    private String firstName;

    // ...

}
```

The resulting column will be generated as a _VARCHAR(3),_ and trying to insert a longer string will result in an SQL error.

### Converting one entity to database attribute:

Suppose wev have PersonName which is needed to be converted

```java
public class PersonName implements Serializable {

    private String name;
    private String surname;

    // getters and setters
}
```

And is used  in Person
```java
@Entity(name = "PersonTable")
public class Person {
    @Convert(converter = PersonNameConverter.class)   // added to converter
    private PersonName personName;

    //...
}
```

convertor
```java
@Converter
public class PersonNameConverter implements 
  AttributeConverter<PersonName, String> {

    private static final String SEPARATOR = ", ";

    @Override
    public String convertToDatabaseColumn(PersonName personName) {
        if (personName == null) {
            return null;
        }

        StringBuilder sb = new StringBuilder();
        if (personName.getSurname() != null && !personName.getSurname()
            .isEmpty()) {
            sb.append(personName.getSurname());
            sb.append(SEPARATOR);
        }

        if (personName.getName() != null 
          && !personName.getName().isEmpty()) {
            sb.append(personName.getName());
        }

        return sb.toString();
    }

    @Override
    public PersonName convertToEntityAttribute(String dbPersonName) {
        if (dbPersonName == null || dbPersonName.isEmpty()) {
            return null;
        }

        String[] pieces = dbPersonName.split(SEPARATOR);

        if (pieces == null || pieces.length == 0) {
            return null;
        }

        PersonName personName = new PersonName();        
        String firstPiece = !pieces[0].isEmpty() ? pieces[0] : null;
        if (dbPersonName.contains(SEPARATOR)) {
            personName.setSurname(firstPiece);

            if (pieces.length >= 2 && pieces[1] != null 
              && !pieces[1].isEmpty()) {
                personName.setName(pieces[1]);
            }
        } else {
            personName.setName(firstPiece);
        }

        return personName;
    }
}
```

### Joining two tables:

We can implement with foreign key in JPA 
There can be many types of mapping and in entity we usually handle the entire object for example there is an order and each order can have multiple products.
So although we may store the data as mapping in form of table but in java we store it as concrete objects.

Now how to have a mapping.

```java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id")
    private Long id;
    //... 

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "address_id", referencedColumnName = "id")
    private Address address;

    // ... getters and setters
}
```

Note that **we place the _@OneToOne_ annotation** on the related entity field, _Address_.

Also, **we need to place [the _@JoinColumn_ annotation](https://www.baeldung.com/jpa-join-column)** to configure the name of the column in the _users_ table that maps to the primary key in the _address_ table. If we don’t provide a name, Hibernate will [follow some rules](http://docs.jboss.org/hibernate/jpa/2.2/api/javax/persistence/JoinColumn.html) to select a default one.

Finally, note in the next entity that we won’t use the _@JoinColumn_ annotation there. This is because we only need it on the owning side of the foreign key relationship. **Simply put, whoever owns the foreign key column gets the _@JoinColumn_ annotation.**

```java
@Entity
@Table(name = "address")
public class Address {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id")
    private Long id;
    //...

    @OneToOne(mappedBy = "address")
    private User user;

    //... getters and setters
}
```

Using and modelling with shared primary key.

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id")
    private Long id;

    //...

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
    @PrimaryKeyJoinColumn
    private Address address;

    //... getters and setters
}
```

```java
@Entity
@Table(name = "address")
public class Address {

    @Id
    @Column(name = "user_id")
    private Long id;

    //...

    @OneToOne
    @MapsId
    @JoinColumn(name = "user_id")
    private User user;
   
    //... getters and setters
}
```

Modelling with Join table:

A join table is a table which bridges the gap in two tables. For example User and userworkstation

```java
@Entity
@Table(name = "employee")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id")
    private Long id;

    //...

    @OneToOne(cascade = CascadeType.ALL)
    @JoinTable(name = "emp_workstation", 
      joinColumns = 
        { @JoinColumn(name = "employee_id", referencedColumnName = "id") },
      inverseJoinColumns = 
        { @JoinColumn(name = "workstation_id", referencedColumnName = "id") })
    private WorkStation workStation;

    //... getters and setters
}
```

```java
@Entity
@Table(name = "workstation")
public class WorkStation {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id")
    private Long id;

    //...

    @OneToOne(mappedBy = "workStation")
    private Employee employee;

    //... getters and setters
}
```

