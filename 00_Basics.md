# Pillars of OOPS

1. Abstraction : Expose only essential feature via interfaces

```java
public interface PaymentProcessor {
    void process(payment p);
    
}

public class CreditCardPaymentProcessor implements PaymentProcessor {
    public void process(payment p){
        //Charge card
    }
}

public class UPIPaymentProcessor implements PaymentProcessor {
    public void process(payment p){
        //Charge card
    }
}



User be creating an object of PaymentProcessor
PaymentProcessor.process(CC)

```

2. Inheritance: Resusing code again and again. Have common code in base class.

3. Ploymorphsim: One name different behavior/forms
                 Treating different types via common interface.

4. Encapsulation: Wrapping/Bundling data members and member functions.




                 

