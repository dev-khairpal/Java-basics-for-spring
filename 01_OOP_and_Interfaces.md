# 📘 Module 1: OOP & Interface-Driven Design

> **Goal:** Master object modeling, encapsulation, access modifiers, packages, subclasses, abstract classes, and the golden rule of Spring: *"Program to an interface, not an implementation."*

---

## 1. Java Basics Refresher
* **Definition:** The fundamental syntax of Java including primitive types (`int`, `double`, `boolean`), reference types (`String`), conditional statements (`if-else`, `switch`), and loops (`for`, `while`).
* **Spring Relevance:** All custom business rules inside your services are expressed using these core language basics.

---

## 2. Classes & Objects

### 📖 Definitions
* **Class:** A template or blueprint that defines the variables (state) and methods (behavior) that objects created from it will possess.
* **Object (Instance):** The actual, physical entity created in computer memory based on the class blueprint.

### 💡 Real-World Analogy
* **Class:** The architectural blueprint of a house drawn on paper.
* **Object:** The actual brick-and-mortar house constructed on a street (`new House()`). You can build 50 houses from 1 blueprint.

```java
// Blueprint (Class)
public class User {
    private String name;
    
    public User(String name) {
        this.name = name;
    }
}

// In-memory Instances (Objects)
User user1 = new User("Alice");
User user2 = new User("Bob");
```

### 🧠 Spring Relevance
In Spring, your Controllers, Services, and Repositories are Java classes. Spring creates objects from these classes and manages them in memory as **Spring Beans**.

---

## 3. Encapsulation: What "Restricting Access" Really Means

### 📖 Definition
**Encapsulation** is the practice of hiding internal data (using `private` fields) and providing **controlled / restricted access** through `public` methods (`getters` and `setters`).

### ❓ *"If getters and setters allow access, how are they 'restricting' access?"*
"Restricting" does not mean completely blocking access. It means **guarding access on the class's own terms** instead of allowing raw, direct modification.

### 💡 The Bouncer Analogy:
* **Public field (No encapsulation):** Your house has no front door. Anyone from the street can walk straight into your bedroom and rearrange your furniture.
* **Encapsulation (Private field + Getters/Setters):** Your house has a security guard at the door. People can enter, **but only if they obey the guard's rules**.

---

### 🛡️ Four Ways Getters/Setters Restrict & Control Access:

#### 1. Restricting by Adding Validation Rules
```java
public class BankAccount {
    private double balance; // 🔒 Locked from direct tampering

    // 🛡️ Restricting modifications to only positive, valid numbers:
    public void setBalance(double balance) {
        if (balance < 0) {
            throw new IllegalArgumentException("Balance cannot be negative!");
        }
        this.balance = balance;
    }

    public double getBalance() {
        return this.balance;
    }
}
```

#### 2. Restricting to "Read-Only" (Omitting the Setter)
If a field has a `getter` but **no `setter`**, external code can read the value but can **never** overwrite it:
```java
public class User {
    private final Long id;
    private final LocalDateTime createdAt;

    public User(Long id) {
        this.id = id;
        this.createdAt = LocalDateTime.now();
    }

    public Long getId() { return this.id; } // ✅ Allowed to read
    public LocalDateTime getCreatedAt() { return this.createdAt; } // ✅ Allowed to read
    // ❌ NO setters exist! The ID and creation time can never be modified.
}
```

#### 3. Restricting Formats (Data Transformation)
```java
public class Customer {
    private String creditCard = "1234567890123456";

    // 🛡️ Restrict exposure by only returning masked data:
    public String getMaskedCreditCard() {
        return "****-****-****-" + creditCard.substring(12);
    }
}
```

#### 4. Protecting Lists from External Mutation
```java
public class Order {
    private List<String> items = new ArrayList<>();

    // 🛡️ Return an unmodifiable view so callers cannot .clear() or .add() illegally:
    public List<String> getItems() {
        return Collections.unmodifiableList(this.items);
    }
}
```

---

## 4. Packages: How Java Groups Files

### 📖 Definition
A **Package** in Java is simply a **folder** on your file system used to group related `.java` files together and prevent naming collisions.

### 📁 Project Folder Structure Example:
```
src/
└── com/
    └── myapp/
        ├── payment/                  <-- Package: com.myapp.payment
        │   ├── PaymentGateway.java
        │   ├── StripeGateway.java
        │   └── InternalTaxHelper.java
        │
        └── user/                     <-- Package: com.myapp.user
            └── UserController.java
```

### Declaring Packages in Code:
At the very top of every `.java` file, you write the package declaration:

```java
// Inside src/com/myapp/payment/StripeGateway.java
package com.myapp.payment;

public class StripeGateway {
    // ...
}
```

---

## 5. Subclasses: Inheritance with `extends`

