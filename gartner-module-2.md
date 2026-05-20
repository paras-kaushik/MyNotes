Your interview is for:

> **Gartner Lead Software Engineer — Java backend + Angular/React + AWS**  
> **Interviewer: Sr Architect**  
> **Duration: 60 minutes**  
> **Friday, May 22, 2026, 3:30 PM IST**

The most likely interview focus:

```text
1. Current project / architecture deep dive
2. Core Java
3. Spring Boot backend
4. REST API design
5. SQL / PostgreSQL
6. DSA in Java
7. AWS / cloud deployment
8. System design
9. Production readiness / observability
10. Leadership / ownership examples
```

Now we should continue with:

# Module 2: Core Java + Spring Boot + Backend — real interview Q&A

I’ll split this module into smaller high-quality parts:

```text
Module 2A: Core Java interview questions
Module 2B: Spring Boot interview questions
Module 2C: REST API + backend design questions
Module 2D: JPA / Hibernate / transactions
Module 2E: Production debugging / performance / observability
```

Let’s start now with:

# Module 2A: Core Java Interview Questions

This will be **interviewer-style**, not textbook-style.

---

# Module 2A — Core Java for Gartner Sr Architect Round

## What they are really testing

A Sr Architect does not only want definitions. They want to know:

```text
Can this person write reliable Java code?
Can this person explain tradeoffs?
Can this person debug production issues?
Can this person mentor others?
Does this person know collections, concurrency, memory, exceptions, OOP?
```

---

# 1. Java Collections

## Q1. How does `HashMap` work internally?

### Strong interview answer

`HashMap` stores key-value pairs. Internally, it uses an array of buckets. When we put a key-value pair, Java calculates the key’s `hashCode()`, converts that hash into a bucket index, and stores the entry there.

If two keys map to the same bucket, that is called a collision. Java handles collisions using linked lists, and when collisions in a bucket become too many, it can convert the bucket into a balanced tree for better lookup performance.

For custom keys, it is very important to correctly implement both `equals()` and `hashCode()`.

### Beginner explanation

Think of `HashMap` like a building with many rooms.

```text
Key → hashCode() → room number → store value in that room
```

If two people get the same room number, Java keeps both in that room and checks equality.

### Code example

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapDemo {

    public static void main(String[] args) {
        Map<String, Integer> salaryMap = new HashMap<>();

        salaryMap.put("Paras", 100);
        salaryMap.put("Amit", 90);

        System.out.println(salaryMap.get("Paras")); // 100
    }
}
```

### Custom key example

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

class EmployeeKey {
    private final String employeeId;
    private final String country;

    public EmployeeKey(String employeeId, String country) {
        this.employeeId = employeeId;
        this.country = country;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof EmployeeKey other)) {
            return false;
        }

        return Objects.equals(employeeId, other.employeeId)
                && Objects.equals(country, other.country);
    }

    @Override
    public int hashCode() {
        return Objects.hash(employeeId, country);
    }
}

public class CustomKeyDemo {

    public static void main(String[] args) {
        Map<EmployeeKey, String> map = new HashMap<>();

        EmployeeKey key = new EmployeeKey("E101", "IN");

        map.put(key, "Paras");

        System.out.println(map.get(new EmployeeKey("E101", "IN"))); // Paras
    }
}
```

### Common follow-up

#### What happens if `hashCode()` is same but `equals()` is false?

Both objects can exist in the same bucket. Java first finds the bucket using hash, then compares keys using `equals()`.

#### What happens if `equals()` is true but `hashCode()` is different?

That breaks the contract. The object may not be found in a `HashMap`.

### Senior-level note

> In production, I avoid using mutable objects as HashMap keys. If a key’s fields change after insertion, its hash bucket may change logically, and lookup can fail.

---

# 2. HashMap vs ConcurrentHashMap

## Q2. Difference between `HashMap` and `ConcurrentHashMap`?

### Strong interview answer

`HashMap` is not thread-safe. If multiple threads modify it concurrently, it can produce inconsistent results.

