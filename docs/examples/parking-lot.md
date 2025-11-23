# Parking Lot System - Low-Level Design

## Problem Statement

Design a parking lot system that can:
- Have multiple floors with multiple parking spots per floor
- Support different types of vehicles (Car, Truck, Motorcycle, Van)
- Support different types of parking spots (Compact, Large, Handicapped, Motorcycle)
- Track which spots are available/occupied
- Generate tickets when vehicles enter
- Calculate parking fees when vehicles exit
- Support multiple entry/exit points

## Requirements

### Functional Requirements
1. Multiple floors with multiple spots per floor
2. Multiple entry and exit points
3. Support for different vehicle types
4. Assign appropriate spots based on vehicle type
5. Generate parking tickets
6. Calculate fees based on parking duration
7. Handle payment processing

### Non-Functional Requirements
1. System should be scalable
2. High availability
3. Real-time spot availability
4. Thread-safe for concurrent access

## Classes and Relationships

### Core Entities

```
ParkingLot (Singleton)
├── List<ParkingFloor>
├── List<EntrancePanel>
├── List<ExitPanel>
└── ParkingRate

ParkingFloor
├── floorNumber: int
└── Map<String, ParkingSpot>

ParkingSpot (Abstract)
├── spotNumber: String
├── spotType: SpotType
├── vehicle: Vehicle
└── isFree: boolean

Vehicle (Abstract)
├── licensePlate: String
├── vehicleType: VehicleType
└── ticket: ParkingTicket

ParkingTicket
├── ticketNumber: String
├── entryTime: Date
├── exitTime: Date
├── vehicle: Vehicle
├── spot: ParkingSpot
└── fee: double
```

## Implementation

### 1. Enums

```java
// Vehicle types
public enum VehicleType {
    CAR,
    TRUCK,
    MOTORCYCLE,
    VAN
}

// Parking spot types
public enum SpotType {
    COMPACT,
    LARGE,
    HANDICAPPED,
    MOTORCYCLE
}

// Parking spot status
public enum SpotStatus {
    AVAILABLE,
    OCCUPIED,
    RESERVED
}

// Ticket status
public enum TicketStatus {
    ACTIVE,
    PAID,
    LOST
}
```

### 2. Vehicle Classes

```java
// Abstract Vehicle class
public abstract class Vehicle {
    protected String licensePlate;
    protected VehicleType vehicleType;
    protected ParkingTicket ticket;
    
    public Vehicle(String licensePlate, VehicleType vehicleType) {
        this.licensePlate = licensePlate;
        this.vehicleType = vehicleType;
    }
    
    public String getLicensePlate() { return licensePlate; }
    public VehicleType getVehicleType() { return vehicleType; }
    public ParkingTicket getTicket() { return ticket; }
    public void setTicket(ParkingTicket ticket) { this.ticket = ticket; }
}

// Concrete vehicle classes
public class Car extends Vehicle {
    public Car(String licensePlate) {
        super(licensePlate, VehicleType.CAR);
    }
}

public class Truck extends Vehicle {
    public Truck(String licensePlate) {
        super(licensePlate, VehicleType.TRUCK);
    }
}

public class Motorcycle extends Vehicle {
    public Motorcycle(String licensePlate) {
        super(licensePlate, VehicleType.MOTORCYCLE);
    }
}

public class Van extends Vehicle {
    public Van(String licensePlate) {
        super(licensePlate, VehicleType.VAN);
    }
}
```

### 3. Parking Spot Classes

