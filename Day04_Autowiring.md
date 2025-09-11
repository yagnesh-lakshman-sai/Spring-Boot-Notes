# 📅 Day 4: Autowiring & Dependency Resolution

## 🎯 Today Topics
- Master `@Autowired` annotation and its variants
- Understand `@Qualifier` for precise dependency injection
- Learn Constructor vs Field vs Setter injection
- Solve circular dependency problems
- Real-world dependency injection patterns from tech giants

---

## 🔌 Understanding Autowiring

### What is Autowiring?
**Autowiring** is Spring's mechanism to automatically inject dependencies into beans without explicit configuration. It's the "magic" that makes Dependency Injection seamless.

### How Spring Resolves Dependencies:
1. **By Type** → Look for beans matching the required type
2. **By Name** → If multiple beans of same type, match by name
3. **By Qualifier** → Use `@Qualifier` for explicit selection
4. **By Primary** → Use `@Primary` bean if multiple candidates exist

---

## 🎯 @Autowired - The Dependency Injection Powerhouse

### Definition:
`@Autowired` tells Spring to automatically inject a dependency. Spring will find the appropriate bean and inject it.

### Autowiring Modes:
- **required = true** (default): Injection fails if no bean found
- **required = false**: Injection is optional, null if no bean found

### Real-World Example: Google Search Service Architecture
```java
// Main search service - heart of Google Search
@Service
public class GoogleSearchService {
    
    // Multiple dependencies autowired
    @Autowired
    private IndexingService indexingService;
    
    @Autowired
    private RankingService rankingService;
    
    @Autowired
    private CacheService cacheService;
    
    @Autowired
    private AnalyticsService analyticsService;
    
    @Autowired
    private SecurityService securityService;
    
    @Autowired(required = false) // Optional dependency
    private SpellCheckService spellCheckService;
    
    public SearchResponse search(SearchRequest request) {
        
        // 1. Security validation
        securityService.validateRequest(request);
        
        // 2. Check cache first
        String cacheKey = generateCacheKey(request);
        SearchResponse cachedResult = cacheService.get(cacheKey, SearchResponse.class);
        
        if (cachedResult != null) {
            analyticsService.trackCacheHit(request);
            return cachedResult;
        }
        
        // 3. Spell check (optional)
        if (spellCheckService != null) {
            request = spellCheckService.correctQuery(request);
        }
        
        // 4. Get documents from index
        List<Document> documents = indexingService.searchDocuments(request.getQuery());
        
        // 5. Apply ranking algorithm
        List<Document> rankedDocuments = rankingService.rankDocuments(documents, request);
        
        // 6. Build response
        SearchResponse response = SearchResponse.builder()
            .query(request.getQuery())
            .results(rankedDocuments)
            .totalResults(documents.size())
            .executionTime(calculateExecutionTime())
            .build();
        
        // 7. Cache result
        cacheService.put(cacheKey, response, Duration.ofMinutes(15));
        
        // 8. Track analytics
        analyticsService.trackSearch(request, response);
        
        return response;
    }
    
    private String generateCacheKey(SearchRequest request) {
        return "search:" + request.getQuery().hashCode() + 
               ":" + request.getFilters().hashCode() +
               ":" + request.getLanguage();
    }
    
    private Duration calculateExecutionTime() {
        // Implementation to calculate search execution time
        return Duration.ofMillis(150);
    }
}
```

---

## 🎯 Interview Questions & Answers

### Q1: "What's the difference between @Autowired, @Inject, and @Resource?"
**Answer:**
- `@Autowired`: Spring-specific, injection by type, supports required=false
- `@Inject`: JSR-330 standard, injection by type, no required attribute
- `@Resource`: JSR-250 standard, injection by name first, then by type
- **Best Practice**: Use @Autowired with Spring Boot for consistency

### Q2: "Why is Constructor Injection recommended over Field Injection?"
**Answer:**
1. **Immutability**: Dependencies can be final
2. **Testability**: Easy to create instances without Spring
3. **Fail-Fast**: Missing dependencies detected at startup
4. **Clear Dependencies**: Constructor shows all required dependencies
5. **Circular Dependency Detection**: Detected at startup, not runtime

