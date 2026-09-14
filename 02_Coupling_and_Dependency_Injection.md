# 📘 Module 2: Loose Coupling & Dependency Injection (DI)

> **Goal:** Understand why hardcoding `new` inside a class creates serious software design problems, and master **Constructor Dependency Injection** in pure Java before seeing Spring automate it.

---

## 1. Composition vs. Inheritance

### 📖 Definitions
* **Inheritance ("is-a" relationship):** A class derives from a parent (`Car is a Vehicle`). While useful, excessive inheritance creates rigid hierarchies that are hard to alter later.
* **Composition ("has-a" relationship):** A class contains references to other helper classes (`Car has an Engine`, `OrderService has a NotificationService`).

### 💡 Real-World Analogy
* **Inheritance:** Being born with your genetic traits (fixed at birth, hard to change).
* **Composition:** Using a smartphone with interchangeable phone cases and modular earbuds (flexible, easily swapped anytime).

> 🔑 **Software Rule:** *"Favor object composition over class inheritance."* In Spring applications, almost all components (Controllers, Services, Repositories) are wired together using **Composition**.

---

## 2. The Problem: Tight Coupling (The `new` Keyword Trap)

### 📖 Definition: Tight Coupling
**Tight Coupling** occurs when a class is directly dependent on concrete implementations of other classes. If one class changes, the other breaks or must be rewritten.

### ❌ The Anti-Pattern: Instantiating Dependencies Directly
Look at this `OrderService`:

```java
// ❌ BAD: Tightly Coupled to EmailService
public class OrderService {
    // Problem 1: OrderService creates its own dependency using 'new'
    private EmailService emailService = new EmailService();

    public void processOrder(String customerEmail, double amount) {
        System.out.println("Order processed for $" + amount);
        
        // Problem 2: What if management wants to switch from Email to SMS notifications?
        // You have to open OrderService and rewrite the code!
        emailService.sendEmail(customerEmail, "Your order is confirmed!");
    }
}
```

### Why Tight Coupling is Dangerous:
1. **Zero Flexibility:** You cannot switch to SMS notifications or push notifications without altering the `OrderService` class.
2. **Untestable:** You cannot write a unit test for `OrderService` without sending a real email over the internet.
3. **Violates Single Responsibility Principle:** `OrderService` should only care about order logic—not about how to construct and configure `EmailService`.

---

## 3. The Solution: Loose Coupling & Dependency Injection (DI)

### 📖 Definition: Dependency Injection (DI)
**Dependency Injection** is a software design pattern where a class does **NOT** instantiate the objects it needs (`new`). Instead, those dependencies are passed (injected) into the class from the outside—usually through its **constructor**.

### 💡 Real-World Analogy: The Lamp and the Power Plant
* **Tight Coupling:** Building an entire electrical power plant inside every desk lamp you manufacture. (Absurd and impossible to manage).
* **Dependency Injection:** Building the lamp with a power cord and plug. The electrical power (dependency) is **injected** from the outside wall socket.

---

## 4. Step-by-Step: Implementing Manual DI in Pure Java

### Step 1: Define the Contract (Interface)
```java
public interface NotificationService {
    void sendNotification(String recipient, String message);
}
```

### Step 2: Create Multiple Implementations
```java
public class EmailNotificationService implements NotificationService {
    @Override
    public void sendNotification(String recipient, String message) {
        System.out.println("📧 [Email Sent to " + recipient + "]: " + message);
    }
}

public class SmsNotificationService implements NotificationService {
    @Override
    public void sendNotification(String recipient, String message) {
        System.out.println("📱 [SMS Sent to " + recipient + "]: " + message);
    }
}
```

### Step 3: Inject via Constructor (Constructor Injection)
```java
// ✅ GOOD: Loosely Coupled and Dependency Injected
public class OrderService {
    // 1. Reference the INTERFACE, not a concrete class
    private final NotificationService notificationService;

    // 2. The dependency is injected through the constructor
    public OrderService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    public void processOrder(String customerContact, double amount) {
        System.out.println("Order processed for $" + amount);
        notificationService.sendNotification(customerContact, "Order of $" + amount + " confirmed!");
    }
}
```

### Step 4: The Assembler / Main Method
```java
public class Application {
    public static void main(String[] args) {
        // We can choose Email:
        NotificationService emailService = new EmailNotificationService();
        OrderService orderService1 = new OrderService(emailService);
        orderService1.processOrder("user@example.com", 150.0);

        // Or we can choose SMS WITHOUT CHANGING A SINGLE LINE of OrderService!
        NotificationService smsService = new SmsNotificationService();
        OrderService orderService2 = new OrderService(smsService);
        orderService2.processOrder("+1-555-0199", 75.0);
    }
}
```

---

## 5. Inversion of Control (IoC) & The Spring Bridge

### 📖 Definition: Inversion of Control (IoC)
* In traditional programming, **your code** controls the flow and calls `new` to instantiate dependencies.
* With **Inversion of Control**, the control is inverted: a **Framework / IoC Container** is responsible for creating objects, wiring their dependencies together, and managing their lifecycle.

| Aspect | Traditional Plain Java | Spring Framework (IoC) |
| :--- | :--- | :--- |
| **Object Creation** | You write `new OrderService(new EmailService())` | Spring creates objects automatically based on `@Service` annotations |
| **Dependency Wiring** | You manually pass objects into constructors | Spring scans constructors and injects required beans automatically |
| **Object Management** | Stored in local variables | Managed inside the Spring **IoC Container (`ApplicationContext`)** |

---

## 📝 Practice Check
Make sure you can answer these before moving to Module 3:
1. What is the difference between "is-a" and "has-a"?
2. Why is using `new` inside a business service considered an anti-pattern?
3. What is Constructor Dependency Injection?
4. How does Dependency Injection make unit testing easy?
