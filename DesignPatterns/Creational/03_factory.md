# Factory Design Pattern

Factory Design pattern provides a way to create objects without specifying the exact class of object that will be created.

It encapsulates object creation logic, making code flexible and extensible.

# Simple Example

```java
// Produc tinterface

public interface Product{
    void use();
}

//concreate products
public class Book implements Product{
    public void use(){
        SOP("Reading Book.....");
    }
}

public class Pen implements Product{
    public void use(){
        SOP("Writting with Pen....");
    }
}

public class ProductFactory{
    public static Product createProduct(String type){
        if(type.equals("book")) return new Book();
        if(type.equals("pen")) return new Pen();
        throw new IlllegalArgumentExeption("Unknown type");

    }
}

product p1 = ProductFactory.createProduct("book");
roduct p1 = ProductFactory.createProduct("pen");
p1.use();
p2.use(); 


//product p1 = new Book();






```