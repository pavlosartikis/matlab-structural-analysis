# Project Brief

**Module:** H7137 — Numerical Modelling & Engineering Simulations
**Institution:** University of Sussex
**Year:** 2025–26
**Format:** Individual portfolio, 5 linked problems, parameterised per-candidate

## Scenario

The brief frames the whole portfolio around a single structure: a simply
supported **Warren truss** used as an equipment gantry (the kind of thing
you'd see suspending services or rail equipment above a walkway — the brief
uses a monorail support truss in Chiba, Japan as the real-world reference).

Each of the five problems builds on the previous one. The stated intent
(quoting the brief) was to move the work "from simple *solvers* to
*modellers and analysts*" — i.e. don't just get a number out once, build
something that can be re-run, swept, calibrated, and extended. You're not
expected to derive new structural theory; the truss analysis itself
(method of joints, pin-jointed, statically determinate) is assumed
knowledge. The actual assessment target is **problem formulation and
numerical implementation**.

## Structure of the truss

7 joints, 11 members, pin support at N1, roller support at N4, three
downward point loads (P1, P2, P3) applied at the top-chord joints N5, N6, N7.

| Joint | Label | X (m) | Y (m) |
|-------|-------|-------|-------|
| 1 | N1 | 0.0 | 0 |
| 2 | N2 | 3.0 | 0 |
| 3 | N3 | 6.0 | 0 |
| 4 | N4 | 9.0 | 0 |
| 5 | N5 | 1.5 | 3 |
| 6 | N6 | 4.5 | 3 |
| 7 | N7 | 7.5 | 3 |

Loads and member areas were **parameterised per candidate number** — every
student in the cohort solved a structurally identical truss but with
different load magnitudes, so answers can't just be copied between
students. My candidate number is **295536** (`d1 d2 d3 d4 d5 d6`), which
sets:

- P1 = 8 + d4 = 8 + 5 = **13 kN**
- P2 = 8 + d5 = 8 + 3 = **11 kN**
- P3 = 8 + d6 = 8 + 6 = **14 kN**

## The five problems, as briefed

1. **Static Truss Solver** — derive the 14 joint-equilibrium equations by
   hand, assemble them manually into a 14×14 matrix system **Ax = b**
   (no auto-assembly from geometry allowed at this stage), solve with
   `A\b`, and sanity-check against global equilibrium.
2. **Parametric Design Tool** — turn the Problem 1 solver into a reusable
   MATLAB function, then sweep P1 and P3 each across 100 values (0 → 200%
   of their Problem 1 value, via `linspace`) in nested loops — 10,000 load
   combinations total — and find the worst-case tension/compression and
   Factor of Safety (allowable: 60 kN compression / 40 kN tension) across
   the whole design space.
3. **Calibrated Structural Model** — given synthetic "experimental" noisy
   strain/force data for a physical scale model (unique per candidate),
   fit a regression line, use it to back out a calibrated cross-sectional
   area for the diagonal member (F8), then use root-finding to extrapolate
   the load at which that member reaches an allowable stress of 120 MPa.
4. **Strain Energy** — compute strain energy per member from the Problem 1
   forces, using the trapezoidal rule over n = 50 points per member (even
   though the integrand is constant and the exact form needs no
   integration — the point is to exercise the numerical method), then use
   Castigliano's second theorem via finite differences to estimate the
   displacement at the loaded joint N6, and from that an equivalent global
   stiffness.
5. **Dynamic Response Simulator** — treat the truss as an equivalent
   single-degree-of-freedom mass-spring-damper system, convert the 2nd
   order ODE to a first-order state-space system, and solve it numerically
   with `ode45` under a sinusoidal forcing load, for 5 seconds.

## AI use policy (as stated in the brief)

> "AI is permitted where it accelerates implementation, testing, or
> exploration, but not where it replaces modelling judgement or numerical
> reasoning."

In practice this meant: AI tools were fine for things like "how do I
overlay a specific contour line in MATLAB" or "how do I read `ode45`
output at fixed time steps", but not for deriving the equilibrium
equations or making engineering judgement calls. The brief explicitly
required this to be disclosed — see the AI Usage Disclosure appendix in
the [full report](../report/full-report.pdf).

## A note on the source documents

The five problem-brief slide decks supplied for this assignment use
slightly inconsistent problem numbering against each other and against
the submitted report — e.g. the calibration/root-finding task is labelled
differently between the brief deck and the final report's own table of
contents. This summary follows the **final report's** numbering (1–5 as
listed above), since that's what was actually submitted and assessed.
