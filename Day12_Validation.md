# 📅 Day 12: Validation

## 🎯 Today Topic
- Master `@Valid` annotation for request validation
- Use built-in validation annotations (`@NotNull`, `@Size`, `@Email`)
- Create custom validation annotations
- Handle validation errors properly
- Best practices for data validation

---

## ✅ Why Validation Matters

### Without Validation:
```java
// 😱 No validation - accepts any data
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    return userService.save(user); // What if name is null? Email invalid?
}
```

### With Validation:
```java
// 😊 Proper validation - safe data processing
@PostMapping("/users")
public User createUser(@Valid @RequestBody User user) {
    return userService.save(user); // Data is validated before processing
}
```

---

## 🔒 @Valid - Enable Validation

### Basic @Valid Usage:
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        User savedUser = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedUser);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(
            @PathVariable Long id, 
            @Valid @RequestBody User user) {
        User updatedUser = userService.update(id, user);
        return ResponseEntity.ok(updatedUser);
    }
}
```

---

## 📋 Built-in Validation Annotations

### User Model with Validations:
```java
public class User {
    
    @NotNull(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be between 2 and 50 characters")
    private String name;
    
    @NotBlank(message = "Email cannot be blank")
    @Email(message = "Email format is invalid")
    private String email;
    
    @NotNull(message = "Age is required")
    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 120, message = "Age must be less than 120")
    private Integer age;
    
    @Pattern(regexp = "^\\+?[1-9]\\d{9,14}$", message = "Invalid phone number format")
    private String phone;
    
    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;
    
    // Constructors, getters, setters
    public User() {}
    
    public User(String name, String email, Integer age) {
        this.name = name;
        this.email = email;
        this.age = age;
    }
    
    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public Integer getAge() { return age; }
    public void setAge(Integer age) { this.age = age; }
    
    public String getPhone() { return phone; }
    public void setPhone(String phone) { this.phone = phone; }
    
    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }
}
```

### Product Model Example:
```java
public class Product {
    
    @NotBlank(message = "Product name is required")
    @Size(max = 100, message = "Product name cannot exceed 100 characters")
    private String name;
    
    @NotNull(message = "Price is required")
    @DecimalMin(value = "0.01", message = "Price must be greater than 0")
    @DecimalMax(value = "99999.99", message = "Price cannot exceed 99999.99")
    private BigDecimal price;
    
    @Size(max = 500, message = "Description cannot exceed 500 characters")
    private String description;
    
    @NotBlank(message = "Category is required")
    private String category;
    
    @Min(value = 0, message = "Stock cannot be negative")
    private Integer stock;
    
    @Future(message = "Launch date must be in the future")
    private LocalDate launchDate;
    
    // Constructors, getters, setters
    public Product() {}
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public BigDecimal getPrice() { return price; }
    public void setPrice(BigDecimal price) { this.price = price; }
    
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    
    public String getCategory() { return category; }
    public void setCategory(String category) { this.category = category; }
    
    public Integer getStock() { return stock; }
    public void setStock(Integer stock) { this.stock = stock; }
    
    public LocalDate getLaunchDate() { return launchDate; }
    public void setLaunchDate(LocalDate launchDate) { this.launchDate = launchDate; }
}
```

---

## 🛠️ Common Validation Annotations

| Annotation | Purpose | Example |
|------------|---------|---------|
| `@NotNull` | Value cannot be null | `@NotNull private String name;` |
| `@NotBlank` | String cannot be null/empty/whitespace | `@NotBlank private String email;` |
| `@Size` | String/Collection size limits | `@Size(min=2, max=50) private String name;` |
| `@Min/@Max` | Numeric range validation | `@Min(18) @Max(65) private Integer age;` |
| `@Email` | Email format validation | `@Email private String email;` |
| `@Pattern` | Regex pattern matching | `@Pattern(regexp="^\\d{10}$") private String phone;` |
| `@Past/@Future` | Date validation | `@Past private LocalDate birthDate;` |
| `@DecimalMin/@DecimalMax` | Decimal range | `@DecimalMin("0.01") private BigDecimal price;` |

---

## 🎯 Complete Example: Order API

### Order Request DTO:
```java
public class OrderRequest {
    
