# Singleton Design Pattern

## One Liner
Ensures a class has only one instance and provides a global access to it.

- For shared resources like configuration, logging, database (Here resources should shared i.e only once instance should be created)

## Example

```java
//AppConfig.java
public class AppConfig{
    private static AppConfig instance;

    private AppConfig(){
    }
    public static AppConfig getInstance(){
        if(instance  == null)
        {
            instance = new AppConfig();
        }
    return instance;
    }
}
```

Here the constructor is private?
-- No other class should be able to create an instance directly.
-- The static method 'getInstance()' checks if an instance already exists or not, if not, create one.
-- All caller get the same instance.



```java
public class DatabaseConnection{
    private static DatabaseConnection instance;

    private DabateConnections(){}

    // Public method to provide access to single instance
    public static DatabaseConnection getInstance(){
        if(instance  == null)
        {
            instance = new DabateConnections();
        }
    return instance;
    }

    public void connect(){
        SOP("Connecting to database")
    }
}

public class Demo{
    public static void main(String[] args){

        DabateConnections db1 = DabateConnections.getInstance();
        // Get the singleton instance again (return the same object)
        DabateConnections db2 = DabateConnections.getInstance();

        db1.connect();

        System.out.println(db1==db2); // True, same instance

    }
}
```

- new DatabaseConnection() in main ---> compilation error

Lazy Instantiation - [Class Loads] --> [First getInstance() called] - Instance created
                     (No memory will be utilized unitll needed.)




Eager Instantiation - [Class loads] --> Instance get created immediately
                      Memory will get used even if not used.


