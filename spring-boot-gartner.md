Excellent call. For your interview, **project-building is not the highest ROI right now**.

We’ll do **Spring Boot through interview questions**, exactly like this:

```text
Question
What interviewer is really testing
Strong answer to say
Code pattern
Follow-ups/traps
```

This is the right prep for a **Sr Architect** round.

---

# Spring Boot + Backend Interview Questions — High-Value Module

## Problem budget for this module

For your interview, master these **15 questions** first:

1. How does a Spring Boot request flow work?
2. How do you handle exceptions globally?
3. How do you validate API requests?
4. What is `@Transactional`?
5. What happens if we don’t use `@Transactional`?
6. What are common `@Transactional` pitfalls?
7. How would you optimize a slow API?
8. What is the N+1 query problem and how do you fix it?
9. DTO vs Entity — why not expose entities directly?
10. How do you design REST APIs?
11. Controller vs Service vs Repository?
12. How do you secure APIs?
13. How do you implement caching?
14. How do you handle idempotency?
15. How do you make a Spring Boot service production-ready?

If you can answer these well, you’ll cover a huge part of the backend interview.

---

# 1. How does a Spring Boot request flow work?

## Interview question

> What happens when an HTTP request hits a Spring Boot REST API?

---

## What interviewer is testing

They want to know if you understand backend layering:

```text
Client → Controller → Service → Repository → Database → Response
```

They may also test whether you know:

- `DispatcherServlet`
- request mapping
- JSON deserialization
- validation
- business logic
- persistence
- response serialization

---

## Strong answer

Say:

> “When a request reaches a Spring Boot application, it first goes through Spring MVC’s dispatcher layer. Spring finds the matching controller method using annotations like `@GetMapping` or `@PostMapping`. Request parameters, path variables, and JSON request bodies are converted into Java objects. Validation can run using `@Valid`. The controller delegates business logic to the service layer. The service may call repositories for database operations. Finally, the response object is serialized to JSON and returned with the appropriate HTTP status code.”

---

## Typical request flow

```text
HTTP Request
   ↓
Filter chain / Security filters
   ↓
DispatcherServlet
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
   ↓
Response DTO
   ↓
JSON Response
```

---

## Code example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    // Constructor injection
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public UserResponse getUserById(@PathVariable Long id) {
        return userService.getUserById(id);
    }
}
```

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public UserResponse getUserById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new UserNotFoundException(id));

        return new UserResponse(
                user.getId(),
                user.getName(),
                user.getEmail()
        );
    }
}
```

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
}
```

---

## Follow-up trap

### Bad answer

> “Controller calls database and returns response.”

This sounds junior.

### Better answer

> “Controller should stay thin and delegate business logic to service. Repository handles persistence. This separation improves testability, maintainability, and clear ownership.”

---

# 2. How do you handle exceptions globally?

## Interview question

> How will you handle exceptions in Spring Boot? Globally?

This is very likely.

---

## What interviewer is testing

They want to know if you can build clean APIs where errors are:

- consistent
- meaningful
- not leaking internal stack traces
- useful for frontend
- useful for production debugging

---

## Strong answer

Say:

> “I avoid handling exceptions repeatedly in every controller. I define custom exceptions for business cases and handle them centrally using `@RestControllerAdvice` and `@ExceptionHandler`. I return a consistent error response containing timestamp, status, error code, message, path, and optionally correlation ID. This keeps controllers clean and gives API consumers predictable error responses.”

---

## Bad style

Do not do this in every controller:

```java
@GetMapping("/{id}")
public ResponseEntity<?> getUser(@PathVariable Long id) {
    try {
        UserResponse user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    } catch (UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body("User not found");
    } catch (Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body("Something went wrong");
    }
}
```

Why bad?

- Repeated code.
- Inconsistent errors.
- Controllers become messy.
- Hard to maintain.

---

## Good style: custom exception

```java
public class UserNotFoundException extends RuntimeException {

    public UserNotFoundException(Long userId) {
        super("User not found with id: " + userId);
    }
}
```

---

## Error response DTO

You can use a class:

```java
import java.time.Instant;
import java.util.Map;

public class ApiErrorResponse {

    private Instant timestamp;
    private int status;
    private String errorCode;
    private String message;
    private String path;
    private Map<String, String> fieldErrors;

    public ApiErrorResponse(
            int status,
            String errorCode,
            String message,
            String path,
            Map<String, String> fieldErrors
    ) {
        this.timestamp = Instant.now();
        this.status = status;
        this.errorCode = errorCode;
        this.message = message;
        this.path = path;
        this.fieldErrors = fieldErrors;
    }

    public Instant getTimestamp() {
        return timestamp;
    }

    public int getStatus() {
        return status;
    }

    public String getErrorCode() {
        return errorCode;
    }

    public String getMessage() {
        return message;
    }

    public String getPath() {
        return path;
    }

    public Map<String, String> getFieldErrors() {
        return fieldErrors;
    }
}
```

Or in modern Java, you can use a record:

```java
import java.time.Instant;
import java.util.Map;

public record ApiErrorResponse(
        Instant timestamp,
        int status,
        String errorCode,
        String message,
        String path,
        Map<String, String> fieldErrors
) {
}
```

---

## Global exception handler

```java
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.*;

