# Algorithms

Pedagogical Python implementations of the most practically useful algorithms from the book. **These are for understanding, not production.** For real work, use the library implementations listed in `methods.md` — they handle numerical stability, edge cases, convergence criteria, and scale in ways these simple versions don't.

All code assumes NumPy is imported as `np`. No scipy, no torch — the point is to show the algorithm's logic. Where an algorithm takes a random number generator, it uses `np.random.default_rng()`.

```python
import numpy as np
from typing import Callable
```

---

## Common utilities

A few test functions from the book's Appendix B, used throughout. Useful both for trying the algorithms and as sanity-checks.

```python
def sphere(x: np.ndarray) -> float:
    """Sphere function: f(x) = sum(x_i^2). Convex. Minimum 0 at origin."""
    return float(np.sum(x**2))

def sphere_grad(x: np.ndarray) -> np.ndarray:
    return 2.0 * x

def rosenbrock(x: np.ndarray, a: float = 1.0, b: float = 5.0) -> float:
    """Rosenbrock's banana function. Non-convex, narrow curved valley.
    Minimum 0 at (a, a^2). Book uses b=5; classical version uses b=100."""
    return (a - x[0])**2 + b * (x[1] - x[0]**2)**2

def rosenbrock_grad(x: np.ndarray, a: float = 1.0, b: float = 5.0) -> np.ndarray:
    return np.array([
        -2.0 * (a - x[0]) - 4.0 * b * x[0] * (x[1] - x[0]**2),
        2.0 * b * (x[1] - x[0]**2),
    ])

def ackley(x: np.ndarray, a: float = 20.0, b: float = 0.2,
           c: float = 2.0 * np.pi) -> float:
    """Ackley's function: many local minima, global minimum 0 at origin."""
    n = len(x)
    s1 = np.sum(x**2) / n
    s2 = np.sum(np.cos(c * x)) / n
    return float(-a * np.exp(-b * np.sqrt(s1)) - np.exp(s2) + a + np.e)
```

---

## 1. Derivatives

The starting point for every gradient-based method. In production, use automatic differentiation (JAX, PyTorch) rather than these. Finite differences are for sanity-checking AD or when AD is impossible (black-box function).

### Forward and central finite differences

```python
def forward_difference(f: Callable[[np.ndarray], float],
                       x: np.ndarray, h: float = 1e-6) -> np.ndarray:
    """Approximate gradient by forward finite differences. Costs n+1 evals.
    Error is O(h). Simple but inaccurate."""
    n = len(x)
    grad = np.zeros(n)
    fx = f(x)
    for i in range(n):
        x_p = x.copy()
        x_p[i] += h
        grad[i] = (f(x_p) - fx) / h
    return grad

def central_difference(f: Callable[[np.ndarray], float],
                       x: np.ndarray, h: float = 1e-6) -> np.ndarray:
    """Approximate gradient by central differences. Costs 2n evals.
    Error is O(h^2). Worth the extra cost in almost every case."""
    n = len(x)
    grad = np.zeros(n)
    for i in range(n):
        x_p, x_m = x.copy(), x.copy()
        x_p[i] += h
        x_m[i] -= h
        grad[i] = (f(x_p) - f(x_m)) / (2.0 * h)
    return grad
```

### Complex step derivative

```python
def complex_step_derivative(f: Callable[[np.ndarray], complex],
                            x: np.ndarray, h: float = 1e-20) -> np.ndarray:
    """Complex-step derivative. Exact to machine precision with no subtractive
    cancellation. Requires f to accept complex inputs and return a complex
    result: any float() cast, abs() on intermediates, or if-statements on
    non-complex-valued conditions will silently break it. Elegant when it works.
    Test with f = lambda x: np.sum(x**2) (not float(np.sum(x**2)))."""
    n = len(x)
    grad = np.zeros(n)
    for i in range(n):
        x_c = x.astype(complex)
        x_c[i] += 1j * h
        grad[i] = f(x_c).imag / h
    return grad
```

---

## 2. Bracketing (1D minimization)

Before you can search for a minimum of a univariate function, you need an interval that contains one. Then you refine.

### Bracket a minimum

```python
def bracket_minimum(f: Callable[[float], float], x: float = 0.0,
                    s: float = 1e-2, k: float = 2.0) -> tuple[float, float]:
    """Find an interval (a, b) containing a local minimum of f.
    Walk in the direction that decreases f, growing step size by factor k each time."""
    a, ya = x, f(x)
    b, yb = a + s, f(a + s)
    if yb > ya:
        a, b = b, a
        ya, yb = yb, ya
        s = -s
    while True:
        c, yc = b + s, f(b + s)
        if yc > yb:
            return (a, c) if a < c else (c, a)
        a, ya = b, yb
        b, yb = c, yc
        s *= k
```

### Golden section search

```python
def golden_section_search(f: Callable[[float], float], a: float, b: float,
                          n: int = 50) -> tuple[float, float]:
    """Shrink a bracket (a, b) around a unimodal minimum using the golden ratio.
    Each iteration reduces the interval by factor ~0.618 with one new evaluation."""
    phi = (1.0 + np.sqrt(5.0)) / 2.0
    rho = phi - 1.0  # ~0.618
    d = rho * b + (1.0 - rho) * a
    yd = f(d)
    for _ in range(n - 1):
        c = rho * a + (1.0 - rho) * b
        yc = f(c)
        if yc < yd:
            b, d, yd = d, c, yc
        else:
            a, b = b, c
    return (a, b) if a < b else (b, a)
```

