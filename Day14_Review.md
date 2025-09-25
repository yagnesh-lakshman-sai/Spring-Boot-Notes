# 📅 Day 14: Week 2 Review & Complete REST API Project

## 🎯 Today's Objectives
- **Week 2 Complete Review**: All REST & Web Layer concepts
- **Complete Project**: Task Management API combining all concepts
- **Interview Cheat Sheet**: Quick reference for all topics

---

## 📚 Week 2 Complete Summary

### Day 8: Spring MVC Basics ✅
```java
@RestController  // = @Controller + @ResponseBody
@RequestMapping("/api")
public class ApiController {
    @RequestMapping("/users")
    public List<User> getUsers() {
        return userService.findAll(); // Automatically converts to JSON
    }
}
```
**Key Takeaway**: @RestController for APIs, @Controller for web pages.

---

### Day 9: HTTP Method Annotations ✅
```java
@GetMapping("/users")           // Retrieve data
@PostMapping("/users")          // Create resource
@PutMapping("/users/{id}")      // Replace entire resource  
@PatchMapping("/users/{id}")    // Partial update
@DeleteMapping("/users/{id}")   // Remove resource
```
**Key Takeaway**: Use appropriate HTTP methods with correct status codes.

---

### Day 10: Request & Response Handling ✅
```java
@GetMapping("/search")
public List<User> search(@RequestParam String name) { }  // Query params

@GetMapping("/users/{id}")  
public User getUser(@PathVariable Long id) { }           // Path variables

@PostMapping("/users")
public User create(@RequestBody User user) { }           // Request body

// File upload
@PostMapping("/upload")
public String upload(@RequestParam("file") MultipartFile file) { }
```
**Key Takeaway**: @RequestParam for queries, @PathVariable for paths, @RequestBody for objects.

---

### Day 11: Exception Handling ✅
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        return ResponseEntity.notFound().body(new ErrorResponse(ex.getMessage()));
    }
}
```
**Key Takeaway**: Use @ControllerAdvice for global error handling.

---

### Day 12: Validation ✅
```java
public class User {
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50)
    private String name;
    
    @Email(message = "Invalid email format")
    private String email;
}

@PostMapping("/users")
public User createUser(@Valid @RequestBody User user) { }
```
**Key Takeaway**: Use @Valid with validation annotations for data integrity.

---

### Day 13: CORS & Filters ✅
```java
@CrossOrigin(origins = "http://localhost:3000")
@RestController
public class ApiController { }

@Component
public class AuthFilter implements Filter {
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        // Authentication logic
        chain.doFilter(request, response);
    }
}
```
**Key Takeaway**: @CrossOrigin for browser security, Filters for cross-cutting concerns.

---

## 🏗️ Complete Project: Task Management API

Let's build a complete REST API that combines ALL Week 2 concepts:

### Project Structure:
```
src/main/java/com/taskmanager/
├── TaskManagerApplication.java
├── model/
│   ├── Task.java
│   ├── User.java
│   └── TaskStatus.java
├── dto/
│   ├── TaskRequest.java
│   ├── TaskResponse.java
│   └── ErrorResponse.java
├── controller/
│   ├── TaskController.java
│   └── UserController.java
├── service/
│   ├── TaskService.java
│   └── UserService.java
├── exception/
│   ├── TaskNotFoundException.java
│   ├── UserNotFoundException.java
│   └── GlobalExceptionHandler.java
├── filter/
│   ├── RequestLoggingFilter.java
│   └── AuthenticationFilter.java
└── config/
    └── WebConfig.java
```

### Models with Validation:
```java
// Task.java
public class Task {
    private Long id;
    private String title;
    private String description;
    private TaskStatus status;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    private LocalDateTime dueDate;
    private Long assignedUserId;
    
    // Constructors
    public Task() {
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
        this.status = TaskStatus.TODO;
    }
    
    public Task(String title, String description, LocalDateTime dueDate) {
        this();
        this.title = title;
        this.description = description;
        this.dueDate = dueDate;
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    
    public TaskStatus getStatus() { return status; }
    public void setStatus(TaskStatus status) { 
        this.status = status; 
        this.updatedAt = LocalDateTime.now();
    }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
    
    public LocalDateTime getDueDate() { return dueDate; }
    public void setDueDate(LocalDateTime dueDate) { this.dueDate = dueDate; }
    
    public Long getAssignedUserId() { return assignedUserId; }
    public void setAssignedUserId(Long assignedUserId) { this.assignedUserId = assignedUserId; }
}

// TaskStatus.java
public enum TaskStatus {
    TODO, IN_PROGRESS, DONE, CANCELLED
}

// User.java
public class User {
    private Long id;
    private String name;
    private String email;
    private LocalDateTime createdAt;
    
