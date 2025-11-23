# Singleton Pattern

## Intent

**Ensure a class has only one instance and provide a global point of access to it.**

The Singleton pattern is one of the simplest design patterns. It restricts the instantiation of a class to one single instance.

## Problem

Sometimes you need to ensure that a class has exactly one instance:
- Database connection pool
- Configuration manager
- Logger
- Cache
- Thread pool
- File manager

Creating multiple instances of these classes can lead to:
- Resource wastage
- Inconsistent state
- Conflicts

## Solution

Make the class itself responsible for keeping track of its sole instance:
1. Make the constructor private
2. Provide a static method that returns the single instance
3. Store the instance in a static variable

## Structure

```
┌─────────────────────────┐
│      Singleton          │
├─────────────────────────┤
│ - instance: Singleton   │ (static)
├─────────────────────────┤
│ - Singleton()           │ (private constructor)
│ + getInstance(): Singleton │ (static)
│ + businessMethod()      │
└─────────────────────────┘
```

## Implementation

### 1. Eager Initialization (Thread-Safe)

```java
public class DatabaseConnection {
    // Instance created at class loading time
    private static final DatabaseConnection instance = new DatabaseConnection();
    
    // Private constructor prevents instantiation
    private DatabaseConnection() {
        System.out.println("Database connection established");
    }
    
    // Global access point
    public static DatabaseConnection getInstance() {
        return instance;
    }
    
    public void executeQuery(String query) {
        System.out.println("Executing: " + query);
    }
}

// Usage
DatabaseConnection db1 = DatabaseConnection.getInstance();
DatabaseConnection db2 = DatabaseConnection.getInstance();
System.out.println(db1 == db2); // true - same instance
```

**Pros:**
- Thread-safe without synchronization
- Simple and easy to implement

**Cons:**
- Instance created even if never used
- No exception handling during creation

### 2. Lazy Initialization (Not Thread-Safe)

```java
public class Logger {
    private static Logger instance;
    
    private Logger() {
        System.out.println("Logger initialized");
    }
    
    public static Logger getInstance() {
        if (instance == null) {
            instance = new Logger(); // Not thread-safe!
        }
        return instance;
    }
    
    public void log(String message) {
        System.out.println("[LOG] " + message);
    }
}
```

**Pros:**
- Instance created only when needed (lazy loading)

**Cons:**
- Not thread-safe - multiple threads can create multiple instances

### 3. Thread-Safe Lazy Initialization (Synchronized Method)

```java
public class ConfigurationManager {
    private static ConfigurationManager instance;
    private Map<String, String> config;
    
    private ConfigurationManager() {
        config = new HashMap<>();
        loadConfiguration();
    }
    
    // Synchronized method - thread-safe but slow
    public static synchronized ConfigurationManager getInstance() {
        if (instance == null) {
            instance = new ConfigurationManager();
        }
        return instance;
    }
    
    private void loadConfiguration() {
        System.out.println("Loading configuration...");
        config.put("app.name", "MyApp");
        config.put("app.version", "1.0");
    }
    
    public String getConfig(String key) {
        return config.get(key);
    }
}
```

**Pros:**
- Thread-safe
- Lazy initialization

**Cons:**
- Synchronized method slows down every call
- Performance overhead

### 4. Double-Checked Locking (Recommended)

```java
public class CacheManager {
    // volatile ensures visibility across threads
    private static volatile CacheManager instance;
    private Map<String, Object> cache;
    
    private CacheManager() {
        cache = new ConcurrentHashMap<>();
        System.out.println("Cache initialized");
    }
    
    public static CacheManager getInstance() {
        if (instance == null) { // First check (no locking)
            synchronized (CacheManager.class) {
                if (instance == null) { // Second check (with locking)
                    instance = new CacheManager();
                }
            }
        }
        return instance;
    }
    
    public void put(String key, Object value) {
        cache.put(key, value);
    }
    
    public Object get(String key) {
        return cache.get(key);
    }
    
    public void clear() {
        cache.clear();
    }
}
```

**Pros:**
- Thread-safe
- Lazy initialization
- Good performance (synchronization only at creation time)

**Cons:**
- Complex implementation
- Requires volatile keyword (Java 5+)

### 5. Bill Pugh Singleton (Best Practice)

```java
public class ThreadPool {
    // Private constructor
    private ThreadPool() {
        System.out.println("ThreadPool initialized");
    }
    
    // Static inner class - loaded only when getInstance() is called
    private static class SingletonHelper {
        private static final ThreadPool INSTANCE = new ThreadPool();
    }
    
    public static ThreadPool getInstance() {
        return SingletonHelper.INSTANCE;
    }
    
    public void execute(Runnable task) {
        System.out.println("Executing task: " + task);
    }
}
```

**Pros:**
- Thread-safe without synchronization
- Lazy initialization
- Simple and clean
- JVM handles thread safety

**Cons:**
- Slightly more code

### 6. Enum Singleton (Most Robust)

```java
public enum ApplicationContext {
    INSTANCE;
    
    private Map<String, Object> beans;
    
    // Constructor called only once
    ApplicationContext() {
        beans = new HashMap<>();
        System.out.println("Application context initialized");
    }
    
    public void registerBean(String name, Object bean) {
        beans.put(name, bean);
    }
    
    public Object getBean(String name) {
        return beans.get(name);
    }
}

// Usage
ApplicationContext.INSTANCE.registerBean("userService", new UserService());
Object service = ApplicationContext.INSTANCE.getBean("userService");
```

