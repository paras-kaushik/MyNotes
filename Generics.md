### Java Generics Crash Course

Generics in Java provide a way to create classes, interfaces, and methods with a placeholder for types. This allows for type safety and code reusability without sacrificing performance.

### Key Concepts

1. **Type Safety**: Generics ensure that you can only use the specified type, reducing runtime errors.
2. **Code Reusability**: Write a single class or method that can operate on different types.
3. **Compile-Time Checking**: Errors are caught at compile time rather than runtime.

### Basic Syntax

**Generic Class Example**:

```java
public class Box<T> {
    private T item;

    public void setItem(T item) {
        this.item = item;
    }

    public T getItem() {
        return item;
    }
}
```

**Usage**:

```java
Box<String> stringBox = new Box<>();
stringBox.setItem("Hello");
String item = stringBox.getItem();
```

### Practical Use Cases

#### 1. Collections Framework

**Problem**: Before generics, collections could store any object, leading to potential `ClassCastException`.

**Solution**: Generics provide type safety.

**Example**:

```java
import java.util.ArrayList;
import java.util.List;

public class GenericsInCollections {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Hello");
        // list.add(123); // Compile-time error

        for (String item : list) {
            System.out.println(item);
        }
    }
}
```

#### 2. Generic Methods

**Problem**: Methods that operate on different types required multiple overloads.

**Solution**: Generic methods allow a single method to handle different types.

**Example**:

```java
public class GenericMethodExample {

    public static <T> void printArray(T[] array) {
        for (T element : array) {
            System.out.println(element);
        }
    }

    public static void main(String[] args) {
        Integer[] intArray = {1, 2, 3};
        String[] strArray = {"A", "B", "C"};

        printArray(intArray);
        printArray(strArray);
    }
}
```

#### 3. Bounded Types

**Problem**: Need to restrict the types that can be used with generics.

**Solution**: Bounded types allow you to specify constraints.

**Example**:

```java
public class BoundedTypeExample {

    public static <T extends Number> double sum(T a, T b) {
        return a.doubleValue() + b.doubleValue();
    }

    public static void main(String[] args) {
        System.out.println(sum(10, 20)); // Works with Integer
        System.out.println(sum(10.5, 20.5)); // Works with Double
        // System.out.println(sum("10", "20")); // Compile-time error
    }
}
```

#### 4. Wildcards

**Problem**: Need flexibility in accepting different types of generic classes.

**Solution**: Wildcards provide flexibility in method parameters.

**Example**:

```java
import java.util.List;

public class WildcardExample {

    public static void printList(List<?> list) {
        for (Object item : list) {
            System.out.println(item);
        }
    }

    public static void main(String[] args) {
        List<Integer> intList = List.of(1, 2, 3);
        List<String> strList = List.of("A", "B", "C");

        printList(intList);
        printList(strList);
    }
}
```

### Common Pitfalls

1. **Type Erasure**: Generics are implemented using type erasure, meaning the generic type information is not available at runtime. This can lead to issues with reflection and casting.

2. **Primitive Types**: Generics do not work with primitive types. Use wrapper classes like `Integer` instead of `int`.

3. **Overuse**: Avoid overcomplicating code with excessive use of generics. Use them where they provide clear benefits.

### Conclusion

Generics in Java enhance type safety, code reusability, and maintainability. By understanding and applying generics effectively, you can write more robust and flexible code. Practice using generics in different scenarios to become proficient in their use.
