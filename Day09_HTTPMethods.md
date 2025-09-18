# 📅 Day 9: HTTP Method Annotations

## 🎯 Today Topics
- Master `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`
- Understand HTTP semantics and proper method usage
- Build complete CRUD APIs with best practices
- Handle different request/response scenarios
- Real-world examples with proper HTTP status codes

---

## 🌐 HTTP Method Annotations Overview

### Why Specific Method Annotations?
Instead of using `@RequestMapping(method = RequestMethod.GET)`, Spring provides convenient shortcuts that make code cleaner and more expressive.

### Method Annotations Comparison:
```java
// Old way - verbose
@RequestMapping(value = "/users", method = RequestMethod.GET)
public List<User> getUsers() { }

// New way - clean and expressive  
@GetMapping("/users")
public List<User> getUsers() { }
```

### HTTP Methods & Their Purpose:
| Method | Purpose | Idempotent | Safe | Body |
|--------|---------|------------|------|------|
| **GET** | Retrieve data | ✅ Yes | ✅ Yes | ❌ No |
| **POST** | Create new resource | ❌ No | ❌ No | ✅ Yes |
| **PUT** | Replace entire resource | ✅ Yes | ❌ No | ✅ Yes |
| **PATCH** | Partial update | ❌ No | ❌ No | ✅ Yes |
| **DELETE** | Remove resource | ✅ Yes | ❌ No | ❌ No |

---

## 📖 @GetMapping - Retrieving Data

### Basic @GetMapping Usage:
```java
@RestController
@RequestMapping("/api/books")
public class BookController {
    
    private final BookService bookService;
    
    public BookController(BookService bookService) {
        this.bookService = bookService;
    }
    
    // Get all books
    @GetMapping
    public List<Book> getAllBooks() {
        System.out.println("📚 Fetching all books");
        return bookService.findAll();
    }
    
    // Get single book by ID
    @GetMapping("/{id}")
    public ResponseEntity<Book> getBookById(@PathVariable Long id) {
        System.out.println("📖 Fetching book with ID: " + id);
        
        Optional<Book> book = bookService.findById(id);
        if (book.isPresent()) {
            return ResponseEntity.ok(book.get());
        } else {
            return ResponseEntity.notFound().build();
        }
    }
    
    // Get books by author
    @GetMapping("/author/{authorName}")
    public List<Book> getBooksByAuthor(@PathVariable String authorName) {
        System.out.println("👤 Fetching books by author: " + authorName);
        return bookService.findByAuthor(authorName);
    }
    
    // Get books with query parameters
    @GetMapping("/search")
    public List<Book> searchBooks(
            @RequestParam(required = false) String title,
            @RequestParam(required = false) String author,
            @RequestParam(required = false) String genre,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        
        System.out.printf("🔍 Searching books - Title: %s, Author: %s, Genre: %s, Page: %d, Size: %d%n",
                         title, author, genre, page, size);
        
        return bookService.searchBooks(title, author, genre, page, size);
    }
    
    // Get books published in a year range
    @GetMapping("/published")
    public List<Book> getBooksByYearRange(
            @RequestParam int startYear,
            @RequestParam int endYear) {
        
        System.out.printf("📅 Fetching books published between %d and %d%n", startYear, endYear);
        
        if (startYear > endYear) {
            throw new IllegalArgumentException("Start year cannot be greater than end year");
        }
        
        return bookService.findByPublicationYearBetween(startYear, endYear);
    }
}
```

---

## ➕ @PostMapping - Creating Resources

