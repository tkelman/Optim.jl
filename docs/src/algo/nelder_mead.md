# Nelder-Mead
Nelder-Mead is currently the standard algorithm when no derivatives are provided.
## Constructor
```julia
NelderMead(; parameters = adaptive_parameters,
             initial_simplex = relative_simplex)
```
The keywords in the constructor are used to control the following parts of the
solver:

* `parameters` is a function that returns the four parameters controlling the different
types of steps taken by the Nelder-Mead algorithm. It takes an argument `n` which
is the length of the argument to the objective function. It defaults to an adaptive
parameters version, but the original parameters from Nelder and Mead (1965) can
be chosen by specifying `parameters = Optim.fixed_parameters`.
* `initial_simplex` is a function that creates a matrix representing the initial simplex,
where each column is a vertex. It must accept an array as an argument,
as the initial point will be passed to it.


## Description
Our current implementation of the Nelder-Mead algorithm is based on Nelder and Mead (1965) and
Gao and Han (2010). Gradient free methods can be a bit sensitive to starting values
and tuning parameters, so it is a good idea to be careful with the defaults provided
in Optim.

Instead of using gradient information, Nelder-Mead is a direct search method.
It keeps track of the function value at a number
of points in the search space. Together, the points form a simplex. Given a simplex,
we can perform one of four actions: reflect, expand, contract, or shrink. Basically,
the goal is to iteratively replace the worst point with a better point. More information
can be found in Nelder and Mead (1965) or Gao and Han (2010).

The stopping rule is the same as in the original paper, and is the standard
error of the function values at the vertices. To set the tolerance level for this
convergence criterion, set the `g_tol` level as described in the Configurable Options
section.

When the solver finishes, we return a minimizer which is either the centroid or one of the vertices.
The function value at the centroid adds a function evaluation, as we need to evaluate the objection
at the centroid to choose the smallest function value. However, even if the function value at the centroid can be returned
as the minimum, we do not trace it during the optimization iterations. This is to avoid
too many evaluations of the objective function which can be computationally expensive.
Typically, there should be no more than twice as many `f_calls` than `iterations`.
 Adding an evaluation at the centroid when tracing could considerably increase the total
run-time of the algorithm.

### Specifying the initial simplex
As mentioned above, there is a keyword called `initial_simplex` in the Nelder-Mead constructor.
The default choice is called `relative_simplex`. It uses the initial `x` the user
provides to create the initial simplex. To
construct the $i$th vertex, it simply multiplies entry $i$ in the initial vector with
a constant `b`, and adds a constant `a`. This means that the $i$th of the $n$ additional
vertices is of the form

$ (x_0^1, x_0^2, \ldots, x_0^i, \ldots, 0,0) + (0, 0, \ldots, x_0^i\cdot b+a,\ldots, 0,0) $

If an $x_0^i$ is zero, we need the $a$ to make sure all vertices are unique. Generally,
it is advised to start with a relatively large simplex.

If a specific simplex is wanted, it is possible to construct the $n$ by $n+1$ matrix,
and pass a constant anonymous function to `initial_simplex`. For example, let us minimize
the two-dimensional Rosenbrock function, and choose three vertices that have elements
that are simply standard uniform draws.
```julia
using Optim
f(x) = (1.0 - x[1])^2 + 100.0 * (x[2] - x[1]^2)^2
optimize(f, [.0, .0], NelderMead(initial_simplex = x->rand(2,3)))
```
Obviously, the initial point won't really make sense here, but it is an easy way
to construct custom initial simplices.
### The parameters of Nelder-Mead
The different types of steps in the algorithm are governed by four parameters:
$\alpha$ for the reflection, $\beta$ for the expansion, $\gamma$ for the contraction,
and $\delta$ for the shrink step. We default to the adaptive parameters scheme in
Gao and Han (2010). These are based on the dimensionality of the problem, and
are given by

$ \alpha = 1, \quad \beta = 1+2/n,\quad \gamma =0.75 + 1/2n,\quad \delta = 1-1/n $

It is also possible to specify the original parameters from Nelder and Mead (1965)

$ \alpha = 1,\quad \beta = 2, \quad\gamma = 1/2, \quad\delta = 1/2 $

by specifying `parameters  = Optim.fixed_parameters` in the constructor. If another
parameter specification is wanted, simply pass a function to the constructor that
returns a four element tuple of the four parameters.

## References
Nelder, John A. and R. Mead (1965). "A simplex method for function minimization". Computer Journal 7: 308–313. doi:10.1093/comjnl/7.4.308.

Gao, Fuchang and Lixing Han (2010). "Implementing the Nelder-Mead simplex algorithm with adaptive parameters". Computational Optimization and Applications [DOI 10.1007/s10589-010-9329-3]
