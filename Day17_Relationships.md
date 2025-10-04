# 📅 Day 17: Entity Relationships

## 🎯 Today Topics
- Master `@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany`
- Understand bidirectional vs unidirectional relationships
- Learn cascade operations and fetch types
- Handle relationship best practices

---

## 🔗 Understanding Entity Relationships

### Types of Relationships:
1. **@OneToOne**: One User has One Profile
2. **@OneToMany / @ManyToOne**: One Author has Many Books
3. **@ManyToMany**: Many Students have Many Courses

---

## 👤 @OneToOne - One to One Relationship

### Example: User and Profile
```java
// User.java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String username;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    // One User has One Profile
    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private UserProfile profile;
    
    // Constructors
    public User() {}
    
    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public UserProfile getProfile() { return profile; }
    public void setProfile(UserProfile profile) { 
        this.profile = profile;
        if (profile != null) {
            profile.setUser(this);
        }
    }
}

// UserProfile.java
@Entity
@Table(name = "user_profiles")
public class UserProfile {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String bio;
    private String phoneNumber;
    private String address;
    
    // One Profile belongs to One User
    @OneToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    // Constructors
    public UserProfile() {}
    
    public UserProfile(String bio, String phoneNumber, String address) {
        this.bio = bio;
        this.phoneNumber = phoneNumber;
        this.address = address;
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getBio() { return bio; }
    public void setBio(String bio) { this.bio = bio; }
    
    public String getPhoneNumber() { return phoneNumber; }
    public void setPhoneNumber(String phoneNumber) { this.phoneNumber = phoneNumber; }
    
    public String getAddress() { return address; }
    public void setAddress(String address) { this.address = address; }
    
    public User getUser() { return user; }
    public void setUser(User user) { this.user = user; }
}

// Usage Example
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public User createUserWithProfile() {
        // Create user
        User user = new User("john_doe", "john@example.com");
        
        // Create profile
        UserProfile profile = new UserProfile(
            "Software Developer", 
            "+1234567890", 
            "123 Main St"
        );
        
        // Link them together
        user.setProfile(profile);
        
        // Save user (profile is saved automatically due to cascade)
        return userRepository.save(user);
    }
}
```

---

## 📚 @OneToMany / @ManyToOne - One to Many

### Example: Author and Books
```java
// Author.java
@Entity
@Table(name = "authors")
public class Author {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    private String bio;
    
    // One Author has Many Books
    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Book> books = new ArrayList<>();
    
    // Constructors
    public Author() {}
    
    public Author(String name, String bio) {
        this.name = name;
        this.bio = bio;
    }
    
    // Helper methods for bidirectional relationship
    public void addBook(Book book) {
        books.add(book);
        book.setAuthor(this);
    }
    
    public void removeBook(Book book) {
        books.remove(book);
        book.setAuthor(null);
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getBio() { return bio; }
    public void setBio(String bio) { this.bio = bio; }
    
    public List<Book> getBooks() { return books; }
    public void setBooks(List<Book> books) { this.books = books; }
}

// Book.java
@Entity
@Table(name = "books")
public class Book {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String title;
    
    @Column(unique = true)
    private String isbn;
    
    private BigDecimal price;
    
    // Many Books belong to One Author
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id", nullable = false)
    private Author author;
    
    // Constructors
    public Book() {}
    
    public Book(String title, String isbn, BigDecimal price) {
        this.title = title;
        this.isbn = isbn;
        this.price = price;
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    
    public String getIsbn() { return isbn; }
    public void setIsbn(String isbn) { this.isbn = isbn; }
    
    public BigDecimal getPrice() { return price; }
    public void setPrice(BigDecimal price) { this.price = price; }
    
    public Author getAuthor() { return author; }
    public void setAuthor(Author author) { this.author = author; }
}

// Usage Example
@Service
public class AuthorService {
    
    @Autowired
    private AuthorRepository authorRepository;
    
    public Author createAuthorWithBooks() {
        // Create author
        Author author = new Author("J.K. Rowling", "British author");
        
        // Create books
        Book book1 = new Book("Harry Potter 1", "ISBN-001", new BigDecimal("19.99"));
        Book book2 = new Book("Harry Potter 2", "ISBN-002", new BigDecimal("21.99"));
        Book book3 = new Book("Harry Potter 3", "ISBN-003", new BigDecimal("22.99"));
        
        // Add books to author (helper method handles bidirectional relationship)
        author.addBook(book1);
        author.addBook(book2);
        author.addBook(book3);
        
        // Save author (books are saved automatically due to cascade)
        return authorRepository.save(author);
    }
    
    public void removeBookFromAuthor(Long authorId, Long bookId) {
        Author author = authorRepository.findById(authorId)
                .orElseThrow(() -> new RuntimeException("Author not found"));
        
        Book bookToRemove = author.getBooks().stream()
                .filter(book -> book.getId().equals(bookId))
                .findFirst()
                .orElseThrow(() -> new RuntimeException("Book not found"));
        
        // Remove book (orphanRemoval will delete it from database)
        author.removeBook(bookToRemove);
        
        authorRepository.save(author);
    }
}
```

