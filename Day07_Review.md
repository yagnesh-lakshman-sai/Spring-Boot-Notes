# 📅 Day 7: Week 1 Review & Mini Project

## 🎯 Today's Objectives
- **Comprehensive Week 1 Review**: All concepts with quick examples
- **Mini Project**: E-Commerce system combining all annotations
- **Interview Cheat Sheet**: Key points for rapid review

## 📚 Week 1 Complete Summary

### Day 1: Spring Boot Fundamentals ✅
```java
// The foundation - everything starts here
@SpringBootApplication // = @Configuration + @EnableAutoConfiguration + @ComponentScan
public class ECommerceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ECommerceApplication.class, args);
    }
}
```
**Key Takeaway**: Spring Boot eliminates 80% of configuration through auto-configuration.

---

### Day 2: Dependency Injection & Stereotypes ✅
```java
@Controller  // Web layer - handles requests
@Service     // Business layer - contains logic  
@Repository  // Data layer - handles persistence
@Component   // Generic component

// Best practice: Constructor injection
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    
    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
}
```
**Key Takeaway**: Use stereotypes for semantic clarity and constructor injection for immutability.

---

### Day 3: Bean Configuration & Scopes ✅
```java
@Configuration
public class AppConfig {
    
    @Bean
    @Scope("singleton") // Default - one instance
    public PaymentService paymentService() {
        return new StripePaymentService();
    }
    
    @Bean
    @Scope("prototype") // New instance each time
    public ShoppingCart shoppingCart() {
        return new ShoppingCart();
    }
}
```
**Key Takeaway**: Use @Bean for third-party classes, Singleton for services, Prototype for stateful objects.

---

### Day 4: Autowiring & Dependency Resolution ✅
```java
@Service
public class CheckoutService {
    
    private final PaymentService primaryPayment;
    private final PaymentService backupPayment;
    private final List<ValidationService> validators;
    
    public CheckoutService(
            @Qualifier("stripe") PaymentService primaryPayment,
            @Qualifier("paypal") PaymentService backupPayment,
            List<ValidationService> validators) {
        // Constructor injection with qualifiers
    }
}
```
**Key Takeaway**: Use @Qualifier for precision, @Primary for defaults, Lists for multiple beans.

---

### Day 5: Application Properties ✅
```yaml
# application.yml
app:
  name: E-Commerce Platform
  timeout: 30
  features:
    recommendations: true
    reviews: false

payment:
  stripe:
    api-key: ${STRIPE_API_KEY}
    timeout: 10
```

```java
@ConfigurationProperties(prefix = "app")
@Component
public class AppProperties {
    private String name;
    private int timeout;
    private Features features = new Features();
    // Getters and setters...
}
```
**Key Takeaway**: Use @Value for simple properties, @ConfigurationProperties for complex structures.

---

### Day 6: Profiles & Environment Management ✅
```java
@Service
@Profile("dev")
public class MockPaymentService implements PaymentService {
    // Development implementation
}

@Service  
@Profile("prod")
public class StripePaymentService implements PaymentService {
    // Production implementation
}
```

```yaml
# application-prod.yml
spring:
  profiles:
    active: prod,monitoring,security
```
**Key Takeaway**: Use profiles for environment-specific beans and configurations.

---

## 🏗️ Mini Project: Complete E-Commerce System

Let's build a mini e-commerce system that demonstrates ALL Week 1 concepts:

### Project Structure:
```
src/main/java/com/ecommerce/
├── ECommerceApplication.java          # @SpringBootApplication
├── config/
│   ├── DatabaseConfig.java           # @Configuration, @Bean
│   └── PaymentConfig.java           # @Profile configurations
├── controller/
│   └── OrderController.java         # @RestController  
├── service/
│   ├── OrderService.java            # @Service, @Autowired
│   ├── PaymentService.java          # Interface
│   ├── MockPaymentService.java      # @Profile("dev")
│   └── StripePaymentService.java    # @Profile("prod")
├── repository/
│   └── OrderRepository.java         # @Repository
├── component/
│   └── EmailNotificationService.java # @Component
└── properties/
    └── AppProperties.java            # @ConfigurationProperties

src/main/resources/
├── application.yml
├── application-dev.yml
└── application-prod.yml
```

