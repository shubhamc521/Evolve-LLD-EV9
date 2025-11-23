# Observer Pattern

## Intent

**Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.**

The Observer pattern is also known as the Publish-Subscribe pattern.

## Problem

Imagine you have:
- A weather station that measures temperature, humidity, and pressure
- Multiple displays that show weather data (current conditions, statistics, forecast)
- Displays need to update whenever weather data changes

How do you ensure displays are automatically updated without tightly coupling them to the weather station?

## Solution

The Observer pattern provides a way to:
1. Define a Subject (Observable) that maintains a list of observers
2. Observers register themselves with the Subject
3. When Subject's state changes, it notifies all observers
4. Observers can then pull the updated data

## Structure

```
┌─────────────────┐                  ┌──────────────────┐
│    Subject      │◆────────────────▷│    Observer      │
├─────────────────┤                  ├──────────────────┤
│ + attach()      │                  │ + update()       │
│ + detach()      │                  └──────────────────┘
│ + notify()      │                           △
└─────────────────┘                           │
         △                                    │
         │                                    │
┌─────────────────┐                  ┌──────────────────┐
│ ConcreteSubject │                  │ ConcreteObserver │
├─────────────────┤                  ├──────────────────┤
│ - state         │                  │ - subject        │
│ + getState()    │                  │ + update()       │
│ + setState()    │                  └──────────────────┘
└─────────────────┘
```

## Implementation

### Basic Implementation

```java
// Observer interface
public interface Observer {
    void update(String message);
}

// Subject interface
public interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

// Concrete Subject - News Agency
public class NewsAgency implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String news;
    
    @Override
    public void attach(Observer observer) {
        observers.add(observer);
        System.out.println("Observer attached");
    }
    
    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
        System.out.println("Observer detached");
    }
    
    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }
    
    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }
    
    public String getNews() {
        return news;
    }
}

// Concrete Observer - News Channel
public class NewsChannel implements Observer {
    private String name;
    private String news;
    
    public NewsChannel(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String news) {
        this.news = news;
        display();
    }
    
    private void display() {
        System.out.println(name + " received news: " + news);
    }
}

// Another Concrete Observer - Mobile App
public class MobileApp implements Observer {
    private String username;
    private String news;
    
    public MobileApp(String username) {
        this.username = username;
    }
    
    @Override
    public void update(String news) {
        this.news = news;
        sendPushNotification();
    }
    
    private void sendPushNotification() {
        System.out.println("Push notification to " + username + ": " + news);
    }
}

// Usage
public class ObserverDemo {
    public static void main(String[] args) {
        NewsAgency agency = new NewsAgency();
        
        // Create observers
        NewsChannel cnn = new NewsChannel("CNN");
        NewsChannel bbc = new NewsChannel("BBC");
        MobileApp userApp = new MobileApp("John");
        
        // Register observers
        agency.attach(cnn);
        agency.attach(bbc);
        agency.attach(userApp);
        
        // Update news - all observers notified
        agency.setNews("Breaking: New LLD pattern discovered!");
        
        System.out.println("\n--- Detaching BBC ---\n");
        agency.detach(bbc);
        
        // Update again - BBC won't be notified
        agency.setNews("Update: Pattern is awesome!");
    }
}
```

**Output:**
```
Observer attached
Observer attached
Observer attached
CNN received news: Breaking: New LLD pattern discovered!
BBC received news: Breaking: New LLD pattern discovered!
Push notification to John: Breaking: New LLD pattern discovered!

--- Detaching BBC ---

Observer detached
CNN received news: Update: Pattern is awesome!
Push notification to John: Update: Pattern is awesome!
```

## Weather Station Example

