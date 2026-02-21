# 06 — REST APIs (Interview Ready)

> **Level:** 3-4 Years Experience | **Priority:** 🔴 Critical  
> Covers: REST Principles, HTTP Methods, Status Codes, Versioning, Pagination, HATEOAS, Best Practices

---

## 📌 Table of Contents

1. [REST Fundamentals](#1-rest-fundamentals) — 6 Constraints, Resource Naming, Richardson Maturity
2. [HTTP Methods & Status Codes](#2-http-methods--status-codes)
3. [Spring Boot REST Controller](#3-spring-boot-rest-controller) — CRUD, DTO, ResponseEntity
4. [Request Annotations Compared](#4-request-annotations-compared)
5. [Request Validation](#5-request-validation) — Bean Validation, Custom Validator
6. [Pagination & Sorting](#6-pagination--sorting)
7. [API Versioning](#7-api-versioning)
8. [File Upload & Download](#8-file-upload--download)
9. [Swagger / OpenAPI](#9-swagger--openapi)
10. [HATEOAS](#10-hateoas)
11. [ETags & Caching](#11-etags--caching)
12. [REST Best Practices](#12-rest-best-practices)
13. [REST vs SOAP vs GraphQL](#13-rest-vs-soap-vs-graphql)
14. [Interview Questions & Answers](#14-interview-questions--answers) — 14 Q&As

---

## 1. REST Fundamentals

**REST** = **RE**presentational **S**tate **T**ransfer (Architectural Style)

### 6 Constraints
| Constraint | Meaning |
|-----------|---------|
| **Client-Server** | Separation of concerns |
| **Stateless** | No session on server; each request is self-contained |
| **Cacheable** | Responses must indicate cacheability |
| **Uniform Interface** | Consistent URI structure, standard methods |
| **Layered System** | Client doesn't know if connected to end server or intermediary |
| **Code on Demand** (optional) | Server can send executable code |

### Resource Naming Conventions
```
✅ GOOD (nouns, plural, lowercase)
GET    /api/v1/users
GET    /api/v1/users/123
GET    /api/v1/users/123/orders
POST   /api/v1/users
PUT    /api/v1/users/123
DELETE /api/v1/users/123

❌ BAD (verbs, inconsistent)
GET    /api/getUser/123
POST   /api/createUser
GET    /api/User/123
DELETE /api/deleteUser/123
```

### Richardson Maturity Model

```
Level 0: Swamp of POX — single endpoint, single method (POST everything)
  POST /api   {"action": "getUser", "id": 123}

Level 1: Resources — separate URIs per resource
  POST /api/users/123  {"action": "get"}

Level 2: HTTP Methods — proper use of GET, POST, PUT, DELETE + status codes
  GET /api/users/123  → 200 OK

Level 3: HATEOAS — responses include links to related resources ("Glory of REST")
  GET /api/users/123 → { ... "_links": { "orders": "/api/users/123/orders" } }
```

**Most production APIs are Level 2.** Level 3 (HATEOAS) is rare in practice but good to know for interviews.

---

## 2. HTTP Methods & Status Codes

### HTTP Methods (CRUD Mapping)

| Method | CRUD | Idempotent? | Safe? | Request Body |
|--------|------|-------------|-------|-------------|
| GET | Read | ✅ Yes | ✅ Yes | No |
| POST | Create | ❌ No | ❌ No | Yes |
| PUT | Update (full) | ✅ Yes | ❌ No | Yes |
| PATCH | Update (partial) | ❌ No | ❌ No | Yes |
| DELETE | Delete | ✅ Yes | ❌ No | Optional |

**Idempotent** = calling multiple times produces the same result.  
**Safe** = doesn't modify the resource.

### Status Codes (Must Know)

| Code | Meaning | When to Use |
|------|---------|-------------|
| **200** | OK | Successful GET, PUT |
| **201** | Created | Successful POST (resource created) |
| **204** | No Content | Successful DELETE (no body) |
| **400** | Bad Request | Validation error, malformed request |
| **401** | Unauthorized | Not authenticated (missing/invalid token) |
| **403** | Forbidden | Authenticated but not authorized |
| **404** | Not Found | Resource doesn't exist |
| **405** | Method Not Allowed | Wrong HTTP method |
| **409** | Conflict | Duplicate resource, version conflict |
| **422** | Unprocessable Entity | Semantic validation error |
| **429** | Too Many Requests | Rate limiting |
| **500** | Internal Server Error | Unexpected server error |
| **502** | Bad Gateway | Upstream service error |
| **503** | Service Unavailable | Server overloaded/maintenance |

---

## 3. Spring Boot REST Controller

### Complete CRUD Example

```java
@RestController
@RequestMapping("/api/v1/employees")
@Validated
public class EmployeeController {

    private final EmployeeService employeeService;

    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    // GET all with pagination
    @GetMapping
    public ResponseEntity<Page<EmployeeDTO>> getAll(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy) {
        Page<EmployeeDTO> employees = employeeService.getAll(page, size, sortBy);
        return ResponseEntity.ok(employees);
    }

    // GET by ID
    @GetMapping("/{id}")
    public ResponseEntity<EmployeeDTO> getById(@PathVariable Long id) {
        EmployeeDTO employee = employeeService.getById(id);
        return ResponseEntity.ok(employee);
    }

    // POST — Create
    @PostMapping
    public ResponseEntity<EmployeeDTO> create(@Valid @RequestBody CreateEmployeeRequest request) {
        EmployeeDTO created = employeeService.create(request);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(created.getId())
                .toUri();
        return ResponseEntity.created(location).body(created); // 201 + Location header
    }

    // PUT — Full Update
    @PutMapping("/{id}")
    public ResponseEntity<EmployeeDTO> update(
            @PathVariable Long id,
            @Valid @RequestBody UpdateEmployeeRequest request) {
        EmployeeDTO updated = employeeService.update(id, request);
        return ResponseEntity.ok(updated);
    }

    // PATCH — Partial Update
    @PatchMapping("/{id}")
    public ResponseEntity<EmployeeDTO> partialUpdate(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {
        EmployeeDTO updated = employeeService.partialUpdate(id, updates);
        return ResponseEntity.ok(updated);
    }

    // DELETE
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        employeeService.delete(id);
        return ResponseEntity.noContent().build(); // 204
    }

    // GET — Search/Filter
    @GetMapping("/search")
    public ResponseEntity<List<EmployeeDTO>> search(
            @RequestParam(required = false) String department,
            @RequestParam(required = false) Double minSalary,
            @RequestParam(required = false) String name) {
        List<EmployeeDTO> results = employeeService.search(department, minSalary, name);
        return ResponseEntity.ok(results);
    }
}
```

### DTO Pattern

```java
// Request DTO
public record CreateEmployeeRequest(
    @NotBlank(message = "Name is required")
    String name,

    @Email(message = "Invalid email")
    String email,

    @NotNull
    @Min(value = 0, message = "Salary must be positive")
    Double salary,

    @NotBlank
    String department
) {}

// Response DTO
public record EmployeeDTO(
    Long id,
    String name,
    String email,
    Double salary,
    String department,
    LocalDateTime createdAt
) {}

// ⚠️ NEVER expose Entity directly in API response
// Always use DTOs to control what data goes to the client
```

### ResponseEntity Deep Dive

```java
// ResponseEntity gives full control: status + headers + body

// 200 OK
return ResponseEntity.ok(body);
return ResponseEntity.ok().build();                     // no body

// 201 Created with Location header
URI location = URI.create("/api/users/" + user.getId());
return ResponseEntity.created(location).body(user);

// 204 No Content
return ResponseEntity.noContent().build();

// 400 Bad Request
return ResponseEntity.badRequest().body(errorResponse);

// 404 Not Found
return ResponseEntity.notFound().build();

// Custom status + custom headers
return ResponseEntity
    .status(HttpStatus.ACCEPTED)                       // 202
    .header("X-Request-Id", requestId)
    .header("Cache-Control", "max-age=3600")
    .body(response);

// Conditional return
return employeeService.findById(id)
    .map(ResponseEntity::ok)                            // 200 if found
    .orElse(ResponseEntity.notFound().build());          // 404 if not
```

---

## 4. Request Annotations Compared

```java
@GetMapping("/api/orders/{orderId}")
public ResponseEntity<Order> getOrder(
    @PathVariable Long orderId,                         // from URL path
    @RequestParam(defaultValue = "USD") String currency, // from query string
    @RequestHeader("Authorization") String token,        // from HTTP header
    @CookieValue("sessionId") String session             // from cookie
) { ... }

@PostMapping("/api/orders")
public ResponseEntity<Order> create(
    @Valid @RequestBody CreateOrderRequest request       // from JSON body
) { ... }
```

| Annotation | Source | Example URL/Header | Use Case |
|------------|--------|-------------------|----------|
| `@PathVariable` | URL path | `/users/{id}` | Resource identification |
| `@RequestParam` | Query string | `/users?name=John&age=30` | Filtering, sorting, pagination |
| `@RequestBody` | HTTP body (JSON) | `{"name": "John"}` | Create/Update payloads |
| `@RequestHeader` | HTTP header | `Authorization: Bearer ...` | Auth tokens, API keys, metadata |
| `@CookieValue` | Cookie | `Cookie: sessionId=abc` | Session data |
| `@MatrixVariable` | Matrix params | `/users;age=30;city=NYC` | Rarely used, filtering |

---

## 5. Request Validation

```java
// Add dependency: spring-boot-starter-validation

// DTO with validation
public class CreateUserRequest {
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100)
    private String name;

    @Email(message = "Invalid email format")
    @NotBlank
    private String email;

    @NotNull
    @Min(18) @Max(120)
    private Integer age;

    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "Invalid phone number")
    private String phone;

    @Past(message = "Birth date must be in the past")
    private LocalDate birthDate;
}

// Custom Validator
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueEmailValidator.class)
public @interface UniqueEmail {
    String message() default "Email already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {
    @Autowired
    private UserRepository userRepository;

    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        return !userRepository.existsByEmail(email);
    }
}

// Validation error handling
@RestControllerAdvice
public class ValidationExceptionHandler {
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(err ->
            errors.put(err.getField(), err.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errors);
    }
}
```

---

## 6. Pagination & Sorting

```java
// Spring Data JPA provides built-in pagination
@GetMapping
public ResponseEntity<Page<EmployeeDTO>> getEmployees(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id,asc") String[] sort) {

    Pageable pageable = PageRequest.of(page, size, Sort.by(
        Sort.Order.asc("department"),
        Sort.Order.desc("salary")
    ));

    Page<EmployeeDTO> result = employeeService.findAll(pageable);
    return ResponseEntity.ok(result);
}

// Response includes:
// {
//   "content": [...],
//   "totalElements": 100,
//   "totalPages": 10,
//   "size": 10,
//   "number": 0,        // current page
//   "first": true,
//   "last": false
// }
```

---

## 7. API Versioning

```java
// Strategy 1: URI Versioning (Most Common)
@RequestMapping("/api/v1/users")
@RequestMapping("/api/v2/users")

// Strategy 2: Header Versioning
@GetMapping(value = "/api/users", headers = "X-API-VERSION=1")
@GetMapping(value = "/api/users", headers = "X-API-VERSION=2")

// Strategy 3: Media Type Versioning (Content Negotiation)
@GetMapping(value = "/api/users", produces = "application/vnd.company.v1+json")

// Strategy 4: Request Parameter
@GetMapping(value = "/api/users", params = "version=1")
```

| Strategy | Pros | Cons |
|----------|------|------|
| URI | Simple, visible, cacheable | URL changes, not pure REST |
| Header | Clean URLs | Hidden, harder to test |
| Media Type | Most RESTful | Complex, hard to test |
| Parameter | Easy to switch | Messy with many params |

**Recommendation:** URI versioning — most widely adopted, easiest to understand.

---

## 8. File Upload & Download

```java
@RestController
@RequestMapping("/api/v1/files")
public class FileController {

    // Upload
    @PostMapping("/upload")
    public ResponseEntity<String> upload(@RequestParam("file") MultipartFile file) {
        if (file.isEmpty()) {
            return ResponseEntity.badRequest().body("File is empty");
        }

        // Validate file type
        String contentType = file.getContentType();
        if (!List.of("image/png", "image/jpeg", "application/pdf").contains(contentType)) {
            return ResponseEntity.badRequest().body("Unsupported file type");
        }

        // Validate size (also configure in properties)
        // spring.servlet.multipart.max-file-size=10MB
        // spring.servlet.multipart.max-request-size=10MB

        String fileName = fileService.store(file);
        URI location = URI.create("/api/v1/files/" + fileName);
        return ResponseEntity.created(location).body("Uploaded: " + fileName);
    }

    // Download
    @GetMapping("/download/{fileName}")
    public ResponseEntity<Resource> download(@PathVariable String fileName) {
        Resource resource = fileService.loadAsResource(fileName);
        return ResponseEntity.ok()
            .contentType(MediaType.APPLICATION_OCTET_STREAM)
            .header(HttpHeaders.CONTENT_DISPOSITION,
                    "attachment; filename=\"" + resource.getFilename() + "\"")
            .body(resource);
    }
}
```

---

## 9. Swagger / OpenAPI

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
<!-- Access: http://localhost:8080/swagger-ui.html -->
```

```java
@RestController
@RequestMapping("/api/v1/users")
@Tag(name = "User Management", description = "CRUD operations for users")
public class UserController {

    @Operation(summary = "Get user by ID", description = "Returns a single user")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "User found"),
        @ApiResponse(responseCode = "404", description = "User not found")
    })
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getById(
            @Parameter(description = "User ID", example = "1")
            @PathVariable Long id) {
        return ResponseEntity.ok(userService.getById(id));
    }
}
```

```yaml
# application.yml — customize Swagger
springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
    operations-sorter: method
```

---

## 10. HATEOAS

```java
// Hypermedia As The Engine Of Application State
// API responses include links to related resources

@GetMapping("/{id}")
public EntityModel<EmployeeDTO> getById(@PathVariable Long id) {
    EmployeeDTO employee = service.getById(id);

    return EntityModel.of(employee,
        linkTo(methodOn(EmployeeController.class).getById(id)).withSelfRel(),
        linkTo(methodOn(EmployeeController.class).getAll(0, 10, "id")).withRel("all-employees"),
        linkTo(methodOn(OrderController.class).getByEmployee(id)).withRel("orders")
    );
}

// Response:
// {
//   "id": 1,
//   "name": "John",
//   "_links": {
//     "self": { "href": "/api/v1/employees/1" },
//     "all-employees": { "href": "/api/v1/employees" },
//     "orders": { "href": "/api/v1/employees/1/orders" }
//   }
// }
```

---

## 11. ETags & Caching

```java
// ETag = hash/version of resource — avoids sending data if unchanged

// Server returns ETag in response
@GetMapping("/{id}")
public ResponseEntity<EmployeeDTO> getById(@PathVariable Long id) {
    EmployeeDTO employee = service.getById(id);
    String etag = String.valueOf(employee.hashCode()); // or version field

    return ResponseEntity.ok()
        .eTag(etag)
        .cacheControl(CacheControl.maxAge(1, TimeUnit.HOURS))
        .body(employee);
}

// Client sends: If-None-Match: "abc123"
// Server checks: if ETag matches → 304 Not Modified (no body sent)
// If changed → 200 OK with new data and new ETag
```

**How it works:**
```
1. Client: GET /api/users/1
2. Server: 200 OK, ETag: "v1", body: {...}
3. Client: GET /api/users/1, If-None-Match: "v1"
4. Server: 304 Not Modified (no body — saves bandwidth!)
   OR: 200 OK, ETag: "v2", body: {...} (data changed)
```

| Header | Direction | Purpose |
|--------|-----------|--------|
| `ETag` | Response | Current version hash |
| `If-None-Match` | Request | "Give me data only if changed" (GET) |
| `If-Match` | Request | "Update only if still this version" (PUT — optimistic locking) |
| `Cache-Control` | Response | How long client can cache (`max-age`, `no-cache`, `no-store`) |

---

## 12. REST Best Practices

### ✅ DO
1. Use **nouns** for resources, not verbs
2. Use **plural** names (`/users` not `/user`)
3. Use proper **HTTP status codes**
4. Use **DTOs** — never expose entities
5. **Version** your API from day one
6. Return **Location** header on POST (201)
7. Use **pagination** for list endpoints
8. Validate **all input** with Bean Validation
9. Use **consistent error response format**
10. Document with **Swagger/OpenAPI**

### ❌ DON'T
1. Don't use verbs in URLs (`/getUser`, `/createUser`)
2. Don't return raw entities
3. Don't use 200 for everything
4. Don't expose internal errors to client
5. Don't mix singular/plural (`/user/1/orders`)
6. Don't ignore idempotency

### Standard Error Response Format
```java
public record ApiError(
    String status,
    int code,
    String message,
    String path,
    LocalDateTime timestamp,
    List<FieldError> errors  // for validation errors
) {
    public record FieldError(String field, String message) {}
}
```

---

## 13. REST vs SOAP vs GraphQL

| Feature | REST | SOAP | GraphQL |
|---------|------|------|---------|
| Protocol | HTTP | Any (usually HTTP) | HTTP |
| Format | JSON (usually) | XML only | JSON |
| Contract | OpenAPI/Swagger | WSDL | Schema |
| Over-fetching | ⚠️ Possible | ⚠️ Possible | ✅ No (client picks fields) |
| Under-fetching | ⚠️ Multiple calls needed | ⚠️ Yes | ✅ Single query |
| Caching | ✅ Easy (HTTP caching) | ❌ Difficult | ⚠️ Complex |
| Learning curve | Low | High | Medium |
| Best for | Simple CRUD APIs | Enterprise, WS-Security | Complex nested data |

---

## 14. Interview Questions & Answers

### Q1: What is idempotency? Which HTTP methods are idempotent?
**A:** Idempotent = calling N times produces the same result as calling once. **GET, PUT, DELETE** are idempotent. **POST, PATCH** are NOT. Example: PUT `/users/1` with `{name: "John"}` always results in the same state. POST `/users` creates a new user each time.

### Q2: PUT vs PATCH — what's the difference?
**A:** **PUT** replaces the entire resource (all fields required). **PATCH** updates only the specified fields (partial update). PUT is idempotent; PATCH is not necessarily. Use PUT when client sends the complete updated resource; PATCH when updating specific fields.

### Q3: What is content negotiation?
**A:** The client and server agree on the response format. Client sends `Accept: application/json` header. Server responds in that format or returns `406 Not Acceptable`. Spring Boot handles this automatically with `produces` attribute.

### Q4: How do you secure a REST API?
**A:** 
1. **Authentication:** JWT tokens, OAuth 2.0
2. **Authorization:** Role-based (RBAC), Spring Security
3. **HTTPS:** Always use TLS
4. **Input validation:** Prevent injection
5. **Rate limiting:** Prevent abuse
6. **CORS:** Control cross-origin access
7. **API keys:** For service-to-service

### Q5: What is CORS and how do you handle it in Spring Boot?
**A:** Cross-Origin Resource Sharing — browsers block requests to different domains. Configure in Spring:
```java
@CrossOrigin(origins = "http://localhost:3000") // per controller
// or globally:
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("http://localhost:3000")
                .allowedMethods("GET", "POST", "PUT", "DELETE");
    }
}
```

---

### Q6: What is the difference between 401 and 403?
**A:**
- **401 Unauthorized** — Client is NOT authenticated (no token, expired token, invalid credentials). Fix: log in / provide valid token.
- **403 Forbidden** — Client IS authenticated but NOT authorized for this resource. Fix: need higher role/permissions.

**Analogy:** 401 = "Who are you?" | 403 = "I know who you are, but you can't do this."

---

### Q7: How to handle file uploads in REST?
**A:** Use `multipart/form-data` content type with `@RequestParam("file") MultipartFile file`. Always validate:
1. File size (configure `spring.servlet.multipart.max-file-size`)
2. File type (check `contentType`, never trust file extension)
3. File name (sanitize to prevent path traversal attacks)
4. Return 201 with Location header pointing to the uploaded file

---

### Q8: What is ETag and how does it help?
**A:** ETag (Entity Tag) is a hash/version of a resource sent in the response header. On subsequent requests, the client sends `If-None-Match: <etag>`. If the resource hasn't changed, the server responds with **304 Not Modified** (no body) — saving bandwidth and improving performance. For PUT/PATCH, `If-Match` prevents concurrent update conflicts (optimistic locking).

---

### Q9: How do you document REST APIs?
**A:** Use **OpenAPI 3.0 (Swagger)** with `springdoc-openapi`. Add the dependency, annotate controllers with `@Operation`, `@ApiResponse`, `@Parameter`, `@Tag`. Swagger UI auto-generates interactive documentation at `/swagger-ui.html`. API spec available at `/api-docs` (JSON). Can also export to Postman collections.

---

### Q10: How to handle long-running operations in REST?
**A:** REST is synchronous by design. For long operations:
1. **Async pattern:** POST returns `202 Accepted` with a status URL
2. Client polls `GET /api/jobs/{jobId}` for progress
3. When done, response includes the result or a link to it

```
POST /api/reports → 202 Accepted, Location: /api/jobs/abc123
GET  /api/jobs/abc123 → {"status": "PROCESSING", "progress": 60}
GET  /api/jobs/abc123 → {"status": "COMPLETED", "result": "/api/reports/456"}
```

---

### Q11: `@RequestBody` vs `@RequestParam` vs `@PathVariable`?
**A:**
| | Source | Example |
|---|---|---|
| `@PathVariable` | URL path segment | `GET /users/123` → `id = 123` |
| `@RequestParam` | Query string | `GET /users?name=John` → `name = "John"` |
| `@RequestBody` | HTTP request body (JSON) | `POST /users` with `{"name": "John"}` |

`@PathVariable` for identifying resources. `@RequestParam` for filtering/sorting. `@RequestBody` for create/update payloads.

---

### Q12: How to implement rate limiting?
**A:** Options:
1. **API Gateway level:** AWS API Gateway, Kong, Nginx (best for microservices)
2. **Spring Boot filter:** Custom `OncePerRequestFilter` with in-memory counter or Redis
3. **Bucket4j library:** Token bucket algorithm, Spring Boot integration
4. **Resilience4j RateLimiter:** Annotation-based `@RateLimiter`

Return `429 Too Many Requests` with `Retry-After` header when limit is exceeded.

---

### Q13: What is the Richardson Maturity Model?
**A:** A model that grades REST APIs from Level 0 to Level 3:
- **Level 0:** Single URI, single HTTP method (RPC-style)
- **Level 1:** Multiple URIs (resources), but single method
- **Level 2:** Multiple URIs + proper HTTP methods + status codes (★ most APIs)
- **Level 3:** HATEOAS — responses include links to related resources ("Glory of REST")

---

### Q14: How to handle API deprecation?
**A:**
1. Add `Deprecation` and `Sunset` headers to responses
2. Document in Swagger with `@Deprecated` and `@Operation(deprecated = true)`
3. Log warnings when deprecated endpoints are called
4. Support both old and new versions simultaneously during migration period
5. Communicate timeline to consumers (changelogs, emails)
6. Eventually return `410 Gone` after sunset date

---

> **Next:** [07_Hibernate_JPA.md](07_Hibernate_JPA.md) →
