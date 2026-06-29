# Chapter 4: Inference of Regime-Specific Climate-Response Functions

Updated: 2026-06-28

This file controls the Chapter 4 method note after the shift from a single GAM to a Markov-Switching GAM framework.

Important convention:

> All formulas are written for one destination. Different destinations are estimated separately, not as a pooled panel.

## 1. Role of Chapter 4

Chapter 4 is the inference bridge between the conceptual system in Chapter 2 and the comparative and predictive chapters that follow.

Its job is to recover three objects from observed data:

| Object | Meaning | Used in |
|---|---|---|
| \(S_t\) | latent regime state | Chapter 5, 6, 7 |
| \(\mathcal{F}^{(k)}(X_t)\) | regime-specific climate-response function | Chapter 5, 7 |
| \(\Pi_t\) | filtered posterior regime probabilities | Chapter 6, 7 |

So this chapter is no longer a plain "fit one GAM" section. It is the chapter where latent regimes and regime-specific smooth response functions are inferred jointly.

## 2. Observed data objects

Let monthly tourism demand be:

[
y_t=\log(Y_t+1)
]

Let the observed climate-state block be:

[
X_t=(T_t,R_t,E_t,m_t,t)
]

where:

- \(T_t\): monthly temperature state
- \(R_t\): monthly rainfall / precipitation state
- \(E_t\): extreme-weather exposure or intensity
- \(m_t\): calendar month
- \(t\): time index for long-run trend

## 3. Latent regime process

The latent regime is:

[
S_t \in \{1,\dots,K\}
]

with first-order Markov transition:

[
P(S_t=j\mid S_{t-1}=i)=p_{ij}
]

and transition matrix:

[
\mathbf{P}=(p_{ij})_{K\times K}
]

This is the switching layer of the model. Structural break analysis may still be used as diagnostic evidence, but the main regime object is the latent stochastic state \(S_t\).

## 4. Regime-dependent observation equation

For each regime \(k\), define the regime-specific smooth response function:

[
\mathcal{F}^{(k)}(X_t)
=
s_k^{trend}(t)
+
s_k^{season}(m_t)
+
f_k^{temp}(T_t)
+
f_k^{rain}(R_t)
+
te_k(T_t,R_t)
+
f_k^{ext}(E_t)
]

The observation equation is:

[
y_t \mid S_t=k
\sim
\mathcal{N}
\left(
\mathcal{F}^{(k)}(X_t),
\sigma_k^2
\right)
]

Equivalently:

[
y_t
=
\sum_{k=1}^{K}
\mathbf{1}(S_t=k)\mathcal{F}^{(k)}(X_t)
+
\varepsilon_t
]

This makes Chapter 4 an inference problem over a family of smooth functions rather than a single destination-level response curve.

## 5. Joint likelihood

The joint model can be written as:

[
p(y_{1:T},S_{1:T}\mid X_{1:T})
=
p(S_1)
\prod_{t=2}^{T}P(S_t\mid S_{t-1})
\prod_{t=1}^{T}
p(y_t\mid S_t,X_t)
]

with:

[
p(y_t\mid S_t=k,X_t)
=
\phi
\left(
y_t;
\mathcal{F}^{(k)}(X_t),
\sigma_k^2
\right)
]

where \(\phi(\cdot)\) denotes the Gaussian density.

This likelihood is the formal link that legitimizes the Markov-switching layer. It also clarifies that regime inference and function estimation are part of the same system.

## 6. Inference outputs

The estimation stage should deliver four outputs.

### 6.1 Regime-specific smooth functions

[
\hat{\mathcal{F}}^{(1)},\dots,\hat{\mathcal{F}}^{(K)}
]

These are the core objects for Chapter 5.

### 6.2 Filtered regime probabilities

At forecast origin \(t\), define:

[
\Pi_t
=
\left(
P(S_t=1\mid y_{1:t},X_{1:t}),
\dots,
P(S_t=K\mid y_{1:t},X_{1:t})
\right)
]

These are the forecast-safe structural inputs for Chapter 6.

### 6.3 Smoothed regime probabilities

[
P(S_t=k\mid y_{1:T},X_{1:T})
]

These are useful for interpretation, regime labeling, and structural diagnostics, but they should not enter the forecasting layer because they use future information.

### 6.4 Estimated transition dynamics

[
\hat{\mathbf{P}}
=
(\hat p_{ij})_{K\times K}
]

This is needed for regime persistence and transition analysis in Chapter 7.

## 7. Practical estimation strategy

The implementation can be described as an iterative inference scheme:

| Step | Input | Action | Output |
|---|---|---|---|
| 1 | \(K\), initial state probabilities | initialize latent regimes | starting weights |
| 2 | current state weights | fit regime-weighted GAM components | \(\hat{\mathcal{F}}^{(k)}\) |
| 3 | fitted regime means and variances | update state probabilities and transition parameters | \(\Pi_t\), \(\hat{\mathbf{P}}\) |
| 4 | updated states | iterate until convergence | final regime-function system |

In exposition, it is enough to say that Chapter 4 jointly alternates between regime inference and regime-specific GAM smoothing. The chapter does not need to become a full algorithm paper.

## 8. What Chapter 4 passes forward

The chapter hands off different objects to later sections:

| Chapter | Object handed off | Why it matters |
|---|---|---|
| Chapter 5 | \(\hat{\mathcal{F}}^{(k)}\) | compare regime-specific shapes and interaction geometry |
| Chapter 6 | \(\Pi_t\) and derived \(Z_t\) | build structure-aware forecasts |
| Chapter 7 | \(\hat{\mathbf{P}}\), \(\hat{\mathcal{F}}^{(k)}\), posterior regime paths | test structural robustness |

## 9. One-line closure

Chapter 4 can be summarized as:

[
\boxed{
(y_{1:T},X_{1:T})
\Rightarrow
\left(
\hat{\mathcal{F}}^{(1:K)},
\hat{\mathbf{P}},
\Pi_{1:T}
\right)
}
]

This is the inference layer of the paper. Later chapters should not fall back to a single, regime-free destination function.

------

