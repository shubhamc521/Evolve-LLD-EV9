# Interface Segregation Principle (ISP)

## Definition

**Clients should not be forced to depend on interfaces they do not use.**

This principle states that many client-specific interfaces are better than one general-purpose interface. No client should be forced to implement methods it doesn't need.

## Why is it Important?

1. **Focused Interfaces**: Classes implement only what they need
2. **Reduced Coupling**: Changes to unused methods don't affect clients
3. **Better Maintainability**: Smaller, focused interfaces are easier to understand
4. **Improved Flexibility**: Easier to add new implementations
5. **Cleaner Code**: No empty or throw-not-implemented methods

## Bad Example (Violating ISP)

```java
// Fat interface with too many responsibilities
public interface Worker {
    void work();
    void eat();
    void sleep();
    void attendMeeting();
    void writeCode();
    void designArchitecture();
}

// Human worker - implements all methods
public class HumanWorker implements Worker {
    @Override
    public void work() {
        System.out.println("Human working");
    }
    
    @Override
    public void eat() {
        System.out.println("Human eating");
    }
    
    @Override
    public void sleep() {
        System.out.println("Human sleeping");
    }
    
    @Override
    public void attendMeeting() {
        System.out.println("Human attending meeting");
    }
    
    @Override
    public void writeCode() {
        System.out.println("Human writing code");
    }
    
    @Override
    public void designArchitecture() {
        System.out.println("Human designing architecture");
    }
}

// Robot worker - forced to implement irrelevant methods
public class RobotWorker implements Worker {
    @Override
    public void work() {
        System.out.println("Robot working");
    }
    
    @Override
    public void eat() {
        // Robots don't eat - forced to implement anyway!
        throw new UnsupportedOperationException("Robots don't eat");
    }
    
    @Override
    public void sleep() {
        // Robots don't sleep - forced to implement anyway!
        throw new UnsupportedOperationException("Robots don't sleep");
    }
    
    @Override
    public void attendMeeting() {
        // Robots don't attend meetings
        throw new UnsupportedOperationException("Robots don't attend meetings");
    }
    
    @Override
    public void writeCode() {
        System.out.println("Robot writing code");
    }
    
    @Override
    public void designArchitecture() {
        // Robots might not design architecture
        throw new UnsupportedOperationException("Robots don't design");
    }
}
```

**Problems with this approach:**
- `RobotWorker` forced to implement methods it doesn't use
- Empty implementations or throwing exceptions
- Interface is too broad and inflexible
- Changes to the interface affect all implementations
- Violates the principle of least knowledge

## Good Example (Following ISP)

```java
// Segregated interfaces - focused and specific
public interface Workable {
    void work();
}

public interface Eatable {
    void eat();
}

public interface Sleepable {
    void sleep();
}

public interface Meetable {
    void attendMeeting();
}

public interface Codable {
    void writeCode();
}

public interface Designable {
    void designArchitecture();
}

// Human worker implements only relevant interfaces
public class HumanWorker implements Workable, Eatable, Sleepable, Meetable, Codable {
    @Override
    public void work() {
        System.out.println("Human working");
    }
    
    @Override
    public void eat() {
        System.out.println("Human eating");
    }
    
    @Override
    public void sleep() {
        System.out.println("Human sleeping");
    }
    
    @Override
    public void attendMeeting() {
        System.out.println("Human attending meeting");
    }
    
    @Override
    public void writeCode() {
        System.out.println("Human writing code");
    }
}

// Senior developer has additional capability
public class SeniorDeveloper extends HumanWorker implements Designable {
    @Override
    public void designArchitecture() {
        System.out.println("Senior developer designing architecture");
    }
}

// Robot worker implements only what it can do
public class RobotWorker implements Workable, Codable {
    @Override
    public void work() {
        System.out.println("Robot working 24/7");
    }
    
    @Override
    public void writeCode() {
        System.out.println("Robot writing code automatically");
    }
    // No forced empty implementations!
}

// Manager class that works with workable entities
public class WorkManager {
    public void manageWork(Workable worker) {
        worker.work();
    }
    
    public void scheduleLunch(Eatable entity) {
        entity.eat();
    }
    
    public void organizeMeeting(List<Meetable> attendees) {
        attendees.forEach(Meetable::attendMeeting);
    }
}
```

**Benefits of this approach:**
- Each class implements only what it needs
- No empty or exception-throwing methods
- Easy to add new worker types
- Changes to one interface don't affect unrelated classes
- Clear separation of concerns

## Another Example: Printer Interface

### Bad Example (Violating ISP)

```java
// Fat interface forcing all printers to implement all methods
public interface Printer {
    void print(Document doc);
    void scan(Document doc);
    void fax(Document doc);
    void photocopy(Document doc);
}

// All-in-one printer - works fine
public class AllInOnePrinter implements Printer {
    @Override
    public void print(Document doc) {
        System.out.println("Printing document");
    }
    
    @Override
    public void scan(Document doc) {
        System.out.println("Scanning document");
    }
    
    @Override
    public void fax(Document doc) {
        System.out.println("Faxing document");
    }
    
    @Override
    public void photocopy(Document doc) {
        System.out.println("Photocopying document");
    }
}

// Simple printer - forced to implement unsupported features
public class SimplePrinter implements Printer {
    @Override
    public void print(Document doc) {
        System.out.println("Printing document");
    }
    
    @Override
    public void scan(Document doc) {
        throw new UnsupportedOperationException("Scan not supported");
    }
    
    @Override
    public void fax(Document doc) {
        throw new UnsupportedOperationException("Fax not supported");
    }
    
    @Override
    public void photocopy(Document doc) {
        throw new UnsupportedOperationException("Photocopy not supported");
    }
}
```

