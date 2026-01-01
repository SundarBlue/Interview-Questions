# Spring Boot Interview Questions

## 1. Constructor Dependency Injection vs @Autowired
**Concept:** Two ways to inject dependencies in Spring.

### Constructor Injection (Recommended ✅)
Dependencies are passed through the constructor.

```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    // Constructor injection
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}
```

### Field Injection (@Autowired)
Dependencies are injected directly into fields.

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private EmailService emailService;
}
```

### Why Constructor Injection is Better:

| Aspect | Constructor Injection | @Autowired (Field Injection) |
|--------|----------------------|----------------------------|
| **Immutability** | ✅ Can use `final` fields | ❌ Cannot use `final` |
| **Null Safety** | ✅ NPE at startup if missing | ❌ NPE at runtime when used |
| **Testing** | ✅ Easy to test (pass mocks in constructor) | ❌ Harder to test (need reflection or Spring context) |
| **Circular Dependencies** | ✅ Fails fast at startup | ❌ May hide circular dependencies |
| **Code Quality** | ✅ Makes dependencies explicit | ❌ Hidden dependencies |
| **Spring 4.3+** | ✅ No @Autowired needed for single constructor | ❌ Always need @Autowired |

**Real-World Analogy:**
- **Constructor Injection** = Building a car 🚗 - You must have an engine before the car can work. If the engine is missing, the car won't even start.
- **Field Injection** = Adding features later - The car starts, but when you try to use the radio (dependency), you realize it's missing (NPE at runtime).

**Testing Example:**
```java
// Constructor Injection - Easy to test
public class UserServiceTest {
    @Test
    public void testCreateUser() {
        // Create mock dependencies
        UserRepository mockRepo = mock(UserRepository.class);
        EmailService mockEmail = mock(EmailService.class);
        
        // Pass mocks via constructor - clean and simple
        UserService service = new UserService(mockRepo, mockEmail);
        
        // Test your logic
        service.createUser("John");
        verify(mockRepo).save(any());
    }
}

// Field Injection - Need Spring context or reflection
public class UserServiceTest {
    @InjectMocks
    private UserService service;
    
    @Mock
    private UserRepository userRepository;
    
    // More complex setup needed
}
```

---

## 2. Spring Boot Bean Lifecycle
**Concept:** Understanding how Spring creates, initializes, and destroys beans.

### Bean Lifecycle Phases:

```
Instantiation → Populate Properties → BeanNameAware → BeanFactoryAware 
→ ApplicationContextAware → @PostConstruct/InitializingBean 
→ Custom Init Method → Bean Ready to Use 
→ @PreDestroy/DisposableBean → Custom Destroy Method → Bean Destroyed
```

**Example with All Lifecycle Hooks:**
```java
@Component
public class LifecycleBean implements BeanNameAware, BeanFactoryAware, 
    ApplicationContextAware, InitializingBean, DisposableBean {
    
    private String beanName;
    
    // 1. Constructor called
    public LifecycleBean() {
        System.out.println("1. Constructor called");
    }
    
    // 2. Setter injection (if any)
    @Autowired
    public void setDependency(SomeDependency dependency) {
        System.out.println("2. Dependencies injected");
    }
    
    // 3. BeanNameAware
    @Override
    public void setBeanName(String name) {
        this.beanName = name;
        System.out.println("3. BeanNameAware: Bean name is " + name);
    }
    
    // 4. BeanFactoryAware
    @Override
    public void setBeanFactory(BeanFactory beanFactory) {
        System.out.println("4. BeanFactoryAware called");
    }
    
    // 5. ApplicationContextAware
    @Override
    public void setApplicationContext(ApplicationContext context) {
        System.out.println("5. ApplicationContextAware called");
    }
    
    // 6. @PostConstruct (Recommended)
    @PostConstruct
    public void init() {
        System.out.println("6. @PostConstruct called - Bean initialized");
    }
    
    // 7. InitializingBean
    @Override
    public void afterPropertiesSet() {
        System.out.println("7. InitializingBean.afterPropertiesSet() called");
    }
    
    // 8. Custom init method (via @Bean annotation)
    public void customInit() {
        System.out.println("8. Custom init method called");
    }
    
    // ====== Bean is now ready to use ======
    
    // 9. @PreDestroy (when context is closing)
    @PreDestroy
    public void cleanup() {
        System.out.println("9. @PreDestroy called - Cleaning up");
    }
    
    // 10. DisposableBean
    @Override
    public void destroy() {
        System.out.println("10. DisposableBean.destroy() called");
    }
    
    // 11. Custom destroy method
    public void customDestroy() {
        System.out.println("11. Custom destroy method called");
    }
}

// Configuration class
@Configuration
public class AppConfig {
    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
    public LifecycleBean lifecycleBean() {
        return new LifecycleBean();
    }
}
```

**Practical Example - Database Connection Pool:**
```java
@Component
public class DatabaseConnectionPool {
    private List<Connection> connectionPool;
    
