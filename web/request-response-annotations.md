# Spring Path & Parameter Annotations - Complete Guide

A comprehensive guide to Spring annotations used for handling request parameters, path variables, headers, and request data.

---

## 📑 Table of Contents

1. [@PathVariable](#1-pathvariable)
2. [@RequestParam](#2-requestparam)
3. [@RequestBody](#3-requestbody)
4. [@RequestHeader](#4-requestheader)
5. [@CookieValue](#5-cookievalue)
6. [@ModelAttribute](#6-modelattribute)
7. [@Valid](#7-valid)

---

## 1. @PathVariable

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Extracts values from URI path template variables. |
| **Use Level** | Method Parameter |
| **Common Use Case** | RESTful URLs, resource identification |

### Definition

`@PathVariable` extracts values from the URI path and binds them to method parameters. It's a key component of RESTful API design, allowing you to embed resource identifiers directly in the URL path. Path variables make URLs more readable and SEO-friendly compared to query parameters.

**How it works:**
When a request comes in with a URL like `/users/123`, Spring matches the URL pattern defined in the mapping annotation (e.g., `@GetMapping("/users/{id}")`). It extracts the value `123` from the `{id}` placeholder and binds it to the method parameter annotated with `@PathVariable`. The value is automatically converted to the parameter's type.

### Example

```java
@RestController
@RequestMapping("/api")
public class ResourceController {
    
    // Simple path variable
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    // Multiple path variables
    @GetMapping("/users/{userId}/orders/{orderId}")
    public Order getOrder(
        @PathVariable Long userId,
        @PathVariable Long orderId
    ) {
        return orderService.findByUserAndOrder(userId, orderId);
    }
    
    // Custom name
    @GetMapping("/products/{productId}")
    public Product getProduct(@PathVariable("productId") Long id) {
        return productService.findById(id);
    }
    
    // Optional path variable
    @GetMapping({"/products", "/products/{id}"})
    public ResponseEntity<?> getProducts(@PathVariable(required = false) Long id) {
        if (id == null) {
            return ResponseEntity.ok(productService.findAll());
        }
        return ResponseEntity.ok(productService.findById(id));
    }
}
```

---

### Interview Questions

**Q1: What's the difference between @PathVariable and @RequestParam?**

**Answer:**
- `@PathVariable`: Part of URL path (`/users/123`)
- `@RequestParam`: Query string (`/users?id=123`)
- Path variables are more RESTful for resource identification
- Request params are better for filtering, sorting, and optional parameters

```java
// @PathVariable - Resource identification
@GetMapping("/users/{id}")
public User getById(@PathVariable Long id) {
    return userService.findById(id);
}
// URL: /users/123

// @RequestParam - Filtering
@GetMapping("/users")
public List<User> search(@RequestParam String name) {
    return userService.findByName(name);
}
// URL: /users?name=John
```

**Q2: How do you handle optional path variables?**

**Answer:**
Create multiple mappings or use `required = false`:

```java
// Approach 1: Multiple mappings (RECOMMENDED)
@GetMapping("/users")
public List<User> getAllUsers() {
    return userService.findAll();
}

@GetMapping("/users/{id}")
public User getUserById(@PathVariable Long id) {
    return userService.findById(id);
}

// Approach 2: Optional path variable
@GetMapping({"/products", "/products/{id}"})
public ResponseEntity<?> getProducts(@PathVariable(required = false) Long id) {
    if (id == null) {
        return ResponseEntity.ok(productService.findAll());
    }
    return ResponseEntity.ok(productService.findById(id));
}
```

---

### Common Pitfalls

**PITFALL 1: Path variable name mismatch**

**Problem:**
```java
@GetMapping("/users/{userId}")
public User getUser(@PathVariable Long id) { // Mismatch!
    return userService.findById(id);
}
// Error: Required path variable 'userId' is not present
```

**Solution:**
```java
// Match names
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id);
}

// Or specify name explicitly
@GetMapping("/users/{userId}")
public User getUser(@PathVariable("userId") Long id) {
    return userService.findById(id);
}
```

**PITFALL 2: Not validating path variables**

**Problem:**
```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id); // What if id is negative?
}
```

**Solution:**
```java
// Use validation
@GetMapping("/users/{id}")
public User getUser(@PathVariable @Min(1) Long id) {
    return userService.findById(id);
}

// Or use regex in path
@GetMapping("/users/{id:[0-9]+}") // Only numbers
public User getUser(@PathVariable Long id) {
    return userService.findById(id);
}
```

---

## 2. @RequestParam

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Extracts query parameters from URL. |
| **Use Level** | Method Parameter |
| **Common Use Case** | Filtering, pagination, sorting, optional parameters |

### Definition

`@RequestParam` binds HTTP request parameters (query string parameters) to method parameters. It's ideal for optional parameters, filtering, sorting, and pagination. Unlike path variables, request parameters appear after the `?` in the URL and are typically optional or have default values.

**How it works:**
When a request arrives with query parameters like `/products?category=electronics&minPrice=100`, Spring extracts each parameter value and binds it to the corresponding `@RequestParam` annotated parameter. It performs automatic type conversion and can provide default values if parameters are missing.

### Example

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    // Simple request param
    @GetMapping("/search")
    public List<Product> search(@RequestParam String keyword) {
        return productService.search(keyword);
    }
    // URL: /products/search?keyword=laptop
    
    // Multiple params with defaults
    @GetMapping
    public Page<Product> getAll(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "name") String sortBy
    ) {
        return productService.findAll(PageRequest.of(page, size, Sort.by(sortBy)));
    }
    // URL: /products?page=0&size=20&sortBy=price
    
    // Optional parameter
    @GetMapping("/filter")
    public List<Product> filter(
        @RequestParam String category,
        @RequestParam(required = false) BigDecimal minPrice
    ) {
        return productService.filter(category, minPrice);
    }
    
    // Multiple values
    @GetMapping("/by-ids")
    public List<Product> getByIds(@RequestParam List<Long> ids) {
        return productService.findAllById(ids);
    }
    // URL: /products/by-ids?ids=1&ids=2&ids=3
}
```

---

### Interview Questions

**Q1: How do you handle multiple values for the same request parameter?**

**Answer:**
Use a collection type (List, Set):

```java
@GetMapping("/products")
public List<Product> filterByCategories(@RequestParam List<String> categories) {
    return productService.findByCategories(categories);
}
// URL: /products?categories=electronics&categories=books
// categories = ["electronics", "books"]
```

**Q2: How do you implement pagination with @RequestParam?**

**Answer:**
```java
@GetMapping("/products")
public Page<Product> getProducts(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size,
    @RequestParam(defaultValue = "id") String sortBy
) {
    Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
    return productService.findAll(pageable);
}
// URL: /products?page=0&size=20&sortBy=price
```

---

### Common Pitfalls

**PITFALL 1: Not providing default values for optional parameters**

**Problem:**
```java
@GetMapping("/products")
public Page<Product> getProducts(
    @RequestParam int page,  // Required!
    @RequestParam int size   // Required!
) {
    return productService.findAll(PageRequest.of(page, size));
}
// URL: /products -> ERROR: Required parameter 'page' is not present
```

**Solution:**
```java
@GetMapping("/products")
public Page<Product> getProducts(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size
) {
    return productService.findAll(PageRequest.of(page, size));
}
```

**PITFALL 2: Type mismatch errors**

**Problem:**
```java
@GetMapping("/products")
public List<Product> filter(@RequestParam BigDecimal minPrice) {
    return productService.filterByMinPrice(minPrice);
}
// URL: /products?minPrice=abc -> ERROR: Type mismatch
```

**Solution:** Handle globally with exception handler:
```java
@RestControllerAdvice
public class ExceptionHandler {
    @org.springframework.web.bind.annotation.ExceptionHandler(
        MethodArgumentTypeMismatchException.class
    )
    public ResponseEntity<ErrorResponse> handleTypeMismatch(
        MethodArgumentTypeMismatchException ex
    ) {
        ErrorResponse error = new ErrorResponse(
            "Invalid value for parameter: " + ex.getName()
        );
        return ResponseEntity.badRequest().body(error);
    }
}
```

---

## 3. @RequestBody

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Binds HTTP request body to Java object (deserializes JSON/XML). |
| **Use Level** | Method Parameter |
| **Common Use Case** | POST/PUT/PATCH requests with JSON/XML payload |

### Definition

`@RequestBody` binds the HTTP request body to a Java object. It's essential for REST APIs that accept JSON or XML data. Spring uses HttpMessageConverters (Jackson for JSON) to automatically deserialize the request body into the specified object type.

**How it works:**
When a POST/PUT/PATCH request arrives with a JSON body, Spring reads the entire request body, uses Jackson to convert the JSON to a Java object, and injects it into the method parameter. If `@Valid` is present, validation occurs after deserialization but before method execution.

### Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // Simple request body
    @PostMapping
    public ResponseEntity<User> create(@RequestBody User user) {
        User saved = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }
    
    // With validation
    @PostMapping("/register")
    public User register(@Valid @RequestBody UserDTO dto) {
        return userService.register(dto);
    }
    
    // List of objects
    @PostMapping("/batch")
    public List<User> createBatch(@Valid @RequestBody List<User> users) {
        return userService.saveAll(users);
    }
}

// DTO with validation
public class UserDTO {
    @NotBlank
    @Size(min = 3, max = 20)
    private String username;
    
    @Email
    private String email;
    
    @Size(min = 8)
    private String password;
}
```

---

### Interview Questions

**Q1: What's the difference between @RequestBody and @RequestParam?**

**Answer:**
- `@RequestBody`: Data from HTTP request body (JSON/XML)
- `@RequestParam`: Data from URL query string

```java
// @RequestBody - JSON in body
@PostMapping("/users")
public User create(@RequestBody User user) {
    return userService.save(user);
}
// Request: POST /users
// Body: {"name":"John","email":"john@example.com"}

// @RequestParam - Query string
@GetMapping("/users/search")
public List<User> search(@RequestParam String name) {
    return userService.search(name);
}
// Request: GET /users/search?name=John
```

**Q2: How does Spring deserialize @RequestBody?**

**Answer:**
Spring uses Jackson library through `HttpMessageConverter`:
1. Request arrives with JSON in body
2. Spring selects `MappingJackson2HttpMessageConverter`
3. Jackson deserializes JSON to Java object
4. If `@Valid` is present, validation runs
5. Method executes with populated object

---

### Common Pitfalls

**PITFALL 1: Forgetting @RequestBody annotation**

**Problem:**
```java
@PostMapping("/users")
public User create(User user) { // Missing @RequestBody
    return userService.save(user); // user is null!
}
```

**Solution:**
```java
@PostMapping("/users")
public User create(@RequestBody User user) {
    return userService.save(user);
}
```

**PITFALL 2: Not validating request body**

**Problem:**
```java
@PostMapping("/users")
public User create(@RequestBody User user) {
    return userService.save(user); // No validation!
}
```

**Solution:**
```java
@PostMapping("/users")
public User create(@Valid @RequestBody UserDTO dto) {
    return userService.create(dto);
}

public class UserDTO {
    @NotBlank @Size(min = 2, max = 50)
    private String name;
    
    @Email
    private String email;
}
```

---

## 4. @RequestHeader

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Extracts values from HTTP request headers. |
| **Use Level** | Method Parameter |
| **Common Use Case** | Authentication tokens, API keys, custom headers |

### Definition

`@RequestHeader` binds HTTP request header values to method parameters. It's commonly used for authentication (JWT tokens), API versioning, correlation IDs, and custom metadata. Headers provide a way to pass information separate from the URL or body.

**How it works:**
When a request arrives, Spring extracts the specified header value from the HTTP headers and binds it to the method parameter. If the header is missing and `required=true` (default), Spring throws an exception. You can make headers optional or provide default values.

### Example

```java
@RestController
@RequestMapping("/api")
public class ApiController {
    
    // Simple header
    @GetMapping("/secure")
    public String secureEndpoint(@RequestHeader("Authorization") String token) {
        // Validate token
        return "Authenticated!";
    }
    
    // Multiple headers with default
    @GetMapping("/info")
    public Map<String, String> getInfo(
        @RequestHeader("User-Agent") String userAgent,
        @RequestHeader(value = "X-API-Version", defaultValue = "1.0") String version
    ) {
        Map<String, String> info = new HashMap<>();
        info.put("userAgent", userAgent);
        info.put("version", version);
        return info;
    }
    
    // Optional header
    @GetMapping("/data")
    public Data getData(
        @RequestHeader(value = "X-Request-ID", required = false) String requestId
    ) {
        if (requestId != null) {
            log.info("Processing request: {}", requestId);
        }
        return dataService.get();
    }
}
```

---

### Interview Questions

**Q1: How do you extract JWT token from Authorization header?**

**Answer:**
```java
@GetMapping("/profile")
public UserProfile getProfile(@RequestHeader("Authorization") String authHeader) {
    // Remove "Bearer " prefix
    String token = authHeader.substring(7);
    
    // Validate and extract user
    String username = tokenProvider.getUsernameFromToken(token);
    return profileService.getByUsername(username);
}
```

**Q2: How do you handle missing required headers?**

**Answer:**
```java
// Make it optional
@GetMapping("/api")
public ResponseEntity<String> api(
    @RequestHeader(value = "X-API-Key", required = false) String apiKey
) {
    if (apiKey == null) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
            .body("API key required");
    }
    return ResponseEntity.ok("Success");
}

// Or handle globally
@RestControllerAdvice
public class HeaderExceptionHandler {
    @ExceptionHandler(MissingRequestHeaderException.class)
    public ResponseEntity<ErrorResponse> handleMissingHeader(
        MissingRequestHeaderException ex
    ) {
        return ResponseEntity.badRequest()
            .body(new ErrorResponse("Missing header: " + ex.getHeaderName()));
    }
}
```

---

### Common Pitfalls

**PITFALL 1: Not handling header case sensitivity**

**Problem:** Headers are case-insensitive in HTTP, but being inconsistent in code.

**Solution:** Use `HttpHeaders` constants for standard headers:
```java
@GetMapping("/data")
public String getData(
    @RequestHeader(HttpHeaders.AUTHORIZATION) String auth,
    @RequestHeader(HttpHeaders.USER_AGENT) String userAgent
) {
    return "Success";
}
```

**PITFALL 2: Not validating header values**

**Problem:**
```java
@GetMapping("/api")
public Data getData(@RequestHeader("X-API-Key") String apiKey) {
    return dataService.get(); // No validation!
}
```

**Solution:**
```java
@GetMapping("/api")
public ResponseEntity<Data> getData(@RequestHeader("X-API-Key") String apiKey) {
    if (!apiKeyService.isValid(apiKey)) {
        return ResponseEntity.status(HttpStatus.FORBIDDEN).build();
    }
    return ResponseEntity.ok(dataService.get());
}
```

---

## 5. @CookieValue

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Extracts values from HTTP cookies. |
| **Use Level** | Method Parameter |
| **Common Use Case** | Session management, user preferences |

### Definition

`@CookieValue` extracts cookie values from HTTP requests and binds them to method parameters. Cookies are small pieces of data stored on the client side, commonly used for session tracking, user preferences, and "remember me" functionality.

**How it works:**
When a request arrives with cookies, Spring reads the Cookie header, finds the specified cookie by name, and binds its value to the method parameter. If the cookie doesn't exist and `required=true`, Spring throws an exception.

### Example

```java
@RestController
@RequestMapping("/api")
public class CookieController {
    
    // Simple cookie
    @GetMapping("/preferences")
    public String getPreferences(@CookieValue("theme") String theme) {
        return "Current theme: " + theme;
    }
    
    // Optional cookie with default
    @GetMapping("/language")
    public String getLanguage(
        @CookieValue(value = "lang", defaultValue = "en") String language
    ) {
        return "Language: " + language;
    }
    
    // Setting a cookie
    @PostMapping("/theme")
    public ResponseEntity<String> setTheme(
        @RequestParam String theme,
        HttpServletResponse response
    ) {
        Cookie cookie = new Cookie("theme", theme);
        cookie.setMaxAge(7 * 24 * 60 * 60); // 7 days
        cookie.setPath("/");
        cookie.setHttpOnly(true);
        response.addCookie(cookie);
        
        return ResponseEntity.ok("Theme set");
    }
}
```

---

### Interview Questions

**Q1: What's the difference between @CookieValue and @RequestHeader?**

**Answer:**
- `@CookieValue`: Extracts specific cookie directly
- `@RequestHeader("Cookie")`: Returns entire Cookie header as string (needs parsing)

```java
// Using @CookieValue (RECOMMENDED)
@GetMapping("/preferences")
public String getTheme(@CookieValue("theme") String theme) {
    return "Theme: " + theme;
}

// Using @RequestHeader (needs parsing)
@GetMapping("/preferences")
public String getTheme(@RequestHeader("Cookie") String cookieHeader) {
    // Must parse: "theme=dark; lang=en"
}
```

**Q2: How do you set and read cookies securely?**

**Answer:**
```java
// Setting secure cookie
@PostMapping("/login")
public void login(HttpServletResponse response) {
    Cookie cookie = new Cookie("sessionId", "abc123");
    cookie.setHttpOnly(true);  // Prevents JavaScript access
    cookie.setSecure(true);    // HTTPS only
    cookie.setMaxAge(3600);    // 1 hour
    response.addCookie(cookie);
}

// Reading cookie
@GetMapping("/profile")
public UserProfile getProfile(@CookieValue("sessionId") String sessionId) {
    return profileService.getBySessionId(sessionId);
}
```

---

### Common Pitfalls

**PITFALL 1: Not handling missing cookies**

**Problem:**
```java
@GetMapping("/profile")
public UserProfile getProfile(@CookieValue("userId") Long userId) {
    return profileService.getByUserId(userId);
}
// Missing cookie -> 400 Bad Request
```

**Solution:**
```java
@GetMapping("/profile")
public ResponseEntity<?> getProfile(
    @CookieValue(value = "userId", required = false) Long userId
) {
    if (userId == null) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
            .body("Please log in");
    }
    return ResponseEntity.ok(profileService.getByUserId(userId));
}
```

**PITFALL 2: Not setting cookie security attributes**

**Problem:**
```java
@PostMapping("/login")
public void login(HttpServletResponse response) {
    Cookie cookie = new Cookie("sessionId", "abc123");
    response.addCookie(cookie); // Insecure!
}
```

**Solution:**
```java
@PostMapping("/login")
public void login(HttpServletResponse response) {
    Cookie cookie = new Cookie("sessionId", "abc123");
    cookie.setHttpOnly(true);  // XSS protection
    cookie.setSecure(true);    // HTTPS only
    cookie.setMaxAge(3600);
    response.addCookie(cookie);
}
```

---

## 6. @ModelAttribute

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Binds form data to Java object. |
| **Use Level** | Method Level, Method Parameter |
| **Common Use Case** | HTML form submissions |

### Definition

`@ModelAttribute` binds HTML form data to Java objects. It's used in traditional web applications (not REST APIs) where forms submit data as `application/x-www-form-urlencoded` instead of JSON. It can also add attributes to the model that are available across all requests.

**How it works:**
When a form is submitted, Spring maps form field names to object properties and creates an instance of the specified class. Field values are automatically converted to appropriate types and injected into the object. You can also use `@ModelAttribute` on methods to add data to the model before any handler method executes.

### Example

```java
@Controller
@RequestMapping("/users")
public class UserController {
    
    // Bind form data to object
    @PostMapping("/register")
    public String register(@ModelAttribute User user) {
        userService.save(user);
        return "redirect:/success";
    }
    
    // With validation
    @PostMapping("/create")
    public String create(
        @Valid @ModelAttribute User user,
        BindingResult result
    ) {
        if (result.hasErrors()) {
            return "user-form";
        }
        userService.save(user);
        return "redirect:/users";
    }
}
```

---

### Interview Questions

**Q1: What's the difference between @ModelAttribute and @RequestBody?**

**Answer:**
- `@ModelAttribute`: Form data (`application/x-www-form-urlencoded`)
- `@RequestBody`: JSON/XML data

```java
// @ModelAttribute - Form submission
@PostMapping("/users/form")
public String create(@ModelAttribute User user) {
    userService.save(user);
    return "redirect:/success";
}
// Content-Type: application/x-www-form-urlencoded
// Body: name=John&email=john@example.com

// @RequestBody - JSON API
@PostMapping("/api/users")
public User create(@RequestBody User user) {
    return userService.save(user);
}
// Content-Type: application/json
// Body: {"name":"John","email":"john@example.com"}
```

**Q2: How does @ModelAttribute work as a method annotation?**

**Answer:**
When on a method, it adds data to the model before any request handler:

```java
@Controller
public class ProductController {
    
    // Runs BEFORE every request handler
    @ModelAttribute("categories")
    public List<String> categories() {
        return Arrays.asList("Electronics", "Books");
    }
    
    @GetMapping("/products")
    public String list() {
        // Model already has "categories"
        return "product-list";
    }
}
```

---

### Common Pitfalls

**PITFALL 1: Using @ModelAttribute with @RestController**

**Problem:**
```java
@RestController
public class ApiController {
    @PostMapping("/users")
    public User create(@ModelAttribute User user) {
        return userService.save(user);
    }
}
// Doesn't work with JSON!
```

**Solution:** Use `@RequestBody` for REST APIs, `@ModelAttribute` for HTML forms.

**PITFALL 2: Not handling validation errors**

**Problem:**
```java
@PostMapping("/register")
public String register(@Valid @ModelAttribute User user) {
    userService.save(user);
    return "redirect:/success";
}
// If validation fails, throws exception!
```

**Solution:**
```java
@PostMapping("/register")
public String register(
    @Valid @ModelAttribute User user,
    BindingResult result
) {
    if (result.hasErrors()) {
        return "registration-form"; // Show errors
    }
    userService.save(user);
    return "redirect:/success";
}
```

---

## 7. @Valid

| Property | Details |
|----------|---------|
| **Package** | `javax.validation` |
| **Description** | Triggers validation of annotated object. |
| **Use Level** | Method Parameter, Field |
| **Common Use Case** | Input validation for request bodies and form data |

### Definition

`@Valid` triggers Bean Validation (JSR-303/JSR-380) on the annotated object. It works with validation annotations like `@NotNull`, `@Size`, `@Email`, etc., to ensure data integrity before processing. When validation fails, Spring throws a `MethodArgumentNotValidException`.

**How it works:**
Before executing the method, Spring validates the object against its validation annotations. If any constraint is violated, it collects all validation errors and throws an exception. You can catch this exception globally using `@ControllerAdvice` to return custom error responses.

### Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping
    public User create(@Valid @RequestBody UserDTO dto) {
        return userService.create(dto);
    }
}

// DTO with validation
public class UserDTO {
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50)
    private String name;
    
    @Email(message = "Invalid email")
    private String email;
    
    @Min(value = 18, message = "Must be 18+")
    private Integer age;
}
```

### Common Validation Annotations

| Annotation | Description |
|-----------|-------------|
| `@NotNull` | Value cannot be null |
| `@NotBlank` | String not null/empty/whitespace |
| `@Email` | Valid email format |
| `@Size` | String/collection size constraint |
| `@Min` / `@Max` | Numeric min/max |
| `@Pattern` | Regex pattern match |

---

### Interview Questions

**Q1: How do you handle validation errors globally?**

**Answer:**
```java
@RestControllerAdvice
public class ValidationExceptionHandler {
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidation(
        MethodArgumentNotValidException ex
    ) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );
        return ResponseEntity.badRequest().body(errors);
    }
}
```

**Q2: How do you create custom validation annotations?**

**Answer:**
```java
// 1. Create annotation
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueEmailValidator.class)
public @interface UniqueEmail {
    String message() default "Email already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 2. Create validator
@Component
public class UniqueEmailValidator 
    implements ConstraintValidator<UniqueEmail, String> {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        return !userRepository.existsByEmail(email);
    }
}