import java.time.Instant;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ApiErrorResponse handleUserNotFound(
            UserNotFoundException ex,
            HttpServletRequest request
    ) {
        return new ApiErrorResponse(
                Instant.now(),
                HttpStatus.NOT_FOUND.value(),
                "USER_NOT_FOUND",
                ex.getMessage(),
                request.getRequestURI(),
                null
        );
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiErrorResponse handleValidationException(
            MethodArgumentNotValidException ex,
            HttpServletRequest request
    ) {
        Map<String, String> fieldErrors = new HashMap<>();

        ex.getBindingResult().getFieldErrors().forEach(error -> {
            fieldErrors.put(error.getField(), error.getDefaultMessage());
        });

        return new ApiErrorResponse(
                Instant.now(),
                HttpStatus.BAD_REQUEST.value(),
                "VALIDATION_ERROR",
                "Request validation failed",
                request.getRequestURI(),
                fieldErrors
        );
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ApiErrorResponse handleGenericException(
            Exception ex,
            HttpServletRequest request
    ) {
        return new ApiErrorResponse(
                Instant.now(),
                HttpStatus.INTERNAL_SERVER_ERROR.value(),
                "INTERNAL_SERVER_ERROR",
                "Something went wrong. Please contact support.",
                request.getRequestURI(),
                null
        );
    }
}
```

---

## Example response

```json
{
  "timestamp": "2026-05-20T10:15:30Z",
  "status": 404,
  "errorCode": "USER_NOT_FOUND",
  "message": "User not found with id: 10",
  "path": "/api/users/10",
  "fieldErrors": null
}
```

Validation error:

```json
{
  "timestamp": "2026-05-20T10:15:30Z",
  "status": 400,
  "errorCode": "VALIDATION_ERROR",
  "message": "Request validation failed",
  "path": "/api/users",
  "fieldErrors": {
    "email": "Email must be valid",
    "name": "Name is required"
  }
}
```

---

## Follow-up questions

### Q: Why not expose raw exception messages?

Because raw messages can leak:

- table names
- SQL details
- internal class names
- security-sensitive implementation details

Better:

```text
Log detailed error internally.
Return safe error response externally.
```

---

### Q: What status code would you use?

| Scenario | HTTP Status |
|---|---:|
| Invalid request body | `400 Bad Request` |
| Unauthorized/no token | `401 Unauthorized` |
| Authenticated but no permission | `403 Forbidden` |
| Resource not found | `404 Not Found` |
| Duplicate resource | `409 Conflict` |
| Validation error | `400 Bad Request` |
| Server error | `500 Internal Server Error` |

---

# 3. How do you validate API requests?

## Interview question

> How do you validate incoming request payloads in Spring Boot?

---

## Strong answer

Say:

> “I use request DTOs with Bean Validation annotations like `@NotBlank`, `@Email`, `@Size`, `@Min`, and `@NotNull`. In the controller, I use `@Valid` so Spring validates the request before it reaches service logic. For business validations like duplicate email or entitlement checks, I handle them in the service layer.”

---

## Request DTO

```java
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public class CreateUserRequest {

    @NotBlank(message = "Name is required")
    private String name;

    @Email(message = "Email must be valid")
    @NotBlank(message = "Email is required")
    private String email;

    @Size(min = 8, max = 50, message = "Password must be between 8 and 50 characters")
    private String password;

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    public String getPassword() {
        return password;
    }
}
```

---

## Controller

```java
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public UserResponse createUser(@Valid @RequestBody CreateUserRequest request) {
    return userService.createUser(request);
}
```

---

## Service-level business validation

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public UserResponse createUser(CreateUserRequest request) {
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new DuplicateEmailException(request.getEmail());
        }

        User user = new User(
                request.getName(),
                request.getEmail()
        );

        User savedUser = userRepository.save(user);

        return new UserResponse(
                savedUser.getId(),
                savedUser.getName(),
                savedUser.getEmail()
        );
    }
}
```

---

## Key distinction

| Validation type | Where |
|---|---|
| Required field | DTO annotation |
| Email format | DTO annotation |
| Length constraints | DTO annotation |
| Duplicate email | Service |
| User permission | Service/security |
| Subscription entitlement | Service/domain logic |

---

## Follow-up trap

Do not say:

> “I validate everything in controller.”

Better:

> “I validate request shape in DTO/controller and business rules in service layer.”

---

# 4. What is `@Transactional`?

## Interview question

> What is `@Transactional` in Spring Boot?

This is extremely likely.

---

## Strong answer

Say:

> “`@Transactional` defines a transaction boundary. All database operations inside that method participate in the same transaction. If the method completes successfully, changes are committed. If a runtime exception occurs, changes are rolled back by default. I usually place `@Transactional` at the service layer because service methods represent business use cases.”

---

## Simple example: money transfer

Without transaction safety, money transfer can corrupt data.

Business operation:

```text
Debit account A
Credit account B
```

Both should succeed or both should fail.

---

## Entity

```java
import jakarta.persistence.*;
import java.math.BigDecimal;

@Entity
public class BankAccount {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private BigDecimal balance;

    protected BankAccount() {
    }

    public BankAccount(BigDecimal balance) {
        this.balance = balance;
    }

    public Long getId() {
        return id;
    }

    public BigDecimal getBalance() {
        return balance;
    }

    public void debit(BigDecimal amount) {
        if (balance.compareTo(amount) < 0) {
            throw new IllegalArgumentException("Insufficient balance");
        }

        this.balance = this.balance.subtract(amount);
    }

    public void credit(BigDecimal amount) {
        this.balance = this.balance.add(amount);
    }
}
```

---

## Repository

```java
public interface BankAccountRepository extends JpaRepository<BankAccount, Long> {
}
```

---

## Service with transaction

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;

@Service
public class TransferService {

    private final BankAccountRepository bankAccountRepository;

    public TransferService(BankAccountRepository bankAccountRepository) {
        this.bankAccountRepository = bankAccountRepository;
    }

    @Transactional
    public void transfer(Long fromAccountId, Long toAccountId, BigDecimal amount) {
        BankAccount fromAccount = bankAccountRepository.findById(fromAccountId)
                .orElseThrow(() -> new RuntimeException("From account not found"));

        BankAccount toAccount = bankAccountRepository.findById(toAccountId)
                .orElseThrow(() -> new RuntimeException("To account not found"));

        fromAccount.debit(amount);

        // Suppose something fails after debit
        // if (true) throw new RuntimeException("Unexpected failure");

        toAccount.credit(amount);
    }
}
```

---

## What happens here?

If everything succeeds:

```text
Debit fromAccount
Credit toAccount
Commit transaction
```

If runtime exception occurs after debit:

```text
Debit happened in memory/persistence context
Exception thrown
Transaction rollback
Debit not committed
Credit not committed
```

---

# 5. What happens if we don’t use `@Transactional`?

## Interview question

> What will happen if we don’t use `@Transactional`?

---

## Strong answer

Say:

> “Without `@Transactional`, each repository operation may execute in its own transaction depending on repository defaults. For multi-step business operations, partial updates can be committed if a later step fails. That can leave the system in an inconsistent state. `@Transactional` ensures atomicity across the whole business operation.”

---

## Example without transaction

```java
@Service
public class TransferServiceWithoutTransaction {

    private final BankAccountRepository bankAccountRepository;

    public TransferServiceWithoutTransaction(BankAccountRepository bankAccountRepository) {
        this.bankAccountRepository = bankAccountRepository;
    }

    public void transfer(Long fromAccountId, Long toAccountId, BigDecimal amount) {
        BankAccount fromAccount = bankAccountRepository.findById(fromAccountId)
                .orElseThrow(() -> new RuntimeException("From account not found"));

        fromAccount.debit(amount);

        bankAccountRepository.save(fromAccount);

        // Failure happens here
        if (true) {
            throw new RuntimeException("Network failure");
        }

        BankAccount toAccount = bankAccountRepository.findById(toAccountId)
                .orElseThrow(() -> new RuntimeException("To account not found"));

        toAccount.credit(amount);

        bankAccountRepository.save(toAccount);
    }
}
```

Possible issue:

```text
fromAccount debit committed
failure occurs
toAccount credit never happens
money disappears
```

---

## Interview phrase

Use this:

> “The main risk is partial commit. `@Transactional` protects consistency by making the operation all-or-nothing.”

---

# 6. Common `@Transactional` pitfalls

## Pitfall 1: Rollback happens by default only for unchecked exceptions

By default, Spring rolls back for:

```text
RuntimeException
Error
```

But not necessarily for checked exceptions.

---

## Example

```java
@Transactional
public void importUsers() throws IOException {
    userRepository.save(new User("Paras", "paras@test.com"));

    throw new IOException("File read failed");
}
```

This may not roll back by default because `IOException` is checked.

---

## Fix

```java
@Transactional(rollbackFor = IOException.class)
public void importUsers() throws IOException {
    userRepository.save(new User("Paras", "paras@test.com"));

    throw new IOException("File read failed");
}
```

---

## Pitfall 2: Self-invocation problem

Bad:

```java
@Service
public class UserService {

    public void outerMethod() {
        innerTransactionalMethod();
    }

    @Transactional
    public void innerTransactionalMethod() {
        // database operations
    }
}
```

Why bad?

Spring transactions are usually applied through proxies.

When one method inside the same class calls another method directly, the call may bypass the proxy.

So `@Transactional` may not apply.

---

## Fix

Move transactional method to another Spring bean:

```java
@Service
public class UserRegistrationService {

    private final UserTransactionService userTransactionService;

    public UserRegistrationService(UserTransactionService userTransactionService) {
        this.userTransactionService = userTransactionService;
    }

    public void registerUser() {
        userTransactionService.createUserInTransaction();
    }
}
```

```java
@Service
public class UserTransactionService {

    @Transactional
    public void createUserInTransaction() {
        // database operations
    }
}
```

---

## Pitfall 3: Calling external APIs inside a transaction

Bad:

```java
@Transactional
public void placeOrder(CreateOrderRequest request) {
    Order order = orderRepository.save(new Order(request.getUserId()));

    paymentClient.charge(request.getPaymentDetails());

    inventoryRepository.reduceStock(request.getProductId());
}
```

Problem:

- Transaction stays open while external API runs.
- DB locks may be held longer.
- External API may be slow.
- External API may succeed but DB may later roll back.
- Hard to maintain consistency.

---

## Better approach

```java
@Transactional
public Order createOrder(CreateOrderRequest request) {
    Order order = orderRepository.save(new Order(request.getUserId()));

    inventoryRepository.reserveStock(request.getProductId());

    return order;
}

// Outside transaction or via async event:
public void processPayment(Long orderId) {
    paymentClient.charge(orderId);
}
```

Or use:

```text
Outbox pattern
Saga pattern
State machine
Async event processing
```

For high-scale systems.

---

## Pitfall 4: `readOnly = true`

```java
@Transactional(readOnly = true)
public UserResponse getUserById(Long id) {
    User user = userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));

    return new UserResponse(user.getId(), user.getName(), user.getEmail());
}
```

Why use it?

- Indicates intent.
- Can help ORM optimize dirty checking.
- Prevents accidental writes depending on setup.

---

## Interview answer for pitfalls

Say:

> “Important pitfalls are rollback behavior for checked exceptions, proxy-based self-invocation, long transactions during external calls, transaction scope being too broad, and misunderstanding `readOnly`. I keep transactions at service-layer business operation boundaries and keep them as short as possible.”

---

# 7. How would you optimize a slow API?

## Interview question

> This API is slow in production. How will you optimize it?

This is very likely for a Lead/Sr role.

---

## Strong answer

Say:

> “I would not randomly optimize. First, I would measure where the latency is coming from: frontend, network, application code, database, cache, or downstream services. I would check logs, metrics, traces, DB query time, thread pool usage, connection pool usage, and recent deployments. Once bottleneck is identified, common fixes include pagination, indexing, query optimization, avoiding N+1 queries, reducing payload size, caching, async processing, and adding timeouts.”

---

## Framework to answer

Use this structure:

```text
1. Measure
2. Identify bottleneck
3. Apply targeted fix
4. Add monitoring
5. Prevent recurrence
```

---

## Case 1: API returns too much data

Bad endpoint:

```java
@GetMapping("/users")
public List<UserResponse> getAllUsers() {
    return userService.getAllUsers();
}
```

Problem:

```text
Returns unbounded data.
Can become slow as table grows.
```

---

## Fix: pagination

```java
@GetMapping("/users")
public Page<UserResponse> getUsers(Pageable pageable) {
    return userService.getUsers(pageable);
}
```

Service:

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Transactional(readOnly = true)
    public Page<UserResponse> getUsers(Pageable pageable) {
        return userRepository.findAll(pageable)
                .map(user -> new UserResponse(
                        user.getId(),
                        user.getName(),
                        user.getEmail()
                ));
    }
}
```

