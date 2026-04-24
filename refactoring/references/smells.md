# Code Smells

The 24 smells from *Refactoring* 2nd ed., Ch. 3 (Fowler & Beck). Each smell names a structural signal that *invites* refactoring — not a rule violation, and not a moral failing. Use them as diagnostic vocabulary: "this is `Shotgun Surgery`" is a far more useful statement than "this is messy."

Against each smell are the named refactorings the book prescribes. See `refactorings.md` for what each one does.

---

## 1. Mysterious Name

A name that doesn't reveal intent. Single-letter variables, abbreviations, generic names like `data`, `handler`, `process`.

Fix: **Rename Variable**, **Rename Field**, **Change Function Declaration** (to rename functions). If you can't come up with a good name, that's itself a signal — the thing being named probably has no coherent purpose and wants `Extract Function` / `Extract Class` / `Split Variable` first.

---

## 2. Duplicated Code

The same (or near-same) expression in two or more places.

Fix: **Extract Function** and call it from both places. If the duplication is in sibling classes, **Pull Up Method**. If the duplicated fragments are similar but not identical, use **Extract Function** on the similar parts and **Slide Statements** / **Form Template Method** to align them.

---

## 3. Long Function

Functions longer than a few lines tend to hide intent. Size is not the real issue — the semantic distance between *what* and *how* is.

Fix: **Extract Function**, aggressively, whenever a block needs a comment to explain it. Use the comment as the candidate name. Handle supporting problems: **Replace Temp with Query** (removes temp variables that block extraction), **Introduce Parameter Object** and **Preserve Whole Object** (shrinks parameter lists), **Decompose Conditional** (extracts branches into named functions), **Split Loop** (a loop doing two things becomes two loops, each extractable), **Replace Function with Command** (when neither function nor method is enough).

Heuristic: any function where you feel the urge to add a block comment contains a candidate for extraction.

---

## 4. Long Parameter List

More than 3–4 parameters, especially when some cluster together.

Fix: **Replace Parameter with Query** (if one parameter can be derived from another or from the receiver). **Preserve Whole Object** (pass the object, not a dozen fields from it). **Introduce Parameter Object** (cluster related parameters into a type). **Remove Flag Argument** (split one function with a boolean into two functions). **Combine Functions into Class** (when multiple functions share the same parameters, they probably want to be methods on a shared object).

---

## 5. Global Data

Any state visible and mutable from anywhere: globals, class variables, singletons. Bugs that emerge from global data are hard to track because any code can touch it.

Fix: **Encapsulate Variable** — wrap access in functions so you can see where it's read and written. Then narrow scope: move the variable into a module, class, or closure. Immutable globals are tolerable; mutable ones are not.

---

## 6. Mutable Data

Changes to shared data produce spooky bugs. Immutability is a design choice that dramatically reduces this class of bug.

Fix: **Encapsulate Variable** to funnel updates through narrow APIs. **Split Variable** if one variable is being reassigned for different purposes. **Slide Statements** + **Extract Function** to separate pure logic from mutation. **Separate Query from Modifier** on APIs that both return and mutate. **Remove Setting Method** when a field shouldn't be changed after construction. **Replace Derived Variable with Query** to eliminate state that's computable from other state. **Change Reference to Value** for small structured data that's easier to replace than mutate.

---

## 7. Divergent Change

One module changes for multiple unrelated reasons. You edit it when the database changes, and also when the financial rules change, and also when the UI changes.

Fix: Separate the contexts. **Split Phase** if the concerns form a sequence (e.g., fetch then transform). **Move Function** to push each concern into its own module. **Extract Class** to formalize the split if the module is a class.

---

## 8. Shotgun Surgery

The inverse of Divergent Change. One logical change forces edits across many modules. Easy to miss a spot, hard to do consistently.

Fix: **Move Function** and **Move Field** to consolidate the scattered pieces into one place. **Combine Functions into Class** for functions that operate on the same data. **Combine Functions into Transform** for read-only enrichment flows. As a tactic: it's fine to **Inline Function** / **Inline Class** first to gather the scattered code into one blob, then re-extract it into better-shaped pieces — creating a temporary big thing on the way to better small things is legitimate.

---

## 9. Feature Envy

A function in module A spends most of its time calling methods on an object from module B. The function wants to be *in* B.

Fix: **Move Function** to its preferred home. If only part of the function is envious, **Extract Function** on the envious part, then **Move Function**. Heuristic: the function belongs with the data it uses the most. Exceptions exist (Strategy, Visitor) but they're intentional design choices, not accidents.

---

## 10. Data Clumps

The same two, three, or four variables travel together everywhere — as fields in several classes, as parameters in many method signatures.

Fix: **Extract Class** for the clump's fields. **Introduce Parameter Object** or **Preserve Whole Object** for the parameter lists. Test: if you deleted one of the clumped values, would the others still make sense? If not, they're a latent object.

---

## 11. Primitive Obsession

Using raw primitives — strings, ints, floats — where a domain type would do. Phone numbers as strings, money as decimals without currency, coordinates as unrelated `x`/`y`, ranges as `(min, max)` pairs scattered everywhere.

