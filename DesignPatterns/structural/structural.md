# What are Design Patterns?
- Solution to common coding problems.
- Ways to solve specifics problems.

## Types of Design Pattern:

- Creational - Control HOW objects are created
- Structural - Controls HOW objects are organized
- Behavioral - Control HOW objects communicate with each other

# Structural Design Patterns 

## What Are Structural Design Patterns?
- Structural patterns describe how to compose classes and objects to form larger, more flexible structures while keeping parts loosely coupled. 
- Instead of focusing on how objects are created (creational) or how they communicate/coordinate behavior (behavioral), structural patterns focus on how objects are arranged and related so that systems stay easy to extend and maintain.

How structural differs from the other families:
- Creational: how objects are instantiated (Factory, Builder, Singleton, etc.).
- Structural: how objects are hooked together (Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy, Mixin).
- Behavioral: how objects collaborate/communicate to do work (Strategy, Observer, Command, etc.)

## Side‑by‑Side Comparison (Cheat Sheet)

| Pattern | Core Idea | Main Relationship | Client Depends On | Typical Use | Changes Interface? | Adds Behavior? | Controls Access? | Simplifies Subsystem? | Shares State? |
|---|---|---|---|---|---:|---:|---:|---:|---:|
| Composite | Uniform tree handling | HAS‑A (composite contains component) | Component interface | File trees, UI trees, categories | No | Sometimes (via recursion) | No | No | No |
| Adapter | Translate API | IS‑A (implements target) + HAS‑A (adaptee) | Target interface | Integrate 3rd‑party libs | Yes (to target) | No | No | No | No |
| Bridge | Separate WHAT from HOW | HAS‑A bridge between hierarchies | Abstraction interface | Avoid N×M subclasses | No | In abstraction | No | No | No |
| Proxy | Same interface, control | IS‑A (same subject) + HAS‑A (real subject) | Subject interface | Lazy, security, caching | No | Maybe (logging) | Yes | No | No |
| Decorator | Wrap to extend | IS‑A (same component) + HAS‑A (wrapped) | Component interface | Optional features, formatting | No | Yes (stackable) | No | No | No |
| Facade | Unified entry point | HAS‑A many subsystems | Facade class | Orchestrate complex subsystems | No | Not its goal | No | Yes | No |
| Flyweight | Share intrinsic state | HAS‑A (context → flyweight), Factory cache | Flyweight via factory | Massive similar objects | No | No | No | No | Yes |
| Mixin | Reuse behavior bits | IS‑A to interfaces or HAS‑A helpers | Behavior interfaces | Cross‑cutting features | No | Yes | No | No | No |

---

## Choosing Guide (Ask These Questions)
- Do I need a tree of parts and wholes? → Composite
- Do I have an object whose interface doesn’t match what I need? → Adapter
- Do I have two dimensions that vary independently (WHAT vs HOW)? → Bridge
- Do I need to control access/lifecycle but keep the same interface? → Proxy
- Do I want to add features dynamically without subclassing? → Decorator
- Is the subsystem complex and I want a simple entry point? → Facade
- Do I have millions of similar objects with large shared state? → Flyweight
- Do different classes need the same behavior without deep inheritance? → Mixin