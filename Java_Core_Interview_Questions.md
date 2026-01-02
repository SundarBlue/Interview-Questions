# Java Core & Advanced Interview Questions

---

## **Core Java and Interfaces**

### Q1. In Java 8, if two interfaces A and B have the same default method and a class implements both, how does JVM know which default method to call?

**Concept:** This is the **diamond problem** in Java.

**Answer:** The compiler throws an error and forces the class to override that method and explicitly choose which interface implementation to call using `InterfaceA.super.methodName()` or `InterfaceB.super.methodName()`.

**Example:**
```java
interface InterfaceA {
    default void display() {
        System.out.println("Display from Interface A");
    }
}

interface InterfaceB {
    default void display() {
        System.out.println("Display from Interface B");
    }
}

// ❌ This will cause a compile error
class MyClass implements InterfaceA, InterfaceB {
    // Error: MyClass inherits unrelated defaults for display() from types InterfaceA and InterfaceB
}

// ✅ Correct way - override and choose explicitly
class MyClass implements InterfaceA, InterfaceB {
    @Override
    public void display() {
        InterfaceA.super.display(); // Explicitly choose Interface A's method
        // OR
        // InterfaceB.super.display(); // Explicitly choose Interface B's method
    }
}
```

---

### Q2. Why were private methods introduced in interfaces?

**Answer:** Private methods (from Java 9) help refactor and reuse common logic shared by multiple default/static methods inside the same interface, improving code reusability, readability, and maintainability while hiding helper logic from outside.

**Example:**
```java
interface Calculator {
    
    // Private helper method (Java 9+)
    private int add(int a, int b) {
        return a + b;
    }
    
    // Default methods can use private method
    default int addAndPrint(int a, int b) {
        int result = add(a, b); // Reuse common logic
        System.out.println("Result: " + result);
        return result;
    }
    
    default int addThreeNumbers(int a, int b, int c) {
        return add(add(a, b), c); // Reuse common logic
    }
}
```

**Benefits:**
- ✅ Code reusability within the interface
- ✅ Avoid code duplication
- ✅ Hide implementation details from implementers

---

### Q3. When should we use default methods vs static methods in interfaces?

**Answer:**

| Feature | Default Methods | Static Methods |
|---------|----------------|----------------|
| **Purpose** | Provide backward-compatible behavior | Provide utility behavior |
| **Can be Overridden** | ✅ Yes | ❌ No |
| **Accessed Via** | Instance (`obj.method()`) | Interface name (`Interface.method()`) |
| **Use Case** | When implementing classes may want to override | When logic is fixed and shouldn't change |

**Example:**
```java
interface Vehicle {
    // Default method - can be overridden
    default void start() {
        System.out.println("Vehicle starting with default behavior");
    }
    
    // Static method - cannot be overridden
    static void service() {
        System.out.println("Standard service procedure (cannot be changed)");
    }
}

class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car starting with custom behavior"); // Override allowed
    }
}

// Usage
Car car = new Car();
car.start(); // "Car starting with custom behavior"
Vehicle.service(); // "Standard service procedure" (called via interface name)
```

**Use default methods:** To provide backward-compatible behavior so implementing classes get a default implementation but can override it if needed.

**Use static methods:** For utility-like behavior that cannot be overridden and is called via `InterfaceName.methodName()`.

---

### Q4. What is a varargs (variable argument) in Java?

**Answer:** It allows a method to accept a variable number of arguments of the same type using `type... param`, which is treated as an array inside the method and can be iterated (e.g., with a for-each loop).

**Example:**
```java
public class VarargsExample {
    
    // Varargs method
    public static int sum(int... numbers) {
        int total = 0;
        for (int num : numbers) {
            total += num;
        }
        return total;
    }
    
    public static void main(String[] args) {
        System.out.println(sum(1, 2));           // 3
        System.out.println(sum(1, 2, 3, 4, 5));  // 15
        System.out.println(sum());               // 0 (no arguments)
    }
}
```

