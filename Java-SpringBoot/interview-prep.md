# Java & Spring Boot — Interview Prep

## Table of Contents
- [1. OOP Concepts](#1-oop-concepts)
- [2. Multithreading](#2-multithreading)
- [3. Design Patterns](#3-design-patterns)
- [4. Dependency Injection & Autowiring](#4-dependency-injection--autowiring)
- [5. Spring Boot](#5-spring-boot)
- [6. REST APIs](#6-rest-apis)
- [7. Data Structures](#7-data-structures)
- [8. JUnit & Testing](#8-junit--testing)

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

**Q: What exactly is an interface in Java? What can it contain?**

An interface is a **contract** — it defines *what* a class can do, without specifying *how*. Any class that `implements` the interface must provide implementations for all its abstract methods (unless the class is itself abstract).

Before Java 8, interfaces could only have abstract method signatures and `public static final` constants. Modern Java has expanded this significantly:

```java
public interface PaymentProcessor {

    // Abstract method — every implementor must provide this
    void process(Payment payment);

    // Default method (Java 8+) — has a body, inherited unless overridden
    default void validate(Payment payment) {
        if (payment.getAmount() <= 0) throw new IllegalArgumentException("Invalid amount");
    }

    // Static method (Java 8+) — belongs to the interface, not the instance
    static PaymentProcessor noOp() {
        return payment -> {};  // factory helper
    }

    // Constant — implicitly public static final
    int MAX_RETRY = 3;

    // Private method (Java 9+) — shared helper for default methods, not exposed
    private void log(String msg) {
        System.out.println("[Payment] " + msg);
    }
}
```

A class can implement multiple interfaces — this is Java's answer to multiple inheritance:

```java
public class StripeProcessor implements PaymentProcessor, Auditable, Retryable {
    @Override
    public void process(Payment payment) {
        // Stripe-specific implementation
    }
}
```

---

**Q: What is the difference between an interface and an abstract class — and when do you choose each?**

| Feature | Interface | Abstract Class |
|---|---|---|
| Instance fields (state) | No — only `static final` constants | Yes — can have any fields |
| Constructor | No | Yes |
| Method types | `abstract`, `default`, `static`, `private` | Any — abstract or concrete |
| Multiple inheritance | Yes — a class can implement many | No — a class can extend only one |
| Access modifiers on methods | Implicitly `public` (before Java 9) | Any modifier |
| `IS-A` relationship | Describes a capability or role | Describes a type with shared base |
| Instantiatable | No | No |

**When to use an interface:**
- You want to define a **capability or role** that unrelated classes can share.
- You need **multiple inheritance** of type.
- You want to define a **contract** that any type can fulfill, regardless of its class hierarchy.

```java
// Unrelated classes sharing a capability — interface is right
class Dog implements Swimmable, Runnable { }
class Boat implements Swimmable { }
// Dog and Boat share no meaningful base class, but both can swim
```

**When to use an abstract class:**
- You have a **family of closely related classes** that share state and behavior.
- You want to provide a **partial implementation** that subclasses build on.
- You need **protected or package-private** members (interfaces can't have these pre-Java 9).
- You need a **constructor** to enforce initialization logic.

```java
abstract class Animal {
    protected String name;   // shared state — not possible in interface
    protected int age;

    Animal(String name, int age) {   // constructor — interface can't have this
        this.name = name;
        this.age = age;
    }

    abstract void speak();           // each animal speaks differently

    void breathe() {                 // shared behavior — all animals breathe the same way
        System.out.println(name + " breathes");
    }
}

class Dog extends Animal {
    Dog(String name, int age) { super(name, age); }

    @Override
    void speak() { System.out.println(name + " barks"); }
}
```

**The practical rule of thumb:** default to an **interface**. Switch to an abstract class only when you genuinely need shared state (instance fields) or a constructor.

---

**Q: How do interfaces enable polymorphism in practice? Why is this powerful?**

The real power of interfaces is that code depending on an interface doesn't care what the concrete type is — you can swap implementations without changing the consumer.

```java
// Define the contract
public interface NotificationService {
    void send(String recipient, String message);
}

// Multiple implementations
@Service("emailNotification")
public class EmailNotificationService implements NotificationService {
    public void send(String recipient, String message) {
        // send via email
    }
}

@Service("smsNotification")
public class SmsNotificationService implements NotificationService {
    public void send(String recipient, String message) {
        // send via SMS
    }
}

// Consumer depends on the interface — not the implementation
@Service
public class OrderService {
    private final NotificationService notificationService;

    public OrderService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    public void placeOrder(Order order) {
        // ... business logic ...
        notificationService.send(order.getEmail(), "Order confirmed!");
        // doesn't know or care if this is email, SMS, or push
    }
}
```

In Spring, you swap the implementation by changing which bean is injected (via `@Qualifier` or `@Primary`) — the `OrderService` code doesn't change at all. This is the **Open/Closed Principle**: open for extension (new `NotificationService` implementation), closed for modification (no changes to `OrderService`).

This pattern also makes testing trivial — inject a mock implementation of the interface in unit tests, no real email sent.

---

**Q: Can an interface extend another interface? Can a class implement multiple interfaces that have the same default method?**

Yes — interfaces can extend other interfaces (and multiple at once):

```java
public interface Readable { String read(); }
public interface Writable { void write(String data); }
public interface ReadWritable extends Readable, Writable { }  // inherits both contracts

class FileHandler implements ReadWritable {
    public String read() { return "data"; }
    public void write(String data) { /* write */ }
}
```

**Diamond problem with default methods:** if two interfaces provide a `default` method with the same signature, the implementing class gets a compile error and must override to resolve the conflict:

```java
interface A { default void greet() { System.out.println("Hello from A"); } }
interface B { default void greet() { System.out.println("Hello from B"); } }

class C implements A, B {
    @Override
    public void greet() {
        A.super.greet();   // explicitly choose A's version, or write your own
    }
}
```

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

---

## 7. Data Structures

**Q: What is an Array and when do you use it?**

An array is a fixed-size, contiguous block of memory holding elements of the same type. Size is set at creation and cannot change.

```java
int[] nums = new int[5];
int[] primes = {2, 3, 5, 7, 11};

primes[0];       // O(1) — direct index access
primes[2] = 99;  // O(1) — direct write
```

| Operation | Time Complexity |
|---|---|
| Access by index | O(1) |
| Search (unsorted) | O(n) |
| Insert / Delete (middle) | O(n) — must shift elements |

**Use an array when:**
- Size is known and fixed upfront.
- You need maximum memory efficiency (no overhead per element).
- You're doing index-heavy access patterns (e.g., lookup tables, matrix math).

In practice, `ArrayList` is used far more often in Java because it's dynamic.

---

**Q: What is a List? `ArrayList` vs `LinkedList` — which should you use?**

`List` is an ordered, index-accessible collection that allows duplicates. The two main implementations have very different internal structures.

**`ArrayList`** — backed by a resizable array. Elements stored contiguously in memory.

**`LinkedList`** — backed by a doubly-linked list. Each element (node) holds a value and pointers to the previous and next node.

| Operation | ArrayList | LinkedList |
|---|---|---|
| Access by index (`get(i)`) | O(1) | O(n) — must traverse |
| Add/remove at end | O(1) amortized | O(1) |
| Add/remove at beginning/middle | O(n) — shifts elements | O(1) — just relink pointers |
| Memory | Less (contiguous array) | More (two pointers per node) |
| Cache performance | Excellent (contiguous) | Poor (scattered in heap) |

**Default choice: `ArrayList`.** It wins in the vast majority of use cases because index access is fast and cache locality makes iteration faster than it looks on paper. `LinkedList` is theoretically better for frequent insertions at the head, but its cache-unfriendly memory layout usually makes `ArrayList` faster even then. Use `LinkedList` only when you have profiled and confirmed it's better, or when you specifically need the `Deque` interface.

```java
List<String> list = new ArrayList<>();   // almost always this
list.add("Alice");
list.add("Bob");
list.get(0);        // "Alice" — O(1)
list.remove(0);     // shifts remaining elements
```

---

**Q: Array vs List — deep comparison. When do you actually reach for one over the other?**

This is one of those questions where the short answer ("use List, it's dynamic") hides important nuance.

**Memory and type differences**

A `int[]` stores raw primitive integers packed tightly in memory — no object overhead, no boxing. An `ArrayList<Integer>` stores references to `Integer` objects on the heap — each element is a boxed object with its own header (~16 bytes) plus the reference itself (4–8 bytes). For a million integers:

- `int[]` → ~4 MB
- `ArrayList<Integer>` → ~20+ MB + GC pressure

This matters in performance-critical or memory-constrained code.

```java
int[] primitiveArray = new int[1_000_000];        // ~4 MB, zero boxing
List<Integer> boxedList = new ArrayList<>(1_000_000); // ~20+ MB, every element a heap object
```

**Resizability**

Arrays are fixed-size — you declare the size at creation and it never changes. If you need more room, you allocate a new larger array and copy everything over (which is exactly what `ArrayList` does internally when it grows).

`ArrayList` starts with a default capacity of 10 and doubles when full. You can pre-size it to avoid repeated resizing:

```java
// If you know you'll have ~500 elements, pre-size to avoid 5-6 resize operations
List<String> names = new ArrayList<>(500);
```

**Multi-dimensional data**

Arrays are natural for matrices and grids. A `int[][]` is a true 2D structure. Doing the same with nested lists is verbose and slower:

```java
// Clean and efficient
int[][] matrix = new int[3][3];
matrix[1][2] = 42;

// Verbose with lists
List<List<Integer>> matrix = new ArrayList<>();
```

**Type safety and covariance — a tricky difference**

Arrays are **covariant** — `String[]` is a subtype of `Object[]`. This can cause runtime errors:

```java
String[] strings = new String[3];
Object[] objects = strings;       // compiles fine — covariant
objects[0] = 42;                  // compiles fine — then throws ArrayStoreException at runtime

// Generics are invariant — this fails at compile time (safer)
List<String> strList = new ArrayList<>();
List<Object> objList = strList;   // compile error — caught early
```

**API richness**

`List` (and the Collections API generally) comes with a huge toolkit: `Collections.sort()`, `Collections.shuffle()`, `stream()`, `forEach()`, `removeIf()`, `subList()`. Arrays need `Arrays.sort()`, `Arrays.stream()`, and manual loops for most operations. In practice, the Collections API makes `List` dramatically more ergonomic.

```java
List<String> names = new ArrayList<>(List.of("Charlie", "Alice", "Bob"));
Collections.sort(names);
names.removeIf(n -> n.startsWith("C"));
names.stream().map(String::toUpperCase).forEach(System.out::println);
```

**Null handling**

Both `int[]` and `ArrayList` have gotchas:

```java
int[] arr = new int[5];   // all elements default to 0
String[] sarr = new String[5]; // all elements default to null — NullPointerException if not set

List<String> list = new ArrayList<>();
list.get(0);  // IndexOutOfBoundsException — list is empty, not null-filled
```

**Decision guide — array vs list:**

| Situation | Use |
|---|---|
| Size is fixed and known | Array — simpler, less overhead |
| Primitive types, performance-critical, large dataset | Array — no boxing, better memory |
| Multi-dimensional grid / matrix | `int[][]` array |
| Size changes dynamically | `ArrayList` |
| Need Collections API (`sort`, `stream`, `removeIf`) | `ArrayList` |
| Passing to legacy or low-level APIs | May require array — `toArray()` converts |
| General purpose, everyday code | `ArrayList` — default choice |

**Converting between them:**

```java
// Array → List (fixed-size view, no add/remove)
String[] arr = {"a", "b", "c"};
List<String> fixed = Arrays.asList(arr);       // backed by array — no structural changes
List<String> mutable = new ArrayList<>(Arrays.asList(arr));  // fully mutable copy

// List → Array
String[] back = list.toArray(new String[0]);   // pass typed array

// Primitive array → stream
int[] nums = {1, 2, 3};
IntStream stream = Arrays.stream(nums);
int sum = Arrays.stream(nums).sum();
```

---

**Q: What is a Stack and when do you use it?**

A Stack is a **Last-In, First-Out (LIFO)** structure — the last element pushed is the first one popped. Think of a stack of plates.

In Java, avoid the legacy `Stack` class (extends `Vector`, thread-synchronized, slow). Use `ArrayDeque` instead:

```java
Deque<Integer> stack = new ArrayDeque<>();

stack.push(1);    // [1]
stack.push(2);    // [2, 1]
stack.push(3);    // [3, 2, 1]

stack.peek();     // 3  — look at top without removing
stack.pop();      // 3  — removes and returns top → [2, 1]
```

| Operation | Time Complexity |
|---|---|
| push / pop / peek | O(1) |

**Use a Stack when:**
- You need to reverse something (push all, then pop all).
- Tracking "undo" history — each action is pushed; undo pops the last.
- Parsing balanced brackets / expressions — push opens, pop on close.
- DFS (depth-first search) — either explicit stack or recursion (call stack).
- Backtracking algorithms.

---

**Q: What is a Queue and when do you use it?**

A Queue is a **First-In, First-Out (FIFO)** structure — elements are added at the back and removed from the front. Think of a line at a checkout.

In Java, use `ArrayDeque` for a plain queue, or `LinkedList` when you need the `Queue` interface explicitly:

```java
Queue<String> queue = new ArrayDeque<>();

queue.offer("Alice");   // enqueue — ["Alice"]
queue.offer("Bob");     // ["Alice", "Bob"]
queue.offer("Carol");   // ["Alice", "Bob", "Carol"]

queue.peek();    // "Alice" — front without removing
queue.poll();    // "Alice" — removes and returns front → ["Bob", "Carol"]
```

| Operation | Time Complexity |
|---|---|
| offer (enqueue) / poll (dequeue) / peek | O(1) |

**Use a Queue when:**
- Processing tasks in the order they arrived (job queues, request buffers).
- BFS (breadth-first search) — process level by level.
- Rate limiting — hold requests in a queue.
- Producer-consumer patterns (`BlockingQueue` variants in `java.util.concurrent`).

**`PriorityQueue`** — a special queue where each element has a priority and the element with the highest priority (smallest by default) is dequeued first. Backed by a min-heap. Useful for scheduling and Dijkstra's algorithm.

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.offer(5); pq.offer(1); pq.offer(3);
pq.poll();  // 1 — smallest first
```

**`ArrayDeque` as both Stack and Queue:** `ArrayDeque` implements both `Stack` and `Queue` operations efficiently. It's the recommended replacement for both `Stack` and `LinkedList` as a queue.

---

**Q: What is a Tree? What is a Binary Search Tree?**

A **tree** is a hierarchical data structure with a root node, where each node has zero or more child nodes. No cycles — each node has exactly one parent (except the root).

A **Binary Search Tree (BST)** is a binary tree (max 2 children per node) where:
- Left subtree contains only values **less than** the node.
- Right subtree contains only values **greater than** the node.

```
       8
      / \
     3   10
    / \    \
   1   6    14
      / \
     4   7
```

| Operation | Average (balanced) | Worst (skewed) |
|---|---|---|
| Search | O(log n) | O(n) |
| Insert | O(log n) | O(n) |
| Delete | O(log n) | O(n) |

A skewed BST (e.g., inserting already-sorted data) degrades to a linked list. **Balanced BST variants** (AVL tree, Red-Black tree) self-balance to guarantee O(log n). Java's `TreeMap` and `TreeSet` are backed by a Red-Black tree.

**Common tree types and uses:**

| Tree Type | Used in |
|---|---|
| Binary Search Tree | Ordered data, range queries |
| Red-Black Tree | `TreeMap`, `TreeSet` in Java |
| Heap (complete binary tree) | `PriorityQueue` — O(1) max/min access |
| Trie (prefix tree) | Autocomplete, spell check |
| B-Tree / B+ Tree | Database indexes, filesystems |

**When to use a tree:**
- You need **sorted order** plus O(log n) search/insert/delete (`TreeMap`, `TreeSet`).
- You need range queries (find all keys between A and B).
- Hierarchical data (file systems, org charts, XML/JSON parsing).

---

**Q: What is a HashMap? How does it work internally?**

A `HashMap` stores **key-value pairs** with O(1) average-case get and put. Internally, it uses an array of "buckets." The key's `hashCode()` determines which bucket the entry goes into.

```java
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 95);
scores.put("Bob", 87);

scores.get("Alice");           // 95 — O(1) average
scores.containsKey("Carol");   // false
scores.getOrDefault("Carol", 0); // 0

// Iterate entries
for (Map.Entry<String, Integer> entry : scores.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

**How it works internally:**
1. `key.hashCode()` → hashed → maps to a bucket index.
2. If the bucket is empty → store the entry there.
3. If the bucket is occupied (**hash collision**) → entries are stored in a linked list (or a balanced tree in Java 8+ when the list gets long).
4. On `get`, hash the key, find the bucket, then check `equals()` to find the exact entry.

This is why **both `hashCode()` and `equals()` must be implemented correctly** on keys. Two keys that are `equals()` must have the same `hashCode()`. If you override `equals()` without overriding `hashCode()`, lookups will break.

**Key `Map` implementations:**

| | `HashMap` | `LinkedHashMap` | `TreeMap` |
|---|---|---|---|
| Order | None | Insertion order | Sorted by key |
| Performance | O(1) avg | O(1) avg | O(log n) |
| Use when | Fast lookup, order irrelevant | Need insertion order (LRU cache) | Need sorted keys or range queries |

---

**Q: What is a Set? `HashSet` vs `TreeSet` vs `LinkedHashSet`?**

A `Set` is a collection of **unique elements** — no duplicates allowed. Adding a duplicate is silently ignored.

```java
Set<String> set = new HashSet<>();
set.add("apple");
set.add("banana");
set.add("apple");   // duplicate — ignored
set.size();         // 2

set.contains("banana");  // true — O(1)
```

| | `HashSet` | `LinkedHashSet` | `TreeSet` |
|---|---|---|---|
| Backed by | `HashMap` | `LinkedHashMap` | `TreeMap` (Red-Black tree) |
| Order | None | Insertion order | Sorted (natural or `Comparator`) |
| `contains` / `add` / `remove` | O(1) avg | O(1) avg | O(log n) |
| Use when | Fast membership test, order irrelevant | Unique + insertion order | Unique + sorted order |

---

**Q: Set vs List — when do you use one over the other?**

| | `List` | `Set` |
|---|---|---|
| Duplicates | Allowed | Not allowed |
| Order | Insertion order maintained | Depends on implementation |
| Access by index | Yes — `get(i)` | No |
| `contains()` performance | O(n) for `ArrayList` | O(1) for `HashSet` |
| Use when | Order matters, duplicates valid, index access needed | Uniqueness required, fast membership test |

**Choose `List` when:**
- You care about order or position.
- Duplicates are valid (e.g., a shopping cart with two of the same item).
- You need index-based access.

**Choose `Set` when:**
- You need to eliminate duplicates (e.g., unique visitors, distinct tags).
- You need to check membership frequently and performance matters (`contains` is O(1) vs O(n)).
- You're computing set operations: union (`addAll`), intersection (`retainAll`), difference (`removeAll`).

```java
// Fast deduplication
List<String> withDupes = List.of("a", "b", "a", "c", "b");
Set<String> unique = new HashSet<>(withDupes);   // {"a", "b", "c"}

// Fast membership test — prefer Set over List
Set<String> validRoles = Set.of("ADMIN", "USER", "MODERATOR");
if (validRoles.contains(userRole)) { ... }   // O(1)
// vs List.contains() → O(n) — iterates every element
```

---

**Q: Quick reference — which data structure should I reach for?**

| Need | Use |
|---|---|
| Ordered list, index access, duplicates OK | `ArrayList` |
| Fast lookup by key | `HashMap` |
| Unique elements, fast membership test | `HashSet` |
| Sorted unique elements | `TreeSet` |
| Sorted key-value map | `TreeMap` |
| Preserve insertion order, unique | `LinkedHashSet` |
| LIFO (undo, parsing, DFS) | `ArrayDeque` as stack |
| FIFO (task queue, BFS) | `ArrayDeque` as queue |
| Priority ordering (scheduling) | `PriorityQueue` |
| Thread-safe key-value map | `ConcurrentHashMap` |
| Hierarchical / sorted with range queries | `TreeMap` / `TreeSet` |

---

## 8. JUnit & Testing

**Q: What is JUnit and why is testing important?**

JUnit is the standard unit testing framework for Java. A **unit test** verifies a single unit of code (usually one method or class) in isolation — fast, deterministic, no external dependencies like databases or network calls.

Testing matters because:
- **Catches regressions** — a change that breaks existing behaviour fails immediately, not in production.
- **Documents intent** — a test shows exactly what a method is supposed to do.
- **Enables confident refactoring** — if all tests pass after a change, behaviour is preserved.
- **Forces good design** — code that is hard to test is usually tightly coupled or has too many responsibilities.

JUnit 5 (Jupiter) is the current version. It ships as three modules: `junit-jupiter-api` (annotations/assertions), `junit-jupiter-engine` (runs tests), `junit-jupiter-params` (parameterised tests).

---

**Q: What are the core JUnit 5 annotations?**

| Annotation | Purpose |
|---|---|
| `@Test` | Marks a method as a test case |
| `@BeforeEach` | Runs before every test method — set up fresh state |
| `@AfterEach` | Runs after every test method — clean up |
| `@BeforeAll` | Runs once before all tests in the class — must be `static` |
| `@AfterAll` | Runs once after all tests — must be `static` |
| `@DisplayName` | Human-readable test name shown in reports |
| `@Disabled` | Skip a test (with a reason) |
| `@Nested` | Group related tests in an inner class |
| `@ParameterizedTest` | Run the same test with multiple inputs |
| `@ValueSource` | Supply primitive/string values to a parameterised test |
| `@MethodSource` | Supply values from a factory method |
| `@ExtendWith` | Register extensions (e.g., Mockito, Spring) |

```java
@DisplayName("OrderService tests")
class OrderServiceTest {

    private OrderService orderService;

    @BeforeEach
    void setUp() {
        orderService = new OrderService();   // fresh instance before each test
    }

    @Test
    @DisplayName("should calculate total correctly")
    void shouldCalculateTotal() {
        double total = orderService.calculateTotal(List.of(10.0, 20.0, 5.0));
        assertEquals(35.0, total);
    }

    @Test
    @Disabled("price calculation not implemented yet")
    void shouldApplyDiscount() { }
}
```

---

**Q: What are the main JUnit 5 assertions?**

Assertions are in `org.junit.jupiter.api.Assertions`. If an assertion fails, the test fails with a clear message.

```java
assertEquals(expected, actual);
assertEquals(3.14, result, 0.001);         // with delta for floating point
assertNotEquals(unexpected, actual);

assertTrue(condition);
assertFalse(condition);

assertNull(object);
assertNotNull(object);

assertThrows(IllegalArgumentException.class, () -> service.method(null));

assertAll(                                 // run all assertions even if some fail
    () -> assertEquals("Alice", user.getName()),
    () -> assertEquals(30, user.getAge()),
    () -> assertNotNull(user.getEmail())
);

// Custom failure message (message first in JUnit 5)
assertEquals(5, list.size(), "List should contain 5 elements after add");
```

**`assertThrows` — testing expected exceptions:**
```java
@Test
void shouldThrowWhenAmountIsNegative() {
    IllegalArgumentException ex = assertThrows(
        IllegalArgumentException.class,
        () -> paymentService.process(-50)
    );
    assertTrue(ex.getMessage().contains("negative"));
}
```

---

**Q: What is Mockito and why is it used alongside JUnit?**

Mockito is a mocking framework. When unit testing a class that depends on other classes (like a Service that calls a Repository), you don't want the real dependencies — they'd hit a database, make HTTP calls, etc. Mockito lets you create **fakes (mocks)** that you control.

```java
@ExtendWith(MockitoExtension.class)          // activates Mockito in JUnit 5
class UserServiceTest {

    @Mock
    private UserRepository userRepository;   // Mockito creates a fake implementation

    @InjectMocks
    private UserService userService;         // injects the mock into this

    @Test
    void shouldReturnUserById() {
        User fakeUser = new User(1L, "Alice");
        when(userRepository.findById(1L)).thenReturn(Optional.of(fakeUser));

        User result = userService.getUser(1L);

        assertEquals("Alice", result.getName());
        verify(userRepository).findById(1L);  // assert the method was actually called
    }

    @Test
    void shouldThrowWhenUserNotFound() {
        when(userRepository.findById(99L)).thenReturn(Optional.empty());

        assertThrows(UserNotFoundException.class, () -> userService.getUser(99L));
    }
}
```

**Key Mockito concepts:**

| Concept | Description |
|---|---|
| `@Mock` | Creates a mock object (all methods return defaults unless stubbed) |
| `@InjectMocks` | Creates the class under test and injects `@Mock` fields into it |
| `@Spy` | Wraps a real object — real methods run unless you stub them |
| `@Captor` | Captures arguments passed to a mock for assertion |
| `when(...).thenReturn(...)` | Stub — define what a mock returns when called |
| `when(...).thenThrow(...)` | Stub a method to throw an exception |
| `verify(mock).method(args)` | Assert the method was called with those arguments |
| `verify(mock, times(2)).method()` | Assert called exactly twice |
| `verify(mock, never()).method()` | Assert never called |
| `ArgumentMatchers.any()` | Match any argument: `when(repo.findById(any()))` |

---

**Q: What is the difference between a mock, a stub, a spy, and a fake?**

These terms are often used loosely but have distinct meanings:

- **Mock** — a fully fake object. All methods return defaults (null, 0, false) unless you stub them. You typically verify that certain methods were called. Created by `@Mock`.
- **Stub** — a fake that returns predetermined responses. Focus is on controlling input/output, not on verifying calls. In Mockito, `when(...).thenReturn(...)` turns a mock into a stub for that method.
- **Spy** — wraps a real object. Real methods run unless you stub specific ones. Useful when you want most real behaviour but need to control one or two methods. Created by `@Spy`.
- **Fake** — a lightweight working implementation of a dependency (e.g., an in-memory repository instead of a real DB). Written by hand, not generated by Mockito.

```java
// Spy — real ArrayList, but we can stub/verify specific calls
@Spy
List<String> spyList = new ArrayList<>();

@Test
void spyExample() {
    spyList.add("Alice");          // real add — actually adds to the list
    spyList.add("Bob");

    assertEquals(2, spyList.size());  // real size

    doReturn(100).when(spyList).size();  // now stub size()
    assertEquals(100, spyList.size());   // returns stubbed value
}
```

---

**Q: How do you test a Spring Boot application? What is `@SpringBootTest` vs slice tests?**

Spring Boot provides testing support that integrates with JUnit 5. There are two broad strategies:

**Full integration test — `@SpringBootTest`:**
Loads the complete Spring application context. Slow but tests real wiring.

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void shouldReturnUser() throws Exception {
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Alice"));
    }
}
```

**Slice tests — test only one layer:**
Much faster — loads only the beans relevant to that slice.

| Annotation | What it loads | Use for |
|---|---|---|
| `@WebMvcTest(Controller.class)` | Web layer only (Controller, filters, security) | Testing controller logic, request mapping, validation |
| `@DataJpaTest` | JPA layer only (repositories, in-memory H2 DB) | Testing repository queries |
| `@JsonTest` | JSON serialization/deserialization only | Testing `@JsonComponent`, ObjectMapper config |
| `@MockBean` | Adds a Mockito mock to the Spring context | Replace a real bean with a mock in slice tests |

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;           // mock the service — don't load real one

    @Test
    void shouldReturn404WhenUserNotFound() throws Exception {
        when(userService.getUser(99L)).thenThrow(new UserNotFoundException("not found"));

        mockMvc.perform(get("/api/users/99"))
            .andExpect(status().isNotFound());
    }
}
```

---

**Q: What makes a good unit test? What is the AAA pattern?**

**AAA — Arrange, Act, Assert** is the standard structure for a readable test:

```java
@Test
void shouldApplyDiscountForPremiumUsers() {
    // Arrange — set up inputs and dependencies
    User premiumUser = new User("Alice", UserType.PREMIUM);
    PricingService pricingService = new PricingService();

    // Act — call the method under test
    double discountedPrice = pricingService.getPrice(100.0, premiumUser);

    // Assert — verify the outcome
    assertEquals(80.0, discountedPrice, "Premium users should get 20% discount");
}
```

**Characteristics of a good unit test (FIRST):**
- **Fast** — runs in milliseconds. No I/O, no network, no database.
- **Isolated** — tests one thing. Fails only when that one thing is broken.
- **Repeatable** — same result every time, regardless of order or environment.
- **Self-validating** — pass or fail without manual inspection of output.
- **Timely** — written alongside or before the code (TDD), not long after.

**What to avoid:**
- Multiple unrelated assertions in one test (hard to diagnose which failed).
- Tests that depend on execution order.
- Logic in tests (if/else, loops) — tests should be dumb and obvious.
- Testing implementation details — test *what* a method does, not *how*.
- Ignoring edge cases: null inputs, empty collections, boundary values (0, -1, max).

---

**Q: What is Test-Driven Development (TDD)?**

TDD is a development practice where you write the test **before** writing the implementation. The cycle is:

1. **Red** — write a test for behaviour that doesn't exist yet. It fails.
2. **Green** — write the minimum code to make the test pass.
3. **Refactor** — clean up the code while keeping the tests green.

```java
// Step 1 — Red: write the test first
@Test
void shouldReturnFizzForMultiplesOfThree() {
    assertEquals("Fizz", FizzBuzz.evaluate(3));   // FizzBuzz class doesn't exist yet
}

// Step 2 — Green: write just enough to pass
class FizzBuzz {
    static String evaluate(int n) {
        if (n % 3 == 0) return "Fizz";
        return String.valueOf(n);
    }
}

// Step 3 — Refactor: add more cases, keep tests green
```

**Benefits:** forces you to think about the interface before the implementation, guarantees all code has test coverage, produces a test suite that documents every requirement.
