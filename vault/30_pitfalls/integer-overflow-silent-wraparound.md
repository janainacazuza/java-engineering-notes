---
type: pitfall
tags: [java, arithmetic, overflow, fintech, types]
level: junior
status: draft
java_versions: [21, 25]
date_created: 2026-05-03
date_revised: 2026-05-03
primary_source: "JLS §4.2.2 (integer operations); Math.addExact(int,int) Javadoc (Java 21)"
secondary_sources: ["Effective Java, 3rd ed., Item 62"]
revisit_on: 2026-08-03
---

# Integer overflow silent wraparound

## Why this changes how I write code

Java integer arithmetic never throws on overflow. Without knowing this, a developer accumulates transaction counts or financial totals in int, the value wraps silently to a large negative number when it exceeds Integer.MAX_VALUE (2,147,483,647), and the system propagates the corrupted value without any exception or log entry. This changes the declared type of every accumulator that may grow beyond int range from int to long, and it introduces Math.addExact as the tool for cases where overflow must be treated as an immediate detectable failure rather than a silent type change.

## The pitfall

Java integer arithmetic uses two's complement representation without overflow detection. When an int addition exceeds Integer.MAX_VALUE, the carry bit beyond position 31 is discarded and the result wraps to Integer.MIN_VALUE. No exception is thrown. No flag is set. The result is indistinguishable from a valid negative value.

The same wraparound applies to subtraction below Integer.MIN_VALUE and to multiplication when the product exceeds the int range. long has a larger range (up to 9,223,372,036,854,775,807) but wraps by the same mechanism when its own limit is exceeded.

Math.addExact(a, b), Math.subtractExact(a, b), and Math.multiplyExact(a, b) perform the same arithmetic as their operator equivalents but throw ArithmeticException immediately if overflow occurs. They are the only standard mechanism that makes integer overflow visible at the point of computation. The exception message is "integer overflow".

Three tools serve three distinct contexts. long expands the range and is appropriate for accumulators that may grow beyond int range but whose maximum is bounded. BigDecimal is appropriate for monetary amounts where decimal precision is required. Math.addExact is appropriate when overflow is a business logic failure that must be caught immediately rather than propagated, regardless of whether the accumulator type is int or long.

## Manifestation in real code

```java
// Silent overflow: result is mathematically wrong, no exception
int total = Integer.MAX_VALUE;
int newBatch = 1;
System.out.println(total + newBatch); // -2147483648

// Math.addExact: throws ArithmeticException on overflow
try {
    int safe = Math.addExact(total, newBatch);
} catch (ArithmeticException e) {
    // overflow detected at the computation site — reject, alert, log
    log.error("Overflow detected in settlement accumulator: {}", e.getMessage());
    throw new SettlementException("Transaction count exceeded int range", e);
}

// Long: correct for large but bounded accumulators
long runningTotal = 0L;
runningTotal = Math.addExact(runningTotal, newBatch); // throws if long overflows

// Binary search bug — classic overflow in algorithm code
int mid = (low + high) / 2;     // overflows when low + high > Integer.MAX_VALUE
int safeMid = low + (high - low) / 2; // correct — subtraction stays in range
```

## Production signal

A settlement service accumulates daily transaction counts in int. At peak volume, the daily total exceeds Integer.MAX_VALUE. The counter wraps to a large negative value. No exception surfaces in the application logs. The reconciliation report carries a negative transaction count, which the BACEN reporting endpoint rejects as invalid. The failure is discovered at batch close, not at transaction time. The root cause is found by examining the type declaration of the accumulator variable, not by a stack trace or error log.

## Connections

- Related pitfall: [[modulo-negative-dividend]]
- Decision that depends on this: [[numeric-type-selection-for-amounts]]
- Related tool: [[math-exact-arithmetic-methods]]

## Interview question this answers

- "What does Integer.MAX_VALUE + 1 print in Java? What is the mechanism and how do you protect against it in a financial system?"
- "A daily transaction counter in a payment service shows a negative value in the reconciliation report. No exception was thrown. What is the likely cause and how do you fix it?"
