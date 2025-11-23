# Dependency Inversion Principle (DIP)

## Definition

**High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.**

In simpler terms: Depend on interfaces or abstract classes rather than concrete classes.

## Why is it Important?

1. **Loose Coupling**: Reduces dependencies between modules
2. **Flexibility**: Easy to change implementations
3. **Testability**: Easy to mock dependencies for testing
4. **Maintainability**: Changes in low-level modules don't affect high-level logic
5. **Reusability**: Components can be reused in different contexts

## Bad Example (Violating DIP)

```java
// Low-level module - concrete implementation
public class MySQLDatabase {
    public void connect() {
        System.out.println("Connecting to MySQL database");
    }
    
    public void saveData(String data) {
        System.out.println("Saving to MySQL: " + data);
    }
    
    public String getData(String id) {
        return "Data from MySQL for " + id;
    }
}

// High-level module directly depends on low-level concrete class
public class UserService {
    private MySQLDatabase database; // Tight coupling to MySQL!
    
    public UserService() {
        this.database = new MySQLDatabase(); // Creates concrete instance
    }
    
    public void saveUser(String userData) {
        database.connect();
        database.saveData(userData);
    }
    
    public String getUser(String userId) {
        database.connect();
        return database.getData(userId);
    }
}

// What if we want to switch to MongoDB or PostgreSQL?
// We need to modify UserService class!
```

**Problems with this approach:**
- `UserService` is tightly coupled to `MySQLDatabase`
- Cannot easily switch to a different database
- Difficult to test `UserService` in isolation
- Changes to `MySQLDatabase` may break `UserService`
- Cannot reuse `UserService` with different databases

## Good Example (Following DIP)

```java
// Abstraction - interface that both high and low-level modules depend on
public interface Database {
    void connect();
    void saveData(String data);
    String getData(String id);
}

// Low-level module - MySQL implementation
public class MySQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to MySQL database");
    }
    
    @Override
    public void saveData(String data) {
        System.out.println("Saving to MySQL: " + data);
    }
    
    @Override
    public String getData(String id) {
        return "Data from MySQL for " + id;
    }
}

// Low-level module - MongoDB implementation
public class MongoDBDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to MongoDB database");
    }
    
    @Override
    public void saveData(String data) {
        System.out.println("Saving to MongoDB: " + data);
    }
    
    @Override
    public String getData(String id) {
        return "Data from MongoDB for " + id;
    }
}

// Low-level module - PostgreSQL implementation
public class PostgreSQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to PostgreSQL database");
    }
    
    @Override
    public void saveData(String data) {
        System.out.println("Saving to PostgreSQL: " + data);
    }
    
    @Override
    public String getData(String id) {
        return "Data from PostgreSQL for " + id;
    }
}

// High-level module depends on abstraction (Database interface)
public class UserService {
    private Database database; // Depends on abstraction!
    
    // Dependency injection through constructor
    public UserService(Database database) {
        this.database = database;
    }
    
    public void saveUser(String userData) {
        database.connect();
        database.saveData(userData);
    }
    
    public String getUser(String userId) {
        database.connect();
        return database.getData(userId);
    }
}

// Usage - dependency is injected from outside
public class Application {
    public static void main(String[] args) {
        // Easy to switch implementations!
        Database mysqlDb = new MySQLDatabase();
        UserService userService1 = new UserService(mysqlDb);
        userService1.saveUser("John Doe");
        
        // Switch to MongoDB without changing UserService
        Database mongoDb = new MongoDBDatabase();
        UserService userService2 = new UserService(mongoDb);
        userService2.saveUser("Jane Smith");
        
        // Switch to PostgreSQL
        Database postgresDb = new PostgreSQLDatabase();
        UserService userService3 = new UserService(postgresDb);
        userService3.saveUser("Bob Johnson");
    }
}

// Easy to test with mock implementation
public class MockDatabase implements Database {
    private Map<String, String> storage = new HashMap<>();
    
    @Override
    public void connect() {
        System.out.println("Mock connection");
    }
    
    @Override
    public void saveData(String data) {
        storage.put("test", data);
    }
    
    @Override
    public String getData(String id) {
        return storage.getOrDefault(id, "default");
    }
}

// Unit test
public class UserServiceTest {
    @Test
    public void testSaveUser() {
        Database mockDb = new MockDatabase();
        UserService service = new UserService(mockDb);
        service.saveUser("Test User");
        // Easy to test without real database!
    }
}
```

**Benefits of this approach:**
- `UserService` doesn't depend on concrete implementations
- Easy to switch database implementations
- Simple to test with mock objects
- Changes to database implementations don't affect `UserService`
- New database types can be added without modifying existing code

## Another Example: Notification System

### Bad Example (Violating DIP)

```java
// High-level module tightly coupled to low-level modules
public class NotificationService {
    private EmailSender emailSender;
    private SMSSender smsSender;
    
    public NotificationService() {
        this.emailSender = new EmailSender(); // Concrete dependency
        this.smsSender = new SMSSender();     // Concrete dependency
    }
    
    public void notifyByEmail(String message) {
        emailSender.sendEmail(message);
    }
    
    public void notifyBySMS(String message) {
        smsSender.sendSMS(message);
    }
    
    // What if we want to add push notifications? Need to modify this class!
}

class EmailSender {
    public void sendEmail(String message) {
        System.out.println("Sending email: " + message);
    }
}

class SMSSender {
    public void sendSMS(String message) {
        System.out.println("Sending SMS: " + message);
    }
}
```

### Good Example (Following DIP)

