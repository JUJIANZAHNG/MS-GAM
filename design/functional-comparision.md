# Chapter 5: Regime-Specific Functional Geometry

Updated: 2026-06-28

This file controls the comparison chapter after the move to separate destination-level MS-GAM estimation.

Important convention:

> The comparison object is no longer a single destination function. It is the regime-specific function family \(\mathcal{F}_r^{(k)}\).

## 1. Core comparison object

For destination \(r\) and regime \(k\), define:

[
\mathcal{F}_r^{(k)}
=
\left(
f_{r,k}^{temp},
f_{r,k}^{rain},
te_{r,k},
f_{r,k}^{ext}
\right)
]

where:

- \(f_{r,k}^{temp}\): marginal temperature-response function
- \(f_{r,k}^{rain}\): marginal rainfall-response function
- \(te_{r,k}\): temperature-rainfall interaction surface
- \(f_{r,k}^{ext}\): explicit extreme-weather response

Chapter 5 therefore compares regime-specific climate-response structures, not just destination-level regression curves.

## 2. Regime labels are local, not global

Because destinations are estimated separately, regime labels are local to each destination:

[
S_{r,t}\in\{1,\dots,K_r\}
]

So "regime 1" in one destination should not be compared mechanically with "regime 1" in another destination.

Cross-destination comparison requires a post-estimation alignment rule. Two practical routes are acceptable:

| Alignment rule | Meaning | Use |
|---|---|---|
| within-destination severity ordering | order regimes by fitted mean level, climate sensitivity, or stress intensity | simple baseline alignment |
| feature-based matching | match regimes by geometric summaries or distance minimization | stronger cross-system comparison |

The chapter should say this explicitly. Otherwise cross-destination regime comparison looks cleaner than it really is.

## 3. Common comparison domain

Before comparing functions across destinations, climate inputs must be mapped to a common scale.

Two usable options:

| Option | Mapping | Interpretation |
|---|---|---|
| anomaly scale | z-score or centered anomaly | compare responses relative to local climate norm |
| percentile scale | \(x \mapsto F_X(x)\) | compare shapes over relative climate positions |

The point is simple:

> comparison should target shape and geometry, not raw local-unit magnitude differences.

## 4. Within-destination regime contrast

The first question of Chapter 5 is whether response functions differ across regimes within the same destination.

Define:

[
d_r^{within}(k,\ell)
=
d\left(\mathcal{F}_r^{(k)},\mathcal{F}_r^{(\ell)}\right)
]

This is the cleanest answer to:

> Do latent regimes correspond to genuinely different climate-response structures?

If within-destination regime distances are small, the regime layer is weak. If they are large, the MS-GAM is picking up meaningful structural shifts.

## 5. Cross-destination regime contrast

After regime alignment, compare matched regime functions across destinations:

[
d_{ij}^{cross}(k,\ell)
=
d\left(\mathcal{F}_i^{(k)},\mathcal{F}_j^{(\ell)}\right)
]

The chapter then asks:

1. Are high-sensitivity regimes geometrically similar across destinations?
2. Are some destinations regime-fragile while others are regime-stable?
3. Does interaction geometry strengthen in comparable high-risk states?

This makes Chapter 5 a comparison of regime-conditioned systems rather than a flat destination ranking.

## 6. Geometry summaries

The comparison chapter needs a compact feature map for each regime-specific function family.

Define:

[
\Phi_r^{(k)}
=
\left(
C_{r,k}^{temp},
C_{r,k}^{rain},
G_{r,k}^{int},
A_{r,k}^{ext}
\right)
]

where the components can be defined as:

[
C_{r,k}^{temp}
=
\int
\left|
\frac{\partial^2 f_{r,k}^{temp}(T)}{\partial T^2}
\right|dT
]

[
C_{r,k}^{rain}
=
\int
\left|
\frac{\partial^2 f_{r,k}^{rain}(R)}{\partial R^2}
\right|dR
]

[
G_{r,k}^{int}
=
\iint
\left|
\frac{\partial^2 te_{r,k}(T,R)}{\partial T\,\partial R}
\right|dT\,dR
]

[
A_{r,k}^{ext}
=
\int
\left|
f_{r,k}^{ext}(E)
\right|dE
]

Interpretation:

| Feature | What it measures |
|---|---|
| \(C^{temp}\), \(C^{rain}\) | curvature / nonlinearity of marginal climate effects |
| \(G^{int}\) | strength of temperature-rainfall coupling |
| \(A^{ext}\) | salience of explicit extreme-weather exposure |

These summaries are not substitutes for the functions themselves. They are compact geometric projections used for distance calculation, clustering, and visualization.

## 7. Distance design

The comparison layer needs a distance metric that respects the multi-component structure of \(\mathcal{F}_r^{(k)}\).

One practical design is:

[
d\left(\mathcal{F}_i^{(k)},\mathcal{F}_j^{(\ell)}\right)
=
w_T d_T
+
w_R d_R
+
w_{TR} d_{TR}
+
w_E d_E
]

where:

- \(d_T\): distance between temperature-response functions
- \(d_R\): distance between rainfall-response functions
- \(d_{TR}\): distance between interaction surfaces
- \(d_E\): distance between extreme-weather components

The weights should be fixed ex ante or normalized to keep one component from dominating mechanically.

## 8. Output forms of Chapter 5

The chapter can produce three levels of comparison output:

| Output | Object | Purpose |
|---|---|---|
| regime-difference plots | \(\mathcal{F}_r^{(k)}\) within one destination | show that regimes matter |
| distance tables / heatmaps | \(d(\mathcal{F}_i^{(k)},\mathcal{F}_j^{(\ell)})\) | compare systems |
| clustering / embedding | \(\Phi_r^{(k)}\) or distance matrix | build typology of regime-conditioned tourism systems |

## 9. What Chapter 5 should not do

To keep the method shift clean:

- do not fall back to coefficient-style heterogeneity language;
- do not compare destination-level functions while ignoring regime index \(k\);
- do not treat resilience as a loose metaphor here when Chapter 7 is responsible for structural stability tests.

Chapter 5 is the geometry chapter. Stability and invariance belong to Chapter 7.

## 10. One-line closure

Chapter 5 can be summarized as:

[
\boxed{
\hat{\mathcal{F}}_r^{(k)}
\Rightarrow
\Phi_r^{(k)}
\Rightarrow
d\left(\mathcal{F}_i^{(k)},\mathcal{F}_j^{(\ell)}\right)
}
]

This is the chapter where regime-specific response functions become measurable geometric objects.
