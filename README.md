# Java OOP Learning Notes --- 24 November 2026

Today I studied three important Object-Oriented Programming (OOP)
concepts in Java:

1.  Polymorphism
2.  Encapsulation
3.  Abstraction

These are three of the four pillars of OOP. The fourth pillar is
inheritance.

------------------------------------------------------------------------

# 1. Polymorphism in Java

## 1.1 Meaning

Polymorphism means **one thing having many forms**.

In Java, the same method call can behave differently depending on the
arguments passed or the actual object involved.

Java supports two main types:

-   Compile-time polymorphism --- method overloading.
-   Runtime polymorphism --- method overriding.

## 1.2 Compile-time polymorphism: Method overloading

Method overloading means defining multiple methods with the same name
but different parameter lists in the same class.

The parameter lists can differ by:

-   Number of parameters.
-   Types of parameters.
-   Order of parameter types.

### Example

``` java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }

    double add(double a, double b) {
        return a + b;
    }
}

public class Main {
    public static void main(String[] args) {
        Calculator calc = new Calculator();

        System.out.println(calc.add(10, 20));
        System.out.println(calc.add(10, 20, 30));
        System.out.println(calc.add(10.5, 20.5));
    }
}
```

Output:

``` text
30
60
31.0
```

The compiler selects the matching overloaded method based on the
arguments.

**Important:** Methods cannot be overloaded by changing only their
return type. The method name and parameter list would still be
identical.

## 1.3 Runtime polymorphism: Method overriding

Method overriding occurs when a child class provides its own
implementation of an inherited instance method.

### Example

``` java
class Animal {
    void sound() {
        System.out.println("Animal makes a sound");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Dog barks");
    }
}

class Cat extends Animal {
    @Override
    void sound() {
        System.out.println("Cat meows");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal animal1 = new Dog();
        Animal animal2 = new Cat();

        animal1.sound();
        animal2.sound();
    }
}
```

Output:

``` text
Dog barks
Cat meows
```

### Understanding parent references and child objects

``` java
Animal animal = new Dog();
```

-   `Animal` is the reference type.
-   `animal` is the reference variable.
-   `new Dog()` creates the actual object.
-   The reference type determines which members are accessible at
    compile time.
-   The actual object type determines which overridden instance method
    executes at runtime.

This is runtime polymorphism.

## 1.4 Dynamic method dispatch

Dynamic method dispatch is the mechanism Java uses to select the
overridden instance method at runtime based on the actual object.

``` java
Animal animal;

animal = new Dog();
animal.sound(); // Dog barks

animal = new Cat();
animal.sound(); // Cat meows
```

The same reference variable can refer to different child objects, and
the overridden method behavior changes accordingly.

## 1.5 Variables and methods in polymorphism

Consider:

``` java
class Parent {
    int value = 10;

    void show() {
        System.out.println("Parent method");
    }
}

class Child extends Parent {
    int value = 20;

    @Override
    void show() {
        System.out.println("Child method");
    }

    void display() {
        System.out.println("Child display");
    }
}

public class Main {
    public static void main(String[] args) {
        Parent obj = new Child();

        System.out.println(obj.value);
        obj.show();

        // obj.display(); // Compilation error
    }
}
```

Output:

``` text
10
Child method
```

Remember:

-   Instance variables are hidden, not overridden. Access to a variable
    is determined by the reference type.
-   Overridden instance methods are selected using the actual object
    type at runtime.
-   A parent reference cannot directly access child-only methods that
    are not declared in the parent type.

## 1.6 Can static methods be overridden?

No. Static methods are hidden, not overridden. Static method calls are
resolved using the reference or class type.

``` java
class Parent {
    static void show() {
        System.out.println("Parent static method");
    }
}

class Child extends Parent {
    static void show() {
        System.out.println("Child static method");
    }
}

public class Main {
    public static void main(String[] args) {
        Parent obj = new Child();
        obj.show();
    }
}
```

Output:

``` text
Parent static method
```

