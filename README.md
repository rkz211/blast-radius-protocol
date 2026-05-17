# The Blast Radius Protocol
### A Coding Architecture for Human-Directed, AI-Executed Software Development

**Author:** Roark Pinkerton  
**Date:** May 2026

---

## What This Is

The Blast Radius Protocol is a codebase architecture for a specific kind of software team: one human, one AI, no developers.

The human is a systems architect. They define what gets built, direct the AI, and evaluate the finished product. They never read the code. They never write the code. They never debug the code. Their job is vision, direction, and judgment.

The AI is the entire development team. It writes the code, defines the contracts, tests, debugs, and fixes regressions — across sessions, across context resets, across model switches — without the human descending into the implementation to fix anything.

The protocol is what makes that division of labor actually work.

---

## The Problem It Solves

AI coding assistants fail in a predictable way: ask one to fix a thing in a large file, and it fixes the thing and breaks something adjacent. The model isn't careless. It just treats everything in the context window as fair game. When the file is 400 lines with multiple concerns, the blast radius of any edit is 400 lines.

The standard response — careful prompting, "don't touch X" instructions — doesn't scale. You can't enumerate everything you don't want changed. And if the human has to watch every AI edit to catch collateral damage, you haven't built a team of one. You've built a very expensive autocomplete.

The root cause is the shape of the code. **Code written for human developers is structured for human readers — and that structure is actively hostile to AI execution.**

---

## The Core Insight

Traditional software architecture organizes code around readability conventions: logical groupings, shared context, DRY abstractions, appropriate abstraction levels. These are good for humans. They produce files that are 200-600 lines long with multiple concerns interleaved — and that means any AI edit has a wide blast radius.

The Blast Radius Protocol organizes code around a different principle: **rewrite units**.

The question isn't "what belongs together conceptually?" It's "what is the smallest unit the AI can rewrite in isolation, without understanding anything outside this file?"

When you structure code this way, regressions become physically impossible to propagate. The AI can only touch what's in the file. The structure enforces the boundaries — no human oversight required between edits.

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

The human doesn't write these. The AI writes them when it creates the block, and maintains them when it edits. They are the AI's own documentation of its work — and the primary tool it uses to re-orient itself when a new session starts cold.

### Rule 4 — The regression surface is one file

When something breaks, the AI's debug protocol is: identify the block, rewrite only that block, everything else frozen.

No human needs to isolate the problem. No human needs to read the code. The structure makes the blast radius visible and contained. The AI finds the block, reads its contract, rewrites just that file.

### Rule 5 — Version blocks, not apps

When a block needs a significant change, the AI creates `block.v2.ts` alongside the original. The original stays frozen. The new version is built and tested in isolation. The import in `App.tsx` is swapped when it's verified.

Instant rollback at zero cost. The human never needs to know a version swap happened — they just see the product working or not.

### Rule 6 — The codebase is the handoff document

Every new AI session starts cold. No memory of the last conversation. In a traditional codebase, this means the human has to re-explain the project — or the AI makes changes without context and breaks things.

In a Blast Radius codebase, the AI re-orients itself by reading `App.tsx` (the wiring map) and the contracts of the blocks it needs to touch. That's the entire handoff. The human doesn't manage it. The structure carries the context automatically.

---

## The Operating Model

This protocol enables a specific way of building software:

**The human's role:** Define what the product should do. Direct the AI toward the next thing. Look at the finished product and say whether it's right. Provide judgment when the AI is stuck — not technical answers, architectural ones.

**The AI's role:** Everything else. Architecture of individual blocks, contracts, implementation, testing, debugging, regression fixes, session handoffs.

**What the human never does:** Read the code. Write the code. Isolate a bug. Explain a file structure. Review a diff.

The human evaluates the product, not the implementation. The protocol makes that possible by ensuring the AI can operate on itself without human intervention in the technical layer.

---

## Why This Works

The context window is not a bug to route around — it's a constraint to design for. When a block is 80 lines with a clear contract, the AI has exactly what it needs and nothing it can accidentally damage. The constraint is the safety mechanism.

"Leave this alone" stops being an instruction you give the model. It becomes a property of the file structure. The model literally cannot touch what isn't in the file.

And critically: this produces code that is **native to how AI executes**, not how humans read. The AI isn't being asked to work inside a structure built for someone else. It's working inside a structure built for it.

---

## Observed Results

Built in practice across several weeks of daily AI-directed development — production AI infrastructure and a real-time agent dashboard — with no human writing or reading code at any point.

- **Regressions dropped to near zero** after the protocol was formalized
- **Debug cycles collapsed** — identify the block, rewrite it, done
- **Cold session recovery takes minutes, not hours** — `App.tsx` + relevant contracts is the entire handoff
- **The human stayed entirely out of the implementation** — product review only, start to finish

---

## Quick Reference

| Principle | Rule |
|---|---|
| File size | ~80-100 lines, one concern |
| Assembly layer | Pure wiring only — imports and composition, zero logic |
| Contracts | AI writes and owns Input / Output / Must Never |
| Regression scope | Identify block → rewrite that file only |
| Versioning | `block.v2.ts` alongside original, swap import when verified |
| Session handoff | `App.tsx` + relevant block contracts — no human required |
| Human role | Vision, direction, product review — never the code |

---

## Examples

See the [`/examples`](/examples) directory for annotated code samples showing what a Blast Radius codebase looks like in practice.

---

*The Blast Radius Protocol is not a framework. It has no dependencies. It requires no tooling. It is an operating model for a new kind of software team.*
