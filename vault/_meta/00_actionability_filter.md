---
type: meta
---

# Actionability Filter

The single rule that governs whether content enters `java-engineering-notes`:

> "Would I write or design code differently if I knew this, even months from now?"

No or maybe means it does not enter. Yes means it enters in the prescribed note format.

## Always passes

- Internal mechanisms that explain observable behavior (GC decisions, `synchronized` bytecode, connection pool socket reuse, virtual thread scheduling).
- Concrete trade-offs with numbers or scenarios (HashMap vs TreeMap vs ConcurrentHashMap under which workload, virtual threads vs platform threads under which I/O profile).
- Documented industry pitfalls (`equals` without `hashCode`, deadlock by lock acquisition order, race conditions in check-then-act, memory leaks from unremoved listeners).
- Patterns with strict applicability criteria.
- Production diagnostic procedures (heap dump reading, GC log interpretation, `jstack` deadlock diagnosis).
- Canonical and anti-snippets with explanation of why each is correct or incorrect.
- Version-specific behavior worth tracking between Java 21 and Java 25.

## Never passes

- Definitions and biographies.
- Command lists.
- Chapter summaries.
- Technical trivia (dates, version numbers without trade-off relevance).
- Notes that end in "interesting".
- Knowledge already covered by existing notes.

## The test before saving

Every note must fill the section "Why this changes how I write code" with two to four concrete lines. If that section cannot be filled, the note should not exist.

Correct form: "Without this, I would call `equals` on HashMap keys without ensuring `hashCode` consistency, producing maps with lost keys in production."

Incorrect form: "Important for interviews."
