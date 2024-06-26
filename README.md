# **Mean-variance optimization of a portfolio with a limited choice of assets**
Did you ever ask yourself what the ultimate rate of return could be on a limited-choice 401k if one could guess the market a little better? Or even perfectly? A related and more practical question is *Can one determine if the past choice of asset allocation is consistent with one's tolerance to risk?* Answering these questions could help better manage current and future choices for asset allocation in a retirement portfolio.

Formulating and answering theses questions is what these notebooks are all about. Following common practice, we consider the volatility of the market as a measure of risk and maintain the volatility of the allocated portfolio below a desired value as a constraint, ensuring that the choice of asset allocation is in line with the desired risk tolerance. The frequency at which changes in asset allocation could be performed can be selected from monthly to yearly, all the way down to only once in the 16-year period that represents the range of historical data considered here and obtained from a public source.

Mathematically, the solution to this problem consists in maximizing the portfolio return under either a variance inequality constraint or a penalty term, and under the constraint of being a long-only portfolio, i.e., a portfolio in which one can only invest in the assets, not short them. This problem has no analytical solution, but it can easily be solved numerically with modern algorithms. 

These notebooks use sequential quadratic programming to solve the Markowitz formulation of asset allocation in a portfolio having the following choice of assets:

- ExxonMobil stock (XOM)
- S&P 500 index tracking assets (^GSPC)
- Dow Jones US Completion Total (^DWCPF)
- MSCI World ex US Market Index (ACWX)
- Bloomberg Aggregate Bond market (AGG)
- Risk free cash represented by 3-month Treasury bills (^IRX)

Ignoring the *balanced* option offered by Voya, these assets correspond to the classes available in the ExxonMobil 401k plan in the US.
For tracking the US Aggregate Bond market, we use the AGG ETF which has an inception year of 2003.
Only growth is considered, inflation is not.

### Mathematical formulation of the problem
The level of mathematics involved here only requires basic linear algebra, in particular, matrix-vector multiplication, and basic statistics. There are two strategies commonly used to solve this problem: it can be solved through imposing an inequality constraint on the variance, or by imposing a penalty term in the objective function. The example shown here uses the first approach. The second strategy is described in notebook MPT-4.pynb.

Following Markowitz modern portfolio theory, we consider the variance of a market portfolio consisting of assets allocated with weights stored in a vector $w$, and having a covariance matrix typically represented by $\Sigma$ which is calculated between the time series of these assets. The variance is then expressed as
```math
\sigma^2 = w^T \Sigma w,
```
where $T$ is the transpose operator changing a column vector into a row vector. The square root of the variance, $\sigma$ is the standard deviation that quantifies the volatility. Under this approach, the standard deviation is a measure of risk.

The rate of return on the market part of the portfolio, i.e., excluding risk-free assets is
```math
w^T \alpha, 
```
where $\alpha$ is a vector containing the average rates of return for each of the $n$ assets in which the portfolio is invested over the period considered. It is just a weighted sum of average rates, where the weights are a fraction of unity.

Let vector $1_n$ be a vector having 1 for all its elements. 
We consider a portfolio which also has a risk-free asset available for investment with a rate of return $r_0$, in which we
can invest the remaining fraction $(1 - w^T 1_n)$ not invested in the market.
The objective function that we would like to maximize is the total return of a portfolio that can also invest in a risk-free asset with return $r_0$, 
```math
f(w) = w^T \alpha + (1 - w^T 1_n)r_0,
```
subject to the variance of the whole portfolio (i.e., considering the risk-free part) 
```math
(w^T 1_m)^2\  w^T\Sigma w \le \sigma_o^2,
```
being smaller than a target value $\sigma_o^2$,
and under the condition that we only invest, requiring that $w \ge 0$ element-wise, (i.e., no short position), and no borrowing on our cash position, 
```math
w^T 1_n \le 1. 
```
It can be observed that if $w=0$, then the portfolio is totally invested in the risk-free part, the variance is 0, and the return is $r_0$.
The desired volatility $\sigma_o$ is specified as the standard deviation on the annual return of the total portfolio, the one containing the risk-free asset.

While the inequality constraint on variance is a 4th-order equation, the problem can still be solved using sequential quadratic programming with the inequality constraints and bounds presented here. For this purpose, we  use the scipy library. We run the optimization using historical daily values divided by periods of one to several months depending on the user selection. The time series for the assets are downloaded from Yahoo finance. For each period, optimal weights are calculated and values of the portfolio's annualized rates of return are computed and stored for displaying in graphs at the end of the calculations. We restrict the weights $w$ to be between 0 and 1. We use the 252 days of trading in the year to convert annual rates to trading-day rates. More details on the computation are given in the comments inserted in the code of the Python notebooks.

