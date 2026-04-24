# The 70 Tips

The numbered tips from *The Pragmatic Programmer*. Each is a compact handle for a larger idea — the book cross-references them throughout. When applying one, cite it by number ("per Tip 11: DRY") so a reader knows exactly which principle is at stake.

Organized by chapter for reference. In the book itself they're scattered through the relevant text.

---

## Chapter 1 — A Pragmatic Philosophy

**Tip 1: Care About Your Craft.** There is no point in developing software unless you care about doing it well. Set that as a prerequisite.

**Tip 2: Think! About Your Work.** Software is not an autopilot activity. Every decision — what to build, how to structure it, what to test, what to defer — deserves active thought. The cost of turning your brain off is measured in bugs and rework weeks later.

**Tip 3: Provide Options, Don't Make Lame Excuses.** When something is broken, late, or impossible as stated, don't present that as "can't be done." Present it as: here's the situation, here are two or three paths forward, here's what each costs and buys. Own the problem even if you didn't cause it.

**Tip 4: Don't Live with Broken Windows.** One unfixed bad design, disabled test, or known-bad code path signals to everyone that quality isn't valued here. The rot accelerates fast. Fix broken windows on sight. If you can't fix, *visibly* board them up (explicit comment, ticket, failing test renamed to document the known issue) so nobody thinks the neglect is deliberate.

**Tip 5: Be a Catalyst for Change.** When the situation demands change but nobody wants to take it on, start small. Do the first piece of the work. Invite others to contribute to something that's already moving. A partial system in motion draws participation that a proposal in a meeting doesn't.

**Tip 6: Remember the Big Picture.** It's easy to get absorbed in a small optimization or a local fight and lose sight of the point. Step back periodically. Ask: is what I'm working on still the right thing to be working on?

**Tip 7: Make Quality a Requirements Issue.** Quality isn't something you add at the end. It's a dimension of what's being built, and it should be discussed with the same rigor as features. "How reliable does this need to be?" is a valid requirements question.

**Tip 8: Invest Regularly in Your Knowledge Portfolio.** Knowledge is an investment that depreciates. Diversify (multiple languages, paradigms, domains). Invest regularly (a little every week beats occasional binges). Rebalance (what's hot now won't be in five years). Read outside your immediate stack.

**Tip 9: Critically Analyze What You Read and Hear.** Don't accept advice, benchmarks, or claims at face value — including this book's. Ask who's saying it, why, and whether their context matches yours.

**Tip 10: It's Both What You Say and the Way You Say It.** Engineering work is communication work. An audit-correct technical answer delivered confusingly still fails. Pay attention to audience, medium, timing, and structure — not just accuracy.

---

## Chapter 2 — A Pragmatic Approach

**Tip 11: DRY — Don't Repeat Yourself.** Every piece of knowledge should have a single, unambiguous, authoritative representation within a system. This is the book's most important technical rule. Duplication isn't just ugly — it's a guarantee that one copy will change and the other won't, creating a bug.

**Tip 12: Make It Easy to Reuse.** If reuse is painful (complicated APIs, tangled dependencies, inconsistent conventions), people won't reuse — they'll re-implement. Duplication follows friction. Design components to be easy to pick up and drop into a new context.

**Tip 13: Eliminate Effects Between Unrelated Things.** Orthogonality. Two things are orthogonal if changes to one don't force changes in the other. In well-designed systems, the database code is orthogonal to the UI. Test: if a requirement for component A changes dramatically, how many other components have to change? Ideal answer: one.

**Tip 14: There Are No Final Decisions.** Vendors, frameworks, requirements, architectures — all of it changes. Design with reversibility in mind. Wrap external dependencies. Abstract deployment models. Don't carve commitments in stone; write them in sand.

**Tip 15: Use Tracer Bullets to Find the Target.** Build the thinnest possible end-to-end working version of the system first, exercising every component. It becomes the skeleton you grow. Gives users something to react to early. Gives developers a structure to fill in. Different from prototypes (which are thrown away) — tracer code is kept.

**Tip 16: Prototype to Learn.** Prototypes are throwaway experiments to answer specific questions: will this algorithm work? Will users understand this UI? Is this architecture viable under load? Build the cheapest thing that answers the question and discard it.