```java
// Abstraction
public interface MessageSender {
    void send(String recipient, String message);
}

// Low-level modules implement the abstraction
public class EmailSender implements MessageSender {
    @Override
    public void send(String recipient, String message) {
        System.out.println("Sending email to " + recipient + ": " + message);
        // Email-specific implementation
    }
}

public class SMSSender implements MessageSender {
    @Override
    public void send(String recipient, String message) {
        System.out.println("Sending SMS to " + recipient + ": " + message);
        // SMS-specific implementation
    }
}

public class PushNotificationSender implements MessageSender {
    @Override
    public void send(String recipient, String message) {
        System.out.println("Sending push notification to " + recipient + ": " + message);
        // Push notification-specific implementation
    }
}

// High-level module depends on abstraction
public class NotificationService {
    private List<MessageSender> senders;
    
    public NotificationService(List<MessageSender> senders) {
        this.senders = senders;
    }
    
    public void notifyAll(String recipient, String message) {
        for (MessageSender sender : senders) {
            sender.send(recipient, message);
        }
    }
    
    public void notify(MessageSender sender, String recipient, String message) {
        sender.send(recipient, message);
    }
}

// Usage with dependency injection
public class Application {
    public static void main(String[] args) {
        // Configure dependencies
        List<MessageSender> senders = Arrays.asList(
            new EmailSender(),
            new SMSSender(),
            new PushNotificationSender()
        );
        
        NotificationService service = new NotificationService(senders);
        service.notifyAll("user@example.com", "Hello!");
        
        // Or use specific sender
        service.notify(new EmailSender(), "user@example.com", "Email only");
    }
}
```

## Dependency Injection (DI)

DI is a technique to implement DIP. There are three common types:

### 1. Constructor Injection (Recommended)

```java
public class OrderService {
    private final PaymentProcessor paymentProcessor;
    private final InventoryManager inventoryManager;
    
    // Dependencies injected through constructor
    public OrderService(PaymentProcessor paymentProcessor, 
                       InventoryManager inventoryManager) {
        this.paymentProcessor = paymentProcessor;
        this.inventoryManager = inventoryManager;
    }
}
```

**Advantages:**
- Dependencies are required and immutable
- Easy to test
- Clear what dependencies are needed

### 2. Setter Injection

```java
public class OrderService {
    private PaymentProcessor paymentProcessor;
    private InventoryManager inventoryManager;
    
    // Dependencies injected through setters
    public void setPaymentProcessor(PaymentProcessor processor) {
        this.paymentProcessor = processor;
    }
    
    public void setInventoryManager(InventoryManager manager) {
        this.inventoryManager = manager;
    }
}
```

**Advantages:**
- Optional dependencies
- Can change dependencies at runtime

**Disadvantages:**
- Object can be in invalid state if setters not called
- Not immutable

### 3. Interface Injection

```java
public interface PaymentProcessorAware {
    void setPaymentProcessor(PaymentProcessor processor);
}

public class OrderService implements PaymentProcessorAware {
    private PaymentProcessor paymentProcessor;
    
    @Override
    public void setPaymentProcessor(PaymentProcessor processor) {
        this.paymentProcessor = processor;
    }
}
```

## Inversion of Control (IoC)

DIP is related to Inversion of Control:
- **Traditional approach**: High-level module controls low-level modules
- **IoC**: Framework or container controls object creation and injection

```java
// IoC Container (simplified example)
public class IoCContainer {
    private Map<Class<?>, Object> services = new HashMap<>();
    
    public <T> void register(Class<T> interfaceType, T implementation) {
        services.put(interfaceType, implementation);
    }
    
    @SuppressWarnings("unchecked")
    public <T> T resolve(Class<T> interfaceType) {
        return (T) services.get(interfaceType);
    }
}

// Usage
IoCContainer container = new IoCContainer();
container.register(Database.class, new MySQLDatabase());
container.register(MessageSender.class, new EmailSender());

Database db = container.resolve(Database.class);
MessageSender sender = container.resolve(MessageSender.class);
```

## Real-World Analogy

Think of a power plug:
- **Bad Design**: TV hard-wired to wall (tight coupling)
  - Can't move TV easily
  - Can't test TV without wall power
- **Good Design**: TV plugs into any standard outlet (abstraction)
  - Can use different power sources
  - Can test with different adapters
  - TV doesn't care about power generation details

## Key Takeaways

1. ✅ **Depend on abstractions**, not concretions
2. ✅ **Use interfaces or abstract classes**
3. ✅ **Inject dependencies** from outside
4. ✅ **High-level modules define interfaces** they need
5. ✅ **Low-level modules implement** those interfaces
6. ✅ **Use dependency injection** (constructor, setter, or interface)

## Common Violations

❌ Creating objects with `new` inside classes
❌ Depending on concrete classes instead of interfaces
❌ Static method calls to concrete classes
❌ Tight coupling between layers
❌ Using service locator anti-pattern

## Benefits

✅ **Loose coupling**: Easy to change implementations
✅ **Testability**: Mock dependencies for unit tests
✅ **Flexibility**: Swap implementations at runtime
✅ **Maintainability**: Changes are localized
✅ **Reusability**: Components work with any compatible implementation

## When to Apply

- When designing system architecture
- When you need to swap implementations
- When unit testing with mocks
- When building plugin systems
- When following layered architecture

## DIP vs DI

- **DIP** (Dependency Inversion Principle): A design principle
- **DI** (Dependency Injection): A technique to implement DIP
- **IoC** (Inversion of Control): A broader concept that includes DI

## Related Patterns

- **Factory Pattern**: Creates objects implementing interfaces
- **Strategy Pattern**: Defines family of algorithms behind interface
- **Adapter Pattern**: Adapts concrete classes to expected interfaces
- **Repository Pattern**: Abstracts data access

---

[← Previous: Interface Segregation Principle](./interface-segregation.md) | [Back to SOLID Principles](../../README.md#solid-principles)
