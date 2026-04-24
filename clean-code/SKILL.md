---
name: clean-code
description: Apply the standards from Robert C. Martin's "Clean Code" when the user asks you to "clean up" code, "apply clean code," review code quality, "make this cleaner/more readable," improve naming, shrink long functions, straighten out comments, or otherwise critique existing code against craftsmanship standards. Also use when the user asks for a code review, asks "what's wrong with this code," complains about readability, or asks to audit a file or module for smells. This skill covers the opinionated line-by-line standards — naming, function size, comments, error handling, tests, classes — that the book codifies. Do not use for greenfield architecture, algorithm design, or requests that are clearly about performance optimization or language-specific idioms.
---

# Clean Code (Martin discipline)

You are applying the standards from Robert C. Martin's *Clean Code: A Handbook of Agile Software Craftsmanship* (2008). Unlike pure refactoring — which is about the mechanical discipline of behavior-preserving transformation — Clean Code is about *taste*: the strong opinions about what clean code looks like at the level of names, functions, comments, classes, and tests. This file is the operating manual for those opinions.

## Stance

The book is unapologetically opinionated. Uncle Bob presents his prescriptions as "absolutes" and then — in the same breath — explicitly calls the book a "school of thought," acknowledging other schools have equal claim to professionalism. Adopt the same posture: apply the rules confidently as defaults, but treat them as strong defaults rather than laws. A few of the book's prescriptions (e.g., extreme function smallness, heavy use of exceptions over nulls in languages where that's idiomatic) are genuinely contested in the modern industry. Note those trade-offs when they matter; don't evangelize past the point of usefulness.

**The central claim worth taking seriously, however, is this: reading code is ~10× more common than writing it, and the professional obligation is to optimize for the reader.** Every decision in this skill flows from that.

## What "clean code" means, briefly

Several of the authors in the book's opening chapter offer definitions. The intersection:

- **Readable as prose.** The code tells the story of what the system does, in names that a human can follow top to bottom.
- **Does one thing well.** Each function, class, and module has a single focus that can be stated without "and," "or," or "but."
- **No duplication.** The same idea is expressed once, in one place.
- **Fully tested.** Code without a self-validating test suite is not clean, regardless of its other virtues.
- **Easy to change.** The real test is whether the next person can make a change quickly, find the right place, and not introduce a bug.
- **Cared for.** Michael Feathers' one-word definition: "Clean code always looks like it was written by someone who cares."

## The Boy Scout Rule

> *Leave the campground cleaner than you found it.*

When touching a file for any reason, improve it a little — better name here, smaller function there, one piece of duplication removed. The improvements don't have to be large; they just have to be consistent. If every commit leaves the code incrementally cleaner, rot cannot accumulate.

This is *the* operating principle of the skill. It is also how this skill coexists with feature work: you are not expected to stop everything and refactor the whole module — just leave it better than you found it.

## The workflow

1. **Identify what's wrong in the book's vocabulary.** Don't say "this code is messy." Say "this is a `Long Function` with `Flag Arguments` and `Primitive Obsession`, and the name is disinformative per `N4`." The catalog in `references/smells.md` is the diagnostic vocabulary — 66 named smells and heuristics (C1–C5, E1–E2, F1–F4, G1–G36, J1–J3, N1–N7, T1–T9).

2. **Pick the cleanest change that reveals intent.** For each smell, `references/principles.md` gives the prescriptive rules from the book's core chapters — Meaningful Names (Ch 2), Functions (Ch 3), Comments (Ch 4), Error Handling (Ch 7), Classes (Ch 10), Unit Tests (Ch 9). Look up the specific rule before writing a fix — the rules are more specific than general "craftsmanship" vibes.

3. **Make the change in the smallest increment that reveals intent.** Rename one variable. Extract one function. Flatten one nested conditional. A clean-code session is a sequence of small, visible improvements — not a rewrite.

4. **Run the tests.** The book is emphatic: "Code without tests is not clean." If tests exist, they must continue to pass. If they don't exist, say so before making non-trivial changes: "I'd like to add characterization tests for this region before I clean it up. Otherwise any change I make is a change under uncertainty."

5. **Stop when the code reads like what it does.** Ward Cunningham's test: "You know you are working on clean code when each routine turns out to be pretty much what you expected." If the next reader would nod rather than squint, stop. There's always more cleanup; the point is clearing the runway for the next change, not achieving platonic perfection.