### Q3: "How do you resolve circular dependencies?"
**Answer:**
1. **Best Solution**: Redesign architecture to remove cycle
2. **@Lazy**: Delay injection until first use
3. **ApplicationContextAware**: Manual bean lookup
4. **@PostConstruct**: Initialize dependencies after construction
5. **Extract Common Logic**: Create shared service to break cycle

### Q4: "What happens if Spring can't find a bean to autowire?"
**Answer:**
- **required=true** (default): `NoSuchBeanDefinitionException` thrown at startup
- **required=false**: `null` injected, application continues
- **Optional<T>**: Empty Optional injected
- **@Autowired List<T>**: Empty list injected

### Q5: "How does @Qualifier work with @Primary?"
**Answer:**
- `@Primary`: Default bean when multiple candidates exist
- `@Qualifier`: Explicit bean selection, overrides @Primary
- If both present: @Qualifier takes precedence
- Without either: Spring throws `NoUniqueBeanDefinitionException`

### Q6: "What's the performance impact of different injection types?"
**Answer:**
- **Constructor**: Fastest (resolved once at startup)
- **Field**: Slightly slower (reflection-based)
- **Setter**: Slowest (method call + potential validation)
- **Impact**: Negligible in most applications, design benefits outweigh performance

---

## 🔧 Advanced Autowiring Patterns

### 1. Conditional Autowiring
```java
@Service
public class PaymentService {
    
    private final List<PaymentProcessor> paymentProcessors;
    private final PaymentProcessor defaultProcessor;
    
    public PaymentService(
            List<PaymentProcessor> paymentProcessors, // Injects ALL beans of this type
            @Qualifier("primaryProcessor") PaymentProcessor defaultProcessor) {
        
        this.paymentProcessors = paymentProcessors;
        this.defaultProcessor = defaultProcessor;
    }
    
    public PaymentResult processPayment(PaymentRequest request) {
        // Try each processor until one succeeds
        for (PaymentProcessor processor : paymentProcessors) {
            if (processor.supports(request.getPaymentMethod())) {
                try {
                    return processor.process(request);
                } catch (PaymentException e) {
                    // Continue to next processor
                }
            }
        }
        
        // Fallback to default processor
        return defaultProcessor.process(request);
    }
}
```

### 2. Optional Dependencies
```java
@Service
public class EmailService {
    
    private final EmailProvider emailProvider;
    private final Optional<SmsProvider> smsProvider;
    private final List<NotificationPlugin> plugins;
    
    public EmailService(
            EmailProvider emailProvider,
            Optional<SmsProvider> smsProvider, // Optional dependency
            List<NotificationPlugin> plugins) { // List can be empty
        
        this.emailProvider = emailProvider;
        this.smsProvider = smsProvider;
        this.plugins = plugins;
    }
    
    public void sendNotification(Notification notification) {
        // Always send email
        emailProvider.send(notification.getEmailContent());
        
        // Send SMS if provider available
        smsProvider.ifPresent(provider -> 
            provider.send(notification.getSmsContent())
        );
        
        // Execute all plugins
        plugins.forEach(plugin -> plugin.execute(notification));
    }
}
```

### 3. Generic Type Injection
```java
@Component
public class CacheService<T> {
    
    @Autowired
    private RedisTemplate<String, T> redisTemplate;
    
    public void cache(String key, T value) {
        redisTemplate.opsForValue().set(key, value);
    }
    
    public T get(String key) {
        return redisTemplate.opsForValue().get(key);
    }
}

// Usage
@Service
public class UserService {
    
    @Autowired
    private CacheService<User> userCache; // Type-safe cache
    
    public User findUser(Long id) {
        return userCache.get("user:" + id);
    }
}
```

---

## 🚀 Industry Best Practices

### 1. **Dependency Organization**
```java
// ✅ Good: Clear, organized dependencies
@Service
public class OrderService {
    
    // Core dependencies
    private final OrderRepository orderRepository;
    private final CustomerService customerService;
    
    // External services
    private final PaymentService paymentService;
    private final ShippingService shippingService;
    
    // Optional services
    private final Optional<RecommendationService> recommendationService;
    
    public OrderService(
            OrderRepository orderRepository,
            CustomerService customerService,
            PaymentService paymentService,
            ShippingService shippingService,
            Optional<RecommendationService> recommendationService) {
        // Constructor...
    }
}
```

### 2. **Profile-Based Injection**
```java
@Service
@Profile("production")
public class ProductionEmailService implements EmailService {
    // Production email implementation
}

@Service
@Profile("development")
public class MockEmailService implements EmailService {
    // Mock implementation for testing
}
```

