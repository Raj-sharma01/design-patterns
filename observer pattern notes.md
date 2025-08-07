
# Observer Pattern Notes

## Overview

The **Observer Pattern** is a behavioral design pattern that defines a one-to-many dependency between objects. When a **Subject (Observable)** changes its state, it **automatically notifies all registered Observers**. This promotes a flexible and extensible event-driven design by decoupling subjects from their dependents.

> **Note:** The Observer pattern is the conceptual foundation for event listener models used in GUIs and browsers. Event listeners, like DOM events in the browser, are practical implementations of this pattern tailored to user interface event handling.

---

## Key Concepts

- **Subject (Observable):** Holds state and a list of observers. Notifies them when state changes.
- **Observer:** Receives notifications and reacts accordingly.
- **Concrete Subject:** Implements the Subject interface, manages state, and triggers updates.
- **Concrete Observer:** Implements the Observer interface and defines the response to updates.

---

## What You Should Know About the Pattern

### 1. Relationship and Variants

- **In-Process vs Distributed:** Classic Observer pattern is local (in-memory). Pub/Sub extends the idea to distributed systems.
- **Push vs Pull Model:** In push, data is sent with the notification. In pull, observer queries the subject for data.
- **Synchronous vs Asynchronous Notification:** Updates can be immediate (blocking) or deferred (non-blocking).
- **Strong vs Weak References:** Subjects using strong references can cause memory leaks if observers are not removed. Weak references help avoid this.

  > **Example Memory Leak Cause:** Observers that are nullified elsewhere but still strongly referenced in the subject cannot be garbage collected, causing memory leaks. Using weak references for observer storage allows these to be collected when no longer referenced outside.

- **Typed vs Untyped Notifications:** Notifications can be tightly typed or generic, depending on system flexibility.

### 2. Why Use Observer Pattern?

- Decouples components and supports dynamic subscription.
- Enables reusable and modular designs for state change handling.
- Useful for in-process Pub/Sub style interaction without external tools.
- Foundation for event-driven UI, real-time updates, or any cascading changes.

### 3. Common Confusions and Pitfalls

- **Confusing with Pub/Sub:** Observer is usually in-memory and synchronous; Pub/Sub is distributed and often async.
- **Manual Notification:** Directly calling observers breaks the abstraction. Let the subject manage updates.
- **Order of Notification:** No guarantee unless explicitly enforced.
- **Memory Leaks:** Common if observers aren’t properly unregistered or weak references aren’t used.
- **Too Many Observers:** Technically supported, but can hurt performance without throttling or batching.
- **Interface Usage:** Not strictly required, but improves clarity and flexibility.
- **Still Relevant:** Used in frameworks like React (state updates), Node.js (`EventEmitter`), and Spring Boot (`@EventListener`).
- **Thread Safety:** Many real-world Observer implementations require thread-safe observer lists and notification mechanisms to avoid race conditions and concurrency issues. This can be managed using concurrent collections, synchronization, or making defensive copies of the observer list before notification.

---

## Key Variants and Code Examples

### 1. Basic Observer Pattern (Push Model)

```java
public interface Observer {
    void update(String data);
}

public interface Subject {
    void registerObserver(Observer o);
    void unregisterObserver(Observer o);
    void notifyObservers();
}

public class ConcreteSubject implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String state;

    public void setState(String data) {
        this.state = data;
        notifyObservers();
    }

    public String getState() { return state; }

    @Override
    public void registerObserver(Observer o) { observers.add(o); }

    @Override
    public void unregisterObserver(Observer o) { observers.remove(o); }

    @Override
    public void notifyObservers() {
        // Defensive copy for thread safety and to avoid ConcurrentModificationException
        List<Observer> observersSnapshot = new ArrayList<>(observers);
        for (Observer o : observersSnapshot) {
            o.update(state); // Push model
        }
    }
}

public class ConcreteObserver implements Observer {
    @Override
    public void update(String data) {
        System.out.println("Received update: " + data);
    }
}
````

---

### 2. Pull Model Variant

```java
public interface Observer {
    void update();
}

public class ConcreteObserver implements Observer {
    private ConcreteSubject subject;

    public ConcreteObserver(ConcreteSubject subject) {
        this.subject = subject;
    }

    @Override
    public void update() {
        String data = subject.getState(); // Pull data
        System.out.println("Pulled update: " + data);
    }
}
```

---

### 3. Weak Reference Observers (Memory Leak Prevention)

```java
private List<WeakReference<Observer>> observers = new ArrayList<>();

public void registerObserver(Observer o) {
    observers.add(new WeakReference<>(o));
}

@Override
public void notifyObservers() {
    for (Iterator<WeakReference<Observer>> it = observers.iterator(); it.hasNext();) {
        Observer observer = it.next().get();
        if (observer != null) {
            observer.update(...);
        } else {
            it.remove(); // Remove garbage collected observers
        }
    }
}
```

---

### 4. Asynchronous Notification

```java
ExecutorService executor = Executors.newSingleThreadExecutor();

public void notifyObserversAsync() {
    for (Observer o : observers) {
        executor.submit(() -> o.update(state)); // Async push
    }
}
```

---

### 5. Topic-Based Observer Pattern

```java
public interface Subscriber {
    void update(String topic, String message);
}

public interface Subject {
    void subscribe(String topic, Subscriber subscriber);
    void unsubscribe(String topic, Subscriber subscriber);
    void publish(String topic, String message);
}

public class TopicBasedNewsLetter implements Subject {
    private Map<String, List<Subscriber>> subscriptions = new HashMap<>();

    @Override
    public void subscribe(String topic, Subscriber subscriber) {
        subscriptions.computeIfAbsent(topic, k -> new ArrayList<>()).add(subscriber);
    }

