# 📅 Day 13: CORS & Filters

## 🎯 Today Topics
- Master `@CrossOrigin` for cross-origin requests
- Understand CORS concepts and browser security
- Create custom filters for request/response processing
- Implement interceptors for common functionality
- Handle security and logging with filters

---

## 🌐 Understanding CORS (Cross-Origin Resource Sharing)

### What is CORS?
**CORS** is a security feature that controls which web pages can access your API from different domains.

### The Problem:
```javascript
// Frontend at http://localhost:3000 trying to call API at http://localhost:8080
fetch('http://localhost:8080/api/users')
  .then(response => response.json())
  .catch(error => console.error('CORS Error:', error));

// Browser blocks this request by default!
// Error: Access to fetch at 'http://localhost:8080/api/users' 
// from origin 'http://localhost:3000' has been blocked by CORS policy
```

### The Solution:
```java
// Enable CORS in Spring Boot
@CrossOrigin(origins = "http://localhost:3000")
@RestController
public class UserController {
    // Now frontend can access this API
}
```

---

## ✅ @CrossOrigin - Method & Class Level

### Method Level CORS:
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // Allow specific origin for this method only
    @CrossOrigin(origins = "http://localhost:3000")
    @GetMapping
    public List<User> getAllUsers() {
        System.out.println("🌍 CORS enabled for getAllUsers()");
        return userService.findAll();
    }
    
    // Allow multiple origins
    @CrossOrigin(origins = {"http://localhost:3000", "https://myapp.com"})
    @PostMapping
    public User createUser(@RequestBody User user) {
        System.out.println("🌍 CORS enabled for multiple origins");
        return userService.save(user);
    }
    
    // Allow all origins (NOT recommended for production)
    @CrossOrigin(origins = "*")
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        System.out.println("⚠️ CORS enabled for ALL origins");
        return userService.findById(id);
    }
    
    // Configure specific HTTP methods
    @CrossOrigin(
        origins = "http://localhost:3000",
        methods = {RequestMethod.GET, RequestMethod.POST}
    )
    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        System.out.println("🔒 CORS limited to GET and POST only");
        return userService.update(id, user);
    }
}
```

### Class Level CORS:
```java
// Apply CORS to all methods in controller
@CrossOrigin(origins = "http://localhost:3000")
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    @GetMapping
    public List<Product> getAllProducts() {
        System.out.println("📦 All product endpoints have CORS enabled");
        return productService.findAll();
    }
    
    @PostMapping
    public Product createProduct(@RequestBody Product product) {
        return productService.save(product);
    }
    
    // Override class-level CORS for specific method
    @CrossOrigin(origins = {"http://localhost:3000", "https://admin.myapp.com"})
    @DeleteMapping("/{id}")
    public String deleteProduct(@PathVariable Long id) {
        System.out.println("🗑️ Delete endpoint has extended CORS");
        productService.deleteById(id);
        return "Product deleted";
    }
}
```

---

## ⚙️ Global CORS Configuration

### Configuration Class:
```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        System.out.println("🌍 Configuring global CORS settings");
        
        // Configure CORS for all endpoints
        registry.addMapping("/api/**")
                .allowedOrigins("http://localhost:3000", "https://myapp.com")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600); // Cache preflight for 1 hour
        
        // Separate configuration for admin endpoints
        registry.addMapping("/admin/**")
                .allowedOrigins("https://admin.myapp.com")
                .allowedMethods("GET", "POST", "PUT", "DELETE")
                .allowedHeaders("Authorization", "Content-Type")
                .allowCredentials(true);
        
        System.out.println("✅ CORS configuration applied");
    }
}
```

### Environment-Specific CORS:
```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    
    @Value("${app.cors.allowed-origins:http://localhost:3000}")
    private String[] allowedOrigins;
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        System.out.println("🌍 Allowed origins: " + Arrays.toString(allowedOrigins));
        
        registry.addMapping("/api/**")
                .allowedOrigins(allowedOrigins)
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true);
    }
}
```

**application.yml:**
```yaml
app:
  cors:
    allowed-origins:
      - http://localhost:3000    # Development
      - https://myapp.com        # Production
      - https://staging.myapp.com # Staging
