# 📅 Day 16: Repository Layer

## 🎯 Today Topics
- Master `JpaRepository` and its built-in methods
- Create custom query methods with method naming
- Use `@Query` for complex queries
- Understand pagination and sorting
- Best practices for repository layer

---

## 📚 Understanding Repository Layer

### What is a Repository?
**Repository** provides abstraction for data access - no SQL needed!

### Repository Hierarchy:
```
Repository (base interface)
    ↓
CrudRepository (basic CRUD)
    ↓
PagingAndSortingRepository (pagination + sorting)
    ↓
JpaRepository (JPA specific methods) ← Most commonly used
```

---

## 🚀 JpaRepository - Built-in Methods

### Basic Repository Setup:
```java
// Just extend JpaRepository - that's it!
public interface UserRepository extends JpaRepository<User, Long> {
    // No code needed - you get 20+ methods for free!
}
```

### Built-in Methods (You Get for Free):
```java
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public void demonstrateBuiltInMethods() {
        
        // 1. Save single entity
        User user = new User("John Doe", "john@example.com", 25);
        userRepository.save(user);
        
        // 2. Save multiple entities
        List<User> users = Arrays.asList(
            new User("Jane Smith", "jane@example.com", 30),
            new User("Bob Wilson", "bob@example.com", 28)
        );
        userRepository.saveAll(users);
        
        // 3. Find by ID
        Optional<User> foundUser = userRepository.findById(1L);
        foundUser.ifPresent(u -> System.out.println("Found: " + u.getName()));
        
        // 4. Find all
        List<User> allUsers = userRepository.findAll();
        System.out.println("Total users: " + allUsers.size());
        
        // 5. Check if exists
        boolean exists = userRepository.existsById(1L);
        System.out.println("User 1 exists: " + exists);
        
        // 6. Count entities
        long count = userRepository.count();
        System.out.println("Total count: " + count);
        
        // 7. Delete by ID
        userRepository.deleteById(1L);
        
        // 8. Delete entity
        userRepository.delete(user);
        
        // 9. Delete all
        userRepository.deleteAll();
    }
}
```

---

## 🔍 Custom Query Methods (Method Naming)

### Simple Queries:
```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Find by single field
    User findByEmail(String email);
    List<User> findByName(String name);
    
    // Find by multiple fields (AND)
    User findByNameAndEmail(String name, String email);
    List<User> findByNameAndAge(String name, Integer age);
    
    // Find with OR condition
    List<User> findByNameOrEmail(String name, String email);
    
    // Greater than / Less than
    List<User> findByAgeGreaterThan(Integer age);
    List<User> findByAgeLessThan(Integer age);
    List<User> findByAgeBetween(Integer startAge, Integer endAge);
    
    // Like / Contains
    List<User> findByNameContaining(String keyword);
    List<User> findByNameStartingWith(String prefix);
    List<User> findByNameEndingWith(String suffix);
    
    // Ordering
    List<User> findByAgeOrderByNameAsc(Integer age);
    List<User> findByNameOrderByAgeDesc(String name);
    
    // Null checks
    List<User> findByEmailIsNull();
    List<User> findByEmailIsNotNull();
    
    // Boolean checks
    List<User> findByActiveTrue();
    List<User> findByActiveFalse();
}
```

### Complete Example: Book Repository:
```java
public interface BookRepository extends JpaRepository<Book, Long> {
    
    // Simple finds
    Book findByIsbn(String isbn);
    List<Book> findByAuthor(String author);
    List<Book> findByGenre(String genre);
    
    // Price queries
    List<Book> findByPriceLessThan(BigDecimal price);
    List<Book> findByPriceGreaterThan(BigDecimal price);
    List<Book> findByPriceBetween(BigDecimal minPrice, BigDecimal maxPrice);
    
    // Stock queries
    List<Book> findByInStockTrue();
    List<Book> findByStockQuantityGreaterThan(Integer quantity);
    
    // Year queries
    List<Book> findByPublicationYear(Integer year);
    List<Book> findByPublicationYearBetween(Integer startYear, Integer endYear);
    
    // Text search
    List<Book> findByTitleContaining(String keyword);
    List<Book> findByAuthorContaining(String keyword);
    List<Book> findByTitleContainingOrAuthorContaining(String titleKeyword, String authorKeyword);
    
    // Combined queries with ordering
    List<Book> findByGenreOrderByPriceAsc(String genre);
    List<Book> findByAuthorOrderByPublicationYearDesc(String author);
    List<Book> findByInStockTrueOrderByTitleAsc();
    
    // Count queries
    long countByGenre(String genre);
    long countByInStockTrue();
    
    // Exists queries
    boolean existsByIsbn(String isbn);
    boolean existsByTitleAndAuthor(String title, String author);
    
    // Delete queries
    void deleteByIsbn(String isbn);
    long deleteByInStockFalse();
}
```

