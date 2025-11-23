# Elevator System - Low-Level Design

## Problem Statement

Design an elevator control system for a multi-floor building that can:
- Handle multiple elevators
- Process requests from different floors
- Optimize elevator movement to minimize wait time
- Support both internal (from inside elevator) and external (from floor) requests
- Handle edge cases like overweight, maintenance mode, emergency stop

## Requirements

### Functional Requirements
1. Multiple elevators in a building
2. Each elevator can move up and down
3. Internal and external requests
4. Display current floor and direction
5. Door open/close mechanism
6. Weight limit enforcement
7. Emergency stop functionality
8. Maintenance mode

### Non-Functional Requirements
1. Minimize average wait time
2. Energy efficiency
3. Fair distribution of requests
4. Thread-safe operations
5. Fault tolerance

## Core Components

### Enums and Constants

```java
// Direction of elevator movement
public enum Direction {
    UP,
    DOWN,
    IDLE
}

// Status of elevator
public enum ElevatorStatus {
    IDLE,
    MOVING,
    STOPPED,
    MAINTENANCE,
    EMERGENCY
}

// Door status
public enum DoorStatus {
    OPEN,
    CLOSED,
    OPENING,
    CLOSING
}

// Request type
public enum RequestType {
    INTERNAL,  // From inside elevator
    EXTERNAL   // From floor
}
```

## Class Design

### 1. Request Class

```java
public class Request {
    private int floor;
    private Direction direction;
    private RequestType type;
    private long timestamp;
    
    public Request(int floor, Direction direction, RequestType type) {
        this.floor = floor;
        this.direction = direction;
        this.type = type;
        this.timestamp = System.currentTimeMillis();
    }
    
    public int getFloor() { return floor; }
    public Direction getDirection() { return direction; }
    public RequestType getType() { return type; }
    public long getTimestamp() { return timestamp; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Request request = (Request) o;
        return floor == request.floor && direction == request.direction;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(floor, direction);
    }
    
    @Override
    public String toString() {
        return "Request{floor=" + floor + ", direction=" + direction + ", type=" + type + "}";
    }
}
```

### 2. Button Classes

```java
// Internal button (inside elevator)
public class InternalButton {
    private int floor;
    private boolean pressed;
    
    public InternalButton(int floor) {
        this.floor = floor;
        this.pressed = false;
    }
    
    public void press() {
        this.pressed = true;
        System.out.println("Internal button pressed for floor: " + floor);
    }
    
    public void reset() {
        this.pressed = false;
    }
    
    public boolean isPressed() { return pressed; }
    public int getFloor() { return floor; }
}

// External button (at each floor)
public class ExternalButton {
    private int floor;
    private Direction direction;
    private boolean pressed;
    
    public ExternalButton(int floor, Direction direction) {
        this.floor = floor;
        this.direction = direction;
        this.pressed = false;
    }
    
    public void press() {
        this.pressed = true;
        System.out.println("External button pressed at floor " + floor + " going " + direction);
    }
    
    public void reset() {
        this.pressed = false;
    }
    
    public boolean isPressed() { return pressed; }
    public int getFloor() { return floor; }
    public Direction getDirection() { return direction; }
}
```

### 3. Door Class

```java
public class Door {
    private DoorStatus status;
    
    public Door() {
        this.status = DoorStatus.CLOSED;
    }
    
    public void open() {
        if (status == DoorStatus.CLOSED) {
            System.out.println("Door opening...");
            status = DoorStatus.OPENING;
            // Simulate door opening time
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            status = DoorStatus.OPEN;
            System.out.println("Door opened");
        }
    }
    
    public void close() {
        if (status == DoorStatus.OPEN) {
            System.out.println("Door closing...");
            status = DoorStatus.CLOSING;
            // Simulate door closing time
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            status = DoorStatus.CLOSED;
            System.out.println("Door closed");
        }
    }
    
    public boolean isOpen() {
        return status == DoorStatus.OPEN;
    }
    
    public DoorStatus getStatus() { return status; }
}
```

### 4. Display Class

```java
public class Display {
    private int currentFloor;
    private Direction direction;
    
    public Display() {
        this.currentFloor = 0;
        this.direction = Direction.IDLE;
    }
    
    public void update(int floor, Direction direction) {
        this.currentFloor = floor;
        this.direction = direction;
        show();
    }
    
    private void show() {
        System.out.println("Display: Floor " + currentFloor + " | Direction: " + direction);
    }
    
    public int getCurrentFloor() { return currentFloor; }
    public Direction getDirection() { return direction; }
}
```

### 5. Elevator Class

