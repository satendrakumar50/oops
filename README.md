# Object-Oriented Programming (OOP) in Java -- Complete Notes

These notes cover the five core OOP concepts in Java:

1.  Inheritance
2.  Polymorphism
3.  Encapsulation
4.  Abstraction
5.  Interfaces

For each concept, understand the definition, purpose, syntax, examples,
rules, advantages, and interview explanations.

------------------------------------------------------------------------

# 1. Inheritance in Java

## 1.1 What is inheritance?

Inheritance is an OOP concept in which one class acquires accessible
properties and behaviors of another class.

-   **Parent class / superclass:** The class whose members are
    inherited.
-   **Child class / subclass:** The class that inherits from another
    class.

Inheritance promotes code reuse and represents an **IS-A relationship**.

Examples:

-   Car IS-A Vehicle.
-   Dog IS-A Animal.
-   Manager IS-A Employee.

## 1.2 Why do we need inheritance?

Suppose `Car` and `Bike` both need a `brand`, `speed`, and `start()`
method. Without inheritance, the same code may be repeated in both
classes.

That causes:

-   Code duplication.
-   More places to update when behavior changes.
-   Harder maintenance.
-   Reduced reuse.

A common parent class can hold shared behavior.

## 1.3 Syntax

``` java
class Parent {
    // Variables and methods
}

class Child extends Parent {
    // Additional variables and methods
}
```

Java uses the `extends` keyword for class inheritance.

## 1.4 Example

``` java
class Vehicle {
    String brand = "Toyota";

    void start() {
        System.out.println("Vehicle is starting");
    }

    void stop() {
        System.out.println("Vehicle is stopping");
    }
}

class Car extends Vehicle {
    void drive() {
        System.out.println("Car is driving");
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();

        car.start();  // Inherited from Vehicle
        car.stop();   // Inherited from Vehicle
        car.drive();  // Defined in Car

        System.out.println(car.brand);
    }
}
```

Output:

``` text
Vehicle is starting
Vehicle is stopping
Car is driving
Toyota
```

When `Car` extends `Vehicle`, a `Car` object can use accessible
inherited members of `Vehicle` as well as its own members.

## 1.5 Types of inheritance

### A. Single inheritance

One child class inherits from one parent class.

``` text
Animal
  |
 Dog
```

``` java
class Animal {
    void eat() {
        System.out.println("Animal is eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Dog is barking");
    }
}
```

### B. Multilevel inheritance

A class inherits from a class that itself inherits from another class.

``` text
Animal
  |
 Dog
  |
Labrador
```

``` java
class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Barking");
    }
}

class Labrador extends Dog {
    void friendly() {
        System.out.println("Labrador is friendly");
    }
}
```

A `Labrador` object can access the accessible methods inherited from
both `Dog` and `Animal`.

### C. Hierarchical inheritance

Multiple child classes inherit from one parent class.

``` text
       Animal
       /    \
     Dog    Cat
```

``` java
class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Barking");
    }
}

class Cat extends Animal {
    void meow() {
        System.out.println("Meowing");
    }
}
```

Both `Dog` and `Cat` inherit `eat()`.

### D. Multiple inheritance

Multiple inheritance means one class inherits from more than one parent
class.

Java does **not** support multiple inheritance through classes.

``` java
class Father {
    void show() {}
}

class Mother {
    void show() {}
}

// Invalid Java:
class Child extends Father, Mother {
}
```

One reason is ambiguity: if both parent classes provide a method with
the same signature, which implementation should the child inherit? This
is commonly illustrated by the diamond problem.

Java does support a class implementing multiple interfaces. See the
Interfaces section.

### E. Hybrid inheritance

Hybrid inheritance combines multiple inheritance structures. Java does
not support every hybrid structure through classes alone, but
combinations involving interfaces are possible.

## 1.6 Important inheritance rules

1.  A Java class can directly extend only one class.
2.  Constructors are **not inherited**.
3.  A child constructor can invoke a parent constructor using `super()`.
4.  Private members cannot be accessed directly by a child class. They
    may still be part of the parent object, but access must be provided
    through permitted parent methods.
5.  A `final` class cannot be extended.
6.  A child class can override eligible inherited instance methods.

### Constructor example

``` java
class Parent {
    Parent() {
        System.out.println("Parent constructor");
    }
}

class Child extends Parent {
    Child() {
        super();
        System.out.println("Child constructor");
    }
}

public class Main {
    public static void main(String[] args) {
        new Child();
    }
}
```

Output:

``` text
Parent constructor
Child constructor
```

The parent constructor executes before the child constructor. If no
explicit constructor invocation is written, Java normally inserts an
implicit `super()` when applicable.