    // Constructors, getters, setters
    public User() {
        this.createdAt = LocalDateTime.now();
    }
    
    public User(String name, String email) {
        this();
        this.name = name;
        this.email = email;
    }
    
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}
```

### Request/Response DTOs with Validation:
```java
// TaskRequest.java
public class TaskRequest {
    
    @NotBlank(message = "Title is required")
    @Size(min = 3, max = 100, message = "Title must be between 3 and 100 characters")
    private String title;
    
    @Size(max = 500, message = "Description cannot exceed 500 characters")
    private String description;
    
    @Future(message = "Due date must be in the future")
    private LocalDateTime dueDate;
    
    private Long assignedUserId;
    
    // Constructors, getters, setters
    public TaskRequest() {}
    
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    
    public LocalDateTime getDueDate() { return dueDate; }
    public void setDueDate(LocalDateTime dueDate) { this.dueDate = dueDate; }
    
    public Long getAssignedUserId() { return assignedUserId; }
    public void setAssignedUserId(Long assignedUserId) { this.assignedUserId = assignedUserId; }
}

// TaskResponse.java
public class TaskResponse {
    private Long id;
    private String title;
    private String description;
    private TaskStatus status;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    private LocalDateTime dueDate;
    private User assignedUser;
    private boolean overdue;
    
    // Constructors
    public TaskResponse() {}
    
