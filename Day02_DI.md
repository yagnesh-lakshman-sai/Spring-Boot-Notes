# 📅 Day 2: Dependency Injection & Stereotype Annotations

## 🎯 Topics to Learn :
- Master Dependency Injection concepts
- Understand `@Component`, `@Service`, `@Repository`, `@Controller`
- Learn when and why to use each annotation
- Real-world examples from FAANG companies

---

## 🧠 Understanding Dependency Injection (DI)

### What is Dependency Injection?
**Dependency Injection** is a design pattern where objects receive their dependencies from external sources rather than creating them internally.

### The Problem Without DI:
```java
// ❌ BAD: Tight Coupling
public class OrderService {
    private PaymentService paymentService;
    private EmailService emailService;
    
    public OrderService() {
        // Hard-coded dependencies - nightmare to test!
        this.paymentService = new PayPalPaymentService();
        this.emailService = new GmailEmailService();
    }
}
```

### The Solution With DI:
```java
// ✅ GOOD: Loose Coupling
@Service
public class OrderService {
    private final PaymentService paymentService;
    private final EmailService emailService;
    
    // Spring injects dependencies automatically!
    public OrderService(PaymentService paymentService, EmailService emailService) {
        this.paymentService = paymentService;
        this.emailService = emailService;
    }
}
```

---

## 🏷️ Spring Stereotype Annotations Hierarchy

```
      @Component (Base)
           |
    ┌──────┼──────┐
    |      |      |
@Controller @Service @Repository
```

### Why Multiple Annotations?
1. **Semantic Clarity** → Code is self-documenting
2. **AOP Enhancement** → Different behaviors for each layer
3. **Future Extensibility** → Spring can add layer-specific features
4. **Exception Translation** → `@Repository` gets automatic exception handling

---

## 🎯 @Component - The Universal Annotation

### Definition:
Generic stereotype annotation for any Spring-managed component.

### When to Use:
- General-purpose beans
- Utility classes
- Configuration helpers
- When other stereotypes don't fit

### Real-World Example: Netflix's Content Recommendation
```java
@Component
public class RecommendationEngine {
    
    @Autowired
    private UserPreferenceAnalyzer userAnalyzer;
    
    @Autowired
    private ContentSimilarityCalculator similarityCalculator;
    
    public List<Movie> recommendMovies(Long userId) {
        UserPreferences preferences = userAnalyzer.analyze(userId);
        return similarityCalculator.findSimilarContent(preferences);
    }
}

// Usage in Netflix-style streaming service
@RestController
public class MovieController {
    
    @Autowired
    private RecommendationEngine recommendationEngine;
    
    @GetMapping("/api/recommendations/{userId}")
    public List<Movie> getRecommendations(@PathVariable Long userId) {
        return recommendationEngine.recommendMovies(userId);
    }
}
```

---

## 🎮 @Controller - Web Layer Handler

### Definition:
Handles HTTP requests and returns views or data. Combined with `@ResponseBody` becomes `@RestController`.

### When to Use:
- Handle web requests
- REST API endpoints
- MVC controllers
- User interface logic

### Real-World Example: Instagram Photo Upload API
```java
@RestController
@RequestMapping("/api/photos")
public class PhotoController {
    
    @Autowired
    private PhotoService photoService;
    
    @Autowired
    private UserService userService;
    
    @PostMapping("/upload")
    public ResponseEntity<PhotoResponse> uploadPhoto(
            @RequestParam("file") MultipartFile file,
            @RequestParam("caption") String caption,
            @RequestHeader("Authorization") String token) {
        
        // Validate user
        User user = userService.validateToken(token);
        
        // Process photo upload
        Photo savedPhoto = photoService.uploadPhoto(file, caption, user);
        
        return ResponseEntity.ok(new PhotoResponse(savedPhoto));
    }
    
    @GetMapping("/feed/{userId}")
    public List<Photo> getUserFeed(@PathVariable Long userId) {
        return photoService.getUserFeed(userId);
    }
    
    @PostMapping("/{photoId}/like")
    public ResponseEntity<String> likePhoto(
            @PathVariable Long photoId,
            @RequestHeader("Authorization") String token) {
        
        User user = userService.validateToken(token);
        photoService.likePhoto(photoId, user.getId());
        
        return ResponseEntity.ok("Photo liked successfully");
    }
}
```

---

