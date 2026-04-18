# Java & Spring Boot — Interview Prep

---

## 1. OOP Concepts

**Q: What are the four pillars of OOP?**

- **Encapsulation** — bundling data and methods together, hiding internal state behind an interface.
- **Abstraction** — exposing only what's necessary and hiding implementation details.
- **Inheritance** — a class (child) acquires properties and behavior from another class (parent).
- **Polymorphism** — the same interface behaves differently depending on the underlying type.

---

**Q: What are the member access modifiers in Java?**

| Modifier | Same Class | Same Package | Subclass | Anywhere |
|---|---|---|---|---|
| `private` | Yes | No | No | No |
| (default/package-private) | Yes | Yes | No | No |
| `protected` | Yes | Yes | Yes | No |
| `public` | Yes | Yes | Yes | Yes |

`private` is the most restrictive. Use it by default and loosen only when needed.

---

**Q: What is method overloading?**

Overloading is defining multiple methods with the **same name but different parameter lists** (type, number, or order of parameters) in the same class. The compiler picks the right one at **compile time** — this is called *static/compile-time polymorphism*.

```java
int add(int a, int b) { return a + b; }
double add(double a, double b) { return a + b; }
```

---

**Q: What is method overriding?**

Overriding is when a **subclass provides its own implementation** of a method already defined in its parent class, keeping the same signature. The JVM picks the right version at **runtime** based on the actual object type — this is *dynamic/runtime polymorphism*.

```java
class Animal { void speak() { System.out.println("..."); } }
class Dog extends Animal { @Override void speak() { System.out.println("Woof"); } }

Animal a = new Dog();
a.speak(); // prints "Woof" — runtime dispatch
```

Key rules: same method name, same parameter list, same or wider return type (covariant), same or less restrictive access modifier.

---

**Q: What is polymorphism? Compile-time vs runtime?**

**Polymorphism** means "many forms" — the same method call behaves differently depending on context.

- **Compile-time (static) polymorphism** — resolved by the compiler. Achieved via method overloading.
- **Runtime (dynamic) polymorphism** — resolved at runtime by the JVM. Achieved via method overriding + inheritance. The reference type can be a parent class/interface, but the actual object determines which method runs.

---

**Q: How does Java handle virtual methods / runtime dispatch?**

In Java, **all non-static, non-final, non-private instance methods are virtual by default**. When you call a method on a reference, the JVM looks up the actual runtime type of the object (not the declared reference type) and dispatches to the correct implementation. This is implemented via a vtable (virtual method table) under the hood.

Unlike C++, Java doesn't require an explicit `virtual` keyword — it's the default behavior.

---

**Q: What is inheritance? Abstract class vs Interface?**

**Inheritance** lets a class (`extends`) inherit fields and methods from a parent, promoting reuse.

| | Abstract Class | Interface |
|---|---|---|
| Can have fields | Yes | Only `static final` constants |
| Can have constructors | Yes | No |
| Can have concrete methods | Yes | Yes (via `default` methods since Java 8) |
| Multiple inheritance | No (single `extends`) | Yes (multiple `implements`) |
| Use when | Sharing state/behavior among related classes | Defining a contract any type can fulfill |

An **abstract class** cannot be instantiated and may have abstract methods that subclasses must implement. An **interface** is a pure contract. Prefer interfaces for type declaration; use abstract classes when you need shared implementation.

---

## 2. Multithreading

**Q: What is multithreading?**

Multithreading is running multiple **threads** (lightweight units of execution) within a single process, sharing the same memory space. It allows concurrent execution — doing multiple things at once (or appearing to). Contrast with **multiprocessing**, where separate processes have separate memory.

- **Concurrency** — multiple tasks making progress, possibly interleaved on one CPU.
- **Parallelism** — multiple tasks literally running simultaneously on multiple CPUs/cores.

In Java, threads are created via `Thread`, `Runnable`, `Callable`, or `ExecutorService`.

---

**Q: What are the key problems in multithreaded programming?**

- **Race condition** — two threads read/write shared data simultaneously, producing unpredictable results.
- **Deadlock** — thread A holds lock 1 waiting for lock 2; thread B holds lock 2 waiting for lock 1. Both wait forever.
- **Starvation** — a thread never gets CPU time because higher-priority threads keep running.
- **Visibility problem** — changes made by one thread may not be visible to another due to CPU caching.

