# 🍃 SPRING BOOT - INTERVIEW CHEATSHEET

> **Phiên bản đầy đủ nhất** — đọc phát hiểu luôn, không cần tra thêm  
> Spring Boot 3.x | Spring 6 | Java 17+

---

## 📋 MỤC LỤC

1. [Spring Framework là gì? Spring vs Spring Boot](#1-spring-vs-spring-boot)
2. [IoC Container & Dependency Injection](#2-ioc--dependency-injection)
3. [Bean — Lifecycle, Scope, Annotations](#3-bean)
4. [Auto-Configuration — Magic bên trong](#4-auto-configuration)
5. [application.properties / application.yml](#5-configuration)
6. [Spring MVC — Request Flow](#6-spring-mvc)
7. [REST API — Controllers, Request Mapping](#7-rest-api)
8. [Request/Response — DTO, Validation](#8-dto--validation)
9. [Exception Handling — @ControllerAdvice](#9-exception-handling)
10. [Spring Data JPA](#10-spring-data-jpa)
11. [Repository Pattern — JpaRepository, Query Methods](#11-repository--query)
12. [Transaction Management — @Transactional](#12-transactional)
13. [Spring Security — Authentication & Authorization](#13-spring-security)
14. [JWT Authentication Flow](#14-jwt)
15. [Spring AOP — Aspect Oriented Programming](#15-spring-aop)
16. [Caching — @Cacheable, Redis](#16-caching)
17. [Spring Events](#17-spring-events)
18. [Profiles — dev, staging, prod](#18-profiles)
19. [Testing — Unit & Integration Tests](#19-testing)
20. [Actuator — Monitoring & Health](#20-actuator)
21. [Microservices — Spring Cloud cơ bản](#21-microservices--spring-cloud)
22. [Câu hỏi phỏng vấn hay gặp](#22-câu-hỏi-hay-gặp)

---

## 1. Spring vs Spring Boot

### Spring Framework là gì?

Spring Framework là một **Java application framework** cung cấp:
- **IoC Container** — quản lý object lifecycle
- **Dependency Injection** — inject dependency tự động
- **AOP** — cross-cutting concerns (logging, transaction...)
- **Spring MVC** — web layer
- **Spring Data** — data access layer

**Vấn đề của Spring thuần:**
- Cấu hình XML/Java quá nhiều (boilerplate)
- Phải tự cấu hình từng component (DataSource, Tomcat, Jackson...)
- Nhiều dependency conflict

### Spring Boot giải quyết gì?

```
Spring Boot = Spring Framework
           + Auto-Configuration (tự cấu hình)
           + Embedded Server (Tomcat built-in)
           + Starter Dependencies (bundle dependency)
           + Opinionated Defaults (convention over configuration)
```

```java
// Spring MVC thuần — cần ~50 dòng XML config
// web.xml, dispatcher-servlet.xml, root-context.xml...

// Spring Boot — chỉ cần:
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
// Tự động: embedded Tomcat, component scan, auto-config
```

### @SpringBootApplication là gì?

```java
@SpringBootApplication
// = @Configuration + @EnableAutoConfiguration + @ComponentScan

// @Configuration: class này là nguồn khai báo bean
// @EnableAutoConfiguration: bật auto-config mechanism
// @ComponentScan: scan tất cả @Component trong package hiện tại và con
```

---

## 2. IoC & Dependency Injection

### IoC (Inversion of Control)

> **Bình thường:** Code của bạn tạo object → bạn kiểm soát  
> **IoC:** Framework tạo & quản lý object → framework kiểm soát  
> Bạn chỉ "khai báo" cần gì, Spring lo phần còn lại

```java
// Không có IoC — tightly coupled
public class OrderService {
    private EmailService emailService = new EmailService(); // Tự tạo
    private OrderRepository repo = new OrderRepository();   // Tự tạo
}

// Có IoC — loosely coupled
@Service
public class OrderService {
    private final EmailService emailService;   // Spring inject
    private final OrderRepository repo;        // Spring inject

    public OrderService(EmailService emailService, OrderRepository repo) {
        this.emailService = emailService;
        this.repo = repo;
    }
}
```

### 3 kiểu Dependency Injection

```java
// 1. Constructor Injection — RECOMMENDED
@Service
public class UserService {
    private final UserRepository userRepo;
    private final EmailService emailService;

    // @Autowired không cần nếu chỉ có 1 constructor (Spring 4.3+)
    public UserService(UserRepository userRepo, EmailService emailService) {
        this.userRepo = userRepo;
        this.emailService = emailService;
    }
}
// Ưu điểm: immutable field, dễ test (pass mock qua constructor), fail fast

// 2. Setter Injection — Dùng cho optional dependencies
@Service
public class UserService {
    private NotificationService notificationService;

    @Autowired(required = false) // Optional dependency
    public void setNotificationService(NotificationService ns) {
        this.notificationService = ns;
    }
}

// 3. Field Injection — Không nên dùng trong production
@Service
public class UserService {
    @Autowired // Không thể dùng final, khó test, che giấu dependency
    private UserRepository userRepo;
}
```

**Tại sao Constructor Injection được recommend?**
- Fields có thể là `final` → immutable, thread-safe hơn
- Dependencies rõ ràng — nhìn vào constructor biết ngay cần gì
- Dễ viết unit test không cần Spring context (chỉ cần `new Service(mockRepo)`)
- Phát hiện circular dependency sớm hơn (lúc startup)

### @Autowired hoạt động thế nào?

```java
// Spring tìm bean theo:
// 1. Tìm theo TYPE (interface/class)
// 2. Nếu có nhiều bean cùng type → tìm theo NAME (tên field/param)
// 3. Vẫn ambiguous → dùng @Qualifier

@Service("premiumEmailService")
public class PremiumEmailService implements EmailService { ... }

@Service("basicEmailService")
public class BasicEmailService implements EmailService { ... }

// Dùng @Qualifier để chỉ rõ
@Service
public class OrderService {
    private final EmailService emailService;

    public OrderService(@Qualifier("premiumEmailService") EmailService emailService) {
        this.emailService = emailService;
    }
}

// Hoặc dùng @Primary — bean này được ưu tiên khi inject
@Primary
@Service
public class PremiumEmailService implements EmailService { ... }
```

---

## 3. Bean

### Bean là gì?

> **Bean** = object được **Spring IoC Container** quản lý (tạo, configure, wire, destroy).

### Khai báo Bean

```java
// Cách 1: @Component và các stereotype annotations
@Component      // Bean thông thường
@Service        // Business logic layer (= @Component về kỹ thuật)
@Repository     // Data access layer — tự translate DB exception
@Controller     // Web layer
@RestController // @Controller + @ResponseBody

// Cách 2: @Bean trong @Configuration class
@Configuration
public class AppConfig {
    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper()
            .configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
    }

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

// Cách 3: @Bean với init và destroy
@Bean(initMethod = "init", destroyMethod = "cleanup")
public SomeService someService() {
    return new SomeService();
}
```

### Bean Scope

| Scope | Mô tả | Dùng khi |
|-------|-------|----------|
| **singleton** | 1 instance cho toàn app (default) | Stateless service |
| **prototype** | Tạo mới mỗi lần inject/request | Stateful object |
| **request** | 1 instance per HTTP request | Web app |
| **session** | 1 instance per HTTP session | Web app, user data |
| **application** | 1 instance per ServletContext | Web app global |

```java
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Component
public class ShoppingCart { // Mỗi user cần 1 cart riêng
    private List<Item> items = new ArrayList<>();
}

// Vấn đề: Inject prototype vào singleton
@Service // singleton
public class OrderService {
    @Autowired
    private ShoppingCart cart; // Chỉ inject 1 lần — cùng instance mãi!
}

// Giải pháp: ApplicationContext lookup, hoặc @Lookup
@Service
public class OrderService {
    @Autowired
    private ApplicationContext context;

    public void processOrder() {
        ShoppingCart cart = context.getBean(ShoppingCart.class); // Fresh bean
    }
}
```

### Bean Lifecycle

```
Container startup
      ↓
1. Instantiate (gọi constructor)
      ↓
2. Populate Properties (inject dependencies)
      ↓
3. BeanNameAware.setBeanName()
      ↓
4. BeanFactoryAware.setBeanFactory()
      ↓
5. ApplicationContextAware.setApplicationContext()
      ↓
6. @PostConstruct / InitializingBean.afterPropertiesSet()
      ↓
7. Bean sẵn sàng sử dụng
      ↓
Container shutdown
      ↓
8. @PreDestroy / DisposableBean.destroy()
```

```java
@Service
public class DatabaseService {
    private Connection connection;

    @PostConstruct
    public void init() {
        // Chạy sau khi inject xong — khởi tạo connection pool
        connection = createConnection();
        System.out.println("DatabaseService initialized");
    }

    @PreDestroy
    public void cleanup() {
        // Chạy trước khi bean bị destroy
        if (connection != null) connection.close();
        System.out.println("DatabaseService destroyed");
    }
}
```

---

## 4. Auto-Configuration

### Spring Boot tự cấu hình thế nào?

```
1. @EnableAutoConfiguration scan META-INF/spring/
   org.springframework.boot.autoconfigure.AutoConfiguration.imports

2. Hàng trăm AutoConfiguration classes được load

3. Mỗi class có @ConditionalOn... để check điều kiện:
   - @ConditionalOnClass: class X có trong classpath không?
   - @ConditionalOnMissingBean: bean Y đã được define chưa?
   - @ConditionalOnProperty: property Z có được set không?

4. Nếu điều kiện thỏa → tạo bean mặc định
```

```java
// Ví dụ DataSourceAutoConfiguration (simplified)
@AutoConfiguration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@ConditionalOnMissingBean(type = "io.r2dbc.spi.ConnectionFactory")
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean // Chỉ tạo nếu bạn CHƯA define DataSource
    public DataSource dataSource(DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder().build();
    }
}
```

**Vậy khi nào auto-config bị override?**
- Khi bạn tự khai báo `@Bean` cùng type → `@ConditionalOnMissingBean` không thỏa → auto-config không tạo bean nữa

```java
// Bạn muốn config DataSource riêng:
@Configuration
public class DatabaseConfig {
    @Bean // Tự khai báo → Spring Boot không dùng default nữa
    public DataSource dataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:postgresql://localhost/mydb");
        ds.setMaximumPoolSize(20);
        return ds;
    }
}
```

### Debug Auto-Configuration

```bash
# Xem auto-config nào được apply và tại sao
java -jar app.jar --debug

# Hoặc trong application.properties:
debug=true
```

---

## 5. Configuration

### application.properties vs application.yml

```properties
# application.properties
server.port=8080
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=postgres
spring.datasource.password=secret
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
logging.level.com.myapp=DEBUG
```

```yaml
# application.yml — cấu trúc rõ hơn khi có nhiều property lồng nhau
server:
  port: 8080

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: postgres
    password: secret
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

logging:
  level:
    com.myapp: DEBUG
```

### @Value — Inject single property

```java
@Service
public class PaymentService {
    @Value("${payment.api.key}")
    private String apiKey;

    @Value("${payment.timeout:5000}") // Default value nếu không có
    private int timeout;

    @Value("${app.features.list}") // Inject list
    private List<String> features;
}
```

### @ConfigurationProperties — Inject nhóm properties (Recommended)

```yaml
# application.yml
app:
  payment:
    api-key: sk_live_abc123
    timeout: 5000
    retry-count: 3
    base-url: https://payment.example.com
```

```java
@ConfigurationProperties(prefix = "app.payment")
@Component // hoặc dùng @EnableConfigurationProperties(PaymentProperties.class)
public class PaymentProperties {
    private String apiKey;
    private int timeout;
    private int retryCount;
    private String baseUrl;

    // Getters & setters (hoặc dùng @ConfigurationProperties + Record)
}

// Java 17+ — dùng Record
@ConfigurationProperties(prefix = "app.payment")
public record PaymentProperties(
    String apiKey,
    int timeout,
    int retryCount,
    String baseUrl
) {}

// Inject và dùng
@Service
public class PaymentService {
    private final PaymentProperties props;

    public PaymentService(PaymentProperties props) {
        this.props = props;
    }

    public void pay() {
        // props.apiKey(), props.timeout()...
    }
}
```

### Environment-specific config

```
src/main/resources/
├── application.yml           # Config chung
├── application-dev.yml       # Override cho dev
├── application-staging.yml   # Override cho staging
└── application-prod.yml      # Override cho prod
```

```yaml
# application-prod.yml
spring:
  datasource:
    url: ${DATABASE_URL}  # Đọc từ env variable
    username: ${DB_USER}
    password: ${DB_PASSWORD}
  jpa:
    show-sql: false
logging:
  level:
    root: WARN
```

---

## 6. Spring MVC

### Request Flow

```
HTTP Request
     ↓
DispatcherServlet (Front Controller)
     ↓
HandlerMapping → Tìm @Controller phù hợp với URL
     ↓
HandlerAdapter → Gọi method trong Controller
     ↓ (nếu có @ExceptionHandler/@ControllerAdvice → bắt exception)
Controller Method → Xử lý business logic
     ↓
ViewResolver (REST API → MessageConverter thay vì View)
     ↓
HTTP Response (JSON/XML)
```

### DispatcherServlet

> Là **Front Controller** — 1 servlet duy nhất nhận toàn bộ request, sau đó route đến đúng handler.

Spring Boot tự đăng ký DispatcherServlet tại `/` (map tất cả request).

---

## 7. REST API

### Controller Annotations

```java
@RestController // = @Controller + @ResponseBody
@RequestMapping("/api/v1/users") // Base path
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    // GET /api/v1/users
    @GetMapping
    public List<UserResponse> getAllUsers() {
        return userService.findAll();
    }

    // GET /api/v1/users/{id}
    @GetMapping("/{id}")
    public UserResponse getUserById(@PathVariable Long id) {
        return userService.findById(id);
    }

    // GET /api/v1/users?page=0&size=10&sort=name
    @GetMapping("/search")
    public Page<UserResponse> searchUsers(
        @RequestParam(defaultValue = "") String name,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size
    ) {
        return userService.search(name, page, size);
    }

    // POST /api/v1/users
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED) // 201
    public UserResponse createUser(@Valid @RequestBody CreateUserRequest request) {
        return userService.create(request);
    }

    // PUT /api/v1/users/{id}
    @PutMapping("/{id}")
    public UserResponse updateUser(
        @PathVariable Long id,
        @Valid @RequestBody UpdateUserRequest request
    ) {
        return userService.update(id, request);
    }

    // PATCH /api/v1/users/{id}
    @PatchMapping("/{id}")
    public UserResponse partialUpdate(
        @PathVariable Long id,
        @RequestBody Map<String, Object> updates
    ) {
        return userService.patch(id, updates);
    }

    // DELETE /api/v1/users/{id}
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT) // 204
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

### ResponseEntity — Kiểm soát response đầy đủ

```java
@GetMapping("/{id}")
public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
    return userService.findById(id)
        .map(user -> ResponseEntity.ok(user))
        .orElse(ResponseEntity.notFound().build());
}

@PostMapping
public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest req) {
    UserResponse created = userService.create(req);
    URI location = ServletUriComponentsBuilder
        .fromCurrentRequest()
        .path("/{id}")
        .buildAndExpand(created.id())
        .toUri();
    return ResponseEntity.created(location).body(created); // 201 + Location header
}
```

---

## 8. DTO & Validation

### DTO Pattern

```java
// Entity — database model, không expose ra ngoài
@Entity
@Table(name = "users")
public class User {
    @Id @GeneratedValue
    private Long id;
    private String username;
    private String passwordHash; // Không bao giờ trả về password!
    private String email;
    private LocalDateTime createdAt;
}

// Request DTO — nhận từ client
public record CreateUserRequest(
    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 50, message = "Username must be 3-50 characters")
    String username,

    @NotBlank
    @Email(message = "Invalid email format")
    String email,

    @NotBlank
    @Pattern(regexp = "^(?=.*[A-Za-z])(?=.*\\d).{8,}$",
             message = "Password must be at least 8 chars with letters and numbers")
    String password
) {}

// Response DTO — trả về client
public record UserResponse(
    Long id,
    String username,
    String email,
    LocalDateTime createdAt
) {}
```

### Validation Annotations

```java
// javax.validation (Jakarta Validation)
@NotNull           // Không null (nhưng có thể empty string)
@NotBlank          // Không null VÀ không blank (trim xong vẫn có ký tự)
@NotEmpty          // Không null VÀ không empty
@Size(min=1, max=100)  // Độ dài chuỗi / collection
@Min(0) @Max(100)  // Số min/max
@Positive          // Số dương (> 0)
@PositiveOrZero    // Số >= 0
@Email             // Format email
@Pattern(regexp="")// Regex pattern
@Past              // Ngày trong quá khứ
@Future            // Ngày trong tương lai
@DecimalMin("0.0") // Decimal min
```

```java
// Kích hoạt validation với @Valid hoặc @Validated
@PostMapping
public UserResponse create(@Valid @RequestBody CreateUserRequest req) { ... }

// Nested validation
public record OrderRequest(
    @NotNull @Valid ShippingAddress address, // @Valid trigger validation trên object lồng
    @NotEmpty List<@Valid OrderItem> items
) {}
```

### Custom Validator

```java
// 1. Tạo annotation
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueUsernameValidator.class)
public @interface UniqueUsername {
    String message() default "Username already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 2. Implement Validator
@Component
public class UniqueUsernameValidator implements ConstraintValidator<UniqueUsername, String> {
    @Autowired
    private UserRepository userRepo;

    @Override
    public boolean isValid(String username, ConstraintValidatorContext ctx) {
        return !userRepo.existsByUsername(username);
    }
}

// 3. Dùng
public record CreateUserRequest(
    @UniqueUsername
    String username
) {}
```

---

## 9. Exception Handling

### @ControllerAdvice — Global Exception Handler

```java
// Chuẩn bị ErrorResponse DTO
public record ErrorResponse(
    int status,
    String error,
    String message,
    LocalDateTime timestamp,
    Map<String, String> fieldErrors
) {}

@RestControllerAdvice // = @ControllerAdvice + @ResponseBody
public class GlobalExceptionHandler {

    // Handle validation errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidationErrors(MethodArgumentNotValidException ex) {
        Map<String, String> fieldErrors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .collect(Collectors.toMap(
                FieldError::getField,
                fe -> fe.getDefaultMessage() != null ? fe.getDefaultMessage() : "Invalid"
            ));

        return new ErrorResponse(
            400, "Validation Failed", "Invalid request body",
            LocalDateTime.now(), fieldErrors
        );
    }

    // Custom exception
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse(
            404, "Not Found", ex.getMessage(),
            LocalDateTime.now(), null
        );
    }

    // Unauthorized
    @ExceptionHandler(AccessDeniedException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public ErrorResponse handleForbidden(AccessDeniedException ex) {
        return new ErrorResponse(403, "Forbidden", "Access denied",
            LocalDateTime.now(), null);
    }

    // Catch-all
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex) {
        // Log đây — đừng expose stack trace ra client
        log.error("Unhandled exception", ex);
        return new ErrorResponse(
            500, "Internal Server Error", "An unexpected error occurred",
            LocalDateTime.now(), null
        );
    }
}
```

### Custom Exceptions

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String resource, Long id) {
        super(resource + " not found with id: " + id);
    }
}

public class BusinessException extends RuntimeException {
    private final String errorCode;

    public BusinessException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }

    public String getErrorCode() { return errorCode; }
}

// Sử dụng
@Service
public class UserService {
    public User findById(Long id) {
        return userRepo.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User", id));
    }
}
```

---

## 10. Spring Data JPA

### Entity

```java
@Entity
@Table(name = "products",
       indexes = {
           @Index(name = "idx_products_name", columnList = "name"),
           @Index(name = "idx_products_category", columnList = "category_id")
       })
@EntityListeners(AuditingEntityListener.class) // Tự fill createdAt/updatedAt
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 200)
    private String name;

    @Column(precision = 10, scale = 2)
    private BigDecimal price;

    @Enumerated(EnumType.STRING) // Lưu tên enum thay vì số
    private ProductStatus status;

    @ManyToOne(fetch = FetchType.LAZY) // Lazy load — không load category cho đến khi cần
    @JoinColumn(name = "category_id")
    private Category category;

    @OneToMany(mappedBy = "product", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<ProductImage> images = new ArrayList<>();

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    @Version // Optimistic locking
    private Long version;
}
```

### Fetch Type — Rất hay hỏi

```java
// EAGER — Load ngay lập tức cùng với entity cha
@ManyToOne(fetch = FetchType.EAGER) // Default cho @ManyToOne, @OneToOne
private Category category;
// Khi load Product → tự động load Category luôn
// Nguy hiểm với @OneToMany EAGER → N+1 problem

// LAZY — Chỉ load khi access thuộc tính
@OneToMany(fetch = FetchType.LAZY) // Default cho @OneToMany, @ManyToMany
private List<OrderItem> items;
// Khi load Order → KHÔNG load items
// order.getItems() được gọi → mới query items
```

### N+1 Problem & Giải pháp

```java
// N+1 Problem: Load 100 orders → 1 query orders + 100 query items
List<Order> orders = orderRepo.findAll(); // Query 1: lấy orders
orders.forEach(o -> o.getItems().size()); // Query N: mỗi order 1 query items
// Tổng: 101 queries!

// Giải pháp 1: JOIN FETCH
@Query("SELECT o FROM Order o JOIN FETCH o.items WHERE o.status = :status")
List<Order> findByStatusWithItems(@Param("status") OrderStatus status);

// Giải pháp 2: @EntityGraph
@EntityGraph(attributePaths = {"items", "items.product"})
List<Order> findByUserId(Long userId);

// Giải pháp 3: @BatchSize (load theo batch)
@OneToMany
@BatchSize(size = 30) // Load items cho 30 orders một lúc
private List<OrderItem> items;
```

### Cascade Types

```java
// CascadeType.PERSIST — Lưu parent → tự lưu children
// CascadeType.MERGE   — Update parent → tự update children
// CascadeType.REMOVE  — Xóa parent → tự xóa children
// CascadeType.ALL     — Tất cả các loại trên
// orphanRemoval = true → Xóa item khỏi collection → tự xóa khỏi DB

@OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
private List<OrderItem> items;

// Thêm item: order.getItems().add(item) → persist tự động khi save order
// Xóa item: order.getItems().remove(item) → delete tự động khi save order
```

---

## 11. Repository & Query

### JpaRepository Methods

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Có sẵn: save, findById, findAll, deleteById, count, existsById...
}

// Dùng:
userRepo.save(user);                    // INSERT hoặc UPDATE
userRepo.findById(1L);                  // Optional<User>
userRepo.findAll();                     // List<User>
userRepo.findAll(PageRequest.of(0, 10)); // Page<User>
userRepo.deleteById(1L);
userRepo.count();
```

### Query Methods — Derived Query

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Spring tự parse method name thành SQL

    // SELECT * FROM users WHERE email = ?
    Optional<User> findByEmail(String email);

    // SELECT * FROM users WHERE username = ? AND active = ?
    List<User> findByUsernameAndActive(String username, boolean active);

    // SELECT * FROM users WHERE email LIKE '%?%'
    List<User> findByEmailContaining(String keyword);

    // SELECT * FROM users WHERE created_at BETWEEN ? AND ?
    List<User> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end);

    // SELECT * FROM users WHERE age >= ?
    List<User> findByAgeGreaterThanEqual(int age);

    // SELECT * FROM users ORDER BY created_at DESC LIMIT ?
    List<User> findTop10ByOrderByCreatedAtDesc();

    // EXISTS query
    boolean existsByEmail(String email);
    boolean existsByUsernameIgnoreCase(String username);

    // COUNT
    long countByStatus(UserStatus status);

    // DELETE
    void deleteByStatus(UserStatus status);

    // Paging + Sorting
    Page<User> findByStatus(UserStatus status, Pageable pageable);
}

// Sử dụng Paging
Pageable pageable = PageRequest.of(
    0,          // page number (0-based)
    20,         // page size
    Sort.by(Sort.Direction.DESC, "createdAt") // sort
);
Page<User> page = userRepo.findByStatus(UserStatus.ACTIVE, pageable);
page.getContent();  // List<User>
page.getTotalPages();
page.getTotalElements();
page.hasNext();
```

### @Query — Custom JPQL/Native

```java
public interface ProductRepository extends JpaRepository<Product, Long> {

    // JPQL — dùng tên class và field Java (không phải tên table/column)
    @Query("SELECT p FROM Product p WHERE p.price BETWEEN :min AND :max AND p.status = 'ACTIVE'")
    List<Product> findByPriceRange(
        @Param("min") BigDecimal min,
        @Param("max") BigDecimal max
    );

    // Native SQL — dùng tên table/column thật
    @Query(
        value = "SELECT * FROM products p WHERE p.category_id = :catId AND p.stock > 0",
        nativeQuery = true
    )
    List<Product> findInStockByCategory(@Param("catId") Long categoryId);

    // Modifying query — UPDATE/DELETE
    @Modifying
    @Transactional
    @Query("UPDATE Product p SET p.status = :status WHERE p.id IN :ids")
    int updateStatusBatch(@Param("status") ProductStatus status, @Param("ids") List<Long> ids);

    // Projection — chỉ lấy một số field (performance)
    @Query("SELECT new com.myapp.dto.ProductSummary(p.id, p.name, p.price) FROM Product p")
    List<ProductSummary> findAllSummaries();
}
```

### Specification — Dynamic Query

```java
// Dùng khi filter điều kiện động (nhiều filter tùy chọn)
public interface ProductRepository extends JpaRepository<Product, Long>,
    JpaSpecificationExecutor<Product> { }

public class ProductSpecifications {
    public static Specification<Product> hasCategory(Long categoryId) {
        return (root, query, cb) ->
            categoryId == null ? null : cb.equal(root.get("category").get("id"), categoryId);
    }

    public static Specification<Product> priceBetween(BigDecimal min, BigDecimal max) {
        return (root, query, cb) -> {
            if (min == null && max == null) return null;
            if (min == null) return cb.lessThanOrEqualTo(root.get("price"), max);
            if (max == null) return cb.greaterThanOrEqualTo(root.get("price"), min);
            return cb.between(root.get("price"), min, max);
        };
    }
}

// Kết hợp
Specification<Product> spec = Specification
    .where(ProductSpecifications.hasCategory(categoryId))
    .and(ProductSpecifications.priceBetween(minPrice, maxPrice));

List<Product> products = productRepo.findAll(spec);
```

---

## 12. @Transactional

### Transaction là gì?

> Đảm bảo một nhóm operations **thành công tất cả hoặc rollback tất cả** (ACID).

### Cách dùng

```java
@Service
public class TransferService {

    @Transactional // Spring tạo transaction, commit sau method, rollback nếu exception
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        Account from = accountRepo.findById(fromId).orElseThrow();
        Account to = accountRepo.findById(toId).orElseThrow();

        if (from.getBalance().compareTo(amount) < 0) {
            throw new InsufficientFundsException(); // Rollback
        }

        from.setBalance(from.getBalance().subtract(amount));
        to.setBalance(to.getBalance().add(amount));

        accountRepo.save(from); // Nếu lỗi ở đây
        accountRepo.save(to);   // Cả 2 đều rollback
    }
}
```

### Transaction Propagation

```java
// REQUIRED (default): Join existing transaction, hoặc tạo mới
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() {
    methodB(); // methodB dùng transaction của methodA
}

// REQUIRES_NEW: Luôn tạo transaction mới, suspend cái cũ
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void sendNotification() {
    // Chạy trong transaction riêng
    // Dù transaction bên ngoài rollback, cái này vẫn commit
}

// NESTED: Transaction lồng nhau (savepoint)
@Transactional(propagation = Propagation.NESTED)
public void nestedOperation() { ... }

// NOT_SUPPORTED: Suspend transaction, chạy không có transaction
// NEVER: Ném exception nếu có active transaction
// SUPPORTS: Dùng transaction nếu có, không có cũng được
// MANDATORY: Phải có transaction, không có thì exception
```

### Isolation Levels

```java
@Transactional(isolation = Isolation.READ_COMMITTED) // Default PostgreSQL/SQL Server

// READ_UNCOMMITTED — Đọc được dirty data (nhanh nhất, không an toàn)
// READ_COMMITTED   — Chỉ đọc data đã commit (tránh dirty read)
// REPEATABLE_READ  — Cùng query trong 1 tx luôn ra kết quả giống nhau (tránh non-repeatable read)
// SERIALIZABLE     — Transaction chạy tuần tự (chậm nhất, an toàn nhất)
```

### Những lỗi thường gặp với @Transactional

```java
// LỖI 1: Self-invocation — gọi @Transactional method trong cùng class
@Service
public class OrderService {
    public void processOrder(Order order) {
        saveOrder(order); // Gọi internal → KHÔNG có transaction!
        // Vì Spring proxy intercept từ bên ngoài, không intercept internal call
    }

    @Transactional
    public void saveOrder(Order order) { ... } // Transaction không apply
}

// FIX: Inject self (ugly) hoặc tách ra class khác
@Service
public class OrderService {
    @Autowired
    private OrderService self; // Inject proxy

    public void processOrder(Order order) {
        self.saveOrder(order); // Gọi qua proxy → transaction apply
    }
}

// LỖI 2: Chỉ rollback với RuntimeException mặc định
@Transactional // Chỉ rollback với unchecked exception!
public void method() throws IOException {
    // IOException là checked → KHÔNG rollback
}

// FIX:
@Transactional(rollbackFor = Exception.class) // Rollback với mọi exception
public void method() throws IOException { ... }

// LỖI 3: @Transactional trên private method
@Transactional // KHÔNG có effect với private method
private void privateMethod() { ... }
// Spring proxy không thể intercept private method
```

---

## 13. Spring Security

### Security Filter Chain

```
HTTP Request
     ↓
SecurityFilterChain (danh sách filters)
├── UsernamePasswordAuthenticationFilter — xử lý form login
├── JwtAuthenticationFilter (custom) — xử lý JWT
├── BasicAuthenticationFilter — xử lý Basic auth
├── ExceptionTranslationFilter — convert SecurityException → HTTP response
└── FilterSecurityInterceptor — check authorization
     ↓
DispatcherServlet → Controller
```

### Cấu hình Security (Spring Security 6.x)

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity // Bật @PreAuthorize, @PostAuthorize
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtFilter;
    private final UserDetailsService userDetailsService;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            // Tắt CSRF (REST API dùng token, không dùng cookie session)
            .csrf(csrf -> csrf.disable())
            // Stateless session (JWT không cần session)
            .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            // Authorization rules
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()      // Public
                .requestMatchers("/api/admin/**").hasRole("ADMIN") // Admin only
                .requestMatchers(HttpMethod.GET, "/api/products/**").permitAll() // GET public
                .anyRequest().authenticated()                      // Còn lại cần login
            )
            // Thêm JWT filter trước UsernamePasswordAuthenticationFilter
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config)
        throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12); // strength 12 (default 10)
    }
}
```

### UserDetailsService

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepo;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepo.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));

        return org.springframework.security.core.userdetails.User.builder()
            .username(user.getUsername())
            .password(user.getPasswordHash())
            .roles(user.getRoles().stream()
                .map(Role::getName)
                .toArray(String[]::new))
            .accountExpired(!user.isActive())
            .build();
    }
}
```

### Method-level Security

```java
@RestController
public class UserController {

    @PreAuthorize("hasRole('ADMIN')")
    @DeleteMapping("/users/{id}")
    public void deleteUser(@PathVariable Long id) { ... }

    @PreAuthorize("hasRole('ADMIN') or #id == authentication.principal.id")
    @GetMapping("/users/{id}")
    public UserResponse getUser(@PathVariable Long id) { ... }

    @PostAuthorize("returnObject.userId == authentication.principal.id")
    @GetMapping("/orders/{id}")
    public OrderResponse getOrder(@PathVariable Long id) { ... }
}
```

---

## 14. JWT

### JWT Structure

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9  ← Header (base64)
.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4iLCJpYXQiOjE1MTYyMzkwMjJ9  ← Payload (base64)
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← Signature (HMAC/RSA)
```

**Payload chứa claims:**
- `sub` — subject (user id)
- `iat` — issued at
- `exp` — expiration time
- Custom: roles, email...

### JWT Implementation

```java
// pom.xml: io.jsonwebtoken:jjwt-api, jjwt-impl, jjwt-jackson

@Component
public class JwtUtil {
    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration:86400000}") // 24h in ms
    private long expiration;

    private SecretKey getSigningKey() {
        return Keys.hmacShaKeyFor(Decoders.BASE64.decode(secret));
    }

    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList()));

        return Jwts.builder()
            .claims(claims)
            .subject(userDetails.getUsername())
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(getSigningKey())
            .compact();
    }

    public String extractUsername(String token) {
        return extractClaims(token).getSubject();
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    private Claims extractClaims(String token) {
        return Jwts.parser()
            .verifyWith(getSigningKey())
            .build()
            .parseSignedClaims(token)
            .getPayload();
    }

    private boolean isTokenExpired(String token) {
        return extractClaims(token).getExpiration().before(new Date());
    }
}
```

### JWT Filter

```java
@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtUtil jwtUtil;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
        throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        String token = authHeader.substring(7); // Remove "Bearer "

        try {
            String username = jwtUtil.extractUsername(token);

            // Chưa có authentication trong context
            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                if (jwtUtil.isTokenValid(token, userDetails)) {
                    UsernamePasswordAuthenticationToken auth =
                        new UsernamePasswordAuthenticationToken(
                            userDetails, null, userDetails.getAuthorities());
                    auth.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    SecurityContextHolder.getContext().setAuthentication(auth);
                }
            }
        } catch (JwtException e) {
            // Token invalid — không set authentication, request tiếp tục nhưng sẽ bị chặn
        }

        filterChain.doFilter(request, response);
    }
}
```

### Auth Controller

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @PostMapping("/login")
    public TokenResponse login(@Valid @RequestBody LoginRequest request) {
        // AuthenticationManager sẽ gọi UserDetailsService và PasswordEncoder
        Authentication auth = authManager.authenticate(
            new UsernamePasswordAuthenticationToken(request.username(), request.password())
        );
        UserDetails user = (UserDetails) auth.getPrincipal();
        String token = jwtUtil.generateToken(user);
        return new TokenResponse(token, "Bearer", jwtUtil.getExpiration());
    }

    @PostMapping("/register")
    @ResponseStatus(HttpStatus.CREATED)
    public UserResponse register(@Valid @RequestBody RegisterRequest request) {
        return userService.register(request);
    }
}
```

---

## 15. Spring AOP

### Khái niệm

> AOP (Aspect-Oriented Programming) — tách **cross-cutting concerns** (logging, security, transaction, metrics) ra khỏi business logic.

```
Vocabulary:
- Aspect: Module chứa cross-cutting logic (Logging, Audit...)
- Advice: Code thực thi (Before, After, Around...)
- Pointcut: Biểu thức xác định method nào bị intercept
- JoinPoint: Điểm cụ thể khi Advice được gọi (method execution)
- Weaving: Quá trình kết hợp Aspect vào code
```

### Ví dụ thực tế

```java
@Aspect
@Component
public class LoggingAspect {

    private static final Logger log = LoggerFactory.getLogger(LoggingAspect.class);

    // Pointcut expression: tất cả method trong package service
    @Pointcut("execution(* com.myapp.service.*.*(..))")
    public void serviceLayer() {}

    // Before: Chạy trước method
    @Before("serviceLayer()")
    public void logBefore(JoinPoint jp) {
        log.info("Calling: {}.{}({})",
            jp.getTarget().getClass().getSimpleName(),
            jp.getSignature().getName(),
            Arrays.toString(jp.getArgs()));
    }

    // AfterReturning: Chạy sau method thành công
    @AfterReturning(pointcut = "serviceLayer()", returning = "result")
    public void logAfterReturning(JoinPoint jp, Object result) {
        log.info("Method {} returned: {}", jp.getSignature().getName(), result);
    }

    // AfterThrowing: Chạy khi có exception
    @AfterThrowing(pointcut = "serviceLayer()", throwing = "ex")
    public void logException(JoinPoint jp, Exception ex) {
        log.error("Exception in {}: {}", jp.getSignature().getName(), ex.getMessage());
    }

    // Around: Bao quanh method — mạnh nhất
    @Around("execution(* com.myapp.service.*.*(..)) && @annotation(com.myapp.annotation.Timed)")
    public Object measureTime(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            Object result = pjp.proceed(); // Gọi method gốc
            long duration = System.currentTimeMillis() - start;
            log.info("{} took {}ms", pjp.getSignature().getName(), duration);
            return result;
        } catch (Exception e) {
            log.error("{} failed after {}ms", pjp.getSignature().getName(),
                System.currentTimeMillis() - start);
            throw e;
        }
    }
}

// Custom annotation để đánh dấu method cần đo thời gian
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Timed {}

// Dùng
@Service
public class ProductService {
    @Timed
    public List<Product> findAll() { ... }
}
```

---

## 16. Caching

### Spring Cache Abstraction

```java
// Bật caching
@SpringBootApplication
@EnableCaching
public class MyApp { ... }

// application.yml
spring:
  cache:
    type: redis  # simple (in-memory), redis, caffeine, ehcache...
  redis:
    host: localhost
    port: 6379
```

```java
@Service
public class ProductService {

    // Cache kết quả lần đầu, lần sau trả từ cache
    @Cacheable(value = "products", key = "#id")
    public Product findById(Long id) {
        return productRepo.findById(id).orElseThrow(); // Chỉ chạy nếu cache miss
    }

    // Cache với điều kiện
    @Cacheable(value = "products", key = "#name", condition = "#name.length() > 2")
    public List<Product> findByName(String name) { ... }

    // Cập nhật cache khi update
    @CachePut(value = "products", key = "#product.id")
    public Product update(Product product) {
        return productRepo.save(product);
    }

    // Xóa cache khi delete
    @CacheEvict(value = "products", key = "#id")
    public void delete(Long id) {
        productRepo.deleteById(id);
    }

    // Xóa toàn bộ cache
    @CacheEvict(value = "products", allEntries = true)
    public void clearCache() {}
}
```

### Redis với Spring Boot

```java
// pom.xml: spring-boot-starter-data-redis

@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }
}

// Dùng trực tiếp (không qua @Cacheable)
@Service
public class SessionService {
    private final RedisTemplate<String, Object> redisTemplate;

    public void saveSession(String sessionId, UserSession session, Duration ttl) {
        redisTemplate.opsForValue().set("session:" + sessionId, session, ttl);
    }

    public UserSession getSession(String sessionId) {
        return (UserSession) redisTemplate.opsForValue().get("session:" + sessionId);
    }

    public void invalidateSession(String sessionId) {
        redisTemplate.delete("session:" + sessionId);
    }
}
```

---

## 17. Spring Events

### Application Events — Loose coupling giữa các component

```java
// 1. Định nghĩa Event
public record UserRegisteredEvent(
    Long userId,
    String email,
    LocalDateTime registeredAt
) {}

// 2. Publish event
@Service
public class UserService {
    private final ApplicationEventPublisher eventPublisher;

    public User register(RegisterRequest req) {
        User user = createUser(req);
        userRepo.save(user);

        // Publish event — không cần biết ai xử lý
        eventPublisher.publishEvent(new UserRegisteredEvent(
            user.getId(), user.getEmail(), LocalDateTime.now()
        ));

        return user;
    }
}

// 3. Listen event — có thể có nhiều listener
@Component
public class EmailEventListener {
    @EventListener
    public void onUserRegistered(UserRegisteredEvent event) {
        emailService.sendWelcomeEmail(event.email());
    }
}

@Component
public class AuditEventListener {
    @EventListener
    public void onUserRegistered(UserRegisteredEvent event) {
        auditLog.record("USER_REGISTERED", event.userId());
    }
}

// Async event listener — không block main thread
@Component
public class NotificationListener {
    @Async
    @EventListener
    public void onUserRegistered(UserRegisteredEvent event) {
        notificationService.sendPush(event.userId(), "Welcome!");
    }
}
```

### @TransactionalEventListener

```java
// Chỉ xử lý event sau khi transaction commit thành công
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void onOrderPlaced(OrderPlacedEvent event) {
    // Gửi email xác nhận sau khi order đã được lưu vào DB
    emailService.sendOrderConfirmation(event.orderId());
}
```

---

## 18. Profiles

### Cấu hình multi-environment

```java
// Khai báo bean chỉ active ở profile cụ thể
@Profile("dev")
@Component
public class MockPaymentGateway implements PaymentGateway {
    public PaymentResult charge(BigDecimal amount) {
        return PaymentResult.success("mock-txn-id"); // Giả lập khi dev
    }
}

@Profile("prod")
@Component
public class StripePaymentGateway implements PaymentGateway {
    public PaymentResult charge(BigDecimal amount) {
        return stripe.charge(amount); // Real payment
    }
}

// Config class theo profile
@Configuration
@Profile("dev")
public class DevConfig {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
}
```

### Kích hoạt profile

```bash
# Command line
java -jar app.jar --spring.profiles.active=prod

# Environment variable
SPRING_PROFILES_ACTIVE=prod java -jar app.jar

# application.yml
spring:
  profiles:
    active: dev
```

### @ConditionalOnProperty

```java
// Bean chỉ tạo khi property có giá trị cụ thể
@Bean
@ConditionalOnProperty(name = "feature.new-checkout.enabled", havingValue = "true")
public NewCheckoutService newCheckoutService() {
    return new NewCheckoutService();
}
```

---

## 19. Testing

### Unit Test với Mockito

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepo;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private UserService userService;

    @Test
    void findById_whenUserExists_returnsUser() {
        // Arrange
        User user = new User(1L, "john", "john@example.com");
        when(userRepo.findById(1L)).thenReturn(Optional.of(user));

        // Act
        UserResponse result = userService.findById(1L);

        // Assert
        assertThat(result.id()).isEqualTo(1L);
        assertThat(result.username()).isEqualTo("john");
        verify(userRepo, times(1)).findById(1L);
    }

    @Test
    void findById_whenNotFound_throwsException() {
        when(userRepo.findById(99L)).thenReturn(Optional.empty());

        assertThatThrownBy(() -> userService.findById(99L))
            .isInstanceOf(ResourceNotFoundException.class)
            .hasMessageContaining("User not found with id: 99");
    }

    @Test
    void register_shouldHashPasswordAndSendEmail() {
        // Arrange
        CreateUserRequest req = new CreateUserRequest("john", "john@email.com", "password123");
        when(userRepo.existsByEmail(any())).thenReturn(false);
        when(userRepo.save(any())).thenAnswer(invocation -> {
            User u = invocation.getArgument(0);
            u.setId(1L);
            return u;
        });

        // Act
        userService.register(req);

        // Assert
        verify(emailService).sendWelcomeEmail("john@email.com");
        verify(userRepo).save(argThat(u ->
            !u.getPasswordHash().equals("password123") // Password đã được hash
        ));
    }
}
```

### Integration Test với @SpringBootTest

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
class UserControllerIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepo;

    @BeforeEach
    void setup() {
        userRepo.deleteAll();
    }

    @Test
    void createUser_returnsCreatedUser() {
        CreateUserRequest req = new CreateUserRequest("john", "john@email.com", "Pass123!");

        ResponseEntity<UserResponse> response = restTemplate.postForEntity(
            "/api/v1/users", req, UserResponse.class
        );

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody().username()).isEqualTo("john");
        assertThat(userRepo.count()).isEqualTo(1);
    }
}
```

### MockMvc — Test Controller Layer

```java
@WebMvcTest(UserController.class) // Chỉ load web layer, không load service/repo
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean // Mock bean trong Spring context
    private UserService userService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void getUser_returnsOk() throws Exception {
        UserResponse user = new UserResponse(1L, "john", "john@email.com", LocalDateTime.now());
        when(userService.findById(1L)).thenReturn(user);

        mockMvc.perform(get("/api/v1/users/1")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.username").value("john"));
    }

    @Test
    void createUser_withInvalidEmail_returnsBadRequest() throws Exception {
        CreateUserRequest req = new CreateUserRequest("john", "not-an-email", "Pass123!");

        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(req)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.fieldErrors.email").exists());
    }
}
```

### @DataJpaTest — Test Repository Layer

```java
@DataJpaTest // Chỉ load JPA layer, dùng H2 in-memory
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE) // Giữ real DB
@ActiveProfiles("test")
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepo;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void findByEmail_whenExists_returnsUser() {
        User user = new User(null, "john", "john@email.com", "hash");
        entityManager.persist(user);
        entityManager.flush();

        Optional<User> found = userRepo.findByEmail("john@email.com");

        assertThat(found).isPresent();
        assertThat(found.get().getUsername()).isEqualTo("john");
    }
}
```

---

## 20. Actuator

### Spring Boot Actuator — Monitoring

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,env,loggers,threaddump,heapdump
  endpoint:
    health:
      show-details: when-authorized # always, never, when-authorized
  info:
    env:
      enabled: true

info:
  app:
    name: My Application
    version: 1.0.0
    description: Backend service
```

### Endpoints quan trọng

| Endpoint | Mô tả |
|----------|-------|
| `/actuator/health` | Health check — UP/DOWN + components |
| `/actuator/metrics` | JVM metrics, HTTP metrics, custom |
| `/actuator/env` | Environment properties |
| `/actuator/info` | App info |
| `/actuator/loggers` | View/change log level at runtime |
| `/actuator/threaddump` | JVM thread dump |
| `/actuator/heapdump` | JVM heap dump |
| `/actuator/beans` | Tất cả beans được load |
| `/actuator/mappings` | Tất cả URL mappings |

### Custom Health Indicator

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {

    private final DataSource dataSource;

    @Override
    public Health health() {
        try (Connection conn = dataSource.getConnection()) {
            boolean valid = conn.isValid(2);
            if (valid) {
                return Health.up()
                    .withDetail("database", "PostgreSQL")
                    .withDetail("status", "reachable")
                    .build();
            }
            return Health.down().withDetail("reason", "Connection invalid").build();
        } catch (SQLException e) {
            return Health.down(e).withDetail("reason", e.getMessage()).build();
        }
    }
}
```

---

## 21. Microservices & Spring Cloud

### Các thành phần cơ bản

```
Microservices Architecture:
├── API Gateway (Spring Cloud Gateway)
│   └── Routing, Auth, Rate limiting, Load balancing
├── Service Discovery (Eureka / Consul)
│   └── Services tự đăng ký, tự tìm nhau
├── Config Server (Spring Cloud Config)
│   └── Quản lý config tập trung từ Git
├── Load Balancer (Spring Cloud LoadBalancer)
│   └── Phân tải giữa các instance
├── Circuit Breaker (Resilience4j)
│   └── Ngừng call service lỗi, fallback
└── Distributed Tracing (Micrometer + Zipkin)
    └── Trace request qua nhiều service
```

### Service Discovery với Eureka

```java
// Eureka Server
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApp { ... }

// Client service — tự đăng ký với Eureka
@SpringBootApplication
@EnableEurekaClient
public class ProductServiceApp { ... }
```

```yaml
# product-service application.yml
spring:
  application:
    name: product-service  # Tên đăng ký trên Eureka

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
```

### RestTemplate / WebClient — Gọi giữa services

```java
// Dùng service name thay vì URL cứng
@Service
public class OrderService {
    private final RestTemplate restTemplate;

    // @LoadBalanced — tự tìm URL từ Eureka và load balance
    public ProductResponse getProduct(Long productId) {
        return restTemplate.getForObject(
            "http://product-service/api/products/" + productId, // "product-service" là tên service
            ProductResponse.class
        );
    }
}

// Config RestTemplate với @LoadBalanced
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

### Circuit Breaker với Resilience4j

```java
// Tránh cascade failure: service A gọi B → B down → A bị treo → C gọi A bị treo...
// Circuit Breaker: Sau N lần lỗi → "mở mạch" → ngay lập tức trả fallback

@Service
public class ProductService {

    @CircuitBreaker(name = "productService", fallbackMethod = "getProductFallback")
    @Retry(name = "productService") // Retry trước khi circuit breaker mở
    @TimeLimiter(name = "productService") // Timeout
    public ProductResponse getProduct(Long id) {
        return restTemplate.getForObject("http://product-service/api/products/" + id,
            ProductResponse.class);
    }

    // Fallback method — phải có cùng signature + Throwable
    public ProductResponse getProductFallback(Long id, Throwable ex) {
        log.warn("Product service down, returning cached data for {}", id);
        return ProductResponse.placeholder(id); // Trả data placeholder
    }
}
```

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      productService:
        sliding-window-size: 10       # Xét 10 request gần nhất
        failure-rate-threshold: 50    # Mở circuit khi 50% fail
        wait-duration-in-open-state: 30s # Thử lại sau 30s
        permitted-number-of-calls-in-half-open-state: 3
  retry:
    instances:
      productService:
        max-attempts: 3
        wait-duration: 1s
```

---

## 22. Câu hỏi hay gặp

| Câu hỏi | Trả lời ngắn gọn |
|---------|-----------------|
| Spring Boot auto-config hoạt động thế nào? | `@EnableAutoConfiguration` scan `AutoConfiguration.imports`, dùng `@ConditionalOn*` để quyết định có tạo bean không |
| Bean scope mặc định là gì? | `singleton` — 1 instance cho toàn app |
| @Component vs @Service vs @Repository | Kỹ thuật như nhau; @Repository thêm exception translation; @Service là semantic |
| @RestController vs @Controller | @RestController = @Controller + @ResponseBody — tất cả method trả JSON |
| @Transactional rollback khi nào? | Mặc định chỉ với RuntimeException. Dùng `rollbackFor = Exception.class` cho checked |
| N+1 problem là gì? | Load N entity → N thêm query cho relationship. Fix bằng JOIN FETCH / @EntityGraph |
| EAGER vs LAZY fetch? | EAGER load ngay; LAZY load khi access. Default: @ManyToOne/@OneToOne=EAGER, @OneToMany=LAZY |
| Self-invocation @Transactional không work? | Spring AOP proxy chỉ intercept call từ bên ngoài. Fix: inject self hoặc tách class |
| JWT vs Session? | JWT stateless (server không lưu), scale tốt; Session stateful (server lưu) |
| Constructor vs Field Injection? | Constructor: immutable, testable, fail-fast. Field: tiện nhưng không nên dùng prod |
| Spring Security filter chain? | Danh sách filter xử lý auth/authz trước khi request vào controller |
| @Cacheable vs @CachePut? | @Cacheable: đọc cache trước; @CachePut: luôn chạy method và update cache |
| Circular dependency là gì? | A inject B, B inject A → Spring không biết tạo cái nào trước. Fix: @Lazy hoặc refactor |
| ApplicationContext vs BeanFactory? | BeanFactory là base; ApplicationContext là advanced (events, i18n, AOP). Luôn dùng ApplicationContext |
| ddl-auto các giá trị? | `none`, `validate`, `update`, `create`, `create-drop`. Production dùng `validate` hoặc `none` |

---

*README này cover ~90% câu hỏi Spring Boot trong interview — chúc bạn ace! 🚀*