## 🏢 @Service - Business Logic Layer

### Definition:
Contains business logic and acts as a service layer between controllers and repositories.

### When to Use:
- Business logic implementation
- Transaction management
- Data validation
- Complex operations

### Real-World Example: Amazon Order Processing
```java
@Service
@Transactional
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private InventoryService inventoryService;
    
    @Autowired
    private PaymentService paymentService;
    
    @Autowired
    private ShippingService shippingService;
    
    @Autowired
    private NotificationService notificationService;
    
    public OrderResponse processOrder(OrderRequest orderRequest) {
        // 1. Validate order
        validateOrder(orderRequest);
        
        // 2. Check inventory
        if (!inventoryService.isAvailable(orderRequest.getItems())) {
            throw new InsufficientInventoryException("Items not available");
        }
        
        // 3. Calculate total
        BigDecimal totalAmount = calculateTotal(orderRequest.getItems());
        
        // 4. Process payment
        PaymentResult paymentResult = paymentService.processPayment(
            orderRequest.getPaymentMethod(), totalAmount);
        
        if (!paymentResult.isSuccessful()) {
            throw new PaymentFailedException("Payment processing failed");
        }
        
        // 5. Reserve inventory
        inventoryService.reserveItems(orderRequest.getItems());
        
        // 6. Create order
        Order order = new Order();
        order.setItems(orderRequest.getItems());
        order.setTotalAmount(totalAmount);
        order.setStatus(OrderStatus.CONFIRMED);
        order.setPaymentId(paymentResult.getTransactionId());
        
        Order savedOrder = orderRepository.save(order);
        
        // 7. Initiate shipping
        shippingService.createShipment(savedOrder);
        
        // 8. Send notifications
        notificationService.sendOrderConfirmation(savedOrder);
        
        return new OrderResponse(savedOrder.getId(), "Order processed successfully");
    }
    
    private void validateOrder(OrderRequest request) {
        if (request.getItems().isEmpty()) {
            throw new InvalidOrderException("Order must contain at least one item");
        }
        // Additional validations...
    }
    
    private BigDecimal calculateTotal(List<OrderItem> items) {
        return items.stream()
                .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

---

## 🗄️ @Repository - Data Access Layer

### Definition:
Handles data access logic and provides automatic exception translation from database-specific exceptions to Spring's DataAccessException hierarchy.

### When to Use:
- Database operations
- Data persistence
- Query execution
- Custom data access logic

### Special Feature: Exception Translation
Spring automatically converts database exceptions to more meaningful exceptions.

### Real-World Example: Twitter User Repository
```java
@Repository
public class UserRepository {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    public User findByUsername(String username) {
        // First check Redis cache
        String cacheKey = "user:" + username;
        User cachedUser = (User) redisTemplate.opsForValue().get(cacheKey);
        
        if (cachedUser != null) {
            return cachedUser;
        }
        
        // If not in cache, query database
        String sql = """
            SELECT u.id, u.username, u.email, u.created_date,
                   p.bio, p.profile_picture_url, p.follower_count
            FROM users u
            LEFT JOIN user_profiles p ON u.id = p.user_id
            WHERE u.username = ? AND u.is_active = true
            """;
        
        try {
            User user = jdbcTemplate.queryForObject(sql, 
                new Object[]{username}, 
                (rs, rowNum) -> {
                    User u = new User();
                    u.setId(rs.getLong("id"));
                    u.setUsername(rs.getString("username"));
                    u.setEmail(rs.getString("email"));
                    u.setCreatedDate(rs.getTimestamp("created_date").toLocalDateTime());
                    
                    // Profile information
                    u.setBio(rs.getString("bio"));
                    u.setProfilePictureUrl(rs.getString("profile_picture_url"));
                    u.setFollowerCount(rs.getInt("follower_count"));
                    
                    return u;
                });
            
            // Cache for 1 hour
            redisTemplate.opsForValue().set(cacheKey, user, 3600, TimeUnit.SECONDS);
            
            return user;
            
        } catch (EmptyResultDataAccessException e) {
            // Spring's exception translation in action!
            // Database-specific exceptions are converted to Spring exceptions
            throw new UserNotFoundException("User not found: " + username);
        }
    }
    