---

## 🎓 @ManyToMany - Many to Many

### Example: Students and Courses
```java
// Student.java
@Entity
@Table(name = "students")
public class Student {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Column(unique = true)
    private String studentId;
    
    // Many Students have Many Courses
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_courses",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
    
    // Constructors
    public Student() {}
    
    public Student(String name, String studentId) {
        this.name = name;
        this.studentId = studentId;
    }
    
    // Helper methods
    public void enrollInCourse(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }
    
    public void dropCourse(Course course) {
        courses.remove(course);
        course.getStudents().remove(this);
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getStudentId() { return studentId; }
    public void setStudentId(String studentId) { this.studentId = studentId; }
    
    public Set<Course> getCourses() { return courses; }
    public void setCourses(Set<Course> courses) { this.courses = courses; }
}

// Course.java
@Entity
@Table(name = "courses")
public class Course {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Column(unique = true)
    private String courseCode;
    
    private Integer credits;
    
    // Many Courses have Many Students
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
    
    // Constructors
    public Course() {}
    
    public Course(String name, String courseCode, Integer credits) {
        this.name = name;
        this.courseCode = courseCode;
        this.credits = credits;
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getCourseCode() { return courseCode; }
    public void setCourseCode(String courseCode) { this.courseCode = courseCode; }
    
    public Integer getCredits() { return credits; }
    public void setCredits(Integer credits) { this.credits = credits; }
    
    public Set<Student> getStudents() { return students; }
    public void setStudents(Set<Student> students) { this.students = students; }
}

// Usage Example
@Service
public class StudentService {
    
    @Autowired
    private StudentRepository studentRepository;
    
    @Autowired
    private CourseRepository courseRepository;
    
    public Student enrollStudentInCourses() {
        // Create student
        Student student = new Student("John Doe", "STU001");
        
        // Create courses
        Course course1 = new Course("Java Programming", "CS101", 3);
        Course course2 = new Course("Database Design", "CS102", 4);
        Course course3 = new Course("Web Development", "CS103", 3);
        
        // Save courses first
        courseRepository.save(course1);
        courseRepository.save(course2);
        courseRepository.save(course3);
        
        // Enroll student in courses
        student.enrollInCourse(course1);
        student.enrollInCourse(course2);
        student.enrollInCourse(course3);
        
        // Save student
        return studentRepository.save(student);
    }
    
    public void dropCourse(Long studentId, Long courseId) {
        Student student = studentRepository.findById(studentId)
                .orElseThrow(() -> new RuntimeException("Student not found"));
        
        Course course = courseRepository.findById(courseId)
                .orElseThrow(() -> new RuntimeException("Course not found"));
        
        student.dropCourse(course);
        
        studentRepository.save(student);
    }
}
```

---

## ⚙️ Cascade Types & Fetch Types

### Cascade Types:
```java
// CascadeType.ALL - All operations cascade
@OneToMany(cascade = CascadeType.ALL)
private List<Book> books;

// CascadeType.PERSIST - Only save operations cascade
@OneToMany(cascade = CascadeType.PERSIST)
private List<Book> books;

// CascadeType.MERGE - Only update operations cascade
@OneToMany(cascade = CascadeType.MERGE)
private List<Book> books;

// CascadeType.REMOVE - Delete operations cascade
@OneToMany(cascade = CascadeType.REMOVE)
private List<Book> books;

// Multiple cascade types
@OneToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
private List<Book> books;

// orphanRemoval - Delete child when removed from parent
@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
private List<Book> books;
```

### Fetch Types:
```java
// LAZY - Load data only when accessed (recommended)
@ManyToOne(fetch = FetchType.LAZY)
private Author author;

// EAGER - Load data immediately with parent (use sparingly)
@ManyToOne(fetch = FetchType.EAGER)
private Author author;
```

---

## 🎯 Complete Example: Blog System

