# Strain Energy & Equivalent Stiffness (Problem 4)

Working for computing total strain energy stored in the truss under the
Problem 1 load case, and using it to estimate an equivalent stiffness at
the loaded joint N6 (needed as an input to the Problem 5 dynamic model).

## Setup

Loads: P1 = 13 kN, P2 = 11 kN, P3 = 14 kN (Problem 1 baseline).
Material: E = 200 × 10⁶ kN/m² (200 GPa).

Member areas (from the candidate data file) are **not uniform** —
different member groups use different cross-sections:

| Group | Members | Area |
|---|---|---|
| Bottom chord | 1, 2, 3 | 900 mm² |
| Top chord | 4, 5 | 730 mm² |
| Diagonals | 6, 7, 8, 9, 10, 11 | 660 mm² |

Member lengths: chord members (1–5) are all 3 m; diagonal members (6–11)
are all $\sqrt{1.5^2+3^2} \approx 3.354$ m (same geometry as the diagonal
angle derivation in Problem 1).

## Method

For each member, the strain energy integrand is constant along its
length (since axial force is constant in a pin-jointed member under
static load):

$$g_i = \frac{F_i^2}{2EA_i}$$

This was deliberately integrated **numerically** using the trapezoidal
rule with **n = 50** discretisation points per member, even though the
closed-form exact result needs no integration at all
(see [concept-design.md](../docs/concept-design.md#6-problem-4-strain-energy-why-integrate-numerically-when-you-dont-have-to)
for why):

$$U_i = \frac{\Delta x}{2}\left(g_i(x_0) + 2\sum_{k=1}^{n-1}g_i(x_k) + g_i(x_n)\right), \qquad U_{\text{total}} = \sum_{i=1}^{11}U_i$$

run alongside the exact closed-form result for comparison:

$$U_i^{\text{exact}} = \frac{F_i^2 L_i}{2EA_i}$$

## Results

| Member | U_exact (kN·m) | U_numerical (kN·m) | Error (%) |
|---|---|---|---|
| F1 | 0.00072593 | 0.00072593 | 1.5×10⁻¹² % |
| F2 | 0.001875 | 0.001875 | 1.2×10⁻¹² % |
| F3 | 0.0007787 | 0.0007787 | 4.2×10⁻¹² % |
| F4 | 0.0015208 | 0.0015208 | 1.4×10⁻¹² % |
| F5 | 0.0015628 | 0.0015628 | 1.4×10⁻¹² % |
| F6 | 0.0055337 | 0.0055337 | 1.6×10⁻¹² % |
| F7 | 0.00050996 | 0.00050996 | 2.1×10⁻¹² % |
| F8 | 0.00050996 | 0.00050996 | 0 % |
| F9 | 0.00045173 | 0.00045173 | 1.2×10⁻¹² % |
| F10 | 0.00045173 | 0.00045173 | 2.4×10⁻¹² % |
| F11 | 0.005936 | 0.005936 | 1.5×10⁻¹² % |
| **TOTAL** | **0.019856** | **0.019856** | **0 %** |

**Total strain energy U ≈ 0.019856 kN·m** (numerical and exact agree to
within floating-point precision, i.e. the "error" shown above is just
numerical noise from floating-point arithmetic, not a real discrepancy).

### Equivalent stiffness via Castigliano's second theorem

Displacement at the loaded joint N6, via finite-difference derivative of
strain energy with respect to P2:

$$\delta = \frac{\partial U}{\partial P_2} \approx \frac{U(P_2+\Delta P) - U(P_2-\Delta P)}{2\Delta P}, \qquad \Delta P = 0.1\ \text{kN}$$

$$k_{eq} = \frac{2U}{\delta^2}$$

| Quantity | Value |
|---|---|
| Displacement at N6 | **1.4712 mm** |
| Equivalent vertical stiffness $k_{eq}$ | **18,346.68 kN/m** |

## Engineering interpretation

**Is the numerical integration actually being tested here?** Not really,
and I want to be upfront about that rather than presenting the ~0% error
as if it validates something. The trapezoidal rule integrates a
**constant function exactly**, for *any* number of points n — because
$g_i(x)$ doesn't vary along the member, there is zero discretisation
error to begin with. The near-zero error confirms the code is implemented
correctly (no bugs in the trapezoidal sum, correct indexing, etc.), but
it does **not** demonstrate that trapezoidal integration is accurate in
general — that would need a genuinely varying integrand (e.g. a member
under a distributed load, or non-constant cross-section), which this
truss doesn't have. The brief's framing acknowledges this directly: the
whole point of integrating a constant function this way is to build the
*pattern* you'd need for a real FEA-style energy calculation, not to
stress-test the method here.

**Is the displacement reasonable?** Yes. 1.47 mm at the loaded joint, for
a 9 m-span steel truss carrying ~38 kN of total load, is well within
typical serviceability limits (a common rule of thumb is span/360, which
for this truss would be 25 mm — the calculated displacement is about
17× smaller than that limit). It's a stiff, lightly-loaded structure at
this baseline load case, which is consistent with the Problem 2 result
that even +200% loading kept the Factor of Safety above 1.

This equivalent stiffness ($k_{eq}$ ≈ 18,347 kN/m) is **not** the same
quantity used as the spring stiffness `k` in the Problem 5 dynamic model
— Problem 5 uses a separately parameterised stiffness (`k0` scaled by
candidate digits) rather than this Castigliano-derived value. I'm noting
that explicitly here because the two numbers could easily be confused:
this section's $k_{eq}$ is a structural-mechanics cross-check of the
static model, while Problem 5's `k` is an independent input given by the
brief's own parameterisation formula. They are not required to match,
and in this submission they don't.
