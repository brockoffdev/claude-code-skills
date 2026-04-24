# Practices

The substantive content from *The Pragmatic Programmer*, organized by theme. Where `tips.md` gives the numbered aphorisms, this file gives the reasoning and mechanics behind the tips that get applied most often.

Organized by topic. Read the relevant section when a habit needs more than a one-line reminder.

---

## Software Entropy and Broken Windows

**The core claim.** Software rots. Not because of physics but because of culture. When disorder accumulates and nobody fixes it, people stop caring, and the rate of decay accelerates. The same pattern shows up in buildings ("broken window theory"): one unrepaired broken window signals abandonment, and within weeks the building is stripped.

**The practical consequence.** When you see a problem in the code — a bad design, a disabled test, a TODO that's been there for six months — the worst thing is to walk past it. Walking past *teaches* the team that quality isn't a priority here, and more broken windows follow.

**What to do.**
- Fix it if it's small. The five-minute fix is usually worth it.
- If you can't fix it now, *visibly* board it up — a comment, a ticket with a link, a failing test named "KNOWN_BUG_123," anything that makes the neglect intentional and attributed rather than just decay.
- Don't let "the rest of this code is already a mess" become an excuse to add to the mess. That's the moment the rot accelerates.
- If a new feature is going into an area that's already broken, fix the windows first. Otherwise your feature inherits the mess.

---

## Good-Enough Software

**The core claim.** Perfect software is not a coherent goal. The real goal is software good enough for its users, its maintainers, and its own future self — delivered on a schedule that matters. "Good enough" is not a euphemism for "shoddy"; it's an explicit acknowledgment that quality is a dimension with costs, and the right amount of quality is the amount that serves the users.

**The habits that follow.**
- **Make quality a requirements discussion.** How reliable does this need to be? How fast? How fault-tolerant? These are specifiable with the user, not decided unilaterally by the developer.
- **Involve users in the quality tradeoff.** Users will often accept earlier, less polished versions if asked — and getting real feedback on a rough version beats guessing for three more months.
- **Know when to stop polishing.** Past a certain point, additional effort produces diminishing returns. Shipping teaches you things that more polishing can't.
- **Don't ship garbage and call it good-enough.** The principle isn't an excuse for laziness — it's a constraint against perfectionism.

---

## Your Knowledge Portfolio

**The core claim.** A programmer's knowledge is an investment portfolio. Like any portfolio, it depreciates, has to be diversified, requires regular investment, and needs rebalancing over time.

**The four rules (borrowed from investing).**
- **Serious investors invest regularly — as a habit.** A small amount of study every week compounds. Occasional binges don't.
- **Diversification is the key to long-term success.** One language, one stack, one domain makes you fragile. Learn a new language every year or two. Learn paradigms you don't already use (functional if you're OO, systems if you're high-level). Learn adjacent domains.
- **Balance conservative and speculative investments.** Core skills (your primary language, core tools, fundamentals) are the conservative portion. Speculative investments (emerging tech, new frameworks) might not pay off — but some will.
- **Rebalance periodically.** Review what you know versus what's becoming important in the industry and in your work. Retire skills that no longer pay off; invest in ones that are rising.

**Beyond the investment metaphor.**
- **Read outside your immediate needs.** The book that changes your approach next year is rarely the one you'd pick to solve this week's problem.
- **Critically analyze what you read.** Authors have biases, contexts, and incentives. Your situation may not match theirs.
- **Ask why, not just how.** Understanding why an approach works lets you generalize; understanding only how lets you copy-paste.

---

## DRY — The Evils of Duplication

**The principle.** Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.

**Why this matters so much.** When the same knowledge exists in two places, the two will drift. When they drift, the system becomes inconsistent, and users/tests/code start hitting contradictions. It is not a question of whether you'll forget to update both — it's a question of when.

