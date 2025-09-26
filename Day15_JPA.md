# 📅 Day 15: Spring Data JPA Basics

## 🎯 Today Topics
- Understand Spring Data JPA and its benefits
- Master `@Entity`, `@Table`, `@Id`, `@GeneratedValue`
- Learn column mapping with `@Column`
- Configure database connections
- Create your first JPA entities

---

## 🗄️ What is Spring Data JPA?

### JPA vs Spring Data JPA:
- **JPA**: Java Persistence API - specification for ORM
- **Hibernate**: Most popular JPA implementation  
- **Spring Data JPA**: Spring's wrapper around JPA with extra features

### Benefits:
- **No SQL writing** - automatic CRUD operations
- **Repository pattern** - clean data access layer
- **Query generation** - methods like `findByName()` 
- **Database agnostic** - works with MySQL, PostgreSQL, H2, etc.

---

## ⚙️ Setup & Configuration

### Dependencies (pom.xml):
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

### Database Configuration (application.yml):
```yaml
spring:
  # H2 Database (for development)
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password: password
  
  # JPA Configuration
  jpa:
    hibernate:
      ddl-auto: update    # create, update, validate, none
    show-sql: true        # Show SQL queries in console
    properties:
      hibernate:
        format_sql: true  # Format SQL for readability
  
  # H2 Console (for testing)
  h2:
    console:
      enabled: true
      path: /h2-console
```

---

## 📋 @Entity - Basic Entity

### Simple Entity Example:
```java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 50)
    private String name;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    private Integer age;
    
    @Column(name = "created_at")
    private LocalDateTime createdAt;
    
    // Default constructor (required by JPA)
    public User() {}
    
    // Constructor for convenience
    public User(String name, String email, Integer age) {
        this.name = name;
        this.email = email;
        this.age = age;
        this.createdAt = LocalDateTime.now();
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public Integer getAge() { return age; }
    public void setAge(Integer age) { this.age = age; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    @Override
    public String toString() {
        return String.format("User{id=%d, name='%s', email='%s', age=%d}", 
                           id, name, email, age);
    }
}
```

### Product Entity with More Annotations:
```java
@Entity
@Table(name = "products", indexes = {
    @Index(name = "idx_product_name", columnList = "name"),
    @Index(name = "idx_product_category", columnList = "category")
})
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String name;
    
    @Column(columnDefinition = "TEXT")
    private String description;
    
    @Column(precision = 10, scale = 2, nullable = false)
    private BigDecimal price;
    
    @Column(nullable = false)
    private String category;
    
    @Column(name = "stock_quantity")
    private Integer stockQuantity = 0;
    
    @Column(name = "is_active")
    private Boolean isActive = true;
    
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Constructors
    public Product() {}
    
    public Product(String name, String description, BigDecimal price, String category) {
        this.name = name;
        this.description = description;
        this.price = price;
        this.category = category;
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
    }
    
    // JPA Lifecycle callbacks
    @PrePersist
    public void prePersist() {
        if (createdAt == null) {
            createdAt = LocalDateTime.now();
        }
        updatedAt = LocalDateTime.now();
        System.out.println("🆕 Creating product: " + name);
    }
    
    @PreUpdate
    public void preUpdate() {
        updatedAt = LocalDateTime.now();
        System.out.println("🔄 Updating product: " + name);
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    
    public BigDecimal getPrice() { return price; }
    public void setPrice(BigDecimal price) { this.price = price; }
    
    public String getCategory() { return category; }
    public void setCategory(String category) { this.category = category; }
    
    public Integer getStockQuantity() { return stockQuantity; }
    public void setStockQuantity(Integer stockQuantity) { this.stockQuantity = stockQuantity; }
    
    public Boolean getIsActive() { return isActive; }
    public void setIsActive(Boolean isActive) { this.isActive = isActive; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
}
```

---

## 📊 Key JPA Annotations Explained

### Primary Key Annotations:
```java
// Auto-increment ID (most common)
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

// UUID as primary key
@Id
@GeneratedValue(strategy = GenerationType.UUID)
private String id;

// Manual ID assignment
@Id
private Long id;
```