### @PostMapping for Resource Creation:
```java
@RestController
@RequestMapping("/api/books")
public class BookController {
    
    // Create new book
    @PostMapping
    public ResponseEntity<Book> createBook(@RequestBody Book book) {
        System.out.println("➕ Creating new book: " + book.getTitle());
        
        try {
            // Validate book data
            validateBookData(book);
            
            // Set creation timestamp
            book.setCreatedAt(LocalDateTime.now());
            book.setUpdatedAt(LocalDateTime.now());
            
            // Save book
            Book savedBook = bookService.save(book);
            
            System.out.println("✅ Book created successfully with ID: " + savedBook.getId());
            
            // Return 201 Created with Location header
            return ResponseEntity
                    .status(HttpStatus.CREATED)
                    .header("Location", "/api/books/" + savedBook.getId())
                    .body(savedBook);
                    
        } catch (IllegalArgumentException e) {
            System.out.println("❌ Invalid book data: " + e.getMessage());
            return ResponseEntity.badRequest().build();
        } catch (Exception e) {
            System.out.println("💥 Error creating book: " + e.getMessage());
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
    
    // Bulk create books
    @PostMapping("/batch")
    public ResponseEntity<List<Book>> createBooks(@RequestBody List<Book> books) {
        System.out.println("📦 Creating " + books.size() + " books in batch");
        
        try {
            List<Book> savedBooks = new ArrayList<>();
            
            for (Book book : books) {
                validateBookData(book);
                book.setCreatedAt(LocalDateTime.now());
                book.setUpdatedAt(LocalDateTime.now());
                
                Book savedBook = bookService.save(book);
                savedBooks.add(savedBook);
            }
            
            System.out.println("✅ Batch created " + savedBooks.size() + " books successfully");
            return ResponseEntity.status(HttpStatus.CREATED).body(savedBooks);
            
        } catch (Exception e) {
            System.out.println("❌ Batch creation failed: " + e.getMessage());
            return ResponseEntity.badRequest().build();
        }
    }
    
    // Upload book cover image
    @PostMapping("/{id}/cover")
    public ResponseEntity<String> uploadBookCover(
            @PathVariable Long id,
            @RequestParam("file") MultipartFile file) {
        
        System.out.println("🖼️ Uploading cover for book ID: " + id);
        
        try {
            // Validate file
            if (file.isEmpty()) {
                return ResponseEntity.badRequest().body("File is empty");
            }
            
            if (!isImageFile(file)) {
                return ResponseEntity.badRequest().body("File must be an image");
            }
            
            // Check if book exists
            if (!bookService.existsById(id)) {
                return ResponseEntity.notFound().build();
            }
            
            // Save file and update book
            String fileName = bookService.saveCoverImage(id, file);
            
            System.out.println("✅ Cover uploaded successfully: " + fileName);
            return ResponseEntity.ok("Cover uploaded successfully: " + fileName);
            
        } catch (IOException e) {
            System.out.println("💥 File upload error: " + e.getMessage());
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body("File upload failed");
        }
    }
    
    private void validateBookData(Book book) {
        if (book.getTitle() == null || book.getTitle().trim().isEmpty()) {
            throw new IllegalArgumentException("Book title is required");
        }
        
        if (book.getAuthor() == null || book.getAuthor().trim().isEmpty()) {
            throw new IllegalArgumentException("Book author is required");
        }
        
        if (book.getPrice() != null && book.getPrice() < 0) {
            throw new IllegalArgumentException("Book price cannot be negative");
        }
        
        if (book.getPublicationYear() != null) {
            int currentYear = LocalDate.now().getYear();
            if (book.getPublicationYear() < 1000 || book.getPublicationYear() > currentYear + 1) {
                throw new IllegalArgumentException("Invalid publication year");
            }
        }
    }
    
    private boolean isImageFile(MultipartFile file) {
        String contentType = file.getContentType();
        return contentType != null && contentType.startsWith("image/");
    }
}
```

---

## 🔄 @PutMapping - Full Resource Replacement

### @PutMapping for Complete Updates:
```java
@RestController
@RequestMapping("/api/books")
public class BookController {
    
    // Update entire book (replace)
    @PutMapping("/{id}")
    public ResponseEntity<Book> updateBook(@PathVariable Long id, @RequestBody Book updatedBook) {
        System.out.println("🔄 Updating book with ID: " + id);
        
        try {
            // Check if book exists
            Optional<Book> existingBookOpt = bookService.findById(id);
            if (!existingBookOpt.isPresent()) {
                System.out.println("❌ Book not found with ID: " + id);
                return ResponseEntity.notFound().build();
            }
            
            Book existingBook = existingBookOpt.get();
            
            // Validate updated data
            validateBookData(updatedBook);
            
            // Keep original ID and creation time, update everything else
            updatedBook.setId(id);
            updatedBook.setCreatedAt(existingBook.getCreatedAt());
            updatedBook.setUpdatedAt(LocalDateTime.now());
            
            // Save updated book
            Book savedBook = bookService.save(updatedBook);
            
            System.out.println("✅ Book updated successfully");
            return ResponseEntity.ok(savedBook);
            
        } catch (IllegalArgumentException e) {
            System.out.println("❌ Invalid book data: " + e.getMessage());
            return ResponseEntity.badRequest().build();
        } catch (Exception e) {
            System.out.println("💥 Error updating book: " + e.getMessage());
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
    
    // Update book availability status
    @PutMapping("/{id}/availability")
    public ResponseEntity<Book> updateBookAvailability(
            @PathVariable Long id,
            @RequestBody Map<String, Object> availabilityData) {
        
        System.out.println("📦 Updating availability for book ID: " + id);
        
        try {
            Optional<Book> bookOpt = bookService.findById(id);
            if (!bookOpt.isPresent()) {
                return ResponseEntity.notFound().build();
            }
            
            Book book = bookOpt.get();
            
            // Update availability fields
            if (availabilityData.containsKey("inStock")) {
                book.setInStock((Boolean) availabilityData.get("inStock"));
            }
            
            if (availabilityData.containsKey("quantity")) {
                Integer quantity = (Integer) availabilityData.get("quantity");
                if (quantity < 0) {
                    throw new IllegalArgumentException("Quantity cannot be negative");
                }
                book.setQuantity(quantity);
                book.setInStock(quantity > 0);
            }
            
            if (availabilityData.containsKey("restockDate")) {
                String restockDateStr = (String) availabilityData.get("restockDate");
                book.setRestockDate(LocalDate.parse(restockDateStr));
            }
            
            book.setUpdatedAt(LocalDateTime.now());
            Book savedBook = bookService.save(book);
            
            System.out.printf("✅ Availability updated - In Stock: %s, Quantity: %d%n",
                            savedBook.getInStock(), savedBook.getQuantity());
            
            return ResponseEntity.ok(savedBook);
            
        } catch (Exception e) {
            System.out.println("❌ Error updating availability: " + e.getMessage());
            return ResponseEntity.badRequest().build();
        }
    }
}
```

