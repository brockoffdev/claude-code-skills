# Principles

The prescriptive rules from the book's core chapters. `smells.md` tells you what's *wrong*; this file tells you what *right* looks like.

Organized by chapter. Read the relevant section before applying — the book is specific, and the specificity is the whole point.

---

## Meaningful Names (Ch 2)

Names are 90% of readability. The following rules, applied consistently, dominate everything else in this book.

**Use Intention-Revealing Names.** The name should answer: why does this exist? What does it do? How is it used? If the answer requires a comment, the name is wrong.

```java
int d; // elapsed time in days          // WRONG
int elapsedTimeInDays;                  // RIGHT
int daysSinceCreation;                  // also fine, depending on use
```

**Avoid Disinformation.** Don't use words whose existing meaning conflicts with yours. Don't call something `accountList` unless it's actually a `List` — `accounts` or `accountGroup` is better. Beware names that differ in small ways (`XYZControllerForEfficientHandlingOfStrings` vs `XYZControllerForEfficientStorageOfStrings` — the eye slides right past the difference).

**Make Meaningful Distinctions.** `a1, a2, a3` tells you nothing. `ProductInfo` vs `ProductData` is a distinction without a difference. If you have two things with different names, the names should tell you how they differ.

**Use Pronounceable Names.** `genymdhms` (generation year month day hour minute second) is unpronounceable and therefore undiscussable. `generationTimestamp` can be read aloud in a meeting.

**Use Searchable Names.** Single letters and numeric literals are hard to grep. `MAX_CLASSES_PER_STUDENT` is easy to find; `7` is not. Use single letters only for very short scopes (loop counter in a 5-line for-loop is fine).

**Avoid Encodings.** No Hungarian notation (`strName`, `iCount`), no `m_` prefixes, no `I` prefix on interfaces. Type systems and IDEs handle this now; encoding adds clutter and makes renames harder.

**Avoid Mental Mapping.** The reader shouldn't have to remember that `r` is "the lower-cased version of the URL with host and scheme removed." Just name it `urlPath`. Smart programmers want to show off mental juggling; professional programmers write code others can understand.

**Class Names are nouns.** `Customer`, `WikiPage`, `Account`, `AddressParser`. Avoid `Manager`, `Processor`, `Data`, `Info` — weasel words that usually indicate an unfocused class.

**Method Names are verbs or verb phrases.** `postPayment`, `deletePage`, `save`. Accessors, mutators, and predicates use the `get`/`set`/`is` convention.

**Pick One Word Per Concept.** Don't have `fetch`, `retrieve`, and `get` doing the same thing across classes. Consistent vocabulary is a great kindness to readers.

**Don't Pun.** Inverse of "one word per concept." If `add` means "create a new value by combining two existing ones" everywhere else, don't reuse it for "append to a collection" — use `insert` or `append`.

**Use Solution Domain Names.** Readers are programmers. Pattern names (`AccountVisitor`), algorithm names (`JobQueue`), CS terms — all fair game.

**Use Problem Domain Names.** When there's no solution-domain term, use the problem-domain term. The maintainer can ask a domain expert.

**Add Meaningful Context.** Names like `state` are clearer as `addrState` when they live in an address. But be careful — the better solution is often to group them into a class (`Address`) where the context is implicit.

**Don't Add Gratuitous Context.** Don't prefix every class in "Gas Station Deluxe" with `GSD`. You're working against your tools; the IDE's autocomplete becomes useless.

---

## Functions (Ch 3)

**Small!** Functions should be small. The book's target is a few lines; the direction (not the specific line count) is what matters. A function too big to hold in your head is a function doing too much.

**Blocks and Indenting.** The body of an `if`, `else`, or `while` should usually be one line — a function call. Nesting indent levels greater than one or two is a smell.

**Do One Thing.** The canonical rule: *functions should do one thing, they should do it well, they should do it only.* Operational tests:

- Can you describe the function as a TO-paragraph? "TO `renderPage`, we check if the page is a test page, and if so, include setup and teardown, then render in HTML."
- If you can extract a function from this one and the extracted function's name is *not* merely a restatement of its body, the original was doing multiple things.
- If the function has obvious "sections" (`// declarations`, `// initialize`, `// sieve`), it's doing multiple things.

