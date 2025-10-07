# 📅 Day 20: Week 3 Review & Final Project

## 🎯 Today's Objectives
- **Week 3 Complete Review**: Data & Persistence layer summary
- **Final Project**: Complete Blog System with all concepts
- **20-Day Journey Recap**: All Spring Boot concepts mastered
- **Interview Preparation**: Final cheat sheet and tips
- **Next Steps**: Your Spring Boot mastery roadmap

---

## 📚 Week 3 Complete Summary

### Day 15: Spring Data JPA Basics ✅
```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @PrePersist
    public void onCreate() {
        createdAt = LocalDateTime.now();
    }
}
```
**Key Takeaway**: @Entity maps Java classes to database tables, JPA handles persistence automatically.

---

### Day 16: Repository Layer ✅
```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Method naming - Spring generates query automatically
    User findByEmail(String email);
    List<User> findByAgeGreaterThan(Integer age);
    
    // Custom query with JPQL
    @Query("SELECT u FROM User u WHERE u.name LIKE %:keyword%")
    List<User> searchByName(@Param("keyword") String keyword);
    
    // Pagination
    Page<User> findAll(Pageable pageable);
}
```
**Key Takeaway**: JpaRepository provides CRUD for free, method naming generates queries, use @Query for complex operations.

---

### Day 17: Entity Relationships ✅
```java
// One-to-One
@OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
private UserProfile profile;

// One-to-Many / Many-to-One
@OneToMany(mappedBy = "author", cascade = CascadeType.ALL)
private List<Book> books;

@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "author_id")
private Author author;

// Many-to-Many
@ManyToMany
@JoinTable(name = "student_courses",
    joinColumns = @JoinColumn(name = "student_id"),
    inverseJoinColumns = @JoinColumn(name = "course_id"))
private Set<Course> courses;
```
**Key Takeaway**: Model real-world relationships, use LAZY fetching, maintain bidirectional relationships properly.

---

### Day 18: Transaction Management ✅
```java
@Service
@Transactional
public class OrderService {
    
    @Transactional(
        propagation = Propagation.REQUIRED,
        isolation = Isolation.READ_COMMITTED,
        rollbackFor = Exception.class
    )
    public Order processOrder(OrderRequest request) {
        // All operations succeed together or fail together
        inventoryService.reserveItems(request.getItems());
        paymentService.processPayment(request.getPayment());
        return orderRepository.save(order);
    }
    
    @Transactional(readOnly = true)
    public List<Order> findAll() {
        return orderRepository.findAll();
    }
}
```
**Key Takeaway**: @Transactional ensures data consistency, use readOnly for queries, specify rollback rules.

---

### Day 19: Caching ✅
```java
@Service
public class ProductService {
    
    @Cacheable(value = "products", key = "#id")
    public Product findById(Long id) {
        // Cached after first call
        return productRepository.findById(id).orElseThrow();
    }
    
    @CachePut(value = "products", key = "#result.id")
    public Product updateProduct(Product product) {
        // Updates cache with new value
        return productRepository.save(product);
    }
    
    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(Long id) {
        // Removes from cache
        productRepository.deleteById(id);
    }
}
```
**Key Takeaway**: Cache expensive operations, set TTL, evict on updates for performance optimization.

---

## 🎯 Final Project: Complete Blog System

### Project Overview:
**Features**:
- User authentication and profiles
- Blog post CRUD with categories
- Comments and likes
- Caching for performance
- Transactions for consistency
- Full REST API

### Entity Models:

