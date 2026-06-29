# MS-GAM Core Formula Draft

Updated: 2026-06-26

This note records the current core model direction.

Important convention:

> The formulas below are written for one destination.  
> Different destinations are estimated separately, not as a pooled panel.

## 1. Latent regime process

Let the latent tourism-demand regime be:

[
S_t \in \{1,\dots,K\}
]

The regime evolves as a first-order Markov chain:

[
P(S_t=j \mid S_{t-1}=i)=p_{ij}
]

with transition matrix:

[
\mathbf{P}=(p_{ij})_{K\times K}
]

## 2. Regime-specific climate-response function

For each latent regime \(k\), define a regime-specific smooth response function:

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

where:

- \(T_t\): monthly temperature state
- \(R_t\): monthly rainfall / precipitation state
- \(E_t\): extreme-weather exposure or intensity
- \(m_t\): calendar month / seasonality
- \(te_k(T_t,R_t)\): regime-specific temperature-rainfall interaction surface

## 3. Switching observation equation

Let:

[
y_t=\log(Y_t+1)
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
\varepsilon_t,
\qquad
\varepsilon_t\mid S_t=k\sim \mathcal{N}(0,\sigma_k^2)
]

## 4. Role of structural breaks

Structural break analysis is not equivalent to Markov switching.

In the current design:

[
\text{breakpoints}
\Rightarrow
\text{diagnostic evidence of regime instability}
]

[
\text{MS-GAM}
\Rightarrow
\text{main latent-regime climate-response model}
]

So breakpoints can motivate or validate regime changes, but the main model treats \(S_t\) as a latent stochastic state inferred from the data.

## 5. Extreme-weather term

The extreme term is kept abstract in the main formula:

[
f_k^{ext}(E_t)
]

Candidate definitions of \(E_t\):

[
E_t^{bin}
=
\mathbf{1}(\text{extreme event occurs in month }t)
]

[
E_t^{int}
=
\text{continuous tail intensity in month }t
]

[
E_t^{comp}
=
\text{compound extreme exposure in month }t
]

The current preference is:

- keep \(te_k(T_t,R_t)\) for climate interaction surface;
- use \(E_t\) for explicit extreme-weather exposure;
- decide the final \(E_t\) construction before writing the empirical code.