### 📖 Definitions
* **Superclass (Parent Class):** The base class containing common fields and methods.
* **Subclass (Child Class):** A class that inherits from a superclass using the **`extends`** keyword.

```
          ┌───────────────────────────┐
          │   Parent (Superclass)     │
          │   BankAccount             │
          │   protected double balance│
          └─────────────┬─────────────┘
                        │
                  extends (inherits)
                        │
          ┌─────────────▼─────────────┐
          │   Child (Subclass)        │
          │   SavingsAccount          │
          └───────────────────────────┘
```

### How to write Parent and Subclass in Code:

**1. Parent Class (`BankAccount.java`):**
```java
package com.myapp.bank;

public class BankAccount {
    // 🛡️ 'protected' allows subclasses to access this directly!
    protected double balance;

    public BankAccount(double initialBalance) {
        this.balance = initialBalance;
    }
}
```

**2. Child Subclass (`SavingsAccount.java` in another package):**
```java
package com.myapp.accounts; // 👈 Notice: Different package/folder!

import com.myapp.bank.BankAccount;

// 👇 'extends BankAccount' makes this a Subclass
public class SavingsAccount extends BankAccount {

    public SavingsAccount(double initialBalance) {
        super(initialBalance); // Calls the parent constructor
    }

    public void applyInterestRate(double rate) {
        // ✅ WORKS! Because 'balance' is protected in parent BankAccount,
        // this child subclass can access it directly even from a different package!
        this.balance = this.balance + (this.balance * rate);
    }
}
```

### ❓ Does the Package Root have to be the same for Subclasses?
**No, absolutely not!** A subclass can exist anywhere:
1. In the **same package** (`com.myapp.bank`)
2. In a **different sub-package** (`com.myapp.accounts`)
3. In a **completely different root** (`org.finance.services`)
4. Or inside a **completely different external JAR / library** (like Spring Boot itself)!

#### 💡 Example: Extending an external library from a different root
```java
// File: src/com/myproject/security/JwtFilter.java
package com.myproject.security; // 👈 Your project root: 'com'

// 👈 External Spring Framework root: 'org'
import org.springframework.web.filter.OncePerRequestFilter;

// 👇 Your class is a subclass of an external Spring class!
public class JwtFilter extends OncePerRequestFilter {
    // You inherit and override Spring's filter behavior here
}
```

### 🔑 The Only Rules to Create a Subclass:
1. **`extends ParentName`** must be on your class declaration.
2. The parent class must be **`public`** (so other packages can see it).
3. The parent class must **NOT** be marked `final` (a `final class` blocks inheritance).
4. If the parent is in a different package or external library, you simply add an **`import path.to.ParentClass;`** at the top.

---

## 6. Access Modifiers: The Complete Reference

Access modifiers define the visibility scope of classes, variables, constructors, and methods.

| Modifier | Same Class | Same Package | Subclass (Anywhere) | Everywhere | Analogy | Spring Boot Example |
| :--- | :---: | :---: | :---: | :---: | :--- | :--- |
| **`public`** | ✅ | ✅ | ✅ | ✅ | **Public Billboard** (Visible to the entire world) | `@GetMapping` REST endpoints, public service methods |
| **`protected`** | ✅ | ✅ | ✅ | ❌ | **Family Heirloom** (Visible in same folder + all child subclasses) | Base template methods in abstract classes |
| *(default)* | ✅ | ✅ | ❌ | ❌ | **Neighborhood Noticeboard** (Only inside the same folder/package) | Internal package helper classes (omit `public`) |
| **`private`** | ✅ | ❌ | ❌ | ❌ | **Personal Diary** (Visible only inside this class `{ }`) | Model fields (`private Long id;`), helper methods |

### Visual Code Summary:
```java
package com.myapp.demo;

public class ExampleClass {
    public int a = 1;       // 🟢 Accessible anywhere in the whole app
    protected int b = 2;    // 🟡 Accessible by same package + any subclass ('extends ExampleClass')
    int c = 3;              // 🟠 (default) Accessible ONLY inside package 'com.myapp.demo'
    private int d = 4;      // 🔴 Accessible ONLY inside ExampleClass itself
}
```

---

## 7. Abstract Classes vs. Interfaces: Deep Dive

### 💡 The Core Intuition

```
┌────────────────────────────────────────┐       ┌────────────────────────────────────────┐
│         ABSTRACT CLASS                 │       │              INTERFACE                 │
│   "A Half-Built House Blueprint"       │       │      "A Pure Signed Contract"          │
├────────────────────────────────────────┤       ├────────────────────────────────────────┤
│ Provides SOME ready-made code          │       │ Provides ZERO ready-made code          │
│ (foundation, walls), but leaves        │       │ (just lists rules: "You must implement │
│ specific rooms for children to finish. │       │ methods X, Y, and Z").                 │
│                                        │       │                                        │
│ 👉 Relationship: "IS-A" (Family)       │       │ 👉 Relationship: "CAN-DO" (Capability) │
│ (A Dog IS-AN Animal)                   │       │ (An Invoice, Photo & Ticket CAN-DO     │
│                                        │       │  Printable)                            │
└────────────────────────────────────────┘       └────────────────────────────────────────┘
```

