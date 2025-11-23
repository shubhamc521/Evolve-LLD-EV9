# Liskov Substitution Principle (LSP)

## Definition

**Objects of a superclass should be replaceable with objects of its subclasses without breaking the application.**

Formally stated by Barbara Liskov: *"If S is a subtype of T, then objects of type T may be replaced with objects of type S without altering any of the desirable properties of the program."*

## Why is it Important?

1. **Correctness**: Ensures derived classes don't break parent class contracts
2. **Predictability**: Subclasses behave consistently with parent classes
3. **Polymorphism**: Enables true polymorphic behavior
4. **Maintainability**: Changes to subclasses don't break existing code
5. **Reliability**: Guarantees substitutability without unexpected behavior

## Bad Example (Violating LSP)

```java
// Base class representing a bird
public class Bird {
    public void fly() {
        System.out.println("Flying in the sky");
    }
}

// Sparrow can fly - LSP is maintained
public class Sparrow extends Bird {
    @Override
    public void fly() {
        System.out.println("Sparrow flying");
    }
}

// Penguin cannot fly - LSP is violated!
public class Penguin extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Penguins can't fly!");
    }
}

// This code will break when a Penguin is used
public class BirdWatcher {
    public void makeBirdFly(Bird bird) {
        bird.fly(); // Works for Sparrow, throws exception for Penguin!
    }
}

// Usage
BirdWatcher watcher = new BirdWatcher();
watcher.makeBirdFly(new Sparrow()); // Works fine
watcher.makeBirdFly(new Penguin()); // Runtime exception! Violates LSP
```

**Problems with this approach:**
- `Penguin` breaks the contract established by `Bird`
- Cannot substitute `Penguin` where `Bird` is expected
- Runtime exceptions instead of compile-time safety
- Client code needs to know about specific subclass limitations

## Good Example (Following LSP)

```java
// More abstract base class
public abstract class Bird {
    public abstract void move();
    public abstract void eat();
}

// Interface for flying behavior
public interface Flyable {
    void fly();
}

// Sparrow can fly
public class Sparrow extends Bird implements Flyable {
    @Override
    public void move() {
        fly();
    }
    
    @Override
    public void fly() {
        System.out.println("Sparrow flying in the sky");
    }
    
    @Override
    public void eat() {
        System.out.println("Sparrow eating seeds");
    }
}

// Penguin cannot fly, but can swim
public class Penguin extends Bird {
    @Override
    public void move() {
        swim();
    }
    
    public void swim() {
        System.out.println("Penguin swimming in water");
    }
    
    @Override
    public void eat() {
        System.out.println("Penguin eating fish");
    }
}

// Now we can work with birds correctly
public class BirdWatcher {
    public void observeBird(Bird bird) {
        bird.move(); // Works for all birds
        bird.eat();  // Works for all birds
    }
    
    public void observeFlyingBird(Flyable flyingBird) {
        flyingBird.fly(); // Only accepts birds that can fly
    }
}

// Usage
BirdWatcher watcher = new BirdWatcher();
watcher.observeBird(new Sparrow());  // Works
watcher.observeBird(new Penguin());  // Works
watcher.observeFlyingBird(new Sparrow()); // Works
// watcher.observeFlyingBird(new Penguin()); // Compile error - won't compile!
```

**Benefits of this approach:**
- No violated contracts or unexpected exceptions
- Clear separation of capabilities
- Type-safe substitution
- Compiler catches incorrect usage

## Another Example: Rectangle-Square Problem

### Bad Example (Violating LSP)

```java
public class Rectangle {
    protected int width;
    protected int height;
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    public int getArea() {
        return width * height;
    }
}

// Square is-a Rectangle (mathematically), but violates LSP in OOP
public class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width; // Must keep width = height
    }
    
    @Override
    public void setHeight(int height) {
        this.width = height; // Must keep width = height
        this.height = height;
    }
}

// This code demonstrates the LSP violation
public class AreaCalculator {
    public void demonstrateViolation() {
        Rectangle rect = new Rectangle();
        rect.setWidth(5);
        rect.setHeight(4);
        System.out.println("Expected area: 20, Actual: " + rect.getArea()); // 20 ✓
        
        Rectangle square = new Square();
        square.setWidth(5);
        square.setHeight(4);
        System.out.println("Expected area: 20, Actual: " + square.getArea()); // 16 ✗
        // Square changes the behavior! Client code breaks!
    }
}
```