```java
// User.java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String username;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private UserProfile profile;
    
    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Post> posts = new ArrayList<>();
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }
    
    // Constructors, getters, setters
    public User() {}
    
    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }
    
    public void addPost(Post post) {
        posts.add(post);
        post.setAuthor(this);
    }
    
    // Getters and setters...
}

// UserProfile.java
@Entity
@Table(name = "user_profiles")
public class UserProfile {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String bio;
    private String avatarUrl;
    private String location;
    
    @OneToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    // Constructors, getters, setters...
}

// Post.java
@Entity
@Table(name = "posts", indexes = {
    @Index(name = "idx_author", columnList = "author_id"),
    @Index(name = "idx_category", columnList = "category_id")
})
public class Post {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 200)
    private String title;
    
    @Column(nullable = false, columnDefinition = "TEXT")
    private String content;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id", nullable = false)
    private User author;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;
    
    @OneToMany(mappedBy = "post", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();
    
    @Column(name = "likes_count")
    private Integer likesCount = 0;
    
    @Column(name = "views_count")
    private Integer viewsCount = 0;
    
    private Boolean published = false;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
    
    public void addComment(Comment comment) {
        comments.add(comment);
        comment.setPost(this);
    }
    
    public void incrementViews() {
        this.viewsCount++;
    }
    
    public void incrementLikes() {
        this.likesCount++;
    }
    
    // Constructors, getters, setters...
}

// Category.java
@Entity
@Table(name = "categories")
public class Category {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String name;
    
    private String description;
    
    @OneToMany(mappedBy = "category")
    private List<Post> posts = new ArrayList<>();
    
    // Constructors, getters, setters...
}

// Comment.java
@Entity
@Table(name = "comments")
public class Comment {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, columnDefinition = "TEXT")
    private String content;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id", nullable = false)
    private Post post;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }
    
    // Constructors, getters, setters...
}
```

### Repositories:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
}

public interface PostRepository extends JpaRepository<Post, Long> {
    List<Post> findByAuthorId(Long authorId);
    List<Post> findByCategoryId(Long categoryId);
    List<Post> findByPublishedTrue();
    
    @Query("SELECT p FROM Post p WHERE p.published = true ORDER BY p.createdAt DESC")
    Page<Post> findPublishedPosts(Pageable pageable);
    
    @Query("SELECT p FROM Post p WHERE p.published = true AND " +
           "(p.title LIKE %:keyword% OR p.content LIKE %:keyword%)")
    List<Post> searchPublishedPosts(@Param("keyword") String keyword);
    
    @Query("SELECT p FROM Post p WHERE p.published = true ORDER BY p.viewsCount DESC")
    List<Post> findPopularPosts(Pageable pageable);
}

public interface CategoryRepository extends JpaRepository<Category, Long> {
    Optional<Category> findByName(String name);
}

public interface CommentRepository extends JpaRepository<Comment, Long> {
    List<Comment> findByPostId(Long postId);
    long countByPostId(Long postId);
}
```

### Services with Caching & Transactions:

```java
@Service
@Transactional
public class PostService {
    
    @Autowired
    private PostRepository postRepository;
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private CategoryRepository categoryRepository;
    
    // Create post (transactional)
    @CachePut(value = "posts", key = "#result.id")
    @CacheEvict(value = "allPosts", allEntries = true)
    public Post createPost(Long userId, PostRequest request) {
        User author = userRepository.findById(userId)
                .orElseThrow(() -> new UserNotFoundException(userId));
        
        Category category = categoryRepository.findById(request.getCategoryId())
                .orElseThrow(() -> new CategoryNotFoundException(request.getCategoryId()));
        
        Post post = new Post();
        post.setTitle(request.getTitle());
        post.setContent(request.getContent());
        post.setAuthor(author);
        post.setCategory(category);
        post.setPublished(false);
        
        return postRepository.save(post);
    }
    
    // Get post with caching
    @Cacheable(value = "posts", key = "#id")
    @Transactional(readOnly = true)
    public Post getPostById(Long id) {
        Post post = postRepository.findById(id)
                .orElseThrow(() -> new PostNotFoundException(id));
        
        // Increment views (separate transaction)
        incrementViews(id);
        
        return post;
    }
    
    // Get all published posts with pagination and caching
    @Cacheable(value = "allPosts")
    @Transactional(readOnly = true)
    public Page<Post> getPublishedPosts(int page, int size) {
        Pageable pageable = PageRequest.of(page, size);
        return postRepository.findPublishedPosts(pageable);
    }
    
    // Update post
    @CachePut(value = "posts", key = "#id")
    @CacheEvict(value = "allPosts", allEntries = true)
    public Post updatePost(Long id, PostRequest request) {
        Post post = postRepository.findById(id)
                .orElseThrow(() -> new PostNotFoundException(id));
        
        post.setTitle(request.getTitle());
        post.setContent(request.getContent());
        
        if (request.getCategoryId() != null) {
            Category category = categoryRepository.findById(request.getCategoryId())
                    .orElseThrow(() -> new CategoryNotFoundException(request.getCategoryId()));
            post.setCategory(category);
        }
        
        return postRepository.save(post);
    }
    
