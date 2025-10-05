# 📅 Day 18: Transaction Management

## 🎯 Today Topics
- Master `@Transactional` annotation
- Understand transaction propagation levels
- Learn isolation levels and rollback strategies
- Handle data consistency with transactions

---

## 💡 What is a Transaction?

### Definition:
**Transaction** = A unit of work that either completes fully or fails completely (all-or-nothing).

### ACID Properties:
- **Atomicity**: All operations succeed or all fail
- **Consistency**: Data remains valid before and after
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Completed transactions are permanent

### Example:
```
Bank Transfer (Atomic Operation):
1. Deduct $100 from Account A
2. Add $100 to Account B
→ Both must succeed or both must fail!
```

---

## 🎯 @Transactional Basics

### Simple Usage:
```java
@Service
public class BankService {
    
    @Autowired
    private AccountRepository accountRepository;
    
    // Method-level transaction
    @Transactional
    public void transferMoney(Long fromAccountId, Long toAccountId, BigDecimal amount) {
        
        // Find accounts
        Account fromAccount = accountRepository.findById(fromAccountId)
                .orElseThrow(() -> new AccountNotFoundException(fromAccountId));
        
        Account toAccount = accountRepository.findById(toAccountId)
                .orElseThrow(() -> new AccountNotFoundException(toAccountId));
        
        // Check balance
        if (fromAccount.getBalance().compareTo(amount) < 0) {
            throw new InsufficientFundsException("Insufficient balance");
        }
        
        // Deduct from source
        fromAccount.setBalance(fromAccount.getBalance().subtract(amount));
        accountRepository.save(fromAccount);
        
        // Add to destination
        toAccount.setBalance(toAccount.getBalance().add(amount));
        accountRepository.save(toAccount);
        
        System.out.printf("✅ Transferred $%.2f from Account %d to Account %d%n", 
                         amount, fromAccountId, toAccountId);
        
        // If any exception occurs, entire transaction rolls back
    }
}
```

### Class-Level Transaction:
```java
// All public methods become transactional
@Service
@Transactional
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private InventoryService inventoryService;
    
    @Autowired
    private PaymentService paymentService;
    
    // This method is transactional
    public Order createOrder(OrderRequest request) {
        // Create order
        Order order = new Order();
        order.setCustomerId(request.getCustomerId());
        order.setTotalAmount(request.getTotalAmount());
        
        // Reserve inventory
        inventoryService.reserveItems(request.getItems());
        
        // Process payment
        paymentService.processPayment(request.getPaymentDetails());
        
        // Save order
        return orderRepository.save(order);
        
        // All operations succeed together or roll back together
    }
    
    // Read-only transaction (performance optimization)
    @Transactional(readOnly = true)
    public List<Order> findAllOrders() {
        return orderRepository.findAll();
    }
}
```

---

## 🔄 Transaction Propagation

### Propagation Types:

#### 1. REQUIRED (Default):
```java
// Use existing transaction or create new one
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() {
    // Transaction starts here if not exists
}

@Transactional(propagation = Propagation.REQUIRED)
public void methodB() {
    // Joins methodA's transaction if called from methodA
}
```

#### 2. REQUIRES_NEW:
```java
// Always create new transaction
@Service
public class OrderService {
    
    @Transactional
    public void processOrder(Order order) {
        // Transaction 1 starts
        orderRepository.save(order);
        
        // Call audit method
        auditService.logOrder(order);
        
        // If processOrder fails, audit is still saved!
    }
}

@Service
public class AuditService {
    
    // Creates separate transaction
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logOrder(Order order) {
        // Transaction 2 (independent of Transaction 1)
        AuditLog log = new AuditLog();
        log.setOrderId(order.getId());
        log.setAction("ORDER_CREATED");
        log.setTimestamp(LocalDateTime.now());
        
        auditRepository.save(log);
        // This saves even if parent transaction fails
    }
}
```

#### 3. NESTED:
```java
@Transactional
public void parentMethod() {
    // Parent transaction
    
    nestedMethod(); // Can roll back independently
}

@Transactional(propagation = Propagation.NESTED)
public void nestedMethod() {
    // Nested transaction (savepoint)
    // Can roll back without affecting parent
}
```