`ConcurrentHashMap` is designed for concurrent access. It allows multiple threads to read and update safely with better performance than synchronizing the whole map.

Also, `ConcurrentHashMap` does not allow null keys or null values.

### Code example

```java
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentHashMapDemo {

    private final Map<String, Integer> requestCount = new ConcurrentHashMap<>();

    public void increment(String userId) {
        requestCount.merge(userId, 1, Integer::sum);
    }

    public int getCount(String userId) {
        return requestCount.getOrDefault(userId, 0);
    }
}
```

### Beginner explanation

```text
HashMap = okay when one thread is using it
ConcurrentHashMap = safer when many threads use it together
```

### Common follow-up

#### Is `ConcurrentHashMap` always enough for thread safety?

No. Individual operations are thread-safe, but multi-step business logic may still need synchronization or atomic operations.

Example problem:

```java
if (!map.containsKey("A")) {
    map.put("A", 1);
}
```

This is not atomic as a whole.

Better:

```java
map.putIfAbsent("A", 1);
```

### Senior-level answer

> I use `ConcurrentHashMap` for shared concurrent maps, but in backend applications I try to avoid shared mutable state when possible. Stateless services are easier to scale horizontally.

---

# 3. `equals()` and `hashCode()`

## Q3. Why do we override both `equals()` and `hashCode()`?

### Strong answer

If two objects are logically equal according to `equals()`, they must return the same `hashCode()`. Collections like `HashMap` and `HashSet` depend on this contract.

If we override only one, lookup, duplicate detection, and set behavior can break.

### Code example

```java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

class User {
    private final Long id;
    private final String email;

    public User(Long id, String email) {
        this.id = id;
        this.email = email;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof User other)) {
            return false;
        }

        return Objects.equals(id, other.id)
                && Objects.equals(email, other.email);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, email);
    }
}

public class EqualsHashCodeDemo {

    public static void main(String[] args) {
        Set<User> users = new HashSet<>();

        users.add(new User(1L, "paras@example.com"));
        users.add(new User(1L, "paras@example.com"));

        System.out.println(users.size()); // 1
    }
}
```

### Interview trap

If `equals()` says two users are same but `hashCode()` differs, `HashSet` may store duplicates.

### Strong phrase

> `equals()` defines logical equality. `hashCode()` helps hash-based collections locate the object efficiently.

---

# 4. ArrayList vs LinkedList

## Q4. Difference between `ArrayList` and `LinkedList`?

### Strong answer

`ArrayList` is backed by a dynamic array. It is fast for random access using index, but insertions/removals in the middle can be costly because elements need shifting.

`LinkedList` is backed by nodes connected through pointers. Insertions/removals are efficient if we already have the node reference, but random access is slower because traversal is required.

In most application code, `ArrayList` is preferred because it has better cache locality and simpler memory layout.

### Table

| Operation | ArrayList | LinkedList |
|---|---:|---:|
| Get by index | O(1) | O(n) |
| Add at end | Amortized O(1) | O(1) |
| Add/remove middle | O(n) | O(n) to find + O(1) remove |
| Memory | Less overhead | More overhead due to nodes |

### Code example

```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

public class ListDemo {

    public static void main(String[] args) {
        List<String> arrayList = new ArrayList<>();
        List<String> linkedList = new LinkedList<>();

        arrayList.add("Java");
        linkedList.add("Spring Boot");

        System.out.println(arrayList.get(0));
        System.out.println(linkedList.get(0));
    }
}
```

### Strong interview phrase

> In real-world backend development, I usually use `ArrayList` unless I specifically need queue/deque behavior or frequent insertions/removals with node references.

---

# 5. Set vs List vs Map

## Q5. Difference between `List`, `Set`, and `Map`?

### Answer

```text
List = ordered collection, allows duplicates
Set = unique elements, no duplicates
Map = key-value pairs
```

### Code example

