# Problem Classification

The 10 questions from `SKILL.md`, expanded. For each axis, this file gives: what the axis means, how to tell which side the problem is on (sometimes by asking the user, sometimes by inspecting the code or the data), and what's at stake if you get it wrong.

The order matters. Question 1 has to be answered before question 2 is well-defined, and so on.

---

## Axis 1: Continuous vs discrete decision variables

**What this means.** The decision variables `x` either take real-number values (weights, temperatures, fractions, angles) or discrete values (counts, binary choices, categorical selections, permutations).

**How to tell.**
- If the user writes `x = [0.3, 1.7, -0.2, ...]` or normalizes things to [0,1]: **continuous.**
- If the user says "choose k items," "which subset," "integer quantities," "route through these cities," "schedule these tasks," or "assign workers to jobs": **discrete.**
- If some are continuous and others are discrete (integer stock levels with continuous prices): **mixed.**

**What's at stake.** Continuous methods don't work on discrete problems — you can't take a gradient with respect to an integer. Relaxing the integer constraint and rounding at the end often gives an infeasible or clearly suboptimal answer.

**Method families implied.**
- Continuous → chapters 2–10, 12, 14–18 methods (first-order, second-order, direct, stochastic, surrogate)
- Discrete → integer programming, branch-and-bound, dynamic programming, combinatorial metaheuristics (simulated annealing, ant colony, GA)
- Mixed → mixed-integer programming (MILP for linear case; MINLP for nonlinear), which is its own sub-discipline

---

## Axis 2: Linear vs nonlinear

**What this means.** The objective `f(x)` and every constraint are linear functions of `x` (of the form `c·x + d`).

**How to tell.**
- Objective is a sum of `coefficient × variable` terms, constraints are inequalities of the same form → **linear program (LP).**
- Any variable is multiplied by another variable, squared, exponentiated, inside a `log`, etc. → **nonlinear.**

**What's at stake.** Linear programs are solved *exactly*, *globally*, and *fast* by the simplex algorithm or interior-point methods. The guarantee disappears the moment you introduce any nonlinearity. Recognizing an LP hidden inside a messier-looking problem saves enormous effort.

**A common reformulation.** Many problems that look nonlinear can be linearized with auxiliary variables. For example, `|x|` is nonlinear, but `minimize t subject to t ≥ x, t ≥ -x` is equivalent and linear. Before abandoning the LP path, check whether the nonlinearity can be refactored away.

**Method families implied.**
- LP → simplex or interior-point LP solver (scipy.optimize.linprog, CVXPY, Gurobi, CBC, HiGHS)
- Nonlinear → the rest of the classification pipeline

---

## Axis 3: Convex vs non-convex

**What this means.** A function is convex if every line segment between two points on its graph lies on or above the graph. A feasible set is convex if every line segment between two feasible points lies entirely in the feasible set. If both hold, the problem is convex.

**How to tell.**
- Quadratic `f(x) = x'Qx + ...` with positive semidefinite `Q` → convex. Common in least-squares and regularized regression.
- `log`, `exp`, `-log`, `x log x`, norms, max of linear functions → convex (or concave, depending on sign).
- Any inner multiplication of variables (bilinear), `sin`/`cos`, polynomials of degree ≥ 3 → typically non-convex.
- Books exist (Boyd & Vandenberghe, *Convex Optimization*) that catalog convex functions and composition rules. When in doubt: if the user can't immediately tell, assume non-convex.

**What's at stake.** A convex problem has a **global** minimum and most solvers find it reliably. A non-convex problem has local minima, potentially many of them, and most solvers find *a* local minimum — you may be arbitrarily far from the global one.

**The convex recognition payoff.** If you can model the problem as convex, you get optimality guarantees that no non-convex method can match, regardless of how sophisticated. Least-squares, linear programming, quadratic programming, semidefinite programming, geometric programming, and many others are all convex. Recognizing this structure and using a convex solver (CVXPY with a disciplined convex programming ruleset, or a commercial solver) is often the single biggest lever on the entire problem.

**Method families implied.**
- Convex → CVXPY and dedicated convex solvers (SCS, ECOS, MOSEK, CLARABEL)
- Non-convex → continue the pipeline, and accept local optimality unless you explicitly do global optimization

---

## Axis 4: Smooth vs non-smooth

**What this means.** "Smooth" here means differentiable almost everywhere, with a derivative that doesn't jump. `|x|` is *continuous* but not smooth at zero. Indicator functions, max-of-discrete, and floor/ceiling are neither.

**How to tell.**
- Polynomials, `exp`, `log`, `sin`, standard neural network activations (even ReLU, in practice) → smooth enough.
- `max(...)`, `if x > threshold`, floor, ceiling, rank, sort → non-smooth.
- Simulation output that depends on discrete internal events → often non-smooth in ways that break gradient methods.