```java
public class Elevator implements Runnable {
    private int id;
    private int currentFloor;
    private Direction currentDirection;
    private ElevatorStatus status;
    private Door door;
    private Display display;
    private int capacity;
    private int currentWeight;
    private static final int MAX_WEIGHT = 1000; // kg
    
    private Set<Integer> upStops;
    private Set<Integer> downStops;
    private PriorityQueue<Integer> upQueue;
    private PriorityQueue<Integer> downQueue;
    
    public Elevator(int id, int capacity) {
        this.id = id;
        this.currentFloor = 0;
        this.currentDirection = Direction.IDLE;
        this.status = ElevatorStatus.IDLE;
        this.door = new Door();
        this.display = new Display();
        this.capacity = capacity;
        this.currentWeight = 0;
        
        this.upStops = new HashSet<>();
        this.downStops = new HashSet<>();
        this.upQueue = new PriorityQueue<>(); // Min heap for going up
        this.downQueue = new PriorityQueue<>(Collections.reverseOrder()); // Max heap for going down
    }
    
    public synchronized void addRequest(Request request) {
        int targetFloor = request.getFloor();
        
        if (targetFloor == currentFloor) {
            openDoor();
            closeDoor();
            return;
        }
        
        if (targetFloor > currentFloor) {
            upStops.add(targetFloor);
            upQueue.offer(targetFloor);
        } else {
            downStops.add(targetFloor);
            downQueue.offer(targetFloor);
        }
        
        System.out.println("Elevator " + id + " received request: " + request);
    }
    
    @Override
    public void run() {
        while (true) {
            try {
                processRequests();
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    
    private synchronized void processRequests() {
        if (status == ElevatorStatus.MAINTENANCE || status == ElevatorStatus.EMERGENCY) {
            return;
        }
        
        if (currentDirection == Direction.IDLE) {
            if (!upQueue.isEmpty()) {
                currentDirection = Direction.UP;
            } else if (!downQueue.isEmpty()) {
                currentDirection = Direction.DOWN;
            }
        }
        
        if (currentDirection == Direction.UP) {
            processUpRequests();
        } else if (currentDirection == Direction.DOWN) {
            processDownRequests();
        }
        
        display.update(currentFloor, currentDirection);
    }
    
    private void processUpRequests() {
        if (upQueue.isEmpty()) {
            currentDirection = Direction.IDLE;
            return;
        }
        
        int nextFloor = upQueue.peek();
        
        if (nextFloor == currentFloor) {
            upQueue.poll();
            upStops.remove(currentFloor);
            stop();
        } else if (nextFloor > currentFloor) {
            moveUp();
            if (upStops.contains(currentFloor)) {
                upStops.remove(currentFloor);
                stop();
            }
        }
        
        if (upQueue.isEmpty()) {
            currentDirection = Direction.IDLE;
        }
    }
    
    private void processDownRequests() {
        if (downQueue.isEmpty()) {
            currentDirection = Direction.IDLE;
            return;
        }
        
        int nextFloor = downQueue.peek();
        
        if (nextFloor == currentFloor) {
            downQueue.poll();
            downStops.remove(currentFloor);
            stop();
        } else if (nextFloor < currentFloor) {
            moveDown();
            if (downStops.contains(currentFloor)) {
                downStops.remove(currentFloor);
                stop();
            }
        }
        
        if (downQueue.isEmpty()) {
            currentDirection = Direction.IDLE;
        }
    }
    
    private void moveUp() {
        status = ElevatorStatus.MOVING;
        System.out.println("Elevator " + id + " moving up from floor " + currentFloor);
        try {
            Thread.sleep(2000); // Simulate travel time
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        currentFloor++;
        System.out.println("Elevator " + id + " reached floor " + currentFloor);
    }
    
    private void moveDown() {
        status = ElevatorStatus.MOVING;
        System.out.println("Elevator " + id + " moving down from floor " + currentFloor);
        try {
            Thread.sleep(2000); // Simulate travel time
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        currentFloor--;
        System.out.println("Elevator " + id + " reached floor " + currentFloor);
    }
    
    private void stop() {
        status = ElevatorStatus.STOPPED;
        System.out.println("Elevator " + id + " stopped at floor " + currentFloor);
        openDoor();
        try {
            Thread.sleep(3000); // Wait for passengers
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        closeDoor();
        status = ElevatorStatus.IDLE;
    }
    
    private void openDoor() {
        door.open();
    }
    
    private void closeDoor() {
        door.close();
    }
    
    public void emergencyStop() {
        status = ElevatorStatus.EMERGENCY;
        System.out.println("EMERGENCY STOP - Elevator " + id);
    }
    
    public void setMaintenanceMode(boolean maintenance) {
        if (maintenance) {
            status = ElevatorStatus.MAINTENANCE;
            System.out.println("Elevator " + id + " in maintenance mode");
        } else {
            status = ElevatorStatus.IDLE;
            System.out.println("Elevator " + id + " back to normal operation");
        }
    }
    
    public boolean isOverweight() {
        return currentWeight > MAX_WEIGHT;
    }
    
    public void addWeight(int weight) {
        currentWeight += weight;
        if (isOverweight()) {
            System.out.println("WARNING: Elevator " + id + " is overweight!");
        }
    }
    
    public void removeWeight(int weight) {
        currentWeight -= weight;
    }
    
    // Getters
    public int getId() { return id; }
    public int getCurrentFloor() { return currentFloor; }
    public Direction getCurrentDirection() { return currentDirection; }
    public ElevatorStatus getStatus() { return status; }
    public boolean isIdle() { return status == ElevatorStatus.IDLE && currentDirection == Direction.IDLE; }
    public int getRequestCount() { return upQueue.size() + downQueue.size(); }
}
```

