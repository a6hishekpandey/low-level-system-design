# 📘 Low Level System Design

---

## ⚙️ Types of Relationships in LLD

**Association (Generic has-a relationship):**  
When one class uses or is connected to another. No ownership. (UML Symbol: Solid line)

```java
class Teacher {
    String name;
}

class Student {
    String name;
    Teacher teacher; // Associated with Teacher
}
```

**Aggregation (Weak has-a relationship):**  
A specialized form of Association where one class is a collection or container of others, but they can exist independently. (UML Symbol: ◇ Hollow Diamond)

```java
class Professor {
    String name;
}

class Department {
    List<Professor> professors; // Aggregation
}
```

**Composition (Strong has-a relationship):**  
A stronger form of Aggregation where the child cannot exist without the parent. (UML Symbol: ◆ Filled Diamond)

```java
class Room {
    String name;

    Room(String name) {
        this.name = name;
    }
}

class House {
    List<Room> rooms = new ArrayList<>();

    House() {
        rooms.add(new Room("Living Room"));
        rooms.add(new Room("Bedroom"));
    }
}
```

**Inheritance (Is-a relationship):**  
One class inherits the structure/behavior of another (superclass/subclass). (UML Symbol: △ Hollow Triangle)

```java
class Animal {
    void makeSound() {
        System.out.println("Some sound");
    }
}

class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Bark");
    }
}
```

---

## 🧠 SOLID Principles

**S - Single Responsibilty Principle**  
A class should have only one reason to change, meaning it should only have one responsibilty.  

Example: An Invoice class should only handle invoice-related logic, while database related operations should be handled by a separate InvoiceRepository class. 

```java
class Invoice {
    private double amount;

    Invoice(double amount) {
        this.amount = amount;
    }

    public void createInvoice() {
        System.out.println("Invoice create for amount " + this.amount + ".");
    }
}

class InvoiceRepository {
    public void saveToDatabase() {
        System.out.println("Saving invoice to database.");
    }
}

class EmailService {
    public void sendNotification() {
        System.out.println("Invoice notification.");
    }
}
```

**O - Open/Close Principle**  
Software entities (classes, functions, modules) should be open for extension but cloased for modification.  

Example: Adding new functionality to a system using inheritance or composition without modifying existing code.

Bad code:
```java
public class PaymentProcessor {
    public void pay(String type) {
        if (type.equals("creditcard")) {
            System.out.println("Paid with Credit Card");
        } else if (type.equals("paypal")) {
            System.out.println("Paid with PayPal");
        } else if (type.equals("upi")) {
            System.out.println("Paid with UPI");
        }
    }
}
```

Good code:
```java
public interface PaymentMethod {
    void pay(double amount);
}

public class CreditCardPayment implements PaymentMethod {
    public void pay(double amount) {
        System.out.println("Paid ₹" + amount + " using Credit Card");
    }
}

public class UPIPayment implements PaymentMethod {
    public void pay(double amount) {
        System.out.println("Paid ₹" + amount + " using UPI");
    }
}

public class PaymentProcessor {
    public void processPayment(PaymentMethod method, double amount) {
        method.pay(amount);
    }
}

public class Main {
    public static void main(String[] args) {
        PaymentProcessor processor = new PaymentProcessor();
        PaymentMethod upi = new UPIPayment();

        processor.processPayment(upi, 300.0);
    }
}
```

**L - Liskov Substitution Principle**  
The Liskov Substitution Principle (LSP) states that objects of a superclass should be replaceable with objects of a subclass without altering the correctness of the program. It ensures that a subclass can stand in for its parent class and function correctly in any context that expects the parent class.  

No client should be forced to depend methods it doesn't use. Split large interfaces into smaller, more specific ones.

Bad code:
```java
class Bird {
    public void fly() {
        System.out.println("Bird is flying");
    }
}

class Sparrow extends Bird {
    @Override
    public void fly() {
        System.out.println("Sparrow is flying");
    }
}

class Penguin extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Penguins can't fly");
    }
}

public class Main {
    public static void main(String[] args) {
        Bird sparrow = new Sparrow();
        Bird penguin = new Penguin();

        sparrow.fly();  // OK
        penguin.fly();  // ❌ Violates LSP — throws exception
    }
}
```

