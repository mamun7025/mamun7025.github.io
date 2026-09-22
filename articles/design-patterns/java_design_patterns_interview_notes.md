# Design Patterns — Java Interview Notes

## Quick Overview

The easiest way to remember the three GoF design pattern categories:

- **Creational** → How objects are **created**
- **Structural** → How objects are **connected**
- **Behavioral** → How objects **communicate / behave**

---

# 1. Creational Design Patterns

These patterns deal with **object creation**.

The most useful patterns to remember:

| Pattern | Easy Example | Remember As |
|---|---|---|
| Singleton | One configuration manager | Only ONE object |
| Factory | Payment processor selection | Which object should I create? |
| Builder | Building a complex User/Request object | Build step-by-step |
| Abstract Factory | Family of related objects | Factory of factories |
| Prototype | Clone an existing object | Copy an object |

---

## 1.1 Singleton Pattern

### Real-world example

Imagine an application has a **Configuration Manager**.

We don't want multiple configuration manager objects:

```text
Application → ConfigManager #1
Service     → ConfigManager #2
Controller  → ConfigManager #3
```

Instead, we want:

```text
              ┌─────────────────┐
Application ─→│ Configuration   │
Service ─────→│ Manager         │
Controller ──→│ ONE object      │
              └─────────────────┘
```

### Java Example

```java
class ConfigurationManager {

    private static ConfigurationManager instance;

    private ConfigurationManager() {
    }

    public static ConfigurationManager getInstance() {

        if (instance == null) {
            instance = new ConfigurationManager();
        }

        return instance;
    }
}
```

Usage:

```java
ConfigurationManager c1 =
        ConfigurationManager.getInstance();

ConfigurationManager c2 =
        ConfigurationManager.getInstance();

System.out.println(c1 == c2);
```

Output:

```text
true
```

### Interview memory trick

> **Singleton = Only ONE instance.**

### Spring Note

Spring beans are commonly **singleton-scoped by default**. However, Spring's singleton scope is managed by the Spring container and is not exactly the same implementation as the classic GoF Singleton pattern.

---

# 2. Factory Pattern

The Factory Pattern is very important for Java interviews.

### Real-world example

Imagine a payment system:

```text
                 Payment Request
                       ↓
                Payment Factory
                 /     |      \
                /      |       \
             VISA    PayPal    ApplePay
```

Instead of business code directly creating objects:

```java
new VisaPayment();
new PayPalPayment();
new ApplePayPayment();
```

the client asks the Factory:

```java
Payment payment = PaymentFactory.create("VISA");
```

### Java Example

```java
interface Payment {
    void pay();
}
```

```java
class VisaPayment implements Payment {

    public void pay() {
        System.out.println("Paying using Visa");
    }
}
```

```java
class PaypalPayment implements Payment {

    public void pay() {
        System.out.println("Paying using PayPal");
    }
}
```

Factory:

```java
class PaymentFactory {

    public static Payment create(String type) {

        if ("VISA".equals(type)) {
            return new VisaPayment();
        }

        if ("PAYPAL".equals(type)) {
            return new PaypalPayment();
        }

        throw new IllegalArgumentException("Unknown payment");
    }
}
```

Usage:

```java
Payment payment =
        PaymentFactory.create("VISA");

payment.pay();
```

### Interview memory trick

> **Factory = "Tell me what object you need; I'll create it."**

---

# 3. Builder Pattern

Builder is very useful when creating an object with **many optional parameters**.

### Problem

Without Builder:

```java
User user = new User(
    "Mamun",
    "mamun@email.com",
    33,
    "Dubai",
    "Java",
    true
);
```

It can be difficult to remember what each parameter means.

### Builder

```java
User user = User.builder()
        .name("Mamun")
        .email("mamun@email.com")
        .age(33)
        .city("Dubai")
        .skill("Java")
        .active(true)
        .build();
```

This is why you frequently see:

```java
@Builder
```

with Lombok in Spring Boot applications.

### Interview memory trick

> **Builder = Build a complicated object step-by-step.**

---

# 4. Structural Design Patterns

Structural patterns deal with:

> **How objects/classes are connected together.**

Important patterns:

| Pattern | Easy Example | Remember As |
|---|---|---|
| Adapter | Convert one API/interface to another | Translator |
| Decorator | Add functionality dynamically | Wrapper |
| Facade | Simplify a complex subsystem | One front door |
| Proxy | Control access to another object | Gatekeeper |
| Composite | Tree structure | Whole / part |
| Bridge | Separate abstraction and implementation | Two dimensions |

---

# 5. Adapter Pattern

This is one of the easiest patterns to remember.