---

## 🔒 Transaction Isolation Levels

### Isolation Types:

#### 1. READ_UNCOMMITTED (Lowest):
```java
// Can read uncommitted data from other transactions
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
public BigDecimal getAccountBalance(Long accountId) {
    // May read dirty data (not recommended)
    return accountRepository.findById(accountId).getBalance();
}
```

#### 2. READ_COMMITTED (Common):
```java
// Only reads committed data
@Transactional(isolation = Isolation.READ_COMMITTED)
public BigDecimal getAccountBalance(Long accountId) {
    // Safe from dirty reads
    return accountRepository.findById(accountId).getBalance();
}
```

#### 3. REPEATABLE_READ:
```java
// Same data is returned within transaction
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void processWithConsistentData(Long accountId) {
    Account account1 = accountRepository.findById(accountId);
    // ... do some work ...
    Account account2 = accountRepository.findById(accountId);
    // account1.balance == account2.balance (guaranteed)
}
```

#### 4. SERIALIZABLE (Highest):
```java
// Complete isolation - transactions run serially
@Transactional(isolation = Isolation.SERIALIZABLE)
public void criticalOperation(Long accountId) {
    // No concurrent access - safest but slowest
}
```

---

## ⚠️ Rollback Strategies

### Default Rollback (RuntimeException):
```java
@Transactional
public void createUser(User user) {
    userRepository.save(user);
    
    // This rolls back transaction
    throw new RuntimeException("Error!");
    
    // User is NOT saved
}
```

### Custom Rollback Rules:
```java
@Service
public class UserService {
    
    // Rollback on any exception
    @Transactional(rollbackFor = Exception.class)
    public void createUser(User user) throws Exception {
        userRepository.save(user);
        
        // Checked exceptions also roll back now
        throw new Exception("Error!");
    }
    
    // Don't rollback on specific exception
    @Transactional(noRollbackFor = ValidationException.class)
    public void updateUser(User user) {
        userRepository.save(user);
        
        // This won't roll back the transaction
        throw new ValidationException("Invalid data");
    }
    
    // Multiple exceptions
    @Transactional(
        rollbackFor = {SQLException.class, IOException.class},
        noRollbackFor = {ValidationException.class}
    )
    public void complexOperation() {
        // Custom rollback rules
    }
}
```

---

## 🎯 Complete Example: E-Commerce Order Processing

