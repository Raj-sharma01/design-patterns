# 📝 Notes on Command Pattern

## 🔹 Motivation
- Sometimes we need to decouple the requester of an action (the “Invoker”) from the object that performs it (the “Receiver”).
- Instead of calling methods directly, we wrap requests as objects (Commands).

Benefits:
- Uniform treatment of actions (store, queue, log, schedule them).
- Undo/Redo functionality.
- Composing/batching commands.
- Flexibility with callbacks and lambdas.

***

## 🔹 Definition (GoF)
Command Pattern: Encapsulate a request as an object, thereby letting you parameterize clients with queues, requests, and operations, and support undoable operations.

***

## 🔹 Core Participants
1. Command (Interface/Abstract class)
   - Declares execute() (and optionally undo()).
2. ConcreteCommand
   - Implements execute() by binding a Receiver with an action.
3. Receiver
   - The actual object that knows how to perform the work.
4. Invoker
   - Asks the Command to carry out the request.
5. Client
   - Creates the command, sets the receiver, assigns it to the invoker.

***

## 🔹 Basic Example
```java
// Command interface
interface Command {
    void execute();
}

// Receiver
class Light {
    void turnOn()  { System.out.println("Light ON"); }
    void turnOff() { System.out.println("Light OFF"); }
}

// Concrete Commands
class LightOnCommand implements Command {
    private final Light light;
    public LightOnCommand(Light light) { this.light = light; }
    public void execute() { light.turnOn(); }
}

class LightOffCommand implements Command {
    private final Light light;
    public LightOffCommand(Light light) { this.light = light; }
    public void execute() { light.turnOff(); }
}

// Invoker
class RemoteControl {
    private Command command;
    public void setCommand(Command command) { this.command = command; }
    public void pressButton() { command.execute(); }
}

// Client
public class CommandDemo {
    public static void main(String[] args) {
        Light livingRoomLight = new Light();
        Command on  = new LightOnCommand(livingRoomLight);
        Command off = new LightOffCommand(livingRoomLight);

        RemoteControl remote = new RemoteControl();
        remote.setCommand(on);
        remote.pressButton();   // Light ON
        remote.setCommand(off);
        remote.pressButton();   // Light OFF
    }
}
```

***

### 1. Naive Way (without Command Pattern)
```java
class Light {
    void turnOn()  { System.out.println("Light ON"); }
    void turnOff() { System.out.println("Light OFF"); }
}

class RemoteControl {
    private final Light light;
    RemoteControl(Light light) { this.light = light; }
    void pressOn()  { light.turnOn(); }
    void pressOff() { light.turnOff(); }
}

public class DemoNaive {
    public static void main(String[] args) {
        Light livingRoomLight = new Light();
        RemoteControl remote = new RemoteControl(livingRoomLight);
        remote.pressOn();   // Light ON
        remote.pressOff();  // Light OFF
    }
}
```

Works fine for a single device.
- But if we add more devices: Fan, TV, Stereo:
  - The RemoteControl will explode with pressFanOn(), pressTVOff(), etc.
  - Remote is tightly coupled to concrete devices (Light, Fan, etc).
  - Hard to add new actions (Dim Light, Increase TV Volume) — violates OCP.
  - No history (so no Undo/Redo).
  - Can’t batch multiple operations together (like a Macro command: “Turn off all lights”).

***

### 2. With Command Pattern
Introduce a Command interface with optional undo:

```java
interface Command {
    void execute();
    void undo();   // optional
}
```

Concrete commands wrap the receiver (Light):

```java
class LightOnCommand implements Command {
    private final Light light;
    LightOnCommand(Light light) { this.light = light; }
    public void execute() { light.turnOn(); }
    public void undo()    { light.turnOff(); }
}

class LightOffCommand implements Command {
    private final Light light;
    LightOffCommand(Light light) { this.light = light; }
    public void execute() { light.turnOff(); }
    public void undo()    { light.turnOn(); }
}
```

Invoker depends only on Command:

