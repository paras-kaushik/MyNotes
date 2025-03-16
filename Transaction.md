### Understanding `@Transactional` in Spring Boot

The `@Transactional` annotation in Spring Boot is used to manage transaction boundaries in a declarative way. It ensures that a series of operations are executed within a transaction context, providing atomicity, consistency, isolation, and durability (ACID properties).

#### Key Functionalities

1. **Atomicity**: Ensures that all operations within the transaction are completed successfully. If any operation fails, the transaction is rolled back.

2. **Consistency**: Ensures that the database remains in a consistent state before and after the transaction.

3. **Isolation**: Controls the visibility of transaction changes to other transactions.

4. **Durability**: Ensures that once a transaction is committed, it remains so, even in the event of a system failure.

#### Usage of `@Transactional`

- **Propagation**: Defines how transactions relate to each other. Common options include:
  - `REQUIRED`: Joins an existing transaction or creates a new one if none exists.
  - `REQUIRES_NEW`: Suspends the current transaction and creates a new one.
  - `MANDATORY`: Must run within an existing transaction.
  - `SUPPORTS`: Runs within a transaction if one exists.

- **Isolation Levels**: Controls the visibility of changes made by one transaction to other transactions. Options include:
  - `READ_UNCOMMITTED`
  - `READ_COMMITTED`
  - `REPEATABLE_READ`
  - `SERIALIZABLE`

- **Timeout**: Specifies the maximum time a transaction can run before being rolled back.

- **Read-Only**: Optimizes performance for transactions that only read data.

#### Code Examples

**Without `@Transactional`:**

```java
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public void updateUser(User user) {
        userRepository.save(user);
        // Additional operations
    }
}
```

**With `@Transactional`:**

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Transactional
    public void updateUser(User user) {
        userRepository.save(user);
        // Additional operations
    }
}
```

#### Practical Applications and Use-Cases

1. **Batch Processing**: Ensures that all records in a batch are processed successfully. If one fails, the entire batch is rolled back.

2. **Financial Transactions**: Guarantees that all steps in a financial transaction (e.g., debit and credit) are completed successfully.

3. **Data Integrity**: Maintains data integrity by ensuring that complex operations involving multiple database updates are completed atomically.

4. **Error Handling**: Simplifies error handling by automatically rolling back transactions in case of exceptions.

#### Advanced Example

**Using Propagation and Isolation:**

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@Service
public class OrderService {
    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    @Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.SERIALIZABLE, timeout = 5)
    public void placeOrder(Order order) {
        orderRepository.save(order);
        // Additional operations
    }
}
```

### Practice Exercise

- **Task:** Implement a `BankService` with methods for `transferFunds` between accounts. Use `@Transactional` to ensure atomicity.

- **Steps:**
  1. Create `Account` and `Transaction` entities.
  2. Implement `transferFunds` to debit one account and credit another.
  3. Use `@Transactional` to ensure the entire operation is atomic.

By using `@Transactional`, you can manage complex transactions easily, ensuring data integrity and consistency across your Spring Boot applications.
