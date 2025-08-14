# Factory Pattern Notes

## 1. Why Factories? (Motivation)

In normal code, the client directly creates objects using `new`.

* This couples the client to concrete classes, which **violates the Dependency Inversion Principle (DIP)**:

  * High-level modules (client code) depend on low-level details (concrete classes).
* It also makes code harder to extend without editing existing, tested code — a potential **OCP violation**.

**Factory Pattern**: Instead of letting the client decide *which concrete class* to instantiate, we delegate that decision to a separate component (a factory).

* Centralizes object creation.
* Reduces duplication (`new PayPalPayment()` sprinkled everywhere).
* Localizes OCP violations (ideally in a single predictable place).
* Makes testing easier (swap implementations without changing clients).
* Handles **shared workflows** (e.g., logging, validation, setup before returning the object).
* Encapsulates **complex preparation logic** needed before an object is usable (instead of pushing it into the client).

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
```

**Pros:**

* Centralizes creation.
* Removes `new` from client code.
* Good when object creation needs extra setup.

**Cons:**

* Still uses `if/else` → violates OCP (adding a new payment method requires modifying `PaymentFactory`).
* Works best for small/simple cases.

---

## 3. Factory Method Pattern

Instead of a single static factory, we define a **creator class** with a method that subclasses override.

### (a) Unparameterized Factory Method

Each subclass decides what to instantiate.

```java
abstract class PizzaStore {
    public abstract Pizza createPizza(); // factory method

    public void orderPizza() {
        Pizza pizza = createPizza();
        pizza.prepare();
    }
}
```

* **Pros:** New pizza = new subclass, no changes to existing classes → closer to OCP.
* **Cons:** Explosion of subclasses, client still needs to know *which* factory to use.

---

### (b) Parameterized Factory Method

Factory method takes an argument to decide product.

```java
class SimplePizzaStore extends PizzaStore {
    public Pizza createPizza(String type) {
        if (type.equals("margherita")) return new MargheritaPizza();
        if (type.equals("pepperoni")) return new PepperoniPizza();
        throw new IllegalArgumentException("Unknown type");
    }
}
```

* **Pros:** Fewer subclasses, cleaner clients.
* **Cons:** Brings back `if/else` → partial OCP violation.

---

## 4. Registry + OCP

Use a map instead of `if/else`.

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

* More maintainable than `if/else`.
* Still not “pure OCP” since registry needs editing.

---

## 5. Factories, OCP & DIP

* **DIP:** Client depends on abstractions (`Payment`, `Pizza`) not concrete classes.
* **OCP:** Factories *localize* object-creation logic so code changes are minimized.

---

## 6. Dependency Injection & Factories

* Choosing *which factory to use* is often left to **Dependency Injection frameworks**.
* Example: DI container injects `PayPalPayment` at runtime instead of hardcoding it.

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
6. Should I think of factories more as a **DIP/OCP enabler**, or as a **practical place to handle setup, workflow, and prep logic**?