    public List<User> findFollowers(Long userId, int limit, int offset) {
        String sql = """
            SELECT u.id, u.username, u.email, p.profile_picture_url
            FROM users u
            JOIN followers f ON u.id = f.follower_id
            LEFT JOIN user_profiles p ON u.id = p.user_id
            WHERE f.following_id = ? AND u.is_active = true
            ORDER BY f.created_date DESC
            LIMIT ? OFFSET ?
            """;
        
        return jdbcTemplate.query(sql, 
            new Object[]{userId, limit, offset},
            (rs, rowNum) -> {
                User user = new User();
                user.setId(rs.getLong("id"));
                user.setUsername(rs.getString("username"));
                user.setEmail(rs.getString("email"));
                user.setProfilePictureUrl(rs.getString("profile_picture_url"));
                return user;
            });
    }
    
    @Transactional
    public void followUser(Long followerId, Long followingId) {
        // Check if already following
        String checkSql = "SELECT COUNT(*) FROM followers WHERE follower_id = ? AND following_id = ?";
        int count = jdbcTemplate.queryForObject(checkSql, Integer.class, followerId, followingId);
        
        if (count > 0) {
            throw new AlreadyFollowingException("User already following");
        }
        
        // Insert follow relationship
        String insertSql = "INSERT INTO followers (follower_id, following_id, created_date) VALUES (?, ?, NOW())";
        jdbcTemplate.update(insertSql, followerId, followingId);
        
        // Update follower count (denormalized for performance)
        String updateCountSql = "UPDATE user_profiles SET follower_count = follower_count + 1 WHERE user_id = ?";
        jdbcTemplate.update(updateCountSql, followingId);
    }
}
```

---

## 📊 Complete Layered Architecture Example: E-Commerce System

```java
// Controller Layer - Handles HTTP requests
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    @Autowired
    private ProductService productService;
    
    @GetMapping
    public ResponseEntity<List<Product>> getAllProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(required = false) String category) {
        
        List<Product> products = productService.getProducts(page, size, category);
        return ResponseEntity.ok(products);
    }
    
    @PostMapping
    public ResponseEntity<Product> createProduct(@Valid @RequestBody ProductRequest request) {
        Product product = productService.createProduct(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(product);
    }
}

// Service Layer - Business logic
@Service
@Transactional
public class ProductService {
    
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private CategoryRepository categoryRepository;
    
    @Autowired
    private InventoryService inventoryService;
    
    @Autowired
    private CacheManager cacheManager;
    
    public List<Product> getProducts(int page, int size, String category) {
        if (category != null) {
            return productRepository.findByCategory(category, page, size);
        }
        return productRepository.findAll(page, size);
    }
    
    public Product createProduct(ProductRequest request) {
        // Validate category exists
        Category category = categoryRepository.findByName(request.getCategoryName());
        if (category == null) {
            throw new CategoryNotFoundException("Category not found: " + request.getCategoryName());
        }
        
        // Create product
        Product product = new Product();
        product.setName(request.getName());
        product.setDescription(request.getDescription());
        product.setPrice(request.getPrice());
        product.setCategory(category);
        
        Product savedProduct = productRepository.save(product);
        
        // Initialize inventory
        inventoryService.createInventoryRecord(savedProduct.getId(), request.getInitialStock());
        
        // Clear cache
        cacheManager.clearCache("products");
        
        return savedProduct;
    }
}

// Repository Layer - Data access
@Repository
public class ProductRepository {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public List<Product> findAll(int page, int size) {
        int offset = page * size;
        String sql = """
            SELECT p.id, p.name, p.description, p.price, p.created_date,
                   c.id as category_id, c.name as category_name
            FROM products p
            JOIN categories c ON p.category_id = c.id
            WHERE p.is_active = true
            ORDER BY p.created_date DESC
            LIMIT ? OFFSET ?
            """;
        
        return jdbcTemplate.query(sql, new Object[]{size, offset}, this::mapRowToProduct);
    }
    
    public List<Product> findByCategory(String categoryName, int page, int size) {
        int offset = page * size;
        String sql = """
            SELECT p.id, p.name, p.description, p.price, p.created_date,
                   c.id as category_id, c.name as category_name
            FROM products p
            JOIN categories c ON p.category_id = c.id
            WHERE c.name = ? AND p.is_active = true
            ORDER BY p.created_date DESC
            LIMIT ? OFFSET ?
            """;
        
        return jdbcTemplate.query(sql, new Object[]{categoryName, size, offset}, this::mapRowToProduct);
    }
    