---

## 🔧 @PatchMapping - Partial Updates

### @PatchMapping for Selective Updates:
```java
@RestController
@RequestMapping("/api/books")
public class BookController {
    
    // Partial update - only update provided fields
    @PatchMapping("/{id}")
    public ResponseEntity<Book> partialUpdateBook(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {
        
        System.out.println("🔧 Partial update for book ID: " + id);
        System.out.println("Fields to update: " + updates.keySet());
        
        try {
            Optional<Book> bookOpt = bookService.findById(id);
            if (!bookOpt.isPresent()) {
                System.out.println("❌ Book not found with ID: " + id);
                return ResponseEntity.notFound().build();
            }
            
            Book book = bookOpt.get();
            
            // Update only provided fields
            updates.forEach((field, value) -> {
                switch (field) {
                    case "title":
                        if (value != null && !value.toString().trim().isEmpty()) {
                            book.setTitle(value.toString());
                            System.out.println("  ✏️ Updated title: " + value);
                        }
                        break;
                        
                    case "author":
                        if (value != null && !value.toString().trim().isEmpty()) {
                            book.setAuthor(value.toString());
                            System.out.println("  ✏️ Updated author: " + value);
                        }
                        break;
                        
                    case "price":
                        if (value != null) {
                            Double price = Double.valueOf(value.toString());
                            if (price >= 0) {
                                book.setPrice(price);
                                System.out.println("  ✏️ Updated price: $" + price);
                            } else {
                                throw new IllegalArgumentException("Price cannot be negative");
                            }
                        }
                        break;
                        
                    case "genre":
                        if (value != null) {
                            book.setGenre(value.toString());
                            System.out.println("  ✏️ Updated genre: " + value);
                        }
                        break;
                        
                    case "description":
                        book.setDescription(value != null ? value.toString() : null);
                        System.out.println("  ✏️ Updated description");
                        break;
                        
                    case "publicationYear":
                        if (value != null) {
                            Integer year = Integer.valueOf(value.toString());
                            int currentYear = LocalDate.now().getYear();
                            if (year >= 1000 && year <= currentYear + 1) {
                                book.setPublicationYear(year);
                                System.out.println("  ✏️ Updated publication year: " + year);
                            } else {
                                throw new IllegalArgumentException("Invalid publication year");
                            }
                        }
                        break;
                        
                    case "rating":
                        if (value != null) {
                            Double rating = Double.valueOf(value.toString());
                            if (rating >= 0 && rating <= 5) {
                                book.setRating(rating);
                                System.out.println("  ✏️ Updated rating: " + rating);
                            } else {
                                throw new IllegalArgumentException("Rating must be between 0 and 5");
                            }
                        }
                        break;
                        
                    default:
                        System.out.println("  ⚠️ Ignored unknown field: " + field);
                        break;
                }
            });
            
            // Update timestamp
            book.setUpdatedAt(LocalDateTime.now());
            
            // Save updated book
            Book savedBook = bookService.save(book);
            
            System.out.println("✅ Partial update completed successfully");
            return ResponseEntity.ok(savedBook);
            
        } catch (IllegalArgumentException e) {
            System.out.println("❌ Invalid update data: " + e.getMessage());
            return ResponseEntity.badRequest().build();
        } catch (Exception e) {
            System.out.println("💥 Error during partial update: " + e.getMessage());
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
    
    // Update book rating
    @PatchMapping("/{id}/rating")
    public ResponseEntity<Book> updateBookRating(@PathVariable Long id, @RequestBody Map<String, Double> ratingData) {
        System.out.println("⭐ Updating rating for book ID: " + id);
        
        try {
            Optional<Book> bookOpt = bookService.findById(id);
            if (!bookOpt.isPresent()) {
                return ResponseEntity.notFound().build();
            }
            
            Book book = bookOpt.get();
            Double newRating = ratingData.get("rating");
            
            if (newRating == null || newRating < 0 || newRating > 5) {
                return ResponseEntity.badRequest().build();
            }
            
            // Update rating and review count
            book.setRating(newRating);
            book.setReviewCount(book.getReviewCount() + 1);
            book.setUpdatedAt(LocalDateTime.now());
            
            Book savedBook = bookService.save(book);
            
            System.out.printf("✅ Rating updated to %.1f (Total reviews: %d)%n", 
                            newRating, savedBook.getReviewCount());
            
            return ResponseEntity.ok(savedBook);
            
        } catch (Exception e) {
            System.out.println("❌ Error updating rating: " + e.getMessage());
            return ResponseEntity.badRequest().build();
        }
    }
}
```