**Rules:**
- ✅ Only one varargs parameter per method
- ✅ Must be the last parameter
- ❌ Cannot have multiple varargs: `void method(int... a, String... b)` // Error

---

### Q5. In `public static void main(String[] args)`, can we replace `String[]` with `String...`?

**Answer:** Technically yes; it compiles and runs. But Java specs recommend using exactly `String[] args` for compatibility with the Java runtime and tools, so `String...` in main is not considered a good practice.

**Example:**
```java
// ✅ Standard way (recommended)
public static void main(String[] args) {
    System.out.println("Standard main method");
}

// ⚠️ Works but not recommended
public static void main(String... args) {
    System.out.println("Varargs main method");
}
```

---

### Q6. If we have a class Test with a static method and call it via a null reference (`Test obj = null; obj.staticMethod()`), what happens?

**Answer:** It works fine. Static methods belong to the class, not the object instance, so the call is resolved at compile time based on the class, even if the reference is null.

**Example:**
```java
class Test {
    public static void staticMethod() {
        System.out.println("Static method called!");
    }
}

public class Main {
    public static void main(String[] args) {
        Test obj = null;
        obj.staticMethod(); // ✅ Works! Prints "Static method called!"
        
        // Reason: Resolved at compile time as Test.staticMethod()
        // Equivalent to:
        Test.staticMethod(); // This is the correct way
    }
}
```

**Why it works:** Static methods are bound to the class at compile time, not the object at runtime.

---

## **equals–hashCode and OOP Basics**

### Q7. What is the equals and hashCode contract?

**Answer:** If `equals()` is overridden, `hashCode()` must also be overridden. Collections like `HashMap`, `HashSet`, etc., rely on consistent behavior between the two for correct bucket placement and equality checks.

**Contract Rules:**
1. If `obj1.equals(obj2)` is true, then `obj1.hashCode() == obj2.hashCode()` must be true
2. If `obj1.hashCode() == obj2.hashCode()`, it doesn't necessarily mean `obj1.equals(obj2)` (hash collisions are allowed)
3. If `equals()` is not overridden, don't override `hashCode()`

**Example:**
```java
class Student {
    private int rollNo;
    private String name;
    
    public Student(int rollNo, String name) {
        this.rollNo = rollNo;
        this.name = name;
    }
    
    // ✅ Correct way - override both
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Student student = (Student) obj;
        return rollNo == student.rollNo;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(rollNo); // Must be consistent with equals
    }
}
```

---

### Q8. Why is overriding only equals not enough? Give a scenario.

**Answer:** Example: a `Student` class with `rollNo` and `name` where `equals()` is overridden based on `rollNo` but `hashCode()` is not. Then two logically equal students may get different hash codes (default from `Object`), end up in different buckets in a hash-based collection, and be treated as different, breaking logical equality.

**Problem Scenario:**
```java
class Student {
    private int rollNo;
    private String name;
    
    public Student(int rollNo, String name) {
        this.rollNo = rollNo;
        this.name = name;
    }
    
    // ❌ Only equals overridden, hashCode not overridden
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Student student = (Student) obj;
        return rollNo == student.rollNo;
    }
    
    // hashCode() NOT overridden - uses Object's default (memory address)
}

public class Main {
    public static void main(String[] args) {
        Student s1 = new Student(101, "John");
        Student s2 = new Student(101, "John");
        
        System.out.println(s1.equals(s2)); // ✅ true (equals works)
        
        // ❌ Problem with HashSet
        Set<Student> set = new HashSet<>();
        set.add(s1);
        set.add(s2);
        System.out.println(set.size()); // ❌ 2 (Expected 1!)
        
        // Reason: Different hashCodes → Different buckets → Treated as different objects
    }
}
```

**Solution: Override both**
```java
@Override
public int hashCode() {
    return Objects.hash(rollNo); // Now both students have same hashCode
}
```

---

### Q9. Difference between a base class and an abstract class.

**Answer:**

