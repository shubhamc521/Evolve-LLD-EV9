# Open Closed Principle

# One Liner
Software entities (classes, modules, functions) should be open for extention and closed for modification.


Lets take an example of Discount Strategy.
Employee Discount,
BirthDay Discount.

# Bad Code
```java

public class DisocuntCalculator{
    private double total;

    public DiscountCalculator(double total){
        this.total = total;
    }

    public double calculate(Customer c){
        if (c.isEmployee())
        {
            return total * 0.8; //20%0ff
        }
        if (c.isBirthDay())
        {
            return total * 0.9; //10%0ff
        }
        // Modified the exsiting class. To add student discount.
        // if (c.isStudent())
        // {
        //     return total * 0.05;
        // }

        return total;
    }
}
```

Interface defined a contract. 
 - Provides only method singnature and not implementation.
 - Cannot create object of an interface

 - Contract, No Implementation, No Objects.

 Class: Provides both defination and declaration of methods.
 - Implementation
 - Fieleds and Methods.

Why Interface not class? 
- No common code.
- Contract


# Good Code
```java

public interface DisocuntStrategy{
    double apply(double total);
}

public class EmployeeDiscount implements DisocuntStrategy{
    @Override
    public double apply(dobule total){
        return total * 0.8; // 20%off
    }
}

public class BirthdayDiscount implements DisocuntStrategy{
    @Override
    public double apply(dobule total){
        return total * 0.9; // 100%off
    }
}

public class StudentDiscount implements DisocuntStrategy{
    @Override
    public double apply(dobule total){
        return total * 0.05; // 5%off
    }
}

public class DiscountCalculator {
    private DiscountStrategy strategy;

    public DiscountCalculator(DiscountStrategy strategy){
        this.strategy = strategy;
    }
    public double calculate(double total){
        return strategy.apply(total);
    }
}

public class DemoDiscount {
    public static void main(String[] args){
        double total = 100.0;
        DiscountCalculator customer1 = new DiscountCalculator(new EmployeeDiscount);
        System.out.println(customer1.calculate(total));
    }
}




```