---

### 1. Abstract Class: The Incomplete Blueprint

* **Definition:** A class that **cannot be instantiated with `new`** because it is intentionally incomplete. It provides common, reusable logic for children while forcing them to complete abstract method blanks.
* **Analogy (Half-Built House):** A builder builds the foundation, plumbing, and roof for 10 houses. But the builder leaves the kitchen design blank and says: *"You get the foundation and roof for free, but you MUST design your own kitchen."*

```java
// ❌ CANNOT do: new PaymentProcessor() (It's incomplete!)
public abstract class PaymentProcessor {

    // 1. REUSABLE SHARED CODE (Children get this for free without rewriting)
    public void printAuditReceipt(double amount) {
        System.out.println("[AUDIT LOG] Payment receipt generated for: $" + amount);
    }

    // 2. ABSTRACT METHOD (No body {} - Every child MUST fill this blank)
    public abstract void processPayment(double amount);
}

// Concrete Child 1
public class StripeProcessor extends PaymentProcessor {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing $" + amount + " through Stripe API.");
    }
}

// Concrete Child 2
public class PaypalProcessor extends PaymentProcessor {
    @Override
    public void processPayment(double amount) {
        System.out.println("Redirecting to PayPal for $" + amount);
    }
}
```

---

### 2. Interface: The Pure Contract

* **Definition:** A pure contract containing only method declarations (what must be done), with no internal state or instance variables.
* **Analogy (The USB Socket):** A laptop USB port doesn't care if you plug in a mouse, keyboard, camera, or fan. It only demands: *"Follow the USB standard contract and you will work."*

```java
// The Contract
public interface Printable {
    void print(); // No body {}, just a requirement!
}

// Unrelated classes signing the same contract:
public class Invoice implements Printable {
    public void print() { System.out.println("Printing Tax Invoice #1023"); }
}

public class Photo implements Printable {
    public void print() { System.out.println("Printing 4K Glossy Photo"); }
}

public class BoardingPass implements Printable {
    public void print() { System.out.println("Printing Flight Ticket Gate B12"); }
}
```

---

### 3. Quick Decision Guide: Which One to Choose?

| Scenario | Choose | Why? |
| :--- | :---: | :--- |
| Classes are closely related family members sharing common variables & code | **Abstract Class** | Reuse shared state & common methods (`BaseAuditEntity`, `BaseController`) |
| You want to define a contract for what a class *can do*, regardless of its family | **Interface** | Maximum decoupling, multiple classes can implement (`PaymentGateway`, `JpaRepository`) |
| You need a class to adopt multiple behaviors | **Interface** | Java allows `implements InterfaceA, InterfaceB`, but only `extends ClassA` |

```java
// A class can inherit 1 parent, but sign MULTIPLE contracts:
public class SmartPhone extends ElectronicDevice implements Phone, Camera, GPSLocation {
    // 1 parent class (ElectronicDevice)
    // 3 capability contracts (Can make calls, take photos, track GPS)
}
```

---

## 8. Interfaces ⭐ (The Wall Power Socket Analogy)

### 📖 Definition
An **Interface** is a pure contract that specifies **WHAT** methods an object must provide, without dictating **HOW** they are implemented.

### 💡 Real-World Analogy:
* A wall power socket defines a standard contract (two round holes, 230V).
* The socket does not care if you plug in a fan, a microwave, or a laptop charger.
* You can swap the device at any time without changing the wall wiring.

```java
// 1. The Interface (Contract)
public interface PaymentGateway {
    boolean processPayment(double amount);
}

// 2. Implementation A
public class StripePaymentGateway implements PaymentGateway {
    @Override
    public boolean processPayment(double amount) {
        System.out.println("Processing $" + amount + " via Stripe API.");
        return true;
    }
}

// 3. Implementation B
public class PaypalPaymentGateway implements PaymentGateway {
    @Override
    public boolean processPayment(double amount) {
        System.out.println("Processing $" + amount + " via PayPal API.");
        return true;
    }
}
```

---

## 9. Polymorphism ⭐ (Program to an Interface)

### 💡 The "Universal Remote Control" Mental Model: Left vs. Right Side of `=`

When you see a line like this in Java:
```java
PaymentGateway gateway = new StripePaymentGateway();
```
Break it into two distinct parts:

