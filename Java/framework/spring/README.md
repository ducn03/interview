\# 🍃 SPRING BOOT - INTERVIEW CHEATSHEET



> \*\*Phiên bản đầy đủ nhất\*\* — đọc phát hiểu luôn, không cần tra thêm  

> Spring Boot 3.x | Spring 6 | Java 17+



\---



\## 📋 MỤC LỤC



1\. \[Spring Framework là gì? Spring vs Spring Boot](#1-spring-vs-spring-boot)

2\. \[IoC Container \& Dependency Injection](#2-ioc--dependency-injection)

3\. \[Bean — Lifecycle, Scope, Annotations](#3-bean)

4\. \[Auto-Configuration — Magic bên trong](#4-auto-configuration)

5\. \[application.properties / application.yml](#5-configuration)

6\. \[Spring MVC — Request Flow](#6-spring-mvc)

7\. \[REST API — Controllers, Request Mapping](#7-rest-api)

8\. \[Request/Response — DTO, Validation](#8-dto--validation)

9\. \[Exception Handling — @ControllerAdvice](#9-exception-handling)

10\. \[Spring Data JPA](#10-spring-data-jpa)

11\. \[Repository Pattern — JpaRepository, Query Methods](#11-repository--query)

12\. \[Transaction Management — @Transactional](#12-transactional)

13\. \[Spring Security — Authentication \& Authorization](#13-spring-security)

14\. \[JWT Authentication Flow](#14-jwt)

15\. \[Spring AOP — Aspect Oriented Programming](#15-spring-aop)

16\. \[Caching — @Cacheable, Redis](#16-caching)

17\. \[Spring Events](#17-spring-events)

18\. \[Profiles — dev, staging, prod](#18-profiles)

19\. \[Testing — Unit \& Integration Tests](#19-testing)

20\. \[Actuator — Monitoring \& Health](#20-actuator)

21\. \[Microservices — Spring Cloud cơ bản](#21-microservices--spring-cloud)

22\. \[Câu hỏi phỏng vấn hay gặp](#22-câu-hỏi-hay-gặp)



\---



\## 1. Spring vs Spring Boot



\### Spring Framework là gì?



Spring Framework là một \*\*Java application framework\*\* cung cấp:

\- \*\*IoC Container\*\* — quản lý object lifecycle

\- \*\*Dependency Injection\*\* — inject dependency tự động

\- \*\*AOP\*\* — cross-cutting concerns (logging, transaction...)

\- \*\*Spring MVC\*\* — web layer

\- \*\*Spring Data\*\* — data access layer



\*\*Vấn đề của Spring thuần:\*\*

\- Cấu hình XML/Java quá nhiều (boilerplate)

\- Phải tự cấu hình từng component (DataSource, Tomcat, Jackson...)

\- Nhiều dependency conflict



\### Spring Boot giải quyết gì?



```

Spring Boot = Spring Framework

&#x20;          + Auto-Configuration (tự cấu hình)

&#x20;          + Embedded Server (Tomcat built-in)

&#x20;          + Starter Dependencies (bundle dependency)

&#x20;          + Opinionated Defaults (convention over configuration)

```



```java

// Spring MVC thuần — cần \~50 dòng XML config

// web.xml, dispatcher-servlet.xml, root-context.xml...



// Spring Boot — chỉ cần:

@SpringBootApplication

public class MyApp {

&#x20;   public static void main(String\[] args) {

&#x20;       SpringApplication.run(MyApp.class, args);

&#x20;   }

}

// Tự động: embedded Tomcat, component scan, auto-config

```



\### @SpringBootApplication là gì?



```java

@SpringBootApplication

// = @Configuration + @EnableAutoConfiguration + @ComponentScan



// @Configuration: class này là nguồn khai báo bean

// @EnableAutoConfiguration: bật auto-config mechanism

// @ComponentScan: scan tất cả @Component trong package hiện tại và con

```



\---



\## 2. IoC \& Dependency Injection



\### IoC (Inversion of Control)



> \*\*Bình thường:\*\* Code của bạn tạo object → bạn kiểm soát  

> \*\*IoC:\*\* Framework tạo \& quản lý object → framework kiểm soát  

> Bạn chỉ "khai báo" cần gì, Spring lo phần còn lại



```java

// Không có IoC — tightly coupled

public class OrderService {

&#x20;   private EmailService emailService = new EmailService(); // Tự tạo

&#x20;   private OrderRepository repo = new OrderRepository();   // Tự tạo

}



// Có IoC — loosely coupled

@Service

public class OrderService {

&#x20;   private final EmailService emailService;   // Spring inject

&#x20;   private final OrderRepository repo;        // Spring inject



&#x20;   public OrderService(EmailService emailService, OrderRepository repo) {

&#x20;       this.emailService = emailService;

&#x20;       this.repo = repo;

&#x20;   }

}

```



\### 3 kiểu Dependency Injection



```java

// 1. Constructor Injection — RECOMMENDED

@Service

public class UserService {

&#x20;   private final UserRepository userRepo;

&#x20;   private final EmailService emailService;



&#x20;   // @Autowired không cần nếu chỉ có 1 constructor (Spring 4.3+)

&#x20;   public UserService(UserRepository userRepo, EmailService emailService) {

&#x20;       this.userRepo = userRepo;

&#x20;       this.emailService = emailService;

&#x20;   }

}

// Ưu điểm: immutable field, dễ test (pass mock qua constructor), fail fast



// 2. Setter Injection — Dùng cho optional dependencies

@Service

public class UserService {

&#x20;   private NotificationService notificationService;



&#x20;   @Autowired(required = false) // Optional dependency

&#x20;   public void setNotificationService(NotificationService ns) {

&#x20;       this.notificationService = ns;

&#x20;   }

}



// 3. Field Injection — Không nên dùng trong production

@Service

public class UserService {

&#x20;   @Autowired // Không thể dùng final, khó test, che giấu dependency

&#x20;   private UserRepository userRepo;

}

```



\*\*Tại sao Constructor Injection được recommend?\*\*

\- Fields có thể là `final` → immutable, thread-safe hơn

\- Dependencies rõ ràng — nhìn vào constructor biết ngay cần gì

\- Dễ viết unit test không cần Spring context (chỉ cần `new Service(mockRepo)`)

\- Phát hiện circular dependency sớm hơn (lúc startup)



\### @Autowired hoạt động thế nào?



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

&#x20;   private final EmailService emailService;



&#x20;   public OrderService(@Qualifier("premiumEmailService") EmailService emailService) {

&#x20;       this.emailService = emailService;

&#x20;   }

}



// Hoặc dùng @Primary — bean này được ưu tiên khi inject

@Primary

@Service

public class PremiumEmailService implements EmailService { ... }

```



\---



\## 3. Bean



\### Bean là gì?



> \*\*Bean\*\* = object được \*\*Spring IoC Container\*\* quản lý (tạo, configure, wire, destroy).



\### Khai báo Bean



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

&#x20;   @Bean

&#x20;   public ObjectMapper objectMapper() {

&#x20;       return new ObjectMapper()

&#x20;           .configure(SerializationFeature.WRITE\_DATES\_AS\_TIMESTAMPS, false);

&#x20;   }



&#x20;   @Bean

&#x20;   public RestTemplate restTemplate() {

&#x20;       return new RestTemplate();

&#x20;   }

}



// Cách 3: @Bean với init và destroy

@Bean(initMethod = "init", destroyMethod = "cleanup")

public SomeService someService() {

&#x20;   return new SomeService();

}

```



\### Bean Scope



| Scope | Mô tả | Dùng khi |

|-------|-------|----------|

| \*\*singleton\*\* | 1 instance cho toàn app (default) | Stateless service |

| \*\*prototype\*\* | Tạo mới mỗi lần inject/request | Stateful object |

| \*\*request\*\* | 1 instance per HTTP request | Web app |

| \*\*session\*\* | 1 instance per HTTP session | Web app, user data |

| \*\*application\*\* | 1 instance per ServletContext | Web app global |



```java

@Scope(ConfigurableBeanFactory.SCOPE\_PROTOTYPE)

@Component

public class ShoppingCart { // Mỗi user cần 1 cart riêng

&#x20;   private List<Item> items = new ArrayList<>();

}



// Vấn đề: Inject prototype vào singleton

@Service // singleton

public class OrderService {

&#x20;   @Autowired

&#x20;   private ShoppingCart cart; // Chỉ inject 1 lần — cùng instance mãi!

}



// Giải pháp: ApplicationContext lookup, hoặc @Lookup

@Service

public class OrderService {

&#x20;   @Autowired

&#x20;   private ApplicationContext context;



&#x20;   public void processOrder() {

&#x20;       ShoppingCart cart = context.getBean(ShoppingCart.class); // Fresh bean

&#x20;   }

}

```



\### Bean Lifecycle



```

Container startup

&#x20;     ↓

1\. Instantiate (gọi constructor)

&#x20;     ↓

2\. Populate Properties (inject dependencies)

&#x20;     ↓

3\. BeanNameAware.setBeanName()

&#x20;     ↓

4\. BeanFactoryAware.setBeanFactory()

&#x20;     ↓

5\. ApplicationContextAware.setApplicationContext()

&#x20;     ↓

6\. @PostConstruct / InitializingBean.afterPropertiesSet()

&#x20;     ↓

7\. Bean sẵn sàng sử dụng

&#x20;     ↓

Container shutdown

&#x20;     ↓

8\. @PreDestroy / DisposableBean.destroy()

```



```java

@Service

public class DatabaseService {

&#x20;   private Connection connection;



&#x20;   @PostConstruct

&#x20;   public void init() {

&#x20;       // Chạy sau khi inject xong — khởi tạo connection pool

&#x20;       connection = createConnection();

&#x20;       System.out.println("DatabaseService initialized");

&#x20;   }



&#x20;   @PreDestroy

&#x20;   public void cleanup() {

&#x20;       // Chạy trước khi bean bị destroy

&#x20;       if (connection != null) connection.close();

&#x20;       System.out.println("DatabaseService destroyed");

&#x20;   }

}

```



\---



\## 4. Auto-Configuration



\### Spring Boot tự cấu hình thế nào?



```

1\. @EnableAutoConfiguration scan META-INF/spring/

&#x20;  org.springframework.boot.autoconfigure.AutoConfiguration.imports



2\. Hàng trăm AutoConfiguration classes được load



3\. Mỗi class có @ConditionalOn... để check điều kiện:

&#x20;  - @ConditionalOnClass: class X có trong classpath không?

&#x20;  - @ConditionalOnMissingBean: bean Y đã được define chưa?

&#x20;  - @ConditionalOnProperty: property Z có được set không?



4\. Nếu điều kiện thỏa → tạo bean mặc định

```



```java

// Ví dụ DataSourceAutoConfiguration (simplified)

@AutoConfiguration

@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })

@ConditionalOnMissingBean(type = "io.r2dbc.spi.ConnectionFactory")

@EnableConfigurationProperties(DataSourceProperties.class)

public class DataSourceAutoConfiguration {



&#x20;   @Bean

&#x20;   @ConditionalOnMissingBean // Chỉ tạo nếu bạn CHƯA define DataSource

&#x20;   public DataSource dataSource(DataSourceProperties properties) {

&#x20;       return properties.initializeDataSourceBuilder().build();

&#x20;   }

}

```



\*\*Vậy khi nào auto-config bị override?\*\*

\- Khi bạn tự khai báo `@Bean` cùng type → `@ConditionalOnMissingBean` không thỏa → auto-config không tạo bean nữa



```java

// Bạn muốn config DataSource riêng:

@Configuration

public class DatabaseConfig {

&#x20;   @Bean // Tự khai báo → Spring Boot không dùng default nữa

&#x20;   public DataSource dataSource() {

&#x20;       HikariDataSource ds = new HikariDataSource();

&#x20;       ds.setJdbcUrl("jdbc:postgresql://localhost/mydb");

&#x20;       ds.setMaximumPoolSize(20);

&#x20;       return ds;

&#x20;   }

}

```



\### Debug Auto-Configuration



```bash

\# Xem auto-config nào được apply và tại sao

java -jar app.jar --debug



\# Hoặc trong application.properties:

debug=true

```



\---



\## 5. Configuration



\### application.properties vs application.yml



```properties

\# application.properties

server.port=8080

spring.datasource.url=jdbc:postgresql://localhost:5432/mydb

spring.datasource.username=postgres

spring.datasource.password=secret

spring.jpa.hibernate.ddl-auto=update

spring.jpa.show-sql=true

logging.level.com.myapp=DEBUG

```



```yaml

\# application.yml — cấu trúc rõ hơn khi có nhiều property lồng nhau

server:

&#x20; port: 8080



spring:

&#x20; datasource:

&#x20;   url: jdbc:postgresql://localhost:5432/mydb

&#x20;   username: postgres

&#x20;   password: secret

&#x20; jpa:

&#x20;   hibernate:

&#x20;     ddl-auto: update

&#x20;   show-sql: true



logging:

&#x20; level:

&#x20;   com.myapp: DEBUG

```



\### @Value — Inject single property



```java

@Service

public class PaymentService {

&#x20;   @Value("${payment.api.key}")

&#x20;   private String apiKey;



&#x20;   @Value("${payment.timeout:5000}") // Default value nếu không có

&#x20;   private int timeout;



&#x20;   @Value("${app.features.list}") // Inject list

&#x20;   private List<String> features;

}

```



\### @ConfigurationProperties — Inject nhóm properties (Recommended)



```yaml

\# application.yml

app:

&#x20; payment:

&#x20;   api-key: sk\_live\_abc123

&#x20;   timeout: 5000

&#x20;   retry-count: 3

&#x20;   base-url: https://payment.example.com

```



```java

@ConfigurationProperties(prefix = "app.payment")

@Component // hoặc dùng @EnableConfigurationProperties(PaymentProperties.class)

public class PaymentProperties {

&#x20;   private String apiKey;

&#x20;   private int timeout;

&#x20;   private int retryCount;

&#x20;   private String baseUrl;



&#x20;   // Getters \& setters (hoặc dùng @ConfigurationProperties + Record)

}



// Java 17+ — dùng Record

@ConfigurationProperties(prefix = "app.payment")

public record PaymentProperties(

&#x20;   String apiKey,

&#x20;   int timeout,

&#x20;   int retryCount,

&#x20;   String baseUrl

) {}



// Inject và dùng

@Service

public class PaymentService {

&#x20;   private final PaymentProperties props;



&#x20;   public PaymentService(PaymentProperties props) {

&#x20;       this.props = props;

&#x20;   }



&#x20;   public void pay() {

&#x20;       // props.apiKey(), props.timeout()...

&#x20;   }

}

```



\### Environment-specific config



```

src/main/resources/

├── application.yml           # Config chung

├── application-dev.yml       # Override cho dev

├── application-staging.yml   # Override cho staging

└── application-prod.yml      # Override cho prod

```



```yaml

\# application-prod.yml

spring:

&#x20; datasource:

&#x20;   url: ${DATABASE\_URL}  # Đọc từ env variable

&#x20;   username: ${DB\_USER}

&#x20;   password: ${DB\_PASSWORD}

&#x20; jpa:

&#x20;   show-sql: false

logging:

&#x20; level:

&#x20;   root: WARN

```



\---



\## 6. Spring MVC



\### Request Flow



```

HTTP Request

&#x20;    ↓

DispatcherServlet (Front Controller)

&#x20;    ↓

HandlerMapping → Tìm @Controller phù hợp với URL

&#x20;    ↓

HandlerAdapter → Gọi method trong Controller

&#x20;    ↓ (nếu có @ExceptionHandler/@ControllerAdvice → bắt exception)

Controller Method → Xử lý business logic

&#x20;    ↓

ViewResolver (REST API → MessageConverter thay vì View)

&#x20;    ↓

HTTP Response (JSON/XML)

```



\### DispatcherServlet



> Là \*\*Front Controller\*\* — 1 servlet duy nhất nhận toàn bộ request, sau đó route đến đúng handler.



Spring Boot tự đăng ký DispatcherServlet tại `/` (map tất cả request).



\---



\## 7. REST API



\### Controller Annotations



```java

@RestController // = @Controller + @ResponseBody

@RequestMapping("/api/v1/users") // Base path

public class UserController {



&#x20;   private final UserService userService;



&#x20;   public UserController(UserService userService) {

&#x20;       this.userService = userService;

&#x20;   }



&#x20;   // GET /api/v1/users

&#x20;   @GetMapping

&#x20;   public List<UserResponse> getAllUsers() {

&#x20;       return userService.findAll();

&#x20;   }



&#x20;   // GET /api/v1/users/{id}

&#x20;   @GetMapping("/{id}")

&#x20;   public UserResponse getUserById(@PathVariable Long id) {

&#x20;       return userService.findById(id);

&#x20;   }



&#x20;   // GET /api/v1/users?page=0\&size=10\&sort=name

&#x20;   @GetMapping("/search")

&#x20;   public Page<UserResponse> searchUsers(

&#x20;       @RequestParam(defaultValue = "") String name,

&#x20;       @RequestParam(defaultValue = "0") int page,

&#x20;       @RequestParam(defaultValue = "10") int size

&#x20;   ) {

&#x20;       return userService.search(name, page, size);

&#x20;   }



&#x20;   // POST /api/v1/users

&#x20;   @PostMapping

&#x20;   @ResponseStatus(HttpStatus.CREATED) // 201

&#x20;   public UserResponse createUser(@Valid @RequestBody CreateUserRequest request) {

&#x20;       return userService.create(request);

&#x20;   }



&#x20;   // PUT /api/v1/users/{id}

&#x20;   @PutMapping("/{id}")

&#x20;   public UserResponse updateUser(

&#x20;       @PathVariable Long id,

&#x20;       @Valid @RequestBody UpdateUserRequest request

&#x20;   ) {

&#x20;       return userService.update(id, request);

&#x20;   }



&#x20;   // PATCH /api/v1/users/{id}

&#x20;   @PatchMapping("/{id}")

&#x20;   public UserResponse partialUpdate(

&#x20;       @PathVariable Long id,

&#x20;       @RequestBody Map<String, Object> updates

&#x20;   ) {

&#x20;       return userService.patch(id, updates);

&#x20;   }



&#x20;   // DELETE /api/v1/users/{id}

&#x20;   @DeleteMapping("/{id}")

&#x20;   @ResponseStatus(HttpStatus.NO\_CONTENT) // 204

&#x20;   public void deleteUser(@PathVariable Long id) {

&#x20;       userService.delete(id);

&#x20;   }

}

```



\### ResponseEntity — Kiểm soát response đầy đủ



```java

@GetMapping("/{id}")

public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {

&#x20;   return userService.findById(id)

&#x20;       .map(user -> ResponseEntity.ok(user))

&#x20;       .orElse(ResponseEntity.notFound().build());

}



@PostMapping

public ResponseEntity<UserResponse> createUser(@Valid @RequestBody CreateUserRequest req) {

&#x20;   UserResponse created = userService.create(req);

&#x20;   URI location = ServletUriComponentsBuilder

&#x20;       .fromCurrentRequest()

&#x20;       .path("/{id}")

&#x20;       .buildAndExpand(created.id())

&#x20;       .toUri();

&#x20;   return ResponseEntity.created(location).body(created); // 201 + Location header

}

```



\---



\## 8. DTO \& Validation



\### DTO Pattern



```java

// Entity — database model, không expose ra ngoài

@Entity

@Table(name = "users")

public class User {

&#x20;   @Id @GeneratedValue

&#x20;   private Long id;

&#x20;   private String username;

&#x20;   private String passwordHash; // Không bao giờ trả về password!

&#x20;   private String email;

&#x20;   private LocalDateTime createdAt;

}



// Request DTO — nhận từ client

public record CreateUserRequest(

&#x20;   @NotBlank(message = "Username is required")

&#x20;   @Size(min = 3, max = 50, message = "Username must be 3-50 characters")

&#x20;   String username,



&#x20;   @NotBlank

&#x20;   @Email(message = "Invalid email format")

&#x20;   String email,



&#x20;   @NotBlank

&#x20;   @Pattern(regexp = "^(?=.\*\[A-Za-z])(?=.\*\\\\d).{8,}$",

&#x20;            message = "Password must be at least 8 chars with letters and numbers")

&#x20;   String password

) {}



// Response DTO — trả về client

public record UserResponse(

&#x20;   Long id,

&#x20;   String username,

&#x20;   String email,

&#x20;   LocalDateTime createdAt

) {}

```



\### Validation Annotations



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

&#x20;   @NotNull @Valid ShippingAddress address, // @Valid trigger validation trên object lồng

&#x20;   @NotEmpty List<@Valid OrderItem> items

) {}

```



\### Custom Validator



```java

// 1. Tạo annotation

@Target({ElementType.FIELD})

@Retention(RetentionPolicy.RUNTIME)

@Constraint(validatedBy = UniqueUsernameValidator.class)

public @interface UniqueUsername {

&#x20;   String message() default "Username already exists";

&#x20;   Class<?>\[] groups() default {};

&#x20;   Class<? extends Payload>\[] payload() default {};

}



// 2. Implement Validator

@Component

public class UniqueUsernameValidator implements ConstraintValidator<UniqueUsername, String> {

&#x20;   @Autowired

&#x20;   private UserRepository userRepo;



&#x20;   @Override

&#x20;   public boolean isValid(String username, ConstraintValidatorContext ctx) {

&#x20;       return !userRepo.existsByUsername(username);

&#x20;   }

}



// 3. Dùng

public record CreateUserRequest(

&#x20;   @UniqueUsername

&#x20;   String username

) {}

```



\---



\## 9. Exception Handling



\### @ControllerAdvice — Global Exception Handler



```java

// Chuẩn bị ErrorResponse DTO

public record ErrorResponse(

&#x20;   int status,

&#x20;   String error,

&#x20;   String message,

&#x20;   LocalDateTime timestamp,

&#x20;   Map<String, String> fieldErrors

) {}



@RestControllerAdvice // = @ControllerAdvice + @ResponseBody

public class GlobalExceptionHandler {



&#x20;   // Handle validation errors

&#x20;   @ExceptionHandler(MethodArgumentNotValidException.class)

&#x20;   @ResponseStatus(HttpStatus.BAD\_REQUEST)

&#x20;   public ErrorResponse handleValidationErrors(MethodArgumentNotValidException ex) {

&#x20;       Map<String, String> fieldErrors = ex.getBindingResult()

&#x20;           .getFieldErrors()

&#x20;           .stream()

&#x20;           .collect(Collectors.toMap(

&#x20;               FieldError::getField,

&#x20;               fe -> fe.getDefaultMessage() != null ? fe.getDefaultMessage() : "Invalid"

&#x20;           ));



&#x20;       return new ErrorResponse(

&#x20;           400, "Validation Failed", "Invalid request body",

&#x20;           LocalDateTime.now(), fieldErrors

&#x20;       );

&#x20;   }



&#x20;   // Custom exception

&#x20;   @ExceptionHandler(ResourceNotFoundException.class)

&#x20;   @ResponseStatus(HttpStatus.NOT\_FOUND)

&#x20;   public ErrorResponse handleNotFound(ResourceNotFoundException ex) {

&#x20;       return new ErrorResponse(

&#x20;           404, "Not Found", ex.getMessage(),

&#x20;           LocalDateTime.now(), null

&#x20;       );

&#x20;   }



&#x20;   // Unauthorized

&#x20;   @ExceptionHandler(AccessDeniedException.class)

&#x20;   @ResponseStatus(HttpStatus.FORBIDDEN)

&#x20;   public ErrorResponse handleForbidden(AccessDeniedException ex) {

&#x20;       return new ErrorResponse(403, "Forbidden", "Access denied",

&#x20;           LocalDateTime.now(), null);

&#x20;   }



&#x20;   // Catch-all

&#x20;   @ExceptionHandler(Exception.class)

&#x20;   @ResponseStatus(HttpStatus.INTERNAL\_SERVER\_ERROR)

&#x20;   public ErrorResponse handleGeneral(Exception ex) {

&#x20;       // Log đây — đừng expose stack trace ra client

&#x20;       log.error("Unhandled exception", ex);

&#x20;       return new ErrorResponse(

&#x20;           500, "Internal Server Error", "An unexpected error occurred",

&#x20;           LocalDateTime.now(), null

&#x20;       );

&#x20;   }

}

```



\### Custom Exceptions



```java

public class ResourceNotFoundException extends RuntimeException {

&#x20;   public ResourceNotFoundException(String resource, Long id) {

&#x20;       super(resource + " not found with id: " + id);

&#x20;   }

}



public class BusinessException extends RuntimeException {

&#x20;   private final String errorCode;



&#x20;   public BusinessException(String errorCode, String message) {

&#x20;       super(message);

&#x20;       this.errorCode = errorCode;

&#x20;   }



&#x20;   public String getErrorCode() { return errorCode; }

}



// Sử dụng

@Service

public class UserService {

&#x20;   public User findById(Long id) {

&#x20;       return userRepo.findById(id)

&#x20;           .orElseThrow(() -> new ResourceNotFoundException("User", id));

&#x20;   }

}

```



\---



\## 10. Spring Data JPA



\### Entity



```java

@Entity

@Table(name = "products",

&#x20;      indexes = {

&#x20;          @Index(name = "idx\_products\_name", columnList = "name"),

&#x20;          @Index(name = "idx\_products\_category", columnList = "category\_id")

&#x20;      })

@EntityListeners(AuditingEntityListener.class) // Tự fill createdAt/updatedAt

public class Product {



&#x20;   @Id

&#x20;   @GeneratedValue(strategy = GenerationType.IDENTITY)

&#x20;   private Long id;



&#x20;   @Column(nullable = false, length = 200)

&#x20;   private String name;



&#x20;   @Column(precision = 10, scale = 2)

&#x20;   private BigDecimal price;



&#x20;   @Enumerated(EnumType.STRING) // Lưu tên enum thay vì số

&#x20;   private ProductStatus status;



&#x20;   @ManyToOne(fetch = FetchType.LAZY) // Lazy load — không load category cho đến khi cần

&#x20;   @JoinColumn(name = "category\_id")

&#x20;   private Category category;



&#x20;   @OneToMany(mappedBy = "product", cascade = CascadeType.ALL, orphanRemoval = true)

&#x20;   private List<ProductImage> images = new ArrayList<>();



&#x20;   @CreatedDate

&#x20;   @Column(updatable = false)

&#x20;   private LocalDateTime createdAt;



&#x20;   @LastModifiedDate

&#x20;   private LocalDateTime updatedAt;



&#x20;   @Version // Optimistic locking

&#x20;   private Long version;

}

```



\### Fetch Type — Rất hay hỏi



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



\### N+1 Problem \& Giải pháp



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



\### Cascade Types



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



\---



\## 11. Repository \& Query



\### JpaRepository Methods



```java

public interface UserRepository extends JpaRepository<User, Long> {

&#x20;   // Có sẵn: save, findById, findAll, deleteById, count, existsById...

}



// Dùng:

userRepo.save(user);                    // INSERT hoặc UPDATE

userRepo.findById(1L);                  // Optional<User>

userRepo.findAll();                     // List<User>

userRepo.findAll(PageRequest.of(0, 10)); // Page<User>

userRepo.deleteById(1L);

userRepo.count();

```



\### Query Methods — Derived Query



```java

public interface UserRepository extends JpaRepository<User, Long> {

&#x20;   // Spring tự parse method name thành SQL



&#x20;   // SELECT \* FROM users WHERE email = ?

&#x20;   Optional<User> findByEmail(String email);



&#x20;   // SELECT \* FROM users WHERE username = ? AND active = ?

&#x20;   List<User> findByUsernameAndActive(String username, boolean active);



&#x20;   // SELECT \* FROM users WHERE email LIKE '%?%'

&#x20;   List<User> findByEmailContaining(String keyword);



&#x20;   // SELECT \* FROM users WHERE created\_at BETWEEN ? AND ?

&#x20;   List<User> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end);



&#x20;   // SELECT \* FROM users WHERE age >= ?

&#x20;   List<User> findByAgeGreaterThanEqual(int age);



&#x20;   // SELECT \* FROM users ORDER BY created\_at DESC LIMIT ?

&#x20;   List<User> findTop10ByOrderByCreatedAtDesc();



&#x20;   // EXISTS query

&#x20;   boolean existsByEmail(String email);

&#x20;   boolean existsByUsernameIgnoreCase(String username);



&#x20;   // COUNT

&#x20;   long countByStatus(UserStatus status);



&#x20;   // DELETE

&#x20;   void deleteByStatus(UserStatus status);



&#x20;   // Paging + Sorting

&#x20;   Page<User> findByStatus(UserStatus status, Pageable pageable);

}



// Sử dụng Paging

Pageable pageable = PageRequest.of(

&#x20;   0,          // page number (0-based)

&#x20;   20,         // page size

&#x20;   Sort.by(Sort.Direction.DESC, "createdAt") // sort

);

Page<User> page = userRepo.findByStatus(UserStatus.ACTIVE, pageable);

page.getContent();  // List<User>

page.getTotalPages();

page.getTotalElements();

page.hasNext();

```



\### @Query — Custom JPQL/Native



```java

public interface ProductRepository extends JpaRepository<Product, Long> {



&#x20;   // JPQL — dùng tên class và field Java (không phải tên table/column)

&#x20;   @Query("SELECT p FROM Product p WHERE p.price BETWEEN :min AND :max AND p.status = 'ACTIVE'")

&#x20;   List<Product> findByPriceRange(

&#x20;       @Param("min") BigDecimal min,

&#x20;       @Param("max") BigDecimal max

&#x20;   );



&#x20;   // Native SQL — dùng tên table/column thật

&#x20;   @Query(

&#x20;       value = "SELECT \* FROM products p WHERE p.category\_id = :catId AND p.stock > 0",

&#x20;       nativeQuery = true

&#x20;   )

&#x20;   List<Product> findInStockByCategory(@Param("catId") Long categoryId);



&#x20;   // Modifying query — UPDATE/DELETE

&#x20;   @Modifying

&#x20;   @Transactional

&#x20;   @Query("UPDATE Product p SET p.status = :status WHERE p.id IN :ids")

&#x20;   int updateStatusBatch(@Param("status") ProductStatus status, @Param("ids") List<Long> ids);



&#x20;   // Projection — chỉ lấy một số field (performance)

&#x20;   @Query("SELECT new com.myapp.dto.ProductSummary(p.id, p.name, p.price) FROM Product p")

&#x20;   List<ProductSummary> findAllSummaries();

}

```



\### Specification — Dynamic Query



```java

// Dùng khi filter điều kiện động (nhiều filter tùy chọn)

public interface ProductRepository extends JpaRepository<Product, Long>,

&#x20;   JpaSpecificationExecutor<Product> { }



public class ProductSpecifications {

&#x20;   public static Specification<Product> hasCategory(Long categoryId) {

&#x20;       return (root, query, cb) ->

&#x20;           categoryId == null ? null : cb.equal(root.get("category").get("id"), categoryId);

&#x20;   }



&#x20;   public static Specification<Product> priceBetween(BigDecimal min, BigDecimal max) {

&#x20;       return (root, query, cb) -> {

&#x20;           if (min == null \&\& max == null) return null;

&#x20;           if (min == null) return cb.lessThanOrEqualTo(root.get("price"), max);

&#x20;           if (max == null) return cb.greaterThanOrEqualTo(root.get("price"), min);

&#x20;           return cb.between(root.get("price"), min, max);

&#x20;       };

&#x20;   }

}



// Kết hợp

Specification<Product> spec = Specification

&#x20;   .where(ProductSpecifications.hasCategory(categoryId))

&#x20;   .and(ProductSpecifications.priceBetween(minPrice, maxPrice));



List<Product> products = productRepo.findAll(spec);

```



\---



\## 12. @Transactional



\### Transaction là gì?



> Đảm bảo một nhóm operations \*\*thành công tất cả hoặc rollback tất cả\*\* (ACID).



\### Cách dùng



```java

@Service

public class TransferService {



&#x20;   @Transactional // Spring tạo transaction, commit sau method, rollback nếu exception

&#x20;   public void transfer(Long fromId, Long toId, BigDecimal amount) {

&#x20;       Account from = accountRepo.findById(fromId).orElseThrow();

&#x20;       Account to = accountRepo.findById(toId).orElseThrow();



&#x20;       if (from.getBalance().compareTo(amount) < 0) {

&#x20;           throw new InsufficientFundsException(); // Rollback

&#x20;       }



&#x20;       from.setBalance(from.getBalance().subtract(amount));

&#x20;       to.setBalance(to.getBalance().add(amount));



&#x20;       accountRepo.save(from); // Nếu lỗi ở đây

&#x20;       accountRepo.save(to);   // Cả 2 đều rollback

&#x20;   }

}

```



\### Transaction Propagation



```java

// REQUIRED (default): Join existing transaction, hoặc tạo mới

@Transactional(propagation = Propagation.REQUIRED)

public void methodA() {

&#x20;   methodB(); // methodB dùng transaction của methodA

}



// REQUIRES\_NEW: Luôn tạo transaction mới, suspend cái cũ

@Transactional(propagation = Propagation.REQUIRES\_NEW)

public void sendNotification() {

&#x20;   // Chạy trong transaction riêng

&#x20;   // Dù transaction bên ngoài rollback, cái này vẫn commit

}



// NESTED: Transaction lồng nhau (savepoint)

@Transactional(propagation = Propagation.NESTED)

public void nestedOperation() { ... }



// NOT\_SUPPORTED: Suspend transaction, chạy không có transaction

// NEVER: Ném exception nếu có active transaction

// SUPPORTS: Dùng transaction nếu có, không có cũng được

// MANDATORY: Phải có transaction, không có thì exception

```



\### Isolation Levels



```java

@Transactional(isolation = Isolation.READ\_COMMITTED) // Default PostgreSQL/SQL Server



// READ\_UNCOMMITTED — Đọc được dirty data (nhanh nhất, không an toàn)

// READ\_COMMITTED   — Chỉ đọc data đã commit (tránh dirty read)

// REPEATABLE\_READ  — Cùng query trong 1 tx luôn ra kết quả giống nhau (tránh non-repeatable read)

// SERIALIZABLE     — Transaction chạy tuần tự (chậm nhất, an toàn nhất)

```



\### Những lỗi thường gặp với @Transactional



```java

// LỖI 1: Self-invocation — gọi @Transactional method trong cùng class

@Service

public class OrderService {

&#x20;   public void processOrder(Order order) {

&#x20;       saveOrder(order); // Gọi internal → KHÔNG có transaction!

&#x20;       // Vì Spring proxy intercept từ bên ngoài, không intercept internal call

&#x20;   }



&#x20;   @Transactional

&#x20;   public void saveOrder(Order order) { ... } // Transaction không apply

}



// FIX: Inject self (ugly) hoặc tách ra class khác

@Service

public class OrderService {

&#x20;   @Autowired

&#x20;   private OrderService self; // Inject proxy



&#x20;   public void processOrder(Order order) {

&#x20;       self.saveOrder(order); // Gọi qua proxy → transaction apply

&#x20;   }

}



// LỖI 2: Chỉ rollback với RuntimeException mặc định

@Transactional // Chỉ rollback với unchecked exception!

public void method() throws IOException {

&#x20;   // IOException là checked → KHÔNG rollback

}



// FIX:

@Transactional(rollbackFor = Exception.class) // Rollback với mọi exception

public void method() throws IOException { ... }



// LỖI 3: @Transactional trên private method

@Transactional // KHÔNG có effect với private method

private void privateMethod() { ... }

// Spring proxy không thể intercept private method

```



\---



\## 13. Spring Security



\### Security Filter Chain



```

HTTP Request

&#x20;    ↓

SecurityFilterChain (danh sách filters)

├── UsernamePasswordAuthenticationFilter — xử lý form login

├── JwtAuthenticationFilter (custom) — xử lý JWT

├── BasicAuthenticationFilter — xử lý Basic auth

├── ExceptionTranslationFilter — convert SecurityException → HTTP response

└── FilterSecurityInterceptor — check authorization

&#x20;    ↓

DispatcherServlet → Controller

```



\### Cấu hình Security (Spring Security 6.x)



```java

@Configuration

@EnableWebSecurity

@EnableMethodSecurity // Bật @PreAuthorize, @PostAuthorize

public class SecurityConfig {



&#x20;   private final JwtAuthenticationFilter jwtFilter;

&#x20;   private final UserDetailsService userDetailsService;



&#x20;   @Bean

&#x20;   public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

&#x20;       return http

&#x20;           // Tắt CSRF (REST API dùng token, không dùng cookie session)

&#x20;           .csrf(csrf -> csrf.disable())

&#x20;           // Stateless session (JWT không cần session)

&#x20;           .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))

&#x20;           // Authorization rules

&#x20;           .authorizeHttpRequests(auth -> auth

&#x20;               .requestMatchers("/api/auth/\*\*").permitAll()      // Public

&#x20;               .requestMatchers("/api/admin/\*\*").hasRole("ADMIN") // Admin only

&#x20;               .requestMatchers(HttpMethod.GET, "/api/products/\*\*").permitAll() // GET public

&#x20;               .anyRequest().authenticated()                      // Còn lại cần login

&#x20;           )

&#x20;           // Thêm JWT filter trước UsernamePasswordAuthenticationFilter

&#x20;           .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)

&#x20;           .build();

&#x20;   }



&#x20;   @Bean

&#x20;   public AuthenticationManager authenticationManager(AuthenticationConfiguration config)

&#x20;       throws Exception {

&#x20;       return config.getAuthenticationManager();

&#x20;   }



&#x20;   @Bean

&#x20;   public PasswordEncoder passwordEncoder() {

&#x20;       return new BCryptPasswordEncoder(12); // strength 12 (default 10)

&#x20;   }

}

```



\### UserDetailsService



```java

@Service

public class CustomUserDetailsService implements UserDetailsService {



&#x20;   private final UserRepository userRepo;



&#x20;   @Override

&#x20;   public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

&#x20;       User user = userRepo.findByUsername(username)

&#x20;           .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));



&#x20;       return org.springframework.security.core.userdetails.User.builder()

&#x20;           .username(user.getUsername())

&#x20;           .password(user.getPasswordHash())

&#x20;           .roles(user.getRoles().stream()

&#x20;               .map(Role::getName)

&#x20;               .toArray(String\[]::new))

&#x20;           .accountExpired(!user.isActive())

&#x20;           .build();

&#x20;   }

}

```



\### Method-level Security



```java

@RestController

public class UserController {



&#x20;   @PreAuthorize("hasRole('ADMIN')")

&#x20;   @DeleteMapping("/users/{id}")

&#x20;   public void deleteUser(@PathVariable Long id) { ... }



&#x20;   @PreAuthorize("hasRole('ADMIN') or #id == authentication.principal.id")

&#x20;   @GetMapping("/users/{id}")

&#x20;   public UserResponse getUser(@PathVariable Long id) { ... }



&#x20;   @PostAuthorize("returnObject.userId == authentication.principal.id")

&#x20;   @GetMapping("/orders/{id}")

&#x20;   public OrderResponse getOrder(@PathVariable Long id) { ... }

}

```



\---



\## 14. JWT



\### JWT Structure



```

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9  ← Header (base64)

.

eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4iLCJpYXQiOjE1MTYyMzkwMjJ9  ← Payload (base64)

.

SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV\_adQssw5c  ← Signature (HMAC/RSA)

```



\*\*Payload chứa claims:\*\*

\- `sub` — subject (user id)

\- `iat` — issued at

\- `exp` — expiration time

\- Custom: roles, email...



\### JWT Implementation



```java

// pom.xml: io.jsonwebtoken:jjwt-api, jjwt-impl, jjwt-jackson



@Component

public class JwtUtil {

&#x20;   @Value("${jwt.secret}")

&#x20;   private String secret;



&#x20;   @Value("${jwt.expiration:86400000}") // 24h in ms

&#x20;   private long expiration;



&#x20;   private SecretKey getSigningKey() {

&#x20;       return Keys.hmacShaKeyFor(Decoders.BASE64.decode(secret));

&#x20;   }



&#x20;   public String generateToken(UserDetails userDetails) {

&#x20;       Map<String, Object> claims = new HashMap<>();

&#x20;       claims.put("roles", userDetails.getAuthorities().stream()

&#x20;           .map(GrantedAuthority::getAuthority)

&#x20;           .collect(Collectors.toList()));



&#x20;       return Jwts.builder()

&#x20;           .claims(claims)

&#x20;           .subject(userDetails.getUsername())

&#x20;           .issuedAt(new Date())

&#x20;           .expiration(new Date(System.currentTimeMillis() + expiration))

&#x20;           .signWith(getSigningKey())

&#x20;           .compact();

&#x20;   }



&#x20;   public String extractUsername(String token) {

&#x20;       return extractClaims(token).getSubject();

&#x20;   }



&#x20;   public boolean isTokenValid(String token, UserDetails userDetails) {

&#x20;       String username = extractUsername(token);

&#x20;       return username.equals(userDetails.getUsername()) \&\& !isTokenExpired(token);

&#x20;   }



&#x20;   private Claims extractClaims(String token) {

&#x20;       return Jwts.parser()

&#x20;           .verifyWith(getSigningKey())

&#x20;           .build()

&#x20;           .parseSignedClaims(token)

&#x20;           .getPayload();

&#x20;   }



&#x20;   private boolean isTokenExpired(String token) {

&#x20;       return extractClaims(token).getExpiration().before(new Date());

&#x20;   }

}

```



\### JWT Filter



```java

@Component

@RequiredArgsConstructor

public class JwtAuthenticationFilter extends OncePerRequestFilter {



&#x20;   private final JwtUtil jwtUtil;

&#x20;   private final UserDetailsService userDetailsService;



&#x20;   @Override

&#x20;   protected void doFilterInternal(HttpServletRequest request,

&#x20;                                   HttpServletResponse response,

&#x20;                                   FilterChain filterChain)

&#x20;       throws ServletException, IOException {



&#x20;       String authHeader = request.getHeader("Authorization");



&#x20;       if (authHeader == null || !authHeader.startsWith("Bearer ")) {

&#x20;           filterChain.doFilter(request, response);

&#x20;           return;

&#x20;       }



&#x20;       String token = authHeader.substring(7); // Remove "Bearer "



&#x20;       try {

&#x20;           String username = jwtUtil.extractUsername(token);



&#x20;           // Chưa có authentication trong context

&#x20;           if (username != null \&\& SecurityContextHolder.getContext().getAuthentication() == null) {

&#x20;               UserDetails userDetails = userDetailsService.loadUserByUsername(username);



&#x20;               if (jwtUtil.isTokenValid(token, userDetails)) {

&#x20;                   UsernamePasswordAuthenticationToken auth =

&#x20;                       new UsernamePasswordAuthenticationToken(

&#x20;                           userDetails, null, userDetails.getAuthorities());

&#x20;                   auth.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));

&#x20;                   SecurityContextHolder.getContext().setAuthentication(auth);

&#x20;               }

&#x20;           }

&#x20;       } catch (JwtException e) {

&#x20;           // Token invalid — không set authentication, request tiếp tục nhưng sẽ bị chặn

&#x20;       }



&#x20;       filterChain.doFilter(request, response);

&#x20;   }

}

```



\### Auth Controller



```java

@RestController

@RequestMapping("/api/auth")

public class AuthController {



&#x20;   @PostMapping("/login")

&#x20;   public TokenResponse login(@Valid @RequestBody LoginRequest request) {

&#x20;       // AuthenticationManager sẽ gọi UserDetailsService và PasswordEncoder

&#x20;       Authentication auth = authManager.authenticate(

&#x20;           new UsernamePasswordAuthenticationToken(request.username(), request.password())

&#x20;       );

&#x20;       UserDetails user = (UserDetails) auth.getPrincipal();

&#x20;       String token = jwtUtil.generateToken(user);

&#x20;       return new TokenResponse(token, "Bearer", jwtUtil.getExpiration());

&#x20;   }



&#x20;   @PostMapping("/register")

&#x20;   @ResponseStatus(HttpStatus.CREATED)

&#x20;   public UserResponse register(@Valid @RequestBody RegisterRequest request) {

&#x20;       return userService.register(request);

&#x20;   }

}

```



\---



\## 15. Spring AOP



\### Khái niệm



> AOP (Aspect-Oriented Programming) — tách \*\*cross-cutting concerns\*\* (logging, security, transaction, metrics) ra khỏi business logic.



```

Vocabulary:

\- Aspect: Module chứa cross-cutting logic (Logging, Audit...)

\- Advice: Code thực thi (Before, After, Around...)

\- Pointcut: Biểu thức xác định method nào bị intercept

\- JoinPoint: Điểm cụ thể khi Advice được gọi (method execution)

\- Weaving: Quá trình kết hợp Aspect vào code

```



\### Ví dụ thực tế



```java

@Aspect

@Component

public class LoggingAspect {



&#x20;   private static final Logger log = LoggerFactory.getLogger(LoggingAspect.class);



&#x20;   // Pointcut expression: tất cả method trong package service

&#x20;   @Pointcut("execution(\* com.myapp.service.\*.\*(..))")

&#x20;   public void serviceLayer() {}



&#x20;   // Before: Chạy trước method

&#x20;   @Before("serviceLayer()")

&#x20;   public void logBefore(JoinPoint jp) {

&#x20;       log.info("Calling: {}.{}({})",

&#x20;           jp.getTarget().getClass().getSimpleName(),

&#x20;           jp.getSignature().getName(),

&#x20;           Arrays.toString(jp.getArgs()));

&#x20;   }



&#x20;   // AfterReturning: Chạy sau method thành công

&#x20;   @AfterReturning(pointcut = "serviceLayer()", returning = "result")

&#x20;   public void logAfterReturning(JoinPoint jp, Object result) {

&#x20;       log.info("Method {} returned: {}", jp.getSignature().getName(), result);

&#x20;   }



&#x20;   // AfterThrowing: Chạy khi có exception

&#x20;   @AfterThrowing(pointcut = "serviceLayer()", throwing = "ex")

&#x20;   public void logException(JoinPoint jp, Exception ex) {

&#x20;       log.error("Exception in {}: {}", jp.getSignature().getName(), ex.getMessage());

&#x20;   }



&#x20;   // Around: Bao quanh method — mạnh nhất

&#x20;   @Around("execution(\* com.myapp.service.\*.\*(..)) \&\& @annotation(com.myapp.annotation.Timed)")

&#x20;   public Object measureTime(ProceedingJoinPoint pjp) throws Throwable {

&#x20;       long start = System.currentTimeMillis();

&#x20;       try {

&#x20;           Object result = pjp.proceed(); // Gọi method gốc

&#x20;           long duration = System.currentTimeMillis() - start;

&#x20;           log.info("{} took {}ms", pjp.getSignature().getName(), duration);

&#x20;           return result;

&#x20;       } catch (Exception e) {

&#x20;           log.error("{} failed after {}ms", pjp.getSignature().getName(),

&#x20;               System.currentTimeMillis() - start);

&#x20;           throw e;

&#x20;       }

&#x20;   }

}



// Custom annotation để đánh dấu method cần đo thời gian

@Target(ElementType.METHOD)

@Retention(RetentionPolicy.RUNTIME)

public @interface Timed {}



// Dùng

@Service

public class ProductService {

&#x20;   @Timed

&#x20;   public List<Product> findAll() { ... }

}

```



\---



\## 16. Caching



\### Spring Cache Abstraction



```java

// Bật caching

@SpringBootApplication

@EnableCaching

public class MyApp { ... }



// application.yml

spring:

&#x20; cache:

&#x20;   type: redis  # simple (in-memory), redis, caffeine, ehcache...

&#x20; redis:

&#x20;   host: localhost

&#x20;   port: 6379

```



```java

@Service

public class ProductService {



&#x20;   // Cache kết quả lần đầu, lần sau trả từ cache

&#x20;   @Cacheable(value = "products", key = "#id")

&#x20;   public Product findById(Long id) {

&#x20;       return productRepo.findById(id).orElseThrow(); // Chỉ chạy nếu cache miss

&#x20;   }



&#x20;   // Cache với điều kiện

&#x20;   @Cacheable(value = "products", key = "#name", condition = "#name.length() > 2")

&#x20;   public List<Product> findByName(String name) { ... }



&#x20;   // Cập nhật cache khi update

&#x20;   @CachePut(value = "products", key = "#product.id")

&#x20;   public Product update(Product product) {

&#x20;       return productRepo.save(product);

&#x20;   }



&#x20;   // Xóa cache khi delete

&#x20;   @CacheEvict(value = "products", key = "#id")

&#x20;   public void delete(Long id) {

&#x20;       productRepo.deleteById(id);

&#x20;   }



&#x20;   // Xóa toàn bộ cache

&#x20;   @CacheEvict(value = "products", allEntries = true)

&#x20;   public void clearCache() {}

}

```



\### Redis với Spring Boot



```java

// pom.xml: spring-boot-starter-data-redis



@Configuration

public class RedisConfig {

&#x20;   @Bean

&#x20;   public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {

&#x20;       RedisTemplate<String, Object> template = new RedisTemplate<>();

&#x20;       template.setConnectionFactory(factory);

&#x20;       template.setKeySerializer(new StringRedisSerializer());

&#x20;       template.setValueSerializer(new GenericJackson2JsonRedisSerializer());

&#x20;       return template;

&#x20;   }

}



// Dùng trực tiếp (không qua @Cacheable)

@Service

public class SessionService {

&#x20;   private final RedisTemplate<String, Object> redisTemplate;



&#x20;   public void saveSession(String sessionId, UserSession session, Duration ttl) {

&#x20;       redisTemplate.opsForValue().set("session:" + sessionId, session, ttl);

&#x20;   }



&#x20;   public UserSession getSession(String sessionId) {

&#x20;       return (UserSession) redisTemplate.opsForValue().get("session:" + sessionId);

&#x20;   }



&#x20;   public void invalidateSession(String sessionId) {

&#x20;       redisTemplate.delete("session:" + sessionId);

&#x20;   }

}

```



\---



\## 17. Spring Events



\### Application Events — Loose coupling giữa các component



```java

// 1. Định nghĩa Event

public record UserRegisteredEvent(

&#x20;   Long userId,

&#x20;   String email,

&#x20;   LocalDateTime registeredAt

) {}



// 2. Publish event

@Service

public class UserService {

&#x20;   private final ApplicationEventPublisher eventPublisher;



&#x20;   public User register(RegisterRequest req) {

&#x20;       User user = createUser(req);

&#x20;       userRepo.save(user);



&#x20;       // Publish event — không cần biết ai xử lý

&#x20;       eventPublisher.publishEvent(new UserRegisteredEvent(

&#x20;           user.getId(), user.getEmail(), LocalDateTime.now()

&#x20;       ));



&#x20;       return user;

&#x20;   }

}



// 3. Listen event — có thể có nhiều listener

@Component

public class EmailEventListener {

&#x20;   @EventListener

&#x20;   public void onUserRegistered(UserRegisteredEvent event) {

&#x20;       emailService.sendWelcomeEmail(event.email());

&#x20;   }

}



@Component

public class AuditEventListener {

&#x20;   @EventListener

&#x20;   public void onUserRegistered(UserRegisteredEvent event) {

&#x20;       auditLog.record("USER\_REGISTERED", event.userId());

&#x20;   }

}



// Async event listener — không block main thread

@Component

public class NotificationListener {

&#x20;   @Async

&#x20;   @EventListener

&#x20;   public void onUserRegistered(UserRegisteredEvent event) {

&#x20;       notificationService.sendPush(event.userId(), "Welcome!");

&#x20;   }

}

```



\### @TransactionalEventListener



```java

// Chỉ xử lý event sau khi transaction commit thành công

@TransactionalEventListener(phase = TransactionPhase.AFTER\_COMMIT)

public void onOrderPlaced(OrderPlacedEvent event) {

&#x20;   // Gửi email xác nhận sau khi order đã được lưu vào DB

&#x20;   emailService.sendOrderConfirmation(event.orderId());

}

```



\---



\## 18. Profiles



\### Cấu hình multi-environment



```java

// Khai báo bean chỉ active ở profile cụ thể

@Profile("dev")

@Component

public class MockPaymentGateway implements PaymentGateway {

&#x20;   public PaymentResult charge(BigDecimal amount) {

&#x20;       return PaymentResult.success("mock-txn-id"); // Giả lập khi dev

&#x20;   }

}



@Profile("prod")

@Component

public class StripePaymentGateway implements PaymentGateway {

&#x20;   public PaymentResult charge(BigDecimal amount) {

&#x20;       return stripe.charge(amount); // Real payment

&#x20;   }

}



// Config class theo profile

@Configuration

@Profile("dev")

public class DevConfig {

&#x20;   @Bean

&#x20;   public DataSource dataSource() {

&#x20;       return new EmbeddedDatabaseBuilder()

&#x20;           .setType(EmbeddedDatabaseType.H2)

&#x20;           .build();

&#x20;   }

}

```



\### Kích hoạt profile



```bash

\# Command line

java -jar app.jar --spring.profiles.active=prod



\# Environment variable

SPRING\_PROFILES\_ACTIVE=prod java -jar app.jar



\# application.yml

spring:

&#x20; profiles:

&#x20;   active: dev

```



\### @ConditionalOnProperty



```java

// Bean chỉ tạo khi property có giá trị cụ thể

@Bean

@ConditionalOnProperty(name = "feature.new-checkout.enabled", havingValue = "true")

public NewCheckoutService newCheckoutService() {

&#x20;   return new NewCheckoutService();

}

```



\---



\## 19. Testing



\### Unit Test với Mockito



```java

@ExtendWith(MockitoExtension.class)

class UserServiceTest {



&#x20;   @Mock

&#x20;   private UserRepository userRepo;



&#x20;   @Mock

&#x20;   private EmailService emailService;



&#x20;   @InjectMocks

&#x20;   private UserService userService;



&#x20;   @Test

&#x20;   void findById\_whenUserExists\_returnsUser() {

&#x20;       // Arrange

&#x20;       User user = new User(1L, "john", "john@example.com");

&#x20;       when(userRepo.findById(1L)).thenReturn(Optional.of(user));



&#x20;       // Act

&#x20;       UserResponse result = userService.findById(1L);



&#x20;       // Assert

&#x20;       assertThat(result.id()).isEqualTo(1L);

&#x20;       assertThat(result.username()).isEqualTo("john");

&#x20;       verify(userRepo, times(1)).findById(1L);

&#x20;   }



&#x20;   @Test

&#x20;   void findById\_whenNotFound\_throwsException() {

&#x20;       when(userRepo.findById(99L)).thenReturn(Optional.empty());



&#x20;       assertThatThrownBy(() -> userService.findById(99L))

&#x20;           .isInstanceOf(ResourceNotFoundException.class)

&#x20;           .hasMessageContaining("User not found with id: 99");

&#x20;   }



&#x20;   @Test

&#x20;   void register\_shouldHashPasswordAndSendEmail() {

&#x20;       // Arrange

&#x20;       CreateUserRequest req = new CreateUserRequest("john", "john@email.com", "password123");

&#x20;       when(userRepo.existsByEmail(any())).thenReturn(false);

&#x20;       when(userRepo.save(any())).thenAnswer(invocation -> {

&#x20;           User u = invocation.getArgument(0);

&#x20;           u.setId(1L);

&#x20;           return u;

&#x20;       });



&#x20;       // Act

&#x20;       userService.register(req);



&#x20;       // Assert

&#x20;       verify(emailService).sendWelcomeEmail("john@email.com");

&#x20;       verify(userRepo).save(argThat(u ->

&#x20;           !u.getPasswordHash().equals("password123") // Password đã được hash

&#x20;       ));

&#x20;   }

}

```



\### Integration Test với @SpringBootTest



```java

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM\_PORT)

@ActiveProfiles("test")

class UserControllerIntegrationTest {



&#x20;   @Autowired

&#x20;   private TestRestTemplate restTemplate;



&#x20;   @Autowired

&#x20;   private UserRepository userRepo;



&#x20;   @BeforeEach

&#x20;   void setup() {

&#x20;       userRepo.deleteAll();

&#x20;   }



&#x20;   @Test

&#x20;   void createUser\_returnsCreatedUser() {

&#x20;       CreateUserRequest req = new CreateUserRequest("john", "john@email.com", "Pass123!");



&#x20;       ResponseEntity<UserResponse> response = restTemplate.postForEntity(

&#x20;           "/api/v1/users", req, UserResponse.class

&#x20;       );



&#x20;       assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);

&#x20;       assertThat(response.getBody().username()).isEqualTo("john");

&#x20;       assertThat(userRepo.count()).isEqualTo(1);

&#x20;   }

}

```



\### MockMvc — Test Controller Layer



```java

@WebMvcTest(UserController.class) // Chỉ load web layer, không load service/repo

class UserControllerTest {



&#x20;   @Autowired

&#x20;   private MockMvc mockMvc;



&#x20;   @MockBean // Mock bean trong Spring context

&#x20;   private UserService userService;



&#x20;   @Autowired

&#x20;   private ObjectMapper objectMapper;



&#x20;   @Test

&#x20;   void getUser\_returnsOk() throws Exception {

&#x20;       UserResponse user = new UserResponse(1L, "john", "john@email.com", LocalDateTime.now());

&#x20;       when(userService.findById(1L)).thenReturn(user);



&#x20;       mockMvc.perform(get("/api/v1/users/1")

&#x20;               .contentType(MediaType.APPLICATION\_JSON))

&#x20;           .andExpect(status().isOk())

&#x20;           .andExpect(jsonPath("$.id").value(1))

&#x20;           .andExpect(jsonPath("$.username").value("john"));

&#x20;   }



&#x20;   @Test

&#x20;   void createUser\_withInvalidEmail\_returnsBadRequest() throws Exception {

&#x20;       CreateUserRequest req = new CreateUserRequest("john", "not-an-email", "Pass123!");



&#x20;       mockMvc.perform(post("/api/v1/users")

&#x20;               .contentType(MediaType.APPLICATION\_JSON)

&#x20;               .content(objectMapper.writeValueAsString(req)))

&#x20;           .andExpect(status().isBadRequest())

&#x20;           .andExpect(jsonPath("$.fieldErrors.email").exists());

&#x20;   }

}

```



\### @DataJpaTest — Test Repository Layer



```java

@DataJpaTest // Chỉ load JPA layer, dùng H2 in-memory

@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE) // Giữ real DB

@ActiveProfiles("test")

class UserRepositoryTest {



&#x20;   @Autowired

&#x20;   private UserRepository userRepo;



&#x20;   @Autowired

&#x20;   private TestEntityManager entityManager;



&#x20;   @Test

&#x20;   void findByEmail\_whenExists\_returnsUser() {

&#x20;       User user = new User(null, "john", "john@email.com", "hash");

&#x20;       entityManager.persist(user);

&#x20;       entityManager.flush();



&#x20;       Optional<User> found = userRepo.findByEmail("john@email.com");



&#x20;       assertThat(found).isPresent();

&#x20;       assertThat(found.get().getUsername()).isEqualTo("john");

&#x20;   }

}

```



\---



\## 20. Actuator



\### Spring Boot Actuator — Monitoring



```xml

<!-- pom.xml -->

<dependency>

&#x20;   <groupId>org.springframework.boot</groupId>

&#x20;   <artifactId>spring-boot-starter-actuator</artifactId>

</dependency>

```



```yaml

\# application.yml

management:

&#x20; endpoints:

&#x20;   web:

&#x20;     exposure:

&#x20;       include: health,info,metrics,env,loggers,threaddump,heapdump

&#x20; endpoint:

&#x20;   health:

&#x20;     show-details: when-authorized # always, never, when-authorized

&#x20; info:

&#x20;   env:

&#x20;     enabled: true



info:

&#x20; app:

&#x20;   name: My Application

&#x20;   version: 1.0.0

&#x20;   description: Backend service

```



\### Endpoints quan trọng



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



\### Custom Health Indicator



```java

@Component

public class DatabaseHealthIndicator implements HealthIndicator {



&#x20;   private final DataSource dataSource;



&#x20;   @Override

&#x20;   public Health health() {

&#x20;       try (Connection conn = dataSource.getConnection()) {

&#x20;           boolean valid = conn.isValid(2);

&#x20;           if (valid) {

&#x20;               return Health.up()

&#x20;                   .withDetail("database", "PostgreSQL")

&#x20;                   .withDetail("status", "reachable")

&#x20;                   .build();

&#x20;           }

&#x20;           return Health.down().withDetail("reason", "Connection invalid").build();

&#x20;       } catch (SQLException e) {

&#x20;           return Health.down(e).withDetail("reason", e.getMessage()).build();

&#x20;       }

&#x20;   }

}

```



\---



\## 21. Microservices \& Spring Cloud



\### Các thành phần cơ bản



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

&#x20;   └── Trace request qua nhiều service

```



\### Service Discovery với Eureka



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

\# product-service application.yml

spring:

&#x20; application:

&#x20;   name: product-service  # Tên đăng ký trên Eureka



eureka:

&#x20; client:

&#x20;   service-url:

&#x20;     defaultZone: http://localhost:8761/eureka

```



\### RestTemplate / WebClient — Gọi giữa services



```java

// Dùng service name thay vì URL cứng

@Service

public class OrderService {

&#x20;   private final RestTemplate restTemplate;



&#x20;   // @LoadBalanced — tự tìm URL từ Eureka và load balance

&#x20;   public ProductResponse getProduct(Long productId) {

&#x20;       return restTemplate.getForObject(

&#x20;           "http://product-service/api/products/" + productId, // "product-service" là tên service

&#x20;           ProductResponse.class

&#x20;       );

&#x20;   }

}



// Config RestTemplate với @LoadBalanced

@Bean

@LoadBalanced

public RestTemplate restTemplate() {

&#x20;   return new RestTemplate();

}

```



\### Circuit Breaker với Resilience4j



```java

// Tránh cascade failure: service A gọi B → B down → A bị treo → C gọi A bị treo...

// Circuit Breaker: Sau N lần lỗi → "mở mạch" → ngay lập tức trả fallback



@Service

public class ProductService {



&#x20;   @CircuitBreaker(name = "productService", fallbackMethod = "getProductFallback")

&#x20;   @Retry(name = "productService") // Retry trước khi circuit breaker mở

&#x20;   @TimeLimiter(name = "productService") // Timeout

&#x20;   public ProductResponse getProduct(Long id) {

&#x20;       return restTemplate.getForObject("http://product-service/api/products/" + id,

&#x20;           ProductResponse.class);

&#x20;   }



&#x20;   // Fallback method — phải có cùng signature + Throwable

&#x20;   public ProductResponse getProductFallback(Long id, Throwable ex) {

&#x20;       log.warn("Product service down, returning cached data for {}", id);

&#x20;       return ProductResponse.placeholder(id); // Trả data placeholder

&#x20;   }

}

```



```yaml

\# application.yml

resilience4j:

&#x20; circuitbreaker:

&#x20;   instances:

&#x20;     productService:

&#x20;       sliding-window-size: 10       # Xét 10 request gần nhất

&#x20;       failure-rate-threshold: 50    # Mở circuit khi 50% fail

&#x20;       wait-duration-in-open-state: 30s # Thử lại sau 30s

&#x20;       permitted-number-of-calls-in-half-open-state: 3

&#x20; retry:

&#x20;   instances:

&#x20;     productService:

&#x20;       max-attempts: 3

&#x20;       wait-duration: 1s

```



\---



\## 22. Câu hỏi hay gặp



| Câu hỏi | Trả lời ngắn gọn |

|---------|-----------------|

| Spring Boot auto-config hoạt động thế nào? | `@EnableAutoConfiguration` scan `AutoConfiguration.imports`, dùng `@ConditionalOn\*` để quyết định có tạo bean không |

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



\---



\*README này cover \~90% câu hỏi Spring Boot trong interview — chúc bạn ace! 🚀\*

