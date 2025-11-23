# Factory Pattern

## Intent

**Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.**

The Factory pattern is also known as Virtual Constructor.

## Problem

Imagine you're building a logistics app:
- Initially supports only truck delivery
- Later need to add ship delivery
- Creating objects directly (`new Truck()`, `new Ship()`) makes code inflexible
- Hard to add new transport types
- Tight coupling between code and concrete classes

## Solution

The Factory pattern:
1. Replace direct object construction calls (`new`) with calls to a factory method
2. Objects are still created via `new`, but from within the factory method
3. Subclasses can override the factory method to change the class of objects being created

## Types of Factory Patterns

1. **Simple Factory** (not a design pattern, but useful idiom)
2. **Factory Method** (classic pattern)
3. **Abstract Factory** (for families of related objects)

## 1. Simple Factory

### Structure

```
┌──────────────────┐
│  SimpleFactory   │
├──────────────────┤
│ + createProduct()│──────┐
└──────────────────┘      │ creates
                          ▼
                   ┌────────────┐
                   │  Product   │
                   └────────────┘
                         △
                         │
              ┌──────────┴──────────┐
              │                     │
      ┌───────────────┐    ┌───────────────┐
      │ ConcreteProductA│   │ ConcreteProductB│
      └───────────────┘    └───────────────┘
```

### Implementation

```java
// Product interface
public interface Vehicle {
    void deliver();
}

// Concrete products
public class Truck implements Vehicle {
    @Override
    public void deliver() {
        System.out.println("Delivering by land in a truck");
    }
}

public class Ship implements Vehicle {
    @Override
    public void deliver() {
        System.out.println("Delivering by sea in a ship");
    }
}

public class Plane implements Vehicle {
    @Override
    public void deliver() {
        System.out.println("Delivering by air in a plane");
    }
}

// Simple Factory
public class VehicleFactory {
    public static Vehicle createVehicle(String type) {
        switch (type.toLowerCase()) {
            case "truck":
                return new Truck();
            case "ship":
                return new Ship();
            case "plane":
                return new Plane();
            default:
                throw new IllegalArgumentException("Unknown vehicle type: " + type);
        }
    }
}

// Client code
public class LogisticsApp {
    public static void main(String[] args) {
        // No direct object creation - use factory
        Vehicle vehicle1 = VehicleFactory.createVehicle("truck");
        vehicle1.deliver();
        
        Vehicle vehicle2 = VehicleFactory.createVehicle("ship");
        vehicle2.deliver();
        
        Vehicle vehicle3 = VehicleFactory.createVehicle("plane");
        vehicle3.deliver();
    }
}
```

**Output:**
```
Delivering by land in a truck
Delivering by sea in a ship
Delivering by air in a plane
```

## 2. Factory Method Pattern

### Structure

```
┌──────────────────┐
│     Creator      │
├──────────────────┤
│ + factoryMethod()│◀────────┐
│ + someOperation()│         │ calls
└──────────────────┘         │
         △                   │
         │                   │
┌────────┴─────────┐         │
│                  │         │
┌─────────────────┐ ┌─────────────────┐
│ConcreteCreatorA │ │ConcreteCreatorB │
├─────────────────┤ ├─────────────────┤
│+ factoryMethod()│ │+ factoryMethod()│
└─────────────────┘ └─────────────────┘
      │                     │
      │ creates             │ creates
      ▼                     ▼
┌─────────────┐      ┌─────────────┐
│ ProductA    │      │ ProductB    │
└─────────────┘      └─────────────┘
```

### Implementation

```java
// Product interface
public interface Button {
    void render();
    void onClick();
}

// Concrete products
public class WindowsButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Windows-style button");
    }
    
    @Override
    public void onClick() {
        System.out.println("Windows button clicked");
    }
}

public class MacButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Mac-style button");
    }
    
    @Override
    public void onClick() {
        System.out.println("Mac button clicked");
    }
}

public class LinuxButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Linux-style button");
    }
    
    @Override
    public void onClick() {
        System.out.println("Linux button clicked");
    }
}

// Creator (Abstract)
public abstract class Dialog {
    // Factory method (to be implemented by subclasses)
    public abstract Button createButton();
    
    // Business logic that uses the factory method
    public void render() {
        Button button = createButton();
        button.render();
        button.onClick();
    }
}

// Concrete creators
public class WindowsDialog extends Dialog {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }
}

public class MacDialog extends Dialog {
    @Override
    public Button createButton() {
        return new MacButton();
    }
}

public class LinuxDialog extends Dialog {
    @Override
    public Button createButton() {
        return new LinuxButton();
    }
}

// Client code
public class Application {
    private static Dialog dialog;
    
    public static void main(String[] args) {
        configure();
        runBusinessLogic();
    }
    
    private static void configure() {
        String osName = System.getProperty("os.name").toLowerCase();
        
        if (osName.contains("win")) {
            dialog = new WindowsDialog();
        } else if (osName.contains("mac")) {
            dialog = new MacDialog();
        } else {
            dialog = new LinuxDialog();
        }
    }
    
    private static void runBusinessLogic() {
        dialog.render();
    }
}
```

