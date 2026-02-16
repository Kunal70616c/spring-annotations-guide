# Spring Exception Handler Annotations - Complete Guide

A comprehensive guide to Spring annotations used for exception handling and error management in REST APIs.

---

## 📑 Table of Contents

1. [@ExceptionHandler](#1-exceptionhandler)
2. [@ControllerAdvice](#2-controlleradvice)
3. [@RestControllerAdvice](#3-restcontrolleradvice)
4. [@ResponseStatus](#4-responsestatus)

---

## 1. @ExceptionHandler

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Handles specific exceptions in a controller. |
| **Use Level** | Method Level |
| **Common Use Case** | Controller-specific exception handling |

### Definition

`@ExceptionHandler` is used to handle exceptions at the controller level. It defines a method that will be invoked when a specific exception is thrown within that controller. This allows you to customize error responses for different types of exceptions, providing meaningful error messages and appropriate HTTP status codes.

**How it works:**
When an exception is thrown in a controller method, Spring searches for an `@ExceptionHandler` method in the same controller that can handle that exception type. If found, it executes that handler method instead of propagating the exception. The handler can return a custom error response, view name, or ResponseEntity.

### Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
    
    // Handle UserNotFoundException in this controller
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    // Handle validation errors
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

---

### Interview Questions

**Q1: What's the difference between @ExceptionHandler and @ControllerAdvice?**

**Answer:**
- `@ExceptionHandler`: Handles exceptions within a single controller only
- `@ControllerAdvice`: Handles exceptions globally across all controllers

```java
// @ExceptionHandler - Local to UserController
@RestController
public class UserController {
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handleNotFound(UserNotFoundException ex) {
        return ResponseEntity.notFound().build();
    }
}

// @ControllerAdvice - Global for all controllers
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse(ex.getMessage()));
    }
}
```

**Q2: Can you have both @ExceptionHandler in controller and @ControllerAdvice?**

**Answer:**
Yes! Controller-level handlers take precedence over global handlers. Spring checks the controller first, then falls back to `@ControllerAdvice` if no local handler is found.

---

### Common Pitfalls

**PITFALL 1: Not returning proper error response structure**

**Problem:**
```java
@ExceptionHandler(UserNotFoundException.class)
public ResponseEntity<String> handleNotFound(UserNotFoundException ex) {
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    // Just a string - no status code or timestamp info
}
```

**Solution:**
```java
@ExceptionHandler(UserNotFoundException.class)
public ResponseEntity<ErrorResponse> handleNotFound(UserNotFoundException ex) {
    ErrorResponse error = new ErrorResponse(
        HttpStatus.NOT_FOUND.value(),
        ex.getMessage(),
        LocalDateTime.now()
    );
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
}
```

**PITFALL 2: Duplicating exception handlers across controllers**

**Problem:** Same exception handler code repeated in multiple controllers.

**Solution:** Move common handlers to a global `@ControllerAdvice` class.

---

## 2. @ControllerAdvice

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Global exception handling across all controllers. |
| **Use Level** | Class Level |
| **Common Use Case** | Centralized exception handling, global model attributes |

### Definition

`@ControllerAdvice` allows you to handle exceptions globally across your entire application. Instead of repeating exception handling logic in every controller, you create a single class annotated with `@ControllerAdvice` containing `@ExceptionHandler` methods that apply to all controllers. It can also add global model attributes and perform data binding across all controllers.

**How it works:**
Spring scans for classes annotated with `@ControllerAdvice` at startup. When an exception is thrown anywhere in the application and no local `@ExceptionHandler` is found, Spring looks for a matching handler in `@ControllerAdvice` classes. You can also limit the scope to specific packages or controller types using attributes.

### Example

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    // Handle UserNotFoundException globally
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(
        UserNotFoundException ex
    ) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    // Handle validation errors
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
    
    // Catch-all handler
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred"
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

---

### Interview Questions

**Q1: What's the difference between @ControllerAdvice and @RestControllerAdvice?**

**Answer:**
- `@RestControllerAdvice` = `@ControllerAdvice` + `@ResponseBody`
- With `@ControllerAdvice`, you need `@ResponseBody` on each handler method
- `@RestControllerAdvice` automatically applies `@ResponseBody` to all methods

```java
// @ControllerAdvice - Need @ResponseBody
@ControllerAdvice
public class GlobalHandler {
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseBody
    public ResponseEntity<ErrorResponse> handleNotFound(UserNotFoundException ex) {
        return ResponseEntity.notFound().build();
    }
}

// @RestControllerAdvice - No @ResponseBody needed
@RestControllerAdvice
public class RestGlobalHandler {
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(UserNotFoundException ex) {
        return ResponseEntity.notFound().build();
    }
}
```

**Q2: How do you limit @ControllerAdvice to specific packages or controllers?**

**Answer:**
Use attributes like `basePackages`, `annotations`, or `assignableTypes`:

```java
// Limit by package
@ControllerAdvice(basePackages = "com.example.api")
public class ApiExceptionHandler {
    // Only handles exceptions from com.example.api package
}

// Limit by annotation
@ControllerAdvice(annotations = RestController.class)
public class RestExceptionHandler {
    // Only handles exceptions from @RestController classes
}
```

---

### Common Pitfalls

**PITFALL 1: Multiple @ControllerAdvice classes handling same exception**

**Problem:**
```java
@ControllerAdvice
public class Handler1 {
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handle(UserNotFoundException ex) { }
}

@ControllerAdvice
public class Handler2 {
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handle(UserNotFoundException ex) { }
}
// Ambiguous! Which one to use?
```

**Solution:** Use `@Order` to set precedence or limit scope with `basePackages`.

```java
@ControllerAdvice
@Order(Ordered.HIGHEST_PRECEDENCE)
public class PrimaryHandler {
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(UserNotFoundException ex) {
        // This one is used
    }
}
```

**PITFALL 2: Not logging exceptions**

**Problem:**
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("Error occurred"));
        // Exception details lost!
    }
}
```

**Solution:**
```java
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        log.error("Unexpected error occurred", ex); // Log with stack trace
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("An unexpected error occurred"));
    }
}
```

---

## 3. @RestControllerAdvice

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Combines @ControllerAdvice and @ResponseBody for REST APIs. |
| **Use Level** | Class Level |
| **Common Use Case** | Global REST API exception handling with automatic JSON serialization |

### Definition

`@RestControllerAdvice` is a specialized version of `@ControllerAdvice` that combines it with `@ResponseBody`. It's specifically designed for REST APIs where all exception handlers return data (JSON/XML) instead of view names. This eliminates the need to annotate each handler method with `@ResponseBody`.

**How it works:**
When an exception occurs in any REST controller, Spring routes it to the appropriate handler in the `@RestControllerAdvice` class. The handler method's return value is automatically serialized to JSON (using Jackson) and written to the response body with the appropriate Content-Type header.

### Example

```java
@RestControllerAdvice
@Slf4j
public class RestExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(
        ResourceNotFoundException ex
    ) {
        log.warn("Resource not found: {}", ex.getMessage());
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidation(
        MethodArgumentNotValidException ex
    ) {
        Map<String, String> fieldErrors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            fieldErrors.put(error.getField(), error.getDefaultMessage())
        );
        
        ValidationErrorResponse response = new ValidationErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed",
            fieldErrors
        );
        return ResponseEntity.badRequest().body(response);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        log.error("Unexpected error", ex);
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred"
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