### Order Service with Transactions:
```java
@Service
@Transactional
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private InventoryService inventoryService;
    
    @Autowired
    private PaymentService paymentService;
    
    @Autowired
    private NotificationService notificationService;
    
    // Main transactional method
    @Transactional(
        isolation = Isolation.READ_COMMITTED,
        propagation = Propagation.REQUIRED,
        rollbackFor = Exception.class,
        timeout = 30
    )
    public Order processOrder(OrderRequest request) {
        
        System.out.println("🛒 Starting order transaction...");
        
        // Step 1: Validate and reserve inventory
        boolean reserved = inventoryService.reserveItems(request.getItems());
        if (!reserved) {
            throw new InsufficientStockException("Items not available");
        }
        System.out.println("✅ Inventory reserved");
        
        // Step 2: Process payment
        PaymentResult payment = paymentService.processPayment(
            request.getCustomerId(), 
            request.getTotalAmount()
        );
        if (!payment.isSuccessful()) {
            throw new PaymentFailedException("Payment failed");
        }
        System.out.println("✅ Payment processed");
        
        // Step 3: Create order
        Order order = new Order();
        order.setCustomerId(request.getCustomerId());
        order.setItems(request.getItems());
        order.setTotalAmount(request.getTotalAmount());
        order.setPaymentId(payment.getTransactionId());
        order.setStatus("CONFIRMED");
        order.setCreatedAt(LocalDateTime.now());
        
        Order savedOrder = orderRepository.save(order);
        System.out.println("✅ Order created: " + savedOrder.getId());
        
        // Step 4: Send notification (independent transaction)
        notificationService.sendOrderConfirmation(savedOrder);
        
        System.out.println("✅ Order transaction completed successfully");
        return savedOrder;
    }
    
    // Read-only transaction (optimization)
    @Transactional(readOnly = true)
    public Order getOrderById(Long orderId) {
        return orderRepository.findById(orderId)
                .orElseThrow(() -> new OrderNotFoundException(orderId));
    }
    
    @Transactional(readOnly = true)
    public List<Order> getCustomerOrders(Long customerId) {
        return orderRepository.findByCustomerId(customerId);
    }
    
    // Cancel order with rollback
    @Transactional(rollbackFor = Exception.class)
    public void cancelOrder(Long orderId) {
        
        System.out.println("❌ Canceling order: " + orderId);
        
        Order order = getOrderById(orderId);
        
        // Restore inventory
        inventoryService.restoreItems(order.getItems());
        
        // Refund payment
        paymentService.refundPayment(order.getPaymentId());
        
        // Update order status
        order.setStatus("CANCELLED");
        order.setCancelledAt(LocalDateTime.now());
        orderRepository.save(order);
        
        System.out.println("✅ Order cancelled successfully");
    }
}

// Inventory Service
@Service
@Transactional
public class InventoryService {
    
    @Autowired
    private ProductRepository productRepository;
    
    public boolean reserveItems(List<OrderItem> items) {
        
        for (OrderItem item : items) {
            Product product = productRepository.findById(item.getProductId())
                    .orElseThrow(() -> new ProductNotFoundException(item.getProductId()));
            
            if (product.getStock() < item.getQuantity()) {
                return false; // Not enough stock
            }
            
            // Reduce stock
            product.setStock(product.getStock() - item.getQuantity());
            productRepository.save(product);
            
            System.out.printf("📦 Reserved %d units of product %d%n", 
                            item.getQuantity(), item.getProductId());
        }
        
        return true;
    }
    
    public void restoreItems(List<OrderItem> items) {
        for (OrderItem item : items) {
            Product product = productRepository.findById(item.getProductId())
                    .orElseThrow(() -> new ProductNotFoundException(item.getProductId()));
            
            // Restore stock
            product.setStock(product.getStock() + item.getQuantity());
            productRepository.save(product);
            
            System.out.printf("📦 Restored %d units of product %d%n", 
                            item.getQuantity(), item.getProductId());
        }
    }
}

// Payment Service
@Service
@Transactional
public class PaymentService {
    
    @Autowired
    private PaymentRepository paymentRepository;
    
    public PaymentResult processPayment(Long customerId, BigDecimal amount) {
        
        System.out.printf("💳 Processing payment: $%.2f for customer %d%n", amount, customerId);
        
        // Create payment record
        Payment payment = new Payment();
        payment.setCustomerId(customerId);
        payment.setAmount(amount);
        payment.setStatus("PROCESSING");
        payment.setCreatedAt(LocalDateTime.now());
        
        Payment savedPayment = paymentRepository.save(payment);
        
        // Simulate payment processing
        try {
            Thread.sleep(100); // Simulate API call
            
            // 90% success rate for demo
            if (Math.random() < 0.9) {
                savedPayment.setStatus("SUCCESS");
                savedPayment.setTransactionId("TXN-" + System.currentTimeMillis());
                paymentRepository.save(savedPayment);
                
                System.out.println("✅ Payment successful: " + savedPayment.getTransactionId());
                return new PaymentResult(true, savedPayment.getTransactionId());
            } else {
                savedPayment.setStatus("FAILED");
                paymentRepository.save(savedPayment);
                
                System.out.println("❌ Payment failed");
                return new PaymentResult(false, null);
            }
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return new PaymentResult(false, null);
        }
    }
    
    public void refundPayment(String transactionId) {
        System.out.println("💰 Refunding payment: " + transactionId);
        
        Payment payment = paymentRepository.findByTransactionId(transactionId)
                .orElseThrow(() -> new PaymentNotFoundException(transactionId));
        
        payment.setStatus("REFUNDED");
        payment.setRefundedAt(LocalDateTime.now());
        paymentRepository.save(payment);
        
        System.out.println("✅ Refund completed");
    }
}

// Notification Service (Independent Transaction)
@Service
public class NotificationService {
    
    @Autowired
    private NotificationRepository notificationRepository;
    
    // Separate transaction - saves even if parent fails
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void sendOrderConfirmation(Order order) {
        
        System.out.println("📧 Sending order confirmation to customer: " + order.getCustomerId());
        
        Notification notification = new Notification();
        notification.setCustomerId(order.getCustomerId());
        notification.setOrderId(order.getId());
        notification.setType("ORDER_CONFIRMATION");
        notification.setMessage("Your order has been confirmed!");
        notification.setSentAt(LocalDateTime.now());
        
        notificationRepository.save(notification);
        
        System.out.println("✅ Notification sent");
    }
}
```

