# The Blast Radius Protocol
### The Coding Architecture That Enables Zero-Dev Software Development

**Author:** Roark Pinkerton  
**Date:** May 2026

---

## Zero-Dev Architecture

Zero-Dev is a way of building software with no developers on the team.

Not "fewer developers." Not "developers assisted by AI." Zero.

One human. One AI. The human is a systems architect — they define what gets built, direct the AI, and evaluate the finished product. They never read the code. They never write the code. They never debug anything. Their job is vision, direction, and judgment.

The AI is the entire development team. It writes the code, tests it, debugs it, fixes regressions, and hands off cleanly to the next session — without the human ever descending into the implementation.

The Blast Radius Protocol is the codebase architecture that makes Zero-Dev possible.

---

## Why You Need a Protocol at All

AI coding assistants fail in a predictable way: ask one to fix something in a large file and it fixes it while breaking something adjacent. The model isn't careless — it treats everything in the context window as fair game. When the file is 400 lines with multiple concerns, the blast radius of any edit is 400 lines.

The standard response — careful prompting, "don't touch X" instructions — doesn't scale. You can't enumerate everything you don't want changed. And if a human has to watch every AI edit to catch the collateral damage, you haven't automated development. You've built expensive autocomplete.

The root cause is the shape of the code. **Code written for human developers is structured for human readers — and that structure is actively hostile to AI execution.**

Zero-Dev only works if the AI can operate on the codebase independently, across sessions, without human intervention in the technical layer. The Blast Radius Protocol is how you get there.

---

## The Core Insight

Traditional software architecture organizes code around readability: logical groupings, shared context, DRY abstractions. These produce files that are 200-600 lines with multiple concerns interleaved — a wide blast radius for every edit.

The Blast Radius Protocol organizes code around **rewrite units** — the smallest chunk the AI can touch in isolation, without understanding anything outside that file.

When you structure code this way, regressions become physically impossible to propagate. The structure enforces the boundaries. No human oversight required between edits.

---

## The Rules

### Rule 1 — Shard below the human-readable threshold

Each block is a single file, ~80-100 lines, single concern. Not "one component" — one *concern*. A hook that fetches data is separate from a hook that transforms it. A component that renders is separate from the component that positions it.

This feels uncomfortably granular. That's the protocol working. The AI doesn't get uncomfortable — it gets efficient.

### Rule 2 — The assembly layer is sacred

`App.tsx` (or equivalent) is **pure wiring**. Imports and composition only. Zero logic. Zero state beyond top-level delegation. No ternaries.

When the AI needs to change how blocks connect, it touches `App.tsx`. When it needs to change what a block does, it touches that block. These are never the same operation. The structure keeps them clean.

### Rule 3 — The AI defines and owns the contracts

The first three lines of every block are a contract:

```
// Input: what this block receives
// Output: what this block produces or renders
// Must never: the behaviors this block is prohibited from having
```

The human doesn't write these. The AI writes them when it creates the block and maintains them when it edits. They are the AI's own documentation — and the primary tool it uses to re-orient itself when a new session starts cold.

### Rule 4 — The regression surface is one file

When something breaks, the AI's debug protocol is: identify the block, rewrite only that block, everything else frozen.

No human needs to isolate the problem. No human needs to read the code. The AI finds the block, reads its contract, rewrites just that file.

### Rule 5 — Version blocks, not apps

When a block needs a significant change, the AI creates `block.v2.ts` alongside the original. The original stays frozen. The new version is built and tested in isolation. The import in `App.tsx` is swapped when verified.

Instant rollback at zero cost. The human never needs to know a version swap happened — they just see the product working or not.

### Rule 6 — The codebase is the handoff document

Every new AI session starts cold. In a traditional codebase, that means the human has to re-explain the project — or the AI makes changes without context and breaks things.

In a Blast Radius codebase, the AI re-orients by reading `App.tsx` and the contracts of the blocks it needs to touch. That's the entire handoff. The human doesn't manage it. The structure carries the context automatically, session to session.

---

## The Operating Model

| Role | Responsibility |
|---|---|
| Human | Vision. Direction. Product review. Judgment when the AI is stuck. |
| AI | Contracts. Implementation. Testing. Debugging. Session handoffs. |
| Human never | Reads code. Writes code. Isolates a bug. Reviews a diff. |

The human evaluates the product, not the implementation. The protocol makes that possible by ensuring the AI can operate on itself without the human in the technical loop.

---

## Observed Results

Built in practice across several weeks of daily AI-directed development — production AI infrastructure and a real-time agent dashboard — with no human writing or reading code at any point.

- **Regressions dropped to near zero** after the protocol was formalized
- **Debug cycles collapsed** — identify the block, rewrite it, done
- **Cold session recovery takes minutes** — `App.tsx` + relevant contracts is the entire handoff
- **The human stayed entirely out of the implementation** — product review only, start to finish

---

## Quick Reference

| Principle | Rule |
|---|---|
| File size | ~80-100 lines, one concern |
| Assembly layer | Pure wiring — imports and composition, zero logic |
| Contracts | AI writes and owns Input / Output / Must Never |
| Regression scope | Identify block → rewrite that file only |
| Versioning | `block.v2.ts` alongside original, swap import when verified |
| Session handoff | `App.tsx` + relevant contracts — no human required |
| Human role | Vision, direction, product review — never the code |

---

## Examples

See the [`/examples`](/examples) directory for annotated code samples showing what a Blast Radius codebase looks like in practice.

---

*The Blast Radius Protocol is not a framework. It has no dependencies. It requires no tooling. It is the architecture layer that makes Zero-Dev possible.*