---

## 📝 @Query - Custom JPQL Queries

### JPQL Queries (Java Persistence Query Language):
```java
public interface BookRepository extends JpaRepository<Book, Long> {
    
    // Simple JPQL query
    @Query("SELECT b FROM Book b WHERE b.author = :author")
    List<Book> findBooksByAuthor(@Param("author") String author);
    
    // Multiple parameters
    @Query("SELECT b FROM Book b WHERE b.genre = :genre AND b.price < :maxPrice")
    List<Book> findAffordableBooksByGenre(
        @Param("genre") String genre, 
        @Param("maxPrice") BigDecimal maxPrice
    );
    
    // LIKE queries
    @Query("SELECT b FROM Book b WHERE b.title LIKE %:keyword% OR b.description LIKE %:keyword%")
    List<Book> searchBooks(@Param("keyword") String keyword);
    
    // Aggregation queries
    @Query("SELECT AVG(b.price) FROM Book b WHERE b.genre = :genre")
    BigDecimal findAveragePriceByGenre(@Param("genre") String genre);
    
    @Query("SELECT COUNT(b) FROM Book b WHERE b.inStock = true")
    long countBooksInStock();
    
    // Complex queries
    @Query("SELECT b FROM Book b WHERE b.publicationYear >= :year AND b.price BETWEEN :minPrice AND :maxPrice ORDER BY b.price ASC")
    List<Book> findRecentBooksByPriceRange(
        @Param("year") Integer year,
        @Param("minPrice") BigDecimal minPrice,
        @Param("maxPrice") BigDecimal maxPrice
    );
}
```

### Native SQL Queries:
```java
public interface BookRepository extends JpaRepository<Book, Long> {
    
    // Native SQL query
    @Query(value = "SELECT * FROM books WHERE author = ?1", nativeQuery = true)
    List<Book> findBooksByAuthorNative(String author);
    
    // Native query with named parameters
    @Query(value = "SELECT * FROM books WHERE genre = :genre AND price < :maxPrice", nativeQuery = true)
    List<Book> findAffordableBooksByGenreNative(
        @Param("genre") String genre,
        @Param("maxPrice") BigDecimal maxPrice
    );
    
    // Complex native query
    @Query(value = """
        SELECT b.* FROM books b 
        WHERE b.in_stock = true 
        AND b.stock_quantity > 0 
        ORDER BY b.created_at DESC 
        LIMIT :limit
        """, nativeQuery = true)
    List<Book> findLatestBooksInStock(@Param("limit") int limit);
}
```

---

## 📄 Pagination & Sorting

### Pagination Example:
```java
public interface BookRepository extends JpaRepository<Book, Long> {
    
    // Pageable methods
    Page<Book> findByGenre(String genre, Pageable pageable);
    Page<Book> findByInStockTrue(Pageable pageable);
    
    @Query("SELECT b FROM Book b WHERE b.price < :maxPrice")
    Page<Book> findAffordableBooks(@Param("maxPrice") BigDecimal maxPrice, Pageable pageable);
}
```

### Using Pagination in Service:
```java
@Service
public class BookService {
    
    @Autowired
    private BookRepository bookRepository;
    
    public Page<Book> getBooks(int page, int size) {
        // Create pageable with page number and size
        Pageable pageable = PageRequest.of(page, size);
        return bookRepository.findAll(pageable);
    }
    
    public Page<Book> getBooksSorted(int page, int size, String sortBy) {
        // With sorting
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy).ascending());
        return bookRepository.findAll(pageable);
    }
    
    public Page<Book> getBooksWithMultipleSort(int page, int size) {
        // Multiple sort fields
        Sort sort = Sort.by(
            Sort.Order.desc("createdAt"),
            Sort.Order.asc("title")
        );
        Pageable pageable = PageRequest.of(page, size, sort);
        return bookRepository.findAll(pageable);
    }
    
    public void demonstratePagination() {
        // Get first page (10 items)
        Page<Book> firstPage = bookRepository.findAll(PageRequest.of(0, 10));
        
        System.out.println("Total elements: " + firstPage.getTotalElements());
        System.out.println("Total pages: " + firstPage.getTotalPages());
        System.out.println("Current page: " + firstPage.getNumber());
        System.out.println("Page size: " + firstPage.getSize());
        System.out.println("Has next: " + firstPage.hasNext());
        System.out.println("Has previous: " + firstPage.hasPrevious());
        
        List<Book> books = firstPage.getContent();
        books.forEach(book -> System.out.println(book.getTitle()));
    }
}
```