**One Level of Abstraction per Function.** Don't mix high-level (`getHtml()`) and low-level (`.append("\n")`) in the same function. The mix confuses readers and tends to attract more detail over time.

**Reading Code from Top to Bottom: The Stepdown Rule.** Code should read as a top-down narrative. Each function is followed by those one level below it in abstraction. The program reads like a set of TO-paragraphs.

**Switch Statements.** Switches are inherently "do N things" and tend to violate SRP and OCP. The book's rule: a switch is tolerable only if it appears *once*, is used to create polymorphic objects (e.g., in an abstract factory), and is hidden behind an inheritance relationship. See G23.

**Use Descriptive Names.** A long descriptive name beats a short enigmatic one. A long descriptive name beats a long descriptive comment. Try multiple names, read each one in context, pick the clearest. Don't be afraid to rename.

**Function Arguments.** Argument counts, in order of preference:

- 0 (niladic) — best.
- 1 (monadic) — next best. Two common forms: *asking* (`boolean fileExists(name)`) or *transforming* (`InputStream fileOpen(name)`). A third form, *event handler*, has an input but no output (`void passwordAttemptFailedNtimes(int)`) — use carefully.
- 2 (dyadic) — harder. OK when the two arguments are naturally ordered components of one value (`Point(x, y)`). Otherwise, consider making one a field or restructuring.
- 3 (triadic) — think very carefully. The ordering/ignoring problems double.
- More than 3 — almost never. Wrap in an argument object.

**Flag Arguments are ugly.** A boolean in a signature declares "this function does more than one thing." `render(pageData, true)` is opaque at the call site. Split into two functions: `renderForSuite()` and `renderForSingleTest()`.

**Have No Side Effects.** A function named `checkPassword` that also initializes the session is lying. Either rename to reveal both actions, or separate into two functions.

**Output Arguments.** Readers expect information out through return values. If a function must mutate something, mutate its owning object.

**Command Query Separation.** A function either *does* something or *answers* something. Never both. `public boolean set(String attr, String value)` produces confusion — is that a getter or a setter? Split.

**Prefer Exceptions to Returning Error Codes.** Error codes force the caller to check immediately at every call site, and produce deeply nested `if` ladders. Exceptions separate happy path from error handling. (In languages where error values are idiomatic — Go, Rust — apply the underlying principle: keep error handling from obscuring the main flow.)

**Extract Try/Catch Blocks.** Try/catch blocks are ugly. Extract their bodies into named functions so the try becomes a one-liner.

**Error Handling Is One Thing.** A function that handles errors should do *only* that. If `try` appears, it should be the first word, and the function should end after the `catch`/`finally`.

**Don't Repeat Yourself (DRY).** Every duplication is a missed abstraction. Uncle Bob considers this one of the most important rules — many design practices (OO, structured programming, subroutines, Codd's normal forms) are at heart strategies against duplication.

**Structured Programming.** Dijkstra's single-entry/single-exit rule is overkill for small functions. Early returns, `break`, `continue` are fine in a short function. `goto` is never fine.

---

## Comments (Ch 4)

**Comments compensate for our failure to express ourselves in code.** They are a necessary evil — when you write one, wince a little. When you managed not to write one, pat yourself on the back.

Comments lie. Not always, not intentionally, but the older a comment is and the farther from its code, the more likely it is wrong. The code is the only reliable source of truth about what the code does.

**Comments Do Not Make Up for Bad Code.** The reflex to comment messy code is a sign to clean the code instead.

**Explain Yourself in Code.**

```java
// Check to see if the employee is eligible for full benefits
if ((employee.flags & HOURLY_FLAG) && (employee.age > 65))
```

vs.

```java
if (employee.isEligibleForFullBenefits())
```

The second doesn't need a comment. Most comments fall to this treatment.

### Good Comments (rare)

**Legal Comments.** Copyright, license notices. Collapse in the IDE if they're verbose.

**Informative Comments.** When a name can't carry all the information. A regex's format string, for instance.

**Explanation of Intent.** *Why* you did something, especially when the choice is non-obvious. "Sorting objects of this class higher than others" is valuable context.