---

**Q: How do you achieve thread safety in Java?**

- **`synchronized`** — only one thread at a time can execute a synchronized block/method. Ensures mutual exclusion and visibility.
- **`volatile`** — guarantees visibility of changes across threads. Does NOT ensure atomicity for compound operations.
- **`ReentrantLock`** — more flexible than `synchronized`; supports tryLock, timed locks, fair ordering.
- **Atomic classes** (`AtomicInteger`, `AtomicReference`) — lock-free thread-safe operations.
- **`java.util.concurrent` collections** — `ConcurrentHashMap`, `CopyOnWriteArrayList`, etc.
- **Stateless design** — if a class has no shared mutable state, it's inherently thread-safe (the ideal approach in Spring Boot).

---

**Q: Things to keep in mind when writing multithreaded code?**

1. Minimize shared mutable state — the root of most threading bugs.
2. Use the highest-level abstraction available (`ExecutorService`, `CompletableFuture`) over raw threads.
3. Prefer immutable objects.
4. Never call blocking code in a thread that holds a lock.
5. Always acquire locks in the same order everywhere to avoid deadlocks.
6. Use `volatile` for simple flags; use `AtomicXxx` for counters/single variables; use `synchronized` or locks for compound operations.
7. Test concurrency explicitly — bugs are non-deterministic and hard to reproduce.

---

## 3. Design Patterns

**Q: What is the Singleton pattern?**

Ensures that **only one instance** of a class exists throughout the application, and provides a global access point to it.

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {           // double-checked locking
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

Use when: shared resources like a config manager, connection pool, or logger.  
Spring beans are singletons by default — Spring manages this for you.

---

**Q: What is the Factory pattern?**

Defines an interface for creating objects but lets subclasses/methods decide which class to instantiate. Decouples object creation from usage.

```java
interface Shape { void draw(); }
class Circle implements Shape { public void draw() { System.out.println("Circle"); } }
class Square implements Shape { public void draw() { System.out.println("Square"); } }

class ShapeFactory {
    public static Shape create(String type) {
        return switch (type) {
            case "circle" -> new Circle();
            case "square" -> new Square();
            default -> throw new IllegalArgumentException("Unknown shape");
        };
    }
}
```

Use when: you want to create objects without exposing instantiation logic, or the type is determined at runtime.

---

**Q: What is the Observer pattern?**

Defines a one-to-many dependency: when one object (subject/publisher) changes state, all registered observers (subscribers) are notified automatically.

```java
interface Observer { void update(String event); }

class EventSource {
    private List<Observer> observers = new ArrayList<>();
    public void subscribe(Observer o) { observers.add(o); }
    public void publish(String event) { observers.forEach(o -> o.update(event)); }
}
```

Use when: event systems, UI listeners, pub/sub messaging, any scenario where state changes in one object should trigger reactions in others. Spring's `ApplicationEvent` system uses this pattern.

---

## 4. Dependency Injection & Autowiring

**Q: What is Dependency Injection (DI)?**

Dependency Injection is a design principle where an object **receives its dependencies from an external source** rather than creating them itself. Instead of writing `new ServiceImpl()` inside your class, something else (the Spring IoC container) hands the dependency in.

```java
// Without DI — tight coupling, hard to test
public class OrderService {
    private PaymentService paymentService = new PaymentService(); // hardcoded
}

// With DI — loose coupling, easy to swap or mock
public class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) { // injected
        this.paymentService = paymentService;
    }
}
```

The Spring container manages object creation and wiring. You just declare what you need.

---

**Q: Why is Dependency Injection important?**

- **Loose coupling** — classes don't know about each other's concrete implementations; they depend on abstractions (interfaces). Swap implementations without changing consumers.
- **Testability** — you can inject a mock or stub during unit tests instead of the real dependency.
- **Single Responsibility** — classes focus on their own logic, not on creating collaborators.
- **Reusability** — the same bean can be injected wherever needed; Spring manages its lifecycle.
- **Inversion of Control (IoC)** — control over object creation is inverted from your code to the framework. DI is the mechanism that implements IoC.

Without DI, changing a dependency means hunting down every `new ConcreteImpl()` in the codebase. With DI, you change one configuration and every consumer is updated automatically.

---

**Q: What are the types of Dependency Injection?**

Spring supports three types:

### 1. Constructor Injection (recommended)
Dependencies are passed via the constructor. With a single constructor, `@Autowired` is optional since Spring 4.3.

```java
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;

    // @Autowired optional here (single constructor)
    public OrderService(PaymentService paymentService, InventoryService inventoryService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
    }
}
```

**Why it's best:** dependencies are `final` (immutable after construction), clearly declared, and the class is impossible to instantiate without all required dependencies — making missing deps a compile-time issue, not a runtime surprise. Also easiest to unit test (just call `new` with mocks).

### 2. Setter Injection
Dependencies are injected via setter methods after object construction. Use for **optional** dependencies.

```java
@Service
public class ReportService {
    private NotificationService notificationService;

    @Autowired(required = false)
    public void setNotificationService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }
}
```

**Drawback:** the dependency is not `final` and could be null if not set, requiring null checks.

### 3. Field Injection
`@Autowired` directly on the field. Spring uses reflection to inject it.

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository; // injected by Spring via reflection
}
```

**Convenient but discouraged** for production code: fields can't be `final`, you can't write a plain constructor call in tests (you need a Spring context or reflection to inject), and it hides dependencies — the class's requirements aren't visible in its constructor.

---

**Q: What is `@Autowired` and how does it work?**

`@Autowired` tells Spring to inject a matching bean from the application context into the annotated constructor, setter, or field.

Spring resolves the dependency by **type first**: it looks for a bean whose type matches (or is assignable to) the declared type. If exactly one match is found, it injects it.

```java
@Service
public class InvoiceService {
    private final TaxCalculator taxCalculator;

