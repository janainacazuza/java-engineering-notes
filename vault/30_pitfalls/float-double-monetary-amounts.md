---
type: pitfall
tags: [java, types, primitives, precision, fintech]
level: junior
status: draft
java_versions: [21, 25]
date_created: 2026-05-02
date_revised: 2026-05-02
primary_source: "JLS §4.2.3 (floating-point types); IEEE 754-2019"
secondary_sources: ["Effective Java, 3rd ed., Item 60"]
revisit_on: 2026-08-02
---

# Float and double for monetary amounts

## Why this changes how I write code

float and double cannot represent most decimal fractions exactly under IEEE 754 binary arithmetic. Without knowing this, a developer uses double for transaction amounts and accumulates rounding errors across operations. In a payment system, totals computed in double do not match the sum of their parts, producing reconciliation failures that surface in batch settlement rather than at transaction time. This changes the declared type of every monetary field and every intermediate arithmetic result in financial code: double is replaced by BigDecimal with a defined rounding mode.

## The pitfall

float and double are IEEE 754 binary floating-point types. The binary representation cannot express most decimal fractions exactly: 0.1 has no finite binary expansion, just as 1/3 has no finite decimal expansion. The representation stores the closest approximation, and arithmetic on those approximations accumulates error.

The error is visible directly:

```java
double a = 0.1 + 0.2;
System.out.println(a); // 0.30000000000000004
```

The error per operation is small but cumulative. Summing ten thousand transaction amounts, each with a small representation error, produces a total that differs from the authoritative decimal result by amounts that vary with the order of summation.

The correct type for monetary amounts in Java is java.math.BigDecimal. BigDecimal stores the value as an arbitrary-precision integer scaled by a power of ten, which matches the decimal arithmetic that financial regulation and double-entry bookkeeping require. Every division or rounding operation on BigDecimal requires an explicit RoundingMode. RoundingMode.HALF_EVEN (banker's rounding) is the standard for financial systems because it minimizes cumulative rounding bias across large datasets.

float introduces the same representation error with a smaller range (32-bit vs 64-bit). Neither float nor double is acceptable for monetary amounts.

## Manifestation in real code

```java
// Wrong: double accumulates representation error
double unitPrice = 19.99;
double quantity = 3;
double total = unitPrice * quantity;
System.out.println(total); // 59.97 -- happens to print correctly here
double tax = total * 0.1;
System.out.println(tax);  // 5.996999999999999 -- representation error visible

// Correct: BigDecimal with explicit scale and rounding mode
BigDecimal price = new BigDecimal("19.99");
BigDecimal qty = new BigDecimal("3");
BigDecimal subtotal = price.multiply(qty);

BigDecimal taxRate = new BigDecimal("0.10");
BigDecimal taxAmount = subtotal.multiply(taxRate)
    .setScale(2, RoundingMode.HALF_EVEN);

// Always construct BigDecimal from String, not from double.
// new BigDecimal(0.1) captures the double's representation error.
// new BigDecimal("0.1") is exact.
BigDecimal wrong = new BigDecimal(0.1);   // 0.1000000000000000055511...
BigDecimal correct = new BigDecimal("0.1"); // exactly 0.1
```

## Production signal

A batch settlement job sums double amounts across tens of thousands of transactions. The computed total differs from the authoritative total held by the ledger by a fraction of a cent. The discrepancy is non-deterministic: it depends on the order of summation and the specific values in the batch. BACEN reconciliation rejects the settlement report. No single transaction log entry shows an error because each individual computation prints correctly at the displayed precision. The cause is found only by examining the type declaration of the accumulator variable.

## Connections

- Related decision: [[bigdecimal-rounding-modes]]
- Related pitfall: [[bigdecimal-from-double-constructor]]
- Decision that depends on this: [[numeric-type-selection-for-amounts]]

## Interview question this answers

- "What is wrong with using double to represent monetary values in a payment system? What type should be used instead?"
- "A settlement job produces totals that differ from the ledger by small amounts that vary between runs. What is the likely cause?"