```java
class RemoteControl {
    private Command onCommand;
    private Command offCommand;

    void setCommands(Command onCommand, Command offCommand) {
        this.onCommand  = onCommand;
        this.offCommand = offCommand;
    }

    void pressOn()  { onCommand.execute(); }
    void pressOff() { offCommand.execute(); }
}
```

Usage:
```java
public class DemoCommand {
    public static void main(String[] args) {
        Light light = new Light();
        Command lightOn  = new LightOnCommand(light);
        Command lightOff = new LightOffCommand(light);

        RemoteControl remote = new RemoteControl();
        remote.setCommands(lightOn, lightOff);

        remote.pressOn();   // Light ON
        remote.pressOff();  // Light OFF
    }
}
```

***

### ✅ Problems Solved by Command Pattern
1. Decoupling
   - Remote doesn’t know about Light, Fan, or TV; it only knows Command.
2. Extensibility (OCP)
   - Want “DimLightCommand”? Add a new class; no change to Remote.
3. Undo Support
   - Every command can reverse itself if designed to capture prior state.
4. Batch (Macro)
   - Store commands in a List and execute them together (e.g., “Good Night Mode”).
5. History/Logging
   - Keep a stack of executed commands to enable undo or replay.

***

## 🔹 Undo Functionality
Add undo() to Command and track last executed command in the invoker.

```java
interface Command {
    void execute();
    void undo();
}

class LightOnCommand implements Command {
    private final Light light;
    public LightOnCommand(Light light) { this.light = light; }
    public void execute() { light.turnOn(); }
    public void undo()    { light.turnOff(); }
}

class LightOffCommand implements Command {
    private final Light light;
    public LightOffCommand(Light light) { this.light = light; }
    public void execute() { light.turnOff(); }
    public void undo()    { light.turnOn(); }
}

// Null Object for safety (optional but useful)
class NoCommand implements Command {
    public void execute() {}
    public void undo() {}
}

class RemoteControl {
    private Command command     = new NoCommand();
    private Command lastCommand = new NoCommand();

    public void setCommand(Command command) { this.command = command; }
    public void pressButton() {
        command.execute();
        lastCommand = command;
    }
    public void pressUndo() {
        lastCommand.undo();
    }
}
```

***

## 🔹 Batch / MacroCommand (a.k.a. Meta-Command)
Group multiple commands and execute them together; undo in reverse order.

```java
import java.util.ArrayList;
import java.util.List;

class MacroCommand implements Command {
    private final List commands = new ArrayList<>();
    public void add(Command command) { commands.add(command); }

    public void execute() {
        for (Command c : commands) c.execute();
    }

    public void undo() {
        for (int i = commands.size()-1; i >= 0; i--) {
            commands.get(i).undo();
        }
    }
}
```

Usage:
```java
Light livingRoomLight = new Light();
Light kitchenLight    = new Light();

MacroCommand partyMode = new MacroCommand();
partyMode.add(new LightOnCommand(livingRoomLight));
partyMode.add(new LightOnCommand(kitchenLight));

RemoteControl remote = new RemoteControl();
remote.setCommand(partyMode);
remote.pressButton();  // Turns on both lights
remote.pressUndo();    // Undo all in reverse
```

***

## 🔹 Lambdas in Java (Modern Command Pattern)
Since Command can be a functional interface (void execute()), we can use lambdas for simple cases.

```java
@FunctionalInterface
interface Command {
    void execute();
}

class Remote {
    private Command command;
    public void setCommand(Command c) { command = c; }
    public void press() { command.execute(); }
}

class Light {
    void turnOn()  { System.out.println("Light ON"); }
    void turnOff() { System.out.println("Light OFF"); }
}

public class LambdaCommandDemo {
    public static void main(String[] args) {
        Light light = new Light();
        Remote remote = new Remote();

        remote.setCommand(light::turnOn);
        remote.press();   // Light ON

        remote.setCommand(light::turnOff);
        remote.press();   // Light OFF
    }
}
```