```

---

## 🔧 Custom Filters

### Basic Request Logging Filter:
```java
@Component
@Order(1) // Execute first
public class RequestLoggingFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        
        // Log request details
        String method = httpRequest.getMethod();
        String uri = httpRequest.getRequestURI();
        String userAgent = httpRequest.getHeader("User-Agent");
        String clientIp = getClientIpAddress(httpRequest);
        
        long startTime = System.currentTimeMillis();
        
        System.out.printf("🌐 [%s] %s %s - IP: %s - User-Agent: %s%n", 
                         LocalDateTime.now(), method, uri, clientIp, userAgent);
        
        try {
            // Continue with the request
            chain.doFilter(request, response);
            
            // Log response details
            long duration = System.currentTimeMillis() - startTime;
            System.out.printf("✅ [%s] %s %s - Status: %d - Duration: %dms%n",
                             LocalDateTime.now(), method, uri, httpResponse.getStatus(), duration);
            
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            System.out.printf("❌ [%s] %s %s - Error: %s - Duration: %dms%n",
                             LocalDateTime.now(), method, uri, e.getMessage(), duration);
            throw e;
        }
    }
    
    private String getClientIpAddress(HttpServletRequest request) {
        String xForwardedFor = request.getHeader("X-Forwarded-For");
        if (xForwardedFor != null && !xForwardedFor.isEmpty()) {
            return xForwardedFor.split(",")[0].trim();
        }
        
        String xRealIp = request.getHeader("X-Real-IP");
        if (xRealIp != null && !xRealIp.isEmpty()) {
            return xRealIp;
        }
        
        return request.getRemoteAddr();
    }
}
```

### Authentication Filter:
```java
@Component
@Order(2) // Execute after logging filter
public class AuthenticationFilter implements Filter {
    
    private static final List<String> EXCLUDED_PATHS = Arrays.asList(
        "/api/auth/login",
        "/api/auth/register",
        "/api/public",
        "/health",
        "/actuator"
    );
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        
        String uri = httpRequest.getRequestURI();
        
        // Skip authentication for excluded paths
        if (isExcludedPath(uri)) {
            System.out.println("🔓 Skipping authentication for: " + uri);
            chain.doFilter(request, response);
            return;
        }
        
        // Check for Authorization header
        String authHeader = httpRequest.getHeader("Authorization");
        
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            System.out.println("❌ Missing or invalid Authorization header for: " + uri);
            httpResponse.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            httpResponse.setContentType("application/json");
            httpResponse.getWriter().write("{\"error\":\"Authentication required\"}");
            return;
        }
        
        // Extract and validate token
        String token = authHeader.substring(7); // Remove "Bearer " prefix
        
        if (isValidToken(token)) {
            System.out.println("✅ Authentication successful for: " + uri);
            // Add user info to request attributes
            httpRequest.setAttribute("userId", getUserIdFromToken(token));
            httpRequest.setAttribute("username", getUsernameFromToken(token));
            chain.doFilter(request, response);
        } else {
            System.out.println("❌ Invalid token for: " + uri);
            httpResponse.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            httpResponse.setContentType("application/json");
            httpResponse.getWriter().write("{\"error\":\"Invalid token\"}");
        }
    }
    
    private boolean isExcludedPath(String uri) {
        return EXCLUDED_PATHS.stream().anyMatch(path -> uri.startsWith(path));
    }
    
    private boolean isValidToken(String token) {
        // Simple validation - in production, use JWT validation
        return token.length() > 10 && !token.equals("invalid");
    }
    
    private Long getUserIdFromToken(String token) {
        // Mock implementation - extract user ID from token
        return 123L;
    }
    
    private String getUsernameFromToken(String token) {
        // Mock implementation - extract username from token
        return "john.doe";
    }
}
```

### Rate Limiting Filter:
```java
@Component
@Order(3)
public class RateLimitingFilter implements Filter {
    