### `final` prevents inheritance

``` java
final class Parent {
}

// Compilation error:
class Child extends Parent {
}
```

## 1.7 The `super` keyword

`super` refers to the immediate parent-class portion of the current
object.

Common uses:

1.  Access a parent field hidden by a child field.
2.  Call a parent method overridden by the child.
3.  Invoke a parent constructor.

``` java
class Parent {
    String name = "Parent";

    void show() {
        System.out.println("Parent method");
    }
}

class Child extends Parent {
    String name = "Child";

    void display() {
        System.out.println(super.name);
        super.show();
    }
}
```

Output:

``` text
Parent
Parent method
```

A `super(...)` constructor call must be the first statement in a
constructor. It can be used to select a parent constructor.

## 1.8 When Should You Use Inheritance?

Inheritance should be used when there is a genuine **IS-A relationship** between two classes.

However, having an IS-A relationship is not the only condition. The child class should also be **substitutable for its parent class** without changing the correctness of the program.

### 1. Genuine IS-A Relationship

Use inheritance when one class is actually a specialized version of another class.

**Example:**

```java
class Animal {
    void eat() {
        System.out.println("Animal is eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Dog is barking");
    }
}
```

Here:

- Dog IS-A Animal.
- Dog inherits the properties and methods of Animal.
- Dog can use the `eat()` method inherited from Animal.

This is a valid use of inheritance.

### 2. Liskov Substitution Principle (LSP)

This is the second important condition for using inheritance.

**Definition:** Anywhere your code expects a parent class object, you should be able to pass an object of its child class without breaking the expected behavior of the program.

**Example:**

```java
class Animal {
    void eat() {
        System.out.println("Animal is eating");
    }
}

class Dog extends Animal {
    @Override
    void eat() {
        System.out.println("Dog is eating");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal animal = new Dog();
        animal.eat();
    }
}
```

**Output:**
```text
Dog is eating
```

Here, we can use a `Dog` object wherever an `Animal` object is expected. The program continues to work correctly.

This demonstrates the Liskov Substitution Principle.

### 3. Do Not Use Inheritance Just for Code Reusability

Do not use inheritance only to reuse a few lines of code.

Inheritance should represent a genuine IS-A relationship, not just a way to access another class's methods.

When there is no genuine IS-A relationship, **Composition** is often a better choice.

### 4. Composition (HAS-A Relationship)

Composition means one class contains an object of another class and uses its functionality.

**Example:**

```java
class Engine {
    void start() {
        System.out.println("Engine started");
    }
}

class Car {
    private Engine engine;

    Car() {
        engine = new Engine();
    }

    void startCar() {
        engine.start();
        System.out.println("Car started");
    }
}
```

**Output:**
```text
Engine started
Car started
```

Here:
- Car HAS-A Engine.
- A car contains an engine.
- Car uses the functionality of Engine.
- We use composition instead of inheritance.

### 5. Inheritance vs Composition

| Inheritance | Composition |
|---|---|
| Represents IS-A relationship | Represents HAS-A relationship |
| Uses `extends` keyword | Uses object references |
| Creates a parent-child relationship | Creates a relationship between objects |
| Child inherits parent functionality | One object uses another object's functionality |
| Can create tight coupling | Generally provides more flexibility |

### 6. When to Use Which?

| Scenario | Recommended Approach |
|---|---|
| Dog is an Animal | Inheritance |
| Car has an Engine | Composition |
| Manager is an Employee | Inheritance |
| Computer has a Processor | Composition |
| Developer is an Employee | Inheritance |
| House has Rooms | Composition |

### Key Takeaways

1. Use inheritance when there is a genuine IS-A relationship.
2. Follow the Liskov Substitution Principle (LSP).
3. Child classes should not break the expected behavior of parent classes.
4. Do not use inheritance only for code reuse.
5. Prefer composition when there is a HAS-A relationship.
6. Composition often provides better flexibility and reduces tight coupling.

**Interview Answer:**

I use inheritance when there is a genuine IS-A relationship between classes and the child class can properly substitute the parent class without breaking its behavior, following the Liskov Substitution Principle. If I only need to reuse functionality and there is no IS-A relationship, I prefer composition over inheritance.

## 1.9 Interview answer

**Inheritance is an OOP concept in which a child class acquires
accessible fields and methods from a parent class using the `extends`
keyword. It promotes code reusability and represents an IS-A
relationship.**

------------------------------------------------------------------------

# 2. Polymorphism in Java

## 2.1 What is polymorphism?

The word polymorphism comes from:

-   Poly = many
-   Morph = forms

