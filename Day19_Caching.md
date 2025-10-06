# 📅 Day 19: Caching

## 🎯 Today Topics
- Master Spring Cache abstraction with `@EnableCaching`
- Use `@Cacheable`, `@CachePut`, `@CacheEvict` annotations
- Understand cache strategies and TTL
- Implement caching with simple providers
- Optimize application performance

---

## 💡 What is Caching?

### Definition:
**Caching** = Store frequently accessed data in memory to avoid expensive operations (database queries, API calls, computations).

### Benefits:
- **Faster response times** - Data retrieved from memory
- **Reduced database load** - Fewer queries
- **Better scalability** - Handle more requests
- **Cost savings** - Less server resources needed

### Example:
```
Without Cache: Database query takes 200ms
With Cache: Memory retrieval takes 2ms
→ 100x faster!
```

---

## ⚙️ Enable Caching

### Step 1: Add Dependency
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

### Step 2: Enable Caching
```java
@SpringBootApplication
@EnableCaching // Enable caching support
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

---

## 📦 @Cacheable - Cache Results

### Basic Usage:
```java
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    // Cache the result by user ID
    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        System.out.println("🔍 Fetching user from database: " + id);
        
        // Simulate slow database query
        try {
            Thread.sleep(2000); // 2 second delay
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        return userRepository.findById(id)
                .orElseThrow(() -> new UserNotFoundException(id));
    }
    
    // Cache all users
    @Cacheable(value = "allUsers")
    public List<User> findAll() {
        System.out.println("🔍 Fetching all users from database");
        return userRepository.findAll();
    }
    
    // Cache with multiple parameters
    @Cacheable(value = "usersByNameAndAge", key = "#name + '-' + #age")
    public List<User> findByNameAndAge(String name, Integer age) {
        System.out.println("🔍 Fetching users: " + name + ", " + age);
        return userRepository.findByNameAndAge(name, age);
    }
    
    // Conditional caching
    @Cacheable(value = "activeUsers", condition = "#age > 18")
    public List<User> findAdultUsers(Integer age) {
        System.out.println("🔍 Fetching adult users");
        return userRepository.findByAgeGreaterThan(age);
    }
}

/* How it works:
   1st call: findById(1) → Database query (slow) → Cache result
   2nd call: findById(1) → Return from cache (fast)
   3rd call: findById(1) → Return from cache (fast)
*/
```

---

## 🔄 @CachePut - Update Cache

### Usage:
```java
@Service
public class UserService {
    
    // Always execute and update cache
    @CachePut(value = "users", key = "#user.id")
    public User updateUser(User user) {
        System.out.println("🔄 Updating user in database and cache: " + user.getId());
        return userRepository.save(user);
    }
    
    // Create new user and add to cache
    @CachePut(value = "users", key = "#result.id")
    public User createUser(User user) {
        System.out.println("➕ Creating user and adding to cache");
        return userRepository.save(user);
    }
}

/* Difference from @Cacheable:
   @Cacheable: Skips method if cached
   @CachePut: Always runs method, updates cache
*/
```

---

## 🗑️ @CacheEvict - Remove from Cache

### Usage:
```java
@Service
public class UserService {
    
    // Remove single entry
    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) {
        System.out.println("🗑️ Deleting user and removing from cache: " + id);
        userRepository.deleteById(id);
    }
    
    // Clear entire cache
    @CacheEvict(value = "users", allEntries = true)
    public void deleteAllUsers() {
        System.out.println("🗑️ Deleting all users and clearing cache");
        userRepository.deleteAll();
    }
    
    // Evict multiple caches
    @CacheEvict(value = {"users", "allUsers"}, allEntries = true)
    public void clearAllUserCaches() {
        System.out.println("🗑️ Clearing all user-related caches");
    }
    
    // Evict before method execution
    @CacheEvict(value = "users", key = "#id", beforeInvocation = true)
    public void dangerousDelete(Long id) {
        // Cache cleared even if method throws exception
        userRepository.deleteById(id);
    }
}
```

---

## 🎯 Complete Example: Product Service

### Product Service with Caching:
```java
@Service
public class ProductService {
    
    @Autowired
    private ProductRepository productRepository;
    
    // Cache product by ID (most common query)
    @Cacheable(value = "products", key = "#id")
    public Product findById(Long id) {
        System.out.println("📦 [DB Query] Fetching product: " + id);
        simulateSlowQuery();
        
        return productRepository.findById(id)
                .orElseThrow(() -> new ProductNotFoundException(id));
    }
    
    // Cache all products (refresh every hour via TTL)
    @Cacheable(value = "allProducts")
    public List<Product> findAll() {
        System.out.println("📦 [DB Query] Fetching all products");
        simulateSlowQuery();
        return productRepository.findAll();
    }
    
    // Cache products by category
    @Cacheable(value = "productsByCategory", key = "#category")
    public List<Product> findByCategory(String category) {
        System.out.println("📦 [DB Query] Fetching products in category: " + category);
        simulateSlowQuery();
        return productRepository.findByCategory(category);
    }
    
