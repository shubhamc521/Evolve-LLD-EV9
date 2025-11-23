# Evolve-LLD-EV9: Low-Level Design Patterns & Principles

A comprehensive guide to Low-Level Design (LLD) patterns, principles, and best practices with detailed explanations and practical examples.

## 📚 Table of Contents

1. [Introduction](#introduction)
2. [SOLID Principles](#solid-principles)
3. [Design Patterns](#design-patterns)
   - [Creational Patterns](#creational-patterns)
   - [Structural Patterns](#structural-patterns)
   - [Behavioral Patterns](#behavioral-patterns)
4. [System Design Examples](#system-design-examples)
5. [Best Practices](#best-practices)

## Introduction

Low-Level Design (LLD) focuses on the detailed design of individual components and modules in a software system. This repository provides comprehensive notes and explanations for various design patterns, SOLID principles, and practical implementation examples.

## SOLID Principles

The SOLID principles are five design principles that make software designs more understandable, flexible, and maintainable.

- **[S]ingle Responsibility Principle (SRP)** - [Learn More](./docs/solid/single-responsibility.md)
- **[O]pen/Closed Principle (OCP)** - [Learn More](./docs/solid/open-closed.md)
- **[L]iskov Substitution Principle (LSP)** - [Learn More](./docs/solid/liskov-substitution.md)
- **[I]nterface Segregation Principle (ISP)** - [Learn More](./docs/solid/interface-segregation.md)
- **[D]ependency Inversion Principle (DIP)** - [Learn More](./docs/solid/dependency-inversion.md)

## Design Patterns

### Creational Patterns

Patterns that deal with object creation mechanisms:

- **[Singleton Pattern](./docs/patterns/creational/singleton.md)** - Ensures a class has only one instance
- **[Factory Pattern](./docs/patterns/creational/factory.md)** - Creates objects without specifying exact classes
- **[Abstract Factory Pattern](./docs/patterns/creational/abstract-factory.md)** - Creates families of related objects
- **[Builder Pattern](./docs/patterns/creational/builder.md)** - Constructs complex objects step by step
- **[Prototype Pattern](./docs/patterns/creational/prototype.md)** - Creates objects by cloning existing ones

### Structural Patterns

Patterns that deal with object composition:

- **[Adapter Pattern](./docs/patterns/structural/adapter.md)** - Allows incompatible interfaces to work together
- **[Decorator Pattern](./docs/patterns/structural/decorator.md)** - Adds behavior to objects dynamically
- **[Facade Pattern](./docs/patterns/structural/facade.md)** - Provides a simplified interface to a complex system
- **[Proxy Pattern](./docs/patterns/structural/proxy.md)** - Controls access to objects
- **[Composite Pattern](./docs/patterns/structural/composite.md)** - Composes objects into tree structures

### Behavioral Patterns

Patterns that deal with object collaboration and responsibilities:

- **[Observer Pattern](./docs/patterns/behavioral/observer.md)** - Defines a one-to-many dependency between objects
- **[Strategy Pattern](./docs/patterns/behavioral/strategy.md)** - Defines a family of algorithms
- **[Command Pattern](./docs/patterns/behavioral/command.md)** - Encapsulates requests as objects
- **[State Pattern](./docs/patterns/behavioral/state.md)** - Allows an object to alter its behavior
- **[Chain of Responsibility](./docs/patterns/behavioral/chain-of-responsibility.md)** - Passes requests along a chain

## System Design Examples

Real-world LLD examples:

- **[Parking Lot System](./docs/examples/parking-lot.md)**
- **[Library Management System](./docs/examples/library-management.md)**
- **[Elevator System](./docs/examples/elevator-system.md)**
- **[Vending Machine](./docs/examples/vending-machine.md)**
- **[Chess Game](./docs/examples/chess-game.md)**

## Best Practices

- **[Code Organization](./docs/best-practices/code-organization.md)**
- **[Naming Conventions](./docs/best-practices/naming-conventions.md)**
- **[Error Handling](./docs/best-practices/error-handling.md)**
- **[Testing Strategies](./docs/best-practices/testing-strategies.md)**

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is open source and available under the MIT License.