### Complete Implementation:

```java
// 1. Main Application Class
@SpringBootApplication
public class ECommerceApplication {
    
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(ECommerceApplication.class);
        
        // Set default profile if none specified
        app.setDefaultProperties(Collections.singletonMap(
            "spring.profiles.default", "dev"
        ));
        
        app.run(args);
    }
}

// 2. Configuration Properties
@ConfigurationProperties(prefix = "ecommerce")
@Component
public class ECommerceProperties {
    
    private String name;
    private String version;
    private int orderTimeout;
    private Features features = new Features();
    private Payment payment = new Payment();
    
    public static class Features {
        private boolean emailNotifications;
        private boolean orderTracking;
        private boolean recommendations;
        
        // Getters and setters
        public boolean isEmailNotifications() { return emailNotifications; }
        public void setEmailNotifications(boolean emailNotifications) { 
            this.emailNotifications = emailNotifications; 
        }
        
        public boolean isOrderTracking() { return orderTracking; }
        public void setOrderTracking(boolean orderTracking) { 
            this.orderTracking = orderTracking; 
        }
        
        public boolean isRecommendations() { return recommendations; }
        public void setRecommendations(boolean recommendations) { 
            this.recommendations = recommendations; 
        }
    }
    
    public static class Payment {
        private int timeout;
        private int retryAttempts;
        
        // Getters and setters
        public int getTimeout() { return timeout; }
        public void setTimeout(int timeout) { this.timeout = timeout; }
        
        public int getRetryAttempts() { return retryAttempts; }
        public void setRetryAttempts(int retryAttempts) { this.retryAttempts = retryAttempts; }
    }
    
    // Main class getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getVersion() { return version; }
    public void setVersion(String version) { this.version = version; }
    
    public int getOrderTimeout() { return orderTimeout; }
    public void setOrderTimeout(int orderTimeout) { this.orderTimeout = orderTimeout; }
    
    public Features getFeatures() { return features; }
    public void setFeatures(Features features) { this.features = features; }
    
    public Payment getPayment() { return payment; }
    public void setPayment(Payment payment) { this.payment = payment; }
}

// 3. Configuration Classes
@Configuration
@EnableConfigurationProperties(ECommerceProperties.class)
public class PaymentConfig {
    
    @Bean("stripePayment")
    @Profile("prod")
    public PaymentService stripePaymentService() {
        return new StripePaymentService();
    }
    
    @Bean("mockPayment")
    @Profile({"dev", "test"})
    @Primary
    public PaymentService mockPaymentService() {
        return new MockPaymentService();
    }
    
    @Bean
    @Scope("prototype") // New cart for each user session
    public ShoppingCart shoppingCart() {
        return new ShoppingCart();
    }
}

// 4. Service Layer - Business Logic
@Service
@Transactional
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    private final EmailNotificationService emailService;
    private final ECommerceProperties properties;
    private final ApplicationContext applicationContext;
    
    // Constructor injection - all dependencies
    public OrderService(
            OrderRepository orderRepository,
            PaymentService paymentService,
            EmailNotificationService emailService,
            ECommerceProperties properties,
            ApplicationContext applicationContext) {
        
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
        this.emailService = emailService;
        this.properties = properties;
        this.applicationContext = applicationContext;
    }
    
    public OrderResponse createOrder(OrderRequest request) {
        
        // 1. Create new shopping cart (prototype bean)
        ShoppingCart cart = applicationContext.getBean(ShoppingCart.class);
        cart.setUserId(request.getUserId());
        
        // 2. Add items to cart
        request.getItems().forEach(item -> 
            cart.addItem(item.getProductId(), item.getQuantity(), item.getPrice())
        );
        
        // 3. Process payment with timeout from properties
        PaymentRequest paymentRequest = new PaymentRequest();
        paymentRequest.setAmount(cart.getTotal());
        paymentRequest.setCustomerId(request.getUserId());
        paymentRequest.setTimeout(properties.getPayment().getTimeout());
        
        PaymentResult paymentResult = paymentService.processPayment(paymentRequest);
        
        if (!paymentResult.isSuccessful()) {
            throw new PaymentFailedException("Payment failed: " + paymentResult.getErrorMessage());
        }
        
        // 4. Save order
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setItems(cart.getItems());
        order.setTotalAmount(cart.getTotal());
        order.setPaymentTransactionId(paymentResult.getTransactionId());
        order.setStatus("CONFIRMED");
        order.setCreatedDate(LocalDateTime.now());
        
        Order savedOrder = orderRepository.save(order);
        
        // 5. Send notification if enabled
        if (properties.getFeatures().isEmailNotifications()) {
            emailService.sendOrderConfirmation(savedOrder);
        }
        
        return OrderResponse.builder()
                .orderId(savedOrder.getId())
                .status(savedOrder.getStatus())
                .totalAmount(savedOrder.getTotalAmount())
                .message("Order created successfully")
                .build();
    }
    
    public List<Order> getUserOrders(Long userId) {
        return orderRepository.findByUserId(userId);
    }
    
    public String getSystemInfo() {
        return String.format("""
            === E-Commerce System Info ===
            Name: %s v%s
            Order Timeout: %d seconds
            Payment Service: %s
            Features Enabled: %s
            """,
            properties.getName(),
            properties.getVersion(),
            properties.getOrderTimeout(),
            paymentService.getClass().getSimpleName(),
            getEnabledFeatures()
        );
    }
    
    private String getEnabledFeatures() {
        List<String> features = new ArrayList<>();
        if (properties.getFeatures().isEmailNotifications()) features.add("Email");
        if (properties.getFeatures().isOrderTracking()) features.add("Tracking");
        if (properties.getFeatures().isRecommendations()) features.add("Recommendations");
        return String.join(", ", features);
    }
}

// 5. Repository Layer - Data Access
@Repository
public class OrderRepository {
    
    private final Map<Long, Order> orders = new ConcurrentHashMap<>();
    private final AtomicLong orderIdGenerator = new AtomicLong(1);
    
    public Order save(Order order) {
        if (order.getId() == null) {
            order.setId(orderIdGenerator.getAndIncrement());
        }
        orders.put(order.getId(), order);
        
        System.out.println("💾 Order saved: " + order.getId());
        return order;
    }
    
    public Optional<Order> findById(Long id) {
        return Optional.ofNullable(orders.get(id));
    }
    
    public List<Order> findByUserId(Long userId) {
        return orders.values().stream()
                .filter(order -> order.getUserId().equals(userId))
                .collect(Collectors.toList());
    }
    
    public List<Order> findAll() {
        return new ArrayList<>(orders.values());
    }
}

// 6. Payment Services - Profile-based implementations
public interface PaymentService {
    PaymentResult processPayment(PaymentRequest request);
}

@Service
@Profile({"dev", "test"})
public class MockPaymentService implements PaymentService {
    
    @Value("${payment.mock.always-succeed:true}")
    private boolean alwaysSucceed;
    
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        System.out.println("🧪 [MOCK] Processing payment: $" + request.getAmount());
        
        // Simulate processing delay
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        if (alwaysSucceed) {
            return PaymentResult.builder()
                    .successful(true)
                    .transactionId("MOCK-" + System.currentTimeMillis())
                    .message("Mock payment successful")
                    .build();
        } else {
            return PaymentResult.builder()
                    .successful(false)
                    .errorMessage("Mock payment failure")
                    .build();
        }
    }
}

@Service
@Profile("prod")
public class StripePaymentService implements PaymentService {
    
    @Value("${payment.stripe.api-key}")
    private String stripeApiKey;
    
    @Value("${payment.stripe.timeout:30}")
    private int timeoutSeconds;
    
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        System.out.println("💳 [STRIPE] Processing payment: $" + request.getAmount());
        System.out.println("Using API key: " + maskApiKey(stripeApiKey));
        System.out.println("Timeout: " + timeoutSeconds + "s");
        
        // Simulate Stripe API call
        try {
            Thread.sleep(1000); // Simulate network delay
            
            // 90% success rate simulation
            if (Math.random() < 0.9) {
                return PaymentResult.builder()
                        .successful(true)
                        .transactionId("stripe_" + System.currentTimeMillis())
                        .message("Payment processed via Stripe")
                        .build();
            } else {
                return PaymentResult.builder()
                        .successful(false)
                        .errorMessage("Card declined")
                        .build();
            }
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return PaymentResult.builder()
                    .successful(false)
                    .errorMessage("Payment timeout")
                    .build();
        }
    }
    
    private String maskApiKey(String apiKey) {
        if (apiKey == null || apiKey.length() < 8) return "****";
        return apiKey.substring(0, 4) + "****" + apiKey.substring(apiKey.length() - 4);
    }
}

// 7. Component - Generic functionality
@Component
public class EmailNotificationService {
    
    private final ECommerceProperties properties;
    
    @Value("${spring.profiles.active:default}")
    private String activeProfile;
    
    public EmailNotificationService(ECommerceProperties properties) {
        this.properties = properties;
    }
    
    public void sendOrderConfirmation(Order order) {
        if (!properties.getFeatures().isEmailNotifications()) {
            System.out.println("📧 Email notifications disabled - skipping");
            return;
        }
        
        System.out.printf("""
            📧 [%s] Sending order confirmation:
              Order ID: %d
              Customer: %d
              Amount: $%.2f
              Status: %s
              Timestamp: %s
            """,
            activeProfile.toUpperCase(),
            order.getId(),
            order.getUserId(),
            order.getTotalAmount(),
            order.getStatus(),
            order.getCreatedDate()
        );
    }
    
    public void sendWelcomeEmail(Long userId) {
        System.out.printf("📧 [%s] Welcome email sent to user: %d%n", 
            activeProfile.toUpperCase(), userId);
    }
}

// 8. Controller Layer - REST endpoints
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    private final OrderService orderService;
    
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody OrderRequest request) {
        try {
            OrderResponse response = orderService.createOrder(request);
            return ResponseEntity.ok(response);
        } catch (PaymentFailedException e) {
            return ResponseEntity.badRequest()
                    .body(OrderResponse.builder()
                            .message("Payment failed: " + e.getMessage())
                            .build());
        }
    }
    
    @GetMapping("/user/{userId}")
    public ResponseEntity<List<Order>> getUserOrders(@PathVariable Long userId) {
        List<Order> orders = orderService.getUserOrders(userId);
        return ResponseEntity.ok(orders);
    }
    
    @GetMapping("/system/info")
    public ResponseEntity<String> getSystemInfo() {
        return ResponseEntity.ok(orderService.getSystemInfo());
    }
}

// 9. Supporting Classes
@Component
@Scope("prototype")
public class ShoppingCart {
    
    private Long userId;
    private List<CartItem> items = new ArrayList<>();
    private LocalDateTime createdAt;
    
    public ShoppingCart() {
        this.createdAt = LocalDateTime.now();
        System.out.println("🛒 New shopping cart created at: " + createdAt);
    }
    
    public void addItem(Long productId, int quantity, double price) {
        CartItem item = new CartItem(productId, quantity, price);
        items.add(item);
        System.out.printf("➕ Added to cart: Product %d, Qty: %d, Price: $%.2f%n", 
            productId, quantity, price);
    }
    
    public double getTotal() {
        return items.stream()
                .mapToDouble(item -> item.getQuantity() * item.getPrice())
                .sum();
    }
    
    // Getters and setters
    public Long getUserId() { return userId; }
    public void setUserId(Long userId) { this.userId = userId; }
    public List<CartItem> getItems() { return new ArrayList<>(items); }
    public LocalDateTime getCreatedAt() { return createdAt; }
}

// Data classes for the system
public class Order {
    private Long id;
    private Long userId;
    private List<CartItem> items;
    private double totalAmount;
    private String paymentTransactionId;
    private String status;
    private LocalDateTime createdDate;
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public Long getUserId() { return userId; }
    public void setUserId(Long userId) { this.userId = userId; }
    
    public List<CartItem> getItems() { return items; }
    public void setItems(List<CartItem> items) { this.items = items; }
    
    public double getTotalAmount() { return totalAmount; }
    public void setTotalAmount(double totalAmount) { this.totalAmount = totalAmount; }
    
    public String getPaymentTransactionId() { return paymentTransactionId; }
    public void setPaymentTransactionId(String paymentTransactionId) { 
        this.paymentTransactionId = paymentTransactionId; 
    }
    
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
    
    public LocalDateTime getCreatedDate() { return createdDate; }
    public void setCreatedDate(LocalDateTime createdDate) { this.createdDate = createdDate; }
}

public class CartItem {
    private Long productId;
    private int quantity;
    private double price;
    
    public CartItem(Long productId, int quantity, double price) {
        this.productId = productId;
        this.quantity = quantity;
        this.price = price;
    }
    
    // Getters
    public Long getProductId() { return productId; }
    public int getQuantity() { return quantity; }
    public double getPrice() { return price; }
}

// Request/Response classes
public class OrderRequest {
    private Long userId;
    private List<OrderItem> items;
    
    public static class OrderItem {
        private Long productId;
        private int quantity;
        private double price;
        
        // Getters and setters
        public Long getProductId() { return productId; }
        public void setProductId(Long productId) { this.productId = productId; }
        
        public int getQuantity() { return quantity; }
        public void setQuantity(int quantity) { this.quantity = quantity; }
        
        public double getPrice() { return price; }
        public void setPrice(double price) { this.price = price; }
    }
    
    // Getters and setters
    public Long getUserId() { return userId; }
    public void setUserId(Long userId) { this.userId = userId; }
    
    public List<OrderItem> getItems() { return items; }
    public void setItems(List<OrderItem> items) { this.items = items; }
}

public class OrderResponse {
    private Long orderId;
    private String status;
    private double totalAmount;
    private String message;
    
    public static OrderResponseBuilder builder() {
        return new OrderResponseBuilder();
    }
    
    public static class OrderResponseBuilder {
        private OrderResponse response = new OrderResponse();
        
        public OrderResponseBuilder orderId(Long orderId) {
            response.orderId = orderId;
            return this;
        }
        
        public OrderResponseBuilder status(String status) {
            response.status = status;
            return this;
        }
        
        public OrderResponseBuilder totalAmount(double totalAmount) {
            response.totalAmount = totalAmount;
            return this;
        }
        
        public OrderResponseBuilder message(String message) {
            response.message = message;
            return this;
        }
        
        public OrderResponse build() {
            return response;
        }
    }
    
    // Getters
    public Long getOrderId() { return orderId; }
    public String getStatus() { return status; }
    public double getTotalAmount() { return totalAmount; }
    public String getMessage() { return message; }
}

public class PaymentRequest {
    private double amount;
    private Long customerId;
    private int timeout;
    
    // Getters and setters
    public double getAmount() { return amount; }
    public void setAmount(double amount) { this.amount = amount; }
    
    public Long getCustomerId() { return customerId; }
    public void setCustomerId(Long customerId) { this.customerId = customerId; }
    
    public int getTimeout() { return timeout; }
    public void setTimeout(int timeout) { this.timeout = timeout; }
}

public class PaymentResult {
    private boolean successful;
    private String transactionId;
    private String message;
    private String errorMessage;
    
    public static PaymentResultBuilder builder() {
        return new PaymentResultBuilder();
    }
    
    public static class PaymentResultBuilder {
        private PaymentResult result = new PaymentResult();
        
        public PaymentResultBuilder successful(boolean successful) {
            result.successful = successful;
            return this;
        }
        
        public PaymentResultBuilder transactionId(String transactionId) {
            result.transactionId = transactionId;
            return this;
        }
        
        public PaymentResultBuilder message(String message) {
            result.message = message;
            return this;
        }
        
        public PaymentResultBuilder errorMessage(String errorMessage) {
            result.errorMessage = errorMessage;
            return this;
        }
        
        public PaymentResult build() {
            return result;
        }
    }
    
    // Getters
    public boolean isSuccessful() { return successful; }
    public String getTransactionId() { return transactionId; }
    public String getMessage() { return message; }
    public String getErrorMessage() { return errorMessage; }
}

// Exception classes
public class PaymentFailedException extends RuntimeException {
    public PaymentFailedException(String message) {
        super(message);
    }
}
```

