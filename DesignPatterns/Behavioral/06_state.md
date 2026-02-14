# State

Change behavior when state changes

Context HAS-A State object

## Example
Vending Machine: NoCoin → HasCoin → Dispensed

## UML Diagram

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

## Key Points

- VendingMachine HAS-A State
- Context delegates to current state
- States manage transitions themselves

## Code

```java
public interface State {
    void insertCoin(VendingMachine vm);
    void ejectCoin(VendingMachine vm);
    void dispense(VendingMachine vm);
}

public class NoCoinState implements State {
    public void insertCoin(VendingMachine vm) {
        System.out.println("Coin inserted");
        vm.setState(new HasCoinState());
    }
    
    public void ejectCoin(VendingMachine vm) {
        System.out.println("No coin to eject");
    }
    
    public void dispense(VendingMachine vm) {
        System.out.println("Insert coin first");
    }
}

public class VendingMachine {
    private State state;
    
    public VendingMachine() {
        state = new NoCoinState();
    }
    
    public void setState(State state) {
        this.state = state;
    }
    
    public void insertCoin() {
        state.insertCoin(this);
    }
    
    public void dispense() {
        state.dispense(this);
    }
}

// Usage
VendingMachine vm = new VendingMachine();
vm.insertCoin();  // State changes to HasCoinState
vm.dispense();    // State changes to DispensingState
```

## When to use?

- Object behavior depends on state
- Avoid large if-else/switch based on state
- State transitions managed by states themselves