```java
// Abstract Parking Spot
public abstract class ParkingSpot {
    protected String spotNumber;
    protected SpotType spotType;
    protected Vehicle vehicle;
    protected SpotStatus status;
    
    public ParkingSpot(String spotNumber, SpotType spotType) {
        this.spotNumber = spotNumber;
        this.spotType = spotType;
        this.status = SpotStatus.AVAILABLE;
    }
    
    public synchronized boolean assignVehicle(Vehicle vehicle) {
        if (status == SpotStatus.AVAILABLE && canFitVehicle(vehicle)) {
            this.vehicle = vehicle;
            this.status = SpotStatus.OCCUPIED;
            return true;
        }
        return false;
    }
    
    public synchronized void removeVehicle() {
        this.vehicle = null;
        this.status = SpotStatus.AVAILABLE;
    }
    
    protected abstract boolean canFitVehicle(Vehicle vehicle);
    
    public boolean isAvailable() {
        return status == SpotStatus.AVAILABLE;
    }
    
    public String getSpotNumber() { return spotNumber; }
    public SpotType getSpotType() { return spotType; }
    public Vehicle getVehicle() { return vehicle; }
}

// Concrete spot classes
public class CompactSpot extends ParkingSpot {
    public CompactSpot(String spotNumber) {
        super(spotNumber, SpotType.COMPACT);
    }
    
    @Override
    protected boolean canFitVehicle(Vehicle vehicle) {
        return vehicle.getVehicleType() == VehicleType.CAR || 
               vehicle.getVehicleType() == VehicleType.MOTORCYCLE;
    }
}

public class LargeSpot extends ParkingSpot {
    public LargeSpot(String spotNumber) {
        super(spotNumber, SpotType.LARGE);
    }
    
    @Override
    protected boolean canFitVehicle(Vehicle vehicle) {
        return true; // Can fit any vehicle
    }
}

public class MotorcycleSpot extends ParkingSpot {
    public MotorcycleSpot(String spotNumber) {
        super(spotNumber, SpotType.MOTORCYCLE);
    }
    
    @Override
    protected boolean canFitVehicle(Vehicle vehicle) {
        return vehicle.getVehicleType() == VehicleType.MOTORCYCLE;
    }
}

public class HandicappedSpot extends ParkingSpot {
    public HandicappedSpot(String spotNumber) {
        super(spotNumber, SpotType.HANDICAPPED);
    }
    
    @Override
    protected boolean canFitVehicle(Vehicle vehicle) {
        return vehicle.getVehicleType() == VehicleType.CAR;
    }
}
```

### 4. Parking Floor

```java
public class ParkingFloor {
    private int floorNumber;
    private Map<String, ParkingSpot> spots;
    private Map<SpotType, List<ParkingSpot>> spotsByType;
    
    public ParkingFloor(int floorNumber) {
        this.floorNumber = floorNumber;
        this.spots = new HashMap<>();
        this.spotsByType = new HashMap<>();
        
        // Initialize spot type lists
        for (SpotType type : SpotType.values()) {
            spotsByType.put(type, new ArrayList<>());
        }
    }
    
    public void addSpot(ParkingSpot spot) {
        spots.put(spot.getSpotNumber(), spot);
        spotsByType.get(spot.getSpotType()).add(spot);
    }
    
    public ParkingSpot findAvailableSpot(Vehicle vehicle) {
        // Try to find appropriate spot based on vehicle type
        List<SpotType> preferredTypes = getPreferredSpotTypes(vehicle.getVehicleType());
        
        for (SpotType type : preferredTypes) {
            for (ParkingSpot spot : spotsByType.get(type)) {
                if (spot.isAvailable() && spot.assignVehicle(vehicle)) {
                    return spot;
                }
            }
        }
        return null;
    }
    
    private List<SpotType> getPreferredSpotTypes(VehicleType vehicleType) {
        List<SpotType> types = new ArrayList<>();
        switch (vehicleType) {
            case MOTORCYCLE:
                types.add(SpotType.MOTORCYCLE);
                types.add(SpotType.COMPACT);
                types.add(SpotType.LARGE);
                break;
            case CAR:
                types.add(SpotType.COMPACT);
                types.add(SpotType.LARGE);
                break;
            case TRUCK:
            case VAN:
                types.add(SpotType.LARGE);
                break;
        }
        return types;
    }
    
    public int getAvailableSpotCount() {
        return (int) spots.values().stream()
                         .filter(ParkingSpot::isAvailable)
                         .count();
    }
    
    public int getFloorNumber() { return floorNumber; }
}
```

### 5. Parking Ticket

```java
public class ParkingTicket {
    private String ticketNumber;
    private Date entryTime;
    private Date exitTime;
    private Vehicle vehicle;
    private ParkingSpot assignedSpot;
    private TicketStatus status;
    private double fee;
    
    public ParkingTicket(Vehicle vehicle, ParkingSpot spot) {
        this.ticketNumber = generateTicketNumber();
        this.entryTime = new Date();
        this.vehicle = vehicle;
        this.assignedSpot = spot;
        this.status = TicketStatus.ACTIVE;
        this.fee = 0.0;
    }
    
    private String generateTicketNumber() {
        return "TICKET-" + System.currentTimeMillis() + "-" + 
               (int)(Math.random() * 1000);
    }
    
    public void markExit() {
        this.exitTime = new Date();
    }
    
    public long getParkingDurationInHours() {
        Date end = (exitTime != null) ? exitTime : new Date();
        long durationMs = end.getTime() - entryTime.getTime();
        return (durationMs / (1000 * 60 * 60)) + 1; // Round up to nearest hour
    }
    
    // Getters and setters
    public String getTicketNumber() { return ticketNumber; }
    public Date getEntryTime() { return entryTime; }
    public Date getExitTime() { return exitTime; }
    public Vehicle getVehicle() { return vehicle; }
    public ParkingSpot getAssignedSpot() { return assignedSpot; }
    public TicketStatus getStatus() { return status; }
    public void setStatus(TicketStatus status) { this.status = status; }
    public double getFee() { return fee; }
    public void setFee(double fee) { this.fee = fee; }
}
```

