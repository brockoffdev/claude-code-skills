# Named Refactorings

The vocabulary of refactoring. Each entry gives the name, its inverse (if meaningful), a brief motivation, and concise mechanics. Mechanics are *safety checklists*, not tutorials — the whole point is that each refactoring is a small, defined, behavior-preserving transformation you can perform mechanically and verify with tests.

Organized by chapter of the book. The set here is the most frequently used subset; consult Fowler's catalog directly for the full list and worked examples.

---

## Composing Methods (Ch. 6)

### Extract Function (inverse: Inline Function)
A fragment of code → a named function.

**Motivation.** Close the semantic distance between *what* the code does and *how*. If you need a comment to explain a block, extract it and use the comment as the name. Works at any size, including one-liners, if the name adds clarity.

**Mechanics.**
1. Create the new function; name it by purpose, not implementation.
2. Copy the fragment into it.
3. Scan for local variables the fragment references; pass them as parameters.
4. For variables assigned inside, either return a value or treat the extracted code as a query returning all assigned values.
5. Replace the fragment with a call. Compile. Test.
6. Look for other places that could call the new function.

### Inline Function (inverse: Extract Function)
A trivial or misleading function → its body inlined at call sites.

**Motivation.** Indirection that doesn't pay for itself. Pass-through functions, one-liners that don't clarify, wrappers that are harder to read than the call they contain.

**Mechanics.** Check no polymorphism (no override will be lost). Find every caller. Replace each call with the body. Test after each replacement. Delete the function.

### Extract Variable (inverse: Inline Variable)
A complex expression → a well-named local.

**Motivation.** Name an intermediate value that currently has no name. Makes debugging and later extraction easier.

**Mechanics.** Ensure the expression has no side effects. Declare a new immutable variable set to the expression. Replace the expression with the variable. Test.

### Inline Variable (inverse: Extract Variable)
A local name that doesn't add meaning → the expression inlined.

**Motivation.** The variable name is redundant with or less expressive than the expression itself.

**Mechanics.** Confirm the RHS has no side effects and isn't reassigned. Replace each reference with the RHS. Remove the declaration. Test.

### Change Function Declaration (also: Rename Function, Add Parameter, Remove Parameter)
Change a function's name or signature.

**Motivation.** Names are how we navigate the codebase. Signatures express contracts. Both drift as understanding improves.

**Mechanics (simple case).** Use the IDE/LSP tool when available — it operates on the AST and is safe. Manual: change declaration and all callers in one edit. Test.