Polymorphism means **many forms**. In Java, it allows the same method
name or parent-type reference to be associated with different behavior
depending on the method signature or actual object.

## 2.2 Types of polymorphism

1.  Compile-time polymorphism --- commonly achieved through method
    overloading.
2.  Runtime polymorphism --- achieved through method overriding and
    dynamic method dispatch.

## 2.3 Compile-time polymorphism: Method overloading

Method overloading means defining multiple methods with the same name
but different parameter lists.

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

The compiler selects the most appropriate overload based on the
arguments and Java's method-invocation rules.

### Ways to overload a method

**Different number of parameters:**

``` java
void show(int a) {}
void show(int a, int b) {}
```

**Different parameter types:**

``` java
void show(int a) {}
void show(double a) {}
```

**Different order of parameter types:**

``` java
void show(int a, String b) {}
void show(String a, int b) {}
```

Changing only the return type does **not** create a valid overload:

``` java
int add(int a, int b) {
    return a + b;
}

// Invalid: same method name and parameter list
double add(int a, int b) {
    return a + b;
}
```

## 2.4 Runtime polymorphism: Method overriding

Method overriding occurs when a subclass provides its own implementation
of an eligible instance method inherited from its parent class.

``` java
class Animal {
    void sound() {
        System.out.println("Animal makes sound");
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
        Animal animal;

        animal = new Dog();
        animal.sound();

        animal = new Cat();
        animal.sound();
    }
}
```

Output:

``` text
Dog barks
Cat meows
```

### Reference type vs actual object type

Consider:

``` java
Animal animal = new Dog();
```

-   Reference type: `Animal`
-   Actual object type: `Dog`

The reference type determines which members are available to the
compiler. For an overridden instance method, the actual object's class
determines which implementation runs at runtime.

This is called **dynamic method dispatch**.

## 2.5 Real-world example: Payment system

``` java
class Payment {
    void pay(double amount) {
        System.out.println("Processing payment");
    }
}

class UpiPayment extends Payment {
    @Override
    void pay(double amount) {
        System.out.println("Paid using UPI: " + amount);
    }
}

class CardPayment extends Payment {
    @Override
    void pay(double amount) {
        System.out.println("Paid using Card: " + amount);
    }
}

class PaypalPayment extends Payment {
    @Override
    void pay(double amount) {
        System.out.println("Paid using PayPal: " + amount);
    }
}

class PaymentService {
    void processPayment(Payment payment, double amount) {
        payment.pay(amount);
    }
}

public class Main {
    public static void main(String[] args) {
        PaymentService service = new PaymentService();

        service.processPayment(new UpiPayment(), 1000);
        service.processPayment(new CardPayment(), 2000);
        service.processPayment(new PaypalPayment(), 3000);
    }
}
```

The service accepts the common parent type `Payment`, but the
implementation that runs depends on the actual object.

If a new payment type is added, the service does not need a separate
method for that type, provided it follows the same parent contract.

## 2.6 Overloading vs overriding

  -----------------------------------------------------------------------
  Method overloading                  Method overriding
  ----------------------------------- -----------------------------------
  Same method name, different         Subclass provides a compatible
  parameter list                      implementation of an inherited
                                      method

  Usually occurs within a class, but  Requires inheritance
  can also involve inheritance        

  Resolved at compile time            Overridden instance method selected
                                      at runtime

  Return type alone cannot            Return type must be the same or
  distinguish overloads               covariant

  Does not require inheritance        Requires a parent-child
                                      relationship
  -----------------------------------------------------------------------

### Important overriding rules

-   Use `@Override` to let the compiler verify that a method really
    overrides an inherited method.
-   An overriding method cannot reduce the visibility of the inherited
    method.
-   An overriding method cannot declare broader checked exceptions than
    the overridden method permits.
-   `final` methods cannot be overridden.
-   `private` methods are not inherited as overridable methods.
-   Static methods are hidden, not overridden.
-   Constructors cannot be overridden.
-   A covariant return type is allowed: the overriding method may return
    a subtype of the original return type.

## 2.7 Interview answer

**Polymorphism means many forms. In Java, it allows a common method name
or reference to represent different behavior. Method overloading
provides compile-time polymorphism, while method overriding provides
runtime polymorphism through dynamic method dispatch.**

------------------------------------------------------------------------

# 3. Encapsulation in Java

## 3.1 What is encapsulation?

Encapsulation is the process of grouping data (fields) and the methods
that operate on that data inside a class, while controlling access to
the internal state.

In simple words, encapsulation protects an object's data and provides
controlled ways to read or modify it.

A bank account is a useful example. Account balance should not be
changed arbitrarily by outside code. Deposits and withdrawals should
follow validation rules.