### Bisection (find a root of f', i.e. a stationary point)

```python
def bisection(f_prime: Callable[[float], float], a: float, b: float,
              eps: float = 1e-6) -> tuple[float, float]:
    """Bisection on the derivative to find a stationary point.
    Requires f_prime(a) and f_prime(b) to have opposite signs."""
    ya, yb = f_prime(a), f_prime(b)
    assert ya * yb < 0, "f_prime(a) and f_prime(b) must have opposite signs"
    while (b - a) > eps:
        m = (a + b) / 2.0
        ym = f_prime(m)
        if ym * ya > 0:
            a, ya = m, ym
        else:
            b, yb = m, ym
    return a, b
```

---

## 3. Line search

A subroutine used inside multi-dimensional methods: given a current point `x` and a descent direction `d`, find a step size `α > 0` such that `f(x + α d)` is sufficiently smaller than `f(x)`.

### Backtracking line search (Armijo condition)

```python
def backtracking_line_search(f: Callable[[np.ndarray], float], grad_f: Callable,
                             x: np.ndarray, d: np.ndarray,
                             alpha: float = 1.0, beta: float = 0.5,
                             c1: float = 1e-4) -> float:
    """Halve the step size until the Armijo sufficient-decrease condition holds:
        f(x + α d) <= f(x) + c1 α ∇f(x)·d
    Standard cheap-and-reliable line search for gradient-based methods."""
    fx = f(x)
    gx = grad_f(x)
    slope = float(gx @ d)
    while f(x + alpha * d) > fx + c1 * alpha * slope:
        alpha *= beta
        if alpha < 1e-16:
            break
    return alpha
```

---

## 4. First-order methods

Use only the gradient. Strong defaults when the objective is smooth and gradients are cheap.

### Gradient descent

```python
def gradient_descent(f: Callable, grad_f: Callable, x0: np.ndarray,
                     lr: float = 0.01, max_iter: int = 1000,
                     tol: float = 1e-6) -> np.ndarray:
    """Simplest first-order method. Step in the negative-gradient direction.
    Fixed learning rate. Slow on ill-conditioned problems (zig-zags)."""
    x = x0.copy()
    for _ in range(max_iter):
        g = grad_f(x)
        if np.linalg.norm(g) < tol:
            break
        x = x - lr * g
    return x
```

### Conjugate gradient (Fletcher-Reeves)

```python
def conjugate_gradient(f: Callable, grad_f: Callable, x0: np.ndarray,
                       max_iter: int = 1000, tol: float = 1e-6) -> np.ndarray:
    """Build descent directions that are conjugate to previous ones, avoiding
    the zig-zag of steepest descent. Fletcher-Reeves variant of β. Converges
    in at most n iterations for a convex quadratic."""
    x = x0.copy()
    g = grad_f(x)
    d = -g
    for _ in range(max_iter):
        if np.linalg.norm(g) < tol:
            break
        alpha = backtracking_line_search(f, grad_f, x, d)
        x = x + alpha * d
        g_new = grad_f(x)
        beta = float(g_new @ g_new) / max(float(g @ g), 1e-16)
        d = -g_new + beta * d
        g = g_new
    return x
```

### Momentum (heavy ball)

```python
def gradient_descent_with_momentum(grad_f: Callable, x0: np.ndarray,
                                   lr: float = 0.01, momentum: float = 0.9,
                                   max_iter: int = 1000, tol: float = 1e-6) -> np.ndarray:
    """Accumulate a velocity vector from past gradients. Smooths noisy updates
    and accelerates through narrow valleys."""
    x = x0.copy()
    v = np.zeros_like(x)
    for _ in range(max_iter):
        g = grad_f(x)
        if np.linalg.norm(g) < tol:
            break
        v = momentum * v - lr * g
        x = x + v
    return x
```

### Nesterov momentum

```python
def nesterov_momentum(grad_f: Callable, x0: np.ndarray,
                      lr: float = 0.01, momentum: float = 0.9,
                      max_iter: int = 1000, tol: float = 1e-6) -> np.ndarray:
    """Like momentum, but evaluate the gradient at the *lookahead* point
    x + momentum·v. Gets a small but reliable speedup on convex problems."""
    x = x0.copy()
    v = np.zeros_like(x)
    for _ in range(max_iter):
        lookahead = x + momentum * v
        g = grad_f(lookahead)
        if np.linalg.norm(g) < tol:
            break
        v = momentum * v - lr * g
        x = x + v
    return x
```

### Adagrad

```python
def adagrad(grad_f: Callable, x0: np.ndarray,
            lr: float = 0.1, eps: float = 1e-8,
            max_iter: int = 1000, tol: float = 1e-6) -> np.ndarray:
    """Per-coordinate learning rate that shrinks based on sum of squared past
    gradients for that coordinate. Good for sparse features; can stall as the
    accumulator grows without bound."""
    x = x0.copy()
    accum = np.zeros_like(x)
    for _ in range(max_iter):
        g = grad_f(x)
        if np.linalg.norm(g) < tol:
            break
        accum += g * g
        x = x - lr * g / (np.sqrt(accum) + eps)
    return x
```

### RMSProp