    public Product save(Product product) {
        if (product.getId() == null) {
            // Insert new product
            String sql = "INSERT INTO products (name, description, price, category_id, created_date, is_active) VALUES (?, ?, ?, ?, NOW(), true)";
            
            KeyHolder keyHolder = new GeneratedKeyHolder();
            jdbcTemplate.update(connection -> {
                PreparedStatement ps = connection.prepareStatement(sql, new String[]{"id"});
                ps.setString(1, product.getName());
                ps.setString(2, product.getDescription());
                ps.setBigDecimal(3, product.getPrice());
                ps.setLong(4, product.getCategory().getId());
                return ps;
            }, keyHolder);
            
            product.setId(keyHolder.getKey().longValue());
        } else {
            // Update existing product
            String sql = "UPDATE products SET name = ?, description = ?, price = ? WHERE id = ?";
            jdbcTemplate.update(sql, product.getName(), product.getDescription(), product.getPrice(), product.getId());
        }
        
        return product;
    }
    
    private Product mapRowToProduct(ResultSet rs, int rowNum) throws SQLException {
        Product product = new Product();
        product.setId(rs.getLong("id"));
        product.setName(rs.getString("name"));
        product.setDescription(rs.getString("description"));
        product.setPrice(rs.getBigDecimal("price"));
        product.setCreatedDate(rs.getTimestamp("created_date").toLocalDateTime());
        
        Category category = new Category();
        category.setId(rs.getLong("category_id"));
        category.setName(rs.getString("category_name"));
        product.setCategory(category);
        
        return product;
    }
}

// Utility Component
@Component
public class CacheManager {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    public void clearCache(String cacheKey) {
        Set<String> keys = redisTemplate.keys(cacheKey + "*");
        if (keys != null && !keys.isEmpty()) {
            redisTemplate.delete(keys);
        }
    }
}
```

---

## 🎯 Interview Questions & Answers

### Q1: "What's the difference between @Component, @Service, @Repository, and @Controller?"
**Answer:**
- `@Component`: Generic stereotype, base for others
- `@Controller`: Web layer, handles HTTP requests
- `@Service`: Business layer, contains business logic
- `@Repository`: Data layer, handles data access + exception translation
- **All are specialized @Component annotations with specific purposes**

### Q2: "Why not just use @Component everywhere?"
**Answer:**
1. **Semantic Clarity**: Code is self-documenting
2. **AOP Enhancement**: Each layer can have different cross-cutting concerns
3. **Exception Translation**: @Repository provides automatic database exception translation
4. **Future Features**: Spring can add layer-specific functionality
5. **Testing**: Easier to mock specific layers

### Q3: "What is exception translation in @Repository?"
**Answer:**
```java
// Database throws: SQLException
// Spring converts to: DataAccessException (unchecked)
// Benefits: 
// 1. Database-agnostic error handling
// 2. Consistent exception hierarchy
// 3. No need to handle checked exceptions everywhere
```

### Q4: "How does Spring resolve dependencies between these annotations?"
**Answer:**
1. **Component Scanning**: Spring scans packages
2. **Bean Registration**: Creates bean definitions for annotated classes
3. **Dependency Resolution**: Matches types and injects dependencies
4. **Circular Dependency Detection**: Prevents infinite loops
5. **Singleton by Default**: Same instance reused unless configured otherwise

---

## 🚀 Industry Best Practices

### 1. **Layer Separation (Clean Architecture)**
```java
// ❌ DON'T: Service calling Controller
@Service
public class OrderService {
    @Autowired
    private OrderController orderController; // WRONG!
}

// ✅ DO: Proper layer flow
Controller → Service → Repository
```

### 2. **Constructor Injection Over Field Injection**
```java
// ❌ Field Injection (harder to test)
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}

// ✅ Constructor Injection (easier to test, immutable)
@Service
public class UserService {
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### 3. **Interface-Based Design**
```java
// Define interface
public interface PaymentService {
    PaymentResult processPayment(PaymentRequest request);
}

// Multiple implementations
@Service("paypal")
public class PayPalPaymentService implements PaymentService { }

@Service("stripe")
public class StripePaymentService implements PaymentService { }

// Inject specific implementation
@Autowired
@Qualifier("stripe")
private PaymentService paymentService;
```