**Tip 17: Program Close to the Problem Domain.** Let the vocabulary and structure of the code match the vocabulary and structure of the problem. Where possible, build a mini-language (DSL, internal API, config format) in which the business rules can be expressed directly.

**Tip 18: Estimate to Avoid Surprises.** Estimating is a skill, practiced like any other. Always state units that reflect your confidence (days, weeks, months, quarters). Don't refuse to estimate; a rough estimate honestly refined is more useful than silence.

**Tip 19: Iterate the Schedule with the Code.** Estimates are not contracts. As the work progresses and teaches you about the problem, update the estimate. This is not a failure of the original estimate — it's what estimates are for.

---

## Chapter 3 — The Basic Tools

**Tip 20: Keep Knowledge in Plain Text.** Text formats outlast binaries, are diff-able, grep-able, scriptable, and survive tool changes. When you have a choice about how to represent something, prefer text.

**Tip 21: Use the Power of Command Shells.** A good shell composes small tools into custom workflows in ways that GUIs can't. Learn your shell. Learn pipes and filters. Build small scripts for recurring tasks.

**Tip 22: Use a Single Editor Well.** Rather than flitting between editors, pick one and learn it deeply — keystrokes, extensions, scripting, navigation. The depth compounds over time.

**Tip 23: Always Use Source Code Control.** Everything. Code, configuration, documentation, ops scripts, personal pet projects. Even if you're alone. Even for a one-week job. Version control is the substrate that enables fearless change.

**Tip 24: Fix the Problem, Not the Blame.** When a bug surfaces, the goal is to understand and fix it. Whether it's your code or someone else's is irrelevant to the fix — and dwelling on blame wastes the time that should go to diagnosis.

**Tip 25: Don't Panic.** When debugging, panic produces flailing. Step back. The fault is almost never what your first reaction says it is. "That's impossible" is wrong — whatever you're seeing, clearly it *is* possible, and the job is to find out why.

**Tip 26: "select" Isn't Broken.** The OS, standard library, and compiler are almost never the bug. Before blaming them, exhaust the possibility that it's your code. Even when it really is a platform bug, the working assumption has to be that it's yours first.

**Tip 27: Don't Assume It — Prove It.** When diagnosing, don't reason from what the code "should" do. Check. Add logging, run the debugger, write a test. Real data beats speculation.

**Tip 28: Learn a Text Manipulation Language.** Awk, sed, Perl, Python, Ruby — whatever. Being able to throw together a small text-processing program in minutes is disproportionately useful for data munging, log analysis, one-off generators, and glue.

**Tip 29: Write Code That Writes Code.** Code generation solves whole classes of duplication that would otherwise have to be maintained by hand. Generate source from schemas. Generate schemas from source. Generate tests from specifications. Always prefer generated to hand-maintained.

---

## Chapter 4 — Pragmatic Paranoia

**Tip 30: You Can't Write Perfect Software.** Accept this as an axiom. Perfect software doesn't exist and won't start with you. Everything that follows in this chapter is about working productively within that reality.

