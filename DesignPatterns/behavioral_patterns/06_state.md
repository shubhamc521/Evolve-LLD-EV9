# State Design Pattern

## What is State Pattern?
State allows an object to alter its behavior when its internal state changes. The object will appear to change its class.

**Key Idea:** Encapsulate state-specific behavior into separate state classes. Context delegates behavior to current state object.

---

## Why Use State? (Problem it solves)

**Problem:**
- Object behavior depends on its state
- Large conditional statements (if-else or switch) based on state
- State transitions scattered across code
- Adding new states requires changing multiple places

**Solution:**
- Define state interface with methods for state-specific behavior
- Create concrete state classes implementing interface
- Context holds current state object and delegates requests
- State transitions managed by state objects themselves

---

## Real-World Analogy

**Traffic Light:**
- Light has states: Red, Yellow, Green
- Behavior changes per state:
  - Red: Stop, next → Green
  - Yellow: Caution, next → Red
  - Green: Go, next → Yellow
- Light doesn't contain if-else logic; each state knows its behavior

---

## Simple Example (Vending Machine)

### UML Diagram: State Pattern Relationships

```mermaid
classDiagram
    class State {
        <<interface>>
        +insertCoin(VendingMachine) void
        +ejectCoin(VendingMachine) void
        +dispense(VendingMachine) void
    }
    
    class VendingMachine {
        -state: State
        +setState(State) void
        +insertCoin() void
        +ejectCoin() void
        +dispense() void
    }
    
    class NoCoinState {
        +insertCoin(VendingMachine) void
        +ejectCoin(VendingMachine) void
        +dispense(VendingMachine) void
    }
    class HasCoinState {
        +insertCoin(VendingMachine) void
        +ejectCoin(VendingMachine) void
        +dispense(VendingMachine) void
    }
    class DispensingState {
        +insertCoin(VendingMachine) void
        +ejectCoin(VendingMachine) void
        +dispense(VendingMachine) void
    }
    
    State <|.. NoCoinState : implements
    State <|.. HasCoinState : implements
    State <|.. DispensingState : implements
    VendingMachine o-- State : HAS-A
```

### Relationship Explanations

**1. IS-A Relationships:**
- `NoCoinState` **IS-A** `State` → implements state interface
- `HasCoinState` **IS-A** `State` → implements state interface
- `DispensingState` **IS-A** `State` → implements state interface

**2. HAS-A Relationship:**
- `VendingMachine` **HAS-A** `State` → context holds current state
- **Key:** Context delegates behavior to state object

**3. State Transition Flow:**
```
1. VendingMachine.state = NoCoinState
2. VendingMachine.insertCoin() → delegates to state.insertCoin()
3. NoCoinState.insertCoin() → changes context state to HasCoinState
4. VendingMachine.state = HasCoinState (transition complete)
```

