# 📅 Day 6: Profiles & Environment Management

## 🎯 Today Topics
- Master `@Profile` annotation for conditional beans
- Understand environment-specific configurations
- Learn profile activation strategies
- Real-world multi-environment deployment patterns
- Simple examples for immediate implementation

---

## 🌍 Understanding Spring Profiles

### What are Spring Profiles?
**Spring Profiles** provide a way to segregate parts of your application configuration and make it available only in certain environments. Think of profiles as different "modes" your application can run in.

### Why Use Profiles?
- **Environment Separation**: Different configs for dev/test/prod
- **Feature Toggling**: Enable/disable features per environment
- **Resource Optimization**: Different database/cache configs
- **Security Levels**: Different security settings per environment
- **Testing**: Mock services in test environments

---

## 🏷️ @Profile Annotation - Conditional Bean Creation

### Basic @Profile Usage:
```java
@Component
@Profile("development")
public class DevEmailService implements EmailService {
    // Development implementation
}

@Component
@Profile("production")
public class ProdEmailService implements EmailService {
    // Production implementation
}
```

### Simple Example: Database Configuration
```java
// Development Database - H2 In-Memory
@Configuration
@Profile("dev")
public class DevDatabaseConfig {
    
    @Bean
    public DataSource dataSource() {
        System.out.println("🔧 Creating Development Database (H2 In-Memory)");
        
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .addScript("classpath:schema.sql")
                .addScript("classpath:test-data.sql")
                .build();
    }
    
    @Bean
    public DatabaseHealthChecker databaseHealthChecker() {
        return new SimpleDatabaseHealthChecker("Development H2 Database");
    }
}

// Production Database - PostgreSQL
@Configuration
@Profile("prod")
public class ProdDatabaseConfig {
    
    @Value("${database.url}")
    private String url;
    
    @Value("${database.username}")
    private String username;
    
    @Value("${database.password}")
    private String password;
    
    @Bean
    public DataSource dataSource() {
        System.out.println("🚀 Creating Production Database (PostgreSQL)");
        
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        dataSource.setMaximumPoolSize(50);
        dataSource.setMinimumIdle(10);
        dataSource.setConnectionTimeout(30000);
        return dataSource;
    }
    
    @Bean
    public DatabaseHealthChecker databaseHealthChecker() {
        return new ProductionDatabaseHealthChecker("Production PostgreSQL");
    }
}

// Test Database - H2 with Different Config
@Configuration
@Profile("test")
public class TestDatabaseConfig {
    
    @Bean
    public DataSource dataSource() {
        System.out.println("🧪 Creating Test Database (H2 File-based)");
        
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .setName("testdb")
                .addScript("classpath:schema.sql")
                .build();
    }
    
    @Bean
    public DatabaseHealthChecker databaseHealthChecker() {
        return new TestDatabaseHealthChecker("Test H2 Database");
    }
}
```

