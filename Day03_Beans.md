# 📅 Day 3: Bean Configuration & Bean Scopes

## 🎯 What You'll Learn Today
- Master `@Bean`, `@Configuration`, `@ComponentScan`
- Understand Bean Scopes: Singleton vs Prototype
- Learn advanced configuration patterns
- Real-world examples from distributed systems

---

## 🏗️ Understanding Bean Configuration

### What is a Bean?
A **Bean** is an object that is instantiated, assembled, and managed by the Spring IoC container. Think of it as Spring's way of managing object lifecycles.

### Three Ways to Create Beans:

1. **Annotation-Based** (`@Component`, `@Service`, etc.) - Day 2 ✅
2. **Java Configuration** (`@Configuration` + `@Bean`) - Today's Focus 🎯  
3. **XML Configuration** (Legacy - rarely used) ❌

---

## ⚙️ @Configuration - The Configuration Class

### Definition:
`@Configuration` indicates that a class declares one or more `@Bean` methods and may be processed by the Spring container to generate bean definitions.

### When to Use:
- Third-party library integration
- Complex bean initialization
- Conditional bean creation
- Database/Security/Cache configuration

### Real-World Example: Netflix Microservices Configuration
```java
@Configuration
@EnableConfigurationProperties({DatabaseProperties.class, CacheProperties.class})
public class NetflixMicroserviceConfig {
    
    @Autowired
    private DatabaseProperties databaseProperties;
    
    @Autowired  
    private CacheProperties cacheProperties;
    
    // Database Configuration
    @Bean
    @Primary
    public DataSource primaryDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(databaseProperties.getPrimaryUrl());
        dataSource.setUsername(databaseProperties.getUsername());
        dataSource.setPassword(databaseProperties.getPassword());
        dataSource.setMaximumPoolSize(50);
        dataSource.setMinimumIdle(10);
        dataSource.setConnectionTimeout(30000);
        dataSource.setIdleTimeout(600000);
        dataSource.setMaxLifetime(1800000);
        return dataSource;
    }
    
    @Bean("readOnlyDataSource")
    public DataSource readOnlyDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(databaseProperties.getReadOnlyUrl());
        dataSource.setUsername(databaseProperties.getReadOnlyUsername());
        dataSource.setPassword(databaseProperties.getReadOnlyPassword());
        dataSource.setMaximumPoolSize(30);
        dataSource.setReadOnly(true);
        return dataSource;
    }
    
    // Redis Cache Configuration
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        LettuceConnectionFactory factory = new LettuceConnectionFactory(
            cacheProperties.getHost(), 
            cacheProperties.getPort()
        );
        factory.setPassword(cacheProperties.getPassword());
        factory.setDatabase(cacheProperties.getDatabase());
        return factory;
    }
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        
        // JSON serialization for better readability
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.activateDefaultTyping(LazyLoadingAwareJavaType.NON_FINAL);
        serializer.setObjectMapper(objectMapper);
        
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(serializer);
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);
        template.afterPropertiesSet();
        
        return template;
    }
    
    // Message Queue Configuration
    @Bean
    public RabbitConnectionFactory rabbitConnectionFactory() {
        CachingConnectionFactory factory = new CachingConnectionFactory();
        factory.setHost(cacheProperties.getRabbitHost());
        factory.setPort(cacheProperties.getRabbitPort());
        factory.setUsername(cacheProperties.getRabbitUsername());
        factory.setPassword(cacheProperties.getRabbitPassword());
        return factory;
    }
    
    @Bean
    public RabbitTemplate rabbitTemplate(RabbitConnectionFactory connectionFactory) {
        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(new Jackson2JsonMessageConverter());
        template.setRetryTemplate(retryTemplate());
        return template;
    }
    
    // Retry Configuration for Resilience
    @Bean
    public RetryTemplate retryTemplate() {
        RetryTemplate retryTemplate = new RetryTemplate();
        
        FixedBackOffPolicy backOffPolicy = new FixedBackOffPolicy();
        backOffPolicy.setBackOffPeriod(2000); // 2 seconds
        retryTemplate.setBackOffPolicy(backOffPolicy);
        
        SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy();
        retryPolicy.setMaxAttempts(3);
        retryTemplate.setRetryPolicy(retryPolicy);
        
        return retryTemplate;
    }
}
```

