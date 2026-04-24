# Smells and Heuristics

The full catalog from *Clean Code* Chapter 17, preserving Uncle Bob's numbering so citations (`G5`, `N4`, `C3`, etc.) are unambiguous. Use these codes in output: "this has `G30` and `N7`" tells a reviewer exactly what you found, and the book's index lets them look up the full rationale.

---

## Comments (C1–C5)

**C1: Inappropriate Information.** Change histories, author metadata, issue numbers, SPR numbers — anything belonging in version control or issue tracking. Comments are for technical notes about the code, not meta-data.

**C2: Obsolete Comment.** A comment that used to be true. Obsolete comments migrate away from the code they once described and become floating islands of misinformation. Update or delete on sight.

**C3: Redundant Comment.** A comment that says what the code already says. `i++; // increment i`. Javadoc that just repeats the signature. Delete.

**C4: Poorly Written Comment.** A comment worth writing is worth writing well. Correct grammar, clear wording, no rambling, no obviousness.

**C5: Commented-Out Code.** An abomination. Nobody will delete it because everyone assumes someone else needs it. Delete it — source control remembers.

---

## Environment (E1–E2)

**E1: Build Requires More Than One Step.** One command to check out, one command to build. If your build needs a scavenger hunt of JARs or context-dependent scripts, fix the build.

**E2: Tests Require More Than One Step.** One button, one command — all tests run. If it's not this easy, people won't run them.

---

## Functions (F1–F4)

**F1: Too Many Arguments.** Zero is best, one next, two ok, three questionable, more almost always wrong. (See also Ch 3 "Function Arguments"; consider argument objects.)

**F2: Output Arguments.** Readers expect arguments to be inputs. If the function must change state, have it change the state of the object it's called on (`this` in OO languages).

**F3: Flag Arguments.** A boolean parameter loudly declares the function does more than one thing. Split into two functions with descriptive names.

**F4: Dead Function.** Methods never called are waste. Delete them. Version control remembers.

---

## General (G1–G36)

**G1: Multiple Languages in One Source File.** Java + XML + HTML + JS + JavaDoc + shell all in one file. Minimize both the number and extent.

**G2: Obvious Behavior Is Unimplemented.** Principle of Least Astonishment. A `stringToDay` function should handle case variations and common abbreviations. When intuition fails, readers lose trust in every function name.

**G3: Incorrect Behavior at the Boundaries.** Intuition is unreliable at edges. Test every boundary, every corner case, every quirk — don't trust that the elegant algorithm "obviously" works.

**G4: Overridden Safeties.** Turning off compiler warnings, disabling failing tests "for now," overriding `serialVersionUID`. Small conveniences that produce massive risk. Each override is a small Chernobyl.

**G5: Duplication.** Kent Beck: "Once, and only once." Every duplication is a missed abstraction. Three forms:
- *Exact copies* → extract a function.
- *Repeated switch/if chains on the same type* → polymorphism.
- *Similar algorithms with different details* → Template Method or Strategy pattern.

One of the most important rules in the book. Take it very seriously.

**G6: Code at Wrong Level of Abstraction.** Base classes shouldn't hold details only relevant to some derivatives. High-level constants shouldn't sit in low-level modules. Separation must be complete — no leakage.

**G7: Base Classes Depending on Their Derivatives.** Base classes that mention the names of their derivatives. Violates the whole point of the split. Rare exceptions exist (finite state machine patterns) but the default is: base classes know nothing of their subclasses.

**G8: Too Much Information.** Well-defined modules have small interfaces that let you do a lot with little. Hide data, hide utility functions, hide constants, hide temporaries. Don't create modules with lots of accessible methods and variables.

**G9: Dead Code.** Code that isn't executed. Conditions that can't be true. Catches for exceptions that can't be thrown. Find it and give it a decent burial.

**G10: Vertical Separation.** Variables and functions should be defined close to where they are used. Local variables declared just above their first use. Private helper right below the function that calls it.

**G11: Inconsistency.** If you do X one way in one place, do it the same way everywhere. Consistency is a form of information; inconsistency is disinformation.

**G12: Clutter.** Unused variables. Never-called functions. Uninformative comments. No-op defaults. Delete.

**G13: Artificial Coupling.** Things that don't depend on each other shouldn't be coupled. An enum declared inside one class that's used by several should be lifted to the shared level.

**G14: Feature Envy.** A method more interested in another class's data than its own. Move the method to the class it envies, or extract the envious piece and move that.

**G15: Selector Arguments.** Boolean or enum arguments that choose between behaviors. Worse than flag arguments because the selector isn't even binary. `calculateWeeklyPay(boolean overtime)` should be two functions.