### Real-world example

Suppose your application expects:

```java
PaymentService
```

But a third-party bank provides:

```java
BankPaymentAPI
```

Their interfaces don't match.

We create an **Adapter**.

```text
Your Application
       ↓
 PaymentService
       ↓
    Adapter
       ↓
BankPaymentAPI
```

The Adapter acts like a **translator**.

### Java Example

Your application interface:

```java
interface PaymentService {

    void pay(double amount);
}
```

Third-party API:

```java
class BankPaymentAPI {

    public void makePayment(double amount) {
        System.out.println(
            "Bank payment: " + amount
        );
    }
}
```

Adapter:

```java
class BankPaymentAdapter implements PaymentService {

    private BankPaymentAPI bankAPI =
            new BankPaymentAPI();

    @Override
    public void pay(double amount) {
        bankAPI.makePayment(amount);
    }
}
```

Usage:

```java
PaymentService payment =
        new BankPaymentAdapter();

payment.pay(1000);
```

### Interview memory trick

> **Adapter = Translator between incompatible interfaces.**

---

# 6. Decorator Pattern

Imagine a coffee:

```text
Coffee
  + Milk
  + Sugar
  + Whipped Cream
```

Instead of creating many classes such as:

```text
MilkCoffee
SugarCoffee
MilkSugarCoffee
MilkSugarCreamCoffee
...
```

we use Decorators.

```text
Coffee
  ↓
Milk Decorator
  ↓
Sugar Decorator
  ↓
Cream Decorator
```

Each wrapper adds functionality.

### Java Example Concept

```java
Coffee coffee = new BasicCoffee();

coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
coffee = new CreamDecorator(coffee);
```

A good real Java example is Java I/O:

```text
InputStream
    ↓
BufferedInputStream
    ↓
DataInputStream
```

Each wrapper adds functionality.

### Interview memory trick

> **Decorator = Add behavior/functionality by wrapping the original object.**

---

# 7. Facade Pattern

Imagine making a payment requires several steps:

```text
Payment
   ↓
Validate Account
   ↓
Check Balance
   ↓
Fraud Check
   ↓
Process Payment
   ↓
Send Notification
```

Without a Facade, the client needs to know about all these services.

With a Facade:

```text
Client
  ↓
PaymentFacade
  ↓
 ├── AccountService
 ├── FraudService
 ├── PaymentService
 └── NotificationService
```

The client simply calls:

```java
paymentFacade.pay(amount);
```

### Interview memory trick

> **Facade = One simple front door to a complicated system.**

---

# 8. Proxy Pattern

Think about a **security guard**.

```text
Client
  ↓
Security Guard / Proxy
  ↓
Real Service
```

The client does not directly access the real object.

The Proxy can:

- Check authentication
- Check authorization
- Log
- Cache
- Control access

### Spring Example

When using:

```java
@Transactional
public void transferMoney() {
    // ...
}
```

Spring can create a proxy around your bean to provide transactional behavior.

Conceptually:

```text
Client
   ↓
Spring Proxy
   ↓
Transaction handling
   ↓
Real Service
```

### Interview memory trick

> **Proxy = Gatekeeper in front of the real object.**

---

# 9. Behavioral Design Patterns

Behavioral patterns deal with:

> **How objects communicate and distribute responsibilities.**

Important patterns:

| Pattern | Easy Example | Remember As |
|---|---|---|
| Chain of Responsibility | Approval levels | Pass along the chain |
| Strategy | Different payment algorithms | Choose behavior |
| Observer | Notification/event system | Notify subscribers |
| Template Method | Common workflow | Fixed skeleton |
| Command | Queue an operation | Encapsulate request |
| State | Order/payment states | Behavior changes with state |
| Mediator | Chat room | Central communication |
| Iterator | Collection traversal | Traverse collection |

---

# 10. Chain of Responsibility Pattern

This is a very easy real-world example: **expense approval**.

Imagine an organization has different approval levels:

```text
Employee
   ↓
Manager       ($100)
   ↓
Director      ($1,000)
   ↓
VP            ($10,000)
   ↓
CEO           (above $10,000)
```

If an employee submits an expense of `$5,000`:

```text
Manager → Director → VP → handles it
```

Each handler asks:

1. Can I handle this?
2. If not, pass it to the next handler.

### Java Example

Base handler:

```java
abstract class ExpenseHandler {

    protected ExpenseHandler next;

    public void setNext(ExpenseHandler next) {
        this.next = next;
    }

    public abstract void approveExpense(double amount);
}
```

Manager:

```java
class Manager extends ExpenseHandler {

    @Override
    public void approveExpense(double amount) {

        if (amount <= 100) {
            System.out.println(
                "Manager approved: $" + amount
            );
        } else if (next != null) {
            next.approveExpense(amount);
        }
    }
}
```

Director:

```java
class Director extends ExpenseHandler {

    @Override
    public void approveExpense(double amount) {

        if (amount <= 1000) {
            System.out.println(
                "Director approved: $" + amount
            );
        } else if (next != null) {
            next.approveExpense(amount);
        }
    }
}
```

VP:

```java
class VP extends ExpenseHandler {

    @Override
    public void approveExpense(double amount) {

        if (amount <= 10000) {
            System.out.println(
                "VP approved: $" + amount
            );
        } else if (next != null) {
            next.approveExpense(amount);
        }
    }
}
```

Creating the chain:

```java
public class Main {

    public static void main(String[] args) {

        ExpenseHandler manager = new Manager();
        ExpenseHandler director = new Director();
        ExpenseHandler vp = new VP();

        manager.setNext(director);
        director.setNext(vp);

        manager.approveExpense(5000);
    }
}
```

Output:

```text
VP approved: $5000.0
```

### How it works

```text
manager.approveExpense(5000)
        ↓
Manager: Can I approve $5000?
        ↓ No
Director: Can I approve $5000?
        ↓ No
VP: Can I approve $5000?
        ↓ Yes
VP approves
```

### Interview memory trick

> **Chain of Responsibility = Pass the request through a chain until someone can handle it.**

### Common Spring Boot use cases

A similar approach can be seen in:

- HTTP request filters
- Authentication/authorization
- Validation pipelines
- Exception handling
- Logging
- Payment processing rules

For example:

```text
HTTP Request
     ↓
Authentication Filter
     ↓
Authorization Filter
     ↓
Logging Filter
     ↓
Validation Filter
     ↓
Controller
```

Each filter can process the request or pass it to the next handler.

---

# 11. Strategy Pattern

This is one of the most important patterns for Spring Boot interviews.

### Real-world example

Suppose an application supports multiple payment methods:

```text
Payment
 ├── Credit Card
 ├── PayPal
 └── Bank Transfer
```

Instead of creating a large `if/else` block:

```java
if (type.equals("CARD")) {
    // ...
} else if (type.equals("PAYPAL")) {
    // ...
} else if (type.equals("BANK")) {
    // ...
}
```

we create separate strategies.

```text
PaymentService
      ↓
   Strategy
   /   |   \
Card PayPal Bank
```

### Java Example

```java
interface PaymentStrategy {
    void pay(double amount);
}
```

Card:

```java
class CardPayment implements PaymentStrategy {

    public void pay(double amount) {
        System.out.println("Card payment");
    }
}
```

PayPal:

```java
class PaypalPayment implements PaymentStrategy {

    public void pay(double amount) {
        System.out.println("PayPal payment");
    }
}
```

Service:

```java
class PaymentService {

    private PaymentStrategy strategy;

    public PaymentService(
            PaymentStrategy strategy) {

        this.strategy = strategy;
    }

    public void pay(double amount) {
        strategy.pay(amount);
    }
}
```

Usage:

```java
PaymentService service =
        new PaymentService(
                new CardPayment()
        );

service.pay(1000);
```

### Interview memory trick

> **Strategy = Same job, different ways of doing it.**

---

# 12. Observer Pattern

This is very relevant to **event-driven architecture and Kafka**.

Imagine an order is created:

```text
Order Created
     ↓
   Event
     ↓
 ┌───────┬──────────┬────────────┐
 ↓       ↓          ↓
Email   SMS      Notification
```

The Order Service does not need to know exactly who is interested in the event.

It publishes:

```text
OrderCreated
```

Subscribers react to it.

### Interview memory trick

> **Observer = One event → notify many interested parties.**

### Spring Example

A similar concept can be implemented using:

```java
@EventListener
```

For distributed systems, Kafka and other messaging systems provide a broader event-driven version of the same idea.

---

# 13. Template Method Pattern

Imagine every payment follows the same high-level process:

```text
1. Validate
2. Authenticate
3. Process
4. Send notification
```

But the actual payment processing differs.

```text
Payment Template
      ↓
 ┌────┴─────┐
 ↓          ↓
Card       Bank
Payment    Payment
```

### Java Example

Base class:

```java
abstract class Payment {

    public final void process() {

        validate();
        authenticate();
        makePayment();
        notifyCustomer();
    }

    abstract void makePayment();

    void validate() {
        System.out.println("Validate");
    }

    void authenticate() {
        System.out.println("Authenticate");
    }

    void notifyCustomer() {
        System.out.println("Notify");
    }
}
```