    private final Map<String, List<Long>> requestCounts = new ConcurrentHashMap<>();
    private final int MAX_REQUESTS_PER_MINUTE = 60;
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        
        String clientIp = getClientIp(httpRequest);
        long currentTime = System.currentTimeMillis();
        
        // Clean old requests (older than 1 minute)
        cleanOldRequests(clientIp, currentTime);
        
        // Check rate limit
        List<Long> requests = requestCounts.computeIfAbsent(clientIp, k -> new ArrayList<>());
        
        if (requests.size() >= MAX_REQUESTS_PER_MINUTE) {
            System.out.printf("🚫 Rate limit exceeded for IP: %s (%d requests)%n", 
                            clientIp, requests.size());
            
            httpResponse.setStatus(HttpServletResponse.SC_TOO_MANY_REQUESTS);
            httpResponse.setContentType("application/json");
            httpResponse.getWriter().write(
                "{\"error\":\"Rate limit exceeded. Maximum " + MAX_REQUESTS_PER_MINUTE + " requests per minute.\"}"
            );
            return;
        }
        
        // Record this request
        requests.add(currentTime);
        System.out.printf("📊 Rate limit check passed for IP: %s (%d/%d requests)%n", 
                         clientIp, requests.size(), MAX_REQUESTS_PER_MINUTE);
        
        chain.doFilter(request, response);
    }
    
    private void cleanOldRequests(String clientIp, long currentTime) {
        List<Long> requests = requestCounts.get(clientIp);
        if (requests != null) {
            long oneMinuteAgo = currentTime - 60000; // 60 seconds
            requests.removeIf(timestamp -> timestamp < oneMinuteAgo);
            
            if (requests.isEmpty()) {
                requestCounts.remove(clientIp);
            }
        }
    }
    
    private String getClientIp(HttpServletRequest request) {
        String xForwardedFor = request.getHeader("X-Forwarded-For");
        if (xForwardedFor != null && !xForwardedFor.isEmpty()) {
            return xForwardedFor.split(",")[0].trim();
        }
        return request.getRemoteAddr();
    }
}
```

---

## 🎯 Interceptors vs Filters

### HandlerInterceptor Example:
```java
@Component
public class PerformanceInterceptor implements HandlerInterceptor {
    
    private static final String START_TIME = "startTime";
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, 
                           Object handler) throws Exception {
        
        long startTime = System.currentTimeMillis();
        request.setAttribute(START_TIME, startTime);
        
        String method = request.getMethod();
        String uri = request.getRequestURI();
        
        System.out.printf("⏱️ [START] %s %s - %s%n", method, uri, LocalDateTime.now());
        
        return true; // Continue with the request
    }
    
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, 
                          Object handler, ModelAndView modelAndView) throws Exception {
        
        System.out.println("🎯 Controller method completed, preparing response...");
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, 
                               Object handler, Exception ex) throws Exception {
        
        Long startTime = (Long) request.getAttribute(START_TIME);
        if (startTime != null) {
            long duration = System.currentTimeMillis() - startTime;
            String method = request.getMethod();
            String uri = request.getRequestURI();
            int status = response.getStatus();
            
            if (duration > 1000) {
                System.out.printf("🐌 [SLOW] %s %s - Status: %d - Duration: %dms%n", 
                                method, uri, status, duration);
            } else {
                System.out.printf("⚡ [FAST] %s %s - Status: %d - Duration: %dms%n", 
                                method, uri, status, duration);
            }
        }
        
        if (ex != null) {
            System.out.printf("💥 [ERROR] %s %s - Exception: %s%n", 
                             request.getMethod(), request.getRequestURI(), ex.getMessage());
        }
    }
}