### Configuration Files:

**application.yml**:
```yaml
ecommerce:
  name: E-Commerce Platform
  version: 1.0.0
  order-timeout: 300
  features:
    email-notifications: true
    order-tracking: true
    recommendations: false
  payment:
    timeout: 30
    retry-attempts: 3

server:
  port: 8080

logging:
  level:
    com.ecommerce: INFO
```

**application-dev.yml**:
```yaml
ecommerce:
  name: E-Commerce Platform (DEV)
  features:
    email-notifications: true
    order-tracking: true
    recommendations: true

payment:
  mock:
    always-succeed: true
  timeout: 10

logging:
  level:
    com.ecommerce: DEBUG
    org.springframework: DEBUG

server:
  port: 8081
```

**application-prod.yml**:
```yaml
ecommerce:
  name: E-Commerce Platform
  features:
    email-notifications: true
    order-tracking: true
    recommendations: true

payment:
  stripe:
    api-key: ${STRIPE_API_KEY}
    timeout: 30
  timeout: 30

logging:
  level:
    com.ecommerce: WARN
    root: ERROR
  file:
    name: /logs/ecommerce.log

server:
  port: 8080
```

---

## 🎯 Interview Cheat Sheet

### Quick Reference Card:
```
🏷️  ANNOTATIONS SUMMARY:

@SpringBootApplication → Main class (auto-config + component scan)
@Component → Generic Spring bean
@Service → Business logic layer  
@Repository → Data access layer
@Controller → Web request handler
@Configuration → Java-based config class
@Bean → Manual bean creation method
@Autowired → Dependency injection
@Qualifier → Specific bean selection  
@Value → Property injection
@ConfigurationProperties → Type-safe config binding
@Profile → Environment-specific beans
@Scope → Bean lifecycle (singleton/prototype)
@Primary → Default bean when multiple exist
@Lazy → Delay bean creation until needed

🔧 KEY CONCEPTS:
- Constructor Injection > Field Injection
- Singleton (default) vs Prototype scopes
- Profile activation: spring.profiles.active=dev
- YAML > Properties for configuration
- Environment variables for sensitive data
```

### Common Interview Questions & Answers:

**Q: Explain @SpringBootApplication**
**A:** Meta-annotation combining @Configuration (Java config), @EnableAutoConfiguration (smart defaults), and @ComponentScan (find components). It's the starting point that eliminates manual configuration.

**Q: Constructor vs Field Injection?**
**A:** Constructor injection is preferred because it creates immutable dependencies, enables fail-fast behavior, makes testing easier, and clearly shows all dependencies.

**Q: When to use @Profile?**
**A:** For environment-specific beans (dev/test/prod), feature toggles, different implementations per environment (mock vs real services), and resource optimization.

**Q: Singleton vs Prototype scope?**
**A:** Singleton (default) creates one instance per container - use for stateless services. Prototype creates new instance each time - use for stateful objects like shopping carts.

**Q: How to handle circular dependencies?**
**A:** Best solution is architectural redesign. Alternatives: @Lazy annotation, ApplicationContextAware interface, or @PostConstruct initialization.

---