### Service Implementations Using Profiles:
```java
// Email Service Interface
public interface EmailService {
    void sendEmail(String to, String subject, String body);
    String getServiceInfo();
}

// Development - Console Logging
@Service
@Profile("dev")
public class DevEmailService implements EmailService {
    
    @Override
    public void sendEmail(String to, String subject, String body) {
        System.out.println("📧 [DEV MODE] Email would be sent:");
        System.out.println("  To: " + to);
        System.out.println("  Subject: " + subject);
        System.out.println("  Body: " + body);
        System.out.println("  Status: ✅ Simulated send successful");
    }
    
    @Override
    public String getServiceInfo() {
        return "Development Email Service - Console Output Only";
    }
}

// Production - Real SMTP
@Service
@Profile("prod")
public class ProdEmailService implements EmailService {
    
    @Value("${email.smtp.host}")
    private String smtpHost;
    
    @Value("${email.smtp.port}")
    private int smtpPort;
    
    @Value("${email.from}")
    private String fromEmail;
    
    @Override
    public void sendEmail(String to, String subject, String body) {
        System.out.printf("📧 [PROD MODE] Sending email via %s:%d%n", smtpHost, smtpPort);
        System.out.printf("  From: %s%n", fromEmail);
        System.out.printf("  To: %s%n", to);
        System.out.printf("  Subject: %s%n", subject);
        
        // Real SMTP implementation would go here
        simulateRealEmailSend(to, subject, body);
    }
    
    @Override
    public String getServiceInfo() {
        return String.format("Production Email Service - SMTP %s:%d", smtpHost, smtpPort);
    }
    
    private void simulateRealEmailSend(String to, String subject, String body) {
        try {
            Thread.sleep(200); // Simulate network delay
            System.out.println("  Status: ✅ Email sent via SMTP");
        } catch (InterruptedException e) {
            System.out.println("  Status: ❌ Email send failed");
        }
    }
}

// Test - Mock Service
@Service
@Profile("test")
public class MockEmailService implements EmailService {
    
    private List<EmailRecord> sentEmails = new ArrayList<>();
    
    @Override
    public void sendEmail(String to, String subject, String body) {
        EmailRecord record = new EmailRecord(to, subject, body, LocalDateTime.now());
        sentEmails.add(record);
        
        System.out.println("🧪 [TEST MODE] Email captured for testing:");
        System.out.println("  To: " + to);
        System.out.println("  Subject: " + subject);
        System.out.println("  Total emails captured: " + sentEmails.size());
    }
    
    @Override
    public String getServiceInfo() {
        return "Test Email Service - Mock Implementation (Emails captured: " + sentEmails.size() + ")";
    }
    
    public List<EmailRecord> getSentEmails() {
        return new ArrayList<>(sentEmails);
    }
    
    public void clearSentEmails() {
        sentEmails.clear();
    }
    
    public static class EmailRecord {
        private final String to;
        private final String subject;
        private final String body;
        private final LocalDateTime sentAt;
        
        public EmailRecord(String to, String subject, String body, LocalDateTime sentAt) {
            this.to = to;
            this.subject = subject;
            this.body = body;
            this.sentAt = sentAt;
        }
        
        // Getters
        public String getTo() { return to; }
        public String getSubject() { return subject; }
        public String getBody() { return body; }
        public LocalDateTime getSentAt() { return sentAt; }
    }
}
```

---

## 📋 Profile-Specific Configuration Files

### File Naming Convention:
- `application.yml` - Default configuration
- `application-dev.yml` - Development profile
- `application-test.yml` - Test profile
- `application-prod.yml` - Production profile

### Configuration Files Example:

**application.yml** (Default):
```yaml
# Default configuration
app:
  name: My Application
  version: 1.0.0

logging:
  level:
    com.mycompany: INFO

management:
  endpoints:
    web:
      exposure:
        include: health,info
```

**application-dev.yml** (Development):
```yaml
# Development environment
app:
  name: My Application (DEV)
  debug-mode: true

database:
  url: jdbc:h2:mem:devdb
  show-sql: true

logging:
  level:
    com.mycompany: DEBUG
    org.springframework: DEBUG
    org.hibernate.SQL: DEBUG

email:
  enabled: false
  mode: console

cache:
  enabled: false

management:
  endpoints:
    web:
      exposure:
        include: "*"  # All endpoints for development
```

**application-test.yml** (Testing):
```yaml
# Test environment
app:
  name: My Application (TEST)
  debug-mode: true

database:
  url: jdbc:h2:mem:testdb
  
logging:
  level:
    com.mycompany: DEBUG

email:
  enabled: false
  mode: mock

cache:
  enabled: false
  
spring:
  jpa:
    hibernate:
      ddl-auto: create-drop  # Recreate schema for each test
```

**application-prod.yml** (Production):
```yaml
# Production environment
app:
  name: My Application
  debug-mode: false

database:
  url: ${DATABASE_URL}
  username: ${DATABASE_USERNAME}
  password: ${DATABASE_PASSWORD}
  pool:
    max-size: 50
    min-idle: 10

logging:
  level:
    com.mycompany: WARN
    root: ERROR
  file:
    name: /logs/application.log

email:
  enabled: true
  smtp:
    host: ${SMTP_HOST}
    port: ${SMTP_PORT}
    username: ${SMTP_USERNAME}
    password: ${SMTP_PASSWORD}

cache:
  enabled: true
  redis:
    host: ${REDIS_HOST}
    port: ${REDIS_PORT}

management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
```

---

## 🚀 Multiple Profile Conditions

### Complex Profile Expressions:
```java
// Multiple profiles (OR condition)
@Component
@Profile({"dev", "test"})
public class DevTestOnlyService {
    // Available in both dev AND test
}

// NOT condition
@Component
@Profile("!prod")
public class NonProductionService {
    // Available in all profiles EXCEPT production
}

// Complex expressions
@Component
@Profile("dev & debug")  // dev AND debug both active
public class DevDebugService {
}

@Component  
@Profile("prod | staging")  // prod OR staging
public class ProductionLikeService {
}
```