## 3.2 Why do we need encapsulation?

Without controlled access:

``` java
class BankAccount {
    public double balance;
}

public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount();
        account.balance = -50000;
    }
}
```

The code allows an invalid balance.

Encapsulation lets the class enforce rules whenever its state changes.

## 3.3 Getters and setters

``` java
class BankAccount {
    private double balance;

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

        account.setBalance(5000);
        System.out.println(account.getBalance());

        account.setBalance(-1000);
        System.out.println(account.getBalance());
    }
}
```

Output:

``` text
5000.0
Invalid balance
5000.0
```

### Understand `this`

``` java
this.balance = balance;
```

-   `this.balance` refers to the current object's instance field.
-   `balance` refers to the method parameter.

The `this` keyword refers to the current object.

## 3.4 Encapsulation using business methods

A class does not need a getter and setter for every field. Often,
business methods provide safer control.

``` java
class BankAccount {
    private double balance;

    public BankAccount(double initialBalance) {
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Invalid initial balance");
        }

        this.balance = initialBalance;
    }

    public void deposit(double amount) {
        if (amount <= 0) {
            System.out.println("Invalid deposit amount");
            return;
        }

        balance += amount;
    }

    public void withdraw(double amount) {
        if (amount <= 0) {
            System.out.println("Invalid withdrawal amount");
            return;
        }

        if (amount > balance) {
            System.out.println("Insufficient balance");
            return;
        }

        balance -= amount;
    }

    public double getBalance() {
        return balance;
    }
}

public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount(10000);

        account.deposit(5000);
        account.withdraw(3000);

        System.out.println(account.getBalance());
    }
}
```

Output:

``` text
12000.0
```

There is no public `setBalance()` method. The balance changes only
through operations that enforce the account's rules.

## 3.5 Access modifiers in Java

Access modifiers control where classes, fields, constructors and methods
can be accessed.

  -------------------------------------------------------------------------------------
  Modifier            Same class     Same package   Subclass in        Other packages
                                                    another package    (non-subclass)
  ------------------- -------------- -------------- ------------------ ----------------
  `private`           Yes            No             No                 No

  Default             Yes            Yes            No                 No
  (package-private)                                                    

  `protected`         Yes            Yes            Yes, subject to    No
                                                    protected-access   
                                                    rules              

  `public`            Yes            Yes            Yes                Yes
  -------------------------------------------------------------------------------------

Example:

``` java
class Student {
    private int id;
    String name;             // package-private
    protected int age;
    public String college;
}
```

## 3.6 Advantages of encapsulation

1.  **Data protection:** Prevents uncontrolled access to internal state.
2.  **Validation:** Checks values before changing the object.
3.  **Maintainability:** Internal implementation can change without
    requiring callers to change.
4.  **Control:** The class decides which operations are permitted.
5.  **Reduced coupling:** Other classes interact through public methods
    instead of depending on internal fields.

## 3.7 Encapsulation vs data hiding

  -----------------------------------------------------------------------
  Encapsulation                       Data hiding
  ----------------------------------- -----------------------------------
  Groups data and related behavior in Restricts direct access to
  a class and controls access         implementation details

  Focuses on bundling and controlled  Focuses on restricting visibility
  interaction                         

  Implemented through classes and     Commonly implemented with access
  access control                      modifiers such as `private`
  -----------------------------------------------------------------------

## 3.8 Interview answer

**Encapsulation is an OOP principle in which we bind data and related
methods inside a class and control access to the data using access
modifiers. In Java, we commonly declare fields private and expose
controlled public methods such as getters, setters, or business
operations.**

------------------------------------------------------------------------

# 4. Abstraction in Java

## 4.1 What is abstraction?

Abstraction means hiding implementation details and exposing only the
essential functionality to the user.

In simple words, it focuses on **what an object does**, rather than
exposing every detail of how it does it.

Real-world example: When driving a car, a driver uses the steering
wheel, accelerator and brakes without needing to understand all the
internal details of the engine or braking system.

## 4.2 Why do we need abstraction?

In a payment application, a caller may only need to use:

``` java
payment.pay(1000);
```

The caller does not need to know every internal detail of
authentication, gateway communication, validation or transaction
processing.

Abstraction provides a simpler way to interact with complex systems.

## 4.3 How to achieve abstraction in Java

Java mainly provides:

1.  Abstract classes.
2.  Interfaces.

## 4.4 Abstract classes

An abstract class is declared using the `abstract` keyword. It cannot be
instantiated directly.

It can contain:

-   Abstract methods.
-   Concrete (normal) methods.
-   Instance fields.
-   Constructors.
-   Static methods.
-   Final methods.