    @Autowired
    public InvoiceService(TaxCalculator taxCalculator) {
        this.taxCalculator = taxCalculator;
    }
}
```

**What happens if there are multiple matching beans?** Spring throws `NoUniqueBeanDefinitionException`. Resolve it with:

- **`@Qualifier("beanName")`** — specify exactly which bean to inject by name.
- **`@Primary`** — mark one bean as the default when multiple candidates exist.

```java
@Autowired
@Qualifier("usdTaxCalculator")
private TaxCalculator taxCalculator;
```

**`required = false`** — marks the dependency as optional. If no matching bean is found, Spring injects `null` instead of throwing an exception.

```java
@Autowired(required = false)
private OptionalFeature optionalFeature;
```

---

**Q: How does Spring decide which bean to inject? (Type vs Name resolution)**

Spring's autowiring resolution order:
1. **By type** — finds all beans assignable to the declared type.
2. **If multiple found** — narrows by `@Qualifier` name or, if absent, by field/parameter name matching a bean name.
3. **If `@Primary` is present on one candidate** — that one wins.
4. **If still ambiguous** — throws `NoUniqueBeanDefinitionException`.

```java
@Component("fastEngine")
public class FastEngine implements Engine { ... }

@Component("slowEngine")
public class SlowEngine implements Engine { ... }

@Service
public class Car {
    @Autowired
    @Qualifier("fastEngine")   // disambiguate
    private Engine engine;
}
```

---

**Q: Constructor injection vs field injection — which should you use and why?**

Always prefer **constructor injection** in production code:

| | Constructor Injection | Field Injection |
|---|---|---|
| Dependencies `final` | Yes | No |
| Visible in constructor | Yes — clear contract | No — hidden |
| Unit testable without Spring | Yes — just `new Class(mock)` | No — need Spring context or reflection |
| Detects circular deps | At startup | At runtime (may succeed) |
| Recommended by Spring team | Yes | No |

Field injection is fine for quick prototypes or test classes (`@MockBean` in tests), but in application code constructor injection is the standard.

---

**Q: What is the difference between letting Spring inject a dependency (`@Autowired`) vs creating one with `new`?**

This is one of the most important things to understand about Spring.

When you write `new SomeService()`, you are creating an object **outside of the Spring container**. Spring has no knowledge of this object — it was never registered as a bean, never passed through the application context, and its lifecycle is entirely yours to manage.

The critical consequence: **any `@Autowired` fields or constructor dependencies inside that manually created object will NOT be injected**. Spring only wires dependencies into objects it creates and manages.

```java
@Service
public class PaymentService {
    @Autowired
    private TransactionRepository transactionRepo;  // Spring injects this normally

