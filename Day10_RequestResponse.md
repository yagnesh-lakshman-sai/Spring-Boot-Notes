# 📅 Day 10: Request & Response Handling

## 🎯 Today Topics
- Master `@RequestParam`, `@PathVariable`, `@RequestBody`, `@ResponseBody`
- Understand data binding and type conversion
- Handle different content types and formats
- Implement file upload/download functionality
- Build flexible APIs with content negotiation

---

## 🎯 Understanding Request Parameters

### @RequestParam - Query Parameters & Form Data
`@RequestParam` binds request parameters (query string or form data) to method parameters.

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    private final ProductService productService;
    
    public ProductController(ProductService productService) {
        this.productService = productService;
    }
    
    // Basic query parameters
    @GetMapping("/search")
    public List<Product> searchProducts(
            @RequestParam String name,                    // Required parameter
            @RequestParam(required = false) String category, // Optional parameter
            @RequestParam(defaultValue = "0") int page,   // Default value
            @RequestParam(defaultValue = "10") int size   // Default value
    ) {
        System.out.printf("🔍 Searching products: name='%s', category='%s', page=%d, size=%d%n",
                         name, category, page, size);
        
        return productService.searchProducts(name, category, page, size);
    }
    
    // Multiple values for same parameter
    @GetMapping("/filter")
    public List<Product> filterProducts(
            @RequestParam List<String> categories,     // ?categories=electronics&categories=books
            @RequestParam(required = false) List<String> tags,
            @RequestParam(defaultValue = "price") String sortBy
    ) {
        System.out.println("📋 Categories: " + categories);
        System.out.println("🏷️ Tags: " + tags);
        System.out.println("📈 Sort by: " + sortBy);
        
        return productService.filterProducts(categories, tags, sortBy);
    }
    
    // Parameter with different name
    @GetMapping("/price-range")
    public List<Product> getProductsByPriceRange(
            @RequestParam("min") Double minPrice,      // URL: ?min=10.99
            @RequestParam("max") Double maxPrice       // URL: ?max=99.99
    ) {
        System.out.printf("💰 Price range: $%.2f - $%.2f%n", minPrice, maxPrice);
        
        if (minPrice < 0 || maxPrice < 0) {
            throw new IllegalArgumentException("Price cannot be negative");
        }
        
        if (minPrice > maxPrice) {
            throw new IllegalArgumentException("Minimum price cannot be greater than maximum price");
        }
        
        return productService.findByPriceBetween(minPrice, maxPrice);
    }
    
    // Complex filtering with optional parameters
    @GetMapping("/advanced-search")
    public ResponseEntity<Map<String, Object>> advancedSearch(
            @RequestParam(required = false) String keyword,
            @RequestParam(required = false) String category,
            @RequestParam(required = false) Double minPrice,
            @RequestParam(required = false) Double maxPrice,
            @RequestParam(required = false) Integer minRating,
            @RequestParam(defaultValue = "created_date") String sortBy,
            @RequestParam(defaultValue = "desc") String sortDirection,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size
    ) {
        System.out.println("🎯 Advanced search with parameters:");
        System.out.println("  Keyword: " + keyword);
        System.out.println("  Category: " + category);
        System.out.println("  Price range: " + minPrice + " - " + maxPrice);
        System.out.println("  Min rating: " + minRating);
        System.out.println("  Sort: " + sortBy + " " + sortDirection);
        System.out.println("  Pagination: page " + page + ", size " + size);
        
        SearchCriteria criteria = SearchCriteria.builder()
            .keyword(keyword)
            .category(category)
            .minPrice(minPrice)
            .maxPrice(maxPrice)
            .minRating(minRating)
            .sortBy(sortBy)
            .sortDirection(sortDirection)
            .page(page)
            .size(size)
            .build();
        
        SearchResult result = productService.advancedSearch(criteria);
        
        Map<String, Object> response = new HashMap<>();
        response.put("products", result.getProducts());
        response.put("totalElements", result.getTotalElements());
        response.put("totalPages", result.getTotalPages());
        response.put("currentPage", page);
        response.put("pageSize", size);
        response.put("hasNext", result.isHasNext());
        response.put("hasPrevious", page > 0);
        
        return ResponseEntity.ok(response);
    }
}
```

---

## 🛤️ @PathVariable - URL Path Parameters

### Basic Path Variables:
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    // Single path variable
    @GetMapping("/{userId}")
    public ResponseEntity<User> getUser(@PathVariable Long userId) {
        System.out.println("👤 Getting user with ID: " + userId);
        
        Optional<User> user = userService.findById(userId);
        if (user.isPresent()) {
            return ResponseEntity.ok(user.get());
        } else {
            return ResponseEntity.notFound().build();
        }
    }
    
    // Multiple path variables
    @GetMapping("/{userId}/orders/{orderId}")
    public ResponseEntity<Order> getUserOrder(
            @PathVariable Long userId,
            @PathVariable Long orderId
    ) {
        System.out.printf("📋 Getting order %d for user %d%n", orderId, userId);
        
        // Verify user exists
        if (!userService.existsById(userId)) {
            return ResponseEntity.notFound().build();
        }
        
        Optional<Order> order = userService.findUserOrder(userId, orderId);
        return order.map(ResponseEntity::ok)
                   .orElse(ResponseEntity.notFound().build());
    }
    
    // Path variable with different name
    @GetMapping("/profile/{username}")
    public ResponseEntity<User> getUserByUsername(
            @PathVariable("username") String userName  // URL path uses 'username', parameter is 'userName'
    ) {
        System.out.println("🔍 Finding user by username: " + userName);
        
        Optional<User> user = userService.findByUsername(userName);
        return user.map(ResponseEntity::ok)
                   .orElse(ResponseEntity.notFound().build());
    }
    
    // Optional path variable (using method overloading)
    @GetMapping("/{userId}/orders")
    public List<Order> getUserOrders(@PathVariable Long userId) {
        return getUserOrdersWithStatus(userId, null);
    }
    
    @GetMapping("/{userId}/orders/status/{status}")
    public List<Order> getUserOrdersWithStatus(
            @PathVariable Long userId,
            @PathVariable String status
    ) {
        System.out.printf("📦 Getting orders for user %d with status: %s%n", userId, status);
        
        if (status == null) {
            return userService.findUserOrders(userId);
        } else {
            return userService.findUserOrdersByStatus(userId, status);
        }
    }
    
    // Complex path with validation
    @GetMapping("/{userId}/settings/{settingCategory}/{settingKey}")
    public ResponseEntity<String> getUserSetting(
            @PathVariable Long userId,
            @PathVariable String settingCategory,
            @PathVariable String settingKey
    ) {
        System.out.printf("⚙️ Getting setting %s/%s for user %d%n", 
                         settingCategory, settingKey, userId);
        
        // Validate setting category
        if (!isValidSettingCategory(settingCategory)) {
            return ResponseEntity.badRequest().build();
        }
        
        Optional<String> setting = userService.getUserSetting(userId, settingCategory, settingKey);
        return setting.map(ResponseEntity::ok)
                     .orElse(ResponseEntity.notFound().build());
    }
    
    private boolean isValidSettingCategory(String category) {
        return List.of("privacy", "notifications", "preferences", "security")
                   .contains(category.toLowerCase());
    }
}
```