    // Cache expensive search (only if results > 0)
    @Cacheable(value = "productSearch", key = "#keyword", unless = "#result.isEmpty()")
    public List<Product> searchProducts(String keyword) {
        System.out.println("🔍 [DB Query] Searching products: " + keyword);
        simulateSlowQuery();
        return productRepository.findByNameContainingOrDescriptionContaining(keyword, keyword);
    }
    
    // Create product and add to cache
    @CachePut(value = "products", key = "#result.id")
    @CacheEvict(value = {"allProducts", "productsByCategory"}, allEntries = true)
    public Product createProduct(Product product) {
        System.out.println("➕ Creating product: " + product.getName());
        Product saved = productRepository.save(product);
        System.out.println("✅ Product cached with ID: " + saved.getId());
        return saved;
    }
    
    // Update product and refresh cache
    @CachePut(value = "products", key = "#id")
    @CacheEvict(value = {"allProducts", "productsByCategory"}, allEntries = true)
    public Product updateProduct(Long id, Product productDetails) {
        System.out.println("🔄 Updating product: " + id);
        
        Product product = findById(id); // Uses cache if available
        product.setName(productDetails.getName());
        product.setPrice(productDetails.getPrice());
        product.setCategory(productDetails.getCategory());
        
        Product updated = productRepository.save(product);
        System.out.println("✅ Product cache updated");
        return updated;
    }
    
    // Delete and evict from cache
    @CacheEvict(value = "products", key = "#id")
    @CacheEvict(value = {"allProducts", "productsByCategory"}, allEntries = true)
    public void deleteProduct(Long id) {
        System.out.println("🗑️ Deleting product: " + id);
        productRepository.deleteById(id);
        System.out.println("✅ Product removed from cache");
    }
    
    // Update stock (high frequency operation)
    @CachePut(value = "products", key = "#id")
    public Product updateStock(Long id, Integer quantity) {
        System.out.println("📦 Updating stock for product: " + id);
        
        Product product = productRepository.findById(id)
                .orElseThrow(() -> new ProductNotFoundException(id));
        
        product.setStockQuantity(quantity);
        return productRepository.save(product);
    }
    
    // Manual cache management
    @Autowired
    private CacheManager cacheManager;
    
    public void clearProductCache(Long productId) {
        Cache cache = cacheManager.getCache("products");
        if (cache != null) {
            cache.evict(productId);
            System.out.println("🗑️ Manually cleared cache for product: " + productId);
        }
    }
    
    public void clearAllProductCaches() {
        cacheManager.getCacheNames().forEach(cacheName -> {
            if (cacheName.startsWith("product")) {
                Cache cache = cacheManager.getCache(cacheName);
                if (cache != null) {
                    cache.clear();
                    System.out.println("🗑️ Cleared cache: " + cacheName);
                }
            }
        });
    }
    
    private void simulateSlowQuery() {
        try {
            Thread.sleep(1000); // Simulate 1 second database query
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

### Testing Cache Performance:
```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    @Autowired
    private ProductService productService;
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        long startTime = System.currentTimeMillis();
        
        Product product = productService.findById(id);
        
        long duration = System.currentTimeMillis() - startTime;
        System.out.printf("⏱️ Request completed in %dms%n", duration);
        
        return ResponseEntity.ok(product);
    }
    
    @GetMapping
    public ResponseEntity<List<Product>> getAllProducts() {
        long startTime = System.currentTimeMillis();
        
        List<Product> products = productService.findAll();
        
        long duration = System.currentTimeMillis() - startTime;
        System.out.printf("⏱️ Request completed in %dms (returned %d products)%n", 
                         duration, products.size());
        
        return ResponseEntity.ok(products);
    }
    
    @GetMapping("/search")
    public ResponseEntity<List<Product>> searchProducts(@RequestParam String keyword) {
        long startTime = System.currentTimeMillis();
        
        List<Product> products = productService.searchProducts(keyword);
        
        long duration = System.currentTimeMillis() - startTime;
        System.out.printf("⏱️ Search completed in %dms (found %d products)%n", 
                         duration, products.size());
        
        return ResponseEntity.ok(products);
    }
}

/* Test Results:
   1st call: GET /api/products/1 → 1002ms (database query)
   2nd call: GET /api/products/1 → 2ms (from cache) 
   3rd call: GET /api/products/1 → 2ms (from cache)
   
   500x faster with caching! ⚡
*/
```

---

## ⏰ Cache Configuration with TTL

### Simple Cache Config:
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        
        cacheManager.setCaches(Arrays.asList(
            new ConcurrentMapCache("users"),
            new ConcurrentMapCache("products"),
            new ConcurrentMapCache("allProducts")
        ));
        
        System.out.println("✅ Cache manager configured");
        return cacheManager;
    }
}
```

### Caffeine Cache with TTL (Recommended):
```xml
<!-- Add Caffeine dependency -->
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager(
            "users", "products", "allProducts", "productsByCategory"
        );
        
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .expireAfterWrite(10, TimeUnit.MINUTES)  // TTL: 10 minutes
            .maximumSize(1000)                       // Max 1000 entries
            .recordStats());                         // Enable statistics
        
        System.out.println("✅ Caffeine cache configured with 10min TTL");
        return cacheManager;
    }
}
```

### Different TTL per Cache:
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        
        // Users cache - 1 hour
        cacheManager.registerCustomCache("users", 
            Caffeine.newBuilder()
                .expireAfterWrite(1, TimeUnit.HOURS)
                .maximumSize(500)
                .build());
        
        // Products cache - 30 minutes
        cacheManager.registerCustomCache("products",
            Caffeine.newBuilder()
                .expireAfterWrite(30, TimeUnit.MINUTES)
                .maximumSize(1000)
                .build());
        
        // Search results cache - 5 minutes
        cacheManager.registerCustomCache("productSearch",
            Caffeine.newBuilder()
                .expireAfterWrite(5, TimeUnit.MINUTES)
                .maximumSize(200)
                .build());
        
        System.out.println("✅ Multi-TTL cache configuration loaded");
        return cacheManager;
    }
}
```

