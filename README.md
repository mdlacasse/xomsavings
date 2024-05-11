# Mean-variance optimization of a portfolio of limited-choice assets
This file uses sequential quadratic programming to solve the Markowitz formulation of asset allocation in a portfolio having the following choice of assets:

- ExxonMobil stock (XOM)
- S&P 500 index tracking assets (^GSPC)
- Dow Jones US Completion Total (^DWCPF)
- MSCI World ex US Market Index (ACWX)
- Bloomberg Aggregate Bond market (AGG)
- Risk free cash represented by 3-month Treasury bills (^IRX)

### Mathematical formulation of the problem
The level of mathematics involved here only requires basic linear algebra, in particular, matrix-vector multiplication, and basic statistics. This problem can be solved through imposing an inequality constraint on the variance, or by imposing a penalty on the variance through the objective function. The example here uses the first approach. The second is described in notebook MPT_4.pynb.

Following Markowitz modern portfolio theory, we consider the variance of a market portfolio consisting of assets allocated with weights stored in a vector $w$, and having a covariance matrix typically represented by $\Sigma$ which is calculated between the time series of these assets. The variance is then expressed as \\[ \sigma^2 = w^T \Sigma w, \\] where $T$ is the transpose operator changing a column vector into a row vector. The square root of the variance, $\sigma$ is the standard deviation that quantifies the volatility. Under this approach, the standard deviation is a measure of risk.

We consider a portfolio which also has a risk-free asset available for investment with a rate of return $r_0$. The rate of return on the market part of the portfolio, i.e., excluding the risk-free asset is \\[ w^T \alpha, \\] where $\alpha$ is a vector containing the average rates of return for each of the $n$ assets in which the portfolio is invested over the period considered. It is just a weighted sum of average rates, where the weights are a fraction of unity.

Let vector $1_n$ be a vector having 1 for all its elements. The objective function that we would like to maximize is the total return of a portfolio that can also invest in a risk-free asset with return $r_0$, \\[f(w) = w^T \alpha + (1 - w^T 1_n)r_0, \\] subject to the variance of the whole portfolio (i.e., considering the risk-free part) \\[ (w^T 1_m)^2\  w^T\Sigma w \le \sigma_o^2,\\] being smaller than a target value $\sigma_o^2$,
and under the condition that we only invest, requiring that $w \ge 0$ element-wise, (i.e., no short position), and no borrowing on our cash position, \\[ w^T 1_n \le 1. \\] It can be observed that if $w=0$, then the portfolio is totally invested in the risk-free part, the variance is 0, and the return is $r_0$.

The desired volatility $\sigma_o$ is specified as the standard deviation on the annual return of the total portfolio, the one containing the risk-free asset.

While the inequality constraint on variance is a 4th-order equation, the problem can still be solved using sequential quadratic programming with the inequality constraints and bounds presented here. For this purpose, we  use the scipy library. We run the optimization using historical daily values divided by periods of one to several months depending on the user selection. The time series for the assets are downloaded from Yahoo finance. For each period, optimal weights are calculated and values of the portfolio's annualized rates of return are computed and stored for displaying in graphs at the end of the calculations. We restrict the weights $w$ to be between 0 and 1. We use the 252 days of trading in the year to convert annual rates to trading-day rates. More details on the computation are given in the comments inserted in the Python code below.

Additional bounds can be imposed on the fraction of investment made in the risk-free asset (`maxCash`) and on holding positions for each market asset (`maxAssetFraction`). The data used are the adjusted daily data at closing (adjusted for splits and dividends). Daily data are grouped in time periods represented in multiples of months (`nMonths`). Choosing 12 months gives an optimization that can re-adjust asset allocation once a year, while choosing 192 months only allows for a single set of asset allocation over the 16 year considered. Note that this is not re-balancing as accounts are implicitly assumed to be always in balance with the chosen asset allocation during each period. Choosing a high number such as 96 months (8 years) gives the historical rate of return from a scenario where one chooses a constant allocation ratio over the first 8 years, and another one for the other 8 following years.

The last thing to select is the desired annualized volatility $\sigma_o$ denoted by `desiredVolatility`. When selecting long-term periods, such as 96 months, one can easily realize that the tolerance for volatility needs to be relaxed in order to achieve higher rates of returns. Interestingly, for the last 16 years, the optimal long-term asset allocation is not the 60/40 stock/bonds portfolio commonly recommended by financial advisors, but rather something else (cash and stocks). While bonds are part of the solution in downturns years, when `nMonths` $\le$ 96, no choice of volatility yielded a 60/40 stocks/bonds portfolio for a 16-year long-term allocation solution. For 2 blocks of 8 years, the solution has only 2 assets: extended stocks/bonds for the first block, but stock/cash allocation for the second half. This echoes what many analysts have said regarding the fate of the 60/40 portfolio wisdom with recent market performance.

It is hoped that this script will help the investor understand the long-term impact of the choices that were made on asset allocation. But as always, past market performance is no guarantee of future performance. Nevertheless, there are still useful lessons to be learned here.

Enjoy!

&copy;  2024 Martin-D. Lacasse

This notebook comes with no guarantee. Use at your own risk.