```python
def rmsprop(grad_f: Callable, x0: np.ndarray,
            lr: float = 0.01, decay: float = 0.9, eps: float = 1e-8,
            max_iter: int = 1000, tol: float = 1e-6) -> np.ndarray:
    """Like Adagrad but uses an *exponentially weighted* running average of
    squared gradients, so the learning rate doesn't decay to zero."""
    x = x0.copy()
    v = np.zeros_like(x)
    for _ in range(max_iter):
        g = grad_f(x)
        if np.linalg.norm(g) < tol:
            break
        v = decay * v + (1.0 - decay) * g * g
        x = x - lr * g / (np.sqrt(v) + eps)
    return x
```

### Adam

```python
def adam(grad_f: Callable, x0: np.ndarray,
         lr: float = 1e-3, beta1: float = 0.9, beta2: float = 0.999,
         eps: float = 1e-8, max_iter: int = 1000,
         tol: float = 1e-6) -> np.ndarray:
    """RMSProp-style scale adaptation combined with momentum, plus bias
    correction. Defaults (1e-3, 0.9, 0.999) work remarkably often.
    The de facto default for deep learning."""
    x = x0.copy()
    m = np.zeros_like(x)
    v = np.zeros_like(x)
    for t in range(1, max_iter + 1):
        g = grad_f(x)
        if np.linalg.norm(g) < tol:
            break
        m = beta1 * m + (1.0 - beta1) * g
        v = beta2 * v + (1.0 - beta2) * g * g
        m_hat = m / (1.0 - beta1**t)
        v_hat = v / (1.0 - beta2**t)
        x = x - lr * m_hat / (np.sqrt(v_hat) + eps)
    return x
```

---

## 5. Second-order and quasi-Newton methods

Use curvature information in addition to the gradient. Converge much faster on smooth problems, at the cost of Hessian storage (full Newton) or history storage (L-BFGS).

### Newton's method

```python
def newtons_method(grad_f: Callable, hess_f: Callable, x0: np.ndarray,
                   max_iter: int = 100, tol: float = 1e-6) -> np.ndarray:
    """Use both gradient and Hessian. Step to the minimum of the local
    quadratic approximation. Quadratic convergence near the optimum.
    Assumes Hessian is positive definite; regularize or use trust region
    if not (see book ch. 4)."""
    x = x0.copy()
    for _ in range(max_iter):
        g = grad_f(x)
        if np.linalg.norm(g) < tol:
            break
        H = hess_f(x)
        try:
            step = np.linalg.solve(H, g)
        except np.linalg.LinAlgError:
            step = g  # fall back to gradient step if Hessian singular
        x = x - step
    return x
```

### BFGS (full quasi-Newton)

```python
def bfgs(f: Callable, grad_f: Callable, x0: np.ndarray,
         max_iter: int = 1000, tol: float = 1e-6) -> np.ndarray:
    """Build an approximation to the *inverse* Hessian from gradient differences.
    Storage O(n^2). For n > ~1000, use L-BFGS instead."""
    n = len(x0)
    x = x0.copy()
    H_inv = np.eye(n)
    g = grad_f(x)
    for _ in range(max_iter):
        if np.linalg.norm(g) < tol:
            break
        d = -H_inv @ g
        alpha = backtracking_line_search(f, grad_f, x, d)
        s = alpha * d
        x_new = x + s
        g_new = grad_f(x_new)
        y = g_new - g
        sy = float(s @ y)
        if sy > 1e-10:  # skip update if curvature condition violated
            rho = 1.0 / sy
            I = np.eye(n)
            V = I - rho * np.outer(s, y)
            H_inv = V @ H_inv @ V.T + rho * np.outer(s, s)
        x, g = x_new, g_new
    return x
```

### L-BFGS (limited-memory BFGS)

```python
def lbfgs(f: Callable, grad_f: Callable, x0: np.ndarray,
          m: int = 10, max_iter: int = 1000, tol: float = 1e-6) -> np.ndarray:
    """L-BFGS uses a two-loop recursion to compute the search direction using
    only the last m (s, y) pairs, without storing any matrix. Scales to
    millions of variables. Strongest default for smooth unconstrained problems
    when n is large."""
    x = x0.copy()
    g = grad_f(x)
    s_hist, y_hist, rho_hist = [], [], []
    for _ in range(max_iter):
        if np.linalg.norm(g) < tol:
            break
        # Two-loop recursion to compute H_k g
        q = g.copy()
        alphas = []
        for s_i, y_i, rho_i in zip(reversed(s_hist), reversed(y_hist), reversed(rho_hist)):
            a = rho_i * float(s_i @ q)
            q = q - a * y_i
            alphas.append(a)
        if s_hist:
            gamma = float(s_hist[-1] @ y_hist[-1]) / float(y_hist[-1] @ y_hist[-1])
        else:
            gamma = 1.0
        r = gamma * q
        for s_i, y_i, rho_i, a in zip(s_hist, y_hist, rho_hist, reversed(alphas)):
            b = rho_i * float(y_i @ r)
            r = r + (a - b) * s_i
        d = -r
        alpha = backtracking_line_search(f, grad_f, x, d)
        s = alpha * d
        x_new = x + s
        g_new = grad_f(x_new)
        y = g_new - g
        sy = float(s @ y)
        if sy > 1e-10:
            s_hist.append(s)
            y_hist.append(y)
            rho_hist.append(1.0 / sy)
            if len(s_hist) > m:
                s_hist.pop(0); y_hist.pop(0); rho_hist.pop(0)
        x, g = x_new, g_new
    return x
```

---

## 6. Direct (derivative-free) methods