---

## 🔧 @Bean - Custom Bean Creation

### Definition:
`@Bean` is used to explicitly declare a single bean, rather than letting Spring do it automatically.

### When to Use:
- Third-party classes (you can't add annotations)
- Complex initialization logic
- Conditional bean creation
- Method-based bean configuration

### Real-World Example: Uber's Payment Gateway Integration
```java
@Configuration
public class PaymentGatewayConfig {
    
    @Value("${payment.stripe.secret-key}")
    private String stripeSecretKey;
    
    @Value("${payment.paypal.client-id}")
    private String paypalClientId;
    
    @Value("${payment.paypal.client-secret}")
    private String paypalClientSecret;
    
    // Stripe Payment Gateway
    @Bean("stripePaymentGateway")
    @ConditionalOnProperty(name = "payment.stripe.enabled", havingValue = "true")
    public PaymentGateway stripePaymentGateway() {
        StripeConfiguration config = StripeConfiguration.builder()
            .secretKey(stripeSecretKey)
            .apiVersion("2023-10-16")
            .timeout(Duration.ofSeconds(30))
            .maxRetries(3)
            .build();
            
        return new StripePaymentGateway(config);
    }
    
    // PayPal Payment Gateway
    @Bean("paypalPaymentGateway")
    @ConditionalOnProperty(name = "payment.paypal.enabled", havingValue = "true")
    public PaymentGateway paypalPaymentGateway() {
        PayPalConfiguration config = PayPalConfiguration.builder()
            .clientId(paypalClientId)
            .clientSecret(paypalClientSecret)
            .mode(PayPalMode.SANDBOX) // LIVE for production
            .timeout(Duration.ofSeconds(45))
            .build();
            
        return new PayPalPaymentGateway(config);
    }
    
    // Primary Payment Gateway (Strategy Pattern)
    @Bean
    @Primary
    public PaymentGateway primaryPaymentGateway(
            @Qualifier("stripePaymentGateway") PaymentGateway stripeGateway,
            @Qualifier("paypalPaymentGateway") PaymentGateway paypalGateway) {
        
        // Composite pattern for multiple gateways
        return new CompositePaymentGateway(List.of(stripeGateway, paypalGateway));
    }
    
    // Payment Processor with Circuit Breaker
    @Bean
    public PaymentProcessor paymentProcessor(PaymentGateway primaryGateway) {
        CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
            .failureRateThreshold(50)
            .waitDurationInOpenState(Duration.ofMillis(1000))
            .slidingWindowSize(2)
            .build();
            
        CircuitBreaker circuitBreaker = CircuitBreaker.of("paymentGateway", circuitBreakerConfig);
        
        return new ResilientPaymentProcessor(primaryGateway, circuitBreaker);
    }
    
    // Async Task Executor for Payment Processing
    @Bean("paymentTaskExecutor")
    public TaskExecutor paymentTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("payment-");
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(30);
        executor.initialize();
        return executor;
    }
}
```

---

## 🔍 @ComponentScan - Finding Your Beans

### Definition:
`@ComponentScan` tells Spring where to look for components (classes annotated with `@Component`, `@Service`, etc.).

### Default Behavior:
- Scans current package and all sub-packages
- Looks for stereotype annotations
- Creates bean definitions automatically

### Advanced Configuration:
```java
@Configuration
@ComponentScan(
    basePackages = {"com.uber.rides", "com.uber.payments", "com.uber.notifications"},
    excludeFilters = @ComponentScan.Filter(type = FilterType.REGEX, pattern = ".*Test.*"),
    includeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Repository.class)
)
public class UberMicroserviceConfig {
    // Configuration beans...
}
```

### Real-World Example: Airbnb's Modular Architecture
```java
@SpringBootApplication
@ComponentScan(
    basePackages = {
        "com.airbnb.listings",      // Property listings service
        "com.airbnb.bookings",      // Reservation service
        "com.airbnb.payments",      // Payment processing
        "com.airbnb.messaging",     // Host-Guest communication
        "com.airbnb.reviews",       // Review and rating system
        "com.airbnb.search"         // Search and filtering
    },
    excludeFilters = {
        @ComponentScan.Filter(type = FilterType.REGEX, pattern = ".*TestConfig.*"),
        @ComponentScan.Filter(type = FilterType.REGEX, pattern = ".*MockService.*")
    }
)
public class AirbnbApplication {
    public static void main(String[] args) {
        SpringApplication.run(AirbnbApplication.class, args);
    }
}

// Custom component scanning for specific environments
@Configuration
@Profile("development")
@ComponentScan(
    basePackages = "com.airbnb.dev.tools",
    includeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION, 
        classes = {DevTool.class, MockService.class}
    )
)
public class DevelopmentConfig {
    
    @Bean
    @Profile("development")
    public DataSource developmentDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .addScript("schema.sql")
            .addScript("test-data.sql")
            .build();
    }
}
```

---

## 📊 Bean Scopes - Managing Object Lifecycles

### Bean Scope Types:

| Scope | Description | Use Case |
|-------|-------------|----------|
| **Singleton** | One instance per Spring container | Services, Repositories |
| **Prototype** | New instance every time | Stateful objects |
| **Request** | One instance per HTTP request | Web-specific data |
| **Session** | One instance per HTTP session | User session data |
| **Application** | One instance per ServletContext | App-wide shared data |

---

## 🔄 Singleton vs Prototype Deep Dive

### Singleton Scope (Default)
**One instance shared across the entire application**

```java
@Configuration
public class SingletonExampleConfig {
    
    // Singleton Bean - Thread-Safe Service
    @Bean
    @Scope("singleton") // Default, can be omitted
    public UserService userService() {
        return new UserService();
    }
    
    // Connection Pool - Perfect for Singleton
    @Bean
    public ConnectionPool connectionPool() {
        HikariDataSource pool = new HikariDataSource();
        pool.setMaximumPoolSize(20);
        pool.setMinimumIdle(5);
        return pool;
    }
}

// Usage Example: Facebook User Service
@Service
@Scope("singleton") // Default scope
public class FacebookUserService {
    
    private final UserRepository userRepository;
    private final CacheManager cacheManager;
    
    // Thread-safe because it's stateless
    public FacebookUserService(UserRepository userRepository, CacheManager cacheManager) {
        this.userRepository = userRepository;
        this.cacheManager = cacheManager;
    }
    
    public User findUserById(Long userId) {
        // This method is thread-safe
        String cacheKey = "user:" + userId;
        User cachedUser = cacheManager.get(cacheKey, User.class);
        
        if (cachedUser != null) {
            return cachedUser;
        }
        
        User user = userRepository.findById(userId);
        cacheManager.put(cacheKey, user);
        return user;
    }
}
```

### Prototype Scope
**New instance created every time bean is requested**

```java
@Configuration
public class PrototypeExampleConfig {
    
    // Prototype Bean - Stateful Object
    @Bean
    @Scope("prototype")
    public ShoppingCart shoppingCart() {
        return new ShoppingCart();
    }
    
    // Request Processor - New instance per request
    @Bean
    @Scope("prototype")
    public PaymentRequestProcessor paymentProcessor() {
        return new PaymentRequestProcessor();
    }
}

// Usage Example: Amazon Shopping Cart
@Component
@Scope("prototype") // New instance for each user
public class AmazonShoppingCart {
    
    private List<CartItem> items = new ArrayList<>();
    private BigDecimal totalAmount = BigDecimal.ZERO;
    private String userId;
    private LocalDateTime createdAt;
    
    public AmazonShoppingCart() {
        this.createdAt = LocalDateTime.now();
        System.out.println("New shopping cart created at: " + createdAt);
    }
    
    public void addItem(Product product, int quantity) {
        CartItem item = new CartItem(product, quantity);
        items.add(item);
        recalculateTotal();
    }
    
    public void removeItem(Long productId) {
        items.removeIf(item -> item.getProduct().getId().equals(productId));
        recalculateTotal();
    }
    
    public BigDecimal checkout() {
        if (items.isEmpty()) {
            throw new EmptyCartException("Cannot checkout empty cart");
        }
        
        // Process checkout logic
        BigDecimal finalAmount = calculateFinalAmount();
        
        // Clear cart after checkout
        items.clear();
        totalAmount = BigDecimal.ZERO;
        
        return finalAmount;
    }
    
    private void recalculateTotal() {
        totalAmount = items.stream()
            .map(item -> item.getProduct().getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
    
    private BigDecimal calculateFinalAmount() {
        // Apply discounts, taxes, shipping
        BigDecimal discount = applyDiscounts();
        BigDecimal tax = calculateTax();
        BigDecimal shipping = calculateShipping();
        
        return totalAmount.subtract(discount).add(tax).add(shipping);
    }
    
    // Getter methods...
    public List<CartItem> getItems() { return new ArrayList<>(items); }
    public BigDecimal getTotalAmount() { return totalAmount; }
    public String getUserId() { return userId; }
    public void setUserId(String userId) { this.userId = userId; }
}

// Service that uses Prototype beans
@Service
public class CartService {
    
    @Autowired
    private ApplicationContext applicationContext;
    
    public AmazonShoppingCart createCartForUser(String userId) {
        // Each call creates a new instance
        AmazonShoppingCart cart = applicationContext.getBean(AmazonShoppingCart.class);
        cart.setUserId(userId);
        return cart;
    }
    
    // Alternative: Using @Lookup for method injection
    @Lookup
    public AmazonShoppingCart createNewCart() {
        return null; // Spring will override this method
    }
}
```

---

## 🌐 Web Scopes - Request & Session

### Request Scope Example: Google Analytics Tracking
```java
@Configuration
@EnableWebMvc
public class WebScopeConfig {
    
    // Request Scope - One per HTTP request
    @Bean
    @Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
    public AnalyticsTracker analyticsTracker() {
        return new AnalyticsTracker();
    }
    
    // Session Scope - One per user session
    @Bean
    @Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
    public UserSession userSession() {
        return new UserSession();
    }
}

// Request-scoped bean for tracking
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class GoogleAnalyticsTracker {
    
    private String requestId;
    private String userAgent;
    private String ipAddress;
    private LocalDateTime requestTime;
    private List<String> events = new ArrayList<>();
    
    @PostConstruct
    public void initialize() {
        this.requestId = UUID.randomUUID().toString();
        this.requestTime = LocalDateTime.now();
        System.out.println("Analytics tracker created for request: " + requestId);
    }
    
    public void trackEvent(String event, Map<String, Object> properties) {
        String eventData = String.format("[%s] %s: %s", 
            requestTime.format(DateTimeFormatter.ISO_LOCAL_TIME), event, properties);
        events.add(eventData);
    }
    
    public void trackPageView(String page) {
        trackEvent("page_view", Map.of("page", page, "timestamp", System.currentTimeMillis()));
    }
    
    @PreDestroy
    public void sendAnalytics() {
        // Send all tracked events to analytics service
        System.out.println("Sending " + events.size() + " events for request: " + requestId);
        // Implementation to send to Google Analytics API
    }
    
    // Getters and setters...
}

// Session-scoped bean for user state
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class UserSession {
    
    private String sessionId;
    private Long userId;
    private String username;
    private LocalDateTime loginTime;
    private Map<String, Object> preferences = new HashMap<>();
    private List<String> visitedPages = new ArrayList<>();
    
    @PostConstruct
    public void initialize() {
        this.sessionId = UUID.randomUUID().toString();
        this.loginTime = LocalDateTime.now();
        System.out.println("User session created: " + sessionId);
    }
    
    public void setUserInfo(Long userId, String username) {
        this.userId = userId;
        this.username = username;
    }
    
    public void addVisitedPage(String page) {
        visitedPages.add(page);
        // Keep only last 50 pages
        if (visitedPages.size() > 50) {
            visitedPages.remove(0);
        }
    }
    
    public void setPreference(String key, Object value) {
        preferences.put(key, value);
    }
    
    @PreDestroy
    public void cleanup() {
        System.out.println("Session ended for user: " + username + " (Session: " + sessionId + ")");
        // Cleanup logic, save session data, etc.
    }
    
    // Getters...
    public String getSessionId() { return sessionId; }
    public Long getUserId() { return userId; }
    public String getUsername() { return username; }
    public List<String> getVisitedPages() { return new ArrayList<>(visitedPages); }
}
```

---

## 🎯 Complete Real-World Example: LinkedIn's Post Processing System

```java
// Configuration for LinkedIn's post processing
@Configuration
@ComponentScan(basePackages = "com.linkedin.posts")
public class LinkedInPostConfig {
    
    // Singleton: Post validation service (stateless)
    @Bean
    public PostValidationService postValidationService() {
        return new PostValidationService();
    }
    
    // Prototype: Post processor for each post (stateful)
    @Bean
    @Scope("prototype")
    public PostProcessor postProcessor() {
        return new PostProcessor();
    }
    
    // Singleton: Notification sender (thread-safe)
    @Bean
    public NotificationSender notificationSender() {
        return new LinkedInNotificationSender();
    }
    
    // Request Scope: Request context for each API call
    @Bean
    @Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
    public RequestContext requestContext() {
        return new RequestContext();
    }
}

// Controller handling post creation
@RestController
@RequestMapping("/api/posts")
public class PostController {
    
    @Autowired
    private PostService postService;
    
    @Autowired
    private RequestContext requestContext; // Request-scoped
    
    @PostMapping
    public ResponseEntity<PostResponse> createPost(@RequestBody PostRequest request, HttpServletRequest httpRequest) {
        
        // Set request context (injected as proxy)
        requestContext.setUserId(request.getUserId());
        requestContext.setIpAddress(httpRequest.getRemoteAddr());
        requestContext.setUserAgent(httpRequest.getHeader("User-Agent"));
        
        PostResponse response = postService.createPost(request);
        return ResponseEntity.ok(response);
    }
}

// Service using different scoped beans
@Service
public class PostService {
    
    @Autowired
    private PostValidationService validationService; // Singleton
    
    @Autowired
    private ApplicationContext applicationContext;
    
    @Autowired
    private RequestContext requestContext; // Request-scoped proxy
    
    public PostResponse createPost(PostRequest request) {
        
        // Use singleton service
        validationService.validate(request);
        
        // Create new prototype bean for each post
        PostProcessor processor = applicationContext.getBean(PostProcessor.class);
        processor.setRequestContext(requestContext);
        
        // Process the post
        ProcessedPost processedPost = processor.process(request);
        
        // Save and return response
        Post savedPost = savePost(processedPost);
        return new PostResponse(savedPost.getId(), "Post created successfully");
    }
    
    private Post savePost(ProcessedPost processedPost) {
        // Implementation to save post
        return new Post();
    }
}

// Prototype-scoped post processor (stateful)
@Component
@Scope("prototype")
public class PostProcessor {
    
    private RequestContext requestContext;
    private List<String> processingSteps = new ArrayList<>();
    private LocalDateTime startTime;
    
    public PostProcessor() {
        this.startTime = LocalDateTime.now();
        System.out.println("New PostProcessor created at: " + startTime);
    }
    
    public void setRequestContext(RequestContext requestContext) {
        this.requestContext = requestContext;
    }
    
    public ProcessedPost process(PostRequest request) {
        
        processingSteps.add("Started processing");
        
        // Extract mentions
        List<String> mentions = extractMentions(request.getContent());
        processingSteps.add("Extracted " + mentions.size() + " mentions");
        
        // Extract hashtags
        List<String> hashtags = extractHashtags(request.getContent());
        processingSteps.add("Extracted " + hashtags.size() + " hashtags");
        
        // Analyze sentiment
        SentimentScore sentiment = analyzeSentiment(request.getContent());
        processingSteps.add("Analyzed sentiment: " + sentiment.getScore());
        
        // Generate preview
        String preview = generatePreview(request.getContent());
        processingSteps.add("Generated preview");
        
        // Create processed post
        ProcessedPost processedPost = new ProcessedPost();
        processedPost.setOriginalContent(request.getContent());
        processedPost.setMentions(mentions);
        processedPost.setHashtags(hashtags);
        processedPost.setSentiment(sentiment);
        processedPost.setPreview(preview);
        processedPost.setProcessingSteps(new ArrayList<>(processingSteps));
        processedPost.setProcessingTime(Duration.between(startTime, LocalDateTime.now()));
        
        // Add request context information
        processedPost.setUserId(requestContext.getUserId());
        processedPost.setIpAddress(requestContext.getIpAddress());
        
        System.out.println("Post processed in " + processedPost.getProcessingTime().toMillis() + "ms");
        
        return processedPost;
    }
    
    private List<String> extractMentions(String content) {
        // Extract @mentions from content
        return List.of("@john", "@jane");
    }
    
    private List<String> extractHashtags(String content) {
        // Extract #hashtags from content
        return List.of("#technology", "#career");
    }
    
    private SentimentScore analyzeSentiment(String content) {
        // Analyze sentiment using ML service
        return new SentimentScore(0.8); // Positive sentiment
    }
    
    private String generatePreview(String content) {
        return content.length() > 100 ? content.substring(0, 100) + "..." : content;
    }
}

// Request-scoped context
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestContext {
    
    private String requestId;
    private Long userId;
    private String ipAddress;
    private String userAgent;
    private LocalDateTime requestTime;
    
    @PostConstruct
    public void initialize() {
        this.requestId = UUID.randomUUID().toString();
        this.requestTime = LocalDateTime.now();
    }
    
    // Getters and setters...
    public String getRequestId() { return requestId; }
    public Long getUserId() { return userId; }
    public void setUserId(Long userId) { this.userId = userId; }
    public String getIpAddress() { return ipAddress; }
    public void setIpAddress(String ipAddress) { this.ipAddress = ipAddress; }
    public String getUserAgent() { return userAgent; }
    public void setUserAgent(String userAgent) { this.userAgent = userAgent; }
}
```

---

## 🎯 Interview Questions & Answers

### Q1: "What's the difference between @Component and @Bean?"
**Answer:**
- `@Component`: Class-level annotation, Spring creates bean automatically through component scanning
- `@Bean`: Method-level annotation in `@Configuration` class, you control bean creation manually
- **Use @Component for your own classes, @Bean for third-party classes or complex initialization**

### Q2: "When would you use Prototype scope over Singleton?"
**Answer:**
- **Singleton**: Stateless services, repositories, configurations (99% of cases)
- **Prototype**: Stateful objects like shopping carts, form data, user sessions
- **Key insight**: Prototype creates new instance every time, Singleton reuses same instance

### Q3: "What are the problems with Prototype scope?"
**Answer:**
1. **Memory Issues**: Objects not garbage collected if held by singletons
2. **Performance**: Creating new instances is expensive
3. **Dependency Issues**: Singleton beans can't inject prototype beans directly
4. **Solution**: Use `@Lookup` or `ApplicationContext.getBean()`

### Q4: "How do you inject Prototype bean into Singleton?"
**Answer:**
```java
@Service
public class SingletonService {
    @Autowired
    private ApplicationContext context;
    
    public void doSomething() {
        PrototypeBean bean = context.getBean(PrototypeBean.class);
        // Use fresh instance
    }
    
    // Alternative: Method injection
    @Lookup
    public PrototypeBean getPrototypeBean() {
        return null; // Spring overrides this
    }
}
```

### Q5: "What is @Primary and @Qualifier?"
**Answer:**
- `@Primary`: Default bean when multiple beans of same type exist
- `@Qualifier`: Specify which bean to inject by name
```java
@Bean @Primary
public PaymentService stripePayment() { }

@Bean
public PaymentService paypalPayment() { }

@Autowired @Qualifier("paypalPayment")
private PaymentService paymentService;
```

---

## 🚀 Industry Best Practices

### 1. **Configuration Organization**
```java
// ❌ DON'T: Everything in one config
@Configuration
public class AppConfig {
    // 100+ @Bean methods
}

// ✅ DO: Separate by concern
@Configuration
public class DatabaseConfig { }

@Configuration  
public class CacheConfig { }

@Configuration
public class SecurityConfig { }
```

### 2. **Bean Naming Conventions**
```java
@Bean("userServicePrimary")
public UserService primaryUserService() { }

@Bean("userServiceBackup") 
public UserService backupUserService() { }
```

### 3. **Conditional Bean Creation**
```java
@Bean
@ConditionalOnProperty(name = "feature.advanced-search", havingValue = "true")
public AdvancedSearchService advancedSearchService() {
    return new ElasticsearchService();
}

@Bean
@ConditionalOnMissingBean(AdvancedSearchService.class)
public AdvancedSearchService basicSearchService() {
    return new BasicSearchService();
}
```

