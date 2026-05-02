---
type: pitfall
tags: [java, types, boxing, null, operators]
level: junior
status: draft
java_versions: [21, 25]
date_created: 2026-05-02
date_revised: 2026-05-02
primary_source: "JLS §5.1.8 (unboxing conversion)"
secondary_sources: ["Effective Java, 3rd ed., Item 61"]
revisit_on: 2026-08-02
---

# Unboxing null wrapper

## Why this changes how I write code

Auto-unboxing converts a wrapper object to its primitive value by calling the wrapper's primitiveValue() method (intValue(), longValue(), doubleValue(), etc.) on the reference. If the reference is null, the call throws NullPointerException. Without knowing this, a developer assigns the return value of a repository method typed as Integer directly to an int variable, getting NullPointerException at runtime when the database row is absent. This changes whether the target variable is declared as int or Integer when calling any API that returns a nullable numeric type, and it changes how map lookups on numeric values are handled.

## The pitfall

Auto-unboxing is a compiler transformation. When a wrapper type is used where a primitive is required, the compiler inserts a call to the corresponding primitive accessor method. The bytecode for int limit = nullableInteger expands to:

```
aload  <nullableInteger reference>
invokevirtual Integer.intValue()
istore <limit>
```

The invokevirtual on a null reference produces NullPointerException at the assignment line. The stack trace identifies the assignment, not the code path that produced the null.

The same mechanism occurs in three common situations. First, assigning a nullable repository return value typed as Integer to an int local variable. Second, reading a numeric value from a Map: Map.get() returns null when the key is absent, and assigning that result to int unboxes null. Third, conditional expressions: Integer x = condition ? intValue : null; followed by int y = x; produces NPE when the condition takes the null branch.

Boolean unboxing under this pitfall has an additional failure mode: Boolean flag = null; if (flag) {...} throws NPE because the if condition requires the boolean primitive, triggering unboxing.

## Manifestation in real code

```java
// Repository method typed as Integer -- returns null when no row matches
Integer fetchDailyLimit(String accountId) {
    return jdbcTemplate.queryForObject(sql, Integer.class, accountId);
    // returns null if no limit is configured for this account
}

// NPE: unboxing calls intValue() on null
int limit = fetchDailyLimit("acc-001"); // NullPointerException for new accounts

// Safe: null check before unboxing
Integer rawLimit = fetchDailyLimit("acc-001");
int limit = (rawLimit != null) ? rawLimit : DEFAULT_LIMIT;

// Safe: Optional forces the caller to acknowledge absence
Optional<Integer> maybeLimit = fetchOptionalLimit("acc-001");
int limit = maybeLimit.orElse(DEFAULT_LIMIT);

// Map lookup: same mechanism
Map<String, Integer> feeTable = buildFeeTable();
int pixFee = feeTable.get("PIX");              // NPE if "PIX" key is absent
int pixFee = feeTable.getOrDefault("PIX", 0); // safe

// Boolean unboxing
Boolean featureEnabled = featureFlags.get("new_flow");
if (featureEnabled) { ... }                          // NPE if key is absent
if (Boolean.TRUE.equals(featureEnabled)) { ... }     // safe -- null-tolerant
```

## Production signal

An account service fetches a per-account transaction limit from the database. New accounts have no configured limit row. The service assigns the Integer return value of the JDBC query directly to int. Every transaction request on a new account throws NullPointerException at the limit assignment line. The failure is intermittent in staging because staging accounts are pre-configured with limits. In production, the failure rate spikes when new account creation peaks. The stack trace names the assignment line, making the initial investigation focus on the JDBC call rather than on the null handling at the assignment site.

## Connections

- Related pitfall: [[null-receiver-equals-npe]]
- Underlying mechanism: [[integer-autoboxing-cache]]
- Decision that depends on this: [[optional-vs-null-for-absent-values]]

## Interview question this answers

- "A service throws NullPointerException on an int assignment line, but the Integer variable it reads from looks valid. What is the mechanism?"
- "What happens when you call Map.get() on an absent key and assign the result directly to int?"