---

## 🗑️ @DeleteMapping - Removing Resources

### @DeleteMapping for Resource Removal:
```java
@RestController
@RequestMapping("/api/books")
public class BookController {
    
    // Delete single book
    @DeleteMapping("/{id}")
    public ResponseEntity<String> deleteBook(@PathVariable Long id) {
        System.out.println("🗑️ Deleting book with ID: " + id);
        
        try {
            // Check if book exists
            if (!bookService.existsById(id)) {
                System.out.println("❌ Book not found with ID: " + id);
                return ResponseEntity.notFound().build();
            }
            
            // Get book details for logging
            Optional<Book> bookOpt = bookService.findById(id);
            if (bookOpt.isPresent()) {
                Book book = bookOpt.get();
                System.out.printf("📖 Deleting book: '%s' by %s%n", book.getTitle(), book.getAuthor());
                
                // Check if book has active orders/reservations
                if (bookService.hasActiveReservations(id)) {
                    System.out.println("⚠️ Cannot delete book with active reservations");
                    return ResponseEntity.status(HttpStatus.CONFLICT)
                            .body("Cannot delete book with active reservations");
                }
                
                // Soft delete (mark as deleted instead of actual removal)
                bookService.softDelete(id);
                
                System.out.println("✅ Book marked as deleted successfully");
                return ResponseEntity.ok("Book deleted successfully");
            }
            
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body("Error occurred while deleting book");
            
        } catch (Exception e) {
            System.out.println("💥 Error deleting book: " + e.getMessage());
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body("Error occurred while deleting book");
        }
    }
    
    // Permanently delete book (admin only)
    @DeleteMapping("/{id}/permanent")
    public ResponseEntity<String> permanentlyDeleteBook(
            @PathVariable Long id,
            @RequestHeader("X-Admin-Token") String adminToken) {
        
        System.out.println("💥 Permanent deletion requested for book ID: " + id);
        
        try {
            // Validate admin token
            if (!bookService.validateAdminToken(adminToken)) {
                System.out.println("❌ Invalid admin token");
                return ResponseEntity.status(HttpStatus.FORBIDDEN)
                        .body("Admin access required");
            }
            
            // Check if book exists
            if (!bookService.existsById(id)) {
                return ResponseEntity.notFound().build();
            }
            
            // Get book details for audit log
            Optional<Book> bookOpt = bookService.findById(id);
            if (bookOpt.isPresent()) {
                Book book = bookOpt.get();
                
                // Log the permanent deletion
                System.out.printf("🔥 PERMANENT DELETION: '%s' by %s (ID: %d)%n", 
                                book.getTitle(), book.getAuthor(), id);
                
                // Perform actual deletion
                bookService.permanentlyDelete(id);
                
                System.out.println("✅ Book permanently deleted from database");
                return ResponseEntity.ok("Book permanently deleted");
            }
            
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body("Error occurred during deletion");
            
        } catch (Exception e) {
            System.out.println("💥 Error during permanent deletion: " + e.getMessage());
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body("Permanent deletion failed");
        }
    }
    
    // Delete multiple books
    @DeleteMapping("/batch")
    public ResponseEntity<Map<String, Object>> deleteBooks(@RequestBody List<Long> bookIds) {
        System.out.println("🗑️ Batch deleting " + bookIds.size() + " books");
        
        Map<String, Object> result = new HashMap<>();
        List<Long> deletedIds = new ArrayList<>();
        List<Long> notFoundIds = new ArrayList<>();
        List<Long> conflictIds = new ArrayList<>();
        
        try {
            for (Long id : bookIds) {
                if (!bookService.existsById(id)) {
                    notFoundIds.add(id);
                    continue;
                }
                
                if (bookService.hasActiveReservations(id)) {
                    conflictIds.add(id);
                    continue;
                }
                
                bookService.softDelete(id);
                deletedIds.add(id);
                System.out.println("  ✅ Deleted book ID: " + id);
            }
            
            result.put("deleted", deletedIds);
            result.put("notFound", notFoundIds);
            result.put("conflicts", conflictIds);
            result.put("summary", String.format("Deleted: %d, Not Found: %d, Conflicts: %d",
                                               deletedIds.size(), notFoundIds.size(), conflictIds.size()));
            
            System.out.printf("✅ Batch deletion completed - Deleted: %d, Not Found: %d, Conflicts: %d%n",
                            deletedIds.size(), notFoundIds.size(), conflictIds.size());
            
            if (deletedIds.isEmpty()) {
                return ResponseEntity.badRequest().body(result);
            }
            
            return ResponseEntity.ok(result);
            
        } catch (Exception e) {
            System.out.println("💥 Error during batch deletion: " + e.getMessage());
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(result);
        }
    }
    
    // Restore deleted book
    @DeleteMapping("/{id}/restore")
    public ResponseEntity<Book> restoreBook(@PathVariable Long id) {
        System.out.println("🔄 Restoring deleted book ID: " + id);
        
        try {
            Optional<Book> restoredBook = bookService.restoreDeleted(id);
            
            if (restoredBook.isPresent()) {
                System.out.printf("✅ Book restored: '%s' by %s%n", 
                                restoredBook.get().getTitle(), restoredBook.get().getAuthor());
                return ResponseEntity.ok(restoredBook.get());
            } else {
                System.out.println("❌ No deleted book found with ID: " + id);
                return ResponseEntity.notFound().build();
            }
            
        } catch (Exception e) {
            System.out.println("💥 Error restoring book: " + e.getMessage());
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
}
```