// Register the interceptor
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Autowired
    private PerformanceInterceptor performanceInterceptor;
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(performanceInterceptor)
                .addPathPatterns("/api/**")
                .excludePathPatterns("/api/health", "/api/public/**");
        
        System.out.println("🎯 Performance interceptor registered for /api/** paths");
    }
}
```

---

## 📋 Complete Example: API Gateway-style Setup

### Main Configuration:
```java
@SpringBootApplication
public class Application {
    
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
        System.out.println("🚀 API Gateway started with CORS and Filters enabled!");
    }
}
```

### Combined Security & CORS Configuration:
```java
@Configuration
public class WebSecurityConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins(
                    "http://localhost:3000",           // React dev server
                    "http://localhost:4200",           // Angular dev server
                    "https://myapp.com",               // Production frontend
                    "https://admin.myapp.com"          // Admin panel
                )
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS", "PATCH")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
        
        System.out.println("🌍 CORS configured for multiple frontend origins");
    }
    
    @Bean
    public FilterRegistrationBean<RequestLoggingFilter> loggingFilter() {
        FilterRegistrationBean<RequestLoggingFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new RequestLoggingFilter());
        registrationBean.addUrlPatterns("/api/*");
        registrationBean.setOrder(1);
        System.out.println("📝 Request logging filter registered");
        return registrationBean;
    }
    
    @Bean
    public FilterRegistrationBean<RateLimitingFilter> rateLimitFilter() {
        FilterRegistrationBean<RateLimitingFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new RateLimitingFilter());
        registrationBean.addUrlPatterns("/api/*");
        registrationBean.setOrder(2);
        System.out.println("🚦 Rate limiting filter registered");
        return registrationBean;
    }
    
    @Bean
    public FilterRegistrationBean<AuthenticationFilter> authFilter() {
        FilterRegistrationBean<AuthenticationFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new AuthenticationFilter());
        registrationBean.addUrlPatterns("/api/*");
        registrationBean.setOrder(3);
        System.out.println("🔐 Authentication filter registered");
        return registrationBean;
    }
}
```

### Test Controller:
```java
@RestController
@RequestMapping("/api/test")
public class TestController {
    
    @GetMapping("/cors")
    public Map<String, Object> testCors(HttpServletRequest request) {
        Map<String, Object> response = new HashMap<>();
        response.put("message", "CORS is working!");
        response.put("origin", request.getHeader("Origin"));
        response.put("userAgent", request.getHeader("User-Agent"));
        response.put("timestamp", LocalDateTime.now());
        
        System.out.println("🧪 CORS test endpoint called");
        return response;
    }
    
    @PostMapping("/auth")
    public Map<String, Object> testAuth(HttpServletRequest request, @RequestBody Map<String, String> data) {
        Long userId = (Long) request.getAttribute("userId");
        String username = (String) request.getAttribute("username");
        
        Map<String, Object> response = new HashMap<>();
        response.put("message", "Authentication successful!");
        response.put("userId", userId);
        response.put("username", username);
        response.put("data", data);
        
        System.out.println("🔒 Authenticated endpoint called by user: " + username);
        return response;
    }
    
    @GetMapping("/rate-limit")
    public Map<String, Object> testRateLimit(HttpServletRequest request) {
        Map<String, Object> response = new HashMap<>();
        response.put("message", "Rate limit test successful!");
        response.put("clientIp", request.getRemoteAddr());
        response.put("timestamp", LocalDateTime.now());
        
        System.out.println("🚦 Rate limit test endpoint called");
        return response;
    }
}
```

---

## 🧪 Testing CORS & Filters

### Frontend Test (HTML):
```html
<!DOCTYPE html>
<html>
<head>
    <title>CORS Test</title>
