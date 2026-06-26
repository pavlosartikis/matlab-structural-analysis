# Consolidated Results Table

All final numeric results from the portfolio, in one place, for quick
reference. Full working for each is in the correspondingly-named file in
this folder.

## Problem 1 — Static member forces (baseline load case)

Candidate 295536 loads: P1 = 13 kN, P2 = 11 kN, P3 = 14 kN.

| Member | Force (kN) | Type |
|---|---|---|
| Rx1 (reaction) | 0 | — |
| Ry1 (reaction) | 18.667 | — |
| Ry4 (reaction) | 19.333 | — |
| F1 | +9.333 | Tension |
| F2 | +15.000 | Tension |
| F3 | +9.667 | Tension |
| F4 | −12.167 | Compression |
| F5 | −12.333 | Compression |
| F6 | −20.870 | Compression |
| F7 | +6.336 | Tension |
| F8 | −6.336 | Compression |
| F9 | −5.963 | Compression |
| F10 | +5.963 | Tension |
| **F11** | **−21.615** | **Compression — most highly loaded** |

Global checks: ΣFy = 38.0 kN ✓, ΣFx ≈ 0 ✓, moment about N1 ≈ 0 ✓.

## Problem 2 — Design space (10,000-combination sweep)

P1 swept 0→39 kN, P3 swept 0→42 kN (each +200% of baseline, 100 points),
P2 fixed at 11 kN.

| Quantity | Value |
|---|---|
| Combinations tested | 10,000 |
| Combinations failed (FoS < 1) | 0 |
| **Minimum FoS found** | **1.142** |
| Location of minimum | P1 = 39 kN, P3 = 42 kN |
| Governing member | F11, −52.5 kN (compression) |

## Problem 3 — Calibration & root-finding (member F8)

| Quantity | Value |
|---|---|
| Nominal diagonal area | 660.00 mm² |
| **Calibrated diagonal area** | **894.64 mm²** (+35.5%) |
| Regression fit (ε = cP₂ + d) | c = 3.1243, d = 2.4434 |
| Root-finding method | Secant (2 iterations to converge) |
| **Critical load P2\*** | **191.71 kN** (stress = 120 MPa) |
| Recommended service limit (P2\*/2) | ≈96 kN |

## Problem 4 — Strain energy & equivalent stiffness

| Quantity | Value |
|---|---|
| Total strain energy U (exact) | 0.019856 kN·m |
| Total strain energy U (numerical, n=50 trapezoidal) | 0.019856 kN·m |
| Numerical vs exact error | ~0% (constant integrand, see note below) |
| Displacement at N6 (Castigliano, finite difference) | 1.4712 mm |
| Equivalent vertical stiffness $k_{eq}$ | 18,346.68 kN/m |

## Problem 5 — Dynamic response

| Quantity | Value |
|---|---|
| Mass, m | 1200 kg |
| Stiffness, k | 3.84×10⁶ N/m |
| Damping ratio, ζ | 0.10 |
| Natural frequency, $f_n$ | 9.0032 Hz |
| Forcing frequency, f | 6.0 Hz |
| **Peak displacement** | **5.3179 mm** at t = 0.2099 s |
| Steady-state amplitude | 4.5557 mm |
| Static deflection ($P_0/k$) | 2.6042 mm |
| Dynamic Amplification Factor | 1.7494 |
| Time to steady-state (≈5τ) | ≈0.9 s |

## Cross-checks and things worth flagging together

These are the numbers from across different problems that look like they
*should* match, do or don't, and why:

- **Problem 4's static N6 displacement (1.47 mm)** and **Problem 5's
  static deflection P0/k (2.60 mm)** are *not* the same calculation and
  are not expected to agree — Problem 4 uses the original P1/P2/P3 static
  load case with a Castigliano-derived stiffness; Problem 5 uses a
  separately-parameterised dynamic stiffness (k = 3.84 MN/m, from the
  brief's own candidate-digit formula) and a different forcing amplitude
  (P0 = 10 kN sinusoidal vs. the static P2 = 11 kN). Both are
  low-single-digit-millimetre displacements for a stiff steel truss,
  which is the qualitative consistency check that matters, not an exact
  numeric match.
- **Problem 1's F11 (−21.615 kN)** at the baseline load case and
  **Problem 2's F11 (−52.5 kN)** at the worst-case swept corner are
  consistent with each other — F11 is the governing member in *both*
  problems, which makes sense since Problem 2 is a superset of Problem
  1's load case (P1 and P3 scaled up, P2 unchanged), so whatever member
  governed at baseline was always likely to keep governing as load
  increases, absent some other member's force growing disproportionately
  faster.
- **Problem 4's near-zero numerical integration error** is real but
  doesn't mean what it might look like at a glance — see the dedicated
  caveat in [strain-energy.md](strain-energy.md#engineering-interpretation):
  the trapezoidal rule integrates a constant function exactly regardless
  of the number of points used, so this result confirms the code is
  correct, not that the numerical method is being meaningfully
  stress-tested.