| Feature | Base (Normal) Class | Abstract Class |
|---------|-------------------|----------------|
| **Can be instantiated** | ✅ Yes | ❌ No |
| **Abstract methods** | ❌ No | ✅ Yes (can have) |
| **Purpose** | General reusable class | Meant to be subclassed |
| **Concrete methods** | ✅ Yes | ✅ Yes (can have) |

**Example:**
```java
// Base class - can be instantiated
class Vehicle {
    public void start() {
        System.out.println("Vehicle starting");
    }
}

// Abstract class - cannot be instantiated
abstract class Animal {
    // Abstract method - no implementation
    abstract void makeSound();
    
    // Concrete method - has implementation
    public void sleep() {
        System.out.println("Sleeping...");
    }
}

class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Woof!");
    }
}

// Usage
Vehicle v = new Vehicle(); // ✅ OK - base class
// Animal a = new Animal(); // ❌ Error - abstract class cannot be instantiated
Dog dog = new Dog(); // ✅ OK - concrete subclass
```

**A base (normal) class** can be instantiated and its methods/fields can be inherited by subclasses.

**An abstract class** cannot be instantiated directly, is meant to be subclassed, and can contain abstract methods (no implementation) as well as concrete methods.

---

## **Marker Interfaces, volatile, Functional Interfaces**

### Q10. What is a marker interface in Java? Examples.

**Answer:** An interface with no methods or fields used to "mark" classes so JVM or frameworks can treat them specially. Examples: `Serializable`, `Cloneable`; JVM interprets them to allow serialization or cloning, and throws exceptions if cloning is attempted without `Cloneable`.

**Example:**
```java
import java.io.*;

// Marker interface - no methods
class User implements Serializable {
    private String name;
    private int age;
    
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

class Document {
    // Not implementing Serializable
    private String content;
}

public class Main {
    public static void main(String[] args) {
        User user = new User("John", 25);
        
        // ✅ Can serialize because User implements Serializable
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("user.dat"))) {
            out.writeObject(user); // Works!
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // ❌ Cannot serialize Document (not marked as Serializable)
        Document doc = new Document();
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("doc.dat"))) {
            out.writeObject(doc); // Throws NotSerializableException!
        } catch (IOException e) {
            System.out.println("Error: " + e); // NotSerializableException
        }
    }
}
```

**Common Marker Interfaces:**
- `Serializable` → Enables object serialization
- `Cloneable` → Enables object cloning
- `Remote` → Marks remote objects (RMI)

---

### Q11. Can we create a custom marker interface, and how?

**Answer:** Yes. Define an empty interface and let a class implement it, then in another place check with `instanceof` and apply special behavior only if the object implements that marker.

**Example:**
```java
// Custom marker interface
interface Premium {
    // No methods
}

class Customer {
    private String name;
    
    public Customer(String name) {
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
}

// Premium customer marked with interface
class PremiumCustomer extends Customer implements Premium {
    public PremiumCustomer(String name) {
        super(name);
    }
}

// Regular customer
class RegularCustomer extends Customer {
    public RegularCustomer(String name) {
        super(name);
    }
}

public class BillingSystem {
    public static void calculateBill(Customer customer, double amount) {
        // Check if customer is premium using instanceof
        if (customer instanceof Premium) {
            double discount = amount * 0.20; // 20% discount
            System.out.println("Premium customer " + customer.getName() + 
                             " - Discount: $" + discount);
            System.out.println("Final amount: $" + (amount - discount));
        } else {
            System.out.println("Regular customer " + customer.getName() + 
                             " - No discount");
            System.out.println("Amount: $" + amount);
        }
    }
    
    public static void main(String[] args) {
        Customer c1 = new PremiumCustomer("Alice");
        Customer c2 = new RegularCustomer("Bob");
        
        calculateBill(c1, 100); // Premium - gets 20% discount
        calculateBill(c2, 100); // Regular - no discount
    }
}
```

---

### Q12. What is the volatile keyword?

**Answer:** Used in multithreading to guarantee visibility of updates to a variable across threads and prevent certain kinds of reordering and stale value caching. A volatile read always sees the most recent write by any thread.

