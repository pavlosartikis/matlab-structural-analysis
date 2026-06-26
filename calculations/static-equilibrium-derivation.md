# Static Equilibrium Derivation (Problem 1)

Full hand-derivation of the joint equilibrium equations for the Warren
truss, transcribed from my handwritten working
([scan 1](../images/hand-calc-fbd-and-node-equations.png),
[scan 2](../images/hand-calc-matrix-assembly.png)).

## Geometry and candidate loads

Candidate number: **295536** → digits `d1 d2 d3 d4 d5 d6` = `2 9 5 5 3 6`

$$P_1 = 8 + d_4 = 8 + 5 = 13\ \text{kN}$$
$$P_2 = 8 + d_5 = 8 + 3 = 11\ \text{kN}$$
$$P_3 = 8 + d_6 = 8 + 6 = 14\ \text{kN}$$

All three loads fall inside the 8–17 kN sanity-check range given in the
brief.

### Diagonal member angle

Bottom-chord bay width is 3 m, top chord sits at height 3 m, offset
horizontally by 1.5 m — so every diagonal member spans a horizontal
distance of 1.5 m and a vertical distance of 3 m:

$$\tan\alpha = \frac{3}{1.5} = 2 \quad\Rightarrow\quad \alpha = \tan^{-1}(2) \approx 63.43^\circ$$

$$c = \cos\alpha, \qquad s = \sin\alpha$$

This single angle applies to *every* diagonal member (F6, F7, F8, F9,
F10, F11) since all six bays are geometrically identical.

## Sign convention and unknown ordering

- Tension positive, compression negative.
- Unknown vector (14 unknowns, matching the brief's recommended order):

$$\mathbf{x} = \begin{bmatrix} R_{x1} & R_{y1} & R_{y4} & F_1 & F_2 & F_3 & F_4 & F_5 & F_6 & F_7 & F_8 & F_9 & F_{10} & F_{11} \end{bmatrix}^\top$$

## Joint-by-joint equilibrium

Working through ΣFx = 0 and ΣFy = 0 at each of the 7 joints, taking
member forces as tension-positive and resolving diagonals through the
angle α found above:

**Node 1** (pin support, members F1 and F6 meet here):

$$x:\quad R_{x1} + F_1 + cF_6 = 0$$
$$y:\quad R_{y1} + sF_6 = 0$$

**Node 2** (bottom chord, members F1, F2, F7, F8):

$$x:\quad -F_1 - F_7c + F_2 + F_8c = 0$$
$$y:\quad F_7s + F_8s = 0$$

**Node 3** (bottom chord, members F2, F3, F9, F10):

$$x:\quad -F_2 - F_9c + F_3 + F_{10}c = 0$$
$$y:\quad F_9s + F_{10}s = 0$$

**Node 4** (roller support, members F3 and F11 meet here):

$$x:\quad -F_3 - F_{11}c = 0$$
$$y:\quad R_{y4} + F_{11}s = 0$$

**Node 5** (top chord, loaded with P1, members F4, F6, F7):

$$x:\quad F_4 + F_7c - F_6c = 0$$
$$y:\quad -F_7s - F_6s = P_1$$

**Node 6** (top chord, loaded with P2, members F4, F5, F8, F9):

$$x:\quad -F_4 + F_5 - F_8c + F_9c = 0$$
$$y:\quad -F_8s - F_9s = P_2$$

**Node 7** (top chord, loaded with P3, members F5, F10, F11):

$$x:\quad -F_5 + F_{11}c - F_{10}c = 0$$
$$y:\quad F_{10}s - F_{11}s = P_3$$

That's 14 equations (2 per joint × 7 joints) for 14 unknowns
(3 reactions + 11 member forces) — exactly determinate, as expected for a
simply-supported, pin-jointed, single-bay-pattern Warren truss with no
redundant members.

## Assembled matrix system

Substituting the angle (c = cos α, s = sin α) and the load values into the
14 equations above, ordered [Fx@N1, Fy@N1, Fx@N2, Fy@N2, ..., Fx@N7, Fy@N7]
down the rows and [Rx1, Ry1, Ry4, F1, ..., F11] across the columns of A:

$$
\mathbf{A} =
\begin{bmatrix}
1 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & c & 0 & 0 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & s & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 & -c & c & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & s & s & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 & 0 & 0 & 0 & -c & c & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & s & s & 0 \\
0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -c \\
0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & s \\
0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & -c & c & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -s & -s & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & -1 & 1 & 0 & 0 & -c & c & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -s & -s & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & -1 & 0 & 0 & 0 & 0 & -c & c \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & -s & -s
\end{bmatrix}
,\qquad
\mathbf{b} =
\begin{bmatrix} 0\\0\\0\\0\\0\\0\\0\\0\\0\\13\\0\\11\\0\\14 \end{bmatrix} \text{kN}
$$

Solved directly in MATLAB with `x = A\b` (no matrix inversion, per the
brief's instruction) — full code in
[`Problem1.m`](../report/full-report.pdf) (reproduced in the report
appendix).

## Results

| Quantity | Value | Type |
|---|---|---|
| Rx1 | 0 kN | Reaction |
| Ry1 | 18.667 kN | Reaction |
| Ry4 | 19.333 kN | Reaction |
| F1 | +9.333 kN | Tension |
| F2 | +15.000 kN | Tension |
| F3 | +9.667 kN | Tension |
| F4 | −12.167 kN | Compression |
| F5 | −12.333 kN | Compression |
| F6 | −20.870 kN | Compression |
| F7 | +6.336 kN | Tension |
| F8 | −6.336 kN | Compression |
| F9 | −5.963 kN | Compression |
| F10 | +5.963 kN | Tension |
| F11 | −21.615 kN | Compression |

**Most highly loaded member: F11, −21.62 kN (compression)** — the
diagonal nearest the roller support. F2 carries the largest tension, at
+15.00 kN.

## Global equilibrium checks (all passed)

$$\sum F_y = 0: \quad R_{y1} + R_{y4} = 18.667 + 19.333 = 38.0\ \text{kN} = P_1+P_2+P_3 \ ✓$$
$$\sum F_x = 0: \quad R_{x1} \approx 0 \ ✓$$
$$\text{Moment about N1:}\quad R_{y4}\cdot 9 - (13\times 1.5 + 11\times 4.5 + 14\times 7.5) \approx 0 \ ✓$$

## Why F11 governs, not a symmetric member

The load case isn't symmetric (P1 = 13, P2 = 11, P3 = 14 — P3 is the
largest, sitting nearest the roller end), so the truss doesn't load up
symmetrically either. F11 has to carry the full shear transferred into
the bay adjacent to the roller support, where the heaviest individual
load (P3 = 14 kN) is closest to the support reaction it has to resolve
against — which is why it ends up the most heavily loaded member rather
than, say, F6 at the pin end.
