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

Example: A User class should only handle user-related logic, while database related operations should be handled by a separate UserRepository class. 

```java
class Invoice {
    private double amount;

    Invoice(double amount) {
        this.amount = amount;
    }

    public void createInvoice() {
        System.out.println("Invoice create for amount " + this.amount);
    }
}

class InvoiceRepository {
    public void saveToDatabase() {
        System.out.println("Saving invoice to database.");
    }
}

class EmailService {
    public void sendNotification() {
        System.out.println("Invoice has been created successfully");
    }
}
```

---