```java
// Subject interface
public interface WeatherData {
    void registerObserver(WeatherDisplay observer);
    void removeObserver(WeatherDisplay observer);
    void notifyObservers();
}

// Observer interface
public interface WeatherDisplay {
    void update(float temperature, float humidity, float pressure);
    void display();
}

// Concrete Subject
public class WeatherStation implements WeatherData {
    private List<WeatherDisplay> observers;
    private float temperature;
    private float humidity;
    private float pressure;
    
    public WeatherStation() {
        observers = new ArrayList<>();
    }
    
    @Override
    public void registerObserver(WeatherDisplay observer) {
        observers.add(observer);
    }
    
    @Override
    public void removeObserver(WeatherDisplay observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers() {
        for (WeatherDisplay observer : observers) {
            observer.update(temperature, humidity, pressure);
        }
    }
    
    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();
    }
    
    private void measurementsChanged() {
        notifyObservers();
    }
    
    // Getters
    public float getTemperature() { return temperature; }
    public float getHumidity() { return humidity; }
    public float getPressure() { return pressure; }
}

// Concrete Observer - Current Conditions Display
public class CurrentConditionsDisplay implements WeatherDisplay {
    private float temperature;
    private float humidity;
    
    @Override
    public void update(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }
    
    @Override
    public void display() {
        System.out.println("Current conditions: " + temperature + "°C and " + humidity + "% humidity");
    }
}

// Concrete Observer - Statistics Display
public class StatisticsDisplay implements WeatherDisplay {
    private float maxTemp = Float.MIN_VALUE;
    private float minTemp = Float.MAX_VALUE;
    private float tempSum = 0.0f;
    private int numReadings = 0;
    
    @Override
    public void update(float temperature, float humidity, float pressure) {
        tempSum += temperature;
        numReadings++;
        
        if (temperature > maxTemp) {
            maxTemp = temperature;
        }
        
        if (temperature < minTemp) {
            minTemp = temperature;
        }
        
        display();
    }
    
    @Override
    public void display() {
        System.out.println("Avg/Max/Min temperature: " + 
            (tempSum / numReadings) + "/" + maxTemp + "/" + minTemp);
    }
}

// Concrete Observer - Forecast Display
public class ForecastDisplay implements WeatherDisplay {
    private float currentPressure = 29.92f;
    private float lastPressure;
    
    @Override
    public void update(float temperature, float humidity, float pressure) {
        lastPressure = currentPressure;
        currentPressure = pressure;
        display();
    }
    
    @Override
    public void display() {
        System.out.print("Forecast: ");
        if (currentPressure > lastPressure) {
            System.out.println("Improving weather on the way!");
        } else if (currentPressure == lastPressure) {
            System.out.println("More of the same");
        } else {
            System.out.println("Watch out for cooler, rainy weather");
        }
    }
}

// Usage
public class WeatherStationDemo {
    public static void main(String[] args) {
        WeatherStation weatherStation = new WeatherStation();
        
        // Create displays
        CurrentConditionsDisplay currentDisplay = new CurrentConditionsDisplay();
        StatisticsDisplay statisticsDisplay = new StatisticsDisplay();
        ForecastDisplay forecastDisplay = new ForecastDisplay();
        
        // Register displays
        weatherStation.registerObserver(currentDisplay);
        weatherStation.registerObserver(statisticsDisplay);
        weatherStation.registerObserver(forecastDisplay);
        
        // Simulate weather measurements
        System.out.println("=== Weather Update 1 ===");
        weatherStation.setMeasurements(26.6f, 65f, 30.4f);
        
        System.out.println("\n=== Weather Update 2 ===");
        weatherStation.setMeasurements(27.7f, 70f, 29.2f);
        
        System.out.println("\n=== Weather Update 3 ===");
        weatherStation.setMeasurements(25.5f, 90f, 29.2f);
    }
}
```

## Push vs Pull Model

### Push Model (Data pushed to observers)

```java
public interface Observer {
    void update(Data data); // All data pushed
}

public void notifyObservers() {
    for (Observer observer : observers) {
        observer.update(this.data); // Push data
    }
}
```

### Pull Model (Observers pull data they need)

```java
public interface Observer {
    void update(Subject subject); // Reference to subject
}

public void notifyObservers() {
    for (Observer observer : observers) {
        observer.update(this); // Pass subject reference
    }
}

// Observer pulls only what it needs
public void update(Subject subject) {
    this.temperature = ((WeatherStation) subject).getTemperature();
}
```

## Using Java's Built-in Observer (Deprecated in Java 9)