---

## 📚 Complete Book Model & Service

### Book Model:
```java
public class Book {
    private Long id;
    private String title;
    private String author;
    private String genre;
    private String description;
    private Double price;
    private Integer publicationYear;
    private String isbn;
    private Boolean inStock;
    private Integer quantity;
    private LocalDate restockDate;
    private Double rating;
    private Integer reviewCount;
    private String coverImageUrl;
    private Boolean deleted;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    // Constructors
    public Book() {
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
        this.inStock = true;
        this.quantity = 0;
        this.rating = 0.0;
        this.reviewCount = 0;
        this.deleted = false;
    }
    
    public Book(String title, String author, String genre, Double price) {
        this();
        this.title = title;
        this.author = author;
        this.genre = genre;
        this.price = price;
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
    
    public String getGenre() { return genre; }
    public void setGenre(String genre) { this.genre = genre; }
    
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    
    public Double getPrice() { return price; }
    public void setPrice(Double price) { this.price = price; }
    
    public Integer getPublicationYear() { return publicationYear; }
    public void setPublicationYear(Integer publicationYear) { this.publicationYear = publicationYear; }
    
    public String getIsbn() { return isbn; }
    public void setIsbn(String isbn) { this.isbn = isbn; }
    
    public Boolean getInStock() { return inStock; }
    public void setInStock(Boolean inStock) { this.inStock = inStock; }
    
    public Integer getQuantity() { return quantity; }
    public void setQuantity(Integer quantity) { this.quantity = quantity; }
    
    public LocalDate getRestockDate() { return restockDate; }
    public void setRestockDate(LocalDate restockDate) { this.restockDate = restockDate; }
    
    public Double getRating() { return rating; }
    public void setRating(Double rating) { this.rating = rating; }
    
    public Integer getReviewCount() { return reviewCount; }
    public void setReviewCount(Integer reviewCount) { this.reviewCount = reviewCount; }
    
    public String getCoverImageUrl() { return coverImageUrl; }
    public void setCoverImageUrl(String coverImageUrl) { this.coverImageUrl = coverImageUrl; }
    
    public Boolean getDeleted() { return deleted; }
    public void setDeleted(Boolean deleted) { this.deleted = deleted; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
}
```

