# quantitative-portfolio-analytics-engine
End-to-end quantitative portfolio optimization, risk analytics, and walk-forward backtesting framework in Python
# Quantitative Portfolio Analytics Engine

An end-to-end quantitative portfolio optimization, risk analytics, and backtesting framework built in Python.

This project examines a practical question in quantitative portfolio management:

> **Can a constrained mean-variance optimization strategy produce robust out-of-sample risk-adjusted performance after accounting for portfolio drift and transaction costs, and how does it compare with simpler benchmarks?**

The analysis progresses from Modern Portfolio Theory and Monte Carlo simulation to numerical optimization, tail-risk modeling, out-of-sample testing, and a realistic rolling walk-forward backtest.

---

## Project Overview

The project analyzes a diversified portfolio of eight U.S. equities:

* AAPL — Apple
* AMZN — Amazon
* JNJ — Johnson & Johnson
* JPM — JPMorgan Chase
* MSFT — Microsoft
* NVDA — NVIDIA
* PG — Procter & Gamble
* XOM — Exxon Mobil

SPY is used as the market benchmark for performance comparison and beta/alpha analysis.

Historical market data is used to estimate expected returns, volatility, covariance, and correlation before constructing and testing multiple portfolio strategies.

---

## Methodology

The project follows a progression from theoretical portfolio construction to increasingly realistic investment testing.

### 1. Portfolio Mathematics & Diversification

Historical daily returns are used to estimate:

* Annualized expected returns
* Annualized volatility
* Covariance matrix
* Correlation matrix
* Portfolio expected return
* Portfolio variance and volatility
* Diversification benefit

Portfolio return and risk are calculated using:

$$
E[R_p] = w^T\mu
$$

and

$$
\sigma_p = \sqrt{w^T\Sigma w}
$$

The equal-weight portfolio demonstrates how covariance between assets can reduce total portfolio risk relative to the weighted average volatility of the individual securities.

---

### 2. Monte Carlo Portfolio Simulation

50,000 random long-only portfolios are generated to explore the feasible risk-return opportunity set.

For each portfolio, the framework calculates:

* Expected annual return
* Annualized volatility
* Sharpe ratio
* Portfolio weights

The simulation identifies approximate minimum-volatility and maximum-Sharpe portfolios and provides a visual representation of the portfolio opportunity set.

---

### 3. Mean-Variance Optimization

Numerical optimization using SciPy's Sequential Least Squares Programming (`SLSQP`) algorithm is used to directly solve for:

* Minimum-variance portfolio
* Maximum-Sharpe portfolio

The optimization is subject to:

$$
\sum w_i = 1
$$

and long-only constraints:

$$
0 \leq w_i \leq 1
$$

The optimized maximum-Sharpe portfolio produced approximately:

| Metric          | Result |
| --------------- | -----: |
| Expected Return | 30.72% |
| Volatility      | 16.56% |
| Sharpe Ratio    |  1.855 |

The optimized minimum-variance portfolio produced approximately:

| Metric          | Result |
| --------------- | -----: |
| Expected Return | 16.06% |
| Volatility      | 12.42% |

An efficient frontier is then constructed by minimizing portfolio volatility across a range of target returns.

---

## Risk Analytics

The optimized portfolio is evaluated using multiple risk measures rather than relying solely on standard deviation.

### Value at Risk & Expected Shortfall

Three approaches are implemented:

1. Historical simulation
2. Parametric normal model
3. Monte Carlo simulation using correlated asset returns

For a hypothetical **$1 million portfolio**, selected one-day results include:

| Method             |  95% VaR |  99% VaR |   95% ES |   99% ES |
| ------------------ | -------: | -------: | -------: | -------: |
| Historical         |  $15,646 |  $26,214 |  $22,460 |  $34,060 |
| Parametric Normal  |  $15,938 |  $23,047 |  $20,297 |  $26,582 |
| Monte Carlo Normal | ~$15,971 | ~$23,195 | ~$20,329 | ~$26,550 |

The parametric and Monte Carlo normal estimates are relatively similar because both rely on normal-distribution assumptions.

Historical Expected Shortfall becomes substantially larger in the extreme tail, illustrating how a normal model can underestimate severe downside observations.

### Additional Risk Measures

The project also evaluates:

* Maximum Drawdown
* Sharpe Ratio
* Sortino Ratio
* Calmar Ratio

These metrics provide complementary perspectives on volatility, downside risk, and peak-to-trough portfolio losses.

---

## Benchmark & Factor Analysis

The optimized portfolio is compared with SPY and an equal-weight portfolio.

The analysis includes:

* Beta
* Alpha
* Regression analysis
* R-squared
* Cumulative wealth comparison

The optimized portfolio exhibited a beta of approximately **0.72** relative to SPY during the full historical sample.

---

## Out-of-Sample Testing

A major objective of the project is to distinguish between strong historical optimization results and strategies that remain effective on unseen data.

The dataset is therefore separated into estimation and out-of-sample periods.

