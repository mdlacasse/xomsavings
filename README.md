# **Mean-variance optimization of a portfolio with a limited choice of assets**
Did you ever ask yourself what the ultimate rate of return could be on a limited-choice 401k if one could guess the market a little better? Or even perfectly? A related and more practical question is Can one determine if the past choice of asset allocation is consistent with one's tolerance to risk? Answering these questions could help better manage current and future choices for asset allocation in a retirement portfolio.

Formulating and answering theses questions is what this script is all about. Following common practice, it considers the volatility of the market as a measure of risk and maintains the volatility of the allocated portfolio below a desired value as a constraint, ensuring that the choice of asset allocation is in line with the desired risk tolerance. The frequency at which changes in asset allocation could be performed can be selected from monthly to yearly, all the way down to only once in the 16-year period that represents the range of historical data considered here and obtained from a public source.

Mathematically, the solution to this problem consists in maximizing the portfolio return under a variance inequality constraint and a long-only portfolio, i.e., a portfolio in which one can only invest in the assets, not short them. This problem has no analytical solution, but it can easily be solved numerically with modern algorithms. 

This file uses sequential quadratic programming to solve the Markowitz formulation of asset allocation in a portfolio having the following choice of assets:

- ExxonMobil stock (XOM)
- S&P 500 index tracking assets (^GSPC)
- Dow Jones US Completion Total (^DWCPF)
- MSCI World ex US Market Index (ACWX)
- Bloomberg Aggregate Bond market (AGG)
- Risk free cash represented by 3-month Treasury bills (^IRX)

Ignoring the *balanced* option offered by Voya, these assets correspond to the classes available in the ExxonMobil 401k plan in the US.
Only growth is considered, inflation is not.
For tracking the US Aggregate Bond market, we use the AGG ETF which has an inception year of 2003.

### Mathematical formulation of the problem
The level of mathematics involved here only requires basic linear algebra, in particular, matrix-vector multiplication, and basic statistics. Thee are two strategies commonly used to solve this problem: it can be solved through imposing an inequality constraint on the variance, or by imposing a penalty term in the objective function. The example shown here uses the first approach. The second is described in notebook MPT_4.pynb.

Following Markowitz modern portfolio theory, we consider the variance of a market portfolio consisting of assets allocated with weights stored in a vector $w$, and having a covariance matrix typically represented by $\Sigma$ which is calculated between the time series of these assets. The variance is then expressed as
```math
\sigma^2 = w^T \Sigma w,
```
where $T$ is the transpose operator changing a column vector into a row vector. The square root of the variance, $\sigma$ is the standard deviation that quantifies the volatility. Under this approach, the standard deviation is a measure of risk.

The rate of return on the market part of the portfolio, i.e., excluding the risk-free asset is
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

While the inequality constraint on variance is a 4th-order equation, the problem can still be solved using sequential quadratic programming with the inequality constraints and bounds presented here. For this purpose, we  use the scipy library. We run the optimization using historical daily values divided by periods of one to several months depending on the user selection. The time series for the assets are downloaded from Yahoo finance. For each period, optimal weights are calculated and values of the portfolio's annualized rates of return are computed and stored for displaying in graphs at the end of the calculations. We restrict the weights $w$ to be between 0 and 1. We use the 252 days of trading in the year to convert annual rates to trading-day rates. More details on the computation are given in the comments inserted in the Python code below.

Additional bounds can be imposed on the fraction of investment made in the risk-free asset (`maxCash`) and on holding positions for each market asset (`maxAssetFraction`). The data used are the adjusted daily data at closing (adjusted for splits and dividends). Daily data are grouped in time periods represented in multiples of months (`nMonths`). Choosing 12 months gives an optimization that can re-adjust asset allocation once a year, while choosing 192 months only allows for a single set of asset allocation over the 16 year considered. Note that this is not re-balancing as accounts are implicitly assumed to be always in balance with the chosen asset allocation during each period. Choosing a high number such as 96 months (8 years) gives the historical rate of return from a scenario where one chooses a constant allocation ratio over the first 8 years, and another one for the other 8 following years.

The last thing to select is the desired annualized volatility $\sigma_o$ denoted by `desiredVolatility`. When selecting long-term periods, such as 96 months, one can easily realize that the tolerance for volatility needs to be relaxed in order to achieve higher rates of returns. Interestingly, for the last 16 years, the optimal long-term asset allocation is not the 60/40 stock/bonds portfolio commonly recommended by financial advisors, but rather something else (cash and stocks). While bonds are part of the solution in downturns years, when `nMonths` $\le$ 96, no choice of volatility yielded a 60/40 stocks/bonds portfolio for a 16-year long-term allocation solution. For 2 blocks of 8 years, the solution has only 2 assets: extended stocks/bonds for the first block, but stock/cash allocation for the second half. This echoes what many analysts have said regarding the fate of the 60/40 portfolio wisdom with recent market performance.

It is hoped that this script will help the investor understand the long-term impact of the choices that were made on asset allocation. But as always, past market performance is no guarantee of future performance. Nevertheless, there are still useful lessons to be learned here.

Enjoy!

&copy;  2024 Martin-D. Lacasse

This notebook comes with no guarantee. Use at your own risk.