### 6. Parking Rate

```java
public class ParkingRate {
    private static final Map<VehicleType, Double> hourlyRates = new HashMap<>();
    
    static {
        hourlyRates.put(VehicleType.MOTORCYCLE, 2.0);
        hourlyRates.put(VehicleType.CAR, 5.0);
        hourlyRates.put(VehicleType.VAN, 7.0);
        hourlyRates.put(VehicleType.TRUCK, 10.0);
    }
    
    public static double calculateFee(ParkingTicket ticket) {
        long hours = ticket.getParkingDurationInHours();
        double rate = hourlyRates.get(ticket.getVehicle().getVehicleType());
        return hours * rate;
    }
}
```

### 7. Payment Processing

```java
// Payment interface (Strategy pattern)
public interface PaymentStrategy {
    boolean processPayment(double amount);
}

public class CashPayment implements PaymentStrategy {
    @Override
    public boolean processPayment(double amount) {
        System.out.println("Processing cash payment of $" + amount);
        return true;
    }
}

public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    
    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    @Override
    public boolean processPayment(double amount) {
        System.out.println("Processing credit card payment of $" + amount);
        // Credit card processing logic
        return true;
    }
}

public class Payment {
    private PaymentStrategy strategy;
    
    public Payment(PaymentStrategy strategy) {
        this.strategy = strategy;
    }
    
    public boolean makePayment(double amount) {
        return strategy.processPayment(amount);
    }
}
```

### 8. Entry and Exit Panels

```java
public class EntrancePanel {
    private String panelId;
    
    public EntrancePanel(String panelId) {
        this.panelId = panelId;
    }
    
    public ParkingTicket issueTicket(Vehicle vehicle) {
        ParkingLot parkingLot = ParkingLot.getInstance();
        ParkingSpot spot = parkingLot.findAvailableSpot(vehicle);
        
        if (spot == null) {
            System.out.println("No available spot for vehicle: " + vehicle.getLicensePlate());
            return null;
        }
        
        ParkingTicket ticket = new ParkingTicket(vehicle, spot);
        vehicle.setTicket(ticket);
        
        System.out.println("Ticket issued: " + ticket.getTicketNumber());
        System.out.println("Assigned spot: " + spot.getSpotNumber());
        
        return ticket;
    }
}

public class ExitPanel {
    private String panelId;
    
    public ExitPanel(String panelId) {
        this.panelId = panelId;
    }
    
    public boolean processExit(ParkingTicket ticket, PaymentStrategy paymentStrategy) {
        ticket.markExit();
        
        double fee = ParkingRate.calculateFee(ticket);
        ticket.setFee(fee);
        
        System.out.println("\n=== Parking Summary ===");
        System.out.println("Ticket: " + ticket.getTicketNumber());
        System.out.println("Duration: " + ticket.getParkingDurationInHours() + " hour(s)");
        System.out.println("Fee: $" + fee);
        
        Payment payment = new Payment(paymentStrategy);
        boolean paymentSuccess = payment.makePayment(fee);
        
        if (paymentSuccess) {
            ticket.setStatus(TicketStatus.PAID);
            ticket.getAssignedSpot().removeVehicle();
            System.out.println("Payment successful. Have a nice day!");
            return true;
        }
        
        System.out.println("Payment failed!");
        return false;
    }
}
```

### 9. Parking Lot (Singleton)

