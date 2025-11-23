# Code Organization Best Practices

## Overview

Good code organization makes your codebase maintainable, scalable, and easy to navigate. This guide covers best practices for structuring your code effectively.

## Package/Module Structure

### Layered Architecture

Organize code into logical layers:

```
src/
├── main/
│   ├── java/
│   │   └── com/company/project/
│   │       ├── controller/      # Presentation layer
│   │       ├── service/         # Business logic layer
│   │       ├── repository/      # Data access layer
│   │       ├── model/           # Domain models
│   │       ├── dto/             # Data transfer objects
│   │       ├── exception/       # Custom exceptions
│   │       ├── config/          # Configuration classes
│   │       └── util/            # Utility classes
│   └── resources/
│       ├── application.properties
│       └── static/
└── test/
    └── java/
        └── com/company/project/
```

### Feature-Based Structure

Organize by features/modules:

```
src/
├── user/
│   ├── UserController.java
│   ├── UserService.java
│   ├── UserRepository.java
│   └── User.java
├── order/
│   ├── OrderController.java
│   ├── OrderService.java
│   ├── OrderRepository.java
│   └── Order.java
└── product/
    ├── ProductController.java
    ├── ProductService.java
    ├── ProductRepository.java
    └── Product.java
```

### Hybrid Approach (Recommended)

Combine both approaches:

```
src/
└── com/company/project/
    ├── user/
    │   ├── controller/
    │   ├── service/
    │   ├── repository/
    │   └── model/
    ├── order/
    │   ├── controller/
    │   ├── service/
    │   ├── repository/
    │   └── model/
    └── common/
        ├── exception/
        ├── config/
        └── util/
```

## Class Organization

### Order of Class Members

```java
public class WellOrganizedClass {
    // 1. Constants (public static final)
    public static final int MAX_SIZE = 100;
    private static final String DEFAULT_NAME = "Default";
    
    // 2. Static variables
    private static int instanceCount = 0;
    
    // 3. Instance variables (private, then protected, then public if necessary)
    private String name;
    private int value;
    protected Date createdDate;
    
    // 4. Constructors
    public WellOrganizedClass() {
        this("Default", 0);
    }
    
    public WellOrganizedClass(String name, int value) {
        this.name = name;
        this.value = value;
        this.createdDate = new Date();
        instanceCount++;
    }
    
    // 5. Static factory methods (if any)
    public static WellOrganizedClass create(String name) {
        return new WellOrganizedClass(name, 0);
    }
    
    // 6. Public methods (API)
    public void doSomething() {
        validate();
        process();
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    // 7. Protected methods
    protected void processInternal() {
        // Implementation
    }
    
    // 8. Private methods (helpers)
    private void validate() {
        if (name == null || name.isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
    }
    
    private void process() {
        // Processing logic
    }
    
    // 9. Static utility methods
    private static String formatName(String name) {
        return name.trim().toLowerCase();
    }
    
    // 10. Override methods (equals, hashCode, toString)
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        WellOrganizedClass that = (WellOrganizedClass) o;
        return value == that.value && Objects.equals(name, that.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, value);
    }
    
    @Override
    public String toString() {
        return "WellOrganizedClass{" +
               "name='" + name + '\'' +
               ", value=" + value +
               '}';
    }
    
    // 11. Inner classes/interfaces (if needed)
    private static class Helper {
        // Helper implementation
    }
}
```

## File Organization

### One Public Class Per File

```java
// Good: User.java
public class User {
    // User implementation
}

// Bad: Models.java (multiple public classes)
public class User { }
public class Order { }
public class Product { }
```

### Related Classes Can Share File

```java
// OrderStatus.java
public class Order {
    private OrderStatus status;
    
    // Enum in same file is acceptable
    public enum OrderStatus {
        PENDING, PROCESSING, SHIPPED, DELIVERED, CANCELLED
    }
}
```

## Separation of Concerns

### Single Responsibility

Each class should have one reason to change:

```java
// Good: Separated concerns
public class UserValidator {
    public boolean validate(User user) {
        return user.getEmail() != null && user.getEmail().contains("@");
    }
}

public class UserRepository {
    public void save(User user) {
        // Database logic
    }
}

public class UserService {
    private UserValidator validator;
    private UserRepository repository;
    
    public void registerUser(User user) {
        if (validator.validate(user)) {
            repository.save(user);
        }
    }
}

// Bad: Mixed concerns
public class UserManager {
    public void registerUser(User user) {
        // Validation
        if (user.getEmail() == null) return;
        
        // Database connection
        Connection conn = DriverManager.getConnection(url);
        
        // SQL query
        PreparedStatement stmt = conn.prepareStatement("INSERT...");
        
        // Email sending
        sendWelcomeEmail(user.getEmail());
    }
}
```