**Tip 31: Design with Contracts.** Every function/method has preconditions (what must be true for it to be called), postconditions (what it guarantees), and invariants (what's true throughout). Writing these down — even just as comments or asserts — documents the interface and catches violations fast. Be strict in what you accept; promise as little as possible.

**Tip 32: Crash Early.** When something impossible happens, crash immediately with a clear message. A program that limps on producing increasingly wrong output is worse than one that stops. Dead programs tell no lies; zombie programs tell plausible ones.

**Tip 33: If It Can't Happen, Use Assertions to Ensure That It Won't.** Assertions document your model of reality. If you "know" x can't be null here, assert it. Either the assertion holds (self-documentation) or it fires (you were wrong about your model, which is information you badly needed). Never use asserts as production error handling — use them for impossible conditions.

**Tip 34: Use Exceptions for Exceptional Problems.** Exceptions are a control-flow mechanism for situations that truly prevent the normal flow from continuing — not for ordinary branching. A function that "throws on file not found" when file-not-found is an expected case is abusing the mechanism. Reserve exceptions for cases the caller genuinely cannot handle as part of normal logic.

**Tip 35: Finish What You Start.** The routine that allocates a resource should release it. The function that opens the file should close it. The caller that acquired the lock should free it. Don't scatter the lifecycle of a resource across unrelated parts of the system.

---

## Chapter 5 — Bend, or Break

**Tip 36: Minimize Coupling Between Modules.** Modules that know about each other's internals are fragile to each other's changes. Write "shy" code — modules reveal little and ask little. Law of Demeter applies: a method should only call methods of the object itself, its parameters, objects it creates, or its direct components.

**Tip 37: Configure, Don't Integrate.** Embedding choices (database, transport, algorithm) into the code as conditionals hard-wires them. Express the choices as configuration, loaded at startup. The code is the same; the config determines behavior. Makes switching easier and reversibility cheaper.

**Tip 38: Put Abstractions in Code, Details in Metadata.** Code should express the mechanism (how to render a report, how to validate data). The particular details (what columns, which validations) belong outside the code, where they can change without rebuilding. Pushes decisions to the edges, where they're cheap to revisit.

**Tip 39: Analyze Workflow to Improve Concurrency.** Look at the tasks in the system and ask which ones actually depend on which. Often what looks sequential is only sequential by accident of how it was coded. True data flow reveals opportunities for parallelism.

**Tip 40: Design Using Services.** Build systems as cooperating services with clear interfaces, not as a monolith with internal tentacles. Services can be developed independently, deployed independently, and reasoned about independently.

**Tip 41: Always Design for Concurrency.** Even if today's deployment is single-threaded, design as if concurrent access were possible. This shows up in API choices (don't depend on temporal coupling between calls), state management (avoid shared mutable state), and resource handling. It's much harder to retrofit concurrency safety than to design it in.

**Tip 42: Separate Views from Models.** The data and the display of the data are different concerns. Multiple views of the same data should be possible without duplication. When the UI changes drastically, the model shouldn't have to.

**Tip 43: Use Blackboards to Coordinate Workflow.** When multiple independent processes need to collaborate on a problem, a shared "blackboard" (queue, tuple space, message bus) is often cleaner than point-to-point coupling. Each process posts what it learned; others react to what's posted. No process needs to know about the others directly.

---

## Chapter 6 — While You Are Coding

**Tip 44: Don't Program by Coincidence.** If your code works and you're not sure why, you have a bug — you just haven't seen it yet. Understand *why* every piece works. Rely only on documented behavior. If you don't know whether a routine handles null, find out — don't assume based on the one case you tried.

**Tip 45: Estimate the Order of Your Algorithms.** Know the big-O of what you're writing. If you've got a nested loop over N items that's fine for N=100, will it still be fine when N=100,000? Don't guess — count. Don't sweat over constant-factor differences; do sweat over going from linear to quadratic.

**Tip 46: Test Your Estimates.** Big-O analysis is a theoretical estimate. Real systems surprise you. Time your code against representative data sizes. Graph it. The shape of the curve will tell you whether your analysis matches reality.

**Tip 47: Refactor Early, Refactor Often.** When you see code that could be clearer or better structured, improve it — especially while it's small and the change is cheap. Deferred cleanup accumulates into a mass that's too scary to tackle. Like gardening: pull weeds early while they're small.

**Tip 48: Design to Test.** Code that's hard to test is hard to use, because tests are just the first users. If you can't isolate a unit to test it, your module boundaries are probably wrong. If you can't set up test data easily, your data dependencies are probably wrong. Testability is a design quality.

**Tip 49: Test Your Software, or Your Users Will.** Either you test it or they do. They're much less forgiving. Invest in tests that exercise real conditions, not just the happy path — boundary conditions, error paths, unusual data, unexpected orderings.

**Tip 50: Don't Use Wizard Code You Don't Understand.** If a tool generates code for you and you don't understand what that code does, you don't own your application — the tool does. When the generated code breaks, or when you need to change it, you'll be helpless. Use generators whose output you can read.

---

## Chapter 7 — Before the Project

**Tip 51: Don't Gather Requirements — Dig for Them.** Stated requirements are almost never the real ones. Users describe their guess at a solution, not their underlying problem. Dig: ask why, watch them work, ask what currently hurts, ask what "done" looks like. The real requirement is underneath the stated one.

**Tip 52: Work with a User to Think Like a User.** Sit with actual users. Shadow them doing the work the software is supposed to help with. You'll see assumptions and workflows that no specification would have captured.

