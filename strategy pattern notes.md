
# 🎯 Strategy Pattern – Notes

## ✅ Overview

The **Strategy Pattern** is a behavioral design pattern that:

* Defines a family of interchangeable algorithms or behaviors.
* Encapsulates each one in its own class.
* Lets the context (i.e., the object using the behavior) switch between them at runtime.

This avoids hardcoding behavior and gives you flexibility to change how objects behave dynamically.

---

## 💡 What I’ve Learned

### 1. **Strategy Pattern vs. Passing Functions**

In languages like JavaScript, Python, or Java 8+, we can pass lambdas or methods to achieve dynamic behavior. So why Strategy?

* Strategy pattern provides:

  * A **clear and explicit family of interchangeable behaviors**.
  * Better **encapsulation** and **intent**—these are behaviors meant to be swapped.
  * **Extensibility**: Behaviors can hold state, logic, and support full OOP features.

Use callbacks/lambdas for small tasks. Use Strategy when you need **structure and scalability**.

---

### 2. **Strategy Pattern ≠ Dependency Injection (DI)**

* Strategy pattern is about **selecting behavior** dynamically.
* DI is a **way to provide** that behavior (e.g., via constructor/setter).
* They can work together:

  * You can inject a strategy via a DI container.
  * But you can also just `new` up the strategy if you don't need full DI.

In short: **Strategy is what; DI is how**.

---

### 3. **Runtime Swapping**

A key benefit: strategies can be swapped at runtime.

```java
duck.setFlyBehavior(new FlyWithWings());
duck.setFlyBehavior(new FlyNoWay());
```

This is much better than relying on inheritance or creating multiple subclasses just to tweak behavior.

---

### 4. **When to Use the Strategy Pattern**

Use it when:

* You have **multiple ways to perform a task**.
* You see **lots of conditionals** (`if`, `switch`) choosing between algorithms.
* You anticipate **adding new behaviors** over time.
* You want to **change behavior at runtime** without rewriting the object.

---

## 📘 Insights from “Head First Design Patterns”

* Introduced via the **SimUDuck** example: ducks with varying fly/quack behaviors.
* Promotes **composition over inheritance**.
* Follows the **Open/Closed Principle (OCP)**:

  * You can add new strategies without modifying existing classes.
* Helps **eliminate duplicate or scattered logic** by reusing strategy classes.

---


## 🧱 Code Template

```java
// Strategy interface
public interface Strategy {
    void execute();
}

// Concrete strategies
public class StrategyA implements Strategy {
    public void execute() {
        System.out.println("Executing Strategy A");
    }
}

public class StrategyB implements Strategy {
    public void execute() {
        System.out.println("Executing Strategy B");
    }
}

// Context class
public class Context {
    private Strategy strategy;

    public Context(Strategy strategy) {
        this.strategy = strategy;
    }

    public void setStrategy(Strategy strategy) {
        this.strategy = strategy;
    }

    public void perform() {
        strategy.execute();
    }
}
```

**Usage:**

```java
Context ctx = new Context(new StrategyA());
ctx.perform(); // Output: Executing Strategy A

ctx.setStrategy(new StrategyB());
ctx.perform(); // Output: Executing Strategy B
```