Additional bounds can be imposed on the fraction of investment made in the risk-free asset (`maxCash`) and on holding positions for each market asset (`maxAssetFraction`). The data used are the adjusted daily data at closing (adjusted for splits and dividends). Daily data are grouped in time periods represented in multiples of months (`nMonths`). Choosing 12 months gives an optimization that can re-adjust asset allocation once a year, while choosing 192 months only allows for a single set of asset allocation over the 16 year considered. Note that this is not re-balancing as accounts are implicitly assumed to be always in balance with the chosen asset allocation during each period. Choosing a high number such as 96 months (8 years) gives the historical rate of return from a scenario where one chooses a constant allocation ratio over the first 8 years, and another one for the other 8 following years.

The last thing to select is the desired annualized volatility $\sigma_o$ denoted by `desiredVolatility`. When selecting long-term periods, such as 96 months, one can easily realize that the tolerance for volatility needs to be relaxed in order to achieve higher rates of returns. Interestingly, for the last 16 years, the optimal long-term asset allocation is not the 60/40 stock/bonds portfolio commonly recommended by financial advisors, but rather something else (cash and stocks). While bonds are part of the solution in downturns years when `nMonths` $\le$ 96 months, no choice of volatility yielded a 60/40 stocks/bonds portfolio for a 16-year long-term allocation solution. For 2 blocks of 8 years, the solution has only 2 assets: extended stocks/bonds for the first block, but stock/cash allocation for the second half. This echoes what many analysts have said regarding the fate of the 60/40 portfolio wisdom with recent market performance. Exploring this in more details, for the whole period of the last 192 months, the average annual returns of a portfolio of S&P500/bonds compared to a portfolio of S&P500/cash look like the following:

| S&P500 | Bonds | Cash | Return| Volatility|
| ------: |-------:|------:|-------:|-----------:|
| 80% | 20% | -- | 7.7%| 16.3%|
| 80% | -- | 20% | 7.5%| 13.0%|
| 75% | 25% | -- | 7.5%| 15.3%|
| 75% | -- | 25% | 7.3%| 11.4%|
| 70% | 30% | -- | 7.2%| 14.3%|
| 70% | -- | 30% | 7.0%| 10.0%|
| 65% | 35% | -- | 7.0%| 13.3%|
| 65% | -- | 35% | 6.7%|  8.6%|
| 60% | 40% | -- | 6.7%| 12.4%|
| 60% | -- | 40% | 6.4%|  7.3%|

This shows that holding bonds instead of cash was beneficial when considering the last 16 years, but this observation is reversed when only considering the last 8 years:
| S&P500 | Bonds | Cash | Return| Volatility|
| ------:|-------:|------:|-------:|-------:|
| 80% | 20% | -- | 10.7%| 15.0%|
| 80% | -- | 20% | 10.8%| 11.9%|
| 75% | 25% | -- | 10.2%| 14.1%|
| 75% | -- | 25% | 10.4%| 10.4%|
| 70% | 30% | -- | 9.7%| 13.3%|
| 70% | -- | 30% | 9.9%|  9.1%|
| 65% | 35% | -- | 9.2%| 12.4%|
| 65% | -- | 35% | 9.5%|  7.4%|
| 60% | 40% | -- | 8.7%| 11.6%|
| 60% | -- | 40% | 9.0%|  6.7%|


Other fixed asset combinations can be easily explored by properly adjusting the `fixedWeigths` and `myWeights` variables in the **Other Adjustable Parameters** section of the notebook.
These cases use fixed weights on the asset allocations by imposing strict bounds. This is in contrast with an optimization approach where the weights are left to be adjusted to
maximize the total return of the portfolio. As an example, here are the outputs generated when specifying a 12-month period with a constraint volatility of 5%, `maxCash = 0.80` and `maxAssetFraction = 0.85`.

![image](https://github.com/mdlacasse/xomsavings/assets/19365223/01a6c9cc-82e5-48e7-98c7-44652e62f229)

![image](https://github.com/mdlacasse/xomsavings/assets/19365223/29c4dd3e-26f5-4403-8fb5-99c5559f047c)

![image](https://github.com/mdlacasse/xomsavings/assets/19365223/3f24fcc2-2470-454d-aa7d-6342212a8513)



It is hoped that this script will help the investor understand the long-term impact of the choices that were made on asset allocation. But as always, past market performance is no guarantee of future performance. Nevertheless, there are still useful lessons to be learned here.

Enjoy!

&copy;  2024 Martin-D. Lacasse

This notebook comes with no guarantee. Use at your own risk.

## Required software
These notebooks are jupyter notebooks that require the installation of Anaconda, which is a free implementation of a Python environment. Installation instructions can be found on the Anaconda [website](www.anaconda.com).