### 6. Elevator Controller (Strategy Pattern)

```java
// Strategy interface for elevator selection
public interface ElevatorSelectionStrategy {
    Elevator selectElevator(List<Elevator> elevators, Request request);
}

// Nearest elevator strategy
public class NearestElevatorStrategy implements ElevatorSelectionStrategy {
    @Override
    public Elevator selectElevator(List<Elevator> elevators, Request request) {
        Elevator nearestElevator = null;
        int minDistance = Integer.MAX_VALUE;
        
        for (Elevator elevator : elevators) {
            if (elevator.getStatus() == ElevatorStatus.MAINTENANCE || 
                elevator.getStatus() == ElevatorStatus.EMERGENCY) {
                continue;
            }
            
            int distance = Math.abs(elevator.getCurrentFloor() - request.getFloor());
            
            if (distance < minDistance) {
                minDistance = distance;
                nearestElevator = elevator;
            }
        }
        
        return nearestElevator;
    }
}

// Least loaded strategy
public class LeastLoadedStrategy implements ElevatorSelectionStrategy {
    @Override
    public Elevator selectElevator(List<Elevator> elevators, Request request) {
        Elevator leastLoaded = null;
        int minRequests = Integer.MAX_VALUE;
        
        for (Elevator elevator : elevators) {
            if (elevator.getStatus() == ElevatorStatus.MAINTENANCE || 
                elevator.getStatus() == ElevatorStatus.EMERGENCY) {
                continue;
            }
            
            int requestCount = elevator.getRequestCount();
            
            if (requestCount < minRequests) {
                minRequests = requestCount;
                leastLoaded = elevator;
            }
        }
        
        return leastLoaded;
    }
}
```

### 7. Elevator Controller

```java
public class ElevatorController {
    private List<Elevator> elevators;
    private ElevatorSelectionStrategy strategy;
    private int minFloor;
    private int maxFloor;
    
    public ElevatorController(int numberOfElevators, int minFloor, int maxFloor) {
        this.elevators = new ArrayList<>();
        this.minFloor = minFloor;
        this.maxFloor = maxFloor;
        this.strategy = new NearestElevatorStrategy(); // Default strategy
        
        // Initialize elevators
        for (int i = 0; i < numberOfElevators; i++) {
            Elevator elevator = new Elevator(i + 1, 10); // 10 person capacity
            elevators.add(elevator);
            new Thread(elevator).start(); // Start elevator thread
        }
        
        System.out.println("Elevator system initialized with " + numberOfElevators + " elevators");
    }
    
    public void setStrategy(ElevatorSelectionStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void requestElevator(int floor, Direction direction) {
        if (floor < minFloor || floor > maxFloor) {
            System.out.println("Invalid floor: " + floor);
            return;
        }
        
        Request request = new Request(floor, direction, RequestType.EXTERNAL);
        Elevator selectedElevator = strategy.selectElevator(elevators, request);
        
        if (selectedElevator != null) {
            System.out.println("Assigned Elevator " + selectedElevator.getId() + " to request: " + request);
            selectedElevator.addRequest(request);
        } else {
            System.out.println("No elevator available for request: " + request);
        }
    }
    
    public void goToFloor(int elevatorId, int targetFloor) {
        if (elevatorId < 1 || elevatorId > elevators.size()) {
            System.out.println("Invalid elevator ID: " + elevatorId);
            return;
        }
        
        if (targetFloor < minFloor || targetFloor > maxFloor) {
            System.out.println("Invalid floor: " + targetFloor);
            return;
        }
        
        Elevator elevator = elevators.get(elevatorId - 1);
        Direction direction = targetFloor > elevator.getCurrentFloor() ? Direction.UP : Direction.DOWN;
        Request request = new Request(targetFloor, direction, RequestType.INTERNAL);
        elevator.addRequest(request);
    }
    
    public void emergencyStop(int elevatorId) {
        if (elevatorId < 1 || elevatorId > elevators.size()) {
            System.out.println("Invalid elevator ID: " + elevatorId);
            return;
        }
        elevators.get(elevatorId - 1).emergencyStop();
    }
    
    public void displayStatus() {
        System.out.println("\n=== Elevator System Status ===");
        for (Elevator elevator : elevators) {
            System.out.println("Elevator " + elevator.getId() + 
                             " | Floor: " + elevator.getCurrentFloor() + 
                             " | Direction: " + elevator.getCurrentDirection() + 
                             " | Status: " + elevator.getStatus() + 
                             " | Pending: " + elevator.getRequestCount());
        }
        System.out.println("==============================\n");
    }
}
```