    public void process() {
        // transactionRepo is available — Spring created this bean
    }
}

@Service
public class OrderService {
    public void placeOrder() {
        // WRONG — creating PaymentService manually
        PaymentService ps = new PaymentService();
        ps.process();
        // ps.transactionRepo is NULL — Spring never touched this object
        // This will throw NullPointerException when process() tries to use transactionRepo
    }
}
```

**Why is this?**

Spring's dependency injection works through a **proxy and post-processing mechanism**. When Spring creates a bean, it:
1. Instantiates the object.
2. Runs `BeanPostProcessor` logic to scan for `@Autowired`, `@Value`, etc.
3. Injects the dependencies.
4. Returns the fully wired object (often a proxy, especially for `@Transactional`).

When you call `new`, you skip all of that. You get a plain Java object with no Spring processing applied — `@Autowired` annotations are just metadata sitting there doing nothing.

**The same problem applies to other Spring features:**

Not just `@Autowired` — any Spring annotation on the manually created object is silently ignored:

```java
@Service
public class ReportService {
    @Autowired
    private DataSource dataSource;      // ignored — Spring didn't create this

    @Transactional                      // ignored — no Spring proxy
    public void generateReport() {
        // dataSource is NULL — NullPointerException
        // @Transactional has no effect — no transaction will be opened
    }
}

// Somewhere else — the wrong way
ReportService rs = new ReportService();   // completely bypasses Spring
rs.generateReport();                      // crash or silent wrong behavior
```

**The right way — always get beans from Spring:**

```java
@Service
public class OrderService {
    private final PaymentService paymentService;   // Spring-managed

    // Spring injects a fully wired PaymentService here
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder() {
        paymentService.process();   // transactionRepo is wired, @Transactional works
    }
}
```

**Summary — `@Autowired` (Spring-managed) vs `new` (manually created):**

| | Spring-managed (`@Autowired`) | Manually created (`new`) |
|---|---|---|
| Spring aware | Yes | No |
| Dependencies injected | Yes | No — all `@Autowired` fields are `null` |
| `@Transactional` works | Yes (Spring wraps with proxy) | No — annotation is ignored |
| `@Value` properties injected | Yes | No |
| Singleton scope enforced | Yes | No — new instance every time |
| Lifecycle hooks (`@PostConstruct`) | Called | Not called |

**The rule:** never instantiate a Spring component with `new` in application code. If you need an object that has dependencies, make it a bean and inject it. If you need a fresh instance per use (not a singleton), use `@Scope("prototype")` and inject the provider, or use `ApplicationContext.getBean()` — but the object must still be created by Spring.

---

## 5. Spring Boot

**Q: What is Spring Boot?**

Spring Boot is an opinionated framework built on top of the Spring Framework that simplifies building production-ready applications. It provides **auto-configuration**, an **embedded server** (Tomcat/Jetty by default), **starter dependencies**, and sensible defaults — so you can get a running app with minimal setup.

---

**Q: What are the main advantages of Spring Boot?**

- **Auto-configuration** — Spring Boot guesses what you need based on classpath and configures it automatically.
- **Embedded server** — no need to deploy WAR files; the app runs as a self-contained JAR (`java -jar`).
- **Starter dependencies** — `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, etc. bundle related dependencies together.
- **Production-ready features** — Actuator provides health checks, metrics, and monitoring out of the box.
- **Minimal boilerplate** — less XML configuration, more convention over configuration.
- **Spring ecosystem** — integrates seamlessly with Spring Security, Spring Data, Spring Cloud, etc.

---

**Q: What is the Spring Bean lifecycle?**