## Real-World Example: Document Creator

```java
// Product interface
public interface Document {
    void open();
    void save();
    void close();
}

// Concrete products
public class WordDocument implements Document {
    private String content = "";
    
    @Override
    public void open() {
        System.out.println("Opening Word document");
    }
    
    @Override
    public void save() {
        System.out.println("Saving Word document: " + content);
    }
    
    @Override
    public void close() {
        System.out.println("Closing Word document");
    }
}

public class PDFDocument implements Document {
    @Override
    public void open() {
        System.out.println("Opening PDF document");
    }
    
    @Override
    public void save() {
        System.out.println("Saving PDF document");
    }
    
    @Override
    public void close() {
        System.out.println("Closing PDF document");
    }
}

public class ExcelDocument implements Document {
    @Override
    public void open() {
        System.out.println("Opening Excel document");
    }
    
    @Override
    public void save() {
        System.out.println("Saving Excel document");
    }
    
    @Override
    public void close() {
        System.out.println("Closing Excel document");
    }
}

// Creator
public abstract class Application {
    public abstract Document createDocument();
    
    public void newDocument() {
        Document doc = createDocument();
        doc.open();
        // Work with document
        doc.save();
        doc.close();
    }
}

// Concrete creators
public class WordApplication extends Application {
    @Override
    public Document createDocument() {
        return new WordDocument();
    }
}

public class PDFApplication extends Application {
    @Override
    public Document createDocument() {
        return new PDFDocument();
    }
}

public class ExcelApplication extends Application {
    @Override
    public Document createDocument() {
        return new ExcelDocument();
    }
}

// Usage
public class DocumentDemo {
    public static void main(String[] args) {
        Application wordApp = new WordApplication();
        wordApp.newDocument();
        
        System.out.println();
        
        Application pdfApp = new PDFApplication();
        pdfApp.newDocument();
    }
}
```

## Notification System Example

```java
// Product interface
public interface Notification {
    void send(String message, String recipient);
}

// Concrete products
public class EmailNotification implements Notification {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending email to " + recipient);
        System.out.println("Message: " + message);
    }
}

public class SMSNotification implements Notification {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending SMS to " + recipient);
        System.out.println("Message: " + message);
    }
}

public class PushNotification implements Notification {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending push notification to " + recipient);
        System.out.println("Message: " + message);
    }
}

// Creator
public abstract class NotificationService {
    public abstract Notification createNotification();
    
    public void notify(String message, String recipient) {
        Notification notification = createNotification();
        notification.send(message, recipient);
        logNotification(message, recipient);
    }
    
    private void logNotification(String message, String recipient) {
        System.out.println("Logged notification to: " + recipient);
    }
}

// Concrete creators
public class EmailNotificationService extends NotificationService {
    @Override
    public Notification createNotification() {
        return new EmailNotification();
    }
}

public class SMSNotificationService extends NotificationService {
    @Override
    public Notification createNotification() {
        return new SMSNotification();
    }
}

public class PushNotificationService extends NotificationService {
    @Override
    public Notification createNotification() {
        return new PushNotification();
    }
}

// Client code with factory selection
public class NotificationFactory {
    public static NotificationService getService(String type) {
        switch (type.toLowerCase()) {
            case "email":
                return new EmailNotificationService();
            case "sms":
                return new SMSNotificationService();
            case "push":
                return new PushNotificationService();
            default:
                throw new IllegalArgumentException("Unknown notification type");
        }
    }
}

// Usage
public class NotificationDemo {
    public static void main(String[] args) {
        // User preferences
        String preferredMethod = "email";
        
        NotificationService service = NotificationFactory.getService(preferredMethod);
        service.notify("Your order has been shipped!", "user@example.com");
        
        System.out.println("\n--- Different notification method ---\n");
        
        NotificationService smsService = NotificationFactory.getService("sms");
        smsService.notify("Your OTP is 123456", "+1234567890");
    }
}
```

## Pizza Store Example

