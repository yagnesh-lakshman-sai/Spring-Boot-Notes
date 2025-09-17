# 📅 Day 8: Spring MVC Basics

## 🎯 Today Topics
- Understand Spring MVC architecture and flow
- Master `@RestController` vs `@Controller`  
- Learn `@RequestMapping` and its variants
- Build real REST APIs with practical examples
- Handle different response formats (JSON, XML)

---

## 🏗️ Spring MVC Architecture Overview

### What is Spring MVC?
**Spring MVC** is Spring's web framework that follows the Model-View-Controller pattern. It provides a clean separation between web layer concerns and business logic.

### MVC Flow:
```
Client Request → DispatcherServlet → Handler Mapping → Controller → Service → Repository
                      ↑                                      ↓
                 View Resolver ← Model & View ← Controller Response
```

### Key Components:
- **DispatcherServlet**: Front controller that handles all requests
- **HandlerMapping**: Maps requests to appropriate controllers
- **Controller**: Handles the request and returns response
- **ViewResolver**: Resolves view names to actual views
- **Model**: Data container passed between controller and view

---

## 🎮 @Controller vs @RestController

### @Controller - Traditional MVC
```java
// Traditional controller returning views
@Controller
@RequestMapping("/web")
public class WebController {
    
    @RequestMapping("/home")
    public String home(Model model) {
        model.addAttribute("message", "Welcome to our website!");
        return "home"; // Returns view name (home.html/jsp)
    }
    
    @RequestMapping("/user/{id}")
    public String getUserProfile(@PathVariable Long id, Model model) {
        // Simulate user lookup
        User user = new User(id, "John Doe", "john@example.com");
        model.addAttribute("user", user);
        return "user-profile"; // user-profile.html
    }
}
```

### @RestController - REST API
```java
// REST controller returning data (JSON/XML)
@RestController
@RequestMapping("/api")
public class ApiController {
    
    @RequestMapping("/users")
    public List<User> getUsers() {
        // Returns JSON automatically
        return Arrays.asList(
            new User(1L, "John Doe", "john@example.com"),
            new User(2L, "Jane Smith", "jane@example.com")
        );
    }
    
    @RequestMapping("/user/{id}")
    public User getUser(@PathVariable Long id) {
        // Returns single user as JSON
        return new User(id, "John Doe", "john@example.com");
    }
}
```

### Key Differences:
| Aspect | @Controller | @RestController |
|--------|-------------|-----------------|
| **Purpose** | Web pages (MVC) | REST APIs (JSON/XML) |
| **Returns** | View names | Data objects |
| **Response** | HTML pages | JSON/XML |
| **Equivalent** | @Controller | @Controller + @ResponseBody |

---

## 🗺️ @RequestMapping - The Foundation

### Basic @RequestMapping:
```java
@RestController
public class BasicController {
    
    // Simple mapping
    @RequestMapping("/hello")
    public String hello() {
        return "Hello, World!";
    }
    
    // Multiple URLs for same method
    @RequestMapping({"/home", "/", "/index"})
    public String home() {
        return "Welcome to Home Page";
    }
    
    // Path with variables
    @RequestMapping("/user/{id}")
    public String getUser(@PathVariable Long id) {
        return "User ID: " + id;
    }
    
    // Multiple path variables
    @RequestMapping("/user/{userId}/order/{orderId}")
    public String getUserOrder(@PathVariable Long userId, @PathVariable Long orderId) {
        return String.format("User %d, Order %d", userId, orderId);
    }
}
```

### HTTP Method Specification:
```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    // GET method (default)
    @RequestMapping(method = RequestMethod.GET)
    public List<Product> getAllProducts() {
        return productService.findAll();
    }
    
    // POST method
    @RequestMapping(method = RequestMethod.POST)
    public Product createProduct(@RequestBody Product product) {
        return productService.save(product);
    }
    
    // PUT method  
    @RequestMapping(value = "/{id}", method = RequestMethod.PUT)
    public Product updateProduct(@PathVariable Long id, @RequestBody Product product) {
        return productService.update(id, product);
    }
    
    // DELETE method
    @RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
    public String deleteProduct(@PathVariable Long id) {
        productService.delete(id);
        return "Product deleted successfully";
    }
}
```