1. **Instantiation** — Spring creates the bean instance (calls constructor).
2. **Dependency Injection** — Spring injects dependencies (field, setter, or constructor injection).
3. **`BeanNameAware`, `BeanFactoryAware`** — Spring calls awareness interfaces if implemented.
4. **`BeanPostProcessor.postProcessBeforeInitialization`** — pre-init processing.
5. **`@PostConstruct` / `InitializingBean.afterPropertiesSet()`** — custom init logic runs here.
6. **`BeanPostProcessor.postProcessAfterInitialization`** — post-init processing.
7. **Bean is ready and in use.**
8. **`@PreDestroy` / `DisposableBean.destroy()`** — cleanup logic when the context is shutting down.
9. **Bean is destroyed.**

---

**Q: What is bean scope? What is the default?**

Bean scope defines how many instances Spring creates and manages.

| Scope | Description |
|---|---|
| **`singleton`** (default) | One instance per Spring ApplicationContext. Shared everywhere. |
| **`prototype`** | A new instance is created every time it's requested. |
| **`request`** | One instance per HTTP request. (Web apps only) |
| **`session`** | One instance per HTTP session. (Web apps only) |
| **`application`** | One instance per ServletContext. (Web apps only) |

Default is `singleton` — suitable for stateless beans. Use `prototype` when the bean holds per-use state.

---

**Q: What does `@SpringBootApplication` do?**

It is a convenience annotation that combines three annotations:
- **`@SpringBootConfiguration`** — marks the class as a configuration source (specialization of `@Configuration`).
- **`@EnableAutoConfiguration`** — tells Spring Boot to automatically configure beans based on the classpath.
- **`@ComponentScan`** — scans the current package and all sub-packages for Spring-managed components.

This single annotation on your main class bootstraps the entire application.

---

**Q: What is `@Configuration`?**

Marks a class as a source of **bean definitions**. Methods annotated with `@Bean` inside a `@Configuration` class return objects that Spring registers as beans in the application context.

```java
@Configuration
public class AppConfig {
    @Bean
    public DataSource dataSource() {
        return new HikariDataSource(...);
    }
}
```

---

**Q: What is `@ComponentScan`?**

Tells Spring where to look for components (classes annotated with `@Component`, `@Service`, `@Repository`, `@Controller`, etc.). By default, `@SpringBootApplication` scans the package of the main class and all sub-packages. You can customize it: `@ComponentScan(basePackages = "com.example")`.

---

**Q: `@Controller` vs `@RestController`?**

- **`@Controller`** — marks a class as an MVC controller. Methods typically return view names (for server-side rendering with templates like Thymeleaf). You need `@ResponseBody` on each method to return data directly.
- **`@RestController`** — shorthand for `@Controller` + `@ResponseBody` on the class. Every method returns data (JSON/XML) directly in the HTTP response body. Use this for REST APIs.

---

**Q: What is `@Service`, `@Component`, `@Repository`, and `@Bean`?**

All except `@Bean` are **stereotype annotations** — they mark classes for auto-detection via component scanning.

- **`@Component`** — generic Spring-managed component. Base annotation.
- **`@Service`** — marks business logic layer classes. Functionally same as `@Component`, but communicates intent.
- **`@Repository`** — marks DAO/data access layer classes. Additionally, Spring translates persistence exceptions (e.g., `SQLException`) into Spring's `DataAccessException` hierarchy.
- **`@Bean`** — used on a **method** inside a `@Configuration` class to explicitly declare a bean. Useful when you don't own the class (e.g., third-party libraries).

---

**Q: What does a typical Spring Boot application look like?**

```
com.example.app
├── controller/       ← @RestController — handles HTTP requests, calls service
├── service/          ← @Service — business logic, calls repository
├── repository/       ← @Repository — data access (Spring Data JPA interfaces)
├── model/            ← Entity/DTO classes
└── config/           ← @Configuration classes
```

**Flow:** HTTP Request → Controller → Service → Repository → Database → back up the chain.

Controllers should be thin (no business logic). Services contain the logic. Repositories handle persistence. This separation keeps the code testable and maintainable.

---

**Q: Multithreading issues with singleton beans — how to handle?**

Since Spring beans are singletons by default and shared across all threads handling concurrent HTTP requests, **instance variables on singleton beans create race conditions**.