### 3. **Configuration Properties Injection**
```java
@ConfigurationProperties(prefix = "payment")
@Component
public class PaymentConfiguration {
    
    private String stripeApiKey;
    private String paypalClientId;
    private Duration timeout;
    private boolean retryEnabled;
    
    // Getters and setters...
}

@Service
public class PaymentService {
    
    private final PaymentConfiguration config;
    
    public PaymentService(PaymentConfiguration config) {
        this.config = config;
    }
    
    // Use configuration throughout service...
}
```

---

## 🎯 Tomorrow's Preview: Day 5
We'll explore **Application Properties** with:
- `@Value` annotation for property injection
- `@ConfigurationProperties` for type-safe configuration
- `application.properties` vs `application.yml`
- Environment-specific configurations

---

## 📝 Quick Recap
✅ **@Autowired**: Spring's dependency injection mechanism  
✅ **Constructor Injection**: Recommended approach (immutable, testable)  
✅ **@Qualifier**: Precise bean selection when multiple candidates exist  
✅ **Circular Dependencies**: Best solved by architectural redesign  
✅ **Optional Dependencies**: Use Optional<T> or required=false  

**Commit Message:** `🔌 Day 4: Mastered autowiring patterns with real-world dependency injection examples`

// Supporting services
@Service
public class IndexingService {
    
    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;
    
    @Autowired
    private DocumentProcessor documentProcessor;
    
    public List<Document> searchDocuments(String query) {
        // Complex search logic using Elasticsearch
        SearchQuery searchQuery = new NativeSearchQueryBuilder()
            .withQuery(QueryBuilders.multiMatchQuery(query)
                .field("title", 2.0f)
                .field("content", 1.0f)
                .field("tags", 1.5f))
            .withPageable(PageRequest.of(0, 100))
            .build();
            
        return elasticsearchTemplate.queryForList(searchQuery, Document.class);
    }
}

@Service
public class RankingService {
    
    @Autowired
    private PageRankCalculator pageRankCalculator;
    
    @Autowired
    private RelevanceScorer relevanceScorer;
    
    @Autowired
    private UserBehaviorAnalyzer userBehaviorAnalyzer;
    
    public List<Document> rankDocuments(List<Document> documents, SearchRequest request) {
        
        return documents.stream()
            .peek(doc -> {
                // Calculate multiple ranking signals
                double pageRankScore = pageRankCalculator.calculate(doc);
                double relevanceScore = relevanceScorer.calculateRelevance(doc, request.getQuery());
                double userSignalScore = userBehaviorAnalyzer.getUserEngagementScore(doc);
                
                // Combine scores with weights
                double finalScore = (pageRankScore * 0.4) + 
                                  (relevanceScore * 0.4) + 
                                  (userSignalScore * 0.2);
                
                doc.setRankingScore(finalScore);
            })
            .sorted((doc1, doc2) -> Double.compare(doc2.getRankingScore(), doc1.getRankingScore()))
            .collect(Collectors.toList());
    }
}
```

---

## 🏗️ Three Types of Dependency Injection

### 1. Constructor Injection (RECOMMENDED ✅)

**Why it's the best:**
- Immutable dependencies
- Fail-fast on startup
- Easy to test
- Forces good design (too many dependencies = code smell)

```java
// Tesla's Vehicle Control System
@Service
public class TeslaVehicleControlService {
    
    private final BatteryManagementSystem batterySystem;
    private final MotorControlUnit motorControl;
    private final AutopilotService autopilotService;
    private final ChargingService chargingService;
    private final TelematicsService telematicsService;
    
    // Constructor injection - Spring 4.3+ doesn't need @Autowired
    public TeslaVehicleControlService(
            BatteryManagementSystem batterySystem,
            MotorControlUnit motorControl,
            AutopilotService autopilotService,
            ChargingService chargingService,
            TelematicsService telematicsService) {
        
        this.batterySystem = batterySystem;
        this.motorControl = motorControl;
        this.autopilotService = autopilotService;
        this.chargingService = chargingService;
        this.telematicsService = telematicsService;
        
        // Validation at construction time
        validateSystemIntegrity();
    }
    
