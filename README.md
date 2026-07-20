# Stochastic Calculus & Finance: The Heath-Jarrow-Morton Model

This repository accompanies my presentation for the **Summer 2026 University of Texas at Austin Directed Reading Program**. It contains both the talk itself and a complete, working implementation of the one-factor Heath-Jarrow-Morton (HJM) model calibrated to historical Federal Reserve data.

The reading followed Steven E. Shreve's *Stochastic Calculus for Finance II: Continuous-Time Models*, building up the measure-theoretic probability needed to make sense of the stochastic differential equation

$$
    df(t, T) = \alpha(t, T)\,dt + \sigma(t, T)\,dW(t),
$$

where $f(t, T)$ is the instantaneous forward rate for maturity $T > t$ that can be locked in at time $t$. This is the HJM model, and it describes how the entire forward-rate curve evolves over time.

## Contents

| File | Description |
| --- | --- |
| `main.tex` / `main.pdf` | The presentation. A `beamer` talk that develops the probability theory behind HJM, from the coin-toss space up to Brownian motion, and closes with the model itself. |
| `main.ipynb` | The main deliverable. A complete one-factor HJM simulation calibrated to historical Federal Reserve yield-curve data. |
| `basic_ex.ipynb` | A short toy example. The HJM model simulated on a synthetic initial forward curve $f(0, T) = 0.05\,e^{-0.3T}$, included as the coded example shown in the talk. |
| `feds200628.csv` | The historical dataset (see below). |
| `images/` | Figures used in the presentation. |

## The Presentation

The talk is aimed at an audience with a background in basic calculus and probability, and introduces the remaining machinery as it is needed. Its rough ordering of topics is:

- **Measures**: assigning size and likelihood to events on the coin-toss space $\Omega = \{-1, 1\}^{\mathbb{N}}$.
- **Sigma-algebras, random variables, and information**: probability spaces $(\Omega, \mathcal{F}, \mathbb{P})$, and filtrations $\mathcal{F}(t)$ as a model of "what we know so far."
- **Random walks and Brownian motion**: from the symmetric random walk $S_n$, to the scaled random walk $W^{(n)}(t)$, to Brownian motion $W(t)$ in the limit.
- **Putting it all together**: the HJM model, why the forward rate is driven by a Brownian motion, and how it is used in practice.

## The Notebook

`main.ipynb` gives a self-contained, implementation-level treatment of the one-factor HJM model. Financial quantities, modeling assumptions, and numerical methods are introduced as they are used; the measure-theoretic construction of Brownian motion and the derivation of the risk-neutral measure are left to the talk and to Shreve.

### Data

The simulation is calibrated to the **Gürkaynak-Sack-Wright (GSW) U.S. Treasury yield-curve dataset** (`feds200628.csv`), a Federal Reserve staff research product covering the Treasury curve from 1961 to the present. The dataset provides the six Svensson parameters ($\beta_0, \beta_1, \beta_2, \beta_3, \tau_1, \tau_2$) for each date, from which the instantaneous forward rate at any maturity can be reconstructed:

$$
    f(t, t+\tau) = \beta_0 + \beta_1 e^{-\tau/\tau_1} + \beta_2 \frac{\tau}{\tau_1} e^{-\tau/\tau_1} + \beta_3 \frac{\tau}{\tau_2} e^{-\tau/\tau_2}.
$$

### Method

The notebook proceeds in the following stages:

1. **Build forward-rate panels.** Using the Svensson parameters, the daily forward curve is reconstructed on a grid of maturities $\tau \in \{1.0, 1.5, \dots, 30.0\}$ years from `1990-01-01` onward, together with a one-day-ahead panel offset by $\delta = 1/252$.

2. **Estimate volatility.** Normalized daily forward-rate increments form an empirical covariance matrix $C$. Since a change of measure only alters the drift, this covariance (estimated under the real-world measure) is enough to recover the volatility loading $\sigma(t, T)$ under the risk-neutral measure. A single principal component captures the dominant mode of curve movement; the notebook reports the first-factor explained variance and the rank-one approximation error.

3. **Enforce no arbitrage.** Under the risk-neutral measure, the one-factor HJM drift is fully determined by the volatility:

$$
    \alpha(t, T) = \sigma(t, T)\,\sigma^{*}(t, T), \qquad \sigma^{*}(t, T) = \int_t^T \sigma(t, u)\,du.
$$

   So once $\sigma$ is estimated, no further calibration is required.

4. **Simulate the curve.** Starting from the most recent observed forward curve as the initial condition $f(0, \cdot)$, the curve is evolved forward with an Euler-Maruyama scheme, drawing a single Brownian increment at each step and rolling the maturity grid forward as observation time advances. The result is a family of simulated forward curves out to a chosen horizon.

### Running it

The notebook depends on `numpy`, `pandas`, `scipy`, and `matplotlib`. Open `main.ipynb` and run the cells top to bottom; `feds200628.csv` must be present in the repository root.

## References

Steven E. Shreve. *Stochastic Calculus for Finance II: Continuous-Time Models*. Springer Finance, Springer, 2004.
