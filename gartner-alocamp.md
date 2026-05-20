Yes. We’ll convert that huge syllabus into an **interview-first backend mastery track**.

But I’ll be honest: that syllabus is basically a **3–6 month senior backend engineering course**. If I try to cover every item in one response, quality will drop. So we’ll do it in **interview packs**.

No theory dump. Every topic will be taught through:

```text
Interview Question
What interviewer is testing
Strong answer
Code / architecture example
Common follow-ups
Traps
Senior-level comments
```

---

# Backend Interview Master Plan

## Pack 1 — Spring Boot API + Backend Fundamentals

Highest ROI for your Gartner interview.

Covers:

- REST API design
- Global exception handling
- Validation
- `@Transactional`
- slow API optimization
- pagination/filtering
- JPA N+1
- lazy vs eager
- optimistic/pessimistic locking
- idempotency
- caching
- security basics
- observability
- testing

## Pack 2 — Core Java for Backend Engineers

Covers:

- `HashMap` internals
- `equals()` / `hashCode()`
- immutability
- concurrency
- `ExecutorService`
- `CompletableFuture`
- JVM/GC basics
- design patterns
- streams refresher

## Pack 3 — SQL/PostgreSQL + JPA

Covers:

- joins
- indexes
- transactions
- isolation
- deadlocks
- query optimization
- projections
- Flyway
- schema migration
- JPA relationships

## Pack 4 — Microservices + Resilience

Covers:

- monolith vs modular monolith vs microservices
- service discovery
- Feign/WebClient
- retries
- circuit breakers
- bulkheads
- API gateways
- config server
- service mesh basics

## Pack 5 — Kafka + Event-Driven Systems

Covers:

- Kafka partitions
- consumer groups
- retries
- DLQ
- ordering
- idempotent consumers
- outbox pattern
- CDC/Debezium
- event sourcing

## Pack 6 — Redis + Caching + Distributed Locks

Covers:

- cache-aside
- read-through/write-through/write-around
- invalidation
- TTL
- cache stampede
- Redis distributed locks
- Redisson
- Redis GEO

## Pack 7 — Advanced Architecture

Covers:

- CQRS
- SAGA orchestration/choreography
- materialized views
- API composition
- consistency models
- sharding
- replication
- CAP
- system design patterns

## Pack 8 — Security/Auth

Covers:

- Spring Security
- JWT
- refresh tokens
- RBAC
- method-level security
- OAuth2
- API security

## Pack 9 — Testing, CI/CD, Observability

Covers:

- JUnit
- Mockito
- Testcontainers
- REST Assured
- Docker
- GitHub Actions
- Prometheus/Grafana
- ELK
- OpenTelemetry/Jaeger

---

# For Now: Pack 1 — Spring Boot API + Backend Interview Questions

This is the best next step for your interview.

---

# Q1. How do you handle exceptions globally in Spring Boot?

## What interviewer is testing

They want to know if you can build clean production APIs.

Bad backend:

```text
Controller throws random exceptions.
Frontend receives inconsistent errors.
Logs are unclear.
HTTP status codes are wrong.
```

Good backend:

```text
Consistent error response.
Correct HTTP status code.
Clean controller code.
Centralized logging.
Frontend-friendly error format.
```

---

## Strong interview answer

> “I handle exceptions globally using `@RestControllerAdvice` and `@ExceptionHandler`. I create custom exceptions for business errors and return a consistent error response containing timestamp, status, error code, message, path, and optionally correlation ID. This keeps controllers clean and makes frontend integration and production debugging easier.”

---

## Code: Error response DTO

```java
import java.time.Instant;

public class ApiErrorResponse {

    private Instant timestamp;
    private int status;
    private String errorCode;
    private String message;
    private String path;

    public ApiErrorResponse(
            int status,
            String errorCode,
            String message,
            String path
    ) {
        this.timestamp = Instant.now();
        this.status = status;
        this.errorCode = errorCode;
        this.message = message;
        this.path = path;
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
}
```

---

## Code: Custom exception

```java
public class UserNotFoundException extends RuntimeException {

    public UserNotFoundException(Long userId) {
        super("User not found with id: " + userId);
    }
}
```

