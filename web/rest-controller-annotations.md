# Spring REST Controller Annotations - Complete Guide

A comprehensive guide to Spring annotations used for building REST APIs and controllers.

---

## 📑 Table of Contents

1. [@RestController](#1-restcontroller)
2. [@Controller](#2-controller)
3. [@GetMapping](#3-getmapping)
4. [@PostMapping](#4-postmapping)
5. [@PutMapping](#5-putmapping)
6. [@PatchMapping](#6-patchmapping)
7. [@DeleteMapping](#7-deletemapping)
8. [@ResponseBody](#8-responsebody)
9. [@ResponseStatus](#9-responsestatus)

---

## 1. @RestController

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Combines @Controller and @ResponseBody for REST API development |
| **Use Level** | Class Level |
| **Common Use Case** | Building RESTful web services that return JSON/XML data |

### Definition

`@RestController` is a specialized version of `@Controller` that combines `@Controller` and `@ResponseBody`. It marks a class as a web controller where every method automatically serializes return objects into JSON or XML and writes them directly to the HTTP response body.

**How it works:**
When Spring sees `@RestController`, it registers the class as a Spring component and enables automatic serialization. The framework uses Jackson (for JSON) or JAXB (for XML) to convert Java objects to the response format based on the request's `Accept` header.

### Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping
    public List<User> getAllUsers() {
        return userService.findAll();
    }
    
    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.save(user);
    }
}
```

---

### Interview Questions

**Q1: What is the difference between @Controller and @RestController?**

**Answer:**
- `@RestController` = `@Controller` + `@ResponseBody`
- `@Controller` returns view names (HTML pages), `@RestController` returns data (JSON/XML)
- With `@Controller`, you need `@ResponseBody` on each method that returns data
- `@RestController` automatically applies `@ResponseBody` to all methods

**Q2: How does @RestController convert Java objects to JSON?**

**Answer:**
Spring uses Jackson library through `HttpMessageConverter`. When a method returns an object, `MappingJackson2HttpMessageConverter` converts it to JSON and writes it to the response body with Content-Type `application/json`.

---

### Common Pitfalls

**PITFALL 1: Trying to return view names from @RestController**

**Problem:**
```java
@RestController
public class HomeController {
    @GetMapping("/home")
    public String home() {
        return "home"; // Returns literal string "home", not home.html!
    }
}
```

**Solution:**
```java
@Controller // Use @Controller for views
public class HomeController {
    @GetMapping("/home")
    public String home() {
        return "home"; // Returns home.html template
    }
}
```

**PITFALL 2: Circular reference causing infinite loop**

**Problem:**
```java
@Entity
public class User {
    @OneToMany(mappedBy = "user")
    private List<Order> orders;
}

@Entity
public class Order {
    @ManyToOne
    private User user; // Circular reference!
}
```

**Solution:**
```java
@Entity
public class Order {
    @ManyToOne
    @JsonIgnore // Break the cycle
    private User user;
}
```

---

## 2. @Controller

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.stereotype` |
| **Description** | Marks a class as a Spring MVC controller for handling web requests |
| **Use Level** | Class Level |
| **Common Use Case** | Server-side rendering with Thymeleaf, JSP, or other template engines |

### Definition

`@Controller` is a Spring stereotype annotation that marks a class as a web controller in the MVC pattern. It's used for traditional web applications that render server-side views. Methods return view names which Spring resolves to template files using a ViewResolver.

**How it works:**
When a request arrives, Spring's DispatcherServlet routes it to the appropriate `@Controller` method based on URL mappings. The method processes the request, populates a Model with data, and returns a view name. Spring uses ViewResolver to find and render the corresponding template.

### Example

```java
@Controller
public class HomeController {
    
    @GetMapping("/")
    public String home(Model model) {
        model.addAttribute("message", "Welcome!");
        return "home"; // Returns home.html
    }
    
    @PostMapping("/register")
    public String register(@ModelAttribute User user) {
        userService.save(user);
        return "redirect:/success";
    }
}
```

---

### Interview Questions

**Q1: When should you use @Controller instead of @RestController?**

**Answer:**
Use `@Controller` for traditional web apps with server-side rendering (Thymeleaf, JSP). Use `@RestController` for REST APIs that return JSON/XML data.

**Q2: What is the Model object used for?**

**Answer:**
Model is a container that holds data to pass to the view. Attributes added to Model are available in the template for rendering.

```java
@GetMapping("/profile")
public String profile(Model model) {
    model.addAttribute("user", currentUser);
    return "profile"; // Access in view: ${user.name}
}
```

---

### Common Pitfalls

**PITFALL 1: Missing view resolver configuration**

**Problem:**
```java
@Controller
public class HomeController {
    @GetMapping("/")
    public String home() {
        return "home"; // Error: Could not resolve view
    }
}
```

**Solution:**
```properties
# application.properties
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

**PITFALL 2: Not using @ModelAttribute for form binding**

**Problem:**
```java
@PostMapping("/submit")
public String submit(String name, String email) {
    User user = new User();
    user.setName(name);
    user.setEmail(email);
    return "success";
}
```

**Solution:**
```java
@PostMapping("/submit")
public String submit(@ModelAttribute User user) {
    userService.save(user);
    return "redirect:/success";
}
```

---

## 3. @GetMapping

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Maps HTTP GET requests to handler methods |
| **Use Level** | Method Level |
| **Common Use Case** | Retrieving/reading data, searching, listing resources |

### Definition

`@GetMapping` is a shortcut for `@RequestMapping(method = RequestMethod.GET)`. It maps HTTP GET requests to specific handler methods. GET requests are for retrieving data without modifying server state (idempotent and safe).

**How it works:**
When an HTTP GET request arrives, Spring's DispatcherServlet matches the URL to the method with `@GetMapping`. The method executes and returns data (JSON for `@RestController`) or a view name (for `@Controller`).

### Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping
    public List<User> getAllUsers() {
        return userService.findAll();
    }
    
    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    @GetMapping("/search")
    public List<User> search(@RequestParam String name) {
        return userService.findByName(name);
    }
}
```

---

### Interview Questions

**Q1: Why should GET requests be idempotent and safe?**

**Answer:**
GET requests should be idempotent (same result every time) and safe (no side effects). They can be cached, bookmarked, and crawled by search engines safely.

**Q2: How do you handle "resource not found" in GET requests?**

**Answer:**
Use ResponseEntity to return proper HTTP status codes:

```java
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return userService.findById(id)
        .map(ResponseEntity::ok)              // 200 if found
        .orElse(ResponseEntity.notFound().build()); // 404 if not found
}
```

---

### Common Pitfalls

**PITFALL 1: Using GET for operations that modify data**

**Problem:**
```java
@GetMapping("/users/{id}/delete")
public String delete(@PathVariable Long id) {
    userService.delete(id); // WRONG - modifies data
    return "Deleted";
}
```

**Solution:**
```java
@DeleteMapping("/users/{id}")
public void delete(@PathVariable Long id) {
    userService.delete(id);
}
```

**PITFALL 2: Not handling null values**

**Problem:**
```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id); // Returns null if not found
}
```

**Solution:**
```java
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return userService.findById(id)
        .map(ResponseEntity::ok)
        .orElseThrow(() -> new UserNotFoundException(id));
}
```

---

## 4. @PostMapping

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Maps HTTP POST requests to handler methods |
| **Use Level** | Method Level |
| **Common Use Case** | Creating new resources, submitting forms, uploading data |

### Definition

`@PostMapping` is a shortcut for `@RequestMapping(method = RequestMethod.POST)`. It maps HTTP POST requests to handler methods. POST is used to create new resources and is not idempotent—multiple identical requests may create multiple resources.

**How it works:**
When a POST request arrives, Spring routes it to the matching `@PostMapping` method. The method receives data via `@RequestBody` (JSON) or `@ModelAttribute` (forms), processes it, and typically returns the created resource with HTTP status 201 (Created).

### Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User saved = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }
    
    @PostMapping("/register")
    public User register(@Valid @RequestBody UserDTO dto) {
        return userService.register(dto);
    }
}
```

---

### Interview Questions

**Q1: What's the difference between POST and PUT?**

**Answer:**
POST creates new resources (not idempotent), PUT updates existing resources (idempotent). POST URL points to collection, PUT points to specific resource.

**Q2: What HTTP status code should POST return?**

**Answer:**
**201 Created** for successfully created resources, **200 OK** for successful requests without creating resources, **400 Bad Request** for invalid data.

---

### Common Pitfalls

**PITFALL 1: Forgetting @RequestBody**

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

**PITFALL 2: Not returning proper status code**

**Problem:**
```java
@PostMapping("/users")
public User create(@RequestBody User user) {
    return userService.save(user); // Returns 200 OK
}
```

**Solution:**
```java
@PostMapping("/users")
public ResponseEntity<User> create(@RequestBody User user) {
    User saved = userService.save(user);
    return ResponseEntity.status(HttpStatus.CREATED).body(saved); // 201
}
```

---

## 5. @PutMapping

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Maps HTTP PUT requests to handler methods |
| **Use Level** | Method Level |
| **Common Use Case** | Updating/replacing entire resources |

### Definition

`@PutMapping` is a shortcut for `@RequestMapping(method = RequestMethod.PUT)`. PUT is used to update or replace an entire resource. It's idempotent—multiple identical requests produce the same result. The client must send the complete resource representation.

**How it works:**
When a PUT request arrives with a resource identifier, Spring routes it to the `@PutMapping` method. The method receives complete updated data via `@RequestBody`, replaces the existing resource, and returns the updated resource with HTTP status 200.

### Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        return userService.update(id, user);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> update(
        @PathVariable Long id,
        @Valid @RequestBody User user
    ) {
        User updated = userService.update(id, user);
        return ResponseEntity.ok(updated);
    }
}
```

---

### Interview Questions

**Q1: What's the difference between PUT and PATCH?**

**Answer:**
PUT replaces the entire resource (send all fields), PATCH updates specific fields (send only changed fields).

**Q2: Should PUT be idempotent?**

**Answer:**
Yes! Calling PUT multiple times with the same data should always produce the same result. Don't increment counters or perform calculations in PUT.

---

### Common Pitfalls

**PITFALL 1: Using PUT when PATCH is more appropriate**

**Problem:**
```java
@PutMapping("/users/{id}")
public User update(@PathVariable Long id, @RequestBody User user) {
    return userService.update(id, user); // Client must send all fields
}
```

**Solution:**
```java
@PatchMapping("/users/{id}")
public User partialUpdate(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
    return userService.partialUpdate(id, updates); // Send only changed fields
}
```

**PITFALL 2: Not making PUT idempotent**

**Problem:**
```java
@PutMapping("/users/{id}")
public User update(@PathVariable Long id, @RequestBody User user) {
    user.setVisitCount(user.getVisitCount() + 1); // Not idempotent!
    return userService.update(id, user);
}
```

**Solution:**
```java
@PutMapping("/users/{id}")
public User update(@PathVariable Long id, @RequestBody User user) {
    return userService.update(id, user); // Only set exact values
}
```

---

## 6. @PatchMapping

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Maps HTTP PATCH requests to handler methods |
| **Use Level** | Method Level |
| **Common Use Case** | Partial updates of resources |

### Definition

`@PatchMapping` is a shortcut for `@RequestMapping(method = RequestMethod.PATCH)`. PATCH is for partial updates—send only the fields that need changing, not the entire resource. This is more efficient than PUT for large resources.

**How it works:**
When a PATCH request arrives, Spring routes it to the matching method. The method receives only the changed fields (typically as a Map or DTO with optional fields), updates those specific fields, and preserves the rest of the resource's data.

### Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PatchMapping("/{id}")
    public User partialUpdate(
        @PathVariable Long id,
        @RequestBody Map<String, Object> updates
    ) {
        return userService.partialUpdate(id, updates);
    }
    
    @PatchMapping("/{id}/email")
    public User updateEmail(@PathVariable Long id, @RequestBody String email) {
        return userService.updateEmail(id, email);
    }
}
```

---

### Interview Questions

**Q1: How do you implement partial updates with PATCH?**

**Answer:**
Use a Map to accept only the fields being updated:

```java
@PatchMapping("/users/{id}")
public User patch(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
    User user = userService.findById(id);
    if (updates.containsKey("name")) {
        user.setName((String) updates.get("name"));
    }
    return userService.save(user);
}
```

**Q2: Is PATCH idempotent?**

**Answer:**
PATCH can be idempotent (setting specific values) but isn't required to be (incrementing counters).

---

### Common Pitfalls

**PITFALL 1: Not handling null vs missing fields**

**Problem:**
```java
@PatchMapping("/users/{id}")
public User patch(@PathVariable Long id, @RequestBody User user) {
    User existing = userService.findById(id);
    existing.setName(user.getName()); // Sets to null if not sent!
    return userService.save(existing);
}
```

**Solution:**
```java
@PatchMapping("/users/{id}")
public User patch(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
    User existing = userService.findById(id);
    if (updates.containsKey("name")) {
        existing.setName((String) updates.get("name"));
    }
    return userService.save(existing);
}
```

**PITFALL 2: Not validating partial updates**

**Problem:**
```java
@PatchMapping("/users/{id}")
public User patch(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
    return userService.patch(id, updates); // No validation!
}
```

**Solution:**
```java
@PatchMapping("/users/{id}")
public User patch(@PathVariable Long id, @Valid @RequestBody UserPatchDTO dto) {
    return userService.patch(id, dto);
}

public class UserPatchDTO {
    @Size(min = 2, max = 50)
    private String name;
}
```

---

## 7. @DeleteMapping

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Maps HTTP DELETE requests to handler methods |
| **Use Level** | Method Level |
| **Common Use Case** | Deleting/removing resources |

### Definition

`@DeleteMapping` is a shortcut for `@RequestMapping(method = RequestMethod.DELETE)`. DELETE removes resources from the server. It should be idempotent—deleting the same resource multiple times results in the same state.

**How it works:**
When a DELETE request arrives with a resource identifier, Spring routes it to the `@DeleteMapping` method. The method deletes the resource and typically returns HTTP status 204 (No Content) for success or 404 if the resource doesn't exist.

### Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        if (!userService.exists(id)) {
            return ResponseEntity.notFound().build();
        }
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

### Interview Questions

**Q1: What HTTP status code should DELETE return?**

**Answer:**
**204 No Content** (most common), **200 OK** with response body, or **404 Not Found** if resource doesn't exist.

**Q2: Should DELETE be idempotent?**

**Answer:**
Yes! Deleting the same resource multiple times should result in the same state (resource doesn't exist).

---

### Common Pitfalls

**PITFALL 1: Not checking if resource exists**

**Problem:**
```java
@DeleteMapping("/{id}")
public void delete(@PathVariable Long id) {
    userService.delete(id); // Returns success even if doesn't exist
}
```

**Solution:**
```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> delete(@PathVariable Long id) {
    if (!userService.exists(id)) {
        return ResponseEntity.notFound().build();
    }
    userService.delete(id);
    return ResponseEntity.noContent().build();
}
```

**PITFALL 2: Not handling cascade deletes**

**Problem:**
```java
@DeleteMapping("/users/{id}")
public void delete(@PathVariable Long id) {
    userService.delete(id); // What about related data?
}
```

**Solution:**
```java
@DeleteMapping("/users/{id}")
public ResponseEntity<String> delete(@PathVariable Long id) {
    if (userService.hasOrders(id)) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
            .body("Cannot delete user with orders");
    }
    userService.delete(id);
    return ResponseEntity.noContent().build();
}
```

---

## 8. @ResponseBody

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Serializes method return value to HTTP response body |
| **Use Level** | Method Level, Class Level |
| **Common Use Case** | Returning data (JSON/XML) from @Controller methods |

### Definition

`@ResponseBody` tells Spring to serialize the method's return value and write it directly to the HTTP response body as JSON/XML, rather than treating it as a view name. It's automatically included in `@RestController`.

**How it works:**
When a method with `@ResponseBody` returns an object, Spring uses HttpMessageConverter (Jackson for JSON) to convert it to the response format and sets the appropriate Content-Type header.

### Example

```java
@Controller
public class DataController {
    
    @GetMapping("/api/users")
    @ResponseBody
    public List<User> getUsers() {
        return userService.findAll();
    }
    
    @GetMapping("/page")
    public String getPage() {
        return "page"; // Returns page.html (no @ResponseBody)
    }
}
```

---

### Interview Questions

**Q1: What's the difference between @ResponseBody and @RestController?**

**Answer:**
`@RestController` = `@Controller` + `@ResponseBody` on all methods. Use `@ResponseBody` when you have a `@Controller` with some methods returning views and others returning data.

**Q2: How does @ResponseBody serialize objects?**

**Answer:**
Spring uses Jackson's `MappingJackson2HttpMessageConverter` to convert Java objects to JSON and writes it to the response body with Content-Type `application/json`.

---

### Common Pitfalls

**PITFALL 1: Forgetting @ResponseBody in @Controller**

**Problem:**
```java
@Controller
public class ApiController {
    @GetMapping("/users")
    public List<User> getUsers() { // Missing @ResponseBody
        return userService.findAll(); // Spring looks for "User" view
    }
}
```

**Solution:**
```java
@Controller
public class ApiController {
    @GetMapping("/users")
    @ResponseBody
    public List<User> getUsers() {
        return userService.findAll();
    }
}
```

**PITFALL 2: Circular reference in JSON**

**Problem:**
```java
@Entity
public class User {
    @OneToMany
    private List<Order> orders;
}
@Entity
public class Order {
    @ManyToOne
    private User user; // Infinite loop!
}
```

**Solution:**
```java
@Entity
public class Order {
    @ManyToOne
    @JsonIgnore
    private User user;
}
```

---

## 9. @ResponseStatus

| Property | Details |
|----------|---------|
| **Package** | `org.springframework.web.bind.annotation` |
| **Description** | Sets HTTP status code for response |
| **Use Level** | Method Level, Class Level (Exception classes) |
| **Common Use Case** | Custom HTTP status codes, exception status mapping |

### Definition

`@ResponseStatus` sets the HTTP status code for the response. It can be applied to methods to set the status code for successful responses, or to exception classes to define what status code should be returned when that exception is thrown.

**How it works:**
When a method with `@ResponseStatus` executes successfully, Spring sets the specified HTTP status code in the response. When applied to an exception class, Spring automatically returns that status code whenever the exception is thrown.

### Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED) // Returns 201
    public User createUser(@RequestBody User user) {
        return userService.save(user);
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT) // Returns 204
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}

@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "User not found")
public class UserNotFoundException extends RuntimeException {
}
```

---

### Interview Questions

**Q1: When should you use @ResponseStatus vs ResponseEntity?**

**Answer:**
Use `@ResponseStatus` for simple, static status codes. Use `ResponseEntity` when you need dynamic status codes, custom headers, or conditional responses.

**Q2: How do you use @ResponseStatus with exceptions?**

**Answer:**
Apply it to exception classes to automatically return specific status codes:

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
}

@GetMapping("/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException()); // Returns 404
}
```

---

### Common Pitfalls

**PITFALL 1: Using @ResponseStatus when ResponseEntity is better**

**Problem:**
```java
@GetMapping("/users/{id}")
@ResponseStatus(HttpStatus.OK) // Inflexible
public User getUser(@PathVariable Long id) {
    return userService.findById(id); // Always 200, even if not found
}
```

**Solution:**
```java
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return userService.findById(id)
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
}
```

**PITFALL 2: Not providing meaningful exception messages**

**Problem:**
```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class NotFoundException extends RuntimeException {
    // No custom message
}
```

**Solution:**
```java
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "Resource not found")
public class NotFoundException extends RuntimeException {
    public NotFoundException(String message) {
        super(message);
    }
}
```

---

## 📚 Summary

This guide covered 9 REST Controller annotations:
- **@RestController** - REST API controller
- **@Controller** - MVC controller
- **@GetMapping** - GET requests
- **@PostMapping** - POST requests
- **@PutMapping** - PUT requests
- **@PatchMapping** - PATCH requests
- **@DeleteMapping** - DELETE requests
- **@ResponseBody** - Serialize to JSON/XML
- **@ResponseStatus** - Set HTTP status code

Each annotation includes detailed explanation, examples, interview questions, and common pitfalls.