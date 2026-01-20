
---

| Type      | Size (bits) | Default Value | Example                |
| --------- | ----------- | ------------- | ---------------------- |
| `byte`    | 8           | `0`           | `byte b = 10;`         |
| `short`   | 16          | `0`           | `short s = 1000;`      |
| `int`     | 32          | `0`           | `int x = 100000;`      |
| `long`    | 64          | `0L`          | `long l = 100000000L;` |
| `float`   | 32          | `0.0f`        | `float f = 10.5f;`     |
| `double`  | 64          | `0.0d`        | `double d = 99.99;`    |
| `char`    | 16          | `'\u0000'`    | `char c = 'A';`        |
| `boolean` | 1 (logical) | `false`       | `boolean b = true;`    |

### Type convertions: 

This happens automatically where we convert small to large type.

```java
byte b = 10;
int x = b;       // byte → int
long l = x;      // int → long
float f = l;     // long → float
double d = f;    // float → double
```

Explicit type convertion : large to small

```java
double d = 99.99;
int x = (int) d;  // lose decimal part → 99
long l = 1000000L;
short s = (short) l;  // may lose data if too big
```

Java provides **object versions** of primitive types — called **wrapper classes**. Found in java.lang package.
These are objects and typically take more space than primitive counterparts.

| Primitive | Wrapper Class |
| --------- | ------------- |
| `byte`    | `Byte`        |
| `short`   | `Short`       |
| `int`     | `Integer`     |
| `long`    | `Long`        |
| `float`   | `Float`       |
| `double`  | `Double`      |
| `char`    | `Character`   |
| `boolean` | `Boolean`     |
### Convertions(Autoboxing and unboxing):


> [!NOTE] Title
> **Autoboxing** is the automatic **conversion of a primitive type to its corresponding wrapper class** when an object is needed.
> Unboxing** is the automatic **conversion of a wrapper object back to its primitive type**.


```java
// Autoboxing: primitive → wrapper
int a = 5;
Integer obj = a;  // automatically boxed

// Unboxing: wrapper → primitive
Integer x = 100;
int b = x;       // automatically unboxed
```


This autboxing and unboxing can happen easily in following situations:
- Collections (`ArrayList<Integer>`)
- Method calls expecting objects
- Assignments

### Why use wrappers?

1. Allow utility methods.
2. Nullability (Primitives can not be null)
3. Can be used with collections.

### Autoboxing in action

```java
List<Integer> list = new ArrayList<>();
list.add(5);  // int → Integer automatically (autoboxing)
```

Behind the schene

```java
list.add(Integer.valueOf(5));
```

### Primitive methods:

##### parseXXX(String s) -> Convert String → Primitive:

```java
int x = Integer.parseInt("123");
```

##### valueOf(String s) Convert String → Wrapper Object

```java
Integer x = Integer.valueOf("123");
```

##### toString() Convert Primitive → String

```java
String s = Integer.toString(123);
```

##### xxxValue() Convert wrapper -> Primitive

```java
Integer x = 5;
int y = Integer.intValue(x);
```

### Comprison methods:

#### equals(Obj obj):

Checks for equality of value and returns true or false. 
It also checks type so if type don't match returns false even if abstract value is same.
```java
Integer a = 100;
Integer b = 100;
System.out.println(a.equals(b));  // true ✅

Integer a = 100;
Long l = 100L;
System.out.println(a.equals(l));  // false ❌ (different types)
```

#### compare(x,y):

Static method to **compare two primitive values**.
This is used to compare the two values. returns 0,-1,1 depending upon the condition
```java
System.out.println(Integer.compare(5, 10));   // -1 (5 < 10)
System.out.println(Integer.compare(10, 10));  //  0 (equal)
System.out.println(Integer.compare(20, 10));  //  1 (20 > 10)
```

#### compareTo():
Object method for comparison
Used when comparing **two wrapper objects**.

```java
Integer a = 10;
Integer b = 20;
System.out.println(a.compareTo(b));  // -1 (a < b)
System.out.println(b.compareTo(a));  //  1 (b > a)
System.out.println(a.compareTo(10)); //  0 (equal)
```

#### Constants:

```java
System.out.println(Integer.MAX_VALUE);  // 2147483647
System.out.println(Integer.MIN_VALUE);  // -2147483648
System.out.println(Double.MAX_VALUE);   // 1.7976931348623157E308
```

### Inheritance and implementation:

```kotlin
                    java.lang.Object
                          │
        ┌─────────────┬──────────────┬─────────────┐
    java.lang.Number   Character      Boolean
        │
   ┌────┬────┬────┬────┬────┐
 Byte Short Int  Long Float Double
```

Every wrapper class ultimately inherits from `java.lang.Object`, which provides methods like:
- `toString()`
- `hashCode()`
- `equals(Object obj)`
- `getClass()`

Each wrapper class (except `Void`) implements `Comparable<T>`, allowing them to be compared and sorted.