### Advanced @RequestMapping Features:
```java
@RestController
public class AdvancedController {
    
    // Content-Type specific mapping
    @RequestMapping(
        value = "/upload", 
        method = RequestMethod.POST,
        consumes = "multipart/form-data"
    )
    public String uploadFile(@RequestParam("file") MultipartFile file) {
        return "File uploaded: " + file.getOriginalFilename();
    }
    
    // Accept header specific mapping
    @RequestMapping(
        value = "/data", 
        produces = {"application/json", "application/xml"}
    )
    public DataResponse getData() {
        return new DataResponse("Sample data", LocalDateTime.now());
    }
    
    // Header-based mapping
    @RequestMapping(
        value = "/admin", 
        headers = "X-API-Version=1.0"
    )
    public String adminEndpoint() {
        return "Admin endpoint for API v1.0";
    }
    
    // Parameter-based mapping
    @RequestMapping(
        value = "/search", 
        params = "type=advanced"
    )
    public String advancedSearch() {
        return "Advanced search endpoint";
    }
}
```

---

## 🌟 Real-World Example: Social Media API

Let's build a complete social media REST API demonstrating all concepts:

```java
// User model
public class User {
    private Long id;
    private String username;
    private String email;
    private String fullName;
    private LocalDateTime createdAt;
    private int followersCount;
    private int followingCount;
    
    // Constructors
    public User() {
        this.createdAt = LocalDateTime.now();
    }
    
    public User(Long id, String username, String email, String fullName) {
        this();
        this.id = id;
        this.username = username;
        this.email = email;
        this.fullName = fullName;
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public String getFullName() { return fullName; }
    public void setFullName(String fullName) { this.fullName = fullName; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    public int getFollowersCount() { return followersCount; }
    public void setFollowersCount(int followersCount) { this.followersCount = followersCount; }
    
    public int getFollowingCount() { return followingCount; }
    public void setFollowingCount(int followingCount) { this.followingCount = followingCount; }
}

// Post model
public class Post {
    private Long id;
    private Long userId;
    private String content;
    private String imageUrl;
    private LocalDateTime createdAt;
    private int likesCount;
    private int commentsCount;
    private List<String> tags;
    
    // Constructors
    public Post() {
        this.createdAt = LocalDateTime.now();
        this.tags = new ArrayList<>();
    }
    
    public Post(Long id, Long userId, String content) {
        this();
        this.id = id;
        this.userId = userId;
        this.content = content;
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public Long getUserId() { return userId; }
    public void setUserId(Long userId) { this.userId = userId; }
    
    public String getContent() { return content; }
    public void setContent(String content) { this.content = content; }
    
    public String getImageUrl() { return imageUrl; }
    public void setImageUrl(String imageUrl) { this.imageUrl = imageUrl; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    public int getLikesCount() { return likesCount; }
    public void setLikesCount(int likesCount) { this.likesCount = likesCount; }
    
    public int getCommentsCount() { return commentsCount; }
    public void setCommentsCount(int commentsCount) { this.commentsCount = commentsCount; }
    
    public List<String> getTags() { return tags; }
    public void setTags(List<String> tags) { this.tags = tags; }
}

// Social Media API Controller
@RestController
@RequestMapping("/api/v1")
public class SocialMediaController {
    
    // Simulated in-memory storage (in real app, use database)
    private Map<Long, User> users = new ConcurrentHashMap<>();
    private Map<Long, Post> posts = new ConcurrentHashMap<>();
    private AtomicLong userIdGenerator = new AtomicLong(1);
    private AtomicLong postIdGenerator = new AtomicLong(1);
    
    // Initialize with sample data
    public SocialMediaController() {
        initializeSampleData();
    }
    
    // ================ USER ENDPOINTS ================
    
    // Get all users
    @RequestMapping(value = "/users", method = RequestMethod.GET)
    public List<User> getAllUsers() {
        System.out.println("📋 Getting all users - Total: " + users.size());
        return new ArrayList<>(users.values());
    }
    
    // Get user by ID
    @RequestMapping(value = "/users/{id}", method = RequestMethod.GET)
    public User getUserById(@PathVariable Long id) {
        System.out.println("👤 Getting user by ID: " + id);
        User user = users.get(id);
        if (user == null) {
            throw new RuntimeException("User not found with ID: " + id);
        }
        return user;
    }
    
    // Get user by username
    @RequestMapping(value = "/users", method = RequestMethod.GET, params = "username")
    public User getUserByUsername(@RequestParam String username) {
        System.out.println("🔍 Searching user by username: " + username);
        return users.values().stream()
                .filter(user -> user.getUsername().equalsIgnoreCase(username))
                .findFirst()
                .orElseThrow(() -> new RuntimeException("User not found: " + username));
    }
    
    // Create new user
    @RequestMapping(value = "/users", method = RequestMethod.POST)
    public User createUser(@RequestBody User user) {
        System.out.println("➕ Creating new user: " + user.getUsername());
        
        // Generate ID and set creation time
        user.setId(userIdGenerator.getAndIncrement());
        user.setCreatedAt(LocalDateTime.now());
        
        // Save user
        users.put(user.getId(), user);
        
        System.out.println("✅ User created successfully with ID: " + user.getId());
        return user;
    }
    
    // Update user
    @RequestMapping(value = "/users/{id}", method = RequestMethod.PUT)
    public User updateUser(@PathVariable Long id, @RequestBody User updatedUser) {
        System.out.println("✏️ Updating user: " + id);
        
        User existingUser = users.get(id);
        if (existingUser == null) {
            throw new RuntimeException("User not found with ID: " + id);
        }
        
        // Update fields (keep original ID and creation date)
        updatedUser.setId(id);
        updatedUser.setCreatedAt(existingUser.getCreatedAt());
        users.put(id, updatedUser);
        
        System.out.println("✅ User updated successfully");
        return updatedUser;
    }
    
    // Delete user
    @RequestMapping(value = "/users/{id}", method = RequestMethod.DELETE)
    public String deleteUser(@PathVariable Long id) {
        System.out.println("🗑️ Deleting user: " + id);
        
        User deletedUser = users.remove(id);
        if (deletedUser == null) {
            throw new RuntimeException("User not found with ID: " + id);
        }
        
        // Also delete user's posts
        posts.entrySet().removeIf(entry -> entry.getValue().getUserId().equals(id));
        
        System.out.println("✅ User and associated posts deleted successfully");
        return "User deleted successfully";
    }
    
    // ================ POST ENDPOINTS ================
    
    // Get all posts
    @RequestMapping(value = "/posts", method = RequestMethod.GET)
    public List<Post> getAllPosts() {
        System.out.println("📝 Getting all posts - Total: " + posts.size());
        return posts.values().stream()
                .sorted((p1, p2) -> p2.getCreatedAt().compareTo(p1.getCreatedAt())) // Latest first
                .collect(Collectors.toList());
    }
    
    // Get posts by user
    @RequestMapping(value = "/users/{userId}/posts", method = RequestMethod.GET)
    public List<Post> getPostsByUser(@PathVariable Long userId) {
        System.out.println("📝 Getting posts for user: " + userId);
        
        // Verify user exists
        if (!users.containsKey(userId)) {
            throw new RuntimeException("User not found with ID: " + userId);
        }
        
        return posts.values().stream()
                .filter(post -> post.getUserId().equals(userId))
                .sorted((p1, p2) -> p2.getCreatedAt().compareTo(p1.getCreatedAt()))
                .collect(Collectors.toList());
    }
    
    // Get single post
    @RequestMapping(value = "/posts/{id}", method = RequestMethod.GET)
    public Post getPostById(@PathVariable Long id) {
        System.out.println("📖 Getting post by ID: " + id);
        Post post = posts.get(id);
        if (post == null) {
            throw new RuntimeException("Post not found with ID: " + id);
        }
        return post;
    }
    
    // Create new post
    @RequestMapping(value = "/posts", method = RequestMethod.POST)
    public Post createPost(@RequestBody Post post) {
        System.out.println("➕ Creating new post for user: " + post.getUserId());
        
        // Verify user exists
        if (!users.containsKey(post.getUserId())) {
            throw new RuntimeException("User not found with ID: " + post.getUserId());
        }
        
        // Generate ID and set creation time
        post.setId(postIdGenerator.getAndIncrement());
        post.setCreatedAt(LocalDateTime.now());
        
        // Extract hashtags from content
        extractHashtags(post);
        
        // Save post
        posts.put(post.getId(), post);
        
        System.out.println("✅ Post created successfully with ID: " + post.getId());
        return post;
    }
    
    // Update post
    @RequestMapping(value = "/posts/{id}", method = RequestMethod.PUT)
    public Post updatePost(@PathVariable Long id, @RequestBody Post updatedPost) {
        System.out.println("✏️ Updating post: " + id);
        
        Post existingPost = posts.get(id);
        if (existingPost == null) {
            throw new RuntimeException("Post not found with ID: " + id);
        }
        
        // Update fields (keep original ID, user ID, and creation date)
        updatedPost.setId(id);
        updatedPost.setUserId(existingPost.getUserId());
        updatedPost.setCreatedAt(existingPost.getCreatedAt());
        
        // Extract hashtags from updated content
        extractHashtags(updatedPost);
        
        posts.put(id, updatedPost);
        
        System.out.println("✅ Post updated successfully");
        return updatedPost;
    }
    
    // Delete post
    @RequestMapping(value = "/posts/{id}", method = RequestMethod.DELETE)
    public String deletePost(@PathVariable Long id) {
        System.out.println("🗑️ Deleting post: " + id);
        
        Post deletedPost = posts.remove(id);
        if (deletedPost == null) {
            throw new RuntimeException("Post not found with ID: " + id);
        }
        
        System.out.println("✅ Post deleted successfully");
        return "Post deleted successfully";
    }
    
    // Like a post
    @RequestMapping(value = "/posts/{id}/like", method = RequestMethod.POST)
    public Post likePost(@PathVariable Long id) {
        System.out.println("❤️ Liking post: " + id);
        
        Post post = posts.get(id);
        if (post == null) {
            throw new RuntimeException("Post not found with ID: " + id);
        }
        
        post.setLikesCount(post.getLikesCount() + 1);
        System.out.println("✅ Post liked! Total likes: " + post.getLikesCount());
        
        return post;
    }
    
    // ================ SEARCH ENDPOINTS ================
    
    // Search posts by content
    @RequestMapping(value = "/posts/search", method = RequestMethod.GET)
    public List<Post> searchPosts(@RequestParam String query) {
        System.out.println("🔍 Searching posts with query: " + query);
        
        String searchQuery = query.toLowerCase();
        return posts.values().stream()
                .filter(post -> post.getContent().toLowerCase().contains(searchQuery))
                .sorted((p1, p2) -> p2.getCreatedAt().compareTo(p1.getCreatedAt()))
                .collect(Collectors.toList());
    }
    
    // Get posts by hashtag
    @RequestMapping(value = "/posts/hashtag/{tag}", method = RequestMethod.GET)
    public List<Post> getPostsByHashtag(@PathVariable String tag) {
        System.out.println("🏷️ Getting posts with hashtag: " + tag);
        
        String hashtag = tag.startsWith("#") ? tag : "#" + tag;
        return posts.values().stream()
                .filter(post -> post.getTags().contains(hashtag))
                .sorted((p1, p2) -> p2.getCreatedAt().compareTo(p1.getCreatedAt()))
                .collect(Collectors.toList());
    }
    
    // ================ ANALYTICS ENDPOINTS ================
    
    // Get platform statistics
    @RequestMapping(value = "/stats", method = RequestMethod.GET)
    public Map<String, Object> getPlatformStats() {
        System.out.println("📊 Getting platform statistics");
        
        Map<String, Object> stats = new HashMap<>();
        stats.put("totalUsers", users.size());
        stats.put("totalPosts", posts.size());
        stats.put("totalLikes", posts.values().stream().mapToInt(Post::getLikesCount).sum());
        stats.put("totalComments", posts.values().stream().mapToInt(Post::getCommentsCount).sum());
        stats.put("mostActiveUser", getMostActiveUser());
        stats.put("popularHashtags", getPopularHashtags());
        
        return stats;
    }
    
    // ================ HELPER METHODS ================
    
    private void initializeSampleData() {
        // Create sample users
        User user1 = new User(1L, "john_doe", "john@example.com", "John Doe");
        user1.setFollowersCount(150);
        user1.setFollowingCount(75);
        users.put(1L, user1);
        
        User user2 = new User(2L, "jane_smith", "jane@example.com", "Jane Smith");
        user2.setFollowersCount(200);
        user2.setFollowingCount(100);
        users.put(2L, user2);
        
        User user3 = new User(3L, "tech_guru", "guru@example.com", "Tech Guru");
        user3.setFollowersCount(500);
        user3.setFollowingCount(50);
        users.put(3L, user3);
        
        // Create sample posts
        Post post1 = new Post(1L, 1L, "Just finished an amazing Spring Boot project! #springboot #java #coding");
        post1.setLikesCount(25);
        post1.setCommentsCount(5);
        posts.put(1L, post1);
        
        Post post2 = new Post(2L, 2L, "Beautiful sunset today! Nature never fails to amaze me. #sunset #nature #photography");
        post2.setLikesCount(45);
        post2.setCommentsCount(8);
        posts.put(2L, post2);
        
        Post post3 = new Post(3L, 3L, "Top 5 Java frameworks every developer should know #java #frameworks #webdev");
        post3.setLikesCount(100);
        post3.setCommentsCount(20);
        posts.put(3L, post3);
        
        // Update generators
        userIdGenerator.set(4L);
        postIdGenerator.set(4L);
        
        // Extract hashtags for all posts
        posts.values().forEach(this::extractHashtags);
        
        System.out.println("✅ Sample data initialized - " + users.size() + " users, " + posts.size() + " posts");
    }
    
    private void extractHashtags(Post post) {
        List<String> hashtags = new ArrayList<>();
        String[] words = post.getContent().split("\\s+");
        
        for (String word : words) {
            if (word.startsWith("#") && word.length() > 1) {
                hashtags.add(word.toLowerCase());
            }
        }
        
        post.setTags(hashtags);
    }
    
    private String getMostActiveUser() {
        return users.values().stream()
                .max((u1, u2) -> {
                    int u1Posts = (int) posts.values().stream().filter(p -> p.getUserId().equals(u1.getId())).count();
                    int u2Posts = (int) posts.values().stream().filter(p -> p.getUserId().equals(u2.getId())).count();
                    return Integer.compare(u1Posts, u2Posts);
                })
                .map(User::getUsername)
                .orElse("None");
    }
    
    private List<String> getPopularHashtags() {
        Map<String, Long> tagCounts = posts.values().stream()
                .flatMap(post -> post.getTags().stream())
                .collect(Collectors.groupingBy(tag -> tag, Collectors.counting()));
        
        return tagCounts.entrySet().stream()
                .sorted((e1, e2) -> Long.compare(e2.getValue(), e1.getValue()))
                .limit(5)
                .map(Map.Entry::getKey)
                .collect(Collectors.toList());
    }
}
```

