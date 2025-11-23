# Strategy Pattern

## Intent

**Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.**

The Strategy pattern is also known as the Policy pattern.

## Problem

Imagine you're building a navigation app:
- Different routing strategies: shortest path, fastest route, scenic route
- Hard-coding all algorithms in one class leads to:
  - Complex conditional logic (if-else chains)
  - Difficult to add new strategies
  - Hard to test individual strategies
  - Violates Open/Closed Principle

## Solution

The Strategy pattern:
1. Defines a family of algorithms
2. Encapsulates each algorithm in a separate class
3. Makes algorithms interchangeable
4. Clients can choose which algorithm to use at runtime

## Structure

```
┌─────────────┐               ┌──────────────────┐
│   Context   │──────────────▷│    Strategy      │
├─────────────┤               ├──────────────────┤
│ - strategy  │               │ + execute()      │
├─────────────┤               └──────────────────┘
│ + setStrategy()│                     △
│ + doSomething()│                     │
└─────────────┘              ┌─────────┴─────────┐
                             │                   │
                    ┌────────────────┐  ┌────────────────┐
                    │ ConcreteStrategyA│ │ConcreteStrategyB│
                    ├────────────────┤  ├────────────────┤
                    │ + execute()    │  │ + execute()    │
                    └────────────────┘  └────────────────┘
```

## Implementation

### Basic Example: Payment Processing

```java
// Strategy interface
public interface PaymentStrategy {
    void pay(int amount);
}

// Concrete Strategy: Credit Card
public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    private String cvv;
    private String expiryDate;
    
    public CreditCardPayment(String cardNumber, String cvv, String expiryDate) {
        this.cardNumber = cardNumber;
        this.cvv = cvv;
        this.expiryDate = expiryDate;
    }
    
    @Override
    public void pay(int amount) {
        System.out.println("Paid $" + amount + " using Credit Card ending in " + 
                         cardNumber.substring(cardNumber.length() - 4));
        // Credit card processing logic
    }
}

// Concrete Strategy: PayPal
public class PayPalPayment implements PaymentStrategy {
    private String email;
    private String password;
    
    public PayPalPayment(String email, String password) {
        this.email = email;
        this.password = password;
    }
    
    @Override
    public void pay(int amount) {
        System.out.println("Paid $" + amount + " using PayPal account: " + email);
        // PayPal processing logic
    }
}

// Concrete Strategy: Bitcoin
public class BitcoinPayment implements PaymentStrategy {
    private String walletAddress;
    
    public BitcoinPayment(String walletAddress) {
        this.walletAddress = walletAddress;
    }
    
    @Override
    public void pay(int amount) {
        System.out.println("Paid $" + amount + " using Bitcoin wallet: " + walletAddress);
        // Bitcoin processing logic
    }
}

// Context
public class ShoppingCart {
    private List<Item> items;
    private PaymentStrategy paymentStrategy;
    
    public ShoppingCart() {
        this.items = new ArrayList<>();
    }
    
    public void addItem(Item item) {
        items.add(item);
    }
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void checkout() {
        int total = calculateTotal();
        paymentStrategy.pay(total);
    }
    
    private int calculateTotal() {
        return items.stream().mapToInt(Item::getPrice).sum();
    }
}

// Item class
class Item {
    private String name;
    private int price;
    
    public Item(String name, int price) {
        this.name = name;
        this.price = price;
    }
    
    public int getPrice() { return price; }
    public String getName() { return name; }
}

// Usage
public class StrategyDemo {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        cart.addItem(new Item("Laptop", 1000));
        cart.addItem(new Item("Mouse", 50));
        
        // Pay with credit card
        cart.setPaymentStrategy(new CreditCardPayment("1234567890123456", "123", "12/25"));
        cart.checkout();
        
        // Change strategy to PayPal
        cart.setPaymentStrategy(new PayPalPayment("user@example.com", "password"));
        cart.checkout();
        
        // Change strategy to Bitcoin
        cart.setPaymentStrategy(new BitcoinPayment("1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"));
        cart.checkout();
    }
}
```

**Output:**
```
Paid $1050 using Credit Card ending in 3456
Paid $1050 using PayPal account: user@example.com
Paid $1050 using Bitcoin wallet: 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa
```

## Sorting Strategy Example

