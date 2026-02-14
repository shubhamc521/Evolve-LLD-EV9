# Behavioral Design Patterns

## What ARE Behavioral Design Patterns?

**Behavioral patterns** focus on **communication between objects** and how they **interact and distribute responsibility**. Unlike structural patterns (which focus on object composition) and creational patterns (which focus on object creation), behavioral patterns are all about **algorithms and assignment of responsibilities between objects**.

### Mental Model

Think of behavioral patterns as the "**verb**" patterns:
- **Creational patterns** → nouns (what to create)
- **Structural patterns** → adjectives (how objects are composed)
- **Behavioral patterns** → verbs (how objects communicate and behave)

### Core Characteristics

1. **Communication Focus:** How objects talk to each other
2. **Responsibility Distribution:** Who does what in interactions
3. **Control Flow:** How requests flow through object structures
4. **Algorithm Encapsulation:** Isolating algorithms from objects

---

## The 10 Behavioral Patterns (Example: E-commerce)

### 1. **Chain of Responsibility** - Pass request along chain until handled
**E-commerce Example:** Discount chain (Member → Bulk → Seasonal discounts)
- Customer request passes through discount handlers
- Each handler can process OR forward to next
- Self-referential HAS-A creates chain

### 2. **Command** - Encapsulate request as object
**E-commerce Example:** Order processing with undo/history
- Order operations wrapped in command objects
- Supports undo/redo via command history
- Command HAS-A Receiver, Invoker HAS-A Command

### 3. **Interpreter** - Define grammar and interpreter for language
**E-commerce Example:** Discount rule parser (e.g., "member AND cart > 100")
- Rules expressed as abstract syntax tree
- Terminal expressions (leaves) + non-terminal expressions (operations)
- Recursive evaluation of expression tree

### 4. **Mediator** - Centralize communication via mediator
**E-commerce Example:** Checkout coordinator (orchestrates inventory/payment/shipping/notification)
- Services communicate through mediator, not directly
- Hub-and-spoke pattern (reduces coupling)
- Mediator knows all colleagues, colleagues know mediator

### 5. **Memento** - Save/restore object state without breaking encapsulation
**E-commerce Example:** Shopping cart snapshots for state restoration
- Originator (Cart) creates Memento (snapshot)
- Caretaker (History) stores Mementos but can't examine them
- Opaque snapshots preserve encapsulation

### 6. **Observer** - One-to-many dependency, notify observers on state change
**E-commerce Example:** Product stock notifications (email/SMS when back in stock)
- Subject (Inventory) maintains list of observers
- Observers subscribe/unsubscribe dynamically
- Subject notifies all observers on state change

### 7. **State** - Object behavior changes based on internal state
**E-commerce Example:** Order state machine (Pending → Processing → Shipped → Delivered)
- State-specific behavior encapsulated in state classes
- Context delegates to current state object
- States manage transitions themselves

### 8. **Strategy** - Encapsulate algorithm family, make interchangeable
**E-commerce Example:** Payment strategies (CreditCard/PayPal/Crypto)
- Context HAS-A Strategy reference
- Client selects strategy at runtime
- Algorithms interchangeable via composition

### 9. **Template Method** - Define algorithm skeleton, subclasses implement steps
**E-commerce Example:** Order processing template (validate → calculate → payment → ship)
- Base class defines template method (final)
- Subclasses implement abstract steps
- Base class controls flow, subclasses fill details

### 10. **Visitor** - Define new operation without changing element classes
**E-commerce Example:** Product operations (tax/discount/shipping calculators)
- Visitor interface with visit methods for each element type
- Elements accept visitor (double dispatch)
- New operations = new visitors (no element changes)

---

## Behavioral Patterns Comparison Table

| Pattern | Key Structure | Communication Model | When to Use |
|---------|---------------|---------------------|-------------|
| **Chain of Responsibility** | Self-referential HAS-A (handler → nextHandler) | Request passes through chain | Multiple objects might handle request, handler unknown |
| **Command** | Command HAS-A Receiver, Invoker HAS-A Command | Invoker decoupled from receiver | Need undo/redo, queue operations, log requests |
| **Interpreter** | Composite tree (Terminal + Non-Terminal expressions) | Recursive evaluation | Have grammar to interpret (rules, expressions) |
| **Mediator** | Colleagues HAS-A Mediator, Mediator HAS-MANY Colleagues | Hub-and-spoke | Many-to-many communication, reduce coupling |
| **Memento** | Originator creates Memento, Caretaker stores | Originator ↔ Caretaker via opaque Memento | Need undo/snapshots without breaking encapsulation |
| **Observer** | Subject HAS-MANY Observers | One-to-many broadcast | One object change affects many, need loose coupling |
| **State** | Context HAS-A State | Context delegates to current state | Behavior depends on state, avoid state conditionals |
| **Strategy** | Context HAS-A Strategy | Context delegates to strategy | Multiple algorithms, runtime selection |
| **Template Method** | Base class with template method, subclasses override steps | Base controls flow, subclasses implement | Common algorithm structure, varying steps |
| **Visitor** | Element accepts Visitor (double dispatch) | Visitor operates on element structure | Add operations without changing elements |