### Good Example (Following ISP)

```java
// Segregated interfaces
public interface Printable {
    void print(Document doc);
}

public interface Scannable {
    void scan(Document doc);
}

public interface Faxable {
    void fax(Document doc);
}

public interface Photocopiable {
    void photocopy(Document doc);
}

// Simple printer implements only printing
public class SimplePrinter implements Printable {
    @Override
    public void print(Document doc) {
        System.out.println("Printing document");
    }
}

// All-in-one printer implements all features
public class AllInOnePrinter implements Printable, Scannable, Faxable, Photocopiable {
    @Override
    public void print(Document doc) {
        System.out.println("Printing document");
    }
    
    @Override
    public void scan(Document doc) {
        System.out.println("Scanning document");
    }
    
    @Override
    public void fax(Document doc) {
        System.out.println("Faxing document");
    }
    
    @Override
    public void photocopy(Document doc) {
        System.out.println("Photocopying document");
    }
}

// Print & Scan printer
public class PrintScanPrinter implements Printable, Scannable {
    @Override
    public void print(Document doc) {
        System.out.println("Printing document");
    }
    
    @Override
    public void scan(Document doc) {
        System.out.println("Scanning document");
    }
}

// Office manager works with specific capabilities
public class OfficeManager {
    public void printDocument(Printable printer, Document doc) {
        printer.print(doc);
    }
    
    public void scanDocument(Scannable scanner, Document doc) {
        scanner.scan(doc);
    }
    
    public void sendFax(Faxable fax, Document doc) {
        fax.fax(doc);
    }
}
```

## Interface Granularity

### Too Coarse (Violates ISP)
```java
public interface DatabaseOperations {
    void connect();
    void disconnect();
    void executeQuery(String sql);
    void executeUpdate(String sql);
    void beginTransaction();
    void commit();
    void rollback();
    void backup();
    void restore();
    void optimize();
}
```

### Too Fine (Over-segmentation)
```java
public interface Connectable { void connect(); }
public interface Disconnectable { void disconnect(); }
public interface QueryExecutable { void executeQuery(String sql); }
public interface UpdateExecutable { void executeUpdate(String sql); }
public interface TransactionBeginner { void beginTransaction(); }
public interface Commitable { void commit(); }
public interface Rollbackable { void rollback(); }
// Too many tiny interfaces - harder to maintain
```

### Just Right (Balanced)
```java
public interface DatabaseConnection {
    void connect();
    void disconnect();
}

public interface QueryExecutor {
    void executeQuery(String sql);
    void executeUpdate(String sql);
}

public interface TransactionManager {
    void beginTransaction();
    void commit();
    void rollback();
}

public interface DatabaseMaintenance {
    void backup();
    void restore();
    void optimize();
}
```

## Real-World Analogy

Think of a smartphone:
- **Bad Design**: One interface for all features (phone, camera, GPS, music, games)
  - Old phone can't use camera interface
- **Good Design**: Separate apps/interfaces for each feature
  - You only use the apps (interfaces) you need
  - Can install new apps without affecting existing ones

## Key Takeaways

1. ✅ **Keep interfaces focused and cohesive**
2. ✅ **Split large interfaces into smaller, specific ones**
3. ✅ **Clients depend only on methods they use**
4. ✅ **Prefer role interfaces over header interfaces**
5. ✅ **Balance granularity** - not too coarse, not too fine

## Common Violations

❌ Fat interfaces with many unrelated methods
❌ Forcing empty implementations
❌ Throwing UnsupportedOperationException
❌ Implementing methods that do nothing
❌ Using marker methods (methods that just return without doing anything)

## When to Apply

- When designing new interfaces
- When an interface grows too large
- When implementations have many empty methods
- When different clients need different subsets of methods
- During interface refactoring

## Benefits

✅ **Reduced coupling**: Changes don't ripple through unrelated code
✅ **Improved readability**: Smaller interfaces are easier to understand
✅ **Enhanced flexibility**: Easy to add new implementations
✅ **Better testability**: Mock only what you need
✅ **Cleaner code**: No dummy implementations

## Relationship with Other SOLID Principles

- **SRP**: Both favor focused, single-purpose abstractions
- **OCP**: ISP makes it easier to extend functionality
- **LSP**: Smaller interfaces reduce chance of LSP violations
- **DIP**: Work with ISP-compliant interfaces for better abstractions

## Anti-Pattern: Empty Methods

```java
// Bad - violates ISP
public class ReadOnlyFile implements FileOperations {
    @Override
    public String read() {
        return "content";
    }
    
    @Override
    public void write(String content) {
        // Empty - should not be here!
    }
    
    @Override
    public void delete() {
        // Empty - should not be here!
    }
}

// Good - follows ISP
public class ReadOnlyFile implements Readable {
    @Override
    public String read() {
        return "content";
    }
}
```

## Guidelines

1. **Role Interfaces**: Design interfaces around client needs, not provider capabilities
2. **High Cohesion**: Keep related methods together
3. **Client-Specific**: Different clients should have different interfaces
4. **Composition**: Combine multiple focused interfaces instead of one large interface

---

[← Previous: Liskov Substitution Principle](./liskov-substitution.md) | [Back to SOLID Principles](../../README.md#solid-principles) | [Next: Dependency Inversion Principle →](./dependency-inversion.md)