### Real-World Example: Payment Service
```java
// Payment Service Interface
public interface PaymentService {
    PaymentResult processPayment(PaymentRequest request);
    String getServiceInfo();
}

// Development - Mock Always Success
@Service
@Profile("dev")
public class DevPaymentService implements PaymentService {
    
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        System.out.println("💳 [DEV] Processing payment: $" + request.getAmount());
        
        // Always succeed in development
        return PaymentResult.builder()
                .transactionId("DEV-" + System.currentTimeMillis())
                .status("SUCCESS")
                .amount(request.getAmount())
                .message("Development payment always succeeds")
                .build();
    }
    
    @Override
    public String getServiceInfo() {
        return "Development Payment Service - Always Successful";
    }
}

// Test - Controllable Mock
@Service
@Profile("test")
public class TestPaymentService implements PaymentService {
    
    private boolean shouldFail = false;
    private String failureReason = "Test failure";
    
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        System.out.println("🧪 [TEST] Processing payment: $" + request.getAmount());
        
        if (shouldFail) {
            return PaymentResult.builder()
                    .transactionId("TEST-FAIL-" + System.currentTimeMillis())
                    .status("FAILED")
                    .amount(request.getAmount())
                    .message(failureReason)
                    .build();
        }
        
        return PaymentResult.builder()
                .transactionId("TEST-" + System.currentTimeMillis())
                .status("SUCCESS")
                .amount(request.getAmount())
                .message("Test payment processed")
                .build();
    }
    
    @Override
    public String getServiceInfo() {
        return "Test Payment Service - Controllable Mock (Fail mode: " + shouldFail + ")";
    }
    
    // Test control methods
    public void setShouldFail(boolean shouldFail) {
        this.shouldFail = shouldFail;
    }
    
    public void setFailureReason(String failureReason) {
        this.failureReason = failureReason;
    }
}

// Production - Real Implementation
@Service
@Profile("prod")
public class ProdPaymentService implements PaymentService {
    
    @Value("${payment.stripe.secret-key}")
    private String stripeSecretKey;
    
    @Value("${payment.timeout:30}")
    private int timeoutSeconds;
    
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        System.out.println("🚀 [PROD] Processing real payment: $" + request.getAmount());
        System.out.println("Using Stripe API with timeout: " + timeoutSeconds + "s");
        
        // Real Stripe integration would go here
        return simulateStripePayment(request);
    }
    
    @Override
    public String getServiceInfo() {
        return "Production Payment Service - Stripe Integration (Timeout: " + timeoutSeconds + "s)";
    }
    
    private PaymentResult simulateStripePayment(PaymentRequest request) {
        try {
            // Simulate API call delay
            Thread.sleep(1000);
            
            // Simulate 95% success rate
            if (Math.random() < 0.95) {
                return PaymentResult.builder()
                        .transactionId("stripe_" + System.currentTimeMillis())
                        .status("SUCCESS")
                        .amount(request.getAmount())
                        .message("Payment processed successfully via Stripe")
                        .build();
            } else {
                return PaymentResult.builder()
                        .transactionId("stripe_fail_" + System.currentTimeMillis())
                        .status("FAILED")
                        .amount(request.getAmount())
                        .message("Payment declined by bank")
                        .build();
            }
        } catch (InterruptedException e) {
            return PaymentResult.builder()
                    .status("ERROR")
                    .amount(request.getAmount())
                    .message("Payment processing timeout")
                    .build();
        }
    }
}

// Staging - Similar to prod but with logging
@Service
@Profile("staging")
public class StagingPaymentService implements PaymentService {
    
    @Autowired
    private ProdPaymentService prodPaymentService;
    
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        System.out.println("🎭 [STAGING] Enhanced logging for payment: $" + request.getAmount());
        
        PaymentResult result = prodPaymentService.processPayment(request);
        
        // Additional staging logging
        System.out.println("Payment Result Details:");
        System.out.println("  Transaction ID: " + result.getTransactionId());
        System.out.println("  Status: " + result.getStatus());
        System.out.println("  Message: " + result.getMessage());
        
        return result;
    }
    
    @Override
    public String getServiceInfo() {
        return "Staging Payment Service - Production with Enhanced Logging";
    }
}
```

---

## ⚙️ Profile Activation Methods

### 1. Application Properties:
```properties
# application.properties
spring.profiles.active=dev
```