```java
import java.util.*;

public class CollectionTypesDemo {

    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Java");
        list.add("Java");

        Set<String> set = new HashSet<>();
        set.add("Java");
        set.add("Java");

        Map<String, Integer> map = new HashMap<>();
        map.put("Java", 21);

        System.out.println(list.size()); // 2
        System.out.println(set.size());  // 1
        System.out.println(map.get("Java")); // 21
    }
}
```

---

# 6. Java Immutability

## Q6. What is an immutable class? How do you create one?

### Strong answer

An immutable class is a class whose object state cannot change after creation.

To create one:

```text
1. Make class final.
2. Make fields private and final.
3. Do not provide setters.
4. Initialize fields through constructor.
5. Return defensive copies for mutable fields.
```

### Code example

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public final class Employee {

    private final Long id;
    private final String name;
    private final List<String> skills;

    public Employee(Long id, String name, List<String> skills) {
        this.id = id;
        this.name = name;

        // Defensive copy
        this.skills = new ArrayList<>(skills);
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public List<String> getSkills() {
        // Return unmodifiable copy/view
        return Collections.unmodifiableList(skills);
    }
}
```

### Interview follow-up

#### Why is immutability useful?

```text
1. Thread-safety
2. Simpler reasoning
3. Safe sharing
4. Good for keys in HashMap
5. Reduces accidental bugs
```

### Senior-level phrase

> I prefer immutability for DTOs, value objects, config objects, and map keys because it makes code safer and easier to reason about.

---

# 7. Java Records

## Q7. What are records in Java?

### Strong answer

Records are a concise way to create immutable data carrier classes. Java automatically provides constructor, getters, `equals()`, `hashCode()`, and `toString()`.

They are useful for DTOs and value objects.

### Code example

```java
public record UserResponse(
        Long id,
        String name,
        String email
) {
}
```

Usage:

```java
public class RecordDemo {

    public static void main(String[] args) {
        UserResponse response = new UserResponse(
                1L,
                "Paras",
                "paras@example.com"
        );

        System.out.println(response.id());
        System.out.println(response.name());
        System.out.println(response.email());
    }
}
```

### Spring Boot DTO example

```java
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;

public record CreateUserRequest(

        @NotBlank(message = "Name is required")
        String name,

        @Email(message = "Email should be valid")
        @NotBlank(message = "Email is required")
        String email
) {
}
```

### Senior-level note

> I use records for simple DTOs, but avoid them for JPA entities because JPA entities usually require proxying, no-arg constructors, and mutable lifecycle management.

---

# 8. Checked vs Unchecked Exceptions

## Q8. Difference between checked and unchecked exceptions?

### Strong answer

Checked exceptions are checked at compile time and must be handled or declared. They extend `Exception`.

Unchecked exceptions occur at runtime and extend `RuntimeException`. They do not need to be explicitly declared.

In Spring Boot applications, business exceptions are often unchecked custom exceptions handled centrally using `@ControllerAdvice`.

### Code example

```java
import java.io.FileReader;
import java.io.IOException;

public class CheckedExceptionDemo {

    public void readFile() throws IOException {
        FileReader reader = new FileReader("test.txt");
    }
}
```

Unchecked exception:

```java
public class UserNotFoundException extends RuntimeException {

    public UserNotFoundException(Long id) {
        super("User not found with id: " + id);
    }
}
```

Usage:

```java
public User getUser(Long id) {
    return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
}
```

### Strong interview phrase

> For recoverable external failures, checked exceptions can make sense. For business rule violations in Spring Boot APIs, I generally use unchecked custom exceptions and map them to proper HTTP responses.

---

# 9. `final`, `finally`, and `finalize`

## Q9. Difference between `final`, `finally`, and `finalize()`?

### Answer

| Term | Meaning |
|---|---|
| `final` | Keyword used for variables, methods, classes |
| `finally` | Block used with try-catch that usually executes after try/catch |
| `finalize()` | Old object cleanup method called by GC, deprecated and should not be used |

### Code example

```java
public final class PaymentService {

    private final String providerName = "Stripe";

    public final void processPayment() {
        System.out.println("Processing payment");
    }
}
```

`finally` example:

```java
public class FinallyDemo {

    public static void main(String[] args) {
        try {
            System.out.println("Try block");
        } catch (Exception ex) {
            System.out.println("Catch block");
        } finally {
            System.out.println("Finally block");
        }
    }
}
```

### Interview phrase

> `finalize()` should not be used for resource cleanup. We should use try-with-resources or explicit lifecycle management.

---

# 10. Try-with-resources

## Q10. What is try-with-resources?

### Strong answer

Try-with-resources automatically closes resources that implement `AutoCloseable`, such as files, streams, and database connections.

### Code example

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class TryWithResourcesDemo {

    public String readFirstLine(String filePath) throws IOException {
        try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
            return reader.readLine();
        }
    }
}
```

### Why this is better

Without try-with-resources, you may forget to close resources.

```java
BufferedReader reader = null;

try {
    reader = new BufferedReader(new FileReader("file.txt"));
    return reader.readLine();
} finally {
    if (reader != null) {
        reader.close();
    }
}
```

### Senior-level phrase

> In backend applications, resource leaks can cause production issues. Try-with-resources helps prevent file, stream, and connection leaks.

---

# 11. Java Streams

## Q11. What are Java Streams?

### Strong answer

Streams provide a declarative way to process collections. They are useful for filtering, mapping, sorting, grouping, and aggregation.

Streams do not store data. They process data from a source such as a collection.

### Code example

```java
import java.util.List;

public class StreamDemo {

    public List<String> getActiveEmails(List<UserDto> users) {
        return users.stream()
                .filter(UserDto::active)
                .map(UserDto::email)
                .distinct()
                .sorted()
                .toList();
    }
}

record UserDto(String email, boolean active) {
}
```

### Grouping example

```java
import java.util.*;
import java.util.stream.Collectors;

public class GroupingDemo {

    public Map<String, List<Employee>> groupByDepartment(List<Employee> employees) {
        return employees.stream()
                .collect(Collectors.groupingBy(Employee::department));
    }
}

record Employee(String name, String department) {
}
```

### Counting example

```java
import java.util.*;
import java.util.stream.Collectors;

public class FrequencyDemo {

    public Map<String, Long> countWords(List<String> words) {
        return words.stream()
                .collect(Collectors.groupingBy(
                        word -> word,
                        Collectors.counting()
                ));
    }
}
```

### Common follow-up

#### Stream vs parallelStream?

`parallelStream()` can use multiple threads, but it should not be used blindly. It can hurt performance if operations are small, blocking, stateful, or if the common fork-join pool is overloaded.

### Senior-level phrase

> I use streams when they improve readability. For complex branching logic or performance-critical code, a normal loop may be clearer and easier to debug.

---

# 12. Functional Interface and Lambda

## Q12. What is a functional interface?

### Strong answer

A functional interface has exactly one abstract method. It can be implemented using a lambda expression.

Examples:

```text
Predicate<T>
Function<T, R>
Consumer<T>
Supplier<T>
Runnable
Callable
```

### Code example

```java
import java.util.function.Predicate;

public class FunctionalInterfaceDemo {

    public static void main(String[] args) {
        Predicate<Integer> isEven = number -> number % 2 == 0;

        System.out.println(isEven.test(10)); // true
        System.out.println(isEven.test(7));  // false
    }
}
```

### Custom functional interface

```java
@FunctionalInterface
interface DiscountCalculator {
    double calculate(double amount);
}

public class LambdaDemo {

    public static void main(String[] args) {
        DiscountCalculator tenPercentDiscount = amount -> amount * 0.10;

        System.out.println(tenPercentDiscount.calculate(1000)); // 100.0
    }
}
```

### Interview phrase

> Lambdas help write concise behavior-passing code, especially with streams and callbacks.

---

# 13. OOP Concepts

## Q13. Explain OOP principles.

### Strong answer

The four main OOP principles are:

```text
1. Encapsulation
2. Abstraction
3. Inheritance
4. Polymorphism
```

### Encapsulation

Hide internal state and expose controlled methods.

```java
public class BankAccount {

    private double balance;

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }

        balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

### Abstraction

Expose what an object does, hide how it does it.

```java
public interface PaymentProcessor {
    void process(double amount);
}
```

```java
public class CreditCardPaymentProcessor implements PaymentProcessor {

    @Override
    public void process(double amount) {
        System.out.println("Processing credit card payment: " + amount);
    }
}
```

### Inheritance

A class can inherit behavior from another class.

```java
public class Animal {
    public void eat() {
        System.out.println("Eating");
    }
}

public class Dog extends Animal {
    public void bark() {
        System.out.println("Barking");
    }
}
```

### Polymorphism

Same interface, different implementations.

```java
public class PaymentService {

    private final PaymentProcessor paymentProcessor;

    public PaymentService(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }

    public void pay(double amount) {
        paymentProcessor.process(amount);
    }
}
```

### Senior-level phrase

> In Spring Boot, polymorphism is commonly used with interfaces and dependency injection. For example, different payment providers can implement the same interface, and Spring can inject the required implementation.

---

# 14. Interface vs Abstract Class

## Q14. Difference between interface and abstract class?

### Answer

| Interface | Abstract Class |
|---|---|
| Defines contract | Defines base behavior |
| A class can implement multiple interfaces | A class can extend only one class |
| Good for capability | Good for shared state/logic |
| Supports default methods | Can have constructors and instance fields |

### Interface example

```java
public interface NotificationSender {
    void send(String message);
}
```

```java
public class EmailNotificationSender implements NotificationSender {

    @Override
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}
```

### Abstract class example

```java
public abstract class BaseReportGenerator {

    public void generateReport() {
        validate();
        generateContent();
        export();
    }

    protected void validate() {
        System.out.println("Common validation");
    }

    protected abstract void generateContent();

    protected void export() {
        System.out.println("Exporting report");
    }
}
```

### Senior-level phrase

> I use interfaces for service contracts and abstract classes when multiple implementations share common behavior or template logic.

---

# 15. SOLID Principles

## Q15. Explain SOLID principles with examples.

### 1. Single Responsibility Principle

A class should have one reason to change.

Bad:

```java
public class UserService {

    public void createUser() {
        // create user
    }

    public void sendEmail() {
        // send email
    }

    public void generateReport() {
        // generate report
    }
}
```

Better:

```java
public class UserService {
    public void createUser() {
        System.out.println("Creating user");
    }
}

public class EmailService {
    public void sendEmail() {
        System.out.println("Sending email");
    }
}

public class ReportService {
    public void generateReport() {
        System.out.println("Generating report");
    }
}
```

### 2. Open/Closed Principle

Open for extension, closed for modification.

Bad:

```java
public class DiscountService {

    public double calculateDiscount(String type, double amount) {
        if (type.equals("REGULAR")) {
            return amount * 0.05;
        }

        if (type.equals("PREMIUM")) {
            return amount * 0.10;
        }

        return 0;
    }
}
```

Better with strategy pattern:

```java
public interface DiscountStrategy {
    double calculate(double amount);
}
```

```java
public class RegularDiscountStrategy implements DiscountStrategy {

    @Override
    public double calculate(double amount) {
        return amount * 0.05;
    }
}
```

```java
public class PremiumDiscountStrategy implements DiscountStrategy {

    @Override
    public double calculate(double amount) {
        return amount * 0.10;
    }
}
```

```java
public class DiscountService {

    private final DiscountStrategy discountStrategy;

    public DiscountService(DiscountStrategy discountStrategy) {
        this.discountStrategy = discountStrategy;
    }

    public double calculateDiscount(double amount) {
        return discountStrategy.calculate(amount);
    }
}
```

### Senior-level phrase

> SOLID matters because it keeps code extensible, testable, and maintainable as business rules evolve.

---

# 16. Java Memory: Stack vs Heap

## Q16. Difference between stack and heap memory?

### Strong answer

Stack memory stores method calls, local variables, and references. Each thread has its own stack.

Heap memory stores objects. Heap is shared across threads and managed by garbage collection.

### Example

```java
public class MemoryDemo {

    public static void main(String[] args) {
        int age = 30;

        User user = new User("Paras");
    }
}

class User {
    private String name;

    public User(String name) {
        this.name = name;
    }
}
```

Memory explanation:

```text
age               → stored on stack
user reference    → stored on stack
new User("Paras") → object stored on heap
```

### Senior-level phrase

> In production, memory issues usually show up as high heap usage, frequent GC, memory leaks through unintended object retention, or excessive object creation.

---

# 17. Garbage Collection

## Q17. What is garbage collection in Java?

### Strong answer

Garbage collection is the process by which JVM automatically reclaims memory from objects that are no longer reachable.

An object becomes eligible for garbage collection when there are no live references to it.

### Example

```java
public class GCDemo {

    public static void main(String[] args) {
        User user = new User("Paras");

        user = null;

        // Now the User object may be eligible for garbage collection.
    }
}
```

### Interview follow-up

#### Can we force garbage collection?

No. We can request it using:

```java
System.gc();
```

But JVM decides whether and when to actually run GC.

### Senior-level phrase

> I usually don’t manually trigger GC. Instead, I analyze heap dumps, GC logs, object allocation patterns, and memory retention if there is a memory issue.

---

# 18. Multithreading Basics

## Q18. What is a thread?

### Strong answer

A thread is a lightweight unit of execution. Multiple threads can run concurrently within the same process.

In backend systems, threads are used to handle concurrent requests, async processing, scheduled jobs, and parallel operations.

### Creating thread directly

```java
public class ThreadDemo {

    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            System.out.println("Running in thread: " + Thread.currentThread().getName());
        });

        thread.start();
    }
}
```

### But better: use ExecutorService

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ExecutorDemo {

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(5);

        executorService.submit(() -> {
            System.out.println("Task running in: " + Thread.currentThread().getName());
        });

        executorService.shutdown();
    }
}
```