**Mechanics (migration case, when callers can't all be changed at once — e.g., published APIs).** Apply the body change first if needed. Extract the body into a new function with the new signature. Have the old signature delegate to the new one. Migrate callers one by one. Remove the old when no callers remain (or leave as a deprecated shim).

### Encapsulate Variable
A bare variable → variable behind getter/setter (or accessor functions).

**Motivation.** Prerequisite to almost any data-centric refactoring. You can't rename a variable safely, move it, or restrict mutation if every reference reads and writes it directly. Encapsulation gives you a single place to intercept.

**Mechanics.** Create accessor functions. Replace references with accessor calls. For mutation-sensitive data, restrict writes (make the variable private, throw on unwanted writes). Test.

### Rename Variable
Replace the name of a variable.

**Motivation.** A bad name now is a bug magnet later. Rename as soon as you have a better name.

**Mechanics.** If the variable's scope is narrow, rename directly. If wide, `Encapsulate Variable` first so only the accessors need updating. Use the IDE tool when available.

### Introduce Parameter Object
Several parameters that travel together → a single object parameter.

**Motivation.** Data clumps signal a missing abstraction. The new object often attracts behavior over time, becoming a genuine domain concept.

**Mechanics.** Create the new class (or record, or struct). `Change Function Declaration` to add the new parameter. Migrate callers to construct the object and pass it. Remove the old parameters one at a time, replacing `param` with `obj.param` in the body. Test after each step.

### Combine Functions into Class
Several functions that share data and parameters → a class with that data as fields and those functions as methods.

**Motivation.** Repeated parameter passing of the same arguments is the functional-to-OO signal. The class makes the relationship explicit and opens the door to richer behavior.

**Mechanics.** Create the class. `Encapsulate Record` for the shared data if not already. `Move Function` each related function into the class. Drop parameters that are now fields. Test.

### Combine Functions into Transform
Several functions that derive values from the same data → one function that produces an enriched record.

**Motivation.** Alternative to `Combine Functions into Class` when the data is read-only. Keeps derivations in one place to avoid inconsistency.

**Mechanics.** Create a transform function that takes the input, deep-copies it to avoid aliasing, adds derived fields, returns the enriched record. Move each derivation into the transform. Update callers to use the transform output.

### Split Phase
One function doing two sequential things → two functions with an intermediate data structure between them.

**Motivation.** Mixing unrelated concerns (e.g., calculation and formatting, parsing and interpretation) in one function breeds duplication when either concern needs variation. The intermediate data structure makes each phase independently changeable.

**Mechanics.** Extract the second phase into its own function, passing it an intermediate data structure. Populate that structure from the first phase. Move fields of the intermediate structure one at a time out of the second phase's parameter list. Test after each.

---

## Encapsulation (Ch. 7)

### Encapsulate Record
A public record (fields accessible directly) → class with accessors.

**Motivation.** The ability to intercept reads and writes; the flexibility to change internal representation; the ability to add derived fields transparently.

**Mechanics.** Wrap the record. Replace direct field access with accessors. Where possible return copies (not references) to nested mutable data.

### Encapsulate Collection
A collection field exposed directly → collection field with controlled mutation methods.

**Motivation.** Direct access to a collection lets callers bypass invariants. Even returning the collection by reference is dangerous if callers mutate it.

**Mechanics.** Provide `add`, `remove`, etc., methods on the owning class. Have the getter return an unmodifiable view or a copy. Route all mutations through the class methods.

### Replace Primitive with Object
A primitive that needs more behavior → a small domain class.

**Motivation.** Money, phone numbers, identifiers, units — all want their own type with validation, formatting, and operations.

**Mechanics.** `Encapsulate Variable` on the primitive field. Create a small class wrapping the primitive. Change the setter to wrap; change the getter to unwrap. Migrate callers to use the class directly.

### Replace Temp with Query
A temporary variable holding a computed value → a query function that computes it.

**Motivation.** Temps clutter extraction. A query can be called from anywhere; a temp can't. Especially valuable before `Extract Function` on a block that refers to the temp.

**Mechanics.** Confirm the temp is computed once and depends only on stable inputs. Extract its computation into a query function. Replace references to the temp with calls to the query. Remove the temp.

### Extract Class
A class doing too much → two classes.

**Motivation.** A class accumulates responsibilities over time. When you can name two distinct concerns, they want to be two classes.

**Mechanics.** Create the new class. Move fields, one at a time, using `Move Field`. Move related methods with `Move Function`. Decide whether the old class holds a reference to the new one, or vice versa, or both. Decide whether to expose the new class to clients or keep it hidden inside the old one.

### Inline Class
A class doing too little → fold into another class.

**Motivation.** The inverse of `Extract Class`. Sometimes the extracted class never grew into its own and just adds noise.

**Mechanics.** Move each method from the shrinking class to its host using `Move Function`. Move each field. Delete the original class.

### Hide Delegate
Client calls `a.getB().getC()` → client calls `a.getC()`.

**Motivation.** Demeter's Law. The client should not need to know that `a` has a `b` that has a `c`.

**Mechanics.** On `a`, add a delegating method that returns what the client needs. Replace the chain with the delegating call. If no client needs `getB` directly, hide it.

### Remove Middle Man
Class whose methods are mostly pass-throughs → clients talk directly to the delegate.

**Motivation.** Indirection you're not paying for. Opposite of `Hide Delegate`; pick the right level.

**Mechanics.** On clients, replace `a.doSomething()` (which forwards to `b.doSomething()`) with `a.getB().doSomething()`. Remove pass-through methods from `a` when no longer used.

### Substitute Algorithm
One algorithm → a clearer or better algorithm.

**Motivation.** Sometimes the right move isn't a series of small refactorings — it's replacing a block wholesale with a simpler implementation. Only safe when tests cover the behavior thoroughly.

**Mechanics.** Ensure tests exercise the algorithm well. Replace the body. Test.

---

## Moving Features (Ch. 8)

### Move Function
A function in the wrong module → the right one.

**Motivation.** Encapsulation and cohesion. Functions want to be near the data they act on.

**Mechanics.** Examine every reference the function makes (variables, other functions, types); decide how each will be reached from the new home. Copy the function to the new location; adjust to compile. Replace old-site references with delegating calls to the new location. Inline or delete the old function when all references are migrated. Test at each step.

### Move Field
A field in the wrong class → the right one.

**Motivation.** Data attracts behavior; badly-placed data produces feature envy.

**Mechanics.** Ensure the source field is encapsulated. Add the field to the target. Have accessors on the source forward to the target. Migrate references. Remove the field from the source when clean.

### Move Statements into Function / Move Statements to Callers
A statement currently outside → inside a called function (or vice versa).

**Motivation.** Duplicated pre/post logic around every call → move it into the function. Or: callers need variation that the function doesn't support → move the rigid part out.

**Mechanics.** Identify the statement that's always (or never) paired with the call. Move it. Verify each caller still gets correct behavior.

### Slide Statements
Reorder statements in a function so related things are adjacent.

**Motivation.** Related code grouped together is a prerequisite to extracting it. Slide first, then extract.

**Mechanics.** Confirm the statements can be reordered without changing behavior (check dependencies carefully). Move them. Test.

### Split Loop
One loop doing N things → N loops each doing one.

**Motivation.** A loop that sums *and* finds a max *and* filters is doing too much to extract. Splitting it makes each pass extractable.

**Mechanics.** Duplicate the loop. Eliminate side effects so each copy does one thing. `Extract Function` on each.

### Replace Loop with Pipeline
A for-loop with accumulators → map/filter/reduce style.

**Motivation.** Pipelines often read more declaratively — you see the steps of the transformation.

**Mechanics.** Start with a loop over the source. Incrementally move its operations into chained collection methods. Test after each.

### Remove Dead Code
Unreachable or unused code → gone.

**Motivation.** Dead code tricks readers into spending mental effort on something that doesn't run. Version control remembers it if you ever need it back.

**Mechanics.** Confirm it's unreachable (look for dynamic dispatch, reflection, script callers, external entrypoints). Delete. Test.

### Split Variable
One variable reassigned for multiple purposes → multiple variables.

**Motivation.** Variables with multiple meanings are hard to name well. Each role gets its own name and the code reads clearer.

**Mechanics.** Rename the first usage and declare a new variable for it. Replace references. Repeat for each subsequent purpose. Make each single-assignment where possible.

### Rename Field
Rename a class field.

**Motivation.** Same as variable rename: as understanding of the domain improves, so should names.

**Mechanics.** If the class is encapsulated, rename the field and update internal references. Update accessor names via `Change Function Declaration`.

### Replace Derived Variable with Query
A mutable field that shadows a computable value → a query that computes it.

**Motivation.** Storing derived state that other state already determines invites inconsistency. Every update site has to remember to update the derivation.

**Mechanics.** Identify the source of truth. Write a query that derives the value. Replace reads of the stored field with calls to the query. Remove the field.

### Change Reference to Value / Change Value to Reference
Small mutable object passed by reference → immutable value type (or vice versa).

**Motivation.** Value semantics are simpler when the object is small and equality is structural. Reference semantics are right when there's conceptually one instance shared across holders.

**Mechanics.** For ref→value: ensure no callers depend on shared mutation; make the class immutable; replace equality with structural equality. For value→ref: introduce a repository or factory that returns the single instance; update callers to go through it.

---

## Simplifying Conditional Logic (Ch. 10)

### Decompose Conditional
A complex `if`/`else` → extracted functions for condition and branches.

**Motivation.** The conditions and branches of a complex conditional are usually more comprehensible as named functions than as inline logic.

**Mechanics.** `Extract Function` on the condition. `Extract Function` on each branch. Name them by intent.

### Consolidate Conditional Expression
Several conditionals with the same result → one combined conditional.

**Motivation.** If the branches all do the same thing, the separate tests obscure the fact that there is really one condition.

**Mechanics.** Combine the tests with `&&` / `||` as appropriate. Consider extracting the combined condition to a named function.

### Replace Nested Conditional with Guard Clauses
Nested `if`/`else` → early returns for special cases.

**Motivation.** Guard clauses say "this case is weird, here's what we do and we're done." The main path of the function reads straight through.

**Mechanics.** Identify the exceptional cases. Hoist each into an early-return `if` at the top. Leave the main path at the bottom with no remaining nesting.

### Replace Conditional with Polymorphism
Repeated type-based switch → dispatch through a class hierarchy.

**Motivation.** When the same switch-on-type appears in multiple places, polymorphism collapses them into one place per subtype. This is a strong tool — not a default.

**Mechanics.** `Extract Class` or create a small hierarchy. Create a factory that returns the right subclass for the type code. Move each switch branch into an overridden method on the corresponding subclass. Replace the switch site with a polymorphic call.

### Introduce Special Case (also: Null Object)
Repeated checks for the same special case → a subclass (or sentinel value) that handles it.

**Motivation.** `if (customer == null) { ... defaults ...}` scattered everywhere is repetitive. A `NullCustomer` with the default behavior built in makes callers unconditional.

**Mechanics.** Create a subclass for the special case (or an instance of the class that represents "nothing"). Implement its methods to return the sensible defaults. Update the factory to return it when appropriate. Remove the scattered checks.

### Introduce Assertion
An unwritten assumption → an explicit assertion.

**Motivation.** Some functions only work when certain conditions hold. Writing those down as assertions documents the contract and catches violations.

**Mechanics.** Add an `assert` at the top of the function for each precondition. Do not use assertions for things that happen at runtime from user input — those need real error handling, not assertions.

---

## Refactoring APIs (Ch. 11)

### Separate Query from Modifier
A function that both returns a value and has side effects → two functions.

**Motivation.** Separating queries from modifiers lets you call the query anywhere without fear. "Command-query separation" (Bertrand Meyer).

**Mechanics.** Copy the function. In the copy, remove side effects (make it pure query). In the original, remove the return value (make it pure modifier). Update callers to call one or both as needed.

### Parameterize Function
Several nearly-identical functions differing by a literal value → one function with a parameter.

**Motivation.** Duplication. One function with a parameter says what's actually varying.

**Mechanics.** Pick one of the similar functions. Replace the literal with a parameter. Update its callers to pass the old literal. Migrate other similar functions to call the parameterized one with their respective values. Delete them.

### Remove Flag Argument
A function with a boolean argument that selects between behaviors → two separate functions.

**Motivation.** `setDimension("height", 10)` hides what the call does. `setHeight(10)` is obvious.

**Mechanics.** For each value of the flag, create a dedicated function with that path baked in. Update callers. Remove the original.

### Preserve Whole Object
Passing several fields from the same object → pass the object.

**Motivation.** The function can't use other fields or ask the object questions when you've pre-chewed the data. Passing the whole object respects its encapsulation.

**Mechanics.** Add the object as a parameter. Replace uses of individual fields with `obj.field`. Drop the old parameters.

### Replace Parameter with Query
A parameter whose value can be determined by the function itself → drop the parameter and compute inside.

**Motivation.** Simplifies the caller's job. But only safe when the function can compute the value as well as (or better than) every caller.

**Mechanics.** Ensure the computation is feasible and has no side effects. Replace parameter references with the computation. Update callers to stop passing the value.

### Remove Setting Method
A setter for a field that shouldn't change after construction → remove it.

**Motivation.** Immutability by default. A setter's existence suggests mutation is expected; removing it makes mistakes visible.

**Mechanics.** Ensure the field is only set in the constructor. Delete the setter. Migrate callers that used the setter to set the field during construction instead.

### Replace Constructor with Factory Function
A constructor with awkward constraints (can't return subtypes, can't fail gracefully, can't be named) → a factory function.

**Motivation.** Factories can return subclasses, cached instances, or null; they can have descriptive names like `Employee.hire(...)` instead of the class name.

**Mechanics.** Introduce the factory. Move construction through it. Hide or remove the public constructor.

### Replace Function with Command
A function with too much complex local state → a command object.

**Motivation.** When you can't easily extract pieces of a function because everything references everything else, promote the whole function to an object so those locals become fields. Much easier to refactor once that's done.

**Mechanics.** Create a class with the function's body as an `execute` method. Turn each local into a field. Now you can extract methods freely because everything is `this.x`. Commonly a stepping stone, not a final form.

### Replace Command with Function
A command object that's overkill for what it does → a plain function.

**Motivation.** The inverse. Once the refactoring that needed the command is done, the command object may be excess structure.

---

## Dealing with Inheritance (Ch. 12)

### Pull Up Method
A method duplicated in siblings → one method on the parent.

**Motivation.** Remove duplication across a hierarchy.

**Mechanics.** Confirm the methods are effectively the same. Align signatures with `Change Function Declaration` if needed. Copy one to the parent. Remove from each subclass one at a time. Test after each.

### Pull Up Field
A field duplicated across siblings → one field on the parent.

**Mechanics.** Align types and names. Add to the parent. Remove from children.

### Pull Up Constructor Body
Sibling constructors that do nearly the same thing → shared body on the parent.

**Mechanics.** Extract the common body into the parent constructor. Call `super(...)` from each subclass constructor with the shared pieces.

### Push Down Method / Push Down Field
A member on the parent that only some children use → move it down to those children.

**Motivation.** A member that the parent declares but not all children should have is a sign the hierarchy is wrong or the member is misplaced.

**Mechanics.** Move the member to the relevant subclasses. Remove from the parent. Adjust call sites.

### Replace Type Code with Subclasses
A field whose value drives different behavior → a subclass per value.

**Motivation.** When the type code drives real behavioral differences, subclasses let the language handle dispatch. Enables `Replace Conditional with Polymorphism`.

**Mechanics.** Encapsulate the type code. Create a subclass per value. Use a factory to instantiate the right subclass based on the code. Migrate conditional behavior into overrides. Remove the original field.

### Remove Subclass
A subclass that no longer justifies itself → collapse into parent.

**Motivation.** The subclass's differences have evaporated or can be handled by a field or a flag.

**Mechanics.** Replace instances of the subclass with instances of the parent (or a sibling). Migrate remaining differences into the parent as needed. Delete the subclass.

### Extract Superclass
Two classes with shared features → extract a common parent.

**Motivation.** An alternative to `Extract Class` (composition) when the relationship is genuinely is-a.

**Mechanics.** Create the empty superclass. `Pull Up Field` / `Pull Up Method` each shared member. Test after each.

### Collapse Hierarchy
A parent and child that are barely distinct → fold into one class.

**Motivation.** Inheritance that isn't pulling its weight is drag.

**Mechanics.** Pick which class to keep. `Pull Up` or `Push Down` every member into that class. Delete the other.

### Replace Subclass with Delegate
A subclass whose differences are inheritance-inappropriate → delegation to a helper object.

**Motivation.** Inheritance is a strong coupling; often delegation gives you the same flexibility with less tangling. Particularly useful when multiple axes of variation collide (you can't inherit from two dimensions at once).

**Mechanics.** Introduce a delegate interface. Create delegate classes corresponding to the former subclasses. Have the base class hold a delegate and forward the varying methods. Remove the subclasses.

### Replace Superclass with Delegate
A parent whose subclasses don't really want everything from it → make the parent-child relationship delegation instead.

**Motivation.** "Is-a" was wrong; "has-a" was the honest relationship. E.g., a `Stack` extending `List` is a classic error — stack has-a list, not is-a list.

**Mechanics.** Introduce a field for the former superclass. Forward the methods you still want. Drop the inheritance. Remove forwarding for methods the subclass never really wanted.
