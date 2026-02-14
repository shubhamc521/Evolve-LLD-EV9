# UML Class Diagrams - LLD Systems

## 1. Event Bus System

### Class Diagram

```mermaid
classDiagram
    class Event {
        -EventID id
        -String name
        -Map~String, String~ attributes
        -Timestamp timestamp
        +Event(id, name, attributes, timestamp)
        +getId() EventID
        +getName() String
        +getAttributes() Map
        +getTimestamp() Timestamp
        +toString() String
    }

    class FailureEvent {
        -Throwable error
        -Timestamp timestamp
        +FailureEvent(event, error, timestamp)
    }

    class Subscription {
        -Topic topicId
        -EntityID subscriberId
        -SubscriptionType type
        -Function~Event, Boolean~ precondition
        -Function~Event, Void~ handler
        +Subscription(topicId, subscriberId, type, precondition, handler)
        +handler() Function
        +getType() SubscriptionType
        +getTopicId() Topic
        +getSubscriberId() EntityID
    }

    class EventBus {
        -KeyedExecutor executor
        -Map~Topic, List~Event~~ buses
        -Map~Topic, Set~Subscription~~ subscriptions
        -Map~Topic, Map~EntityID, Index~~ subscriberIndexes
        -Map~Topic, Map~EventID, Index~~ eventIndex
        -Map~Topic, ConcurrentSkipListMap~Timestamp, Index~~ timestampIndex
        -RetryAlgorithm retryAlgorithm
        -EventBus deadLetterQueue
        -Timer timer
        +EventBus(threads, retryAlgorithm, deadLetterQueue, timer)
        +publish(topic, event) CompletionStage
        +poll(topic, subscriber) CompletionStage
        +registerTopic(topic) void
        +subscribe(subscription) CompletionStage
        +setIndexAfterTimestamp(topic, subscriber, timestamp) CompletionStage
        +setIndexAfterEvent(topic, subscriber, eventId) CompletionStage
        +getEvent(topic, eventId) CompletionStage
        -addEventToBus(topic, event) void
        -push(event, subscription) void
    }

    class KeyedExecutor {
        -Executor[] executor
        +KeyedExecutor(threads)
        +submit(id, task) CompletionStage
        +get(id, task) CompletionStage
    }

    class RetryAlgorithm {
        -int maxAttempts
        -Function~Integer, Long~ retryTimeCalculator
        +RetryAlgorithm(maxAttempts, retryTimeCalculator)
        +attempt(task, parameter, attempts) RESULT
    }

    class Index {
        -int val
        +Index(val)
        +increment() Index
        +getVal() int
    }

    class Topic {
        -String name
        +getName() String
    }

    class EntityID {
        -String id
        +getId() String
    }

    class EventID {
        -String id
        +EventID(id)
    }

    class Timestamp {
        -long time
    }

    class SubscriptionType {
        <<enumeration>>
        PUSH
        PULL
    }

    FailureEvent --|> Event
    EventBus o-- KeyedExecutor
    EventBus o-- RetryAlgorithm
    EventBus o-- EventBus : deadLetterQueue
    EventBus --> Event
    EventBus --> Subscription
    EventBus --> Topic
    EventBus --> Index
    Subscription --> Topic
    Subscription --> EntityID
    Subscription --> SubscriptionType
    Subscription --> Event
    Event --> EventID
    Event --> Timestamp
```

### Sequence Diagram - Publish Event Flow

```mermaid
sequenceDiagram
    participant P as Publisher
    participant EB as EventBus
    participant KE as KeyedExecutor
    participant T as Topic Queue
    participant S as PUSH Subscriber
    
    P->>EB: publish(topic, event)
    EB->>KE: submit(topic.name, task)
    KE->>EB: addEventToBus(topic, event)
    EB->>T: Add event to queue
    EB->>T: Update timestamp index
    EB->>T: Update event ID index
    EB->>S: push(event, subscription)
    S->>S: handler.apply(event)
    alt Success
        S-->>EB: ACK
    else Failure (RetryAble)
        S-->>EB: Error
        EB->>EB: Retry with backoff
        alt Max Retry Exceeded
            EB->>EB: Send to DLQ
        end
    end
```