Card:

```java
class CardPayment extends Payment {

    @Override
    void makePayment() {
        System.out.println("Card payment");
    }
}
```

Bank:

```java
class BankPayment extends Payment {

    @Override
    void makePayment() {
        System.out.println("Bank payment");
    }
}
```

### Interview memory trick

> **Template Method = Same steps, different implementation of some steps.**

---

# 14. Command Pattern

Think about a restaurant waiter.

The customer says:

```text
"Bring me a burger."
```

The waiter does not prepare the burger. The request can be packaged as a **Command** and sent to the appropriate receiver.

```text
Client
  ↓
Command
  ↓
Invoker
  ↓
Receiver
```

Command is useful for:

- Job queues
- Undo/redo
- Task scheduling
- Message processing

### Interview memory trick

> **Command = Turn a request/action into an object.**

---

# 15. State Pattern

Imagine an order lifecycle:

```text
Created
   ↓
Paid
   ↓
Shipped
   ↓
Delivered
```

The behavior depends on the current state.

For example:

```text
Created   → Can cancel
Paid      → Can refund
Shipped   → Cannot cancel
Delivered → Can review
```

Instead of having a huge block:

```java
if (state == CREATED) {
    // ...
} else if (state == PAID) {
    // ...
} else if (state == SHIPPED) {
    // ...
}
```

we can model each state separately.

### Interview memory trick

> **State = Object behaves differently depending on its current state.**

---

# 16. Most Important Patterns for Java/Spring Boot Interviews

You do not need to memorize all 23 GoF patterns equally.

Focus on these **10** first:

```text
                    DESIGN PATTERNS

CREATIONAL
│
├── ⭐ Factory
├── ⭐ Builder
└── Singleton


STRUCTURAL
│
├── ⭐ Adapter
├── ⭐ Facade
├── ⭐ Proxy
└── Decorator


BEHAVIORAL
│
├── ⭐ Strategy
├── ⭐ Chain of Responsibility
├── ⭐ Observer
└── Template Method
```

---

# 17. One-Line Memory Map

```text
Factory       → Create the right object

Builder       → Build a complex object step-by-step

Singleton     → Only one instance


Adapter       → Translator

Facade        → One simple front door

Proxy         → Gatekeeper

Decorator     → Add functionality by wrapping


Strategy      → Choose an algorithm/behavior

Chain         → Pass request through handlers

Observer      → Notify subscribers

Template      → Same workflow, customizable steps
```

---

# 18. Important Interview Distinction

A common interview question is:

> What is the difference between Factory, Strategy, and Chain of Responsibility?

The easiest way to remember:

```text
Factory
   ↓
"Which OBJECT should I create?"


Strategy
   ↓
"Which BEHAVIOR / ALGORITHM should I use?"


Chain of Responsibility
   ↓
"WHO should handle this REQUEST?"
```

### Example

```text
Factory:
PaymentFactory → creates CardPayment

Strategy:
PaymentService → uses CardPaymentStrategy

Chain of Responsibility:
Expense → Manager → Director → VP → CEO
```

### Final memory shortcut

```text
FACTORY  → CREATE
STRATEGY  → CHOOSE
CHAIN     → PASS
ADAPTER   → TRANSLATE
FACADE    → SIMPLIFY
PROXY     → CONTROL
DECORATOR → WRAP
OBSERVER  → NOTIFY
BUILDER   → CONSTRUCT
SINGLETON → ONE
TEMPLATE  → FOLLOW STEPS
```

---

## Quick Revision Table

| Pattern | Category | One-line memory |
|---|---|---|
| Singleton | Creational | One object |
| Factory | Creational | Create the right object |
| Builder | Creational | Build step-by-step |
| Adapter | Structural | Translate interfaces |
| Decorator | Structural | Wrap and add functionality |
| Facade | Structural | One front door |
| Proxy | Structural | Gatekeeper |
| Strategy | Behavioral | Choose behavior |
| Chain of Responsibility | Behavioral | Pass request along |
| Observer | Behavioral | Notify subscribers |
| Template Method | Behavioral | Same workflow, different steps |
| Command | Behavioral | Request as an object |
| State | Behavioral | Behavior depends on state |

---

## Best Patterns to Study First

For a Java/Spring Boot backend engineer, a good study order is:

```text
1. Strategy
2. Factory
3. Builder
4. Chain of Responsibility
5. Adapter
6. Proxy
7. Observer
8. Template Method
9. Facade
10. Singleton
```

These give you a strong foundation for understanding both **object-oriented design** and common patterns found in **Spring Boot enterprise applications**.
