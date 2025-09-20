# 📅 Day 11: Exception Handling

## 🎯 Today Topic
- Master `@ExceptionHandler` for method-level error handling
- Understand `@ControllerAdvice` for global exception management
- Create custom exceptions and error responses
- Handle validation errors and HTTP status codes
- Best practices for production-ready error handling

---

## ⚠️ Why Exception Handling Matters

### Without Proper Exception Handling:
```json
// 😱 Default Spring Boot error (not user-friendly)
{
  "timestamp": "2024-01-15T10:30:00.000+00:00",
  "status": 500,
  "error": "Internal Server Error",
  "path": "/api/users/999"
}
```

### With Proper Exception Handling:
```json
// 😊 User-friendly error response
{
  "success": false,
  "message": "User not found with ID: 999",
  "errorCode": "USER_NOT_FOUND",
  "timestamp": "2024-01-15T10:30:00",
  "path": "/api/users/999"
}
```

---

## 🔧 @ExceptionHandler - Method Level

### Basic Exception Handling:
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id); // May throw UserNotFoundException
    }
    
    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.save(user); // May throw ValidationException
    }
    
    // Handle specific exception in this controller
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        System.out.println("❌ User not found: " + ex.getMessage());
        
        ErrorResponse error = new ErrorResponse(
            false,
            ex.getMessage(),
            "USER_NOT_FOUND",
            LocalDateTime.now()
        );
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    // Handle validation errors
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        System.out.println("❌ Validation failed: " + ex.getMessage());
        
        ErrorResponse error = new ErrorResponse(
            false,
            "Invalid input: " + ex.getMessage(),
            "VALIDATION_ERROR",
            LocalDateTime.now()
        );
        
        return ResponseEntity.badRequest().body(error);
    }
    
    // Handle multiple exception types
    @ExceptionHandler({IllegalArgumentException.class, IllegalStateException.class})
    public ResponseEntity<ErrorResponse> handleBadRequest(Exception ex) {
        System.out.println("❌ Bad request: " + ex.getMessage());
        
        ErrorResponse error = new ErrorResponse(
            false,
            ex.getMessage(),
            "BAD_REQUEST",
            LocalDateTime.now()
        );
        
        return ResponseEntity.badRequest().body(error);
    }
}
```

---

## 🌍 @ControllerAdvice - Global Exception Handling

### Global Exception Handler:
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    // Handle user not found across all controllers
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(
            UserNotFoundException ex, 
            HttpServletRequest request) {
        
        System.out.println("🌍 Global: User not found - " + ex.getMessage());
        
        ErrorResponse error = ErrorResponse.builder()
            .success(false)
            .message(ex.getMessage())
            .errorCode("USER_NOT_FOUND")
            .timestamp(LocalDateTime.now())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    // Handle validation errors globally
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            ValidationException ex,
            HttpServletRequest request) {
        
        System.out.println("🌍 Global: Validation error - " + ex.getMessage());
        
        ErrorResponse error = ErrorResponse.builder()
            .success(false)
            .message("Validation failed: " + ex.getMessage())
            .errorCode("VALIDATION_ERROR")
            .timestamp(LocalDateTime.now())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.badRequest().body(error);
    }
    
    // Handle all other exceptions
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(
            Exception ex,
            HttpServletRequest request) {
        
        System.out.println("🌍 Global: Unexpected error - " + ex.getMessage());
        ex.printStackTrace(); // Log full stack trace
        
        ErrorResponse error = ErrorResponse.builder()
            .success(false)
            .message("An unexpected error occurred")
            .errorCode("INTERNAL_ERROR")
            .timestamp(LocalDateTime.now())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
    
    // Handle method argument validation
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleMethodArgumentNotValid(
            MethodArgumentNotValidException ex,
            HttpServletRequest request) {
        
        System.out.println("🌍 Global: Method argument validation failed");
        
        // Extract field errors
        List<String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.toList());
        
        ErrorResponse error = ErrorResponse.builder()
            .success(false)
            .message("Validation failed")
            .errorCode("VALIDATION_ERROR")
            .details(errors)
            .timestamp(LocalDateTime.now())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.badRequest().body(error);
    }
    
    // Handle access denied
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(
            AccessDeniedException ex,
            HttpServletRequest request) {
        
        System.out.println("🌍 Global: Access denied - " + ex.getMessage());
        
        ErrorResponse error = ErrorResponse.builder()
            .success(false)
            .message("Access denied")
            .errorCode("ACCESS_DENIED")
            .timestamp(LocalDateTime.now())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(error);
    }
}
```

---

## 📝 Custom Exceptions