```java
// Strategy interface
public interface SortStrategy {
    void sort(int[] array);
}

// Concrete Strategy: Bubble Sort
public class BubbleSort implements SortStrategy {
    @Override
    public void sort(int[] array) {
        System.out.println("Sorting using Bubble Sort");
        int n = array.length;
        for (int i = 0; i < n - 1; i++) {
            for (int j = 0; j < n - i - 1; j++) {
                if (array[j] > array[j + 1]) {
                    int temp = array[j];
                    array[j] = array[j + 1];
                    array[j + 1] = temp;
                }
            }
        }
    }
}

// Concrete Strategy: Quick Sort
public class QuickSort implements SortStrategy {
    @Override
    public void sort(int[] array) {
        System.out.println("Sorting using Quick Sort");
        quickSort(array, 0, array.length - 1);
    }
    
    private void quickSort(int[] array, int low, int high) {
        if (low < high) {
            int pi = partition(array, low, high);
            quickSort(array, low, pi - 1);
            quickSort(array, pi + 1, high);
        }
    }
    
    private int partition(int[] array, int low, int high) {
        int pivot = array[high];
        int i = low - 1;
        for (int j = low; j < high; j++) {
            if (array[j] < pivot) {
                i++;
                int temp = array[i];
                array[i] = array[j];
                array[j] = temp;
            }
        }
        int temp = array[i + 1];
        array[i + 1] = array[high];
        array[high] = temp;
        return i + 1;
    }
}

// Concrete Strategy: Merge Sort
public class MergeSort implements SortStrategy {
    @Override
    public void sort(int[] array) {
        System.out.println("Sorting using Merge Sort");
        mergeSort(array, 0, array.length - 1);
    }
    
    private void mergeSort(int[] array, int left, int right) {
        if (left < right) {
            int mid = (left + right) / 2;
            mergeSort(array, left, mid);
            mergeSort(array, mid + 1, right);
            merge(array, left, mid, right);
        }
    }
    
    private void merge(int[] array, int left, int mid, int right) {
        int n1 = mid - left + 1;
        int n2 = right - mid;
        
        int[] L = new int[n1];
        int[] R = new int[n2];
        
        System.arraycopy(array, left, L, 0, n1);
        System.arraycopy(array, mid + 1, R, 0, n2);
        
        int i = 0, j = 0, k = left;
        while (i < n1 && j < n2) {
            if (L[i] <= R[j]) {
                array[k++] = L[i++];
            } else {
                array[k++] = R[j++];
            }
        }
        
        while (i < n1) array[k++] = L[i++];
        while (j < n2) array[k++] = R[j++];
    }
}

// Context
public class Sorter {
    private SortStrategy strategy;
    
    public void setStrategy(SortStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void sort(int[] array) {
        strategy.sort(array);
    }
}

// Usage
public class SortDemo {
    public static void main(String[] args) {
        int[] data = {64, 34, 25, 12, 22, 11, 90};
        
        Sorter sorter = new Sorter();
        
        // Use Bubble Sort
        sorter.setStrategy(new BubbleSort());
        sorter.sort(data.clone());
        
        // Use Quick Sort for large datasets
        sorter.setStrategy(new QuickSort());
        sorter.sort(data.clone());
        
        // Use Merge Sort for stability
        sorter.setStrategy(new MergeSort());
        sorter.sort(data.clone());
    }
}
```

## Compression Strategy Example

```java
// Strategy interface
public interface CompressionStrategy {
    void compress(String fileName);
    void decompress(String fileName);
}

// Concrete Strategy: ZIP
public class ZipCompression implements CompressionStrategy {
    @Override
    public void compress(String fileName) {
        System.out.println("Compressing " + fileName + " using ZIP compression");
        // ZIP compression logic
    }
    
    @Override
    public void decompress(String fileName) {
        System.out.println("Decompressing " + fileName + " using ZIP decompression");
        // ZIP decompression logic
    }
}

// Concrete Strategy: RAR
public class RarCompression implements CompressionStrategy {
    @Override
    public void compress(String fileName) {
        System.out.println("Compressing " + fileName + " using RAR compression");
        // RAR compression logic
    }
    
    @Override
    public void decompress(String fileName) {
        System.out.println("Decompressing " + fileName + " using RAR decompression");
        // RAR decompression logic
    }
}

// Context
public class FileCompressor {
    private CompressionStrategy strategy;
    
    public FileCompressor(CompressionStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void setCompressionStrategy(CompressionStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void compressFile(String fileName) {
        strategy.compress(fileName);
    }
    
    public void decompressFile(String fileName) {
        strategy.decompress(fileName);
    }
}
```

## Duck Behavior Example (Classic)

