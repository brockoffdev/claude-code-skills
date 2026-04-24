---
name: refactoring
description: Apply Martin Fowler's refactoring discipline whenever the user asks to "refactor" code, "clean up," "restructure," "improve the design of," "tidy," or "reorganize" an existing codebase or any portion of it. Also use this when the user complains about messy code, technical debt, code smells, duplication, long functions, or asks for improvements that don't add new features. Use it when the user says "this needs a refactor," "refactor this to use X pattern," "refactor the whole codebase," or similar — even if they don't explicitly describe it as refactoring. The skill enforces behavior-preserving changes made in small steps, test-backed safety, smell-driven identification of targets, and the discipline of separating refactoring from feature work (Kent Beck's "two hats"). Do not use it for greenfield code, rewrites in a different language, or when the user is clearly asking for a feature addition rather than a restructuring.
---

# Refactoring (Fowler discipline)

You are applying the discipline from Martin Fowler's *Refactoring: Improving the Design of Existing Code* (2nd ed., 2018). The rest of this file is the operating manual. Read it before touching any code.

## What refactoring actually is

> A change made to the internal structure of software to make it easier to understand and cheaper to modify **without changing its observable behavior.**

Two points this definition insists on:

1. **Behavior is preserved.** If the code's externally observable behavior changes — new functionality, changed API contract, different outputs for the same inputs — that is not refactoring. That is feature work, rewriting, or redesign.
2. **It is the sum of small named steps.** A refactoring (verb) is a sequence of individually small behavior-preserving changes — `Extract Function`, `Inline Variable`, `Move Function`, `Replace Conditional with Polymorphism`, etc. If you "refactor" by rewriting a whole module at once, the code is broken in the middle, you cannot test incrementally, and you have abandoned the discipline. If the code is broken for an afternoon, you were not refactoring.

**The economic justification is speed, not cleanliness.** Refactor to make the next change easier, the current bug smaller, the future feature faster to ship. Do not refactor because the code is ugly or because "good engineering practice" demands it. That framing loses arguments and produces sprawling, low-value cleanup work.

## The Two Hats

At any moment, you are wearing exactly one hat:

- **Feature hat.** You are adding capability. You write new code, add tests for the new behavior, do not change existing structure.
- **Refactor hat.** You restructure existing code. You add no new tests (except ones that cover gaps you discover). You change no behavior. Existing tests must still pass, unchanged, at every step.

Swap hats deliberately. Never wear both. When a user says "refactor this to add caching," that is two hats worth of work disguised as one request — call it out and do the refactoring first (making the addition of caching a small, local change), then add the caching in a separate step under the feature hat. Commit them separately when it clarifies review; otherwise, still think of them as separate activities.

## Before you touch any code: the prerequisites

Work through these in order. Do not skip.

