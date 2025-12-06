# Builder Design Pattern


Construct a install application - FAN, TV, Freeze, all 
Hiring a Electrians (Builder) - Providing Everything at once. He exactly knows that Fan should installed on ceilling, 
                                                              TV should be installed on WALL.


Stading on a stool to install fan, But I gave him TV.


EmployeeData
{
    name
    eno
    address
    House no
    State
    Skills
    .
    .
    .
    .
    .
}

EmployeeData(name, eno, address,Houseno ......)
EmployeeData(address, houseno)

Whever there are lot of data members to be assigned, the client or the user need to know the sequence of it.



# Example
```java
public class User{

    private String name;
    private int age;
    private String address;
    -
    -
    -
    -
    -
    -

    public User(Builder builder){
        this.name = builder.name;
        this.age = builder.age;
        this.address = builder.address;
    }

    public static class Builder{
        private String name;
        private int age;
        private String address;

        public Builder setName(String name){
            this.name = name;
            return this;
        }

        public Builder setAge(int age){
            this.age = age;
            return this;
        }
        public Builder setAddress(String name){
            this.name = name;
            return this;
        }

        public User build(){
            return new User(this)
        }
    }




}

User u1 = new User(name,age,address); // you need to pass the arguments in specific sequence, it becomes hard when the argument list > 5 , 10,15
//User u2 = new User(name, address, age); // compilation arror

User u2 = new User.Builder().setName("John").setAge("15").setAddress("ABC").build();



```