### Column Mapping:
```java
@Column(name = "full_name")           // Different column name
private String name;

@Column(nullable = false)             // NOT NULL constraint
private String email;

@Column(unique = true)                // UNIQUE constraint  
private String username;

@Column(length = 500)                 // VARCHAR(500)
private String description;

@Column(precision = 10, scale = 2)    // DECIMAL(10,2) for money
private BigDecimal price;

@Column(columnDefinition = "TEXT")    // Specific SQL type
private String longText;
```

### Table Configuration:
```java
@Entity
@Table(
    name = "user_profiles",
    indexes = {
        @Index(name = "idx_email", columnList = "email"),
        @Index(name = "idx_name_age", columnList = "name, age")
    },
    uniqueConstraints = {
        @UniqueConstraint(columnNames = {"email"}),
        @UniqueConstraint(columnNames = {"username"})
    }
)
public class UserProfile { }
```

---

## 🚀 Complete Example: Book Store

### Book Entity:
```java
@Entity
@Table(name = "books")
public class Book {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 200)
    private String title;
    
    @Column(nullable = false, length = 100)  
    private String author;
    
    @Column(unique = true, length = 20)
    private String isbn;
    
    @Column(precision = 8, scale = 2)
    private BigDecimal price;
    
    @Column(name = "publication_year")
    private Integer publicationYear;
    
    @Column(length = 50)
    private String genre;
    
    @Column(columnDefinition = "TEXT")
    private String description;
    
    @Column(name = "page_count")
    private Integer pageCount;
    
    @Column(name = "in_stock")
    private Boolean inStock = true;
    
    @Column(name = "stock_quantity")
    private Integer stockQuantity = 0;
    
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Constructors
    public Book() {}
    
    public Book(String title, String author, String isbn, BigDecimal price) {
        this.title = title;
        this.author = author;
        this.isbn = isbn;
        this.price = price;
    }
    
    // Lifecycle methods
    @PrePersist
    public void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
        System.out.printf("📚 Creating book: %s by %s%n", title, author);
    }
    
    @PreUpdate
    public void onUpdate() {
        updatedAt = LocalDateTime.now();
        System.out.printf("📝 Updating book: %s%n", title);
    }
    
    @PostPersist
    public void afterCreate() {
        System.out.printf("✅ Book created with ID: %d%n", id);
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
    
    public String getIsbn() { return isbn; }
    public void setIsbn(String isbn) { this.isbn = isbn; }
    
    public BigDecimal getPrice() { return price; }
    public void setPrice(BigDecimal price) { this.price = price; }
    
    public Integer getPublicationYear() { return publicationYear; }
    public void setPublicationYear(Integer publicationYear) { this.publicationYear = publicationYear; }
    
    public String getGenre() { return genre; }
    public void setGenre(String genre) { this.genre = genre; }
    
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    
    public Integer getPageCount() { return pageCount; }
    public void setPageCount(Integer pageCount) { this.pageCount = pageCount; }
    
    public Boolean getInStock() { return inStock; }
    public void setInStock(Boolean inStock) { this.inStock = inStock; }
    
    public Integer getStockQuantity() { return stockQuantity; }
    public void setStockQuantity(Integer stockQuantity) { this.stockQuantity = stockQuantity; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
    
    @Override
    public String toString() {
        return String.format("Book{id=%d, title='%s', author='%s', price=%.2f}", 
                           id, title, author, price);
    }
}
```

### Test the Entity:
```java
@SpringBootApplication  
public class BookStoreApplication implements CommandLineRunner {
    
    public static void main(String[] args) {
        SpringApplication.run(BookStoreApplication.class, args);
    }
    
    @Override
    public void run(String... args) throws Exception {
        System.out.println("🚀 Book Store Application Started!");
        System.out.println("📊 H2 Console: http://localhost:8080/h2-console");
        System.out.println("   JDBC URL: jdbc:h2:mem:testdb");
        System.out.println("   Username: sa");
        System.out.println("   Password: password");
    }
}
```

---

## 🧪 Testing Your Entities