---

### Interview Questions

**Q1: How would you structure a complete REST API error handling solution?**

**Answer:**
Create a hierarchy of custom exceptions, structured error responses, and a global handler:

```java
// 1. Custom exceptions
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

// 2. Error response DTO
public class ErrorResponse {
    private int status;
    private String message;
    private LocalDateTime timestamp;
    // getters, setters
}

// 3. Global handler
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
}
```

**Q2: How do you handle different error formats for different API versions?**

**Answer:**
Use separate `@RestControllerAdvice` classes with `basePackages`:

```java
// API v1
@RestControllerAdvice(basePackages = "com.example.api.v1")
public class ApiV1ExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<SimpleErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new SimpleErrorResponse(404, ex.getMessage()));
    }
}

// API v2
@RestControllerAdvice(basePackages = "com.example.api.v2")
public class ApiV2ExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<DetailedErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new DetailedErrorResponse(404, ex.getMessage(), LocalDateTime.now()));
    }
}
```

---

### Common Pitfalls

**PITFALL 1: Returning too much information in error responses**

**Problem:**
```java
@RestControllerAdvice
public class ExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handle(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(ex.toString() + "\n" + Arrays.toString(ex.getStackTrace()));
        // Exposes internal details! Security risk!
    }
}
```

