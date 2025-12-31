# Low Level System Design

---

## SOLID Principles

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
        penguin.fly();  // Violates LSP — throws exception
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

        sparrow.fly(); // OK
        penguin.fly(); // Compile-time error — avoids LSP violation
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

## Design Patterns

**Strategy**  
Strategy is a behavioral design pattern that lets you define a set of interchangeable algorithms (strategies), put each of them into a separate class, and allows the client to choose which algorithm to use at runtime. 

```java
public interface QuackBehavior {
    void quack();
}

public class Quack implements QuackBehavior {
    public void quack() {
        System.out.println("Quack!");
    }
}

public class Squeak implements QuackBehavior {
    public void quack() {
        System.out.println("Squeak!");
    }
}

public abstract class Duck {
    protected QuackBehavior quackBehavior;

    public Duck() {}

    public abstract void display();

    public void performQuack() {
        quackBehavior.quack();
    }

    public void swim() {
        System.out.println("All ducks float, even decoys!");
    }

    // Runtime behavior change
    public void setQuackBehavior(QuackBehavior qb) {
        quackBehavior = qb;
    }
}

public class MallardDuck extends Duck {
    public MallardDuck(QuackBehavior qb) {
        quackBehavior = qb;
    }

    @Override
    public void display() {
        System.out.println("I'm a real Mallard duck");
    }
}

public class RubberDuck extends Duck {
    public RubberDuck(QuackBehavior qb) {
        quackBehavior = qb;
    }

    @Override
    public void display() {
        System.out.println("I'm a real Rubber duck");
    }
}

public class Main {
    public static void main(String[] args) {
        Duck mallard = new MallardDuck(new Quack());        
        mallard.performQuack(); // Quack!
        mallard.setQuackBehavior(new Squeak());
        mallard.performQuack(); // Squeak!
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

**Decorator**  
Decorator is a structural design pattern that lets you attach new behaviors to objects by placing these objects inside special wrapper objects that contain the behaviors.   

```java
abstract class Beverage {
    public abstract double cost();
}

class Coffee extends Beverage {
    public double cost() {
        return 100;
    }
}

abstract class AddinDecorator extends Beverage {
    protected Beverage beverage;

    public AddinDecorator(Beverage beverage) {
        this.beverage = beverage;
    }
}

class Milk extends AddinDecorator {
    public Milk(Beverage beverage) {
        super(beverage);
    }

    public double cost() {
        return beverage.cost() + 20;
    }
}

class Mocha extends AddinDecorator {
    public Mocha(Beverage beverage) {
        super(beverage);
    }

    public double cost() {
        return beverage.cost() + 40;
    }
}

public class Main {
    public static void main(String[] args) {
        Beverage drink =
            new Mocha(
                new Milk(
                    new Coffee()
                )
            );
        System.out.println(drink.cost()); // 160
    }
}
```

**Factory Method**  
Factory Method is a creational design pattern that provides an interface for creating objects in a superclass, but allows subclasses to alter the type of objects that will be created.  

```java
abstract class Pizza {
    String name;

    void prepare() {
        System.out.println("Preparing " + name);
    }

    void bake() {
        System.out.println("Baking " + name);
    }

    void cut() {
        System.out.println("Cutting " + name);
    }

    void box() {
        System.out.println("Boxing " + name);
    }

    String getName() {
        return name;
    }
}

abstract class PizzaStore {
    public Pizza orderPizza(String type) {
        Pizza pizza = createPizza(type);

        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }

    // FACTORY METHOD
    protected abstract Pizza createPizza(String type);
}

class NYStyleCheesePizza extends Pizza {
    public NYStyleCheesePizza() {
        name = "NY Style Cheese Pizza";
    }
}

class NYStylePepperoniPizza extends Pizza {
    public NYStylePepperoniPizza() {
        name = "NY Style Pepperoni Pizza";
    }
}

class NYPizzaStore extends PizzaStore {

    @Override
    protected Pizza createPizza(String type) {
        if (type.equals("cheese")) {
            return new NYStyleCheesePizza();
        } else if (type.equals("pepperoni")) {
            return new NYStylePepperoniPizza();
        }
        return null;
    }
}

class ChicagoStyleCheesePizza extends Pizza {
    public ChicagoStyleCheesePizza() {
        name = "Chicago Style Cheese Pizza";
    }

    @Override
    void cut() {
        System.out.println("Cutting the pizza into square slices");
    }
}

class ChicagoPizzaStore extends PizzaStore {

    @Override
    protected Pizza createPizza(String type) {
        if (type.equals("cheese")) {
            return new ChicagoStyleCheesePizza();
        }
        return null;
    }
}