### Sequence Diagram - Poll Event Flow

```mermaid
sequenceDiagram
    participant S as PULL Subscriber
    participant EB as EventBus
    participant KE as KeyedExecutor
    participant T as Topic Queue
    
    S->>EB: poll(topic, subscriberId)
    EB->>KE: get(topic + subscriber, task)
    KE->>EB: Execute on specific thread
    EB->>EB: Get current index for subscriber
    EB->>T: Get event at index
    T-->>EB: Event
    EB->>EB: Increment subscriber index
    EB-->>S: Event
```

---

## 2. Parking Lot System

### Class Diagram

```mermaid
classDiagram
    class ParkingLot {
        -String name
        -Address address
        -List~ParkingFloor~ floors
        -int maxFloors
        +ParkingLot(name, address, maxFloors)
        +addFloor(floor) boolean
        +removeFloor(floor) boolean
        +getAvailableSpot(vehicleType) ParkingSpot
        +parkVehicle(vehicle) ParkingTicket
        +unparkVehicle(ticket) boolean
    }

    class ParkingFloor {
        -int floorNumber
        -List~ParkingSpot~ spots
        -Map~VehicleType, List~ParkingSpot~~ spotsByType
        +ParkingFloor(floorNumber)
        +addSpot(spot) boolean
        +getAvailableSpot(vehicleType) ParkingSpot
        +getFreeSpotCount(vehicleType) int
    }

    class ParkingSpot {
        -String spotId
        -VehicleType type
        -SpotStatus status
        -Vehicle vehicle
        +ParkingSpot(spotId, type)
        +assignVehicle(vehicle) boolean
        +removeVehicle() boolean
        +isAvailable() boolean
        +getType() VehicleType
    }

    class Vehicle {
        -String licensePlate
        -VehicleType type
        -String color
        +Vehicle(licensePlate, type, color)
        +getLicensePlate() String
        +getType() VehicleType
    }

    class ParkingTicket {
        -String ticketId
        -Vehicle vehicle
        -ParkingSpot spot
        -Timestamp entryTime
        -Timestamp exitTime
        -double fee
        +ParkingTicket(vehicle, spot, entryTime)
        +calculateFee(exitTime) double
        +getTicketId() String
    }

    class PaymentProcessor {
        +processPayment(ticket, amount, method) Payment
        +validatePayment(payment) boolean
    }

    class Payment {
        -String paymentId
        -double amount
        -PaymentMethod method
        -PaymentStatus status
        -Timestamp timestamp
        +Payment(amount, method)
        +getStatus() PaymentStatus
    }

    class Address {
        -String street
        -String city
        -String state
        -String zipCode
        +Address(street, city, state, zipCode)
    }

    class VehicleType {
        <<enumeration>>
        CAR
        BIKE
        TRUCK
        VAN
    }

    class SpotStatus {
        <<enumeration>>
        FREE
        OCCUPIED
        RESERVED
    }

    class PaymentMethod {
        <<enumeration>>
        CASH
        CREDIT_CARD
        UPI
    }

    class PaymentStatus {
        <<enumeration>>
        PENDING
        COMPLETED
        FAILED
    }

    ParkingLot "1" *-- "many" ParkingFloor
    ParkingLot --> Address
    ParkingFloor "1" *-- "many" ParkingSpot
    ParkingSpot --> Vehicle
    ParkingSpot --> VehicleType
    ParkingSpot --> SpotStatus
    Vehicle --> VehicleType
    ParkingTicket --> Vehicle
    ParkingTicket --> ParkingSpot
    PaymentProcessor --> Payment
    Payment --> PaymentMethod
    Payment --> PaymentStatus
```

