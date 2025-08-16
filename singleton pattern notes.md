# Singleton Pattern — Notes

## 🔹 What is Singleton?

* **Definition (GoF)**: Ensure a class has **only one instance** and provide a **global point of access** to it.
* **Motivation**: Useful when exactly one object is needed to coordinate actions across the system.

  * Examples: configuration manager, logging service, connection pool, cache.

---

## 🔹 Why not just a global variable?

* **Global variable issues**:

  * The instance may be created even if never used → resource waste.
  * No control over initialization or destruction order.
  * Difficult to test, mock, or replace in different contexts.
* **Constants are fine as globals**: `public static final int MAX_USERS = 100;`

  * But for objects with *behavior* or *resources*, prefer controlled creation (Singleton).

---

## 🔹 Classic Implementations

### 1. Eager Initialization

```java
public class Singleton {
    private static final Singleton instance = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() { return instance; }
}
```

✅ Thread-safe
❌ May waste resources if never used

---

### 2. Lazy Initialization (synchronized)

```java
public class Singleton {
    private static Singleton instance;
    private Singleton() {}
    public static synchronized Singleton getInstance() {
        if (instance == null) instance = new Singleton();
        return instance;
    }
}
```

✅ Creates only when needed
❌ Synchronization cost

---

### 3. Double-Checked Locking

```java
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) instance = new Singleton();
            }
        }
        return instance;
    }
}
```

✅ Lazy + thread-safe
✅ Synchronization only first time

---

## 🔹 Enum Singleton (Best in Java)

```java
public enum ConfigManager {
    INSTANCE;

    private Properties props;

    ConfigManager() {
        props = new Properties();
        try {
            props.load(new FileInputStream("app.properties"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public String get(String key) {
        return props.getProperty(key);
    }
}
```

* ✅ Thread-safe (JVM guarantees enum initialization is atomic & safe)
* ✅ Prevents reflection & serialization issues
* ✅ Very concise

👉 The `INSTANCE` is created when the enum class is **first loaded** by the JVM’s classloader.

---

## 🔹 Singleton vs Spring Context

* **GoF definition**: Singleton = one instance per **JVM ClassLoader**.
* **Spring**: Singleton beans = one instance per **ApplicationContext**.

  * Multiple contexts = multiple “singletons.”
  * Useful for testing and modular apps.

📌 Example:

```java
ApplicationContext ctx1 = new AnnotationConfigApplicationContext(AppConfig.class);
ApplicationContext ctx2 = new AnnotationConfigApplicationContext(AppConfig.class);

MyService s1 = ctx1.getBean(MyService.class);
MyService s2 = ctx2.getBean(MyService.class);

System.out.println(s1 == s2); // false (different contexts)
```

---

## 🔹 Downsides of Singleton

* ❌ **Testing difficulty** → hard to mock a global instance
* ❌ **Hidden dependencies** → makes code less explicit
* ❌ **Tight coupling** → classes depend on concrete implementation
* ❌ **Debugging complexity** → global state makes issues harder to isolate

This is why many consider it an **anti-pattern** if overused.

---

## 🔹 Real-World Uses

* **Logging frameworks** (Log4j, SLF4J)
* **Configuration managers** (like `ConfigManager` example)
* **Thread pools / DB connection pools**
* **Device drivers / printer spoolers**
* **Spring-managed beans** (default scope is Singleton for performance and shared workflow)

---

## 🔹 Quotes & Notes

### Why is global object bad?

* Because of **early initialization**, **tight coupling**, and **hidden dependencies**.
* Constants are fine → they’re *values*, not *objects with lifecycle*.

### "One man’s constant is another man’s variable"

* Quote from **Alan Perlis (Epigrams on Programming, 1982)**.
* Means what you consider "constant" may change in someone else’s context.
* Relevance: storing things as global constants can backfire if they later need to vary → better to encapsulate.

---


## 🔹 History & Evolution of Singleton in Java

**Early days (pre-Java 5)**

* Implemented with `private static Singleton instance` + `getInstance()`.
* Relied on `synchronized` for thread safety → slow and error-prone.
* Vulnerable to issues with **reflection** and **serialization**.

**Java 5 (after JMM fixes)**

* **Double-checked locking** became reliable with `volatile`.
* Gave a balance of lazy initialization + thread safety + better performance.

**Modern Java (Java 5+)**

* **Enum Singleton** introduced: simplest, safest, handles reflection/serialization automatically.
* Became the recommended approach.

**Framework Era (Spring, Jakarta EE, etc.)**

* Frameworks manage object lifecycles.
* “Singleton” scope = one instance per container `ApplicationContext` (not JVM-wide).
* Often removes the need for hand-rolled singletons.

