---
type: pattern
tags: [java, constructors, maintainability, design]
level: junior
status: draft
java_versions: [21, 25]
date_created: 2026-05-01
date_revised: 2026-05-01
primary_source: "JLS §8.8.7 (explicit constructor invocation)"
secondary_sources: ["Effective Java, 3rd ed., Item 1"]
revisit_on: 2026-08-01
---

# Constructor chaining with this()

## Why this changes how I write code

Without this(), a class with multiple constructors duplicates initialization logic: adding one field requires editing every constructor that initializes state. With this(), only the canonical constructor changes. The others delegate to it and supply defaults for the parameters they omit. Without this pattern, a single field addition across four constructors creates four edit sites and four opportunities for divergence. One site diverges silently and objects enter production with inconsistent initial state.

## The pattern

One constructor is canonical: it initializes all fields and is the single authority on what constitutes a valid initial object state. Every other constructor calls this() as its first instruction, passing explicit defaults for the parameters it does not expose. The canonical constructor is typically the one with the most parameters.

this() must be the first instruction in the delegating constructor. The JVM requires that the initialization chain completes before any other operation on the object. this() and super() are mutually exclusive in the same constructor: a constructor that calls this() does not call super() directly, because the canonical constructor already calls super().

Applicability criteria:
- Two or more constructors share initialization logic.
- The class is expected to acquire new fields over time.
- At least one constructor can be expressed as a special case of the canonical constructor with fixed defaults for omitted parameters.

This pattern does not apply when constructors represent genuinely independent initialization paths with no shared logic. In that case, extracting a private initialization method is more explicit.

## Manifestation in real code

```java
// Without this(): three edit sites when a field is added
public class PaymentOrder {
    String currency;
    double amount;
    boolean processed;

    public PaymentOrder(String currency, double amount, boolean processed) {
        this.currency = currency;
        this.amount = amount;
        this.processed = processed;
    }

    public PaymentOrder(String currency, double amount) {
        this.currency = currency; // duplicated
        this.amount = amount;     // duplicated
        this.processed = false;
    }
}

// With this(): one edit site for any field addition
public class PaymentOrder {
    String currency;
    double amount;
    boolean processed;

    public PaymentOrder(String currency, double amount, boolean processed) {
        this.currency = currency;
        this.amount = amount;
        this.processed = processed;
    }

    public PaymentOrder(String currency, double amount) {
        this(currency, amount, false);
    }
}

// Adding String referenceId: edit only the canonical constructor,
// update the default in each this() call
public PaymentOrder(String currency, double amount) {
    this(currency, amount, false, null);
}
```

The bytecode of this() is an invokespecial directed at the canonical constructor, without a preceding invokespecial to Object."init": the canonical constructor handles the super() call. iconst_0 encodes the boolean literal false passed as a default.

## Production signal

A class with duplicated constructor initialization diverges after a field addition: one constructor initializes the new field, the others do not. Objects created through different constructors enter production with inconsistent state. In a payment system, an order created via the convenience constructor carries a null referenceId while one created via the canonical constructor does not. Downstream reconciliation fails for a subset of orders without a clear creation-path audit trail.

## Connections

- Underlying mechanism: [[default-constructor-implicit-removal]]
- Decision that supersedes this at higher complexity: [[when-to-use-builder-pattern]]
- Related pitfall: [[constructor-state-inconsistency-from-duplication]]

## Interview question this answers

- "A PaymentOrder class has four constructors with overlapping initialization logic. A new required field is added. How many sites need to change? How do you redesign this?"
