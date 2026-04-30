# Contributing to java-engineering-notes

This project follows a closed curatorial model to preserve editorial coherence. The rules below exist to keep the vault useful over the long term.

## Contribution model

**Pull requests** are accepted only from the maintainer at this time. Pull requests opened by any other account, including the designated reviewer, will be closed without merge.

**Issues are open to anyone** and are the primary channel for community contribution and for review feedback from the designated reviewer:
- Technical correction in an existing note.
- Proposal of a new topic.
- Suggestion of editorial improvement.
- Report of outdated information.

Every issue receives a response within 14 days. Valid corrections are incorporated by the maintainer with credit in the commit message.

## Editorial criterion

Every note in this vault passes the question:

> "Would I write or design code differently if I knew this, even months from now?"

Proposals that do not pass this criterion are declined courteously. The full criterion lives in `vault/_meta/00_actionability_filter.md`.

## How to propose a new topic

1. Open an issue using the **Note proposal** template.
2. Describe the concept, justify why it meets the actionability criterion, and indicate which folder it would live in.
3. Wait for approval before any writing.

Approved proposals are labeled `approved`. Only after approval may the topic be developed.

## Mandatory note template

Every note follows the template defined in `vault/_meta/01_note_anatomy.md`. Notes without complete frontmatter or without the section "Why this changes how I write code" are declined.

## Conventional commits

All commits follow [Conventional Commits](https://www.conventionalcommits.org/):

````
docs(pitfalls): add equals-without-hashcode
docs(mechanisms): add hotspot-jit-tiered-compilation
fix(pitfalls): correct code example in silent-integer-overflow
refactor(meta): update actionability criterion
````

Valid scopes: `mechanisms`, `decisions`, `pitfalls`, `patterns`, `diagnostics`, `code`, `interview`, `meta`, `governance`.

## Attribution

Corrections and suggestions incorporated via issue receive credit in the commit body:

````
fix(pitfalls): correct code example in silent-integer-overflow

Correction suggested by @username via issue #42.
````