```java
// Strategy interfaces
public interface FlyBehavior {
    void fly();
}

public interface QuackBehavior {
    void quack();
}

// Concrete fly behaviors
public class FlyWithWings implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("Flying with wings!");
    }
}

public class FlyNoWay implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("Can't fly");
    }
}

// Concrete quack behaviors
public class Quack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Quack quack!");
    }
}

public class Squeak implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Squeak!");
    }
}

public class MuteQuack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("<< Silence >>");
    }
}

// Context
public abstract class Duck {
    protected FlyBehavior flyBehavior;
    protected QuackBehavior quackBehavior;
    
    public void performFly() {
        flyBehavior.fly();
    }
    
    public void performQuack() {
        quackBehavior.quack();
    }
    
    public void swim() {
        System.out.println("All ducks float!");
    }
    
    public void setFlyBehavior(FlyBehavior fb) {
        flyBehavior = fb;
    }
    
    public void setQuackBehavior(QuackBehavior qb) {
        quackBehavior = qb;
    }
    
    public abstract void display();
}

// Concrete ducks
public class MallardDuck extends Duck {
    public MallardDuck() {
        flyBehavior = new FlyWithWings();
        quackBehavior = new Quack();
    }
    
    @Override
    public void display() {
        System.out.println("I'm a Mallard duck");
    }
}

public class RubberDuck extends Duck {
    public RubberDuck() {
        flyBehavior = new FlyNoWay();
        quackBehavior = new Squeak();
    }
    
    @Override
    public void display() {
        System.out.println("I'm a Rubber duck");
    }
}

// Usage
public class DuckSimulator {
    public static void main(String[] args) {
        Duck mallard = new MallardDuck();
        mallard.display();
        mallard.performFly();
        mallard.performQuack();
        
        System.out.println();
        
        Duck rubber = new RubberDuck();
        rubber.display();
        rubber.performFly();
        rubber.performQuack();
        
        System.out.println("\n--- Changing rubber duck behavior ---");
        rubber.setFlyBehavior(new FlyWithWings());
        rubber.performFly();
    }
}
```

## Strategy with Lambda (Java 8+)

```java
// Functional interface
@FunctionalInterface
public interface DiscountStrategy {
    double applyDiscount(double price);
}

// Context
public class PriceCalculator {
    public double calculatePrice(double price, DiscountStrategy strategy) {
        return strategy.applyDiscount(price);
    }
}

// Usage with lambdas
public class LambdaStrategyDemo {
    public static void main(String[] args) {
        PriceCalculator calculator = new PriceCalculator();
        double originalPrice = 100.0;
        
        // No discount
        double price1 = calculator.calculatePrice(originalPrice, price -> price);
        System.out.println("No discount: $" + price1);
        
        // 10% discount
        double price2 = calculator.calculatePrice(originalPrice, price -> price * 0.9);
        System.out.println("10% discount: $" + price2);
        
        // $20 off
        double price3 = calculator.calculatePrice(originalPrice, price -> price - 20);
        System.out.println("$20 off: $" + price3);
        
        // Black Friday: 50% off
        DiscountStrategy blackFriday = price -> price * 0.5;
        double price4 = calculator.calculatePrice(originalPrice, blackFriday);
        System.out.println("Black Friday: $" + price4);
    }
}
```

## When to Use

✅ **Use Strategy when:**
- You have multiple algorithms for a specific task
- You want to switch algorithms at runtime
- You want to avoid complex conditional statements
- Related classes differ only in behavior
- You need different variants of an algorithm

Examples:
- Payment processing systems
- Sorting algorithms
- Compression algorithms
- Validation rules
- Navigation routing
- Pricing strategies
- Authentication methods

## Advantages

✅ Open/Closed Principle - add new strategies without changing context
✅ Single Responsibility - isolate algorithm implementation
✅ Runtime algorithm selection
✅ Eliminates conditional statements
✅ Better testability - test strategies independently
✅ Composition over inheritance

## Disadvantages

❌ Increased number of classes
❌ Clients must be aware of different strategies
❌ Communication overhead between context and strategy
❌ May be overkill for simple algorithms

## Strategy vs State Pattern

| Strategy | State |
|----------|-------|
| Focuses on interchangeable algorithms | Focuses on state transitions |
| Client chooses strategy | Context manages state changes |
| Strategies independent | States may know about other states |
| Behavior doesn't affect object state | Behavior changes with state |

## Best Practices

1. **Use interfaces**: Define strategy interface clearly
2. **Favor composition**: Context has-a strategy
3. **Immutability**: Make strategies immutable when possible
4. **Factory pattern**: Create strategies using factory
5. **Default strategy**: Provide sensible default
6. **Lambda expressions**: Use for simple strategies (Java 8+)

## Related Patterns

- **Factory Method**: Create appropriate strategy
- **Template Method**: Alternative to Strategy (uses inheritance)
- **State**: Similar structure but different intent
- **Command**: Encapsulates requests, Strategy encapsulates algorithms

## Real-World Examples

- `java.util.Comparator` (sorting strategies)
- `javax.servlet.http.HttpServlet` (doGet, doPost methods)
- Spring Framework validators
- Java 8 Streams (sorting, filtering)
- Layout managers in Java Swing

---

[← Previous: Observer Pattern](./observer.md) | [Back to Design Patterns](../../../README.md#design-patterns)