---

## 📊 Cache Annotations Quick Reference

| Annotation | Purpose | When to Use |
|------------|---------|-------------|
| `@Cacheable` | Cache method result | Read operations (findById, findAll) |
| `@CachePut` | Update cache | Create/Update operations |
| `@CacheEvict` | Remove from cache | Delete operations |
| `@Caching` | Combine multiple cache operations | Complex scenarios |

---

## 🎯 Interview Questions & Answers

### Q1: "What's the difference between @Cacheable and @CachePut?"
**Answer:**
- **@Cacheable**: Skips method execution if value is cached, returns cached value
- **@CachePut**: Always executes method, then updates cache with result
- **Use @Cacheable for reads** (findById, search)
- **Use @CachePut for writes** (create, update)

### Q2: "How do you invalidate cache when data changes?"
**Answer:**
```java
// Option 1: @CacheEvict on delete
@CacheEvict(value = "users", key = "#id")
public void deleteUser(Long id) { }

// Option 2: Clear all entries
@CacheEvict(value = "users", allEntries = true)
public void updateManyUsers() { }

// Option 3: Manual eviction
cacheManager.getCache("users").evict(userId);
```

### Q3: "What are common caching strategies?"
**Answer:**
- **Cache-Aside**: Application manages cache (Spring default)
- **Write-Through**: Write to cache and database simultaneously
- **Write-Behind**: Write to cache first, database later asynchronously
- **Read-Through**: Cache loads from database if miss

### Q4: "What is cache TTL and why is it important?"
**Answer:**
- **TTL (Time To Live)**: How long data stays in cache before expiring
- **Prevents stale data**: Ensures cache is refreshed periodically
- **Balances freshness vs performance**: Shorter TTL = fresher data but more queries
- **Configure based on data volatility**: Frequently changing data = shorter TTL

---

## 🚀 Best Practices

### ✅ Do's:
```java
// 1. Cache expensive operations only
@Cacheable(value = "expensiveQuery")
public List<Data> complexDatabaseQuery() { }

// 2. Use meaningful cache names
@Cacheable(value = "usersByEmail") // Clear name
public User findByEmail(String email) { }

// 3. Set appropriate TTL
Caffeine.newBuilder().expireAfterWrite(10, TimeUnit.MINUTES)

// 4. Evict cache on updates
@CacheEvict(value = "products", key = "#id")
public void updateProduct(Long id) { }

// 5. Use condition for selective caching
@Cacheable(value = "users", condition = "#age > 18")
public List<User> findUsers(Integer age) { }
```

### ❌ Don'ts:
```java
// 1. Don't cache frequently changing data
@Cacheable(value = "stockPrices") // Changes every second - bad idea!

// 2. Don't cache large objects without size limits
@Cacheable(value = "largeFiles") // May cause memory issues

// 3. Don't forget to evict on updates
public void updateUser(User user) { 
    userRepository.save(user); 
    // Cache not updated - stale data!
}

// 4. Don't use cache for everything
@Cacheable(value = "simpleCalc")
public int add(int a, int b) { return a + b; } // Unnecessary!

// 5. Don't ignore cache size limits
// Without maximumSize(), cache can grow infinitely
```

---

## 💡 When to Use Caching

### ✅ Good Use Cases:
- **Read-heavy operations** (80% reads, 20% writes)
- **Expensive database queries** (complex joins, aggregations)
- **External API calls** (slow third-party services)
- **Computed results** (complex calculations)
- **Static or rarely changing data** (product catalogs, configurations)

### ❌ Avoid Caching:
- **Real-time data** (stock prices, live scores)
- **User-specific data** (shopping carts, sessions)
- **Frequently updated data** (inventory counts)
- **Large objects** (videos, large files)
- **Security-sensitive data** (passwords, tokens)

---