---

## 📊 Transaction Propagation Quick Reference

| Propagation | Description | Use Case |
|-------------|-------------|----------|
| **REQUIRED** | Use existing or create new (default) | Most common scenarios |
| **REQUIRES_NEW** | Always create new transaction | Audit logs, notifications |
| **NESTED** | Nested within parent | Savepoints, partial rollback |
| **MANDATORY** | Must have existing transaction | Called methods only |
| **SUPPORTS** | Use transaction if exists | Flexible methods |
| **NOT_SUPPORTED** | Execute without transaction | Read operations |
| **NEVER** | Throw exception if transaction exists | Non-transactional methods |

---

## 🎯 Interview Questions & Answers

### Q1: "What happens if you don't use @Transactional?"
**Answer:**
- Each database operation is **auto-committed** individually
- **No rollback** if something fails midway
- **Data inconsistency** possible (e.g., money deducted but not added)
- **Use @Transactional** for multi-step operations that must be atomic

### Q2: "What's the difference between REQUIRED and REQUIRES_NEW?"
**Answer:**
- **REQUIRED**: Joins existing transaction or creates new (most common)
- **REQUIRES_NEW**: Always creates new transaction, suspends parent
- **Use REQUIRES_NEW** for audit logs, notifications that should save even if parent fails
- **Example**: Logging should work even if main operation fails

### Q3: "When should you use readOnly = true?"
**Answer:**
- **For queries** that only read data, never modify
- **Performance benefit**: Database can optimize read-only operations
- **Prevents accidental writes**: Exception thrown if write attempted
- **Example**: `findAll()`, `findById()`, search methods

### Q4: "What exceptions cause rollback by default?"
**Answer:**
- **RuntimeException and its subclasses**: Auto rollback
- **Checked exceptions (Exception)**: NO rollback by default
- **To rollback on checked exceptions**: Use `rollbackFor = Exception.class`
- **To prevent rollback**: Use `noRollbackFor = SpecificException.class`

---

## 🚀 Best Practices

### ✅ Do's:
```java
// 1. Use @Transactional on service layer
@Service
@Transactional
public class UserService { }

// 2. Use readOnly for queries
@Transactional(readOnly = true)
public List<User> findAll() { }

// 3. Specify rollback rules for checked exceptions
@Transactional(rollbackFor = Exception.class)
public void criticalOperation() throws Exception { }

// 4. Use REQUIRES_NEW for independent operations
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void auditLog() { }

// 5. Set appropriate timeout
@Transactional(timeout = 30) // 30 seconds
public void longRunningOperation() { }
```

### ❌ Don'ts:
```java
// 1. Don't use @Transactional on private methods
@Transactional
private void helperMethod() { } // Won't work!

// 2. Don't catch and suppress exceptions
@Transactional
public void saveUser(User user) {
    try {
        userRepository.save(user);
    } catch (Exception e) {
        // Suppressed - transaction won't rollback!
    }
}

// 3. Don't use @Transactional on repository layer
@Repository
@Transactional // Redundant - already managed
public interface UserRepository extends JpaRepository<User, Long> { }

// 4. Don't forget timeout for long operations
@Transactional // May hang forever
public void veryLongOperation() { }
```

---