Request:

```text
GET /api/users?page=0&size=20&sort=name,asc
```

---

## Case 2: API fetches full entity when only few fields are needed

Bad:

```java
List<User> users = userRepository.findAll();
```

Maybe entity has:

- profile
- orders
- permissions
- preferences
- audit logs

But UI only needs:

```text
id, name, email
```

---

## Fix: DTO projection

DTO:

```java
public class UserSummaryResponse {

    private Long id;
    private String name;
    private String email;

    public UserSummaryResponse(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }
}
```

Repository:

```java
public interface UserRepository extends JpaRepository<User, Long> {

    @Query("""
           select new com.example.UserSummaryResponse(u.id, u.name, u.email)
           from User u
           where (:status is null or u.status = :status)
           """)
    Page<UserSummaryResponse> findUserSummaries(
            @Param("status") String status,
            Pageable pageable
    );
}
```

Service:

```java
@Transactional(readOnly = true)
public Page<UserSummaryResponse> searchUsers(String status, Pageable pageable) {
    return userRepository.findUserSummaries(status, pageable);
}
```

Benefit:

```text
Only fetch required columns.
Less DB load.
Less memory usage.
Smaller response.
```

---

## Case 3: DB query is slow

Checklist:

```text
Check query plan
Check missing indexes
Check join conditions
Check sorting columns
Check filtering columns
Check pagination strategy
Check large OFFSET
```

