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

## 4. Serialization: Breaking and Fixing Singleton

So far, we saw that reflection can bypass Singleton’s private constructor.
But serialization introduces another subtle way to break Singleton.

**Problem:**

* Serialize a Singleton instance to disk/network.
* Deserialize it later.
* The JVM will create a *new* instance, violating the “single instance” rule.

### Demonstration: Singleton broken by serialization

**BrokenSingleton.java**

```java
import java.io.Serializable;

public class BrokenSingleton implements Serializable {
    private static final long serialVersionUID = 1L;

    private static final BrokenSingleton INSTANCE = new BrokenSingleton();

    private BrokenSingleton() {
        // private to prevent external instantiation
    }

    public static BrokenSingleton getInstance() {
        return INSTANCE;
    }

    public String id() {
        return "I am BrokenSingleton@" + System.identityHashCode(this);
    }
}
```

**TestBroken.java**

```java
import java.io.*;

public class TestBroken {
    public static void main(String[] args) throws Exception {
        BrokenSingleton s1 = BrokenSingleton.getInstance();

        // Serialize
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("broken.ser"))) {
            oos.writeObject(s1);
        }

        // Deserialize
        BrokenSingleton s2;
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("broken.ser"))) {
            s2 = (BrokenSingleton) ois.readObject();
        }

        System.out.println("s1 == s2 ? " + (s1 == s2)); // false -> broken
        System.out.println(s1.id());
        System.out.println(s2.id());
    }
}
```

**Output**

```
s1 == s2 ? false
I am BrokenSingleton@12345678
I am BrokenSingleton@87654321
```

👉 Reason: Deserialization bypasses our static field and creates a fresh instance.

---

### Fix: `readResolve()` method

Java serialization allows us to enforce canonical instances with `readResolve()`.

**FixedSingleton.java**

```java
import java.io.Serializable;

public class FixedSingleton implements Serializable {
    private static final long serialVersionUID = 1L;

    private static final FixedSingleton INSTANCE = new FixedSingleton();

    private FixedSingleton() {}

    public static FixedSingleton getInstance() {
        return INSTANCE;
    }

    public String id() {
        return "I am FixedSingleton@" + System.identityHashCode(this);
    }

    // Called after deserialization, replaces the object with the canonical one
    protected Object readResolve() {
        return INSTANCE;
    }
}
```

**TestFixed.java**

```java
import java.io.*;

public class TestFixed {
    public static void main(String[] args) throws Exception {
        FixedSingleton s1 = FixedSingleton.getInstance();

        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("fixed.ser"))) {
            oos.writeObject(s1);
        }

        FixedSingleton s2;
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("fixed.ser"))) {
            s2 = (FixedSingleton) ois.readObject();
        }

        System.out.println("s1 == s2 ? " + (s1 == s2)); // true -> fixed
        System.out.println(s1.id());
        System.out.println(s2.id());
    }
}
```

**Output**

```
s1 == s2 ? true
I am FixedSingleton@13572468
I am FixedSingleton@13572468
```

✅ `readResolve()` ensures deserialization doesn’t create duplicates.

---

### Best Fix: Enum Singleton

Enums are automatically serialization-safe and reflection-safe.

```java
public enum EnumSingleton {
    INSTANCE;

    public String id() {
        return "I am EnumSingleton@" + System.identityHashCode(this);
    }
}
```

---

## 5. GoF Singleton vs Spring “Singleton”

A frequent confusion: **Spring also uses the word “singleton,” but it means something different.**

| Aspect              | GoF Singleton                                    | Spring “Singleton”                               |
| ------------------- | ------------------------------------------------ | ------------------------------------------------ |
| Scope of uniqueness | One per JVM *ClassLoader*                        | One per *ApplicationContext*                     |
| Access              | `getInstance()` / enum constant                  | Dependency Injection (`@Autowired`, `getBean()`) |
| Enforced by         | Class logic (private constructor + static field) | IoC container caching                            |
| Multiple contexts?  | Still one instance                               | One per container (so multiple possible)         |

### Demonstration: Spring singleton is per container

**MyService.java**

```java
package demo;

public class MyService {
    public String id() {
        return "MyService@" + System.identityHashCode(this);
    }
}
```

**AppConfig.java**

```java
package demo;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyService(); // default: singleton scope
    }
}
```

**TestSpringSingleton.java**

```java
package demo;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class TestSpringSingleton {
    public static void main(String[] args) {
        var ctx1 = new AnnotationConfigApplicationContext(AppConfig.class);
        var ctx2 = new AnnotationConfigApplicationContext(AppConfig.class);

        MyService a1 = ctx1.getBean(MyService.class);
        MyService a2 = ctx1.getBean(MyService.class);
        MyService b1 = ctx2.getBean(MyService.class);

        System.out.println("ctx1 a1 == a2 ? " + (a1 == a2)); // true
        System.out.println("ctx1 a1 == ctx2 b1 ? " + (a1 == b1)); // false

        ctx1.close();
        ctx2.close();
    }
}
```

**Output**

```
ctx1 a1 == a2 ? true
ctx1 a1 == ctx2 b1 ? false
```

👉 Within one container: true singleton. Across containers: multiple instances.

---

## Practical Guidance

* For **pure Java** (outside Spring):
  Use Enum-based Singleton if you truly need JVM-wide uniqueness.

* For **Spring apps**:
  Rely on Spring’s default `@Bean` singleton scope. Don’t mix in GoF singleton unless JVM-wide uniqueness is absolutely required.

* For **distributed systems**:
  Neither GoF Singleton nor Spring Singleton enforces cluster-wide uniqueness — you need DB locks, leader election, or coordination services.

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




