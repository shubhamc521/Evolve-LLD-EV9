# Dependency Injection Principle

## One Liner
High Level modules should not be dependent on low level module; both should depend on abstraction.
Abstraction should not depend on details - details should depend on abstraction.




Ecommerce site ---> Payment - UPI, CC, DB


High level module - Auth Service
Low Level Module: Mailer1


#Bad Code
```java

public class GoogleMailer{
    public void send(String to, String body){
        SOP("Sending Email.......")
    }
}

public class AuthService{ 
    // BAD - It is creating the gmailer itslef
    private GoogleMailer gmailer = new Mailer(); //Direct mailer instantation

    public void register(String email){
        gmailer.mail(email, "Welcome")
    }
}
```

Why this is bad?
- If have onboarded different mailer.
  Edit the AuthService

## Refactor to depend on Abstraction
```java

public interface Mailer{
    public void send(String to, String body);
}

public class GMailer implements the Mailer{
    @Override
    public void send(String to, String body){
        SOP("Sending Email.......")
    }
}


public class AuthService{
    private Mailer mailer;

    // Constructor Injection // Accepting the mailer from outside world.
    public AuthService(Mailer mailer){
        this.mailer = mailer;
    }

    public void register(String email){
        mailer.send(email, "Welcome")
    }
}

public class DemoAuth {
    public static void main(String[] args){
        Mailer gmailer = new gmailer();
        AuthService svc = new AuthService(gmailer); // Dependency is being injected
        svc.register("john@gmail.com");
    }
}

```
- AuthService doesn't need to create a gmailer itself
- Receving it through constructor
- Dependency Injection - Dependency Injected from outside world.