```
    LEFT SIDE                           RIGHT SIDE
[ Compile-Time Type ]             [ Runtime Object in Memory ]
  PaymentGateway        gateway =    new StripePaymentGateway();
        │                                       │
        ▼                                       ▼
 "Universal Remote"                        "Actual Device"
 (Defines which buttons you can press)     (Defines how the device actually runs)
```

```
┌──────────────────────────────────────┐          ┌──────────────────────────────────────┐
│  LEFT SIDE: PaymentGateway (Remote)  │          │  RIGHT SIDE: new StripePaymentGateway│
├──────────────────────────────────────┤          ├──────────────────────────────────────┤
│ Button: [ processPayment(amount) ]   │ ──wires─►│ Hits Stripe REST API via HTTPS,      │
│                                      │   to     │ validates token, returns boolean.    │
└──────────────────────────────────────┘          └──────────────────────────────────────┘
```

#### Why not just write `StripePaymentGateway gateway = new StripePaymentGateway()`?
If you write `StripePaymentGateway gateway = ...`:
* Your entire application is now hard-glued (tightly coupled) to Stripe.
* If you want to switch to PayPal tomorrow, you have to find and modify 50 different files.
* If you write `PaymentGateway gateway = ...`, you only change the `new ...` instantiation in **1 place** (or let Spring do it automatically), and the rest of your 50 classes remain untouched!

```java
// ✅ Look how easily we swap payment processors without changing any business logic:
PaymentGateway gateway;

if (userCountry.equals("US")) {
    gateway = new StripePaymentGateway(); // Wires remote to Stripe
} else {
    gateway = new PaypalPaymentGateway(); // Wires same remote to PayPal
}

// 🎯 The caller code below never changes! It just presses the button:
gateway.processPayment(250.0);
```

---

### 🧠 Stack vs. Heap Memory: What `new` Actually Does

Understanding where things live in memory removes all confusion around references and objects:

```java
User user1 = new User("Alice");
```

```
     STACK MEMORY (Fast, Reference Variables)       HEAP MEMORY (Dynamic, Actual Objects)
    ┌────────────────────────────────────────┐     ┌──────────────────────────────────────┐
    │ Variable: user1                        │     │ Object at Address: 0x4A2F            │
    │ Value: 0x4A2F (Memory Address Pointer) ├────►│ ├─ name: "Alice"                     │
    └────────────────────────────────────────┘     │ └─ createdAt: 2026-09-14 20:30       │
                                                   └──────────────────────────────────────┘
```

1. **Stack Memory:** Stores local variable names (`user1`, `gateway`) containing the 64-bit memory address pointing to the Heap.
2. **Heap Memory:** Stores the actual heavyweight object data created by the `new` keyword.

---

### 🔍 `super` and `this` Demystified

When working with Subclasses and Inheritance, beginners often wonder: *"Why does `super(...)` have to be called?"*

#### 💡 The Foundation Analogy:
You cannot build the 2nd floor of a house without building the 1st floor foundation first!

```java
public class BankAccount {
    protected double balance;

    public BankAccount(double balance) {
        this.balance = balance; // 'this' means: THIS current class's field
    }
}

public class SavingsAccount extends BankAccount {
    private double interestRate;

    public SavingsAccount(double balance, double interestRate) {
        super(balance); // 👈 'super' calls the Parent constructor FIRST (builds foundation)
        this.interestRate = interestRate; // Then initializes child-specific fields
    }
}
```

```
When you run: new SavingsAccount(1000.0, 0.05);

1. [STEP 1] super(1000.0) executes -> Parent BankAccount initializes 'balance = 1000.0'
2. [STEP 2] this.interestRate executes -> Child SavingsAccount sets 'interestRate = 0.05'
3. Object is fully constructed and ready in Heap memory!
```

---

### 🧠 Why Spring Requires Interface-Driven Polymorphism:
1. **Dynamic Proxies & AOP:** Spring wraps your services in dynamic proxies to manage database transactions (`@Transactional`), security checks, and logging. The proxy implements your interface so the rest of your app doesn't know it's talking to a proxy!
2. **Spring Data JPA:** You create an interface (`UserRepository extends JpaRepository<User, Long>`), and Spring generates the actual implementation class automatically in memory at runtime!
3. **Unit Testing:** You can pass a `MockPaymentGateway` during tests instead of charging a real credit card.

---

## 📝 Self-Assessment Checklist
- [ ] Can you explain the **Universal Remote Control** analogy (Left side vs. Right side of `=`)?
- [ ] Do you know what happens in **Stack vs. Heap** memory when `new` is called?
- [ ] Do you understand why `super()` must run before child constructor logic?
- [ ] Can you explain why getters/setters give *controlled/restricted* access?
- [ ] Do you understand why we write `PaymentGateway p = new StripePaymentGateway()` instead of concrete classes?