**Example:**
```java
class Task implements Runnable {
    // Without volatile, thread may cache the value
    // With volatile, ensures visibility across threads
    private volatile boolean running = true;
    
    @Override
    public void run() {
        System.out.println("Task started");
        while (running) {
            // Do some work
        }
        System.out.println("Task stopped");
    }
    
    public void stop() {
        running = false; // Change visible to other thread immediately
    }
}

public class Main {
    public static void main(String[] args) throws InterruptedException {
        Task task = new Task();
        Thread thread = new Thread(task);
        thread.start();
        
        Thread.sleep(1000);
        
        task.stop(); // Thread will see this change immediately due to volatile
    }
}
```

**Key Points:**
- ✅ Ensures visibility across threads
- ✅ Prevents caching in thread-local memory
- ❌ Does NOT guarantee atomicity (use `synchronized` or `AtomicInteger` for that)

---

### Q13. Can we have a functional interface without an abstract method?

**Answer:** No. A functional interface must have exactly one abstract method; that is what is targeted by lambdas and method references.

**Example:**
```java
// ❌ Invalid - no abstract method
@FunctionalInterface
interface Invalid {
    default void method1() {}
    static void method2() {}
    // Error: No abstract method!
}

// ✅ Valid - exactly one abstract method
@FunctionalInterface
interface Valid {
    void execute(); // One abstract method
    
    default void log() {
        System.out.println("Logging...");
    }
    
    static void info() {
        System.out.println("Info");
    }
}
```

---

### Q14. In a TreeSet, if we add null and "ABC" and then print it, what happens?

**Answer:** A runtime exception occurs. `TreeSet` uses `Comparator`/`Comparable` and tries to compare null with "ABC", leading to a `NullPointerException`, so null is effectively not allowed.

**Example:**
```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        TreeSet<String> set = new TreeSet<>();
        set.add("ABC");
        set.add(null); // ❌ Throws NullPointerException!
        
        System.out.println(set);
    }
}

// Exception in thread "main" java.lang.NullPointerException
```

**Why:** `TreeSet` needs to compare elements to maintain sorted order. Since `null` cannot be compared, it throws `NullPointerException`.

**Note:** `HashSet` allows one null value, but `TreeSet` does not.

---

## **Exception Handling and finally**

### Q15. If we put a return or System.exit in try or catch, will finally still execute?

**Answer:** 

| Case | Finally Executes? |
|------|------------------|
| `return` in try/catch | ✅ Yes |
| `System.exit(0)` | ❌ No (JVM terminates) |
| Exception thrown | ✅ Yes |
| Normal flow | ✅ Yes |

**Example:**
```java
public class FinallyTest {
    
    // Case 1: return in try
    public static int testReturn() {
        try {
            System.out.println("Try block");
            return 1;
        } finally {
            System.out.println("Finally block"); // ✅ Executes!
        }
    }
    
    // Case 2: System.exit
    public static int testExit() {
        try {
            System.out.println("Try block");
            System.exit(0); // JVM terminates
            return 1;
        } finally {
            System.out.println("Finally block"); // ❌ Does NOT execute!
        }
    }
    
    public static void main(String[] args) {
        System.out.println("Result: " + testReturn());
        // Output:
        // Try block
        // Finally block
        // Result: 1
        
        // testExit(); // JVM exits before finally
    }
}
```

**Note:** The interview answer states "finally will always execute even with return or System.exit", but in real Java, `System.exit()` terminates the JVM and finally does not run.

---

## **Microservices Concepts**

### Q16. What are different components of a microservices architecture?

**Answer:** Service discovery, API gateway, load balancer, service registry, AOP (for cross-cutting concerns), and circuit breakers.

**Components:**

| Component | Purpose | Example Tools |
|-----------|---------|---------------|
| **API Gateway** | Single entry point, routing, authentication | Spring Cloud Gateway, Zuul |
| **Service Discovery** | Locate services dynamically | Eureka, Consul |
| **Load Balancer** | Distribute traffic across instances | Ribbon, Nginx |
| **Service Registry** | Store service locations | Eureka Server |
| **Circuit Breaker** | Fault tolerance, fallback | Resilience4j, Hystrix |
| **Config Server** | Centralized configuration | Spring Cloud Config |