**What's at stake.** Gradient-based methods assume smoothness. Applied to a non-smooth function, they may take wild steps, oscillate around discontinuities, or converge to a non-stationary point. Non-smooth methods exist (subgradient, proximal methods, bundle methods) but the simpler move is usually to use a derivative-free method.

**Method families implied.**
- Smooth → any first- or second-order method
- Non-smooth, convex → subgradient methods, proximal gradient (good for L1-regularized problems)
- Non-smooth, non-convex → derivative-free: Nelder-Mead, pattern search, CMA-ES, Bayesian optimization

---

## Axis 5: Gradient availability

Even if the objective is smooth in theory, the gradient may or may not be cheaply available in practice.

**Three tiers.**

**Tier 1: Analytic or autodiff gradient.** You can compute `∇f(x)` exactly, fast, at every `x`. Ideal. Automatic differentiation tools (PyTorch, JAX, TensorFlow, Zygote.jl, Enzyme) make this the default for anything expressible in numerical code.

**Tier 2: Finite differences.** You can approximate `∇f` by `(f(x+h) - f(x)) / h` for each coordinate. This costs `n+1` or `2n` evaluations per gradient (forward vs central). Acceptable for moderate `n` and cheap `f`, disastrous for expensive `f` or high `n`.

**Tier 3: No gradient.** `f` is a black box: a simulation, a physical experiment, a non-differentiable library, a human rating. You can call it but there's nothing to differentiate.

**How to tell.** If the objective is defined in code and the code is written in a framework you control (numpy/torch/jax), you're almost always Tier 1 — use autodiff. If the objective calls an external binary, a remote service, a simulation with internal random state, or a lookup table: Tier 3.

**What's at stake.** Using gradient-free methods when gradients are available wastes the most valuable information you have. Using gradient-based methods when gradients don't exist produces nonsense.

**Method families implied.**
- Tier 1 → first- and second-order methods, deep learning optimizers
- Tier 2 → scipy's quasi-Newton methods work, but watch the evaluation budget
- Tier 3 → derivative-free (direct methods, stochastic, population, Bayesian)

---

## Axis 6: Evaluation cost

**The four cost tiers.**

- **Microseconds.** Closed-form mathematical function. Any method works. Run a thousand iterations and don't worry about it.
- **Milliseconds to seconds.** Normal numerical code, small simulations, trained-model inference. Standard methods (L-BFGS, Nelder-Mead, CMA-ES) handle hundreds to thousands of iterations comfortably.
- **Seconds to minutes.** Substantial simulation, a training run, a long database query, a physical rendering. Budget is constrained — can't afford tens of thousands of evaluations. Choose methods with low iteration counts: L-BFGS if gradients are available, CMA-ES otherwise.
- **Minutes to hours (or more).** Training a large model from scratch, a CFD simulation, a real-world experiment. Budget is ≤ a few hundred evaluations, sometimes ≤ tens. This is the regime where **Bayesian optimization** (surrogate-model-based) becomes the correct frame.

**What's at stake.** The wrong method for the cost regime runs out of budget before finding anything useful. Running CMA-ES with a 30-minute-per-eval objective and a 100-eval budget is a waste; Bayesian optimization can often find the minimum in 20 evaluations.

**Method families implied.**
- Microseconds or milliseconds → anything
- Seconds → first/second-order if smooth; CMA-ES or Nelder-Mead if not
- Minutes+ → Bayesian optimization with Gaussian process surrogate, acquisition functions (Expected Improvement, Upper Confidence Bound, or Thompson sampling)

---

## Axis 7: Constraints

**Constraint categories.**

- **Unconstrained.** `x` can be anything in `R^n`. Simplest case.
- **Box/bound constraints.** `l_i ≤ x_i ≤ u_i`. Every modern solver handles these natively. Don't hack this with clipping; pass the bounds to the solver.
- **Linear equality / inequality constraints.** `A x = b`, `A x ≤ b`. Handled by LP/QP solvers directly; handled by nonlinear solvers through explicit constraint arguments (`scipy.optimize.minimize(..., constraints=...)`).
- **Nonlinear equality / inequality constraints.** `g(x) = 0`, `h(x) ≤ 0`. Requires a solver that supports nonlinear constraints (SLSQP, IPOPT, Knitro) or a penalty/augmented Lagrangian approach.
- **Implicit constraints.** "The objective just returns NaN or a large penalty if the input is infeasible." Brittle. Prefer explicit constraints.

**How to tell.** From the user's problem statement: "subject to X < budget" is a constraint. "x must be between 0 and 1" is a bound. "the sum must equal 1" is a linear equality constraint (simplex constraint, very common in portfolio and allocation problems).

**What's at stake.** Ignoring constraints and hoping the solver stays in bounds often fails silently. A solver that doesn't know about the constraints will gladly return an infeasible minimum.