This massively reduces boilerplate for simple use cases.
Note — In Java, a lambda expression is a concise way to represent an instance of a functional interface. While it provides a syntax that resembles a function, it is not a standalone function in the same way functions exist in purely functional programming languages.

***

## 🔹 Real-World Use Cases
1. GUI Buttons / Menu Items → each action can be a command.
2. Task Queues (e.g. job schedulers) → store commands and run later.
3. Undo/Redo in editors → commands with inverse actions.
4. Macro recording (Excel, IDEs) → replay commands.
5. Transactional systems → batch + rollback using undo.
6. Remote APIs (RPC, message queues) → serialize commands and send.
7. Spring Batch → Tasklet resembles a Command:
   - A Tasklet defines a single method: execute(...).
   - You implement it to encapsulate a piece of work (e.g., cleanup, processing a chunk).
   - The framework invokes execute(...) at runtime.

***

## 🔹 Advantages
- Decouples sender (Invoker) from receiver.
- Allows undo/redo, logging, replay.
- Commands can be composed.
- Enables deferred execution.
- With lambdas → super lightweight.

***

## 🔹 Downsides
- Adds extra classes/boilerplate (though lambdas reduce this).
- Undo requires careful tracking of state (not always trivial).
- Can get over-engineered for very simple cases.

***

## 🔹 Relation to Other Patterns
- Strategy vs Command:
  - Strategy = chooses how to do something.
  - Command = represents when and what action to perform.
- Memento + Command:
  - Memento can store state snapshots for advanced undo.
  - Together, used in editors/IDEs.

***

## 🔹 Extra Example: Job Queue Simulation
```java
import java.util.LinkedList;
import java.util.Queue;

class PrintCommand implements Command {
    private final String message;
    public PrintCommand(String message) { this.message = message; }
    public void execute() { System.out.println(message); }
}

public class QueueExample {
    public static void main(String[] args) {
        Queue jobQueue = new LinkedList<>();
        jobQueue.add(new PrintCommand("Job 1"));
        jobQueue.add(new PrintCommand("Job 2"));
        jobQueue.add(() -> System.out.println("Job 3 (Lambda)"));

        while(!jobQueue.isEmpty()) {
            jobQueue.poll().execute();
        }
    }
}
```

***

## ❓ Questions for Future Self
- How far should undo go? (What about external systems like databases?)
- Is a lambda-based approach enough, or should we use classes for clarity?
- How do we persist commands (serialize) for distributed job queues?
- How do we prevent “god object” Receivers where too many commands map to one class?
- how is it any different than unit of work pattern and cqrs pattern?
- How to design idempotent commands for distributed execution?
- How to represent commands as DTOs for persistence/transport (and version them)?
- Where to maintain history: invoker (stack) vs central store (event sourcing)?
- When to prefer Strategy over Command (e.g., no need for queue/undo/history)?

***

## ⚡ Problems with Serializable Commands

### 1. Capturing Context / State
- A command often needs context (the receiver object, DB connection, file handle, etc.).
- If you make it Serializable, you must serialize that context too.
- But receivers (Socket, ThreadPoolExecutor, EntityManager) are usually not serializable.
  - You’ll get NotSerializableException, or
  - You strip away the receiver and reattach it later, complicating design.

***

## ✅ How It’s Handled in Practice
- Don’t serialize receivers → Keep commands lightweight (just data + intent), and re-inject the receiver when deserialized.
- Use DTO-like commands: Instead of making the actual command serializable, create a simple “CommandMessage” (just parameters), and rebuild the executable command later.
- Alternative serialization: JSON, Protocol Buffers, Avro instead of raw Java serialization.
- Framework solutions:
  - In JMS, Kafka, Spring Batch, commands are typically represented as messages or steps with data, not full-blown serializable objects with logic.