---

### Q17. Briefly explain these microservices components.

**Answer:**

**Flow:**
1. Request first hits the **API gateway**
2. **Load balancer** at the gateway routes requests to different instances of microservices
3. **Service discovery/registry** (e.g., Eureka) knows all running service instances and their dynamically registered URLs, helping distribute traffic away from overloaded instances

**Example:**
```
Client Request
    ↓
API Gateway (Port 8080)
    ↓
Load Balancer
    ↓
Service Registry (Eureka)
    ├─→ User Service Instance 1 (Port 8081)
    ├─→ User Service Instance 2 (Port 8082)
    └─→ User Service Instance 3 (Port 8083)
```

---

### Q18. What is a circuit breaker pattern?

**Answer:** Used when one microservice cannot reach another. With a circuit breaker (e.g., Hystrix), a fallback method is defined; if calls fail repeatedly, the circuit "opens" and subsequent calls go to the fallback logic, returning a safe response instead of repeatedly hitting the failing service.

**Example:**
```java
@Service
public class UserService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    // Circuit breaker annotation
    @CircuitBreaker(name = "orderService", fallbackMethod = "getDefaultOrders")
    public List<Order> getOrders(int userId) {
        // Call to Order Service
        return restTemplate.getForObject(
            "http://order-service/orders/" + userId,
            List.class
        );
    }
    
    // Fallback method - called when circuit is open
    public List<Order> getDefaultOrders(int userId, Exception e) {
        System.out.println("Order service is down! Using fallback.");
        return new ArrayList<>(); // Return empty list
    }
}
```

**Circuit States:**
- **Closed:** Normal operation, requests pass through
- **Open:** Too many failures, requests go to fallback
- **Half-Open:** Testing if service recovered

---

### Q19. What is fault isolation?

**Answer:** A design principle that limits the impact of a failure to a single component so it does not cause cascading failures. The goal is to contain faults in the failing part and prevent entire system malfunction.

**Example:**
```
Without Fault Isolation:
Service A fails → Service B fails → Service C fails → Entire system down ❌

With Fault Isolation:
Service A fails → Circuit Breaker → Service B continues with fallback ✅
                                  → Service C unaffected ✅
```

**Techniques:**
- Circuit Breakers
- Bulkhead Pattern (isolate thread pools)
- Timeouts
- Retry with exponential backoff

---

## **Spring Boot Basics**

### Q20. What are the different ways to create a Spring Boot application?

**Answer:**

| Method | Description |
|--------|-------------|
| **Spring Initializr** | Website (start.spring.io) to select dependencies and generate project |
| **IDE Support** | STS, IntelliJ IDEA have built-in Spring Boot project creation |
| **Command-Line** | Spring Boot CLI for project generation |
| **Manual Setup** | Create Maven/Gradle project and add Spring Boot dependencies |

**Example using Spring Initializr:**
1. Go to https://start.spring.io
2. Select: Maven, Java, Spring Boot version
3. Add dependencies: Spring Web, Spring Data JPA
4. Generate → Download → Import into IDE

---

### Q21. What is Spring Boot Actuator and its advantages?

**Answer:** A module providing production-ready features like monitoring and management endpoints. It exposes endpoints such as `/health`, `/info`, and metrics to check application health, environment, and performance.

**Common Endpoints:**

| Endpoint | Purpose |
|----------|---------|
| `/actuator/health` | Application health status |
| `/actuator/info` | Application info |
| `/actuator/metrics` | Application metrics |
| `/actuator/env` | Environment properties |
| `/actuator/loggers` | Logger configuration |

**Example:**
```java
// Add dependency
// <dependency>
//     <groupId>org.springframework.boot</groupId>
//     <artifactId>spring-boot-starter-actuator</artifactId>
// </dependency>

// application.properties
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=always

// Access endpoints:
// http://localhost:8080/actuator/health
// http://localhost:8080/actuator/info
```