**Tip 53: Abstractions Live Longer than Details.** When capturing requirements, go for the abstractions. "Customer" and "order" and "fulfillment" will still mean something five years from now; specific business rules will have turned over many times. Push the details to the edges (metadata, configuration) where they can change.

**Tip 54: Use a Project Glossary.** Build and maintain a shared vocabulary of terms specific to the project. When "customer" means one thing to sales, another to legal, and a third to engineering, you will ship bugs. A glossary forces the team to confront and resolve ambiguity.

**Tip 55: Don't Think Outside the Box — Find the Box.** When a problem seems impossible, don't immediately try to "think outside the box." Examine the box itself — the constraints you're assuming. Often the constraint you thought was binding is negotiable, and the problem dissolves.

**Tip 56: Listen to Nagging Doubts — Start When You're Ready.** Experienced engineers develop an intuition for "this isn't right" that fires before they can explain it rationally. Listen to that signal. It's usually your subconscious having spotted a pattern it can't yet articulate.

**Tip 57: Some Things Are Better Done than Described.** When specifications are bogged down in detail, sometimes the fastest way to clarity is to build a small version of the thing and let it make the specification precise. Code is the ultimate unambiguous specification.

**Tip 58: Don't Be a Slave to Formal Methods.** Formal methods and heavyweight processes have their place, but they are tools, not religions. If a formal method is not helping you deliver, stop using it. Pragmatism beats orthodoxy.

**Tip 59: Expensive Tools Do Not Produce Better Designs.** A great designer with a whiteboard beats a mediocre designer with a six-figure CASE tool. Tools help, but they don't substitute for thinking.

---

## Chapter 8 — Pragmatic Projects

**Tip 60: Organize Around Functionality, Not Job Functions.** Teams organized as "the database team," "the UI team," "the back-end team" get slowed down by every cross-team coordination. Teams organized around delivering a feature end-to-end move faster. Mix the skills; align on the outcome.

**Tip 61: Don't Use Manual Procedures.** Any procedure done by hand will eventually be done wrong. Any procedure done repeatedly will waste time. If you've done something twice by hand, automate it by the third time.

**Tip 62: Test Early. Test Often. Test Automatically.** Automated tests running on every change are the difference between knowing the system works and hoping it works. Manual testing is an input to writing the automated test, not a substitute for it.

**Tip 63: Coding Ain't Done 'Til All the Tests Run.** A feature with passing tests is done. A feature without passing tests is an unverified claim. Tests are not a separate phase to tack on at the end — they are the definition of done.

**Tip 64: Use Saboteurs to Test Your Testing.** Deliberately introduce bugs into the code and confirm the tests catch them. If they don't, your tests aren't testing what you think. Mutation testing formalizes this — either way, the underlying question is: do your tests actually catch defects, or just exercise code?

**Tip 65: Test State Coverage, Not Code Coverage.** Line-coverage is a weak metric. A test that executes a line without checking its effects passes coverage but catches no bugs. What matters is whether the meaningful *states* of the system — valid inputs, invalid inputs, boundary conditions, error cases — are exercised and checked.

**Tip 66: Find Bugs Once.** When a bug escapes to users, write a test that would have caught it. That bug should not be possible to reintroduce. Over time this test suite becomes a record of every bug the system has ever had and an insurance policy against their return.

**Tip 67: Treat English as Just Another Programming Language.** The docs, specifications, error messages, and commit messages you write are code too. Apply the same rigor: accuracy, clarity, minimalism, revision. Bad prose in a spec produces bugs as reliably as bad code.

**Tip 68: Build Documentation In, Don't Bolt It On.** Documentation written at the end is always wrong, because the system it describes has moved on. Documentation written alongside — in comments, in the spec, in the README — evolves with the code and has a chance of being accurate.

**Tip 69: Gently Exceed Your Users' Expectations.** Deliver what you promised, plus one thing more than they expected. Not a flashy feature — a small kindness. A better error message, a faster response, a setup step removed. This is how trust accumulates.

**Tip 70: Sign Your Work.** Craftspeople in other trades are proud of their work and own it publicly. Do the same. Put your name on what you ship. Take pride in the code. Accept responsibility for its defects. Accept credit for its successes.