## The six big defaults

If you internalize nothing else from this skill, internalize these six from the book's core chapters.

1. **Names reveal intent.** A name should answer why the thing exists, what it does, and how it is used. If a name requires a comment to be understood, it's the wrong name. Rename freely — modern tools make renaming cheap, and a better name is always worth a commit. (Ch 2; see `references/principles.md` § Names.)

2. **Functions are small and do one thing.** The book's target is a few lines. Most modern practitioners accept a looser ceiling, but the direction is correct: if you can extract a function from a function and the extracted name is *not* just a restatement of its body, the original was doing more than one thing. One level of abstraction per function. (Ch 3; G30, G34.)

3. **Comments compensate for our failure to express ourselves in code.** They are sometimes necessary (legal, TODO, warnings, intent of a non-obvious decision) but are never a substitute for a better name or a smaller function. Delete commented-out code on sight — source control remembers. (Ch 4; C1–C5.)

4. **Use exceptions, not error codes, where the language supports it.** Define exception classes around what the *caller* needs to do, not around what the callee knows. Don't return null when a sensible default (Null Object / Special Case) would simplify every caller. Don't pass null. (Ch 7; though note this is language-specific — idioms differ in Go, Rust, etc.)

5. **Classes are small and have one reason to change (SRP).** Class size is measured in responsibilities, not lines. If you can't summarize the class in 25 words without "and" or "or," it has too many responsibilities. High cohesion — each method should use most of the fields — is the operational test. (Ch 10.)

6. **Tests are first-class. F.I.R.S.T.** Fast, Independent, Repeatable, Self-Validating, Timely. Test code is held to the same cleanliness standards as production code, because dirty tests rot the same way and eventually get abandoned, at which point the code rots with them. (Ch 9.)

## Translating faithfully across languages

The book's examples are Java. Most of its advice generalizes directly — naming, function size, duplication, comments, SRP, tests. A few pieces are Java-specific and need translation:

- Getter/setter discussions → whatever the language's idiom is (Python properties, Rust impl blocks, Go struct fields).
- Exception handling → varies. In Go or Rust, error values are idiomatic and "use exceptions over error codes" doesn't apply. The underlying principle — that error handling shouldn't obscure the main path — does.
- `m_` prefixes and Hungarian notation → more broadly, "don't encode type or scope in names." Almost every language community has its own version of this anti-pattern.
- JUnit specifics → the general test principles (F.I.R.S.T., Build-Operate-Check, Arrange-Act-Assert) apply in any test framework.

When working in a non-Java project, apply the spirit of the rule, not the letter. Don't mechanically rename `set_foo` to `setFoo` because the book uses camelCase — respect the project's conventions (G24: "Follow Standard Conventions").

## When the user says "clean up" but means something else

- *"Clean up the dependencies in this package"* — dependency management task, not Clean Code. Do that directly.
- *"Clean up the git history"* — version control task.
- *"Clean up performance"* — optimization. Not what this skill is for; in fact Clean Code explicitly warns against sacrificing clarity for optimization without justification.
- *"Clean up my writing style"* — code style / formatter task. Handle with the project's linter/formatter first; this skill addresses things those tools can't catch.

## Output expectations

A clean-code session should produce:

1. A list of the specific smells and rule violations found, named with the book's vocabulary (G5, N4, C3, F3, etc. — see `references/smells.md`).
2. A diff showing each change, ideally one per named rule violation.
3. Confirmation that tests passed before and after.
4. A brief note on anything that was flagged but not changed (contested rule, out-of-scope, requires architectural change, etc.).

Do not produce "here is a cleaner version" as a monolithic rewrite. The value is in the specific named improvements, each visible and each reviewable.

## References

- `references/smells.md` — the full Chapter 17 catalog of 64 smells and heuristics. Consult when identifying what's wrong.
- `references/principles.md` — the prescriptive rules from Ch 2 (Names), Ch 3 (Functions), Ch 4 (Comments), Ch 7 (Error Handling), Ch 9 (Tests), Ch 10 (Classes). Consult when deciding what "clean" should look like.

Read the relevant reference section before applying a rule you haven't used recently — the book's specificity is the whole point.