```java
// Product interface
public abstract class Pizza {
    protected String name;
    protected String dough;
    protected String sauce;
    protected List<String> toppings = new ArrayList<>();
    
    public void prepare() {
        System.out.println("Preparing " + name);
        System.out.println("Tossing dough: " + dough);
        System.out.println("Adding sauce: " + sauce);
        System.out.println("Adding toppings: ");
        for (String topping : toppings) {
            System.out.println("   " + topping);
        }
    }
    
    public void bake() {
        System.out.println("Baking for 25 minutes at 350");
    }
    
    public void cut() {
        System.out.println("Cutting the pizza into diagonal slices");
    }
    
    public void box() {
        System.out.println("Placing pizza in official PizzaStore box");
    }
    
    public String getName() {
        return name;
    }
}

// Concrete products - NY Style
public class NYStyleCheesePizza extends Pizza {
    public NYStyleCheesePizza() {
        name = "NY Style Sauce and Cheese Pizza";
        dough = "Thin Crust Dough";
        sauce = "Marinara Sauce";
        toppings.add("Grated Reggiano Cheese");
    }
}

public class NYStyleVeggiePizza extends Pizza {
    public NYStyleVeggiePizza() {
        name = "NY Style Veggie Pizza";
        dough = "Thin Crust Dough";
        sauce = "Marinara Sauce";
        toppings.add("Mushrooms");
        toppings.add("Onions");
        toppings.add("Bell Peppers");
    }
}

// Concrete products - Chicago Style
public class ChicagoStyleCheesePizza extends Pizza {
    public ChicagoStyleCheesePizza() {
        name = "Chicago Style Deep Dish Cheese Pizza";
        dough = "Extra Thick Crust Dough";
        sauce = "Plum Tomato Sauce";
        toppings.add("Shredded Mozzarella Cheese");
    }
    
    @Override
    public void cut() {
        System.out.println("Cutting the pizza into square slices");
    }
}

public class ChicagoStyleVeggiePizza extends Pizza {
    public ChicagoStyleVeggiePizza() {
        name = "Chicago Style Deep Dish Veggie Pizza";
        dough = "Extra Thick Crust Dough";
        sauce = "Plum Tomato Sauce";
        toppings.add("Black Olives");
        toppings.add("Spinach");
        toppings.add("Eggplant");
    }
    
    @Override
    public void cut() {
        System.out.println("Cutting the pizza into square slices");
    }
}

// Creator
public abstract class PizzaStore {
    // Factory method
    protected abstract Pizza createPizza(String type);
    
    // Template method that uses factory method
    public Pizza orderPizza(String type) {
        Pizza pizza = createPizza(type);
        
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        
        return pizza;
    }
}

// Concrete creators
public class NYPizzaStore extends PizzaStore {
    @Override
    protected Pizza createPizza(String type) {
        switch (type) {
            case "cheese":
                return new NYStyleCheesePizza();
            case "veggie":
                return new NYStyleVeggiePizza();
            default:
                return null;
        }
    }
}

public class ChicagoPizzaStore extends PizzaStore {
    @Override
    protected Pizza createPizza(String type) {
        switch (type) {
            case "cheese":
                return new ChicagoStyleCheesePizza();
            case "veggie":
                return new ChicagoStyleVeggiePizza();
            default:
                return null;
        }
    }
}

// Usage
public class PizzaDemo {
    public static void main(String[] args) {
        PizzaStore nyStore = new NYPizzaStore();
        PizzaStore chicagoStore = new ChicagoPizzaStore();
        
        System.out.println("=== Order 1 ===");
        Pizza pizza1 = nyStore.orderPizza("cheese");
        System.out.println("Ordered: " + pizza1.getName() + "\n");
        
        System.out.println("=== Order 2 ===");
        Pizza pizza2 = chicagoStore.orderPizza("veggie");
        System.out.println("Ordered: " + pizza2.getName() + "\n");
    }
}
```

## When to Use

✅ **Use Factory pattern when:**
- You don't know ahead of time the exact types of objects your code will work with
- You want to provide a library and hide implementation details
- You want to save system resources by reusing existing objects
- The creation process is complex
- You want to decouple object creation from usage

Examples:
- GUI frameworks (creating UI components)
- Database connections
- Logging frameworks
- Document processing systems
- Notification systems

## Advantages

✅ Single Responsibility - object creation is separated from usage
✅ Open/Closed Principle - easy to add new product types
✅ Loose coupling - code works with interfaces, not concrete classes
✅ Flexibility - easy to extend and customize
✅ Code reuse - factory logic centralized

## Disadvantages

❌ Increased complexity - more classes and interfaces
❌ Can be overkill for simple object creation
❌ May require parallel class hierarchies

## Factory Method vs Simple Factory

| Factory Method | Simple Factory |
|----------------|----------------|
| Uses inheritance | Uses composition |
| Subclasses decide what to create | Method decides what to create |
| More flexible | Simpler |
| Follows OCP better | Violates OCP when adding new types |

## Best Practices

1. **Use interfaces**: Products should implement common interface
2. **Factory as abstract method**: Let subclasses override
3. **Naming convention**: Use descriptive names (createXXX, makeXXX)
4. **Parameter validation**: Validate input parameters
5. **Return type**: Use most abstract type possible
6. **Combine with Singleton**: Factory can be singleton

## Related Patterns

- **Abstract Factory**: Creates families of related objects
- **Prototype**: Alternative to Factory for creating objects
- **Singleton**: Factory can be implemented as Singleton
- **Template Method**: Often used together with Factory Method

## Real-World Examples

- `java.util.Calendar.getInstance()`
- `java.text.NumberFormat.getInstance()`
- `java.nio.charset.Charset.forName()`
- JDBC `DriverManager.getConnection()`
- Spring Framework bean factories

---

[← Previous: Singleton Pattern](./singleton.md) | [Back to Design Patterns](../../../README.md#design-patterns)