### 2. Command Line Arguments:
```bash
# Single profile
java -jar app.jar --spring.profiles.active=prod

# Multiple profiles
java -jar app.jar --spring.profiles.active=prod,monitoring

# JVM system property
java -Dspring.profiles.active=dev -jar app.jar
```

### 3. Environment Variables:
```bash
export SPRING_PROFILES_ACTIVE=prod
java -jar app.jar
```

### 4. Programmatic Activation:
```java
@SpringBootApplication
public class MyApplication {
    
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApplication.class);
        
        // Set default profiles programmatically
        app.setDefaultProperties(Collections.singletonMap(
            "spring.profiles.default", "dev"
        ));
        
        app.run(args);
    }
}
```

### 5. IDE Configuration:
```
IntelliJ IDEA:
Run Configuration → Environment Variables → SPRING_PROFILES_ACTIVE=dev

Eclipse:
Run Configuration → Arguments → VM Arguments → -Dspring.profiles.active=dev
```

---

## 🏗️ Complete Real-World Example: E-Commerce Application

### Service Configuration with Profiles:
```java
@RestController
@RequestMapping("/api/system")
public class SystemController {
    
    @Autowired
    private EmailService emailService;
    
    @Autowired
    private PaymentService paymentService;
    
    @Autowired
    private DatabaseHealthChecker databaseHealthChecker;
    
    @Value("${spring.profiles.active:default}")
    private String activeProfile;
    
    @GetMapping("/info")
    public Map<String, String> getSystemInfo() {
        Map<String, String> info = new HashMap<>();
        info.put("activeProfile", activeProfile);
        info.put("emailService", emailService.getServiceInfo());
        info.put("paymentService", paymentService.getServiceInfo());
        info.put("database", databaseHealthChecker.getHealthInfo());
        info.put("timestamp", LocalDateTime.now().toString());
        return info;
    }
    
    @PostMapping("/test-email")
    public String testEmail(@RequestParam String email) {
        emailService.sendEmail(email, "Test Email", "This is a test email from " + activeProfile + " environment");
        return "Email sent using: " + emailService.getServiceInfo();
    }
    
    @PostMapping("/test-payment")
    public PaymentResult testPayment(@RequestParam double amount) {
        PaymentRequest request = new PaymentRequest();
        request.setAmount(amount);
        request.setCardNumber("4242424242424242");
        request.setCustomerId("test-customer");
        
        return paymentService.processPayment(request);
    }
}
```

### Health Check Implementations:
```java
public interface DatabaseHealthChecker {
    String getHealthInfo();
    boolean isHealthy();
}

@Component
@Profile("dev")
public class SimpleDatabaseHealthChecker implements DatabaseHealthChecker {
    
    private final String databaseName;
    
    public SimpleDatabaseHealthChecker(String databaseName) {
        this.databaseName = databaseName;
    }
    
    @Override
    public String getHealthInfo() {
        return databaseName + " - Always Healthy (Development)";
    }
    
    @Override
    public boolean isHealthy() {
        return true; // Always healthy in development
    }
}

@Component
@Profile("prod")
public class ProductionDatabaseHealthChecker implements DatabaseHealthChecker {
    
    private final String databaseName;
    
    @Autowired
    private DataSource dataSource;
    
    public ProductionDatabaseHealthChecker(String databaseName) {
        this.databaseName = databaseName;
    }
    
    @Override
    public String getHealthInfo() {
        boolean healthy = isHealthy();
        return String.format("%s - %s", 
            databaseName, 
            healthy ? "✅ Healthy" : "❌ Unhealthy"
        );
    }
    
    @Override
    public boolean isHealthy() {
        try {
            Connection connection = dataSource.getConnection();
            boolean valid = connection.isValid(5); // 5 second timeout
            connection.close();
            return valid;
        } catch (Exception e) {
            return false;
        }
    }
}

@Component
@Profile("test")
public class TestDatabaseHealthChecker implements DatabaseHealthChecker {
    
    private final String databaseName;
    
    public TestDatabaseHealthChecker(String databaseName) {
        this.databaseName = databaseName;
    }
    
    @Override
    public String getHealthInfo() {
        return databaseName + " - Test Environment (Simulated Health)";
    }
    
    @Override
    public boolean isHealthy() {
        return true; // Always healthy in test
    }
}
```

