# Parametric Design Space (Problem 2)

Full working for the 10,000-combination load sweep used to find the
truss's operating envelope and minimum Factor of Safety.

## Setup

The Problem 1 solver was rebuilt as a MATLAB function (`solveproblem2.m`)
accepting P1 and P3 as arguments, with P2 fixed at its Problem 1 value
(11 kN) and the same 14×14 matrix system from
[the equilibrium derivation](static-equilibrium-derivation.md) solved
inside the function on every call.

P1 and P3 are swept from 0 up to **+200% of their Problem 1 baseline**,
each as 100 linearly spaced values (`linspace`):

$$P_{1,\max} = P_1 + 2P_1 = 13 + 2(13) = 39\ \text{kN}$$
$$P_{3,\max} = P_3 + 2P_3 = 14 + 2(14) = 42\ \text{kN}$$

Nested `for` loops cycle through all $100 \times 100 = 10{,}000$
combinations of (P1, P3), calling `solveproblem2` each time and recording:

- The full 14-element solution vector (reactions + all 11 member forces)
- The maximum tension and maximum compression present in the truss for
  that specific load combination
- A Factor of Safety, against allowable limits of **40 kN tension** /
  **60 kN compression**:

$$\text{FoS} = \frac{F_{\text{allow}}}{|F_{\text{actual}}|}$$

taking the lower of the tension-side and compression-side FoS as the
governing value for that load combination.

## Result: minimum Factor of Safety across the full design space

| Quantity | Value |
|---|---|
| Combinations swept | 10,000 |
| Combinations with FoS < 1 (failure) | **0** |
| Minimum FoS found | **1.142** |
| Location of minimum FoS | P1 = 39 kN, P3 = 42 kN (the extreme corner) |
| Governing member at minimum FoS | F11 (compression) |

At the worst-case corner (P1 = 39 kN, P3 = 42 kN — both loads at their
full +200% limit simultaneously), member forces were:

| Member | Force (kN) | Type |
|---|---|---|
| F6 | −50.3 | Compression |
| F7 | +6.7 | Tension |
| F1 | +22.5 | Tension |
| F4 | −25.5 | Compression |
| F8 | −6.7 | Compression |
| F9 | −5.6 | Compression |
| F2 | +28.5 | Tension |
| F5 | −26.0 | Compression |
| F10 | +5.6 | Tension |
| F11 | −52.5 | Compression |
| F3 | +23.5 | Tension |

(see [`images/problem2-interactive-truss-tool.png`](../images/problem2-interactive-truss-tool.png)
for the annotated diagram at this exact load case)

## Where the FoS = 2 contour sits

The brief's mid-range marking criterion asks for the loading combinations
where FoS drops below 2 to be identified and flagged graphically. Rather
than just listing the failing points, I overlaid the FoS = 2 contour
directly on the tension/compression maps
(see [`images/problem2-fos-contour-maps.png`](../images/problem2-fos-contour-maps.png)):
combinations to the upper-right of that contour line (roughly P1 > 20 kN
combined with P3 > 20 kN) drop below an FoS of 2, even though every single
one of the 10,000 combinations stayed above FoS = 1 (no contour line for
FoS = 1 is visible on the plot at all, because nothing reached it).

## Engineering interpretation

The truss does not fail anywhere in the swept design space — even at
+200% of both P1 and P3 simultaneously, with P2 unchanged, the minimum FoS
is 1.142. But that number is below the 1.5–2.0 margin typically used for
this kind of structural design, so while the truss is technically safe
everywhere I tested, I would not actually recommend operating it anywhere
near that corner of the load space. A sensible service load limit would
sit well inside the FoS = 2 contour shown in the figure, governed by F11
in compression near the roller support — consistent with F11 already
being the most heavily loaded member at the baseline Problem 1 load case.
