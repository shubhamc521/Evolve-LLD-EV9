# Liskov Substitution Principle.

# One Liner
Subtypes must be substitutable for thier base types without altering desirable program properties.

Subclass can be substitutable for thir baseclass without altering any program.

# Bad Code

```java
//Base Class: BankAcount that allows deposit and withdraw
public class BankAccount{
    private double accountBal;

    public BankAcount( double intialBal)
    {
        this.accountBal = initialBal;
    }

    public void deposit (double depositAmount){
        accountBal += depositAmount;
    }

    public void withdraw (double withdrawAmount){
        accountBal -= withdrawAmount;
    }

    public double getbalance(){
        return accountBal;
    }
}

public class SavingsAccount extends BankAccount{
    public SavingsAccount( double intialBal)
    {
        super(inialBal); // Calls contructor of base/super class.
    }

    //Deposit method

    //Withdraw Method

    //Quaterly Interest method

    //getbal
}

public class CurrentAccount extends BankAccount{
    public CurrentAccount( double intialBal)
    {
        super(inialBal); // Calls contructor of base/super class.
    }

    //Deposit method

    //Withdraw Method

    //getbal
}

public class FixedDepositAccount extends BankAccount{
    public FixedDepositAccount( double intialBal)
    {
        super(inialBal); // Calls contructor of base/super class.
    }

    //Deposit method

    //Withdraw Method. --XXXXXXX Not supported

    //getbal
}
```
id1 = Savings
id2 = Current
id3 = Fixed
id4 = Savings

Client -->for (List of BankAccount [ id1, id2, id3, id4]) 
          {
            id1.deposit(10);
            id2.withdraw(10);
            id3.withdraw(10); // This will throw erorr because Fixed Deposit ac doent have withdraw method.
            id3.getbal();
          }

base Class :
deposit
Withdraw

chlid class:
deposit 
Withdraw.


# Good Code
```java
//Base Class: BankAcount that allows deposit and withdraw
public class BankAccount{
    private double accountBal;

    public BankAcount( double intialBal)
    {
        this.accountBal = initialBal;
    }

    public void deposit (double depositAmount){
        accountBal += depositAmount;
    }

    public double getbalance(){
        return accountBal;
    }
}

public class SavingsAccount extends BankAccount{
    public SavingsAccount( double intialBal)
    {
        super(inialBal); // Calls contructor of base/super class.
    }

    //Deposit method -- overide, implement only if behavior change is there.

    //Withdraw Method -- -- overide, implement only if behavior change is there.

    //Quaterly Interest method

    //getbal
}

public class CurrentAccount extends BankAccount{
    public CurrentAccount( double intialBal)
    {
        super(inialBal); // Calls contructor of base/super class.
    }

    //Deposit method

    //Withdraw Method

    //getbal
}

public class FixedDepositAccount extends BankAccount{
    public FixedDepositAccount( double intialBal)
    {
        super(inialBal); // Calls contructor of base/super class.
    }

    //Deposit method

    //Whtih
    // throw exception
    //Withdraw Method. --XXXXXXX Not supported - 

    //getbal
}
```

Other Approach:

Base classes         Chlid Classes
WithDrawAccount --> Savings and Current
Non WithdrawAccounts. --> FixedD

Client checks ----> if its a withdrawAcount or non withdrawaccount



The concept of Inheritance:

Base Class:
Aminal - Eat, Sleep, fly

Human - Eat, Sleep, 

Bird = fly

Human.Fly()

# InShort:
inheritance says :
Child IS-A Parent (It creates relationship)

liskov says Behavioural correctness
if Child IS-A Parent , it must behave like Parent in All situations. (it validates that relationship)