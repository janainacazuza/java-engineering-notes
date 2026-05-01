---
type: pitfall
tags: [java, constructors, reflection, frameworks, jpa, spring]
level: junior
status: draft
java_versions: [21, 25]
date_created: 2026-05-01
date_revised: 2026-05-01
primary_source: "JLS §8.8.9 (default constructor)"
secondary_sources: ["Spring Framework documentation, bean instantiation; JPA 3.1 spec §2.1"]
revisit_on: 2026-08-01
---

# Default constructor implicit removal

## Why this changes how I write code

Adding any parameterized constructor to a class silently removes the compiler-generated no-arg constructor. JPA requires a no-arg constructor to materialize entities from query results. Spring requires it for no-argument bean instantiation. Jackson requires it for JSON deserialization. Without knowing this, a developer adds a parameterized constructor to an entity or a deserialization target and the application compiles without error but fails at runtime with a reflection-based instantiation exception. This forces explicit declaration of a public or protected no-arg whenever a class is used by a framework that reflectively creates instances.

## The pitfall

The Java compiler (javac) generates a public no-arg constructor if and only if the class declares no constructor at all. The generated constructor does exactly one thing: call super(). When any constructor is declared explicitly, including a no-arg constructor, the compiler generates nothing additional. The implicit no-arg disappears as soon as the class takes ownership of constructor declaration.

The bytecode of the compiler-generated default constructor:

```
public Payment();
  Code:
     0: aload_0
     1: invokespecial   // Method java/lang/Object."<init>":()V
     4: return
```

No putfield instructions. Field default values (null, 0, 0.0, false) are assigned by the JVM at object allocation time, before the constructor executes, not by the constructor itself.

The failure produced by the missing no-arg is a runtime exception during reflective instantiation, not a compile-time error. The stack trace references java.lang.reflect.Constructor.newInstance or a framework proxy creation mechanism, not the class that lost its no-arg. This makes the root cause non-obvious without knowing the rule.

## Manifestation in real code

```java
// Before: compiler generates no-arg. Frameworks instantiate correctly.
public class PaymentOrder {
    String currency;
    double amount;
}

// After: no-arg is gone. JPA, Spring, and Jackson fail at runtime.
public class PaymentOrder {
    String currency;
    double amount;

    public PaymentOrder(String currency, double amount) {
        this.currency = currency;
        this.amount = amount;
    }
}

// Correct: explicit no-arg alongside the parameterized constructor
public class PaymentOrder {
    String currency;
    double amount;

    protected PaymentOrder() {} // for JPA; protected prevents accidental direct use

    public PaymentOrder(String currency, double amount) {
        this.currency = currency;
        this.amount = amount;
    }
}
```

## Production signal

A JPA entity or Spring-managed class that previously worked without a declared constructor receives a parameterized constructor in a refactoring. The application compiles. At startup or at first query execution, the framework throws InstantiationException, NoSuchMethodException, or a framework-specific variant. The stack trace points into reflection internals. The cause is the absent no-arg constructor, but the error message names the reflective operation, not the constructor declaration site.

## Connections

- Underlying mechanism: [[jvm-object-allocation-sequence]]
- Pattern that depends on this: [[constructor-chaining-this]]
- Related pitfall: [[jpa-entity-no-arg-requirement]]

## Interview question this answers

- "A JPA entity that was working starts throwing an exception after a developer adds a parameterized constructor. What happened and how do you fix it?"
- "When does the Java compiler generate a default constructor? What makes it stop?"