---

## Code: Global exception handler

```java
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.*;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ApiErrorResponse handleUserNotFound(
            UserNotFoundException exception,
            HttpServletRequest request
    ) {
        return new ApiErrorResponse(
                HttpStatus.NOT_FOUND.value(),
                "USER_NOT_FOUND",
                exception.getMessage(),
                request.getRequestURI()
        );
    }

    @ExceptionHandler(IllegalArgumentException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiErrorResponse handleBadRequest(
            IllegalArgumentException exception,
            HttpServletRequest request
    ) {
        return new ApiErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                "BAD_REQUEST",
                exception.getMessage(),
                request.getRequestURI()
        );
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiErrorResponse handleValidationError(
            MethodArgumentNotValidException exception,
            HttpServletRequest request
    ) {
        String message = exception.getBindingResult()
                .getFieldErrors()
                .stream()
                .findFirst()
                .map(error -> error.getField() + ": " + error.getDefaultMessage())
                .orElse("Validation failed");

        return new ApiErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                "VALIDATION_ERROR",
                message,
                request.getRequestURI()
        );
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ApiErrorResponse handleGenericException(
            Exception exception,
            HttpServletRequest request
    ) {
        return new ApiErrorResponse(
                HttpStatus.INTERNAL_SERVER_ERROR.value(),
                "INTERNAL_SERVER_ERROR",
                "Something went wrong",
                request.getRequestURI()
        );
    }
}
```

---

## Syntax explanation

```java
@RestControllerAdvice
```

Makes this class a global exception handler for REST controllers.

```java
@ExceptionHandler(UserNotFoundException.class)
```

This method handles only `UserNotFoundException`.

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
```

Returns HTTP 404.

```java
HttpServletRequest request
```

Used to get request path:

```java
request.getRequestURI()
```

---

## Common follow-up: Should you expose exception messages?

Answer:

> “For business exceptions, user-safe messages are okay. For unexpected system exceptions, I avoid exposing internal details like stack traces, SQL errors, or class names. I log details internally and return a generic message to clients.”

---

# Q2. How do you validate API requests?

## Interview question

> “How do you validate incoming request payloads in Spring Boot?”

---

## Strong answer

> “I use Jakarta Bean Validation annotations on request DTOs and `@Valid` in controller methods. Simple structural validations like not-null, size, email format belong in DTOs. Business validations, like checking whether email already exists, belong in the service layer.”

---

## Code: Request DTO

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

## Code: Controller

```java
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public UserResponse createUser(@Valid @RequestBody CreateUserRequest request) {
        return userService.createUser(request);
    }
}
```

---

## Syntax explanation

```java
@Valid
```

Triggers validation on request body.

```java
@RequestBody
```

Converts JSON into Java object.

```java
@NotBlank
```

Rejects null, empty, and whitespace-only values.

```java
@Email
```

Checks email format.

---

## Follow-up: Where do business validations go?

Example: email format is DTO validation.

```java
@Email
private String email;
```

But this is business validation:

```text
Is email already registered?
```

That belongs in service:

```java
if (userRepository.existsByEmail(request.getEmail())) {
    throw new IllegalArgumentException("Email already exists");
}
```

---

# Q3. What is `@Transactional`?

## Interview question

> “What is `@Transactional`? What happens if we don’t use it?”

This is extremely important.

---

## Strong answer

> “`@Transactional` defines a transaction boundary. All database operations inside the method either commit together or roll back together. Without it, a multi-step operation can partially succeed, leaving inconsistent data. I usually place `@Transactional` at the service layer because service methods represent business use cases.”

---

## Example scenario: Create order

Business flow:

```text
1. Create order
2. Reduce inventory
3. Create payment record
```

If step 1 succeeds but step 2 fails, we should not keep the order.

---

## Bad code without transaction

```java
public OrderResponse createOrder(CreateOrderRequest request) {
    Order order = orderRepository.save(new Order(request.getUserId()));

    Inventory inventory = inventoryRepository.findByProductId(request.getProductId())
            .orElseThrow(() -> new RuntimeException("Inventory not found"));

    inventory.reduce(request.getQuantity());
    inventoryRepository.save(inventory);

    paymentRepository.save(new Payment(order.getId(), request.getAmount()));

    return new OrderResponse(order.getId());
}
```

Problem:

If order is saved, then inventory update fails:

```text
Order exists.
Inventory not reduced.
Payment not created.
System is inconsistent.
```

---

## Correct code with transaction

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class OrderService {

    private final OrderRepository orderRepository;
    private final InventoryRepository inventoryRepository;
    private final PaymentRepository paymentRepository;

    public OrderService(
            OrderRepository orderRepository,
            InventoryRepository inventoryRepository,
            PaymentRepository paymentRepository
    ) {
        this.orderRepository = orderRepository;
        this.inventoryRepository = inventoryRepository;
        this.paymentRepository = paymentRepository;
    }

    @Transactional
    public OrderResponse createOrder(CreateOrderRequest request) {
        Order order = orderRepository.save(new Order(request.getUserId()));

        Inventory inventory = inventoryRepository.findByProductId(request.getProductId())
                .orElseThrow(() -> new RuntimeException("Inventory not found"));

        inventory.reduce(request.getQuantity());

        paymentRepository.save(new Payment(order.getId(), request.getAmount()));

        return new OrderResponse(order.getId());
    }
}
```