---

### Q22. What are Spring Boot profiles?

**Answer:** A way to separate configuration per environment (dev, QA, prod) via `application.properties` or `application.yml` and `spring.profiles.active`. Different profiles load different property sets depending on the environment.

**Example:**
```properties
# application.properties (default)
app.name=MyApp

# application-dev.properties
spring.datasource.url=jdbc:h2:mem:devdb
logging.level.root=DEBUG

# application-prod.properties
spring.datasource.url=jdbc:mysql://prod-server:3306/proddb
logging.level.root=WARN

# Activate profile
# application.properties
spring.profiles.active=dev
```

```java
@Configuration
@Profile("dev")
public class DevConfig {
    @Bean
    public DataSource dataSource() {
        // Dev-specific configuration
        return new H2DataSource();
    }
}

@Configuration
@Profile("prod")
public class ProdConfig {
    @Bean
    public DataSource dataSource() {
        // Prod-specific configuration
        return new MySQLDataSource();
    }
}
```

---

### Q23. How do you handle exceptions globally in a Spring Boot application?

**Answer:** Using `@ControllerAdvice` with `@ExceptionHandler` methods. This centralizes exception handling; each handler method declares the exception type, and Spring routes matching exceptions there for a global handling strategy.

**Example:**
```java
// Custom exception
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

// Global exception handler
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Internal server error",
            System.currentTimeMillis()
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}

// Controller
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable int id) {
        User user = userService.findById(id);
        if (user == null) {
            throw new ResourceNotFoundException("User not found with id: " + id);
        }
        return user;
    }
}
```

---

## **Maven**

### Q24. Which build tool do you use?

**Answer:** Maven is used most frequently.

**Alternatives:** Gradle, Ant

---

### Q25. What are Maven build life cycle phases?

**Answer:** Validate, compile, test, package, clean, install, deploy (as listed by the candidate).

**Complete Lifecycle:**

| Phase | Purpose |
|-------|---------|
| `validate` | Validate project structure |
| `compile` | Compile source code |
| `test` | Run unit tests |
| `package` | Create JAR/WAR |
| `verify` | Run integration tests |
| `install` | Install to local repository |
| `deploy` | Deploy to remote repository |
| `clean` | Remove target directory |

**Example:**
```bash
mvn clean          # Remove target folder
mvn compile        # Compile source code
mvn test          # Run tests
mvn package       # Create JAR/WAR
mvn install       # Install to local Maven repo (~/.m2)
mvn deploy        # Deploy to remote repository
```

---

### Q26. What is the default scope in Maven?

**Answer:** `compile` scope.

**Maven Scopes:**

| Scope | Available At | Included in Package |
|-------|-------------|-------------------|
| `compile` | Compile, Test, Runtime | ✅ Yes |
| `provided` | Compile, Test | ❌ No (server provides it) |
| `runtime` | Test, Runtime | ✅ Yes |
| `test` | Test only | ❌ No |
| `system` | Compile, Test | ✅ Yes (must specify path) |

**Example:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- No scope specified = compile (default) -->
</dependency>

<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <scope>provided</scope> <!-- Server provides this -->
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope> <!-- Only for testing -->
</dependency>
```

---

### Q27. What is a Maven plugin, and how have you used it?

**Answer:** Plugins extend Maven behavior; the candidate used the assembly plugin with an `assembly.xml` to control how generated JARs are packaged, specifying where libraries, resource files, and property files should go.

**Example:**
```xml
<build>
    <plugins>
        <!-- Maven Assembly Plugin -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>3.3.0</version>
            <configuration>
                <descriptors>
                    <descriptor>src/assembly/assembly.xml</descriptor>
                </descriptors>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

**assembly.xml:**
```xml
<assembly>
    <id>dist</id>
    <formats>
        <format>zip</format>
    </formats>
    <fileSets>
        <fileSet>
            <directory>${project.build.directory}</directory>
            <outputDirectory>/lib</outputDirectory>
            <includes>
                <include>*.jar</include>
            </includes>
        </fileSet>
    </fileSets>
</assembly>
```