### Good Example (Following LSP)

```java
// Abstract base for all shapes
public interface Shape {
    int calculateArea();
}

public class Rectangle implements Shape {
    private int width;
    private int height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public int calculateArea() {
        return width * height;
    }
    
    public int getWidth() { return width; }
    public int getHeight() { return height; }
}

public class Square implements Shape {
    private int side;
    
    public Square(int side) {
        this.side = side;
    }
    
    @Override
    public int calculateArea() {
        return side * side;
    }
    
    public int getSide() { return side; }
}

// Now both can be used interchangeably as Shape
public class AreaCalculator {
    public int calculateTotalArea(List<Shape> shapes) {
        return shapes.stream()
                    .mapToInt(Shape::calculateArea)
                    .sum();
    }
}
```

## LSP Rules and Guidelines

### Pre-conditions and Post-conditions

1. **Pre-conditions cannot be strengthened** in subclass
   - If parent requires parameter ≥ 0, child cannot require ≥ 10

2. **Post-conditions cannot be weakened** in subclass
   - If parent guarantees result > 0, child must also guarantee result > 0

3. **Invariants must be preserved** by subclass
   - If parent maintains a state constraint, child must maintain it too

### Method Signatures

```java
// Good: Subclass maintains parent's contract
public class Parent {
    public Number process(Integer input) {
        return input * 2;
    }
}

public class Child extends Parent {
    // Covariant return type (more specific) - OK
    @Override
    public Integer process(Integer input) {
        return input * 3;
    }
}

// Bad: Violates LSP
public class BadChild extends Parent {
    @Override
    public Number process(Integer input) {
        if (input < 0) {
            throw new IllegalArgumentException(); // Strengthens pre-condition!
        }
        return input * 2;
    }
}
```

## Exception Handling

```java
// Good: Maintains exception contract
public class FileReader {
    public String readFile(String path) throws IOException {
        // Implementation
        return "";
    }
}

public class CachedFileReader extends FileReader {
    @Override
    public String readFile(String path) throws IOException {
        // Can throw same or fewer exceptions
        return "";
    }
}

// Bad: Changes exception behavior
public class BadFileReader extends FileReader {
    @Override
    public String readFile(String path) throws IOException, SecurityException {
        // Adding new exception violates LSP!
        return "";
    }
}
```

## Real-World Analogy

Think of a remote control:
- **Universal Remote** (parent) works with any TV
- **Brand-Specific Remote** (child) should work everywhere the universal remote works
- If the brand-specific remote requires special setup or doesn't work in some cases, it violates LSP

## Key Takeaways

1. ✅ **Subclasses must honor parent's contract**
2. ✅ **Don't strengthen preconditions**
3. ✅ **Don't weaken postconditions**
4. ✅ **Preserve invariants**
5. ✅ **No unexpected exceptions**
6. ✅ **Maintain behavioral consistency**

## Design by Contract

LSP is closely related to "Design by Contract":

- **Preconditions**: What must be true before method execution
- **Postconditions**: What must be true after method execution
- **Invariants**: What must always be true

Subclasses must respect these contracts.

## Common Violations

❌ Throwing unexpected exceptions in overridden methods
❌ Strengthening input validation in subclass
❌ Weakening output guarantees in subclass
❌ Changing method behavior in surprising ways
❌ Making methods do nothing (empty implementations)
❌ Using `instanceof` checks in client code

## When to Apply

- When designing inheritance hierarchies
- When creating polymorphic interfaces
- When refactoring to remove type checks
- During code reviews of subclass implementations

## Testing for LSP

```java
public class LSPTest {
    @Test
    public void testSubstitutability() {
        // Test with parent type
        Shape rectangle = new Rectangle(5, 4);
        assertEquals(20, rectangle.calculateArea());
        
        // Test with child type - should work the same way
        Shape square = new Square(5);
        assertTrue(square.calculateArea() > 0); // Maintains contract
    }
}
```

## Related Principles

- **Open/Closed Principle**: LSP enables OCP through proper abstraction
- **Interface Segregation Principle**: ISP helps avoid LSP violations
- **Dependency Inversion Principle**: Both rely on abstractions

---

[← Previous: Open/Closed Principle](./open-closed.md) | [Back to SOLID Principles](../../README.md#solid-principles) | [Next: Interface Segregation Principle →](./interface-segregation.md)
