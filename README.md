# Weather Derivative Pricing and Risk Analysis

## Abstract
This project investigates the valuation and risk assessment of **weather derivatives**—financial instruments whose payoffs depend on observable meteorological variables such as temperature or precipitation. Unlike conventional insurance contracts, which indemnify specific losses, weather derivatives enable firms in weather-sensitive sectors (e.g., energy, agriculture, logistics) to **hedge against adverse weather-driven revenue fluctuations**.

The work implements a Monte Carlo–based pricing framework for temperature-indexed derivatives such as **Heating Degree Day (HDD)** and **Cooling Degree Day (CDD)** contracts. Empirical weather data are obtained from NOAA’s Climate Data Online (CDO) service, and stochastic temperature evolution is modeled using a **seasonal Ornstein–Uhlenbeck process** with additive noise. Numerical experiments demonstrate the dependence of derivative prices on volatility, seasonality, and strike specification.

---

## 1. Introduction
Weather derivatives represent an emerging class of **non-catastrophic risk-transfer instruments**, facilitating financial hedging against deviations in ambient temperature from long-term climatological norms. The contracts are typically settled based on accumulated temperature indices:

$$
\text{HDD}_t = \max(T_{\mathrm{ref}} - T_t, 0), \qquad
\text{CDD}_t = \max(T_t - T_{\mathrm{ref}}, 0),
$$

where $T_t$ denotes the daily average temperature and $T_{\mathrm{ref}}$ (commonly 18.33 °C or 65 °F) is the baseline comfort threshold.  
The cumulative index across the contract horizon $(t=1,\dots,N)$ defines the derivative payoff base:

$$
\text{HDD} = \sum_{t=1}^{N} \text{HDD}_t, \quad
\text{CDD} = \sum_{t=1}^{N} \text{CDD}_t.
$$

A **European-style call** on the cumulative index is valued as

$$
P_0 = e^{-rT}\mathbb{E}^{\mathbb{Q}}\!\left[\max(X - K, 0)\right],
$$

where $r$ is the risk-free rate, $T$ the time to maturity, $K$ the strike, and the expectation is computed under the risk-neutral measure $\mathbb{Q}$.

---

## 2. Methodology

### 2.1 Data Preprocessing
- **Data Source:** NOAA Climate Data Online (CDO) API or local CSV fallback.  
- **Station:** Seattle–Tacoma International Airport (example).  
- **Schema Normalization:** Each daily record contains columns `TMIN`, `TMAX`, and `TAVG` (in °C). Missing observations are imputed via linear interpolation.

### 2.2 Temperature Modeling
Temperature dynamics are modeled using a **mean-reverting Ornstein–Uhlenbeck (OU)** process with seasonal drift:

$$
dT_t = \kappa(\theta_t - T_t)\,dt + \sigma_t\,dW_t,
$$

where  
- $\kappa$ controls the rate of mean reversion,  
- $\theta_t$ encodes deterministic seasonal trends, and  
- $\sigma_t$ captures stochastic fluctuations.

Model parameters are calibrated by least squares fitting of daily temperature increments to OU residuals.

### 2.3 Monte Carlo Pricing
1. Simulate $M$ temperature paths under the calibrated OU model.  
2. Compute daily degree-day indices (HDD/CDD).  
3. Evaluate the payoff $\max(X - K, 0)$ for each path.  
4. Discount and average under the risk-neutral measure to estimate the fair premium $P_0$.

### 2.4 Sensitivity and Risk Analysis
The notebook explores:
- Sensitivity of derivative price to **strike** $K$, **volatility** $\sigma$, and **mean reversion** $\kappa$.
- Empirical **distribution of payoffs**, **expected payout variance**, and **hedging implications** for energy utilities.

---

## 3. Numerical Results
Simulation results confirm key theoretical insights:
- Higher volatility and slower mean reversion increase derivative premiums.  
- Seasonal bias in $\theta_t$ strongly influences contract moneyness.  
- The OU process accurately reproduces observed temperature autocorrelation, ensuring statistical realism.  

Representative plots include:
- Historical vs. simulated temperature trajectories.  
- Empirical distribution of cumulative HDD/CDD indices.  
- Monte Carlo convergence diagnostics and sensitivity curves.

---

## 4. Implementation Details

| Component | Description |
|------------|-------------|
| **Language** | Python 3.11 |
| **Core Libraries** | `numpy`, `pandas`, `matplotlib`, `scipy`, `requests` |
| **Data Source** | NOAA CDO API or local CSV |
| **Pricing Technique** | Monte Carlo simulation under seasonal OU dynamics |
| **Random Seed** | Fixed for reproducibility |
| **Notebook** | `Weather_derivative_final_code.ipynb` |

---

## References

- Hooda, Soumil, Shubham Sharma, and Kunal Bansal. "Quantifying Seasonal Weather Risk in Indian Markets: Stochastic Model for Risk-Averse State-Specific Temperature Derivative Pricing." arXiv preprint arXiv:2409.04541 (2024).
- Richards, Timothy J., Mark R. Manfredo, and Dwight R. Sanders. "Pricing weather derivatives." American Journal of Agricultural Economics 86, no. 4 (2004): 1005-1017.
- Considine, Geoffrey. "Introduction to weather derivatives." Weather derivatives group, Aquila energy (2000): 1-10.