**Clarification.** Translating an obscure return value or argument into something readable — especially for code you can't change (library APIs). Risky: the clarification can drift from the code.

**Warning of Consequences.** "Don't run this test unless you have time to kill." "This test is time-consuming; set up before running."

**TODO Comments.** Future work. Most IDEs list all TODOs. Don't abuse — scan periodically and resolve them.

**Amplification.** Drawing attention to something easy to miss. "This trim is important — it removes the leading characters which will corrupt parsing if they remain."

**Javadocs in Public APIs.** If you're authoring a library others will use, Javadoc (or equivalent) is essential.

### Bad Comments (most)

Mumbling, redundant, misleading, mandated, journal (check-in history), noise (`// default constructor`), scary noise, position markers (`/////////////////`), closing brace comments, attributions (`// added by Rick`), commented-out code, HTML tags in comments, nonlocal information, too much information, inobvious connection, function headers (for small well-named functions), Javadocs in nonpublic code. See also C1–C5 in `smells.md`.

---

## Error Handling (Ch 7)

The overall goal: error handling should not obscure the main logic. If you can't see what the code does because of the error handling, the error handling is wrong.

**Use Exceptions Rather Than Return Codes.** (In languages that support exceptions well — Java, C#, Python, JavaScript, C++.) Exceptions let the algorithm of the happy path be visible, with error handling separated.

**Write Your Try-Catch-Finally Statement First.** When the operation can throw, start with the scope the exception will define. This helps you think about what state the program should be in at each catch.

**Use Unchecked Exceptions.** (Java-specific.) Checked exceptions ripple change up the call stack and force intermediate callers to declare exceptions from code they don't care about. Modern consensus sides with the book on this one — even the Java community has largely moved away from checked exceptions.

**Provide Context with Exceptions.** Every thrown exception should carry enough information to identify the source and kind of failure. Include the operation that failed; stack traces alone don't convey intent.

**Define Exception Classes in Terms of a Caller's Needs.** When designing exceptions, the most important question is: how will they be caught? If every caller treats three exception types the same way, you have one exception type, not three. Wrapping third-party APIs in a small adapter with a unified exception type is often the right move.

**Define the Normal Flow.** Business logic shouldn't be littered with exception handling for perfectly normal cases. The *Special Case Pattern*: when "no meal expenses" leads callers to compute a per diem instead, have the API return a `PerDiemMealExpenses` object whose `getTotal()` returns the per diem. Callers stop needing to handle the "no data" case.

**Don't Return Null.** Every null return forces every caller to check. Miss one check and you get an NPE far from the source. Prefer returning an empty collection, throwing an exception, or using a Special Case / Null Object. (In languages with real option types — Rust's `Option`, Haskell's `Maybe`, Kotlin's nullable types — use the option type and let the compiler enforce handling.)

**Don't Pass Null.** Similar problem in reverse. Passing null into a method that doesn't expect it causes an NPE inside the callee. Avoid, and when you can't, document clearly.

---

## Unit Tests (Ch 9)

**The Three Laws of TDD.**
1. You may not write production code until you have written a failing unit test.
2. You may not write more of a unit test than is sufficient to fail (and not compiling is failing).
3. You may not write more production code than is sufficient to pass the currently failing test.

These lock you into roughly 30-second cycles. Many teams don't strictly follow TDD, but the principle — tests are a first-class artifact, written close to the code they test — stands.

**Keeping Tests Clean.** Dirty tests are worse than no tests because they rot the production code along with themselves. Every tangled test makes future tests harder, until the suite becomes so painful it gets discarded, at which point there's no safety net to catch the newly-introduced bugs.

**Test code must be as clean as production code.** This is not negotiable. Tests enable change; dirty tests prevent change; prevented change = code rot.

**Clean tests are readable, readable, readable.** Three things:

- *Clarity* — what is being tested?
- *Simplicity* — minimum ceremony.
- *Density of expression* — a lot in a little.

**Domain-Specific Testing Language.** Tests don't use the production API directly — they use a testing API that *wraps* the production API. Helper functions like `givenPages(...)`, `whenRequestIsIssued(...)`, `thenResponseShouldBeXML()`. This API is not designed up front; it emerges from refactoring test code that has too much detail.