**The four kinds of duplication to watch for.**
- **Imposed duplication.** The language, tooling, or architecture seems to force it — header files duplicating declarations, schemas duplicated between code and database, protocol definitions duplicated between client and server. The answer is usually code generation: generate both copies from a single source of truth.
- **Inadvertent duplication.** Two people wrote the same thing without realizing it existed already. Or one person wrote the same logic twice because it wasn't obvious the first version existed. Countered by visibility — know the codebase, search before writing.
- **Impatient duplication.** Copy-paste because it was faster than extracting a proper abstraction. The canonical mistake: two near-identical blocks that will diverge over the next six months into a bug.
- **Interdeveloper duplication.** Multiple people or teams building the same component independently because nobody knew the others were. Countered by communication, a shared library culture, and code review.

**Beyond code.** DRY applies equally to specifications, documentation, build scripts, and test fixtures. The same knowledge captured in code *and* in a comment *and* in a spec *and* in a test expectation will diverge; when it does, nobody will be sure which is right. Pick one as authoritative and derive the others from it where possible.

**A subtlety.** DRY is about knowledge, not about text. Two functions that happen to have similar-looking bodies may encode unrelated knowledge; merging them creates coupling that didn't exist. Conversely, two very different-looking pieces of code may encode the same knowledge; they should be unified despite the surface differences.

---

## Orthogonality

**The principle.** Two things are orthogonal if changes to one don't affect the other. In software, this means: database code is orthogonal to the UI; business logic is orthogonal to persistence; business rule X is orthogonal to business rule Y.

**Why it matters.**
- **Localized changes.** A change confined to one module means smaller diffs, easier review, lower risk of unintended effects, less regression surface.
- **Easier testing.** Components that are orthogonal can be tested in isolation. Components that are tangled require integration tests for even trivial verification.
- **Better reuse.** Components with a focused responsibility can be composed; components entangled with unrelated concerns can only be used as-is.
- **Reduced risk.** A diseased section stays local. A bug in the payment system doesn't break the user profile page.

**How to test for it.** Take any component in your design. Ask: if the requirements around this one thing change dramatically, how many modules have to change? In an orthogonal design, the answer is ideally one. If the answer is "half the system," you have tangled concerns.