### Senior-level phrase

> In production code, I prefer using managed executors/thread pools instead of creating raw threads manually.

---

# 19. Race Condition

## Q19. What is a race condition?

### Strong answer

A race condition occurs when multiple threads access shared mutable data and the final result depends on timing or execution order.

### Bad example

```java
public class Counter {

    private int count = 0;

    public void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}
```

`count++` is not atomic. It has three steps:

```text
read count
increase value
write back
```

Multiple threads can interfere.

### Fixed using synchronized

```java
public class SafeCounter {

    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}
```

### Fixed using AtomicInteger

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicCounter {

    private final AtomicInteger count = new AtomicInteger(0);

    public void increment() {
        count.incrementAndGet();
    }

    public int getCount() {
        return count.get();
    }
}
```

### Senior-level phrase

> For simple counters, I prefer `AtomicInteger` or `LongAdder`. For larger critical sections involving multiple operations, synchronization or locks may be required.

---

# 20. `synchronized` vs Lock

## Q20. Difference between `synchronized` and `ReentrantLock`?

### Strong answer

`synchronized` is built into Java and automatically releases the lock when the block exits.

`ReentrantLock` provides more control, such as try-lock, timed lock, interruptible lock, and fairness options. But it must be unlocked manually, usually in a `finally` block.

### synchronized example

```java
public class SynchronizedCounter {

