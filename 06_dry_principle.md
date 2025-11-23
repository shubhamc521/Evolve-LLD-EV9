# DRY Principle (Dont't Repeat Yourself)

## One Liner 
Avoid duplicating code or logic.
Every peice of knowledge should have a single, unambigous representation.


## Validation Example
BAD CODE
```java
public class UserServcie{
    public boolean isValidEmail(String email){
        return email.contains("@") && email.endsWith(".com"); // && email.contains("gmail"); //Issue in userservic and he edits the mailchecking logic.
    }
}

public class RegistrationController{
    public boolean isValidEmail(String email){
        return email.contains("@") && email.endsWith(".com"); // && email.contains("gmail");
    }
}
```

```java
public class ValidationUtils{
    public static boolean isValidEmail(String email){
        return email.contains("@") && email.endsWith(".com"); // && email.contains("gmail"); //Issue in userservic and he edits the mailchecking logic.
    }
}
public class UserServcie{
    public boolean isValidEmail(String email){
        return ValidationUtils.isValidEmail(email);
    }
}

public class RegistrationController{
    public boolean isValidEmail(String email){
        return ValidationUtils.isValidEmail(email);
    }
}
```