**Pros:**
- Inherently thread-safe
- Serialization safe
- Protection against reflection attacks
- Concise

**Cons:**
- Cannot extend classes (enums can't extend)
- Less flexible

## Complete Example: Database Connection Manager

```java
public class DatabaseConnectionManager {
    private static volatile DatabaseConnectionManager instance;
    private Connection connection;
    private String url = "jdbc:mysql://localhost:3306/mydb";
    private String username = "admin";
    private String password = "password";
    
    private DatabaseConnectionManager() {
        try {
            // Prevent reflection attack
            if (instance != null) {
                throw new RuntimeException("Use getInstance() method");
            }
            Class.forName("com.mysql.jdbc.Driver");
            this.connection = DriverManager.getConnection(url, username, password);
            System.out.println("Database connection established");
        } catch (ClassNotFoundException | SQLException e) {
            throw new RuntimeException("Error connecting to database", e);
        }
    }
    
    public static DatabaseConnectionManager getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnectionManager.class) {
                if (instance == null) {
                    instance = new DatabaseConnectionManager();
                }
            }
        }
        return instance;
    }
    
    public Connection getConnection() {
        return connection;
    }
    
    public void executeQuery(String query) {
        try {
            Statement stmt = connection.createStatement();
            ResultSet rs = stmt.executeQuery(query);
            System.out.println("Query executed: " + query);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    // Prevent cloning
    @Override
    protected Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
    }
}
```

## Singleton and Serialization

```java
import java.io.Serializable;
import java.io.ObjectStreamException;

public class SerializableSingleton implements Serializable {
    private static final long serialVersionUID = 1L;
    private static volatile SerializableSingleton instance;
    
    private SerializableSingleton() {
        if (instance != null) {
            throw new RuntimeException("Use getInstance() method");
        }
    }
    
    public static SerializableSingleton getInstance() {
        if (instance == null) {
            synchronized (SerializableSingleton.class) {
                if (instance == null) {
                    instance = new SerializableSingleton();
                }
            }
        }
        return instance;
    }
    
    // Ensure same instance is returned after deserialization
    protected Object readResolve() throws ObjectStreamException {
        return getInstance();
    }
}
```

## Breaking Singleton Pattern

### 1. Reflection Attack

```java
// Breaking singleton using reflection
try {
    Constructor<DatabaseConnection> constructor = 
        DatabaseConnection.class.getDeclaredConstructor();
    constructor.setAccessible(true);
    DatabaseConnection instance1 = constructor.newInstance();
    DatabaseConnection instance2 = constructor.newInstance();
    System.out.println(instance1 == instance2); // false!
} catch (Exception e) {
    e.printStackTrace();
}

// Prevention: Throw exception in constructor
private DatabaseConnection() {
    if (instance != null) {
        throw new RuntimeException("Instance already exists!");
    }
}
```

### 2. Serialization Attack

```java
// Breaking singleton through serialization
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("singleton.ser"));
out.writeObject(singleton1);
out.close();

ObjectInputStream in = new ObjectInputStream(new FileInputStream("singleton.ser"));
Singleton singleton2 = (Singleton) in.readObject();
in.close();

System.out.println(singleton1 == singleton2); // false without readResolve()!
```

### 3. Cloning Attack

```java
// Prevention: Override clone method
@Override
protected Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException("Singleton cannot be cloned");
}
```

## When to Use

✅ **Use Singleton when:**
- Exactly one instance of a class is required
- Global access point is needed
- Instance should be extensible by subclassing
- Lazy initialization is beneficial

Examples:
- Database connections
- Logger
- Configuration manager
- Cache
- Thread pools
- File system
- Print spooler

## When NOT to Use

❌ **Avoid Singleton when:**
- You need multiple instances
- You need different configurations
- Testing is difficult (hard to mock)
- You want to avoid global state
- Multithreading adds unnecessary complexity

## Advantages

✅ Controlled access to sole instance
✅ Reduced namespace pollution
✅ Permits refinement of operations and representation
✅ Lazy initialization possible
✅ Better than global variables

## Disadvantages

❌ Difficult to test (tight coupling)
❌ Violates Single Responsibility Principle
❌ Global state can be problematic
❌ Concurrency issues if not implemented correctly
❌ Makes code less flexible

## Testing Singleton

```java
// Make singleton testable with dependency injection
public class UserService {
    private DatabaseConnection db;
    
    // For production
    public UserService() {
        this.db = DatabaseConnection.getInstance();
    }
    
    // For testing - inject mock
    UserService(DatabaseConnection db) {
        this.db = db;
    }
}

// Unit test
@Test
public void testUserService() {
    DatabaseConnection mockDb = mock(DatabaseConnection.class);
    UserService service = new UserService(mockDb);
    // Test with mock...
}
```

## Alternatives

1. **Dependency Injection**: Inject single instance through DI container
2. **Static Class**: If no state is needed
3. **Monostate Pattern**: Multiple instances share same state
4. **Session/Request Scope**: Framework-managed scope

## Related Patterns

- **Abstract Factory**: Often implemented as Singleton
- **Builder**: Often implemented as Singleton
- **Prototype**: Can be used instead of Singleton
- **Facade**: Often implemented as Singleton

## Real-World Examples

- `java.lang.Runtime`
- `java.awt.Desktop`
- Spring Framework beans (default scope is singleton)
- Logger frameworks (Log4j, SLF4J)

---

[Back to Design Patterns](../../../README.md#design-patterns) | [Next: Factory Pattern →](./factory.md)