Use only function values. Necessary when gradients are unavailable or unreliable.

### Nelder-Mead simplex

```python
def nelder_mead(f: Callable, x0: np.ndarray, step: float = 0.1,
                max_iter: int = 1000, tol: float = 1e-6,
                alpha: float = 1.0, gamma: float = 2.0,
                rho: float = 0.5, sigma: float = 0.5) -> np.ndarray:
    """Maintain a simplex of n+1 points; reflect, expand, contract, or shrink
    based on which vertex is worst. No gradient. Struggles past n~10 dims."""
    n = len(x0)
    # Initial simplex: x0 and n perturbations
    simplex = [x0.copy()]
    for i in range(n):
        p = x0.copy()
        p[i] += step
        simplex.append(p)
    simplex = np.array(simplex)
    fvals = np.array([f(p) for p in simplex])

    for _ in range(max_iter):
        # Sort by f value (best first)
        order = np.argsort(fvals)
        simplex, fvals = simplex[order], fvals[order]
        if np.std(fvals) < tol:
            break
        # Centroid of all but worst
        centroid = simplex[:-1].mean(axis=0)
        worst = simplex[-1]
        # Reflection
        xr = centroid + alpha * (centroid - worst)
        fr = f(xr)
        if fvals[0] <= fr < fvals[-2]:
            simplex[-1], fvals[-1] = xr, fr
            continue
        # Expansion
        if fr < fvals[0]:
            xe = centroid + gamma * (xr - centroid)
            fe = f(xe)
            if fe < fr:
                simplex[-1], fvals[-1] = xe, fe
            else:
                simplex[-1], fvals[-1] = xr, fr
            continue
        # Contraction
        xc = centroid + rho * (worst - centroid)
        fc = f(xc)
        if fc < fvals[-1]:
            simplex[-1], fvals[-1] = xc, fc
            continue
        # Shrink toward best
        best = simplex[0]
        simplex = best + sigma * (simplex - best)
        fvals = np.array([f(p) for p in simplex])

    return simplex[np.argmin(fvals)]
```

### Hooke-Jeeves pattern search

```python
def hooke_jeeves(f: Callable, x0: np.ndarray, step: float = 0.5,
                 gamma: float = 0.5, tol: float = 1e-6,
                 max_iter: int = 1000) -> np.ndarray:
    """Exploratory moves along each coordinate axis, shrinking the step size
    when no direction improves. Simple, robust, derivative-free."""
    x = x0.copy()
    n = len(x)
    fx = f(x)
    for _ in range(max_iter):
        if step < tol:
            break
        improved = False
        for i in range(n):
            for sign in (+1, -1):
                trial = x.copy()
                trial[i] += sign * step
                ft = f(trial)
                if ft < fx:
                    x, fx = trial, ft
                    improved = True
                    break  # coordinate done
        if not improved:
            step *= gamma
    return x
```

---

## 7. Stochastic methods

Introduce randomness into the search. Useful for non-convex landscapes and when you need to escape local minima.

### Simulated annealing

```python
def simulated_annealing(f: Callable, x0: np.ndarray, T0: float = 10.0,
                        decay: float = 0.99, proposal_scale: float = 1.0,
                        max_iter: int = 10000,
                        rng: np.random.Generator = None) -> np.ndarray:
    """Metropolis-Hastings with a cooling temperature. Accept improvements
    always; accept worsening moves with probability exp(-ΔE/T).
    T decreases geometrically."""
    rng = rng or np.random.default_rng()
    x = x0.copy()
    fx = f(x)
    best, f_best = x.copy(), fx
    T = T0
    for _ in range(max_iter):
        x_new = x + rng.normal(scale=proposal_scale, size=x.shape)
        f_new = f(x_new)
        dE = f_new - fx
        if dE < 0 or rng.random() < np.exp(-dE / max(T, 1e-12)):
            x, fx = x_new, f_new
            if fx < f_best:
                best, f_best = x.copy(), fx
        T *= decay
    return best
```

### Cross-entropy method

```python
def cross_entropy_method(f: Callable, mean: np.ndarray, std: np.ndarray,
                         n_samples: int = 100, elite_frac: float = 0.2,
                         n_iter: int = 50,
                         rng: np.random.Generator = None) -> np.ndarray:
    """Sample a population from N(mean, std^2); fit a new distribution
    to the best elite_frac of samples; repeat. Converges on good basins."""
    rng = rng or np.random.default_rng()
    mean = mean.astype(float).copy()
    std = std.astype(float).copy()
    n_elite = max(1, int(elite_frac * n_samples))
    for _ in range(n_iter):
        samples = rng.normal(mean, std, size=(n_samples, len(mean)))
        scores = np.array([f(s) for s in samples])
        elite_idx = np.argsort(scores)[:n_elite]
        elite = samples[elite_idx]
        mean = elite.mean(axis=0)
        std = elite.std(axis=0) + 1e-8  # avoid collapse to zero
    return mean
```

### CMA-ES (simplified)

