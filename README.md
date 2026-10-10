# 📈 U.S. Treasury Note Valuation & Interest Rate Sensitivity Analysis

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
[![TreasuryDirect API](https://img.shields.io/badge/Data-TreasuryDirect%20API-orange.svg)](https://www.treasurydirect.gov/)

An end-to-end quantitative fixed-income project that fetches live auction results directly from the official **U.S. TreasuryDirect API**, models semiannual cash flows for newly issued Treasury Notes, solves for implied Yield to Maturity (YTM), and evaluates interest rate risk via Macaulay/Modified Duration and Convexity approximations versus exact repricing.

---

## 📌 Project Overview

This repository demonstrates the practical pricing and risk sensitivity analysis of U.S. government debt:
1. **Automated Data Retrieval:** Queries the TreasuryDirect REST API (`/securities/search`) to pull real-time auction and securities data.
2. **Bond Selection & Parsing:** Identifies the latest active 5-Year Treasury Note (CUSIP: `91282CRN3`), extracting key contractual parameters (issue date, maturity, coupon rate, high/stop-out yield, and auction price).
3. **Valuation Engine:** Builds cash flow schedules ($1,000 par, semiannual frequency) and applies numerical root-finding (`scipy.optimize.brentq`) to solve for exact implied YTM.
4. **Risk & Sensitivity Profiling:** Calculates **Macaulay Duration**, **Modified Duration**, and **Convexity**.
5. **Taylor Series Approximations vs. Exact Repricing:** Benchmarks first-order (Duration-only) and second-order (Duration + Convexity) approximations against full repricing across a range of interest rate shocks (from -100 bps to +300 bps).

---

## 🧮 Mathematical Framework

### 1. Bond Pricing & Yield to Maturity (YTM)
For a bond with face value $M$, coupon payment $C$, and $2n$ semiannual periods:
$$P = \sum_{t=1}^{2n} \frac{C}{\left(1 + \frac{y}{2}\right)^t} + \frac{M}{\left(1 + \frac{y}{2}\right)^{2n}}$$

### 2. Macaulay & Modified Duration
$$\text{MacD} = \frac{\sum_{t=1}^{2n} \frac{t}{2} \cdot \text{PV}(CF_t)}{P}, \quad \text{ModD} = \frac{\text{MacD}}{1 + \frac{y}{2}}$$

### 3. Convexity
$$\text{Convexity} = \frac{1}{P \cdot \left(1 + \frac{y}{2}\right)^2} \sum_{t=1}^{2n} \frac{\frac{t}{2}\left(\frac{t}{2} + 0.5\right) \cdot CF_t}{\left(1 + \frac{y}{2}\right)^t}$$

### 4. Taylor Series Approximation of Price Changes
$$\frac{\Delta P}{P} \approx -\text{ModD} \cdot \Delta y + \frac{1}{2} \cdot \text{Convexity} \cdot (\Delta y)^2$$

---

## 📊 Key Results & Empirical Findings

*Analyzed Asset: 5-Year U.S. Treasury Note (CUSIP `91282CRN3`, Coupon `5.00%`, Maturity: `2031-09-30`)*

| Metric | Value |
| :--- | :--- |
| **Auction Price (Stop-out)** | **$998.5571** (per $1,000 par) |
| **High Yield (Stop-out YTM)** | **5.0330%** |
| **Macaulay Duration** | **4.4850 years** |
| **Modified Duration** | **4.3749 years** |
| **Convexity** | **22.6021 years²** |

### Rate Shock Performance (+100 bps Shock: $y \to 6.0330\%$)
- **Exact Repriced Value:** `$955.9786`
- **Duration-Only Estimate:** `$954.8714` *(Error: -$1.1072)*
- **Duration + Convexity Estimate:** `$955.9998` *(Error: +$0.0212)*

> **Insight:** Incorporating second-order convexity significantly reduces pricing estimation errors, especially for non-marginal yield shifts ($\ge 100\text{ bps}$). However, for extreme macroeconomic shocks ($>200\text{ bps}$), full cash flow repricing remains essential.

---

## 🛠️ Tech Stack & Dependencies

- **Language:** Python 3.9+
- **Data Manipulation:** `pandas`, `numpy`
- **Numerical Optimization:** `scipy` (`scipy.optimize.brentq`)
- **API & Serialization:** `requests`, `json`
- **Visualization:** `matplotlib`

---

## 🚀 Getting Started

### 1. Clone the repository
```bash
git clone https://github.com/your-username/treasury-bond-valuation.git
cd treasury-bond-valuation
