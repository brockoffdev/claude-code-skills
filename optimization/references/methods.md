# Methods

Each family of optimization algorithms covered in the book, with:
- **When to use:** the problem shape(s) it fits
- **How it works:** a one-paragraph summary, enough to understand what it's doing
- **Strengths / weaknesses**
- **Key tuning knobs**
- **Concrete implementations** in common languages/libraries

Cardinal rule throughout: prefer an existing library implementation over writing your own. Decades of numerical analysis have gone into making these solvers robust; starting from scratch gives that up.

---

## First-Order Methods

### Gradient descent (steepest descent)

**When to use.** Teaching; baseline comparisons; very simple problems. Almost never in production.

**How it works.** At each step, move in the direction of `-∇f(x)` with some step size `α`.

**Strengths.** Simple. Fast per iteration (one gradient evaluation). Scales to huge dimensions.

**Weaknesses.** Slow convergence on ill-conditioned problems. Zig-zags in narrow valleys. Vanilla step size is fragile.

**Tuning knobs.** Step size / learning rate. Line search vs fixed step.

**Implementations.** `scipy.optimize.minimize(..., method='CG')` uses conjugate gradient, which is strictly better. Use that instead.

### Conjugate gradient (CG)

**When to use.** Smooth, unconstrained, gradient available, moderate-to-large dimension (hundreds to thousands). Strong for quadratic-like objectives.

**How it works.** Like gradient descent, but the descent direction is adjusted to be *conjugate* to previous directions, avoiding the zig-zag. For quadratic problems, converges in at most `n` steps.

**Strengths.** Very low memory (no matrix storage). Much faster than plain gradient descent.

**Weaknesses.** Less robust than quasi-Newton on non-quadratic problems.

**Implementations.**
- Python: `scipy.optimize.minimize(f, x0, jac=grad, method='CG')`
- Julia: `Optim.jl` with `ConjugateGradient()`

### Momentum / Nesterov momentum

**When to use.** Noisy gradients (stochastic), deep learning, escaping shallow local minima. Almost never a bad idea with SGD.

**How it works.** Maintain a velocity vector that accumulates past gradient directions. The step is a weighted combination of current gradient and past momentum.

**Strengths.** Smooths noisy updates; accelerates convergence on ill-conditioned problems; helps traverse flat regions.

**Weaknesses.** Adds a momentum coefficient hyperparameter (usually 0.9).

**Implementations.**
- PyTorch: `torch.optim.SGD(params, lr=..., momentum=0.9, nesterov=True)`
- JAX/Optax: `optax.sgd(lr, momentum=0.9, nesterov=True)`

### Adagrad / RMSProp / Adadelta / Adam

**When to use.** Deep learning. Large-dimensional stochastic problems. Anywhere gradients are noisy and dimensions differ substantially in scale.

**How it works.** Each maintains a per-coordinate learning rate that adapts based on the history of gradients for that coordinate. Adagrad decays learning rate over time (useful for convex, can stall for non-convex). RMSProp uses an exponentially weighted average instead. Adam combines RMSProp's scale adaptation with momentum — arguably the most-used optimizer in deep learning.

**Strengths.** Robust defaults. Handles varied gradient scales. No line search needed.

**Weaknesses.** Not strictly convergent on some problems; occasionally outperformed by well-tuned SGD with momentum. AdamW fixes Adam's weight decay interaction for regularized problems.

**Tuning knobs.** Learning rate (Adam default 1e-3 is a good starting point). Betas (0.9, 0.999 are standard). Epsilon.

**Implementations.**
- PyTorch: `torch.optim.Adam`, `torch.optim.AdamW`
- JAX/Optax: `optax.adam`, `optax.adamw`
- TensorFlow: `tf.keras.optimizers.Adam`

### L-BFGS (limited-memory BFGS) — properly a quasi-Newton method but listed here for workflow

**When to use.** Smooth, unconstrained or box-constrained, gradient available, medium-to-high dimension. *The default choice for most real-valued smooth optimization problems where gradients are available.*

**How it works.** Approximates the Hessian inverse using a limited-memory history of recent gradient differences. Performs a line search each step.

**Strengths.** Fast convergence (superlinear on smooth problems). Modest memory (`O(n × m)` where `m` is history size, typically 5–20). Handles bound constraints natively (`L-BFGS-B`).

**Weaknesses.** Assumes smoothness; struggles with noise. Can be defeated by numerical gradient errors.

**Tuning knobs.** Memory parameter `m`. Line search tolerance.

**Implementations.**
- Python: `scipy.optimize.minimize(f, x0, jac=grad, method='L-BFGS-B', bounds=...)`
- Julia: `Optim.jl` with `LBFGS()`
- SciKit and many ML libraries use it as a backend