Good code:
```java
class Bird {
    public void layEggs() {
        System.out.println("Bird is laying eggs");
    }
}

class FlyingBird extends Bird {
    public void fly() {
        System.out.println("Flying bird is flying");
    }
}

class Sparrow extends FlyingBird {
    @Override
    public void fly() {
        System.out.println("Sparrow is flying");
    }
}

class Penguin extends Bird {
    public void swim() {
        System.out.println("Penguin is swimming");
    }
}

public class Main {
    public static void main(String[] args) {
        FlyingBird sparrow = new Sparrow();
        Bird penguin = new Penguin();

        sparrow.fly(); // ✅ OK
        penguin.fly(); // ❌ Compile-time error — avoids LSP violation
    }
}
```

**I - Interface Segregation Principle**  
The Interface Segregation Principle ensures that classes are not burdened with methods that they don't need. It promotes better design by breaking large, general-purpose interfaces into smaller, more specific ones.  

Bad code:
```java
interface Worker {
    void work();
    void eat();
}

class HumanWorker implements Worker {
    public void work() {
        System.out.println("Human is working");
    }

    public void eat() {
        System.out.println("Human is eating");
    }
}

class RobotWorker implements Worker {
    public void work() {
        System.out.println("Robot is working");
    }

    public void eat() {
        throw new UnsupportedOperationException("Robots don't eat");
    }
}
```

Good code:
```java
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

class HumanWorker implements Workable, Eatable {
    public void work() {
        System.out.println("Human is working");
    }

    public void eat() {
        System.out.println("Human is eating");
    }
}

class RobotWorker implements Workable {
    public void work() {
        System.out.println("Robot is working");
    }
}
```

**D - Dependency Inversion Principle**  
High-level modules should not depend on low-level modules; both should depend on abstraction.  

Bad code:
```java
class EmailService {
    public void sendEmail(String message) {
        System.out.println("Sending email: " + message);
    }
}

class NotificationManager {
    private EmailService emailService;

    public NotificationManager() {
        this.emailService = new EmailService(); // tightly coupled
    }

    public void send(String message) {
        emailService.sendEmail(message);
    }
}
```

Good code:
```java
interface NotificationService {
    void send(String message);
}

class EmailService implements NotificationService {
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}

class SMSService implements NotificationService {
    public void send(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

class NotificationManager {
    private NotificationService service;

    public NotificationManager(NotificationService service) {
        this.service = service; // depends on abstraction
    }

    public void notifyUser(String message) {
        service.send(message);
    }
}
```

---

## 🧠 Design Patterns

**Memento**  
Memento is a behavioral design pattern that lets you save and restore the previous state of an object without revealing the details of its implementation.  

```java
public class TextEditor {
    private String content;
    
    public void write(String content) {
        this.content = content;
    }

    public String getContent() {
        return this.content;
    }

    public TextEditorMemento save() {
        return new TextEditorMemento(this.content);
    }

    public void restore(TextEditorMemento editorMemento) {
        this.content = editorMemento.getContent();
    }
}

public class TextEditorMemento {
    private final String content;
    
    public TextEditorMemento(String content) {
        this.content = content;
    }

    public String getContent() {
        return this.content;
    }
}

public class Caretaker {
    private Stack<TextEditorMemento> history;

    public Caretaker() {
        this.history = new Stack<>();
    }

    public void saveState(TextEditor editor) {
        history.push(editor.save());
    }

    public void undo(TextEditor editor) {
        if(!history.empty()) {
            history.pop();
            editor.restore(history.peek());
        }
    }
}
```

**Observer**  
Observer is a behavioral design pattern that lets you define a subscription mechanism to notify multiple objects about any events that happen to the object they’re observing.   

```java
public class WeatherStation {
    private double temperature;
    private List<Observer> observers;
    
    public WeatherStation() {
        this.observers = new ArrayList<>();
    }

    public void setTemperature(double temperature) {
        this.temperature = temperature;
        notifyObservers();
    }

    public void attach(Observer obs) {
        this.observers.add(obs);
    }

    public void detach(Observer obs) {
        this.observers.remove(obs);
    }

    private void notifyObservers() {
        for(Observer obs: this.observers) {
            obs.update(this.temperature);
        }
    }
}

interface Observer {
    public void update(double temperature);
}

class Mobile implements Observer {
    private String name;

    public Mobile(String name) {
        this.name = name;
    }

    @Override
    public void update(double temperature) {
        System.out.println("Temperature on " + this.name + " is " + temperature);
    }
} 

class LCDScreen implements Observer {
    private String name;

    public LCDScreen(String name) {
        this.name = name;
    }

    @Override
    public void update(double temperature) {
        System.out.println("Temperature on " + this.name + " is " + temperature);
    }
}
```

---
