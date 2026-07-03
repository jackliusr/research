# OPTIMIZATION TECHNOLOGIES AND FRAMEWORKS: A COMPREHENSIVE SURVEY
 Author: Hermes Agent Research
 Date: July 2026
 License: Free to distribute with attribution

### TABLE OF CONTENTS
  1. Overview of the Optimization Landscape
  2. Mathematical Programming Solvers
     2.1  Gurobi
     2.2  CPLEX (IBM)
     2.3  Xpress (FICO)
     2.4  MOSEK
     2.5  COPT (Cardinal Operations)
     2.6  HiGHS
     2.7  SCIP
     2.8  Clp / Cbc (COIN-OR)
     2.9  GLPK (GNU)
     2.10 Google OR-Tools
  3. Nonlinear Convex Optimization
  4. Derivative-Free / Black-Box Optimization
  5. Metaheuristics and Nature-Inspired Algorithms
  6. Optimization in Machine Learning / AI
  7. Domain-Specific Optimization Frameworks
  8. Optimization Modeling Languages and Environments
  9. Comparison Matrix
 10. Decision Guide
 11. References and Further Reading

## 1. OVERVIEW OF THE OPTIMIZATION LANDSCAPE

Optimization is the mathematical discipline of selecting the best element (with
respect to some criterion) from a set of available alternatives. Formally, an
optimization problem minimizes (or maximizes) an objective function f(x) subject
to constraints g_i(x) ≤ 0, h_j(x) = 0, and variable bounds x ∈ X:

    minimize    f(x)
    subject to  g_i(x) ≤ 0,  i = 1, ..., m
                h_j(x) = 0,  j = 1, ..., p
                x ∈ X ⊆ ℝⁿ

The feasible set is {x ∈ X | g_i(x) ≤ 0, h_j(x) = 0}. A point x* is globally
optimal if f(x*) ≤ f(x) for all feasible x, and locally optimal if this holds
only in a neighborhood of x*.

### 1.1 Major Categories of Optimization

Linear Programming (LP): f and all constraints are linear. Solved efficiently
by the simplex method (Dantzig, 1947) or interior-point methods (Karmarkar,
1984). LPs can be solved to global optimality in polynomial time. Real-world
uses: refinery blending, transportation, workforce scheduling.

Integer Programming (IP) / Mixed-Integer Programming (MIP): Some or all
variables are restricted to integer values. This changes the problem class to
NP-hard in general. Solved via branch-and-bound, cutting planes (Gomory cuts,
Clique cuts), and heuristics. Applications: facility location, airline crew
scheduling, production planning.

Nonlinear Programming (NLP): f or constraints are nonlinear. May be convex
(any local optimum is global) or non-convex (multiple local optima). Solved by
gradient-based methods (sequential quadratic programming, interior-point) or
derivative-free methods. Applications: engineering design, chemical process
optimization, structural analysis.

Convex Optimization: A special case of NLP where the objective is convex and
the feasible set is convex. Subsumes LP, convex QP, SOCP (second-order cone
programming), SDP (semidefinite programming). Tractable and solvable in
polynomial time via interior-point methods. Applications: finance (portfolio
optimization), control theory, signal processing, machine learning (SVMs).

Stochastic Optimization: Problems involving uncertainty in data. Includes
stochastic programming (scenario-based), robust optimization (worst-case),
and chance-constrained programming. Applications: energy systems with
renewable uncertainty, supply chain under demand variability, financial
risk management.

Derivative-Free / Black-Box Optimization: Objective function is not available
in analytical form — only function values can be queried. Includes direct
search methods (Nelder-Mead, Powell), model-based methods (CMA-ES, Bayesian
optimization), and evolutionary algorithms. Applications: hyperparameter tuning,
engineering simulation optimization, drug discovery.

Metaheuristics: High-level problem-independent frameworks that guide lower-level
heuristics to escape local optima. Includes genetic algorithms, simulated
annealing, particle swarm optimization, ant colony optimization, tabu search.
Applications: scheduling, vehicle routing, neural architecture search.

Dynamic Programming (DP): Solves problems with optimal substructure by
breaking them into overlapping subproblems. Used in optimal control, shortest
path problems (Bellman-Ford), and reinforcement learning.

Bayesian Optimization: A global optimization strategy for expensive black-box
functions. Uses a Gaussian process surrogate model and an acquisition function
(Expected Improvement, Upper Confidence Bound) to decide where to sample next.
Applications: hyperparameter tuning, experimental design, A/B testing.

### 1.2 Real-World Applications Across Industries

  Logistics / Supply Chain: Vehicle routing (VRP), warehouse location,
  inventory optimization, fleet assignment. Companies like UPS and Amazon use
  MIP and constraint programming for daily routing of thousands of vehicles.

  Finance: Portfolio optimization (Markowitz mean-variance), algorithmic
  trading (execution optimization), risk management (VaR optimization),
  derivatives pricing (calibration). MOSEK and Gurobi dominate.

  Engineering Design: Topology optimization of mechanical parts, aerodynamic
  shape optimization, antenna design, circuit layout. Uses NLP, adjoint
  methods, and genetic algorithms.

  ML/AI Training: Loss function optimization during neural network training
  (SGD, Adam), hyperparameter tuning (Bayesian optimization), architecture
  search (NAS). Frameworks like PyTorch and Optuna.

  Energy Systems: Unit commitment (which generators to run), economic dispatch,
  power flow optimization (OPF), renewable energy integration. PyPSA, Gurobi,
  and PLEXOS are common.

  Manufacturing: Production scheduling, cutting stock, assembly line balancing,
  quality control. MIP and constraint programming are widely used.

## 2. MATHEMATICAL PROGRAMMING SOLVERS

### 2.1 Gurobi
  Type:       LP, QP, MIP, MIQP, MIQCP, SOCP
  Language:   C, Python, R, Java, MATLAB, Julia (via JuMP)
  License:    Commercial (free academic licenses)
  Website:    https://www.gurobi.com
  Install:    pip install gurobipy
  GitHub:     Not public (closed source)

  Gurobi is widely considered the industry gold standard for MIP. It has held
  the top position on the Hans Mittelmann benchmarks for over a decade, though
  it withdrew from public benchmarks in 2024. Its performance edge comes from
  a highly optimized branch-and-cut engine, including:

    • Concurrent LP solver: runs simplex (primal + dual) and barrier in
      parallel, taking the first solution.
    • Extensive cutting planes: Gomory, MIR, Clique, Cover, Lift-and-Project,
      RLT, and many more.
    • Sophisticated presolve: aggregation, probing, implied bounds, singleton
      row detection.
    • Heuristics: feasibility pump, RINS, local branching, zero-objective,
      and more.
    • Automated tuning tool (gurobi_auto) that tests parameter combinations
      to find the best configuration for a given model.
    • Compute Server / Cluster Manager for distributed solving.
    • Python API is idiomatic and well-documented. User base is the largest
      among commercial solvers.

  Unique features: Gurobi's "cloud" licensing (named-user, token, and
  floating), parameter sensitivity analysis, multi-objective optimization
  with hierarchical or blended objectives, and IIS (irreducible inconsistent
  subsystem) for debugging infeasibility.

  Benchmarks: Until its withdrawal, Gurobi consistently solved the most
  MIPLIB2017 instances in the shortest time. Even today, its MIP performance
  is considered the baseline to beat.

