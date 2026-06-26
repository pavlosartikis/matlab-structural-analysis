# Dynamic Response Simulation (Problem 5)

Working for converting the truss's vertical dynamic response into a
state-space system and solving it numerically with `ode45`.

## Governing equation

The vertical dynamic response at node N6 is approximated as a
single-degree-of-freedom mass-spring-damper system:

$$m\ddot y(t) + c\dot y(t) + ky(t) = F(t)$$

## Converting to first-order state-space form

Let $x_1 = y$, $x_2 = \dot y$. Then $\dot x_1 = x_2$, and substituting into
the governing equation and solving for $\ddot y$:

$$\dot x_2 = \ddot y = \frac{1}{m}\big(F(t) - cx_2 - kx_1\big)$$

giving the first-order system actually solved by `ode45`:

$$\dot x_1 = x_2, \qquad \dot x_2 = \frac{F(t) - cx_2 - kx_1}{m}$$

In matrix form:

$$\frac{d}{dt}\begin{bmatrix}x_1\\x_2\end{bmatrix} = \begin{bmatrix}0 & 1\\ -k/m & -c/m\end{bmatrix}\begin{bmatrix}x_1\\x_2\end{bmatrix} + \begin{bmatrix}0\\F(t)/m\end{bmatrix}$$

with initial conditions $x_1(0) = y(0) = 0$, $x_2(0) = \dot y(0) = 0$
(starts from rest). Full hand-derivation of this conversion is in the
[full report](../report/full-report.pdf), Problem 5 Part A.

## Parameters for candidate 295536

Candidate digits: d4 = 5, d5 = 3, d6 = 6. Base values m₀ = 1200 kg,
k₀ = 4×10⁶ N/m, ζ₀ = 0.05:

| Parameter | Formula | Value |
|---|---|---|
| Mass, m | $m_0(1+0.02(d_4-5))$ | 1200 kg |
| Stiffness, k | $k_0(1+0.02(d_5-5))$ | 3.84×10⁶ N/m |
| Damping ratio, ζ | $\zeta_0 + 0.05(d_6-5)$ | 0.10 |
| Damping coefficient, c | $2\zeta\sqrt{km}$ | 13,576.45 N·s/m |
| Natural frequency, $f_n$ | $\frac{1}{2\pi}\sqrt{k/m}$ | 9.0032 Hz |
| Forcing frequency, f | (given) | 6 Hz |

Forcing function: $F(t) = P_0\sin(2\pi f t)$, with $P_0 = 10\ \text{kN}$,
run for a 5 second duration.

## Result

| Quantity | Value |
|---|---|
| Peak displacement | **5.3179 mm** at t = 0.2099 s |
| Steady-state amplitude | 4.5557 mm |
| Static deflection ($P_0/k$) | 2.6042 mm |
| Dynamic Amplification Factor | 1.7494 |
| Forcing / natural frequency ratio | 0.6664 |

Sample of the time-history output (full table in the report):

| Time (s) | Displacement (m) |
|---|---|
| 0.0 | 0.0000000 |
| 0.2 | 0.0048650 |
| 1.0 | −0.0010559 |
| 3.0 | −0.0010621 |
| 5.0 | −0.0010623 |

A quick note on reading this table together with the peak-displacement
result above, since at a glance they can look inconsistent: the **0.2 s**
row (4.865 mm) and the stated **peak** (5.3179 mm at t = 0.2099 s) are two
different points on the same oscillating curve, sampled 9.9 ms apart, not
a contradiction — the curve is still rising at t = 0.2 s and reaches its
true local maximum a fraction of a second later. The two numbers should
agree closely but not exactly, which they do.

See [`images/problem5-dynamic-response-timehistory.png`](../images/problem5-dynamic-response-timehistory.png)
for the full displacement and velocity time histories over the full 5 s
window.

## Engineering interpretation

**Sub-resonant, not resonant.** The forcing frequency (6 Hz) sits below
the natural frequency (≈9.00 Hz) — forcing/natural ratio of 0.666 — so
this is a sub-resonant condition, not resonance. The Dynamic Amplification
Factor of 1.75 confirms the steady-state response (4.56 mm) is amplified
relative to the static deflection (2.60 mm) by roughly that factor, which
is the expected behaviour for a frequency ratio below 1 with light
damping (ζ = 0.10) — well below the theoretical resonance peak of
$1/(2\zeta) = 5$ that would occur if the forcing frequency matched the
natural frequency exactly.

**Does the dynamic response match static expectations?** Yes, in the
sense that matters: the steady-state amplitude (4.56 mm) is the same
order of magnitude as the static deflection (2.60 mm) and the Problem 4
Castigliano-derived displacement (1.47 mm) — all three are in the
low-single-digit-millimetre range for this truss, which is the
consistency check the brief asks for. They are not expected to be
numerically identical, since they come from different load cases and
different models (Problem 4's 1.47 mm is the static deflection under the
original P1/P2/P3 load case using the strain-energy stiffness; Problem
5's 2.60 mm static deflection is $P_0/k$ using the independently
parameterised dynamic stiffness k = 3.84 MN/m) — see the note in
[strain-energy.md](strain-energy.md#engineering-interpretation) about why
those two stiffness values aren't meant to match.

**How long to stabilise?** The transient decays with time constant
$\tau = 1/(\zeta\omega_n) \approx 0.18$ s. Five time constants
(≈0.9 s) are needed to consider the transient effectively gone, which
matches the time-history plot — visible transient effects (the
larger first-cycle overshoot above the steady dashed peak line) are gone
by around t ≈ 1 s, after which the response settles into the steady
4.56 mm sinusoid for the remaining 4 seconds of simulation.
