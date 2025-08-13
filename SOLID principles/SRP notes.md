### **SRP definition**

A class should have only **one reason to change** — meaning it should be responsible for a single, well-defined concern.

* Multiple methods can still belong to the same class if they change for the **same reason**.
* If different reasons for change are mixed, the class has **multiple responsibilities**.
* The term `Single Responsibility` is **subjective** and often sparks discussion. It’s about *responsibility* in the sense of **the cause of change**, not “number of methods” or “number of tasks” in a literal sense.
* 💡 **Stakeholder perspective shortcut:**
  Think about how many different people (stakeholders) might come to you asking for a change in this class.
  For example:

  * HR department might ask for changes in leave calculation logic.
  * Finance department might request changes in payroll logic.
  * Compliance department might need changes in tax rules.

  If multiple **unrelated departments** could request changes to the same class, you’re likely violating SRP.

---

### **Problem: Hidden coupling between methods**

Even if methods have different reasons to change, if they **depend on each other internally**, they become **tightly coupled**.

* Changing one method can break another, even though they address unrelated concerns.
* Because the dependency is *private* inside the class, it might be **hard to detect** during code reviews.

---

**Example of violation:**

```java
class Employee {
    void calculatePay() {
        updateOvertime(); // depends on unrelated logic
    }
    void updateOvertime() { /* overtime logic */ }
    void generateReport() { /* reporting logic */ }
    // Each method changes for different reasons
}
```

---

### **Refactoring for SRP**

Separate responsibilities into different classes or services, even if they share the same data model.

**Refactored example:**

```java
class Employee { /* employee data */ }

class PayrollService {
    void calculatePay(Employee e) { /* ... */ }
}

class OvertimeService {
    void updateOvertime(Employee e) { /* ... */ }
}

class ReportingService {
    void generateReport(Employee e) { /* ... */ }
}
```

---

### **Benefits**

**1. Change isolation**
SRP says: “One class/module should have only one reason to change.”
That means if the rules for one responsibility change, you don’t have to touch unrelated code.

Even without direct dependencies, putting two unrelated responsibilities in the same file/class means:

* You’ll have to open that file for multiple kinds of changes.
* Higher chance of **merge conflicts** in teams.
* Higher **cognitive load** when reading the code.

---

**2. Testability**
If a class does multiple things, you can’t easily test just one of them in isolation without also setting up or considering the others.
Splitting responsibilities makes unit tests **laser-focused** and reduces test setup complexity.

---

**3. Scalability over time**
Today there’s no dependency — tomorrow, a small “just add this” change can create one.
Once the responsibilities are tangled, **untangling later is painful**.
SRP helps **prevent** that mess from forming in the first place.

---

**4. Clarity of intent**
When a class has exactly one responsibility, its **name** and purpose are obvious.
When it has more, you end up with vague names like `Utils`, `Manager`, or `Handler` that hide complexity.

---

**5. Reusability**
Even if a class has some extra unrelated functionality and it seems harmless, it’s still better to separate it so the functionality can be used by other classes without copy-pasting.
But — **avoid over-engineering**:
Analyse reusability **beforehand** so you don’t prematurely split things that will never realistically be reused.

---

### **Common Misconceptions about SRP**

1. **“SRP means one method per class.”**
   No. SRP doesn’t mean your classes should be microscopic.
   You can have multiple methods as long as they all change for the *same* reason.

2. **“SRP means one class per file.”**
   That’s a style choice, not part of SRP. You can have multiple classes in a file if they share a single responsibility (though most modern style guides still prefer one per file for clarity).

3. **“If there’s no current coupling, SRP doesn’t matter.”**
   Coupling often appears over time. SRP is also about **future-proofing** and making it clear where changes belong.
   Without it, responsibilities can slowly creep into each other without anyone noticing.

4. **“SRP only applies to classes.”**
   It applies to *any* unit of code — functions, modules, services, microservices — not just classes.

5. **“SRP forces you to over-engineer.”**
   SRP is not about blindly splitting everything. It’s about **identifying separate reasons to change** and deciding when separation is worth the cost.
   In small scripts or prototypes, breaking SRP might be fine. In large, long-lived systems, it usually pays off.

6. **“If two responsibilities use the same data, they belong together.”**
   Not necessarily. Sharing the same data model (like `Employee`) doesn’t mean the logic for payroll and reporting should be in the same class. You can still separate them into services that use the same entity.

---


### **Important note**

Always confirm **what things are actually likely to change** in your system.
Don’t over-complicate code just because you *think* something *might* change — if it’s extremely unlikely for your project, SRP compliance there might just add unnecessary indirection.

---

**Analogy:**
SRP-compliant code is like **Lego bricks** — small, reusable, and easy to combine into many forms.
SRP-violating code is like a **big toy brick** — large and not very reusable.

* Lego bricks can be used to build **anything**, from a small toy to a life-sized car sculpture.
* Toy bricks can build only a few things, and making larger, complex structures is **much harder**.

SRP gives your code **flexibility** to adapt to change and be reused easily, without the pain of breaking unrelated parts.




