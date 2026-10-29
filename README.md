# Stock Returns and the Bell Curve

An educational Jupyter notebook that uses real stock price data from Yahoo Finance to show why daily stock returns are often modeled as bell-curve (normal) distributed, and what that model does and does not tell a quant about how far a stock can move.

---

## Table of Contents

- [Overview](#overview)
- [Guiding Questions](#guiding-questions)
- [What You Will Learn](#what-you-will-learn)
- [Notebook Structure](#notebook-structure)
- [Requirements](#requirements)
- [Installation](#installation)
- [How to Run](#how-to-run)
- [Changing the Ticker](#changing-the-ticker)
- [Methodology](#methodology)
- [Key Results to Look For](#key-results-to-look-for)
- [Limitations of the Model](#limitations-of-the-model)
- [Reproducibility](#reproducibility)
- [Project Layout](#project-layout)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)
- [Disclaimer](#disclaimer)

---

## Overview

Stock prices move every trading day, and the size of those moves is one of the most important inputs to pricing, risk management, and portfolio construction. A common starting point in quantitative finance is to assume that daily log returns follow a normal (bell-curve) distribution. This notebook walks through that assumption step by step, using five years of real adjusted closing prices downloaded with `yfinance`.

Every section pairs a short plain-English explanation with code that prints or plots something you can interpret. The notebook runs top to bottom in Google Colab or a local Jupyter environment.

---

## Guiding Questions

1. **Why do returns look bell-shaped?** We compare the histogram of real daily returns against a fitted normal curve and measure asymmetry and tail weight.
2. **What does the bell curve let us say about how far a stock can move?** We use the normal model to simulate prices, compute probabilities of large moves, and then check those probabilities against the real data.

---

## What You Will Learn

- How to download and clean adjusted price data with `yfinance`
- How to compute and interpret daily log returns
- How to test whether returns look normal, using skewness and excess kurtosis
- Why volatility scales with the square root of time (the √252 rule)
- How the 68-95-99.7 rule applies to real returns
- How to simulate price paths with a normal model using a fixed random seed
- How to compute analytic and simulated probabilities of large one-year moves
- Where the normal model breaks down: fat tails, volatility clustering, and skew

---

## Notebook Structure

The notebook has 10 labeled sections:

| # | Section | What It Does |
|---|---------|--------------|
| 1 | Introduction and Goals | States the two guiding questions |
| 2 | Setup and Imports | Imports libraries, sets the seed, prints versions |
| 3 | Downloading Real Stock Data | Downloads 5 years of daily adjusted closes |
| 4 | Computing Daily Log Returns | Calculates and plots log returns |
| 5 | Testing for the Bell Curve | Histogram vs. fitted normal, skewness, excess kurtosis |
| 6 | Volatility and the √252 Rule | Annualizes volatility and shows square-root-of-time scaling |
| 7 | The 68-95-99.7 Rule Applied to Returns | Compares real band frequencies to the normal model |
| 8 | Simulating a Stock | Simulates 10,000 one-year price paths |
| 9 | Final Price Distribution | Computes probabilities of ±16%, ±32%, ±48% moves |
| 10 | Interpreting the Results | Compares model vs. real data and discusses limitations |

---

## Requirements

- Python 3.9 or newer
- [`numpy`](https://numpy.org/)
- [`pandas`](https://pandas.pydata.org/)
- [`matplotlib`](https://matplotlib.org/)
- [`scipy`](https://scipy.org/)
- [`yfinance`](https://github.com/ranaroussi/yfinance)
- Jupyter Notebook, JupyterLab, or Google Colab

No other third-party packages are used.

---

## Installation

**Google Colab:** Open the notebook in Colab. NumPy, pandas, matplotlib, and SciPy are preinstalled. If `yfinance` is missing, run:

```bash
pip install yfinance
```

**Local environment:**

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
python -m venv .venv
source .venv/bin/activate        # On Windows: .venv\Scripts\activate
pip install numpy pandas matplotlib scipy yfinance jupyter
jupyter notebook stock_returns_bell_curve.ipynb
```

---

## How to Run

1. Open the notebook in Colab or Jupyter.
2. Run all cells from top to bottom (**Runtime → Run all** in Colab, or **Cell → Run All** in Jupyter).
3. Read the markdown explanation before each code cell, then review the printed output and plot.

Running out of order can cause errors, because later sections depend on variables defined earlier (for example, `prices` and `log_ret`).

---

## Changing the Ticker

The default ticker is `SPY`, an S&P 500 exchange-traded fund. To analyze a different stock, edit this line in Section 3:

```python
TICKER = "SPY"   # try "AAPL", "MSFT", "TSLA", or an index such as "^GSPC"
```

You can also change the look-back period by editing `YEARS`. Re-run the notebook from Section 3 onward after making changes.

---

## Methodology

**Returns.** Daily log return is defined as `ln(P_t / P_{t-1})`. Log returns add across time, which makes multi-day calculations straightforward.

**Normal model.** Daily returns are assumed to be independent draws from a normal distribution with mean `μ` and standard deviation `σ`, both estimated from the historical sample.

**Annualization.** Under independence, variance grows linearly with time, so annual volatility is `σ_daily × √252`.

**Simulation.** For each of 10,000 paths, 252 daily returns are drawn from `N(μ, σ²)` with `np.random.seed(42)`. Cumulative sums of log returns give log prices, which are exponentiated and scaled by the starting price.

**Probabilities.** The sum of 252 normal daily returns is itself normal with mean `252μ` and standard deviation `σ√252`. This allows exact probabilities of large moves via `scipy.stats.norm`, which are compared against the simulation.

**Empirical check.** Overlapping 252-day windows of real data give a rolling estimate of how often large one-year moves occurred.

---

## Key Results to Look For

- **Excess kurtosis above 0** in Section 5 indicates fatter tails than the normal distribution.
- **Negative skewness** in Section 5 indicates that large drops are more common than large rallies.
- **Analytic and simulated probabilities agree** in Section 9, which confirms the simulation is correct.
- **Real-to-model ratios above 1** in Section 10 indicate that the real market produced large yearly moves more often than the normal model predicts.

---

## Limitations of the Model

The normal model is a useful approximation, but it has known weaknesses:

- **Fat tails:** Extreme daily moves occur more often than a normal distribution predicts.
- **Volatility clustering:** Calm and turbulent periods come in bunches, so volatility is not constant over time. The notebook assumes constant volatility.
- **Skewness:** Crashes tend to be sharper than rallies.
- **Short sample:** Five years of data includes only a few independent yearly observations, so the empirical probabilities in Section 10 are illustrative.
- **Independence:** The model assumes returns are independent from day to day, which is approximately true but not exactly.

Practitioners often combine normal models with fat-tailed distributions (such as Student's t), GARCH-type volatility models, and stress tests.

---

## Reproducibility

- Random seed is fixed at `42` in every section that simulates.
- Library versions are printed in Section 2. Record them when sharing results.
- Yahoo Finance data can be revised (for example, after corporate actions), so results may differ slightly if you re-download data at a later date.

---

## Project Layout

```
.
├── README.md
├── stock_returns_bell_curve.ipynb
└── requirements.txt
```

Example `requirements.txt`:

```
numpy
pandas
matplotlib
scipy
yfinance
jupyter
```

---

## Troubleshooting

**`yfinance` returns an empty DataFrame or raises a rate-limit error.** Yahoo Finance sometimes throttles requests. Wait a few minutes and re-run Section 3. Check that the ticker symbol is valid on finance.yahoo.com.

**`ModuleNotFoundError: No module named 'yfinance'`.** Run `pip install yfinance` and restart the kernel.

**Plot style error with `seaborn-v0_8-whitegrid`.** This style name requires matplotlib 3.6 or newer. Upgrade with `pip install --upgrade matplotlib`, or replace the style line with `plt.style.use("ggplot")`.

**`MultiIndex` errors on the `Close` column.** Recent `yfinance` versions return multi-level column headers. Section 3 handles this with `.squeeze()`. If you modified that cell, make sure you still select a single series.

---

## Contributing

Suggestions and fixes are welcome. Please open an issue to discuss larger changes before submitting a pull request. Keep the educational tone and the restriction to the five approved libraries.

---

## License

Choose a license before publishing. The MIT License is a common choice for educational code:

```
MIT License. See LICENSE file for details.
```

---

## Disclaimer

This project is for educational purposes only. It is not investment advice, a recommendation to buy or sell any security, or a guarantee of future results. Models are simplifications of markets and can fail, especially during periods of stress. Consult a qualified professional before making financial decisions.