    @PostConstruct
    public void initialize() {
        System.out.println("Initializing connection pool...");
        connectionPool = new ArrayList<>();
        // Create initial connections
        for (int i = 0; i < 10; i++) {
            connectionPool.add(createConnection());
        }
        System.out.println("Connection pool ready with " + connectionPool.size() + " connections");
    }
    
    @PreDestroy
    public void cleanup() {
        System.out.println("Closing all database connections...");
        connectionPool.forEach(conn -> {
            try {
                conn.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
        System.out.println("All connections closed");
    }
    
    private Connection createConnection() {
        // Create database connection
        return null; // Simplified
    }
}
```

---

## 3. @RequestBody vs @ResponseBody
**Concept:** Annotations used in Spring MVC for handling HTTP request and response bodies.

### @RequestBody
Converts HTTP request body (JSON/XML) into a Java object.

**Example:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // Client sends JSON in request body
    // { "name": "John", "email": "john@example.com" }
    @PostMapping
    public User createUser(@RequestBody User user) {
        // Spring automatically converts JSON to User object
        System.out.println("Received: " + user.getName());
        return userService.save(user);
    }
}
```

### @ResponseBody
Converts Java object into HTTP response body (JSON/XML).

**Example:**
```java
@Controller
@RequestMapping("/api/products")
public class ProductController {
    
    @GetMapping("/{id}")
    @ResponseBody // Converts Product object to JSON
    public Product getProduct(@PathVariable Long id) {
        Product product = productService.findById(id);
        // Spring converts Product object to JSON response
        return product;
    }
}
```

### Key Differences:

| Aspect | @RequestBody | @ResponseBody |
|--------|-------------|---------------|
| **Purpose** | Convert HTTP request → Java object | Convert Java object → HTTP response |
| **Location** | Method parameter | Method or Class level |
| **Data Flow** | Client → Server | Server → Client |
| **Example** | `public void create(@RequestBody User user)` | `@ResponseBody public User get()` |

**Note:** When using `@RestController` (instead of `@Controller`), `@ResponseBody` is applied to all methods automatically.

```java
// With @Controller - need @ResponseBody on each method
@Controller
public class UserController {
    @GetMapping("/users")
    @ResponseBody // Required
    public List<User> getUsers() {
        return userService.findAll();
    }
}

// With @RestController - @ResponseBody automatic
@RestController
public class UserController {
    @GetMapping("/users")
    // No @ResponseBody needed - automatic
    public List<User> getUsers() {
        return userService.findAll();
    }
}
```

---

## 4. @Controller vs @RestController
**Concept:** Both are used to create Spring MVC controllers, but they serve different purposes.

### @Controller
- Used for traditional web applications (MVC)
- Returns **view names** (HTML pages via Thymeleaf, JSP, etc.)
- Needs `@ResponseBody` to return data as JSON

**Example:**
```java
@Controller
@RequestMapping("/web")
public class WebController {
    
    // Returns a view name (Thymeleaf template)
    @GetMapping("/home")
    public String home(Model model) {
        model.addAttribute("message", "Welcome!");
        return "home"; // Returns home.html view
    }
    
    // To return JSON, need @ResponseBody
    @GetMapping("/data")
    @ResponseBody
    public Map<String, String> getData() {
        return Map.of("status", "success");
    }
}
```

### @RestController
- Used for RESTful web services
- Returns **data** (JSON/XML) directly
- Equivalent to `@Controller` + `@ResponseBody` on every method

**Example:**
```java
@RestController
@RequestMapping("/api")
public class ApiController {
    
    // Automatically returns JSON (no @ResponseBody needed)
    @GetMapping("/users")
    public List<User> getUsers() {
        return userService.findAll(); // Returns JSON
    }
    
    @PostMapping("/users")
    public User createUser(@RequestBody User user) {
        return userService.save(user); // Returns JSON
    }
}
```

### Comparison:

| Feature | @Controller | @RestController |
|---------|------------|----------------|
| **Purpose** | Web pages (MVC) | REST APIs |
| **Returns** | View names (HTML) | Data (JSON/XML) |
| **@ResponseBody** | Required for JSON response | Automatic for all methods |
| **Equivalent to** | `@Controller` | `@Controller` + `@ResponseBody` |
| **Use Case** | Server-side rendering | API endpoints for frontend apps |

**When to use what?**
- Use `@Controller` for traditional web apps with server-side rendering (Thymeleaf, JSP)
- Use `@RestController` for RESTful APIs (returning JSON for React, Angular, Mobile apps)

---

## 5. @PathVariable vs @RequestParam
**Concept:** Both extract values from HTTP requests, but from different parts of the URL.

### @PathVariable
Extracts values from the **URL path** (URI template).

**Example:**
```java
@RestController
@RequestMapping("/api")
public class ProductController {
    
    // URL: /api/products/123
    @GetMapping("/products/{id}")
    public Product getProduct(@PathVariable Long id) {
        return productService.findById(id);
    }
    
    // Multiple path variables
    // URL: /api/categories/electronics/products/laptop
    @GetMapping("/categories/{category}/products/{type}")
    public List<Product> getProducts(
        @PathVariable String category,
        @PathVariable String type
    ) {
        return productService.findByCategoryAndType(category, type);
    }
}
```

### @RequestParam
Extracts values from **query parameters** (after `?` in URL).

**Example:**
```java
@RestController
@RequestMapping("/api")
public class ProductController {
    
    // URL: /api/products?page=1&size=10
    @GetMapping("/products")
    public List<Product> getProducts(
        @RequestParam int page,
        @RequestParam int size
    ) {
        return productService.findAll(page, size);
    }
    
    // Optional parameter with default value
    // URL: /api/search?query=laptop (category optional)
    @GetMapping("/search")
    public List<Product> search(
        @RequestParam String query,
        @RequestParam(required = false, defaultValue = "all") String category
    ) {
        return productService.search(query, category);
    }
}
```

### Comparison:

| Feature | @PathVariable | @RequestParam |
|---------|--------------|---------------|
| **Location in URL** | Part of path `/users/{id}` | Query string `/users?id=123` |
| **Required by default** | Yes | Yes (but can make optional) |
| **URL Example** | `/api/users/5` | `/api/users?id=5` |
| **Use Case** | Identify resource | Filter/sort/paginate |
| **REST Style** | RESTful (clean URLs) | Traditional query parameters |

**Example combining both:**
```java
// URL: /api/users/123/orders?status=pending&page=1
@GetMapping("/users/{userId}/orders")
public List<Order> getUserOrders(
    @PathVariable Long userId,           // From path
    @RequestParam String status,          // From query
    @RequestParam(defaultValue = "0") int page
) {
    return orderService.findByUserAndStatus(userId, status, page);
}
```

---

## 6. What is DispatcherServlet?
**Concept:** DispatcherServlet is the **front controller** in Spring MVC. It's the entry point for all HTTP requests and routes them to appropriate handlers.

### Request Flow:

```
Client Request → DispatcherServlet → Handler Mapping → Controller 
→ Business Logic → View Resolver → Response to Client
```

### Detailed Flow:

1. **Client** sends HTTP request to Spring application
2. **DispatcherServlet** receives the request (configured in web.xml or Java config)
3. **Handler Mapping** determines which Controller should handle the request
4. **Controller** processes the request and returns Model and View name
5. **View Resolver** resolves the view name to actual view (HTML, JSON, etc.)
6. **View** renders the response
7. **DispatcherServlet** sends response back to client

**Example Configuration:**
```java
// Spring Boot auto-configures DispatcherServlet
// But if you need custom configuration:

@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Bean
    public DispatcherServlet dispatcherServlet() {
        DispatcherServlet servlet = new DispatcherServlet();
        // Custom configurations
        servlet.setThrowExceptionIfNoHandlerFound(true);
        return servlet;
    }
}
```

**Real-World Analogy:**
Think of DispatcherServlet as a **receptionist at a hospital**:
- Patient (Client) comes with a problem (HTTP Request)
- Receptionist (DispatcherServlet) checks which doctor (Controller) can help
- Doctor (Controller) treats the patient and sends prescription (Model)
- Receptionist formats the prescription (View Resolver)
- Patient receives the final prescription (Response)

**How it works in Spring Boot:**
```java
// Spring Boot automatically configures DispatcherServlet at "/"
// All requests go through DispatcherServlet

@RestController
@RequestMapping("/api")
public class UserController {
    
    // Request: GET /api/users/123
    // 1. DispatcherServlet receives request
    // 2. Handler Mapping maps "/api/users/{id}" to this method
    // 3. Controller executes
    // 4. Response returned via DispatcherServlet
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}
```

### Key Responsibilities:
- Request routing
- Handler mapping
- View resolution
- Exception handling
- Locale resolution
- Theme resolution
- File upload handling

---

## Summary Table

| Topic | Key Takeaway |
|-------|-------------|
| **Constructor Injection vs @Autowired** | Constructor injection is better: immutable, testable, fail-fast |
| **Spring Boot Lifecycle** | Instantiation → @PostConstruct → Bean Ready → @PreDestroy |
| **@RequestBody** | Converts HTTP request body (JSON) → Java object |
| **@ResponseBody** | Converts Java object → HTTP response body (JSON) |
| **@Controller** | Returns view names (HTML pages) for web apps |
| **@RestController** | Returns data (JSON) for REST APIs |
| **@PathVariable** | Extract values from URL path `/users/{id}` |
| **@RequestParam** | Extract values from query string `/users?id=5` |
| **DispatcherServlet** | Front controller that routes all requests to handlers |