---

## 📥 @RequestBody - Request Body Processing

### Handling JSON Request Bodies:
```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    private final OrderService orderService;
    private final ObjectMapper objectMapper;
    
    public OrderController(OrderService orderService, ObjectMapper objectMapper) {
        this.orderService = orderService;
        this.objectMapper = objectMapper;
    }
    
    // Simple request body
    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody OrderRequest request) {
        System.out.println("📝 Creating new order:");
        System.out.println("  Customer ID: " + request.getCustomerId());
        System.out.println("  Items count: " + request.getItems().size());
        System.out.println("  Shipping address: " + request.getShippingAddress().getStreet());
        
        try {
            // Validate request
            validateOrderRequest(request);
            
            // Create order
            Order order = orderService.createOrder(request);
            
            System.out.println("✅ Order created with ID: " + order.getId());
            return ResponseEntity.status(HttpStatus.CREATED).body(order);
            
        } catch (ValidationException e) {
            System.out.println("❌ Validation failed: " + e.getMessage());
            return ResponseEntity.badRequest().build();
        } catch (Exception e) {
            System.out.println("💥 Error creating order: " + e.getMessage());
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
    
    // Complex nested request body
    @PostMapping("/bulk")
    public ResponseEntity<List<Order>> createBulkOrders(@RequestBody BulkOrderRequest request) {
        System.out.println("📦 Creating bulk orders:");
        System.out.println("  Batch ID: " + request.getBatchId());
        System.out.println("  Orders count: " + request.getOrders().size());
        System.out.println("  Priority: " + request.getPriority());
        
        try {
            List<Order> createdOrders = new ArrayList<>();
            
            for (OrderRequest orderRequest : request.getOrders()) {
                validateOrderRequest(orderRequest);
                Order order = orderService.createOrder(orderRequest);
                createdOrders.add(order);
            }
            
            System.out.printf("✅ Bulk order completed - %d orders created%n", createdOrders.size());
            return ResponseEntity.status(HttpStatus.CREATED).body(createdOrders);
            
        } catch (Exception e) {
            System.out.println("❌ Bulk order failed: " + e.getMessage());
            return ResponseEntity.badRequest().build();
        }
    }
    
    // Partial update with request body
    @PatchMapping("/{orderId}")
    public ResponseEntity<Order> updateOrder(
            @PathVariable Long orderId,
            @RequestBody Map<String, Object> updates
    ) {
        System.out.println("🔧 Updating order " + orderId + " with fields: " + updates.keySet());
        
        try {
            Optional<Order> orderOpt = orderService.findById(orderId);
            if (!orderOpt.isPresent()) {
                return ResponseEntity.notFound().build();
            }
            
            Order order = orderOpt.get();
            
            // Apply updates
            updates.forEach((field, value) -> {
                switch (field) {
                    case "status":
                        if (isValidOrderStatus(value.toString())) {
                            order.setStatus(value.toString());
                            System.out.println("  ✏️ Updated status: " + value);
                        }
                        break;
                        
                    case "shippingAddress":
                        try {
                            Address address = objectMapper.convertValue(value, Address.class);
                            order.setShippingAddress(address);
                            System.out.println("  ✏️ Updated shipping address");
                        } catch (Exception e) {
                            System.out.println("  ⚠️ Invalid address format");
                        }
                        break;
                        
                    case "specialInstructions":
                        order.setSpecialInstructions(value.toString());
                        System.out.println("  ✏️ Updated special instructions");
                        break;
                        
                    default:
                        System.out.println("  ⚠️ Ignored unknown field: " + field);
                        break;
                }
            });
            
            Order updatedOrder = orderService.save(order);
            System.out.println("✅ Order updated successfully");
            
            return ResponseEntity.ok(updatedOrder);
            
        } catch (Exception e) {
            System.out.println("❌ Error updating order: " + e.getMessage());
            return ResponseEntity.badRequest().build();
        }
    }
    
    // Request body with custom validation
    @PostMapping("/quick-order")
    public ResponseEntity<Order> createQuickOrder(@RequestBody QuickOrderRequest request) {
        System.out.println("⚡ Creating quick order:");
        System.out.println("  Product ID: " + request.getProductId());
        System.out.println("  Quantity: " + request.getQuantity());
        System.out.println("  Use default address: " + request.isUseDefaultAddress());
        
        try {
            // Custom validation for quick orders
            if (request.getQuantity() <= 0 || request.getQuantity() > 10) {
                throw new ValidationException("Quick order quantity must be between 1 and 10");
            }
            
            if (request.getProductId() == null) {
                throw new ValidationException("Product ID is required for quick order");
            }
            
            Order order = orderService.createQuickOrder(request);
            
            System.out.println("✅ Quick order created with ID: " + order.getId());
            return ResponseEntity.status(HttpStatus.CREATED).body(order);
            
        } catch (ValidationException e) {
            System.out.println("❌ Quick order validation failed: " + e.getMessage());
            return ResponseEntity.badRequest().build();
        }
    }
    
    private void validateOrderRequest(OrderRequest request) {
        if (request.getCustomerId() == null) {
            throw new ValidationException("Customer ID is required");
        }
        
        if (request.getItems() == null || request.getItems().isEmpty()) {
            throw new ValidationException("Order must contain at least one item");
        }
        
        if (request.getShippingAddress() == null) {
            throw new ValidationException("Shipping address is required");
        }
        
        // Validate each order item
        for (OrderItem item : request.getItems()) {
            if (item.getProductId() == null) {
                throw new ValidationException("Product ID is required for all items");
            }
            
            if (item.getQuantity() <= 0) {
                throw new ValidationException("Item quantity must be positive");
            }
        }
    }
    
    private boolean isValidOrderStatus(String status) {
        return List.of("PENDING", "CONFIRMED", "PROCESSING", "SHIPPED", "DELIVERED", "CANCELLED")
                   .contains(status.toUpperCase());
    }
}
```

