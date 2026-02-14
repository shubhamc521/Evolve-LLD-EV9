# Pillars of OOPS

1. Abstraction : Expose only essential feature via interfaces
                 For hiding internal details
                 
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
                Reusability of code.
                Every base class function should be supported by child class.
Base Class:
Animals - Eat, Sleep, Walk   [X Fly (Functions)?]

Child classes:
Birds - 
Humans - 


3. Ploymorphsim: One name different behavior/forms
                 Treating different types via common interface.

                 void add(int a, int b) {return a+b}
                 void add(float a, float b) {return a+b}

4. Encapsulation: Wrapping/Bundling data members and member functions.
                
                 Class consist of Data Members and member function - Encapsulated?

                 Access Specifier: 
                    Public  - Anyone can access the data member/member function outside the class as well.
                    Private - Can be accessed or modified within the class only.
                    Protected - Can be accessed or modified within the class and its chlid class.
                    default - Can be accessed or modified within the package.




                 