Example SQL index:

```sql
CREATE INDEX idx_users_status_created_at
ON users(status, created_at);
```

If API frequently does:

```sql
WHERE status = ?
ORDER BY created_at DESC
```

then composite index can help.

---

## Case 4: N+1 query problem

We’ll cover separately below, but this is one of the biggest slow API causes.

---

## Case 5: Repeated read-heavy data

Use caching.

```java
@Cacheable(value = "users", key = "#id")
@Transactional(readOnly = true)
public UserResponse getUserById(Long id) {
    User user = userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));

    return new UserResponse(user.getId(), user.getName(), user.getEmail());
}
```

When user changes:

```java
@CacheEvict(value = "users", key = "#id")
@Transactional
public UserResponse updateUser(Long id, UpdateUserRequest request) {
    User user = userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));

    user.updateName(request.getName());

    return new UserResponse(user.getId(), user.getName(), user.getEmail());
}
```

---

## Case 6: Downstream API is slow

Bad:

```java
UserProfile profile = profileClient.getProfile(userId);
OrderSummary orders = orderClient.getOrders(userId);
RecommendationSummary recommendations = recommendationClient.getRecommendations(userId);
```

If each call is independent, this is unnecessarily sequential.

---

## Fix: parallel independent calls

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.Executor;

@Service
public class DashboardService {

    private final ProfileClient profileClient;
    private final OrderClient orderClient;
    private final RecommendationClient recommendationClient;
    private final Executor dashboardExecutor;

    public DashboardService(
            ProfileClient profileClient,
            OrderClient orderClient,
            RecommendationClient recommendationClient,
            Executor dashboardExecutor
    ) {
        this.profileClient = profileClient;
        this.orderClient = orderClient;
        this.recommendationClient = recommendationClient;
        this.dashboardExecutor = dashboardExecutor;
    }

    public DashboardResponse getDashboard(Long userId) {
        CompletableFuture<UserProfile> profileFuture =
                CompletableFuture.supplyAsync(
                        () -> profileClient.getProfile(userId),
                        dashboardExecutor
                );

        CompletableFuture<OrderSummary> orderFuture =
                CompletableFuture.supplyAsync(
                        () -> orderClient.getOrders(userId),
                        dashboardExecutor
                );

        CompletableFuture<RecommendationSummary> recommendationFuture =
                CompletableFuture.supplyAsync(
                        () -> recommendationClient.getRecommendations(userId),
                        dashboardExecutor
                );

        CompletableFuture.allOf(
                profileFuture,
                orderFuture,
                recommendationFuture
        ).join();

        return new DashboardResponse(
                profileFuture.join(),
                orderFuture.join(),
                recommendationFuture.join()
        );
    }
}
```

But mention:

> “In production, I would add timeouts, fallbacks, and avoid unbounded thread pools.”

---

## Case 7: Add timing logs

```java
import jakarta.servlet.*;
import jakarta.servlet.http.HttpServletRequest;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
public class RequestTimingFilter implements Filter {

    private static final Logger log = LoggerFactory.getLogger(RequestTimingFilter.class);

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain
    ) throws IOException, ServletException {

        long startTime = System.currentTimeMillis();

        try {
            chain.doFilter(request, response);
        } finally {
            long duration = System.currentTimeMillis() - startTime;

            HttpServletRequest httpRequest = (HttpServletRequest) request;

            log.info(
                    "method={} uri={} durationMs={}",
                    httpRequest.getMethod(),
                    httpRequest.getRequestURI(),
                    duration
            );
        }
    }
}
```

---

## Interview summary answer

Say:

> “For a slow API, I first measure. If DB is slow, I check query plans, indexes, joins, pagination, and N+1. If app layer is slow, I profile CPU/memory and reduce unnecessary mapping or payload size. If downstream services are slow, I add timeouts, retries carefully, circuit breakers, or parallelize independent calls. If data is read-heavy, I consider caching. After fixing, I add metrics and alerts so it doesn’t regress.”

---

# 8. What is the N+1 query problem?

## Interview question

> What is N+1 problem in JPA/Hibernate? How do you fix it?

Very common.

---

## Strong answer

Say:

> “N+1 happens when the application first executes one query to fetch parent records, and then executes one additional query for each parent to fetch child records. For example, one query fetches 100 users, then 100 more queries fetch orders for each user. This causes performance issues. I fix it using fetch joins, entity graphs, batch fetching, or DTO projections depending on use case.”

---

## Entity example

```java
@Entity
public class User {

    @Id
    private Long id;

    private String name;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Order> orders = new ArrayList<>();

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public List<Order> getOrders() {
        return orders;
    }
}
```

```java
@Entity
public class Order {

    @Id
    private Long id;

    private String orderNumber;

    @ManyToOne(fetch = FetchType.LAZY)
    private User user;
}
```

---

## Bad service code

```java
@Transactional(readOnly = true)
public List<UserOrderSummary> getUserOrderSummaries() {
    List<User> users = userRepository.findAll();

    List<UserOrderSummary> summaries = new ArrayList<>();

    for (User user : users) {
        summaries.add(new UserOrderSummary(
                user.getId(),
                user.getName(),
                user.getOrders().size()
        ));
    }

    return summaries;
}
```

What happens?

```text
1 query: select all users
N queries: select orders for each user
```

If 100 users:

```text
1 + 100 = 101 queries
```

---

## Fix 1: `join fetch`

```java
public interface UserRepository extends JpaRepository<User, Long> {