---

## Pattern Relationships & Comparisons

### Chain of Responsibility vs Command
- **Chain:** Request handled by one handler in chain (OR logic)
- **Command:** Request wrapped as object for undo/queue (encapsulation focus)

### Mediator vs Observer
- **Mediator:** Many-to-many, two-way (colleagues ↔ mediator)
- **Observer:** One-to-many, one-way (subject → observers)

### State vs Strategy
- **State:** Behavior changes based on internal state (transitions managed by states)
- **Strategy:** Algorithm selected by client (no transitions, client chooses)

### Template Method vs Strategy
- **Template Method:** Inheritance (IS-A), base class controls flow
- **Strategy:** Composition (HAS-A), runtime algorithm selection

### Visitor vs Strategy
- **Visitor:** Operations on element structure (double dispatch, Open/Closed for operations)
- **Strategy:** Algorithm selection (single dispatch, runtime interchangeability)

---

## How to Choose Behavioral Pattern?

Use this decision tree:

```
Need to distribute request handling?
├─ Multiple handlers might process → Chain of Responsibility
├─ Encapsulate request as object → Command
└─ Centralize communication → Mediator

Need to manage state/algorithm?
├─ Behavior depends on state → State
├─ Select algorithm at runtime → Strategy
└─ Algorithm skeleton with varying steps → Template Method

Need to process object structure?
├─ Interpret grammar/rules → Interpreter
└─ Add operations without changing elements → Visitor

Need to manage dependencies?
├─ One-to-many notification → Observer
└─ Save/restore state → Memento
```

---


### 1. **Flow Patterns** (how requests move)
- **Chain of Responsibility:** Linear flow through chain
- **Command:** Indirect flow (invoker → command → receiver)
- **Mediator:** Hub-and-spoke flow (colleagues → mediator → colleagues)

### 2. **Encapsulation Patterns** (what they hide)
- **Command:** Encapsulates request
- **Memento:** Encapsulates state snapshot
- **Strategy:** Encapsulates algorithm
- **State:** Encapsulates state-specific behavior

### 3. **Delegation Patterns** (who does the work)
- **State:** Context delegates to state
- **Strategy:** Context delegates to strategy
- **Template Method:** Base class delegates steps to subclasses
- **Visitor:** Element delegates operation to visitor

### 4. **Notification Patterns** (who gets informed)
- **Observer:** Subject notifies observers
- **Mediator:** Mediator notifies colleagues

---

## Common Pitfalls

### 1. **Chain of Responsibility**
- **Pitfall:** Request not handled (no handler catches it)
- **Solution:** Always have default handler at end

### 2. **Command**
- **Pitfall:** Undo stack memory consumption
- **Solution:** Limit stack size, compress old commands

### 3. **Interpreter**
- **Pitfall:** Complex grammars → deep expression trees
- **Solution:** Use parser generators for complex languages

### 4. **Mediator**
- **Pitfall:** Mediator becomes god object
- **Solution:** Split mediator if it handles too many concerns

### 5. **Memento**
- **Pitfall:** Memory cost of snapshots
- **Solution:** Incremental snapshots (store deltas)

### 6. **Observer**
- **Pitfall:** Memory leaks (observers not detached)
- **Solution:** Always detach observers, use weak references

### 7. **State**
- **Pitfall:** State explosion (too many states)
- **Solution:** Combine similar states, use state parameters

### 8. **Strategy**
- **Pitfall:** Client must know all strategies
- **Solution:** Use Factory to select strategy

### 9. **Template Method**
- **Pitfall:** Rigid algorithm structure
- **Solution:** Use hooks for optional steps, consider Strategy for flexibility

### 10. **Visitor**
- **Pitfall:** Adding element type breaks all visitors
- **Solution:** Only use when element hierarchy stable