## Dependency Management

### Constructor Injection

```java
// Good: Dependencies injected through constructor
public class OrderService {
    private final OrderRepository repository;
    private final PaymentService paymentService;
    private final NotificationService notificationService;
    
    public OrderService(OrderRepository repository,
                       PaymentService paymentService,
                       NotificationService notificationService) {
        this.repository = repository;
        this.paymentService = paymentService;
        this.notificationService = notificationService;
    }
}

// Bad: Direct instantiation
public class OrderService {
    private OrderRepository repository = new OrderRepository();
    private PaymentService paymentService = new PaymentService();
}
```

## Constants and Configuration

### Use Constants Classes

```java
// Constants.java
public final class Constants {
    private Constants() {} // Prevent instantiation
    
    public static final class User {
        public static final int MIN_AGE = 18;
        public static final int MAX_NAME_LENGTH = 100;
    }
    
    public static final class Order {
        public static final int MAX_ITEMS = 50;
        public static final double TAX_RATE = 0.08;
    }
}

// Usage
if (user.getAge() < Constants.User.MIN_AGE) {
    throw new IllegalArgumentException("User too young");
}
```

### Configuration Files

```properties
# application.properties
app.name=MyApplication
app.version=1.0.0

database.url=jdbc:mysql://localhost:3306/mydb
database.username=admin
database.pool.size=20

email.host=smtp.gmail.com
email.port=587
```

## Package Naming Conventions

```
com.company.project.feature.layer

Examples:
com.amazon.ecommerce.user.controller
com.amazon.ecommerce.user.service
com.amazon.ecommerce.order.repository
com.amazon.ecommerce.product.model
```

## Organizing Interfaces and Implementations

### Interface in Main Package

```
com.company.project/
├── service/
│   ├── UserService.java (interface)
│   └── impl/
│       └── UserServiceImpl.java
├── repository/
│   ├── UserRepository.java (interface)
│   └── impl/
│       └── UserRepositoryImpl.java
```

### Alternative: Separate Packages

```
com.company.project/
├── api/
│   ├── UserService.java
│   └── OrderService.java
└── impl/
    ├── UserServiceImpl.java
    └── OrderServiceImpl.java
```

## Test Organization

### Mirror Source Structure

```
src/
├── main/java/com/company/project/
│   ├── UserService.java
│   └── OrderService.java
└── test/java/com/company/project/
    ├── UserServiceTest.java
    └── OrderServiceTest.java
```

### Organized by Test Type

```
test/
├── unit/
│   ├── UserServiceTest.java
│   └── OrderServiceTest.java
├── integration/
│   ├── UserRepositoryIntegrationTest.java
│   └── OrderRepositoryIntegrationTest.java
└── e2e/
    └── CheckoutFlowTest.java
```

## Documentation Organization

```
docs/
├── architecture/
│   ├── system-design.md
│   └── database-schema.md
├── api/
│   ├── user-api.md
│   └── order-api.md
├── guides/
│   ├── setup.md
│   └── deployment.md
└── diagrams/
    ├── class-diagram.png
    └── sequence-diagram.png
```

## Common Anti-Patterns to Avoid

### ❌ God Classes

```java
// Bad: Does everything
public class SystemManager {
    public void handleUser() { }
    public void processOrder() { }
    public void sendEmail() { }
    public void connectDatabase() { }
    public void generateReport() { }
    // ... 50 more methods
}
```

### ❌ Circular Dependencies

```java
// Bad: A depends on B, B depends on A
public class ClassA {
    private ClassB b;
}

public class ClassB {
    private ClassA a;
}
```

### ❌ Deep Nesting

```java
// Bad: Too many nested packages
com.company.project.module.submodule.feature.component.service.impl.UserServiceImpl

// Good: Reasonable nesting
com.company.project.user.service.UserServiceImpl
```

## Best Practices Summary

1. ✅ **Group related classes together**
2. ✅ **Separate concerns into layers**
3. ✅ **Use meaningful package names**
4. ✅ **Keep classes focused and small**
5. ✅ **Follow consistent naming conventions**
6. ✅ **Organize class members logically**
7. ✅ **Limit dependencies between packages**
8. ✅ **Use interfaces for abstraction**
9. ✅ **Keep configuration separate from code**
10. ✅ **Mirror test structure to source structure**

## Tools for Code Organization

- **IDE Features**: Package explorer, refactoring tools
- **Static Analysis**: SonarQube, Checkstyle
- **Documentation**: JavaDoc, Swagger
- **Build Tools**: Maven, Gradle (enforce structure)
- **Linters**: ESLint, PMD

---

[Back to Best Practices](../../README.md#best-practices) | [Next: Naming Conventions →](./naming-conventions.md)