### Sequence Diagram - Park Vehicle

```mermaid
sequenceDiagram
    participant D as Driver
    participant PL as ParkingLot
    participant PF as ParkingFloor
    participant PS as ParkingSpot
    participant T as TicketSystem
    
    D->>PL: parkVehicle(vehicle)
    PL->>PL: getAvailableSpot(vehicleType)
    loop For each floor
        PL->>PF: getAvailableSpot(vehicleType)
        PF->>PS: Check if available
        alt Spot available
            PS-->>PF: ParkingSpot
            PF-->>PL: ParkingSpot
        end
    end
    PL->>PS: assignVehicle(vehicle)
    PS->>PS: status = OCCUPIED
    PL->>T: Create ParkingTicket
    T-->>PL: ParkingTicket
    PL-->>D: ParkingTicket
```

### Sequence Diagram - Unpark & Payment

```mermaid
sequenceDiagram
    participant D as Driver
    participant PL as ParkingLot
    participant T as TicketSystem
    participant PS as ParkingSpot
    participant PP as PaymentProcessor
    
    D->>PL: unparkVehicle(ticket)
    PL->>T: calculateFee(exitTime)
    T-->>PL: fee
    PL->>PP: processPayment(ticket, fee, method)
    PP->>PP: validatePayment()
    alt Payment Success
        PP-->>PL: Payment(COMPLETED)
        PL->>PS: removeVehicle()
        PS->>PS: status = FREE
        PL-->>D: Success
    else Payment Failed
        PP-->>PL: Payment(FAILED)
        PL-->>D: Payment Failed
    end
```

---

## 3. Rate Limiter System

### Class Diagram

```mermaid
classDiagram
    class RateLimiter {
        <<interface>>
        +allowRequest(userId) boolean
    }

    class TokenBucketLimiter {
        -int capacity
        -int refillRate
        -Map~String, Bucket~ userBuckets
        +TokenBucketLimiter(capacity, refillRate)
        +allowRequest(userId) boolean
        -refillBucket(bucket) void
    }

    class LeakyBucketLimiter {
        -int capacity
        -int leakRate
        -Map~String, Queue~ userQueues
        +LeakyBucketLimiter(capacity, leakRate)
        +allowRequest(userId) boolean
        -processQueue(queue) void
    }

    class SlidingWindowLimiter {
        -int windowSize
        -int maxRequests
        -Map~String, List~Timestamp~~ userRequests
        +SlidingWindowLimiter(windowSize, maxRequests)
        +allowRequest(userId) boolean
        -cleanOldRequests(userId) void
    }

    class FixedWindowLimiter {
        -int windowSize
        -int maxRequests
        -Map~String, WindowCounter~ userCounters
        +FixedWindowLimiter(windowSize, maxRequests)
        +allowRequest(userId) boolean
        -resetWindow(counter) void
    }

    class Bucket {
        -int tokens
        -Timestamp lastRefillTime
        +Bucket(capacity)
        +consumeToken() boolean
        +refill(rate, currentTime) void
        +getTokens() int
    }

    class WindowCounter {
        -int count
        -Timestamp windowStart
        +WindowCounter()
        +increment() void
        +reset() void
        +getCount() int
        +isExpired(windowSize) boolean
    }

    class Request {
        -String userId
        -String endpoint
        -Timestamp timestamp
        +Request(userId, endpoint)
        +getUserId() String
        +getTimestamp() Timestamp
    }

    class RateLimiterFactory {
        +createLimiter(type, config) RateLimiter
    }

    class RateLimiterType {
        <<enumeration>>
        TOKEN_BUCKET
        LEAKY_BUCKET
        SLIDING_WINDOW
        FIXED_WINDOW
    }

    class Config {
        -int capacity
        -int rate
        -int windowSize
        -int maxRequests
        +Config(params)
    }

    RateLimiter <|.. TokenBucketLimiter
    RateLimiter <|.. LeakyBucketLimiter
    RateLimiter <|.. SlidingWindowLimiter
    RateLimiter <|.. FixedWindowLimiter
    TokenBucketLimiter o-- Bucket
    FixedWindowLimiter o-- WindowCounter
    RateLimiterFactory --> RateLimiter
    RateLimiterFactory --> RateLimiterType
    RateLimiterFactory --> Config
    TokenBucketLimiter --> Request
    LeakyBucketLimiter --> Request
```