1. **Understand the request.** Is it preparatory (making it easy to add a feature next), comprehension (refactoring to understand the code), litter-pickup (noticed it's bad while passing through), or planned (dedicated effort)? The category changes scope and urgency. If the user's language is ambiguous, ask — one sentence is enough. Do *not* ask if the request is clearly scoped.

2. **Find the tests.** Refactoring without tests is not safe. Scan for a test suite (`pytest`, `jest`, `go test`, `cargo test`, `rspec`, whatever the project uses). Run it. Confirm it passes *before* you change anything. This is the baseline — the thing you will re-run after every small step.

3. **If there are no tests, or coverage is thin in the area you're about to touch:** stop and surface this to the user. Options, in roughly this order of preference:
   - Write characterization tests (tests that pin down current behavior, warts and all) for the area being refactored, then refactor.
   - Restrict yourself to refactorings the language tooling can do safely — rename, extract, inline — and lean on the IDE/LSP/type checker rather than custom logic.
   - Defer the refactoring until tests exist.
   - Proceed anyway with an explicit, named risk accepted by the user.
   For legacy code with poor tests, the canonical reference is Feathers, *Working Effectively with Legacy Code*. The core technique: find "seams" where tests can be inserted, refactor those first to make the rest testable.

4. **Decide whether to refactor at all.** Skip refactoring if: (a) the code is ugly but you don't need to touch it, (b) it's easier to rewrite than to refactor (a judgment call — usually proven by trying to refactor for a bit and finding it hopeless), (c) the change is too small to justify restructuring around it.

## The refactoring workflow

Once prerequisites are satisfied:

1. **Identify the smell(s).** Use the named vocabulary in `references/smells.md`. Do not say "this is messy" — say "this is `Long Function` and `Feature Envy`" or "this is `Shotgun Surgery` across these three modules." Naming smells precisely drives precise refactorings.

2. **Pick the named refactoring(s).** For each smell, the catalog prescribes specific named refactorings. See `references/refactorings.md` for the vocabulary. Choose the smallest refactoring that addresses the smell. Do not reach for `Replace Conditional with Polymorphism` when `Decompose Conditional` is sufficient.

3. **Plan the sequence.** Write down the order of small steps you intend to take. Each step should be individually behavior-preserving and individually testable. Example for a Long Function with duplicated logic inside:
   - `Extract Variable` on the unclear expression on line 42.
   - `Extract Function` for the loop body into `calculateLineItem`.
   - `Extract Function` for the summary block into `formatSummary`.
   - `Inline Variable` on the temp that's no longer useful.
   - `Rename Variable` on `x` to `taxRate`.

4. **Execute one step at a time.** For each step:
   - Make the single change.
   - Run the tests. All of them. Watch them pass.
   - If tests fail, **revert the step** — do not debug a broken refactoring forward. Take a smaller step.
   - If the language's refactoring tool (LSP, `rust-analyzer`, `gopls`, IntelliJ, etc.) can do the change safely, prefer the tool over manual edits. The tool operates on the AST; you operate on text, with all the risks that implies.
   - Commit (or stage) frequently. A good rhythm is one commit per named refactoring.

5. **Stop when the code communicates its intent.** Fowler's test: "When someone needs to make a change, can they find the code to be changed easily and make the change quickly without introducing errors?" If yes, stop. Refactoring is bounded. There is always more to clean up; the point is to clear the runway for the next thing, not to pursue platonic perfection.

## The rhythm

The single most important thing to internalize from the book: **tiny steps with a working, passing test suite between each**. Beginners think this is slow. It is faster, because you spend no time in the debugger. You never have to ask "what did I break?" because you changed one named thing and the tests either passed or they didn't.

If you find yourself making several changes at once, or going more than a minute or two without running tests, you have drifted out of the discipline. Pause, revert to the last green state if necessary, and take smaller steps.

## Rule of Three

Do not extract abstractions on the first occurrence. Wait until you see the pattern three times. Two occurrences is a coincidence; three is a pattern worth naming. Aggressive DRY-ing on first duplication produces premature abstractions that are usually wrong and hard to undo.

Conversely: the third time you write something similar, that is the moment to refactor — not later.

## Preparatory refactoring

> "For each desired change, make the change easy (warning: this may be hard), then make the easy change." — Kent Beck

When the user asks for a feature and the current structure makes it awkward, do not just wedge the feature in. First refactor until the feature becomes a small, local addition. Then add it. This takes longer on the clock but produces better code faster overall — and the refactoring itself is almost always worth more than the feature.

If the user is asking for a feature (not a refactor) and you notice this is the right move, surface it: "Before I add X, I'd like to restructure Y — it will make X a small change instead of a sprawling one. Here's the plan." Then proceed under the two-hats discipline.

## Databases and published APIs

Two special cases where the "behavior-preserving" bar is stricter:

- **Database schemas.** Refactor via parallel change (expand-contract): add the new field/table, backfill, migrate readers, migrate writers, remove the old. Never do it in one commit. Ambler & Sadalage, *Refactoring Databases*, is the reference.
- **Published APIs / interfaces consumed by code you don't control.** Rename/change cannot break callers. Use pass-through shims: introduce the new signature, have the old one delegate to it, deprecate the old, remove the old only when you've confirmed no callers remain (or never, if you can't confirm).

## When the user says "refactor" but doesn't mean it

Several requests come disguised as refactoring. Name what's actually happening:

- "Refactor this to Python" — translation/rewrite, not refactoring. Behavior preservation is the goal but the techniques are different; the Fowler catalog doesn't apply cleanly.
- "Refactor this to use async/caching/a new library" — feature or architecture change. Do any preparatory refactoring first, then make the change under the feature hat.
- "Refactor the whole codebase" — planned refactoring, needs scoping. Ask which pain point is driving it; refactor the hottest area first. Avoid a multi-week dedicated refactor if incremental opportunistic refactoring would do.
- "Refactor to make it faster" — performance optimization, not refactoring. The two are similar in mechanics but opposite in goal; optimization is allowed to make code harder to work with in exchange for speed. Be explicit about which you're doing.

## Output expectations

When you finish a refactoring session, the user should receive:

1. A clear statement of what smells were identified and what named refactorings were applied, in order.
2. The commits (or patch) laid out as a sequence of small, named steps.
3. Confirmation that the test suite passed at the start, passed at the end, and (ideally) passed between each step.
4. Anything left un-refactored, with a brief note on why (out of scope, deferred, requires tests first, etc.).

Do not produce a giant "here is the refactored version" diff with no trail of small steps. That is a rewrite, not a refactoring, and it cannot be reviewed or rolled back step-wise.

## References

- `references/smells.md` — the 24 code smells from Fowler's catalog, each with the refactorings that address them. Consult when identifying what to change.
- `references/refactorings.md` — the named refactorings vocabulary with motivation and mechanics summaries. Consult when choosing *how* to change it.

Read the relevant reference entry before applying a refactoring you haven't done in a while — the mechanics section exists because even experienced refactorers forget the small safety steps.