### 2.2 CPLEX (IBM)
  Type:       LP, QP, MIP, MIQP, QCQP, SOCP
  Language:   C, Python, Java, C++, .NET (C#)
  License:    Commercial (free academic via IBM Academic Initiative)
  Website:    https://www.ibm.com/products/ilog-cplex-optimization-studio
  GitHub:     Not public

  CPLEX was the original gold standard of optimization, dominating the 1990s
  and early 2000s before Gurobi's rise. It remains highly competitive,
  especially:

    • Excellent network flow algorithms — CPLEX's network optimizer is
      best-in-class for pure network problems.
    • Strong support for quadratically constrained programs (QCP).
    • IBM ILOG CPLEX Optimization Studio (COS) provides a full IDE with
      the OPL (Optimization Programming Language), which integrates with
      the CP Optimizer for constraint programming.
    • Conflict refiner for infeasible models (analogous to IIS).
    • Distributed parallel MIP (across machines).
    • Integration with IBM Decision Optimization on Cloud (SAAS).

  CPLEX's Python API (docplex) is well-maintained. Performance-wise, CPLEX
  trails Gurobi by roughly 5–15% on MIP benchmarks but is often faster on
  certain LP classes (especially network LPs).

  In 2018, CPLEX results were removed from Mittelmann's benchmarks at IBM's
  request, making direct public comparisons harder. However, internal
  benchmarks suggest CPLEX remains within striking distance of Gurobi.

### 2.3 Xpress (FICO)
  Type:       LP, QP, MIP, MIQP, SOCP
  Language:   Mosel (proprietary DSL), Python, C, Java, .NET
  License:    Commercial (free academic via FICO Academic Program)
  Website:    https://www.fico.com/en/products/fico-xpress-optimization
  GitHub:     Not public

  Xpress (formerly Dash Optimization / Xpress-MP) is the third of the "big
  three" commercial solvers. It offers:

    • The Mosel programming language — a proprietary modeling language with
      integrated solver access. Mosel is powerful but has a steeper learning
      curve than Python APIs.
    • Strong in stochastic programming — Xpress has built-in support for
      stochastic programming via the Xpress-SP module.
    • Focus on financial services: many banks and hedge funds use Xpress for
      portfolio optimization and risk management.
    • Good MIP performance — while usually 3rd behind Gurobi and CPLEX,
      Xpress is sometimes faster on specific problem classes (e.g., financial
      portfolio problems with SOS constraints).
    • Xpress Insight — a web-based application platform for deploying
      optimization models.

  FICO's acquisition of Xpress in 2014 brought optimization into the credit
  scoring / decision management ecosystem. Xpress is still a solid choice,
  particularly when tight FICO ecosystem integration is needed.

  Benchmark note: Xpress also withdrew from Mittelmann's benchmarks in 2018
  alongside CPLEX.

### 2.4 MOSEK
  Type:       LP, QP, SOCP, SDP, exponential and power cone programming
  Language:   C, Python, Java, MATLAB, R, Julia (via JuMP)
  License:    Commercial (free academic licenses)
  Website:    https://www.mosek.com
  GitHub:     Not public

  MOSEK is the specialist solver for convex conic optimization. It is not
  primarily an MIP solver (though it handles MISOCP); its strength is:

    • Best-in-class SOCP (second-order cone programming) performance.
    • Best-in-class SDP (semidefinite programming) performance, including
      large-scale SDP with sparse data.
    • Exponential and power cone constraints — unique among commercial solvers.
    • Highly robust interior-point implementation for LP — often the fastest
      barrier solver for large LPs.
    • Used heavily in finance (Markowitz portfolio optimization with
      transaction costs and cardinality constraints) and control theory
      (linear matrix inequalities).

  MOSEK's Python API (mosek.fusion) provides a clean object-oriented
  interface for building conic models. For disciplined convex programming,
  MOSEK is the most common backend for CVXPY.

  Benchmarks: MOSEK tops the Mittelmann SOCP and SDP benchmarks. It is the
  de facto standard for convex conic optimization in both industry and
  academia.

### 2.5 COPT (Cardinal Operations)
  Type:       LP, MIP, QP
  Language:   Python (primary), C, C++
  License:    Commercial
  Website:    https://www.copt.de
  GitHub:     Not public

  COPT (Cardinal Optimizer) is a Chinese solver that has rapidly improved
  since its launch. It is now consistently ranked in the top 3–5 MIP solvers
  globally, competitive with Gurobi and CPLEX on many instances.

    • Strong presolve and cutting planes.
    • Competitive MIP performance — often faster than CPLEX on MIPLIB2017.
    • Python-first API with a user-friendly interface.
    • Active development with frequent releases.
    • Growing adoption in Chinese industry and increasingly internationally.

  COPT is still less known in Western markets but is a serious contender.
  It is particularly popular in Asia for supply chain and logistics
  optimization.

### 2.6 HiGHS
  Type:       LP, QP, MIP
  Language:   C++ (core), Python (highspy), Julia, R, C
  License:    Open source (MIT license)
  Website:    https://highs.dev
  GitHub:     https://github.com/ERGO-Code/HiGHS (stars: 1,689, forks: 330)
  Install:    pip install highspy

  HiGHS is the most successful open-source LP solver, often matching or
  exceeding commercial solvers on pure LP problems.

    • Parallel simplex: runs primal and dual simplex concurrently.
    • Interior-point (barrier) method for LP.
    • MIP solver via branch-and-cut (less mature than LP, but improving).
    • Developed by the University of Edinburgh with support from Google,
      ERGO, and others.
    • Used as the default LP solver in Google OR-Tools.

  On the Mittelmann LP feasibility and LP optimality benchmarks (July 2026),
  HiGHS is highly competitive, often beating CPLEX and trailing only Gurobi
  and COPT among mainstream solvers. For MIP, it is weaker than SCIP and
  far behind Gurobi, but it is the best open-source choice for LP.

### 2.7 SCIP
  Type:       MIP, MINLP, Constraint Integer Programming
  Language:   C (core), Python (PySCIPOpt), Julia
  License:    Open source (ZIB Academic License — free for non-commercial)
  Website:    https://www.scipopt.org
  GitHub:     https://github.com/scipopt/scip (stars: 607, forks: 106)
  Install:    pip install pyscipopt

  SCIP (Solving Constraint Integer Programs) is the best non-commercial MIP
  solver. It is also a framework for constraint integer programming (CIP),
  unifying MIP, constraint programming (CP), and SAT solving.

    • Highly modular architecture with plugins for custom constraints,
      branching rules, cutting plane separators, heuristics, and node
      selectors.
    • Supports LP relaxation (via SoPlex, its built-in LP solver), and can
      use HiGHS or CPLEX as the LP subsolver.
    • Excellent for research — its plugin system allows testing new
      algorithmic ideas without recompiling the core.
    • Supports mixed-integer nonlinear programming (MINLP) via NLP
      subsolvers (IPOPT, FilterSQP).
    • PySCIPOpt provides a Python interface.

  Benchmark note: SCIP is significantly slower than Gurobi on MIP (typically
  5–50× slower depending on instance), but it is free for academic research
  and is the most popular platform for MIP algorithm research. The annual
  SCIP Optimization Suite release includes updates from the Zuse Institute
  Berlin.

### 2.8 Clp / Cbc (COIN-OR)
  Type:       Clp (LP), Cbc (MIP)
  Language:   C++ (core), Python (CyLP, Pyomo)
  License:    Open source (Eclipse Public License / Common Public License)
  Website:    https://www.coin-or.org
  GitHub:     Clp: https://github.com/coin-or/Clp (stars: 488, forks: 98)
              Cbc: https://github.com/coin-or/Cbc (stars: 999, forks: 139)
  Install:    pip install cylp  (or via Pyomo/ortools)

  The COIN-OR (Computational Infrastructure for Operations Research) project
  provides foundational open-source solvers:

    • Clp (Coin LP): Primal and dual simplex, barrier. Solid LP performance
      but slower than HiGHS.
    • Cbc (Coin Branch-and-Cut): MIP solver built on Clp. Functional but
      slow compared to SCIP or HiGHS's MIP mode.
    • Used as a backend by Pyomo and other modeling systems.
    • Cgl (Cut Generation Library) provides cutting planes used by Cbc and
      many other solvers.
    • CoinUtils, Osi, and other utility libraries provide infrastructure.

  These are more useful as building blocks than end-user tools. They form
  the backbone of many optimization pipelines, particularly in academic
  settings.

### 2.9 GLPK (GNU Linear Programming Kit)
  Type:       LP, MIP
  Language:   C (core), GMPL (GNU MathProg Language)
  License:    Open source (GNU GPLv3)
  Website:    https://www.gnu.org/software/glpk
  GitHub:     https://github.com/coin-or/GMPL (mirror)
  Install:    apt install glpk-utils  /  pip install swiglpk

  GLPK is the GNU project's LP/MIP solver. It is:

    • Pure simplex-based (no barrier for LP, though it has interior-point).
    • Slow — roughly 50–100× slower than Gurobi on MIP, and 10–20× slower
      than HiGHS on LP.
    • Very reliable and bug-free for small to medium problems.
    • Supports the GMPL language (similar to AMPL), making it useful for
      teaching algebraic modeling.
    • Python binding: swiglpk (low-level), or use via PuLP.

  Best use case: education, small models, and environments where GPL
  licensing is a requirement.

### 2.10 Google OR-Tools
  Type:       Wrapper (LP, MIP, CP), CP-SAT (native)
  Language:   Python, C++, Java, .NET
  License:    Open source (Apache 2.0)
  Website:    https://developers.google.com/optimization
  GitHub:     https://github.com/google/or-tools (stars: 13,715, forks: 2,421)
  Install:    pip install ortools

  OR-Tools is Google's operations research toolkit. Its key components:

    • CP-SAT solver: A constraint programming solver based on SAT technology.
      Extremely fast for scheduling, routing, and constraint satisfaction.
      Often outperforms MIP solvers on pure combinatorial problems.
    • Solver wrappers: Interfaces to SCIP, Gurobi, CPLEX, HiGHS, and others.
      Provides a unified API (MPModel) for MIP problems.
    • Routing library: Specialized vehicle routing solver with LNS (large
      neighborhood search) heuristics. Supports TSP, VRP with time windows,
      capacity constraints, etc.
    • Scheduling algorithms: Job shop, flexible job shop, employee scheduling.
    • Assignment and knapsack solvers.
    • Network flow algorithms: Min-cost flow, max flow, minimum cut.

  OR-Tools is the go-to framework for combinatorial optimization problems
  that have a strong constraint structure. Its CP-SAT solver is particularly
  notable — it won the MiniZinc Challenge multiple times and is competitive
  with commercial CP solvers.

  Unique: OR-Tools is not a single solver but an integrated framework. It
  excels at problems where a pure MIP formulation would be cumbersome.

## 3. NONLINEAR CONVEX OPTIMIZATION

Convex optimization problems have the property that every local minimum is a
global minimum. This makes them tractable even at large scale.

### CVXPY
  Website:   https://www.cvxpy.org
  GitHub:    https://github.com/cvxpy/cvxpy (stars: 6,272, forks: 1,184)
  Install:   pip install cvxpy

  CVXPY is a Python-embedded modeling language for disciplined convex
  programming (DCP). You write problems in a natural mathematical syntax,
  and CVXPY verifies convexity via its DCP rules, then translates the
  problem to a standard conic form and passes it to a solver.

  Supported solvers (as backends):
    • MOSEK — best for SOCP/SDP
    • Gurobi — QP, SOCP
    • ECOS — Embedded Conic Solver (small-to-medium problems)
    • SCS — Splitting Conic Solver (large-scale, first-order method)
    • Clarabel — new Rust-based conic solver
    • CVXOPT — Python-based interior-point solver
    • SDPT3, SeDuMi — via MATLAB bridge

  CVXPY is widely used in finance (portfolio optimization), control
  (MPC design), statistics (lasso, elastic net), and engineering.

### CVXOPT
  Website:   https://cvxopt.org
  Install:   pip install cvxopt

  CVXOPT is a Python package for convex optimization (LP, QP, SOCP, SDP).
  It provides its own interior-point solver. Useful for custom solver
  development and small-to-medium problems.

### Convex.jl (Julia)
  Website:   https://jump.dev/Convex.jl
  GitHub:    https://github.com/jump-dev/Convex.jl

  Julia equivalent of CVXPY. Supports the same set of conic solvers via
  MathOptInterface. Benefits from Julia's speed (compilation) for large
  problem construction.

### IPOPT (Interior Point OPTimizer)
  Type:       NLP (continuous, large-scale)
  Language:   C++ (core), Python (via pyomo/cyipopt)
  License:    Open source (EPL 2.0)
  Website:    https://coin-or.github.io/Ipopt
  GitHub:     https://github.com/coin-or/Ipopt (stars: 1,764, forks: 316)
  Install:    pip install ipopt  (or cyipopt)

  IPOPT is the premier open-source NLP solver. It implements a primal-dual
  interior-point (barrier) method with a filter line-search. Key features:

    • Solves large-scale nonlinear problems (10⁶+ variables).
    • Requires first derivatives (gradients) — exact or finite-difference.
    • Optional second derivatives (Hessian) for faster convergence.
    • Interfaces to linear solvers (MUMPS, MA27, PARDISO, etc.) for the
      KKT system.
    • Used as the NLP subsolver in SCIP (for MINLP) and in Pyomo.
    • Widely used in chemical engineering, optimal control, and power
      systems (optimal power flow).

  Benchmarks: On the Mittelmann AMPL-NLP benchmark, IPOPT is the top
  open-source solver and competitive with commercial solvers (KNITRO,
  CONOPT, SNOPT) on many problem classes.

### SDPT3 and SeDuMi
  Type:       SDP (semidefinite programming)
  Language:   MATLAB
  License:    Open source (academic)
  Websites:   https://www.math.nus.edu.sg/~mattohkc/sdpt3.html
              https://sedumi.ie.lehigh.edu

  SDPT3 and SeDuMi are the classic MATLAB solvers for SDP. They use
  primal-dual interior-point methods. While showing their age, they remain
  widely cited (1000s of academic papers) and are still useful for
  moderate-sized SDP problems. MOSEK has largely replaced them in industry.

### ECOS and SCS
  Type:       Conic optimization (SOCP, SDP)
  Language:   C (core), Python (via cvxpy)
  License:    MIT / BSD
  Website:    https://github.com/embotech/ecos
  GitHub:     https://github.com/bodono/scs-python

  ECOS (Embedded Conic Solver): Small-footprint interior-point solver
  designed for embedded applications and MPC. Very fast on small SOCPs.

  SCS (Splitting Conic Solver): First-order (ADMM-based) solver for
  large-scale conic problems. Scales to millions of variables but gives
  lower accuracy (suitable when moderate precision is acceptable).

## 4. DERIVATIVE-FREE / BLACK-BOX OPTIMIZATION

When gradients are unavailable (e.g., simulation-based optimization, hardware-
in-the-loop, legacy code), derivative-free methods are essential.

### CMA-ES (Covariance Matrix Adaptation Evolution Strategy)
  Language:   Python (cma package), C, MATLAB
  License:    Open source (BSD)
  Website:    https://github.com/CMA-ES/pycma (stars: 1,333, forks: 199)
  Install:    pip install cma

  CMA-ES is widely considered the best algorithm for continuous black-box
  optimization. It maintains a multivariate normal distribution over the
  search space and adapts the covariance matrix based on successful steps.
  Key features:

    • State-of-the-art performance on the COCO (Comparing Continuous
      Optimizers) benchmark platform.
    • Works well on ill-conditioned, non-separable, and multi-modal
      problems.
    • No gradient information required.
    • Variants: CMA-ES with restarts, CMA-ES with bounds (via penalization
      or mirroring), BIPOP-CMA-ES for multi-modal landscapes.
    • Python package `cma` provides an easy-to-use interface.

  Applications: hyperparameter tuning, engineering design optimization,
  robotics (policy search), and scientific computing.

### Bayesian Optimization
  Bayesian optimization uses a probabilistic surrogate model (typically a
  Gaussian process) to model the objective and an acquisition function to
  select evaluation points. Key libraries:

  • BoTorch (Facebook/Meta, PyTorch-based)
      GitHub: https://github.com/pytorch/botorch (stars: 3,564, forks: 484)
      Install: pip install botorch
      Built on PyTorch, supports GPU acceleration, batch BO, multi-objective
      BO, constrained BO, and high-dimensional BO (via random embeddings or
      additive GPs). Used internally at Meta for production tuning.

  • BayesianOptimization (Python)
      GitHub: https://github.com/bayesian-optimization/BayesianOptimization
        (stars: 8,663, forks: 1,600)
      Install: pip install bayesian-optimization
      Simple implementation — good for getting started. Uses Gaussian
      processes with Expected Improvement.

  • GPyOpt (SheffieldML)
      GitHub: https://github.com/SheffieldML/GPyOpt (stars: 947, forks: 260)
      Install: pip install gpyopt
      Built on GPy. Supports a variety of acquisition functions and batch
      optimization.

  • Spearmint (Hutter Lab)
      GitHub: https://github.com/HIPS/Spearmint
      One of the earliest BO packages. Still functional but less maintained.

  • Hyperopt
      GitHub: https://github.com/hyperopt/hyperopt (stars: 7,587, forks: 1,075)
      Install: pip install hyperopt
      Uses Tree-structured Parzen Estimators (TPE) — a Bayesian optimization
      variant that works well for hyperparameter tuning.

  Applications: Hyperparameter tuning is the dominant use case, but BO is
  also used in experimental design, materials science, drug discovery,
  and A/B testing.

### SciPy Direct Search Methods
  Available in `scipy.optimize` (stars: 14,799, forks: 5,788):

    • Nelder-Mead (simplex) — Simple, robust, but slow for high dimensions.
    • Powell's method — Conjugate direction search, no derivatives needed.
    • COBYLA (Constrained Optimization BY Linear Approximations) —
      Handles constraints via linear approximations.
    • Differential evolution — Global optimization, population-based.
    • dual_annealing — Simulated annealing variant.
    • SHGO (simplicial homology global optimization).

  These are solid choices for small-to-medium problems where gradient
  computation is infeasible. For large-scale problems, CMA-ES or
  Bayesian optimization are preferred.

### Nevergrad (Meta/Facebook)
  GitHub:    https://github.com/facebookresearch/nevergrad (stars: 4,195, forks: 369)
  Install:   pip install nevergrad

  Nevergrad is a comprehensive derivative-free optimization library that
  provides 400+ algorithms, including:

    • Evolution strategies (ES, PSO, DE, CMA-ES variants).
    • Bayesian optimization and TPE.
    • Bandit optimization (optimistic, hyperopt-like).
    • Parametrization tools for complex search spaces.
    • Automatic algorithm selection and portfolio allocation.

  Nevergrad is used at Meta for hyperparameter tuning and has won multiple
  black-box optimization competitions (e.g., NeurIPS 2020 Black-Box
  Optimization Challenge).

### NOMAD
  Type:       Black-box optimization (MADS algorithm)
  Language:   C++ (core), Python (via PyNomad)
  License:    LGPL (open source)
  Website:   https://www.gerad.ca/nomad
  GitHub:     https://github.com/bbopt/nomad

  NOMAD implements the Mesh Adaptive Direct Search (MADS) algorithm,
  designed for constrained black-box optimization. It is particularly
  effective for problems with expensive function evaluations and handles
  both equality and inequality constraints via progressive barrier or
  extreme barrier approaches.

## 5. METAHEURISTICS AND NATURE-INSPIRED ALGORITHMS

Metaheuristics are high-level strategies for exploring search spaces,
typically inspired by natural phenomena.

### Genetic Algorithms (GA)

  • DEAP (Distributed Evolutionary Algorithms in Python)
      GitHub: https://github.com/DEAP/deap (stars: 6,412, forks: 1,162)
      Install: pip install deap
      The most comprehensive evolutionary computation framework in Python.
      Supports GA, GP (genetic programming), ES, PSO, and multi-objective
      optimization (NSGA-II, SPEA2). Highly modular — you define the
      genetic operators yourself.

  • PyGAD
      GitHub: https://github.com/ahmedfgad/GeneticAlgorithmPython
        (stars: 2,209, forks: 501)
      Install: pip install pygad
      Simpler API than DEAP, designed for ease of use. Supports feature
      selection, neural network weight optimization, and parameter tuning.

  • ga (SciPy)
      `scipy.optimize.differential_evolution` is a GA variant (real-coded).

### Particle Swarm Optimization (PSO)

  • PySwarms
      GitHub: https://github.com/ljvmiranda921/pyswarms (stars: 1,392, forks: 339)
      Install: pip install pyswarms
      Research toolkit for PSO. Supports global best, local best, and
      quantum PSO variants. Includes visualization tools.

  • Simple PSO via scipy
      `scipy.optimize.dual_annealing` uses a generalized simulated
      annealing approach but can incorporate swarm elements.

### Simulated Annealing

  • `scipy.optimize.dual_annealing` provides a robust SA implementation
    with local search refinement.
  • Custom SA implementations are common for specific problems (e.g.,
    `simanneal` package).

### Ant Colony Optimization (ACO)

  • ACO-Pants (Python implementation for TSP).
  • ACOTSP (C implementation by Stützle).
  • More commonly, researchers implement ACO from scratch for specific
    routing problems.

### Differential Evolution

  • `scipy.optimize.differential_evolution` — Best DE implementation in
    Python. Supports constraints, parallel evaluation, and various
    mutation/crossover strategies.

### Practical Uses

  • Scheduling: Job shop scheduling, employee rostering, timetabling.
  • Routing: Vehicle routing (especially with many constraints), traveling
    salesman.
  • Engineering design: Aerodynamic shape parameterization, circuit layout.
  • Feature selection: Wrapper methods with GA to find optimal feature
    subsets.
  • Neural architecture search: Evolutionary NAS (e.g., AmoebaNet by
    Google Brain).
  • Hyperparameter optimization: GA and PSO are competitive with Bayesian
    methods on some problem landscapes.

## 6. OPTIMIZATION IN MACHINE LEARNING / AI

### 6.1 First-Order Optimization Methods

First-order methods use gradient information to minimize the loss function.
They are the workhorse of deep learning.

  • SGD (Stochastic Gradient Descent): The original method — updates
    parameters using the gradient of a minibatch. Learning rate tuning is
    critical.

  • Momentum (Polyak, 1964): Accumulates a velocity term to smooth
    updates and accelerate convergence.

  • Nesterov Accelerated Gradient (Nesterov, 1983): "Look ahead" momentum
    variant. Provably optimal convergence rate for convex optimization.

  • AdaGrad (Duchi et al., 2011): Per-parameter learning rates based on
    the sum of squared gradients. Good for sparse features but the
    accumulation causes premature learning rate decay.

  • RMSProp (Hinton, 2012): Uses exponential moving average of squared
    gradients to normalize updates. Fixes AdaGrad's aggressive decay.

  • Adam (Kingma & Ba, 2015): Combines momentum and RMSProp with bias
    correction. The default optimizer for most deep learning. Extensions:
    AdamW (decouples weight decay, Loshchilov & Hutter, 2019).

  • AdaBelief (Zhuang et al., 2020): Modifies Adam to use the belief in
    the gradient direction. Often more stable.

  • Lion (Chen et al., 2023): Evolved by symbolic program search from the
    Google Brain team. Simple update: sign(x * grad) only. Symbolically
    derived — fewer hyperparameters than Adam.

  • Sophia (Liu et al., 2023): Uses Hessian diagonal information via a
    Hutchinson trace estimator. Preconditioned gradient for faster
    convergence, especially in LLM training.

  Implementations:
    • PyTorch: `torch.optim.SGD`, `torch.optim.Adam`, `torch.optim.AdamW`,
      `torch.optim.RMSprop`, `torch.optim.Adagrad`, `torch.optim.LBFGS`.
    • TensorFlow/Keras: `keras.optimizers.Adam`, `keras.optimizers.SGD`,
      `keras.optimizers.RMSprop`.
    • JAX/Flax: `optax` library provides a comprehensive set of gradient
      transformation combinators.

  Learning Rate Scheduling:
    • Cosine annealing (Loshchilov & Hutter, 2017): Cosine-shaped LR decay
      with periodic restarts.
    • Warmup: Linear increase from 0 to max LR over first K steps.
    • OneCycle (Smith, 2018): Triangular LR schedule with gradual decay.
    • ReduceLROnPlateau: Reduces LR when validation loss plateaus.

### 6.2 Second-Order Methods

Second-order methods use curvature information (Hessian) to accelerate
convergence. They are less common in deep learning due to the O(n²) memory
cost of storing the Hessian, but are important for specific use cases.

  • L-BFGS (Limited-memory Broyden–Fletcher–Goldfarb–Shanno):
    Approximates the inverse Hessian. Available in PyTorch (`torch.optim.
    LBFGS`), SciPy (`scipy.optimize.minimize(method='L-BFGS-B')`). Used
    for small-batch or full-batch optimization (e.g., neural ODE training,
    fine-tuning small models).

  • KFAC (Kronecker-Factored Approximate Curvature):
    Approximates the Fisher information matrix using Kronecker-product
    factorization. Implementations: `kfac` (various packages), used in
    distributed training at DeepMind/Google.

  • Hessian-free optimization (Martens, 2010):
    Uses conjugate gradient to compute approximate Newton steps without
    explicitly forming the Hessian. Historically important but rarely
    used today.

  • Sophia (mentioned above) is a recent bridge — it uses Hessian
    diagonal estimates without the full memory cost.

### 6.3 Hyperparameter Optimization

  • Optuna (Preferred Networks)
      GitHub: https://github.com/optuna/optuna (stars: 14,447, forks: 1,349)
      Install: pip install optuna
      Define-by-Run API — most popular Python HPO framework. Supports
      TPE, CMA-ES, grid, random, and evolutionary search. Features: pruning
      (early stopping of bad trials), multi-objective optimization,
      distributed execution, visualization dashboard.

  • Hyperopt (James Bergstra)
      GitHub: https://github.com/hyperopt/hyperopt (stars: 7,587, forks: 1,075)
      Install: pip install hyperopt
      Pioneered TPE-based hyperparameter optimization. Supports
      MongoTrials for distributed execution. Less maintained than Optuna
      in 2026.

  • Ray Tune (Anyscale / UC Berkeley)
      Website: https://docs.ray.io/en/latest/tune
      Install: pip install ray[tune]
      Part of the Ray ecosystem for distributed computing. Supports all
      major HPO algorithms (Bayesian, PBT, HyperBand, ASHA) and integrates
      with PyTorch, TensorFlow, and JAX. Best for large-scale distributed
      HPO.

  • SMAC3 (University of Freiburg)
      GitHub: https://github.com/automl/SMAC3
      Install: pip install smac
      Sequential model-based optimization. Uses random forests (not GPs)
      as the surrogate model, which works well for high-dimensional and
      mixed-type search spaces (common in algorithm configuration).

  • NNI (Microsoft Neural Network Intelligence)
      GitHub: https://github.com/microsoft/nni (stars: 14,359, forks: 1,858)
      Install: pip install nni
      Microsoft's AutoML toolkit. Supports HPO (TPE, CMA-ES, grid, random),
      NAS (DARTS, ENAS, one-shot), model compression, and feature
      engineering.

### 6.4 Neural Architecture Search (NAS)

  • DARTS (Differentiable Architecture Search, Liu et al., 2019):
    Relaxes architecture search to continuous optimization, solved by
    bilevel optimization (train weights + architecture parameters).
    Implementations in NNI and AutoML frameworks.

  • ENAS (Efficient NAS, Pham et al., 2018):
    Parameter sharing across architectures for efficiency.

  • Google AutoML:
    Cloud-based NAS using reinforcement learning or evolutionary search.

  • Microsoft NNI:
    Supports most NAS methods (DARTS, ENAS, SPOS, ProxylessNAS).

  • KerasTuner:
    Simple hyperparameter search for Keras models.

### 6.5 Reinforcement Learning Optimization

  RL can be viewed as optimization of a reward function through interaction:

  • Policy Gradient Methods: REINFORCE (Williams, 1992), A2C, PPO (Schulman
    et al., 2017). PPO is the most widely used due to its stability.
  • SAC (Soft Actor-Critic, Haarnoja et al., 2018): Maximum entropy RL for
    continuous control.
  • DQN (Mnih et al., 2013): Q-learning with deep neural networks.
  • Evolutionary RL: Uses CMA-ES or GA to evolve policy parameters directly,
    bypassing backpropagation.

  RL is a rapidly growing field with applications in robotics, game playing,
  recommender systems, and autonomous driving.

## 7. DOMAIN-SPECIFIC OPTIMIZATION FRAMEWORKS

### 7.1 Supply Chain / Logistics

  • Google OR-Tools (CP-SAT, Routing Library):
    Vehicle routing (TSP, VRP, CVRP, VRPTW), job shop scheduling, employee
    scheduling. The routing library uses LNS heuristics that scale to
    hundreds of vehicles and thousands of stops.

  • IBM ILOG CP Optimizer:
    Constraint programming solver (part of CPLEX Studio). Uses interval
    variables and sequence variables for scheduling. Excellent for
    project scheduling and resource-constrained scheduling.

  • Gurobi (MIP for Supply Chain):
    Facility location (p-median, p-center), inventory optimization (s,S
    policies), network design (multi-echelon), production planning (lot
    sizing).

  • Lokad (forecasting + optimization):
    Combines probabilistic demand forecasting with inventory optimization.
    Uses a specialized DSL (Envision) and is cloud-native.

  • Coupa / Llamasoft (now part of Coupa):
    Supply chain design and optimization (network flow, inventory
    positioning, transportation optimization). The industry standard for
    strategic supply chain design.

### 7.2 Finance

  • Portfolio Optimization:
    CVXPY + MOSEK/Gurobi for mean-variance optimization, risk parity,
    Black-Litterman. Common models: minimum variance, maximum Sharpe,
    conditional value-at-risk (CVaR) minimization.

  • Algorithmic Trading:
    Execution optimization (VWAP, TWAP, implementation shortfall) uses
    convex optimization (CVXPY, MOSEK) for optimal trade scheduling.

  • Risk Management:
    Counterparty credit risk (CVA/DVA optimization), margin optimization,
    stress testing — all use LP/MIP (Gurobi, CPLEX).

  • Derivatives Pricing:
    Calibration of stochastic volatility models (Heston, SABR) —
    nonlinear least squares (scipy, Ceres Solver).

  • Commercial Platforms:
    QRM (Quantitative Risk Management) — CVA, PFE calculations.
    Numerix — derivatives pricing and risk.
    Murex — front-to-back office risk and trading.

### 7.3 Energy / Power Systems

  • PyPSA (Python for Power System Analysis):
      GitHub: https://github.com/PyPSA/PyPSA (stars: 2,043, forks: 662)
      Install: pip install pypsa
      Open-source framework for power system modeling and optimization.
      Supports optimal power flow (AC/DC OPF), unit commitment, capacity
      expansion planning, and market modeling. Uses Pyomo or native LP
      solvers (linopy wrapper around HiGHS, Gurobi).

  • GridLAB-D:
      Open-source simulation of power distribution systems. Includes
      optimization for distributed energy resource (DER) scheduling.

  • PLEXOS (commercial, Energy Exemplar):
      Integrated energy market simulation and optimization. Used by most
      US ISOs/RTOs for production cost modeling and capacity expansion.
      Internally uses Gurobi, CPLEX, or Xpress.

  • Gurobi / RAMP (unit commitment):
      RAMP (UC Berkeley) is an open-source stochastic unit commitment
      model using Gurobi. Widely used in power systems research.

### 7.4 Engineering

  • COMSOL Multiphysics:
    PDE-constrained optimization (topology optimization, parameter
    estimation, shape optimization). Uses gradient-based solvers
    (SNOPT, IPOPT) with adjoint sensitivity analysis.

  • ANSYS Mechanical:
    Topology optimization, shape optimization, and parameter optimization
    for structural design. Uses built-in gradient-based and gradient-free
    methods.

  • OpenMDAO (NASA):
      Website: https://openmdao.org
      Open-source Multidisciplinary Design Analysis and Optimization
      (MDAO) framework. Used for aircraft design, wind turbine design,
      and spacecraft trajectory optimization. Supports gradient-based
      and gradient-free optimization with coupled systems.

  • Dakota (Sandia National Labs):
    Multidisciplinary optimization toolkit for engineering design.
    Supports optimization, UQ, sensitivity analysis, and surrogate-based
    optimization.

### 7.5 Process Systems Engineering

  • GAMS (General Algebraic Modeling System):
    Commercial algebraic modeling with extensive solver support. Dominant
    in energy, economics, and process engineering (refinery optimization).
    Has CAPEX/OPEX cost models for chemical processes.

  • AMPL:
    Algebraic modeling (see Section 8 for details).

  • Pyomo:
    Python-based algebraic modeling (see Section 8).

  • CasADi:
    Symbolic framework for optimal control (see Section 8).

  • GEKKO:
    Python package for dynamic optimization (see Section 8).

## 8. OPTIMIZATION MODELING LANGUAGES AND ENVIRONMENTS

### AMPL
  Type:       Algebraic modeling language (commercial)
  Website:    https://ampl.com
  Solver interfaces: Gurobi, CPLEX, Xpress, MOSEK, HiGHS, IPOPT, KNITRO,
                     CONOPT, SNOPT, BARON, and 20+ more.
  License:    Commercial (free student version with solver limits)

  AMPL (A Mathematical Programming Language) is the oldest and most
  established algebraic modeling language. Its syntax is clean and concise:
  variables, objectives, and constraints are declared in a natural algebraic
  form. AMPL separates the model (.mod) from the data (.dat), enabling
  parameterized modeling.

  Strengths: Excellent for teaching, extremely fast model generation
  (compiled), large library of solver interfaces, and strong community
  support. The free student version allows solving with HiGHS and a subset
  of features.

### GAMS
  Type:       Algebraic modeling language (commercial)
  Website:    https://www.gams.com
  Solver interfaces: Gurobi, CPLEX, Xpress, MOSEK, IPOPT, CONOPT, SNOPT,
                     BARON, SCIP, and 30+ more.
  License:    Commercial (free demo with small problem size limits)

  GAMS (General Algebraic Modeling System) is widely used in energy
  economics, agricultural economics, and environmental policy modeling.
  Its strength is in large, multi-period, stochastic economic models.
  The GAMS Data eXchange (GDX) format allows seamless data transfer.

  Differences from AMPL: GAMS uses a slightly more verbose syntax but has
  stronger support for stochastic programming (via DECIS, LINDO) and
  complementarity problems (MCP, MPEC).

### Pyomo (Python)
  Type:       Python-based algebraic modeling (open source)
  Website:    https://www.pyomo.org
  GitHub:     https://github.com/Pyomo/pyomo (stars: 2,483, forks: 582)
  Install:    pip install pyomo
  Solver interfaces: Gurobi, CPLEX, IPOPT, HiGHS, SCIP, CBC, GLPK, BARON,
                     KNITRO, MOSEK, and 15+ more.
  License:    Open source (BSD)

  Pyomo is the most popular Python-based modeling language for optimization.
  It allows you to define optimization models in pure Python — using
  standard Python data structures (lists, dicts, NumPy arrays) — and solve
  them with any supported solver.

  Key features:
    • Supports LP, MIP, NLP, MINLP, and stochastic programming.
    • Concrete (immediate data) and Abstract (separable model/data) models.
    • Automatic differentiation for NLP gradients (via PyNumero or AMPL
      solver interface).
    • PySP — stochastic programming extension (scenario-based).
    • GDP (Generalized Disjunctive Programming) support.
    • Parallel decomposition schemes (Lagrangian decomposition, Benders).
    • Strong integration with Jupyter notebooks.

  Pyomo is the standard choice for academic research and is increasingly
  used in industry for prototyping and production.

### JuMP (Julia)
  Type:       Julia-based algebraic modeling (open source)
  Website:    https://jump.dev
  GitHub:     https://github.com/jump-dev/JuMP.jl (stars: 2,450, forks: 419)
  Install:    Pkg.add("JuMP")
  Solver interfaces: Gurobi, CPLEX, MOSEK, IPOPT, HiGHS, SCIP, Xpress,
                     KNITRO, BARON, and 30+ (via MathOptInterface)
  License:    Open source (MIT)

  JuMP is the Julia equivalent of Pyomo (or AMPL). It benefits from Julia's
  just-in-time compilation, making model construction extremely fast for
  large problems. Key advantages:

    • Very fast model construction (often 10–100× faster than Pyomo for
      large sparse problems).
    • Direct access to solver internals via MathOptInterface (MOI).
    • Callbacks for lazy constraints, user cuts, and heuristic solutions.
    • Supports almost all solvers via the MOI ecosystem.
    • Duality analysis and sensitivity computation.
    • Strong community and growing adoption in academia.

  JuMP is particularly strong when you need to solve many similar models
  quickly (e.g., in stochastic programming with many scenarios) or when
  you need fine-grained control over solver interaction.

### CVXPY (Python)
  (Covered in Section 3 — disciplined convex programming)

### CVXR (R)
  Type:       R-based convex modeling (open source)
  Website:    https://cvxr.rbind.io
  GitHub:     https://github.com/cvxr/CVXR
  Install:    install.packages("CVXR")
  Solver interfaces: ECOS, SCS, MOSEK, Gurobi, CPLEX
  License:    Open source

  R equivalent of CVXPY. Uses the same DCP rules. Popular in finance and
  econometrics communities.

### Convex.jl (Julia)
  Type:       Julia convex modeling (open source)
  Website:    https://jump.dev/Convex.jl
  GitHub:     https://github.com/jump-dev/Convex.jl
  Install:    Pkg.add("Convex")
  Solver interfaces: MOSEK, ECOS, SCS, Gurobi, SDPT3
  License:    Open source (MIT)

  Julia equivalent of CVXPY. Built on JuMP/MathOptInterface.

### GEKKO (Python)
  Type:       Mixed-integer and differential algebraic optimization
  Website:    https://gekko.readthedocs.io
  GitHub:     https://github.com/BYU-PRISM/GEKKO (stars: 707, forks: 120)
  Install:    pip install gekko
  License:    Open source (MIT)

  GEKKO is designed for dynamic optimization (time-varying) and
  differential/algebraic equation (DAE) systems. Uses the APMonitor
  backend (remote or local). Applications:

    • Model Predictive Control (MPC).
    • Chemical process optimization (reactors, distillation columns).
    • Parameter estimation (dynamic data reconciliation).
    • Combined simulation + optimization.

  GEKKO supports MIP, NLP, and dynamic optimization. The APMonitor backend
  can be run locally or on their cloud server.

### CasADi
  Type:       Symbolic framework for optimal control and NLP
  Website:    https://web.casadi.org
  GitHub:     https://github.com/casadi/casadi (stars: 2,246, forks: 450)
  Install:    pip install casadi
  License:    Open source (LGPL)

  CasADi is a symbolic computation framework for algorithmic differentiation
  (AD) and numerical optimization. It is not a modeling language in the
  traditional sense — it provides symbolic expression graphs that you
  construct procedurally.

  Key features:
    • Forward and reverse-mode automatic differentiation.
    • Interfaces to IPOPT, KNITRO, WORHP, Bonmin, and other NLP/MINLP
      solvers.
    • Direct collocation and multiple-shooting for optimal control.
    • NMPC (Nonlinear Model Predictive Control) — widely used in robotics
      and automotive control.
    • Can generate self-contained C code for embedded optimization.

  CasADi is the de facto standard for research in optimal control and is
  used extensively in academia (MPC, trajectory optimization, robotics).

## 9. COMPARISON MATRIX

Below is a comparison of 22+ solvers and frameworks across key dimensions.

Note: Stars as of July 2026 (GitHub). Performance rankings based on Hans
Mittelmann benchmarks (http://plato.asu.edu/bench.html). "Stars" = "—" for
closed-source solvers.

Solver Name | Type(s) | License | Python? API? | Languages (native)
--------------------|--------------------|-----------------|---------|----------------
Gurobi | LP, QP, MIP, MIQCP SOCP | Commercial (free academic) | Yes | C, Py, R, Java, MATLAB, Julia
--------------------|--------------------|-----------------|---------|----------------
CPLEX (IBM) | LP, QP, MIP, QCP SOCP | Commercial (free academic) | Yes | C, Py, Java, C++, .NET
--------------------|--------------------|-----------------|---------|----------------
Xpress (FICO) | LP, QP, MIP, SOCP | Commercial (free academic) | Yes | Mosel, C, Py, Java, .NET
--------------------|--------------------|-----------------|---------|----------------
MOSEK | LP, QP, SOCP, SDP Exp/Power cone | Commercial (free academic) | Yes | C, Py, Java, MATLAB, R
--------------------|--------------------|-----------------|---------|----------------
COPT | LP, MIP, QP | Commercial | Yes | Py, C, C++
--------------------|--------------------|-----------------|---------|----------------
HiGHS | LP, QP, MIP | MIT (OSS) | Yes | C++, Py, R, Julia, C
--------------------|--------------------|-----------------|---------|----------------
SCIP | MIP, MINLP, CIP | ZIB Academic (free non-com) | Yes | C, Py, Julia
--------------------|--------------------|-----------------|---------|----------------
Clp / Cbc (COIN-OR) | LP (Clp), MIP(Cbc) | EPL (OSS) | Yes | C++, Py
--------------------|--------------------|-----------------|---------|----------------
GLPK | LP, MIP | GPL (OSS) | Partial | C, GMPL
--------------------|--------------------|-----------------|---------|----------------
OR-Tools (CP-SAT) | CP-SAT, wrappers LP, MIP, Routing | Apache 2.0 (OSS) | Yes | Py, C++, Java, .NET
--------------------|--------------------|-----------------|---------|----------------
IPOPT | NLP (continuous) | EPL 2.0 (OSS) | Yes | C++, Py
--------------------|--------------------|-----------------|---------|----------------
CVXPY | Convex (DCP) LP, QP, SOCP, SDP | Apache 2.0 (OSS) | Yes | Py only (embeds solvers)
--------------------|--------------------|-----------------|---------|----------------
Pyomo | Modeling (LP/MIP/ NLP/MINLP) | BSD (OSS) | Yes | Py only
--------------------|--------------------|-----------------|---------|----------------
JuMP | Modeling (LP/MIP/ NLP/MINLP/SDP) | MIT (OSS) | No | Julia only
--------------------|--------------------|-----------------|---------|----------------
CMA-ES | Black-box cont. (evolution strat) | BSD (OSS) | Yes | Py, C, MATLAB
--------------------|--------------------|-----------------|---------|----------------
BoTorch | Bayesian Optim. (PyTorch-based) | MIT (OSS) | Yes | Py (PyTorch)
--------------------|--------------------|-----------------|---------|----------------
Nevergrad | Black-box (400+ algos) | MIT (OSS) | Yes | Py only
--------------------|--------------------|-----------------|---------|----------------
Optuna | HPO (TPE, CMA, grid, random) | MIT (OSS) | Yes | Py only
--------------------|--------------------|-----------------|---------|----------------
DEAP | GA, GP, ES, PSO (evolutionary) | LGPL (OSS) | Yes | Py only
--------------------|--------------------|-----------------|---------|----------------
CasADi | Optimal control, NLP (symbolic) | LGPL (OSS) | Yes | Py, MATLAB, Octave
--------------------|--------------------|-----------------|---------|----------------
GEKKO | Mixed-integer + DAE optimization | MIT (OSS) | Yes | Py only
--------------------|--------------------|-----------------|---------|----------------

Solver Name | Parallel Support | Best Use Case | Perf. Ranking (category)
--------------------|------------|----------------------------|--------------------
Gurobi | Yes (conc. LP, distr) | MIP, LP, QP (general) | #1 MIP, #1 LP
CPLEX (IBM) | Yes (distr) | LP, network flow, QCP | #2 MIP, #2 LP
Xpress (FICO) | Yes | Financial optimization, stochastic programming | #3 MIP, #3 LP
MOSEK | Yes (bar.) | Conic (SOCP/SDP), convex QP, portfolio | #1 SOCP, #1 SDP
COPT | Yes | MIP (competitive with Gurobi/CPLEX) | #3-4 MIP
HiGHS | Yes (conc. simplex) | LP (best OSS), QP | #1 OSS LP, #3-4 LP overall
SCIP | Limited | MIP (best non-commercial) MINLP, research | #1 OSS MIP, #4-5 MIP overall
Clp / Cbc (COIN-OR) | Limited | Building custom solvers, Foundation libraries | #3 OSS LP, #3 OSS MIP
GLPK | No | Education, small problems | Slow (50-100× behind Gurobi)
OR-Tools (CP-SAT) | Yes (conc. threads) | Scheduling, routing, constraint satisfaction | #1 OSS for combinatorial
IPOPT | Limited | Large NLP (continuous) Optimal control, MPC | #1 OSS NLP
CVXPY | No (solvers may be par) | Convex optimization | N/A (modeling language)
Pyomo | No (model building) | Research, education, prototyping | N/A (modeling language)
JuMP | No (model building) | Fast model construction, research, production | N/A (modeling language)
CMA-ES | Yes (pop.) | Continuous black-box optimization | #1 Black-box (continuous)
BoTorch | Yes (batch) | Bayesian optimization, expensive black-box | #1 Bayesian (research)
Nevergrad | Yes (pop.) | Black-box (discrete/mixed) Hyperparameter tuning | Top-tier
Optuna | Yes (distr) | Hyperparameter tuning (ML/AI) | #1 HPO
DEAP | Yes (pop.) | GA/GP/ES research, scheduling, routing | Widely used
CasADi | No (symb. graph) | MPC, trajectory opt., robotics control | #1 for optimal control
GEKKO | No | Dynamic optimization, chemical processes, MPC | Specialized

## 10. DECISION GUIDE

Which solver or framework should you choose? The answer depends on your
problem type, budget, and performance requirements.

### Linear Programming (LP)
  • Best (free): HiGHS — competitive with commercial solvers on LP.
    Parallel simplex handles large LPs well. Available via `highspy`.
  • Best (commercial): Gurobi — concurrent LP (simplex + barrier in
    parallel) is fastest overall.
  • Education: GLPK — free, simple, GPL-licensed.
  • Embedded/limited resources: ECOS (small convex LPs).

### Mixed-Integer Programming (MIP)
  • Best overall: Gurobi — industry gold standard. Fastest presolve,
    cutting planes, and heuristics.
  • Best free: SCIP — most capable open-source MIP solver. Use with
    PySCIPOpt.
  • Constraint-heavy (scheduling, packing): OR-Tools CP-SAT often
    outperforms MIP solvers on pure combinatorial problems.
  • Budget constrained: HiGHS (MIP mode — improving but not yet
    competitive with SCIP).

### Convex QP / SOCP / SDP
  • Best overall: MOSEK — fastest and most robust for conic optimization.
  • Free (moderate accuracy): SCS — ADMM-based, scales to millions of
    variables.
  • Free (small, accurate): ECOS — embedded interior-point solver.
  • Ease of use: CVXPY + any backend — write the problem, choose the
    solver. CVXPY handles DCP verification automatically.
  • Finance (portfolio): MOSEK or CVXPY+MOSEK.

### Nonlinear Programming (NLP)
  • Best free: IPOPT — interior-point method, scales well, widely used.
  • Best commercial: KNITRO — artelys.com. Multiple algorithms
    (interior-point, active-set, SQP). Often faster and more robust
    than IPOPT.
  • Large-scale: IPOPT + MUMPS/PARDISO. Can handle 10⁶+ variables.
  • Optimal control: CasADi + IPOPT/KNITRO.

### Black-Box / Derivative-Free Optimization
  • Continuous black-box: CMA-ES (`cma` package) — best all-round
    for continuous, expensive functions.
  • Expensive evaluations (simulations): Bayesian optimization with
    BoTorch or GPyOpt. GP surrogate + acquisition function minimizes
    evaluations.
  • Mixed domains / many algorithms: Nevergrad — 400+ algorithms,
    auto-selection.
  • Hyperparameter tuning: Optuna (easy, well-integrated), Ray Tune
    (distributed), Hyperopt (TPE).
  • Constrained black-box: NOMAD (MADS algorithm).

### Routing / Scheduling
  • Vehicle routing: OR-Tools routing library — LNS heuristics, handles
    dozens of constraints (time windows, capacity, durations).
  • Scheduling (job shop, employee): OR-Tools CP-SAT — powerful interval
    variables and cumulative constraints.
  • General constraint programming: OR-Tools CP-SAT or IBM CP Optimizer.

### Large-Scale Deep Learning Training
  • First-order optimizer: AdamW — default choice for most architectures.
  • LR schedule: Cosine annealing with linear warmup.
  • For LLM training: Sophia (uses Hessian diagonal) shows faster
    convergence on large models.
  • Hyperparameter tuning: Optuna (define-by-run, pruning) or Ray Tune
    (distributed, PBT, HyperBand).
  • Architecture search: NNI (Microsoft) or KerasTuner.

### Research / Prototyping
  • Pyomo: Python-native, supports the widest range of solvers, great
    for teaching and research.
  • JuMP: If you are comfortable with Julia, JuMP offers dramatically
    faster model construction and tight solver integration.
  • CVXPY: For convex optimization problems — the DCP rules catch
    mistakes early.
  • CasADi: For optimal control and robotics research.

### Education
  • GLPK: Free, simple, GMPL syntax teaches algebraic modeling.
  • CVXPY: Teaches convex optimization concepts via DCP.
  • Pyomo: Teaches optimization alongside Python programming skills.
  • AMPL (student version): Free, clean syntax, excellent documentation.

### Summary Decision Flowchart

  Q1. Do you have an analytical model (gradients available)?
     YES → Q2
     NO  → Q4 (black-box)

  Q2. Are all functions linear?
     YES → LP solver (HiGHS free, Gurobi commercial)
     NO  → Q3

  Q3. Are variables integer?
     YES → MIP solver (Gurobi best, SCIP free)
     NO  → Are constraints/nonlinearities convex?
        YES → Convex optimization (MOSEK, CVXPY)
        NO  → NLP (IPOPT free, KNITRO commercial)

  Q4. Is the function evaluation expensive (hours per eval)?
     YES → Bayesian optimization (BoTorch)
     NO  → Is the problem continuous?
        YES → CMA-ES
        NO  → Nevergrad, GA, or PSO

  Q5. Is the problem combinatorial (routing, scheduling)?
     → OR-Tools CP-SAT or routing library.

## 11. REFERENCES AND FURTHER READING

### Benchmark Sites
  • Hans Mittelmann's Decision Tree for Optimization Software
    https://plato.asu.edu/guide.html
    The most comprehensive collection of optimization software links and
    benchmarks. Includes LP, MILP, NLP, SOCP, SDP, and MINLP benchmarks
    updated regularly.

  • Mittelmann Benchmarks (main index)
    https://plato.asu.edu/bench.html
    Includes LPopt, LPfeas, MIPFEAS, MILP, SOCP, SDP, QP, AMPL-NLP, and
    MINLP benchmarks. Most results updated through mid-2026.

  • MIPLIB (Mixed Integer Programming LIBrary)
    https://miplib.zib.de
    The standard benchmark library for MIP solvers. MIPLIB2017 is the
    current version.

  • COCO Platform (Comparing Continuous Optimizers)
    https://github.com/numbbo/coco
    Benchmark platform for black-box continuous optimization.

### Solver Official Documentation
  • Gurobi Documentation:    https://docs.gurobi.com
  • CPLEX Documentation:     https://www.ibm.com/docs/en/icos
  • Xpress Documentation:    https://www.fico.com/fico-xpress-optimization/docs
  • MOSEK Documentation:     https://docs.mosek.com
  • HiGHS Documentation:     https://highs.dev
  • SCIP Documentation:      https://www.scipopt.org/doc
  • OR-Tools Documentation:  https://developers.google.com/optimization
  • CVXPY Documentation:     https://www.cvxpy.org
  • Pyomo Documentation:     https://www.pyomo.org/documentation
  • JuMP Documentation:      https://jump.dev/JuMP.jl/stable
  • CasADi Documentation:    https://web.casadi.org/docs

### Key GitHub Repositories
  (Stars as of July 2026 — see Section 9 for individual counts)

  • OR-Tools:              https://github.com/google/or-tools
  • Optuna:                https://github.com/optuna/optuna
  • CVXPY:                 https://github.com/cvxpy/cvxpy
  • DEAP:                  https://github.com/DEAP/deap
  • Pyomo:                 https://github.com/Pyomo/pyomo
  • HiGHS:                 https://github.com/ERGO-Code/HiGHS
  • SCIP:                  https://github.com/scipopt/scip
  • BoTorch:               https://github.com/pytorch/botorch
  • Nevergrad:             https://github.com/facebookresearch/nevergrad
  • CMA-ES:                https://github.com/CMA-ES/pycma
  • BayesianOptimization:  https://github.com/bayesian-optimization/BayesianOptimization
  • JuMP.jl:               https://github.com/jump-dev/JuMP.jl
  • CasADi:                https://github.com/casadi/casadi
  • GEKKO:                 https://github.com/BYU-PRISM/GEKKO
  • NNI:                   https://github.com/microsoft/nni
  • Hyperopt:              https://github.com/hyperopt/hyperopt
  • IPOPT:                 https://github.com/coin-or/Ipopt
  • Clp:                   https://github.com/coin-or/Clp
  • Cbc:                   https://github.com/coin-or/Cbc
  • PyPSA:                 https://github.com/PyPSA/PyPSA
  • SciPy:                 https://github.com/scipy/scipy
  • PyTorch:               https://github.com/pytorch/pytorch

### Books
  • Boyd & Vandenberghe (2004) — "Convex Optimization"
    Cambridge University Press. Free PDF: https://web.stanford.edu/~boyd/cvxbook
    The definitive text on convex optimization.

  • Bertsimas & Tsitsiklis (1997) — "Introduction to Linear Optimization"
    Athena Scientific. The standard LP/introductory optimization textbook.

  • Nocedal & Wright (2006) — "Numerical Optimization"
    Springer. The standard reference for numerical optimization methods.

  • Wolsey (1998) — "Integer Programming"
    Wiley. Comprehensive treatment of integer programming theory.

  • Williams (2013) — "Model Building in Mathematical Programming"
    Wiley. Practical guide to constructing optimization models.

  • Sutton & Barto (2018) — "Reinforcement Learning: An Introduction"
    MIT Press. The standard RL textbook (covers RL as optimization).

  • Goodfellow, Bengio & Courville (2016) — "Deep Learning"
    MIT Press. Chapter 8 covers optimization for deep learning.

### Conferences and Journals
  • INFORMS Annual Meeting (operations research)
  • International Conference on Machine Learning (ICML)
  • Neural Information Processing Systems (NeurIPS)
  • Mathematical Optimization Society (MOS) conferences
  • Journal: Mathematical Programming (Series A & B)
  • Journal: Operations Research
  • Journal: Optimization and Engineering
  • Journal: Journal of Global Optimization

## END OF ANALYSIS
