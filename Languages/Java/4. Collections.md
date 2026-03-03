
---

The **Java Collections API** is a unified architecture for **storing, retrieving, and manipulating groups of objects**. It’s part of the `java.util` package and provides interfaces and classes for different types of collections such as lists, sets, maps, and queues.

```java
import java.util.*;
```

#### Core Interfaces in the Java Collections API

| Interface                              | Description                                       |
| -------------------------------------- | ------------------------------------------------- |
| `Collection<E>`                        | Root of the collection hierarchy                  |
| `List<E>`                              | Ordered collection, allows duplicates             |
| `Set<E>`                               | No duplicates                                     |
| `SortedSet<E>` / `NavigableSet<E>`     | Set with sorted elements                          |
| `Queue<E>`                             | Designed for holding elements prior to processing |
| `Deque<E>`                             | Double-ended queue                                |
| `Map<K,V>`                             | Key-value pairs                                   |
| `SortedMap<K,V>` / `NavigableMap<K,V>` | Map with keys in sorted order                     |
![[Pasted image 20250620093137.png]]

1. **`Iterable`** is the root for anything that can be used in a for-each loop.
2. **`Collection`** is the base for `List`, `Set`, and `Queue`.
3. **`LinkedList`** is both a `List` and a `Deque`, so it fits in both paths.
4. **`TreeSet`** and **`TreeMap`** maintain natural or custom order using **Red-Black trees**.

### ArrayList:

Acts like a dynamic array.

```java
ArrayList<Integer> arr = new ArrayList<>();
List<Integer> arr = new ArrayList<>();

arr.size(); // size of arraylist

arr.add(10); // autoboxing
arr.isEmpty();

arr.set(index,value); // putting some value at index
arr.get(index); // gettting eleement from 

arr.remove(index); // remove by index
arr.remove(new Integer(11)); // remove by value only first occurance and returns the value
```

#### Collections methods:

### `java.util.Collections`

Collections is a utility class having lot of static methods listed below.

|Method|Description|
|---|---|
|`sort(List<T>)`|Sorts a list in natural order|
|`reverse(List<T>)`|Reverses the order of elements|
|`shuffle(List<T>)`|Randomizes elements|
|`min(Collection<T>)` / `max(Collection<T>)`|Finds min/max element|
|`binarySearch(List<T>, key)`|Searches in sorted list|
|`fill(List<T>, val)`|Fills list with a single value|
|`frequency(Collection<T>, obj)`|Counts occurrences of an object|
|`replaceAll(List<T>, oldVal, newVal)`|Replaces all occurrences|
```java
import java.util.*;

List<Integer> list = Arrays.asList(3, 1, 2);
Collections.sort(list);     // [1, 2, 3]
Collections.reverse(list);  // [3, 2, 1]

list.sort((s1, s2) -> s2.marks - s1.marks); // descending by marks sort is also available in array
```


### Static imports:

From java 5 we can directly import static methods. Like sort reverse

```java
import static java.util.Collections.sort;
import static java.util.Collections.reverse;

List<Integer> list = Arrays.asList(3, 2, 1);
sort(list);        // instead of Collections.sort(list)
reverse(list);
```

### HashMap:

Hashed implementation of map , contains key and value pairs.

```java

HashMap<Integer,String> map = new HashMap<>(); // creating hashmap

map.put(key,value); // key must be unique existing key will be updated.
map.containsKey(key); // checks of key is present

map.get(key); // get the value
map.remove(key); // remove entry

map.clear();

map.keySet(); // returns all the keys

map.values(); // gives all values

// iterating

for(Integer each:map.KeySet()){
	System.out.println(map.get(each));
}
```

HashMap is key value data structure that stores using hashing of keys.


> [!NOTE] Working
> - Internally uses **array of buckets**   
> - Each bucket is a **LinkedList (Java 7)** or **Balanced Tree (Java 8+ for many collisions)**  
> - Hash code of the key determines which bucket the pair goes into
> - Compute `hashCode()` of the key.
> - Map it to a bucket index: `hash % capacity`.
> - Store entry as a `Map.Entry<K, V>` object.
> - Initially via **chaining** (LinkedList).
> - In Java 8+, if collisions in a bucket exceed 8, switches to a **balanced tree (Red-Black Tree)**.