---

## 📤 @ResponseBody & Response Formatting

### Different Response Formats:
```java
@RestController
@RequestMapping("/api/reports")
public class ReportsController {
    
    private final ReportsService reportsService;
    
    public ReportsController(ReportsService reportsService) {
        this.reportsService = reportsService;
    }
    
    // JSON response (default)
    @GetMapping("/sales")
    public ResponseEntity<SalesReport> getSalesReport(
            @RequestParam String startDate,
            @RequestParam String endDate
    ) {
        System.out.printf("📊 Generating sales report: %s to %s%n", startDate, endDate);
        
        try {
            LocalDate start = LocalDate.parse(startDate);
            LocalDate end = LocalDate.parse(endDate);
            
            SalesReport report = reportsService.generateSalesReport(start, end);
            
            return ResponseEntity.ok(report);
            
        } catch (DateTimeParseException e) {
            System.out.println("❌ Invalid date format");
            return ResponseEntity.badRequest().build();
        }
    }
    
    // CSV response
    @GetMapping(value = "/sales/csv", produces = "text/csv")
    public ResponseEntity<String> getSalesReportCsv(
            @RequestParam String startDate,
            @RequestParam String endDate
    ) {
        System.out.println("📄 Generating CSV sales report");
        
        try {
            LocalDate start = LocalDate.parse(startDate);
            LocalDate end = LocalDate.parse(endDate);
            
            String csvContent = reportsService.generateSalesReportCsv(start, end);
            
            HttpHeaders headers = new HttpHeaders();
            headers.add("Content-Disposition", 
                       "attachment; filename=\"sales-report-" + startDate + "-to-" + endDate + ".csv\"");
            
            return ResponseEntity.ok()
                    .headers(headers)
                    .body(csvContent);
                    
        } catch (DateTimeParseException e) {
            return ResponseEntity.badRequest().body("Invalid date format");
        }
    }
    
    // XML response
    @GetMapping(value = "/inventory", produces = "application/xml")
    public ResponseEntity<InventoryReport> getInventoryReportXml() {
        System.out.println("📋 Generating XML inventory report");
        
        InventoryReport report = reportsService.generateInventoryReport();
        
        HttpHeaders headers = new HttpHeaders();
        headers.add("Content-Type", "application/xml; charset=UTF-8");
        
        return ResponseEntity.ok()
                .headers(headers)
                .body(report);
    }
    
    // Content negotiation - multiple formats
    @GetMapping(value = "/dashboard", produces = {"application/json", "application/xml"})
    public ResponseEntity<DashboardData> getDashboardData() {
        System.out.println("🎛️ Generating dashboard data");
        
        DashboardData dashboard = reportsService.generateDashboardData();
        
        return ResponseEntity.ok(dashboard);
    }
    
    // Custom response with headers
    @GetMapping("/analytics")
    public ResponseEntity<AnalyticsReport> getAnalyticsReport(
            @RequestParam(defaultValue = "30") int days
    ) {
        System.out.println("📈 Generating analytics report for " + days + " days");
        
        AnalyticsReport report = reportsService.generateAnalyticsReport(days);
        
        HttpHeaders headers = new HttpHeaders();
        headers.add("X-Report-Generated", LocalDateTime.now().toString());
        headers.add("X-Data-Points", String.valueOf(report.getDataPoints().size()));
        headers.add("Cache-Control", "private, max-age=300"); // 5 minutes cache
        
        return ResponseEntity.ok()
                .headers(headers)
                .body(report);
    }
    
    // Binary response (file download)
    @GetMapping("/export/{reportId}")
    public ResponseEntity<byte[]> downloadReport(@PathVariable String reportId) {
        System.out.println("📥 Downloading report: " + reportId);
        
        try {
            byte[] reportData = reportsService.getReportData(reportId);
            String filename = "report-" + reportId + ".pdf";
            
            HttpHeaders headers = new HttpHeaders();
            headers.add("Content-Disposition", "attachment; filename=\"" + filename + "\"");
            headers.add("Content-Type", "application/pdf");
            
            return ResponseEntity.ok()
                    .headers(headers)
                    .body(reportData);
                    
        } catch (ReportNotFoundException e) {
            System.out.println("❌ Report not found: " + reportId);
            return ResponseEntity.notFound().build();
        }
    }
}
```

