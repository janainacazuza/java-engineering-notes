---
type: pitfall
tags: [java, string, equality, null, operators]
level: junior
status: draft
java_versions: [21, 25]
date_created: 2026-05-02
date_revised: 2026-05-02
primary_source: "JLS §15.12.4.4 (method invocation on null reference); Object.equals(Object) Javadoc"
secondary_sources: ["Effective Java, 3rd ed., Item 11"]
revisit_on: 2026-08-02
---

# Null receiver in equals call

## Why this changes how I write code

Calling .equals() on a reference that may be null throws NullPointerException at the call site, not at the source of the null. Without knowing this, a developer writes userInput.equals("expected") in an authentication or validation path. When userInput is null, the method throws NullPointerException instead of returning false. Placing the known-non-null value on the left side of the call, or using Objects.equals(), eliminates the risk without changing the comparison semantics. This changes the order of operands in every String equality check where the left side may be null.

## The pitfall

Object.equals(Object other) is an instance method. Calling any instance method on a null reference throws NullPointerException. The JVM cannot dispatch an instance method without a valid object to dispatch on. The error occurs at the equals call site, which makes the stack trace point at the comparison line, not at the code path that produced the null. This indirection makes the root cause non-obvious.

The pitfall is common in three situations: request input that is absent from the payload, map values for keys that are not present, and database columns that allow null.

Three patterns resolve it, in order of preference for financial code:

Literal-first: place the known-non-null value (a string literal, a constant, or an enum value) on the left side of the call. A literal is never null. The comparison returns false when the right side is null, without throwing.

Objects.equals(a, b): null-safe for both arguments. Returns true only when both are null or when both are non-null and equal by .equals(). Returns false when exactly one is null. Use this form when neither argument is guaranteed non-null.

Explicit null guard before calling .equals(): verbose and error-prone when applied consistently across a codebase. Prefer the two patterns above.

The NullPointerException message in modern JVM (Java 14 and later with helpful NPE messages) names the null variable: "Cannot invoke String.equals(Object) because currencyCode is null." This helps diagnosis but does not prevent the exception.

## Manifestation in real code

```java
// NPE: currencyCode is null when the request body omits the field
if (currencyCode.equals("BRL")) {
    applyBrazilianTaxRules();
}

// Literal-first: safe when currencyCode is null -- returns false, no exception
if ("BRL".equals(currencyCode)) {
    applyBrazilianTaxRules();
}

// Both sides potentially null -- Objects.equals handles it
if (Objects.equals(incomingStatus, storedStatus)) {
    markAsDuplicate(transaction);
}

// Map lookup: get() returns null for absent keys
Map<String, String> config = loadConfig();
if (config.get("routing.mode").equals("async")) { // NPE if key absent
    routeAsync();
}
if ("async".equals(config.get("routing.mode"))) { // safe
    routeAsync();
}
```

## Production signal

A payment gateway receives requests where the currency field is optional for legacy clients. A routing method calls currencyCode.equals("BRL") without a null guard. Requests from legacy clients that omit the currency field produce NullPointerException. The gateway returns 500 for those clients. The exception is logged with a message that names the null variable but not the upstream source of the null, so the investigation starts at the routing method rather than at the deserialization layer that leaves the field null.

## Connections

- Related pitfall: [[unboxing-null-npe]]
- Related mechanism: [[string-literal-pool-equality]]
- Decision that depends on this: [[objects-equals-for-nullable-comparisons]]

## Interview question this answers

- "What is wrong with userInput.equals(\"admin\") and how do you fix it without adding a separate null check?"
- "A method throws NullPointerException on the equals line, but the variable appears to have a value in the logs. What could cause this?"