```java
import java.util.Observable;
import java.util.Observer;

// Subject
public class StockMarket extends Observable {
    private String stockSymbol;
    private double price;
    
    public void setStockPrice(String symbol, double price) {
        this.stockSymbol = symbol;
        this.price = price;
        setChanged(); // Mark as changed
        notifyObservers(symbol + ": $" + price); // Notify
    }
}

// Observer
public class StockTrader implements Observer {
    private String name;
    
    public StockTrader(String name) {
        this.name = name;
    }
    
    @Override
    public void update(Observable o, Object arg) {
        System.out.println(name + " notified: " + arg);
    }
}
```

## Event-Driven Architecture

```java
// Event class
public class Event {
    private String type;
    private Object data;
    private long timestamp;
    
    public Event(String type, Object data) {
        this.type = type;
        this.data = data;
        this.timestamp = System.currentTimeMillis();
    }
    
    // Getters
    public String getType() { return type; }
    public Object getData() { return data; }
    public long getTimestamp() { return timestamp; }
}

// Event Listener
public interface EventListener {
    void onEvent(Event event);
}

// Event Manager (Subject)
public class EventManager {
    private Map<String, List<EventListener>> listeners = new HashMap<>();
    
    public void subscribe(String eventType, EventListener listener) {
        listeners.computeIfAbsent(eventType, k -> new ArrayList<>()).add(listener);
    }
    
    public void unsubscribe(String eventType, EventListener listener) {
        List<EventListener> users = listeners.get(eventType);
        if (users != null) {
            users.remove(listener);
        }
    }
    
    public void notify(String eventType, Object data) {
        List<EventListener> users = listeners.get(eventType);
        if (users != null) {
            Event event = new Event(eventType, data);
            for (EventListener listener : users) {
                listener.onEvent(event);
            }
        }
    }
}

// Concrete listeners
public class EmailNotifier implements EventListener {
    @Override
    public void onEvent(Event event) {
        System.out.println("Email sent for event: " + event.getType() + 
                         " with data: " + event.getData());
    }
}

public class LoggingListener implements EventListener {
    @Override
    public void onEvent(Event event) {
        System.out.println("LOG [" + event.getTimestamp() + "]: " + 
                         event.getType() + " - " + event.getData());
    }
}

// Usage
EventManager eventManager = new EventManager();
eventManager.subscribe("user.login", new EmailNotifier());
eventManager.subscribe("user.login", new LoggingListener());

eventManager.notify("user.login", "User: john@example.com");
```

## When to Use

✅ **Use Observer when:**
- Changes to one object require changing others
- Number of dependents is unknown or dynamic
- Object should notify others without knowing who they are
- Need loose coupling between objects
- Implementing event handling systems

Examples:
- GUI event handlers
- Model-View-Controller (MVC)
- Social media notifications
- Stock market tickers
- Real-time data feeds

## Advantages

✅ Loose coupling between subject and observers
✅ Dynamic relationships - observers can be added/removed at runtime
✅ Broadcast communication
✅ Open/Closed Principle - new observers without changing subject
✅ Subject and observers can vary independently

## Disadvantages

❌ Unexpected updates - observers unaware of each other
❌ Memory leaks if observers not properly detached
❌ Performance issues with many observers
❌ Notification order is not guaranteed
❌ Can be complex to debug

## Best Practices

1. **Avoid memory leaks**: Always detach observers when done
2. **Use weak references**: For automatic cleanup
3. **Thread safety**: Synchronize if multi-threaded
4. **Avoid circular dependencies**: Observer shouldn't update subject
5. **Document update semantics**: Clarify what triggers updates

## Related Patterns

- **Mediator**: Centralizes communication (Observer decentralizes)
- **Singleton**: Subject often implemented as Singleton
- **Chain of Responsibility**: Can use Observer for event propagation
- **MVC**: Model notifies Views using Observer

## Real-World Examples

- Java Swing listeners (ActionListener, MouseListener)
- Android LiveData/ViewModel
- RxJava/Reactive programming
- Event emitters in Node.js
- PropertyChangeListener in Java Beans

---

[Back to Design Patterns](../../../README.md#design-patterns) | [Next: Strategy Pattern →](./strategy.md)