---

## Syntax explanation

```java
@Transactional
```

Starts transaction before method execution.

If method completes successfully:

```text
COMMIT
```

If unchecked exception occurs:

```text
ROLLBACK
```

---

## What rolls back by default?

By default, Spring rolls back on:

```java
RuntimeException
Error
```

But not necessarily on checked exceptions.

---

## Checked exception rollback

```java
@Transactional(rollbackFor = Exception.class)
public void processFile() throws Exception {
    // DB operation 1
    // DB operation 2
    throw new Exception("Checked exception");
}
```

---

## Common trap: self-invocation

Bad:

```java
@Service
public class OrderService {

    public void outerMethod() {
        innerTransactionalMethod();
    }

    @Transactional
    public void innerTransactionalMethod() {
        // transaction may not apply
    }
}
```

Why?

Spring transactions work through proxies. Calling method inside same class bypasses proxy.

---

## Good

Move transactional method to another Spring bean or put transaction on outer method.

```java
@Transactional
public void outerMethod() {
    // all DB logic here
}
```

---

## Common trap: external API inside transaction

Bad:

```java
@Transactional
public void createOrder() {
    orderRepository.save(order);

    paymentGateway.chargeCard(); // external call inside transaction

    inventoryRepository.save(inventory);
}
```

Problem:

- Transaction stays open while external service responds.
- Locks may be held longer.
- External call may succeed but DB rolls back.

Better:

```text
Use transaction for DB state.
Publish event/outbox after DB commit.
Process payment asynchronously if business allows.
```

---

## Interview-ready answer

Say:

> “Without `@Transactional`, each repository call may commit separately, so partial updates can happen. With `@Transactional`, all DB changes inside the method commit or roll back together. I keep transactions short, avoid external calls inside transactions, and place transaction boundaries at service methods.”

---

# Q4. What are transaction propagation types?

## Interview question

> “What is transaction propagation?”

---

## Strong answer

> “Propagation defines how a transactional method behaves when called inside an existing transaction. The most common is `REQUIRED`, which joins the existing transaction or creates a new one if none exists.”

---

## Common propagation types

| Propagation | Meaning |
|---|---|
| `REQUIRED` | Join existing transaction or create new one |
| `REQUIRES_NEW` | Suspend existing transaction and start new one |
| `MANDATORY` | Must run inside existing transaction |
| `SUPPORTS` | Use transaction if exists, else run without |
| `NOT_SUPPORTED` | Run without transaction |
| `NEVER` | Fail if transaction exists |

---

## Code example: audit log should save even if main transaction fails

```java
@Service
public class AuditService {

    private final AuditLogRepository auditLogRepository;

    public AuditService(AuditLogRepository auditLogRepository) {
        this.auditLogRepository = auditLogRepository;
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void saveAuditLog(String message) {
        auditLogRepository.save(new AuditLog(message));
    }
}
```