### Business Logic Exceptions:
```java
// Base exception class
public class BusinessException extends RuntimeException {
    private final String errorCode;
    
    public BusinessException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
    }
    
    public String getErrorCode() {
        return errorCode;
    }
}

// User-related exceptions
public class UserNotFoundException extends BusinessException {
    public UserNotFoundException(Long userId) {
        super("User not found with ID: " + userId, "USER_NOT_FOUND");
    }
    
    public UserNotFoundException(String username) {
        super("User not found with username: " + username, "USER_NOT_FOUND");
    }
}

public class UserAlreadyExistsException extends BusinessException {
    public UserAlreadyExistsException(String email) {
        super("User already exists with email: " + email, "USER_ALREADY_EXISTS");
    }
}

// Validation exception
public class ValidationException extends BusinessException {
    public ValidationException(String message) {
        super(message, "VALIDATION_ERROR");
    }
}

// Order-related exceptions
public class OrderNotFoundException extends BusinessException {
    public OrderNotFoundException(Long orderId) {
        super("Order not found with ID: " + orderId, "ORDER_NOT_FOUND");
    }
}

public class InsufficientStockException extends BusinessException {
    public InsufficientStockException(String productName) {
        super("Insufficient stock for product: " + productName, "INSUFFICIENT_STOCK");
    }
}
```

---

## 📋 Error Response Structure

### Standard Error Response Class:
```java
public class ErrorResponse {
    private boolean success;
    private String message;
    private String errorCode;
    private List<String> details;
    private LocalDateTime timestamp;
    private String path;
    
    // Constructors
    public ErrorResponse() {
        this.timestamp = LocalDateTime.now();
    }
    
    public ErrorResponse(boolean success, String message, String errorCode, LocalDateTime timestamp) {
        this.success = success;
        this.message = message;
        this.errorCode = errorCode;
        this.timestamp = timestamp;
    }
    
    // Builder pattern
    public static ErrorResponseBuilder builder() {
        return new ErrorResponseBuilder();
    }
    
    public static class ErrorResponseBuilder {
        private ErrorResponse response = new ErrorResponse();
        
        public ErrorResponseBuilder success(boolean success) {
            response.success = success;
            return this;
        }
        
        public ErrorResponseBuilder message(String message) {
            response.message = message;
            return this;
        }
        
        public ErrorResponseBuilder errorCode(String errorCode) {
            response.errorCode = errorCode;
            return this;
        }
        
        public ErrorResponseBuilder details(List<String> details) {
            response.details = details;
            return this;
        }
        
        public ErrorResponseBuilder timestamp(LocalDateTime timestamp) {
            response.timestamp = timestamp;
            return this;
        }
        
        public ErrorResponseBuilder path(String path) {
            response.path = path;
            return this;
        }
        
        public ErrorResponse build() {
            return response;
        }
    }
    
    // Getters and setters
    public boolean isSuccess() { return success; }
    public void setSuccess(boolean success) { this.success = success; }
    
    public String getMessage() { return message; }
    public void setMessage(String message) { this.message = message; }
    
    public String getErrorCode() { return errorCode; }
    public void setErrorCode(String errorCode) { this.errorCode = errorCode; }
    
    public List<String> getDetails() { return details; }
    public void setDetails(List<String> details) { this.details = details; }
    
    public LocalDateTime getTimestamp() { return timestamp; }
    public void setTimestamp(LocalDateTime timestamp) { this.timestamp = timestamp; }
    
    public String getPath() { return path; }
    public void setPath(String path) { this.path = path; }
}
```

---

## 🎯 Complete Example: E-Commerce API

### Service with Business Logic:
```java
@Service
public class ProductService {
    
    private final Map<Long, Product> products = new ConcurrentHashMap<>();
    private final AtomicLong idGenerator = new AtomicLong(1);
    
    public ProductService() {
        // Initialize sample data
        products.put(1L, new Product(1L, "Laptop", 999.99, 10));
        products.put(2L, new Product(2L, "Phone", 599.99, 0));
        idGenerator.set(3L);
    }
    
    public Product findById(Long id) {
        Product product = products.get(id);
        if (product == null) {
            throw new ProductNotFoundException(id);
        }
        return product;
    }
    
    public Product save(Product product) {
        // Validation
        if (product.getName() == null || product.getName().trim().isEmpty()) {
            throw new ValidationException("Product name is required");
        }
        
        if (product.getPrice() == null || product.getPrice() <= 0) {
            throw new ValidationException("Product price must be positive");
        }
        
        // Save product
        if (product.getId() == null) {
            product.setId(idGenerator.getAndIncrement());
        }
        
        products.put(product.getId(), product);
        return product;
    }
    
    public void deleteById(Long id) {
        if (!products.containsKey(id)) {
            throw new ProductNotFoundException(id);
        }
        products.remove(id);
    }
    
    public Product updateStock(Long id, Integer quantity) {
        Product product = findById(id);
        
        if (quantity < 0) {
            throw new ValidationException("Stock quantity cannot be negative");
        }
        
        product.setStock(quantity);
        return product;
    }
    
    public void reserveStock(Long id, Integer quantity) {
        Product product = findById(id);
        
        if (product.getStock() < quantity) {
            throw new InsufficientStockException(product.getName());
        }
        
        product.setStock(product.getStock() - quantity);
    }
}
```

