# 📅 Day 5: Application Properties & Configuration

## 🎯 Today Topics
- Master `@Value` annotation for property injection
- Understand `@ConfigurationProperties` for complex configurations
- Learn `application.properties` vs `application.yml`
- Handle environment-specific configurations
- Simple, practical examples you can use immediately

---

## 📄 Understanding Application Properties

### What are Application Properties?
**Application Properties** are external configuration values that your Spring Boot application can read at runtime. Instead of hardcoding values, you store them in configuration files.

### Why Use External Configuration?
- **Environment Flexibility**: Different settings for dev/test/prod
- **Security**: Keep secrets out of code
- **Easy Changes**: Modify behavior without code changes
- **Deployment Friendly**: Same code, different configs

---

## 🔧 @Value - Simple Property Injection

### Basic Syntax:
```java
@Value("${property.name}")
private String propertyValue;

@Value("${property.name:defaultValue}")
private String propertyWithDefault;
```

### Simple Example: Basic Configuration
```properties
# application.properties
app.name=My Awesome App
app.version=1.0.0
app.welcome.message=Welcome to our application!
server.timeout=30
database.max.connections=100
```

```java
@Service
public class AppInfoService {
    
    @Value("${app.name}")
    private String appName;
    
    @Value("${app.version}")
    private String appVersion;
    
    @Value("${app.welcome.message}")
    private String welcomeMessage;
    
    @Value("${server.timeout:60}") // Default 60 if not found
    private int serverTimeout;
    
    @Value("${database.max.connections}")
    private int maxConnections;
    
    public String getAppInfo() {
        return String.format("App: %s v%s, Timeout: %d seconds, DB Connections: %d", 
                           appName, appVersion, serverTimeout, maxConnections);
    }
    
    public String getWelcomeMessage() {
        return welcomeMessage;
    }
}
```

### Real-World Example: Email Service Configuration
```properties
# application.properties
email.smtp.host=smtp.gmail.com
email.smtp.port=587
email.from=noreply@myapp.com
email.enabled=true
email.retry.count=3
```

```java
@Service
public class EmailService {
    
    @Value("${email.smtp.host}")
    private String smtpHost;
    
    @Value("${email.smtp.port}")
    private int smtpPort;
    
    @Value("${email.from}")
    private String fromEmail;
    
    @Value("${email.enabled:true}")
    private boolean emailEnabled;
    
    @Value("${email.retry.count:1}")
    private int retryCount;
    
    public void sendEmail(String to, String subject, String body) {
        if (!emailEnabled) {
            System.out.println("Email disabled - skipping send");
            return;
        }
        
        System.out.printf("Sending email via %s:%d from %s to %s%n", 
                         smtpHost, smtpPort, fromEmail, to);
        System.out.printf("Subject: %s%n", subject);
        System.out.printf("Retry attempts: %d%n", retryCount);
        
        // Actual email sending logic would go here
        simulateEmailSend(to, subject, body);
    }
    
    private void simulateEmailSend(String to, String subject, String body) {
        try {
            // Simulate network delay
            Thread.sleep(100);
            System.out.println("✅ Email sent successfully!");
        } catch (InterruptedException e) {
            System.out.println("❌ Email sending failed!");
        }
    }
}
```

---

## 📋 @ConfigurationProperties - Type-Safe Configuration

### When to use @ConfigurationProperties:
- Multiple related properties
- Complex data types
- Validation needed
- Better IDE support

### Simple Example: Database Configuration
```properties
# application.properties
database.url=jdbc:mysql://localhost:3306/myapp
database.username=admin
database.password=secret123
database.driver=com.mysql.cj.jdbc.Driver
database.pool.min-size=5
database.pool.max-size=20
database.pool.timeout=30000
```

