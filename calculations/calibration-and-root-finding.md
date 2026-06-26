# Calibration & Root-Finding (Problem 3)

Working for fitting the truss model to noisy "experimental" data from a
scale model, and using that calibrated model to find the truss's true
load capacity.

## Data

Candidate-specific synthetic dataset (`synthetic_truss_295536.mat`),
covering member F8 (one of the diagonals) under a swept load P2 from
8–17 kN, 901 data points, with two parallel measurement channels:

- `F_measured_kN` — noisy + biased force measurement
- `epsilon_measured_microstrain` — noisy strain gauge measurement

P1 and P3 are held fixed at their Problem 1 values (13 kN and 14 kN).

## Part A: Area-based calibration

I used the **area-based** calibration route (Option 2 in the brief, the
higher-marks option) rather than the simpler force-based scale-factor
route — see [concept-design.md](../docs/concept-design.md#4-problem-3-calibration-force-based-vs-area-based-i-did-both-kept-area-based)
for why.

Fit a straight line to the measured strain data against load:

$$\varepsilon\,[\mu\varepsilon] = cP_2 + d$$

via `polyfit`. Result:

$$c = 3.1243, \qquad d = 2.4434$$

Compare against the nominal model relation $\varepsilon = F_{\text{model}}/(EA'_{\text{diag}})$
with E = 200 GPa, and solve for the calibrated diagonal area:

$$A_{\text{diag}}^{\text{calibrated}} = \frac{F_{\text{model}} \times 10^6}{200\,(cP_2+d)}\ \text{mm}^2$$

| Quantity | Value |
|---|---|
| Nominal area (from candidate data) | 660.00 mm² |
| **Calibrated area** | **894.64 mm²** |
| Regression: ε = cP₂ + d | c = 3.1243, d = 2.4434 |

The calibrated area is **35.5% larger** than the nominal spec value. That
is a substantial mismatch, not a rounding-level correction — see the
interpretation below for what that means and what I can and can't
conclude from it.

## Part B: Root-finding for maximum safe load

Define the stress in member F8 as a function of P2, using the calibrated
area:

$$\sigma(P_2) = \frac{F_8(P_2)}{A_{\text{calibrated}}}$$

and solve for the load at which it reaches the allowable stress limit:

$$\sigma(P_2^*) = \sigma_{\text{allow}} = 120\ \text{MPa}$$

**Method: secant.** Starting points P2 = 17 kN and 18 kN (just past the
edge of the measured 8–17 kN data range, since the brief confirms the
critical load lies beyond the measured range and has to be extrapolated,
not interpolated). The truss solver function from Problem 2 was reused
directly inside the root-finding loop to get F8 at each trial load.

### Iteration table

| Iter | P2 (kN) | F8 (kN) | Stress (MPa) | Error (MPa) |
|---|---|---|---|---|
| 1 | 18.0000 | 10.2486 | 11.4557 | −108.5443 |
| 2 | 191.7115 | 107.3563 | 120.0000 | 0.0000 |

**Converged in 2 iterations.** This isn't a coincidence or an unusually
lucky starting guess — for this structure, member force scales *exactly*
linearly with P2 (statically determinate truss, linear-elastic), so
stress is an exactly linear function of load, and the secant method's
linear-interpolation update is exact after a single step rather than
approximate.

### Result

$$\boxed{P_2^{*} = 191.71\ \text{kN}}$$

## Engineering interpretation

**What drove the mismatch?** Two separate things, and they're not
separable from this data alone:

1. **Random measurement noise** — visible as the scatter of points around
   the regression line in
   [the strain regression plot](../images/problem3-strain-regression-fit.png).
   This noise doesn't bias the result on average, it just adds
   uncertainty to the fitted slope.
2. **A real, systematic offset** between the as-built diagonal member and
   its 660 mm² nominal spec — picked up by the regression slope itself,
   not the scatter around it. The fitted strain is consistently *below*
   the nominal model curve (visible in the plot: the blue dashed nominal
   curve sits above the red fitted line across the whole load range),
   meaning the real member strains *less* than the nominal model predicts
   for the same load — consistent with a stiffer, larger actual
   cross-section.

I can't fully separate these two effects from a single regression fit —
if the "true" area were exactly 660 mm² and all of the offset were
measurement bias in the strain gauge calibration instead, I'd get the
same fitted line. **This is a real limitation of the calibration, not
just a footnote**: I am assuming all of the systematic offset is due to
area, and reporting it as if it's a structural property, when some
unknown fraction of it could equally be due to instrumentation bias I
have no way to isolate from a single test campaign.

**Does calibration increase or decrease predicted capacity?** It
increases it. A calibrated area of 894.6 mm² (vs. 660 mm² nominal) means
the diagonal is both stiffer and stronger than the original design
assumed, so the calibrated model predicts a *higher* safe load than the
nominal model would have.

**What load limit should be recommended?** The root-finding result
(P2* = 191.71 kN) is the load at which stress reaches the 120 MPa limit —
not a safe operating load, a failure threshold. Applying a typical
engineering factor of safety of 2 on top of that gives a recommended
service limit of:

$$P_2 \lesssim \frac{191.71}{2} \approx 96\ \text{kN}$$

I'd flag that 96 kN is roughly **9× the original Problem 1 baseline load**
of 11 kN at this joint, which is a big enough jump from the brief's
original scenario that I would not rely on this extrapolated number for
anything safety-critical without first re-validating the linear-elastic
assumption at that load level — the calibration was built from data
covering 8–17 kN, and I'm extrapolating more than 10× past the edge of
that range to get the root, then halving it for the recommended limit.
Linear extrapolation that far past your calibration data is a real risk
in itself, independent of the area-vs-bias ambiguity above.
