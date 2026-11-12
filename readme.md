# Cross-Asset Market Risk Capital & Counterparty Credit Risk Engine

**Unified Quantitative Risk Framework | Basel FRTB-SBM, VaR/SVaR, Monte Carlo PFE & SA-CCR**

[![Python](https://img.shields.io/badge/Python-3.11%2B-blue.svg)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Notebook-WQU%20ADSL%20Standard-orange.svg)](https://jupyter.org/)
[![Status](https://img.shields.io/badge/Build-Active%20Sprint%20(Modules%201--2%20Live)-brightgreen.svg)]()

---

## 1. Executive Summary

This repository houses an end-to-end quantitative risk analytics engine built in Python following the **WorldQuant University Applied Data Science Lab (ADSL)** modular standard (`Prepare` $\rightarrow$ `Explore` $\rightarrow$ `Model` $\rightarrow$ `Deliver`).

The engine evaluates a shared **$\$60,000,000$ multi-asset relative-value commodity and securities financing (SFT) netting set** with counterparty **VectaLink Global Ltd.** across two institutional risk lenses:

1. **Market Risk & Regulatory Capital Coverage:** Full revaluation pricing, $1$-day and $10$-day $99\%$ Historical and Parametric $\text{VaR}$, Basel II.5 Stressed $\text{VaR}$ ($\text{SVaR}$) calibrated back to $2007$, first-order Greeks P&L attribution, **FRTB Sensitivities-Based Method (SBM)** commodity capital aggregation, and **Kupiec POF** model backtesting.
2. **Counterparty Credit Risk (CCR) & Pre-Trade Exposure Management:** Multi-factor Cholesky-correlated Monte Carlo path simulation incorporating bilateral **ISDA/CSA** collateral mechanics ($10$-day Margin Period of Risk), multi-tenor Potential Future Exposure ($PFE_{95\%}$, $PFE_{99\%}$), Effective $\text{EPE}$, **Basel III SA-CCR** Exposure at Default ($\text{EAD}$), and pre-trade incremental $\Delta PFE$ and repo haircut stress checks.

---

## 2. Portfolio & Bilateral CSA Architecture

All trades reside within a single legally enforceable ISDA netting set against **VectaLink Global Ltd.**, capturing directional energy exposure, transatlantic crude grade basis risk, non-linear volatility convexity, and collateralized repo financing.

### Netting Set Trade Blotter

| Trade ID | Desk | Instrument | Benchmark | Direction | Notional ($\text{USD}$) | Tenor ($\text{Years}$) |
| :--- | :--- | :--- | :--- | :--- | ---: | ---: |
| `TRD-VL-01` | Commodities | Brent Crude Swap | ICE Brent (`BZ=F`) | Long | $\$25,000,000$ | $1.00$ |
| `TRD-VL-02` | Commodities | WTI Basis Swap | NYMEX WTI (`CL=F`) | Short | $\$15,000,000$ | $2.00$ |
| `TRD-VL-03` | Commodities | Natural Gas Call | Henry Hub (`NG=F`) | Long | $\$10,000,000$ | $0.50$ |
| `TRD-VL-04` | Financing / SFT | US 10Y Reverse Repo | US 10Y Yield (`^TNX`) | Long | $\$10,000,000$ | $0.25$ |

* **Implied Volatility Calibration Anchor:** CBOE Crude Oil Volatility Index (`^OVX`)
* **Bilateral CSA Legal Parameters:**
  * **Threshold ($TH$):** $\$0$ (post-$2008$ unthresholded bilateral standard)
  * **Minimum Transfer Amount ($MTA$):** $\$500,000$
  * **Margin Period of Risk ($MPOR$):** $10$ business days

---

## 3. Development Roadmap & Module Status

This repository is updated iteratively as each quantitative module passes automated validation assertions:

- [x] **Module 1: Prepare (Data Ingestion & Netting Set Schema)**
  - Historical daily settlement ingestion from January $1, 2007$ to present ($>4,500$ trading sessions) capturing the $2008$ GFC, $2014\text{--}2016$ oil downturn, $2020$ COVID shock, and $2022$ energy crisis.
  - Multi-exchange calendar synchronization via bounded forward-fill (`ffill(limit=3)`) aligned with clearinghouse mark-to-market conventions.
  - Continuous logarithmic return transformation and automated unit assertions.
- [x] **Module 2: Explore (Basis Dynamics, Fat Tails & Basel SVaR Calibration)**
  - Transatlantic Brent–WTI basis spread decomposition and rolling $30$-day Pearson correlation regime tracking.
  - Higher-moment tail diagnostics (annualized volatility, skewness, and excess kurtosis) demonstrating non-normality across energy curves.
  - Algorithmic identification of the continuous $250$-day peak historical volatility window ($\text{SVaR}$ calibration slice).
- [ ] **Module 3: Model Part I — Market Risk & FRTB Capital**
  - Analytical Black-76 option pricer, commodity swap discounting, and Greeks engine ($\Delta, \Gamma, \mathcal{V}, \Theta$).
  - $1\text{D}$ & $10\text{D}$ $99\%$ Historical $\text{VaR}$, Parametric Delta-Normal $\text{VaR}$, and Stressed $\text{VaR}$ ($\text{SVaR}$).
  - FRTB-SBM commodity Delta and Vega bucket capital aggregation and Kupiec POF backtesting.
- [ ] **Module 4: Model Part II — Counterparty Credit Risk & Pre-Trade Engine**
  - Cholesky-decomposed Monte Carlo diffusion across correlated risk factors.
  - Dynamic Variation Margin simulation with $10$-day $\text{MPOR}$ lag, $CE$, $EE(t)$, $PFE_{95\%/99\%}$, and Effective $\text{EPE}$.
  - Basel III $\text{SA-CCR}$ $\text{EAD}$ calculation and pre-trade $\Delta PFE$ / sovereign repo haircut stress testing.
- [ ] **Module 5: Deliver — Interactive Risk Visualisations & Governance Memo**
  - Interactive Plotly exposure profiles, basis charts, and backtesting exception plots.
  - Executive Second-Line Risk Challenge & Capital Optimization Memo.

---

## 4. Core Quantitative Formulations

### Market Risk & Regulatory Capital
* **Continuous Log Returns:**
  $$r_{i, t} = \ln\left(\frac{S_{i, t}}{S_{i, t-1}}\right)$$
* **Parametric Delta-Normal Value-at-Risk ($10$-Day Scaling):**
  $$\text{VaR}_{\alpha, 10\text{D}} = Z_{\alpha} \sqrt{\mathbf{w}^T \mathbf{\Sigma} \mathbf{w}} \cdot V_{\text{port}} \cdot \sqrt{10}$$
* **FRTB-SBM Intra-Bucket Commodity Capital:**
  $$K_{\text{Delta}} = \sqrt{\sum_{k} WS_k^2 + \sum_{k} \sum_{l \neq k} \rho_{kl} WS_k WS_l}$$

### Counterparty Credit Risk & CSA Collateralization
* **Collateralized Exposure under CSA with MPOR Lag:**
  $$E_m(t) = \max\left(V_m(t) - C_m(t - \text{MPOR}), 0\right)$$
* **Potential Future Exposure & Effective EPE:**
  $$\text{PFE}_{\alpha}(t) = \text{Quantile}_{\alpha}\left(\{E_m(t)\}_{m=1}^M\right), \quad \text{Effective EPE} = \frac{1}{T} \sum_{k=1}^K \text{Effective EE}(t_k) \Delta t_k$$
* **Basel III SA-CCR Exposure at Default:**
  $$\text{EAD} = 1.4 \times \left(\text{RC} + \text{Multiplier} \times \text{AddOn}_{\text{Commodity}}\right)$$

---

## 5. Repository Structure & Quick Start

```text
unified-risk-engine/
├── README.md                                # Project overview, roadmap & mathematical spec
├── requirements.txt                         # Reproducible Python dependencies
├── unified_risk_engine.ipynb                # Master WQU ADSL quantitative notebook
└── docs/
    └── methodology_and_assumptions_log.md   # Detailed regulatory & modeling reference log
```

### Local Installation & Execution

1. **Clone the repository:**
   ```bash
   git clone https://github.com/<your-username>/unified-risk-engine.git
   cd unified-risk-engine
   ```
2. **Create and activate a virtual environment:**
   ```bash
   python -m venv venv
   source venv/Scripts/activate   # Windows Git Bash
   # source venv/bin/activate     # macOS / Linux
   ```
3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```
4. **Launch Jupyter / VS Code:**
   Open `unified_risk_engine.ipynb`, select the `venv` Python kernel, and run cells sequentially.

---

## 6. Author

**Dhvanil Zaveri**  
*Quantitative Risk & Derivatives Analytics | MBA (Quantitative Finance), IIM Tiruchirappalli | FRM Candidate*  
Location: London, United Kingdom