**Build-Operate-Check.** (Also: Arrange-Act-Assert; Given-When-Then.) Every test has three parts: set up the state, do the operation, verify the result. Structure your tests to make the three parts obvious.

**One Assert per Test.** Aspirational. Tests that end in a single assertion read like a single conclusion. In practice, "one *concept* per test" is the real rule — minimize asserts per concept, and test one concept per function.

**F.I.R.S.T.**

- **Fast.** Slow tests don't run, then they atrophy, then the code rots. Ruthlessly keep them fast.
- **Independent.** Tests don't depend on each other; any test should run alone and in any order. Dependencies cascade failures and hide defects.
- **Repeatable.** Should run in any environment — CI, laptop, offline on a train. Non-repeatable tests become unreliable, then ignored.
- **Self-Validating.** Each test passes or fails by itself. No log-reading, no manual diff, no subjective judgment.
- **Timely.** Tests are written just before the production code that makes them pass. Late tests end up fighting the design.

---

## Classes (Ch 10)

**Class Organization.** Java convention: public static constants → private static variables → private instance variables → public functions → private utilities (placed right after the public function that uses them, following the Stepdown Rule).

**Encapsulation.** Default to private. Loosen only when tests demand it, and even then prefer protected/package-private over public. Encapsulation erosion is a last resort, not a first.

**Classes Should Be Small!** Just as with functions, the first rule is size. But for classes, size is measured in **responsibilities**, not lines.

A class with 5 methods can still be too big, if those 5 methods span 2 unrelated responsibilities.

**The Single Responsibility Principle (SRP).** A class should have one, and only one, reason to change. Operational tests:

- Can you name the class concisely? Vague names (`SuperDashboard`, `Manager`, `Processor`) signal too many responsibilities.
- Can you describe the class in 25 words without using "and," "or," or "but"? If not, it's doing too much.

Identifying responsibilities is usually the best way to find the abstractions hiding in a large class. Extract them.

**Many small classes vs. few large classes.** A system with 100 small focused classes has no more moving parts than a system with 10 large unfocused ones — it just exposes them honestly. The small-class system lets you navigate to exactly what you need; the large-class system forces you to wade through unrelated concerns.

**Cohesion.** Each method of a class should use most of its fields. When cohesion is high, the class is a coherent whole. When some methods use only a subset of fields, those methods plus those fields often want to be their own class.

As you break large functions into smaller ones, you'll find variables that used to be local turn into fields "to avoid passing so many parameters." Watch for that. It often means a new class is trying to be born. Breaking out that class restores cohesion.

**Organizing for Change.** A class has two kinds of clients: the code using it today, and the code that will modify it tomorrow. Clean classes minimize the risk to both. Small focused classes with clear responsibilities are the cheapest to change.

**Isolating from Change.** Depend on abstractions, not concretions (DIP). When external systems (databases, network, third-party APIs) leak into your domain logic, wrap them. Your business code depends on your wrapper; your wrapper depends on the external thing. Change one, the other is insulated.

---

## Emergence (Ch 12) — Four Rules of Simple Design

Kent Beck's rules, in priority order:

1. **Runs all the tests.** If the system isn't testable, nothing else about it matters. Testability forces SRP, dependency injection, minimal coupling — the very things good design demands anyway.
2. **Contains no duplication.** Every duplication is a missed abstraction. Fix it.
3. **Expresses the intent of the programmer.** Names carry most of this weight. Small functions carry more. Standard nomenclature (design pattern names, etc.) carries more. Well-written tests carry a lot.
4. **Minimizes the number of classes and methods.** Applied *last*. Aggressive extraction and abstraction for their own sake produces speculative classes and methods that add complexity without value.

The order matters. Don't minimize classes (rule 4) at the cost of expressiveness (rule 3) or by tolerating duplication (rule 2). When in doubt, follow the higher-priority rule.

---

## One-line summaries

If you forget everything else, remember these:

- **Names reveal intent.**
- **Functions do one thing.**
- **Comments are failures of expression.**
- **Error handling shouldn't obscure logic.**
- **Classes have one reason to change.**
- **Tests are first-class code, held to F.I.R.S.T.**
- **Leave the campground cleaner than you found it.**