An unconstrained optimized portfolio initially demonstrates poor out-of-sample stability despite strong in-sample results.

This illustrates a fundamental problem in mean-variance optimization:

> **Optimal historical weights can be highly sensitive to estimation error in expected returns and covariance.**

---

## Constrained Optimization

To reduce portfolio concentration, a maximum position constraint of **25% per asset** is introduced.

This acts as a form of regularization by preventing the optimizer from allocating excessive capital to securities with unusually attractive historical estimates.

The constrained portfolio demonstrated substantially more stable out-of-sample behavior than the unconstrained optimization.

---

## Walk-Forward Backtesting

A rolling walk-forward framework is implemented to better approximate how an investment strategy could operate in practice.

The strategy uses:

* **3-year rolling estimation window**
* **Quarterly rebalancing**
* **25% maximum position size**
* **Long-only portfolio constraints**
* **No look-ahead bias**

At each rebalance date:

1. Only historical data available at that point is used.
2. Expected returns and covariance are re-estimated.
3. Portfolio weights are optimized.
4. The resulting portfolio is held until the next rebalance date.
5. The process is repeated through the out-of-sample period.

---

## Portfolio Drift & Turnover

The backtest incorporates natural portfolio-weight drift between rebalancing dates.

Rather than assuming portfolio weights remain constant every day, each asset position grows according to its realized return.

This allows the framework to calculate more realistic:

* End-of-period portfolio weights
* Required rebalance trades
* Portfolio turnover

Average drift-adjusted turnover was approximately **12.43% per rebalance**, with total turnover of approximately **111.85%** across the tested rebalancing periods.

---

## Transaction Costs

Transaction costs are incorporated using portfolio turnover.

The base case assumes:

**10 basis points per dollar traded**

Sensitivity analysis is also performed at:

* 5 bps
* 10 bps
* 25 bps

Because portfolio turnover remained relatively moderate, transaction costs had a limited effect on final portfolio wealth during the test period.

---

## Final Out-of-Sample Results

After incorporating rolling optimization, portfolio drift, and transaction costs:

| Strategy         |   CAGR | Annual Return | Volatility | Sharpe | Sortino | Max Drawdown | Calmar | Ending Wealth |
| ---------------- | -----: | ------------: | ---------: | -----: | ------: | -----------: | -----: | ------------: |
| Equal Weight     | 24.54% |        23.05% |     14.86% |   1.55 |    2.33 |      -17.57% |   1.40 |       $16,878 |
| Net Walk-Forward | 20.71% |        20.04% |     15.64% |   1.28 |    1.89 |      -18.32% |   1.13 |       $15,664 |
| SPY              | 18.64% |        18.40% |     16.16% |   1.14 |    1.69 |      -18.76% |   0.99 |       $15,034 |

Initial capital: **$10,000**

---

## Key Finding

The constrained walk-forward optimization strategy outperformed SPY during the out-of-sample period on both absolute and risk-adjusted performance.

However, the **equal-weight portfolio produced the strongest overall results**.

This is one of the most important findings of the project.

The in-sample efficient frontier suggested that optimized portfolios dominated the equal-weight allocation. Once the strategy was evaluated using unseen data, rolling estimation, portfolio drift, and transaction costs, the simple equal-weight strategy ultimately performed better.

This demonstrates a central challenge in quantitative portfolio management:

> **A mathematically optimal portfolio is only as reliable as the estimates used to construct it.**

Expected returns and covariance relationships are estimated from noisy historical data and can change through time. Additional optimization complexity therefore does not guarantee superior out-of-sample performance.

---

## Technologies Used

* Python
* NumPy
* pandas
* SciPy
* Matplotlib
* yfinance
* Jupyter Notebook
* Modern Portfolio Theory
* Monte Carlo Simulation
* Numerical Optimization
* Statistical Risk Modeling
* Walk-Forward Backtesting

---

## Repository Structure

```text
quantitative-portfolio-analytics-engine/
│
├── Quantitative_Portfolio_Analytics_Engine.ipynb
├── README.md
└── requirements.txt
```

The Jupyter Notebook contains the complete analysis, calculations, visualizations, optimization procedures, and backtesting framework.

---

## Limitations

This project is intended as a quantitative research and portfolio analytics exercise rather than a production trading system.

Important limitations include:

* Expected returns are estimated from historical data.
* Covariance relationships may change over time.
* Normal-distribution models may underestimate extreme market events.
* Transaction costs are modeled using simplified basis-point assumptions.
* Taxes, market impact, liquidity constraints, and bid-ask spread dynamics are not fully modeled.
* The investment universe is limited to eight equities.
* Results are sensitive to the selected estimation window, rebalance frequency, and portfolio constraints.

Future extensions could incorporate alternative expected-return models, covariance shrinkage, factor models, volatility targeting, additional asset classes, and more sophisticated transaction-cost modeling.

---

## Disclaimer

This project is for educational and research purposes only and does not constitute investment advice. Historical and simulated performance does not guarantee future results.