```java
@ConfigurationProperties(prefix = "database")
@Component
public class DatabaseProperties {
    
    private String url;
    private String username;
    private String password;
    private String driver;
    private Pool pool = new Pool();
    
    // Nested class for pool properties
    public static class Pool {
        private int minSize;
        private int maxSize;
        private int timeout;
        
        // Getters and setters
        public int getMinSize() { return minSize; }
        public void setMinSize(int minSize) { this.minSize = minSize; }
        
        public int getMaxSize() { return maxSize; }
        public void setMaxSize(int maxSize) { this.maxSize = maxSize; }
        
        public int getTimeout() { return timeout; }
        public void setTimeout(int timeout) { this.timeout = timeout; }
    }
    
    // Main class getters and setters
    public String getUrl() { return url; }
    public void setUrl(String url) { this.url = url; }
    
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    
    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }
    
    public String getDriver() { return driver; }
    public void setDriver(String driver) { this.driver = driver; }
    
    public Pool getPool() { return pool; }
    public void setPool(Pool pool) { this.pool = pool; }
}
```

```java
@Service
public class DatabaseService {
    
    private final DatabaseProperties databaseProperties;
    
    public DatabaseService(DatabaseProperties databaseProperties) {
        this.databaseProperties = databaseProperties;
    }
    
    public void printDatabaseConfig() {
        System.out.println("=== Database Configuration ===");
        System.out.println("URL: " + databaseProperties.getUrl());
        System.out.println("Username: " + databaseProperties.getUsername());
        System.out.println("Driver: " + databaseProperties.getDriver());
        System.out.println("Pool Min Size: " + databaseProperties.getPool().getMinSize());
        System.out.println("Pool Max Size: " + databaseProperties.getPool().getMaxSize());
        System.out.println("Pool Timeout: " + databaseProperties.getPool().getTimeout());
    }
    
    public String getConnectionString() {
        return String.format("Connecting to %s with user %s", 
                           databaseProperties.getUrl(), 
                           databaseProperties.getUsername());
    }
}
```

---

## 📝 Properties vs YAML Configuration

### Properties File Format:
```properties
# application.properties
app.name=My App
app.version=1.0.0
app.features.user-registration=true
app.features.email-notifications=true
app.features.payment-processing=false

# Database configuration
database.host=localhost
database.port=5432
database.name=myapp_db
database.username=admin
database.password=secret

# List of allowed domains
app.allowed-domains[0]=example.com
app.allowed-domains[1]=mycompany.com
app.allowed-domains[2]=partner.com
```

### YAML File Format (RECOMMENDED ✅):
```yaml
# application.yml
app:
  name: My App
  version: 1.0.0
  features:
    user-registration: true
    email-notifications: true
    payment-processing: false
  allowed-domains:
    - example.com
    - mycompany.com
    - partner.com

database:
  host: localhost
  port: 5432
  name: myapp_db
  username: admin
  password: secret
```

### Configuration Class for YAML:
```java
@ConfigurationProperties(prefix = "app")
@Component
public class AppProperties {
    
    private String name;
    private String version;
    private Features features = new Features();
    private List<String> allowedDomains = new ArrayList<>();
    
    public static class Features {
        private boolean userRegistration;
        private boolean emailNotifications;
        private boolean paymentProcessing;
        
        // Getters and setters
        public boolean isUserRegistration() { return userRegistration; }
        public void setUserRegistration(boolean userRegistration) { 
            this.userRegistration = userRegistration; 
        }
        
        public boolean isEmailNotifications() { return emailNotifications; }
        public void setEmailNotifications(boolean emailNotifications) { 
            this.emailNotifications = emailNotifications; 
        }
        
        public boolean isPaymentProcessing() { return paymentProcessing; }
        public void setPaymentProcessing(boolean paymentProcessing) { 
            this.paymentProcessing = paymentProcessing; 
        }
    }
    
    // Main class getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getVersion() { return version; }
    public void setVersion(String version) { this.version = version; }
    
    public Features getFeatures() { return features; }
    public void setFeatures(Features features) { this.features = features; }
    
    public List<String> getAllowedDomains() { return allowedDomains; }
    public void setAllowedDomains(List<String> allowedDomains) { 
        this.allowedDomains = allowedDomains; 
    }
}
```