Usage:

```java
@Transactional
public void createOrder(CreateOrderRequest request) {
    try {
        // main order logic
    } catch (Exception exception) {
        auditService.saveAuditLog("Order failed: " + exception.getMessage());
        throw exception;
    }
}
```

---

## Why `REQUIRES_NEW`?

Audit log commits independently, even if main transaction rolls back.

---

## Interview warning

Use `REQUIRES_NEW` carefully. It creates separate transaction boundaries and can surprise people.

---

# Q5. This API is slow. How will you optimize it?

## Interview question

> “A Spring Boot API is taking 5 seconds in production. How do you debug and optimize it?”

This is very likely for a Sr Architect round.

---

## Strong answer

> “I first measure where time is spent instead of guessing. I check application logs, metrics, traces, DB query timings, external service latency, thread pool usage, and recent deployments. Then I optimize based on bottleneck: database indexes, pagination, projections, caching, fixing N+1 queries, reducing payload size, parallelizing independent calls, adding timeouts, or improving connection pool settings.”

---

## Step-by-step answer

```text
1. Reproduce or inspect production metrics.
2. Check endpoint latency p95/p99.
3. Check logs with correlation ID.
4. Check DB query time.
5. Check external service calls.
6. Check payload size.
7. Check thread/connection pool saturation.
8. Check recent deployments.
9. Optimize identified bottleneck.
10. Add monitoring to prevent recurrence.
```

---

## Code: simple request timing filter

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

        long start = System.currentTimeMillis();

        try {
            chain.doFilter(request, response);
        } finally {
            long duration = System.currentTimeMillis() - start;
            HttpServletRequest httpRequest = (HttpServletRequest) request;

            log.info("method={} uri={} durationMs={}",
                    httpRequest.getMethod(),
                    httpRequest.getRequestURI(),
                    duration);
        }
    }
}
```

---

## Common DB optimization

Bad:

```java
List<Order> orders = orderRepository.findByUserId(userId);
```

If user has 100,000 orders, this is bad.

Better:

```java
Page<Order> orders = orderRepository.findByUserId(
        userId,
        PageRequest.of(0, 20, Sort.by(Sort.Direction.DESC, "createdAt"))
);
```

Repository:

```java
Page<Order> findByUserId(Long userId, Pageable pageable);
```

---

## Projection instead of full entity

Bad:

```java
List<Order> orders = orderRepository.findByUserId(userId);
```

If API needs only:

```text
id, status, totalAmount, createdAt
```

Use projection.

```java
public interface OrderSummaryProjection {

    Long getId();

    String getStatus();

    BigDecimal getTotalAmount();

    LocalDateTime getCreatedAt();
}
```

Repository:

```java
@Query("""
       select o.id as id,
              o.status as status,
              o.totalAmount as totalAmount,
              o.createdAt as createdAt
       from Order o
       where o.user.id = :userId
       order by o.createdAt desc
       """)
Page<OrderSummaryProjection> findOrderSummariesByUserId(
        @Param("userId") Long userId,
        Pageable pageable
);
```

---

## SQL index for this query

If query filters by user and sorts by created date:

```sql
CREATE INDEX idx_orders_user_created_at
ON orders(user_id, created_at DESC);
```

---

## Interview-ready answer

Say:

> “For slow APIs, I don’t start by changing code blindly. I measure first. If DB is bottleneck, I check query plans, add indexes matching filter/sort patterns, use pagination, projections, and fix N+1. If downstream calls are bottleneck, I add timeouts, retries, circuit breakers, or parallelize independent calls. If repeated stable data is read often, I consider caching.”

---

# Q6. What is the N+1 query problem?

## Interview question

> “What is N+1 in JPA, and how do you fix it?”

---

## Strong answer

> “N+1 happens when the application executes one query to fetch parent records and then one additional query per parent to fetch child records. For example, one query fetches 100 orders, then 100 queries fetch order items. This can severely degrade performance.”

---

## Example entities

```java
@Entity
public class Order {

    @Id
    private Long id;

    private String status;

    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderItem> items;