# Postman API Collection - Social Media Platform

## 1. Get All Users
**Method:** `GET`  
**URL:** `http://localhost:8080/api/v1/users`  
**Headers:** None required  
**Body:** None  

---

## 2. Create a New User
**Method:** `POST`  
**URL:** `http://localhost:8080/api/v1/users`  
**Headers:**
- `Content-Type: application/json`

**Body (JSON):**
```json
{
  "username": "new_user",
  "email": "newuser@example.com",
  "fullName": "New User"
}
```

---

## 3. Get User by ID
**Method:** `GET`  
**URL:** `http://localhost:8080/api/v1/users/1`  
**Headers:** None required  
**Body:** None  

---

## 4. Create a New Post
**Method:** `POST`  
**URL:** `http://localhost:8080/api/v1/posts`  
**Headers:**
- `Content-Type: application/json`

**Body (JSON):**
```json
{
  "userId": 1,
  "content": "Learning Spring Boot is amazing! #springboot #learning"
}
```

---

## 5. Like a Post
**Method:** `POST`  
**URL:** `http://localhost:8080/api/v1/posts/1/like`  
**Headers:** None required  
**Body:** None  

---

## 6. Search Posts
**Method:** `GET`  
**URL:** `http://localhost:8080/api/v1/posts/search`  
**Headers:** None required  
**Query Parameters:**
- `query: spring`

