# Behavioral Design Patterns

Behavioral - How objects COMMUNICATE with each other

Creational - HOW to create objects
Structural - HOW objects are organized/composed
Behavioral - HOW objects communicate/behave


## 10 Behavioral Patterns

1. Chain of Responsibility - Pass request along chain
2. Command - Wrap request as object
3. Interpreter - Parse/evaluate language
4. Mediator - Central communication hub
5. Memento - Save/restore state
6. Observer - Notify many when one changes
7. State - Change behavior when state changes
8. Strategy - Swap algorithms at runtime
9. Template Method - Define skeleton, fill steps
10. Visitor - Add operations without changing classes


## Quick Examples

Chain of Responsibility:
Request → Handler1 → Handler2 → Handler3
[ATM: 500 note → 200 note → 100 note]

Command:
Action wrapped in object (Undo/Redo)
[Remote Control - each button is a command]

Observer:
Subject changes → All observers notified
[YouTube Channel → Subscribers get notification]

State:
Context behavior changes with state
[Vending Machine: NoCoin → HasCoin → Dispensed]

Strategy:
Pick algorithm at runtime
[Payment: CreditCard / UPI / Wallet]

Template Method:
Base class defines steps, subclass fills
[Coffee/Tea maker: boil → brew → pour]


## When to use?

Need to pass request through multiple handlers? → Chain
Need undo/redo? → Command
Objects need to react when one changes? → Observer
Behavior changes with state? → State
Multiple algorithms to choose from? → Strategy
Fixed steps but different implementations? → Template Method


## Key Differences

State vs Strategy:
- State: Automatic transition between states
- Strategy: Client picks algorithm explicitly

Observer vs Mediator:
- Observer: One-to-many (Subject → Observers)
- Mediator: Many-to-many via hub (All ↔ Mediator)

Template Method vs Strategy:
- Template: Uses inheritance (extends base)
- Strategy: Uses composition (HAS-A strategy)


## Composition Patterns (HAS-A)

Command HAS-A Receiver
Observer: Subject HAS-A list of Observers
State: Context HAS-A State object
Strategy: Context HAS-A Strategy object
Mediator HAS-A Colleagues


## Self-Referential (Chain)

Chain of Responsibility: Handler HAS-A next Handler
Interpreter: Expression HAS-A sub-expressions
