# Open/Closed Principle (OCP)

## Definition

**Software entities (classes, modules, functions) should be open for extension but closed for modification.**

This means you should be able to add new functionality without changing existing code. The goal is to minimize the risk of breaking existing functionality when adding new features.

## Why is it Important?

1. **Stability**: Existing code remains untouched, reducing bugs
2. **Maintainability**: New features don't require modifying tested code
3. **Flexibility**: Easy to extend system behavior
4. **Reduced Risk**: Changes don't affect existing functionality
5. **Better Design**: Encourages abstraction and loose coupling

## Bad Example (Violating OCP)

```java
public class PaymentProcessor {
    public void processPayment(String paymentType, double amount) {
        if (paymentType.equals("CREDIT_CARD")) {
            System.out.println("Processing credit card payment: $" + amount);
            // Credit card specific logic
        } else if (paymentType.equals("PAYPAL")) {
            System.out.println("Processing PayPal payment: $" + amount);
            // PayPal specific logic
        } else if (paymentType.equals("BITCOIN")) {
            System.out.println("Processing Bitcoin payment: $" + amount);
            // Bitcoin specific logic
        }
        // Every time we add a new payment method, we need to modify this class!
    }
}
```

**Problems with this approach:**
- Must modify the class every time a new payment method is added
- Risk of breaking existing payment methods
- Growing if-else chain becomes difficult to maintain
- Testing requires re-testing all payment methods after each change
- Violates the "closed for modification" principle

## Good Example (Following OCP)

```java
// Abstract base for all payment methods
public interface PaymentMethod {
    void processPayment(double amount);
    String getPaymentType();
}

// Concrete implementation for Credit Card
public class CreditCardPayment implements PaymentMethod {
    private String cardNumber;
    private String cvv;
    
    public CreditCardPayment(String cardNumber, String cvv) {
        this.cardNumber = cardNumber;
        this.cvv = cvv;
    }
    
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing credit card payment: $" + amount);
        // Credit card specific logic
        validateCard();
        chargeCard(amount);
    }
    
    @Override
    public String getPaymentType() {
        return "CREDIT_CARD";
    }
    
    private void validateCard() { /* ... */ }
    private void chargeCard(double amount) { /* ... */ }
}

// Concrete implementation for PayPal
public class PayPalPayment implements PaymentMethod {
    private String email;
    private String password;
    
    public PayPalPayment(String email, String password) {
        this.email = email;
        this.password = password;
    }
    
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing PayPal payment: $" + amount);
        // PayPal specific logic
        authenticatePayPal();
        transferFunds(amount);
    }
    
    @Override
    public String getPaymentType() {
        return "PAYPAL";
    }
    
    private void authenticatePayPal() { /* ... */ }
    private void transferFunds(double amount) { /* ... */ }
}

// Concrete implementation for Bitcoin
public class BitcoinPayment implements PaymentMethod {
    private String walletAddress;
    
    public BitcoinPayment(String walletAddress) {
        this.walletAddress = walletAddress;
    }
    
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing Bitcoin payment: $" + amount);
        // Bitcoin specific logic
        createTransaction();
        broadcastToBlockchain(amount);
    }
    
    @Override
    public String getPaymentType() {
        return "BITCOIN";
    }
    
    private void createTransaction() { /* ... */ }
    private void broadcastToBlockchain(double amount) { /* ... */ }
}

// The processor is now closed for modification, open for extension
public class PaymentProcessor {
    public void processPayment(PaymentMethod paymentMethod, double amount) {
        paymentMethod.processPayment(amount);
        logTransaction(paymentMethod.getPaymentType(), amount);
    }
    
    private void logTransaction(String type, double amount) {
        System.out.println("Transaction logged: " + type + " - $" + amount);
    }
}

// Usage example
public class ShoppingCart {
    private PaymentProcessor processor;
    
    public ShoppingCart() {
        this.processor = new PaymentProcessor();
    }
    
    public void checkout(double amount, PaymentMethod paymentMethod) {
        processor.processPayment(paymentMethod, amount);
    }
}

// Adding a new payment method doesn't require changing existing code!
public class ApplePayPayment implements PaymentMethod {
    private String deviceToken;
    
    public ApplePayPayment(String deviceToken) {
        this.deviceToken = deviceToken;
    }
    
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing Apple Pay payment: $" + amount);
        // Apple Pay specific logic
    }
    
    @Override
    public String getPaymentType() {
        return "APPLE_PAY";
    }
}
```

