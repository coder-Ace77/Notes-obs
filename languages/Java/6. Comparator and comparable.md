
---

Java's Collection.sort method can work on arrayList but internal object must have implemented Comparable or comparator. Otherwise it will give an error.

#### Comparable interface:

- Used to sort collection elements, including objects.
-  Contains single method 
	- public int compareTo(Object obj1) 
	- positive integer, if current object is greater than the specified object.
	- negative integer, if current object is less than the specified object.
	- zero, if current object is equal to the specified object.
- All the Wrapper classes implement the Comparable interface by default.

```java
import java.util.*;

class Student implements Comparable<Student> {
    String name;
    int marks;

    Student(String name, int marks) {
        this.name = name;
        this.marks = marks;
    }
    @Override
    public int compareTo(Student other) {
        return this.marks - other.marks;  // ascending order by marks
    }
    @Override
    public String toString() {
        return name + " (" + marks + ")";
    }
}
public class Main {
    public static void main(String[] args) {
        List<Student> list = new ArrayList<>();
        list.add(new Student("Alice", 85));
        list.add(new Student("Bob", 92));
        list.add(new Student("Charlie", 78));
        Collections.sort(list);  // uses compareTo
        System.out.println(list);
    }
}
```

However we quickly run into problems if we need multiple sorting orders Each time we would not want to change the class itself.

#### Comparator Interface:

The `Comparator<T>` interface is used to define **custom sorting logic** for objects **outside the class** (unlike `Comparable`, which is implemented _inside_ the class).

- Contains two methods 
	- public int compare(Object obj1, Object2 obj) 
		- positive integer, if first object is greater than the second object.
		- negative integer, if first object is less than the second object.
		- zero, if first object is equal to the second object.
- public boolean equals(Object obj). 
-  compare current object with specified object.

We can simple pass the comparator instance to sort method.

```java
import java.util.*;

class Student {
    String name;
    int marks;

    Student(String name, int marks) {
        this.name = name;
        this.marks = marks;
    }

    @Override
    public String toString() {
        return name + " (" + marks + ")";
    }
}

class SortByMarks implements Comparator<Student> {
    public int compare(Student s1, Student s2) {
        return s1.marks - s2.marks;  // Ascending order
    }
}

public class Main {
    public static void main(String[] args) {
        List<Student> list = new ArrayList<>();
        list.add(new Student("Adil", 90));
        list.add(new Student("Zara", 75));
        list.add(new Student("Bob", 85));

        Collections.sort(list, new SortByMarks());
        System.out.println(list);
    }
}
```

#### We can also use lambda:

```java
list.sort((s1, s2) -> s1.name.compareTo(s2.name));  // sort by name
```

Note: Comparator instances can also work with treeSet/ treeMap
```java
TreeSet<Student> set = new TreeSet<>(new SortByMarks());
```

