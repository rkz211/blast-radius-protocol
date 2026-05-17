# The Blast Radius Protocol
### A Discipline for AI-Assisted Software Development

**Author:** Roark Pinkerton  
**Date:** May 2026  
**Context:** Developed in practice while building AI agent infrastructure and React/Amplify applications with an LLM pair-programming partner.

---

## The Problem

AI coding assistants are remarkably good at writing code. They are remarkably bad at *not breaking things*.

The failure mode is consistent: you hand a large file to an LLM, ask it to fix one thing, and it rewrites adjacent logic, touches imports, restructures a function it didn't need to touch, and introduces a regression two layers away from what you asked about. The model wasn't trying to break anything. It just has no concept of "leave this alone." Everything in the context window is fair game.

The standard response to this — careful prompting, explicit "don't touch X" instructions — is a losing game. You can't enumerate everything you don't want changed. And the model will forget halfway through a long generation anyway.

The root cause isn't the model's intelligence. It's the *shape of the code*. Large files have large blast radii. When the regression surface is 400 lines, any change risks 400 lines.

The Blast Radius Protocol is a structural solution to a structural problem.

---

## The Core Insight

**Organize code around rewrite units, not readability conventions.**

Traditional software architecture is designed for human readers — clean module boundaries, logical groupings, DRY principles, appropriate abstraction levels. These are good things. But they produce files that are 200-600 lines long, with multiple concerns interleaved in ways that make sense to a reader but are dangerous for an AI to touch.

The Blast Radius Protocol inverts the organizing principle. The question isn't "what belongs together conceptually?" It's "what is the smallest unit that can be rewritten in isolation?"

When you structure code this way, a regression becomes physically impossible to propagate. The model can only touch what's in the file. If the file is 80-100 lines with one job, the worst case is that one job breaks — and you know exactly where to look.

---

## The Rules

### Rule 1 — Shard below the human-readable threshold

Each block is a single file, ~80-100 lines, single concern. Not "one component." One *concern*. A hook that fetches data is separate from a hook that transforms it. A component that renders a token is separate from the component that positions it.

The threshold feels uncomfortably small at first. That's correct. The discomfort is the protocol working. You're building for rewrite surface, not reading comfort.

### Rule 2 — The assembly layer is sacred

`App.tsx` (or equivalent root file) is **pure wiring**. Imports and JSX composition only. Zero logic. Zero state management beyond top-level selection. Zero inline styles that encode decisions.

If `App.tsx` has a ternary, something went wrong. That ternary belongs in a block.

The reason is surgical: when you need to change how blocks connect, you change `App.tsx`. When you need to change what a block does, you change that block. These are never the same operation. Keeping them in different files keeps them clean.

### Rule 3 — Every block has a contract

The first three lines of every block are a comment:

```
// Input: what this block receives
// Output: what this block produces or renders
// Must never: the behaviors this block is prohibited from having
```

The "must never" is the most important line. It's not documentation — it's a constraint you hand to the model before you hand it the file. *"This block must never fetch data. It receives data as props and renders."* The model treats that as a hard rail.

This also serves as a forcing function during design. If you can't write a one-line "must never" for a block, the block has too many concerns. Split it.

### Rule 4 — The regression surface is one file

When something breaks, the debug protocol is: identify the block, rewrite only that block, everything else frozen.

This sounds obvious. It isn't how most people work with AI assistants. The instinct when something breaks is to paste the whole component, or the whole page, or "here's everything, find the bug." That's how you get a new bug.

Instead: locate the block. Read its contract. Rewrite just that file. The rest of the application is not in play.

### Rule 5 — Version blocks, not apps

When a block needs a significant change, create `useAuthState.v2.ts` alongside the original rather than editing in place. The original stays frozen. You build and test the new version in isolation, then swap the import in `App.tsx` when it's verified.

This gives you instant rollback at zero cost. It also makes the history of a block's evolution legible — you can see exactly what changed between v1 and v2 without reading a diff.

### Rule 6 — Blocks are the documentation

In standard practice, you write code and then write docs. In the Blast Radius Protocol, the block structure *is* the documentation. A new developer (or a new AI session) can understand the system by reading the contracts. They don't need to trace logic through 400-line files.

This matters acutely for AI pair-programming. Every new session starts cold. The model doesn't remember your last conversation. If your codebase is a dense interconnected mesh, you have to re-explain it every time. If it's a set of small files with contracts, you can paste the relevant ones and the model has everything it needs.

---

## Why This Works With AI Specifically

### The context window is a constraint, not a bug