    @Query("""
           select distinct u
           from User u
           left join fetch u.orders
           """)
    List<User> findAllWithOrders();
}
```

Service:

```java
@Transactional(readOnly = true)
public List<UserOrderSummary> getUserOrderSummaries() {
    List<User> users = userRepository.findAllWithOrders();

    return users.stream()
            .map(user -> new UserOrderSummary(
                    user.getId(),
                    user.getName(),
                    user.getOrders().size()
            ))
            .toList();
}
```

---

## Fix 2: `@EntityGraph`

```java
public interface UserRepository extends JpaRepository<User, Long> {

    @EntityGraph(attributePaths = "orders")
    List<User> findByStatus(String status);
}
```

---

## Fix 3: DTO query directly

Often best for read APIs.

```java
public class UserOrderCountResponse {

    private Long userId;
    private String userName;
    private Long orderCount;

    public UserOrderCountResponse(Long userId, String userName, Long orderCount) {
        this.userId = userId;
        this.userName = userName;
        this.orderCount = orderCount;
    }

    public Long getUserId() {
        return userId;
    }

    public String getUserName() {
        return userName;
    }

    public Long getOrderCount() {
        return orderCount;
    }
}
```

Repository:

```java
public interface UserRepository extends JpaRepository<User, Long> {

    @Query("""
           select new com.example.UserOrderCountResponse(
               u.id,
               u.name,
               count(o.id)
           )
           from User u
           left join u.orders o
           group by u.id, u.name
           """)
    List<UserOrderCountResponse> findUserOrderCounts();
}
```

---

## Interview guidance

Say:

> “For API responses, I usually prefer DTO projections because they fetch exactly what the UI needs and avoid loading full entity graphs unnecessarily.”

That sounds senior.

---

# 9. DTO vs Entity — why not expose entities directly?

## Interview question

> Why do we use DTOs? Why not return JPA entities directly from controllers?

---

## Strong answer

Say:

> “Entities represent database structure. DTOs represent API contracts. I avoid exposing entities directly because it can leak sensitive fields, create tight coupling between DB schema and API, cause lazy-loading serialization issues, expose unwanted relationships, and make API evolution harder.”

---

## Bad example

```java
@GetMapping("/{id}")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
}
```

Problems:

- `passwordHash` may leak.
- Lazy relationships may serialize unexpectedly.
- Circular references.
- API changes if entity changes.
- Hard to version.

---

## Entity

```java
@Entity
public class User {

    @Id
    private Long id;

    private String name;

    private String email;

    private String passwordHash;

    private boolean internalFlag;

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    public String getPasswordHash() {
        return passwordHash;
    }

    public boolean isInternalFlag() {
        return internalFlag;
    }
}
```

---

## Response DTO

```java
public class UserResponse {

    private Long id;
    private String name;
    private String email;

    public UserResponse(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }
}
```

---

## Good controller

```java
@GetMapping("/{id}")
public UserResponse getUser(@PathVariable Long id) {
    return userService.getUserById(id);
}
```

---

## Interview line

Say:

> “DTOs allow API contracts to evolve independently from persistence models.”

---

# 10. How do you design REST APIs?

## Interview question

> How do you design REST APIs?

---

## Strong answer

Say:

> “I design APIs around resources. I use proper HTTP methods, meaningful status codes, request/response DTOs, validation, pagination for list APIs, filtering/sorting where required, consistent error responses, authentication/authorization, idempotency for critical operations, and versioning if the API is consumed externally.”

---

## Good API design

```text
GET    /api/users/{id}
POST   /api/users
PUT    /api/users/{id}
PATCH  /api/users/{id}/status
DELETE /api/users/{id}

GET    /api/users?page=0&size=20&sort=name,asc
GET    /api/users?status=ACTIVE&department=Engineering
```

---

## Status codes

| Operation | Status |
|---|---:|
| Create success | `201 Created` |
| Get success | `200 OK` |
| Update success | `200 OK` or `204 No Content` |
| Delete success | `204 No Content` |
| Validation failure | `400 Bad Request` |
| Not found | `404 Not Found` |
| Duplicate/conflict | `409 Conflict` |
| Unauthorized | `401 Unauthorized` |
| Forbidden | `403 Forbidden` |

---

## Controller example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping
    public ResponseEntity<UserResponse> createUser(
            @Valid @RequestBody CreateUserRequest request
    ) {
        UserResponse response = userService.createUser(request);

        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(response);
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }

    @GetMapping
    public ResponseEntity<Page<UserResponse>> searchUsers(
            @RequestParam(required = false) String status,
            Pageable pageable
    ) {
        return ResponseEntity.ok(userService.searchUsers(status, pageable));
    }
}
```

---

## Follow-up: PUT vs PATCH

Say:

> “PUT usually replaces the whole resource. PATCH partially updates the resource.”

Example:

```text
PUT /api/users/10
```

Full update.

```text
PATCH /api/users/10/status
```

Only status update.

---

# 11. Controller vs Service vs Repository

## Interview question

> What logic goes into controller, service, and repository?

---

## Strong answer

Say:

> “Controller handles HTTP concerns: request mapping, status codes, request body, path variables. Service handles business logic, orchestration, validation rules, transaction boundaries. Repository handles database access. Keeping these separate improves maintainability, testing, and code ownership.”

---

## Controller

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(
            @Valid @RequestBody CreateOrderRequest request
    ) {
        OrderResponse response = orderService.createOrder(request);

        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}
```

---

## Service

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;
    private final InventoryRepository inventoryRepository;

    public OrderService(
            OrderRepository orderRepository,
            InventoryRepository inventoryRepository
    ) {
        this.orderRepository = orderRepository;
        this.inventoryRepository = inventoryRepository;
    }

    @Transactional
    public OrderResponse createOrder(CreateOrderRequest request) {
        Inventory inventory = inventoryRepository.findByProductId(request.getProductId())
                .orElseThrow(() -> new RuntimeException("Product not found"));

        inventory.reserve(request.getQuantity());

        Order order = new Order(request.getUserId(), request.getProductId(), request.getQuantity());

        Order savedOrder = orderRepository.save(order);

        return new OrderResponse(savedOrder.getId(), savedOrder.getStatus());
    }
}
```