**G16: Obscured Intent.** Code that's hard to understand because it's dense, poorly structured, or uses cryptic names. Use explanatory variables (G19), better names, smaller functions.

**G17: Misplaced Responsibility.** Code that doesn't live where a reader would expect. Principle of Least Surprise again. Constants belong at the level that uses them; utility functions belong with what they utilize.

**G18: Inappropriate Static.** Default to instance methods. Make something static only when you're sure polymorphism will never be wanted on it. When in doubt, instance.

**G19: Use Explanatory Variables.** Break complex expressions into intermediate variables with meaningful names. You almost can't overdo this — opaque calculations become transparent when the pieces are named.

```java
Matcher match = headerPattern.matcher(line);
if (match.find()) {
    String key = match.group(1);
    String value = match.group(2);
    headers.put(key.toLowerCase(), value);
}
```

The explanatory variables `key` and `value` make the matcher's groups self-documenting.

**G20: Function Names Should Say What They Do.** `date.add(5)` — five what? Days? Weeks? Does it mutate `date` or return a new one? If you must look at the implementation to know, the name is wrong. Prefer `date.addDays(5)` or `date.plusDays(5)` (immutable).

**G21: Understand the Algorithm.** Code that "works" because you poked at it until the tests passed is not clean code. Refactor until the algorithm is obvious — you should *know* it's correct, not merely observe that tests pass.

**G22: Make Logical Dependencies Physical.** If module A assumes something about module B (say, a page size constant), don't hardcode the assumption in A. Have A ask B for the value. A logical dependency pretending not to exist is a bomb.

**G23: Prefer Polymorphism to If/Else or Switch/Case.** Most switches are there because the author reached for the obvious tool, not the right one. Every switch should be suspect. The "ONE SWITCH rule": at most one switch per type of selection, and it must create polymorphic objects used everywhere else.

**G24: Follow Standard Conventions.** Every team should have conventions (naming, bracing, structure) and everyone should follow them. It doesn't matter *which* conventions as much as that they're consistent. The code itself should be the documentation.

**G25: Replace Magic Numbers with Named Constants.** Not just numbers — any unexplained literal. `"John Doe"` used as a test employee name is as much a magic value as `7777`. Exception: some constants (π, 0, 1, small arithmetic multipliers) are self-explanatory in context.

**G26: Be Precise.** Don't be lazy about decisions. If a function might return null, check for null. If you query for what you think is one row, verify. Floats for currency is criminal. Locks matter when concurrency is possible.

**G27: Structure Over Convention.** Prefer structures that enforce design decisions over conventions that merely suggest them. Abstract base classes with required methods beat switch-on-enum patterns because nothing prevents a new case in a switch from being forgotten.

**G28: Encapsulate Conditionals.** `if (shouldBeDeleted(timer))` is better than `if (timer.hasExpired() && !timer.isRecurrent())`. Name the business rule.

**G29: Avoid Negative Conditionals.** `if (buffer.shouldCompact())` reads easier than `if (!buffer.shouldNotCompact())`. Negatives require an extra mental step.

**G30: Functions Should Do One Thing.** A function with multiple sections that do different things should be split. See Ch 3 "Do One Thing."

**G31: Hidden Temporal Couplings.** When A must happen before B, make the API reflect that. Don't let callers call them in any order. Pass A's output into B as an argument — the type system enforces the sequence.

**G32: Don't Be Arbitrary.** Structure code with a reason, and make the reason visible. If something looks arbitrary, readers will feel free to change it; if there's a clear pattern, they'll preserve it.

**G33: Encapsulate Boundary Conditions.** Boundary calculations in a method body invite off-by-one errors in every place they appear. Put the `+1` or `-1` in one named variable or function.

**G34: Functions Should Descend Only One Level of Abstraction.** The Stepdown Rule. Every statement in a function should be at one level below the function's name. When you see a mix of high and low levels in one function, extract.

**G35: Keep Configurable Data at High Levels.** Configuration should be defined at the top of the system and passed down, not hardcoded in low-level functions. High-level configuration is easy to change; buried constants are hard.

**G36: Avoid Transitive Navigation.** The Law of Demeter. `a.getB().getC().doSomething()` is "train wreck" code — A knows about B's internals, which knows about C's internals, which makes the design impossible to change later. Immediate collaborators should offer what you need; you shouldn't roam the object graph.

---

## Java (J1–J3)

Language-specific. Translate the principle to whatever language you're in.

**J1: Avoid Long Import Lists by Using Wildcards.** (Java-specific — dubious in Python or TypeScript where explicit is the convention.)