    @NotNull(message = "Customer ID is required")
    private Long customerId;
    
    @NotEmpty(message = "Order must contain at least one item")
    @Valid // Validate each item in the list
    private List<OrderItem> items;
    
    @NotNull(message = "Shipping address is required")
    @Valid // Validate nested object
    private Address shippingAddress;
    
    @Size(max = 200, message = "Special instructions cannot exceed 200 characters")
    private String specialInstructions;
    
    // Constructors, getters, setters
    public OrderRequest() {}
    
    public Long getCustomerId() { return customerId; }
    public void setCustomerId(Long customerId) { this.customerId = customerId; }
    
    public List<OrderItem> getItems() { return items; }
    public void setItems(List<OrderItem> items) { this.items = items; }
    
    public Address getShippingAddress() { return shippingAddress; }
    public void setShippingAddress(Address shippingAddress) { this.shippingAddress = shippingAddress; }
    
    public String getSpecialInstructions() { return specialInstructions; }
    public void setSpecialInstructions(String specialInstructions) { this.specialInstructions = specialInstructions; }
}

public class OrderItem {
    
    @NotNull(message = "Product ID is required")
    private Long productId;
    
    @NotNull(message = "Quantity is required")
    @Min(value = 1, message = "Quantity must be at least 1")
    @Max(value = 99, message = "Quantity cannot exceed 99")
    private Integer quantity;
    
    // Constructors, getters, setters
    public OrderItem() {}
    
    public OrderItem(Long productId, Integer quantity) {
        this.productId = productId;
        this.quantity = quantity;
    }
    
    public Long getProductId() { return productId; }
    public void setProductId(Long productId) { this.productId = productId; }
    
    public Integer getQuantity() { return quantity; }
    public void setQuantity(Integer quantity) { this.quantity = quantity; }
}

public class Address {
    
    @NotBlank(message = "Street address is required")
    private String street;
    
    @NotBlank(message = "City is required")
    private String city;
    
    @NotBlank(message = "State is required")
    private String state;
    
    @NotBlank(message = "ZIP code is required")
    @Pattern(regexp = "^\\d{5}(-\\d{4})?$", message = "Invalid ZIP code format")
    private String zipCode;
    
    @NotBlank(message = "Country is required")
    private String country;
    
    // Constructors, getters, setters
    public Address() {}
    
    public String getStreet() { return street; }
    public void setStreet(String street) { this.street = street; }
    
    public String getCity() { return city; }
    public void setCity(String city) { this.city = city; }
    
    public String getState() { return state; }
    public void setState(String state) { this.state = state; }
    
    public String getZipCode() { return zipCode; }
    public void setZipCode(String zipCode) { this.zipCode = zipCode; }
    