---

## Repository

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
}
```

---

## Bad practice

```java
@PostMapping
public Order createOrder(@RequestBody CreateOrderRequest request) {
    Inventory inventory = inventoryRepository.findByProductId(request.getProductId()).get();
    inventory.reserve(request.getQuantity());
    return orderRepository.save(new Order(...));
}
```

Why bad?

- Controller is doing business logic.
- Hard to test.
- Hard to reuse.
- Transaction boundary unclear.

---

# 12. How do you secure APIs?

## Interview question

> How do you secure REST APIs?

---

## Strong answer

Say:

> “I secure APIs using authentication and authorization. Authentication verifies who the user is, often using OAuth2/JWT/session-based authentication. Authorization decides what the user can access using roles, permissions, or entitlement checks. I also validate inputs, avoid exposing sensitive data, use HTTPS, secure secrets, apply least privilege, and log security-relevant events.”

---

## Authentication vs Authorization

```text
Authentication: Who are you?
Authorization: What are you allowed to do?
```

Example:

```text
Logged in user = authenticated
Admin-only endpoint = authorization check
```

---

## Method-level security

```java
@RestController
@RequestMapping("/api/admin")
public class AdminController {

    @PreAuthorize("hasRole('ADMIN')")
    @GetMapping("/reports")
    public List<ReportResponse> getAdminReports() {
        return List.of();
    }
}
```

---

## Spring Security style configuration

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .csrf(csrf -> csrf.disable())
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/actuator/health").permitAll()
                        .requestMatchers("/api/admin/**").hasRole("ADMIN")
                        .requestMatchers("/api/users/**").authenticated()
                        .anyRequest().authenticated()
                )
                .oauth2ResourceServer(oauth2 -> oauth2.jwt())
                .build();
    }
}
```

---

## Gartner-specific strong point

For Gartner-like systems, mention entitlement:

> “For customer-facing content systems, role-based authorization may not be enough. A user may be authenticated but only entitled to certain research content based on subscription, region, or license. I would enforce entitlement checks in the service layer.”

Example:

```java
@Service
public class ContentService {

    private final ContentRepository contentRepository;
    private final EntitlementService entitlementService;

    public ContentService(
            ContentRepository contentRepository,
            EntitlementService entitlementService
    ) {
        this.contentRepository = contentRepository;
        this.entitlementService = entitlementService;
    }

    @Transactional(readOnly = true)
    public ContentResponse getContent(Long contentId, Long userId) {
        Content content = contentRepository.findById(contentId)
                .orElseThrow(() -> new RuntimeException("Content not found"));

        if (!entitlementService.canAccess(userId, contentId)) {
            throw new AccessDeniedException("User is not entitled to access this content");
        }

        return new ContentResponse(content.getId(), content.getTitle(), content.getBody());
    }
}
```

---

# 13. How do you implement caching?

## Interview question

> How would you improve performance for frequently accessed data?

---

## Strong answer

Say:

> “For read-heavy and relatively stable data, I consider caching. In Spring Boot, I can use `@Cacheable` for reads and `@CacheEvict` or `@CachePut` for updates. I’m careful about cache invalidation, TTL, stale data, and whether cache should be local or distributed. For a multi-instance production system, Redis is usually preferred over local in-memory cache.”

---

## Enable caching

```java
@SpringBootApplication
@EnableCaching
public class Application {
}
```

---

## Cache read

```java
@Service
public class ContentService {

    private final ContentRepository contentRepository;

    public ContentService(ContentRepository contentRepository) {
        this.contentRepository = contentRepository;
    }

    @Cacheable(value = "content", key = "#contentId")
    @Transactional(readOnly = true)
    public ContentResponse getContent(Long contentId) {
        Content content = contentRepository.findById(contentId)
                .orElseThrow(() -> new RuntimeException("Content not found"));

        return new ContentResponse(content.getId(), content.getTitle(), content.getBody());
    }
}
```

First call:

```text
DB hit
cache populated
```

Second call:

```text
served from cache
```

---

## Evict cache on update

```java
@CacheEvict(value = "content", key = "#contentId")
@Transactional
public ContentResponse updateContent(Long contentId, UpdateContentRequest request) {
    Content content = contentRepository.findById(contentId)
            .orElseThrow(() -> new RuntimeException("Content not found"));

    content.updateTitle(request.getTitle());
    content.updateBody(request.getBody());

    return new ContentResponse(content.getId(), content.getTitle(), content.getBody());
}
```

---

## Common cache mistakes

| Mistake | Problem |
|---|---|
| Cache everything | Memory/staleness problems |
| No eviction strategy | Stale data |
| Local cache in multi-instance app | Inconsistent cache |
| No TTL | Old data lives too long |
| Caching user-specific data incorrectly | Security/data leakage |

---

## Interview line

Say:

> “Caching is not just adding `@Cacheable`; the hard part is invalidation and consistency.”

That sounds senior.

---

# 14. How do you handle idempotency?

## Interview question

> How do you avoid duplicate processing if client retries a POST request?

Very relevant for reliable APIs.

---

## Strong answer

Say:

> “For critical operations like payment, order creation, or subscription changes, I use an idempotency key. The client sends a unique key with the request. The server stores the key and response. If the same key is received again, the server returns the original response instead of creating a duplicate operation.”

---

## Controller

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(
            @RequestHeader("Idempotency-Key") String idempotencyKey,
            @Valid @RequestBody CreateOrderRequest request
    ) {
        OrderResponse response = orderService.createOrder(idempotencyKey, request);

        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}
```

---

## Entity for idempotency

```java
@Entity
@Table(name = "idempotency_records")
public class IdempotencyRecord {

    @Id
    private String idempotencyKey;

    private Long resourceId;

    private String responseJson;

    protected IdempotencyRecord() {
    }

    public IdempotencyRecord(String idempotencyKey, Long resourceId, String responseJson) {
        this.idempotencyKey = idempotencyKey;
        this.resourceId = resourceId;
        this.responseJson = responseJson;
    }

    public String getIdempotencyKey() {
        return idempotencyKey;
    }

    public Long getResourceId() {
        return resourceId;
    }