### Controller with Exception Handling:
```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    private final ProductService productService;
    
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        try {
            Product product = productService.findById(id);
            return ResponseEntity.ok(product);
        } catch (ProductNotFoundException ex) {
            // This will be handled by @ControllerAdvice
            throw ex;
        }
    }
    
    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        Product savedProduct = productService.save(product);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedProduct);
    }
    
    @PutMapping("/{id}/stock")
    public ResponseEntity<Product> updateStock(
            @PathVariable Long id,
            @RequestBody Map<String, Integer> request) {
        
        Integer quantity = request.get("quantity");
        if (quantity == null) {
            throw new ValidationException("Quantity is required");
        }
        
        Product product = productService.updateStock(id, quantity);
        return ResponseEntity.ok(product);
    }
    
    @PostMapping("/{id}/reserve")
    public ResponseEntity<String> reserveStock(
            @PathVariable Long id,
            @RequestBody Map<String, Integer> request) {
        
        Integer quantity = request.get("quantity");
        if (quantity == null || quantity <= 0) {
            throw new ValidationException("Valid quantity is required");
        }
        
        productService.reserveStock(id, quantity);
        return ResponseEntity.ok("Stock reserved successfully");
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<String> deleteProduct(@PathVariable Long id) {
        productService.deleteById(id);
        return ResponseEntity.ok("Product deleted successfully");
    }
}
```

---

## 🧪 Testing Exception Handling

### Test Examples:
```bash
# Test product not found (404)
curl -X GET http://localhost:8080/api/products/999

# Test validation error (400)
curl -X POST http://localhost:8080/api/products \
  -H "Content-Type: application/json" \
  -d '{"name": "", "price": -10}'

# Test insufficient stock (400)
curl -X POST http://localhost:8080/api/products/2/reserve \
  -H "Content-Type: application/json" \
  -d '{"quantity": 5}'

# Test invalid stock update (400)
curl -X PUT http://localhost:8080/api/products/1/stock \
  -H "Content-Type: application/json" \
  -d '{"quantity": -5}'
```

**Expected Responses:**
```json
// Product not found
{
  "success": false,
  "message": "Product not found with ID: 999",
  "errorCode": "PRODUCT_NOT_FOUND",
  "timestamp": "2024-01-15T10:30:00",
  "path": "/api/products/999"
}

// Validation error
{
  "success": false,
  "message": "Validation failed: Product name is required",
  "errorCode": "VALIDATION_ERROR",
  "timestamp": "2024-01-15T10:30:00",
  "path": "/api/products"
}
```

---

## 📚 Key Exception Types & Status Codes

| Exception Type | HTTP Status | Use Case |
|---------------|-------------|----------|
| `UserNotFoundException` | 404 NOT_FOUND | Resource doesn't exist |
| `ValidationException` | 400 BAD_REQUEST | Invalid input data |
| `AccessDeniedException` | 403 FORBIDDEN | Permission denied |
| `DuplicateException` | 409 CONFLICT | Resource already exists |
| `InsufficientStockException` | 400 BAD_REQUEST | Business rule violation |
| `Exception` (general) | 500 INTERNAL_ERROR | Unexpected errors |

---

## 🎯 Interview Questions & Answers

### Q1: "What's the difference between @ExceptionHandler and @ControllerAdvice?"
**Answer:**
- **@ExceptionHandler**: Handles exceptions only in the same controller class
- **@ControllerAdvice**: Handles exceptions globally across all controllers
- **Use @ExceptionHandler for controller-specific errors, @ControllerAdvice for global handling**

### Q2: "How do you create consistent error responses?"
**Answer:**
- Create a standard ErrorResponse class with fields like success, message, errorCode, timestamp
- Use @ControllerAdvice for global exception handling
- Map different exceptions to appropriate HTTP status codes
- Include helpful information like error codes and timestamps

### Q3: "What are best practices for exception handling in REST APIs?"
**Answer:**
- **Use appropriate HTTP status codes** (404, 400, 500, etc.)
- **Provide meaningful error messages** for clients
- **Don't expose sensitive information** in error responses
- **Log detailed errors** server-side for debugging
- **Use consistent error response format** across all endpoints

---

## 🚀 Best Practices

### ✅ Do's:
```java
// 1. Use specific exception types
throw new UserNotFoundException(userId);

// 2. Provide helpful error messages
throw new ValidationException("Email format is invalid");

// 3. Use appropriate HTTP status codes
return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);

// 4. Log errors for debugging
logger.error("Failed to process order: {}", orderId, ex);

// 5. Create consistent error response format
ErrorResponse.builder()
    .success(false)
    .message("User not found")
    .errorCode("USER_NOT_FOUND")
    .build();
```

### ❌ Don'ts:
```java
// 1. Don't expose stack traces to clients
// return ResponseEntity.ok(ex.getStackTrace()); // BAD!

// 2. Don't use generic exception messages
// throw new Exception("Error"); // BAD!

// 3. Don't ignore exceptions
// try { ... } catch (Exception e) { } // BAD!

// 4. Don't use wrong HTTP status codes
// return ResponseEntity.ok(error); // Should be 4xx or 5xx

// 5. Don't expose sensitive information
// throw new Exception("Database password: " + dbPassword); // BAD!
```

---