---

## 🌍 Environment-Specific Configuration

### Profile-Based Configuration Files:

**application.yml** (default):
```yaml
app:
  name: My Application
  environment: default

database:
  host: localhost
  port: 5432

logging:
  level:
    com.mycompany: INFO
```

**application-dev.yml** (development):
```yaml
app:
  environment: development

database:
  host: localhost
  port: 5432
  name: myapp_dev
  username: dev_user
  password: dev_password

logging:
  level:
    com.mycompany: DEBUG
    org.springframework: DEBUG
```

**application-test.yml** (testing):
```yaml
app:
  environment: testing

database:
  host: testdb.company.com
  port: 5432
  name: myapp_test
  username: test_user
  password: test_password

logging:
  level:
    com.mycompany: INFO
```

**application-prod.yml** (production):
```yaml
app:
  environment: production

database:
  host: proddb.company.com
  port: 5432
  name: myapp_prod
  username: ${DB_USERNAME}  # Environment variable
  password: ${DB_PASSWORD}  # Environment variable

logging:
  level:
    com.mycompany: WARN
    root: ERROR
```

### Simple Service Using Profiles:
```java
@Service
public class EnvironmentService {
    
    @Value("${app.environment}")
    private String environment;
    
    @Value("${spring.profiles.active:default}")
    private String activeProfile;
    
    public void printEnvironmentInfo() {
        System.out.println("=== Environment Information ===");
        System.out.println("Environment: " + environment);
        System.out.println("Active Profile: " + activeProfile);
        System.out.println("Is Development: " + isDevelopment());
        System.out.println("Is Production: " + isProduction());
    }
    
    public boolean isDevelopment() {
        return "development".equals(environment);
    }
    
    public boolean isProduction() {
        return "production".equals(environment);
    }
    
    public String getEnvironment() {
        return environment;
    }
}
```

---

## 🔒 Security & Sensitive Properties

### Using Environment Variables:
```yaml
# application-prod.yml
database:
  username: ${DB_USERNAME}
  password: ${DB_PASSWORD}
  
security:
  jwt-secret: ${JWT_SECRET}
  
external-apis:
  stripe-key: ${STRIPE_API_KEY}
  sendgrid-key: ${SENDGRID_API_KEY}
```

### Simple Security Configuration:
```java
@ConfigurationProperties(prefix = "security")
@Component
public class SecurityProperties {
    
    private String jwtSecret;
    private long jwtExpiration = 86400; // 24 hours default
    private boolean enableSsl = false;
    
    // Getters and setters
    public String getJwtSecret() { return jwtSecret; }
    public void setJwtSecret(String jwtSecret) { this.jwtSecret = jwtSecret; }
    
    public long getJwtExpiration() { return jwtExpiration; }
    public void setJwtExpiration(long jwtExpiration) { this.jwtExpiration = jwtExpiration; }
    
    public boolean isEnableSsl() { return enableSsl; }
    public void setEnableSsl(boolean enableSsl) { this.enableSsl = enableSsl; }
}
```

---

## 📊 Complete Simple Example: Blog Application

### Configuration Files:

**application.yml**:
```yaml
blog:
  title: My Personal Blog
  description: Welcome to my blog about technology
  author: John Doe
  posts-per-page: 5
  features:
    comments-enabled: true
    social-sharing: true
    email-subscription: false
  social-media:
    twitter: "@johndoe"
    linkedin: "john-doe-dev"
    github: "johndoe"

email:
  enabled: true
  from: noreply@myblog.com
  smtp:
    host: smtp.gmail.com
    port: 587
    username: ${EMAIL_USERNAME}
    password: ${EMAIL_PASSWORD}
```

### Configuration Classes:

```java
@ConfigurationProperties(prefix = "blog")
@Component
public class BlogProperties {
    
    private String title;
    private String description;
    private String author;
    private int postsPerPage = 10;
    private Features features = new Features();
    private Map<String, String> socialMedia = new HashMap<>();
    
    public static class Features {
        private boolean commentsEnabled;
        private boolean socialSharing;
        private boolean emailSubscription;
        
        // Getters and setters
        public boolean isCommentsEnabled() { return commentsEnabled; }
        public void setCommentsEnabled(boolean commentsEnabled) { 
            this.commentsEnabled = commentsEnabled; 
        }
        
        public boolean isSocialSharing() { return socialSharing; }
        public void setSocialSharing(boolean socialSharing) { 
            this.socialSharing = socialSharing; 
        }
        
        public boolean isEmailSubscription() { return emailSubscription; }
        public void setEmailSubscription(boolean emailSubscription) { 
            this.emailSubscription = emailSubscription; 
        }
    }
    
    // Getters and setters
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
    
    public int getPostsPerPage() { return postsPerPage; }
    public void setPostsPerPage(int postsPerPage) { this.postsPerPage = postsPerPage; }
    
    public Features getFeatures() { return features; }
    public void setFeatures(Features features) { this.features = features; }
    
    public Map<String, String> getSocialMedia() { return socialMedia; }
    public void setSocialMedia(Map<String, String> socialMedia) { 
        this.socialMedia = socialMedia; 
    }
}

@ConfigurationProperties(prefix = "email")
@Component
public class EmailProperties {
    
    private boolean enabled;
    private String from;
    private Smtp smtp = new Smtp();
    
    public static class Smtp {
        private String host;
        private int port;
        private String username;
        private String password;
        
        // Getters and setters
        public String getHost() { return host; }
        public void setHost(String host) { this.host = host; }
        
        public int getPort() { return port; }
        public void setPort(int port) { this.port = port; }
        
        public String getUsername() { return username; }
        public void setUsername(String username) { this.username = username; }
        
        public String getPassword() { return password; }
        public void setPassword(String password) { this.password = password; }
    }
    
    // Getters and setters
    public boolean isEnabled() { return enabled; }
    public void setEnabled(boolean enabled) { this.enabled = enabled; }
    
    public String getFrom() { return from; }
    public void setFrom(String from) { this.from = from; }
    
    public Smtp getSmtp() { return smtp; }
    public void setSmtp(Smtp smtp) { this.smtp = smtp; }
}
```

### Service Using Configuration:

```java
@Service
public class BlogService {
    
    private final BlogProperties blogProperties;
    private final EmailProperties emailProperties;
    
    public BlogService(BlogProperties blogProperties, EmailProperties emailProperties) {
        this.blogProperties = blogProperties;
        this.emailProperties = emailProperties;
    }
    
    public String getBlogInfo() {
        return String.format("""
            === Blog Information ===
            Title: %s
            Author: %s
            Description: %s
            Posts per page: %d
            Comments enabled: %s
            Social sharing: %s
            Email subscription: %s
            Twitter: %s
            GitHub: %s
            """, 
            blogProperties.getTitle(),
            blogProperties.getAuthor(),
            blogProperties.getDescription(),
            blogProperties.getPostsPerPage(),
            blogProperties.getFeatures().isCommentsEnabled(),
            blogProperties.getFeatures().isSocialSharing(),
            blogProperties.getFeatures().isEmailSubscription(),
            blogProperties.getSocialMedia().get("twitter"),
            blogProperties.getSocialMedia().get("github")
        );
    }
    
    public void sendWelcomeEmail(String userEmail) {
        if (!emailProperties.isEnabled()) {
            System.out.println("Email sending is disabled");
            return;
        }
        
        System.out.printf("""
            Sending welcome email:
            From: %s
            To: %s
            SMTP Host: %s:%d
            Username: %s
            Subject: Welcome to %s!
            """,
            emailProperties.getFrom(),
            userEmail,
            emailProperties.getSmtp().getHost(),
            emailProperties.getSmtp().getPort(),
            emailProperties.getSmtp().getUsername(),
            blogProperties.getTitle()
        );
    }
    
    public List<String> getEnabledFeatures() {
        List<String> features = new ArrayList<>();
        
        if (blogProperties.getFeatures().isCommentsEnabled()) {
            features.add("Comments");
        }
        if (blogProperties.getFeatures().isSocialSharing()) {
            features.add("Social Sharing");
        }
        if (blogProperties.getFeatures().isEmailSubscription()) {
            features.add("Email Subscription");
        }
        
        return features;
    }
}
```