    public VehicleStatus startVehicle(String vehicleId) {
        
        // All dependencies are guaranteed to be available
        BatteryStatus batteryStatus = batterySystem.checkBatteryHealth();
        
        if (batteryStatus.getChargeLevel() < 5.0) {
            throw new InsufficientBatteryException("Cannot start vehicle with battery below 5%");
        }
        
        // Initialize motor systems
        motorControl.initializeMotors();
        
        // Start autopilot systems
        autopilotService.initializeSensors();
        
        // Connect to Tesla network
        telematicsService.establishConnection(vehicleId);
        
        return VehicleStatus.builder()
            .vehicleId(vehicleId)
            .status("READY")
            .batteryLevel(batteryStatus.getChargeLevel())
            .range(calculateRange(batteryStatus))
            .autopilotReady(autopilotService.isSystemReady())
            .build();
    }
    
    public ChargingSession startCharging(String vehicleId, ChargingStation station) {
        
        // Validate charging compatibility
        if (!chargingService.isCompatible(station)) {
            throw new IncompatibleChargingStationException("Station not compatible with vehicle");
        }
        
        // Optimize charging based on battery condition
        BatteryStatus batteryStatus = batterySystem.checkBatteryHealth();
        ChargingProfile profile = chargingService.optimizeChargingProfile(batteryStatus);
        
        return chargingService.startCharging(vehicleId, station, profile);
    }
    
    private void validateSystemIntegrity() {
        // Ensure all critical systems are operational
        if (!batterySystem.performSelfTest()) {
            throw new SystemIntegrityException("Battery management system failed self-test");
        }
        
        if (!motorControl.performSelfTest()) {
            throw new SystemIntegrityException("Motor control unit failed self-test");
        }
        
        // Additional validations...
    }
    
    private double calculateRange(BatteryStatus batteryStatus) {
        // Complex range calculation based on battery health, temperature, etc.
        double baseRange = 400; // km
        double batteryHealthFactor = batteryStatus.getHealthPercentage() / 100.0;
        double temperatureFactor = calculateTemperatureFactor();
        
        return baseRange * batteryHealthFactor * temperatureFactor;
    }
    
    private double calculateTemperatureFactor() {
        // Temperature impact on range calculation
        return 0.95; // Simplified
    }
}

// Supporting services with constructor injection
@Service
public class BatteryManagementSystem {
    
    private final TemperatureSensor temperatureSensor;
    private final VoltageSensor voltageSensor;
    private final CurrentSensor currentSensor;
    
    public BatteryManagementSystem(
            TemperatureSensor temperatureSensor,
            VoltageSensor voltageSensor, 
            CurrentSensor currentSensor) {
        this.temperatureSensor = temperatureSensor;
        this.voltageSensor = voltageSensor;
        this.currentSensor = currentSensor;
    }
    
    public BatteryStatus checkBatteryHealth() {
        double temperature = temperatureSensor.readTemperature();
        double voltage = voltageSensor.readVoltage();
        double current = currentSensor.readCurrent();
        
        // Complex battery health algorithm
        double healthPercentage = calculateHealthPercentage(temperature, voltage, current);
        double chargeLevel = calculateChargeLevel(voltage);
        
        return BatteryStatus.builder()
            .healthPercentage(healthPercentage)
            .chargeLevel(chargeLevel)
            .temperature(temperature)
            .voltage(voltage)
            .current(current)
            .build();
    }
    
    public boolean performSelfTest() {
        // Comprehensive battery system self-test
        try {
            temperatureSensor.selfTest();
            voltageSensor.selfTest();
            currentSensor.selfTest();
            return true;
        } catch (SensorException e) {
            return false;
        }
    }
    
    private double calculateHealthPercentage(double temperature, double voltage, double current) {
        // Sophisticated algorithm considering multiple factors
        return 95.5; // Simplified
    }
    
    private double calculateChargeLevel(double voltage) {
        // Convert voltage to charge percentage
        return Math.min(100.0, Math.max(0.0, (voltage - 300) / 100 * 100));
    }
}
```

### 2. Field Injection (NOT RECOMMENDED ❌)

```java
// ❌ DON'T DO THIS - Field Injection Problems
@Service
public class BadUserService {
    
    @Autowired
    private UserRepository userRepository; // Hard to test!
    
    @Autowired
    private EmailService emailService; // Mutable!
    
    @Autowired
    private ValidationService validationService; // Hidden dependencies!
    
