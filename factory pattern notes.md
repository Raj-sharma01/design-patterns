# Factory Pattern Notes

## 1. Why Factories? (Motivation)

* In normal code, the client directly creates objects using `new`.
* This couples the client to concrete classes, which **violates the Dependency Inversion Principle (DIP)**:

  * High-level modules (client code) depend on low-level details (concrete classes).
* It also makes code harder to extend without editing existing, tested code — a potential **OCP violation**.

**Factory Pattern**: Instead of letting the client decide *which concrete class* to instantiate, we delegate that decision to a separate component (a factory).

* This centralizes object creation.
* Reduces duplication (`new PayPalPayment()` sprinkled everywhere).
* Localizes OCP violations (ideally in a single predictable place).
* Makes testing easier (swap implementations without changing clients).

---

## 2. Simple Factory

A simple factory provides a static method that returns instances based on input.

```java
// Product interface
interface Payment {
    void pay(int amount);
}

// Concrete products
class PayPalPayment implements Payment {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using PayPal.");
    }
}
class CreditCardPayment implements Payment {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using Credit Card.");
    }
}

// Simple Factory
class PaymentFactory {
    public static Payment createPayment(String type) {
        if (type.equals("paypal")) return new PayPalPayment();
        if (type.equals("creditcard")) return new CreditCardPayment();
        throw new IllegalArgumentException("Unknown payment type");
    }
}

// Client
public class Client {
    public static void main(String[] args) {
        Payment payment = PaymentFactory.createPayment("paypal");
        payment.pay(100);
    }
}
```

**Pros:**

* Centralizes creation.
* Removes `new` from client code.

**Cons:**

* Still uses `if/else` → violates OCP (adding a new payment method requires modifying `PaymentFactory`).
* Works best for small/simple cases.

---

## 3. Factory Method Pattern

Instead of a single static factory, we define a **creator class** with a method that subclasses override.

### (a) Unparameterized Factory Method

Each subclass decides what to instantiate.

```java
// Product
interface Pizza {
    void prepare();
}

// Concrete products
class MargheritaPizza implements Pizza {
    public void prepare() { System.out.println("Preparing Margherita Pizza"); }
}
class PepperoniPizza implements Pizza {
    public void prepare() { System.out.println("Preparing Pepperoni Pizza"); }
}

// Creator
abstract class PizzaStore {
    public abstract Pizza createPizza(); // factory method

    public void orderPizza() {
        Pizza pizza = createPizza();
        pizza.prepare();
    }
}

// Concrete creators
class MargheritaPizzaStore extends PizzaStore {
    public Pizza createPizza() { return new MargheritaPizza(); }
}
class PepperoniPizzaStore extends PizzaStore {
    public Pizza createPizza() { return new PepperoniPizza(); }
}

// Client
public class Client {
    public static void main(String[] args) {
        PizzaStore store = new MargheritaPizzaStore();
        store.orderPizza();
    }
}
```

**Pros:**

* Adding a new pizza = new subclass, **no changes to existing classes** → closer to OCP.
* Client code depends only on `PizzaStore`, not on concrete `Pizza`.

**Cons:**

* One subclass per product type → can explode in number of classes.
* Choosing *which factory subclass to use* is still a problem (often solved by DI).

---

### (b) Parameterized Factory Method

The factory method itself accepts a parameter to decide which product to create.

```java
abstract class PizzaStore {
    public abstract Pizza createPizza(String type);

    public void orderPizza(String type) {
        Pizza pizza = createPizza(type);
        pizza.prepare();
    }
}

class SimplePizzaStore extends PizzaStore {
    public Pizza createPizza(String type) {
        if (type.equals("margherita")) return new MargheritaPizza();
        if (type.equals("pepperoni")) return new PepperoniPizza();
        throw new IllegalArgumentException("Unknown pizza type");
    }
}
```

**Pros:**

* More flexible, fewer subclasses.
* Client code stays clean.

**Cons:**

* Brings back `if/else` inside the factory method → partial OCP violation.
* Often improved by **using a registry or map** instead of hardcoded `if/else`.

---

## 4. Registry + OCP

Instead of hardcoding, use a map of keys to constructors (localizing OCP violation).

```java
class RegistryPizzaStore extends PizzaStore {
    private static Map<String, Supplier<Pizza>> registry = new HashMap<>();

    static {
        registry.put("margherita", MargheritaPizza::new);
        registry.put("pepperoni", PepperoniPizza::new);
    }

    public Pizza createPizza(String type) {
        Supplier<Pizza> supplier = registry.get(type);
        if (supplier == null) throw new IllegalArgumentException("Unknown pizza type");
        return supplier.get();
    }
}
```

**Note:** Adding new entries *still requires editing this class*.

* **Not “pure OCP”**.
* Can become more OCP-friendly if the registry is externalized (e.g., loaded via config or plugins).

---

## 5. Factories, OCP & DIP

* **DIP:** Client depends on abstractions (`Payment`, `Pizza`), not concrete classes.
* **OCP:** Factories *localize* object-creation logic so existing code changes are minimized.

  * Pure OCP: new products added without editing existing code (possible with plugin systems/config-driven registries).
  * Semi-OCP: registry/if-else is centralized in one file, not spread everywhere.

---

## 6. Dependency Injection & Factories

* Choosing **which factory to use** is often handled by **Dependency Injection (DI)** frameworks.
* Example:

  * Instead of hardcoding `new PayPalPayment()`, DI injects the right factory or object based on configuration.
* This solves the problem of *who decides the factory itself*.

---

## 7. Comparisons

| Pattern                              | Pros                        | Cons                      | OCP Compliance |
| ------------------------------------ | --------------------------- | ------------------------- | -------------- |
| **Simple Factory**                   | Centralized object creation | OCP violated (`if/else`)  | ❌              |
| **Factory Method (unparameterized)** | Extensible via subclassing  | Many subclasses needed    | ✅ (better)     |
| **Factory Method (parameterized)**   | Flexible, fewer subclasses  | Internal `if/else`        | ⚠️ Semi-OCP    |
| **Registry/Map**                     | Cleaner than if/else        | Still edit registry class | ⚠️ Semi-OCP    |
| **DI + Config/Plugins**              | Purest OCP, flexible        | More complex infra needed | ✅ Pure OCP     |

---

## 8. My Open Questions (for Future Self)

1. Are we just shifting the problem from *“which object to instantiate”* → *“which factory to use”*?
2. How is this pattern really different from **Strategy Pattern** (both choose between implementations)?
3. How is **Simple Factory with interface** any different from **Parameterized Factory Method**?
4. In practice, do registries really make code OCP-compliant, or just *centralize the violation*?
5. When is it worth the overhead of factories, and when is `new` perfectly fine?

---