**Method families implied.**
- Unconstrained → basic methods directly
- Box constraints → `L-BFGS-B` (scipy), constrained variants of most solvers
- Linear constraints → `SLSQP`, `trust-constr`, CVXPY for linear/quadratic
- Nonlinear constraints → `SLSQP`, `trust-constr`, IPOPT (via Pyomo or JuMP)
- Penalty-based approaches as a fallback if no solver handles your constraint type

---

## Axis 8: Deterministic vs noisy objective

**What this means.** Does calling `f(x)` twice with the same `x` return the same value?

- **Deterministic.** Same input, same output. Most classical optimization assumes this.
- **Stochastic / noisy.** Different outputs each call. Could be minibatch noise (training loss on a random batch), Monte Carlo noise (simulation), or measurement noise (physical experiment).

**How to tell.** Does the evaluation involve randomness anywhere? Sampling data? Random initialization inside the simulation? Real-world measurement? If yes, it's noisy.

**What's at stake.** Gradient-based methods assume gradients have zero noise. With noisy gradients, vanilla gradient descent zig-zags and can diverge. Methods that average or dampen noise (SGD with momentum, Adam) are purpose-built for this. Classical finite-difference gradient estimation is also defeated by noise if the noise exceeds the step size.

**Method families implied.**
- Deterministic → standard methods
- Noisy, large-scale (ML training) → SGD, Adam, RMSProp
- Noisy, low-dim → SPSA (Simultaneous Perturbation Stochastic Approximation), CMA-ES, noise-aware Bayesian optimization

---

## Axis 9: Single vs multiple objectives

**What this means.** One objective scalar vs multiple, possibly competing, objectives you'd like to improve jointly.

- **Single.** Minimize cost. Minimize loss. Minimize error. Standard case.
- **Multi-objective.** Minimize cost AND maximize reliability. Minimize drag AND maximize lift. Multiple numbers to balance.

**How to tell.** Does the user list more than one thing they want improved, and do those things trade off against each other? If yes, multi-objective.

**What's at stake.** There's no single "best" in a multi-objective problem — there's a *Pareto front* of points where you can't improve one objective without degrading another. Pretending it's single-objective by summing with arbitrary weights bakes a value judgment into the solution.

**Two ways to handle it.**

1. **Scalarize.** Combine objectives into one (weighted sum, ε-constraint, goal programming). Runs faster; requires an explicit tradeoff decision upfront.
2. **Pareto-front methods.** Produce a set of non-dominated solutions; let the user choose. NSGA-II is the workhorse.

**Method families implied.**
- Single → anything
- Multi-objective, quick answer → scalarization with weighted sum or ε-constraint, then any single-objective method
- Multi-objective, full tradeoff curve → NSGA-II (DEAP, Platypus), MOEA/D, SMS-EMOA

---

## Axis 10: Dimensionality

**Practical regimes.**

- **Low dimensional (n ≤ ~10).** Everything is tractable. Direct methods (Nelder-Mead, Powell) are viable. Grid search and full Bayesian optimization work. DIRECT and branch-and-bound can give global guarantees.
- **Moderate (n ~ 10–100).** First-order and quasi-Newton methods dominate. Nelder-Mead starts to struggle around n=20. Bayesian optimization still works but iteration cost per step grows cubically in data size.
- **High (n ~ 100–10,000).** First-order methods (gradient descent, L-BFGS). Quasi-Newton storage becomes limiting; use L-BFGS's limited-memory variant.
- **Very high (n > 10,000, up to billions for deep learning).** Only stochastic first-order methods: SGD, Adam, AdamW, sometimes second-order approximations (K-FAC, Shampoo) for specific structures.

**What's at stake.** Pattern search, Nelder-Mead, and CMA-ES do not scale past a few hundred dimensions. Full Newton's method (with Hessian) doesn't scale past about 10⁴ due to `O(n²)` memory and `O(n³)` factorization. If you reach for the wrong method at the wrong scale, it just doesn't terminate.

**Method families implied.**
- Low → any
- Moderate → first/second-order shine
- High → L-BFGS, conjugate gradient, first-order with momentum
- Very high → Adam/SGD/variants only

---

## Classification summary

Write down (mentally or explicitly) the answers to all 10 before choosing a method:

| Axis | Answer |
|------|--------|
| Continuous / discrete / mixed | ? |
| Linear / nonlinear | ? |
| Convex / non-convex | ? |
| Smooth / non-smooth | ? |
| Gradient: analytic / FD / none | ? |
| Eval cost: μs / ms / s / min+ | ? |
| Constraints: none / box / linear / nonlinear | ? |
| Deterministic / noisy | ? |
| Single / multi-objective | ? |
| Dimension | ? |

The combination determines the method. The table in `SKILL.md` and the detail in `methods.md` handle the mapping from here.
