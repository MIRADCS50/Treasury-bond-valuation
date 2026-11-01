# 📈 U.S. Treasury Note Valuation & Interest Rate Sensitivity Analysis

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
[![Data Source](https://img.shields.io/badge/Data-TreasuryDirect%20API-orange.svg)](https://www.treasurydirect.gov/)

An end-to-end quantitative fixed-income project that retrieves official U.S. Treasury auction data through the **TreasuryDirect API**, constructs semiannual bond cash flows, calculates the implied Yield to Maturity (YTM), and evaluates interest rate risk using **Macaulay Duration, Modified Duration, and Convexity**.

The project also compares first-order and second-order Taylor approximations of bond price changes against exact cash flow repricing under multiple yield shock scenarios.

---

## 📌 Project Overview

This repository demonstrates the practical application of fixed-income valuation and interest rate risk measurement using U.S. Treasury securities.

Key components include:

1. **Automated Data Retrieval:** Queries the TreasuryDirect REST API (`/securities/search`) to retrieve official security and auction information.
2. **Bond Selection & Data Processing:** Identifies the latest active 5-Year Treasury Note and extracts its contractual and auction parameters, including CUSIP, issue date, maturity date, coupon rate, High Yield, and auction price.
3. **Bond Valuation Engine:** Constructs semiannual cash flow schedules based on a $1,000 face value and uses numerical root-finding (`scipy.optimize.brentq`) to calculate the implied YTM.
4. **Interest Rate Risk Measurement:** Calculates Macaulay Duration, Modified Duration, and Convexity.
5. **Sensitivity and Scenario Analysis:** Evaluates bond price changes under multiple yield shocks ranging from -100 to +300 basis points.
6. **Approximation Accuracy Assessment:** Compares Duration-only and Duration + Convexity estimates against exact repricing to assess the accuracy of each approach.

---

## 🧮 Mathematical Framework

### 1. Bond Pricing and Yield to Maturity

For a fixed-rate bond with face value \(M\), periodic coupon payment \(C\), \(N\) remaining payment periods, and nominal annual yield \(y\) compounded \(m\) times per year:

$$
P=\sum_{t=1}^{N}\frac{CF_t}{(1+y/m)^t}
$$

where:

- \(P\): Bond price
- \(CF_t\): Cash flow at period \(t\)
- \(y\): Annual Yield to Maturity
- \(m\): Coupon payment frequency

For this project, \(m=2\), representing semiannual coupon payments.

The implied YTM is obtained numerically by solving:

$$
P(y)-P_{\text{auction}}=0
$$

using the Brent root-finding algorithm.

### 2. Macaulay and Modified Duration

**Macaulay Duration** measures the present-value-weighted average time to receive a bond's cash flows:

$$D_{\text{Mac}}=
\frac{\sum_{t=1}^{N}\frac{t}{m}PV(CF_t)}
{P}
$$

**Modified Duration** measures the approximate percentage price sensitivity to a change in annual yield:

$$
D_{\text{Mod}}=
\frac{D_{\text{Mac}}}{1+y/m}
$$
For a relatively small yield change:

$$
\frac{\Delta P}{P}
\approx -D_{\text{Mod}}\Delta y
$$

### 3. Convexity

Convexity captures the curvature of the bond price-yield relationship and improves the accuracy of duration-based price estimates.

For regular coupon periods and nominal annual yield compounded \(m\) times per year:

$$
C_{\text{Bond}}=
\frac{1}{P m^2}
\sum_{t=1}^{N}
\frac{CF_t\,t(t+1)}
{(1+y/m)^{t+2}}
$$

### 4. Taylor Series Approximation

The second-order approximation combines Modified Duration and Convexity:

$$
\frac{\Delta P}{P}
\approx
-D_{\text{Mod}}\Delta y
+\frac{1}{2}C_{\text{Bond}}(\Delta y)^2
$$

The project evaluates both approximations against exact repricing to quantify their estimation errors.

---

## 📊 Key Results & Empirical Findings

**Analyzed Security:** 5-Year U.S. Treasury Note

| Security Attribute | Value |
|:---|---:|
| CUSIP | `91282CRN3` |
| Issue Date | 2026-09-30 |
| Maturity Date | 2031-09-30 |
| Face Value | $1,000 |
| Annual Coupon Rate | 5.00% |
| Coupon Frequency | Semiannual |

### Bond Valuation and Risk Metrics

| Metric | Result |
|:---|---:|
| Auction Price | **$998.5571** |
| High Yield (Stop-out YTM) | **5.0330%** |
| Macaulay Duration | **4.4850 years** |
| Modified Duration | **4.3749 years** |
| Convexity | **22.6021 years²** |

**Auction Pricing Note:** The reported auction price corresponds to the High Yield (5.033%), rather than the Median Yield (4.950%). In a single-price Treasury auction, successful competitive bidders pay the price determined by the stop-out yield.

### Interest Rate Sensitivity: +100 bps

A 100-basis-point increase in YTM is applied, from 5.0330% to 6.0330%.

| Valuation Method | Estimated Price | Estimation Error |
|:---|---:|---:|
| Exact Repricing | **$955.9786** | — |
| Duration Only | $954.8714 | -$1.1072 |
| Duration + Convexity | $955.9998 | +$0.0212 |

The second-order approximation substantially reduces the estimation error compared with the Duration-only approach.

### Multiple Yield Shock Scenarios

The model also evaluates the following scenarios:

| Yield Shock | Exact Repriced Price | Duration Error | Duration + Convexity Error |
|---:|---:|---:|---:|
| -100 bps | $1,043.39 | -$1.1504 | -$0.0219 |
| +50 bps | — | — | — |
| +100 bps | $955.98 | -$1.1072 | +$0.0213 |
| +200 bps | $915.53 | -$4.3464 | +$0.1675 |
| +300 bps | $877.10 | -$9.5994 | +$0.5569 |

*Note: The +50 bps scenario is omitted from this table because its numerical output has not yet been reported.*

**Key Insight:** Modified Duration provides a useful first-order estimate of bond price sensitivity, while Convexity accounts for the curvature of the price-yield relationship. Across the tested scenarios, incorporating Convexity considerably improves the accuracy of the estimated bond price. However, approximation errors increase as yield shocks become larger, making exact cash flow repricing the more reliable valuation approach.

---

## 📈 Visual Analysis

The project includes a Price-Yield Relationship chart comparing:

- **Exact Price:** Bond prices calculated through cash flow discounting.
- **Duration Approximation:** First-order linear approximation.
- **Duration + Convexity:** Second-order approximation incorporating price-yield curvature.

This visualization illustrates the inverse and convex relationship between bond prices and yields.

---

## 🛠️ Tech Stack & Dependencies

| Category | Tools |
|:---|:---|
| Programming Language | Python 3.9+ |
| Data Manipulation | pandas, NumPy |
| Numerical Optimization | SciPy (`brentq`) |
| API Requests | requests |
| Data Serialization | json |
| Data Visualization | Matplotlib |

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/treasury-bond-valuation.git
cd treasury-bond-valuation
```

Replace `your-username` with your GitHub username.

### 2. Install Dependencies

```bash
pip install pandas numpy scipy requests matplotlib
```

### 3. Run the Notebook

Open the Jupyter Notebook:

```bash
jupyter notebook
```

Run the notebook cells sequentially to retrieve the Treasury data, calculate valuation and risk metrics, and reproduce the sensitivity analysis.

---

## ⚠️ Model Assumptions and Limitations

The valuation framework uses the following simplifying assumptions:

- Cash flows are modeled using equal semiannual periods.
- Settlement is assumed to occur on the issue date.
- Accrued interest is assumed to be zero.
- Exact contractual coupon payment dates are not explicitly modeled.
- The Treasury Actual/Actual day-count convention is not incorporated.
- Yield is assumed to change while the bond's contractual cash flows remain unchanged.

Consequently, the calculated prices and risk measures represent a simplified theoretical valuation rather than a full settlement-date Treasury pricing implementation.

---

## 📚 Data Source

- [U.S. TreasuryDirect](https://www.treasurydirect.gov/)
- [TreasuryDirect Securities Search API](https://www.treasurydirect.gov/)

---

## 📄 License

This project is licensed under the MIT License.