**Common Plugins:**
- `maven-compiler-plugin` → Compile Java code
- `maven-surefire-plugin` → Run tests
- `maven-assembly-plugin` → Create distributable packages
- `maven-jar-plugin` → Create JAR files

---

### Q28. What is settings.xml in Maven and where is it located?

**Answer:** `settings.xml` resides in the `.m2` folder under the user directory (e.g., in `C:\Users\<user>\.m2`). It defines global settings such as repository paths (e.g., company Nexus/Artifactory) and custom local repository locations.

**Example settings.xml:**
```xml
<settings>
    <!-- Local repository location -->
    <localRepository>D:/maven-repo</localRepository>
    
    <!-- Mirror for Maven Central -->
    <mirrors>
        <mirror>
            <id>company-nexus</id>
            <mirrorOf>*</mirrorOf>
            <url>https://nexus.company.com/repository/maven-public/</url>
        </mirror>
    </mirrors>
    
    <!-- Server credentials -->
    <servers>
        <server>
            <id>company-nexus</id>
            <username>admin</username>
            <password>admin123</password>
        </server>
    </servers>
    
    <!-- Active profiles -->
    <activeProfiles>
        <activeProfile>company</activeProfile>
    </activeProfiles>
</settings>
```

**Location:**
- **Windows:** `C:\Users\<username>\.m2\settings.xml`
- **Linux/Mac:** `/home/<username>/.m2/settings.xml`

---

## **SQL**

### Q29. What is a clustered vs non-clustered index?

**Answer:**

| Feature | Clustered Index | Non-Clustered Index |
|---------|----------------|-------------------|
| **Physical order** | Defines physical order of rows | Does not change physical order |
| **Per table** | Only ONE per table | Multiple allowed |
| **Storage** | Data stored in index | Separate structure with pointers |
| **Speed** | Faster for range queries | Faster for specific lookups |
| **Default on** | Primary Key | Can be on any column |

**Example:**
```sql
-- Clustered index (automatically on primary key)
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY, -- Clustered index
    Name VARCHAR(100),
    Department VARCHAR(50)
);

-- Non-clustered index
CREATE INDEX IX_Employees_Department 
ON Employees(Department); -- Non-clustered index

-- Multiple non-clustered indexes allowed
CREATE INDEX IX_Employees_Name 
ON Employees(Name);
```

**A clustered index** defines the physical order of rows in a table; only one per table, typically on the primary key.

**A non-clustered index** is a separate structure with sorted key values and pointers to data rows, does not change physical row order, and there can be multiple per table.

---

### Q30. What is a view in SQL and how is it different from a table?

**Answer:** A view is a virtual table based on an underlying query; it does not store data physically but shows the result of that query. Tables physically store data, while views are stored query definitions and used to simplify complex queries or present data in a certain format.

**Example:**
```sql
-- Create tables
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    Name VARCHAR(100),
    DepartmentID INT,
    Salary DECIMAL(10,2)
);

CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY,
    DepartmentName VARCHAR(100)
);

-- Create view (virtual table)
CREATE VIEW EmployeeDetails AS
SELECT 
    e.EmployeeID,
    e.Name,
    d.DepartmentName,
    e.Salary
FROM Employees e
JOIN Departments d ON e.DepartmentID = d.DepartmentID;

-- Use view like a table
SELECT * FROM EmployeeDetails WHERE Salary > 50000;
```

**Differences:**

| Feature | Table | View |
|---------|-------|------|
| **Stores data** | ✅ Yes (physical storage) | ❌ No (virtual) |
| **Query definition** | ❌ No | ✅ Yes (stores SQL) |
| **Performance** | Fast (direct access) | Slower (executes query) |
| **Can insert/update** | ✅ Yes | ⚠️ Sometimes (simple views only) |
| **Use case** | Store data | Simplify queries, security |

---

### Q31. What is a stored procedure and how is it different from a view?

