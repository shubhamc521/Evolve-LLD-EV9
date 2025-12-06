# Abstract Factory Design Pattern







```java
// Abstract Products
public interface Button {
    void paint();
}

public interface Checkbox {
    void paint();
}

// Concrete Products - Windows
public class WinButton implements Button{
    public void paint(){
        System.out.println("Windows Button");
    }
}

public class WinCheckbox implements Checkbox{
    public void paint(){
        System.out.println("Windows Checkbox");
    }
}

// Concrete Products - Mac
public class MacButton implements Button{
    public void paint(){
        System.out.println("Mac Button");
    }
}

public class MacCheckbox implements Checkbox{
    public void paint(){
        System.out.println("Mac Checkbox");
    }
}

// Abstract Factory
public interface GUIFactory{
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete Factory - Windows
public class WinFactory implements GUIFactory{
    public Button createButton(){
        return new WinButton();
    }
    public Checkbox createCheckbox(){
        return new WinCheckbox();
    }
}

// Concrete Factory - Mac
public class MacFactory implements GUIFactory{
    public Button createButton(){
        return new MacButton();
    }
    public Checkbox createCheckbox(){
        return new MacCheckbox();
    }
}

// Client Code
public class Application{
    private Button button;
    private Checkbox checkbox;

    public Application(GUIFactory factory){
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }
    
    public void paint(){
        button.paint();
        checkbox.paint();
    }
}

// Usage
GUIFactory factory;
if (platform.equals("Win")){
    factory = new WinFactory();
} else if (platform.equals("Mac")){
    factory = new MacFactory();
}

Application app = new Application(factory);
app.paint();
```

## Abstract Factory Pattern Diagram

Interface  GUIFactory
                    --
                    WinFactory Macfactory
                    +createButton

                    WinButton -- Implements button

                    Application ---> GUIFactory.

Client Application depends only on ABSTRACT interfaces, GUIDfactory, Button and Checkbox.
- Concreate factories will create concrete product matching their family.