    public List<OrderItem> getItems() {
        return items;
    }
}
```

---

## Bad code

```java
List<Order> orders = orderRepository.findAll();

for (Order order : orders) {
    System.out.println(order.getItems().size());
}
```

What happens?

```text
1 query  -> fetch all orders
N queries -> fetch items for each order
```

---

## Fix 1: fetch join

```java
@Query("""
       select distinct o
       from Order o
       left join fetch o.items
       where o.status = :status
       """)
List<Order> findOrdersWithItemsByStatus(@Param("status") String status);
```

---

## Fix 2: EntityGraph

```java
@EntityGraph(attributePaths = {"items"})
List<Order> findByStatus(String status);
```

---

## Fix 3: DTO projection

```java
@Query("""
       select new com.example.OrderItemSummaryDto(
           o.id,
           o.status,
           i.productName,
           i.quantity
       )
       from Order o
       join o.items i
       where o.status = :status
       """)
List<OrderItemSummaryDto> findOrderItemSummaries(@Param("status") String status);
```

---

## Interview answer

Say:

> “I fix N+1 by fetching required associations explicitly using fetch joins, entity graphs, batch fetching, or DTO projections. For read APIs, I often prefer projections because they fetch only the fields needed by the response.”

---

# Q7. Lazy loading vs eager loading?

## Interview question

> “Difference between lazy and eager loading?”

---

## Strong answer

> “Lazy loading loads associated data only when accessed. Eager loading loads associated data immediately with the parent. Lazy loading avoids unnecessary data fetches but can cause N+1 if misused. Eager loading can over-fetch and hurt performance. I prefer lazy by default and explicitly fetch what each use case needs.”

---

## Example

```java
@ManyToOne(fetch = FetchType.LAZY)
private User user;
```

This means:

```text
Order is loaded now.
User is loaded only when order.getUser() is accessed.
```

---

## Eager example

```java
@ManyToOne(fetch = FetchType.EAGER)
private User user;
```

This means:

```text
Whenever order loads, user loads too.
```

---

## Senior answer

Say:

> “I avoid making everything eager. It can create large object graphs and unpredictable queries. Instead, I keep associations lazy and use fetch joins or projections for specific read use cases.”

---

# Q8. How do you implement pagination, filtering, and sorting?

## Interview question

> “How do you design list APIs?”

---

## Strong answer

> “List APIs should never return unbounded data. I use pagination, filtering, sorting, and stable ordering. For standard admin-style lists, offset pagination is fine. For large or infinite-scroll APIs, I prefer keyset pagination.”

---

## Spring Pageable example

Controller:

```java
@GetMapping("/orders")
public Page<OrderSummaryProjection> getOrders(
        @RequestParam Long userId,
        @RequestParam(required = false) String status,
        Pageable pageable
) {
    return orderService.getOrders(userId, status, pageable);
}
```

Service:

```java
public Page<OrderSummaryProjection> getOrders(
        Long userId,
        String status,
        Pageable pageable
) {
    if (status == null) {
        return orderRepository.findOrderSummariesByUserId(userId, pageable);
    }

    return orderRepository.findOrderSummariesByUserIdAndStatus(userId, status, pageable);
}
```

Repository:

```java
Page<OrderSummaryProjection> findOrderSummariesByUserId(
        Long userId,
        Pageable pageable
);

