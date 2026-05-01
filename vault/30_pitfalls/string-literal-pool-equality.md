---
type: pitfall
tags: [java, string, operators, equality, memory]
level: junior
status: draft
java_versions: [21, 25]
date_created: 2026-05-01
date_revised: 2026-05-01
primary_source: "JLS §3.10.5 (string literals), §15.21.3 (reference equality)"
secondary_sources: ["JVM Specification §5.1 (constant pool)"]
revisit_on: 2026-08-01
---

# String literal pool equality

## Why this changes how I write code

== on String compares memory addresses, not content. Two String variables holding identical text may or may not share the same reference depending on how the String was created. Without knowing the String pool mechanism, a developer uses == for String comparison, passes tests that compare literals (which share a pool reference), and produces false negatives in production when the String came from a database query, an HTTP response, or any runtime construction. Business logic that branches on String content must always use .equals().

## The pitfall

The JVM maintains a String pool (the interned string table) in the heap. The compiler places String literals into the class file's constant pool. At class loading time, the JVM resolves these into String objects in the pool. When two literals have identical content, the JVM resolves them to the same object. Two variables initialized with the same literal receive the same reference.

new String("hello") bypasses the pool. It always allocates a new object on the heap, separate from the pool entry. The variable receives a reference to this object, not to the pool reference.

Strings produced at runtime -- from StringBuilder, from reading a database column, from parsing a response body, from String.format() -- are heap objects outside the pool unless String.intern() is called explicitly. Explicit intern() in application code is uncommon and generally not warranted.

The bytecode reflects this: a literal is loaded via ldc from the constant pool, which resolves to the pool entry. new String(...) generates a new opcode followed by invokespecial, allocating a distinct object.

## Manifestation in real code

```java
String currency1 = "BRL";             // pool entry
String currency2 = "BRL";             // same pool entry
String currency3 = new String("BRL"); // new heap object

System.out.println(currency1 == currency2);       // true  -- same pool reference
System.out.println(currency1 == currency3);       // false -- different objects
System.out.println(currency1.equals(currency3));  // true  -- same content

// Pitfall: routing logic using ==
String incomingCurrency = parseCurrencyFromRequest(request); // runtime String
if (incomingCurrency == "BRL") { // false even when content is "BRL"
    applyBrazilianRules();
}

// Correct: .equals() compares content regardless of origin
if ("BRL".equals(incomingCurrency)) { // literal first: safe if incomingCurrency is null
    applyBrazilianRules();
}

// Null-safe when both sides may be null
if (Objects.equals(incomingCurrency, expectedCurrency)) {
    applyRules();
}
```

## Production signal

A payment routing rule branches on currency code using ==. Unit tests compare two literals and pass. In production, the currency String arrives from a database query result or an external API response. The runtime String is a distinct heap object. The == condition returns false regardless of content. The routing block is skipped silently. The transaction is processed under the wrong rule set. No exception is thrown and no log entry is produced for the misrouted path.

## Connections

- Related mechanism: [[integer-autoboxing-cache]]
- Decision that depends on this: [[objects-equals-for-nullable-comparisons]]
- Related pitfall: [[null-check-with-equals-on-null-receiver]]

## Interview question this answers

- "Why does String s1 = 'hello'; String s2 = 'hello'; s1 == s2 return true, but String s3 = new String('hello'); s1 == s3 return false?"
- "A developer uses == to compare Strings in a routing condition. Unit tests pass. Production misroutes transactions. What is the cause and how do you fix it?"