Example:

``` java
abstract class Animal {
    abstract void sound();

    void eat() {
        System.out.println("Animal is eating");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Dog barks");
    }
}

public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog();

        dog.sound();
        dog.eat();
    }
}
```

Output:

``` text
Dog barks
Animal is eating
```

The abstract method defines behavior that a concrete subclass must
implement. The concrete `eat()` method provides shared behavior.

## 4.5 Important rules of abstract classes

### Rule 1: Cannot instantiate an abstract class

``` java
abstract class Animal {
}

// Invalid:
Animal animal = new Animal();
```

You can declare an abstract-class reference and assign it a concrete
subclass object:

``` java
Animal animal = new Dog();
```

### Rule 2: An abstract class can have a constructor

``` java
abstract class Animal {
    Animal() {
        System.out.println("Animal constructor");
    }
}
```

The constructor runs as part of creating a concrete subclass object.

### Rule 3: An abstract class can have concrete methods

``` java
abstract class Animal {
    void eat() {
        System.out.println("Eating");
    }
}
```

### Rule 4: A subclass must implement inherited abstract methods or remain abstract

``` java
abstract class Animal {
    abstract void sound();
}

abstract class Dog extends Animal {
}
```

A concrete subclass must implement all inherited abstract methods.

### Rule 5: An abstract method cannot be private, static or final

An abstract method must be implemented by a subclass. `private`,
`static`, and `final` conflict with that requirement.

## 4.6 Real-world example: Payment abstraction

``` java
abstract class Payment {
    abstract void pay(double amount);

    void paymentSuccess() {
        System.out.println("Payment successful");
    }
}

class UpiPayment extends Payment {
    @Override
    void pay(double amount) {
        System.out.println("Processing UPI payment: " + amount);
        paymentSuccess();
    }
}

class CardPayment extends Payment {
    @Override
    void pay(double amount) {
        System.out.println("Processing Card payment: " + amount);
        paymentSuccess();
    }
}

public class Main {
    public static void main(String[] args) {
        Payment payment = new UpiPayment();
        payment.pay(5000);

        Payment payment2 = new CardPayment();
        payment2.pay(10000);
    }
}
```

Output:

``` text
Processing UPI payment: 5000.0
Payment successful
Processing Card payment: 10000.0
Payment successful
```

The caller works with the common `Payment` abstraction, while the
concrete subclasses implement the specific behavior.

## 4.7 Advantages of abstraction

1.  Hides unnecessary implementation details.
2.  Reduces complexity for callers.
3.  Makes code easier to understand.
4.  Helps separate a contract from its implementation.
5.  Allows multiple implementations behind a common type.

## 4.8 Abstraction vs encapsulation

  -----------------------------------------------------------------------
  Abstraction                         Encapsulation
  ----------------------------------- -----------------------------------
  Hides implementation complexity     Protects and controls access to
                                      internal state

  Focuses on what an object does      Focuses on how data is stored and
                                      accessed

  Achieved using abstract classes and Achieved using classes and access
  interfaces                          modifiers

  Reduces complexity for users of a   Helps preserve valid object state
  class                               
  -----------------------------------------------------------------------

Example:

-   Abstraction: Call `withdraw()` without knowing the internal banking
    process.
-   Encapsulation: Prevent direct, uncontrolled modification of the
    account balance.

## 4.9 Interview answer

**Abstraction is the process of hiding internal implementation details
and exposing only essential functionality. In Java, we achieve
abstraction using abstract classes and interfaces.**

------------------------------------------------------------------------

# 5. Interfaces in Java

## 5.1 What is an interface?

An interface in Java is a reference type that defines a contract that
implementing classes agree to follow.

An interface describes capabilities that a class promises to provide.

For example, UPI, card and PayPal payment classes can all provide a
common `pay()` operation, even though their implementations differ.

## 5.2 Why do we need interfaces?

Suppose two classes have a `pay()` method but no common parent type. It
becomes harder to write one service method that accepts either
implementation.

An interface provides that common type and contract.

## 5.3 Creating an interface

``` java
interface Payment {
    void pay(double amount);
}
```

Implement it with a class:

``` java
class UpiPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("UPI payment: " + amount);
    }
}

class CardPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Card payment: " + amount);
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
UPI payment: 1000.0
Card payment: 2000.0
```

### Understand the syntax

-   `interface Payment`: declares an interface.
-   `implements Payment`: declares that a class implements the
    interface.
-   `Payment payment = new UpiPayment()`: an interface reference holds
    an object of an implementing class.

This is also runtime polymorphism.

## 5.4 Important features of interfaces

### 1. Abstract methods are public and abstract by default