### Book Service (Simulated):
```java
@Service
public class BookService {
    
    private final Map<Long, Book> books = new ConcurrentHashMap<>();
    private final AtomicLong idGenerator = new AtomicLong(1);
    
    public BookService() {
        initializeSampleBooks();
    }
    
    public List<Book> findAll() {
        return books.values().stream()
                .filter(book -> !book.getDeleted())
                .collect(Collectors.toList());
    }
    
    public Optional<Book> findById(Long id) {
        Book book = books.get(id);
        return (book != null && !book.getDeleted()) ? Optional.of(book) : Optional.empty();
    }
    
    public List<Book> findByAuthor(String author) {
        return books.values().stream()
                .filter(book -> !book.getDeleted())
                .filter(book -> book.getAuthor().toLowerCase().contains(author.toLowerCase()))
                .collect(Collectors.toList());
    }
    
    public List<Book> searchBooks(String title, String author, String genre, int page, int size) {
        return books.values().stream()
                .filter(book -> !book.getDeleted())
                .filter(book -> title == null || book.getTitle().toLowerCase().contains(title.toLowerCase()))
                .filter(book -> author == null || book.getAuthor().toLowerCase().contains(author.toLowerCase()))
                .filter(book -> genre == null || book.getGenre().toLowerCase().contains(genre.toLowerCase()))
                .skip(page * size)
                .limit(size)
                .collect(Collectors.toList());
    }
    
    public List<Book> findByPublicationYearBetween(int startYear, int endYear) {
        return books.values().stream()
                .filter(book -> !book.getDeleted())
                .filter(book -> book.getPublicationYear() != null)
                .filter(book -> book.getPublicationYear() >= startYear && book.getPublicationYear() <= endYear)
                .collect(Collectors.toList());
    }
    
    public Book save(Book book) {
        if (book.getId() == null) {
            book.setId(idGenerator.getAndIncrement());
        }
        books.put(book.getId(), book);
        return book;
    }
    
    public boolean existsById(Long id) {
        Book book = books.get(id);
        return book != null && !book.getDeleted();
    }
    
    public void softDelete(Long id) {
        Book book = books.get(id);
        if (book != null) {
            book.setDeleted(true);
            book.setUpdatedAt(LocalDateTime.now());
        }
    }
    
    public void permanentlyDelete(Long id) {
        books.remove(id);
    }
    
    public Optional<Book> restoreDeleted(Long id) {
        Book book = books.get(id);
        if (book != null && book.getDeleted()) {
            book.setDeleted(false);
            book.setUpdatedAt(LocalDateTime.now());
            return Optional.of(book);
        }
        return Optional.empty();
    }
    
    public boolean hasActiveReservations(Long id) {
        // Simulate checking for active reservations/orders
        // In real application, this would query a reservations/orders table
        return Math.random() < 0.3; // 30% chance of having active reservations
    }
    
    public boolean validateAdminToken(String token) {
        // Simulate admin token validation
        return "admin-secret-token-123".equals(token);
    }
    
    public String saveCoverImage(Long bookId, MultipartFile file) throws IOException {
        // Simulate saving cover image
        String fileName = "cover_" + bookId + "_" + System.currentTimeMillis() + ".jpg";
        
        // In real application, you would save to file system or cloud storage
        // Files.copy(file.getInputStream(), Paths.get("uploads/" + fileName));
        
        // Update book with cover image URL
        Book book = books.get(bookId);
        if (book != null) {
            book.setCoverImageUrl("/images/covers/" + fileName);
            book.setUpdatedAt(LocalDateTime.now());
        }
        
        return fileName;
    }
    
    private void initializeSampleBooks() {
        // Create sample books
        Book book1 = new Book("Spring Boot in Action", "Craig Walls", "Technology", 45.99);
        book1.setId(1L);
        book1.setPublicationYear(2018);
        book1.setIsbn("978-1617292545");
        book1.setQuantity(25);
        book1.setRating(4.5);
        book1.setReviewCount(128);
        book1.setDescription("A comprehensive guide to building applications with Spring Boot");
        books.put(1L, book1);
        
        Book book2 = new Book("Clean Code", "Robert C. Martin", "Programming", 39.99);
        book2.setId(2L);
        book2.setPublicationYear(2008);
        book2.setIsbn("978-0132350884");
        book2.setQuantity(15);
        book2.setRating(4.8);
        book2.setReviewCount(205);
        book2.setDescription("A handbook of agile software craftsmanship");
        books.put(2L, book2);
        
        Book book3 = new Book("Design Patterns", "Gang of Four", "Software Engineering", 55.99);
        book3.setId(3L);
        book3.setPublicationYear(1994);
        book3.setIsbn("978-0201633612");
        book3.setQuantity(8);
        book3.setRating(4.6);
        book3.setReviewCount(89);
        book3.setDescription("Elements of reusable object-oriented software");
        books.put(3L, book3);
        
        Book book4 = new Book("The Pragmatic Programmer", "David Thomas", "Programming", 42.99);
        book4.setId(4L);
        book4.setPublicationYear(1999);
        book4.setIsbn("978-0201616224");
        book4.setQuantity(0);
        book4.setInStock(false);
        book4.setRestockDate(LocalDate.now().plusDays(14));
        book4.setRating(4.7);
        book4.setReviewCount(156);
        book4.setDescription("From journeyman to master");
        books.put(4L, book4);
        
        idGenerator.set(5L);
        System.out.println("✅ Initialized " + books.size() + " sample books");
    }
}
```