### Entities:
```java
// Post.java
@Entity
@Table(name = "posts")
public class Post {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String title;
    
    @Column(columnDefinition = "TEXT")
    private String content;
    
    // Many Posts belong to One User (Author)
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User author;
    
    // One Post has Many Comments
    @OneToMany(mappedBy = "post", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();
    
    // Many Posts have Many Tags
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "post_tags",
        joinColumns = @JoinColumn(name = "post_id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id")
    )
    private Set<Tag> tags = new HashSet<>();
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    // Constructors
    public Post() {}
    
    public Post(String title, String content, User author) {
        this.title = title;
        this.content = content;
        this.author = author;
        this.createdAt = LocalDateTime.now();
    }
    
    // Helper methods
    public void addComment(Comment comment) {
        comments.add(comment);
        comment.setPost(this);
    }
    
    public void removeComment(Comment comment) {
        comments.remove(comment);
        comment.setPost(null);
    }
    
    public void addTag(Tag tag) {
        tags.add(tag);
        tag.getPosts().add(this);
    }
    
    public void removeTag(Tag tag) {
        tags.remove(tag);
        tag.getPosts().remove(this);
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    
    public String getContent() { return content; }
    public void setContent(String content) { this.content = content; }
    
    public User getAuthor() { return author; }
    public void setAuthor(User author) { this.author = author; }
    
    public List<Comment> getComments() { return comments; }
    public void setComments(List<Comment> comments) { this.comments = comments; }
    
    public Set<Tag> getTags() { return tags; }
    public void setTags(Set<Tag> tags) { this.tags = tags; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
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
    
    // Many Comments belong to One Post
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id", nullable = false)
    private Post post;
    
    // Many Comments belong to One User
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    // Constructors
    public Comment() {}
    
    public Comment(String content, User user) {
        this.content = content;
        this.user = user;
        this.createdAt = LocalDateTime.now();
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getContent() { return content; }
    public void setContent(String content) { this.content = content; }
    
    public Post getPost() { return post; }
    public void setPost(Post post) { this.post = post; }
    
    public User getUser() { return user; }
    public void setUser(User user) { this.user = user; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}

// Tag.java
@Entity
@Table(name = "tags")
public class Tag {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String name;
    
    // Many Tags have Many Posts
    @ManyToMany(mappedBy = "tags")
    private Set<Post> posts = new HashSet<>();
    
    // Constructors
    public Tag() {}
    
    public Tag(String name) {
        this.name = name;
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public Set<Post> getPosts() { return posts; }
    public void setPosts(Set<Post> posts) { this.posts = posts; }
}
```

---

## 📋 Relationship Mapping Quick Reference

| Relationship | Example | Foreign Key | Join Table |
|--------------|---------|-------------|------------|
| **@OneToOne** | User ↔ Profile | In child table | No |
| **@ManyToOne** | Book → Author | In "many" side | No |
| **@OneToMany** | Author → Books | In child table | No |
| **@ManyToMany** | Student ↔ Course | Not used | Yes |

---

## 🎯 Interview Questions & Answers

### Q1: "What's the difference between unidirectional and bidirectional relationships?"
**Answer:**
- **Unidirectional**: Only one entity knows about the relationship
- **Bidirectional**: Both entities know about each other
- **Example**: Unidirectional - Book has Author reference. Bidirectional - Book has Author AND Author has List of Books
- **Use bidirectional when you need to navigate from both sides**

### Q2: "What is cascade and when should you use it?"
**Answer:**
- **Cascade**: Operations on parent automatically apply to children
- **CascadeType.ALL**: Save, update, delete cascade to children
- **CascadeType.PERSIST**: Only save operations cascade
- **Use carefully**: CascadeType.REMOVE can accidentally delete data
- **orphanRemoval**: Delete child when removed from parent collection

### Q3: "What's the difference between LAZY and EAGER fetching?"
**Answer:**
- **LAZY (recommended)**: Load child data only when accessed, prevents unnecessary queries
- **EAGER**: Load child data immediately with parent, can cause performance issues
- **Default**: @ManyToOne/@OneToOne → EAGER, @OneToMany/@ManyToMany → LAZY
- **Best practice**: Use LAZY and load data explicitly when needed

---

## 🚀 Best Practices

### ✅ Do's:
```java
// 1. Use LAZY fetching by default
@ManyToOne(fetch = FetchType.LAZY)
private Author author;

// 2. Use helper methods for bidirectional relationships
public void addBook(Book book) {
    books.add(book);
    book.setAuthor(this);
}

// 3. Use Set for @ManyToMany (avoids duplicates)
@ManyToMany
private Set<Course> courses = new HashSet<>();

// 4. Be careful with cascade operations
@OneToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
private List<Book> books;

// 5. Use orphanRemoval for composition relationships
@OneToMany(orphanRemoval = true)
private List<Comment> comments;
```

### ❌ Don'ts:
```java
// 1. Don't use EAGER everywhere
@ManyToOne(fetch = FetchType.EAGER) // Can cause N+1 problem

// 2. Don't use CascadeType.ALL blindly
@ManyToMany(cascade = CascadeType.ALL) // Dangerous with many-to-many

// 3. Don't forget to maintain both sides of bidirectional relationship
author.getBooks().add(book); // BAD - only one side updated

// 4. Don't use List for @ManyToMany
@ManyToMany
private List<Course> courses; // Use Set instead
```

---