</head>
<body>
    <h1>CORS Test Page</h1>
    <button onclick="testCors()">Test CORS</button>
    <button onclick="testAuth()">Test Auth</button>
    <div id="result"></div>

    <script>
        async function testCors() {
            try {
                const response = await fetch('http://localhost:8080/api/test/cors');
                const data = await response.json();
                document.getElementById('result').innerHTML = 
                    '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
            } catch (error) {
                document.getElementById('result').innerHTML = 
                    '<p style="color: red;">CORS Error: ' + error.message + '</p>';
            }
        }

        async function testAuth() {
            try {
                const response = await fetch('http://localhost:8080/api/test/auth', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer valid-token-123'
                    },
                    body: JSON.stringify({ message: 'Hello from frontend!' })
                });
                const data = await response.json();
                document.getElementById('result').innerHTML = 
                    '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
            } catch (error) {
                document.getElementById('result').innerHTML = 
                    '<p style="color: red;">Auth Error: ' + error.message + '</p>';
            }
        }
    </script>
</body>
</html>
```

### cURL Test Commands:
```bash
# Test CORS preflight
curl -X OPTIONS http://localhost:8080/api/test/cors \
  -H "Origin: http://localhost:3000" \
  -H "Access-Control-Request-Method: GET" \
  -v

# Test authenticated endpoint
curl -X POST http://localhost:8080/api/test/auth \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer valid-token-123" \
  -d '{"test": "data"}'

# Test rate limiting (run multiple times quickly)
for i in {1..70}; do
  curl -X GET http://localhost:8080/api/test/rate-limit
  echo " - Request $i"
done
```

---

## 📊 Filter vs Interceptor Comparison

| Aspect | Filter | Interceptor |
|--------|---------|-------------|
| **Level** | Servlet level | Spring MVC level |
| **When** | Before/after entire request | Around controller methods |
| **Access** | HttpServletRequest/Response | HandlerMethod, ModelAndView |
| **Use Case** | Authentication, CORS, logging | Performance monitoring, caching |
| **Exception Handling** | Limited | Full Spring context |

---

## 🎯 Interview Questions & Answers

### Q1: "What is CORS and why is it needed?"
**Answer:**
- **CORS** = Cross-Origin Resource Sharing
- **Browser security** feature that blocks requests between different domains
- **Prevents malicious sites** from accessing your API without permission
- **@CrossOrigin** annotation allows specific origins to access your endpoints

### Q2: "What's the difference between Filter and Interceptor?"
**Answer:**
- **Filter**: Servlet-level, processes all requests, limited Spring context access
- **Interceptor**: Spring MVC level, processes only controller requests, full Spring access
- **Use Filter for**: Authentication, CORS, request logging
- **Use Interceptor for**: Performance monitoring, caching, model manipulation

### Q3: "How do you configure CORS globally vs per endpoint?"
**Answer:**
```java
// Global CORS
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**").allowedOrigins("http://localhost:3000");
    }
}

// Per endpoint CORS
@CrossOrigin(origins = "http://localhost:3000")
@GetMapping("/users")
public List<User> getUsers() { }
```

---

## 🚀 Best Practices

### ✅ Do's:
```java
// 1. Specify exact origins in production
@CrossOrigin(origins = "https://myapp.com") // GOOD

// 2. Use filters for cross-cutting concerns
@Component
public class LoggingFilter implements Filter { } // GOOD

// 3. Order filters properly
@Order(1) // Authentication first
@Order(2) // Then rate limiting

// 4. Handle preflight requests
registry.addMapping("/api/**")
        .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS");

// 5. Configure appropriate headers
.allowedHeaders("Authorization", "Content-Type")
.allowCredentials(true);
```

### ❌ Don'ts:
```java
// 1. Don't use wildcard origins in production
@CrossOrigin(origins = "*") // BAD - security risk

// 2. Don't block the filter chain
public void doFilter(...) {
    // Do processing but don't call:
    // chain.doFilter(request, response); // BAD - blocks request
}

// 3. Don't ignore filter ordering
@Component
public class AuthFilter implements Filter { } // BAD - no order specified
```

---

