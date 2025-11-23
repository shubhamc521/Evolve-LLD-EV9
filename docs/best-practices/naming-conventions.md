# Naming Conventions Best Practices

## Overview

Good naming is crucial for code readability and maintainability. Names should be descriptive, meaningful, and follow consistent conventions.

## General Principles

### Be Descriptive and Meaningful

```java
// Bad: Unclear abbreviations
int d; // elapsed time in days
Customer c;
String fn;

// Good: Clear and descriptive
int elapsedTimeInDays;
Customer customer;
String firstName;
```

### Use Intention-Revealing Names

```java
// Bad: What is 86400?
if (timeInSeconds > 86400) {
    // Do something
}

// Good: Clear intent
int SECONDS_PER_DAY = 86400;
if (timeInSeconds > SECONDS_PER_DAY) {
    // Do something
}
```

### Avoid Disinformation

```java
// Bad: accountList is not a List
String accountList; // Actually contains comma-separated values

// Good
String accountNames;
String accountsCsv;
List<Account> accountList; // If it's actually a List
```

### Use Pronounceable Names

```java
// Bad: Hard to pronounce and discuss
Date genymdhms; // generation timestamp (year, month, day, hour, minute, second)

// Good: Easy to discuss
Date generationTimestamp;
```

## Naming Conventions by Language

### Java Conventions

#### Classes and Interfaces

```java
// Classes: PascalCase (UpperCamelCase)
public class UserAccount { }
public class OrderProcessor { }
public class PaymentGateway { }

// Interfaces: Same as classes, sometimes with 'I' prefix or 'able' suffix
public interface UserService { }
public interface Serializable { }
public interface Comparable { }
public interface IUserRepository { } // Less common in Java
```

#### Methods

```java
// Methods: camelCase
public void calculateTotalPrice() { }
public boolean isUserActive() { }
public User getUserById(int id) { }
public void setName(String name) { }

// Boolean methods: use is, has, can, should
public boolean isEmpty() { }
public boolean hasPermission() { }
public boolean canAccess() { }
public boolean shouldRetry() { }
```

#### Variables

```java
// Variables: camelCase
String firstName;
int itemCount;
double totalPrice;
boolean isActive;

// Boolean variables: use is, has, can, should
boolean isValid;
boolean hasAccess;
boolean canEdit;
boolean shouldUpdate;
```

#### Constants

```java
// Constants: UPPER_SNAKE_CASE
public static final int MAX_SIZE = 100;
public static final String DEFAULT_USERNAME = "guest";
public static final double PI = 3.14159;
private static final int RETRY_COUNT = 3;
```

#### Packages

```java
// Packages: lowercase, separated by dots
package com.company.project;
package com.company.project.user;
package com.company.project.order.service;
```

#### Enums

```java
// Enum types: PascalCase
public enum OrderStatus { }
public enum UserRole { }

// Enum values: UPPER_SNAKE_CASE
public enum OrderStatus {
    PENDING,
    PROCESSING,
    SHIPPED,
    DELIVERED,
    CANCELLED
}
```

#### Type Parameters (Generics)

```java
// Single uppercase letter
public class Box<T> { }
public class Pair<K, V> { }
public interface Comparable<T> { }

// Common conventions:
// E - Element (collections)
// K - Key
// V - Value
// N - Number
// T - Type
// S, U, V - 2nd, 3rd, 4th types
```

## Method Naming Patterns

### Getters and Setters

```java
public class User {
    private String name;
    private boolean active;
    
    // Getters
    public String getName() {
        return name;
    }
    
    // For boolean, use 'is' instead of 'get'
    public boolean isActive() {
        return active;
    }
    
    // Setters
    public void setName(String name) {
        this.name = name;
    }
    
    public void setActive(boolean active) {
        this.active = active;
    }
}
```

### Action Methods