// 3. Use in DTO
public class UserDTO {
    @UniqueEmail
    @Email
    private String email;
}
```

---

### Common Pitfalls

**PITFALL 1: Forgetting @Valid annotation**

**Problem:**
```java
@PostMapping("/users")
public User create(@RequestBody UserDTO dto) { // Missing @Valid
    return userService.create(dto);
}
// Validation annotations are ignored!
```

**Solution:**
```java
@PostMapping("/users")
public User create(@Valid @RequestBody UserDTO dto) {
    return userService.create(dto);
}
```

**PITFALL 2: Not validating nested objects**

**Problem:**
```java
public class OrderDTO {
    @NotNull
    private AddressDTO address; // Missing @Valid
}
// AddressDTO fields are NOT validated!
```

**Solution:**
```java
public class OrderDTO {
    @Valid @NotNull
    private AddressDTO address;
}

public class AddressDTO {
    @NotBlank
    private String street;
}
```

---

## 📚 Summary

This guide covered Path & Parameter annotations:
- **@PathVariable** - Extract from URL path
- **@RequestParam** - Extract from query string
- **@RequestBody** - Parse JSON/XML to object
- **@RequestHeader** - Extract from headers
- **@CookieValue** - Extract from cookies
- **@ModelAttribute** - Bind form data
- **@Valid** - Trigger validation

Each includes: definition, property table, examples, 2 interview Q&A, 2 common pitfalls.