    // Problems:
    // 1. Can't create instance without Spring
    // 2. Dependencies are mutable
    // 3. Hard to write unit tests
    // 4. Circular dependencies not detected early
    // 5. Violates final field best practices
}
```

### 3. Setter Injection (RARELY USED)

```java
// Setter injection - only for optional dependencies
@Service
public class NotificationService {
    
    private EmailService emailService;
    private SmsService smsService;
    private PushNotificationService pushService;
    
    // Optional dependency - can work without it
    @Autowired(required = false)
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
    
    @Autowired(required = false)
    public void setSmsService(SmsService smsService) {
        this.smsService = smsService;
    }
    
    @Autowired
    public void setPushService(PushNotificationService pushService) {
        this.pushService = pushService;
    }
    
    public void sendNotification(String message, NotificationType type) {
        switch (type) {
            case EMAIL:
                if (emailService != null) {
                    emailService.send(message);
                }
                break;
            case SMS:
                if (smsService != null) {
                    smsService.send(message);
                }
                break;
            case PUSH:
                pushService.send(message); // Required service
                break;
        }
    }
}
```

---

## 🏷️ @Qualifier - Precise Dependency Selection

### When to Use @Qualifier:
- Multiple beans of same type exist
- Need specific implementation
- Want to avoid `@Primary` conflicts

### Real-World Example: Netflix Content Delivery Network
```java
// Netflix has multiple CDN providers for global content delivery
@Configuration
public class NetflixCdnConfig {
    
    @Bean("awsCloudFront")
    public ContentDeliveryNetwork awsCloudFront() {
        return new AWSCloudFrontCDN();
    }
    
    @Bean("azureCdn")
    public ContentDeliveryNetwork azureCdn() {
        return new AzureCDN();
    }
    
    @Bean("cloudflareCdn")
    public ContentDeliveryNetwork cloudflareCdn() {
        return new CloudflareCDN();
    }
    
    @Bean("fastlyCdn")
    @Primary // Default CDN
    public ContentDeliveryNetwork fastlyCdn() {
        return new FastlyCDN();
    }
}

// Content streaming service using specific CDNs
@Service
public class NetflixStreamingService {
    
    private final ContentDeliveryNetwork primaryCdn;
    private final ContentDeliveryNetwork backupCdn;
    private final ContentDeliveryNetwork asiaCdn;
    private final ContentRepository contentRepository;
    private final UserLocationService locationService;
    
    public NetflixStreamingService(
            @Qualifier("fastlyCdn") ContentDeliveryNetwork primaryCdn,
            @Qualifier("awsCloudFront") ContentDeliveryNetwork backupCdn,
            @Qualifier("azureCdn") ContentDeliveryNetwork asiaCdn,
            ContentRepository contentRepository,
            UserLocationService locationService) {
        
        this.primaryCdn = primaryCdn;
        this.backupCdn = backupCdn;
        this.asiaCdn = asiaCdn;
        this.contentRepository = contentRepository;
        this.locationService = locationService;
    }
    
    public StreamingUrl getStreamingUrl(Long contentId, String userId) {
        
        // Get user location for CDN selection
        UserLocation location = locationService.getUserLocation(userId);
        Content content = contentRepository.findById(contentId);
        
        // Select appropriate CDN based on location
        ContentDeliveryNetwork selectedCdn = selectOptimalCdn(location);
        
        try {
            // Try primary CDN selection
            StreamingUrl url = selectedCdn.generateStreamingUrl(content, location);
            
            // Validate URL availability
            if (selectedCdn.isContentAvailable(content, location)) {
                return url;
            }
            
        } catch (CdnException e) {
            // Fallback to backup CDN
            return backupCdn.generateStreamingUrl(content, location);
        }
        
        throw new StreamingException("Unable to generate streaming URL for content: " + contentId);
    }
    
    private ContentDeliveryNetwork selectOptimalCdn(UserLocation location) {
        // Geographic CDN selection logic
        if (location.getContinent().equals("ASIA")) {
            return asiaCdn; // Azure has better performance in Asia
        } else if (location.getCountry().equals("US")) {
            return primaryCdn; // Fastly for US users
        } else {
            return backupCdn; // AWS CloudFront for others
        }
    }
    