```python
def cma_es(f: Callable, mean: np.ndarray, sigma: float = 0.5,
           pop_size: int = None, n_iter: int = 100,
           rng: np.random.Generator = None) -> np.ndarray:
    """Simplified Covariance Matrix Adaptation Evolution Strategy.
    Full CMA-ES includes evolution paths and cumulative step-size control
    (see pycma for a real implementation). This version does the core ideas:
    rank-μ update of C, weighted-recombination mean update, simple sigma
    adaptation by success rate."""
    rng = rng or np.random.default_rng()
    n = len(mean)
    if pop_size is None:
        pop_size = 4 + int(3 * np.log(n))
    mu = pop_size // 2  # number of elite
    # Log-scaled weights for the elite
    weights = np.log(mu + 0.5) - np.log(np.arange(1, mu + 1))
    weights /= weights.sum()
    mu_eff = 1.0 / np.sum(weights**2)
    # Learning rates (simplified)
    c_sigma = (mu_eff + 2) / (n + mu_eff + 5)
    c_c = 4.0 / (n + 4.0)
    c_1 = 2.0 / ((n + 1.3)**2 + mu_eff)
    c_mu = min(1.0 - c_1,
               2 * (mu_eff - 2 + 1/mu_eff) / ((n + 2)**2 + mu_eff))

    mean = mean.astype(float).copy()
    C = np.eye(n)
    pc = np.zeros(n)  # evolution path for C
    best, f_best = mean.copy(), f(mean)

    for _ in range(n_iter):
        # Sample pop_size candidates from N(mean, sigma^2 * C)
        try:
            L = np.linalg.cholesky(C + 1e-10 * np.eye(n))
        except np.linalg.LinAlgError:
            C = np.eye(n)
            L = np.eye(n)
        z = rng.standard_normal((pop_size, n))
        y = z @ L.T  # y_i ~ N(0, C)
        samples = mean + sigma * y
        scores = np.array([f(s) for s in samples])

        # Sort; take elite
        idx = np.argsort(scores)
        y_elite = y[idx[:mu]]
        mean_old = mean.copy()
        mean = mean_old + sigma * (weights @ y_elite)

        # Track best ever
        if scores[idx[0]] < f_best:
            best, f_best = samples[idx[0]].copy(), scores[idx[0]]

        # Update evolution path and covariance
        y_w = weights @ y_elite
        pc = (1 - c_c) * pc + np.sqrt(c_c * (2 - c_c) * mu_eff) * y_w
        # Rank-1 + rank-mu update of C
        rank_mu = sum(w * np.outer(y_i, y_i) for w, y_i in zip(weights, y_elite))
        C = ((1 - c_1 - c_mu) * C
             + c_1 * np.outer(pc, pc)
             + c_mu * rank_mu)

        # Simple sigma adaptation: grow/shrink based on success
        sigma *= np.exp((c_sigma / 2) * (np.linalg.norm(pc)**2 / n - 1))

    return best
```

---

## 8. Population methods

Maintain a population of candidate solutions that collectively explore the search space.

### Genetic algorithm

```python
def genetic_algorithm(f: Callable, x0: np.ndarray, pop_size: int = 50,
                      n_gen: int = 100, mutation_rate: float = 0.1,
                      mutation_scale: float = 0.3, elite_frac: float = 0.1,
                      rng: np.random.Generator = None) -> np.ndarray:
    """Maintain a population; each generation, select parents by fitness,
    produce children via crossover, mutate some, keep elite unchanged.
    Generic real-valued GA. Swap operators for different search spaces."""
    rng = rng or np.random.default_rng()
    n = len(x0)
    # Initial population centered on x0
    pop = x0 + rng.normal(scale=1.0, size=(pop_size, n))
    n_elite = max(1, int(elite_frac * pop_size))

    for _ in range(n_gen):
        scores = np.array([f(ind) for ind in pop])
        idx = np.argsort(scores)
        pop = pop[idx]
        elite = pop[:n_elite]
        # Tournament selection on the rest
        new_pop = [e.copy() for e in elite]
        while len(new_pop) < pop_size:
            i, j = rng.integers(0, pop_size, size=2)
            parent_a = pop[min(i, j)]  # smaller index = better
            k, l = rng.integers(0, pop_size, size=2)
            parent_b = pop[min(k, l)]
            # Uniform crossover
            mask = rng.random(n) < 0.5
            child = np.where(mask, parent_a, parent_b)
            # Gaussian mutation
            if rng.random() < mutation_rate:
                child = child + rng.normal(scale=mutation_scale, size=n)
            new_pop.append(child)
        pop = np.array(new_pop)

    scores = np.array([f(ind) for ind in pop])
    return pop[np.argmin(scores)]
```

### Differential evolution

```python
def differential_evolution(f: Callable, bounds: np.ndarray,
                           pop_size: int = 20, F: float = 0.8,
                           CR: float = 0.9, n_iter: int = 200,
                           rng: np.random.Generator = None) -> np.ndarray:
    """DE/rand/1/bin. For each member x_i of the population, propose
    x_i' = x_a + F·(x_b - x_c) for three other random members;
    crossover with x_i using rate CR; keep x_i' if it's better.
    Remarkably robust; often outperforms GA on real-valued problems.
    bounds: array of shape (n, 2) giving [low, high] per dim."""
    rng = rng or np.random.default_rng()
    lo, hi = bounds[:, 0], bounds[:, 1]
    n = len(lo)
    # Initial population uniform in bounds
    pop = lo + rng.random((pop_size, n)) * (hi - lo)
    scores = np.array([f(ind) for ind in pop])

    for _ in range(n_iter):
        for i in range(pop_size):
            # Pick three distinct others
            candidates = [k for k in range(pop_size) if k != i]
            a, b, c = pop[rng.choice(candidates, 3, replace=False)]
            mutant = a + F * (b - c)
            # Binomial crossover
            mask = rng.random(n) < CR
            mask[rng.integers(n)] = True  # ensure at least one component
            trial = np.where(mask, mutant, pop[i])
            trial = np.clip(trial, lo, hi)
            ft = f(trial)
            if ft < scores[i]:
                pop[i], scores[i] = trial, ft
    return pop[np.argmin(scores)]
```