- Context doesn't know concrete state classes (loose coupling)
- States manage transitions (state object changes context's state)
- Each state encapsulates state-specific behavior (no if-else)

```java
// State interface
public interface State {
    void insertCoin(VendingMachine machine);
    void ejectCoin(VendingMachine machine);
    void dispense(VendingMachine machine);
}

// Context: Vending Machine
public class VendingMachine {
    private State state;
    
    // Initial state: no coin
    public VendingMachine() {
        state = new NoCoinState();
    }
    
    public void setState(State state) {
        this.state = state;
    }
    
    // Delegate to current state
    public void insertCoin() {
        state.insertCoin(this);
    }
    
    public void ejectCoin() {
        state.ejectCoin(this);
    }
    
    public void dispense() {
        state.dispense(this);
    }
}

// Concrete State 1: No Coin
public class NoCoinState implements State {
    @Override
    public void insertCoin(VendingMachine machine) {
        System.out.println("Coin inserted");
        machine.setState(new HasCoinState());  // Transition
    }
    
    @Override
    public void ejectCoin(VendingMachine machine) {
        System.out.println("No coin to eject");
    }
    
    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("Insert coin first");
    }
}

// Concrete State 2: Has Coin
public class HasCoinState implements State {
    @Override
    public void insertCoin(VendingMachine machine) {
        System.out.println("Coin already inserted");
    }
    
    @Override
    public void ejectCoin(VendingMachine machine) {
        System.out.println("Coin ejected");
        machine.setState(new NoCoinState());  // Transition
    }
    
    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("Dispensing...");
        machine.setState(new DispensingState());  // Transition
    }
}

// Concrete State 3: Dispensing
public class DispensingState implements State {
    @Override
    public void insertCoin(VendingMachine machine) {
        System.out.println("Wait, dispensing...");
    }
    
    @Override
    public void ejectCoin(VendingMachine machine) {
        System.out.println("Can't eject, dispensing...");
    }
    
    @Override
    public void dispense(VendingMachine machine) {
        System.out.println("Item dispensed");
        machine.setState(new NoCoinState());  // Back to initial state
    }
}

// Usage
public class StateDemo {
    public static void main(String[] args) {
        VendingMachine machine = new VendingMachine();
        
        // State: NoCoinState
        machine.insertCoin();  // → HasCoinState
        machine.dispense();    // → DispensingState
        machine.dispense();    // → NoCoinState
        
        /* Output:
         * Coin inserted
         * Dispensing...
         * Item dispensed
         */
    }
}
```

---

## E-commerce Example (Order State Machine)

```java
// State interface
public interface OrderState {
    void confirm(Order order);
    void ship(Order order);
    void deliver(Order order);
    void cancel(Order order);
}

// Context: Order
public class Order {
    private String orderId;
    private OrderState state;
    
    public Order(String orderId) {
        this.orderId = orderId;
        this.state = new PendingState();  // Initial state
    }
    
    public void setState(OrderState state) {
        this.state = state;
    }
    
    public String getOrderId() {
        return orderId;
    }
    
    // Delegate to current state
    public void confirm() {
        state.confirm(this);
    }
    
    public void ship() {
        state.ship(this);
    }
    
    public void deliver() {
        state.deliver(this);
    }
    
    public void cancel() {
        state.cancel(this);
    }
}

// Concrete State 1: Pending
public class PendingState implements OrderState {
    @Override
    public void confirm(Order order) {
        System.out.println("Order " + order.getOrderId() + " confirmed");
        order.setState(new ProcessingState());
    }
    
    @Override
    public void ship(Order order) {
        System.out.println("Can't ship: order not confirmed");
    }
    
    @Override
    public void deliver(Order order) {
        System.out.println("Can't deliver: order not confirmed");
    }
    
    @Override
    public void cancel(Order order) {
        System.out.println("Order " + order.getOrderId() + " cancelled");
        order.setState(new CancelledState());
    }
}

// Concrete State 2: Processing
public class ProcessingState implements OrderState {
    @Override
    public void confirm(Order order) {
        System.out.println("Order already confirmed");
    }
    
    @Override
    public void ship(Order order) {
        System.out.println("Order " + order.getOrderId() + " shipped");
        order.setState(new ShippedState());
    }
    
    @Override
    public void deliver(Order order) {
        System.out.println("Can't deliver: order not shipped");
    }
    
    @Override
    public void cancel(Order order) {
        System.out.println("Order " + order.getOrderId() + " cancelled");
        order.setState(new CancelledState());
    }
}

// Concrete State 3: Shipped
public class ShippedState implements OrderState {
    @Override
    public void confirm(Order order) {
        System.out.println("Order already confirmed");
    }
    
    @Override
    public void ship(Order order) {
        System.out.println("Order already shipped");
    }
    
    @Override
    public void deliver(Order order) {
        System.out.println("Order " + order.getOrderId() + " delivered");
        order.setState(new DeliveredState());
    }
    
    @Override
    public void cancel(Order order) {
        System.out.println("Can't cancel: order already shipped");
    }
}

// Concrete State 4: Delivered
public class DeliveredState implements OrderState {
    @Override
    public void confirm(Order order) {
        System.out.println("Order already completed");
    }
    
    @Override
    public void ship(Order order) {
        System.out.println("Order already completed");
    }
    
    @Override
    public void deliver(Order order) {
        System.out.println("Order already delivered");
    }
    
    @Override
    public void cancel(Order order) {
        System.out.println("Can't cancel: order already delivered");
    }
}

// Concrete State 5: Cancelled
public class CancelledState implements OrderState {
    @Override
    public void confirm(Order order) {
        System.out.println("Order is cancelled");
    }
    
    @Override
    public void ship(Order order) {
        System.out.println("Order is cancelled");
    }
    
    @Override
    public void deliver(Order order) {
        System.out.println("Order is cancelled");
    }
    
    @Override
    public void cancel(Order order) {
        System.out.println("Order already cancelled");
    }
}

// Usage
public class OrderStateDemo {
    public static void main(String[] args) {
        Order order = new Order("ORD-123");
        
        // State transitions: Pending → Processing → Shipped → Delivered
        order.confirm();   // Pending → Processing
        order.ship();      // Processing → Shipped
        order.deliver();   // Shipped → Delivered
        
        System.out.println("\nTrying invalid transitions:");
        order.ship();      // Invalid: already delivered
        order.cancel();    // Invalid: can't cancel delivered order
        
        /* Output:
         * Order ORD-123 confirmed
         * Order ORD-123 shipped
         * Order ORD-123 delivered
         * 
         * Trying invalid transitions:
         * Order already completed
         * Can't cancel: order already delivered
         */
    }
}
```

---

## When to Use State

**Use when:**
- Object behavior depends on state
- Large conditional statements based on state
- State transitions are complex
- Example: order workflows, game characters (idle/running/jumping), connections

**Don't use when:**
- Only 2-3 simple states (if-else is simpler)
- State transitions never change
- State logic is trivial

---

End of State Pattern