### Sequence Diagram - Token Bucket Algorithm

```mermaid
sequenceDiagram
    participant C as Client
    participant RL as RateLimiter
    participant TB as TokenBucket
    participant B as Bucket
    
    C->>RL: allowRequest(userId)
    RL->>TB: Check user bucket
    alt Bucket exists
        TB->>B: Get bucket
    else No bucket
        TB->>B: Create new bucket
    end
    B->>B: refill(rate, currentTime)
    B->>B: Calculate tokens to add
    TB->>B: consumeToken()
    alt Tokens available
        B->>B: tokens--
        B-->>TB: true
        TB-->>RL: true
        RL-->>C: Allow request
    else No tokens
        B-->>TB: false
        TB-->>RL: false
        RL-->>C: Rate limit exceeded
    end
```

### Sequence Diagram - Sliding Window Algorithm

```mermaid
sequenceDiagram
    participant C as Client
    participant RL as RateLimiter
    participant SW as SlidingWindow
    
    C->>RL: allowRequest(userId)
    RL->>SW: Check request count
    SW->>SW: cleanOldRequests(userId)
    SW->>SW: Remove timestamps outside window
    SW->>SW: Get current count
    alt Count < maxRequests
        SW->>SW: Add current timestamp
        SW-->>RL: true
        RL-->>C: Allow request
    else Count >= maxRequests
        SW-->>RL: false
        RL-->>C: Rate limit exceeded (429)
    end
```

### Algorithm Comparison

| Algorithm | Pros | Cons | Use Case |
|-----------|------|------|----------|
| **Token Bucket** | Smooth traffic, allows bursts | Complex implementation | API rate limiting with burst handling |
| **Leaky Bucket** | Consistent output rate | Strict, no burst support | Network traffic shaping |
| **Fixed Window** | Simple, memory efficient | Edge case issues (spike at boundary) | Basic rate limiting |
| **Sliding Window** | Accurate, no boundary issues | Higher memory usage | Precise rate limiting needs |

---

## Key Design Patterns Used

### Event Bus
- **Observer Pattern**: PUSH subscribers get notified automatically
- **Publisher-Subscriber Pattern**: Decoupled communication
- **Command Pattern**: Events as commands
- **Strategy Pattern**: Different retry algorithms
- **Thread Pool Pattern**: KeyedExecutor for ordering

### Parking Lot
- **Singleton Pattern**: ParkingLot instance
- **Factory Pattern**: Creating different vehicle types
- **Strategy Pattern**: Different payment methods
- **State Pattern**: ParkingSpot status transitions

### Rate Limiter
- **Strategy Pattern**: Different rate limiting algorithms
- **Factory Pattern**: RateLimiterFactory
- **Singleton Pattern**: Rate limiter instances per user
- **Template Method**: Common rate limiting flow

---

## SOLID Principles Applied

### Single Responsibility
- Each class has one reason to change
- Event handles only event data
- EventBus handles only routing logic

### Open/Closed
- Rate limiter algorithms can be extended without modification
- New vehicle types can be added easily

### Liskov Substitution
- All RateLimiter implementations are interchangeable
- Vehicle subtypes can replace parent type

### Interface Segregation
- Small, focused interfaces (RateLimiter)
- Clients depend only on methods they use

### Dependency Inversion
- High-level modules depend on abstractions (RateLimiter interface)
- EventBus depends on RetryAlgorithm abstraction