**Solution:**
```java
@RestControllerAdvice
@Slf4j
public class SecureExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        log.error("Unexpected error occurred", ex); // Log internally
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred" // Safe message
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

**PITFALL 2: Not handling validation errors properly**

**Problem:**
```java
@RestControllerAdvice
public class ExceptionHandler {
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<String> handleValidation(MethodArgumentNotValidException ex) {
        return ResponseEntity.badRequest().body("Validation failed");
        // Doesn't tell which fields failed!
    }
}
```

**Solution:**
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

---

## 4. @ResponseStatus

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Sets HTTP status code for response or exception. |
| **Use Level** | Method Level, Class Level (Exception classes) |
| **Common Use Case** | Custom HTTP status codes for exceptions |

### Definition

`@ResponseStatus` is used to mark a method or exception class with the HTTP status code that should be returned. When applied to exception classes, it automatically maps the exception to a specific HTTP status without needing explicit exception handlers. This provides a simple way to define what status code should be returned when specific exceptions are thrown.

**How it works:**
When Spring encounters an exception annotated with `@ResponseStatus`, it automatically returns the specified HTTP status code and optional reason message. For methods, it sets the status code for successful responses. This happens before any exception handler or default error handling kicks in.

### Example

```java
// On exception classes
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "User not found")
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(Long id) {
        super("User not found with id: " + id);
    }
}

@ResponseStatus(value = HttpStatus.CONFLICT, reason = "Resource already exists")
public class DuplicateResourceException extends RuntimeException {
    public DuplicateResourceException(String message) {
        super(message);
    }
}

// Using in controller
@RestController
public class ProductController {
    
    @GetMapping("/products/{id}")
    public Product getProduct(@PathVariable Long id) {
        return productService.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
        // Automatically returns 404
    }
    
    @PostMapping("/products")
    public Product create(@RequestBody Product product) {
        if (productService.existsByName(product.getName())) {
            throw new DuplicateResourceException("Product already exists");
            // Automatically returns 409
        }
        return productService.save(product);
    }
}
```

---

### Interview Questions

**Q1: How do you use @ResponseStatus with custom exceptions?**

**Answer:**
Annotate your custom exception class with `@ResponseStatus`:

```java
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "Resource not found")
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

// Usage
@GetMapping("/products/{id}")
public Product getProduct(@PathVariable Long id) {
    return productService.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    // Automatically returns 404
}
```

**Q2: What's the difference between @ResponseStatus on exception vs @ExceptionHandler?**

**Answer:**
- `@ResponseStatus on exception`: Simple, automatic status mapping, limited customization
- `@ExceptionHandler`: Full control over response body, headers, and logic

```java
// @ResponseStatus - Simple
@ResponseStatus(HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException { }
// Returns: 404 with generic message

// @ExceptionHandler - Full control
@ExceptionHandler(UserNotFoundException.class)
public ResponseEntity<ErrorResponse> handle(UserNotFoundException ex) {
    ErrorResponse error = new ErrorResponse(404, ex.getMessage(), LocalDateTime.now());
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
}
// Returns: 404 with custom JSON response
```

---

### Common Pitfalls

**PITFALL 1: Using @ResponseStatus without custom message**

**Problem:**
```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException {
    // No custom message provided
}
```

**Solution:**
```java
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "User not found")
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(Long id) {
        super("User not found with id: " + id);
    }
}
```

**PITFALL 2: Expecting custom response body with @ResponseStatus**

**Problem:**
```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException {
    private Long userId;
    // Expecting userId in response - won't work!
}
```

**Solution:** Use `@ExceptionHandler` for custom response body.

```java
@RestControllerAdvice
public class ExceptionHandler {
    @org.springframework.web.bind.annotation.ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            404,
            ex.getMessage(),
            ex.getUserId()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
}
```

---

## 📚 Summary

This guide covered Exception Handler annotations:
- **@ExceptionHandler** - Controller-level exception handling
- **@ControllerAdvice** - Global exception handling
- **@RestControllerAdvice** - Global REST API exception handling
- **@ResponseStatus** - Exception status code mapping

Each includes: definition, property table, examples, 2 interview Q&A, 2 common pitfalls.