## 1.7 Overloading vs overriding

  -----------------------------------------------------------------------
  Feature                 Overloading             Overriding
  ----------------------- ----------------------- -----------------------
  Polymorphism type       Compile-time            Runtime

  Method name             Same                    Same

  Parameters              Must differ             Must match, subject to
                                                  overriding rules

  Inheritance             Not required            Inheritance is required

  Method selection        Compiler                Runtime dispatch for
                                                  instance methods

  Return type             May differ if           Same or compatible
                          parameters differ       covariant return type
  -----------------------------------------------------------------------

## 1.8 Why polymorphism is useful

-   Reuse a common parent type or interface for different
    implementations.
-   Add new implementations without rewriting all calling code.
-   Reduce dependence on specific implementation classes.
-   Improve maintainability and extensibility.

### Payment example

``` java
interface Payment {
    void pay(double amount);
}

class UpiPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Paid using UPI: " + amount);
    }
}

class CardPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Paid using Card: " + amount);
    }
}

public class Main {
    public static void main(String[] args) {
        Payment payment = new UpiPayment();
        payment.pay(1000);

        payment = new CardPayment();
        payment.pay(2000);
    }
}
```

Output:

``` text
Paid using UPI: 1000.0
Paid using Card: 2000.0
```

------------------------------------------------------------------------

# 2. Encapsulation in Java

## 2.1 Meaning

Encapsulation means **combining data (fields) and the methods that
operate on that data inside a class, while controlling direct access to
the object's internal state**.

In Java, it is commonly achieved using:

1.  `private` fields.
2.  Public methods that provide controlled access.
3.  Validation inside methods before changing the object's state.

A getter and setter are common tools, but they are not mandatory for
every field.

## 2.2 Why encapsulation is needed

Without encapsulation, code outside a class may directly assign invalid
values to its fields.

### Without encapsulation

``` java
class BankAccount {
    public double balance = 10000;
}

public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount();

        account.balance = -50000;

        System.out.println(account.balance);
    }
}
```

Output:

``` text
-50000.0
```

This is undesirable because a bank account should enforce rules about
its balance.

## 2.3 Using private fields, getters and setters

``` java
class BankAccount {
    private double balance = 10000;

    public double getBalance() {
        return balance;
    }

    public void setBalance(double balance) {
        if (balance >= 0) {
            this.balance = balance;
        } else {
            System.out.println("Invalid balance");
        }
    }
}

public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount();

        System.out.println(account.getBalance());

        account.setBalance(5000);
        System.out.println(account.getBalance());

        account.setBalance(-1000);
        System.out.println(account.getBalance());
    }
}
```

Output:

``` text
10000.0
5000.0
Invalid balance
5000.0
```

### Getter

A getter reads a field's value.

``` java
public double getBalance() {
    return balance;
}
```

### Setter

A setter changes a field's value, optionally after validation.

``` java
public void setBalance(double balance) {
    if (balance >= 0) {
        this.balance = balance;
    }
}
```

### Understanding `this`

``` java
this.balance = balance;
```

-   `this.balance` refers to the current object's instance field.
-   `balance` refers to the method parameter.

## 2.4 Encapsulation does not require setters

A better bank-account design can expose business operations rather than
a general-purpose setter.

``` java
class BankAccount {
    private double balance = 10000;

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        } else {
            System.out.println("Invalid deposit amount");
        }
    }

    public void withdraw(double amount) {
        if (amount <= 0) {
            System.out.println("Invalid withdrawal amount");
        } else if (amount > balance) {
            System.out.println("Insufficient balance");
        } else {
            balance -= amount;
        }
    }
}

public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount();

        account.deposit(5000);
        account.withdraw(2000);

        System.out.println(account.getBalance());
    }
}
```

Output:

``` text
13000.0
```

The balance can only be changed through operations that enforce the
account's rules.