### 1. Check H2 Console:
- Go to: `http://localhost:8080/h2-console`
- JDBC URL: `jdbc:h2:mem:testdb`
- Username: `sa`, Password: `password`

### 2. Verify Tables Created:
```sql
-- Check if tables exist
SHOW TABLES;

-- See table structure  
DESCRIBE BOOKS;
DESCRIBE USERS;

-- Check data
SELECT * FROM BOOKS;
SELECT * FROM USERS;
```

### 3. Manual Data Insert (for testing):
```sql
INSERT INTO BOOKS (title, author, isbn, price, publication_year, genre, in_stock, stock_quantity) 
VALUES ('Spring Boot in Action', 'Craig Walls', '978-1617292545', 45.99, 2018, 'Technology', true, 10);

INSERT INTO USERS (name, email, age, created_at)
VALUES ('John Doe', 'john@example.com', 25, CURRENT_TIMESTAMP);
```

---

## 📋 JPA Annotations Quick Reference

| Annotation | Purpose | Example |
|------------|---------|---------|
| `@Entity` | Mark class as JPA entity | `@Entity public class User` |
| `@Table` | Specify table name/properties | `@Table(name = "users")` |
| `@Id` | Mark primary key field | `@Id private Long id;` |
| `@GeneratedValue` | Auto-generate ID values | `@GeneratedValue(strategy = IDENTITY)` |
| `@Column` | Column mapping/constraints | `@Column(nullable = false)` |
| `@PrePersist` | Before entity save | Method called before insert |
| `@PreUpdate` | Before entity update | Method called before update |
| `@PostPersist` | After entity save | Method called after insert |

---

## 🎯 Interview Questions & Answers

### Q1: "What is the difference between JPA and Hibernate?"
**Answer:**
- **JPA**: Java Persistence API - specification/standard for ORM
- **Hibernate**: Implementation of JPA specification (most popular)
- **Spring Data JPA**: Spring's wrapper around JPA with additional features
- **Relationship**: JPA (specification) → Hibernate (implementation) → Spring Data JPA (wrapper)

### Q2: "What does @GeneratedValue(strategy = GenerationType.IDENTITY) do?"
**Answer:**
- **IDENTITY**: Database auto-increment (MySQL AUTO_INCREMENT, PostgreSQL SERIAL)
- **AUTO**: JPA provider chooses strategy automatically
- **SEQUENCE**: Uses database sequences (PostgreSQL, Oracle)
- **TABLE**: Uses separate table to generate IDs
- **UUID**: Generates UUID values

### Q3: "What is the purpose of @PrePersist and @PostPersist?"
**Answer:**
- **@PrePersist**: Called BEFORE entity is saved to database (set timestamps, validate data)
- **@PostPersist**: Called AFTER entity is saved (logging, auditing, notifications)
- **@PreUpdate**: Called BEFORE entity is updated
- **@PostUpdate**: Called AFTER entity is updated
- **Use for**: Auditing, logging, setting timestamps, validation

---

## 🚀 Best Practices

### ✅ Do's:
```java
// 1. Always provide default constructor
public User() {} // Required by JPA

// 2. Use appropriate data types
private BigDecimal price;        // For money
private LocalDateTime createdAt; // For dates
private Boolean isActive;       // For booleans

// 3. Set reasonable column constraints
@Column(nullable = false, length = 100)
private String name;

// 4. Use lifecycle methods for timestamps
@PrePersist
public void onCreate() {
    createdAt = LocalDateTime.now();
}

// 5. Override toString() for debugging
@Override
public String toString() {
    return "User{id=" + id + ", name='" + name + "'}";
}
```

### ❌ Don'ts:
```java
// 1. Don't forget @Entity
public class User { } // BAD - not an entity

// 2. Don't use primitive types for nullable fields
private int age; // BAD - should be Integer for nullable

// 3. Don't make ID mutable
public void setId(Long id) { 
    this.id = id; // BAD - IDs should be immutable
}

// 4. Don't use reserved keywords as column names
@Column(name = "order") // BAD - 'order' is reserved in SQL
```

---