LLMs have context limits. Traditional wisdom says: give the model more context so it understands the codebase better. The Blast Radius Protocol runs the opposite direction. *Give the model less context, intentionally, because less context means less that can go wrong.*

When you hand a model an 80-line file with a clear contract, it has exactly what it needs and nothing it can accidentally damage. The constraint is the safety mechanism.

### AI doesn't understand "leave this alone"

Humans reading code have a strong instinct for what's load-bearing and what's not. We see a comment that says "don't touch this timing logic" and we don't touch it. Models are worse at this — they understand the instruction but don't have the same visceral sense of risk. They'll "clean up" something that didn't need cleaning.

The protocol solves this architecturally. "Leave this alone" is no longer an instruction you give the model. It's a property of the file structure. The model literally cannot touch what isn't in the file.

### Cold session recovery

One of the most underrated costs of AI-assisted development is session restart overhead. Every time you start a new conversation, you re-explain the project. With a Blast Radius codebase, the recovery procedure is: paste `App.tsx` (the wiring map), paste the relevant blocks, paste their contracts. You're live in one message.

Compare this to a traditional codebase where you have to reconstruct context by pasting hundreds of lines and explaining how they connect. The protocol converts session restart from a 5-minute tax to a 30-second one.

---

## What This Is Not

**It's not a component library philosophy.** This isn't about React patterns or atomic design. It applies equally to backend modules, Python scripts, or any codebase where AI will be doing the editing.

**It's not about file size for its own sake.** The 80-100 line guideline is a consequence of the single-concern rule, not the point. A block that naturally fits in 60 lines stays at 60. A block that genuinely needs 120 is fine. The question is always: can this be rewritten in isolation without understanding anything outside this file?

**It's not defensive programming.** This isn't about error handling or input validation. It's purely about structuring code so that AI edits are physically constrained to the right surface.

---

## The Analogy That Clarifies It

Think of a ship's bulkhead system. A ship doesn't have one large open hull — it has compartments separated by watertight walls. When a compartment floods, the damage is contained. The ship doesn't sink.

Traditional codebases are open hulls. One bad edit floods everything. The Blast Radius Protocol is bulkheads. An AI regression in one block cannot reach another. The ship sails.

---

## Observed Results

This protocol emerged from active practice building production AI infrastructure — a multi-agent AI companion system and a Factorio-style real-time agent dashboard — using LLM pair-programming exclusively. No human wrote code outside of a model context.

Observable results over approximately two weeks of daily use:

- **Regression rate dropped to near zero** after the protocol was formalized. Regressions before the protocol were a consistent daily occurrence. After: isolated to rare cases where a block's contract was ambiguous.

- **Debug time collapsed.** When something breaks, finding the block takes 30 seconds. The model rewrites just that block in one pass. Before: debugging required re-explaining large files and hoping the model found the right line.

- **New AI sessions become cheap.** Onboarding a fresh model context into a Blast Radius codebase takes roughly 3-5 messages. Equivalent ramp-up on a traditional codebase took 15-20 exchanges before the model had enough context to be useful.

- **The codebase scales better than expected.** As the project grew, complexity didn't accumulate in the way it typically does. Each new feature is a new block. Blocks don't entangle because their contracts prohibit it.

---

## The Deeper Principle

The Blast Radius Protocol is really an observation about the nature of AI editing vs. human editing.

Human developers edit large files well because they have deep context, strong working memory, and visceral risk intuition built from years of breaking things. They know which lines are load-bearing.

AI assistants don't have those properties in the same way. They have broad knowledge and strong generation ability, but shallow project-specific memory and limited "leave it alone" instinct.

The correct response isn't to try to make AI assistants behave more like humans. It's to structure the codebase so that AI *generation ability* is the property you're using, while the properties AI *lacks* are rendered structurally irrelevant.

That's the whole protocol. Design for the strengths. Architect away the weaknesses.

---

## Quick Reference

| Principle | Rule |
|---|---|
| File size | ~80-100 lines max, one concern |
| Assembly layer | Pure imports + JSX wiring only, zero logic |
| Block contract | Input / Output / Must Never — first 3 lines |
| Regression scope | Identify block → rewrite that file only |
| Versioning | `block.v2.ts` alongside original, swap import when verified |
| Cold session | Paste App.tsx + relevant blocks + contracts |
| Debug | Name the block. Rewrite just that. Done. |

---

## Examples

See the [`/examples`](/examples) directory for annotated code samples showing what a Blast Radius codebase looks like in practice.

---

*The Blast Radius Protocol is not a framework. It has no dependencies. It requires no tooling. It's a discipline — a way of thinking about code structure when your pair-programming partner is a language model.*
