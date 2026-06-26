# Modelling & Implementation Decisions

This isn't a physical design project, so "concept design" here means the
**modelling and numerical-method choices** I made at each stage, and why
I picked what I picked over the alternatives the brief allowed.

## 1. Matrix assembly: hand-derived vs. auto-generated

**The brief required this explicitly for Problem 1** ("you must not
auto-assemble A from geometry in Problem 1; enter the coefficients you
derived by hand") so this wasn't really a free choice — but it's worth
explaining why that constraint exists and what it cost/bought me.

- **Auto-assembly from node coordinates** (looping over members, computing
  direction cosines, building A programmatically) is what I'd do in any
  real downstream tool — it's what I actually built for Problem 2 onward.
  It's less error-prone for a 14×14 system and scales to bigger trusses.
- **Manual derivation** (what Problem 1 required) forces you to actually
  write out ΣFx = 0 / ΣFy = 0 at every one of the 7 joints by hand,
  by inspection of the free-body diagram, and transcribe the coefficients
  into MATLAB directly.

I did the manual version for Problem 1 as required (see
[`calculations/static-equilibrium-derivation.md`](../calculations/static-equilibrium-derivation.md)
for the full working), then immediately rebuilt the same system as a
parameterised function (`solveproblem2.m`) for Problem 2 onward, once the
hand-derivation had been checked against global equilibrium. This is
basically the standard "validate by hand on a simple case, then
generalise" pattern you'd want before trusting an auto-assembled model on
a bigger structure.

## 2. Direct solve vs. matrix inversion

MATLAB lets you solve `Ax = b` either as `x = A\b` (the backslash /
"left divide" operator, using LU decomposition under the hood) or as
`x = inv(A)*b`. The brief explicitly required `A\b` and forbade `inv(A)`.
This is the right call regardless of what the brief said: explicitly
inverting a matrix is slower and less numerically stable than solving the
system directly, and for a structure like this (well-conditioned,
14 equations, 14 unknowns, statically determinate) there's no reason to
pay that cost. I used `A\b` throughout, including inside the
10,000-iteration sweep in Problem 2, where the performance difference
would actually start to matter.

## 3. Problem 2 sweep: brute-force grid vs. smarter search

To find the worst-case loading across the P1–P3 design space, the brief
specified a 100×100 nested-loop sweep (10,000 combinations) using
`linspace`. The alternative would be something like an optimisation
routine (`fminsearch`/`fmincon`) hunting directly for the worst-case
member force. I stuck with the brute-force grid because:

- It's exhaustive over the specified range, so there's no risk of an
  optimiser converging to a local optimum and missing the true worst case
  (compression and tension worst-cases don't necessarily occur at the same
  corner of the load space, and a linear truss problem like this one is
  well-behaved enough that the full grid is cheap to compute anyway).
- It naturally gives you the full Factor-of-Safety **surface**
  (`FoS_results`, 100×100) for free, which is what Part B's contour plots
  actually need — an optimiser would only have given me the single worst
  point, not the surface.

The cost is that it doesn't generalise well — a finer mesh or a bigger
structure would need actual root-finding/optimisation rather than a grid.
For this assignment's scope, the grid was the right level of complexity.

## 4. Problem 3 calibration: force-based vs. area-based (I did both, kept area-based)

The brief offered two calibration routes:

- **Option 1 (pass marks): force-based.** Fit a straight line to the
  noisy measured *force* data directly, and compare its slope to the
  model's slope to get a dimensionless calibration factor `k`.
- **Option 2 (higher marks): area-based.** Fit the noisy *strain* data
  instead, and use Hooke's law (`ε = F/EA`) to back-solve for a calibrated
  cross-sectional area of the diagonal member.

I went with the area-based route. The force-based option gives you a
correction factor with no obvious physical meaning beyond "scale the
answer by this much" — it absorbs whatever is wrong (geometry, material,
gauge calibration, etc.) into one number. The area-based route ties the
mismatch to a specific, physically interpretable parameter (the
diagonal's actual cross-section vs. its nominal spec), which is also the
quantity I actually needed for the Problem 3b stress/root-finding step
(`σ = F / A_calibrated`). It does cost an extra assumption — that *all*
of the mismatch between the nominal model and the measured data is due to
area being wrong, and none of it is due to, say, the modulus being
different — which I flagge explicitly in the
[engineering interpretation](../calculations/force-table.md).

## 5. Root-finding method: secant vs. bisection vs. Newton-Raphson

For Problem 3b (finding the load P2 at which member F8 reaches the
allowable stress), the brief left the method open. Options considered:

- **Bisection** — guaranteed to converge if you bracket the root, but
  slow, and more importantly I don't actually have a bracket: the brief
  states explicitly that the critical load is *beyond* the 8–17 kN
  measured data range, so I'm extrapolating, not interpolating. Bisection
  needs two points that bracket the answer, which I don't have ahead of
  time.
- **Newton-Raphson** — fast, but needs a derivative of stress with
  respect to load, which I'd have to compute analytically or by finite
  difference anyway.
- **Secant method** — only needs two starting guesses (no bracket, no
  derivative), which fits the extrapolation problem exactly. I started
  from P2 = 17 and 18 kN — just past the edge of the measured data — and
  let it extrapolate outward.

Because stress is *exactly linear* in P2 for this statically determinate
truss (F8 scales linearly with P2, and stress is F8 divided by a constant
area), the secant method's linear-interpolation step is actually exact
here, not approximate — it converges in 2 iterations. That's a real
result, not a coincidence, and I noted it in the calculations writeup
rather than presenting it as if secant "happened" to be fast.

## 6. Problem 4 strain energy: why integrate numerically when you don't have to

For a member under constant axial force, strain energy has a closed-form
expression (`F²L / 2EA`) — no integration needed. The brief asks you to
treat it as an integral anyway and solve it with the trapezoidal rule,
explicitly so the method generalises to cases with non-constant force or
varying cross-section (which this truss doesn't have, but a more
realistic structure might). I implemented both the exact and numerical
versions side by side specifically so the comparison would be
meaningful — see the [engineering interpretation](../calculations/force-table.md)
for why the numerical error came out as ~0% (it's not a coincidence, and
it's not free lunch — it's the trapezoidal rule integrating a constant
function exactly, which is a different thing from "this code is accurate").

## 7. Problem 5: state-space ODE solve via `ode45` vs. manual time-marching

For the dynamic response, the obvious manual alternative to `ode45` is a
hand-rolled time-marching scheme (e.g. central difference / Newmark-beta,
which is what's typically used in structural dynamics FE codes). I used
`ode45` (adaptive-step Runge-Kutta) because the brief specifically asked
for it and because, for a single-DOF linear oscillator with no
discontinuities in the forcing function, there's no real benefit to a
custom integrator — `ode45`'s adaptive step sizing handles the stiff
transient at the start (where the sub-resonant 6 Hz forcing has to settle
out from rest) more efficiently than a fixed-step scheme would, without
my having to hand-tune a step size.

## What I'd reconsider with more time

- The Problem 2 design-space sweep stores the **entire** 14-element
  solution vector for every one of the 10,000 combinations
  (`all_forces`, a 100×100×14 array) even though only the max
  tension/compression per combination actually gets used downstream. That
  was a deliberate "keep everything for later lookup" choice per the
  brief's own instruction, but for a bigger structure or a finer mesh it's
  the kind of thing I'd revisit — store only the summary statistics unless
  you specifically need full traceability back to individual member
  forces.
- Problem 4's stiffness extrapolation (`keq` from Castigliano via finite
  difference) is only as good as the ΔP step size used in the central
  difference (I used 0.1 kN, per the brief's recommended 0.1–0.5 kN
  range) — for a genuinely nonlinear structure this would need a proper
  convergence check on ΔP, not just a fixed choice within the recommended
  band.