### 8. Building Class

```java
public class Building {
    private String name;
    private int numberOfFloors;
    private ElevatorController controller;
    
    public Building(String name, int numberOfFloors, int numberOfElevators) {
        this.name = name;
        this.numberOfFloors = numberOfFloors;
        this.controller = new ElevatorController(numberOfElevators, 0, numberOfFloors - 1);
    }
    
    public ElevatorController getController() {
        return controller;
    }
    
    public String getName() { return name; }
    public int getNumberOfFloors() { return numberOfFloors; }
}
```

### 9. Demo Application

```java
public class ElevatorSystemDemo {
    public static void main(String[] args) throws InterruptedException {
        // Create a building with 10 floors and 3 elevators
        Building building = new Building("Tech Tower", 10, 3);
        ElevatorController controller = building.getController();
        
        System.out.println("\n=== Starting Elevator System Demo ===\n");
        
        // Simulate external requests (from floors)
        System.out.println("--- External Requests ---");
        controller.requestElevator(5, Direction.UP);
        Thread.sleep(1000);
        
        controller.requestElevator(2, Direction.UP);
        Thread.sleep(1000);
        
        controller.requestElevator(8, Direction.DOWN);
        Thread.sleep(2000);
        
        // Display status
        controller.displayStatus();
        
        // Simulate internal requests (from inside elevator)
        System.out.println("\n--- Internal Requests ---");
        controller.goToFloor(1, 7);
        Thread.sleep(1000);
        
        controller.goToFloor(2, 4);
        Thread.sleep(1000);
        
        controller.goToFloor(3, 9);
        Thread.sleep(2000);
        
        // Display status
        controller.displayStatus();
        
        // Wait for elevators to process requests
        Thread.sleep(20000);
        
        // Final status
        controller.displayStatus();
        
        // Test with different strategy
        System.out.println("\n--- Changing to Least Loaded Strategy ---");
        controller.setStrategy(new LeastLoadedStrategy());
        controller.requestElevator(6, Direction.UP);
        
        Thread.sleep(5000);
        controller.displayStatus();
        
        System.out.println("\n=== Demo Complete ===");
        System.exit(0);
    }
}
```

## Key Design Patterns Used

1. **Strategy Pattern**: Different elevator selection strategies
2. **State Pattern**: Elevator states (IDLE, MOVING, STOPPED, etc.)
3. **Observer Pattern**: Could notify displays about elevator status
4. **Singleton Pattern**: Could be used for ElevatorController
5. **Command Pattern**: Could encapsulate requests as command objects

## SOLID Principles Applied

1. **SRP**: Each class has a single responsibility
2. **OCP**: Easy to add new selection strategies
3. **LSP**: Different strategies can be substituted
4. **ISP**: Focused interfaces
5. **DIP**: Controller depends on strategy abstraction

## Optimization Strategies

### 1. SCAN Algorithm (Elevator Algorithm)
- Elevator continues in same direction until no more requests
- Then reverses direction

### 2. LOOK Algorithm
- Similar to SCAN but doesn't go to extreme ends
- Reverses when no more requests in current direction

### 3. Destination Control System
- Users input destination before entering
- System optimizes grouping of passengers

## Possible Enhancements

1. VIP/Priority requests
2. Energy-saving mode
3. Load balancing
4. Predictive algorithms (ML-based)
5. Integration with building security
6. Real-time monitoring dashboard
7. Maintenance scheduling
8. Historical data analytics
9. Multi-building support
10. Mobile app integration

## Testing Scenarios

1. Single elevator, multiple requests
2. Multiple elevators, concurrent requests
3. Peak hour simulation
4. Emergency scenarios
5. Maintenance mode handling
6. Overweight scenarios
7. Edge cases (ground floor, top floor)

---

[Back to Examples](../../README.md#system-design-examples)