    private int count;

    public synchronized void increment() {
        count++;
    }
}
```

### ReentrantLock example

```java
import java.util.concurrent.locks.ReentrantLock;

public class LockCounter {

    private int count;
    private final ReentrantLock lock = new ReentrantLock();

    public void increment() {
        lock.lock();

        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
}
```

### Senior-level phrase

> I use `synchronized` for simple locking. I use `ReentrantLock` when I need advanced lock control like timeout or try-lock behavior.

---

# 21. CompletableFuture

## Q21. How do you call multiple APIs in parallel in Java?

### Strong answer

If calls are independent, I can use `CompletableFuture` to execute them asynchronously and combine results. In production, I use a controlled executor, timeouts, and exception handling.

### Code example

```java
import java.util.concurrent.*;

public class DashboardService {

    private final ExecutorService executor = Executors.newFixedThreadPool(10);

    private final UserClient userClient = new UserClient();
    private final OrderClient orderClient = new OrderClient();

    public DashboardResponse getDashboard(Long userId) {
        CompletableFuture<UserProfile> profileFuture =
                CompletableFuture.supplyAsync(() -> userClient.getUserProfile(userId), executor);

        CompletableFuture<OrderSummary> orderFuture =
                CompletableFuture.supplyAsync(() -> orderClient.getOrderSummary(userId), executor);

        CompletableFuture.allOf(profileFuture, orderFuture).join();

        UserProfile profile = profileFuture.join();
        OrderSummary orders = orderFuture.join();

        return new DashboardResponse(profile, orders);
    }
}

class UserClient {
    UserProfile getUserProfile(Long userId) {
        return new UserProfile(userId, "Paras");
    }
}

class OrderClient {
    OrderSummary getOrderSummary(Long userId) {
        return new OrderSummary(5);
    }
}

record UserProfile(Long id, String name) {
}

record OrderSummary(int totalOrders) {
}

record DashboardResponse(UserProfile profile, OrderSummary orderSummary) {
}
```

### Better with timeout and fallback

```java
CompletableFuture<UserProfile> profileFuture =
        CompletableFuture.supplyAsync(() -> userClient.getUserProfile(userId), executor)
                .orTimeout(2, TimeUnit.SECONDS)
                .exceptionally(ex -> new UserProfile(userId, "Unknown"));
```

### Senior-level phrase

> I avoid using async blindly. I use it when operations are independent and latency-sensitive. I also make sure to configure thread pool size, timeouts, fallbacks, and monitoring.

---

# 22. Fail-fast vs Fail-safe Iterators

## Q22. What is fail-fast behavior in Java collections?

### Strong answer

Fail-fast iterators throw `ConcurrentModificationException` if a collection is structurally modified while iterating, except through the iterator’s own remove method.

### Example

```java
import java.util.ArrayList;
import java.util.List;

public class FailFastDemo {

    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("Paras");
        names.add("Amit");