    // Publish post
    @CachePut(value = "posts", key = "#id")
    @CacheEvict(value = "allPosts", allEntries = true)
    public Post publishPost(Long id) {
        Post post = postRepository.findById(id)
                .orElseThrow(() -> new PostNotFoundException(id));
        
        post.setPublished(true);
        return postRepository.save(post);
    }
    
    // Delete post
    @CacheEvict(value = {"posts", "allPosts"}, allEntries = true)
    public void deletePost(Long id) {
        if (!postRepository.existsById(id)) {
            throw new PostNotFoundException(id);
        }
        postRepository.deleteById(id);
    }
    
    // Like post (separate transaction for high frequency)
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void likePost(Long postId) {
        Post post = postRepository.findById(postId)
                .orElseThrow(() -> new PostNotFoundException(postId));
        
        post.incrementLikes();
        postRepository.save(post);
        
        // Clear cache for this post
        clearPostCache(postId);
    }
    
    // Increment views (separate transaction)
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void incrementViews(Long postId) {
        Post post = postRepository.findById(postId)
                .orElseThrow(() -> new PostNotFoundException(postId));
        
        post.incrementViews();
        postRepository.save(post);
    }
    
    // Search posts
    @Cacheable(value = "postSearch", key = "#keyword")
    @Transactional(readOnly = true)
    public List<Post> searchPosts(String keyword) {
        return postRepository.searchPublishedPosts(keyword);
    }
    
    @Autowired
    private CacheManager cacheManager;
    
    private void clearPostCache(Long postId) {
        Cache cache = cacheManager.getCache("posts");
        if (cache != null) {
            cache.evict(postId);
        }
    }
}

@Service
@Transactional
public class CommentService {
    
    @Autowired
    private CommentRepository commentRepository;
    
    @Autowired
    private PostRepository postRepository;
    
    @Autowired
    private UserRepository userRepository;
    
    // Add comment
    public Comment addComment(Long postId, Long userId, String content) {
        Post post = postRepository.findById(postId)
                .orElseThrow(() -> new PostNotFoundException(postId));
        
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new UserNotFoundException(userId));
        
        Comment comment = new Comment();
        comment.setContent(content);
        comment.setPost(post);
        comment.setUser(user);
        
        return commentRepository.save(comment);
    }
    
    // Get comments for post
    @Transactional(readOnly = true)
    public List<Comment> getPostComments(Long postId) {
        return commentRepository.findByPostId(postId);
    }
    
    // Delete comment
    public void deleteComment(Long commentId) {
        if (!commentRepository.existsById(commentId)) {
            throw new CommentNotFoundException(commentId);
        }
        commentRepository.deleteById(commentId);
    }
}
```

### REST Controllers:

```java
@RestController
@RequestMapping("/api/posts")
@CrossOrigin(origins = "http://localhost:3000")
public class PostController {
    
    @Autowired
    private PostService postService;
    
    @Autowired
    private CommentService commentService;
    
    @GetMapping
    public ResponseEntity<Page<Post>> getAllPosts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        return ResponseEntity.ok(postService.getPublishedPosts(page, size));
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Post> getPost(@PathVariable Long id) {
        return ResponseEntity.ok(postService.getPostById(id));
    }
    
    @GetMapping("/search")
    public ResponseEntity<List<Post>> searchPosts(@RequestParam String keyword) {
        return ResponseEntity.ok(postService.searchPosts(keyword));
    }
    