```java
// Use verbs that describe the action
public void save(User user) { }
public void delete(int id) { }
public void update(User user) { }
public void process(Order order) { }
public void validate(Input input) { }
public void send(Email email) { }
public void calculate(Invoice invoice) { }

// CRUD operations
public User create(User user) { }
public User read(int id) { }
public void update(User user) { }
public void delete(int id) { }
```

### Query Methods

```java
// Use get for single object
public User getUser(int id) { }
public Order getOrder(String orderId) { }

// Use find for searching (might return null)
public User findUserByEmail(String email) { }
public List<Order> findOrdersByStatus(OrderStatus status) { }

// Use list/getAll for collections
public List<User> listUsers() { }
public List<User> getAllUsers() { }

// Use count for numeric results
public int countActiveUsers() { }
public long countOrdersByDate(Date date) { }

// Use exists/contains for boolean checks
public boolean existsUser(int id) { }
public boolean containsProduct(String sku) { }
```

### Boolean Methods

```java
// Predicates: is, has, can, should
public boolean isValid() { }
public boolean isEmpty() { }
public boolean isActive() { }

public boolean hasPermission() { }
public boolean hasAccess() { }
public boolean hasItems() { }

public boolean canEdit() { }
public boolean canDelete() { }
public boolean canAccess() { }

public boolean shouldUpdate() { }
public boolean shouldRetry() { }
public boolean shouldNotify() { }
```

### Factory Methods

```java
// Use 'create' or 'from'
public static User createUser(String name) { }
public static User fromJson(String json) { }
public static User fromMap(Map<String, Object> map) { }

// Use 'of' for value objects
public static Color of(int r, int g, int b) { }
public static Point of(int x, int y) { }

// Use 'valueOf' for conversion
public static Status valueOf(String name) { }
```

## Variable Naming

### Loop Variables

```java
// Simple loops: i, j, k
for (int i = 0; i < size; i++) {
    for (int j = 0; j < width; j++) {
        matrix[i][j] = 0;
    }
}

// Enhanced for loops: use descriptive names
for (User user : users) {
    processUser(user);
}

for (Order order : orders) {
    validateOrder(order);
}
```

### Temporary Variables

```java
// Even temporary variables should be descriptive
// Bad
int temp = a + b;
String s = user.getName();

// Good
int sum = firstNumber + secondNumber;
String userName = user.getName();
```

### Collection Variables

```java
// Use plural for collections
List<User> users;
Set<String> emails;
Map<Integer, Order> ordersById;

// Arrays
String[] names;
int[] values;
User[] userArray;
```

## Class Naming Patterns

### Service Classes

```java
public class UserService { }
public class OrderService { }
public class PaymentService { }
public class NotificationService { }
```

### Repository/DAO Classes

```java
public class UserRepository { }
public class OrderRepository { }
public interface UserDao { }
```

### Controller Classes

```java
public class UserController { }
public class OrderController { }
public class ProductController { }
```

### Utility Classes

```java
public class StringUtils { }
public class DateUtils { }
public class ValidationUtils { }

// Or with 'Helper'
public class StringHelper { }
public class DateHelper { }
```

### Manager Classes

```java
public class ConnectionManager { }
public class CacheManager { }
public class SessionManager { }
```

### Factory Classes

```java
public class UserFactory { }
public class VehicleFactory { }
public interface ShapeFactory { }
```

### Builder Classes

```java
public class UserBuilder { }
public class QueryBuilder { }
public class EmailBuilder { }
```

## Exception Naming

```java
// Use 'Exception' suffix
public class UserNotFoundException extends Exception { }
public class InvalidInputException extends Exception { }
public class DatabaseConnectionException extends Exception { }

// Runtime exceptions
public class IllegalUserStateException extends RuntimeException { }
public class OrderProcessingException extends RuntimeException { }
```

## Test Naming

### Test Classes

```java
// Add 'Test' suffix
public class UserServiceTest { }
public class OrderRepositoryTest { }
public class PaymentProcessorTest { }
```

### Test Methods