Page<OrderSummaryProjection> findOrderSummariesByUserIdAndStatus(
        Long userId,
        String status,
        Pageable pageable
);
```

Example URL:

```text
GET /orders?userId=101&status=COMPLETED&page=0&size=20&sort=createdAt,desc
```

---

## Interview answer

Say:

> “For list endpoints, I expose page, size, sort, and filter parameters. I enforce max page size to protect the system. For large datasets, I consider cursor/keyset pagination.”

---

# Q9. How do you enforce idempotency in APIs?

## Interview question

> “If client retries a create-payment API, how do you avoid duplicate payments?”

Very important for backend/system design.

---

## Strong answer

> “For non-idempotent operations like payment or order creation, I use an idempotency key. The client sends a unique key for the operation. The server stores the key and response. If the same key is retried, the server returns the original response instead of executing the operation again.”

---

## Controller

```java
@PostMapping("/payments")
@ResponseStatus(HttpStatus.CREATED)
public PaymentResponse createPayment(
        @RequestHeader("Idempotency-Key") String idempotencyKey,
        @Valid @RequestBody CreatePaymentRequest request
) {
    return paymentService.createPayment(idempotencyKey, request);
}
```

---

## Service

```java
@Transactional
public PaymentResponse createPayment(
        String idempotencyKey,
        CreatePaymentRequest request
) {
    Optional<IdempotencyRecord> existingRecord =
            idempotencyRepository.findByKey(idempotencyKey);

    if (existingRecord.isPresent()) {
        return existingRecord.get().toPaymentResponse();
    }

    Payment payment = paymentRepository.save(
            new Payment(request.getUserId(), request.getAmount())
    );

    PaymentResponse response = new PaymentResponse(payment.getId(), payment.getStatus());

    idempotencyRepository.save(
            new IdempotencyRecord(idempotencyKey, response)
    );

    return response;
}
```

---

## DB constraint

```sql
ALTER TABLE idempotency_records
ADD CONSTRAINT uq_idempotency_key UNIQUE (idempotency_key);
```

---

## Interview answer

Say:

> “The idempotency key must be protected by a unique constraint at the DB level. Application checks alone are not enough under concurrent retries.”

---

# Q10. How do you handle race conditions in booking/inventory?

## Interview question

> “Two users book the last room at the same time. How do you prevent double booking?”

---

## Strong answer

> “I handle this with database constraints and locking. Depending on conflict frequency, I use optimistic locking with a version column or pessimistic locking with `SELECT FOR UPDATE`. I also enforce uniqueness constraints at the database level.”

---

## Optimistic locking

Entity:

```java
@Entity
public class RoomAvailability {

    @Id
    private Long id;

    private Long roomId;

    private LocalDate date;

    private boolean booked;

    @Version
    private Long version;

    public void book() {
        if (booked) {
            throw new IllegalStateException("Room already booked");
        }

        this.booked = true;
    }
}
```

Service:

```java
@Transactional
public void bookRoom(Long availabilityId) {
    RoomAvailability availability = roomAvailabilityRepository.findById(availabilityId)
            .orElseThrow(() -> new RuntimeException("Availability not found"));

    availability.book();
}
```

If two transactions update same row, one fails due to version conflict.

---

## Pessimistic locking

Repository:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("select r from RoomAvailability r where r.id = :id")
Optional<RoomAvailability> findByIdForUpdate(@Param("id") Long id);
```

Service:

```java
@Transactional
public void bookRoom(Long availabilityId) {
    RoomAvailability availability = roomAvailabilityRepository.findByIdForUpdate(availabilityId)
            .orElseThrow(() -> new RuntimeException("Availability not found"));

    availability.book();
}
```

---

## Interview answer

Say:

> “If conflicts are rare, optimistic locking is more scalable. If conflicts are frequent or correctness is critical, pessimistic locking is safer but reduces concurrency. I also add DB constraints so correctness does not depend only on application logic.”

---

# Q11. How do you cache expensive API responses?

## Interview question

> “How do you add caching in Spring Boot?”

---

## Strong answer

> “I use caching for read-heavy data that does not change frequently. In Spring Boot, I can use the Spring Cache abstraction with Redis or local cache. I define cache keys carefully, set TTLs, and handle invalidation on updates.”

---

## Enable caching

```java
@EnableCaching
@SpringBootApplication
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
    public ContentResponse getContent(Long contentId) {
        Content content = contentRepository.findById(contentId)
                .orElseThrow(() -> new RuntimeException("Content not found"));

        return ContentResponse.from(content);
    }
}
```

---

## Evict cache on update

```java
@CacheEvict(value = "content", key = "#contentId")
public void updateContent(Long contentId, UpdateContentRequest request) {
    Content content = contentRepository.findById(contentId)
            .orElseThrow(() -> new RuntimeException("Content not found"));

    content.update(request.getTitle(), request.getBody());
}
```

---

## Interview answer

Say:

> “Caching improves latency and reduces database load, but the hard part is invalidation. I use TTLs, explicit eviction on updates, and avoid caching highly personalized or frequently changing data unless consistency requirements allow it.”