    public String getResponseJson() {
        return responseJson;
    }
}
```

---

## Repository

```java
public interface IdempotencyRecordRepository
        extends JpaRepository<IdempotencyRecord, String> {
}
```

---

## Service

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;
    private final IdempotencyRecordRepository idempotencyRecordRepository;

    public OrderService(
            OrderRepository orderRepository,
            IdempotencyRecordRepository idempotencyRecordRepository
    ) {
        this.orderRepository = orderRepository;
        this.idempotencyRecordRepository = idempotencyRecordRepository;
    }

    @Transactional
    public OrderResponse createOrder(String idempotencyKey, CreateOrderRequest request) {
        Optional<IdempotencyRecord> existingRecord =
                idempotencyRecordRepository.findById(idempotencyKey);

        if (existingRecord.isPresent()) {
            IdempotencyRecord record = existingRecord.get();

            return new OrderResponse(
                    record.getResourceId(),
                    "ALREADY_CREATED"
            );
        }

        Order order = new Order(request.getUserId(), request.getProductId(), request.getQuantity());

        Order savedOrder = orderRepository.save(order);

        OrderResponse response = new OrderResponse(savedOrder.getId(), "CREATED");

        IdempotencyRecord record = new IdempotencyRecord(
                idempotencyKey,
                savedOrder.getId(),
                "{\"orderId\":" + savedOrder.getId() + "}"
        );

        idempotencyRecordRepository.save(record);

        return response;
    }
}
```

---

## Important production note

Add unique constraint on idempotency key.

```sql
ALTER TABLE idempotency_records
ADD CONSTRAINT uq_idempotency_key UNIQUE (idempotency_key);
```

This protects against race conditions.

---

## Interview line

Say:

> “Idempotency is critical when clients retry due to timeouts. Without it, duplicate orders or duplicate payments can be created.”

---

# 15. How do you make a Spring Boot service production-ready?

## Interview question

> What makes a Spring Boot backend service production-ready?

This is very important for a Lead role.

---

## Strong answer

Say:

> “A production-ready service should have clear API contracts, validation, consistent exception handling, security, transaction management, proper database indexing, pagination, logging, metrics, tracing, health checks, timeouts, retries, test coverage, CI/CD, rollback strategy, and runbooks for incidents.”

---

## Production readiness checklist

```text
API quality:
- DTOs
- validation
- error response
- status codes
- pagination

Reliability:
- transactions
- idempotency
- timeouts
- retries
- circuit breakers
- fallback strategy

Performance:
- indexes
- DTO projections
- no N+1
- caching
- connection pool tuning

Security:
- auth/authz
- HTTPS
- secret management
- input validation
- least privilege

Observability:
- structured logs
- correlation IDs
- metrics
- tracing
- dashboards
- alerts

Operations:
- health checks
- actuator
- deployment strategy
- rollback
- incident response
```

---

## Actuator configuration

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when_authorized
```

---

## Correlation ID filter

```java
import jakarta.servlet.*;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.slf4j.MDC;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.Optional;
import java.util.UUID;

@Component
public class CorrelationIdFilter implements Filter {

    private static final String CORRELATION_ID = "X-Correlation-ID";

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain
    ) throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        String correlationId = Optional
                .ofNullable(httpRequest.getHeader(CORRELATION_ID))
                .orElse(UUID.randomUUID().toString());

        try {
            MDC.put("correlationId", correlationId);
            httpResponse.setHeader(CORRELATION_ID, correlationId);

            chain.doFilter(request, response);
        } finally {
            MDC.remove("correlationId");
        }
    }
}
```

---

## Why correlation ID matters

If one request flows through multiple services:

```text
frontend → API gateway → service A → service B → database
```

Correlation ID helps trace the same request across logs.

---

## Strong production answer

Say:

> “For production systems, I care not only about writing business logic but also about how the system behaves under failure: can we detect issues, debug them, roll back safely, and prevent recurrence?”

That is exactly Lead Engineer language.

---

# 16. What is dependency injection and why constructor injection?

## Interview question

> What is dependency injection in Spring? Why prefer constructor injection?

---

## Strong answer

Say:

> “Dependency Injection means Spring creates and provides dependencies instead of classes manually creating them. Constructor injection is preferred because dependencies are explicit, can be final, make the object immutable after creation, and improve testability.”

---

## Bad field injection

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;
}
```

Problems:

- Hidden dependencies.
- Harder to unit test.
- Cannot make dependency final.

---

## Good constructor injection

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

---

## Unit test becomes easy

```java
class UserServiceTest {

    @Test
    void shouldReturnUser() {
        UserRepository userRepository = Mockito.mock(UserRepository.class);

        UserService userService = new UserService(userRepository);

        // test service
    }
}
```

---

# 17. `@RestController` vs `@Controller`

## Interview question

> Difference between `@Controller` and `@RestController`?

---

## Answer

Say:

> “`@Controller` is typically used for MVC applications returning views. `@RestController` is used for REST APIs and automatically serializes return values as response body JSON. Internally, `@RestController` combines `@Controller` and `@ResponseBody`.”

---

## REST API

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public UserResponse getUser(@PathVariable Long id) {
        return new UserResponse(id, "Paras", "paras@test.com");
    }
}
```

Returns JSON.

---

## MVC controller

```java
@Controller
public class PageController {

    @GetMapping("/home")
    public String homePage() {
        return "home";
    }
}
```

Returns view name.

---

# 18. `@Component`, `@Service`, `@Repository`

## Interview question

> Difference between `@Component`, `@Service`, and `@Repository`?

---

## Answer

Say:

> “All three are Spring-managed components. `@Component` is generic. `@Service` indicates business/service-layer logic. `@Repository` indicates persistence layer and can participate in exception translation for database exceptions. Using specific annotations improves readability and communicates intent.”

---

## Example

```java
@Component
public class CsvParser {
}
```

```java
@Service
public class UserService {
}
```

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
}
```

---

# 19. Lazy vs Eager loading

## Interview question

> Difference between lazy and eager loading in JPA?

---

## Strong answer

Say:

> “Lazy loading loads related entities only when accessed. Eager loading loads related entities immediately with the parent. Lazy is usually preferred by default because eager loading can fetch too much data unexpectedly. But lazy loading can cause N+1 queries or LazyInitializationException if accessed outside transaction.”