---

## 🧪 API Testing Examples

### Testing with curl Commands:

```bash
# GET - Fetch all books
curl -X GET "http://localhost:8080/api/books"

# GET - Fetch book by ID
curl -X GET "http://localhost:8080/api/books/1"

# GET - Search books with query parameters
curl -X GET "http://localhost:8080/api/books/search?title=spring&author=walls&page=0&size=10"

# GET - Books by author
curl -X GET "http://localhost:8080/api/books/author/Robert%20C.%20Martin"

# GET - Books published in year range
curl -X GET "http://localhost:8080/api/books/published?startYear=2000&endYear=2020"

# POST - Create new book
curl -X POST "http://localhost:8080/api/books" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Java: The Complete Reference",
    "author": "Herbert Schildt",
    "genre": "Programming",
    "price": 49.99,
    "publicationYear": 2020,
    "isbn": "978-1260440232",
    "quantity": 10,
    "description": "Comprehensive guide to Java programming"
  }'

# POST - Batch create books
curl -X POST "http://localhost:8080/api/books/batch" \
  -H "Content-Type: application/json" \
  -d '[
    {
      "title": "Effective Java",
      "author": "Joshua Bloch",
      "genre": "Programming",
      "price": 44.99,
      "publicationYear": 2018
    },
    {
      "title": "Java Concurrency in Practice",
      "author": "Brian Goetz",
      "genre": "Programming", 
      "price": 39.99,
      "publicationYear": 2006
    }
  ]'

# PUT - Update entire book
curl -X PUT "http://localhost:8080/api/books/1" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Spring Boot in Action - 2nd Edition",
    "author": "Craig Walls",
    "genre": "Technology",
    "price": 49.99,
    "publicationYear": 2021,
    "isbn": "978-1617292545",
    "quantity": 30,
    "description": "Updated comprehensive guide to Spring Boot 2.x"
  }'

# PATCH - Partial update (price and quantity only)
curl -X PATCH "http://localhost:8080/api/books/2" \
  -H "Content-Type: application/json" \
  -d '{
    "price": 34.99,
    "quantity": 20
  }'

# PATCH - Update book rating
curl -X PATCH "http://localhost:8080/api/books/3/rating" \
  -H "Content-Type: application/json" \
  -d '{
    "rating": 4.9
  }'

# DELETE - Soft delete book
curl -X DELETE "http://localhost:8080/api/books/4"

# DELETE - Batch delete books
curl -X DELETE "http://localhost:8080/api/books/batch" \
  -H "Content-Type: application/json" \
  -d '[2, 3, 4]'

# DELETE - Permanent delete (admin only)
curl -X DELETE "http://localhost:8080/api/books/1/permanent" \
  -H "X-Admin-Token: admin-secret-token-123"

# DELETE - Restore deleted book
curl -X DELETE "http://localhost:8080/api/books/4/restore"

# POST - Upload book cover
curl -X POST "http://localhost:8080/api/books/1/cover" \
  -F "file=@book-cover.jpg"
```

---

## 📊 HTTP Status Codes Best Practices

### Proper Status Code Usage:

```java
@RestController
public class BookController {
    
    @GetMapping("/{id}")
    public ResponseEntity<Book> getBook(@PathVariable Long id) {
        Optional<Book> book = bookService.findById(id);
        
        if (book.isPresent()) {
            return ResponseEntity.ok(book.get());           // 200 OK
        } else {
            return ResponseEntity.notFound().build();       // 404 NOT FOUND
        }
    }
    
    @PostMapping
    public ResponseEntity<Book> createBook(@RequestBody Book book) {
        try {
            Book savedBook = bookService.save(book);
            return ResponseEntity
                .status(HttpStatus.CREATED)                 // 201 CREATED
                .header("Location", "/api/books/" + savedBook.getId())
                .body(savedBook);
        } catch (ValidationException e) {
            return ResponseEntity.badRequest().build();     // 400 BAD REQUEST
        }
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Book> updateBook(@PathVariable Long id, @RequestBody Book book) {
        if (!bookService.existsById(id)) {
            return ResponseEntity.notFound().build();       // 404 NOT FOUND
        }
        
        try {
            Book updatedBook = bookService.update(id, book);
            return ResponseEntity.ok(updatedBook);          // 200 OK
        } catch (ValidationException e) {
            return ResponseEntity.badRequest().build();     // 400 BAD REQUEST
        }
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteBook(@PathVariable Long id) {
        if (!bookService.existsById(id)) {
            return ResponseEntity.notFound().build();       // 404 NOT FOUND
        }
        
        if (bookService.hasActiveReservations(id)) {
            return ResponseEntity.status(HttpStatus.CONFLICT).build(); // 409 CONFLICT
        }
        
        bookService.delete(id);
        return ResponseEntity.noContent().build();          // 204 NO CONTENT
    }
    
    @GetMapping("/admin")
    public ResponseEntity<String> adminEndpoint(@RequestHeader("Authorization") String auth) {
        if (!isValidAdmin(auth)) {
            return ResponseEntity.status(HttpStatus.FORBIDDEN) // 403 FORBIDDEN
                .body("Admin access required");
        }
        
        return ResponseEntity.ok("Admin data");             // 200 OK
    }
}
```

