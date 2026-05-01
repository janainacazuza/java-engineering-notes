---
type: pitfall
tags: [java, types, boxing, operators, equality]
level: mid
status: draft
java_versions: [21, 25]
date_created: 2026-05-01
date_revised: 2026-05-01
primary_source: "JLS §5.1.7; Integer.valueOf(int) source (OpenJDK)"
secondary_sources: ["JEP 395"]
revisit_on: 2026-08-01
---

# Integer autoboxing cache

## Why this changes how I write code

Using == to compare Integer objects produces inconsistent results depending on the value: true for the range [-128, 127] because the JVM returns cached instances, false outside that range because it allocates new heap objects. Without knowing this, a developer compares Integer payment amounts with == and gets results that pass in unit tests using small values and are silently wrong in production for real transaction amounts. Any comparison of Integer objects in business code must use .equals() or Objects.equals() when null is possible.

## The pitfall

Autoboxing is a compiler transformation: Integer a = 127 becomes Integer a = Integer.valueOf(127). The method valueOf(int) maintains a static array of pre-allocated Integer objects for values from -128 to 127, initialized at JVM startup. When the argument falls within that range, valueOf() returns the cached instance. When the argument exceeds 127, it allocates a new object on the heap via new Integer(i).

The consequence for == comparison: two Integer variables holding 100 share the same cached reference, so == returns true. Two Integer variables holding 200 reference different heap objects, so == returns false without any exception or diagnostic.

The bytecode of == between references is if_acmpne ("if address compare not equal"), which compares memory addresses. The bytecode of autoboxing is invokestatic Integer.valueOf. These two facts together explain the behavior: address comparison of objects that may or may not be the same instance depending on the value.

The JVM loads values in the byte range with bipush and values outside it with sipush. The cache boundary coincides with the signed byte range by design: -128 and 127 are the limits of a signed 8-bit integer. The upper bound can be extended via -XX:AutoBoxCacheMax but the lower bound of -128 is fixed by the specification.

## Manifestation in real code

```java
// Pitfall: == on Integer produces value-dependent results
Integer limit = 127;
Integer threshold = 127;
System.out.println(limit == threshold); // true -- cached, same reference

Integer amount1 = 200;
Integer amount2 = 200;
System.out.println(amount1 == amount2); // false -- uncached, different objects

// Pitfall in business logic: works in tests, fails in production
public boolean isDuplicate(Integer incoming, Integer stored) {
    return incoming == stored; // correct for values <= 127, wrong above
}

// Correct: .equals() compares content regardless of cache
public boolean isDuplicate(Integer incoming, Integer stored) {
    return Objects.equals(incoming, stored); // null-safe
}
```

## Production signal

A payment deduplication service passes all unit tests because tests use amounts like 1, 10, 50. In production, duplicate transactions with amounts above 127 pass the deduplication check and are processed twice. No exception is thrown. The condition returns false silently. The failure surfaces through reconciliation drift or chargeback reports, not through an application error. The cause is not visible in logs.

## Connections

- Underlying mechanism: [[string-literal-pool-equality]]
- Related pitfall: [[primitive-vs-reference-equality]]
- Decision that depends on this: [[objects-equals-for-nullable-comparisons]]

## Interview question this answers

- "Why does Integer a = 127; Integer b = 127; yield a == b true, but Integer c = 128; Integer d = 128; yield c == d false?"
- "What does Integer.valueOf() do internally that makes == on Integer objects unreliable for business comparisons?"