**Benefits of this approach:**
- Adding new payment methods requires no changes to existing code
- Each payment method is independently testable
- Existing payment methods remain unaffected by new additions
- Clear separation of concerns
- Easy to extend with new functionality

## Another Example: Shape Calculator

### Bad Example (Violating OCP)

```java
public class AreaCalculator {
    public double calculateArea(Object shape) {
        if (shape instanceof Circle) {
            Circle circle = (Circle) shape;
            return Math.PI * circle.radius * circle.radius;
        } else if (shape instanceof Rectangle) {
            Rectangle rectangle = (Rectangle) shape;
            return rectangle.width * rectangle.height;
        } else if (shape instanceof Triangle) {
            Triangle triangle = (Triangle) shape;
            return 0.5 * triangle.base * triangle.height;
        }
        // Must modify this method for each new shape!
        return 0;
    }
}
```

### Good Example (Following OCP)

```java
public interface Shape {
    double calculateArea();
}

public class Circle implements Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle implements Shape {
    private double width;
    private double height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return width * height;
    }
}

public class Triangle implements Shape {
    private double base;
    private double height;
    
    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return 0.5 * base * height;
    }
}

// The calculator is now closed for modification
public class AreaCalculator {
    public double calculateTotalArea(List<Shape> shapes) {
        return shapes.stream()
                    .mapToDouble(Shape::calculateArea)
                    .sum();
    }
}
```

## Techniques to Achieve OCP

1. **Abstraction**: Use interfaces or abstract classes
2. **Inheritance**: Extend base classes for new functionality
3. **Polymorphism**: Use common interfaces for different implementations
4. **Composition**: Inject dependencies through interfaces
5. **Strategy Pattern**: Encapsulate algorithms in separate classes
6. **Decorator Pattern**: Add functionality by wrapping objects

## Real-World Analogy

Think of a power outlet:
- The outlet design is **closed for modification** (standardized)
- But it's **open for extension** - you can plug in any device that follows the standard interface
- You don't need to modify the outlet to use different devices

## Key Takeaways

1. ✅ **Use abstractions** (interfaces/abstract classes)
2. ✅ **Depend on interfaces**, not concrete implementations
3. ✅ **Extend behavior** through new classes, not by modifying existing ones
4. ✅ **Favor composition** over inheritance
5. ✅ **Design for extension** from the start

## Common Mistakes

❌ Using too many if-else or switch statements
❌ Modifying existing classes to add new features
❌ Not using interfaces or abstract classes
❌ Tight coupling to concrete implementations
❌ Over-engineering for hypothetical future requirements

## When to Apply

- When you anticipate future extensions
- When you have multiple variations of behavior
- When you want to add features without risking existing code
- During initial design of extensible systems

## Balance with Other Principles

⚠️ **Don't over-abstract**: Only apply OCP where change is expected
⚠️ **YAGNI** (You Aren't Gonna Need It): Don't create abstractions for hypothetical future needs
⚠️ **Keep it simple**: Start concrete, refactor to OCP when variation is needed

## Related Patterns

- **Strategy Pattern**: Encapsulates algorithms
- **Template Method Pattern**: Defines skeleton, subclasses fill in details
- **Decorator Pattern**: Adds responsibilities dynamically
- **Factory Pattern**: Creates objects without specifying exact class

---

[← Previous: Single Responsibility Principle](./single-responsibility.md) | [Back to SOLID Principles](../../README.md#solid-principles) | [Next: Liskov Substitution Principle →](./liskov-substitution.md)
