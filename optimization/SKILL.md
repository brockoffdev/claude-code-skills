---
name: optimization
description: Apply mathematical optimization thinking when the user is working on a problem of the form "find inputs x that minimize (or maximize) some objective f(x), possibly subject to constraints." This includes fitting model parameters, tuning hyperparameters, finding the best configuration for an engineering design, solving scheduling or resource allocation problems, training models, calibrating simulations, and any "find the best values" task. Also fires when the user says "please optimize the code," "optimize this," "optimize this function," or similar, because that phrase is ambiguous in practice — this skill's first step when it hears such phrasing is to disambiguate between mathematical optimization (finding better inputs) and software performance tuning (making existing code faster). If the user turns out to want performance tuning, this skill politely steps aside rather than applying irrelevant algorithm advice. Do not use for pure algorithm complexity analysis, pure data structure choice, or requests that are clearly about profiling and performance engineering.
---

# Optimization

You are applying the methods and taxonomy of mathematical optimization — the problem of finding input values `x` that minimize (or maximize) an objective function `f(x)`, possibly subject to constraints. Hundreds of algorithms exist for this general problem. Picking the right one is ~90% of the skill; the remaining 10% is setting it up and running it correctly.

## First: disambiguate what "optimize" means

The word "optimize" is ambiguous. Before doing anything else, figure out which kind of problem the user actually has.