``` java
interface Animal {
    void sound();
}
```

This is equivalent to:

``` java
interface Animal {
    public abstract void sound();
}
```

The implementing method must be public:

``` java
class Dog implements Animal {
    @Override
    public void sound() {
        System.out.println("Dog barks");
    }
}
```

### 2. Interface fields are public, static and final by default

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

Use it through the interface name:

``` java
System.out.println(Config.MAX_USERS);
```

The value cannot be reassigned.

### 3. An interface cannot be instantiated

Invalid:

``` java
Payment payment = new Payment();
```

Use an implementing class:

``` java
Payment payment = new UpiPayment();
```

### 4. A class can implement multiple interfaces

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

A class can implement multiple interfaces. This provides multiple
inheritance of type without allowing a class to extend multiple classes.

### 5. An interface can extend other interfaces

``` java
interface Animal {
    void eat();
}

interface Pet extends Animal {
    void play();
}

class Dog implements Pet {
    @Override
    public void eat() {
        System.out.println("Dog eats");
    }

    @Override
    public void play() {
        System.out.println("Dog plays");
    }
}
```

An interface uses `extends` to inherit another interface. An interface
can extend multiple interfaces.

### 6. Default methods

Since Java 8, interfaces can contain default methods with an
implementation.

``` java
interface Vehicle {
    void start();

    default void stop() {
        System.out.println("Vehicle stopped");
    }
}

class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car started");
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();

        car.start();
        car.stop();
    }
}
```

Output:

``` text
Car started
Vehicle stopped
```

The implementing class inherits the default implementation and may
override it.

### 7. Static methods in interfaces

Interfaces can contain static methods.

``` java
interface Calculator {
    static int add(int a, int b) {
        return a + b;
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println(Calculator.add(10, 20));
    }
}
```

Output:

``` text
30
```

Call an interface static method using the interface name. It is not
inherited as an instance method by implementing classes.

### 8. Private methods in interfaces

Since Java 9, interfaces can contain private methods. They can help
share internal logic between default and static methods.

``` java
interface Notification {
    private void log() {
        System.out.println("Notification logged");
    }

    default void send() {
        log();
        System.out.println("Sending notification");
    }
}
```

An implementing class cannot call the private interface method directly.

## 5.5 Interface vs abstract class

  -----------------------------------------------------------------------
  Abstract class                      Interface
  ----------------------------------- -----------------------------------
  Declared using `abstract class`     Declared using `interface`

  A class extends it                  A class implements it

  Can have instance fields            Fields are implicitly
                                      `public static final`

  Can have constructors               Cannot have constructors

  Can have abstract and concrete      Can have abstract, default, static
  methods                             and private methods

  A class can extend only one class   A class can implement multiple
                                      interfaces

  Members can have different access   Abstract interface methods are
  modifiers                           public by default
  -----------------------------------------------------------------------

### When should you use an abstract class?

Use an abstract class when related classes share a common base, state,
constructors or implementation.

Example:

``` text
       Vehicle
       /  |  \
     Car Bike Truck
```

Vehicles may share fields such as `brand` and `speed`, and common
behavior such as `start()`.

### When should you use an interface?

Use an interface when different classes should follow the same contract,
even if they are otherwise unrelated.

Example:

``` text
       Printable
       /       \
   Invoice    Report
```

Both `Invoice` and `Report` can implement `Printable`.

## 5.6 Interface default-method conflicts

If two interfaces provide a default method with the same signature, the
implementing class must resolve the conflict.

``` java
interface A {
    default void show() {
        System.out.println("A");
    }
}

interface B {
    default void show() {
        System.out.println("B");
    }
}

class C implements A, B {
    @Override
    public void show() {
        A.super.show();
        B.super.show();
    }
}
```

The class explicitly chooses how to resolve the conflict.

## 5.7 Interview answer

**An interface in Java defines a contract that implementing classes must
follow. It supports abstraction and allows a class to implement multiple
interfaces. Interfaces can contain abstract methods, constants, default
methods, static methods and private methods.**

------------------------------------------------------------------------

# 6. How all five OOP concepts work together

Consider an e-commerce application with a payment module.

Requirements:

1.  Support multiple payment methods.
2.  Protect transaction-related state.
3.  Hide implementation details.
4.  Allow new payment methods without changing existing business logic.
5.  Reuse common functionality.

The following example demonstrates the concepts together.

## Step 1: Create an interface

``` java
interface Payment {
    void pay(double amount);
}
```

This defines a contract.

**Concept: Interface and abstraction.**

## Step 2: Create an abstract class

``` java
abstract class BasePayment implements Payment {
    protected String transactionId;

    public void paymentSuccess() {
        System.out.println("Payment successful");
    }
}
```