    @Override
    public void publish(String topic, String message) {
        List<Subscriber> subs = subscriptions.get(topic);
        if (subs != null) {
            for (Subscriber s : subs) {
                s.update(topic, message);
            }
        }
    }

    // Unsubscribe logic omitted for brevity
}
```

Another implementation using `TimesNow`:


```java
public class TimesNow implements NewsLetter {
    Map<String, String> topicWiseData = new HashMap<>();
    Map<String, List<Subscriber>> subscriptions = new HashMap<>();


    @Override
    public void subscribe(String topic, Subscriber subscriber) {
        subscriptions.computeIfAbsent(topic, k -> new ArrayList<>()).add(subscriber);
    }


    @Override
    public void unsubscribe(String topic, Subscriber subscriber) {
        List<Subscriber> subs = subscriptions.get(topic);
        if (subs != null) {
            subs.remove(subscriber); // safer unsubscribe
        }
    }


    @Override
    public void sendNotification(String topic) {
        List<Subscriber> subs = subscriptions.get(topic);
        if (subs != null) {
           // Defensive copy for safety
           List<Subscriber> subsSnapshot = new ArrayList<>(subs);
            for (Subscriber subscriber : subsSnapshot) {
                subscriber.update(topic, topicWiseData.get(topic));
            }
        }
    }


    @Override
    public String getData(String topic) {
        return topicWiseData.get(topic);
    }


    @Override
    public void setData(String topic, String data) {
        topicWiseData.put(topic, data);
        sendNotification(topic);
    }


    @Override
    public List<String> getTopicsForSubscriber(Subscriber subscriber) {
        List<String> topics = new ArrayList<>();
        for (Map.Entry<String, List<Subscriber>> entry : subscriptions.entrySet()) {
            if (entry.getValue().contains(subscriber)) {
                topics.add(entry.getKey());
            }
        }
        return topics;
    }
}
```
---

## Summary Table of Variants

| Variant                   | Description                      | Use Case / Benefit                          |
| ------------------------- | -------------------------------- | ------------------------------------------- |
| Push Model                | Subject pushes data              | Quick updates with data included            |
| Pull Model                | Observer pulls data on notify    | Observer gets what it needs                 |
| Synchronous Notification  | Blocking updates                 | Simple control flow                         |
| Asynchronous Notification | Non-blocking with concurrency    | Improves responsiveness, avoids UI freezing |
| Strong Reference          | Default object reference         | Straightforward but risks memory leaks      |
| Weak Reference            | Weak references for GC safety    | Prevents leaks, useful in large systems     |
| Topic-Based               | Notify only interested observers | Scalable for categorized updates            |


---

## Additional Insights from Head First Design Patterns

### 1. Separation of Concerns and Reusability
- The Observer pattern **strongly decouples** the subject and observers.
- Observers can be **reused with different subjects** without modification.
- This **flexibility** allows observers to evolve independently of the subject’s implementation.

### 2. Push vs Pull Model Trade-offs (Deeper Explanation)
- **Push Model:**
  - Subject pushes all relevant state data to observers at notify time.
  - Pros: Observers get everything needed immediately.
  - Cons: May send unnecessary data; observers cannot control what they receive.
- **Pull Model:**
  - Subject notifies observers a change occurred.
  - Observers fetch (pull) only the data they need.
  - Pros: Reduced redundancy, more flexible for observers.
  - Cons: Tighter coupling; observers must know how to query subject state.

```java
class WeatherDisplayPush implements Observer {
    private float temperature;
    private float humidity;

    @Override
    public void update(float temperature, float humidity, float windSpeed, String windDirection) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }

    public void display() {
        System.out.println("Push Display -> Temperature: " + temperature + "°C, Humidity: " + humidity + "%");
    }
}
```

```java
class WeatherDisplayPull implements ObserverPull {
    private WeatherStationPull weatherStation;

    public WeatherDisplayPull(WeatherStationPull weatherStation) {
        this.weatherStation = weatherStation;
    }

    @Override
    public void update() {
        // Observer pulls only required data from subject
        float temperature = weatherStation.getTemperature();  // Observer must know subject's API => tighter coupling
        float humidity = weatherStation.getHumidity();
        display(temperature, humidity);
    }

    public void display(float temperature, float humidity) {
        System.out.println("Pull Display -> Temperature: " + temperature + "°C, Humidity: " + humidity + "%");
    }
}
```
## Coupling Comparison

| Aspect | Push Model | Pull Model |
| :-- | :-- | :-- |
| **Coupling Degree** | Looser coupling:<br>- Subject controls what data is sent.<br>- Observer only needs to implement `update` with known data signature.<br>- Observer does not depend on subject’s interface directly. | Tighter coupling:<br>- Observer must know and depend on subject’s interface to pull specific data.<br>- Observer needs a reference to the subject.<br>- Changes in the subject’s data API may break observers. |
| **Observer Complexity** | Simpler: just handle incoming data in update method. | More complex: observer queries subject for needed data. |
| **Subject’s Role** | More control: decides what data to push, can filter. | Less control: only signals change; observers decide what to pull. |
| **Flexibility for Observers** | Less flexible (receive all pushed info). | More flexible (observers choose what to consume). |
| **Flexibility to data changes** | Less flexible; all observers break if update signature changes | More flexible; observers unaffected unless they use new data |
| **Adding New Data Fields** | Subject may break observers if signature changes (argument list). | Adding new fields less likely to break observers if interface extends well. |
| **Extensibility** | Less extensible for very dynamic data; signature must match. | More extensible; observers can pull extended info as it becomes available. |