**Answer:** A stored procedure is a stored set of one or more SQL statements that can accept parameters, perform operations, and return results. Unlike views, procedures encapsulate logic and can perform updates/complex operations, while a view mainly represents a query result.

**Example:**
```sql
-- Stored procedure with parameters
CREATE PROCEDURE GetEmployeesByDepartment
    @DepartmentName VARCHAR(100)
AS
BEGIN
    SELECT 
        e.EmployeeID,
        e.Name,
        e.Salary
    FROM Employees e
    JOIN Departments d ON e.DepartmentID = d.DepartmentID
    WHERE d.DepartmentName = @DepartmentName;
END;

-- Call stored procedure
EXEC GetEmployeesByDepartment @DepartmentName = 'Engineering';

-- Stored procedure with logic
CREATE PROCEDURE UpdateEmployeeSalary
    @EmployeeID INT,
    @NewSalary DECIMAL(10,2)
AS
BEGIN
    -- Validation logic
    IF @NewSalary > 0
    BEGIN
        UPDATE Employees
        SET Salary = @NewSalary
        WHERE EmployeeID = @EmployeeID;
        
        PRINT 'Salary updated successfully';
    END
    ELSE
    BEGIN
        PRINT 'Invalid salary amount';
    END
END;

-- Call procedure
EXEC UpdateEmployeeSalary @EmployeeID = 101, @NewSalary = 75000;
```

**Differences:**

| Feature | View | Stored Procedure |
|---------|------|-----------------|
| **Type** | Virtual table | Executable program |
| **Parameters** | ❌ No | ✅ Yes |
| **Logic** | ❌ No (just SELECT) | ✅ Yes (IF, WHILE, etc.) |
| **DML operations** | ⚠️ Limited | ✅ Full (INSERT, UPDATE, DELETE) |
| **Returns** | Result set | Result set, values, or nothing |
| **Use case** | Simplify queries | Encapsulate business logic |

---

## Summary

| Topic | Key Takeaway |
|-------|-------------|
| **Diamond Problem** | Class must override and explicitly choose which interface's default method to call |
| **Private Methods in Interfaces** | Help refactor common logic in default/static methods (Java 9+) |
| **Default vs Static Methods** | Default can be overridden, static cannot; static accessed via interface name |
| **Varargs** | Variable number of arguments using `type... param` |
| **Static Method with null** | Works fine - resolved at compile time to class |
| **equals-hashCode Contract** | Override both together for collections to work correctly |
| **Override only equals** | Breaks hash-based collections (different hashCodes) |
| **Base vs Abstract Class** | Base can be instantiated, abstract cannot |
| **Marker Interface** | Empty interface to mark classes for special treatment |
| **Custom Marker Interface** | Define empty interface, check with `instanceof` |
| **volatile** | Ensures visibility across threads |
| **Functional Interface** | Must have exactly one abstract method |
| **TreeSet and null** | Throws `NullPointerException` (cannot compare null) |
| **finally block** | Executes even with return, but NOT with `System.exit()` |
| **Microservices Components** | API Gateway, Service Discovery, Load Balancer, Circuit Breaker |
| **Circuit Breaker** | Prevents cascading failures with fallback mechanism |
| **Fault Isolation** | Limit failure impact to single component |
| **Spring Boot Creation** | Spring Initializr, IDE support, CLI |
| **Spring Boot Actuator** | Production-ready monitoring endpoints |
| **Spring Boot Profiles** | Separate configurations per environment |
| **Global Exception Handler** | `@ControllerAdvice` with `@ExceptionHandler` |
| **Maven Build Tool** | Validate, compile, test, package, install, deploy |
| **Maven Default Scope** | `compile` |
| **Maven Plugin** | Extend Maven behavior (assembly, compiler, surefire) |
| **settings.xml** | Global Maven configuration in `.m2` folder |
| **Clustered Index** | Physical order, one per table, on primary key |
| **Non-Clustered Index** | Separate structure, multiple allowed |
| **View** | Virtual table, stores query definition, no physical data |
| **Stored Procedure** | Executable program with logic, accepts parameters, performs DML operations |