### Particle swarm optimization

```python
def particle_swarm(f: Callable, bounds: np.ndarray, pop_size: int = 30,
                   n_iter: int = 200, w: float = 0.7, c1: float = 1.5,
                   c2: float = 1.5,
                   rng: np.random.Generator = None) -> np.ndarray:
    """Each particle has position and velocity, attracted to its personal
    best and to the swarm's global best. Momentum term w prevents premature
    convergence; c1/c2 balance self-confidence vs herd behavior."""
    rng = rng or np.random.default_rng()
    lo, hi = bounds[:, 0], bounds[:, 1]
    n = len(lo)
    # Init positions uniform; velocities small
    pos = lo + rng.random((pop_size, n)) * (hi - lo)
    vel = rng.normal(scale=0.1, size=(pop_size, n))
    scores = np.array([f(p) for p in pos])
    p_best = pos.copy()
    p_best_score = scores.copy()
    g_best = p_best[np.argmin(p_best_score)].copy()
    g_best_score = p_best_score.min()

    for _ in range(n_iter):
        r1 = rng.random((pop_size, n))
        r2 = rng.random((pop_size, n))
        vel = (w * vel
               + c1 * r1 * (p_best - pos)
               + c2 * r2 * (g_best - pos))
        pos = np.clip(pos + vel, lo, hi)
        scores = np.array([f(p) for p in pos])
        improved = scores < p_best_score
        p_best[improved] = pos[improved]
        p_best_score[improved] = scores[improved]
        if p_best_score.min() < g_best_score:
            g_best = p_best[np.argmin(p_best_score)].copy()
            g_best_score = p_best_score.min()
    return g_best
```

---

## 9. Constrained optimization

Transform a constrained problem into one (or a sequence of) unconstrained problems.

### Penalty method

```python
def penalty_method(f: Callable, g_ineq: list, h_eq: list, x0: np.ndarray,
                   rho: float = 1.0, growth: float = 10.0, n_outer: int = 20,
                   unconstrained_solver: Callable = None) -> np.ndarray:
    """Solve a sequence of unconstrained problems, each adding a quadratic
    penalty for constraint violation. Rho grows each round to tighten
    enforcement. g_ineq: list of callables with g(x) <= 0 desired.
    h_eq: list of callables with h(x) == 0 desired."""
    unconstrained_solver = unconstrained_solver or (
        lambda F, x: nelder_mead(F, x, max_iter=500))

    def penalized(x, rho):
        p = 0.0
        for g in g_ineq:
            p += max(0.0, g(x))**2
        for h in h_eq:
            p += h(x)**2
        return f(x) + rho * p

    x = x0.copy()
    for _ in range(n_outer):
        x = unconstrained_solver(lambda xx: penalized(xx, rho), x)
        rho *= growth
    return x
```

### Augmented Lagrangian

```python
def augmented_lagrangian(f: Callable, h_eq: list, x0: np.ndarray,
                         rho: float = 1.0, growth: float = 2.0,
                         n_outer: int = 20,
                         unconstrained_solver: Callable = None) -> np.ndarray:
    """For equality-constrained problems. More robust than pure penalty because
    the Lagrange multipliers λ are explicitly updated, which means rho doesn't
    need to go to infinity for convergence."""
    unconstrained_solver = unconstrained_solver or (
        lambda F, x: nelder_mead(F, x, max_iter=500))
    lambdas = np.zeros(len(h_eq))

    def augmented(x, lambdas, rho):
        val = f(x)
        for i, h in enumerate(h_eq):
            hv = h(x)
            val += lambdas[i] * hv + 0.5 * rho * hv**2
        return val

    x = x0.copy()
    for _ in range(n_outer):
        x = unconstrained_solver(lambda xx: augmented(xx, lambdas, rho), x)
        # Update multipliers using current constraint violations
        for i, h in enumerate(h_eq):
            lambdas[i] += rho * h(x)
        rho *= growth
    return x
```

---

## 10. Multi-objective optimization

### Weighted sum scalarization

```python
def weighted_sum(objectives: list, weights: np.ndarray,
                 x0: np.ndarray,
                 solver: Callable = None) -> np.ndarray:
    """Combine multiple objectives f_i(x) into a single scalar f(x) = Σ w_i f_i(x).
    Fast, but only finds solutions on the convex hull of the Pareto front;
    misses concave regions. Caller normalizes objectives first, typically."""
    solver = solver or (lambda F, x: nelder_mead(F, x, max_iter=500))
    def combined(x):
        return float(sum(w * obj(x) for w, obj in zip(weights, objectives)))
    return solver(combined, x0)
```

### Nondominated sorting (core of NSGA-II)

