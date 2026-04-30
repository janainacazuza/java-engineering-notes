---
type: meta
---

# Graph Rules

The Obsidian graph is only useful if linkage is intentional and parsimonious.

## The five rules

**Every note has between 2 and 6 outgoing links.** Fewer than 2 means isolation, possibly trivia. More than 6 means folder-level work disguised as a note, should be split.

**Link types are explicit.** Write `Underlying mechanism: [[stack-frame]]` or `Related pitfall: [[stack-overflow-error]]`. Labels make the graph readable months later.

**Bidirectionality is manual and selective.** If A is a particular case of B, both reference each other. If A depends on mechanism B, A references B without requiring reciprocity.

**Hub notes per area.** Each domain area has one hub note such as `jvm-field-view.md`, `concurrency-field-view.md`. A hub has 10 to 20 curated links to the most important notes in the area, with brief narrative connecting them.

**MOC at the root.** A single `MOC.md` at the vault root links to all hubs. This is the entry point when the vault is opened.

## How the graph should look

Dense clusters around each hub. Thin bridges between clusters. No isolated points. MOC at the center, hubs in the second ring, individual notes in the third ring.

If the graph looks like a uniform cloud, linkage has no criterion.
