# Numerical Simulation in Physics and Engineering: Convergence Is Not Correctness

**The discipline behind every solver: how a continuous physical model becomes a computable one, why the mesh decides the answer, and why a residual of zero is not evidence that the number is right.**

> **Jack Liu Shurui, Solution Architect**
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Engineering Simulation · **Topic:** Numerical simulation in physics and engineering — the discipline, the error budget and the V&V that most projects skip
> **Focus:** The chain from physical system to number; the four error classes and which one mesh refinement actually touches; elliptic, parabolic and hyperbolic behaviour and what each demands of a scheme; the discretisation families and when each applies; explicit versus implicit time stepping with the CFL condition derived; the mesh and why one bad cell ruins a good scheme; code verification, solution verification and validation as three different activities; floating point and reproducibility; strong and weak scaling and the I/O wall; the software landscape with licences stated only where verified; the AI-for-simulation frontier evidence-graded; the industries; the finance parallel; and a worked Cymbal Bank review of a vendor CFD study
> **Companion Guides:** [Deterministic Simulation Testing](deterministic_simulation_testing_guide.md) §1 (the false-friend boundary) · [Physical AI](physical_ai_guide.md) §5 (sim-to-real, owned there) · [GPU Optimization](gpu_optimization_guide.md) · [Quantitative Developer Skillset](quantitative_developer_skillset_guide.md) · [Data Center Engineering](data_center_guide.md) · [Risk Management Models](../banking/risk_management_models_guide.md) · [LLM Evaluation vs Validation](ai_llm/llm_evaluation_vs_validation_guide.md)
> **Last Updated:** September 2026

**Source convention:** every mechanism in this guide is attributed in prose and tabulated in the claims audit (§16.2) with its source, the date checked, and its quality — **first-party** (a project documenting its own software or licence), **project claim** (a first-party capability or performance assertion), **vendor claim** (a commercial assertion about its own product), **peer-reviewed** (a paper or standard with a named venue or body), **standard** (an accredited published standard), or **secondary**. Project and vendor claims are labelled where they appear and are never restated as measured facts. Anything that could not be verified at source is recorded in §16.3 rather than asserted. Where the guide reasons rather than reports — the effort distribution in meshing, the scaling walls, the data-centre observation in §14 — the reasoning is marked as the guide's analysis. Nothing here is written from memory.

---

## Table of Contents