```python
def dominates(a: np.ndarray, b: np.ndarray) -> bool:
    """a dominates b if a is <= b in every objective, and strictly < in at least one."""
    return bool(np.all(a <= b) and np.any(a < b))

def nondominated_sort(objective_values: np.ndarray) -> list[list[int]]:
    """Sort a population into Pareto fronts. objective_values: (n_pop, n_obj).
    Returns list of fronts; front 0 is the Pareto-optimal set, front 1 is
    what you'd get by removing front 0, etc."""
    n = len(objective_values)
    S = [[] for _ in range(n)]       # S[i] = individuals that i dominates
    dom_count = [0] * n              # number of individuals dominating i
    fronts = [[]]

    for p in range(n):
        for q in range(n):
            if p == q:
                continue
            if dominates(objective_values[p], objective_values[q]):
                S[p].append(q)
            elif dominates(objective_values[q], objective_values[p]):
                dom_count[p] += 1
        if dom_count[p] == 0:
            fronts[0].append(p)

    i = 0
    while fronts[i]:
        next_front = []
        for p in fronts[i]:
            for q in S[p]:
                dom_count[q] -= 1
                if dom_count[q] == 0:
                    next_front.append(q)
        i += 1
        fronts.append(next_front)
    return fronts[:-1]  # last one is empty

def crowding_distance(objective_values: np.ndarray,
                      front: list[int]) -> np.ndarray:
    """Assign a crowding distance to each solution in a single front, used
    by NSGA-II to preserve diversity along the Pareto front.
    Endpoints get infinity; interior points get the sum of normalized
    neighbor distances across objectives."""
    f = objective_values[front]
    n_pts, n_obj = f.shape
    dist = np.zeros(n_pts)
    for m in range(n_obj):
        order = np.argsort(f[:, m])
        dist[order[0]] = np.inf
        dist[order[-1]] = np.inf
        span = f[order[-1], m] - f[order[0], m]
        if span < 1e-12:
            continue
        for i in range(1, n_pts - 1):
            dist[order[i]] += (f[order[i+1], m] - f[order[i-1], m]) / span
    return dist
```

NSGA-II combines these with a GA loop: evaluate all objectives, nondominated sort, select by (front, then crowding distance), breed a new generation with standard GA operators, and iterate. See `pymoo` for a production implementation.

---

## 11. Sampling

### Latin hypercube sampling

```python
def latin_hypercube(n_samples: int, n_dims: int,
                    rng: np.random.Generator = None) -> np.ndarray:
    """LHS: divide each axis into n_samples equal intervals; pick one sample
    from each interval per axis; shuffle independently. Much better coverage
    than pure random sampling, especially in low sample counts.
    Returns samples in [0, 1]^n_dims; caller scales to the real domain."""
    rng = rng or np.random.default_rng()
    samples = np.zeros((n_samples, n_dims))
    for d in range(n_dims):
        # One point per stratum, randomly placed within each
        points = (np.arange(n_samples) + rng.random(n_samples)) / n_samples
        rng.shuffle(points)
        samples[:, d] = points
    return samples
```

---

## 12. Surrogate models and Bayesian optimization

For expensive objectives (seconds-to-minutes per eval), fit a cheap surrogate and use it to pick the next expensive evaluation wisely.

### Gaussian process regression

```python
def rbf_kernel(X1: np.ndarray, X2: np.ndarray,
               length_scale: float = 1.0, sigma_f: float = 1.0) -> np.ndarray:
    """Radial Basis Function (squared exponential) kernel.
    k(x, x') = sigma_f^2 · exp(-||x - x'||^2 / (2·length_scale^2))"""
    sq = np.sum(X1**2, axis=1)[:, None] + np.sum(X2**2, axis=1)[None, :] - 2 * X1 @ X2.T
    sq = np.maximum(sq, 0.0)
    return sigma_f**2 * np.exp(-0.5 * sq / length_scale**2)

class GaussianProcess:
    """Minimal GP regressor. Fit once, then call predict repeatedly.
    Not optimized: O(N^3) to fit, O(N^2) per prediction. Production GPs
    (GPyTorch, scikit-learn, BoTorch) handle hyperparameter optimization,
    numerical stability (Cholesky jitter, etc.), and GPU acceleration."""

    def __init__(self, length_scale: float = 1.0, sigma_f: float = 1.0,
                 sigma_n: float = 1e-4):
        self.length_scale = length_scale
        self.sigma_f = sigma_f
        self.sigma_n = sigma_n

    def fit(self, X: np.ndarray, y: np.ndarray):
        self.X = np.asarray(X, dtype=float)
        self.y = np.asarray(y, dtype=float)
        N = len(X)
        K = rbf_kernel(self.X, self.X, self.length_scale, self.sigma_f)
        K += self.sigma_n**2 * np.eye(N)
        self.L = np.linalg.cholesky(K + 1e-10 * np.eye(N))
        self.alpha = np.linalg.solve(self.L.T,
                                     np.linalg.solve(self.L, self.y))
        return self

    def predict(self, X_star: np.ndarray) -> tuple[np.ndarray, np.ndarray]:
        X_star = np.atleast_2d(X_star)
        K_s = rbf_kernel(self.X, X_star, self.length_scale, self.sigma_f)
        K_ss = rbf_kernel(X_star, X_star, self.length_scale, self.sigma_f)
        mean = K_s.T @ self.alpha
        v = np.linalg.solve(self.L, K_s)
        var = np.diag(K_ss) - np.sum(v**2, axis=0)
        var = np.maximum(var, 1e-12)
        return mean, np.sqrt(var)
```

### Expected Improvement acquisition