---

# Q12. How do you secure REST APIs?

## Interview question

> “How do you secure APIs in Spring Boot?”

---

## Strong answer

> “I separate authentication and authorization. Authentication verifies who the user is, often using JWT/OAuth2/session. Authorization checks what the user can access using roles, permissions, or entitlements. I also validate input, use HTTPS, avoid sensitive data exposure, protect secrets, and log security-relevant events.”

---

## Method-level authorization

```java
@PreAuthorize("hasRole('ADMIN')")
@GetMapping("/admin/reports")
public List<ReportResponse> getAdminReports() {
    return reportService.getReports();
}
```

---

## Role + ownership check

```java
@PreAuthorize("hasRole('ADMIN') or @securityService.isOwner(#userId, authentication)")
@GetMapping("/users/{userId}/orders")
public List<OrderResponse> getUserOrders(@PathVariable Long userId) {
    return orderService.getOrders(userId);
}
```

---

## Security service

```java
@Component
public class SecurityService {

    public boolean isOwner(Long userId, Authentication authentication) {
        String loggedInUserId = authentication.getName();

        return loggedInUserId.equals(String.valueOf(userId));
    }
}
```

---

## Interview answer

Say:

> “For customer-facing systems, authentication alone is not enough. A logged-in user may still not be entitled to view every resource. I enforce authorization at service or method level and validate ownership/entitlement.”

---

# Q13. How do you call downstream services safely?

## Interview question

> “Your service calls another service. What can go wrong, and how do you protect your API?”

---

## Strong answer

> “Downstream services can be slow, unavailable, or return errors. I protect my service using timeouts, retries with backoff, circuit breakers, fallbacks, bulkheads, and clear error handling. I avoid infinite retries and always set timeouts.”

---

## Resilience4j example

```java
@CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
@Retry(name = "paymentService")
public PaymentStatus getPaymentStatus(Long paymentId) {
    return paymentClient.getPaymentStatus(paymentId);
}

public PaymentStatus paymentFallback(Long paymentId, Exception exception) {
    return PaymentStatus.UNKNOWN;
}
```

---

## Configuration example

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        failure-rate-threshold: 50
        sliding-window-size: 10
        wait-duration-in-open-state: 30s

  retry:
    instances:
      paymentService:
        max-attempts: 3
        wait-duration: 500ms
```

---

## Interview answer

Say:

> “I only retry safe operations or idempotent operations. Retrying payment creation blindly can cause duplicate payments unless idempotency is implemented.”

Strong senior-level point.

---

# Q14. How do you make APIs observable?

## Interview question

> “What do you add to make a Spring Boot service production-ready?”

---

## Strong answer

> “I add structured logs, metrics, traces, health checks, dashboards, alerts, and correlation IDs. I track latency, error rate, throughput, dependency failures, DB timings, and resource usage.”

---

## Actuator config

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
import org.slf4j.MDC;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.UUID;

@Component
public class CorrelationIdFilter implements Filter {

    private static final String CORRELATION_ID = "correlationId";

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain
    ) throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;

        String correlationId = httpRequest.getHeader("X-Correlation-ID");

        if (correlationId == null || correlationId.isBlank()) {
            correlationId = UUID.randomUUID().toString();
        }

        try {
            MDC.put(CORRELATION_ID, correlationId);
            chain.doFilter(request, response);
        } finally {
            MDC.remove(CORRELATION_ID);
        }
    }
}
```

---

## Interview answer

Say:

> “Correlation IDs help trace one request across logs and services. For distributed systems, I’d use OpenTelemetry tracing with tools like Jaeger or Zipkin.”

---

# Q15. How do you test Spring Boot services?

## Interview question

> “How do you test backend code?”

---

## Strong answer

> “I test at multiple levels: unit tests for service logic, controller tests for API behavior, repository tests for database queries, and integration tests for full flows. I use JUnit and Mockito for unit tests, and Testcontainers for real dependencies when needed.”

---

## Unit test with Mockito