Fix: **Replace Primitive with Object** to create a domain type. If the primitive is a type code controlling conditional behavior, **Replace Type Code with Subclasses** followed by **Replace Conditional with Polymorphism**. For groups of related primitives, **Extract Class** + **Introduce Parameter Object**.

---

## 12. Repeated Switches

The same switch (or `if`/`else if` chain) on the same type code appears in multiple places. Adding a new case means editing every copy.

Fix: **Replace Conditional with Polymorphism** is the canonical answer, but it's no longer automatic dogma — a single well-contained switch is often fine. The smell is *repetition* of the switch across the code, not the switch itself.

---

## 13. Loops

Not a strong smell in itself, but first-class collection operations (`map`, `filter`, `reduce`, `flatMap`) often communicate intent better than a loop with accumulators.

Fix: **Replace Loop with Pipeline**. **Split Loop** first if the loop does two things.

---

## 14. Lazy Element

A class, function, or module that doesn't do enough to justify its existence. A one-line function that's just a pass-through. A class with one trivial method.

Fix: **Inline Function**, **Inline Class**. Remove the unnecessary layer. (Exception: abstractions that exist for future extension points may look lazy now but be paying for themselves later — use judgment and the Rule of Three.)

---

## 15. Speculative Generality

"We might need this someday" code. Abstract classes with one subclass. Unused parameters. Extension hooks with no extenders. Flexibility mechanisms for changes that never materialized.

Fix: **Collapse Hierarchy** (fold abstract into concrete when there's only one), **Inline Function**, **Change Function Declaration** (to drop unused parameters), **Remove Dead Code**. If it's genuinely only used by tests, that's a smell too.

---

## 16. Temporary Field

A field on a class that's only populated in certain circumstances. Readers have to know "this field is only set when X."

Fix: **Extract Class** for the temporary fields and the logic that uses them — that subset of behavior wants to be its own thing. **Introduce Special Case** for "the object is in state X, which means these fields are null" can also fit.

---

## 17. Message Chains

`a.getB().getC().getD().doSomething()`. The client is navigating through a web of objects and is now coupled to the whole chain.

Fix: **Hide Delegate** — have `a` expose `doSomethingOnDeeplyNestedD()` directly. Consider **Extract Function** on the chain and **Move Function** to push it closer to the data.

---

## 18. Middle Man

An object whose methods are almost entirely pass-throughs to another object. You suspect most "delegation" is just noise.

Fix: **Remove Middle Man** — talk directly to the delegate. **Inline Function** to inline the trivial pass-throughs. If the middle man is a class, **Replace Superclass with Delegate** or **Replace Subclass with Delegate** may apply. Opposite of Message Chains — both are about getting the level of indirection right.

---

## 19. Insider Trading

Modules that know too much about each other's internals. Includes subclasses knowing too much about parent internals.

Fix: **Move Function** and **Move Field** to reduce cross-module references. **Hide Delegate** to route through an intermediary. For inheritance-based insider trading: **Replace Subclass with Delegate** or **Replace Superclass with Delegate**.

---

## 20. Large Class

A class with too many fields or too many methods. Usually hiding multiple unrelated responsibilities.

Fix: **Extract Class** for cohesive subsets of fields (look for common prefixes/suffixes, or fields that are used together). **Extract Superclass** or **Replace Type Code with Subclasses** when the subsets align with inheritance. Look at clients — if different clients use different subsets, each subset is a candidate class.

---

## 21. Alternative Classes with Different Interfaces

Two classes that do similar things but with different method signatures. You can't substitute one for the other.

Fix: **Change Function Declaration** to align names and signatures. **Move Function** to move behavior until the protocols match. If this creates duplication, **Extract Superclass**.

---

## 22. Data Class

A class with only fields, getters, and setters. No behavior of its own.

Fix: **Encapsulate Record** if fields are public. **Remove Setting Method** on fields that should be immutable post-construction. Look at where the data is used and **Move Function** to pull behavior *into* the class. Exception: immutable result records from a `Split Phase` transform are supposed to be dumb data — that's legitimate.

---

## 23. Refused Bequest

A subclass inherits methods or data it doesn't want.

Fix: If just data/methods are refused and it's not confusing, leave it — most refused bequests are mild. If the subclass refuses the *interface* of its superclass, the hierarchy is wrong: **Replace Subclass with Delegate** or **Replace Superclass with Delegate**. Traditional alternative: **Push Down Method** / **Push Down Field** to move unused items to a sibling class.

---

## 24. Comments

Comments are not always a smell — they're often fine. But comments used as deodorant for bad code are. If you had to write a comment to explain what a chunk of code does, the chunk probably wants to be a function.

Fix: **Extract Function** using the comment as the name. **Change Function Declaration** to rename if the existing function name is lying. **Introduce Assertion** if the comment is documenting a precondition. Good uses remain: why (not what), known caveats, references to tickets or standards, and explicit "I don't know how to handle this yet" markers.