```python
def expected_improvement(gp: GaussianProcess, X_candidates: np.ndarray,
                         y_best: float, xi: float = 0.01) -> np.ndarray:
    """EI(x) = E[max(y_best - f(x) - xi, 0)] under the GP posterior.
    Closed form for Gaussian posterior.
    xi is an exploration parameter (small > 0 nudges toward exploration)."""
    from math import erf
    mu, sigma = gp.predict(X_candidates)
    sigma = np.maximum(sigma, 1e-12)
    imp = y_best - mu - xi
    z = imp / sigma
    # Normal CDF and PDF
    cdf = 0.5 * (1.0 + np.vectorize(erf)(z / np.sqrt(2.0)))
    pdf = np.exp(-0.5 * z**2) / np.sqrt(2.0 * np.pi)
    ei = imp * cdf + sigma * pdf
    ei[imp < 0] = 0.0  # no improvement possible for very bad predictions
    return ei
```

### Bayesian optimization loop

```python
def bayesian_optimization(f: Callable, bounds: np.ndarray,
                          n_initial: int = 5, n_iter: int = 20,
                          n_candidates: int = 500,
                          rng: np.random.Generator = None) -> np.ndarray:
    """Sample-efficient optimization for expensive objectives.
    1) Seed with Latin-hypercube samples.
    2) Each round: fit GP, maximize Expected Improvement over a random set
       of candidate points, evaluate f at the winner, add to the dataset.
    Inner optimization of EI is done by random search here for clarity;
    production BO uses L-BFGS with multiple restarts."""
    rng = rng or np.random.default_rng()
    lo, hi = bounds[:, 0], bounds[:, 1]
    n = len(lo)
    # Initial design
    lhs = latin_hypercube(n_initial, n, rng=rng)
    X = lo + lhs * (hi - lo)
    y = np.array([f(x) for x in X])

    for _ in range(n_iter):
        gp = GaussianProcess(length_scale=1.0, sigma_f=np.std(y) + 1e-6,
                             sigma_n=1e-4)
        # Normalize y for numerical behavior
        y_norm = (y - y.mean()) / (y.std() + 1e-12)
        gp.fit(X, y_norm)
        y_best = y_norm.min()
        # Candidate set: uniform random in bounds
        cands = lo + rng.random((n_candidates, n)) * (hi - lo)
        ei = expected_improvement(gp, cands, y_best)
        x_next = cands[np.argmax(ei)]
        y_next = f(x_next)
        X = np.vstack([X, x_next])
        y = np.append(y, y_next)
    return X[np.argmin(y)]
```

---

## 13. Discrete optimization

### Branch and bound (for 0-1 knapsack)

```python
def knapsack_branch_and_bound(values: np.ndarray, weights: np.ndarray,
                              capacity: float) -> tuple[float, np.ndarray]:
    """Illustrative 0-1 knapsack solver. Sort by value/weight ratio;
    depth-first branch on include/exclude; bound by LP relaxation of
    the remaining items (allow fractional last item).
    Prunes whenever the optimistic upper bound is no better than the
    current best. For real MILP work, use Gurobi/CBC/HiGHS via Pyomo."""
    n = len(values)
    ratios = values / np.maximum(weights, 1e-12)
    order = np.argsort(-ratios)  # best ratio first
    v, w = values[order], weights[order]

    best_value = 0.0
    best_pick = np.zeros(n, dtype=bool)

    def upper_bound(level: int, cur_value: float, cur_weight: float) -> float:
        """LP-relaxation bound: greedily take remaining items in ratio order,
        allowing a fractional last one."""
        bound = cur_value
        weight_used = cur_weight
        for i in range(level, n):
            if weight_used + w[i] <= capacity:
                bound += v[i]
                weight_used += w[i]
            else:
                remaining = capacity - weight_used
                bound += v[i] * remaining / w[i]
                break
        return bound

    def recurse(level: int, cur_value: float, cur_weight: float,
                picked: np.ndarray):
        nonlocal best_value, best_pick
        if cur_weight > capacity:
            return
        if level == n:
            if cur_value > best_value:
                best_value = cur_value
                best_pick = picked.copy()
            return
        if upper_bound(level, cur_value, cur_weight) <= best_value:
            return  # prune
        # Include item
        picked[level] = True
        recurse(level + 1, cur_value + v[level],
                cur_weight + w[level], picked)
        picked[level] = False
        # Exclude item
        recurse(level + 1, cur_value, cur_weight, picked)

    recurse(0, 0.0, 0.0, np.zeros(n, dtype=bool))
    # Un-permute the pick to match input order
    final_pick = np.zeros(n, dtype=bool)
    final_pick[order[best_pick]] = True
    return best_value, final_pick
```

---

## Final notes

**All of these are pedagogical.** Every algorithm here has a faster, more numerically stable, better-tested library implementation. See the table in `methods.md`. The point of this file is to let Claude read and understand *how* each method works, so it can explain behavior, diagnose failures, and decide between methods with real understanding — not to replace `scipy.optimize.minimize`.

**Common failure modes to recognize when reading these:**
- Step sizes, learning rates, and tolerances in the default arguments are sensible starting points, not universal truths. Real problems need tuning.
- Many of these use `backtracking_line_search` as a subroutine; a poor line search sinks the outer method. Production implementations use Wolfe conditions and more sophisticated strategies.
- Gradient-based methods written here have no safeguards against non-finite values. Production code checks for NaN/Inf every iteration.
- CMA-ES, BO, and similar methods are sensitive to initial scale; if `mean` or `bounds` are wildly misscaled vs the problem's natural scale, the algorithms waste iterations reorienting.

When in doubt, translate "what the algorithm does" from this file into "call this library function" from `methods.md`, and trust the library to handle the rest.
