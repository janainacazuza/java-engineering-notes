---
type: mechanism
tags: [java, references, memory, methods, defensive-copy]
level: junior
status: draft
java_versions: [21, 25]
date_created: 2026-05-02
date_revised: 2026-05-02
primary_source: "JLS §8.4.1 (method parameters), §10.7 (array members)"
secondary_sources: ["Effective Java, 3rd ed., Item 50"]
revisit_on: 2026-08-02
---

# Pass-by-value semantics for reference types

## Why this changes how I write code

Java always passes a copy of the variable's value to a method. For reference types, the value is the memory address of the heap object, not the object itself. Without knowing this, a developer passes a mutable list or array to a utility method expecting the caller's data to remain unchanged, and the method modifies the original object through the shared address. This changes whether a defensive copy is required before passing mutable data to methods outside the developer's control, and whether methods that must not mutate their arguments should receive a copy or an unmodifiable view.

## The mechanism

When a method is called, the JVM copies each argument into the called method's stack frame. For primitive types, the copy is the numeric value. For reference types, the copy is the object reference: the memory address that identifies the heap object. The original reference in the caller and the parameter in the method point to the same object.

Two consequences follow:

Mutation through the parameter is visible to the caller. If the method modifies the object's state (changes an array element, adds to a list, sets a field), the caller observes the change because both references point to the same heap object.

Reassignment of the parameter is not visible to the caller. If the method points its local parameter copy at a different object, the caller's reference is unaffected. The method's local copy is what changes, not the caller's variable.

The bytecode distinction is in the opcodes used to load arguments: primitives use iload, dload, lload; references use aload ("address load"). Method calls that receive references receive addresses, not heap data.

Java is never pass-by-reference in the C++ sense. In C++, pass-by-reference means the method receives an alias for the caller's variable: reassigning the parameter reassigns the caller's variable. Java never does this. The precise term for Java's behavior is "pass-by-value where the value of a reference variable is an address."

## Manifestation in real code

```java
// Case 1: Mutation through shared address -- caller sees the change
public static void zeroFirst(int[] arr) {
    arr[0] = 0; // modifies the object the caller also references
}

// Case 2: Reassignment of parameter -- caller does not see the change
public static void replaceArray(int[] arr) {
    arr = new int[]{9, 9, 9}; // rebinds the local copy only
}

public static void main(String[] args) {
    int[] data = {1, 2, 3};

    zeroFirst(data);
    System.out.println(data[0]); // 0 -- mutation is visible to caller

    replaceArray(data);
    System.out.println(data[0]); // 0 -- reassignment has no effect on caller

    // Defensive copy: caller protects its data before passing to external code
    int[] safeData = Arrays.copyOf(data, data.length);
    externalValidator.check(safeData); // external code cannot affect data
}

// Unmodifiable view: exposes list without allowing structural mutation
public List<Transaction> getPending() {
    return Collections.unmodifiableList(pendingTransactions);
}
```

## Production signal

A transaction validation service passes a mutable list of line items to a fee calculation method. The fee method removes items below a minimum threshold as a side effect of computing the total, operating on the same list object the caller holds. The caller records the transaction with fewer line items than were submitted. The bug surfaces in end-of-day reconciliation, not in the API response, because the validation result is computed before the mutation occurs. The investigation starts at the fee method, but the root cause is the missing defensive copy at the call site.

## Connections

- Underlying mechanism: [[jvm-stack-frame-structure]]
- Related pitfall: [[mutable-argument-side-effect]]
- Decision that depends on this: [[unmodifiable-view-vs-defensive-copy]]

## Interview question this answers

- "Java is pass-by-value or pass-by-reference? Explain with a concrete example that shows the difference between mutating an object through a parameter and reassigning the parameter."
- "A method that receives a List modifies it as a side effect. The caller's list is unexpectedly shorter after the call. What is the mechanism and how do you prevent it?"