```java
public class UserServiceTest {
    // Pattern: methodName_scenario_expectedResult
    @Test
    public void getUser_validId_returnsUser() { }
    
    @Test
    public void getUser_invalidId_throwsException() { }
    
    @Test
    public void createUser_validData_createsSuccessfully() { }
    
    @Test
    public void createUser_duplicateEmail_throwsException() { }
    
    // Alternative: should_ExpectedBehavior_When_StateUnderTest
    @Test
    public void should_ReturnUser_When_ValidIdProvided() { }
    
    @Test
    public void should_ThrowException_When_InvalidIdProvided() { }
    
    // Or: Given_When_Then
    @Test
    public void givenValidUser_whenSaving_thenSuccess() { }
    
    @Test
    public void givenInvalidEmail_whenValidating_thenThrowsException() { }
}
```

## Database Naming

### Tables

```sql
-- Snake case, plural
users
order_items
product_categories
user_addresses

-- Or PascalCase (less common)
Users
OrderItems
ProductCategories
```

### Columns

```sql
-- Snake case
user_id
first_name
created_at
is_active

-- Or camelCase
userId
firstName
createdAt
isActive
```

## Common Abbreviations (Use Sparingly)

```java
// Acceptable abbreviations
id (identifier)
url (uniform resource locator)
uri (uniform resource identifier)
html (hypertext markup language)
xml (extensible markup language)
json (JavaScript object notation)
sql (structured query language)
api (application programming interface)
dto (data transfer object)
dao (data access object)

// Avoid unclear abbreviations
str // Use 'string' or be more specific
num // Use 'number' or 'count'
cnt // Use 'count'
tmp // Use 'temporary' or be more specific
```

## Anti-Patterns to Avoid

### ❌ Single Letter Names (except loop counters)

```java
// Bad
String n;
int d;
List<User> u;

// Good
String name;
int dayCount;
List<User> users;
```

### ❌ Hungarian Notation

```java
// Bad (outdated)
String strName;
int iCount;
boolean bIsActive;

// Good
String name;
int count;
boolean isActive;
```

### ❌ Type Encoding in Names

```java
// Bad
IUserService userService; // 'I' prefix not common in Java
UserServiceImpl userServiceImpl; // 'Impl' suffix is redundant

// Good
UserService userService;
JdbcUserService jdbcUserService; // If implementation matters
```

### ❌ Noise Words

```java
// Bad
public class UserData { } // 'Data' is noise
public class UserInfo { } // 'Info' is noise
public class TheUser { } // 'The' is noise

// Good
public class User { }
```

### ❌ Number Series

```java
// Bad
String value1;
String value2;
String value3;

// Good
String firstName;
String lastName;
String email;
```

## Context and Scope

### Short Names for Short Scope

```java
// OK for short scope
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}

// Needs longer name for larger scope
private int currentUserIndex;
```

### Longer Names for Larger Scope

```java
// Bad: Too short for class field
private int n;

// Good: Descriptive for class field
private int numberOfActiveUsers;
```

## Best Practices Summary

1. ✅ **Be descriptive**: Names should reveal intent
2. ✅ **Be consistent**: Follow established conventions
3. ✅ **Be pronounceable**: Easy to discuss in conversations
4. ✅ **Be searchable**: Avoid single-letter names
5. ✅ **Use domain terminology**: Speak the language of the business
6. ✅ **Avoid mental mapping**: Don't require readers to translate
7. ✅ **Don't be cute**: Avoid jokes or culture-specific references
8. ✅ **Pick one word per concept**: Don't mix 'get', 'fetch', 'retrieve'
9. ✅ **Use solution/domain names**: Technical terms are fine
10. ✅ **Add meaningful context**: Grouping related variables

## Tools to Help

- **IDE Features**: Refactoring, code completion
- **Linters**: Checkstyle, PMD, SonarQube
- **Code Reviews**: Team feedback on naming choices
- **Style Guides**: Company/project-specific guidelines

---

[← Previous: Code Organization](./code-organization.md) | [Back to Best Practices](../../README.md#best-practices)