    public void preloadContent(Long contentId, List<String> targetRegions) {
        Content content = contentRepository.findById(contentId);
        
        // Preload on multiple CDNs for popular content
        targetRegions.parallelStream().forEach(region -> {
            ContentDeliveryNetwork cdn = getCdnForRegion(region);
            try {
                cdn.preloadContent(content, region);
            } catch (Exception e) {
                // Log and continue with other regions
                System.err.println("Failed to preload content on CDN for region: " + region);
            }
        });
    }
    
    private ContentDeliveryNetwork getCdnForRegion(String region) {
        switch (region.toLowerCase()) {
            case "asia": return asiaCdn;
            case "us": return primaryCdn;
            case "europe": return backupCdn;
            default: return primaryCdn;
        }
    }
}

// CDN implementations
public interface ContentDeliveryNetwork {
    StreamingUrl generateStreamingUrl(Content content, UserLocation location);
    boolean isContentAvailable(Content content, UserLocation location);
    void preloadContent(Content content, String region);
}

@Component("fastlyCdn")
public class FastlyCDN implements ContentDeliveryNetwork {
    
    @Autowired
    private FastlyApiClient fastlyClient;
    
    @Override
    public StreamingUrl generateStreamingUrl(Content content, UserLocation location) {
        // Fastly-specific URL generation
        String baseUrl = "https://cdn.fastly.netflix.com";
        String token = fastlyClient.generateAccessToken(content.getId());
        
        return StreamingUrl.builder()
            .url(baseUrl + "/content/" + content.getId() + "?token=" + token)
            .cdnProvider("FASTLY")
            .expiresAt(LocalDateTime.now().plusHours(4))
            .build();
    }
    
    @Override
    public boolean isContentAvailable(Content content, UserLocation location) {
        return fastlyClient.checkContentAvailability(content.getId(), location.getRegion());
    }
    
    @Override
    public void preloadContent(Content content, String region) {
        fastlyClient.preloadContentToEdge(content.getId(), region);
    }
}

@Component("awsCloudFront")
public class AWSCloudFrontCDN implements ContentDeliveryNetwork {
    
    @Autowired
    private CloudFrontClient cloudFrontClient;
    
    @Override
    public StreamingUrl generateStreamingUrl(Content content, UserLocation location) {
        // AWS CloudFront signed URL generation
        String distributionDomain = "d123456abcdef8.cloudfront.net";
        String signedUrl = cloudFrontClient.generateSignedUrl(
            distributionDomain, 
            "/content/" + content.getId(),
            Duration.ofHours(6)
        );
        
        return StreamingUrl.builder()
            .url(signedUrl)
            .cdnProvider("AWS_CLOUDFRONT")
            .expiresAt(LocalDateTime.now().plusHours(6))
            .build();
    }
    
    @Override
    public boolean isContentAvailable(Content content, UserLocation location) {
        return cloudFrontClient.checkEdgeLocation(content.getId(), location.getClosestEdge());
    }
    
    @Override
    public void preloadContent(Content content, String region) {
        cloudFrontClient.createInvalidation(content.getId(), region);
    }
}
```

---

## 🔄 Solving Circular Dependencies

### What are Circular Dependencies?
When two or more beans depend on each other, creating a cycle.

```java
// ❌ CIRCULAR DEPENDENCY PROBLEM
@Service
public class OrderService {
    @Autowired
    private CustomerService customerService; // OrderService needs CustomerService
}

@Service
public class CustomerService {
    @Autowired
    private OrderService orderService; // CustomerService needs OrderService - CYCLE!
}
```

### Solutions for Circular Dependencies:

#### 1. Redesign Architecture (BEST SOLUTION ✅)
```java
// ✅ SOLUTION 1: Extract common functionality
@Service
public class OrderService {
    @Autowired
    private CustomerRepository customerRepository; // Direct repository access
    
    @Autowired
    private OrderValidationService validationService; // Separate validation
}

@Service
public class CustomerService {
    @Autowired
    private CustomerRepository customerRepository;
    
    @Autowired
    private OrderRepository orderRepository; // Direct repository access
}

// Shared validation logic
@Service
public class OrderValidationService {
    @Autowired
    private CustomerRepository customerRepository;
    
    @Autowired
    private OrderRepository orderRepository;
    