## 2.5 Access modifiers

  -------------------------------------------------------------------------------
  Modifier            Same class     Same package   Subclass in    Other packages
                                                    another        
                                                    package        
  ------------------- -------------- -------------- -------------- --------------
  `private`           Yes            No             No             No

  Default             Yes            Yes            No             No
  (package-private)                                                

  `protected`         Yes            Yes            Yes, subject   No general
                                                    to subclass    access
                                                    access rules   

  `public`            Yes            Yes            Yes            Yes
  -------------------------------------------------------------------------------

For encapsulation, `private` is commonly used for internal fields, while
`public` methods expose selected operations.

## 2.6 Benefits of encapsulation

-   Protects an object's internal state from direct modification.
-   Allows validation before data changes.
-   Makes implementation easier to change.
-   Controls which fields can be read or modified.
-   Helps enforce business rules.

## 2.7 Encapsulation vs data hiding

Data hiding is an important part of encapsulation, but the two terms are
not exactly interchangeable.

-   Data hiding restricts direct access to internal data.
-   Encapsulation combines data and behavior in a class and controls
    access to its internal state.

------------------------------------------------------------------------

# 3. Abstraction in Java

## 3.1 Meaning

Abstraction means **hiding internal implementation details and exposing
only the essential functionality to the user**.

In simple words:

-   Focus on what an object does.
-   Hide the unnecessary details of how it does it.

For example, when using an ATM, you select an operation such as
withdrawing cash. You do not need to understand the internal bank-server
communication or cash-dispenser implementation.

## 3.2 How to achieve abstraction

Java provides two main mechanisms:

1.  Abstract classes.
2.  Interfaces.

## 3.3 Abstract class

An abstract class is declared using the `abstract` keyword.

It can contain:

-   Abstract methods.
-   Concrete methods.
-   Instance variables.
-   Static variables and methods.
-   Constructors.

An abstract method has a declaration but no method body.

``` java
abstract void makeSound();
```

### Example

``` java
abstract class Animal {

    abstract void makeSound();

    void eat() {
        System.out.println("Animal is eating");
    }
}

class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Dog barks");
    }
}

class Cat extends Animal {
    @Override
    void makeSound() {
        System.out.println("Cat meows");
    }
}

public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.makeSound();
        dog.eat();

        Cat cat = new Cat();
        cat.makeSound();
        cat.eat();
    }
}
```

Output:

``` text
Dog barks
Animal is eating
Cat meows
Animal is eating
```

The abstract class defines a common structure. Each concrete child class
provides its own implementation of `makeSound()`.

### Can we create an abstract class object?

No, an abstract class cannot be instantiated directly.

``` java
abstract class Animal {
    abstract void makeSound();
}

// Animal animal = new Animal(); // Compilation error
```

However, a parent reference can refer to a child object:

``` java
Animal animal = new Dog();
animal.makeSound(); // Dog barks
```

This also demonstrates runtime polymorphism.

## 3.4 Interface

An interface defines a contract that implementing classes agree to
follow.

It is declared using the `interface` keyword. A class uses `implements`
to implement it.

### Example

``` java
interface Payment {
    void pay(double amount);
}

class UpiPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Paid using UPI: " + amount);
    }
}

class CardPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Paid using Card: " + amount);
    }
}

public class Main {
    public static void main(String[] args) {
        Payment payment = new UpiPayment();
        payment.pay(1000);

        payment = new CardPayment();
        payment.pay(2000);
    }
}
```

Output:

``` text
Paid using UPI: 1000.0
Paid using Card: 2000.0
```

The `Payment` interface defines what a payment implementation must
provide. Each class decides how to perform the payment.

## 3.5 Important interface rules

### Rule 1: A class implements an interface

``` java
class Dog implements Animal {
    // Implement required methods
}
```

### Rule 2: A class can implement multiple interfaces

``` java
interface Camera {
    void takePhoto();
}

interface MusicPlayer {
    void playMusic();
}

class Smartphone implements Camera, MusicPlayer {

    @Override
    public void takePhoto() {
        System.out.println("Taking photo");
    }

    @Override
    public void playMusic() {
        System.out.println("Playing music");
    }
}
```

Java does not allow a class to extend multiple classes, but it does
allow a class to implement multiple interfaces.

