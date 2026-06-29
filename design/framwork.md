# Paper Structure Sync Note

Updated: 2026-06-28

This file is a compact chapter map aligned with the current Markov-Switching GAM framework.

Detailed control files:

- `design/framework-ms-gam.md`
- `design/ms-gam-core.md`
- `design/prediction.md`

## 1. Paper spine

The paper now follows this chain:

[
\text{latent regime process}
\rightarrow
\text{regime-specific climate-response functions}
\rightarrow
\text{functional geometry}
\rightarrow
\text{downstream forecasting}
\rightarrow
\text{structural robustness}
]

## 2. Chapter map

| Chapter | Current task | Core object |
|---|---|---|
| 1. Introduction | motivate nonlinear, regime-dependent climate-tourism system | research gap |
| 2. Conceptual Framework | define tourism demand as a regime-dependent functional system | \(f_{r,S_{r,t}}(\mathcal{M}_L(X_t))\) |
| 3. Data and System Design | define sample structure, monthly core frequency, daily support role | \(X_t=(T_t,R_t,E_t)\) |
| 4. MS-GAM Inference | jointly infer latent regimes and regime-specific smooth functions | \(S_t,\mathcal{F}^{(k)},\Pi_t\) |
| 5. Regime-Specific Functional Geometry | compare response structures across regimes and destinations | \(\mathcal{F}_r^{(k)}\) |
| 6. Prediction under Regime and Extreme Climate Structure | test forecasting usefulness of climate, extreme, and regime information | \(M_1\) to \(M_4\), \(Z_t\) |
| 7. Structural Robustness | test stability of regimes, functions, and prediction gains | \(\hat S_t,\hat{\mathcal F}^{(k)},Gain_{structure}\) |
| 8. Discussion | interpret tourism systems as regime-conditioned climate-response geometries | comparative implications |
| 9. Conclusion | summarize structural and predictive findings | final claims |

## 3. What changed relative to the old version

| Old framing | Current framing |
|---|---|
| one GAM per destination as main model | one MS-GAM per destination as main model |
| function comparison across destinations only | regime-specific comparison within and across destinations |
| prediction as simple feature validation | prediction as usefulness test plus structural validation |
| robustness of function shape only | robustness of regimes, functions, and prediction gains |

## 4. Quick reminder

Different destinations are estimated separately.
This is not a pooled panel design.

If a note or section ignores regime index \(k\), it should be treated as background, diagnostic evidence, or legacy material rather than the main empirical spine.