    public TaskResponse(Task task, User assignedUser) {
        this.id = task.getId();
        this.title = task.getTitle();
        this.description = task.getDescription();
        this.status = task.getStatus();
        this.createdAt = task.getCreatedAt();
        this.updatedAt = task.getUpdatedAt();
        this.dueDate = task.getDueDate();
        this.assignedUser = assignedUser;
        this.overdue = task.getDueDate() != null && task.getDueDate().isBefore(LocalDateTime.now()) 
                       && !task.getStatus().equals(TaskStatus.DONE);
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    
    public TaskStatus getStatus() { return status; }
    public void setStatus(TaskStatus status) { this.status = status; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
    
    public LocalDateTime getDueDate() { return dueDate; }
    public void setDueDate(LocalDateTime dueDate) { this.dueDate = dueDate; }
    
    public User getAssignedUser() { return assignedUser; }
    public void setAssignedUser(User assignedUser) { this.assignedUser = assignedUser; }
    
    public boolean isOverdue() { return overdue; }
    public void setOverdue(boolean overdue) { this.overdue = overdue; }
}
```

### Services with Business Logic:
```java
// TaskService.java
@Service
public class TaskService {
    
    private final Map<Long, Task> tasks = new ConcurrentHashMap<>();
    private final UserService userService;
    private final AtomicLong idGenerator = new AtomicLong(1);
    
    public TaskService(UserService userService) {
        this.userService = userService;
        initializeSampleData();
    }
    
    public List<TaskResponse> findAll() {
        return tasks.values().stream()
                .map(this::convertToResponse)
                .sorted((t1, t2) -> t2.getCreatedAt().compareTo(t1.getCreatedAt()))
                .collect(Collectors.toList());
    }
    
    public TaskResponse findById(Long id) {
        Task task = tasks.get(id);
        if (task == null) {
            throw new TaskNotFoundException(id);
        }
        return convertToResponse(task);
    }
    
    public List<TaskResponse> findByStatus(TaskStatus status) {
        return tasks.values().stream()
                .filter(task -> task.getStatus() == status)
                .map(this::convertToResponse)
                .collect(Collectors.toList());
    }
    
    public List<TaskResponse> findByUserId(Long userId) {
        return tasks.values().stream()
                .filter(task -> task.getAssignedUserId() != null && task.getAssignedUserId().equals(userId))
                .map(this::convertToResponse)
                .collect(Collectors.toList());
    }
    
    public List<TaskResponse> searchTasks(String query) {
        String searchQuery = query.toLowerCase();
        return tasks.values().stream()
                .filter(task -> 
                    task.getTitle().toLowerCase().contains(searchQuery) ||
                    (task.getDescription() != null && task.getDescription().toLowerCase().contains(searchQuery))
                )
                .map(this::convertToResponse)
                .collect(Collectors.toList());
    }
    
    public TaskResponse create(TaskRequest request) {
        // Validate assigned user if provided
        if (request.getAssignedUserId() != null) {
            userService.findById(request.getAssignedUserId()); // Throws exception if not found
        }
        
        Task task = new Task();
        task.setId(idGenerator.getAndIncrement());
        task.setTitle(request.getTitle());
        task.setDescription(request.getDescription());
        task.setDueDate(request.getDueDate());
        task.setAssignedUserId(request.getAssignedUserId());
        
        tasks.put(task.getId(), task);
        
        System.out.printf("✅ Task created: '%s' (ID: %d)%n", task.getTitle(), task.getId());
        return convertToResponse(task);
    }
    
    public TaskResponse update(Long id, TaskRequest request) {
        Task task = tasks.get(id);
        if (task == null) {
            throw new TaskNotFoundException(id);
        }
        
        // Validate assigned user if provided
        if (request.getAssignedUserId() != null) {
            userService.findById(request.getAssignedUserId());
        }
        
        task.setTitle(request.getTitle());
        task.setDescription(request.getDescription());
        task.setDueDate(request.getDueDate());
        task.setAssignedUserId(request.getAssignedUserId());
        task.setUpdatedAt(LocalDateTime.now());
        
        System.out.printf("🔄 Task updated: '%s' (ID: %d)%n", task.getTitle(), task.getId());
        return convertToResponse(task);
    }
    
    public TaskResponse updateStatus(Long id, TaskStatus status) {
        Task task = tasks.get(id);
        if (task == null) {
            throw new TaskNotFoundException(id);
        }
        
        TaskStatus oldStatus = task.getStatus();
        task.setStatus(status);
        
        System.out.printf("📋 Task status updated: '%s' %s → %s%n", 
                         task.getTitle(), oldStatus, status);
        return convertToResponse(task);
    }
    
    public void deleteById(Long id) {
        Task task = tasks.remove(id);
        if (task == null) {
            throw new TaskNotFoundException(id);
        }
        
        System.out.printf("🗑️ Task deleted: '%s' (ID: %d)%n", task.getTitle(), task.getId());
    }
    
    public Map<TaskStatus, Long> getTaskStatistics() {
        return tasks.values().stream()
                .collect(Collectors.groupingBy(
                    Task::getStatus,
                    Collectors.counting()
                ));
    }
    
    private TaskResponse convertToResponse(Task task) {
        User assignedUser = null;
        if (task.getAssignedUserId() != null) {
            try {
                assignedUser = userService.findById(task.getAssignedUserId());
            } catch (UserNotFoundException e) {
                // User might have been deleted, handle gracefully
                System.out.printf("⚠️ Assigned user not found for task %d: %d%n", 
                                task.getId(), task.getAssignedUserId());
            }
        }
        return new TaskResponse(task, assignedUser);
    }
    
    private void initializeSampleData() {
        // Create sample tasks
        Task task1 = new Task("Setup project repository", "Initialize Spring Boot project with basic structure", 
                             LocalDateTime.now().plusDays(3));
        task1.setId(1L);
        task1.setAssignedUserId(1L);
        tasks.put(1L, task1);
        
        Task task2 = new Task("Design database schema", "Create ERD and design database tables",
                             LocalDateTime.now().plusDays(5));
        task2.setId(2L);
        task2.setStatus(TaskStatus.IN_PROGRESS);
        task2.setAssignedUserId(2L);
        tasks.put(2L, task2);
        
        Task task3 = new Task("Write API documentation", "Document all REST endpoints with examples",
                             LocalDateTime.now().minusDays(1)); // Overdue
        task3.setId(3L);
        task3.setAssignedUserId(1L);
        tasks.put(3L, task3);
        
        idGenerator.set(4L);
        System.out.println("✅ Sample tasks initialized");
    }
}

// UserService.java  
@Service
public class UserService {
    
    private final Map<Long, User> users = new ConcurrentHashMap<>();
    private final AtomicLong idGenerator = new AtomicLong(1);
    
    public UserService() {
        initializeSampleData();
    }
    
    public List<User> findAll() {
        return new ArrayList<>(users.values());
    }
    
    public User findById(Long id) {
        User user = users.get(id);
        if (user == null) {
            throw new UserNotFoundException(id);
        }
        return user;
    }
    
    public User create(User user) {
        // Check for duplicate email
        boolean emailExists = users.values().stream()
                .anyMatch(u -> u.getEmail().equalsIgnoreCase(user.getEmail()));
        
        if (emailExists) {
            throw new IllegalArgumentException("User with email " + user.getEmail() + " already exists");
        }
        
        user.setId(idGenerator.getAndIncrement());
        users.put(user.getId(), user);
        
        System.out.printf("✅ User created: '%s' (ID: %d)%n", user.getName(), user.getId());
        return user;
    }
    
    private void initializeSampleData() {
        User user1 = new User("John Doe", "john.doe@example.com");
        user1.setId(1L);
        users.put(1L, user1);
        
        User user2 = new User("Jane Smith", "jane.smith@example.com");
        user2.setId(2L);
        users.put(2L, user2);
        
        idGenerator.set(3L);
        System.out.println("✅ Sample users initialized");
    }
}
```

### Controllers with All HTTP Methods:
```java
// TaskController.java
@RestController
@RequestMapping("/api/tasks")
@CrossOrigin(origins = {"http://localhost:3000", "http://localhost:4200"})
public class TaskController {
    
    private final TaskService taskService;
    
    public TaskController(TaskService taskService) {
        this.taskService = taskService;
    }
    
    // GET - Retrieve all tasks
    @GetMapping
    public ResponseEntity<List<TaskResponse>> getAllTasks(
            @RequestParam(required = false) String status,
            @RequestParam(required = false) Long userId,
            @RequestParam(required = false) String search) {
        
        List<TaskResponse> tasks;
        
        if (search != null && !search.trim().isEmpty()) {
            System.out.println("🔍 Searching tasks with query: " + search);
            tasks = taskService.searchTasks(search.trim());
        } else if (status != null) {
            System.out.println("📋 Filtering tasks by status: " + status);
            TaskStatus taskStatus = TaskStatus.valueOf(status.toUpperCase());
            tasks = taskService.findByStatus(taskStatus);
        } else if (userId != null) {
            System.out.println("👤 Filtering tasks by user ID: " + userId);
            tasks = taskService.findByUserId(userId);
        } else {
            System.out.println("📋 Retrieving all tasks");
            tasks = taskService.findAll();
        }
        
        return ResponseEntity.ok(tasks);
    }
    
    // GET - Retrieve single task
    @GetMapping("/{id}")
    public ResponseEntity<TaskResponse> getTask(@PathVariable Long id) {
        System.out.println("📖 Retrieving task with ID: " + id);
        TaskResponse task = taskService.findById(id);
        return ResponseEntity.ok(task);
    }
    
    // POST - Create new task
    @PostMapping
    public ResponseEntity<TaskResponse> createTask(@Valid @RequestBody TaskRequest request) {
        System.out.println("➕ Creating new task: " + request.getTitle());
        TaskResponse task = taskService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(task);
    }
    
    // PUT - Update entire task
    @PutMapping("/{id}")
    public ResponseEntity<TaskResponse> updateTask(
            @PathVariable Long id, 
            @Valid @RequestBody TaskRequest request) {
        System.out.println("🔄 Updating task: " + id);
        TaskResponse task = taskService.update(id, request);
        return ResponseEntity.ok(task);
    }
    
    // PATCH - Update task status
    @PatchMapping("/{id}/status")
    public ResponseEntity<TaskResponse> updateTaskStatus(
            @PathVariable Long id,
            @RequestBody Map<String, String> statusUpdate) {
        
        String statusStr = statusUpdate.get("status");
        if (statusStr == null) {
            throw new IllegalArgumentException("Status is required");
        }
        
        try {
            TaskStatus status = TaskStatus.valueOf(statusStr.toUpperCase());
            System.out.printf("📋 Updating task %d status to: %s%n", id, status);
            TaskResponse task = taskService.updateStatus(id, status);
            return ResponseEntity.ok(task);
        } catch (IllegalArgumentException e) {
            throw new IllegalArgumentException("Invalid status: " + statusStr);
        }
    }
    
    // DELETE - Remove task
    @DeleteMapping("/{id}")
    public ResponseEntity<String> deleteTask(@PathVariable Long id) {
        System.out.println("🗑️ Deleting task: " + id);
        taskService.deleteById(id);
        return ResponseEntity.ok("Task deleted successfully");
    }
    
    // GET - Task statistics
    @GetMapping("/stats")
    public ResponseEntity<Map<TaskStatus, Long>> getTaskStatistics() {
        System.out.println("📊 Retrieving task statistics");
        Map<TaskStatus, Long> stats = taskService.getTaskStatistics();
        return ResponseEntity.ok(stats);
    }
}

// UserController.java
@RestController
@RequestMapping("/api/users")
@CrossOrigin(origins = {"http://localhost:3000", "http://localhost:4200"})
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        System.out.println("👥 Retrieving all users");
        List<User> users = userService.findAll();
        return ResponseEntity.ok(users);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        System.out.println("👤 Retrieving user with ID: " + id);
        User user = userService.findById(id);
        return ResponseEntity.ok(user);
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        System.out.println("➕ Creating new user: " + user.getName());
        User createdUser = userService.create(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(createdUser);
    }
}
```

### Exception Handling:
```java
// Custom Exceptions
public class TaskNotFoundException extends RuntimeException {
    public TaskNotFoundException(Long id) {
        super("Task not found with ID: " + id);
    }
}

public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(Long id) {
        super("User not found with ID: " + id);
    }
}

// Global Exception Handler
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(TaskNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleTaskNotFound(TaskNotFoundException ex, HttpServletRequest request) {
        System.out.println("❌ Task not found: " + ex.getMessage());
        
        ErrorResponse error = ErrorResponse.builder()
            .success(false)
            .message(ex.getMessage())
            .errorCode("TASK_NOT_FOUND")
            .timestamp(LocalDateTime.now())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex, HttpServletRequest request) {
        System.out.println("❌ User not found: " + ex.getMessage());
        
        ErrorResponse error = ErrorResponse.builder()
            .success(false)
            .message(ex.getMessage())
            .errorCode("USER_NOT_FOUND")
            .timestamp(LocalDateTime.now())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationErrors(
            MethodArgumentNotValidException ex, HttpServletRequest request) {
        
        System.out.println("❌ Validation failed:");
        
        List<String> errors = ex.getBindingResult().getFieldErrors().stream()
                .map(error -> error.getField() + ": " + error.getDefaultMessage())
                .collect(Collectors.toList());
        
        errors.forEach(error -> System.out.println("  " + error));
        
        ErrorResponse errorResponse = ErrorResponse.builder()
            .success(false)
            .message("Validation failed")
            .errorCode("VALIDATION_ERROR")
            .details(errors)
            .timestamp(LocalDateTime.now())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.badRequest().body(errorResponse);
    }
    
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleBadRequest(IllegalArgumentException ex, HttpServletRequest request) {
        System.out.println("❌ Bad request: " + ex.getMessage());
        
        ErrorResponse error = ErrorResponse.builder()
            .success(false)
            .message(ex.getMessage())
            .errorCode("BAD_REQUEST")
            .timestamp(LocalDateTime.now())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.badRequest().body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex, HttpServletRequest request) {
        System.out.println("💥 Unexpected error: " + ex.getMessage());
        ex.printStackTrace();
        
        ErrorResponse error = ErrorResponse.builder()
            .success(false)
            .message("An unexpected error occurred")
            .errorCode("INTERNAL_ERROR")
            .timestamp(LocalDateTime.now())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

### Filters & Configuration:
```java
// RequestLoggingFilter.java
@Component
@Order(1)
public class RequestLoggingFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        
        String method = httpRequest.getMethod();
        String uri = httpRequest.getRequestURI();
        long startTime = System.currentTimeMillis();
        
        System.out.printf("🌐 [%s] %s %s - Started%n", LocalDateTime.now().format(DateTimeFormatter.ofPattern("HH:mm:ss")), method, uri);
        
        try {
            chain.doFilter(request, response);
            
            long duration = System.currentTimeMillis() - startTime;
            System.out.printf("✅ [%s] %s %s - Status: %d - Duration: %dms%n",
                             LocalDateTime.now().format(DateTimeFormatter.ofPattern("HH:mm:ss")), 
                             method, uri, httpResponse.getStatus(), duration);
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            System.out.printf("❌ [%s] %s %s - Error: %s - Duration: %dms%n",
                             LocalDateTime.now().format(DateTimeFormatter.ofPattern("HH:mm:ss")), 
                             method, uri, e.getMessage(), duration);
            throw e;
        }
    }
}

// WebConfig.java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("http://localhost:3000", "http://localhost:4200", "https://taskmanager.app")
                .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
        
        System.out.println("🌍 CORS configured for Task Manager API");
    }
}
```

---

## 🧪 Complete API Testing

### Test All Endpoints:
```bash
# Get all tasks
curl -X GET http://localhost:8080/api/tasks

# Get tasks by status
curl -X GET "http://localhost:8080/api/tasks?status=TODO"

# Get tasks by user
curl -X GET "http://localhost:8080/api/tasks?userId=1"

# Search tasks
curl -X GET "http://localhost:8080/api/tasks?search=project"

# Create new task
curl -X POST http://localhost:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Implement user authentication",
    "description": "Add JWT-based authentication system",
    "dueDate": "2024-02-15T10:00:00",
    "assignedUserId": 1
  }'

# Update task
curl -X PUT http://localhost:8080/api/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Setup project repository - Updated",
    "description": "Initialize Spring Boot project with enhanced structure",
    "dueDate": "2024-02-10T15:00:00",
    "assignedUserId": 2
  }'

# Update task status
curl -X PATCH http://localhost:8080/api/tasks/1/status \
  -H "Content-Type: application/json" \
  -d '{"status": "IN_PROGRESS"}'

# Delete task
curl -X DELETE http://localhost:8080/api/tasks/3

# Get task statistics
curl -X GET http://localhost:8080/api/tasks/stats

# Get all users
curl -X GET http://localhost:8080/api/users

# Create new user
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Alice Johnson",
    "email": "alice.johnson@example.com"
  }'

# Test validation error
curl -X POST http://localhost:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title": "",
    "description": "This should fail validation"
  }'
```

---

## 🎯 Week 2 Interview Cheat Sheet

### Quick Reference Card:
```
🌐 SPRING MVC & REST:
@RestController → JSON/XML APIs
@Controller → HTML views
@RequestMapping → Maps URLs to methods
@GetMapping → Retrieve data
@PostMapping → Create resources
@PutMapping → Replace entire resource
@PatchMapping → Partial updates
@DeleteMapping → Remove resources

📥 REQUEST HANDLING:
@RequestParam → Query parameters (?name=value)
@PathVariable → URL path variables (/users/{id})
@RequestBody → JSON/XML request body
@Valid → Enable validation

⚠️ EXCEPTION HANDLING:
@ExceptionHandler → Method-level error handling
@ControllerAdvice → Global error handling
ResponseEntity → Full control over response

✅ VALIDATION:
@NotNull, @NotBlank, @Size → Basic validation
@Email, @Pattern, @Min/@Max → Format validation
@Valid → Enable validation on objects
MethodArgumentNotValidException → Validation errors

🌍 CORS & FILTERS:
@CrossOrigin → Allow cross-origin requests
Filter → Servlet-level processing
Interceptor → Spring MVC-level processing
```

### Common Interview Questions & Quick Answers:

**Q: "Explain the difference between @RestController and @Controller"**
**A:** @RestController = @Controller + @ResponseBody. Use @RestController for APIs (returns JSON/XML), @Controller for web pages (returns views).

**Q: "How do you handle validation errors globally?"**
**A:** Use @ControllerAdvice with @ExceptionHandler(MethodArgumentNotValidException.class) to catch validation errors across all controllers.

**Q: "What's the difference between @RequestParam and @PathVariable?"**
**A:** @RequestParam for query parameters (?name=value), @PathVariable for URL path segments (/users/{id}).

**Q: "How do you enable CORS in Spring Boot?"**
**A:** Use @CrossOrigin annotation on controllers/methods, or configure globally via WebMvcConfigurer.addCorsMappings().

**Q: "What's the purpose of Filters in Spring Boot?"**
**A:** Filters handle cross-cutting concerns like authentication, logging, CORS at the servlet level before requests reach controllers.

---