To use custom object as key. We must override equals and hashCode() function 

```java
class Person {
    String name;
    int age;

    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // Overriding equals()
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Person)) return false;
        Person p = (Person) o;
        return age == p.age && name.equals(p.name);
    }

    // Overriding hashCode()
    @Override
    public int hashCode() {
        return name.hashCode() + 31 * age;
    }
}
```

Remember that every class is subclass of Object and Object has these two methods.

### HashSet:

A **`HashSet<E>`** is a collection that **stores unique elements only** (no duplicates).

| Method          | Description                     |
| --------------- | ------------------------------- |
| `add(E e)`      | Adds the element if not present |
| `remove(E e)`   | Removes the element             |
| `contains(E e)` | Returns true if present         |
| `isEmpty()`     | Checks if the set is empty      |
| `size()`        | Returns number of elements      |
| `clear()`       | Removes all elements            |
| `iterator()`    | Returns iterator for traversal  |

### TreeSet:

| Method          | Description                                |
| --------------- | ------------------------------------------ |
| `add(E e)`      | Adds element (throws if null or duplicate) |
| `remove(E e)`   | Removes element                            |
| `contains(E e)` | Checks presence                            |
| `isEmpty()`     | Checks if set is empty                     |
| `size()`        | Number of elements                         |
| `clear()`       | Clears all elements                        |
| `first()`       | Smallest element                           |
| `last()`        | Largest element                            |
| `ceiling(E e)`  | Smallest ≥ e                               |
| `floor(E e)`    | Largest ≤ e                                |
| `higher(E e)`   | Strictly greater                           |
| `lower(E e)`    | Strictly less                              |
| `pollFirst()`   | Removes and returns first                  |
| `pollLast()`    | Removes and returns last                   |

```java

TreeSet<String> animals = new TreeSet<>();
animals.add("Dog");
animals.add("Cat");

for (String animal : animals) {
    System.out.println(animal);
}

animals.first() // first
animals.last() // largest element
```


> [!NOTE] Note
> **"Poll"** is a term from **queue-based APIs**, meaning: _retrieve and remove the head (or relevant) element_.
> Since `TreeSet` has a sorted order (like a priority queue), this term is used similarly.

```java
// all three sets are iterable
for (String item : set) {
    System.out.println(item);
}
Iterator<String> it = set.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}
```

#### Custom ordering in treeSet

```java

// using comparators
TreeSet<Person> set = new TreeSet<>(Comparator.comparing(p -> p.name));

// pass a comparator
Comparator<Person> comp = Comparator.comparing((Person p) -> p.age).thenComparing(p -> p.name);

TreeSet<Person> set = new TreeSet<>(comp);

// or using lambda function
TreeSet<Person> people = new TreeSet<>((p1, p2) -> p1.age - p2.age);
```

### Iterator

An objecct that allows us to traverse a collection.

```java
Iterator<String> itr = names.iterator(); // some collection
while(itr.hasNext()){
	String ele = itr.next(); // returns current ele and moves to next.
}
```


### Stream API

ArrayList can be directly converted to stream using stream() method.

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.stream().sorted(Comparator.comparing(String::length)).forEach(System.out::println);
```

| Operation            | Description                        | Example                        |
| -------------------- | ---------------------------------- | ------------------------------ |
| `filter(Predicate)`  | Keep elements that match condition | `x -> x % 2 == 0`              |
| `map(Function)`      | Transform each element             | `x -> x * 2`                   |
| `sorted()`           | Sort stream (natural order)        | `sorted()`                     |
| `sorted(Comparator)` | Custom sort                        | `sorted((a, b) -> b - a)`      |
| `limit(n)`           | Take first `n` elements            | `limit(5)`                     |
| `skip(n)`            | Skip first `n` elements            | `skip(2)`                      |
| `distinct()`         | Remove duplicates                  | `distinct()`                   |
| `collect(...)`       | Terminal op to gather results      | `collect(Collectors.toList())` |
| `forEach(...)`       | Apply action on each element       | `forEach(System.out::println)` |
| `reduce(...)`        | Combine elements into one          | `reduce(0, (a, b) -> a + b)`   |
- Stream operations are **lazy** — nothing happens until a **terminal operation** (e.g., `collect`, `count`, `forEach`) is invoked.