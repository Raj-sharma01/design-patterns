# **Open–Closed Principle (OCP)**


## **1️⃣ Meyer’s Original Definition (1988, *Object-Oriented Software Construction*)**

**Open** → *“Open for extension”*
A module can be extended — e.g., by adding new behavior, new data, or overriding methods — **without rewriting its existing source code**.

**Closed** → *“Closed for modification”*
Once the module’s interface and implementation are stable and placed in the baseline, it can be **used by others without fear** that changes for new features will break existing clients.
“Closed” here means *stable enough* that it won’t be edited to satisfy new use-cases — not simply “already in use”.

---

### **The Tension**

* **If it’s open** → you’re tempted to keep editing → risk accidental breakage in dependent modules.
* **If it’s closed** → it’s frozen and can’t adapt to new use-cases without copying/cloning code (which leads to divergence).

---

### **Two “Bad” Pre-OOP Solutions**

1. **Modify the original** (A → add new features) → risk breaking many stable clients (**Sisyphus syndrome**).
2. **Clone and adapt** (A → A′) → creates maintenance hell and configuration management headaches.

---

### **Meyer’s OOP Solution**

* Use **inheritance** to create `A′` from `A`, *overriding or adding only what’s needed*, without touching `A`’s tested code.
* Old clients continue using `A`; new clients use `A′`.

---

### **Important Caution**

OCP is for extending *healthy modules*, not fixing broken ones.
If a module is flawed, fix it; don’t create a subclass just to patch defects.

---

## **2️⃣ Modern Restatements of OCP**

> *“Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification.”* — Robert C. Martin (“Uncle Bob”)

**Key differences from Meyer’s era:**

* “Closed” now typically means **stable contract via abstraction**, not formal baseline control.
* “Extension” is done **by adding new code** (strategies, classes, plugins) — not modifying existing production code.
* Inheritance is just *one* of many tools — modern systems often prefer composition or registration for safety.

---

## **3️⃣ Modern Implementation Tools**

* **Inheritance & Polymorphism** (classic approach)
* **Polymorphism via interfaces**
* **Composition + delegation** (Strategy, State patterns)
* **Registration / plugin architectures**
* **Configuration-based wiring** (DI containers, module loaders)
* **Dynamic discovery** (ServiceLoader, reflection-based plugin loading)

---

## **4️⃣ Example Scenario (Meyer’s OCP)**

### Problem

Module `A` has clients `B`, `C`, `D`. New clients `B′`, `F′` need slightly different behavior.

**Bad Option 1:** Modify `A` → risk breakage to existing clients
**Bad Option 2:** Clone `A` as `A′` and adapt → code duplication

---

### **OCP Fix**

```java
// Original closed module
class A {
    void f() { /* original behavior */ }
}

// Open via extension
class APrime extends A {
    @Override void f() { /* new behavior for new clients */ }
}
```

Old clients remain unaffected; new clients use `APrime`.

---

### **Core Mental Model**

* **Closed** part = stable behavior existing clients rely on.
* **Open** part = an extension mechanism that lets you add new behavior **outside** the module (new subclasses, injected strategies, registered handlers).
* Extension happens *by adding code*, not editing existing modules.

---

## **5️⃣ What OCP is NOT**

* ❌ A reason to avoid fixing bugs in a module.
* ❌ “Never touch code again” — it’s about **isolating change** to extension points.
* ❌ “Must use inheritance” — that was the original tool; modern OCP uses many others.

---

## **6️⃣ Modern / Pragmatic OCP**

In practice:

* Some modification is inevitable (e.g., configuration, registry update).
* The key is to **localize changes** so they don’t ripple through many modules.

**Modern interpretation:**

> Design so that adding new features requires minimal, localized changes — ideally in one predictable place — and no breaking changes to stable code.

---

### **Stages of Design Evolution**

1. **Full Violation** — scattered changes in multiple files.
2. **Localized Violation** — one small “change point” (registry/config).
3. **Pure OCP** — discover/load/compose new behavior with **no code edits**.

---