---

## Second-Order Methods

### Newton's method

**When to use.** Smooth, unconstrained, Hessian available (cheaply), small-to-medium dimension.

**How it works.** Uses both gradient and Hessian. Move to `x - H⁻¹ ∇f(x)` — the minimum of the local quadratic approximation.

**Strengths.** Quadratic convergence near the optimum (very fast in the final iterations).

**Weaknesses.** Requires Hessian: `O(n²)` memory, `O(n³)` factorization. Vulnerable to non-positive-definite Hessians (pure Newton's step can go uphill). Needs a trust region or line search to be robust.

**Tuning knobs.** Trust region radius. Hessian regularization.

**Implementations.**
- Python: `scipy.optimize.minimize(f, x0, jac=grad, hess=hess, method='trust-ncg')` or `'trust-krylov'` or `'Newton-CG'`
- Julia: `Optim.jl` with `Newton()` or `NewtonTrustRegion()`

### Quasi-Newton methods (BFGS, L-BFGS, SR1)

**When to use.** BFGS for small–medium problems where you have gradients but not Hessians. L-BFGS for larger problems. Essentially: smooth optimization with gradients available.

**How it works.** Build up an approximation to the Hessian (or its inverse) using gradient differences between iterations. Get most of Newton's speed without computing or storing the true Hessian.

**Strengths.** Near-Newton convergence rate without Hessian cost. Very well-tested. Usually the best default for smooth unconstrained problems.

**Weaknesses.** Same smoothness assumption as Newton. Memory scales with `n²` for full BFGS (use L-BFGS if `n > ~1000`).

**Implementations.**
- Python: `scipy.optimize.minimize(f, x0, jac=grad, method='BFGS')` or `'L-BFGS-B'`
- Julia: `Optim.jl` with `BFGS()`, `LBFGS()`

---

## Direct / Derivative-Free Methods

When no gradient is available.

### Nelder-Mead simplex

**When to use.** Low-dimensional (n ≤ ~10), gradient-free, noisy-ish objectives. A reasonable default for black-box problems when you don't know anything else.

**How it works.** Maintain a simplex of `n+1` points. At each step, reflect / expand / contract / shrink the simplex based on which vertex has the worst objective.

**Strengths.** Robust. No gradient needed. Few hyperparameters.

**Weaknesses.** Scales poorly past ~10 dimensions. No convergence guarantee on general problems. Can stagnate; restart occasionally.

**Implementations.**
- Python: `scipy.optimize.minimize(f, x0, method='Nelder-Mead')`
- Julia: `Optim.jl` with `NelderMead()`

### Powell's method

**When to use.** Similar niche to Nelder-Mead: low-dim, gradient-free. Uses line searches along adapted directions.

**Implementations.** `scipy.optimize.minimize(f, x0, method='Powell')`

### Pattern search (Hooke-Jeeves, generalized pattern search, MADS)

**When to use.** Gradient-free, low-to-moderate dimensional, where you want more theoretical guarantees than Nelder-Mead. MADS (Mesh Adaptive Direct Search) is state-of-the-art for this family.

**Implementations.**
- NOMAD (http://nomad-solver.com) — C++ with Python/Julia bindings; implements MADS

### DIRECT (Divided Rectangles)

**When to use.** Low-dimensional (n ≤ ~20), Lipschitz-continuous, global optimization. Gives *global* guarantees up to resolution.

**How it works.** Recursively subdivides the search space into hyperrectangles; samples promising ones based on size and current best value.

**Implementations.**
- Python: `scipy.optimize.direct`
- NLopt: `GN_DIRECT` and variants

---

## Stochastic Methods

Randomness *inside* the algorithm (not just the objective).

### Simulated annealing

**When to use.** Combinatorial problems. Non-convex continuous problems where you need global behavior. Particularly good for combinatorial problems with large search spaces (scheduling, routing, layout).

**How it works.** Propose random moves; accept improvements always, accept worsening moves with probability `exp(-ΔE/T)`. Temperature `T` decreases over time (annealing schedule).

**Strengths.** Simple. Can escape local minima. Works on continuous and discrete problems.

**Weaknesses.** Slow. Many hyperparameters (cooling schedule, proposal distribution). Not competitive with purpose-built methods on structured problems.

**Implementations.**
- Python: `scipy.optimize.dual_annealing` (global optimization variant)
- Most metaheuristic libraries include it

### CMA-ES (Covariance Matrix Adaptation Evolution Strategy)

**When to use.** Continuous black-box optimization, moderate dimension (n up to a few hundred), non-convex, possibly noisy. *The strongest default for continuous gradient-free optimization* when the dimension is too high for Nelder-Mead.

**How it works.** Samples candidate solutions from a multivariate normal distribution. Adapts the distribution's mean and covariance based on which samples were best. Self-tuning.

**Strengths.** Remarkably robust. Handles non-convexity, noise, moderate dimension. Very few hyperparameters in practice.

**Weaknesses.** Slow per iteration (population of ~4+3·log(n) evaluations each generation). Memory `O(n²)` from covariance matrix limits dimension.

**Implementations.**
- Python: `pycma` (`pip install cma`), or `scipy.optimize` via `differential_evolution` as an alternative
- Julia: `Evolutionary.jl`

### Cross-Entropy Method

**When to use.** Rare-event simulation; combinatorial optimization; reinforcement learning. Similar niche to CMA-ES but conceptually distinct.

**How it works.** Fit a parametric distribution over the search space to the "elite" fraction of samples; resample from the updated distribution.

---

## Population Methods

### Genetic algorithms

**When to use.** Combinatorial problems, especially when the structure isn't clearly exploitable by exact methods. Problems with natural "recombination" semantics (permutations, subsets, tree structures).

**How it works.** Maintain a population of candidate solutions; combine (crossover) and mutate them; select survivors by fitness.

**Strengths.** Very flexible. Can handle arbitrary search spaces with custom operators.

**Weaknesses.** Many hyperparameters. Often outperformed by problem-specific methods.

**Implementations.**
- Python: `DEAP` (general-purpose), `PyGAD`
- Julia: `Evolutionary.jl`

### Differential evolution

**When to use.** Continuous global optimization; similar niche to CMA-ES. Particularly robust when dimension is moderate and the landscape is bumpy.

**Implementations.** `scipy.optimize.differential_evolution`

### Particle swarm / firefly / cuckoo / ant colony

**When to use.** Rarely. These get a lot of hype but are usually dominated by differential evolution or CMA-ES on continuous problems, and by simulated annealing or MILP on combinatorial problems. Use if your domain has a specific reason to prefer one (e.g., ant colony for routing on a graph).

---

## Surrogate / Bayesian Optimization

### Bayesian optimization with Gaussian process surrogate

**When to use.** Expensive black-box objectives (minutes to hours per evaluation). Hyperparameter tuning for machine learning models. Experiment design in the lab. Any problem where you can afford 20–200 evaluations total.

**How it works.** Fit a probabilistic surrogate (usually a Gaussian process) to observed `(x, f(x))` pairs. Use an acquisition function (Expected Improvement, Upper Confidence Bound, Probability of Improvement, or Thompson sampling) to propose the next point to evaluate, balancing exploration (high uncertainty) against exploitation (low predicted mean).

**Strengths.** *Ruthlessly sample-efficient.* Can find good solutions in a few dozen evaluations when alternatives need thousands.

**Weaknesses.** GP inference is `O(N³)` in observed points — breaks down past a few thousand evaluations. Doesn't scale past ~20 dimensions without extra structure. Hyperparameter choices matter (kernel, acquisition function).

**Tuning knobs.** Kernel (RBF is default; Matern 5/2 often better). Acquisition function (EI is default). Initial design size (often 10–20).

**Implementations.**
- Python: `scikit-optimize` (`skopt`), `BoTorch` (modern, GPU-accelerated), `GPyOpt`, `Ax` (from Meta)
- **Hyperparameter tuning specifically: `Optuna` is the standard.** It uses TPE (tree-structured Parzen estimator) rather than GP but is built for this exact use case.
- Julia: `BayesianOptimization.jl`

### Random and grid search

Worth mentioning as contrast: **random search** is a strong baseline and often beats grid search when only a few dimensions matter. For hyperparameter tuning, random search is a fine starting point; switch to Bayesian when the evaluation cost justifies the machinery.

---

## Linear Programming

### Simplex / Interior Point

**When to use.** Objective and all constraints are linear in `x`. This is a solved problem.

**Strengths.** Global optimum guaranteed. Extremely fast (millions of variables feasible).

**Implementations.**
- Python: `scipy.optimize.linprog`, or better, `CVXPY` for a cleaner API
- Julia: `JuMP` with any LP solver
- Commercial: Gurobi, CPLEX, MOSEK (free academic licenses). Open-source: HiGHS (fast, modern), CBC, GLPK.

---

## Convex Optimization

### Disciplined Convex Programming

**When to use.** Any convex problem: least-squares, quadratic programming, second-order cone programming, semidefinite programming, linear programming, geometric programming.

**How it works.** You express the problem using a language (CVXPY, Convex.jl, CVX for MATLAB) that knows convex calculus rules. The library verifies convexity and routes to an appropriate solver.

**Strengths.** Guaranteed global optimum. Fast. Clean, declarative problem statement.

**Implementations.**
- Python: `CVXPY` (the de facto standard)
- Julia: `Convex.jl`, `JuMP`
- Solvers: ECOS, SCS, CLARABEL (open-source); MOSEK, Gurobi (commercial)

---

## Constrained Nonlinear Optimization

### SLSQP (Sequential Least Squares Programming)

**When to use.** Smooth, nonlinearly-constrained, small-to-medium problems. A reasonable default when you have general nonlinear constraints and gradients.

**Implementations.** `scipy.optimize.minimize(f, x0, jac=..., method='SLSQP', constraints=...)`

### Trust-region constrained (`trust-constr`)

**When to use.** Larger-scale nonlinearly-constrained problems, or when SLSQP struggles. SciPy's best general-purpose constrained nonlinear solver.

**Implementations.** `scipy.optimize.minimize(f, x0, jac=..., method='trust-constr', constraints=...)`

### IPOPT (Interior Point Optimizer)

**When to use.** Large-scale nonlinear constrained problems. Industry standard for nonlinear programming.

**Implementations.**
- Python: via `Pyomo`, `casadi`, or `cyipopt`
- Julia: `JuMP` with `Ipopt.jl`

### Penalty and augmented Lagrangian methods

**When to use.** When you want to re-use an unconstrained solver on a constrained problem. Roll constraint violation into the objective as a penalty term; solve iteratively.

**Strengths.** Conceptually simple; leverages existing unconstrained solvers.

**Weaknesses.** Penalty parameter tuning can be finicky; convergence can be slow. Augmented Lagrangian is more robust than pure penalty.

---

## Discrete / Integer Optimization

### Mixed-Integer Linear Programming (MILP)

**When to use.** Linear objective and constraints, with some variables required to be integer. Enormous class of real-world problems fall here: scheduling, routing, assignment, cutting-stock, many others.

**How it works.** Branch-and-bound with linear-programming relaxations. Modern solvers are remarkably effective; problems with hundreds of thousands of variables are routinely tractable.

**Implementations.**
- Commercial (very fast, free academic license): Gurobi, CPLEX, Mosek
- Open-source: CBC, HiGHS (fast, active), GLPK (older)
- Modeling: `Pyomo`, `PuLP`, `CVXPY` (in Python); `JuMP` (Julia)

### Branch and bound (general)

For problems where an LP relaxation isn't the right lower bound — e.g., constraint programming for combinatorial puzzles. Libraries: Google OR-Tools (particularly `CP-SAT` — extremely strong).

### Dynamic programming

When the problem has optimal-substructure and overlapping-subproblems structure, DP is often the right answer and gives exact solutions. Knapsack, shortest path, sequence alignment, many scheduling problems.

### Metaheuristics for combinatorial problems

When the problem is too big for exact MILP and has no exploitable structure: simulated annealing, tabu search, genetic algorithms, ant colony (for routing/graph problems).

---

## Multi-Objective Optimization

### Scalarization (weighted sum, ε-constraint, goal programming)

**When to use.** When the user can provide relative weights or priority ordering between objectives. Fast; reduces to single-objective.

**Implementations.** Any single-objective solver after scalarizing.

### Pareto front methods (NSGA-II, MOEA/D, SMS-EMOA)

**When to use.** When the user wants to see the tradeoff curve rather than commit to weights upfront. NSGA-II is the workhorse.

**Implementations.**
- Python: `pymoo` (comprehensive), `DEAP`, `Platypus`
- Julia: `Evolutionary.jl`

---

## Quick reference: "what should I actually call?"

For the common cases:

| Situation | Call this |
|-----------|-----------|
| Smooth + unconstrained + Python | `scipy.optimize.minimize(f, x0, jac=grad, method='L-BFGS-B')` |
| Smooth + box bounds + Python | Same, with `bounds=[(lo, hi), ...]` |
| Smooth + nonlinear constraints + Python | `method='SLSQP'` with `constraints=[...]` |
| Deep learning | `torch.optim.AdamW(lr=1e-3)` |
| Hyperparameter tuning | `Optuna` |
| Expensive black-box | `BoTorch` or `scikit-optimize` |
| Linear program | `CVXPY` or `scipy.optimize.linprog(method='highs')` |
| Convex (quadratic / SOCP / etc) | `CVXPY` |
| Mixed-integer linear | `Pyomo` + HiGHS or Gurobi |
| Global, low-dim, black-box | `scipy.optimize.differential_evolution` or `dual_annealing` |
| Gradient-free, moderate dim | `pycma` (`import cma`) |
| Multi-objective, Pareto front | `pymoo` with NSGA-II |

When you don't recognize the situation on sight, go back to `problem-classification.md`, answer the 10 questions, and work from there. That's more efficient than guessing which method to try first.