### Controller Using the Service:

```java
@RestController
@RequestMapping("/api/blog")
public class BlogController {
    
    private final BlogService blogService;
    
    public BlogController(BlogService blogService) {
        this.blogService = blogService;
    }
    
    @GetMapping("/info")
    public ResponseEntity<String> getBlogInfo() {
        return ResponseEntity.ok(blogService.getBlogInfo());
    }
    
    @GetMapping("/features")
    public ResponseEntity<List<String>> getEnabledFeatures() {
        return ResponseEntity.ok(blogService.getEnabledFeatures());
    }
    
    @PostMapping("/subscribe")
    public ResponseEntity<String> subscribe(@RequestParam String email) {
        blogService.sendWelcomeEmail(email);
        return ResponseEntity.ok("Welcome email sent to: " + email);
    }
}
```

---

## 🎯 Interview Questions & Answers

### Q1: "What's the difference between @Value and @ConfigurationProperties?"
**Answer:**
- `@Value`: Single property injection, simple types, scattered across code
- `@ConfigurationProperties`: Multiple related properties, type-safe, organized in one class
- **Use @Value for simple cases, @ConfigurationProperties for complex configurations**

### Q2: "How do you handle missing properties?"
**Answer:**
```java
@Value("${missing.property:defaultValue}") // Default value
@Value("#{null}") // Null if missing
@Autowired(required = false) Optional<String> optionalProperty; // Optional
```

### Q3: "What's the property resolution order in Spring Boot?"
**Answer:**
1. Command line arguments
2. SPRING_APPLICATION_JSON properties
3. ServletConfig init parameters
4. ServletContext init parameters
5. JNDI attributes
6. Java System properties
7. OS environment variables
8. Profile-specific application properties
9. Application properties (application.properties/yml)
10. @PropertySource annotations
11. Default properties

### Q4: "How do you secure sensitive properties?"
**Answer:**
- Use environment variables: `${DB_PASSWORD}`
- External configuration servers
- Encrypted properties with Spring Cloud Config
- Never commit secrets to version control
- Use different configs per environment

### Q5: "What's the difference between .properties and .yml files?"
**Answer:**
- **Properties**: Flat structure, repetitive, harder to read
- **YAML**: Hierarchical, concise, easier to read, supports lists/maps naturally
- **YAML is recommended for new projects**

---

## 🚀 Quick Tips & Best Practices

### 1. **Property Naming Conventions**
```yaml
# ✅ Good: kebab-case
app:
  database-url: jdbc:mysql://localhost
  max-connections: 100
  feature-flags:
    user-registration: true

# ❌ Avoid: Mixed cases
app:
  databaseURL: jdbc:mysql://localhost  # Inconsistent
  maxConnections: 100                  # camelCase
  feature_flags:                       # snake_case
    userRegistration: true
```

### 2. **Use Defaults Wisely**
```java
@Value("${server.port:8080}")
private int serverPort;

@Value("${app.timeout:30}")
private int timeoutSeconds;

@Value("${app.debug:false}")
private boolean debugMode;
```

### 3. **Organize Properties by Domain**
```yaml
# ✅ Well organized
database:
  host: localhost
  port: 5432
  credentials:
    username: admin
    password: secret

cache:
  redis:
    host: localhost
    port: 6379
  ttl: 3600

email:
  smtp:
    host: smtp.gmail.com
    port: 587
  templates:
    welcome: templates/welcome.html
```