**Common leaks of orthogonality.**
- Depending on incidental properties of external things (customer IDs based on phone numbers → when area codes get reassigned, you're hosed).
- Third-party libraries that force their conventions through your code (exceptions that ripple up, threading models that constrain the caller).
- "Hybrid" classes that are neither full objects with hidden state nor clean data structures, exposing just enough internals to couple everything to their implementation.
- Teams structured so that every change requires three other teams' approval.

**A pragmatic note.** Full orthogonality is unattainable. The goal is to be aware of coupling and to choose where it's acceptable. Tight coupling that delivers a real benefit (e.g., a performance-critical path) is fine if the coupling is understood and deliberate.

---

## Reversibility

**The principle.** Most critical decisions made in a project turn out to be wrong, or become wrong as circumstances change. Design so that reversing decisions is cheap.

**Why flexibility beats correctness.** You will not pick the right database, framework, vendor, architecture, or deployment model on the first try. Your users will change their minds. Your company will standardize on something different. A new library will appear that makes your current one obsolete. The whole question is not *whether* you'll want to change — it's *how expensive* that change will be when it comes.

**Practical techniques.**
- **Wrap external dependencies.** A thin adapter between your code and the vendor library means you can replace the vendor without rewriting the application. The adapter becomes the contract.
- **Configuration over code.** Decisions expressed in config files can be changed without rebuilding; decisions baked into `if` statements require redeployment.
- **Metadata at the edges.** Push varying details (fields, validation rules, message types) into data that the code reads at runtime. The code stays stable as the details churn.
- **Deployment flexibility.** If your application can run as a server and as a standalone, or as a monolith and as microservices, a late-breaking requirements change doesn't wreck the architecture.
- **Abstract service boundaries.** Depend on interfaces, not concrete implementations. Let the specific implementation be swappable.

**Don't take this too far.** Excessive abstraction for hypothetical future changes creates its own costs — more indirection, more cognitive load, more places to change when the change you do need doesn't fit the seams you built. The goal is *reasonable* reversibility, not infinite flexibility.

---

## Tracer Bullets vs Prototypes

**The distinction is sharp and matters.**

**Tracer bullets.** A thin but complete end-to-end version of the real system. It exercises every component, has real error handling, is checked into version control, and is *grown* into the final system. The first tracer might just execute one query through the full stack; over time every new feature fills in more of the skeleton. You keep tracer code.

**Prototypes.** Throwaway experiments to answer specific questions. "Will this algorithm work?" "Will users understand this UI?" "Is this architecture viable at scale?" You build the cheapest possible thing that answers the question, learn the answer, and *discard the code*. Prototype in any language convenient (often not the final one). Skip error handling, skip polish. The deliverable is the knowledge, not the code.

**When to use which.**
- Use **tracer bullets** when you know roughly what the system should do but need to see the pieces integrated before trusting any individual piece. Good for unfamiliar architectures, new team members, demos.
- Use **prototypes** when you have a specific unknown you need to resolve before committing. Good for evaluating libraries, validating algorithms, testing UI concepts.

**The typical mistake.** Keeping a prototype — "it works, we'll just polish it up." The polish never happens, and you're left with code that was written to answer a question, not to be maintained. Either commit to rewriting it properly, or commit to tracer bullets from the start.

---

## Design by Contract

**The principle.** Every function has three kinds of specifications, and making them explicit clarifies thinking and catches bugs.

- **Preconditions.** What must be true for the function to be called. The caller's responsibility. If a precondition is violated, the caller has a bug.
- **Postconditions.** What the function guarantees when it completes. The function's responsibility. If a postcondition is violated, the function has a bug.
- **Invariants.** What's true throughout (class invariants for methods, loop invariants for loops).

**The contract.** If the caller meets all preconditions, the function guarantees all postconditions and invariants. If the caller doesn't meet preconditions, the function is not obligated to do anything useful.

**The guiding posture.** Be strict in what you accept, and promise as little as possible in return. A function that "accepts anything and does the right thing" either quietly does wrong things or contains a mountain of defensive code. A function with a narrow, well-specified input space is easier to implement correctly, easier to test, and easier to reason about.

**How to apply in practice.**
- In languages with a built-in contract system, use it.
- In languages without, write assertions at the top of each function (preconditions) and before each return (postconditions) for the important cases. Even unenforced comments documenting the contract are better than leaving it implicit.
- Don't use preconditions for user input validation. User input validation is ordinary logic; preconditions express conditions the caller is *required* to meet, which is a different thing.

**Liskov substitution.** A subclass may *loosen* preconditions (accept more) and *tighten* postconditions (guarantee more). The reverse — tightening preconditions or loosening postconditions in a subclass — breaks the contract the parent advertised, and callers written against the parent will break.

---

## Pragmatic Paranoia — Defensive Programming

**The stance.** You can't write perfect software. Neither can anyone else whose code you're calling. Inputs will be malformed, resources will be exhausted, assumptions will turn out to be wrong. Code in a way that these realities don't cascade into disaster.

**Crash early.** When an impossible state is detected, stop immediately with a clear message. A program that detects something impossible and limps on will produce output that's subtly wrong — sometimes for hours — and you'll spend days tracing the consequence back to the original deviation. Dead programs tell no lies.

**Use assertions for the genuinely impossible.** When you "know" a value can't be null here, or a list can't be empty, or two pointers can't be equal — assert it. Either the assertion holds (your model is correct, and the assertion documents that for anyone reading the code), or it fires (your model was wrong, which is information you desperately needed). Don't use assertions for things that can happen in production — use proper error handling for those.

**Exceptions are for exceptional cases.** If a condition is part of normal operation (file not found, network timeout, user input invalid), that's not exceptional — that's ordinary flow and should be handled as such. If a condition genuinely prevents the function from completing at all (out of memory, violated invariant, caller broke the contract), that's exceptional. Reserve exceptions for the second kind.

**Finish what you start.** Whoever allocated a resource should free it. Whoever acquired a lock should release it. Whoever opened a file should close it. Don't scatter a resource's lifecycle across unrelated parts of the system; put the acquire and release in the same scope (using the language's best facility: RAII, try-with-resources, `defer`, `with`, etc.).

---

## Decoupling and the Law of Demeter

**The principle.** A module's methods should only call:
- Methods of the module itself
- Methods of its parameters
- Methods of objects it creates
- Methods of its direct components (fields)

And crucially, *not* methods of objects returned from any of the above.

**What this rules out.** The "train wreck": `a.getB().getC().doSomething()`. This line couples the current module to the internals of A, the internals of B, and the existence of a C inside B. When any of those evolves, this line breaks.

**What to do instead.** Ask A for what you actually need. If you need a TimeZone ultimately stored inside A's Location inside A's Recorder, add a `getTimeZone()` method to A. A will delegate internally. The caller doesn't know or care how A gets the time zone — that's A's business.

**Why this works.** Coupling is fundamentally about how many independent changes force how many other changes. A train wreck couples your module to three other modules at once. An `a.getTimeZone()` call couples it to one. When A's internals change, only A has to change; when B's or C's do, A adjusts but the caller doesn't notice.

**The tradeoff.** Following the Law strictly means writing a lot of small delegation methods — wrapper methods on A that just forward to B that just forward to C. Runtime cost, source code volume, occasional friction. In most modern applications this is worth it by a large margin; in performance-critical hot paths, deliberate violations are sometimes justified. The key is that the violations are *deliberate* and *local*.

---

## Programming by Coincidence vs Programming Deliberately

**The failure mode.** Fred writes some code, runs it, it works. He writes more, runs it, still works. Weeks later, it stops working and he can't figure out why. The problem: he never understood *why* any of it worked in the first place. He was relying on coincidences — a particular order of operations, an undocumented library behavior, a specific data shape that happened to appear in tests — and when one of those coincidences shifted, everything collapsed.

**Ways this happens.**
- **Accidents of implementation.** Relying on how a routine currently behaves rather than on its documented behavior. When the routine gets fixed, your code breaks.
- **Accidents of context.** The code assumes a GUI is present, or English, or a literate user, or a network connection — without any of those being actual requirements.
- **Implicit assumptions.** You assume X causes Y because whenever X happens Y follows, without checking whether X actually *causes* Y or they just correlate.
- **Adding code until the bug goes away.** You sprinkle extra calls, retries, or sleeps until the failing case stops failing, without ever understanding what was wrong. The underlying bug is still there; you've just moved its symptom.

**How to program deliberately.**
- **Always know what you're doing.** If you can't explain why the code works, treat that as a bug.
- **Rely only on documented behavior.** If behavior isn't documented, document your assumption in a comment or, better, write a test that pins it down.
- **Test your assumptions.** Don't guess — add an assertion, check in the debugger, write a probe. Real data beats reasoning every time.
- **Proceed from a plan.** Even a five-minute plan on a napkin beats diving in blind. What are you trying to accomplish? What will you check along the way to know if you're on track?
- **Prioritize the hard parts.** Spend effort on the parts that will determine whether the system works at all. Decoration can come later.

**The diagnostic question.** When code works and you're not sure why, or when a fix solves the symptom without addressing a root cause, stop and ask: what am I *relying* on here that I haven't verified?

---

## Debugging Approach

**Mindset first.**
- **Don't panic.** Panic produces flailing. Step back; the real bug is rarely what your first reaction says.
- **Fix the problem, not the blame.** Whose fault it is doesn't affect the fix. Spend no time on it.
- **"That's impossible" is always wrong.** Whatever you're seeing clearly *is* possible, because it's happening. Your job is to find out how.
- **Don't assume — prove.** Real data from a probe or debugger beats reasoning.

**Reproducibility first.**
- A bug you can reproduce with a single command is more than halfway solved. A bug you can't reproduce will keep coming back.
- Invest in finding the shortest reproducer. The process of simplifying often reveals the cause.

**Find the root cause, not just the symptom.**
- The first manifestation is rarely where the bug lives. A variable is wrong; trace back to find where it was set wrong; trace back to find what set *that* wrong.
- Resist the urge to patch just the symptom. A patched symptom usually means the underlying bug will return in a different form.

**Bugs cluster.**
- When you find one bug in a function, test the function exhaustively. You'll probably find more.
- When a pattern of failures emerges (all large inputs fail; all negative numbers crash), the pattern itself is a clue to the cause.

**Rubber ducking.**
- Explaining the problem to someone else — even an inanimate someone else — forces you to articulate assumptions you'd glossed over. The act of explanation frequently surfaces the bug without the listener doing anything.

**Tracing and visualization.**
- Debuggers show current state. Sometimes you need history — what the state was a hundred calls ago. Log statements with consistent formatting give you that history.
- For data structures, draw them. A pencil-and-paper diagram of a linked list in trouble often makes the bug obvious in a way that staring at pointer values never does.

---

## Code That's Easy to Test

**The principle.** Testability is a design property, not a separate concern bolted on at the end. Code that's hard to test is hard to *use* — tests are just the first users. If you can't easily write a test, your module boundaries, dependencies, or assumptions are probably wrong.

**Signs of poor testability.**
- You need a running database, network, or UI to verify any behavior.
- Setup code for a test is longer than the thing being tested.
- Test data has to be massaged into existence through many steps.
- Tests depend on each other's execution order.
- Tests pass or fail nondeterministically.

**Design moves that help.**
- **Dependency injection.** Functions receive their dependencies rather than looking them up. A function that takes a clock parameter can be tested with a fake clock; a function that calls `Clock.now()` internally cannot.
- **Pure functions where possible.** A function with no side effects and deterministic output given its inputs is trivially testable.
- **Narrow interfaces.** A module with a small public API exposes fewer things to test and fewer things that can break.
- **Built-in diagnostics.** Log files in a parseable format. A status endpoint. Whatever lets you inspect the system's state from outside without attaching a debugger.

**A test window into running systems.**
- Production code should be inspectable: well-structured logs, a status endpoint, tracing hooks. The ability to see what the system is doing in production without a debugger pays off the first time you have a production incident.

---

## Testing Approach

**Test early, often, automatically.** Manual testing is input to writing automated tests — not a substitute. A test that isn't automated will eventually stop running; a test that runs automatically on every change catches regressions seconds after they're introduced.

**Tests are definition of done.** A feature with failing tests is not done, regardless of what the demo showed. Coding ain't done till all the tests run.

**State coverage, not just line coverage.**
- Line coverage only says which lines ran, not whether they did the right thing. Tests that exercise code without checking its effects pass coverage and catch nothing.
- State coverage asks: were the meaningful states of the system exercised? Valid inputs. Invalid inputs. Empty. Full. Boundary values just-inside and just-outside. Error paths. Concurrent access.

**Test boundary conditions ruthlessly.** We get the middle right and misjudge the edges. Every input range has a lower bound and an upper bound; test both, plus one-below and one-above if applicable.

**Patterns in failure reveal structure.** When 14 of 15 tests pass and the one that fails always involves negative numbers — the pattern itself is a clue. Present tests in a way that makes patterns visible.

**Regression tests for every bug.** When a bug escapes to production, writing a test that would have caught it serves two purposes: it prevents reintroduction, and it slowly builds a history of every mistake the system has made. Find bugs once.

**Test your tests.** Occasionally introduce a deliberate bug and confirm the tests catch it. If they don't, the tests are weaker than they appear.

---

## Estimating

**Estimating is a skill.** It's practiced; it improves with feedback. The way to get better is to estimate, record actuals, and compare.

**Express confidence in the units you use.**
- Days implies high confidence and near-term work.
- Weeks implies medium confidence.
- Months implies low confidence — the estimate is really about scope, not time.
- Quarters implies speculative — you're estimating something you don't understand yet.

Choosing the unit honestly signals how much to trust the number.

**Don't refuse to estimate.** "I don't know" is almost never the right answer. A rough estimate with an honest confidence interval is useful; silence is not. If you genuinely can't estimate, say what you'd need to learn first — which is itself a useful signal about the project state.

**Iterate the schedule.** As the work progresses, update estimates. This is not a failure of the original estimate — it's the estimate doing its job. Estimates are not contracts; they're best guesses that get better with information.

**The estimate is not the commitment.** Separate "how long will this probably take" (estimate) from "what are we committing to deliver by when" (commitment). Padding an estimate to protect a commitment is how estimates get useless.

---

## Requirements

**Stated requirements are almost never the real ones.** Users describe their guess at a solution, not the problem underneath it. The bulk of requirements work is digging — asking why, watching users do the actual work, surfacing the assumptions that nobody noticed they were making.

**Don't gather — dig.** "We need a report that shows X, Y, and Z" is stated as a requirement but is really a guess at how to meet an underlying need. What decision is the report enabling? What would they do differently based on the information? The real requirement is the decision and the information it needs, not the specific report layout.

**Work with users to think like users.** Sit with them during the work the software is meant to help with. Watch what they do. Notice what's tedious, what they already do by hand, what their workarounds are. Requirements emerge from this faster than from any interview.

**Abstractions outlast details.** Capture requirements at the level of stable concepts — "customer," "order," "fulfillment" — and push specifics (fields, rules, formulas) to configuration or metadata where they can change without the core model changing.

**Build a project glossary.** When "customer" means different things to different stakeholders, bugs ship. A glossary forces the team to confront ambiguity early.

**When specs are bogged down — build.** Sometimes arguing over wording is less productive than building a small version of the thing and letting it make the spec precise. Code is the ultimate unambiguous specification.

---

## Tool Mastery

**The overall claim.** Tools multiply your effectiveness. Poor tool skills taxes every task; strong tool skills compound over years. Investment in tools is investment in yourself.

**Plain text.** Text formats outlive tools. They can be diffed, grepped, scripted, versioned, and opened by anything. When you have a choice about representation, prefer text.

**The shell.** A good shell is a composable environment for orchestrating small tools. Pipes and filters let you build custom workflows in minutes that would take hours in any GUI. Learn your shell well — keyboard shortcuts, piping, scripting, process substitution, command history, job control.

**One editor, learned deeply.** Flitting between editors prevents mastery of any. Pick one — whatever kind — and learn it fluently. Muscle-memory for common operations. Plugins for your work. Scripting or macros for repeated tasks. The depth compounds.

**Source control, for everything.** Not just code. Config files. Ops scripts. Documentation. Personal notes. Solo projects. One-off scripts. Version control is the substrate that enables fearless change — and enables recovering when a change turns out to be wrong.

**Text manipulation languages.** awk, sed, Perl, Python, Ruby — pick one. The ability to throw together a small text-processing script in minutes is disproportionately useful. Log analysis, data munging, one-off generators, glue between systems.

**Code that writes code.** Generation solves whole classes of duplication. Generate source from schemas. Generate configurations from specifications. Generate clients from API definitions. Generated code is always consistent; hand-maintained parallel copies drift.

**Automate ruthlessly.** Any procedure done by hand will eventually be done wrong. Any procedure done repeatedly will waste time. If you've done something twice manually, the third time should be scripted.

---

## Project-level Habits

**Organize around functionality, not job functions.** Teams built around "the database team, the UI team, the backend team" are slowed by every cross-team handoff. Teams built around delivering a feature end-to-end move faster. Mix the skills on one team; align on the outcome.

**Ubiquitous automation.** Build, test, deploy, release, reporting — all automated, all reproducible, all triggered by version-control events. Manual release procedures are where weekend incidents come from.

**Ruthless testing.** Run tests on every commit. Run them automatically. Treat a failing test as an emergency — either fix the test (if it's wrong) or fix the code (if the test is right), but don't let "failing test" become a normal state of the main branch.

**Documentation built in, not bolted on.** Documentation written at the end is always wrong because the system it describes has moved on. Documentation written alongside — in comments, in specs, in the README, in the commit message — evolves with the code.

**English is code too.** Specs, docs, error messages, commit messages, PR descriptions. Apply the same rigor: accurate, clear, minimal, revised. Bad prose in a spec produces bugs as reliably as bad code does.

**Gently exceed expectations.** Deliver what you promised, plus one small unexpected kindness — a better error message, a faster response, a setup step removed. Not flashy features; small things that make the user feel taken care of.

**Sign your work.** Take ownership. Put your name on what you ship. Accept credit for what works, responsibility for what doesn't. Craftspeople in other trades do this; software engineers should too.