**Alternative URL with query parameter:**
`http://localhost:8080/api/v1/posts/search?query=spring`

---

## 7. Get Platform Stats
**Method:** `GET`  
**URL:** `http://localhost:8080/api/v1/stats`  
**Headers:** None required  
**Body:** None  

---

## How to Import into Postman:

1. **Option 1 - Manual Setup:**
   - Create a new collection in Postman
   - Add each request manually using the details above

2. **Option 2 - JSON Import:**
   - Copy the JSON collection format below and import it into Postman

### Postman Collection JSON Format:
```json
{
  "info": {
    "name": "Social Media Platform API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "variable": [
    {
      "key": "baseUrl",
      "value": "http://localhost:8080/api/v1"
    }
  ],
  "item": [
    {
      "name": "Get All Users",
      "request": {
        "method": "GET",
        "url": "{{baseUrl}}/users"
      }
    },
    {
      "name": "Create New User",
      "request": {
        "method": "POST",
        "url": "{{baseUrl}}/users",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n  \"username\": \"new_user\",\n  \"email\": \"newuser@example.com\",\n  \"fullName\": \"New User\"\n}"
        }
      }
    },
    {
      "name": "Get User by ID",
      "request": {
        "method": "GET",
        "url": "{{baseUrl}}/users/1"
      }
    },
    {
      "name": "Create New Post",
      "request": {
        "method": "POST",
        "url": "{{baseUrl}}/posts",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n  \"userId\": 1,\n  \"content\": \"Learning Spring Boot is amazing! #springboot #learning\"\n}"
        }
      }
    },
    {
      "name": "Like a Post",
      "request": {
        "method": "POST",
        "url": "{{baseUrl}}/posts/1/like"
      }
    },
    {
      "name": "Search Posts",
      "request": {
        "method": "GET",
        "url": "{{baseUrl}}/posts/search",
        "url": {
          "query": [
            {
              "key": "query",
              "value": "spring"
            }
          ]
        }
      }
    },
    {
      "name": "Get Platform Stats",
      "request": {
        "method": "GET",
        "url": "{{baseUrl}}/stats"
      }
    }
  ]
}
```