    public boolean validateOrder(Order order) {
        Customer customer = customerRepository.findById(order.getCustomerId());
        return customer != null && customer.isActive();
    }
}
```

#### 2. @Lazy Annotation
```java
// ✅ SOLUTION 2: Lazy initialization
@Service
public class OrderService {
    @Autowired
    @Lazy // Delays injection until first use
    private CustomerService customerService;
}

@Service
public class CustomerService {
    @Autowired
    private OrderService orderService;
}
```

#### 3. ApplicationContextAware
```java
// ✅ SOLUTION 3: Manual lookup
@Service
public class OrderService implements ApplicationContextAware {
    
    private ApplicationContext applicationContext;
    
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }
    
    public void processOrder(Order order) {
        // Get CustomerService when needed
        CustomerService customerService = applicationContext.getBean(CustomerService.class);
        Customer customer = customerService.findById(order.getCustomerId());
        // Process order...
    }
}
```

#### 4. @PostConstruct Initialization
```java
// ✅ SOLUTION 4: Post-construction setup
@Service
public class OrderService {
    @Autowired
    private CustomerService customerService;
    
    private OrderValidator orderValidator;
    
    @PostConstruct
    public void initialize() {
        // Set up dependencies after construction
        this.orderValidator = new OrderValidator(customerService);
    }
}
```

---

## 🏢 Complete Real-World Example: Amazon Order Processing System

```java
// Amazon's order processing with complex dependencies
@Configuration
public class AmazonOrderConfig {
    
    @Bean("defaultPaymentProcessor")
    @Primary
    public PaymentProcessor defaultPaymentProcessor() {
        return new StripePaymentProcessor();
    }
    
    @Bean("alternativePaymentProcessor")
    public PaymentProcessor alternativePaymentProcessor() {
        return new PayPalPaymentProcessor();
    }
    
    @Bean("premiumShippingService")
    public ShippingService premiumShippingService() {
        return new AmazonPrimeShippingService();
    }
    
    @Bean("standardShippingService")
    public ShippingService standardShippingService() {
        return new StandardShippingService();
    }
}

// Main order processing service
@Service
@Transactional
public class AmazonOrderProcessingService {
    
    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentProcessor primaryPaymentProcessor;
    private final PaymentProcessor backupPaymentProcessor;
    private final ShippingService premiumShipping;
    private final ShippingService standardShipping;
    private final NotificationService notificationService;
    private final RecommendationService recommendationService;
    
    // Constructor injection with qualifiers
    public AmazonOrderProcessingService(
            OrderRepository orderRepository,
            InventoryService inventoryService,
            @Qualifier("defaultPaymentProcessor") PaymentProcessor primaryPaymentProcessor,
            @Qualifier("alternativePaymentProcessor") PaymentProcessor backupPaymentProcessor,
            @Qualifier("premiumShippingService") ShippingService premiumShipping,
            @Qualifier("standardShippingService") ShippingService standardShipping,
            NotificationService notificationService,
            @Lazy RecommendationService recommendationService) { // Lazy to avoid circular dependency
        
        this.orderRepository = orderRepository;
        this.inventoryService = inventoryService;
        this.primaryPaymentProcessor = primaryPaymentProcessor;
        this.backupPaymentProcessor = backupPaymentProcessor;
        this.premiumShipping = premiumShipping;
        this.standardShipping = standardShipping;
        this.notificationService = notificationService;
        this.recommendationService = recommendationService;
    }
    