---

## Real-World E-commerce Workflow (All Patterns Together)

### Scenario: Customer places order

1. **Chain of Responsibility:** Discount chain applies member/bulk/seasonal discounts
2. **Command:** Order operation wrapped as command for undo support
3. **Interpreter:** Promotional rules interpreted ("member AND cart > 100")
4. **Mediator:** Checkout coordinator orchestrates inventory/payment/shipping
5. **Memento:** Cart state saved before payment (restore on failure)
6. **Observer:** Inventory notifies subscribers when product back in stock
7. **State:** Order transitions through Pending → Processing → Shipped → Delivered
8. **Strategy:** Payment method selected (CreditCard/PayPal/Crypto)
9. **Template Method:** Order processor follows template (validate → calculate → pay → ship)
10. **Visitor:** Tax/discount/shipping calculators visit products

---

## Interview Preparation Cheat Sheet

### Quick Recognition

**"Pass request through chain"** → Chain of Responsibility  
**"Wrap request as object"** → Command  
**"Parse grammar/rules"** → Interpreter  
**"Hub for communication"** → Mediator  
**"Save/restore state"** → Memento  
**"Subscribe to notifications"** → Observer  
**"Behavior depends on state"** → State  
**"Select algorithm at runtime"** → Strategy  
**"Algorithm skeleton"** → Template Method  
**"Add operations without changes"** → Visitor

### Key Interview Questions

**Q: Difference between behavioral and structural patterns?**  
A: Structural patterns focus on object composition (HAS-A relationships). Behavioral patterns focus on communication and responsibility distribution.

**Q: Which patterns support undo/redo?**  
A: Command (undo via command history), Memento (undo via state snapshots)

**Q: Which pattern violates Open/Closed Principle?**  
A: Visitor (adding element type requires changing all visitors). Others support Open/Closed better.

**Q: Composition vs inheritance for behavior?**  
A: Strategy/State use composition (HAS-A), Template Method uses inheritance (IS-A). Prefer composition for runtime flexibility.

**Q: How to prevent memory leaks?**  
A: Observer (detach observers), Memento (limit snapshot history), Command (limit undo stack)

---

## Pattern Selection Examples

### Scenario 1: "Need to calculate total with different discount rules"
**Options:**
- Chain of Responsibility (if discounts stack sequentially)
- Strategy (if one discount selected at runtime)
- Interpreter (if discounts defined as complex rules)

### Scenario 2: "User actions need undo support"
**Options:**
- Command (undo via command history, preferred for operations)
- Memento (undo via state snapshots, preferred for state restoration)

### Scenario 3: "Multiple objects need to react when price changes"
**Options:**
- Observer (one-to-many notification, preferred)
- Mediator (if bidirectional communication needed)

### Scenario 4: "Order workflow: validate → authorize → capture → ship"
**Options:**
- Template Method (if workflow fixed, steps vary)
- State (if order can be in different states with transitions)
- Chain of Responsibility (if each step can reject and stop flow)

### Scenario 5: "Add tax/discount/shipping calculators without changing products"
**Options:**
- Visitor (operations on product hierarchy, preferred)
- Strategy (if only one calculator type needed at runtime)

---

## Behavioral Patterns in Modern Frameworks

### Spring Framework
- **Template Method:** JdbcTemplate, RestTemplate (algorithm skeleton)
- **Observer:** ApplicationEvent listeners
- **Strategy:** Transaction propagation strategies

### Node.js/Express
- **Chain of Responsibility:** Middleware chain (request → middleware1 → middleware2 → route)
- **Observer:** EventEmitter (pub-sub)

### React
- **Observer:** useState hooks (component observes state changes)
- **Strategy:** Different rendering strategies

### Android
- **Observer:** LiveData observers
- **State:** Activity lifecycle states
- **Command:** Intents (encapsulated actions)

---

## Key Takeaways

1. **Behavioral patterns** are about **communication** and **responsibility distribution**
2. **Choose based on problem:**
   - Request handling → Chain/Command/Mediator
   - Algorithm selection → Strategy/State/Template Method
   - Object structure operations → Interpreter/Visitor
   - State management → Memento/State
   - Notifications → Observer
3. **Trade-offs:**
   - Flexibility vs Complexity
   - Open/Closed for operations vs elements
   - Inheritance vs Composition
4. **Real-world:** Most systems use multiple behavioral patterns together

---

End of Behavioral Patterns Summary
