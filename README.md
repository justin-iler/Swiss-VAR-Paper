# Bayesian VAR Forecasting and Fiscal Policy Analysis for Switzerland

> A Bayesian Vector Autoregression (BVAR) analysis of Swiss macroeconomic data (2010–2020) using Minnesota priors for forecasting and structural policy analysis.

**Authors:** Justin Iler & Prathinav Vishnu  
**Institution:** University of Pennsylvania  
**Date:** April 2023 (code revised October 2026)

---

## Overview

This repository contains the implementation of a Bayesian Vector Autoregression (BVAR) for forecasting Swiss interest rates, unemployment and inflation, together with a structural analysis of monetary and fiscal policy shocks.

The model uses a **Minnesota prior** whose tightness is selected by **Marginal Data Density (MDD)**. Forecasts are evaluated recursively without look-ahead and on a held-out year (2021).

---

## Key Results

### Recursive Forecast Accuracy (2013 Q2 – 2020 Q4)

One-step-ahead mean squared error, original units. At each forecast origin the data are standardized, the lag length and λ are chosen by MDD, and the model is estimated using only data available at that date. Following Giannone, Lenza & Primiceri (2015), the Bayesian VAR is compared with a flat-prior (OLS) VAR and a no-change random walk. 31 forecast origins.

| Variable | BVAR | OLS VAR | Random walk | OLS / BVAR | RW / BVAR |
|-----------|-----:|-----:|-----:|-----:|-----:|
| Interest Rate | 0.062 | 0.317 | 0.023 | 5.1× | 0.37× |
| Unemployment | 0.060 | 0.059 | 0.067 | 1.0× | 1.12× |
| Inflation | 0.118 | 0.177 | 0.131 | 1.5× | 1.10× |

Shrinkage helps most for the interest rate, where the unrestricted VAR swings wildly around the 2015 rate cut; for unemployment the two VARs are equally accurate. Against the random walk, the BVAR is about 10% more accurate for unemployment and inflation but much worse for the interest rate, which was pinned near its lower bound for most of the sample and so favors a no-change forecast.

### Hold-Out Validation (2021)

Iterated 1- to 4-quarter-ahead forecasts from 2020 Q4, with no 2021 data used.

| Variable | RMSE | Bias (actual − forecast) |
|-----------|-----:|-----:|
| Interest Rate | **0.034** | −0.022 |
| Unemployment | **0.322** | 0.169 |
| Inflation | **0.351** | 0.329 |

The model tracks the interest rate closely. It underpredicts unemployment early in 2021 (Q1 lies outside the 90% predictive band) and misses the speed of the recovery, and it underpredicts the 2021 pickup in inflation in every quarter.

### Structural Analysis

- **Monetary policy:** a contractionary shock (sign-restricted: rate ↑, unemployment ↑, inflation ↓ on impact) raises the rate by about 6 bp, raises unemployment by about 0.11 pp and lowers annualized inflation by about 0.14 pp on impact. Effects on unemployment and inflation fade within about a year and a half.
- **Fiscal policy:** a quarterly VAR(4) in real government consumption and real GDP (constant + trend, spending ordered first, following Blanchard & Perotti 2002), estimated on 1980–2019. The cumulative CHF multiplier is about 2.2 on impact, peaks near 4 after a year and is about 2.3 at four years. The CHF conversion multiplies the GDP elasticity by GDP/spending, and government consumption is only about 11% of Swiss GDP, so a modest percentage GDP response maps into a large multiplier. On the project's 2010–2020 window alone the spending response dies out within four years and the multiplier is not identified. Estimates come without error bands and should be read as indicative.

---

## Repository Structure

```text
Swiss VAR/
├── Code/
│   └── VAR_Switzerland.Rmd     # full analysis; regenerates all figures
├── Data/                       # input series (CSV)
├── Figures/
│   ├── timeseries.png
│   ├── forecasts.png
│   ├── forecasts_2021.png
│   ├── monetary_irf.png
│   └── fiscal_irf.png
├── Report/
│   ├── VAR_Switzerland.tex     # paper source (revised October 2026)
│   └── VAR_Switzerland.pdf     # compiled paper
└── README.md
```

> **Note:** the paper was originally submitted in April 2023 and revised in October 2026 to reflect the current code and results.

---

## Figures

| Figure | Description |
|---------|-------------|
| `timeseries.png` | Interest rate, unemployment and inflation, 2010–2021 (2021 hold-out shaded) |
| `forecasts.png` | Recursive one-step-ahead forecasts: Bayesian VAR vs. OLS VAR vs. actual (RMSEs incl. random walk in titles) |
| `forecasts_2021.png` | 2021 hold-out forecasts with 90% predictive bands |
| `monetary_irf.png` | Responses to a contractionary monetary policy shock (pp, 90% bands) |
| `fiscal_irf.png` | Responses to a government spending shock and cumulative multiplier |