### Status Code Reference:
| Code | Name | When to Use |
|------|------|-------------|
| **200** | OK | Successful GET, PUT, PATCH |
| **201** | Created | Successful POST |
| **204** | No Content | Successful DELETE |
| **400** | Bad Request | Invalid request data |
| **401** | Unauthorized | Authentication required |
| **403** | Forbidden | Access denied |
| **404** | Not Found | Resource doesn't exist |
| **409** | Conflict | Resource conflict (can't delete) |
| **422** | Unprocessable Entity | Validation errors |
| **500** | Internal Server Error | Server-side errors |

---

## 🎯 Interview Questions & Answers

### Q1: "What's the difference between PUT and PATCH?"
**Answer:**
- **PUT**: Replaces the entire resource - all fields must be provided
- **PATCH**: Partial update - only provided fields are updated
- **Idempotency**: PUT is idempotent, PATCH may or may not be
- **Use PUT for complete replacement, PATCH for selective updates**

### Q2: "Why use specific method annotations instead of @RequestMapping?"
**Answer:**
- **Cleaner Code**: `@GetMapping` vs `@RequestMapping(method = GET)`
- **Better Readability**: Intent is immediately clear
- **IDE Support**: Better autocomplete and error detection
- **Convention**: Follows REST conventions explicitly
- **Less Verbose**: Reduces boilerplate code

### Q3: "What makes HTTP methods idempotent?"
**Answer:**
- **Idempotent**: Same request can be made multiple times with same result
- **GET, PUT, DELETE**: Idempotent (safe to retry)
- **POST, PATCH**: Not idempotent (may cause side effects)
- **Important for**: Retry logic, caching, reliability

### Q4: "How do you handle partial updates safely?"
**Answer:**
```java
@PatchMapping("/{id}")
public ResponseEntity<Book> patchBook(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
    // 1. Fetch existing resource
    // 2. Validate each field in updates
    // 3. Apply only valid changes
    // 4. Return updated resource
}
```

### Q5: "What's the proper way to return HTTP status codes?"
**Answer:**
- **Use ResponseEntity**: Full control over status, headers, body
- **Match HTTP semantics**: 200 for success, 404 for not found, etc.
- **Consistent patterns**: Same status codes for similar operations
- **Include meaningful responses**: Error messages, success confirmations

---

## 🚀 Best Practices Summary

### 1. **Method Selection**
```java
// ✅ Good: Proper HTTP method usage
@GetMapping("/books")           // Retrieve data
@PostMapping("/books")          // Create new resource
@PutMapping("/books/{id}")      // Replace entire resource
@PatchMapping("/books/{id}")    // Update specific fields
@DeleteMapping("/books/{id}")   // Remove resource

// ❌ Avoid: Wrong method usage
@PostMapping("/getBooks")       // Should be GET
@GetMapping("/deleteBook/{id}") // Should be DELETE
```

### 2. **Response Structure**
```java
// ✅ Good: Consistent response structure
return ResponseEntity.ok(book);                    // 200 with data
return ResponseEntity.notFound().build();          // 404 no body
return ResponseEntity.status(CREATED).body(book);  // 201 with data

// ✅ Good: Error responses
return ResponseEntity.badRequest()
    .body("Invalid book data: " + error);
```

### 3. **Validation and Error Handling**
```java
@PostMapping("/books")
public ResponseEntity<Book> createBook(@RequestBody Book book) {
    try {
        validateBook(book);
        Book saved = bookService.save(book);
        return ResponseEntity.status(CREATED).body(saved);
    } catch (ValidationException e) {
        return ResponseEntity.badRequest().body(null);
    } catch (Exception e) {
        return ResponseEntity.status(INTERNAL_SERVER_ERROR).build();
    }
}
```

---