    public OrderResponse processOrder(OrderRequest request) {
        
        try {
            // 1. Validate order
            validateOrder(request);
            
            // 2. Check inventory availability
            List<InventoryItem> availableItems = inventoryService.checkAvailability(request.getItems());
            if (availableItems.size() != request.getItems().size()) {
                return handlePartialAvailability(request, availableItems);
            }
            
            // 3. Calculate pricing
            OrderPricing pricing = calculateOrderPricing(request);
            
            // 4. Process payment with fallback
            PaymentResult paymentResult = processPaymentWithFallback(request.getPaymentMethod(), pricing);
            
            if (!paymentResult.isSuccessful()) {
                throw new PaymentProcessingException("All payment methods failed");
            }
            
            // 5. Reserve inventory
            List<ReservationResult> reservations = inventoryService.reserveItems(request.getItems());
            
            // 6. Create order
            Order order = createOrder(request, pricing, paymentResult, reservations);
            Order savedOrder = orderRepository.save(order);
            
            // 7. Determine shipping method
            ShippingService selectedShippingService = selectShippingService(request);
            ShippingResult shippingResult = selectedShippingService.createShipment(savedOrder);
            
            // 8. Update order with shipping details
            savedOrder.setShippingTrackingNumber(shippingResult.getTrackingNumber());
            savedOrder.setEstimatedDeliveryDate(shippingResult.getEstimatedDelivery());
            orderRepository.save(savedOrder);
            
            // 9. Send notifications
            notificationService.sendOrderConfirmation(savedOrder);
            
            // 10. Generate recommendations asynchronously (Lazy-loaded service)
            CompletableFuture.runAsync(() -> {
                try {
                    recommendationService.generatePostOrderRecommendations(savedOrder);
                } catch (Exception e) {
                    // Log error but don't fail order processing
                    System.err.println("Failed to generate recommendations: " + e.getMessage());
                }
            });
            
            return OrderResponse.builder()
                .orderId(savedOrder.getId())
                .status("CONFIRMED")
                .totalAmount(pricing.getTotalAmount())
                .estimatedDelivery(shippingResult.getEstimatedDelivery())
                .trackingNumber(shippingResult.getTrackingNumber())
                .build();
                
        } catch (Exception e) {
            // Comprehensive error handling
            handleOrderProcessingError(request, e);
            throw new OrderProcessingException("Order processing failed", e);
        }
    }
    
    private PaymentResult processPaymentWithFallback(PaymentMethod paymentMethod, OrderPricing pricing) {
        
        // Try primary payment processor
        try {
            return primaryPaymentProcessor.processPayment(paymentMethod, pricing.getTotalAmount());
        } catch (PaymentException e) {
            // Log primary processor failure
            System.err.println("Primary payment processor failed: " + e.getMessage());
            
            // Try backup payment processor
            try {
                return backupPaymentProcessor.processPayment(paymentMethod, pricing.getTotalAmount());
            } catch (PaymentException backupException) {
                // Both processors failed
                throw new PaymentProcessingException("All payment processors failed", backupException);
            }
        }
    }
    
    private ShippingService selectShippingService(OrderRequest request) {
        // Business logic to select shipping service
        if (request.isPrimeCustomer() || request.getShippingOption() == ShippingOption.EXPEDITED) {
            return premiumShipping;
        }
        return standardShipping;
    }
    
    private void validateOrder(OrderRequest request) {
        if (request.getItems().isEmpty()) {
            throw new InvalidOrderException("Order must contain at least one item");
        }
        
        if (request.getCustomerId() == null) {
            throw new InvalidOrderException("Customer ID is required");
        }
        
        // Additional validations...
    }
    
    private OrderPricing calculateOrderPricing(OrderRequest request) {
        // Complex pricing calculation with discounts, taxes, shipping
        BigDecimal subtotal = request.getItems().stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
            
        BigDecimal discount = calculateDiscount(request);
        BigDecimal tax = calculateTax(subtotal, request.getShippingAddress());
        BigDecimal shipping = calculateShipping(request);
        
        return OrderPricing.builder()
            .subtotal(subtotal)
            .discount(discount)
            .tax(tax)
            .shipping(shipping)
            .totalAmount(subtotal.subtract(discount).add(tax).add(shipping))
            .build();
    }
    
    private Order createOrder(OrderRequest request, OrderPricing pricing, PaymentResult paymentResult, List<ReservationResult> reservations) {
        Order order = new Order();
        order.setCustomerId(request.getCustomerId());
        order.setItems(request.getItems());
        order.setSubtotal(pricing.getSubtotal());
        order.setTax(pricing.getTax());
        order.setShippingCost(pricing.getShipping());
        order.setDiscount(pricing.getDiscount());
        order.setTotalAmount(pricing.getTotalAmount());
        order.setPaymentTransactionId(paymentResult.getTransactionId());
        order.setStatus(OrderStatus.CONFIRMED);
        order.setShippingAddress(request.getShippingAddress());
        order.setCreatedDate(LocalDateTime.now());
        
        return order;
    }
    
    // Additional helper methods...
}

// Supporting services
@Service
public class RecommendationService {
    
    private final OrderRepository orderRepository;
    private final ProductRepository productRepository;
    private final UserBehaviorService userBehaviorService;
    
    public RecommendationService(
            OrderRepository orderRepository,
            ProductRepository productRepository,
            UserBehavior
