# Interface Segregation

## One Liner
Clients should not be forced to depend upon interfaces they do not use.

- Interface: A constract that list methods a class should provide.

- If an interface groups unrelated methods, it becomes hard to follow the contract and implement non required methods as well or throw exception.

Bank Account --> deposit and withdraw ---> Savings AC, Current AC and Fixed Deposit (Withdraw method with exception).

Seperate the Bank Account Interface into multiple Interface:
WithdrawableAC
NonWithDrawableAC

```java
//Machine.java

public interface Machine{
    void print(Document d);
    void scan(Document d);
    void fax(Document d);
}

// Imagine there is a document class with all doc details

// This is a OldPrinter, It can only [Print]
public class OldPrinter implements Machine{
    @Override
    public void print(Document d){
        System.out.println("Printing.....");
    }
    @Override
    public void scan(Document d){
        throw new UnsupportedOperationException("Scan not supported.....");
    }
    @Override
    public void fax(Document d){
        throw new UnsupportedOperationException("Fax not supported.....");
    }
}

//Client Code

public class PrintDemo{
    public static void main(String[] args){
        Machine p = new OldPrinter();
        p.print(new Document("Hello"));
        //p.scan(....) thows wrror
    }
}
```

Call to scan gives exception
Client is not happy as they are getting exception.
Developers are not happy - as they need to implement irrelevant methods.

# Refactor to small, role specific interfaces 
```java

// Small interface
public interface Printer{ void print(Document d); }

public interface Scanner{ void scan(Document d); }

public interface Fax{ void fax(Document d); }

public class OldPrinter implements Printer{
    @Override
    public void print(Document d){
       System.out.println("Printing....."); 
    }
}

public class AllInOnePrinter implements Printer, Scanner, Fax{
    @Override
    public void print(Document d){
        System.out.println("Printing.....");
    }
    @Override
    public void scan(Document d){
        System.out.println("Scanning.....");
    }
    @Override
    public void fax(Document d){
        System.out.println("Fax .....");
    }
}

public class Demo{
    public static void main(String[] args){
        Document doc = new Document("Hi");

        //Client 1 : only needs printer
        Printer simplePrinter = new OldPrinter();
        simplePrinter.print(doc);

        //Client 2: Need ALLINONE Printer
        Printer AIOPrinter = new AllInOnePrinter();
        AIOPrinter.print(doc);
        AIOPrinter.scan(doc);
        AIOPrinter.fax(doc);

        // Seperate Printer and Scanner
        Scanner scanner = AIOPrinter;
        AIOPrinter.scan(doc);
    }
}
```