1. [The Overview, the Boundary and the Decoder](#1-the-overview-the-boundary-and-the-decoder)
2. [What Numerical Simulation Actually Is, and the Error Budget](#2-what-numerical-simulation-actually-is-and-the-error-budget)
3. [The Governing Equations and What They Demand](#3-the-governing-equations-and-what-they-demand)
4. [The Discretisation Families](#4-the-discretisation-families)
5. [The Time Dimension](#5-the-time-dimension)
6. [Stability, Consistency, Convergence](#6-stability-consistency-convergence)
7. [The Mesh](#7-the-mesh)
8. [Boundary and Initial Conditions, and the Modelling Choices That Hide Inside Them](#8-boundary-and-initial-conditions-and-the-modelling-choices-that-hide-inside-them)
9. [Verification and Validation](#9-verification-and-validation)
10. [The Floating-Point and Reproducibility Substrate](#10-the-floating-point-and-reproducibility-substrate)
11. [HPC and the Scaling Question](#11-hpc-and-the-scaling-question)
12. [The Software Landscape, Verified](#12-the-software-landscape-verified)
13. [The AI-for-Simulation Frontier, Evidence-Graded](#13-the-ai-for-simulation-frontier-evidence-graded)
14. [The Industries, and the Finance Parallel](#14-the-industries-and-the-finance-parallel)
15. [The Cymbal Bank Worked Example](#15-the-cymbal-bank-worked-example)
16. [The Anti-Patterns, the Claims Audit, What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-the-anti-patterns-the-claims-audit-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)

---

## 1. The Overview, the Boundary and the Decoder

**The thesis in one line: a converged simulation is a solved discrete problem, not a statement about the world — and the distance between those two things is where engineering judgement lives or dies.**

Every number a solver prints is the end of a chain of choices: which equations, which discretisation, which mesh, which timestep, which boundary conditions, which material properties, which turbulence or constitutive model, which solver tolerance. Convergence speaks to exactly one link of that chain — whether the algebraic problem was solved — and stays silent about all the others. "Convergence is not correctness" is not a caveat bolted onto the discipline. It *is* the discipline.

This section sets three things: what this guide owns, the eleven terms a reader needs to decode any simulation report, and the three false friends in this repository that will otherwise mislead a reader who arrives with adjacent reading.

### 1.1 What This Guide Owns, and What It Does Not

The repository has 630 tracked Markdown files and, in the dedup counts taken for this pass, `finite element` appears in 0, `finite volume` in 0, `FEM` in 0, `OpenFOAM` in 0, `Navier-Stokes` in 0, `molecular dynamics` in 0, `CFL condition` in 0, `spectral method` in 0 and `manufactured solution` in 0. The four files containing `CFD` are all contract-for-difference or cumulative-flow-diagram uses. There is no numerical-simulation guide anywhere in the tree. This guide is the first treatment of the subject, and it owns it.

By contrast, several adjacent subjects are already owned elsewhere and are **cross-referenced, not re-derived**:

| Subject | Owner in this repo | How this guide relates |
|---|---|---|
| Simulation as a software-testing technique | `deterministic_simulation_testing_guide.md`, `antithesis_guide.md`, `deterministic_engineering_guide.md` | Opposite meaning of the word (§1.3.1). Boundary declared, nothing re-derived |
| Physical AI, robotics, sim-to-real transfer | `physical_ai_guide.md` (its §5 owns sim-to-real) | Referenced by name; the transfer problem is theirs |
| Model risk in banking | `../banking/risk_management_models_guide.md`, `../banking/enterprise_risk_management_guide.md`, the quant guides | §14 draws the verification-and-validation parallel; the finance material is theirs |
| Numerical methods for quants | `quantitative_developer_skillset_guide.md` | The one file in the repo with `finite difference` — Black-Scholes PDE, not physics |
| The GPU and accelerator substrate | `gpu_optimization_guide.md`, `gpu_cloud_providers_comparison_guide.md`, `hami_gpu_sharing_guide.md`, `low_latency_cpp_development_guide.md` | §11 uses these as the hardware layer; kernels and CUDA are theirs |
| Machine learning as a subject | `ai_llm/` | §13 is about learned surrogates for physics, not ML in general |
| Software V&V (artefact against requirements) | `ai_llm/llm_evaluation_vs_validation_guide.md` (IEEE 1012), `project_management_methodologies_guide.md` §5.1 | The other V&V. §9 draws the line explicitly |

There is **no** `gpu_cloud_guide.md` in this repository; the accelerator material is the four files listed above. Citing a file that does not exist is a common failure mode in this repo's guides and is called out here deliberately.

### 1.2 The Decoder: Eleven Terms

A simulation report is written in a dialect. These are the terms that decide whether you can read it.

| Term | What it actually means | The question it answers |
|---|---|---|
| **The model** | The mathematical description of the physics you chose to represent, plus the constitutive relations that close it | What physics did we decide matters? |
| **The governing equation** | The PDE (or system) expressing conservation or equilibrium for the model | What must be true at every point? |
| **The discretisation** | The rule that replaces continuous derivatives with relations among a finite set of unknowns | How is the continuum turned into a finite list of numbers? |
| **The mesh / grid** | The geometric decomposition the discretisation lives on — cells, elements, nodes, particles | Where are the unknowns located, and how well does the geometry fit? |
| **The solver** | The algorithm that produces the discrete unknowns, usually by driving a residual to a tolerance | How is the algebraic system solved? |
| **The timestep** | The increment of the independent variable in time-dependent problems, chosen against a stability bound | How far do we advance per step, and may we advance that far? |
| **Order of accuracy** | The exponent governing how fast the error falls as the mesh is refined | If I refine, how much better do I get? |
| **Consistency** | The property that the discrete operator converges to the continuous operator as the mesh vanishes | Does the scheme approximate the equation I think it does? |
| **Stability** | The property that errors already present do not grow without bound as the computation proceeds | Does the computation stay bounded? |
| **Convergence** | The property that the discrete solution approaches the exact solution of the discrete problem as the iteration proceeds — and, separately, that it approaches the solution of the continuous problem as the mesh vanishes | Is the iteration finished / is the discretisation fine enough? |
| **The residual** | The amount by which the current discrete solution fails to satisfy the discrete equations | How far from solved is the algebraic problem right now? |

The trap is in the second half of "convergence". Two different convergences share a word: **iteration convergence** (the residual fell, the linear or nonlinear solve finished) and **grid convergence** (the discretisation error is small and shrinking at the expected rate). A solver can report the first while the second was never examined. Most of this guide is about the gap between them.

### 1.3 The Three False Friends

Three words in this repository already belong to someone else. Read the relevant owner before using them here.

#### 1.3.1 False Friend 1 — "Simulation" Means Software Testing Here

This is the most important boundary in the guide. In this repository, `simulation` overwhelmingly means **software-testing simulation**: simulating a program's environment in order to test the program. `deterministic_simulation_testing_guide.md` carries roughly 66 mentions, `antithesis_guide.md` roughly 50, and `deterministic_engineering_guide.md` roughly 35. Their thesis is explicit — *a deterministic simulator keeps the system and replaces its environment* — and the system under test is **the software**.

Numerical simulation is the **reverse**. The simulation is not a testing instrument standing in for the world; the simulation **is the object of study**, and the system under test is physics itself. Nothing is being mocked. The comparison is between two solutions of the same equations — one on a mesh, one supplied by nature through an experiment — and the standard for success is whether the numbers describe reality, not whether the software behaves as designed.

The distinction is easy to see in how each handles verification. The deterministic-simulation guide states its own boundary precisely: a deterministic simulator "verifies *this implementation against this design* exhaustively under modelled faults. It says nothing about whether the design is right." That is code verification in this guide's vocabulary (§9) — the smallest of the three V&V activities. Numerical simulation needs all three, including validation against physical measurement, which has no DST analogue at all.

A reader who has read the DST guide and arrives here should expect: no simulated network, no simulated clock, no fault injection, no seeds, no replay. Instead: meshes, residuals, order-of-accuracy studies, and an experiment.

#### 1.3.2 False Friend 2 — Physical AI Owns Sim-to-Real

`physical_ai_guide.md` (roughly 51 mentions of simulation) owns physical AI and robotics, and its §5 owns sim-to-real transfer in full: why simulate, why the transfer gap exists, the simulation platforms, synthetic data. That material is not repeated here. The interface between the two guides is narrow and real: a physics engine used to generate robot training data is running a numerical simulation underneath, and this guide explains what makes that underlying solver trustworthy or not. The guide that owns the application is theirs; the guide that owns the solver is this one.

#### 1.3.3 False Friend 3 — "Monte Carlo" Here Is Almost Never the Physics Method

`Monte Carlo` appears in 27 files in this repository, and the honest reading of those 27 is that the phrase has three senses, none of which is the physics-Monte-Carlo method: **financial Monte Carlo** (`../banking/risk_management_models_guide.md` with 12 mentions, `quantitative_developer_skillset_guide.md` with 9, and the wider risk and portfolio material), **the data-observability vendor named Monte Carlo** (montecarlodata.com, appearing across the `data/` guides), and **reinforcement-learning sampling** (`reinforcement_learning_algorithms_guide.md` with 14). The underlying method — estimate an expectation by sampling — is genuinely shared between the financial and the physical uses, and §4.7 treats the physical one. But existing mentions in this repo are not coverage of it, and no reader should count them as such.

---

## 2. What Numerical Simulation Actually Is, and the Error Budget

### 2.1 The Chain from System to Number

A numerical simulation is a chain of translations, and every translation introduces a term that no amount of computing removes:

| Stage | What happens | What can go wrong | Which error class this creates |
|---|---|---|---|
| Physical system | The real object: a wing, a reactor, a data hall | — | — |
| Model | Choose the physics: which conservation laws, which constitutive relations, which closure for unresolved scales | Wrong physics or a poor closure | **Model error** |
| Governing equations | Write the model as PDEs plus boundary and initial conditions | Wrong or ill-posed formulation | Model error |
| Discretisation | Replace derivatives with algebraic relations on a finite set of unknowns | Low order, inconsistency, non-conservation | **Discretisation error** |
| Mesh / grid | Decompose the geometry into cells or elements | Poor resolution, bad cell quality, geometry approximation | Discretisation error |
| Solver | Solve the algebraic system to a tolerance | Incomplete convergence, ill-conditioning | **Iteration error** |
| Number | The reported quantity | Wrong digits from finite precision | **Round-off error** |

The number is the end of that chain of choices. **The chain is the argument.** A result is defensible when each link is defensible — not when the last link terminated cleanly.

### 2.2 The Four Error Classes

These four are routinely conflated, and the conflation is the mechanism by which "it converged" gets promoted into "it is right".

| Error class | What it is | What reduces it | What does *not* reduce it |
|---|---|---|---|
| **Model error** | The difference between the real system and the mathematical model of it — unresolved physics, a constitutive relation fitted to the wrong regime, a closure for turbulence | Validating against physical measurement and changing the model; quantifying uncertainty in the model form | Any mesh refinement, any tighter solver tolerance, any faster computer |
| **Discretisation error** | The difference between the exact solution of the PDE and the exact solution of the discrete equations | Refining the mesh, raising the order of accuracy, improving cell quality, choosing a discretisation suited to the equation | A tighter solver tolerance stops the iteration earlier or later but does not move the discretised solution toward the continuous one |
| **Iteration error** | The difference between the exact solution of the discrete equations and the answer the solver actually returned after a finite number of iterations | More iterations, tighter tolerances, better preconditioning, a better linear solver | Mesh refinement can make it *worse* if the tolerance is held fixed, since the discrete problem hardens |
| **Round-off error** | The difference caused by representing real numbers in finite precision, and by the order in which the operations are performed | Higher precision, better-conditioned formulations, rearranged summations, compensated summation | Mesh refinement does nothing; refining can increase the number of operations and thus accumulate more round-off |

**The load-bearing sentence: refining the mesh addresses exactly one of these four.** It is the most common remediation offered in practice and it touches only discretisation error. A converged solution of the wrong model remains wrong. A converged solution of the right model fed wrong material properties remains wrong. Both are *confidently* wrong, because convergence supplies no signal that anything is amiss — that is precisely the failure mode this guide exists to prevent.

### 2.3 The Arithmetic That Shows Why Reinforcing One Class Is Not Enough

Consider a reported quantity Q with a true value of 100 °C and a stated uncertainty budget. Suppose the four contributions arrive as follows for a first-order-accurate scheme on a mesh of spacing h:

- model error: ±3.0 °C (the closure used for turbulent heat transfer is not valid in the regime simulated)
- discretisation error: 1.2 °C at the current mesh, falling as h for a first-order scheme
- iteration error: 0.05 °C, judged from the residual norm and the solver's tolerance
- round-off error: below 10⁻¹² °C in double precision — negligible here by many orders

Halving h changes the discretisation term from 1.2 °C to 0.6 °C. It halves **one** vector. The total budget moves from √(3.0² + 1.2² + 0.05²) ≈ 3.23 °C to √(3.0² + 0.6² + 0.05²) ≈ 3.06 °C — a 5% reduction in the combined uncertainty for a 2× (in 3-D, 8× cells) increase in cost. This is arithmetic, used here to show the scaling law rather than to report a measurement: the model term dominates, and no refinement touches it.

### 2.4 The Converged-But-Wrong Failure, Mechanically

The mechanism is worth stating step by step, because it explains why the failure is invisible from inside the solver:

1. The engineer fixes the model, the boundary conditions and the material properties. These become *inputs* to everything downstream.
2. The engineer refines the mesh until the solution stops changing at the quantities of interest and reports that refinement stopped changing it. The residual falls by, say, six orders of magnitude.
3. Every signal the software produces is now green. The residual is small, the solution is stable, the mesh study is monotone.
4. But the software was only ever solving *the discrete version of the model as given*. If the closure is invalid, or the inlet enthalpy is wrong because the fluid is not the fluid assumed, or the wall is not adiabatic as the boundary condition asserts, the discrete solution faithfully, stably and reproducibly reproduces the wrong physics.
5. The result is a number with a small error bar attached to the wrong quantity — and the error bar is small precisely because the discretisation was done well.

The only exit is data: a measured temperature, a measured pressure drop, a measured deflection. That is §9's subject, and it is why validation, not verification, is the load-bearing step.

### 2.5 The Errors That Are Not Numerical

Model error and input error deserve their own treatment because they are the two classes that no amount of computing reduces, and they are the two most often left unexamined.

**Model error** is the gap between the system and the mathematics chosen to represent it. In fluid mechanics the canonical instance is turbulence: the governing equations are taken as the Navier–Stokes equations, but resolving every turbulent length scale in an engineering geometry is out of reach, so a *closure* is introduced that models the effect of the unresolved scales on the resolved ones. The closure is a physical approximation, not a numerical one, and it carries its own validity domain. §8 treats this as the modelling choice that hides inside what is presented as a numerical setting.

Model error also appears outside fluids: a linear elastic constitutive law applied where plastic deformation occurs; a first-order reaction substituted for a chain of reactions; a thermal contact resistance assumed zero at an interface that in reality has a gap. In each case the simulation can be perfectly verified and still answer a question nobody is asking.

**Input error** is the error in the numbers fed *into* a correct model: material properties, boundary values, geometric dimensions, source terms, initial fields. The properties problem is the one that gets least attention. Thermal conductivity, emissivity, specific heat, viscosity, permeability and contact resistance vary with temperature, with manufacturing batch, with surface condition and with age, and at the level of precision a simulation implies they are frequently *not measured* for the actual article — they are taken from a handbook table for a nominal material. A boundary condition of a fixed heat-transfer coefficient is often a place where an unmeasured correlation has been silently converted into a hard number.

### 2.6 The Sensitivity Question and Uncertainty Quantification

Once model and input error are on the table, the honest response is to ask how much the output moves when the inputs move — and that is a different question from how much it moves when the mesh moves.

| Question | Method family | What it needs |
|---|---|---|
| How much does the answer move when the mesh is refined? | Grid-convergence study (§9.4) | Only the code and more compute |
| How much does the answer move when an input or a model *form* varies? | Local and global sensitivity analysis, model-form uncertainty methods | A range or distribution per input; several defensible models; many solver runs |

Uncertainty quantification is the discipline of propagating the first three into a defensible interval around the answer. Its cost is the multiplication of solver runs, and its value is that it forces the unmeasured quantities to be named rather than buried. A simulation report that states a single number with no uncertainty and no model validity statement has not answered the UQ question; it has declined it.

### 2.7 The Point of the Budget, and the Banking Material

Refinement, better schemes and tighter tolerances do not remove the model and input error classes. What the budget buys is ordering: numerical error can be made smaller than the physical uncertainty and then be dominated by it — the achievable state, and the state worth aiming for, because beyond that point the compute is being spent on the wrong term.

This repository already contains a mature treatment of models whose inputs are uncertain and whose validity domain matters: `../banking/risk_management_models_guide.md` and `../banking/enterprise_risk_management_guide.md`. §14 draws the structural parallel between a bank's model-risk discipline and this discipline's validation step, and explains where the analogy holds and where it stops; it is not re-derived here.

---

## 3. The Governing Equations and What They Demand

The classification of second-order PDEs as elliptic, parabolic or hyperbolic is usually taught as taxonomy. It is more useful as **behaviour**: the class tells you how information travels in the solution, and that in turn dictates what a numerical scheme must do to survive.

### 3.1 Behaviour, Not Formalism

| Class | Behaviour | Physical example | What the scheme must do |
|---|---|---|---|
| **Elliptic** | Influence is felt immediately everywhere; the solution is determined by boundary data alone; there is no marching direction | Steady heat conduction, linear elasticity at equilibrium, potential flow | Assemble a global system and solve it simultaneously; a boundary value on one side changes the interior everywhere |
| **Parabolic** | Diffusion in time: disturbances spread, smoothing gradients; infinite propagation speed but with decaying amplitude | Transient conduction, viscous diffusion | March in time, usually implicitly for stability; expect smoothing of sharp features and never expect a discontinuity to stay sharp |
| **Hyperbolic** | Waves: information travels at finite speed along characteristics; discontinuities can form and persist | Inviscid flow, acoustics, shock waves | Respect the direction of information travel (upwinding, flux functions); an uncentred scheme that ignores characteristic direction will be unstable or oscillatory |

The practical payoff: in an elliptic problem there is no timestep to choose and no stability limit to obey — the difficulty is the size and conditioning of a global system. In a hyperbolic problem, the characteristic directions *are* the numerical design constraint, and getting the direction wrong is not a matter of accuracy but of instability. In a parabolic problem the timestep limit comes from the diffusion operator and scales with the square of the mesh spacing (§5.4), which is why diffusion-dominated problems are pushed toward implicit methods.

### 3.2 Why the Equation and Geometry Choose the Method

Three properties of the problem narrow the method space before any preference enters:

1. **The equation class** determines whether there is a timestep at all, and whether the scheme needs upwinding (hyperbolic), implicit treatment (parabolic), or a global solve (elliptic).
2. **The geometry** determines the mesh family. Complex three-dimensional industrial geometry with boundary layers narrows you toward unstructured finite volume or finite element grids; a simple Cartesian box admits finite differences and spectral methods; a periodic box in three dimensions invites spectral or pseudo-spectral treatment.
3. **What must be conserved** determines the discretisation family. If mass, momentum and energy must balance to machine precision in and out of every cell, a finite-volume formulation of the flux form is the natural home, because conservation is built into its structure rather than hoped for (§4.2).

Fashion enters at none of these three points. A method's popularity is not an argument about whether its conservation and stability properties match the equation you have.

### 3.3 Well-Posedness, and the Data You Must Supply

A problem is well-posed when a solution exists, is unique, and depends continuously on the data. The third condition is the one that bites: an elliptic problem needs boundary conditions on the whole boundary, of the right *kind* on each part (a value, a flux, or a combination — but not a value in two places on the same patch); an initial-value problem needs a state consistent with the boundary data; and an incompressible-flow problem needs a pressure treatment that fixes the one remaining degree of freedom. Ill-posed *discrete* problems are the routine consequence of getting this wrong, and they present as a solver that will not converge or that converges to a wildly mesh-dependent answer. When a residual stalls, the first question is not "which solver?" but "is the problem I posed well-posed, and is my boundary data consistent?" (§8.1 revisits this from the boundary side.)

---

## 4. The Discretisation Families

Six families cover essentially all of production simulation, plus Monte Carlo in the physical sense. They are **different tools for different equations**.

### 4.1 Finite Difference

**The idea:** replace each derivative with a difference quotient on a structured grid. The truncation error is obtained by Taylor expansion, which is why accuracy order is transparent — for instance the centred second difference (u_{j+1} − 2u_j + u_{j−1})/Δx² = u'' + (Δx²/12)·u'''' + O(Δx⁴), so the scheme is formally second order in Δx with a known leading coefficient, and the centred first difference (u_{j+1} − u_{j−1})/(2Δx) = u' + (Δx²/6)u''' + O(Δx⁴).

**Suits:** simple and moderately complex geometry on structured or block-structured grids; the heart of algorithm development because accuracy analysis is straightforward.

**Conservation and stability:** conservation is *not* automatic — a scheme must be written in a conservative flux form to conserve, and many textbook differences are not. Stability analysis is the easiest of any family, because the operator is explicit and the Fourier (von Neumann) test applies directly (§5.3).

**Cost:** very low per unknown; efficient on structured grids; awkward on complex geometry, where immersed-boundary or cut-cell treatment introduces its own accuracy problems at the boundary.

### 4.2 Finite Volume

**The idea:** integrate the governing equation over each control volume and convert the volume integral of the divergence into a sum of fluxes across the cell faces. The unknown becomes the cell average, and the flux balance is exact by construction.

**Suits:** the workhorse of production computational fluid dynamics and heat transfer, on unstructured polyhedral meshes for complex industrial geometry.

**Conservation and stability:** **discrete conservation is structural** — what leaves one cell enters its neighbour because the face flux is single-valued. This is the family's defining advantage, and the reason it dominates in flows where global balances matter. Stability still depends on the flux function chosen and on the time integration.

**Cost:** higher per unknown than finite differences because of flux reconstruction and mesh data structures; scales acceptably to parallel machines because flux loops are local.

### 4.3 Finite Element

**The idea:** represent the solution as a combination of basis functions on elements and satisfy the equation in a weighted (weak/variational) sense. This moves derivatives off the solution and onto the test functions, which lowers the smoothness demanded of the approximation and makes the method natural for problems with a variational structure.

**Suits:** structural mechanics, heat conduction, electromagnetics, and increasingly fluids; the standard choice where geometry and material heterogeneity are complex and where coupling between physics is required.

**Conservation and stability:** conservation can be built in for the right formulations but is less automatic than in finite volume; stability of mixed problems depends on choosing element pairs that satisfy an inf-sup (LBB-type) condition, a genuine design constraint rather than a detail.

**Cost:** assembly and a global sparse solve; the number of unknowns grows fast with polynomial order in three dimensions; adaptive refinement requires error estimators and mesh management.

### 4.4 Spectral and Pseudo-Spectral

**The idea:** expand the solution in a globally supported basis (Fourier modes, Chebyshev polynomials) so the approximation is exact for a band-limited field and the error decays faster than any power of the mesh spacing for smooth solutions.

**Suits:** simple or periodic domains with smooth solutions — turbulence in a periodic box, stability analysis, wave propagation over long distances.

**Conservation and stability:** excellent accuracy per degree of freedom for smooth fields; **the weakness is sharp features**: a discontinuity or a steep gradient produces oscillations that spread across the whole domain (the Gibbs phenomenon), and the global coupling makes local refinement impossible without abandoning the global basis.

**Cost:** very low error per unknown on smooth problems; the transform cost and the global data exchange limit scaling unless the transform is parallelised carefully.

### 4.5 Particle and Molecular Dynamics

**The idea:** do not discretise a field at all. Represent the system as discrete particles — atoms, molecules, beads, grains — and integrate Newton's equations of motion for each one under a force field.

**Suits:** materials at the atomic scale, biomolecular systems, granular and powder flows, and any regime where the continuum assumption itself fails.

**Conservation and stability:** momentum and energy conservation follow from a symmetric, conservative force field; stability is governed by the timestep relative to the fastest vibrational period in the system, which is why molecular-dynamics timesteps are measured in femtoseconds — a limit that is physical, not numerical, in origin. **Cost:** the number of particles needed to reach a physically meaningful volume is enormous and the accessible time span is short, the two limitations that make molecular dynamics a tool for mechanism and property estimation rather than for engineering-scale prediction.

### 4.6 Lattice and Cellular Methods

**The idea:** put a simplified kinetic description on a fixed lattice and let it evolve by local rules — the lattice Boltzmann family is the best-known instance, evolving particle distribution functions on a lattice with a collision and a streaming step, from which macroscopic fields are recovered as moments.

**Suits:** complex geometry with modest compressibility, multiphase and porous-media flows, and problems where a local rule maps well onto parallel hardware.

**Conservation and stability:** mass and momentum conservation come from the collision operator's invariants; stability depends on the relaxation parameter and degrades toward the edges of the valid range, with a known weak point at high Mach number and at very low viscosity. **Cost:** highly local and therefore parallel-friendly, but memory per node is higher than a finite-volume scheme at the same macroscopic resolution, because the distribution function carries more information than the macroscopic fields.

### 4.7 Monte Carlo in the Physical Sense

**The idea:** represent the system by sampling. Track many random walks of particles or histories and estimate the quantity of interest as an ensemble average — used for radiation transport, neutronics, rarefied gas dynamics and molecular simulation.

**Suits:** high-dimensional problems where the state space is far too large to discretise, and transport problems where the geometry and the cross-sections defeat a mesh.

**Conservation and stability:** there is no stability limit in the ODE sense; instead the error is statistical, falling as the inverse square root of the number of samples, so **each additional decimal digit costs one hundred times the sampling**, and conservation is enforced statistically rather than exactly. **Cost:** embarrassingly parallel and therefore attractive on large machines, but the variance is the enemy and variance-reduction techniques are where the engineering effort goes. The method is shared with financial Monte Carlo (§1.3.3), and the shared part is genuinely the same mathematics — but the domain, the variance structure and the validation standard are not.

### 4.8 The Comparison, and Why This Is Not a Ranking

| Family | Geometry it suits | Conservation | Stability character | Best at | Weak at |
|---|---|---|---|---|---|
| Finite difference | Structured / block-structured | Only if written in flux form | Explicit Fourier analysis available | Simplicity, accuracy analysis, algorithm development | Complex geometry |
| Finite volume | Unstructured, complex | Structural (single-valued face fluxes) | Depends on flux function and time scheme | Conservation, flows, heat transfer | Higher cost per unknown |
| Finite element | Complex, heterogeneous, coupled | Formulation-dependent | Requires stable element pairs for mixed problems | Structures, multiphysics, adaptivity | Assembly cost, mixed-problem constraints |
| Spectral | Simple, periodic, smooth | Depends on formulation | Excellent for smooth fields | Accuracy per unknown on smooth problems | Sharp features, complex geometry |
| Particle / molecular dynamics | Atomic scale, no continuum | From a conservative force field | Timestep set by fastest vibration | Mechanism, atomic-scale properties | Scale and duration |
| Lattice methods | Complex, moderate compressibility | From collision invariants | Sensitive to relaxation parameter near limits | Local parallel rules, multiphase | High Mach, very low viscosity |
| Monte Carlo (physical) | High-dimensional, complex transport | Statistical | No timestep limit; variance-limited | High-dimensional transport | Convergence rate of 1/√N |

The table is a selection aid, not a league table. A method is appropriate when its conservation and stability properties match the equation class and geometry in front of you. Disagreements about methods are usually disagreements about which properties matter for a specific problem — which is a physics question wearing numerical clothing.

---

## 5. The Time Dimension

Time-dependent simulation is governed by one economic trade — the cost of small steps against the cost of solving a coupled system — and by one hard mathematical constraint, the stability bound.

### 5.1 Explicit versus Implicit

| Property | Explicit | Implicit |
|---|---|---|
| How the next state is obtained | Directly from the current state; each unknown updated from already-known values | Solving a system of equations coupling all unknowns at the new time level |
| Cost per step | Low — no matrix solve | High — a linear or nonlinear solve at every step |
| Maximum usable timestep | Bounded by a stability limit tied to the mesh spacing | Not bounded by that limit; bounded instead by accuracy and by how fast the physics must be resolved |
| Memory | Low, only the current state | Higher, a system matrix or the means to apply it |
| Behaviour on stiff problems | Punishing — the limit is set by the fastest, not the most important, timescale | Tolerable, because the fast modes are handled implicitly |
| Typical home | Wave propagation, transient dynamics, shock and impact, advection-dominated flow at small timesteps | Diffusion, stiff chemistry, incompressible flow, long transients, steady state reached by pseudo-transient marching |

The trade is not "which is better" but "is my problem limited by the number of steps or by the cost of one step". If the physics demands thousands of steps anyway — acoustics, impact, fast transients — explicit stepping is cheap and the stability limit is not the binding constraint. If the physics is slow and diffusive, the explicit limit forces so many steps that the implicit solve pays for itself many times over.

### 5.2 Stiffness, by Mechanism

A problem is **stiff** when the timescales present in the system span many orders of magnitude and the fastest timescale is not the one you care about. The mechanism: the explicit stability limit is set by the fastest decaying mode, so a problem whose fastest timescale is 10⁻⁶ s but whose interesting behaviour evolves over 1 s forces a million steps to resolve one second of the thing you wanted, even though the fast mode itself contributes almost nothing to the answer.

Stiffness is a property of the **problem**, not of the scheme or the hardware. Implicit methods address it by making the fast modes unconditionally stable so the step size can be set by accuracy alone — which is why implicit integrators dominate in combustion chemistry, in radiation-coupled heat transfer, and in long geophysical transients.

### 5.3 The Standard Integrator Families, Named

- **Multi-step linear methods:** the Adams–Bashforth family (explicit) and the Adams–Moulton family (implicit), which use several previous time levels; and the backward differentiation formulas (BDF), implicit and widely used for stiff problems.
- **Runge–Kutta methods:** one-step methods evaluating the right-hand side at several stages; explicit variants up to the classical fourth-order RK4 that engineering courses teach, and implicit variants (diagonally implicit and fully implicit) for stiff systems.
- **Leapfrog / centred schemes:** second-order in time and standard in wave and fluid codes for their low dissipation, at the cost of a decoupling that needs occasional filtering — and **operator splitting**, solving transport and reaction (or other coupled processes) in sequence per step, which buys efficiency at the cost of a splitting error that must be accounted for in the order-of-accuracy study.

Every one of these carries a formal order of accuracy in time that is *separate* from the spatial order, and the two must be balanced — refining only the mesh while leaving the timestep alone eventually makes the time error dominant, and refining only the timestep does the reverse.

### 5.4 The CFL Condition, Derived

The Courant–Friedrichs–Lewy condition is usually quoted as a constant to memorise. It is a stability bound, and for a concrete case it can be derived in full, with the units carried through.

**The problem.** One-dimensional linear advection, the simplest hyperbolic model problem:

  ∂u/∂t + a·∂u/∂x = 0,  a > 0 constant.

Information travels to the right at speed a. The exact solution is u(x,t) = u₀(x − a·t).

**The discretisation.** A uniform grid x_j = j·Δx, time levels t^n = n·Δt, and the first-order **upwind** scheme — upwind because for a > 0 the information at x_j arrives from the left, so the scheme uses the left neighbour:

  u_j^{n+1} = u_j^n − C·(u_j^n − u_{j−1}^n),  where C = a·Δt/Δx.

**The units.** [C] = [m·s⁻¹]·[s]/[m] = dimensionless. C is the **Courant number**: the fraction of a cell that information crosses in one timestep. This is the quantity that must be bounded, and the fact that it is dimensionless is why the same bound applies on any grid at any scale.

**The mode argument (von Neumann).** Substitute a single Fourier mode u_j^n = ξ^n·e^{i·k·j·Δx} into the scheme, with wavenumber k and amplification factor ξ. Dividing through by ξ^n·e^{i·k·j·Δx}:

  ξ = 1 − C·(1 − e^{−i·k·Δx}).

With θ = k·Δx, split into real and imaginary parts: ξ = (1 − C + C·cos θ) − i·C·sin θ. The squared modulus is

  |ξ|² = (1 − C + C·cos θ)² + C²·sin²θ = 1 − 2C(1 − C)(1 − cos θ).

(Check by expanding: LHS = (1 − C)² + 2C(1 − C)cos θ + C² = 1 − 2C + 2C² + 2C(1 − C)cos θ, which equals the last expression when 1 − cos θ is written out.)

**Reading the bound off the formula.** The factor (1 − cos θ) ranges from 0 (for the longest, smoothest wave, θ → 0) to 2 (for the grid-scale wave, θ = π). Stability requires |ξ| ≤ 1 for *every* mode the grid can represent, so the worst case is the grid-scale mode at θ = π:

  |ξ|²(θ = π) = 1 − 4C(1 − C),  and 1 − 4C(1 − C) ≤ 1 ⟺ C(1 − C) ≥ 0 ⟺ 0 ≤ C ≤ 1.

Outside that interval the grid-scale mode is amplified: for C > 1 the factor 1 − 4C(1 − C) exceeds 1 and the shortest wave present grows every step. Hence

  Δt ≤ Δx / a.

**The same bound from the domain of dependence (a check, not a coincidence).** At the point (x_j, t^{n+1}), the exact solution depends on the single point x_j − a·Δt. The scheme's stencil takes information from the interval [x_{j−1}, x_j]. The calculation can only be right if the true source of information lies *inside* what the stencil actually sees:

  x_j − a·Δt ≥ x_{j−1} ⟺ a·Δt ≤ Δx ⟺ Δt ≤ Δx/a.

The two derivations agree because the CFL condition is not a numerical artefact — it is the requirement that the discrete domain of dependence contain the continuous one.

**Unit chain of the final bound.** Δx/a has units [m]/[m·s⁻¹] = [s], a time, as Δt must be. The bound is scale-free in the sense that it constrains their ratio, not either quantity separately: halving the mesh halves the largest permitted timestep.

**What the same derivation shows about accuracy.** Performing the equivalent Taylor analysis on the same scheme — expanding u_j^{n+1} to O(Δt³) and u_{j−1}^n to O(Δx³) and substituting — gives, to leading order,

  ∂u/∂t + a·∂u/∂x = ((a·Δx)/2)·(1 − C)·∂²u/∂x² + …,

so the upwind scheme does not solve the advection equation exactly: it solves an advection–**diffusion** equation with a numerical diffusivity

  ν_num = (a·Δx/2)·(1 − C),

with units [m·s⁻¹]·[m] = [m²·s⁻¹], a diffusivity as required. Two consequences follow directly from the formula. First, at fixed C the numerical diffusivity is proportional to Δx, so the scheme is **first-order accurate** and the error is an artificial smearing — the scheme is not merely imprecise, it is solving a different equation. Second, ν_num → 0 as C → 1: at the largest stable Courant number this scheme is anomalously accurate for the pure advection problem, which is exactly the kind of behaviour that makes a single successful comparison run misleading.

**The generalisation, stated as a rule rather than a constant.** The requirement is that the numerical domain of dependence contain the physical one. For explicit diffusion-dominated schemes the corresponding limit is Δt ∝ Δx², which is far more punishing — the rule of thumb is that explicit treatment of a second-order spatial operator forces steps that shrink with the square of the refinement. This is the strongest single argument for implicit time integration in diffusion-dominated problems, and it is derived, not asserted.

---

## 6. Stability, Consistency, Convergence

### 6.1 The Three Properties and the Classical Relationship

- **Consistency:** as the mesh and timestep vanish, the discrete operator tends to the continuous operator — the truncation error goes to zero. A scheme can fail this by approximating the wrong equation or by being inconsistent at boundaries.
- **Stability:** the solution of the discrete problem remains bounded as the computation proceeds, i.e. perturbations do not grow. §5.4 derived the stability bound for a specific scheme as a demonstration of the mechanism.
- **Convergence:** the discrete solution tends to the solution of the partial differential equation as the mesh and timestep vanish.

The classical equivalence result in numerical analysis — due to Lax and Richtmyer, as it is stated throughout the numerical-analysis literature — is that for a **consistent** finite-difference scheme for a **well-posed linear** initial-value problem, stability is necessary and sufficient for convergence. Two cautions are part of the statement, not footnotes to it:

1. The hypotheses are **linear, well-posed, consistent**. Engineering practice is nonlinear, and the theorem does not carry over as a guarantee — it is carried over as a discipline (design for consistency, check stability empirically, then measure convergence by a study).
2. "Stability is necessary and sufficient for convergence" refers to convergence to the exact solution of the *PDE*. It says nothing about model error, nothing about the boundary conditions, and nothing about the input data. A stable, consistent, convergent scheme for an incorrect model converges to the wrong answer with impeccable credentials.

No primary source for the Lax–Richtmyer statement was read at source in this pass; it is cited here as the classical result in the literature and is recorded as such in §16.3.

### 6.2 Order of Accuracy, and What It Buys

For a scheme of formal order p, the leading error term on a uniform mesh of spacing h behaves as

  e(h) ≈ c·h^p,

so the error falls by 2^p when h is halved. The arithmetic of that sentence is where the practical value lives, and it is arithmetic, not a measurement:

| Formal order | Halving h reduces the error by | Third refinement (h → h/8) reduces it by | Typical illustration: from e = 4.0 × 10⁻² |
|---|---|---|---|
| First order (p = 1) | 2× | 8× | 2.0 × 10⁻², then 5.0 × 10⁻³ |
| Second order (p = 2) | 4× | 64× | 1.0 × 10⁻², then 6.25 × 10⁻⁴ |
| Fourth order (p = 4) | 16× | 4096× | 2.5 × 10⁻³, then 9.8 × 10⁻⁶ |

The scaling is what a **refinement study** measures. Running the same problem on three or more progressively finer meshes and computing the *observed* order from the ratios of successive solution differences tells you whether the code is behaving as its theory says it should. When the observed order is markedly below the formal order, the usual causes are a dominant non-smooth feature (a corner, a shock, a boundary singularity) sitting where the error norm looks, an error in the implementation, or a boundary treatment that is of lower order than the interior scheme. Refining further will not fix any of those; it will only spend more compute reproducing the wrong rate.

### 6.3 The Point That Must Not Be Missed

**Order of accuracy is a statement about discretisation error only.** Improving the order of a scheme, refining the mesh, and running a convergence study all act on one of the four error classes of §2.2. A fourth-order scheme applied to a model whose closure is invalid in the simulated regime produces a wrong answer four times faster than a second-order scheme would have produced it.

---

## 7. The Mesh

The mesh is where the geometry, the resolution and the error all meet. It is also, in real projects, where most of the human effort goes.

### 7.1 Geometry, Resolution and the Error Budget

The mesh does three separate jobs, and each one has its own error contribution:

1. **Representing the geometry.** A curved surface approximated by flat cell faces is a different geometry from the one drawn. The geometric error is in the mesh itself and is not removed by refining the solver.
2. **Resolving the physics.** Where gradients are steep — boundary layers, shear layers, wakes, shock structures, contact regions — the resolution must be sufficient for the scheme's order to be realised. An under-resolved gradient produces a *local* error that may dominate the global norm.
3. **Supplying the discretisation's stage.** The unknowns live on the mesh, so its topology and spacing determine the discrete operator.

The practical consequence: a global error norm can look acceptable while the quantity you actually care about — a peak temperature, a wall heat flux, a stress concentration — sits in the one under-resolved region. Mesh studies should therefore be run on the *quantities of interest*, not only on a global residual or a global norm.

### 7.2 Refinement and Adaptivity

- **Uniform refinement** — refine everywhere. Simple, expensive, and the standard basis of a grid-convergence study.
- **Local (h-)refinement** — refine where the error estimator says the error is. Uses a solution-driven error estimator and requires a data structure that supports hanging nodes or re-meshing.
- **Order (p-)refinement** — raise the polynomial order locally rather than refining the mesh; powerful for smooth solutions and the basis of the hp-family of methods — and **boundary-layer resolution**, where the physics has its own length scale and the mesh must be graded to match it.

Adaptivity is not free: it introduces a mesh that changes during the solution, which complicates conservation, parallel load balance and reproducibility (§10.3). In production work, a well-designed non-adaptive mesh built to the physics frequently beats an ambitious adaptive scheme that nobody has time to validate.

### 7.3 Mesh Quality: Why One Bad Cell Ruins a Good Scheme

Mesh quality is not aesthetic. Each defect has a mechanism:

| Defect | Mechanism | Symptom |
|---|---|---|
| High skewness / non-orthogonality | The face-normal and the line joining cell centres are not aligned, so the face flux is computed from a poor interpolation; the discretisation loses order and can lose boundedness | Oscillatory or over-diffusive solutions near the defect; iterating longer does not help |
| High aspect ratio off the layer direction | Error is anisotropic and the truncation term in the stretched direction dominates | Direction-dependent error that a uniform refinement study misreads |
| Rapid cell-size jumps | Interpolation between neighbours is of low order and can break conservation locally | Local error that contaminates a global balance |
| Degenerate or inverted cells | The discrete operator becomes ill-conditioned or the volume is wrong | Solver stalling, non-physical values |
| Poor boundary-layer grading | The near-wall gradient is resolved by too few cells | A wall quantity of interest that is simply wrong, while the interior looks fine |

**The consequence is asymmetric: scheme quality cannot rescue mesh quality.** A second-order scheme applied to a mesh containing a handful of degenerate cells can produce a solution worse than a first-order scheme on a clean mesh, and no amount of solver tolerance addresses it. In a parallel run, the defect may sit in one partition and only appear when the domain is decomposed differently — which is one of the practical reasons reproducibility and meshing are linked (§10.3).

### 7.4 The Guide's Analysis: Meshing Dominates the Effort and Is Absent from the Introductions

**This subsection is the guide's own analysis, not a sourced finding.**

Textbook treatments of numerical simulation are structured around the discretisation: the scheme, its order, its stability, its convergence proof. Real projects are structured around the mesh. Geometry import and repair, defeaturing, meshing strategy, boundary-layer grading, quality checking, re-meshing after each change of the geometry — a competent engineer's week is dominated by these tasks, and the solver is a small, mostly automated part of the workflow.

The asymmetry has three consequences worth stating plainly:

1. **Effort is invisible in the literature's structure.** A journal paper reports a scheme and results; the weeks spent building a mesh that made those results defensible are compressed into "the computational domain is discretised with approximately N cells". Nobody learns meshing from that sentence.
2. **Cost is dominated by the human step, not the compute step** — and the mesh is where most silent errors live, because it is a human artefact with a quality distribution, so it is the most common source of an error that survives verification of the scheme and the code. When people estimate the cost of a simulation campaign they estimate core-hours; the binding constraint is usually the meshing and re-meshing loop, which does not parallelise.

The honest corollary is that a simulation report's credibility can be judged faster from its meshing section — cell counts by region, quality metrics, boundary-layer treatment, the refinement study on the quantities of interest — than from its solver settings.

---

## 8. Boundary and Initial Conditions, and the Modelling Choices That Hide Inside Them

### 8.1 Where Simulation Fails Quietly

Boundary and initial conditions are where a simulation fails without warning, because nothing in the solution process is designed to notice. The solver sees a well-posed problem and solves it.

The failure classes, with their mechanisms:

- **Wrong type of condition.** A fixed value where a flux is physically appropriate (or the reverse) changes the problem, not just the answer's accuracy. In heat transfer this is the difference between a convective coefficient prescribed at a wall and a conjugate solution that computes the wall temperature from the fluid side.
- **Applied at the wrong place.** A "far-field" boundary placed too close turns a physically distant boundary into a constraining one; the domain of influence from the boundary reaches the region of interest, and refining the mesh makes the artificial boundary's effect *sharper*.
- **Inconsistent initial state.** A transient run started from a state that does not satisfy the boundary conditions or the discrete equations produces a startup transient; if the quantity of interest is averaged from time zero, that transient is in the reported answer.
- **Boundary schemes of lower order than the interior, or an unphysical outflow closure.** The interior may be second order and the boundary treatment first order, in which case the boundary error dominates on refined meshes and the observed order in a convergence study flattens — a signature worth recognising rather than a numerical accident. An outlet that is allowed to reflect pressure or acoustic waves back into the domain contaminates the solution upstream in the same quiet way.

### 8.2 The Input-Data Problem: Properties Nobody Measured

The material and boundary data fed to a simulation carry a precision the data rarely has. A required thermal conductivity is entered to four significant figures; the value in the handbook is a nominal figure for a material class, measured on a specimen that is not the article in the plant, at a temperature that may not be the operating temperature, with an uncertainty the handbook does not state. The same applies to emissivity (which depends on surface finish and oxidation), contact resistance (which depends on clamping pressure and surface roughness), and heat-transfer coefficients taken from correlations derived for other geometries and other Reynolds numbers.

Two habits follow. First, **state the provenance of every input** in the report: measured here, taken from this standard table, assumed, or calibrated. Second, **vary the uncertain ones and report the spread** — which is the sensitivity analysis of §2.6, and it is cheaper than the refinements usually done instead.

### 8.3 Turbulence Modelling: a Modelling Choice Presented as a Numerical One

**This subsection carries the guide's own explanation of the three approaches; no source characterising them was verified at source in this pass, and the attributions are recorded accordingly in §16.3.**

The Navier–Stokes equations describe turbulent flow, but resolving every length scale and time scale in an engineering geometry is out of reach for reasons of cost alone (§11). Practice therefore splits into three approaches that are usually described as "modelling options" in software menus, and are in fact three different statements about which physics is being represented:

| Approach | What is represented | What is modelled | Consequence for what the result means |
|---|---|---|---|
| Reynolds-averaged (RANS) | Time-averaged (or ensemble-averaged) flow fields | All turbulent fluctuations, through a closure whose constants are calibrated on canonical flows | The answer is a statement about the *mean* flow under an assumed closure; in strongly separated or strongly unsteady flow the assumption on which the closure rests does not hold |
| Large eddy simulation (LES) | The large, energy-carrying eddies resolved in space and time | The small, sub-grid scales, by a sub-grid model | Much more of the physics is computed; the cost rises sharply with Reynolds number because the resolved range must cover the energy-containing scales |
| Direct simulation (DNS) | All scales, from the energy-containing range down to the dissipative range | Nothing of the turbulence itself; the equations are solved as written | The most physically complete and the most expensive by orders of magnitude; confined in practice to canonical geometries and modest Reynolds numbers |

The point that matters for a review is not which approach is better. It is that **the choice of turbulence treatment is a model-form decision that changes the meaning of the reported number**, and software presents it as a setting. A converged RANS solution is a converged solution of the RANS equations; whether the returned field resembles the real flow in a separated region is a validation question that no residual can answer. When a vendor report says "converged", ask the next question: converged *to what* — and on what basis was the closure known to be valid for this geometry and this regime?

The same structural point recurs outside fluids: a linear-elastic idealisation in a structure that yields, a lumped thermal mass where the Biot number says otherwise, an isothermal wall where the conjugate problem is not. Each is a modelling choice that a numerical report will not flag.

---

## 9. Verification and Validation

This is the guide's load-bearing section. The vocabulary here is defined in accredited standards and in the peer-reviewed literature, and the distinction between the three activities is the single most useful idea in the discipline.

### 9.1 The Three Activities and the Three Questions

| Activity | The question it answers | Evidence it needs | Standards and literature |
|---|---|---|---|
| **Code verification** | Is the numerical implementation solving the equations it claims to solve, correctly? | A known exact solution and a measured order of accuracy; software-quality and unit evidence | AIAA G-077-1998; the method of manufactured solutions (Salari & Knupp, SAND2000-1444; Roache 2002) |
| **Solution verification** | For this specific computation, how large is the numerical error — discretisation, iteration, round-off? | A grid- and timestep-convergence study on the quantities of interest; iteration-tolerance evidence | ASME V&V 20-2009 defines the approach for CFD and heat transfer; AIAA G-077-1998 |
| **Validation** | Does the model as used describe the real world, to the accuracy the decision requires? | Comparison with physical experiment or field measurement, with uncertainty in both | AIAA G-077-1998; ASME V&V 10-2019 for solid mechanics; ASME V&V 20-2009 for CFD and heat transfer; Oberkampf & Trucano (2002) |

The definitions the standards themselves give are worth quoting exactly, because paraphrase usually blurs them.

From **AIAA G-077-1998** (approved 14 January 1998; reaffirmed R2002), the first consensus V&V guide published by an accredited standards developer:

- "Verification is the process of determining if a computational simulation accurately represents the conceptual model, but no claim is made of the relationship of the simulation to the real world."
- "Validation is the process of determining if a computational simulation represents the real world."

The phrase "no claim is made of the relationship of the simulation to the real world" is the entire content of this guide's thesis in one clause.

From **ASME V&V 10-2019** ("Standard for Verification and Validation in Computational Solid Mechanics", superseding V&V 10-2006), the stated purpose is to provide the computational solid mechanics community with "a common language, a conceptual framework, and general guidance for implementing the processes of computational model V&V". Companion documents extend it: **V&V 10.1-2012** presents an illustration of the concepts, **V&V 10.2** addresses uncertainty within the V&V process, and **V&V 10.3** addresses metrics for validation comparisons.

From **ASME V&V 20-2009** ("Standard for Verification and Validation in Computational Fluid Dynamics and Heat Transfer", reissued R2016 and R2021), the stated objective is to specify an approach that "quantifies the degree of accuracy inferred from the comparison of solution and data for a specified variable at a specified validation point", drawing on experimental uncertainty analysis to account for errors and uncertainties in both the solution and the data, and scoped to cases where the conditions of the actual experiment are simulated.

From **Oberkampf & Trucano (2002)**, published in *Progress in Aerospace Sciences* 38(3), 209–272 (DOI 10.1016/S0376-0421(02)00005-2), the publisher's abstract states that V&V "are the primary means to assess accuracy and reliability in computational simulations", and the paper is an extensive literature review that develops extensions to existing ideas and reviews the development of V&V terminology and methodology.

**The three questions, in the form a review can actually ask them:**

1. *Is the code solving its own equations correctly?* (code verification) — answered by the order of accuracy against a manufactured or exact solution.
2. *How much numerical error is in this particular answer?* (solution verification) — answered by refinement studies on the quantities of interest.
3. *Is the model right about the world?* (validation) — answered only by comparison with data.

### 9.2 The Method of Manufactured Solutions

The method of manufactured solutions (MMS) is the standard technique for code verification, and its mechanism is worth spelling out because it is often described as if it were circular. It is not.

**The obstacle it solves:** for a general code you do not usually have an exact solution to compare against, so you cannot directly measure the order of accuracy of the implementation.

**The procedure:**

1. **Choose a manufactured solution.** Pick a smooth, closed-form function u*(x,t) — for example a sine wave — with no regard for whether it solves anything.
2. **Substitute it into the governing operator.** Compute L(u*), where L is the differential operator the code claims to solve. For a nonlinear code this means applying the full operator, term by term, symbolically or with automatic differentiation.
3. **Derive the source term.** Since u* does not generally satisfy L(u) = 0, the residual becomes the source term that *would* make it a solution: define S = L(u*), and solve L(u) = S with boundary and initial conditions taken from u*.
4. **Run the code with S, the boundary data and the initial data, on a sequence of refined meshes, and compare** the computed solution to u* — checking that the difference is falling at the rate the scheme's formal order says it should (the *observed* order of accuracy) and that it continues to fall as the mesh is refined. Deviations localise the defect to the terms, the boundaries or the order of the implementation.

The procedure is described in Salari & Knupp, "Code Verification by the Method of Manufactured Solutions" (SAND2000-1444, Sandia National Laboratories, June 2000), whose abstract states that a procedure for code verification by MMS is presented and that MMS "can be applied to a variety of engineering codes which numerically solve partial differential equations", illustrated with detailed examples from computational fluid dynamics. The journal version is Roache, "Code Verification by the Method of Manufactured Solutions", *ASME Journal of Fluids Engineering* 124(1), 4 (2002). A Springer handbook chapter on MMS for code verification frames the activity as "demonstrating that the code is free of coding errors and is capable, given sufficient discretization, of approaching exact mathematical solutions" — the sentence to hold onto, since it makes clear that verification is about the code, not about nature.

**Illustration of the derivation, on the equation already used in §5.4.** Take the advection–diffusion equation, u_t + a·u_x = ν·u_xx, and manufacture

  u*(x, t) = A·sin(k·x − ω·t).

Then u*_t = −A·ω·cos(k x − ω t), u*_x = A·k·cos(k x − ω t), and u*_xx = −A·k²·sin(k x − ω t). Substituting:

  L(u*) = u*_t + a·u*_x − ν·u*_xx = A·cos(k x − ω t)·(a·k − ω) + A·ν·k²·sin(k x − ω t).

Choosing the dispersion relation ω = a·k for the advection part makes the cosine term vanish, leaving the source term

  S(x, t) = A·ν·k²·sin(k x − ω t),  with units [A][ν][k²] = [u]·[m²·s⁻¹]·[m⁻²] = [u·s⁻¹],

which is the unit of ∂u/∂t, as a source term in this equation must be. The code is then run with this S, with Dirichlet data u* at both ends and with u*(x, 0) as the initial condition, on a sequence of meshes with the Courant number held fixed. On a uniform mesh with well-behaved boundary treatment the difference between the computed solution and u* should fall at the formal rate — first order for the upwind scheme of §5.4, second order for a centred scheme on a smooth solution — and a flattening or a lower-than-expected rate points at an implementation error rather than at the physics.

**What MMS does and does not do.** It verifies the *code*: that the implementation solves the intended equations to the intended order. It says nothing about whether the equations are the right physics, and it says nothing about the boundary or initial conditions of the actual engineering problem, which are exactly the parts a manufactured solution supplies from the same closed-form function. That gap is not a flaw in MMS; it is the boundary between code verification and validation.

**ILLUSTRATIVE — not a real API.** The loop that a code-verification harness implements, in pseudocode:

```
for h in [h0, h0/2, h0/4, h0/8]:
    dt   = C * h / a                 # hold the Courant number fixed
    mesh = build(mesh_spacing = h)
    sol  = solve(L(u) = S, mesh, dt,
                 bc = u_star_on_boundary, ic = u_star_at_t0)
    e[h] = norm(sol - u_star)
p_obs = log(e[h0]/e[h0/2]) / log(2)  # observed order, first pair
         # repeat on successive pairs; p_obs should converge to the
         # scheme's formal order for this norm and this solution
```

### 9.3 Refinement Studies: Grid Convergence and Timestep Convergence

A convergence study is the practical answer to the second question — how much numerical error is in *this* answer. It requires care about three things:

1. **Hold the right quantities fixed.** In a refinement study the Courant number (and any other non-dimensional group the physics depends on) should be held constant so that the refinement tests the discretisation, not a change in the problem. If the timestep scales with the mesh spacing, the time-integration error is being reduced as a side effect, and the two must not be conflated — running separate timestep-refinement and grid-refinement studies separates them.
2. **Converge on the quantity of interest.** Global norms can be quiet while a wall heat flux or a peak stress is not. The quantity the decision depends on is the quantity the study must cover.
3. **Expect a rate, and investigate a deviation.** The study should produce a sequence of solutions whose successive differences fall in a recognisable pattern; the *observed* order of accuracy is computed from those differences and should be compared against the scheme's formal order. A higher-than-expected rate usually means the refinement is not yet in the asymptotic range (the errors include a pre-asymptotic contribution that happened to fall fast); a lower rate usually means a non-smooth feature, a lower-order boundary treatment, an implementation error, or an insufficiently converged iteration.

**When the study does not converge**, the diagnosis is not "refine more" but an investigation: a geometric or boundary singularity the error norm keeps sampling; a tolerance that was not tightened with the mesh; a mesh-quality defect present only at some refinement levels; a coding error in one term; two coupled errors partially cancelling; or a quantity of interest whose definition depends on the mesh itself.

### 9.4 The Central Honesty Point

**Validation is the only one of the three activities that can tell you whether the model describes reality — and it is the step most often skipped, because it is the one that requires data and a physical experiment.**

Code verification is a closed-world exercise: you only need a solution you manufactured yourself. Solution verification is a closed-world exercise too: you only need more compute on the same case. Both can be done entirely inside the software environment, and both produce reassuring artefacts. Validation cannot: it needs a measurement taken on something physical, with its own uncertainty, at conditions that correspond to what was simulated — which is exactly why ASME V&V 20-2009 is scoped to cases where "the conditions of the actual experiment are simulated", and why it prescribes accounting for uncertainty in the data as well as the solution.

The practical consequences of skipping validation:

- **A defensible numerical error attached to an indefensible model.** The study says the discretisation error is 2%; the model is wrong by more than that. Reports that lead with the refinement study create a false impression of rigour.
- **No basis for extrapolation.** Without data, a simulation has no measured basis for being applied outside the range it was built for. The claim "this shows the change will remove 3 °C" is a statement about the model, not about the hall.
- **The failure mode is quiet.** Nothing in the solver output distinguishes a validated model from an unvalidated one. The distinction lives in a document, and if nobody wrote the document, the distinction does not exist.

The separation that must remain exact: **verification is not validation.** A verification study can be complete, rigorous and published, and it will still be, in the AIAA guide's own words, a statement about the conceptual model with "no claim … of the relationship of the simulation to the real world".

### 9.5 The Software-Engineering V&V That Shares the Words

This repository has a second V&V vocabulary, and it is important not to blend them. `ai_llm/llm_evaluation_vs_validation_guide.md` cites **IEEE 1012**, the standard for system, software and hardware verification and validation, and the Boehm formulation; `project_management_methodologies_guide.md` §5.1 has its own V&V subsection. In that world, verification asks whether an artefact was built to its specification and validation asks whether the specification was the right one — the object is the software and the ground truth is the requirements document, whereas in the AIAA/ASME world the object is a numerical solution of a physical model and the ground truth is an experiment. The two traditions agree on the one structural principle that generalises: **verification and validation answer different questions, and passing one is not evidence about the other.**

---

## 10. The Floating-Point and Reproducibility Substrate

Beneath every scheme and every mesh sits arithmetic that is not the arithmetic of real numbers. The standard defining it is **IEEE 754-2019**, "IEEE Standard for Floating-Point Arithmetic": an active standard, PAR approval 2015-09-03, board approval 2019-06-13, published 2019-07-22, superseding IEEE 754-2008, maintained under the IEEE Microprocessor Standards Committee (C/MSC). Its own scope covers "interchange and arithmetic formats and methods for binary and decimal floating-point arithmetic in computer programming environments … exception conditions and their default handling", and notes that the standard "may be realized entirely in software, entirely in hardware, or in any combination" — which is why the same program on two machines can differ in the last bits and why "the same code" is not the same arithmetic.

### 10.1 Conditioning and Catastrophic Cancellation

The relevant distinction is between the conditioning of a *problem* and the stability of an *algorithm*. Subtracting two nearly equal numbers is ill-conditioned in the worst possible way: the inputs are accurate to a relative error of about 2.2 × 10⁻¹⁶ in double precision, and the difference retains a large *absolute* error while losing most of its significant digits, so its *relative* error is amplified without bound as the two inputs approach each other.

A concrete, measured demonstration — computed for this guide in double precision:

| Quantity | Value | Comment |
|---|---|---|
| x = 1.0 × 10⁻⁸ (a small angle) | — | the quantity whose cosine is computed below |
| 1 − cos(x), computed naively | **0.0** | every significant digit lost; the true value is not representable in this subtraction |
| 2·sin²(x/2), the stable rearrangement | 5.0000000000000005 × 10⁻¹⁷ | correct to the last bit shown, and equal to x²/2 to leading order |
| relative error of the naive form | **1.0** | i.e. 100% — the naive answer is exactly zero, the correct one is not |

The mechanism: cos(10⁻⁸) ≈ 1 − 5 × 10⁻¹⁷, and the spacing between representable doubles near 1.0 is 2.22 × 10⁻¹⁶, larger than 5 × 10⁻¹⁷. The subtraction therefore has nothing left to return. This is not an exotic case: the same structure appears in finite differences of a computed field, in variance formulas computed as E[x²] − E[x]², in pressure from a large reference minus a small perturbation, and in residual computations whose whole purpose is to measure a small quantity.

The discipline that follows: **formulate differences to avoid subtracting near-equal quantities** where the formulation allows it, and remember that the accuracy a scheme's order of accuracy promises is bounded below by the conditioning of the arithmetic used to express it.

### 10.2 Round-Off Accumulation and the Order of Operations

Floating-point addition is not associative, so summation order matters. Measured for this guide in double and single precision, summing the first 200,000 terms of the harmonic series:

| Method | Result | Error vs exact rounding |
|---|---|---|
| Naive left-to-right accumulation, double precision | 12.78329081042982 | absolute 1.97 × 10⁻¹³, relative 1.54 × 10⁻¹⁴ |
| Exact rounding within the format (`math.fsum`, compensated) | 12.783290810429623 | — |
| Naive accumulation in single precision (float32) | 12.782756805419922 | relative **4.18 × 10⁻⁵** |

Two lessons with real consequences. First, in double precision the accumulation error (10⁻¹⁴ relative) is negligible for most engineering purposes but is *not* zero, and it grows with the number of operations — which is precisely why refining a mesh can increase round-off while reducing discretisation error, and why round-off has to appear in the error budget separately from the other three. Second, dropping to single precision costs roughly four significant decimal digits of accuracy on this accumulation — the trade in §10.4 is therefore about *place*, not about being cheaper for free.

### 10.3 Reproducibility: Same Code, Different Results

Bitwise-identical results across machines and across core counts are not a property of numerical codes; they are a property that must be engineered, and often cannot be. The mechanisms, all of them sufficient on their own to change the last digits:

- **Parallel reduction order.** A global sum computed by a tree of processors varies with the number of processors and the decomposition, and floating-point addition is not associative. The number of terms is the same; the order is not.
- **Domain decomposition and mesh partitioning.** Different partitioning changes which cells are neighbours in memory, hence the order of flux accumulation and of the iterative solver's inner products, and can change whether a mesh-quality defect is present in a given partition.
- **Compiler optimisation and vectorisation.** Reassociation, fusion of a multiply and add into a fused multiply–add (with a single rounding instead of two), and different vector widths all change results.
- **Transcendental and special functions.** Library implementations of exp, log, sin and the powers differ between toolchains and between vendors, and the differences are in the last bits exactly where a cancellation-sensitive formula is most fragile.
- **Dynamic scheduling and atomics.** Non-deterministic work assignment changes accumulation order between runs of the same binary on the same machine.
- **Hardware floor.** Hardware defect and variation in accelerators means that even a fixed binary on a fixed core count is not always bit-reproducible on some devices.

Consequences for practice: comparisons must be tolerance-based rather than equality-based in tests; a "difference" between two runs must first be attributed to one of the mechanisms above or to a genuine bug; and where a regulator or a safety case requires reproducibility, the requirement has to be designed in (fixed core counts, deterministic reductions, fixed compiler flags), which costs performance. Claiming bit-reproducibility from a highly parallel run without having designed for it is a claim that has not been checked.

### 10.4 The Mixed-Precision Trade

Using lower precision for the bulk of the work and higher precision for the accumulation is now standard practice: iterate and store in single precision, accumulate norms and residuals in double; solve the linear system in low precision and refine the solution by a few high-precision correction steps. The trade is real — memory traffic and the cost of the dominant arithmetic both fall — but it has a specific hazard worth naming: **a small residual does not imply a small error** when the residual is computed in lower precision than the solution. A residual evaluated with a relative precision of 10⁻⁷ cannot certify an error below 10⁻⁷, and may not even measure the error it purports to measure if the residual computation itself suffers cancellation. Mixed precision must therefore be justified by an error analysis for the specific quantity of interest, and the §2 distinction between iteration error and round-off error has to be kept clean: they are separate terms with separate remedies.

### 10.5 What the Substrate Implies for the Discipline

Report precision with the result — six significant figures from single-precision accumulation is over-reporting; make convergence criteria scale-aware, since absolute residual thresholds do not transfer between problems of different size while relative and quantity-of-interest-based criteria do; and compare runs with a stated tolerance rationale rather than with equality, unless the run has been engineered for reproducibility (§10.3).

---

## 11. HPC and the Scaling Question

### 11.1 The Parallelism Models, by Class

| Model | Unit of parallelism | Where the parallelism lives | Communication mechanism | Typical fit |
|---|---|---|---|---|
| **Shared memory** | Threads within one address space | Loop-level: the mesh is one array and threads take slices | Loads and stores through shared cache; synchronisation barriers | One node; easiest to implement; limited by memory bandwidth and by synchronisation |
| **Distributed memory** | Processes with private memory | Domain decomposition: each process owns a sub-domain plus a halo of neighbouring cells | Explicit messages; latency-and-bandwidth-bound; the halo is the overhead | Many nodes; the standard model for large simulations |
| **Accelerator** | Many simple cores under one wide memory system | Data-parallel kernels launched over the mesh | Host-to-device transfers plus a relatively slow interconnect to the device | Bandwidth-bound kernels, dense linear algebra, explicit stencil updates |
| **Hybrid** | Processes × threads × devices, or any subset | Decomposition across processes, threads or kernels within a sub-domain | All of the above, nested | Production HPC today |

The repository's accelerator material — `gpu_optimization_guide.md`, `gpu_cloud_providers_comparison_guide.md`, `hami_gpu_sharing_guide.md`, `low_latency_cpp_development_guide.md` — owns the hardware and kernel side of this. There is no `gpu_cloud_guide.md` in this repository; do not cite one.

### 11.2 Strong versus Weak Scaling

**Strong scaling** fixes the problem and adds processors: the question is how much faster the same job runs. **Weak scaling** fixes the work per processor and adds processors along with problem size: the question is whether the *time to solution* stays roughly constant as the problem grows. They answer different questions and are routinely confused in both directions — a machine can show excellent weak scaling and poor strong scaling, because the weak-scaling test grows the work per processor in a way that amortises the overheads.

The efficiency measures are simply defined, and the arithmetic is worth performing rather than memorising (**the guide's own explanation; no source for these definitions was verified at source in this pass, §16.3**):

- **Speedup:** S(N) = T(1)/T(N), the serial time divided by the time on N processors.
- **Parallel efficiency:** E(N) = S(N)/N, expressed as a percentage.

Taking an idealised model with a fixed serial fraction *s* and perfectly parallelised remainder — the classical Amdahl form S(N) = 1/(s + (1 − s)/N) — the arithmetic below is computed for this guide:

| Serial fraction | N = 64 | N = 1,024 | N = 65,536 | Asymptotic limit |
|---|---|---|---|---|
| 1% | speedup 39.3, efficiency 61.3% | speedup 91.2, efficiency 8.9% | speedup 99.9, efficiency 0.2% | 100× |
| 5% | speedup 15.4, efficiency 24.1% | speedup 19.6, efficiency 1.9% | speedup 20.0, efficiency 0.0% | 20× |
| 10% | speedup 8.8, efficiency 13.7% | speedup 9.9, efficiency 1.0% | speedup 10.0, efficiency 0.0% | 10× |

Read the third column and the last column together: **at a 5% serial fraction the asymptotic speedup limit is 20× no matter how many processors are available**, and 1,024 processors already deliver 1.9% efficiency against that ceiling. The model is an idealisation — it ignores that communication cost grows with processor count, and that solvers usually slow down per unknown as the problem grows — but its conclusion survives the idealisation: fixed overheads that look negligible on paper dominate at scale, and the count of processors is not a measure of capability.

### 11.3 Where the Efficiency Actually Goes

The mechanisms that break scaling, in rough order of how often they are the real cause:

1. **Halo exchange and surface-to-volume ratio.** Communication scales with the sub-domain *surface* while computation scales with its *volume*; as the number of sub-domains grows, each one gets thinner and the ratio worsens. This is a geometric wall, not an implementation defect.
2. **Global operations.** Inner products, norms and global residuals in iterative solvers require collective communication and a synchronisation point, so each iteration ends with the slowest process.
3. **Load imbalance.** Physics is rarely uniform; a mesh with a coarse region and a fine region, or a domain with one hot spot, leaves processors idle at the barrier. Adaptive refinement makes this worse and requires dynamic repartitioning.
4. **Solver algorithmic scaling.** The number of iterations of a simple iterative method grows with problem size unless a multigrid or an equivalent hierarchical preconditioner is used. Adding processors to a method whose iteration count grows with N can leave time-to-solution flat or worse.
5. **Memory bandwidth and the machine balance.** A stencil update performs few floating-point operations per byte loaded, so its speed is set by memory bandwidth rather than by arithmetic throughput — the arithmetic-intensity view of performance (**the guide's explanation; the roofline formulation was not verified at source in this pass**). This is why explicit finite-volume stencil kernels do not speed up in proportion to peak floating-point rate, and why §11.5's output cost is not a side issue.

### 11.4 What the Top of the Range Looks Like

The **TOP500** list is a citable ranking with a published methodology, reissued twice a year, which is why any figure quoted from it must carry its list date. As of the **November 2025** list, rank 1 is **El Capitan** (HPE Cray EX255a, AMD 4th-generation EPYC processors with AMD Instinct MI300A accelerators, installed at DOE/NNSA/LLNL) with **Rmax 1,809.00 PFlop/s** and **Rpeak 2,821.10 PFlop/s**, at a measured power of 29,685 kW; rank 2 is Frontier (ORNL) at Rmax 1,353.00 PFlop/s and rank 3 Aurora (Argonne) at Rmax 1,012.00 PFlop/s. A June 2026 list exists on the site.

Two observations from that table, both matters of arithmetic:

- **The gap between advertised and achieved.** Rmax/Rpeak for El Capitan is 1,809.00/2,821.10 = **64.1%**, computed here from the listed figures. The list's own description of Rpeak notes it is computed from the advertised clock rate and warns that efficiency assessments should account for turbo clock behaviour — so the honest reading is that a third of the nominal capability is not reachable on the measured workload, before any question of whether the workload is a good one.
- **The ceiling is not the application.** A top-ranked machine is a statement about a specific benchmark at scale. Nothing follows from the number about how quickly a particular simulation with a particular mesh, solver and I/O pattern will run on it.

### 11.5 The Wall Most Projects Actually Hit: I/O

The arithmetic of output is unforgiving. On a structured three-dimensional grid with 100 cells per side there are 10⁶ cells; halving the mesh spacing multiplies that by 8, and if the timestep is tied to the mesh spacing by a stability limit, the *number of timesteps* doubles as well, so the work rises by 16× for one refinement level (computed here: (200)³/(100)³ = 8, and 8 × 2 = 16).

Now apply that to storage. At an illustrative 200 bytes per cell per field, a 1000³ cell mesh holds 1000³ × 200 bytes = **186 GiB for one field** — and a real run has several fields (velocity components, pressure, temperature, turbulence quantities) and needs them written at many timesteps. The numbers are illustrative order-of-magnitude arithmetic, not a measured result; the conclusion is robust to the constant.

The consequence is an inversion that surprises people who have not run a large campaign: **compute is frequently cheaper than output.** The run finishes, and the analysis cannot be done because the data was never written, or was written at intervals too coarse to resolve the transient, or fills the parallel filesystem and stalls. The responses are all architectural:

- **In-situ processing** — reduce the data as it is produced, computing the derived quantities (averages, spectra, extrema, loads) inside the solver and writing only the reduced result.
- **Checkpointing, restart discipline and tiered output** — design the restart granularity before the run, not after it fails; write full fields rarely, reduced fields frequently, and scalar monitors at every step.
- **Parallel I/O with a deliberate file layout** — the number of files and the writer count interact with the filesystem's characteristics, and a layout that is fine at 64 ranks may collapse at 4,096.

### 11.6 The Plain Statement

**A solver that does not scale is a solver you cannot afford to run.** Scaling is not a performance refinement added at the end of a project; it is part of the validity envelope of the method, in exactly the way the mesh and the model are. A method whose cost grows too fast, or whose parallel efficiency collapses before the problem is big enough to be interesting, has not been shown to be wrong — it has been shown to be inapplicable to the problem in front of you.

---

## 12. The Software Landscape, Verified

### 12.1 The Tool Classes

| Class | What it is | Where the effort goes | Examples (roles as the projects describe themselves) |
|---|---|---|---|
| **General-purpose multiphysics suites** | A packaged environment: geometry, meshing, solvers for several physics, post-processing, GUI | Learning the suite's abstractions; coupling physics the suite supports; fitting the problem to the tool | Ansys (now part of Synopsys), COMSOL — commercial; code_aster, OpenFOAM — open |
| **Mesh generators** | Geometry preparation and grid generation as a separate discipline | The dominant human effort (§7.4) | Gmsh |
| **Solver libraries** | Numerical infrastructure you call: linear and nonlinear solvers, preconditioners, parallel data structures | Writing the simulation around the library; owning the physics and the discretisation yourself | PETSc; the finite-element libraries FEniCSx, deal.II; the PDE-analysis toolbox SU2 |
| **Structured solver packages** | A complete solver for a physics class with its own input format | Input deck and results interpretation | CalculiX (structural), OpenFOAM (CFD), code_aster (mechanics and thermal) |
| **Molecular-dynamics codes** | Particle simulation at atomic and coarse-grained scales | Force-field selection and equilibration — the modelling choices dominate | LAMMPS, GROMACS, OpenMM |
| **Scripting and notebook layer** | Python (and increasingly Julia) bindings that orchestrate the above | Glue, parametric studies, post-processing, reproducibility of the workflow | FEniCSx Python interface; petsc4py; Gmsh's Python API; meshes and solvers driven from notebooks |

### 12.2 Open-Source Tools, with Each Project's Own Description

Every description below is what the project says about itself, read at its own site in this pass (checked 2026-09-29). Licence models are stated **only where the licence was verified at source**; where the pack flagged a licence as unverified and it could not be verified in this pass, the row carries no licence claim and the gap is recorded in §16.3.

| Tool | What its own site says it is | Licence, as verified |
|---|---|---|
| **OpenFOAM** (openfoam.org, The OpenFOAM Foundation) | Free CFD software; the front page publishes a maintenance campaign in which supporting organisations "currently provide €250k for maintenance of OpenFOAM, i.e. of the order of 0.1% of the revenue of big commercial CFD", with a stated goal of €500k, and three maintenance levels (Platinum €100k/yr, Gold €25k, Silver €5k); core development at CFD Direct, including OpenFOAM's creator Henry Weller | **GNU General Public Licence**, stated on its own licence page (openfoam.org/licence/) |
| **OpenFOAM** (openfoam.com, OpenCFD Ltd) | "the free, open source CFD software developed primarily by OpenCFD Ltd since 2004"; releases every six months in June and December — v2512 (December 2025) and v2606 (26 June 2026), both confirmed on the site's own news page; repositories moved to gitlab.com/openfoam in November 2025. **Corporate position, stated precisely because the two sources on the site do not use identical wording:** the site's own About page describes "OpenCFD Ltd, owner of the OpenFOAM Trademark", as "a wholly owned subsidiary of **ESI Group**", while the site's own news item of 1 November 2024 is titled "**ESI Business Unit, as part of Keysight Technologies**, strengthens commitment to OpenFOAM" and states that OpenCFD Limited was integrated into the ESI UK legal entity and that "both OpenCFD and OpenFOAM Trademarks will remain vital components of the Keysight Technologies brand"; releases from v2506 (June 2025) onward are announced as "**Keysight** OpenCFD". Quote both, assert neither as the sole fact. The site also states that its QA process of "code evaluation, verification and validation" includes "several hundred daily unit tests, a medium-sized test battery run on a weekly basis, and large industry-based test battery run prior to new version releases". **This is a second distribution that shares the name with the Foundation's — it is not the same product line, and the two should never be described as one** | **No licence claim made here**: the licence text was not read at source in this pass (§16.3) |
| **FEniCS / FEniCSx** | "a popular open-source computing platform for solving partial differential equations (PDEs) with the finite element method (FEM)"; high-level Python and C++ interfaces; "runs on a multitude of platforms ranging from laptops to high-performance computers"; a NumFOCUS fiscally supported project. Components are UFL, Basix, FFCx and DOLFINx; FEniCSx **0.11** was released in June 2026; the legacy FEniCS (DOLFIN) line's last release was 2019.1.0 (April 2019) | The DOLFINx source repository reports **LGPL-3.0 and GPL-3.0** licences found (GitHub licence detection on the project's own repository, checked 2026-09-29) |
| **deal.II** | "A C++ program library targeted at the computational solution of partial differential equations using adaptive finite elements"; the project's mission is to provide well-documented tools for building finite element codes "from laptops to supercomputers"; **9.8.0** was released 2026-08-09 | The project describes itself as "open source and available for free"; the specific licence identifier was not read at source in this pass, so no identifier is claimed here (§16.3) |
| **SU2** | An "open-source collection of software tools written in C++ and Python for the analysis of partial differential equations (PDEs) and PDE-constrained optimization problems on unstructured meshes"; applicability listed as aeronautical, automotive, ship and renewable energy industries; hosted by the US non-profit SU2 Foundation; the 7th SU2 Conference is scheduled for Milan in November 2026 | **LGPL 2.1**, stated on its own site, under active development |
| **Gmsh** | "A three-dimensional finite element mesh generator with built-in pre- and post-processing facilities" (Geuzaine & Remacle); four modules — geometry, mesh, solver, post-processing; scriptable via `.geo`, GUI, CLI, with C/C++/Python/Julia/Fortran APIs; current stable **4.15.2** (24 March 2026) | **GNU General Public License (GPL)**, stated on its own site |
| **PETSc** | "the Portable, Extensible Toolkit for Scientific Computation … for the scalable (parallel) solution of scientific applications modeled by partial differential equations"; C, Fortran and Python bindings (petsc4py); includes TAO (Toolkit for Advanced Optimization); supports MPI and GPUs through CUDA, HIP, Kokkos, OpenCL and hybrid MPI-GPU; documentation at 3.25.5; NumFOCUS-affiliated | **No licence claim made here** — the licence was not read at source in this pass (§16.3) |
| **code_aster** | Its own front page: "Open-source finite element solver – mechanics, thermal analysis, dynamics"; developed at EDF; the site states 37 years of development, **4,600 verification test cases**, approximately one million lines of code and 2,000 citations; source hosted on GitLab | The site describes it as open-source; the specific licence identifier was not read at source, so none is claimed here (§16.3) |
| **CalculiX** | "A Free Software Three-Dimensional Structural Finite Element Program"; built by a team of enthusiasts who are employees of **MTU Aero Engines** (Munich), which granted publication; linear and non-linear, static, dynamic and thermal; uses the Abaqus input format; its pre-processor can write mesh data for nastran, abaqus, ansys, code-aster and free-CFD codes; version 2.23 | **GNU General Public License, version 2 or (at its option) any later version**, as stated by the project |
| **LAMMPS** | "a classical molecular dynamics code with a focus on materials modeling" (Large-scale Atomic/Molecular Massively Parallel Simulator); potentials for solid-state materials and soft matter, plus coarse-grained and mesoscopic models; "runs on single processors or in parallel using message-passing techniques and a spatial-decomposition of the simulation domain"; CPU and GPU accelerated versions; developed at and long associated with Sandia; code on GitHub | **GPLv2**, as stated by the project |
| **GROMACS** | "A free and open-source software suite for high-performance molecular dynamics and output analysis"; simulates "the Newtonian equations of motion for systems with hundreds to millions of particles"; community-driven, primarily biochemical but also polymers and fluids; SIMD intrinsics for CPUs and CUDA/OpenCL/SYCL for GPUs, using CPU and GPU simultaneously; releases include 2026.3 (June 2026), 2026.0 (January 2026) and the 2025.5 patch (September 2026) | **No licence claim made here** — the project describes itself as free and open source, but the licence identifier was not read at source in this pass (§16.3) |
| **OpenMM** | "High performance, customizable molecular simulation … Use it as an application, a library, or a flexible programming environment"; bindings for Python, C, C++ and Fortran; optimised for NVIDIA, AMD, Intel and Apple GPUs and for CPUs; custom forces expressed as strings; an OpenMM-ML add-on for machine-learning interatomic potentials; an OpenMM-Setup UI; installable with `pip install openmm` | Distributed under "the permissive **MIT and LGPL** licenses", as stated by the project |

### 12.3 Commercial Tools, with Claims Labelled

| Item | What is verified | Claim quality |
|---|---|---|
| **Ansys is now part of Synopsys** | "Synopsys Completes Acquisition of Ansys" was announced by Synopsys on **17 July 2025** (the transaction having been announced 16 January 2024); press material refers to an expanded "$31 billion total addressable market"; Ansys product pages now redirect into the Synopsys domain | First-party corporate announcement. The market-size figure is a **vendor claim** about a market, not a measurement, and carries no stated methodology — treat it accordingly (§16.3) |
| **Ansys LS-DYNA** | Vendor description on its own product page: "Ansys LS-DYNA is the industry-leading explicit simulation software used for applications like drop tests, impact and penetration, smashes and crashes, occupant safety, and more." | **Vendor claim.** "Industry-leading" is a commercial assertion about position, not a measured fact. The *application list* (drop tests, impact, crash, occupant safety) is the vendor's own statement of scope and can be cited as such |
| **COMSOL** | Sells a "Software Product Suite" spanning products and industries under a commercial licence | **Not verified in this pass**: pricing, licence mechanics and the module list were not read at source. No figure, price or module name is asserted here (§16.3) |
| **Cadence data-centre thermal; ESI Group; Siemens Simcenter; Dassault Systèmes/Abaqus; STAR-CCM+** | Nothing verified in this pass — attempted pages failed or 404'd | **Recorded as unverified.** §16.3 records the absence rather than naming capabilities |

### 12.4 Buying a Suite versus Composing Libraries

This is the decision that most determines what a team can do, and it is not a quality judgement about either option.

**Buying a suite** gets you a validated-for-some-cases workflow, a GUI, documentation, support, a pre-built coupling between physics, and a licence cost that scales with seats, cores or modules. You get the vendor's physics and the vendor's numerics, and you can influence neither. Verification evidence exists but is the vendor's, usually summarised rather than published; you cannot run the vendor's code-verification study yourself. You can still do solution verification and validation on your own case, and you must.

**Composing libraries** — a mesh generator, a linear solver library, a finite-element or finite-volume library, and your own physics — gets you control over the discretisation, the boundary treatment, the solvers, the licence terms and the deployment shape, including running at scale without per-core licence negotiation. You also inherit the responsibility for **everything**: the discretisation, the conservation properties, the preconditioner choice, the parallel correctness, the code-verification harness, and the maintenance of a codebase whose authors may leave.

The honest framing, and the guide's view:

- The suite/library choice is a question about **who owns the verification evidence and the physics assumptions** — not about which is more rigorous. For standard physics on standard geometry, a suite's pre-built verification is worth a great deal, and the licence cost is frequently cheaper than the engineering time to rebuild it.
- For a method that must be modified, for couplings the suite does not support, for scale where licence terms bind, or for a workflow that must be scriptable and reproducible end to end, the library route is the only route — and it makes the verification discipline in §9 your deliverable rather than your assumption.

### 12.5 One Reading of the Landscape

Two facts in the table above are worth noting together, because they are evidence about the discipline rather than about the products:

1. **OpenFOAM's Foundation distribution publishes a maintenance campaign in the hundreds of thousands of euros and describes that as roughly 0.1% of the revenue of big commercial CFD** — the project's own figures, offered as an argument about funding, and citable as a first-party claim about its own funding position and the relative scale of the commercial market — and **code_aster, developed at a utility, states 4,600 verification test cases**, while OpenCFD's distribution describes a QA process with several hundred daily unit tests plus larger batteries before each release. These are project claims, not independently audited counts, but the *shape* of the claim is the point: serious simulation software treats code verification as a permanent, industrialised activity, not as a document written once for an audit. That shape is the answer to the most common objection to §9 — that verification is bureaucratic overhead. Mature projects do it continuously, at industrial scale, because an unverified solver is not a product.

---

## 13. The AI-for-Simulation Frontier, Evidence-Graded

This frontier is real, and it is heavily promoted. The treatment here separates what published work demonstrates, on which setting and which benchmark, from what is asserted.

### 13.1 What Published Work Demonstrates

Each row below is what the authors state they did, read at the work's own abstract page or venue in this pass (checked 2026-09-29).

| Work | Venue | Setting and benchmark | What the authors claim, scoped as they scope it |
|---|---|---|---|
| **Fourier Neural Operator (FNO)** — Li, Kovachki, Azizzadenesheli, Liu, Bhattacharya, Stuart, Anandkumar | arXiv:2010.08895 (v1 18 October 2020; v3 17 May 2021) | A neural operator that parameterises the integral kernel in Fourier space; experiments on **Burgers' equation, Darcy flow and Navier–Stokes** | "the first ML-based method to successfully model turbulent flows with zero-shot super-resolution"; "up to three orders of magnitude faster compared to traditional PDE solvers"; "superior accuracy compared to previous learning-based solvers under fixed resolution". **Author claims on their own cases.** "Three orders of magnitude faster" is a comparison against traditional solvers on those problems and must not be generalised |
| **DeepONet** — Lu, Jin, Karniadakis | arXiv:1910.03193 (v1 8 October 2019); published version in **Nature Machine Intelligence** (DOI 10.1038/s42256-021-00302-5) | Learns nonlinear operators from data with a branch network (input function at fixed sensors) and a trunk network (output locations); tested on dynamic systems and PDEs | Claims it "significantly reduces the generalization error compared to the fully-connected networks", with observed "high-order error convergence … polynomial rates (from half order to fourth order) and even exponential convergence with respect to the training dataset size". The **negative** result on plain fully connected networks is as informative as the positive one: architecture choice, not merely data volume, decides whether a learned operator generalises |
| **Physics-informed neural networks (PINNs)** — Raissi, Perdikaris, Karniadakis | arXiv:1711.10561 (28 November 2017), "Physics Informed Deep Learning (Part I)" | Networks "trained to solve supervised learning tasks while respecting any given law of physics described by general nonlinear partial differential equations"; two problem classes — data-driven solution and data-driven discovery | The originating paper of the PINN line. Results are on the PDEs in the paper; it is not a general capability claim, and the PDE residual used as a training loss is a *soft* constraint, not an exact one |
| **Graph Network-based Simulators (GNS)** — Sanchez-Gonzalez, Godwin, Pfaff, Ying, Leskovec, Battaglia | arXiv:2002.09405, **ICML 2020** | Particles as graph nodes with learned message passing; fluids, rigid solids and deformable materials interacting | Generalises "from single-timestep predictions with thousands of particles during training, to different initial conditions, thousands of timesteps, and at least an order of magnitude more particles at test time"; the main determinants of long-term behaviour are reported as the number of message-passing steps and noise-corrupted training data. **Describe the extrapolation exactly as scoped: one order of magnitude in particle count, on their settings** |
| **NeuralGCM** — Kochkov et al. (16 authors) | arXiv:2311.07222 (v3 8 March 2024); **Nature (2024)**, 92 pages, 54 figures | "the first GCM that combines a differentiable solver for atmospheric dynamics with ML components" — a **hybrid** | Competitive with ML models for 1–10 day forecasts and with ECMWF ensemble prediction for 1–15 days; with prescribed sea-surface temperature, "accurately track climate metrics such as global mean temperature for multiple decades"; at 140 km resolution, climate forecasts "exhibit emergent phenomena such as realistic frequency and trajectories of tropical cyclones"; "orders of magnitude computational savings over conventional GCMs". The strongest evidence class in this set — a peer-reviewed venue, and a hybrid rather than a pure data-driven replacement |
| **GraphCast** — Lam, Sanchez-Gonzalez, Willson et al. | arXiv:2212.12794 (v2 4 August 2023); code and weights published at github.com/deepmind/graphcast | Trained from reanalysis data; predicts hundreds of weather variables over 10 days at 0.25° globally | Predicts "hundreds of weather variables, over 10 days at 0.25 degree resolution globally, in under one minute", and "significantly outperforms the most accurate operational deterministic systems on 90% of 1380 verification targets". **Cite the denominator**: 1,380 verification targets is the honest part of that claim, and the comparison is against operational deterministic systems on those targets |

### 13.2 Reading the Field Correctly

Four different things are being done under one banner, and they have different failure modes:

- **Operator learning from simulated data (FNO, DeepONet).** The training set is generated by a classical solver. The learned model approximates the solution map of that solver — so its accuracy ceiling is set by the training data's fidelity, and any error in the generating solver is inherited.
- **Physics-constrained learning (PINNs).** The PDE residual enters the loss. This is attractive when data is scarce, but a residual that is small at the collocation points is a statement about those points, and the optimisation of a stiff multi-term loss is itself a hard numerical problem.
- **Learned simulators from data (GNS, GraphCast).** The update rule is learned directly. GraphCast's own benchmark is the cleanest example in the set of a claim with a stated denominator.
- **Hybrids (NeuralGCM).** A differentiable classical solver plus learned components. This is where the evidence is strongest, and it is worth noticing *why*: the classical solver is still there, doing the part it is trusted for.

### 13.3 The Structural Limitation

**A surrogate is valid only over its training distribution, and it cannot tell you when it is outside its own validity.** This is the single most important sentence in this section, and it is the guide's analysis of a structural property rather than a quoted finding.

The mechanism: a learned model's confidence is a property of the loss surface it was trained on, not of the physics. A classical solver responds to an out-of-range input by producing a solution that a knowledgeable engineer can recognise as wrong — a negative absolute temperature, a conserved quantity that does not balance, a residual that will not fall. A surrogate produces a number that looks exactly like its other numbers. There is no residual to inspect, because there is no equation being solved.

The practical consequences:

- **Distributional shift is the failure mode, and it is silent.** A surrogate trained on a family of geometries, Reynolds numbers or load cases will be used on the next one, and the boundary between "interpolation" and "extrapolation" is neither published with the model nor detectable from a single output.
- **Uncertainty quantification is the open research problem.** Producing a trustworthy error bar on a surrogate prediction — one that widens outside the training distribution — is not solved in a way that a reviewer can rely on.
- **The economics point the wrong way for high-stakes decisions.** The reason to adopt a surrogate is that it is fast; the reason to reject it is that its errors are unrecognisable. Speed is a benefit in proportion to how many evaluations you do, which is largest exactly where the individual answer matters least.

### 13.4 The Verdict

**An addition to the toolbox, not a replacement for the classical solvers.** Three facts support that verdict directly from §13.1:

1. **The frontier consumes the classical solver's output.** FNO and DeepONet are trained on solutions produced by classical solvers; the learned model's ceiling is the generating solver's accuracy, and the generating solver must be verified before its output is fit to.
2. **The frontier consumes the classical solver's benchmarks.** The verification and validation regimes in §9 — solution verification, order-of-accuracy studies, comparison against experiment — are exactly the instruments used to evaluate the surrogates, and GraphCast's 1,380 targets are a classical verification apparatus applied to a learned model.
3. **The best evidence in the set is a hybrid.** NeuralGCM's strength comes from keeping a differentiable classical solver for the atmospheric dynamics and learning the parts that classical modelling handles poorly.

Where the frontier is already the right tool: many-query outer loops (design optimisation, parameter sweeps, control), real-time approximation where a classical solve cannot meet the latency, and cases where the operator family is well characterised and the training distribution can be controlled. Where it is not: any decision where the failure mode has to be detectable. Adopting a surrogate does not remove the need for the classical solver or for §9; it adds a new entry to the error budget of §2 — a *learned-model* error class with a training distribution for a validity domain.

---

## 14. The Industries, and the Finance Parallel

### 14.1 Where Numerical Simulation Carries the Load

The assertions below are held at the level at which they are attributable: a project's own statement of its scope, or the class of engineering activity, without invented volumes, savings or cost figures. Where a claim about an industry's practice could not be sourced in this pass, it is stated as the *class* of problem the method family addresses rather than as an industry fact.

| Domain | The class of problem | Attribution basis |
|---|---|---|
| **Aerospace** | External aerodynamics around a configuration; structural loads and aeroelastic response; crash, impact and occupant safety; propulsion and combustion | Crash, impact and occupant safety are named in the vendor's own product description of an explicit dynamics code (§12.3), a vendor claim about scope. The verification literature cited in §9 is overwhelmingly drawn from this domain, which is why the discipline's vocabulary was standardised here first |
| **Automotive** | Crashworthiness and occupant safety; thermal management of powertrain and battery systems; aerodynamics and aeroacoustics | The explicit-dynamics vendor description names drop tests, impact, penetration and occupant safety; automotive is also named in SU2's own applicability list (aeronautical, automotive, ship, renewable energy) |
| **Energy** | Structural and thermal analysis of plant and components; neutronics and radiation transport; flow and heat transfer in cooling systems | First-party and directly citable: the code_aster solver is developed by the utility EDF and describes itself as an open-source finite element solver for mechanics, thermal analysis and dynamics, with a stated 4,600 verification test cases |
| **Semiconductor thermal and process** | Package and board-level thermal management; process modelling and equipment-scale transport; wafer-scale uniformity | **Held at class level.** No vendor or standards page for semiconductor or data-centre thermal simulation was successfully read in this pass, so no capability or product claim is made. The method families are those of §4: conduction and conjugate heat transfer on complex geometry, plus radiation |
| **Civil and structural** | Linear and nonlinear static analysis; dynamics and seismic response; stability, contact and material nonlinearity; geotechnical and soil–structure interaction | The finite-element codes in §12.2 describe themselves in exactly these terms — CalculiX as a free three-dimensional structural finite element program with linear and non-linear, static, dynamic and thermal capability, and deal.II as a library for adaptive finite element solution of PDEs |
| **Materials** | Atomistic and coarse-grained simulation of solids and soft matter; defect and interface behaviour; property estimation where continuum models are insufficient | LAMMPS describes itself as a classical molecular dynamics code with a focus on materials modelling, with potentials for solid-state materials and soft matter and coarse-grained/mesoscopic capability |
| **Pharmaceuticals and biochemistry** | Molecular simulation of proteins, ligands and membranes; free-energy estimation; formulation and materials behaviour | GROMACS describes itself as a suite for high-performance molecular dynamics whose systems run from hundreds to millions of particles and whose applications are primarily biochemical, also covering polymers and fluids. **Broader regulatory claims about simulation in drug approval were not verified in this pass and are not made here** |

One cross-cutting observation, offered as the guide's analysis: **the industries that use simulation most heavily are the ones whose verification vocabulary is most explicit.** Aerospace and nuclear engineering did not standardise V&V because they were unusually bureaucratic; they standardised it because a wrong answer in those domains is measured in lives, and because the failure mode — a converged solution of an unvalidated model — is otherwise undetectable. The vocabulary of §9 is a product of consequence, and adoption follows cost-of-failure rather than fashion: where a physical test is cheap and fast, simulation is a convenience; where the test is expensive, slow, destructive or impossible, simulation is the only instrument available and the standard of evidence demanded of it rises accordingly.

### 14.2 The Finance Parallel: Model Risk Is V&V under Another Name

This is the payoff of the guide for a reader in banking, and it is a structural observation rather than a claim about any institution.

A bank's model-risk discipline, as described in this repository's own material (`../banking/risk_management_models_guide.md`, `../banking/enterprise_risk_management_guide.md`, and the quant guides), asks three questions about a model — and they are the same three questions §9 asks, in a different vocabulary:

| The bank's question | The computational-science activity | The question, in both vocabularies |
|---|---|---|
| Is the model implemented correctly, and does the implementation do what the documentation says? | **Code verification** | Does the artefact do what it claims, independent of whether the claim is right? |
| Is the model being used inside its documented domain of applicability? | **Solution verification and scope discipline** | Is *this* use of the model inside the regime where the evidence supports it? |
| Do the model's outputs match reality, and how would we know if they stopped? | **Validation** | Does the model describe the world, with quantified accuracy, against independent data? |

Each side has an institution for each: on the simulation side, an independent V&V review and a code-verification harness; on the banking side, independent model validation, backtesting, benchmarking against challenger models, and documented limitations. Both insist that the second question is not answered by the first, and both insist that the third cannot be answered from inside the model. That is this guide's thesis restated in a bank's procurement language — and it is why a reviewer who has run model validation already knows how to read a simulation report, while a reviewer who has not will accept a residual norm as evidence.

### 14.3 The Counterpoint: Where the Analogy Stops

The parallel must not be pushed further than it holds:

- **The models are different in kind.** A bank's models are predominantly statistical and calibrated to data; a physics simulation solves differential equations derived from conservation laws. A statistical model's validity domain is a distributional statement; a physical model's is a regime statement.
- **The tolerances are set differently.** Model-risk tolerance in banking is often expressed in financial terms — an exception count, a P&L attribution threshold, a capital impact — with a business decision behind it. A simulation's acceptance tolerance is expressed in physical units against an experimental uncertainty. Both are legitimate; neither can be imported into the other.
- **Convergence means something else.** Where banking material discusses model convergence it usually means the numerical convergence of an optimisation or an estimation procedure, or of a Monte Carlo estimate. Neither is grid convergence, and neither is a statement about the physical world — the same word-collision as §1.3.3's Monte Carlo.
- **There is no order of accuracy in a statistical model.** The refinement study of §9.3 has no counterpart; the corresponding discipline is out-of-sample and out-of-time testing, a different instrument answering a related question.
- **Operational conditions differ.** A traded model confronts a moving market; a physics model confronts equipment built once. The first raises the frequency of revalidation; the second raises the cost of measuring at all.

The transferable part is the *discipline of the three questions and their separation* — not the methods, the tolerances or the vocabulary.

### 14.4 The Guide's Analysis: Where a Bank Actually Meets Physics

**This subsection is the guide's own analysis, labelled as such, not a sourced finding.**

For most of a bank's operations the models are financial, and the physical world enters only through the infrastructure that runs them. There is one place where the models are physical and the discipline of this guide applies directly: **the data centre.** A hall's cooling capacity, its airflow pattern, its hot-spot behaviour under a changed load profile and its response to a containment or setpoint change are predictions about a physical system governed by the same equations — mass, momentum and energy conservation in a fluid, plus conduction and radiation — that this guide is about. A bank that commissions a computational fluid dynamics study of a hall, or that changes a rack layout on the strength of one, has bought a numerical simulation model whose validity it must now assess. The apparatus of §9 — is the code verified, is this study's numerical error bounded, has anything been measured in this hall — is exactly what the review needs, and §15 is that review.

The repository's `data_center_guide.md` owns data-centre engineering — power, cooling architecture, containment, redundancy. This guide owns the question of what a *simulation* of a data centre is worth.

---

## 15. The Cymbal Bank Worked Example

**This section is entirely fictional and is labelled as such.** Cymbal Bank is a fictional bank persona used in this repository's worked examples. The vendor, the report, the hall and the numbers in it exist only in this section. No real vendor, institution or project is described, and no real product is asserted to have any behaviour attributed to it here. Cymbal Bank is the only institution used as a worked example anywhere in this guide.

### 15.1 The Situation

Cymbal Bank's infrastructure team is planning a change to the cooling arrangement in one data hall — a containment modification plus a change of supply setpoint — and holds a vendor-supplied computational fluid dynamics (CFD) study of the hall as it is today and as it would be after the change. The study's headline is a predicted reduction in peak rack inlet temperature. The infrastructure team is ready to commit capital. Before it does, it asks for an engineering review, because the decision is about physics.

The review's mandate is the mandate §9 supplies: **establish what the model claims; establish what the meshing and boundary conditions assumed; establish whether any verification or validation evidence exists; and separate the vendor's predictions from the vendor's assumptions.**

### 15.2 What the Model Claims

The report describes a three-dimensional steady-state simulation of the hall using a Reynolds-averaged (RANS) turbulence treatment, with a prescribed total IT heat load distributed across the racks by nameplate figure, a prescribed supply air temperature and flow from the cooling units, and a stated set of leakage paths. Outputs: a field of temperature and velocity; rack inlet temperatures; a recirculation metric; and the headline delta between the "before" and "after" configurations.

The review's first finding is definitional and everything else hangs on it. **What is claimed is a converged solution of a specific set of equations with a specific closure, on a specific mesh, with specific boundary data.** That is a claim about a discrete problem. It is not a claim about the hall, and the report's language does not distinguish the two.

### 15.3 What the Meshing and Boundary Conditions Assumed

| Item the report states | What was actually established | Which error class this touches |
|---|---|---|
| Cell count for the hall, given as a single figure, with no refinement study | No mesh-quality metrics were reported — no skewness or non-orthogonality statistics, no cell-size-jump distribution, no boundary-layer resolution at the racks or the cooling units, no evidence of a defect-free mesh — and no solution-verification evidence on any quantity of interest, so nothing shows whether the reported rack inlet temperatures are stable under mesh refinement or in which direction they move | Discretisation error, unbounded locally and otherwise unquantified (§7.3) |
| "Heat load per rack: nameplate rating" | An assumption, not a measurement. Nameplate rating is the maximum the equipment is provisioned to draw, and the actual load in a mixed hall is typically different, and unevenly distributed across the row | **Input error** — and not what the bank would find if it measured (§8.2) |
| "Supply temperature: setpoint" | An assumption about the cooling unit's delivered temperature at the point that matters. Air leaving a unit at a setpoint does not arrive at a rack at that temperature, and the difference is the physics under study | Input error feeding directly into the predicted quantity |
| "Leakage paths as specified" | An assumption about the openings in the containment. If the containment is leakier or tighter in reality, the airflow pattern that carries the cooled air changes — and the airflow pattern *is* the result | Input error, and a modelling choice about the geometry |
| RANS turbulence treatment | A model-form choice presented as a software setting (§8.3). The closure's validity in the recirculating, buoyancy-influenced, partially confined flow of a data hall is precisely the open question | **Model error** |
| Residuals reported as having fallen by several orders of magnitude | Iteration convergence described, but with no statement of the criterion's basis, no sensitivity of the reported quantities to the criterion, and no evidence that a small residual implies a small error here | Iteration error, only loosely bounded |

### 15.4 Predictions versus Assumptions

The review's central deliverable is the table the report does not contain — the separation of what was computed from what was put in.

| The vendor's number | Predicted or assumed? | Why it matters |
|---|---|---|
| Peak rack inlet temperature, in the as-is and the modified configuration | **Predictions, conditional on every assumption above** | Their credibility is exactly the credibility of the assumed heat load and supply condition; a prediction cannot be better than its inputs, and the conditional structure is identical in the two cases |
| The headline reduction in peak rack inlet temperature | **Arithmetic on two predictions** — a difference of two conditional numbers | A difference of two uncertain numbers is *more* uncertain than either one, because the errors need not cancel. Its stated precision implies a confidence the study never established |
| Recirculation index for the hall | Prediction, conditional on assumed leakage and load distribution | Useful as a comparative diagnostic *given* the assumptions; not a measured property of the hall |
| "No hot spots after the change" | Prediction conditioned on the assumed load being correct and uniform as assumed | The negative claim is the strongest claim in the report and the least supported: it asserts the absence of a condition that, if one rack's load assumption is wrong, is exactly what occurs |
| Mesh, closure, supply condition, load, leakage | **Assumptions** | The report's premises, not its outputs — and every one of them was silent in its summary |

Two further characteristics the review flagged as diagnostic rather than decisive: the absence of any stated uncertainty on the headline figure, and the absence of any statement of the model's validity domain. Neither is a numerical defect; both are the signature of a study that has not been through §9.

### 15.5 The Three Questions of §9, Applied

| Question | Evidence in the vendor study | The review's conclusion |
|---|---|---|
| **Code verification** — is the code solving its equations correctly? | The vendor used a commercial suite. Verification evidence exists in principle, but was not made available to the bank in a form it could assess, and could not be reproduced by the bank | **Not established.** The bank cannot verify a code it does not hold, and should not treat the possession of a licence as evidence |
| **Solution verification** — how much numerical error is in this specific answer? | No refinement study, no mesh-quality evidence, no iteration-tolerance sensitivity, no error estimate on any quantity of interest | **Not established** — and this one is the vendor's to provide, at a small fraction of the cost of the study itself |
| **Validation** — does the model describe the real hall? | Nothing in the report was compared with any measurement taken in the hall. Existing sensor readings — cooling unit return temperatures, some PDU power telemetry — were not used | **Absent.** This is the only one of the three that could have told the bank anything about whether the model was right |

**The review's finding: everything had converged, and the review still could not establish that any of it was right.** The residuals fell. The solution was stable and repeatable. The mesh had enough cells to look serious. And at the end of the exercise, the bank held a confidently stated temperature difference whose two dominant inputs were a nameplate rating and a setpoint, neither of which had been measured in the hall, produced by a closure nobody had justified for this geometry, on a mesh nobody had tested. The convergence was real, and it was evidence about none of that.

### 15.6 The Boundary the Bank's Own Testing Does Not Cross

Cymbal Bank runs deterministic simulation testing of its own software and has a mature practice for it (`deterministic_simulation_testing_guide.md`). It is worth stating, precisely because the same word appears in both activities, that **the two are unrelated and neither substitutes for the other.** In the bank's software testing, the *system under test* is the software and the simulation replaces the world the software runs in; the artefact judged is an implementation and the judge is a set of assertions. In the vendor's hall study, the *simulation is the object of study*, the world is the equations, and the artefact judged is a claim about a physical hall. The bank's testing practice could have been flawless — it was — and it would still have said nothing about whether the hall model was right; conversely, no amount of CFD validation says anything about whether a payment service behaves correctly under a partitioned network. Same word, opposite direction (§1.3.1).

### 15.7 What the Bank Did Because of the Review — and Why It Is Not a Simulation

The review did not ask for a finer mesh or a different turbulence model. It noted those as gaps, and then did the one thing that could change the evidentiary position: it **commissioned an instrumented measurement campaign in the hall — a validation programme** whose specification was:

- **Measure the inputs that were assumed.** Rack-level power draw from existing per-outlet telemetry, aggregated to the row and to the rack over a period long enough to capture the diurnal profile, converting the nameplate assumption into a measured distribution.
- **Measure the quantity that was predicted.** Rack inlet temperature at defined points, at a defined height in the rack front, with the instrument's uncertainty stated — the temperature at the rack inlet is what the study predicted, so it is what must be measured (the ASME V&V 20 idea of a specified variable at a specified validation point, applied in an infrastructure setting).
- **Measure the flow, not just the temperature.** Airflow at a sample of floor tiles and at containment openings, testing the leakage assumption rather than assuming it.
- **Run it before and after the change on a staged basis** — one row or one zone first — so the change is observed rather than predicted and the risk of the full commitment is bounded by evidence, with the comparison rule (including what would count as the model being wrong) stated in advance.

That programme is validation, and it is the only one of §9's three activities that can answer the bank's actual question. It is also the one that has nothing to do with the simulation: it needs thermocouples, a data logger, a schedule and someone to climb into the hall. It costs a fraction of the capital being committed, it improves the model's inputs permanently, and it produces the first evidence the bank has ever held about whether its hall-prediction capability is worth anything at all.

The study itself was not wasted. It located where the questions are, which is what a model is good for when it has not been validated: narrowing where to look. It just was not evidence, and the review's whole job was to say so without being told which number to believe.

---

## 16. The Anti-Patterns, the Claims Audit, What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 The Anti-Patterns

| Anti-pattern | What it looks like in practice | The mechanism that makes it wrong | What to do instead |
|---|---|---|---|
| **"It converged" as a conclusion** | A residual plot pasted into a report as the evidence for the number; a "did not change much" note from a refinement study with no order reported | Convergence is a statement about the algebraic problem, not about the model, the data or the world (§2.2); and a refinement study measures an *order*, so without the rate it cannot detect a lower-than-expected one (§6.2) | State the four-part error budget; compute the observed order and compare it with the scheme's formal order |
| **Mesh refinement as the fix for everything** | A wrong answer met with "let's refine and see", followed by more cores and more cells | Refinement and scale address discretisation error only (§2.2, §11.6) | Identify which error class dominates *first*, then spend compute on it |
| **Validation by comparison with another simulation** | "Our result agrees with the published case" | Same-model agreement is not agreement with the world; it confirms an implementation or a use, not a physical claim (§9.1) | Compare with measurement, and state the experimental uncertainty |
| **The unmeasured input presented as a value** | A conductivity, heat-transfer coefficient or load given to four significant figures with no provenance | The precision implies evidence that does not exist; a converged solution built on it is confidently wrong (§8.2) | State the provenance of every input; vary the uncertain ones and report the spread |
| **A model-form choice treated as a setting** | The turbulence or constitutive model chosen from a menu with no justification | The closure changes what the reported number means; it is a physical approximation, not a numerical option (§8.3) | Justify the closure for this regime, or state that it was not justified |
| **Choosing the mesh to get the answer** | Refining locally until the quantity of interest matches an expectation | This is calibration to the desired output; the mesh has become an input to the conclusion and the study can no longer test it | Decide the mesh strategy from the physics and geometry, and record the decision before seeing the result |
| **Reproducibility assumed rather than engineered, and output nobody planned for** | Confidence in bit-identical reruns of a large parallel job; a run that finishes and cannot be analysed because the fields were not written at the right frequency | Parallel reductions, decomposition, compiler behaviour and library functions all change the last bits (§10.3); and I/O limits come from the arithmetic of the output, routinely the binding constraint (§11.5) | Compare with tolerances and a rationale; engineer reproducibility where it is required; decide the reduction, checkpoint cadence and file layout before the run |
| **The learned surrogate treated as a solver** | A model trained on simulations used outside its training distribution, with no out-of-range signal | A surrogate has no residual and cannot report that its input is outside its validity (§13.3) | Restrict use to the training distribution; check against the classical solver at the boundary |
| **Borrowing the vocabulary without the discipline** | "V&V" used to mean unit tests and user acceptance; "converged" used of a Monte Carlo estimate | Two vocabularies with the same words and different referents (§1.3, §9.5, §14.3) | Name which V&V is meant, and which convergence |

### 16.2 The Claims Audit

Every substantive claim in this guide, with its source, the date it was checked, and its quality. Project, vendor and authorial claims are labelled as such and are not restated as measured facts.

| Claim | Source | Date checked | Quality / flag |
|---|---|---|---|
| AIAA G-077-1998: verification is "the process of determining if a computational simulation accurately represents the conceptual model, but no claim is made of the relationship of the simulation to the real world"; validation is "the process of determining if a computational simulation represents the real world"; approved 14 January 1998, reaffirmed R2002; the first consensus V&V guide from an accredited standards developer | AIAA, via the ANSI webstore preview pages | 2026-09-29 | Standard |
| ASME V&V 10-2019 supersedes V&V 10-2006; purpose "to provide the CSM community with a common language, a conceptual framework, and general guidance for implementing the processes of computational model V&V"; companions V&V 10.1-2012 (illustration), V&V 10.2 (uncertainty), V&V 10.3 (metrics) | ASME / ANSI webstore | 2026-09-29 | Standard |
| ASME V&V 20-2009 (R2016, R2021) objective: an approach that "quantifies the degree of accuracy inferred from the comparison of solution and data for a specified variable at a specified validation point", drawing on experimental uncertainty analysis and scoped to cases where the conditions of the actual experiment are simulated | ASME / ANSI webstore | 2026-09-29 | Standard |
| Oberkampf & Trucano (2002), *Progress in Aerospace Sciences* 38(3) 209–272, DOI 10.1016/S0376-0421(02)00005-2: V&V "are the primary means to assess accuracy and reliability in computational simulations" | ScienceDirect / Sandia | 2026-09-29 | Peer-reviewed |
| The method of manufactured solutions: procedure and applicability to codes that numerically solve partial differential equations (procedure demonstrated in this guide on the advection–diffusion equation, §9.2) | Salari & Knupp, SAND2000-1444, June 2000; journal version Roache, *ASME J. Fluids Engineering* 124(1), 4 (2002) | 2026-09-29 | Peer-reviewed / institutional report |
| Verification "consists in demonstrating that the code is free of coding errors and is capable, given sufficient discretization, of approaching exact mathematical solutions" | Springer, *Handbook of Materials Modeling*, ch. 10.1007/978-3-319-70766-2_12 | 2026-09-29 | Peer-reviewed (abstract read, not the chapter body) |
| IEEE 1012 is the software V&V standard used by this repository's LLM evaluation material; that V&V is artefact-against-requirements, not solution-against-reality | `ai_llm/llm_evaluation_vs_validation_guide.md` | 2026-09-29 | Secondary (repo) |
| IEEE 754-2019: PAR approval 2015-09-03, board approval 2019-06-13, published 2019-07-22, supersedes IEEE 754-2008, committee C/MSC; scope covers interchange and arithmetic formats and methods, exception conditions and their default handling, realisable entirely in software, entirely in hardware, or both | standards.ieee.org and the standard's own abstract | 2026-09-29 | Standard |
| TOP500 November 2025 list: rank 1 El Capitan (HPE Cray EX255a, AMD 4th-gen EPYC + MI300A, DOE/NNSA/LLNL), Rmax 1,809.00 PFlop/s, Rpeak 2,821.10 PFlop/s, 29,685 kW; rank 2 Frontier 1,353.00; rank 3 Aurora 1,012.00; Rpeak computed from advertised clock rate; a June 2026 list exists | top500.org, list 2025/11 | 2026-09-29 | Secondary (list with published methodology) |
| Derivations performed in this guide: the CFL bound Δt ≤ Δx/a from the von Neumann mode argument and independently from the domain of dependence; numerical diffusivity ν_num = (a·Δx/2)(1 − C) from the modified-equation expansion; central-difference truncation terms (Δx²/12)·u'''' and (Δx²/6)·u'''; manufactured-solution source term S = A·ν·k²·sin(kx − ωt); unit chains carried through each | Derived in §5.4 and §9.2 | 2026-09-29 | Guide's own derivation (checkable inline) |
| Measured arithmetic: 1 − cos(1e-8) = 0.0 in double precision while 2·sin²(x/2) = 5.0000000000000005e-17 (relative error 1.0); double unit roundoff 2.220446049250313e-16 and single-precision 1.1920928955078125e-07; 200,000-term harmonic sum 12.78329081042982 in double (relative error 1.54e-14), 12.782756805419922 in float32 (relative error 4.18e-05), exact-rounded 12.783290810429623; maximum amplification exactly 1.000000000000 for C ∈ [0.1, 1.0] and 1.0002, 1.02, 2.0, 3.0 for C = 1.0001, 1.01, 1.5, 2.0 (and 2.0 for C = −0.5); manufactured-solution source check agreeing to 4.8e-06; El Capitan Rmax/Rpeak = 64.1%; one 3-D refinement level multiplying cells by 8 and total work by 16; Amdahl speedups and efficiencies in §11.2; 1000³ cells × 200 bytes = 186 GiB | Computed for this guide in this session | 2026-09-29 | First-party arithmetic (guide's own) |
| OpenFOAM, two distinct distributions: openfoam.org (The OpenFOAM Foundation) is free CFD software, GPL on its licence page, and publishes a maintenance campaign — supporting organisations "currently provide €250k for maintenance of OpenFOAM, i.e. of the order of 0.1% of the revenue of big commercial CFD" with a stated €500k goal and Platinum/Gold/Silver levels of €100k/€25k/€5k — with core development at CFD Direct including OpenFOAM's creator Henry Weller. openfoam.com (OpenCFD Ltd) is "the free, open source CFD software developed primarily by OpenCFD Ltd since 2004", released every six months (v2512 December 2025, v2606 26 June 2026, both on the site's own news page), with repositories moved to gitlab.com/openfoam in November 2025; its About page says OpenCFD Ltd is "a wholly owned subsidiary of ESI Group" while its 1 November 2024 news item is titled "ESI Business Unit, as part of Keysight Technologies", and its QA process of "code evaluation, verification and validation" includes "several hundred daily unit tests", a weekly battery and a pre-release industrial battery | openfoam.org, openfoam.org/licence/, openfoam.com, openfoam.com/about, openfoam.com/news/main-news | 2026-09-29 | First-party / project claims. **The openfoam.com licence was not read at source. Note the two corporate statements on the site are not worded identically and both are quoted in §12.2 rather than resolved into one** |
| FEniCS/FEniCSx is an open-source computing platform for solving PDEs with the finite element method, components UFL, Basix, FFCx and DOLFINx, FEniCSx 0.11 released June 2026, legacy FEniCS 2019.1.0 (April 2019), NumFOCUS fiscally supported; the DOLFINx repository reports LGPL-3.0 and GPL-3.0 licences. deal.II is "a C++ program library targeted at the computational solution of partial differential equations using adaptive finite elements", version 9.8.0 released 2026-08-09, described by the project as "open source and available for free" | fenicsproject.org, github.com/FEniCS/dolfinx, dealii.org, github.com/dealii/dealii | 2026-09-29 | First-party. **deal.II's licence identifier was not read at source** |
| SU2 is an open-source collection of C++ and Python tools for PDE analysis and PDE-constrained optimisation on unstructured meshes, LGPL 2.1, hosted by the non-profit SU2 Foundation, 7th SU2 Conference in Milan, November 2026. Gmsh is "a three-dimensional finite element mesh generator with built-in pre- and post-processing facilities", GPL, current stable 4.15.2 (24 March 2026). PETSc is a toolkit "for the scalable (parallel) solution of scientific applications modeled by partial differential equations", with C/Fortran/Python bindings, TAO, MPI and GPU support through CUDA, HIP, Kokkos and OpenCL, documentation at 3.25.5, NumFOCUS-affiliated | su2code.github.io, gmsh.info, petsc.org | 2026-09-29 | First-party. **PETSc's licence was not read at source** |
| code_aster: "Open-source finite element solver – mechanics, thermal analysis, dynamics", developed at EDF, with a stated 37 years of development, 4,600 verification test cases, ~1 million lines of code and 2,000 citations. CalculiX: "A Free Software Three-Dimensional Structural Finite Element Program", GPL version 2 or later, built by employees of MTU Aero Engines, version 2.23, using the Abaqus input format | code-aster.org, calculix.de | 2026-09-29 | First-party project claims (the counts are not independently audited). **code_aster's licence identifier was not read** |
| LAMMPS is "a classical molecular dynamics code with a focus on materials modeling", GPLv2, parallel by spatial decomposition, with CPU and GPU accelerated versions. GROMACS is "a free and open-source software suite for high-performance molecular dynamics and output analysis" for systems of hundreds to millions of particles, with SIMD CPU and CUDA/OpenCL/SYCL GPU support, releases including 2026.3 (June 2026), 2026.0 (January 2026) and the 2025.5 patch (September 2026). OpenMM is high-performance molecular simulation usable as an application, a library or a programming environment, with Python/C/C++/Fortran bindings, custom forces expressed as strings and an OpenMM-ML add-on, distributed under MIT and LGPL | lammps.org, gromacs.org, openmm.org | 2026-09-29 | First-party. **GROMACS's licence identifier was not read at source** |
| Synopsys completed its acquisition of Ansys on 17 July 2025 (announced 16 January 2024), with press material referring to an expanded "$31 billion total addressable market"; the Ansys product page states that "Ansys LS-DYNA is the industry-leading explicit simulation software used for applications like drop tests, impact and penetration, smashes and crashes, occupant safety, and more" | Synopsys and Ansys investor releases; ansys.com product page | 2026-09-29 | First-party corporate announcement; **the market figure is a vendor claim with no stated methodology, and "industry-leading" is a vendor claim** — the application list is the vendor's own statement of scope |
| COMSOL sells a commercial "Software Product Suite" across products and industries | comsol.com (via the source pack) | 2026-09-29 | **Not verified** beyond the existence of a commercial suite; pricing, licence mechanics and module names were not read and are not asserted |
| FNO: a neural operator parameterising the integral kernel in Fourier space; experiments on Burgers' equation, Darcy flow and Navier–Stokes; "the first ML-based method to successfully model turbulent flows with zero-shot super-resolution"; "up to three orders of magnitude faster compared to traditional PDE solvers" | arXiv:2010.08895 | 2026-09-29 | Author claims on their own benchmark cases; the speed-up comparison is against traditional solvers on those cases and is not generalised here |
| DeepONet: branch and trunk networks learning nonlinear operators from data; "significantly reduces the generalization error compared to the fully-connected networks"; observed error convergence "from half order to fourth order" and exponential in training-set size | arXiv:1910.03193; Nature Machine Intelligence, DOI 10.1038/s42256-021-00302-5 | 2026-09-29 | Peer-reviewed / author claims; the negative result on plain fully connected networks is reported alongside |
| PINNs: networks "trained to solve supervised learning tasks while respecting any given law of physics described by general nonlinear partial differential equations", with data-driven solution and data-driven discovery classes | arXiv:1711.10561 | 2026-09-29 | Preprint (the originating paper of the line); results are on the PDEs in the paper |
| GNS: particles as graph nodes with learned message passing across fluids, rigid solids and deformable materials; generalises from thousands of particles in training to "at least an order of magnitude more particles at test time"; long-term performance driven by message-passing steps and noise-corrupted training data | arXiv:2002.09405, ICML 2020 | 2026-09-29 | Peer-reviewed / author claims, scoped exactly as stated |
| NeuralGCM: "the first GCM that combines a differentiable solver for atmospheric dynamics with ML components"; competitive with ML models at 1–10 days and with ECMWF ensemble prediction at 1–15 days; with prescribed sea-surface temperature tracks global mean temperature "for multiple decades"; at 140 km exhibits realistic tropical-cyclone frequency and trajectories; "orders of magnitude computational savings over conventional GCMs" | arXiv:2311.07222; Nature (2024) | 2026-09-29 | Peer-reviewed / author claims. **A hybrid, and the strongest evidence class in the set** |
| GraphCast: hundreds of weather variables over 10 days at 0.25° globally in under one minute; "significantly outperforms the most accurate operational deterministic systems on 90% of 1380 verification targets" | arXiv:2212.12794; github.com/deepmind/graphcast | 2026-09-29 | Author claims with a stated denominator (1,380 targets) — the denominator is the honest part and is cited here |
| Repo dedup counts over 630 tracked `.md` files: `finite element` 0, `finite volume` 0, `FEM` 0, `OpenFOAM` 0, `Navier-Stokes` 0, `molecular dynamics` 0, `CFL condition` 0, `spectral method` 0, `manufactured solution` 0, `finite difference` 1 file (Black-Scholes PDE for quants), `CFD` 4 files (all contract-for-difference or cumulative-flow-diagram senses), `Monte Carlo` 27 files (financial, the Monte Carlo Data vendor, and RL sampling), `surrogate model` 5 files (all machine-learning senses), `verification and validation` 3 files | Repo dedup pass recorded in the source pack | 2026-09-29 | First-party (repo measurement) |

### 16.3 What Could Not Be Verified

Recorded rather than asserted. Where a licence or capability could not be read at source, the corresponding row above carries no claim.

1. **Licence identifiers not read at source** for deal.II, PETSc, GROMACS, code_aster and the openfoam.com (OpenCFD Ltd) distribution. Each is described here in the project's own words and without a licence claim. FEniCSx/DOLFINx (LGPL-3.0 and GPL-3.0, repository detection) and OpenFOAM from the Foundation (GPL), SU2 (LGPL 2.1), Gmsh (GPL), CalculiX (GPL v2+), LAMMPS (GPLv2) and OpenMM (MIT and LGPL) **were** verified at source in this pass.
2. **Commercial licence mechanics, pricing and module lists**: COMSOL was reached only as a description of a commercial suite; Siemens Simcenter, Dassault Systèmes/Abaqus and STAR-CCM+ were not reached at all. No price, licence mechanism or module name is asserted anywhere in this guide. The Ansys–Synopsys ownership fact **was** verified (17 July 2025).
3. **Data-centre and semiconductor thermal simulation products**: no vendor page was read successfully in this pass, so §14.1 holds that domain at class level and §15's worked example uses a fictional vendor, asserting nothing about any real product.
4. **Classic verification and benchmark cases.** Candidate named cases exist in the discipline — cavity flows, backward-facing steps, turbulent channel flows, shock-tube problems, vortex benchmarks, workshop case collections — but **no primary source for any of them was read in this pass**, so none is named as a case and no reference solution, grid or tolerance is quoted. §9 describes the *class* of verification activity and demonstrates the mechanism by derivation instead. Anyone citing a named benchmark must verify it at source.
5. **Turbulence modelling characterisation.** The RANS / large-eddy / direct-simulation comparison in §8.3 is written as the guide's own explanation of mechanism. No source characterising the three approaches was verified at source in this pass; the treatment states what each represents and what is modelled, asserts no ranking and quotes no accuracy figure.
6. **Amdahl's law, the roofline model and the strong/weak scaling definitions.** Standard concepts, used in §11 as the guide's explanation with the arithmetic shown so a reader can check it. No primary source (Amdahl 1967 or equivalent) was verified at source, so none is cited.
7. **The Lax–Richtmyer equivalence statement** is cited in §6.1 as the classical result "as it is stated throughout the numerical-analysis literature", with its hypotheses (linear, well-posed, consistent) stated explicitly. The primary source was not read in this pass. Roache's *Verification and Validation in Computational Science and Engineering* had an unverified publisher and year, so only the 2002 journal article, which **was** verified, is cited. Grid-convergence-index procedures are referenced only as the activity ASME V&V 20-2009 addresses in scope: no formula, index value or acceptance threshold is quoted anywhere in this guide.
8. **Market size, spend and compensation figures for simulation software, and industry application claims.** No named source with a date and a stated methodology was found for the former, so **no market figure is given** anywhere in this guide — the one monetary figure quoted (the OpenFOAM Foundation's maintenance campaign) is first-party, is about that project's own funding, and is used only as such. Industry claims are held at attributable class level in §14.1 wherever a specific practice could not be sourced, and regulatory or approval-related claims about simulation (for example in pharmaceutical development) were not verified and are not made.
9. **Tool-limitation record.** `web_search` was reported unreliable on this host during this pass, so verification was attempted with direct page extraction on primary URLs only. Several obvious licence-page URLs returned 404 (the FEniCS licence path, the deal.II license path, the PETSc licence path, the GROMACS manual licence path); the corresponding information was then sought on each project's own repository or front page, and where that did not surface the identifier, the absence is recorded above rather than filled with a commonly cited value. An empty search result is a limitation of the tool, not evidence that the material does not exist.

### 16.4 Glossary

| Term | Meaning in this guide |
|---|---|
| **Order of accuracy** | The exponent p in e(h) ≈ c·h^p: the rate at which discretisation error falls as the mesh is refined |
| **Consistency** | The discrete operator tends to the continuous operator as the mesh and timestep vanish |
| **Stability** | Perturbations in the discrete solution do not grow without bound as the computation proceeds |
| **Convergence (iteration)** | The residual of the discrete system has fallen to a tolerance; the algebraic problem is solved |
| **Convergence (grid)** | The discrete solution approaches the exact solution of the PDE as the mesh vanishes, at the expected rate |
| **Residual** | The amount by which the current solution fails to satisfy the discrete equations |
| **CFL condition / Courant number** | The stability bound requiring the numerical domain of dependence to contain the physical one — Δt ≤ Δx/a for explicit advection — expressed through C = a·Δt/Δx (§5.4) |
| **Model error** | Difference between the real system and the mathematical model of it; reduced only by validation and model change |
| **Input error** | Error in the data fed to the model: properties, loads, boundary values, dimensions |
| **Discretisation error** | Difference between the exact solution of the PDE and the exact solution of the discrete equations |
| **Iteration error** | Difference between the exact solution of the discrete equations and the answer the solver returned |
| **Round-off error** | Error from finite-precision representation and the order of operations |
| **Code verification** | Demonstrating that the implementation solves its intended equations correctly (MMS; order of accuracy) |
| **Solution verification** | Estimating the numerical error in one specific computation (grid- and timestep-convergence studies) |
| **Validation** | Determining whether the model represents the real world, by comparison with physical measurement |
| **Method of manufactured solutions** | Choosing a closed-form solution, deriving the source term it implies, and checking the observed order of accuracy (§9.2) |
| **RANS / LES / direct simulation** | Three turbulence treatments: average the flow with a closure, resolve the large eddies and model the small ones, or resolve everything |
| **Surrogate model** | A learned approximation of a solver's solution map, valid over its training distribution only |

### 16.5 Cross-References

| Guide | What it owns, relative to this one |
|---|---|
| `deterministic_simulation_testing_guide.md`, `deterministic_engineering_guide.md`, `antithesis_guide.md` | Simulation as software testing, where the system under test is the software and the environment is simulated — the opposite direction of the word (§1.3.1, §15.6) |
| `physical_ai_guide.md` | Physical AI and robotics; its §5 owns sim-to-real transfer, platforms and synthetic data (§1.3.2) |
| `gpu_optimization_guide.md`, `gpu_cloud_providers_comparison_guide.md`, `hami_gpu_sharing_guide.md`, `low_latency_cpp_development_guide.md` | The accelerator substrate and kernels (§11.1). There is no `gpu_cloud_guide.md` in this repository |
| `quantitative_developer_skillset_guide.md`, `optimization_technologies_survey.md` | Numerical methods as a quant skill (the only `finite difference` file in the tree, Black-Scholes PDE) and optimisation, including a PDE-constrained optimisation row |
| `ai_llm/llm_evaluation_vs_validation_guide.md` | Software V&V (IEEE 1012) and the evaluation/validation distinction in machine learning (§9.5) |
| `data_center_guide.md` | Data-centre engineering — power, cooling architecture, containment, redundancy (§14.4) |
| `../banking/risk_management_models_guide.md`, `../banking/enterprise_risk_management_guide.md`, `../banking/transaction_foundation_model_guide.md`, `../banking/customer_behaviour_modeling_guide.md` | Model risk in banking — the discipline whose three questions parallel V&V (§14.2) — and model development, including the surrogate-fidelity question that mirrors §13.3 |

### 16.6 The Closing Summary

Numerical simulation is the discipline of translating a physical model into a computable one, and the translation is lossy in four separable ways. Model error is the gap between the world and the mathematics. Input error is the gap between what was fed in and what is true. Discretisation error is the gap between the continuous operator and the discrete one. Iteration and round-off error are the gaps between the discrete problem and the number the machine printed. Mesh refinement, higher order and tighter tolerances act on the third and only partially on the fourth. Nothing numerical acts on the first two — and the standards exist because that asymmetry is not visible from inside a solver.

Code verification, solution verification and validation are three different activities answering three different questions, defined in AIAA G-077-1998 and developed in the ASME V&V standards and in Oberkampf & Trucano: whether the code solves its equations correctly, how much numerical error is in this particular answer, and whether the model describes reality. Only the third needs the world, and only the third can be — and routinely is — skipped, because it requires an experiment and data rather than more compute. The techniques themselves are mature and the arithmetic is honest: the CFL bound is a derivable consequence of the domain of dependence, the order of accuracy is a measurable rate, the error budget is a budget, and the reproducibility of a parallel run is an engineering property rather than a given. The AI-for-simulation frontier is real, is producing peer-reviewed results on real benchmarks, and consumes the classical solver's solutions and its verification apparatus — an addition to the toolbox, not a replacement for it.

Everything converges. The residual falls, the mesh study flattens, the solution stabilises, and the report looks clean. And none of it establishes that the answer is right.

convergence is not correctness.

