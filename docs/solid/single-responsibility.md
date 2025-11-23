# Single Responsibility Principle (SRP)

## Definition

**A class should have only one reason to change, meaning it should have only one job or responsibility.**

The Single Responsibility Principle states that every class, module, or function should have responsibility over a single part of the functionality provided by the software, and that responsibility should be entirely encapsulated by the class.

## Why is it Important?

1. **Easier to Understand**: Classes with a single responsibility are simpler and easier to understand
2. **Easier to Maintain**: Changes to one responsibility don't affect other functionalities
3. **Easier to Test**: Testing becomes simpler when each class has a focused purpose
4. **Reduces Coupling**: Dependencies between classes are minimized
5. **Better Reusability**: Single-purpose classes are more likely to be reusable

## Bad Example (Violating SRP)

```java
// This class has multiple responsibilities: user management, email, and persistence
public class User {
    private String name;
    private String email;
    
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
    
    // User data responsibility
    public String getName() { return name; }
    public String getEmail() { return email; }
    
    // Email sending responsibility - VIOLATION!
    public void sendEmail(String message) {
        // SMTP connection logic
        // Email formatting
        // Sending email
        System.out.println("Sending email to " + email + ": " + message);
    }
    
    // Database persistence responsibility - VIOLATION!
    public void saveToDatabase() {
        // Database connection
        // SQL query execution
        // Transaction management
        System.out.println("Saving user to database");
    }
    
    // Validation responsibility - VIOLATION!
    public boolean validateEmail() {
        return email.contains("@");
    }
}
```

**Problems with this approach:**
- The `User` class has 4 different reasons to change:
  1. User data structure changes
  2. Email sending logic changes
  3. Database persistence logic changes
  4. Email validation rules change
- Difficult to test individual responsibilities
- High coupling between unrelated concerns

## Good Example (Following SRP)

```java
// Responsibility: Hold user data
public class User {
    private String name;
    private String email;
    
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
    
    public String getName() { return name; }
    public String getEmail() { return email; }
}

// Responsibility: Validate user data
public class UserValidator {
    public boolean validateEmail(String email) {
        return email != null && email.contains("@") && email.contains(".");
    }
    
    public boolean validateName(String name) {
        return name != null && !name.trim().isEmpty();
    }
}

// Responsibility: Send emails
public class EmailService {
    public void sendEmail(String to, String subject, String message) {
        // SMTP connection logic
        // Email formatting
        // Sending email
        System.out.println("Sending email to " + to);
        System.out.println("Subject: " + subject);
        System.out.println("Message: " + message);
    }
}

// Responsibility: Persist user data
public class UserRepository {
    public void save(User user) {
        // Database connection
        // SQL query execution
        // Transaction management
        System.out.println("Saving user: " + user.getName());
    }
    
    public User findByEmail(String email) {
        // Database query logic
        return null;
    }
}

// Usage example
public class UserService {
    private UserValidator validator;
    private UserRepository repository;
    private EmailService emailService;
    
    public UserService(UserValidator validator, UserRepository repository, EmailService emailService) {
        this.validator = validator;
        this.repository = repository;
        this.emailService = emailService;
    }
    
    public void registerUser(String name, String email) {
        if (!validator.validateName(name) || !validator.validateEmail(email)) {
            throw new IllegalArgumentException("Invalid user data");
        }
        
        User user = new User(name, email);
        repository.save(user);
        emailService.sendEmail(email, "Welcome!", "Welcome to our platform!");
    }
}
```

**Benefits of this approach:**
- Each class has a single, well-defined responsibility
- Easy to test each component independently
- Changes to email logic don't affect user data or persistence
- Better code organization and maintainability
- Classes can be reused in different contexts

## Real-World Analogy

Think of a restaurant:
- **Chef** (cooks food) - one responsibility
- **Waiter** (serves customers) - one responsibility
- **Cashier** (handles payments) - one responsibility
- **Manager** (oversees operations) - one responsibility

If one person tried to do all jobs, it would be inefficient and error-prone. Similarly, each class should focus on doing one thing well.

## Key Takeaways

1. ✅ **One class, one responsibility**
2. ✅ **One reason to change**
3. ✅ **High cohesion within a class**
4. ✅ **Low coupling between classes**
5. ✅ **Separation of concerns**

## Common Mistakes

❌ Mixing business logic with presentation logic
❌ Combining data access with business logic
❌ Including validation, formatting, and persistence in one class
❌ Creating "God classes" that do everything

## When to Apply

- When designing new classes
- When a class becomes difficult to understand or maintain
- When changes to one feature require modifying multiple unrelated parts
- During code refactoring sessions

## Related Principles

- **Separation of Concerns** (SoC)
- **High Cohesion, Low Coupling**
- **KISS** (Keep It Simple, Stupid)

---

[← Back to SOLID Principles](../../README.md#solid-principles) | [Next: Open/Closed Principle →](./open-closed.md)