**Solutions:**
1. **Prototype scope** — `@Scope("prototype")` creates a new instance per injection. Overhead of instantiation each time.
2. **Thread synchronization** — `synchronized` blocks or `ReentrantLock`. Adds contention and reduces throughput.
3. **Stateless design (best option)** — design beans with no instance state. All data flows through method parameters and return values. No shared mutable state means no thread safety issues. This is the standard approach in well-designed Spring apps.

Method-local variables are always on the stack — they are thread-safe by nature.

---

**Q: What are transactions? How do they work in Spring?**

A **transaction** is a unit of work that must be executed completely or not at all, ensuring data consistency. Transactions follow **ACID** properties:
- **Atomicity** — all or nothing.
- **Consistency** — database moves from one valid state to another.
- **Isolation** — concurrent transactions don't interfere.
- **Durability** — committed data persists.

In Spring, `@Transactional` on a method/class tells Spring to wrap that logic in a transaction. On success → commit. On unchecked exception → rollback (by default).

**Transaction Propagation** controls what happens when a transactional method calls another:

| Propagation | Behavior |
|---|---|
| `REQUIRED` (default) | Join existing transaction or create new one. |
| `REQUIRES_NEW` | Always create a new transaction; suspend current one. |
| `NESTED` | Execute within a nested transaction (savepoint). |
| `MANDATORY` | Must run within an existing transaction; else throw exception. |
| `NEVER` | Must NOT run within a transaction; else throw exception. |
| `SUPPORTS` | Run in transaction if one exists; otherwise non-transactional. |
| `NOT_SUPPORTED` | Always run non-transactionally; suspend current transaction. |

```java
@Service
public class OrderService {
    @Transactional(propagation = Propagation.REQUIRED)
    public void placeOrder(Order order) {
        // runs in a transaction
    }
}
```

---

## 6. REST APIs

**Q: What are the HTTP verbs and when do you use them?**

| Verb | Use | Idempotent? | Safe? |
|---|---|---|---|
| **GET** | Retrieve a resource. No side effects. | Yes | Yes |
| **POST** | Create a new resource. Body contains data. | No | No |
| **PUT** | Replace a resource entirely. Idempotent — calling it multiple times has the same result. | Yes | No |
| **PATCH** | Partially update a resource. Only send changed fields. | No (usually) | No |
| **DELETE** | Remove a resource. | Yes | No |

Example REST resource `/users`:
- `GET /users` — list all users
- `GET /users/42` — get user 42
- `POST /users` — create a new user
- `PUT /users/42` — replace user 42 entirely
- `PATCH /users/42` — update specific fields of user 42
- `DELETE /users/42` — delete user 42

---

**Q: What are common HTTP status codes?**

- `200 OK` — success
- `201 Created` — resource created (response to POST)
- `204 No Content` — success, no body (common for DELETE)
- `400 Bad Request` — invalid input from client
- `401 Unauthorized` — not authenticated
- `403 Forbidden` — authenticated but not authorized
- `404 Not Found` — resource doesn't exist
- `500 Internal Server Error` — unexpected server-side failure

---

**Q: What is JWT authentication and how does it work?**

**JWT (JSON Web Token)** is a compact, self-contained token used for stateless authentication. Instead of storing session state on the server, the token itself carries the user's identity and claims.

**Structure:** Three Base64-encoded parts separated by dots:
```
header.payload.signature
```
- **Header** — algorithm used (e.g., `HS256`) and token type.
- **Payload** — claims: user ID, roles, expiry time (`exp`), issuer, etc.
- **Signature** — `HMAC(base64(header) + "." + base64(payload), secret)` — proves the token hasn't been tampered with.

**Authentication flow:**
1. User sends credentials (username + password) to `POST /auth/login`.
2. Server validates credentials, generates a JWT signed with a secret key, returns it.
3. Client stores the token (localStorage or a cookie).
4. Client sends the token in the `Authorization` header on every request: `Authorization: Bearer <token>`.
5. Server verifies the signature and reads claims — no database lookup needed for session validation.
6. Token expires after a set time (`exp` claim); client must re-authenticate or use a refresh token.

**Key point:** JWTs are **stateless** — the server doesn't store session state. This is why they scale well in distributed systems. The trade-off is that tokens cannot be invalidated before expiry without extra infrastructure (a token blacklist).