```java
import org.junit.jupiter.api.Test;
import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

class UserServiceTest {

    @Test
    void shouldReturnUserWhenUserExists() {
        UserRepository userRepository = mock(UserRepository.class);

        UserService userService = new UserService(userRepository);

        User user = new User(1L, "Paras", "paras@example.com");

        when(userRepository.findById(1L)).thenReturn(Optional.of(user));

        UserResponse response = userService.getUserById(1L);

        assertEquals("Paras", response.getName());

        verify(userRepository).findById(1L);
    }

    @Test
    void shouldThrowWhenUserNotFound() {
        UserRepository userRepository = mock(UserRepository.class);

        UserService userService = new UserService(userRepository);

        when(userRepository.findById(99L)).thenReturn(Optional.empty());

        assertThrows(UserNotFoundException.class, () -> userService.getUserById(99L));
    }
}
```

---

## Interview answer

Say:

> “For business logic, I prefer fast unit tests. For repository queries and DB behavior, I use integration tests because mocks cannot catch SQL or mapping issues.”

---

# Q16. What makes a backend service production-ready?

## Interview question

> “What are the things you check before releasing a backend service?”

---

## Strong answer

> “A production-ready service should have validation, exception handling, secure APIs, transaction boundaries, database indexes, pagination, tests, logging, metrics, tracing, health checks, timeouts, retries, CI/CD, rollback strategy, and runbooks.”

---

## Checklist

```text
API correctness:
- validation
- status codes
- consistent errors
- idempotency where needed

Data correctness:
- transactions
- constraints
- indexes
- locking for race conditions

Performance:
- pagination
- projections
- caching
- N+1 prevention

Reliability:
- timeouts
- retries
- circuit breakers
- fallback behavior

Security:
- authentication
- authorization
- input validation
- secret management

Observability:
- logs
- metrics
- traces
- health checks
- alerts

Delivery:
- tests
- CI/CD
- rollback
- feature flags
```

---

# Rapid-Fire Spring Boot Interview Answers

## Q. `@Controller` vs `@RestController`?

> `@Controller` is used for MVC views. `@RestController` combines `@Controller` and `@ResponseBody`, so return values are serialized as JSON/XML response bodies.

---

## Q. `@Component` vs `@Service` vs `@Repository`?

> All are Spring beans. `@Component` is generic. `@Service` marks business logic. `@Repository` marks persistence layer and can translate persistence exceptions.

---

## Q. Constructor injection vs field injection?

> I prefer constructor injection because dependencies are explicit, final, easier to test, and safer for immutability. Field injection hides dependencies and makes testing harder.

---

## Q. What is `@RequestBody`?

> It maps the HTTP request body JSON to a Java object.

---

## Q. What is `@PathVariable` vs `@RequestParam`?

> `@PathVariable` reads values from URL path like `/users/{id}`. `@RequestParam` reads query parameters like `/users?status=ACTIVE`.

---

## Q. What is DTO and why use it?

> DTO represents API input/output. Entities represent database tables. DTOs avoid exposing internal schema, sensitive fields, and lazy-loading issues.

---

## Q. What is Spring Boot Actuator?

> Actuator provides production endpoints like health, metrics, info, and prometheus metrics for monitoring.

---

# Your Study Assignment for This Pack

Do not just read. Practice answering these aloud:

```text
1. How do you handle exceptions globally?
2. How do you validate API requests?
3. What is @Transactional?
4. What happens without @Transactional?
5. What are @Transactional pitfalls?
6. How do you optimize a slow API?
7. What is N+1?
8. Lazy vs eager loading?
9. How do you implement pagination?
10. How do you handle idempotency?
11. How do you prevent double booking?
12. How do you cache API responses?
13. How do you secure APIs?
14. How do you call downstream services safely?
15. How do you make a service production-ready?
```

---

# Best Next Pack

Next we should do:

# Pack 2 — Spring Data JPA + PostgreSQL Through Interview Questions

This will cover:

```text
relationships
cascade
fetch types
N+1 deeper
indexes
transactions
isolation
deadlocks
optimistic/pessimistic locking
Flyway
query optimization
pagination
projections
```

This is the natural continuation from Pack 1 and extremely relevant to your interview.