        for (String name : names) {
            if (name.equals("Paras")) {
                names.remove(name); // May throw ConcurrentModificationException
            }
        }
    }
}
```

### Correct way

```java
import java.util.Iterator;
import java.util.List;
import java.util.ArrayList;

public class IteratorRemoveDemo {

    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("Paras");
        names.add("Amit");

        Iterator<String> iterator = names.iterator();

        while (iterator.hasNext()) {
            String name = iterator.next();

            if (name.equals("Paras")) {
                iterator.remove();
            }
        }

        System.out.println(names);
    }
}
```

### Senior-level phrase

> For concurrent modification scenarios, I consider concurrent collections or copy-on-write collections depending on read/write patterns.

---

# 23. Design Patterns They May Ask

## Q23. Which design patterns have you used?

For this role, prepare these:

```text
1. Singleton
2. Factory
3. Strategy
4. Builder
5. Observer/Event-driven
6. Template Method
```

---

## Strategy Pattern

### When to use

When you have multiple algorithms/behaviors and want to switch them without if-else chains.

### Example

```java
public interface PaymentStrategy {
    void pay(double amount);
}
```

```java
public class CreditCardPaymentStrategy implements PaymentStrategy {

    @Override
    public void pay(double amount) {
        System.out.println("Paid by credit card: " + amount);
    }
}
```

```java
public class UpiPaymentStrategy implements PaymentStrategy {