    public String getCountry() { return country; }
    public void setCountry(String country) { this.country = country; }
}
```

### Controller with Validation:
```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    private final OrderService orderService;
    
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }
    
    @PostMapping
    public ResponseEntity<Order> createOrder(@Valid @RequestBody OrderRequest request) {
        System.out.println("✅ Creating order for customer: " + request.getCustomerId());
        System.out.println("📦 Items count: " + request.getItems().size());
        
        Order order = orderService.createOrder(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Order> updateOrder(
            @PathVariable Long id, 
            @Valid @RequestBody OrderRequest request) {
        System.out.println("🔄 Updating order: " + id);
        
        Order order = orderService.updateOrder(id, request);
        return ResponseEntity.ok(order);
    }
    
    @PatchMapping("/{id}/address")
    public ResponseEntity<Order> updateShippingAddress(
            @PathVariable Long id,
            @Valid @RequestBody Address address) {
        System.out.println("🏠 Updating shipping address for order: " + id);
        
        Order order = orderService.updateShippingAddress(id, address);
        return ResponseEntity.ok(order);
    }
}
```

---

## ⚠️ Handling Validation Errors

### Global Validation Error Handler:
```java
@ControllerAdvice
public class ValidationExceptionHandler {
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidationErrors(
            MethodArgumentNotValidException ex) {
        
        System.out.println("❌ Validation failed:");
        
        List<FieldError> fieldErrors = new ArrayList<>();
        
        ex.getBindingResult().getFieldErrors().forEach(error -> {
            FieldError fieldError = new FieldError(
                error.getField(),
                error.getDefaultMessage(),
                error.getRejectedValue()
            );
            fieldErrors.add(fieldError);
            
            System.out.printf("  Field '%s': %s (rejected value: %s)%n", 
                            error.getField(), error.getDefaultMessage(), error.getRejectedValue());
        });
        
        ValidationErrorResponse response = new ValidationErrorResponse(
            false,
            "Validation failed",
            fieldErrors,
            LocalDateTime.now()
        );
        
        return ResponseEntity.badRequest().body(response);
    }
    
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ValidationErrorResponse> handleConstraintViolation(
            ConstraintViolationException ex) {
        
        System.out.println("❌ Constraint violation:");
        
        List<FieldError> fieldErrors = new ArrayList<>();
        
        ex.getConstraintViolations().forEach(violation -> {
            String field = violation.getPropertyPath().toString();
            String message = violation.getMessage();
            Object value = violation.getInvalidValue();
            
            FieldError fieldError = new FieldError(field, message, value);
            fieldErrors.add(fieldError);
            
            System.out.printf("  Field '%s': %s (rejected value: %s)%n", field, message, value);
        });
        
        ValidationErrorResponse response = new ValidationErrorResponse(
            false,
            "Constraint violation",
            fieldErrors,
            LocalDateTime.now()
        );
        
        return ResponseEntity.badRequest().body(response);
    }
}

// Error Response Classes
public class ValidationErrorResponse {
    private boolean success;
    private String message;
    private List<FieldError> errors;
    private LocalDateTime timestamp;
    
    public ValidationErrorResponse(boolean success, String message, List<FieldError> errors, LocalDateTime timestamp) {
        this.success = success;
        this.message = message;
        this.errors = errors;
        this.timestamp = timestamp;
    }
    
    // Getters and setters
    public boolean isSuccess() { return success; }
    public void setSuccess(boolean success) { this.success = success; }
    
    public String getMessage() { return message; }
    public void setMessage(String message) { this.message = message; }
    
    public List<FieldError> getErrors() { return errors; }
    public void setErrors(List<FieldError> errors) { this.errors = errors; }
    
    public LocalDateTime getTimestamp() { return timestamp; }
    public void setTimestamp(LocalDateTime timestamp) { this.timestamp = timestamp; }
}

public class FieldError {
    private String field;
    private String message;
    private Object rejectedValue;
    
    public FieldError(String field, String message, Object rejectedValue) {
        this.field = field;
        this.message = message;
        this.rejectedValue = rejectedValue;
    }
    
    // Getters and setters
    public String getField() { return field; }
    public void setField(String field) { this.field = field; }
    
    public String getMessage() { return message; }
    public void setMessage(String message) { this.message = message; }
    
    public Object getRejectedValue() { return rejectedValue; }
    public void setRejectedValue(Object rejectedValue) { this.rejectedValue = rejectedValue; }
}
```

---

## 🎨 Custom Validation Annotations

### Custom Validation Example:
```java
// Custom annotation
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = ValidPasswordValidator.class)
@Documented
public @interface ValidPassword {
    String message() default "Password must contain at least one uppercase, one lowercase, one digit, and one special character";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Validator implementation
public class ValidPasswordValidator implements ConstraintValidator<ValidPassword, String> {
    
    private static final String PASSWORD_PATTERN = 
        "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=!])(?=\\S+$).{8,}$";
    
    private static final Pattern pattern = Pattern.compile(PASSWORD_PATTERN);
    
    @Override
    public void initialize(ValidPassword constraintAnnotation) {
        // Initialization logic if needed
    }
    