## Tips for Postman Usage:
- Set up an environment variable `baseUrl` = `http://localhost:8080/api/v1` for easier management
- Use the Tests tab to add assertions for response validation
- Save responses as examples for documentation
- Use the Pre-request Script tab for any setup logic if needed

---

## 🎯 Interview Questions & Answers

### Q1: "What's the difference between @Controller and @RestController?"
**Answer:**
- **@Controller**: Returns view names for traditional MVC, needs @ResponseBody for JSON
- **@RestController**: Combines @Controller + @ResponseBody, automatically converts objects to JSON/XML
- **Use @Controller for web pages, @RestController for REST APIs**

### Q2: "Explain the Spring MVC request flow."
**Answer:**
1. **DispatcherServlet** receives request
2. **HandlerMapping** finds appropriate controller method  
3. **Controller** processes request and returns ModelAndView/data
4. **ViewResolver** resolves view name (if needed)
5. **View** renders response back to client

### Q3: "How does @RequestMapping work?"
**Answer:**
- Maps HTTP requests to controller methods
- Can specify HTTP method, URL pattern, headers, parameters
- Supports path variables with @PathVariable
- Class-level mapping acts as base URL for all methods

### Q4: "What happens when Spring Boot auto-configures MVC?"
**Answer:**
- **DispatcherServlet** registered automatically
- **Default message converters** for JSON/XML
- **Static resource handling** (/static, /public, /resources)
- **Exception handling** with default error pages
- **Content negotiation** based on Accept headers