    @Override
    public void pay(double amount) {
        System.out.println("Paid by UPI: " + amount);
    }
}
```

```java
public class PaymentService {

    private final PaymentStrategy paymentStrategy;

    public PaymentService(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void makePayment(double amount) {
        paymentStrategy.pay(amount);
    }
}
```

### Interview phrase

> I use Strategy when business rules vary by type, for example different payment methods, discount rules, notification channels, or pricing logic.

---

## Factory Pattern

### When to use

When object creation logic is complex or depends on type.

```java
public interface NotificationSender {
    void send(String message);
}
```

```java
public class EmailSender implements NotificationSender {

    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}
```

```java
public class SmsSender implements NotificationSender {

    @Override
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}
```

```java
public class NotificationSenderFactory {

    public NotificationSender getSender(String type) {
        if ("EMAIL".equalsIgnoreCase(type)) {
            return new EmailSender();
        }

        if ("SMS".equalsIgnoreCase(type)) {
            return new SmsSender();
        }

        throw new IllegalArgumentException("Unsupported notification type: " + type);
    }
}
```

### Interview phrase

> Factory helps centralize object creation and keeps calling code independent of concrete classes.

---

## Builder Pattern

### When to use

When an object has many optional fields.

```java
public class SearchRequest {

    private final String query;
    private final Integer page;
    private final Integer size;
    private final String sortBy;