### Rule 3: Interface abstract methods are public by default

An ordinary interface method without a body is implicitly
`public abstract`.

``` java
interface Animal {
    void makeSound();
}
```

This is equivalent to:

``` java
interface Animal {
    public abstract void makeSound();
}
```

An implementing class must provide a `public` implementation.

### Rule 4: Interface fields are constants

Fields declared in an interface are implicitly `public static final`.

``` java
interface Config {
    int MAX_USERS = 100;
}
```

This is equivalent to:

``` java
interface Config {
    public static final int MAX_USERS = 100;
}
```

### Rule 5: Interfaces can have implemented methods

Modern Java interfaces can contain:

-   `default` methods.
-   `static` methods.
-   `private` methods (Java 9 and later).

``` java
interface Payment {

    void pay(double amount);

    default void printReceipt() {
        System.out.println("Receipt generated");
    }

    static void paymentInfo() {
        System.out.println("Payment service");
    }

    private void logPayment() {
        System.out.println("Payment logged");
    }
}
```

A private interface method can be used by other implemented methods
inside the interface; it cannot be called directly by implementing
classes.

## 3.6 Abstract class vs interface

  -----------------------------------------------------------------------
  Feature                 Abstract class          Interface
  ----------------------- ----------------------- -----------------------
  Declaration             `abstract class`        `interface`

  Class relationship      Child uses `extends`    Class uses `implements`

  Methods                 Abstract and concrete   Abstract, default,
                          methods                 static and private
                                                  methods

  Instance variables      Yes                     No ordinary instance
                                                  fields

  Fields                  Instance and static     Implicitly
                          fields                  `public static final`

  Constructor             Yes                     No

  Multiple inheritance    A class can extend only A class can implement
                          one class               multiple interfaces
  -----------------------------------------------------------------------

### When should you use an abstract class?

Use an abstract class when related classes share state, common
implementation, or a common base type.

Examples: `Animal`, `Employee`, `Vehicle`.

### When should you use an interface?

Use an interface when you want to define a capability or contract that
different classes can implement.

Examples: `Runnable`, `Comparable`, `Payment`, `Serializable`.

## 3.7 Abstraction vs encapsulation

  -----------------------------------------------------------------------
  Encapsulation                       Abstraction
  ----------------------------------- -----------------------------------
  Combines data and behavior and      Hides implementation details
  controls access                     

  Focuses on controlled access to     Focuses on exposing essential
  internal state                      functionality

  Commonly uses private fields and    Commonly uses abstract classes and
  public methods                      interfaces

  Example: Private bank balance with  Example: Payment interface
  deposit and withdraw methods        
  -----------------------------------------------------------------------

Easy way to remember:

-   **Encapsulation:** How can someone access or modify my data?
-   **Abstraction:** What functionality should someone see without
    knowing the internal implementation?

## 3.8 Abstraction in a Spring Boot-style design

A service can depend on an interface rather than a concrete
implementation.

``` java
public interface PaymentService {
    void processPayment(double amount);
}
```

Implementations:

``` java
public class UpiPaymentService implements PaymentService {

    @Override
    public void processPayment(double amount) {
        System.out.println("Processing UPI payment: " + amount);
    }
}
```

``` java
public class CardPaymentService implements PaymentService {

    @Override
    public void processPayment(double amount) {
        System.out.println("Processing card payment: " + amount);
    }
}
```

A service can depend on the abstraction:

``` java
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder(double amount) {
        paymentService.processPayment(amount);
    }
}
```

This makes the order-processing code less dependent on a specific
payment implementation. In Spring Boot, dependency injection can provide
the required implementation.

## 3.9 Benefits of abstraction

-   Reduces complexity for users of a class or service.
-   Separates a contract from its implementation.
-   Makes implementations easier to replace.
-   Supports extensibility.
-   Helps reduce coupling when code depends on interfaces.

------------------------------------------------------------------------