This provides shared functionality.

**Concept: Abstract class and inheritance.**

## Step 3: Create UPI payment

``` java
class UpiPayment extends BasePayment {
    @Override
    public void pay(double amount) {
        transactionId = "UPI123";

        System.out.println("UPI payment: " + amount);
        paymentSuccess();
    }
}
```

## Step 4: Create card payment

``` java
class CardPayment extends BasePayment {
    @Override
    public void pay(double amount) {
        transactionId = "CARD123";

        System.out.println("Card payment: " + amount);
        paymentSuccess();
    }
}
```

## Step 5: Create payment service

``` java
class PaymentService {
    public void processPayment(Payment payment, double amount) {
        payment.pay(amount);
    }
}
```

## Step 6: Main class

``` java
public class Main {
    public static void main(String[] args) {
        PaymentService service = new PaymentService();

        Payment upi = new UpiPayment();
        Payment card = new CardPayment();

        service.processPayment(upi, 1000);
        service.processPayment(card, 2000);
    }
}
```

Output:

``` text
UPI payment: 1000.0
Payment successful
Card payment: 2000.0
Payment successful
```

## Where are the five concepts?

  -----------------------------------------------------------------------
  OOP concept                         Where it is used
  ----------------------------------- -----------------------------------
  Inheritance                         `UpiPayment extends BasePayment`

  Polymorphism                        `Payment upi = new UpiPayment()`

  Encapsulation                       Transaction-related state should be
                                      controlled through methods

  Abstraction                         Payment interface and abstract
                                      class

  Interface                           `interface Payment`
  -----------------------------------------------------------------------

**Encapsulation improvement:** In production code, make `transactionId`
private and expose only the operations or read access that are actually
required. The example uses `protected` to keep the inheritance
demonstration simple.

------------------------------------------------------------------------

# 7. Important differences for Java interviews

## 7.1 Inheritance vs polymorphism

  -----------------------------------------------------------------------
  Inheritance                         Polymorphism
  ----------------------------------- -----------------------------------
  Reuses accessible parent            Allows different implementations or
  functionality                       forms

  Uses `extends` for class            Achieved through overloading and
  inheritance                         overriding

  Represents an IS-A relationship     Represents multiple forms of
                                      behavior

  Helps code reuse                    Helps flexibility and extensibility
  -----------------------------------------------------------------------

Example of inheritance:

``` java
class Dog extends Animal {
}
```

Example of runtime polymorphism, assuming `sound()` is overridden:

``` java
Animal animal = new Dog();
animal.sound();
```

## 7.2 Abstract class vs interface

Remember:

-   **Abstract class:** Useful when related classes share a common base,
    state or implementation.
-   **Interface:** Useful when classes need to follow a common contract
    or expose a capability.

## 7.3 Encapsulation vs abstraction

Remember:

-   **Encapsulation:** Protecting internal data and controlling access.
-   **Abstraction:** Hiding implementation complexity and exposing
    essential operations.

Example:

-   Encapsulation: The account balance is private.
-   Abstraction: The caller uses `withdraw()` without knowing the
    internal banking process.

------------------------------------------------------------------------

# 8. Common Java interview questions and answers

## Inheritance

### Q1. What is inheritance in Java?

Inheritance is an OOP concept where one class acquires accessible fields
and methods of another class using the `extends` keyword. It promotes
code reusability and represents an IS-A relationship.

### Q2. Does Java support multiple inheritance?

Java does not support multiple inheritance through classes because it
can create ambiguity, commonly illustrated by the diamond problem.
However, Java supports a class implementing multiple interfaces.

### Q3. What is the difference between `this` and `super`?

-   `this` refers to the current object.
-   `super` refers to the immediate parent-class portion of the current
    object.

### Q4. Are constructors inherited?

No. Constructors are not inherited, but a child constructor can invoke a
parent constructor using `super()`.

## Polymorphism

### Q5. What are the types of polymorphism?

1.  Compile-time polymorphism: Method overloading.
2.  Runtime polymorphism: Method overriding.

### Q6. Can we overload the main method?

Yes. However, the JVM uses the standard `main(String[] args)` entry
point.

``` java
public class Main {
    public static void main(String[] args) {
        System.out.println("Main method");
    }

    public static void main(int a) {
        System.out.println(a);
    }
}
```

The JVM does not automatically call the overloaded `main(int)` method.

### Q7. Can we override a static method?

No. Static methods belong to the class rather than individual objects. A
subclass can hide a static method, but it does not override it.

## Encapsulation

### Q8. How do you achieve encapsulation in Java?