### Q5: "How do you handle different response formats?"
**Answer:**
```java
@RequestMapping(produces = {"application/json", "application/xml"})
public User getUser() {
    return user; // Auto-converts based on Accept header
}
```

---

## 🚀 Best Practices

### 1. **RESTful URL Design**
```java
// ✅ Good: RESTful URLs
GET    /api/users           // Get all users
GET    /api/users/123       // Get specific user
POST   /api/users           // Create new user  
PUT    /api/users/123       // Update user
DELETE /api/users/123       // Delete user

// ❌ Avoid: Non-RESTful URLs  
GET /api/getAllUsers
POST /api/createUser
POST /api/updateUser/123
POST /api/deleteUser/123
```

### 2. **Controller Organization**
```java
// ✅ Good: Organized by resource
@RestController
@RequestMapping("/api/users")
public class UserController {
    // All user-related endpoints
}

@RestController
@RequestMapping("/api/orders") 
public class OrderController {
    // All order-related endpoints
}
```

### 3. **Error Handling**
```java
@RestController
public class UserController {
    
    @RequestMapping("/users/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        try {
            User user = userService.findById(id);
            return ResponseEntity.ok(user);
        } catch (UserNotFoundException e) {
            return ResponseEntity.notFound().build();
        }
    }
}
```