public class PizzaTestDrive {
    public static void main(String[] args) {

        PizzaStore nyStore = new NYPizzaStore();
        PizzaStore chicagoStore = new ChicagoPizzaStore();

        Pizza pizza = nyStore.orderPizza("cheese");
        System.out.println("Ordered a " + pizza.getName());

        System.out.println();

        pizza = chicagoStore.orderPizza("cheese");
        System.out.println("Ordered a " + pizza.getName());
    }
}
```

**Abstract Factory**  
Abstract Factory is a creational design pattern that lets you produce families of related objects without specifying their concrete classes.  

```java
abstract class IngredientFactory {
    Dough getDough();
    Sauce getSauce();
    // more abstract methods
}

class NewyorkPizzaIngredientFactory extends IngredientFactory {
    public Dough getDough() {
        return new ThinCrustDough();
    }

    public Sauce getSauce() {
        return new PlumTomatoSauce();
    }
}

interface Sauce { }
interface Dough { }

public class ThinCrustDough implements Dough { }
public class PlumTomatoSauce implements Sauce { }

public class Main {
    public static void main(String[] args) {
        IngredientFactory ingredientFactory = new NewyorkPizzaIngredientFactory();
        Dough dough = ingredientFactory.getDough(); // Newyork centric dough
        Sauce sauce = ingredientFactory.getSauce(); // Newyork centric sauce
    }
}

```

**Singleton**  
Singleton is a creational design pattern that lets you ensure that a class has only one instance, while providing a global access point to this instance.  

```java
public class Logger {
    private static Logger singleton;
    private Logger() {
        // constructor code goes here
    }

    public static Logger getLogger() {
        if(singleton == null) {
            singleton = new Logger();
        }
        return singleton;
    }

    public void log() {
        System.out.println("Logging...");
    }
}

public class Main {
    public static void main(String[] args) {
        Logger logger1 = Logger.getLogger();
        Logger logger2 = Logger.getLogger();

        System.out.println(logger1 == logger2); // true
    }
}
```

**Command**  
Command is a behavioral design pattern that turns a request into a standalone object. This object contains all the information about the request: what action to perform, when to perform it, and which object it acts upon.   

```java
public class Light {
    public void on() {
        System.out.println("Light is on");
    }

    public void off() {
        System.out.println("Light is off");
    }
}

public interface Command {
    void execute();
}

public class LightOnCommand implements Command {
    Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        this.light.on();
    }
}

public class LightOffCommand implements Command {
    Light light;

    public LightOffCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        this.light.off();
    }
}

class Remote {
    private Command[] commands;

    public Remote() {
        this.commands = new Command[1][2];
    }

    public void setCommands(int index, Command onCommand, Command offCommand) {
        this.commands[index][0] = onCommand;
        this.commands[index][1] = offCommand;
    }

    public void on(int index) {
        this.commands[index][0].execute();
    }

    public void off(int index) {
        this.commands[index][1].execute();
    }
}

public class Main {
    public static void main(String[] args) {
        Light light = new Light();
        Command lightOnCommand = new LightOnCommand(light);
        Command lightOffCommand = new LightOffCommand(light);
        Remote remote = new Remote();
        remote.setCommands(0, lightOnCommand, lightOffCommand);
        remote.on(1); // Light is on
    }
}
```

**Adaptor** 
Adapter is a structural design pattern that allows objects with incompatible interfaces to collaborate.  

```java
public interface Duck {
    void quack();
    void fly();
}

public interface Turkey {
    void gobble();
    void fly();
}

public class MallardDuck implements Duck {
    public void quack() {
        System.out.println("Quack quack!");
    }

    public void fly() {
        System.out.println("Flying long distance...");
    }
}

public class WildTurkey implements Turkey {
    public void gobble() {
        System.out.println("Gobble gobble!");
    }

    public void fly() {
        System.out.println("Flying short distance...");
    }
}

public class TurkeyAdapter implements Duck {
    private Turkey turkey;

    public TurkeyAdapter(Turkey turkey) {
        this.turkey = turkey;
    }

    public void quack() {
        this.turkey.gobble();
    }

    public void fly() {
        this.turkey.fly();
    }
}

public class Main {
    public static void main(String[] args) {
        Turkey wildTurkey = new WildTurkey();
        Duck duck = new TurkeyAdapter(wildTurkey);
        duck.quack(); // Gobble gobble!
        duck.fly(); // Flying short distance...
    }
}

```

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

**Template Method**  
Template Method is a behavioral design pattern that defines the skeleton of an algorithm in the superclass but lets subclasses override specific steps of the algorithm without changing its structure.    

```java
abstract class BaseParser {
    public void open() {
        System.out.println("OPEN");
    }

    public void close() {
        System.out.println("CLOSE");
    }

    public abstract void parseData();

    public void parse() {
        open();
        parseData();
        close();
    }
}

class CSVParser extends BaseParser {
    @override
    public void parseData() {
        System.out.println("PARSING CSV");
    }
}

class XMLParser extends BaseParser {
    @override
    public void parseData() {
        System.out.println("PARSING XML");
    }
}

public class Main {
    public static void main(String[] args) {
        BaseParser csvParser = new CSVParser();
        BaseParser xmlParser = new XMLParser();

        csvParser.parse();
        xmlParser.parse();
    }
}
```

---