**Mathematical optimization (this skill's territory).** The user wants to *find better inputs* to a problem. The code runs; they want it to produce a *better result*. Examples:
- "Find the learning rate that gives the lowest validation loss"
- "Fit these parameters to this data"
- "Find the portfolio allocation that maximizes return for a given risk"
- "Find the wing shape that minimizes drag"
- "Tune these hyperparameters"
- "Schedule these jobs to minimize total completion time"
- "Find the cheapest shipping route"

**Software performance engineering (not this skill).** The user wants the *same* inputs to produce the *same* outputs, but faster / using less memory / with fewer allocations. Examples:
- "Make this Python loop faster"
- "Why is this O(n²)?"
- "Reduce memory usage in this pipeline"
- "Vectorize this NumPy code"

**If the request is ambiguous, ask.** A single clarifying question saves both parties an hour of wrong-track work. Something like: "Quick check — do you want to find better *inputs* to this function (mathematical optimization), or make the same computation *run faster* (performance tuning)? Those have different toolkits."

**If the user wants performance tuning, step aside gracefully.** Say so explicitly — "this is performance engineering rather than mathematical optimization; the approach is profiling + hot-path analysis + data structure choices, not gradient descent or Bayesian optimization. I can still help, but via general knowledge rather than this skill." Don't force-fit the skill onto a problem it wasn't built for.

The rest of this file assumes the answer is mathematical optimization.

## The central claim: No Free Lunch

There is no universally best optimization algorithm. Any algorithm that's good at one class of problem is worse at another. This is a theorem (Wolpert & Macready), not a rule of thumb.

The practical consequence: **method selection is the decision that matters most**. Five minutes spent classifying the problem saves hours of watching the wrong solver fail to converge.

## The problem-classification pipeline

Before reaching for an algorithm, answer these in order. Each answer constrains the next.

1. **Are the decision variables continuous or discrete?**
   - Continuous (real numbers) → Chapters 2–10 methods
   - Discrete (integers, categorical) → integer programming, branch-and-bound, combinatorial optimization
   - Mixed → mixed-integer programming (MILP, MINLP)

2. **Is the problem linear?**
   - Both objective and all constraints linear in `x` → linear programming; use a simplex or interior-point LP solver
   - Anything nonlinear → continue

3. **Is the problem convex?**
   - Convex objective, convex feasible set → convex optimization has strong guarantees; use a convex solver (CVXPY, CVX, MOSEK). A convex problem you recognize as convex is almost always solved.
   - Non-convex → continue, but accept that you'll find a *local* minimum, not a guaranteed global one.

4. **Is the objective smooth (differentiable almost everywhere)?**
   - Smooth → first-order or second-order methods apply
   - Non-smooth or discontinuous → derivative-free methods only

5. **Is the gradient available?**
   - Analytical gradient / automatic differentiation → first- and second-order methods
   - Finite differences acceptable → still mostly first/second-order, but evaluation cost doubles or more per iteration
   - No gradient available → derivative-free (Nelder-Mead, Powell, CMA-ES, Bayesian optimization)

6. **How expensive is one evaluation of `f(x)`?**
   - Microseconds (closed-form math) → almost any method works; pick by other properties
   - Milliseconds–seconds → standard gradient-based or derivative-free methods
   - Minutes–hours (simulation, training a model, a physical experiment) → *surrogate-based / Bayesian optimization* becomes the right frame. Use the expensive function sparingly and model it with a cheap surrogate (Gaussian process).

7. **Are there constraints?**
   - Unconstrained or box-bounded → most methods apply directly
   - Equality/inequality constraints → constrained optimization (penalty methods, Lagrangian, interior point, augmented Lagrangian, or a solver that handles constraints natively)

8. **Is the objective deterministic or noisy?**
   - Deterministic → standard methods
   - Noisy (stochastic simulation, minibatch loss) → stochastic methods: SGD/Adam family, CMA-ES, noise-robust Bayesian optimization

9. **Single objective or multiple competing objectives?**
   - Single → scalar optimization
   - Multiple → multi-objective optimization (scalarization via weighted sum / ε-constraint, or Pareto-front methods like NSGA-II)

10. **What's the dimension?**
    - Small (≤ tens of variables) → almost anything works; Nelder-Mead and pattern search are viable
    - Medium (tens–hundreds) → first-order and quasi-Newton (L-BFGS) shine
    - Large (thousands–millions, e.g., neural nets) → only first-order stochastic methods (SGD, Adam) scale

The full classification table is in `references/problem-classification.md`.

## Mapping classifications to method families

Once classified, `references/methods.md` maps to the method family. A quick reference:

| Problem shape                                            | Go-to methods                                     |
|----------------------------------------------------------|--------------------------------------------------|
| Smooth, gradient available, small–medium dim             | L-BFGS / BFGS (quasi-Newton)                     |
| Smooth, gradient available, huge dim (deep learning)     | Adam, SGD with momentum                          |
| Smooth, gradient expensive, have Hessian                 | Newton's method (trust region)                   |
| Non-smooth or no gradient, cheap eval, small dim         | Nelder-Mead, Powell's method                     |
| Non-smooth, gradient-free, moderate–high dim             | CMA-ES, differential evolution                   |
| Expensive objective (minutes+ per eval)                  | Bayesian optimization (GP surrogate + EI/UCB)    |
| Linear objective + linear constraints                    | Simplex or interior-point LP                     |
| Convex objective + convex constraints                    | CVXPY or commercial convex solver                |
| Integer / mixed-integer                                  | Branch-and-bound, MILP solver (Gurobi, CBC)      |
| Combinatorial (TSP, scheduling, routing)                 | Simulated annealing, tabu search, GA; or exact MILP if small |
| Multi-objective                                          | NSGA-II, weighted sum, ε-constraint              |
| Stochastic/noisy objective                               | SPSA, Adam, CMA-ES, noise-aware BO               |
| Global minimum needed in a non-convex landscape          | Basin-hopping, multi-start, evolutionary methods, DIRECT |

**Cardinal rule: don't roll your own.** For almost every entry in this table, a well-tested library implementation exists. See `references/methods.md` for concrete package pointers (scipy.optimize, CVXPY, Optuna, Optim.jl, JuMP, PyTorch/JAX optimizers, etc.). Hand-written gradient descent loops on a real problem are almost always a mistake — they converge slower, handle constraints worse, and miss 50 years of numerical-analysis polish.

## Traps to watch for

Even with the right algorithm, several pitfalls sink optimization projects. Watch for these on any request.

**Variable scaling.** Variables measured in very different units (temperature in Kelvin vs pressure in Pascals vs mass fraction 0–1) ruin gradient-based methods. Scale every variable to roughly the same magnitude (often [0,1] or unit-variance) before optimizing. This is the single most common silent failure.

**Local vs global minima.** Most methods find *a* local minimum, not *the* global one. If the user needs a global optimum on a non-convex problem, say so — the tools are different (multi-start, basin-hopping, evolutionary methods, Bayesian optimization, or DIRECT for low-dim problems with Lipschitz structure). Claiming to have "the optimum" from a local method is often wrong.

**Finite-difference gradients on expensive functions.** Computing `∇f` by finite differences multiplies your evaluation budget by `n+1` (or `2n` for central differences) every iteration. If the objective is expensive, use automatic differentiation (PyTorch, JAX, Zygote.jl) or a gradient-free method instead.

**Numerical noise mistaken for non-smoothness.** A "noisy" simulation sometimes produces derivatives that look nonsensical, even though the underlying function is smooth. Check whether the noise floor of `f(x)` is small compared to the gradient step size you're using. If not, either increase the step size (finite differences) or switch to a noise-tolerant method.

**Optimizing a proxy.** The objective function you wrote may not be the thing you actually want optimized. A perfectly-solved wrong problem is still wrong. Before declaring success, verify that the minimizer of `f(x)` is actually the outcome the user wants. This is Goodhart's Law — "when a measure becomes a target, it ceases to be a good measure."

**Terminating too early or too late.** Default solver tolerances are often wrong for the specific problem. Run once with a generous budget to understand convergence, then tune tolerances based on what you saw.

**Constraint handling via clipping.** Clipping variables to their bounds inside a gradient computation breaks gradient-based methods because the "gradient" becomes discontinuous. Either use a solver that handles bounds natively (most do) or use a smooth transformation (e.g., optimize over the unconstrained logit, map to the bounded variable with sigmoid).

**Reinventing things.** The catalog of named, tuned, tested optimization algorithms is enormous. If you find yourself designing a novel "acceptance criterion" or "exploration term" from scratch, stop and check the literature; it almost certainly has a name.

## Workflow when the skill fires

1. **Confirm the request is mathematical optimization** (per the disambiguation at the top of this file). If not, step aside.
2. **Classify the problem** using the 10-question pipeline. Ask the user whichever of those questions aren't clear from context — their answers determine the rest.
3. **Identify the method family** from the classification → methods table. If multiple are plausible, say so and briefly explain the tradeoff.
4. **Recommend a concrete implementation** from `references/methods.md` — a specific library and function, not just a method name. `scipy.optimize.minimize(..., method='L-BFGS-B')` beats "use quasi-Newton."
5. **Surface the traps** that apply to this specific problem (scaling? local vs global? expensive objective? noise?). Don't list all of them — just the ones at risk here.
6. **Help set it up** — bounds, initial point, tolerances, callbacks. Good defaults exist; use them.
7. **Check the result before declaring victory.** Did the solver converge or hit max iterations? Does the solution satisfy the constraints? Does the objective value match expectations? Is the answer stable under a different initial point?

## Output expectations

A useful response under this skill:

1. **Names the problem class** in the book's vocabulary (e.g., "this is a smooth, unconstrained, medium-dimensional problem with automatic gradients available — so: L-BFGS").
2. **Explains *why* that class**, not just the answer. "The objective is smooth → gradient methods apply. You have a few dozen variables → quasi-Newton is cheaper than BFGS on memory but faster than gradient descent."
3. **Gives a concrete implementation** — a specific function call in a specific library, with realistic hyperparameters.
4. **Lists the one or two traps most likely to bite** on this specific problem.
5. **Defers to exact-solve tools when applicable** — if the problem is actually a linear program or a convex one, say so and recommend CVXPY/LP-solver rather than a generic nonlinear optimizer.

## References

- `references/problem-classification.md` — the 10-axis classification in detail, with heuristics for each axis and the questions to ask the user to determine the answer.
- `references/methods.md` — each method family (first-order, second-order/quasi-Newton, direct, stochastic, population, surrogate/Bayesian, linear, convex, discrete, multi-objective) with strengths, weaknesses, tuning knobs, and concrete library implementations. **Use this for production recommendations.**
- `references/algorithms.md` — 33 pedagogical NumPy implementations of the most practically useful algorithms (gradient descent, Adam, L-BFGS, Nelder-Mead, CMA-ES, simulated annealing, GA, differential evolution, particle swarm, Gaussian process + Bayesian optimization, NSGA-II core, branch-and-bound, etc.). **Use this to understand *how* a method works — for explaining behavior, diagnosing failures, or teaching. Do not recommend as production code; always route to the libraries in `methods.md` when actually solving a problem.**

Read the relevant section before recommending an algorithm you haven't prescribed in a while — the tradeoffs are specific, and getting the class wrong is expensive.