    private SearchRequest(Builder builder) {
        this.query = builder.query;
        this.page = builder.page;
        this.size = builder.size;
        this.sortBy = builder.sortBy;
    }

    public static class Builder {
        private String query;
        private Integer page = 0;
        private Integer size = 20;
        private String sortBy = "createdDate";

        public Builder query(String query) {
            this.query = query;
            return this;
        }

        public Builder page(Integer page) {
            this.page = page;
            return this;
        }

        public Builder size(Integer size) {
            this.size = size;
            return this;
        }

        public Builder sortBy(String sortBy) {
            this.sortBy = sortBy;
            return this;
        }

        public SearchRequest build() {
            return new SearchRequest(this);
        }
    }
}
```

Usage:

```java
SearchRequest request = new SearchRequest.Builder()
        .query("gartner")
        .page(0)
        .size(10)
        .sortBy("publishedDate")
        .build();
```

---

# 24. Most likely Core Java rapid-fire questions

Practice these aloud.

## Collections

```text
1. How does HashMap work internally?
2. Why do we override equals and hashCode?
3. HashMap vs ConcurrentHashMap?
4. ArrayList vs LinkedList?
5. HashSet vs TreeSet?
6. Fail-fast vs fail-safe?
```

## OOP

```text
1. Interface vs abstract class?
2. Overloading vs overriding?
3. Runtime polymorphism?
4. SOLID principles?
5. Design patterns used?
```

## Java 8+

```text
1. Streams?
2. Lambda?
3. Functional interface?
4. Optional?
5. map vs flatMap?
```

## Exceptions

```text
1. Checked vs unchecked?
2. How do you create custom exception?
3. How do you handle exceptions in Spring Boot?
```

## Concurrency

```text
1. Thread vs process?
2. synchronized?
3. Race condition?
4. ExecutorService?
5. CompletableFuture?
6. AtomicInteger?
7. volatile?
```

## Memory

```text
1. Stack vs heap?
2. Garbage collection?
3. Memory leak in Java?
4. How to debug memory issue?
```

---

# 25. Your Core Java answer formula

For every Java question, answer in this structure:

```text
1. Definition
2. Simple example
3. Production implication
4. Tradeoff / pitfall
```

Example:

> “`HashMap` stores key-value pairs using hashing. It calculates the hash of the key and maps it to a bucket. For custom keys, `equals()` and `hashCode()` must be implemented correctly. In production, I avoid mutable keys because changing a key after insertion can break lookup.”

This is exactly the style you need for a Sr Architect.

---

# Where we go next

Next module should be:

# Module 2B: Spring Boot vetted interview questions

We’ll cover:

```text
1. What happens when a Spring Boot app starts?
2. @SpringBootApplication internals
3. Dependency injection
4. Bean lifecycle
5. @Component vs @Service vs @Repository
6. Controller vs RestController
7. Request lifecycle
8. Validation
9. Global exception handling
10. @Transactional
11. Spring Data JPA
12. Lazy vs eager loading
13. N+1 problem
14. Profiles
15. Actuator
16. Security basics
17. Testing
18. Production-ready Spring Boot service
```

And this time, it will be **actual interview Q&A with code and follow-ups**, not just a basic intro.