    @Override
    public boolean isValid(String password, ConstraintValidatorContext context) {
        if (password == null) {
            return false;
        }
        
        return pattern.matcher(password).matches();
    }
}

// Usage in model
public class User {
    @NotBlank(message = "Password is required")
    @ValidPassword
    private String password;
    
    // Other fields...
}
```

### Custom Validation for Business Rules:
```java
// Custom annotation for unique email
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueEmailValidator.class)
public @interface UniqueEmail {
    String message() default "Email already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Validator with service dependency
@Component
public class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {
    
    @Autowired
    private UserService userService;
    
    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        if (email == null) {
            return true; // Let @NotNull handle null check
        }
        
        return !userService.existsByEmail(email);
    }
}

// Usage
public class UserRegistrationRequest {
    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    @UniqueEmail
    private String email;
    
    // Other fields...
}
```

---

## 🧪 Testing Validation

### Test Examples:
```bash
# Valid request - should succeed
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "age": 25,
    "phone": "+1234567890",
    "password": "SecurePass123!"
  }'

# Invalid request - validation errors
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "",
    "email": "invalid-email",
    "age": 15,
    "phone": "abc",
    "password": "weak"
  }'
```

**Expected Error Response:**
```json
{
  "success": false,
  "message": "Validation failed",
  "errors": [
    {
      "field": "name",
      "message": "Name must be between 2 and 50 characters",
      "rejectedValue": ""
    },
    {
      "field": "email",
      "message": "Email format is invalid",
      "rejectedValue": "invalid-email"
    },
    {
      "field": "age",
      "message": "Age must be at least 18",
      "rejectedValue": 15
    }
  ],
  "timestamp": "2024-01-15T10:30:00"
}
```

---

## 🎯 Interview Questions & Answers

### Q1: "What's the difference between @NotNull, @NotEmpty, and @NotBlank?"
**Answer:**
- **@NotNull**: Value cannot be null
- **@NotEmpty**: Collection/String cannot be null or empty
- **@NotBlank**: String cannot be null, empty, or whitespace only

```java
@NotNull private String name;     // null → invalid
@NotEmpty private String email;   // null or "" → invalid  
@NotBlank private String city;    // null, "", or "   " → invalid
```

### Q2: "How do you validate nested objects?"
**Answer:**
**Use @Valid on nested objects and collections:**
```java
public class Order {
    @Valid
    private Address shippingAddress;  // Validates Address fields
    
    @NotEmpty
    @Valid
    private List<OrderItem> items;    // Validates each OrderItem
}
```

### Q3: "How do you create custom validation annotations?"
**Answer:**
1. Create annotation with @Constraint
2. Implement ConstraintValidator interface  
3. Use annotation on fields

```java
@Constraint(validatedBy = ValidPasswordValidator.class)
public @interface ValidPassword {
    String message() default "Invalid password";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

---

## 🚀 Best Practices

### ✅ Do's:
```java
// 1. Use @Valid with @RequestBody
@PostMapping("/users")
public User createUser(@Valid @RequestBody User user) { }

// 2. Provide meaningful error messages
@NotBlank(message = "Name is required")
@Size(min = 2, max = 50, message = "Name must be between 2 and 50 characters")
private String name;

// 3. Validate nested objects
@Valid
private Address address;

// 4. Handle validation errors globally
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ErrorResponse> handleValidation(...) { }

// 5. Use appropriate validation annotations
@Email private String email;
@Past private LocalDate birthDate;
@DecimalMin("0.01") private BigDecimal price;
```

### ❌ Don'ts:
```java
// 1. Don't forget @Valid
@PostMapping("/users")
public User createUser(@RequestBody User user) { } // BAD - no validation

// 2. Don't use generic error messages
@NotNull(message = "Invalid") // BAD - not helpful
@NotNull(message = "Name is required") // GOOD

// 3. Don't validate in controller
@PostMapping("/users") 
public User createUser(@RequestBody User user) {
    if (user.getName() == null) { ... } // BAD - validate in model
}
```

---