# 4. Quick revision: Three OOP pillars

  ------------------------------------------------------------------------
  Concept           Meaning           Java mechanism    Example
  ----------------- ----------------- ----------------- ------------------
  Polymorphism      Same operation    Overloading and   `Payment.pay()`
                    can have          overriding        
                    different forms                     

  Encapsulation     Combine data and  Private fields    `BankAccount`
                    behavior; control and controlled    
                    access            methods           

  Abstraction       Hide              Abstract classes  `PaymentService`
                    implementation    and interfaces    
                    details and                         
                    expose essential                    
                    behavior                            
  ------------------------------------------------------------------------

Remember the distinction:

-   **Polymorphism:** One interface or parent type, different behaviors.
-   **Encapsulation:** Protect and control an object's state.
-   **Abstraction:** Expose what is needed and hide implementation
    details.

------------------------------------------------------------------------

# 5. Interview revision questions

## Polymorphism

1.  What is polymorphism?
2.  What are compile-time and runtime polymorphism?
3.  What is method overloading?
4.  What is method overriding?
5.  Can methods be overloaded by changing only the return type?
6.  What is dynamic method dispatch?
7.  Can static methods be overridden?
8.  What is the difference between a reference type and an actual object
    type?
9.  Can constructors be overridden?
10. Can private or final methods be overridden?

## Encapsulation

1.  What is encapsulation?
2.  How do you achieve encapsulation in Java?
3.  Why do we use private variables?
4.  What are getters and setters?
5.  Is it mandatory to create a setter for every private variable?
6.  Can we achieve encapsulation without setters?
7.  What is data hiding?
8.  What are Java access modifiers?
9.  What is the difference between encapsulation and abstraction?
10. How does encapsulation help maintain business rules?

## Abstraction

1.  What is abstraction?
2.  How do you achieve abstraction in Java?
3.  What is an abstract class?
4.  What is an abstract method?
5.  Can an abstract class have a constructor?
6.  Can an abstract class contain concrete methods?
7.  Can we instantiate an abstract class?
8.  What is an interface?
9.  What is the difference between an abstract class and an interface?
10. Can a class implement multiple interfaces?
11. Can an interface extend multiple interfaces?
12. What are default and static methods in interfaces?

------------------------------------------------------------------------

# 6. Practical assignments

Try implementing these yourself without copying the examples.

### Assignment 1: Polymorphism --- Payment system

-   Create a `Payment` interface.
-   Create `UpiPayment`, `CardPayment`, and `NetBankingPayment`.
-   Implement the `pay()` method differently in each class.
-   Use a `Payment` reference to call each implementation.

### Assignment 2: Encapsulation --- Employee management

Create an `Employee` class with:

-   Private fields: `name`, `salary`, `age`.
-   Getters where required.
-   An age setter that rejects negative values.
-   An `increaseSalary(double amount)` method that accepts only positive
    amounts.

### Assignment 3: Abstraction --- Shape calculator

-   Create an abstract class called `Shape`.
-   Declare an abstract method `calculateArea()`.
-   Add a concrete method `display()`.
-   Create `Circle` and `Rectangle` classes.
-   Implement the area calculation in each child class.

For a circle with radius 5 and a rectangle with length 10 and width 5,
expected areas are approximately `78.54` and `50.0`.

------------------------------------------------------------------------

# 7. Final revision checklist

Before moving to the next Java topic, make sure you can explain these
without looking at the notes:

-   [ ] Difference between overloading and overriding.
-   [ ] Compile-time vs runtime polymorphism.
-   [ ] Dynamic method dispatch.
-   [ ] Parent reference pointing to a child object.
-   [ ] Why variables are not overridden like instance methods.
-   [ ] Private fields, getters, setters, and validation.
-   [ ] Why encapsulation does not require setters.
-   [ ] Abstract classes and abstract methods.
-   [ ] Interfaces and the `implements` keyword.
-   [ ] Abstract class vs interface.
-   [ ] How abstraction and polymorphism work together.

**Suggested next step:** Revise inheritance and then practice a small
Java project that combines all four OOP pillars.