### Supporting Classes:
```java
// Payment Request/Response classes
public class PaymentRequest {
    private double amount;
    private String cardNumber;
    private String customerId;
    
    // Getters and setters
    public double getAmount() { return amount; }
    public void setAmount(double amount) { this.amount = amount; }
    
    public String getCardNumber() { return cardNumber; }
    public void setCardNumber(String cardNumber) { this.cardNumber = cardNumber; }
    
    public String getCustomerId() { return customerId; }
    public void setCustomerId(String customerId) { this.customerId = customerId; }
}

public class PaymentResult {
    private String transactionId;
    private String status;
    private double amount;
    private String message;
    
    // Builder pattern
    public static PaymentResultBuilder builder() {
        return new PaymentResultBuilder();
    }
    
    public static class PaymentResultBuilder {
        private PaymentResult result = new PaymentResult();
        
        public PaymentResultBuilder transactionId(String transactionId) {
            result.transactionId = transactionId;
            return this;
        }
        
        public PaymentResultBuilder status(String status) {
            result.status = status;
            return this;
        }
        
        public PaymentResultBuilder amount(double amount) {
            result.amount = amount;
            return this;
        }
        
        public PaymentResultBuilder message(String message) {
            result.message = message;
            return this;
        }
        
        public PaymentResult build() {
            return result;
        }
    }
    
    // Getters
    public String getTransactionId() { return transactionId; }
    public String getStatus() { return status; }
    public double getAmount() { return amount; }
    public String getMessage() { return message; }
}
```

---

## 🎯 Interview Questions & Answers

### Q1: "What are Spring Profiles and why use them?"
**Answer:**
- **Profiles are environment-specific configurations** that allow different bean definitions and properties per environment
- **Benefits**: Separate dev/test/prod configs, feature toggling, resource optimization, security levels
- **Use cases**: Different databases, mock vs real services, logging levels, feature flags

### Q2: "How do you activate profiles?"
**Answer:**
1. **application.properties**: `spring.profiles.active=dev`
2. **Command line**: `--spring.profiles.active=prod`
3. **Environment variable**: `SPRING_PROFILES_ACTIVE=test`
4. **JVM property**: `-Dspring.profiles.active=dev`
5. **Programmatically**: `SpringApplication.setDefaultProperties()`

### Q3: "What's the difference between @Profile and profile-specific configuration files?"
**Answer:**
- **@Profile**: Controls which beans are created in different environments
- **Configuration files**: Controls property values per environment
- **Both work together**: @Profile for conditional beans, config files for environment-specific values

### Q4: "Can you have multiple profiles active at once?"
**Answer:**
- **Yes**: `spring.profiles.active=dev,debug,logging`
- **Useful for**: Feature combinations, cross-cutting concerns
- **Order matters**: Later profiles can override earlier ones

### Q5: "What happens if no profile is specified?"
**Answer:**
- **Default profile** is activated automatically
- **@Profile("default")** beans are created
- **application.yml** (without suffix) is used
- **Can set custom default**: `spring.profiles.default=dev`

---

## 🚀 Best Practices & Tips

### 1. **Profile Naming Conventions**
```java
// ✅ Good: Clear, descriptive names
@Profile("dev")          // development
@Profile("test")         // testing
@Profile("staging")      // staging environment
@Profile("prod")         // production

// ✅ Feature-based profiles
@Profile("debug")        // debug features
@Profile("monitoring")   // monitoring tools
@Profile("cache")        // caching enabled

// ❌ Avoid: Unclear names
@Profile("profile1")
@Profile("config-a")
```

### 2. **Hierarchical Configuration**
```yaml
# Base configuration in application.yml
app:
  name: My App
  timeout: 30

# Override in application-prod.yml
app:
  timeout: 60  # Only override what's different
  
logging:
  level:
    root: WARN  # Add production-specific settings
```

### 3. **Profile Groups** (Spring Boot 2.4+)
```yaml
# application.yml
spring:
  profiles:
    group:
      production: "prod,monitoring,security"
      development: "dev,debug"
```

```bash
# Activates prod, monitoring, and security profiles
java -jar app.jar --spring.profiles.active=production
```

### 4. **Conditional Properties**
```java
@Component
public class FeatureToggleService {
    
    @Value("${features.new-ui:false}")
    private boolean newUiEnabled;
    
    @Value("${features.beta-features:false}")
    private boolean betaFeaturesEnabled;
    
    public boolean isFeatureEnabled(String featureName) {
        switch (featureName) {
            case "new-ui": return newUiEnabled;
            case "beta-features": return betaFeaturesEnabled;
            default: return false;
        }
    }
}
```

