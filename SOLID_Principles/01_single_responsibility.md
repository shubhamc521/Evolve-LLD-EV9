# Single Responsibility Principle

## One liner
A class or module should have only one reason to change.


Class: Blueprint of object
Method: A function defined in a class.
Module / Component: File or package containing all related classes.



# Bad Example
```java
public class Order{
    private List<String> items;

    public Order(){
        this.items = new ArrayList<>();
    }

    public void addItem(String item){
        items.add(item);
    }

    public double calculateTotal(){
        double total = 0.0;

        for (String items : items)
        {
            total += 10.0;
        }
        return total;
    }

    public void saveToDatabase(){
        //Logic to store.push it into db
        System.out.println("ISERT INTO Order");
    }

    public String toJson(){
        return "{.     ........}";
    }
}
```

# Good Example

```java

//Order/cart management responsibility (Management)
public class Order{
    private List<String> items;

    public OrderService(){
        this.items = new ArrayList<>();
    }

    public void addItem(String item){
        items.add(item);
    }

    public double calculateTotal(){
        double total = 0.0;

        for (String items : items)
        {
            total += 10.0;
        }
        return total;
    }

    public List<String> getitems(){
        return items;
    }
}
///Responsibility related to DB (Storing)
public class OrderRepository{

    public void saveToDatabase(){
        //Logic to store.push it into db
        System.out.println("ISERT INTO Order");
    }
}
//Reponsibility related to output format (Output)
public class OrderSerializer(){

    public String toJson(){
        return "{.     ........}";
    }

    public String toXML(){
        return "{.     ........}";
    }
}
```

Split it based on responsibility, Don't split much that it will add noise to your code.