---

## Methodology

### Bayesian VAR

- **Sample:** 2010 Q1 – 2020 Q4 (44 quarters); 2021 Q1–Q4 held out
- **Variables:** 3-month interest rate, unemployment rate, CPI inflation (seasonally adjusted with X-13, annualized: 400 × Δlog CPI)
- **Lag length:** chosen jointly with λ₁ by MDD from p ∈ {1, 2, 3, 4}. All candidates are evaluated on the same estimation sample, starting after a 5-quarter pre-sample.
- Variables are standardized before estimation (using only the estimation sample's moments).

### Stationarity

ADF and KPSS tests show that seasonally adjusted inflation is stationary, the interest rate is nonstationary over 2010–2020 (mostly the 2015 level shift), unemployment is borderline, and log government consumption and log GDP are I(1). Both VARs are nevertheless estimated in levels: a levels VAR nests the differenced model, the Minnesota prior is built for persistent series, and levels keep long-run relationships such as the stable spending-to-GDP ratio. The test table is in the paper.

### Minnesota Prior

λ₁ is chosen together with the lag length by maximizing the MDD over

```
λ₁ ∈ {0.05, 0.1, 0.2, 0.5, 1, 2, 5, 10},  p ∈ {1, 2, 3, 4}
```

with λ₂ = 4 and λ₃ = λ₄ = λ₅ = 1, following Del Negro & Schorfheide (2011). The prior's scale and level moments come from the pre-sample, not the estimation sample. On the full 2010–2020 sample the optimum is **p = 1, λ₁ = 1**, an interior maximum, and the same choice is made at every recursive forecast origin. Posterior inference uses 1,000 draws from the Normal–Inverse-Wishart posterior.

### Identification

Impulse responses are reported up to 4 years (16 quarters).

- **Monetary policy:** sign restrictions on impact. The interest rate rises, unemployment rises and inflation falls, using random orthogonal rotations of the Cholesky factor.
- **Fiscal policy:** bivariate VAR(4) in quarterly log real government consumption and log real GDP with a constant and a linear time trend, estimated by OLS on 1980 Q1 – 2019 Q4 (2010–2020 shown for comparison). Identification is recursive (Cholesky), with spending ordered first. Log responses are converted to CHF at the last observation's levels and reported as a cumulative multiplier over 16 quarters.

---

## Running the Analysis

Requirements: R ≥ 4.0, a LaTeX installation with XeLaTeX, and

```r
install.packages(c("rmarkdown", "CholWishart", "MCMCpack", "mvtnorm", "seasonal"))
```

From the `Code/` folder:

```r
rmarkdown::render("VAR_Switzerland.Rmd")
```

This reads `Data/`, writes all five figures to `Figures/`, and produces `Code/VAR_Switzerland.pdf` with the results tables. Seeds are fixed, so results are reproducible.

To rebuild the paper from the figures, run from the `Report/` folder:

```bash
xelatex VAR_Switzerland.tex && xelatex VAR_Switzerland.tex
```

---

## Data

| Variable | Frequency | Source |
|-----------|-----------|--------|
| Interest Rate | Quarterly | Swiss 3-month LIBOR, FRED / ECB |
| Unemployment | Quarterly | State Secretariat for Economic Affairs (SECO), seasonally adjusted |
| Inflation | Quarterly | Swiss National Bank CPI; raw quarter-on-quarter changes, seasonally adjusted (X-13, 1990–2021) and annualized |
| Real GDP | Quarterly | Federal Statistical Office (FSO) |
| Government Consumption | Quarterly | Real general government final consumption, seasonally adjusted. OECD Main Economic Indicators via FRED ([NAEXKP03CHQ652S](https://fred.stlouisfed.org/series/NAEXKP03CHQ652S)) |

---

## Limitations

- The fiscal SVAR uses aggregate rather than per-capita series (no quarterly population series is included) and reports point estimates without error bands. Recursive identification also ignores fiscal anticipation effects, a well-known concern for fiscal SVARs.
- The model assumes stable relationships and cannot anticipate structural breaks such as the post-pandemic recovery and inflation.
- 3-month CHF LIBOR was discontinued at the end of 2021; an extension would switch to SARON.

---

## Citation

```bibtex
@misc{iler2023bvar,
  author = {Justin Iler and Prathinav Vishnu},
  title = {Bayesian VAR Forecasting and Fiscal Policy Analysis for Switzerland},
  year = {2023},
  note = {University of Pennsylvania}
}
```

---

## Acknowledgments

Data from the Swiss National Bank (SNB), Swiss Federal Statistical Office (FSO), State Secretariat for Economic Affairs (SECO) and Federal Reserve Economic Data (FRED).
