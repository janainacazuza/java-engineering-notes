---
type: meta
---

# Note Anatomy

Every note in `java-engineering-notes` has the same structure. Fixed structure makes the graph readable and retrieval fast.

## Mandatory frontmatter

```yaml
---
type: mechanism | decision | pitfall | pattern | diagnostics | snippet | interview
tags: [java, jvm, memory]
level: junior | mid | senior | staff
status: draft | reviewed | validated | archived
java_versions: [21, 25]
date_created: YYYY-MM-DD
date_revised: YYYY-MM-DD
primary_source: "JLS §17.7"
secondary_sources: ["JEP 444", "JCIP ch. 3"]
revisit_on: YYYY-MM-DD
---
```

- `type` determines which folder the note lives in.
- `level` is the professional stage at which this knowledge becomes actionable.
- `status` is `draft` on creation, `reviewed` after technical review by Higor Cazuza, `validated` after defense in a sabbatical session or documented production application, `archived` when superseded.
- `java_versions` defaults to `[21, 25]`. If behavior differs between them, the note body must contain the version comparison section.
- `revisit_on` defaults to three months after creation.
- File names are kebab-case without accents.

## Note body

```markdown
# [Title, short noun phrase, not a sentence]

## Why this changes how I write code

[Two to four concrete lines. The filter materialized.]

## The mechanism / the decision / the pitfall

[Technical core. Specialist-level depth.]

## Java 21 versus Java 25

[Include only when behavior or API differs meaningfully.]

## Manifestation in real code

\`\`\`java
// Short, executable, commented snippet.
\`\`\`

## Production signal

[How this appears when something goes wrong.
Anchor in fintech context when natural.]

## Connections

- Underlying mechanism: [[note-name]]
- Decision that depends on this: [[note-name]]
- Related pitfall: [[note-name]]

## Interview question this answers

[One or two real questions this note answers completely.]
```

## File naming convention

- Kebab-case: `string-pool-on-heap.md`, `equals-without-hashcode.md`.
- Name reflects the concept, not the study module.
- No numeric prefixes in the file name.
- No accents in the file name.