### 📌 Example
```java
// Bad: Directly serializing Command with Receiver
import java.io.Serializable;
import java.io.FileWriter;
import java.io.IOException;

public class FileWriteCommand implements Command, Serializable {
    private final FileWriter writer; // Not serializable ❌
    private final String data;

    public FileWriteCommand(FileWriter writer, String data) {
        this.writer = writer;
        this.data = data;
    }

    public void execute() throws IOException {
        writer.write(data);
    }
}
```

This fails because FileWriter is not serializable.

```java
// Better: Serializable command message, receiver is injected at runtime ✅
import java.io.Serializable;
import java.io.FileWriter;
import java.io.IOException;

public class FileWriteCommandMessage implements Serializable {
    private final String filePath;
    private final String data;

    public FileWriteCommandMessage(String filePath, String data) {
        this.filePath = filePath;
        this.data = data;
    }

    // Factory to rebuild executable command at runtime
    public Command toCommand() throws IOException {
        FileWriter writer = new FileWriter(filePath, true);
        return () -> writer.write(data);
    }
}
```

Here, we serialize just the intent + data, not the execution context.

👉 So the main problem: Serializable Commands break down when commands depend on non-serializable receivers or context. Instead, modern systems decouple commands into data messages and reconstruct the execution later.

***

## ⚡ The Problem: State-Aware Undo
- Some commands are idempotent (don’t care about previous state), so undo is easy.
  - Example: InsertText("abc") → undo = DeleteText(3).
- But many commands depend on the old state to undo properly.
  - Example:
    - Command: DeleteText(5)
    - To undo it, you need to know what text was deleted (was it "Hello"? "World"?).

If the command doesn’t store that old state, undo is impossible.

***

## 🌀 Undo Strategies in Command Pattern
Adding undo functionality is one of the most practical uses of the Command Pattern. To undo correctly, we need to make commands state-aware.

### 1. Backup State Inside Command
Each command stores the necessary state before execution, so it can restore it later.

```java
public class DeleteTextCommand implements Command {
    private final StringBuilder doc;
    private final int length;
    private String backup; // captured before execution

    public DeleteTextCommand(StringBuilder doc, int length) {
        this.doc = doc;
        this.length = length;
    }

    @Override
    public void execute() {
        int start = doc.length() - length;
        backup = doc.substring(start); // save deleted text
        doc.delete(start, doc.length());
    }

    @Override
    public void undo() {
        doc.append(backup); // restore deleted text
    }
}
```

Works well for small state changes; can get heavy if state is large.

### 2. Memento Pattern (Snapshot Approach)
Keep a separate Memento object, and let the editor save/restore snapshots; commands trigger changes.

```java
class TextMemento {
    private final String state;
    public TextMemento(String state) { this.state = state; }
    public String getState() { return state; }
}

class TextEditor {
    private String text = "";
    public TextMemento save()             { return new TextMemento(text); }
    public void restore(TextMemento m)    { text = m.getState(); }
    public void setText(String newText)   { text = newText; }
    public String getText()               { return text; }
}
```

Clean separation; memory-heavy if snapshots are large.

### 3. Event Sourcing (Command History)
Store a log of commands; undo means applying an inverse operation.

```java
import java.util.ArrayList;
import java.util.List;

class BankAccount {
    private int balance = 0;
    private final List history = new ArrayList<>();

    public void execute(Command c) {
        c.execute();
        history.add(c);
    }

    public void undo() {
        if (!history.isEmpty()) {
            Command last = history.remove(history.size() - 1);
            last.undo(); // inverse operation
        }
    }
}
```

Memory-efficient (store deltas), fits distributed systems; inverse must be implemented per command.

***

## 🔄 State-Aware Undo in Real World
- Text Editors (VSCode, Word, Photoshop): Undo stack of commands or snapshots.
- Databases: Don’t “undo” — use rollback or compensating transactions.
- Distributed Systems: Undo often = send a compensating command (e.g., refund after failed order).

***

## ⚠️ Challenges
1. Memory Overhead: Large snapshots eat memory.
2. Granularity: Whole state vs delta changes.
3. Consistency: In multi-threaded or distributed systems, undo isn’t always deterministic.

--- 