```java
public class ParkingLot {
    private static volatile ParkingLot instance;
    private String name;
    private String address;
    private List<ParkingFloor> floors;
    private List<EntrancePanel> entrances;
    private List<ExitPanel> exits;
    
    private ParkingLot(String name, String address) {
        this.name = name;
        this.address = address;
        this.floors = new ArrayList<>();
        this.entrances = new ArrayList<>();
        this.exits = new ArrayList<>();
    }
    
    public static ParkingLot getInstance() {
        if (instance == null) {
            synchronized (ParkingLot.class) {
                if (instance == null) {
                    instance = new ParkingLot("City Parking Lot", "123 Main St");
                }
            }
        }
        return instance;
    }
    
    public void addFloor(ParkingFloor floor) {
        floors.add(floor);
    }
    
    public void addEntrance(EntrancePanel entrance) {
        entrances.add(entrance);
    }
    
    public void addExit(ExitPanel exit) {
        exits.add(exit);
    }
    
    public synchronized ParkingSpot findAvailableSpot(Vehicle vehicle) {
        for (ParkingFloor floor : floors) {
            ParkingSpot spot = floor.findAvailableSpot(vehicle);
            if (spot != null) {
                return spot;
            }
        }
        return null;
    }
    
    public void displayAvailability() {
        System.out.println("\n=== Parking Lot Availability ===");
        for (ParkingFloor floor : floors) {
            System.out.println("Floor " + floor.getFloorNumber() + 
                             ": " + floor.getAvailableSpotCount() + " spots available");
        }
    }
}
```

### 10. Demo Application

```java
public class ParkingLotDemo {
    public static void main(String[] args) {
        // Initialize parking lot
        ParkingLot parkingLot = ParkingLot.getInstance();
        
        // Add floors
        ParkingFloor floor1 = new ParkingFloor(1);
        floor1.addSpot(new CompactSpot("F1-C1"));
        floor1.addSpot(new CompactSpot("F1-C2"));
        floor1.addSpot(new LargeSpot("F1-L1"));
        floor1.addSpot(new MotorcycleSpot("F1-M1"));
        
        ParkingFloor floor2 = new ParkingFloor(2);
        floor2.addSpot(new CompactSpot("F2-C1"));
        floor2.addSpot(new LargeSpot("F2-L1"));
        floor2.addSpot(new LargeSpot("F2-L2"));
        floor2.addSpot(new HandicappedSpot("F2-H1"));
        
        parkingLot.addFloor(floor1);
        parkingLot.addFloor(floor2);
        
        // Add entry and exit points
        EntrancePanel entrance1 = new EntrancePanel("E1");
        ExitPanel exit1 = new ExitPanel("EX1");
        parkingLot.addEntrance(entrance1);
        parkingLot.addExit(exit1);
        
        // Display initial availability
        parkingLot.displayAvailability();
        
        // Simulate vehicles entering
        System.out.println("\n=== Vehicle Entry ===");
        
        Vehicle car1 = new Car("ABC-123");
        ParkingTicket ticket1 = entrance1.issueTicket(car1);
        
        Vehicle motorcycle1 = new Motorcycle("XYZ-789");
        ParkingTicket ticket2 = entrance1.issueTicket(motorcycle1);
        
        Vehicle truck1 = new Truck("TRK-456");
        ParkingTicket ticket3 = entrance1.issueTicket(truck1);
        
        // Display availability after parking
        parkingLot.displayAvailability();
        
        // Simulate time passing
        try {
            Thread.sleep(2000); // Simulate 2 seconds
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        // Simulate vehicle exit
        System.out.println("\n=== Vehicle Exit ===");
        exit1.processExit(ticket1, new CreditCardPayment("1234-5678-9012-3456"));
        
        // Display final availability
        parkingLot.displayAvailability();
    }
}
```

## Key Design Patterns Used

1. **Singleton Pattern**: ParkingLot ensures single instance
2. **Strategy Pattern**: Payment processing strategies
3. **Factory Pattern**: Could be used for creating different vehicle/spot types
4. **Observer Pattern**: Could notify displays about availability changes

## SOLID Principles Applied

1. **SRP**: Each class has single responsibility
2. **OCP**: Easy to add new vehicle/spot types without modifying existing code
3. **LSP**: Subclasses can substitute parent classes
4. **ISP**: Focused interfaces
5. **DIP**: Depends on abstractions (Vehicle, ParkingSpot)

## Possible Enhancements

1. Add reservation system
2. Implement real-time monitoring dashboard
3. Add electric vehicle charging spots
4. Implement valet parking
5. Add membership/loyalty programs
6. Integration with mobile apps
7. License plate recognition
8. Multi-currency support
9. Peak hour pricing
10. Monthly pass holders

---

[Back to Examples](../../README.md#system-design-examples)