---

## 📁 File Upload & Download Handling

### Complete File Handling Example:
```java
@RestController
@RequestMapping("/api/files")
public class FileController {
    
    private final FileService fileService;
    
    @Value("${file.upload.max-size:10MB}")
    private String maxFileSize;
    
    @Value("${file.upload.allowed-types}")
    private List<String> allowedFileTypes;
    
    public FileController(FileService fileService) {
        this.fileService = fileService;
    }
    
    // Single file upload
    @PostMapping("/upload")
    public ResponseEntity<Map<String, Object>> uploadFile(
            @RequestParam("file") MultipartFile file,
            @RequestParam(required = false) String description,
            @RequestParam(defaultValue = "false") boolean isPublic
    ) {
        System.out.println("📤 File upload request:");
        System.out.println("  Filename: " + file.getOriginalFilename());
        System.out.println("  Size: " + formatFileSize(file.getSize()));
        System.out.println("  Content Type: " + file.getContentType());
        System.out.println("  Description: " + description);
        System.out.println("  Public: " + isPublic);
        
        try {
            // Validate file
            validateFile(file);
            
            // Save file
            FileUploadResult result = fileService.saveFile(file, description, isPublic);
            
            Map<String, Object> response = new HashMap<>();
            response.put("success", true);
            response.put("fileId", result.getFileId());
            response.put("filename", result.getFilename());
            response.put("size", file.getSize());
            response.put("url", "/api/files/" + result.getFileId());
            response.put("downloadUrl", "/api/files/" + result.getFileId() + "/download");
            
            System.out.println("✅ File uploaded successfully with ID: " + result.getFileId());
            
            return ResponseEntity.status(HttpStatus.CREATED).body(response);
            
        } catch (FileValidationException e) {
            System.out.println("❌ File validation failed: " + e.getMessage());
            Map<String, Object> errorResponse = new HashMap<>();
            errorResponse.put("success", false);
            errorResponse.put("error", e.getMessage());
            return ResponseEntity.badRequest().body(errorResponse);
            
        } catch (IOException e) {
            System.out.println("💥 File save failed: " + e.getMessage());
            Map<String, Object> errorResponse = new HashMap<>();
            errorResponse.put("success", false);
            errorResponse.put("error", "File save failed");
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(errorResponse);
        }
    }
    
    // Multiple files upload
    @PostMapping("/upload-multiple")
    public ResponseEntity<Map<String, Object>> uploadMultipleFiles(
            @RequestParam("files") MultipartFile[] files,
            @RequestParam(required = false) String description
    ) {
        System.out.println("📤 Multiple files upload: " + files.length + " files");
        
        List<Map<String, Object>> uploadedFiles = new ArrayList<>();
        List<Map<String, Object>> failedFiles = new ArrayList<>();
        
        for (MultipartFile file : files) {
            try {
                System.out.println("  Processing: " + file.getOriginalFilename());
                validateFile(file);
                
                FileUploadResult result = fileService.saveFile(file, description, false);
                
                Map<String, Object> fileInfo = new HashMap<>();
                fileInfo.put("fileId", result.getFileId());
                fileInfo.put("filename", result.getFilename());
                fileInfo.put("size", file.getSize());
                fileInfo.put("url", "/api/files/" + result.getFileId());
                
                uploadedFiles.add(fileInfo);
                System.out.println("    ✅ Uploaded successfully");
                
            } catch (Exception e) {
                Map<String, Object> errorInfo = new HashMap<>();
                errorInfo.put("filename", file.getOriginalFilename());
                errorInfo.put("error", e.getMessage());
                
                failedFiles.add(errorInfo);
                System.out.println("    ❌ Upload failed: " + e.getMessage());
            }
        }
        
        Map<String, Object> response = new HashMap<>();
        response.put("uploadedFiles", uploadedFiles);
        response.put("failedFiles", failedFiles);
        response.put("summary", String.format("Uploaded: %d, Failed: %d", 
                                            uploadedFiles.size(), failedFiles.size()));
        
        System.out.printf("📊 Upload summary: %d successful, %d failed%n", 
                         uploadedFiles.size(), failedFiles.size());
        
        if (uploadedFiles.isEmpty()) {
            return ResponseEntity.badRequest().body(response);
        } else {
            return ResponseEntity.status(HttpStatus.CREATED).body(response);
        }
    }
    
    // File download
    @GetMapping("/{fileId}/download")
    public ResponseEntity<byte[]> downloadFile(@PathVariable String fileId) {
        System.out.println("📥 File download request for ID: " + fileId);
        
        try {
            FileDownloadResult result = fileService.getFile(fileId);
            
            HttpHeaders headers = new HttpHeaders();
            headers.add("Content-Disposition", 
                       "attachment; filename=\"" + result.getFilename() + "\"");
            headers.add("Content-Type", result.getContentType());
            headers.add("Content-Length", String.valueOf(result.getData().length));
            
            System.out.printf("✅ Serving file: %s (%s)%n", 
                            result.getFilename(), formatFileSize(result.getData().length));
            
            return ResponseEntity.ok()
                    .headers(headers)
                    .body(result.getData());
                    
        } catch (FileNotFoundException e) {
            System.out.println("❌ File not found: " + fileId);
            return ResponseEntity.notFound().build();
        } catch (Exception e) {
            System.out.println("💥 Download error: " + e.getMessage());
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }
    
    // File metadata
    @GetMapping("/{fileId}")
    public ResponseEntity<FileMetadata> getFileMetadata(@PathVariable String fileId) {
        System.out.println("ℹ️ Getting metadata for file: " + fileId);
        
        try {
            FileMetadata metadata = fileService.getFileMetadata(fileId);
            return ResponseEntity.ok(metadata);
            
        } catch (FileNotFoundException e) {
            return ResponseEntity.notFound().build();
        }
    }
    
    // List user files
    @GetMapping("/user/{userId}")
    public ResponseEntity<List<FileMetadata>> getUserFiles(
            @PathVariable Long userId,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size
    ) {
        System.out.printf("📋 Getting files for user %d (page %d, size %d)%n", userId, page, size);
        
        List<FileMetadata> files = fileService.getUserFiles(userId, page, size);
        
        return ResponseEntity.ok(files);
    }
    
    // Image upload with thumbnail generation
    @PostMapping("/upload-image")
    public ResponseEntity<Map<String, Object>> uploadImage(
            @RequestParam("image") MultipartFile image,
            @RequestParam(required = false) String alt,
            @RequestParam(defaultValue = "false") boolean generateThumbnail
    ) {
        System.out.println("🖼️ Image upload request:");
        System.out.println("  Filename: " + image.getOriginalFilename());
        System.out.println("  Generate thumbnail: " + generateThumbnail);
        
        try {
            // Validate image file
            if (!isImageFile(image)) {
                throw new FileValidationException("File must be an image");
            }
            
            validateFile(image);
            
            ImageUploadResult result = fileService.saveImage(image, alt, generateThumbnail);
            
            Map<String, Object> response = new HashMap<>();
            response.put("success", true);
            response.put("imageId", result.getImageId());
            response.put("filename", result.getFilename());
            response.put("url", "/api/files/" + result.getImageId());
            response.put("thumbnailUrl", result.getThumbnailUrl());
            response.put("dimensions", result.getDimensions());
            
            System.out.printf("✅ Image uploaded: %dx%d pixels%n", 
                            result.getDimensions().getWidth(), result.getDimensions().getHeight());
            
            return ResponseEntity.status(HttpStatus.CREATED).body(response);
            
        } catch (Exception e) {
            System.out.println("❌ Image upload failed: " + e.getMessage());
            Map<String, Object> errorResponse = new HashMap<>();
            errorResponse.put("success", false);
            errorResponse.put("error", e.getMessage());
            return ResponseEntity.badRequest().body(errorResponse);
        }
    }
    
    // Delete file
    @DeleteMapping("/{fileId}")
    public ResponseEntity<String> deleteFile(@PathVariable String fileId) {
        System.out.println("🗑️ Deleting file: " + fileId);
        
        try {
            boolean deleted = fileService.deleteFile(fileId);
            
            if (deleted) {
                System.out.println("✅ File deleted successfully");
                return ResponseEntity.ok("File deleted successfully");
            } else {
                return ResponseEntity.notFound().build();
            }
            
        } catch (Exception e) {
            System.out.println("💥 Delete error: " + e.getMessage());
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body("Error deleting file");
        }
    }
    
    // Helper methods
    private void validateFile(MultipartFile file) {
        if (file.isEmpty()) {
            throw new FileValidationException("File cannot be empty");
        }
        
        if (file.getSize() > parseSize(maxFileSize)) {
            throw new FileValidationException("File size exceeds maximum allowed size of " + maxFileSize);
        }
        
        String contentType = file.getContentType();
        if (contentType != null && !allowedFileTypes.contains(contentType)) {
            throw new FileValidationException("File type not allowed: " + contentType);
        }
        
         String filename = file.getOriginalFilename();
        if (filename != null && (filename.contains("..") || filename.contains("/"))) {
            throw new FileValidationException("Invalid filename: " + filename);
        }
    }

    private long parseSize(String size) {
        // Parse human-readable sizes (e.g., "10MB", "500KB") into bytes
        size = size.toUpperCase().trim();
        if (size.endsWith("KB")) {
            return Long.parseLong(size.replace("KB", "").trim()) * 1024;
        } else if (size.endsWith("MB")) {
            return Long.parseLong(size.replace("MB", "").trim()) * 1024 * 1024;
        } else if (size.endsWith("GB")) {
            return Long.parseLong(size.replace("GB", "").trim()) * 1024 * 1024 * 1024;
        } else {
            return Long.parseLong(size);
        }
    }

    private boolean isImageFile(MultipartFile file) {
        String contentType = file.getContentType();
        return contentType != null && contentType.startsWith("image/");
    }
}
```
---

## 🎯 Interview Questions & Answers


**Q1. What is the difference between `@RequestParam` and `@PathVariable`?**  
- `@RequestParam` → Used for **query parameters** after `?` in the URL.  
  Example: `/products?category=books`  
- `@PathVariable` → Used for **values embedded in the URL path**.  
  Example: `/products/101`  

---

**Q2. What happens if `@RequestBody` is missing in a POST method?**  
- Spring won’t deserialize the JSON payload into the object.  
- The object fields will remain **`null`**.  

---

**Q3. Is `@ResponseBody` required in a `@RestController`?**  
- No ✅.  
- `@RestController` already includes `@ResponseBody` by default.  

---

**Q4. How to support multiple response formats (JSON, XML, CSV)?**  
- Use the **`produces` attribute** in `@GetMapping` / `@RequestMapping`.  
  ```java
  @GetMapping(value = "/data", produces = {"application/json", "application/xml"})