    @PostMapping
    public ResponseEntity<Post> createPost(@RequestBody PostRequest request) {
        Post post = postService.createPost(request.getUserId(), request);
        return ResponseEntity.status(HttpStatus.CREATED).body(post);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Post> updatePost(@PathVariable Long id, @RequestBody PostRequest request) {
        return ResponseEntity.ok(postService.updatePost(id, request));
    }
    
    @PostMapping("/{id}/publish")
    public ResponseEntity<Post> publishPost(@PathVariable Long id) {
        return ResponseEntity.ok(postService.publishPost(id));
    }
    
    @PostMapping("/{id}/like")
    public ResponseEntity<String> likePost(@PathVariable Long id) {
        postService.likePost(id);
        return ResponseEntity.ok("Post liked successfully");
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<String> deletePost(@PathVariable Long id) {
        postService.deletePost(id);
        return ResponseEntity.ok("Post deleted successfully");
    }
    
    // Comments endpoints
    @GetMapping("/{id}/comments")
    public ResponseEntity<List<Comment>> getComments(@PathVariable Long id) {
        return ResponseEntity.ok(commentService.getPostComments(id));
    }
    
    @PostMapping("/{id}/comments")
    public ResponseEntity<Comment> addComment(
            @PathVariable Long id,
            @RequestBody CommentRequest request) {
        Comment comment = commentService.addComment(id, request.getUserId(), request.getContent());
        return ResponseEntity.status(HttpStatus.CREATED).body(comment);
    }
}
```

---

## 🎓 20-Day Learning Journey Recap

### 📊 Skills Mastered:

**Week 1: Core Foundations** (7 days)
- ✅ Spring Boot fundamentals and auto-configuration
- ✅ Dependency Injection with stereotypes
- ✅ Bean configuration and scopes
- ✅ Autowiring strategies
- ✅ Application properties management
- ✅ Profile-based configurations
- ✅ Mini e-commerce project

**Week 2: REST & Web Layer** (7 days)
- ✅ Spring MVC architecture
- ✅ HTTP method annotations
- ✅ Request/response handling
- ✅ Global exception handling
- ✅ Bean validation
- ✅ CORS and filters
- ✅ Task management REST API

**Week 3: Data & Persistence** (6 days)
- ✅ Spring Data JPA entities
- ✅ Repository layer and queries
- ✅ Entity relationships
- ✅ Transaction management
- ✅ Caching strategies
- ✅ Complete blog system

---

## 🎯 Final Interview Cheat Sheet

### Quick Reference:
```
🏗️ CORE SPRING BOOT:
@SpringBootApplication → Auto-config + component scan
@Component/@Service/@Repository/@Controller → Stereotypes
@Autowired → Dependency injection (prefer constructor)
@Configuration + @Bean → Manual bean creation
@Value/@ConfigurationProperties → External properties
@Profile → Environment-specific beans

🌐 REST API:
@RestController → @Controller + @ResponseBody
@GetMapping/@PostMapping/@PutMapping/@DeleteMapping
@RequestParam → Query parameters
@PathVariable → URL path variables
@RequestBody → JSON request body
@Valid → Enable validation

⚠️ ERROR HANDLING:
@ExceptionHandler → Method-level errors
@ControllerAdvice → Global error handling
ResponseEntity → HTTP status control

✅ VALIDATION:
@NotNull/@NotBlank/@Size/@Email → Field validation
@Valid → Trigger validation
MethodArgumentNotValidException → Validation errors

🗄️ DATA & JPA:
@Entity/@Table → Database mapping
@Id/@GeneratedValue → Primary key
@Column → Column constraints
@OneToOne/@OneToMany/@ManyToOne/@ManyToMany → Relationships
JpaRepository → Built-in CRUD methods
@Query → Custom JPQL/SQL queries

💼 TRANSACTIONS:
@Transactional → Atomic operations
Propagation.REQUIRED → Default (join or create)
Propagation.REQUIRES_NEW → Always new transaction
readOnly = true → Query optimization
rollbackFor = Exception.class → Rollback rules

⚡ CACHING:
@EnableCaching → Enable caching
@Cacheable → Cache results
@CachePut → Update cache
@CacheEvict → Remove from cache
TTL → Expiration time
```
---
## 📝 Final Recap

### 🎯 What You've Built:
1. **E-Commerce Order System** (Week 1)
2. **Task Management API** (Week 2)
3. **Blog System with Full Features** (Week 3)

### 💪 Skills You've Mastered:
- ✅ Complete REST API development
- ✅ Database design with JPA
- ✅ Transaction management
- ✅ Performance optimization with caching
- ✅ Error handling and validation
- ✅ Production-ready patterns

### 🏆 You Can Now:
- Build enterprise-grade Spring Boot applications
- Design and implement RESTful APIs
- Model complex database relationships
- Ensure data consistency with transactions
- Optimize performance with caching
- Handle errors gracefully
- Deploy production-ready applications
---