---

## 🎯 Complete Example: Book Management Service

### Service with Repository Methods:
```java
@Service
public class BookService {
    
    @Autowired
    private BookRepository bookRepository;
    
    // Create
    public Book createBook(Book book) {
        System.out.println("📚 Creating book: " + book.getTitle());
        return bookRepository.save(book);
    }
    
    // Read - All
    public List<Book> getAllBooks() {
        System.out.println("📋 Fetching all books");
        return bookRepository.findAll();
    }
    
    // Read - By ID
    public Book getBookById(Long id) {
        System.out.println("🔍 Fetching book with ID: " + id);
        return bookRepository.findById(id)
                .orElseThrow(() -> new BookNotFoundException(id));
    }
    
    // Read - By ISBN
    public Book getBookByIsbn(String isbn) {
        System.out.println("🔍 Fetching book with ISBN: " + isbn);
        Book book = bookRepository.findByIsbn(isbn);
        if (book == null) {
            throw new BookNotFoundException("Book not found with ISBN: " + isbn);
        }
        return book;
    }
    
    // Read - By Genre with Pagination
    public Page<Book> getBooksByGenre(String genre, int page, int size) {
        System.out.printf("📖 Fetching %s books (page %d, size %d)%n", genre, page, size);
        Pageable pageable = PageRequest.of(page, size, Sort.by("title").ascending());
        return bookRepository.findByGenre(genre, pageable);
    }
    
    // Search
    public List<Book> searchBooks(String keyword) {
        System.out.println("🔍 Searching books with keyword: " + keyword);
        return bookRepository.findByTitleContainingOrAuthorContaining(keyword, keyword);
    }
    
    // Price range
    public List<Book> getBooksByPriceRange(BigDecimal minPrice, BigDecimal maxPrice) {
        System.out.printf("💰 Fetching books between $%.2f and $%.2f%n", minPrice, maxPrice);
        return bookRepository.findByPriceBetween(minPrice, maxPrice);
    }
    
    // In stock books
    public List<Book> getBooksInStock() {
        System.out.println("📦 Fetching books in stock");
        return bookRepository.findByInStockTrue();
    }
    
    // Update
    public Book updateBook(Long id, Book bookDetails) {
        System.out.println("🔄 Updating book: " + id);
        
        Book book = getBookById(id);
        book.setTitle(bookDetails.getTitle());
        book.setAuthor(bookDetails.getAuthor());
        book.setPrice(bookDetails.getPrice());
        book.setGenre(bookDetails.getGenre());
        book.setDescription(bookDetails.getDescription());
        
        return bookRepository.save(book);
    }
    
    // Update stock
    public Book updateStock(Long id, Integer quantity) {
        System.out.printf("📦 Updating stock for book %d: %d units%n", id, quantity);
        
        Book book = getBookById(id);
        book.setStockQuantity(quantity);
        book.setInStock(quantity > 0);
        
        return bookRepository.save(book);
    }
    
    // Delete
    public void deleteBook(Long id) {
        System.out.println("🗑️ Deleting book: " + id);
        
        if (!bookRepository.existsById(id)) {
            throw new BookNotFoundException(id);
        }
        
        bookRepository.deleteById(id);
    }
    
    // Statistics
    public Map<String, Object> getBookStatistics() {
        System.out.println("📊 Calculating book statistics");
        
        Map<String, Object> stats = new HashMap<>();
        stats.put("totalBooks", bookRepository.count());
        stats.put("booksInStock", bookRepository.countByInStockTrue());
        stats.put("genres", getGenreCount());
        
        return stats;
    }
    
    private Map<String, Long> getGenreCount() {
        return bookRepository.findAll().stream()
                .collect(Collectors.groupingBy(
                    Book::getGenre,
                    Collectors.counting()
                ));
    }
}
```