## **7️⃣ Payment Processing Example**

### **Stage 1 – Scattered `if/else` (Full Violation)**

```java
class PaymentService {
    void process(String method) {
        if ("paypal".equals(method)) {
            // PayPal logic
        } else if ("stripe".equals(method)) {
            // Stripe logic
        }
    }
}
```

* Adding `ApplePay` → edit this class → risk regression and merge conflicts.

---

### **Stage 2 – Localized Change (Registry)**

```java
interface PaymentProcessor { void pay(); }

class PayPalProcessor implements PaymentProcessor { public void pay() { /* PayPal logic */ } }
class StripeProcessor implements PaymentProcessor { public void pay() { /* Stripe logic */ } }

class PaymentProcessorRegistry {
    static final Map<String, Supplier<PaymentProcessor>> registry = new HashMap<>();
    static {
        registry.put("paypal", PayPalProcessor::new);
        registry.put("stripe", StripeProcessor::new);
    }
    static PaymentProcessor get(String method) { return registry.get(method).get(); }
}

class PaymentService {
    void process(String method) {
        PaymentProcessor processor = PaymentProcessorRegistry.get(method);
        processor.pay();
    }
}
```

✅ Change localized to one file (registry).
⚠ Still not *pure* OCP — must edit registry for new types.

---

### **Stage 3 – Pure OCP (Dynamic Discovery / Plugins)**

```java
interface PaymentProcessor {
    String getMethodName();
    void pay();
}

class PayPalProcessor implements PaymentProcessor {
    public String getMethodName() { return "paypal"; }
    public void pay() { /* PayPal logic */ }
}

class StripeProcessor implements PaymentProcessor {
    public String getMethodName() { return "stripe"; }
    public void pay() { /* Stripe logic */ }
}

class PaymentProcessorRegistry {
    private final Map<String, PaymentProcessor> registry = new HashMap<>();
    public PaymentProcessorRegistry(ServiceLoader<PaymentProcessor> loader) {
        for (PaymentProcessor p : loader)
            registry.put(p.getMethodName(), p);
    }
    public PaymentProcessor get(String method) { return registry.get(method); }
}

class PaymentService {
    private final PaymentProcessorRegistry registry;
    public PaymentService(PaymentProcessorRegistry registry) { this.registry = registry; }
    void process(String method) { registry.get(method).pay(); }
}
```

* New type = add new implementation → drop in JAR/plugin.
* No edits to `PaymentService` or registry code.

---

## **8️⃣ Pure OCP Techniques**

| Technique                   | Description                                             | Example                   |
| --------------------------- | ------------------------------------------------------- | ------------------------- |
| Interfaces + Inheritance    | Define extension points via interfaces.                 | Shapes, PaymentProcessors |
| Dependency Injection        | Wire new implementations without code changes.          | Spring @Bean injection    |
| Strategy Pattern            | Swap algorithm at runtime.                              | Sorting or payment flows  |
| Plugin Architecture         | Load new modules dynamically.                           | Browser extensions        |
| Service Loader / Reflection | Discover implementations at runtime.                    | Java ServiceLoader        |
| Config-driven logic         | External config chooses implementation.                 | JSON/YAML rules           |
| Event-Driven / Pub-Sub      | New subscribers added without changing publisher logic. | Message bus               |

---

## **9️⃣ Semi / Pseudo OCP Compliance**

**Definition:** Some edits are still required, but they’re:

* Centralized in a single, predictable place.
* Low risk to unrelated code.

**Examples:**

* Central registry file.
* Single “factory” class that’s the *only* file touched for a new type.
* Config file edit.

---

## **🔟 Comparing Approaches**

| Approach               | OCP Status | Breakage Risk | Maintenance Effort |
| ---------------------- | ---------- | ------------- | ------------------ |
| Scattered conditionals | ❌ No       | 🔴 High       | 🔴 High            |
| Localized registry     | ⚠ Semi     | 🟡 Medium     | 🟡 Medium          |
| Plugin/discovery       | ✅ Yes      | 🟢 Low        | 🟢 Low             |

---

