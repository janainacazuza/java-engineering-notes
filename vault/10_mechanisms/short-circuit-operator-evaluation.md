---
type: mechanism
tags: [java, operators, null-safety, boolean, performance]
level: junior
status: draft
java_versions: [21, 25]
date_created: 2026-05-01
date_revised: 2026-05-01
primary_source: "JLS §15.23 (&&), §15.24 (||), §15.22 (& and | for boolean)"
secondary_sources: []
revisit_on: 2026-08-01
---

# Short-circuit operator evaluation

## Why this changes how I write code

&& and || stop evaluating as soon as the result is determined. & and | always evaluate both operands. Without knowing this, a developer writes payment != null & payment.isActive() and gets a NullPointerException when payment is null, because & does not stop at the false result of the null check. Replacing & with && eliminates the exception. The distinction also matters when the right operand triggers I/O or a database call: && and || prevent the call when the left operand already determines the result.

## The mechanism

The JVM evaluates boolean expressions left to right. For &&: if the left operand evaluates to false, the result is false and the right operand is never executed. For ||: if the left operand evaluates to true, the result is true and the right operand is not executed. This is short-circuit evaluation.

& and | are the non-short-circuit boolean operators. They evaluate both operands in all cases, regardless of what the left operand produces. The right operand executes unconditionally.

Operator precedence within compound boolean expressions: arithmetic operators (*, /, %) resolve first, then additive (+, -), then relational (>, <, >=, <=), then equality (==, !=), then &, then |, then &&, then ||. In the expression (x != 0) && (10 / x > 1), the division 10 / x is resolved by precedence before the > comparison, which is resolved before the &&. With &&, if x is 0, the left operand (x != 0) is false and the division is never attempted. With &, the division executes and throws ArithmeticException: / by zero.

The only context where & or | is preferable over && or || is when the right operand has a side effect that must always execute regardless of the left operand result. That scenario is uncommon in production code and requires explicit documentation.

## Manifestation in real code

```java
// NPE: & evaluates isActive() even when payment is null
if (payment != null & payment.isActive()) {
    process(payment);
}

// Safe: && stops at the null check when payment is null
if (payment != null && payment.isActive()) {
    process(payment);
}

// Short-circuit preventing an expensive external call
if (isLocalCacheHit(orderId) || externalRiskCheck(orderId)) {
    // externalRiskCheck is skipped when the cache returns true
    authorize(orderId);
}

// ArithmeticException: & evaluates the right side even when left is false
int x = 0;
boolean result = (x != 0) & (10 / x > 1); // throws ArithmeticException: / by zero

// No exception: && stops after the false left operand
boolean safe = (x != 0) && (10 / x > 1); // evaluates to false, no division attempted
```

## Production signal

A null-guard condition written with & instead of && causes NullPointerException in production when the guarded object is null. The stack trace points to the line of the condition, specifically to the field access or method call on the null reference, not to the null assignment, making the cause non-obvious on first inspection. In a payment authorization service, this surfaces as an uncaught exception that rejects valid requests when a payment object fails to load from the upstream service.

## Connections

- Related pitfall: [[npe-in-boolean-guards]]
- Decision that depends on this: [[null-check-ordering-in-conditions]]
- Underlying mechanism: [[operator-precedence-resolution]]

## Interview question this answers

- "What is the difference between && and & in Java? Give a case where using & instead of && causes a runtime failure."
- "You see a NullPointerException thrown from inside an if condition that contains a null check. How is that possible?"