---

## Lazy example

```java
@OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
private List<Order> orders;
```

Orders are loaded only when:

```java
user.getOrders()
```

is accessed.

---

## Eager example

```java
@OneToMany(mappedBy = "user", fetch = FetchType.EAGER)
private List<Order> orders;
```

Orders are loaded whenever user is loaded.

---

## Problem with eager

```text
Fetching user also fetches orders.
Orders may fetch items.
Items may fetch product.
Suddenly one API loads huge graph.
```

---

## Strong interview line

Say:

> “I prefer lazy by default and explicitly fetch what I need using fetch joins, entity graphs, or DTO projections.”

---

# 20. How do you handle concurrent updates?

## Interview question

> Two users update the same record at the same time. How do you handle it?

This is a strong senior-level question.

---

## Option 1: Optimistic locking

Use when conflicts are rare.

```java
@Entity
public class ProductInventory {

    @Id
    private Long id;

    private int availableQuantity;

    @Version
    private Long version;

    public void reduceStock(int quantity) {
        if (availableQuantity < quantity) {
            throw new RuntimeException("Insufficient stock");
        }

        availableQuantity -= quantity;
    }
}
```

What happens?

```text
Transaction A reads version 1
Transaction B reads version 1
A updates successfully -> version becomes 2
B tries update with version 1 -> fails
```

Spring/JPA throws optimistic locking exception.

---

## Option 2: Pessimistic locking

Use when conflicts are frequent and you need strict locking.

```java
public interface ProductInventoryRepository extends JpaRepository<ProductInventory, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("select p from ProductInventory p where p.id = :id")
    Optional<ProductInventory> findByIdForUpdate(@Param("id") Long id);
}
```

Service:

```java
@Transactional
public void reserveStock(Long productId, int quantity) {
    ProductInventory inventory = productInventoryRepository.findByIdForUpdate(productId)
            .orElseThrow(() -> new RuntimeException("Inventory not found"));

    inventory.reduceStock(quantity);
}
```

---

## Interview answer

Say:

> “If conflicts are rare, I prefer optimistic locking using `@Version`. If conflicts are frequent or the operation is highly sensitive, I may use pessimistic locking. The choice depends on contention, performance requirements, and business correctness.”

---

# Rapid-Fire Spring Boot Answers

## Q: What is Spring Boot auto-configuration?

> Spring Boot auto-configures beans based on classpath dependencies and configuration. For example, if Spring MVC is on the classpath, it configures web-related beans automatically.

---

## Q: What are profiles?

> Profiles allow environment-specific configuration like dev, test, staging, and prod.

```yaml
spring:
  profiles:
    active: dev
```

```yaml
# application-dev.yml
server:
  port: 8081
```

```yaml
# application-prod.yml
server:
  port: 8080
```

---

## Q: How do you externalize configuration?

```java
@ConfigurationProperties(prefix = "payment")
public class PaymentProperties {

    private String baseUrl;
    private int timeoutMillis;

    public String getBaseUrl() {
        return baseUrl;
    }

    public void setBaseUrl(String baseUrl) {
        this.baseUrl = baseUrl;
    }

    public int getTimeoutMillis() {
        return timeoutMillis;
    }

    public void setTimeoutMillis(int timeoutMillis) {
        this.timeoutMillis = timeoutMillis;
    }
}
```

Config:

```yaml
payment:
  base-url: https://payment.example.com
  timeout-millis: 3000
```

---

## Q: How do you test a service?

```java
class UserServiceTest {

    @Test
    void shouldReturnUserWhenExists() {
        UserRepository userRepository = Mockito.mock(UserRepository.class);

        User user = new User("Paras", "paras@test.com");

        Mockito.when(userRepository.findById(1L))
                .thenReturn(Optional.of(user));

        UserService userService = new UserService(userRepository);

        UserResponse response = userService.getUserById(1L);

        Assertions.assertEquals("Paras", response.getName());
    }
}
```

---

# Most Important Answers to Memorize

## Exception handling

> “I use `@RestControllerAdvice` and `@ExceptionHandler` to centralize exception handling and return consistent error responses.”

## Transaction

> “`@Transactional` makes a business operation atomic. If a runtime exception occurs, changes are rolled back by default.”

## No transaction

> “Without `@Transactional`, multi-step operations can partially commit and leave data inconsistent.”

## Slow API

> “Measure first, then optimize the bottleneck: DB query, N+1, payload size, pagination, caching, downstream latency, or app-level CPU/memory.”

## N+1

> “One query fetches parents, then N queries fetch children. Fix using fetch join, entity graph, batch fetching, or DTO projection.”

## DTO

> “Entities are persistence models; DTOs are API contracts. DTOs prevent data leakage and decouple API from DB schema.”

## Security

> “Authentication verifies identity; authorization verifies permission. For content systems, entitlement checks may be required beyond roles.”

## Production-ready

> “A production-ready service has validation, consistent errors, security, transactions, observability, health checks, timeouts, retries, tests, and rollback strategy.”

---

# Your Practice Plan for This Module

Do this actively.

## Round 1: Speak answers aloud

Practice these 8:

1. Global exception handling.
2. `@Transactional`.
3. What happens without transaction?
4. Slow API optimization.
5. N+1 query problem.
6. DTO vs Entity.
7. REST API design.
8. Production readiness.

Budget:

```text
45 minutes
```

## Round 2: Code from memory

Write these snippets:

1. `@RestControllerAdvice`
2. `@Transactional` transfer method
3. Paginated API endpoint
4. `@Query` DTO projection
5. `@Cacheable` / `@CacheEvict`

Budget:

```text
60 minutes
```

## Round 3: Mock interview

Answer this scenario:

> “We have a Spring Boot API that returns user dashboard data. It is slow in production and sometimes returns 500 errors. How would you debug and improve it?”

Your answer should include:

```text
logs
metrics
traces
exception handling
DB query analysis
N+1
pagination
payload size
caching
downstream timeout
correlation ID
monitoring/alerts
```

Budget:

```text
20 minutes
```

---

Next best module after this: **SQL/PostgreSQL through interview questions**, because many Spring Boot backend questions naturally lead into indexing, joins, transactions, isolation levels, pagination, and query optimization.