**J2: Don't Inherit Constants.** Putting constants in an interface and inheriting the interface to get them is "cheating the scoping rules." In Java, use `import static`. The general principle: don't abuse inheritance to avoid qualified names.

**J3: Constants versus Enums.** Use enums over `public static final int` where the language supports them. Enums carry meaning ints don't. (Corresponding rule in most typed languages: use the type system to encode domain concepts.)

---

## Names (N1–N7)

**N1: Choose Descriptive Names.** Names are 90% of what makes code readable. Take the time. Re-evaluate names as meanings drift. Renaming is cheap; leaving a bad name is expensive.

**N2: Choose Names at the Appropriate Level of Abstraction.** A `Modem` interface with `dial(String phoneNumber)` ties the abstraction to PSTN. `connect(String connectionLocator)` generalizes. Don't commit to implementation details in the interface name.

**N3: Use Standard Nomenclature Where Possible.** If you're using the Decorator pattern, call it `FooDecorator`. If the domain has a term (Eric Evans' "ubiquitous language"), use it. Conventions save mental effort.

**N4: Unambiguous Names.** `doRename()` that internally calls `renamePage()` — what's the difference? Be unambiguous even when the name has to be long. A better name is worth characters.

**N5: Use Long Names for Long Scopes.** `for (int i = 0; i < n; i++)` is fine if `i` is alive for 5 lines. For a variable whose scope is a whole class or file, the name needs to carry weight. Length should be proportional to scope.

**N6: Avoid Encodings.** `m_` prefixes, Hungarian notation, type encodings in names (`strName`, `iCount`). Modern tools make these obsolete. Keep names clean.

**N7: Names Should Describe Side-Effects.** `getOos()` that actually creates the ObjectOutputStream if it doesn't exist yet should be `createOrReturnOos()` (or restructured to avoid the side effect). Don't hide side effects in innocent-sounding getters.

---

## Tests (T1–T9)

**T1: Insufficient Tests.** "That seems like enough" is the wrong metric. A test suite should cover everything that could plausibly break.

**T2: Use a Coverage Tool.** Coverage tools find untested branches fast. They're not the whole story (100% line coverage ≠ good tests), but they reveal gaps that are invisible otherwise.

**T3: Don't Skip Trivial Tests.** Cheap to write, high documentary value. Even "this constructor doesn't throw" is a test worth having.

**T4: An Ignored Test Is a Question About an Ambiguity.** If you're not sure what the behavior should be, write the test and `@Ignore` it (or comment it out with a note). It becomes a visible open question in the suite.

**T5: Test Boundary Conditions.** We get the middle right and misjudge the edges. Null, empty, one-element, max-size, zero, negative, past-end-of-buffer — enumerate them explicitly.

**T6: Exhaustively Test Near Bugs.** Bugs cluster. When you find one, test around it thoroughly — you'll almost always find at least one more.

**T7: Patterns of Failure Are Revealing.** "All tests with strings over 5 characters fail" or "every test passing a negative number fails" — patterns in test output diagnose the defect faster than any single failure.

**T8: Test Coverage Patterns Can Be Revealing.** Look at which lines run when a failing test runs; the coverage pattern points at the defect.

**T9: Tests Should Be Fast.** Slow tests don't get run. When pressure rises, slow tests get cut. Ruthlessly keep the suite fast.

---

## Cross-references to other chapters

Ch 17's catalog references earlier chapters frequently. The most important of those (and the ones that belong in diagnostic vocabulary) are in `principles.md`:

- Names: N1–N7 above, plus Ch 2 "Meaningful Names" (intention-revealing, avoid disinformation, meaningful distinctions, pronounceable, searchable, no mental mapping, class names are nouns, method names are verbs, one word per concept, no puns, add meaningful context, no gratuitous context).
- Functions: F1–F4 and G30/G34 above, plus Ch 3 "Functions" (Small, Do One Thing, One Level of Abstraction, Stepdown Rule, switch statements, descriptive names, argument count & shape, side effects, Command-Query Separation, exceptions over error codes, Extract Try/Catch, DRY).
- Comments: C1–C5 above, plus Ch 4 "Comments" (comments do not make up for bad code; good comments: legal, informative, intent, clarification, warnings, TODO, amplification, Javadoc in public APIs).
- Error handling: Ch 7 (use exceptions over return codes, try-catch-finally first, provide context, define exceptions by caller's needs, don't return null, don't pass null, Special Case / Null Object).
- Tests: T1–T9 above, plus Ch 9 "Unit Tests" (Three Laws of TDD, Clean Tests, Build-Operate-Check, domain-specific test language, one concept per test, F.I.R.S.T.).
- Classes: Ch 10 "Classes" (classes should be small, SRP, cohesion, organizing for change).