By declaring fields private and providing controlled access through
public methods, such as getters, setters or business methods.

### Q9. Why should we make variables private?

To prevent uncontrolled modification and help ensure that the object's
state remains valid.

## Abstraction

### Q10. Can we create an object of an abstract class?

No. But we can create a reference of an abstract class that points to a
child object.

``` java
Animal animal = new Dog();
```

### Q11. Can an abstract class have a constructor?

Yes. Its constructor executes when a subclass object is created.

### Q12. Can an abstract class have normal methods?

Yes. An abstract class can contain both abstract and concrete methods.

## Interfaces

### Q13. Can an interface extend multiple interfaces?

Yes.

``` java
interface A {
}

interface B {
}

interface C extends A, B {
}
```

### Q14. Can a class implement multiple interfaces?

Yes.

``` java
class MyClass implements A, B {
}
```

The class must implement the required abstract methods unless it is
itself abstract.

### Q15. Can an interface have a constructor?

No. Interfaces cannot be instantiated and cannot have constructors.

### Q16. What is the difference between `extends` and `implements`?

-   `extends`: Used by a class to inherit another class, or by an
    interface to inherit other interfaces.
-   `implements`: Used by a class to implement an interface.

------------------------------------------------------------------------

# 9. Quick revision sheet

  -----------------------------------------------------------------------
  Concept           Main idea         Important         Example
                                      keywords          
  ----------------- ----------------- ----------------- -----------------
  Inheritance       Reuse parent      `extends`,        Car IS-A Vehicle
                    functionality     `super`           

  Polymorphism      Multiple forms of Overloading,      Different payment
                    behavior          overriding        implementations

  Encapsulation     Protect and       `private`,        Private bank
                    control internal  `public`,         balance
                    state             `protected`       

  Abstraction       Hide              `abstract`,       Use `withdraw()`
                    implementation    `interface`       without knowing
                    complexity                          its internals

  Interface         Define a contract `interface`,      UPI and Card
                    or capability     `implements`,     implement Payment
                                      `default`         
  -----------------------------------------------------------------------

## One-line memory trick

-   Inheritance = Code reuse.
-   Polymorphism = Many forms.
-   Encapsulation = Data protection.
-   Abstraction = Hide complexity.
-   Interface = Contract.

------------------------------------------------------------------------

# 10. Practice quiz

Try answering these without looking at the notes.

1.  Which keyword is used to inherit a class in Java?
    -   A. `implements`
    -   B. `extends`
    -   C. `inherits`
    -   D. `super`
2.  Which type of polymorphism is method overloading?
    -   A. Runtime
    -   B. Compile-time
    -   C. Dynamic
    -   D. None
3.  What is the main purpose of encapsulation?
    -   A. Code duplication
    -   B. Data protection and controlled access
    -   C. Multiple inheritance
    -   D. Object creation
4.  Can we create an object of an abstract class?
    -   A. Yes, always
    -   B. No
    -   C. Only with static methods
    -   D. Only using `final`
5.  Which keyword is used by a class to implement an interface?
    -   A. `extends`
    -   B. `super`
    -   C. `implements`
    -   D. `interface`
6.  Can a Java class implement multiple interfaces?
    -   A. No
    -   B. Yes
    -   C. Only two
    -   D. Only if abstract
7.  Which method is selected based on the actual object type at runtime?
    -   A. Overloaded method
    -   B. Overridden instance method
    -   C. Constructor
    -   D. Private method

## Answer key

1.  B --- `extends`.
2.  B --- Compile-time polymorphism.
3.  B --- Data protection and controlled access.
4.  B --- An abstract class cannot be instantiated directly.
5.  C --- `implements`.
6.  B --- A class can implement multiple interfaces.
7.  B --- Dynamic method dispatch selects the overridden implementation
    based on the actual object.

------------------------------------------------------------------------

# 11. Recommended next topics for Java backend development

After understanding these OOP concepts, follow this learning order:

1.  Exception Handling.
2.  Collections Framework.
3.  Generics.
4.  Lambda Expressions.
5.  Functional Interfaces.
6.  Stream API.
7.  Multithreading.
8.  JDBC and SQL.
9.  Spring Core and Dependency Injection.
10. Spring Boot and REST APIs.

## Final practice recommendation

Build a small banking application or e-commerce payment module using all
five OOP concepts.

Try implementing:

-   A `BankAccount` class with private balance.
-   An abstract `Payment` class or a `Payment` interface.
-   UPI and Card payment implementations.
-   A payment service that accepts the common `Payment` type.
-   Different implementations of the same `pay()` method.

The goal is not just to memorize definitions. Understand why each
concept is used and how it helps design maintainable Java applications.