### Controller Using Service:
```java
@RestController
@RequestMapping("/api/books")
public class BookController {
    
    @Autowired
    private BookService bookService;
    
    @GetMapping
    public ResponseEntity<List<Book>> getAllBooks() {
        return ResponseEntity.ok(bookService.getAllBooks());
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Book> getBook(@PathVariable Long id) {
        return ResponseEntity.ok(bookService.getBookById(id));
    }
    
    @GetMapping("/isbn/{isbn}")
    public ResponseEntity<Book> getBookByIsbn(@PathVariable String isbn) {
        return ResponseEntity.ok(bookService.getBookByIsbn(isbn));
    }
    
    @GetMapping("/genre/{genre}")
    public ResponseEntity<Page<Book>> getBooksByGenre(
            @PathVariable String genre,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        return ResponseEntity.ok(bookService.getBooksByGenre(genre, page, size));
    }
    
    @GetMapping("/search")
    public ResponseEntity<List<Book>> searchBooks(@RequestParam String keyword) {
        return ResponseEntity.ok(bookService.searchBooks(keyword));
    }
    
    @PostMapping
    public ResponseEntity<Book> createBook(@Valid @RequestBody Book book) {
        return ResponseEntity.status(HttpStatus.CREATED)
                .body(bookService.createBook(book));
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Book> updateBook(@PathVariable Long id, @RequestBody Book book) {
        return ResponseEntity.ok(bookService.updateBook(id, book));
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<String> deleteBook(@PathVariable Long id) {
        bookService.deleteBook(id);
        return ResponseEntity.ok("Book deleted successfully");
    }
}
```

---

## 📋 Query Method Keywords Reference

| Keyword | Example | SQL Equivalent |
|---------|---------|----------------|
| `findBy` | `findByName` | `WHERE name = ?` |
| `And` | `findByNameAndAge` | `WHERE name = ? AND age = ?` |
| `Or` | `findByNameOrEmail` | `WHERE name = ? OR email = ?` |
| `Between` | `findByAgeBetween` | `WHERE age BETWEEN ? AND ?` |
| `LessThan` | `findByAgeLessThan` | `WHERE age < ?` |
| `GreaterThan` | `findByAgeGreaterThan` | `WHERE age > ?` |
| `Like` | `findByNameLike` | `WHERE name LIKE ?` |
| `Containing` | `findByNameContaining` | `WHERE name LIKE %?%` |
| `StartingWith` | `findByNameStartingWith` | `WHERE name LIKE ?%` |
| `EndingWith` | `findByNameEndingWith` | `WHERE name LIKE %?` |
| `OrderBy` | `findByAgeOrderByNameAsc` | `ORDER BY name ASC` |

---

## 🎯 Interview Questions & Answers

### Q1: "What's the difference between JPQL and native SQL queries?"
**Answer:**
- **JPQL**: Uses entity names and fields (`SELECT u FROM User u WHERE u.name = :name`)
- **Native SQL**: Uses table and column names (`SELECT * FROM users WHERE name = ?`)
- **JPQL is database-agnostic**, Native SQL is database-specific
- **Use JPQL when possible**, Native SQL for complex database-specific queries

### Q2: "How does Spring Data JPA generate queries from method names?"
**Answer:**
Spring Data JPA parses method names and generates queries automatically:
- `findBy` + field name → WHERE clause
- `And`, `Or` → Logical operators
- `OrderBy` → ORDER BY clause
- Example: `findByNameAndAgeOrderByCreatedAtDesc` → SQL with WHERE and ORDER BY

### Q3: "What's the difference between Page and List in pagination?"
**Answer:**
- **List**: Just data, no metadata
- **Page**: Data + total count, total pages, current page, hasNext, hasPrevious
- **Use Page for UI pagination**, List for simple data retrieval

---

## 🚀 Best Practices

### ✅ Do's:
```java
// 1. Use JpaRepository (most feature-rich)
public interface UserRepository extends JpaRepository<User, Long> { }

// 2. Use method naming for simple queries
List<User> findByEmail(String email);

// 3. Use @Query for complex queries
@Query("SELECT u FROM User u WHERE u.age > :minAge AND u.active = true")
List<User> findActiveUsersAboveAge(@Param("minAge") Integer minAge);

// 4. Use pagination for large datasets
Page<User> findAll(Pageable pageable);

// 5. Handle Optional properly
Optional<User> user = userRepository.findById(1L);
user.ifPresent(u -> System.out.println(u.getName()));
```

### ❌ Don'ts:
```java
// 1. Don't use CrudRepository (use JpaRepository instead)
public interface UserRepository extends CrudRepository<User, Long> { } // Limited

// 2. Don't create overly complex method names
findByNameAndAgeAndEmailAndCityAndCountryOrderByCreatedAtDesc // Too long!

// 3. Don't call repository.findAll() without pagination
List<User> users = userRepository.findAll(); // May load millions of records!

// 4. Don't ignore Optional
User user = userRepository.findById(1L).get(); // May throw exception!
```

---
