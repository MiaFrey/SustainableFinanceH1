# Sustainable Finance - Homework 1
## Introduction
The objectives are the following:

- Evaluate the impact of diversification in portfolio construction
- Build portfolios of stocks based on the mean-variance criterion (effcient frontier)
- Evaluate the effect of imposing restriction on the set of assets

Our Group was asked to look at **European firms with available scope 1 to 3 emissions** (Trucost).

## Data Cleaning
This section aims to briefly explain the data processing part of our code to shed some light on the format of the data we are working with and which securities we targeted for the below analysis.

We are working on the Trucost dataset, which provides us with key metrics of different securities, from December 1999 to December 2020. "Securities are fungible, negotiable financial instruments that hold some type of monetary value" [Investopedia, 2021]. These could be in the form of stocks, bonds, ETFs, or other financial instruments. These securities are identifiable by their ISIN (International Securities Identification Number), provided in the data sets. The ISIN is used as the
unique identifier (instead of the NAME), as certain securities could share the same name, but have different ISINs. Our mission is to identify the European securities with available scope 1, 2 and 3 emissions.

To identify these firms of interest, we merge the scope 1-3 emission data sets with the regional data on ISIN and drop any rows for which the region is different from Europe (’EUR’). We then collect the list of securities’ ISINs for each scope and take the intersection of securities for which we have data in all three scopes (inner join). In our case, the securities are already the same (without duplicates), so there is no need to drop further data. We thus start our analysis with
2’739 different securities (sample size).

For the formatting of our data, we decided to import and transpose the data sets of interest, while only keeping the securities found above. That way, we have the rows as our time indicator (per month, reformatted from string to date type) and the columns represent the securities (ISIN). We saved the cleaned data into new excel files locally, to reduce the compilation time when running the code repeatedly. Lastly, once having calculated the returns, we drop the month December 1999 in our imported data sets, to remain consistent, as this month is "lost" upon calculating the returns. Except for the data set size, for which we use the December 1999 data in later exercises (Figure 1). Further, we make sure to also replace infinite values with NaNs at this point, as these undefined values else would create noise.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183402126-c9d54b9c-b753-4f0c-89d7-7c79da1a4b5f.png" width="800">
</p>

```
Figure 1: Example of the cleaned dataset for the returns
```

## Exercise 1

- **Compute and report the annualized average return and annualized volatility for
all individual assets.**

Given the monthly prices (including dividends) in our data set, we calculated the monthly returns such that:
<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183404380-e3fc86fc-410e-4f91-9f7a-4717b8c07bdb.png" width="800">
</p>


for m = (1, 2, ..., 12)

Then we computed the average return by taking the mean of the monthly return per company. Finally, to calculate the annualized average return we multiply by 12 because we suppose that the distribution for each month is the same. For the analysis of this exercise, we discard the NaNs to have a fair analysis. In other words, only companies with values for all the years are considered such that they are not “in competition” with a company that only has for instance three years, but of high performance. Our data set thus decreases to 1371 firms.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183467852-ca738074-9139-4b6a-942f-4a1c385b253e.png" width="400">
</p>

```
Figure 2: Annualized average return per company
```

The data points on Figure 2 represent the securities. The higher the return the better. We can observe that the annualized average return of many of them is positive. Only a few of them have a negative value. As mentioned in the summary statistics table (Figure 4), the average of all those returns is 0.13. Furthermore, we can observe two outliers. We will go more into depth about these in exercise 3. For the volatility we used the built-in Python function of the standard deviation and multiplied this by the square root of 12 to obtain the annualized volatility per individual asset. This is one of the best ways to measure the risk because it is easier to interpret than the variance [Rockinger, 2020]. Thus, the lower the volatility the better, since it is synonymous with a smaller risk.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183468156-9c5c1397-d338-48de-8fe9-ba941660f41d.png" width="400">
</p>

```
Figure 3: Annualized average volatility per company
```

We find that the graph of the volatility (Figure 3) looks similar to the one of the return. This
shows the highly positive correlation between these two measures (risk and return) which will be
discussed in point 1.2. Again, the two outliers can be nicely observed with high volatility. This
already hints at the fact that their large returns probably stem from a one-time occurrence that
greatly increases the average, since this metric is not robust to extremes. The average of all the
annualized volatilities is 0.40.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183405022-d9e50f80-cf8e-4aa4-a860-df872dfb3aa0.png" width="800">
</p>

```
Figure 4: Summary statistics
```

- **Compute the correlation between individual returns and volatility and comment
on the observed correlation.**

The computed correlation between the individual annualized average returns and volatility is 0.79. It is strongly positive and almost equal to 1. This is in line with the theory because we know that the return acts as compensation for the risk taken. Meaning the higher the volatility, the higher the risk ought to be. If this were not the case, then it would be easy to invest with small risk and gain high returns, so we would all be rich. At this stage, it is also important to look at the correlation between the securities, especially since portfolios will be created in the following exercises. The higher the correlation between the assets, the more difficult it is to benefit from diversification. If the assets are perfectly negatively correlated, then there exists a portfolio such that the expected risk is zero. In reality, this is rarely the case. If the portfolio is big, then the variance terms are diversified away but not the covariance terms. Hence, diversification can eliminate some, but not all the risk of individual securities. There is a systematic risk (or market risk) that defines the lower bound of our portfolio risk.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183405306-9ce6061c-479c-4a87-b81e-0beeffca8cd6.png" width="800">
</p>

```
Figure 5: Heatmap of the correlation between the different assets
```

In the heatmap (Figure 5), we observe that our assets are mostly positively correlated.
But there are not perfectly positive and we have some negative correlation coefficients as well,
indicating that we can benefit from diversification (although not fully).

## Exercise 2

- **Form an equally-weighted and value-weighted portfolio with monthly rebalancing.**

The equally-weighted portfolio is one where all constituent firms have an equal value [Fahlenbrach, 2022]. To form it, each month is taken in turn and the securities which are traded during this month (total_traded) are kept (except NaN). These are the securities used to build our monthly portfolio. Then a weight of $w_i = \frac{1}{totaltraded}$ is allocated to each of the securities to obtain the equally-weighted portfolio.

The value-weighted portfolio is composed of investing a proportion in the outstanding value of each stock. This is done, for instance, by SP 500 or NASDAQ [Fahlenbrach, 2022]. To form it, each month is taken in turn and the securities which are traded during this month (total_traded) are kept and we look at the market capitalization of each company (market_cap_i), given in the size data set. Seeing as though we invest at time t, we only know the size of the company at time t. But the return of the portfolio is only realised at time t + 1. This is why the firm return, return_t+1, is used to compute the return of the portfolio, in combination with the size at time t. Hence, the portfolio return is equal to the sum of weights based on time t and returns in period t + 1. The market capitalization changes each month, thus the weight of the asset does as well, seeing as though we rebalance the portfolio each month.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183417128-6b085495-8194-4775-b31b-cb18110ec854.png" width="800">
</p>

- **Report the following statistics for both portfolios: annualized average return, annualized volatility, minimum return, maximum return, and Sharpe ratio.**

We find the following summary statistics for each portfolio:

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183417961-6d518797-f9e9-4146-98f9-7c6ea4fcac3d.png" width="400"> <img src="https://user-images.githubusercontent.com/110820736/183468552-cb0f450b-5aca-4ada-a882-6c7b11d07e7d.png" width="400">
</p>

```
Figure 6: Summary statistics of the equally-weighted portfolio & Figure 7: Return of the equally-weighted portfolio
```

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183418044-5916ea11-6b43-41e4-b34d-568de8b34fbd.png" width="400"> <img src="https://user-images.githubusercontent.com/110820736/183468715-62db539e-e3ba-4d96-9399-900f8c4f7522.png" width="400">
</p>

```
Figure 8: Summary statistics of the value-weighted portfolio & Figure 9: Return of the value-weighted portfolio
```

We observe that the equally-weighted portfolio has a higher return, volatility, and Sharp ratio than the value-weighted one, as well as a lower minimum and higher maximum return. This last point can be linked to the fact that the equally-weighted portfolio has higher volatility so the range between the minimum and maximum is also bigger.

Plyakha et al. also found that with monthly rebalancing the equally-weighted portfolio outperforms the value-weighted one when it comes to the return and the Sharpe ratio. They found that this is partly because the equally-weighted portfolio is more exposed to the systematic risk factor and the other part is due to the difference in alpha (as defined by the authors). This comes from the fact that we rebalance the equally-weighted to maintain constant weights [Plyakha, 2014]. Additionally, the equally-weighted portfolio gives bigger weights to small firms and thus can have a bigger return since smaller companies tend to have higher returns.

The return of the value-weighted portfolio is rather low, especially if we consider that the risk is almost the same as for the equally-weighted portfolio which gives more than twice the return. This also explains why the Sharpe ratio is more than twice as high.

To compute the Sharpe ratio the average of the monthly European risk-free rates is taken and then multiplied by 12 to annualize it. The reward-to-variability ratio measures the expected return per unit of risk [Fahlenbrach, 2022]. Hence, the higher the ratio, the better. Given our data, it would be better to invest in the equally-weighted portfolio with a Sharpe ratio of 0.65 compared to only 0.29 for the value-weighted.

- **Plot the time series of returns for both portfolios.**

In the above plots (Figure 7 & 9) we can see that the range of the y-axis is bigger in positive terms for the equally-weighted portfolio, again indicating higher returns and volatility. Moreover, the financial crisis in 2008 can be well observed on both graphs with a huge drop of returns and characterises the minimum of the time series.

# Exercise 3
**Supposed that you invested 100% of your wealth in the asset with the highest annualized average return computed in point 1.**

- **Compare the annualized average return and annualized volatility of this one-asset portfolio with the equally-weighted and value-weighted portfolios?**

By selecting the company with the highest annualized average return (computed in Exercise 1), one finds the annualized average return and annualized volatility upon investing only in the asset selected above. However, the values retrieved were both rather high. The plot of the monthly returns of this company (Figure 12) clearly shows an outlier with a monthly return of 81.5, whereas most of the other monthly observations are closer to 0. Hence, this firm was discarded to avoid a biased average (influenced upwards) and to make it better represent reality. The same process is repeated and again shows an outlier that is eliminated following the reasoning as previously (Figure 13).

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183469003-a69a6169-9ef6-49af-b233-f17a53446105.png" width="400"> <img src="https://user-images.githubusercontent.com/110820736/183469120-000e5661-70ff-40d8-8674-fcaccd4470de.png" width="405">
</p>

```
Figure 10: Annualized average returns with the first outlier & Figure 11: Annualized average returns with the second outlier
```

At the third try, the following statistics are found, which don’t seem to be skewed by outliers:

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183434856-e6e46a93-6a37-48d2-8453-032335f14041.png" width="400"> <img src="https://user-images.githubusercontent.com/110820736/183469495-c1f882b5-5ac7-46b2-a673-bc5db0aacd2e.png" width="400">
</p>

```
Figure 12: Summary statistics & Figure 13: Maximal annualized average returns
```

The company with the highest annualized return is called " Israel Canada" and is active in the creation and improvement of properties in Israel [Website of Israel Canada, N/A]. It is not exactly an European firm but it was classified as such. Its annualized average return is 0.62. This is higher than the values obtained in exercise 2.2 (respectively 0.14 for the equally-weighted and 0.06 for the value-weighted portfolio). This is reasonable since the aim was to find the highest annualized average return. One observes a big spike in the plot around September 2006. The Israeli-Lebanese conflict began at that time [Wikipedia, 2022]. Therefore, we believe that this event influenced the returns of the company. This high value impacts the average return and volatility and is thought to slightly inflate the results.

The annualized volatility is around 1.36. It is much higher than the values obtained with the equally-weighted and value-weighted portfolios in exercice 2.2 (respectively 0.20 and 0.18).

- **What explains the differences between a one-asset portfolio and a portfolio composed of many stocks?**

To explain the differences between a one-asset portfolio and a portfolio composed of many stocks
and to understand our results, let us first define some terms. There are two types of risk, the
specific and systematic ones.

```
total _ risk = systematic _ risk + specific _ risk
```
The specific risk depends on each company. It is also called idosyncratic or firm-specific risk. It is linked to the performance and management of
a particular firm. On the other hand, systematic risk is common to all companies. It affects the
whole market and not just one particular firm, for instance events that have a global impact, such
as the subprime mortgage financial crisis in 2008.

The specific risk, unlike systematic risk, can be diversified. An analogy is not to put all one’s
eggs in the same basket. This means that by investing in many different companies, the overall
risk is reduced because the return no longer depends only on one company. For instance, if a
company faces difficulties, one may see one’s return decrease as a result. If it is the only firm one
has invested in, one will see a 100% reduction in one’s returns. Whereas, if one has only invested
a part of one’s assets therein, one’s total return will be negatively affected but it will not depend
solely on the performance of that specific company. Overall, the degree of diversification (and thus
risk reduction) attainable depends on the correlation between the assets one chooses to invest in
(Magnard-Vuibert, 2017, Gestion financière).

Coming back to our results, by investing in a single asset the overall risk is higher because the
specific risk is not diversified and hence its volatility is higher. While the more one diversifies one’s
portfolio, the more one decreases the specific risk and thus the overall risk. This is indeed what
can be observed in the data. The volatility (risk) of the one-asset portfolio is much higher than
that obtained with the equally-weighted and value-weighted portfolios.

Furthermore, a portfolio composed of many stocks, on average, also flattens/smoothens out the
returns. This explains why these portfolios yield a lower return than the one-asset portfolio.

- **What if you invest 100% of your wealth in the asset with the highest average return computed over the first 2 years?**

The goal here is to find the company with the highest average return computed over the first two
years. To be consistent with our data set we choose to annualize the terms to identify the firm
with the highest average return over two years. This decision does not change the interpretation
of the results because it is only a multiplication by factor 12.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183471017-0f4d043b-108e-46ac-928a-844fec4d1554.png" width="400"> <img src="https://user-images.githubusercontent.com/110820736/183472144-12fd0370-cce0-4bc4-ab2a-5ad199163a29.png" width="400">
</p>

```
Figure 14: Summary statistics & Figure 15: Maximal annualized average returns over two years
```
The plot depicts the monthly return of the company with the highest annualized average return
computed over the first two years. The firm is called "SOLAR B" and is a company of sourcing and
services active in Denmark [Company profile on Nasdaq]. One observes a spike in the plot around
September 2000. At the same time the population voted on a referendum to potentially change
to the euro (refused) which could be responsible for the spike observable in Figure 17 [Wikipedia,
2021]. Similarly to what was mentioned in section 3.1., this high value impacts the average return
and volatility and slightly inflates the results.

The highest annualized average return (4.68 versus 1.36 in exercise 3.1) and the volatility (2.
vs. 0.62 in exercise 3.1) are both much higher compared to before. The time horizon considered
is much shorter than in the first part of the exercise. Thus, fewer values are taken into account.
But more data, meaning a longer time series, allow to smooth out the variables according to the
rule of the big numbers (loi des grand nombres) [Rockinger, 2020]. Hence, a wider time horizon is
more representative of future long-term returns and risks.

**NB:** To be cautious, the two outliers found in this exercise will be ommitted from the data for all
following exercises.

# Exercise 4
**For this question, limit your set of firms to 50 randomly selected firms.**

- **Build an optimal portfolio with minimum variance with monthly rebalancing.**

In a world with many risky securities, one can identify the opportunity set of risk-return of different
portfolios, and given this set one can find the minimum variance portfolio. This is the so-called
program of Markowitz [Rockinger, 2020]. He found that the optimal portfolio allocation is reduced
to a simple optimization problem of the mean-variance criterion. To build this optimal portfolio
with minimum variance with monthly rebalancing, a rolling window of five years was implemented
in the code. These five years were chosen because the available data set contains 20 years and we
felt that anything below 5 years was insufficient to have a good representation of the past stock
behavior. At the same time, more than 5 years could have chosen, yet this restricts the available
timeline for the analysis. The decision was thus to settle with a 5-year (thus 60 month) rolling
window.

The firms which do not sell any shares (not publicly traded) were dropped each month and then the
rolling window was applied. Different assets were combined to form different portfolios describing
the efficiency frontier. By adding more and more companies, one can decrease the risk but one
has to keep in mind that risk cannot be reduced to zero and that there is a limit to diversification.
The portfolio on the efficient frontier that is most to the left (in a risk-return graph) is the optimal
minimum variance one [Rockinger, 2020].

Moreover, an agent that minimizes the portfolios’ variance subject to a return constraint will find
the same result as an investor maximizing the portfolio return subject to a volatility constraint
[Fahlenbrach, 2022].

- **Report the following statistics: annualized average return, annualized volatility, minimum return, maximum return, and Sharpe ratio.**

It is important to note that 50 firms were randomly selected for constructing this portfolio, thus
the summary statistics change each time the code is run and they are not necessarily according to
the theory.

The aim of the minimum variance portfolio is, as the name already says, to have the smallest
volatility. We find an annualized volatility of 0.189. This is almost the same as the volatility of
the EU market (0.186) and thus a satisfactory result.

The rest of the portfolio performance is summarized here below.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183472675-56964cc5-932c-492b-aa4d-bfae6fe4f24f.png" width="400"> <img src="https://user-images.githubusercontent.com/110820736/183472738-6bfbe8c4-6448-4a6b-bee9-601cbaade27c.png" width="400">
</p>

```
Figure 16: Summary statistics of the minimum variance protfolio & Figure 17: Returns of the minimal variance portfolio
```

- **Comment on the reported statistics in comparison with the equally-weighted and value-weighted portfolio.**

According to the theory, in comparison, the minimum variance portfolio (MVP) should have the
smallest annualized volatility, since that is its aim. Yet, the value-weighted portfolio is a bit lower
but almost equal (0.182 vs. 0.189). On the other hand, the MVP has a better performance for the
return (0.09 vs. 0.06) and the Sharpe ratio (0.41 vs. 0.29).

The minimum variance portfolio is a unique and robust solution. In the special case where all
assets have the same correlation coefficient and identical means and variances then the equally-
weighted portfolio is the unique portfolio on the frontier. Maillard found that the equally-weighted
portfolio has a higher return and volatility but a smaller Sharpe Ratio [Maillard, 2009]. We find
this holds for the return (0.14 vs. 0.09) and the volatility (0.2 vs. 0.19) but not the Sharpe Ratio
(0.65 vs. 0.41).

# Exercise 5

**For this question, keep the same randomly selected firms from the previous point.**

- **Build an optimal portfolios with various target portfolio returns (e.g., from 2% to 16% with 2% increments).**

We assume considering purely risky portfolios and the optimal ones for different returns (thus
along the efficient frontier). Nevertheless, we would like to discuss the possibility of investing also
in the risk-free asset. This means investing part of our assets in the tangency portfolio, which
has the maximum Sharpe ratio (also called a market portfolio), and the other part in the risk-free
asset. This would decrease the risk and return of the overall portfolio. In both cases (with and
without the risk-free asset) the optimal risk depends on the risk aversion of the agent.

Our target return starts at 2%. After plotting the efficiency frontier we noticed that aiming for
16% is not feasible. This is probably due to the randomly selected 50 firms. Thus, we decided
to have the same number of portfolios but in a range that corresponds to our data. Hence, we
constructed eight portfolios with an increment that is in line with our data range.

- **Plot the efficient frontier as well as the individual assets.**

If the assets of a risk-return graph are vertically aligned that means they have the same volatility
but some have a higher return. Given this volatility, a rational agent would invest in the asset
that yields the highest return.

The data consists of many risky securities, hence the opportunity set of the risk-return combina-
tions of various portfolios is identifiable. With this set, one can determine the minimum variance
portfolio. The efficiency frontier is then the section of the opportunity set above the **minimum
variance portfolio**. It is the limiting relationship between return and risk and contains the op-
timal portfolios, meaning it depicts the maximum attainable return for a given volatility and a
given set of assets.

The portfolio that will be chosen by the investor, which lies **on** the efficient frontier, will depend
on his/her level of risk aversion. Furthermore, a risk-lover investor will choose a portfolio on the
efficient frontier, but further to the right (in a risk-return graph), while a risk-averse investor will
choose a portfolio more to the left.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183473626-7405b3bd-781b-4484-bae6-9c95f819b319.png" width="800">
</p>

```
Figure 18: Efficient frontier (in blue) with the different assets (black data points)
```

- **Which portfolio is the most efficient in terms of Sharpe ratio?**

The Sharpe ratio is found using the following formula:

$SR = \frac{E(Rp) − Rf}{σp}$

where _E(Rp)_ is the expected return of the risky portfolio, _Rf_ is the risk free rate and _σp_ is the
volatility of the risky portfolio.

The Sharpe ratio is used to calculate the relationship between the return and risk of an investment
[Capital, 2020]. As observable in the Figure 19, the risky portfolio with the highest Sharpe ratio is
the tangency portfolio (tangency point between the capital allocation line and the mean-variance
efficient frontier) [Dhankar, Raj and Maheshwari, Supriya, 2016].

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183475065-06eebe4d-c115-45c5-b8ea-c23ce51c33cc.png" width="800">
</p>

```
Figure 19: Graph showing the Sharpe ratio (slope of the CAL) as well as the tangency portfolio
```

Depending on the value of the Sharpe ratio, the following interpretations exist:

1. If the ratio is negative, it means that the profitability of this investment is lower than that
    of a risk-free investment.
2. If the ratio is between 0 and 1, the risk taken by investing is not sufficiently compensated by
    the return.
3. If the ratio is greater than 1, the return on the investment is higher than that of a risk-free
    investment. Although the risk is higher as well, it is "compensated" by the return obtained
    [Wikipedia, 2022].

We obtain the following Sharpe ratios for each targeted return:

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183475341-f7850834-4580-4263-848b-859445a18c9b.png" width="800">
</p>

```
Figure 20: Summary table of the different Sharpe ratios obtained per targeted return
```
The ratios are between 0 and 1, meaning that our portfolios are sub-optimal and that it is un-
desirable to invest in them. As mentioned in bullet point 2, the risk taken by investing is not
sufficiently compensated by the return, hence the return is too low and volatility too high.

From the selection, the portfolio with a targeted return of 0.62 is the most efficient in terms of the
Sharpe ratio because it gives the highest ratio.

# Exercise 6

- **Choose an appropriate benchmark, which should correspond to the region of your data set.**

Given the European dataset, we choose a European market index as the appropriate benchmark
because it gives a representative portfolio composed of traded European firms. There are multiple
possible indices, such as the S&P Europe 350, STOXX Europe 50, or STOXX Europe 600, which
tracks the returns of the 600 largest stock exchange-listed companies out of 17 European countries
[Justetf, 2022]. Seeing as though this report does not specialize in a specific industry, the index
chosen should largely be representative of the European stock market as a whole. A second criterion
is a benchmark with enough available data. So, we choose the "excess return on the market" data
from the Fama/French European 3 Factors dataset (also provided in the eu_market data set for
this exercise). Its formula is the following:

```
Excess _ European _ market _ return = European _ market _ return − Risk _ free _ rate
```
Hence, the benchmark is:

```
European _ market _ return = Excess _ European _ market _ return + Risk _ free _ rate
```

- **Compare the performance of your portfolios (equally-weighted, value-weighted, and minimum variance) with the benchmark. Comment on the differences.**

The European market index (EU market return), starts in the year 2002 and the dataset contains
monthly values. The aim is to compare the performance of this index with the equally-weighted,
value-weighted and minimum-variance portfolios from the exercise before. Here is a graph sum-
marizing all of them in one:

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183475955-f9d3c375-11d3-4c71-8074-7d6c0545dbaa.png" width="800">
</p>

```
Figure 21: Monthly return of different portfolios
```

It depicts well how the 2008 financial crisis influences the entire financial market and is reflected
by a large drop in the returns of all of each portfolio. The more recent dip observed (around 2020)
is due to Covid-19 and had similar impact on the market as a whole.

- Equally-weighted portfolio and European Market Index

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183477549-4705d5f8-76d8-461b-b65d-93e0c365c226.png" width="800">
</p>

```
Figure 22: Comparison of the equally weighted portfolio and the European market
```

In this plot, one can read the performance of the equally-weighted (EW) and European
market portfolios. The monthly return of the EW portfolio is more varied with slightly
larger extremes (see min, max, and volatility in the table below). This higher volatility
of the EW portfolio translates into a higher average annualized return (13.20% > 7.73%).

The spike observed in 2009 is what pulls up the average return of the EW portfolio. This
portfolio is comprised of securities that are negatively correlated to some extent, thus it
allows for diversification and minimizes the risk. Yet, it is impossible to diversify for the
market risks mention above.

- Value-weighted portfolio and European Market Index

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183477739-71ca4108-8b4c-4749-a290-637da7918467.png" width="800">
</p>

```
Figure 23: Comparison of the value weighted portfolio and the European market
```

This graph compares the performance of the value-weighted (VW) and European market
portfolios. The VW portfolio is the closest to the performance of the benchmark. The
reason is that like the European market index, the VW portfolio tries to reflect reality by
attributing larger weights to companies of a bigger size (market indices are generally weighted
by market capitalization/size or sometimes by prices). Hence, they have similar volatility
and return, as well as minimum and maximum returns. This is perfectly according to the
theory.

- Minimum variance portfolio and European Market Index

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183477904-0ab059e4-230c-40de-8d6e-b1e191c8b208.png" width="800">
</p>

```
Figure 24: Comparison of the minimum variance portfolio and the European market
```

Lastly, the comparison between the minimum variance (MV) and European market portfolios.
The MV portfolio has on average a lower volatility. When it comes to the MV portfolio, as
predicted, we will have, on average, a lower volatility than the market portfolio (0.13 <
0.19). This can also be observed on the graph since the EU market return generally has
bigger spikes, meaning more volatility. This is evident as the aim of the MV portfolio is to
actively minimize the risk, meaning in turn that there is no/little return compensation for
the risk taken. Both returns are rather similar, thus it would make more sense to invest in
the MV portfolio than in the market one since for a given return the risk is lower.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183478116-be0599dd-3c19-4dc0-960a-87ebb2729fea.png" width="800">
</p>

```
Figure 25: Summary table of the performances of the different portfolios
```

# Exercise 7

**Compute and comment on the simple correlation between returns, volatility, size.**

- **Correlation between returns and volatility**

Between returns and volatility, a correlation of 0.79 is found. The following graph illustrates this
positive relationship:

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183478359-fc42b950-c948-48c6-a806-5cd73c3701f5.png" width="800">
</p>

This relationship is explained by the fact that an agent will not invest in a riskier asset if there is
no benefit in the form of return. In that case, s/he would only invest in risk-free assets. Therefore,
for a higher risk, the return is higher [Lafinancepourtous, 2022].

- **Correlation between returns and size**

The correlation between returns and size is - 0.14. This result is confirmed in the existing literature.
Indeed, it was already in 1980 that Rolf. W Banz published one of the first studies to highlight
the negative relationship between firm size and returns [reader.elsevier, 2022]. This phenomenon
is well-known as the "small firm effect". It is thought that smaller firms have more growth oppor-
tunities and are more adaptable, hence can benefit better from market trends and thus capture
more market share compared to larger firms [smartcapitalmind, 2022]. Indeed, smaller firms have
much simpler and therefore faster decision-making processes than larger firms.

- **Correlation between volatility and size**

Concerning the relationship between volatility and size, we obtain - 0.19. Again, this is in line
with the theory. Indeed, investing in a smaller firm comes with a larger risk, due to the lack of
in- and tangible resources of the firm, hence it is more vulnerable to market shocks [investopedia,
2022]. Nevertheless, this risk is compensated by a higher return, as explained in 7.1.

# Exercise 8

**For this question, take the same 50 selected firms.**

- **Create a minimum variance portfolio with monthly rebalancing with an additional constraint: you exclude the smallest firms (bottom tercile of the distribution of the firms’ market capitalization in month t–1).**

The 50 randomly selected firms are the starting point. The smallest firms (bottom tercile) are
excluded according to their firm size and this is done for each month. Hence, only the desired
firms are selected from the price database. These databases are used to create the minimum
variance portfolio with monthly rebalancing (the process is the same as in exercise 4).

- **Report summary statistics on the performance of this portfolio and comment on the differences with the minimum variance from exercise 4.**

The following data is obtained:

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183478589-b654bd83-d237-4571-91c4-f1746b68c5e8.png" width="400"> <img src="https://user-images.githubusercontent.com/110820736/183478656-7f5cb2f1-403b-4396-bf45-67514608c5fa.png" width="400">
</p>

```
Figure 26: Summary statistics of the minimum variance portfolio with monthly rebalancing & Figure 27: Return of the minimum variance portfolio with monthly rebalancing
```
The minimum and maximum returns are the same values. This is likely due to the fact that we
have the same data set.

The annualized average return is higher than in exercise 4, where we did the same thing except
from removing the lowest tercile (of size). This result is unexpected because, in question 7, we
found a negative correlation between size and returns. Meaning that the smaller a firm is the higher
its return. In this portfolio, we have eliminated the firms with the smallest size and therefore a
high return that previously would have increased the average return. After removing the lowest
tercile, only the bigger firms remain, thus the annualized average return should be lower compared
to that of exercise 4. The same reasoning can be applied to the volatility, since it is also negatively
correlated with size. One can observe a slight decrease in volatility (0.182 < 0.189)

Given the bigger change in return and a small one in the volatility, the Sharpe ratio increases.

In conclusion, these values depend strongly on the random sample and change at each new run of
the code. Since our sample is rather small it may not fit the theory.

# Exercise 9

- **For each time period, sort firms based on size into quintiles. Create equally-weighted and value-weighted portfolios for each time period and each size quintile. Report the average returns for each quintile portfolio as well as a portfolio that goes long in the lowest quintile and short the highest quintile. Comment on your results.**

We start by retrieving the size data set with all of the securities and sorting them in ascending
order. We then create 5 equally-sized bins (quintiles) and retrieve the ISINs of the firms in each
quintile. With each of these quintiles (q1 to q5), we create equally-weighted and value-weighted
portfolios for each month, similarly to exercise 2.

For the second part of the question, the aim is to construct a portfolio (one equally-weighted and
one value-weighted) that goes long in the lowest quintile (q1) and short in the highest quintile (q5).
Going long (holding a long position) in a financial asset can have different meanings depending on
the type of asset (call, put, etc). In this report, the most common use of the term is considered.
That is a long-position investment in which the investor purchases an asset and owns it with the
expectation that the price is going to rise [Investopedia, 2021]. On the other hand, going short
(holding a short position) means that the investor sells an asset, anticipating the price will fall in
the short term and plans to buy it later [Investopedia, 2021]. In the equally weighted portfolio, we
decide to invest 50% of our wealth in the lowest quintile and the other 50% in the highest quin-
tile. Within these semi-portfolios, we distribute the weight equally, with the weights in the lowest
quintile being **positive** and the weights in the highest quintile (short-position) being **negative**.
The same is done for the value-weighted portfolio, only this time, the weights are proportional to
the size of the firms, with the weights in the lowest quantile remaining **positive** and the weights
in the highest quintile **negative**. We call this portfolio the "Short-long (SL) portfolio".

For the lowest quintile:

```
wi = -
```
#### 0. 5

```
total _ securities _ q 1
```
For the highest quintile:

```
wi =
```
#### 0. 5

```
total _ securities _ q 5
```

Average returns for each quintile portfolio:

```
Figure 28: Average return of the equally
weighted portfolio per quantile
```
```
Figure 29: Average return of the value weighted
portfolio per quantile
```
Above the average returns of the equally-weighted (EW) and value-weighted (VW) portfolios per
quintile, from lowest (q1) to highest (q5) are depicted.

To see the effect of the size more clearly, we plot only the first and last quintiles.

```
Figure 30: Comparision of quantile 1 and 5 for
the equally weighted portfolio
```
```
Figure 31: Comparision of quantile 1 and 5 for
the value weighted portfolio
```
The volatility of the lower quintile (q1) is generally higher. This matches with the theory seen in
exercise 7, since smaller firms generally make for riskier investments. In both the EW and VW 
portfolios, the overall average return (taken across all months) is larger for the higher quintile. For
the EW portfolio, this disparity is less significant (Q1 average return = 0.0069, Q5 average return =
0.011 –>∆= 0.0049). Whereas for the VW portfolio, the difference is larger and more significant
(Q1 average return = -0.009, Q5 average return = 0.006 –>∆= 0.015). This is counter-intuitive,
as one would expect the higher quintile (q5) to have smaller returns as a result of the negative
correlation between size and return as well as the positive correlation between risk and return.

Average returns for the short-long (SL) portfolio:

```
Figure 32: Average return of the short-long
equally weighted portfolio
```
```
Figure 33: Average return of the short-long
value weighted portfolio
```
The average return of the SL portfolio is -0.02 for the EW and -0.07 for the VW. This makes sense
insofar that an EW portfolio allows us to have a greater return (as seen in exercise 2). Nevertheless,
upon this positioning in the market we were expecting to have higher returns than in the base case
(simply investing in q1 firms) because the smaller firms are predicted to have higher returns (so
we go long) and the larger firms a lower return (hence we go short). So overall, our SL portfolio
should have outperformed the previously analyzed portfolios comprising of each size quintile (in
theory). The discrepancy with our results could be due to an error in the data or a computational
error upon constructing the portfolio.

- **What can explain the relationship between returns of your portfolio and firms’ size?**


- **Repeat this exercise but sorting firms based on past stock returns. Compute past returns in month t as the cumulated return of a firm between months t - 13 and t - 1.**

### 9.1

We start by retrieving the size data set with all of the securities and sorting them in ascending
order. We then create 5 equally-sized bins (quintiles) and retrieve the ISINs of the firms in each
quintile. With each of these quintiles (q1 to q5), we create equally-weighted and value-weighted
portfolios for each month, similarly to exercise 2.

For the second part of the question, the aim is to construct a portfolio (one equally-weighted and
one value-weighted) that goes long in the lowest quintile (q1) and short in the highest quintile (q5).
Going long (holding a long position) in a financial asset can have different meanings depending on
the type of asset (call, put, etc). In this report, the most common use of the term is considered.
That is a long-position investment in which the investor purchases an asset and owns it with the
expectation that the price is going to rise [Investopedia, 2021]. On the other hand, going short
(holding a short position) means that the investor sells an asset, anticipating the price will fall in
the short term and plans to buy it later [Investopedia, 2021]. In the equally weighted portfolio, we
decide to invest 50% of our wealth in the lowest quintile and the other 50% in the highest quin-
tile. Within these semi-portfolios, we distribute the weight equally, with the weights in the lowest
quintile being **positive** and the weights in the highest quintile (short-position) being **negative**.
The same is done for the value-weighted portfolio, only this time, the weights are proportional to
the size of the firms, with the weights in the lowest quantile remaining **positive** and the weights
in the highest quintile **negative**. We call this portfolio the "Short-long (SL) portfolio".

For the lowest quintile:

```
wi = -
```
#### 0. 5

```
total _ securities _ q 1
```
For the highest quintile:

```
wi =
```
#### 0. 5

```
total _ securities _ q 5
```

Average returns for each quintile portfolio:

```
Figure 28: Average return of the equally
weighted portfolio per quantile
```
```
Figure 29: Average return of the value weighted
portfolio per quantile
```
Above the average returns of the equally-weighted (EW) and value-weighted (VW) portfolios per
quintile, from lowest (q1) to highest (q5) are depicted.

To see the effect of the size more clearly, we plot only the first and last quintiles.

```
Figure 30: Comparision of quantile 1 and 5 for
the equally weighted portfolio
```
```
Figure 31: Comparision of quantile 1 and 5 for
the value weighted portfolio
```
The volatility of the lower quintile (q1) is generally higher. This matches with the theory seen in
exercise 7, since smaller firms generally make for riskier investments. In both the EW and VW 
portfolios, the overall average return (taken across all months) is larger for the higher quintile. For
the EW portfolio, this disparity is less significant (Q1 average return = 0.0069, Q5 average return =
0.011 –>∆= 0.0049). Whereas for the VW portfolio, the difference is larger and more significant
(Q1 average return = -0.009, Q5 average return = 0.006 –>∆= 0.015). This is counter-intuitive,
as one would expect the higher quintile (q5) to have smaller returns as a result of the negative
correlation between size and return as well as the positive correlation between risk and return.

Average returns for the short-long (SL) portfolio:

```
Figure 32: Average return of the short-long
equally weighted portfolio
```
```
Figure 33: Average return of the short-long
value weighted portfolio
```
The average return of the SL portfolio is -0.02 for the EW and -0.07 for the VW. This makes sense
insofar that an EW portfolio allows us to have a greater return (as seen in exercise 2). Nevertheless,
upon this positioning in the market we were expecting to have higher returns than in the base case
(simply investing in q1 firms) because the smaller firms are predicted to have higher returns (so
we go long) and the larger firms a lower return (hence we go short). So overall, our SL portfolio
should have outperformed the previously analyzed portfolios comprising of each size quintile (in
theory). The discrepancy with our results could be due to an error in the data or a computational
error upon constructing the portfolio.

### 9.2

As discussed in exercise 7, there is indeed a relationship between portfolio return and firm size.
Generally, size and return are negatively correlated. Smaller firms will on average yield higher
returns and vice versa for larger firms. This is due to something called the "small firm effect", which
demonstrates how smaller firms are riskier and hence yield a higher return, but also how small
firms are more flexible, leading to them being able to better benefit from the market conditions
and trends. There is also usually higher growth potential for smaller firms. This is also reflected
in returns (a function of price), as we consider the future potential in the present valuation of a
company’s shares.

### 9.3

In this part of the exercise, the steps under point 9.1 are repeated. However, we separate firms into
quintiles based upon their previous stock returns, more precisely the cumulated returns over the
past year. We then create an equally-weighted and value-weighted portfolio per month for each of
the quintiles, as well as a portfolio than goes long in the lowest quintile and short in the highest.

Average returns for each quintile portfolio and the short-long (SL) portfolio:

```
Figure 34: Average return of the equally
weighted portfolio per quantile
```
```
Figure 35: Average return of the equally
weighted portfolio per quantile
```
```
Figure 36: Comparison of the quantile 1 and 5
for the equally weighted portfolio
```
```
Figure 37: Comparison of the quantile 1 and 5
for the equally weighted portfolio
```

```
Figure 38: Average return of the short-long
equally weighted portfolio
```
```
Figure 39: Average return of the short-long
equally weighted portfolio
```
Above, one observes the different average returns per quintile per type of portfolio. Firstly, one
notes that the scales for the EW and VW portfolio returns are not the same. We have much higher
volatility for the equally-weighted portfolios, partially driven by one very high return of approxi-
mately 125%. Again the lower quintiles have higher volatility compared to q5. The relationship
between average return (of all months) and past return is shown to be positive (Q1 average return
= 0.009, Q5 average return = 0.019, for EW portfolio and Q1 average return = 0.003, Q5 average
return = 0.009, for VW portfolio). In terms of the SL portfolio, there are no significant differences.

How to explain this relationship between returns of the portfolio and the past returns of the firm?
A phenomenon referred to as "momentum" is observed here. Generally, firms with rising prices
(and hence high returns) continue to experience high returns, whereas firms with low past returns
tend to continue on this trend. With this in mind, firms that have performed better in the past
ought to outperform firms that have had lower past returns [12]. This is a _market anomaly_ that
the theory has trouble explaining, but that is often observed in practice. In other words, it can be
said that past "winners" tend to perform better!

## Sources

[1] https://www.vuibert.fr/system/files/ressources/9782311401745-theorie-du-portefeuille.pdf

[2] https://www.investopedia.com/terms/s/short.asp: :text=A%20short%20position%20refers%

20to,fall%20in%20the%20short%20term.

[5] https://www.smartcapitalmind.com/what-is-the-small-firm-effect.htm

[6] https://www.investopedia.com/ask/answers/022715/are-small-cap-companies-more-risky-

investments-large-cap-companies.asp: :text=Small%2Dcap%20companies%20tend%20to,negative%
20events%20and%20bearish%20sentiments.

[7] https://www.investopedia.com/terms/s/smallfirmeffect.asp: :text=The%20small%20firm%

20effect%20theory%20holds%20that%20smaller%20companies%20have,to%20a%20large%

20price%20appreciation.

[8] https://reader.elsevier.com/reader/sd/pii/0304405X81900180?token=E9C6045225284B3

161FF5D4F1B376AA00A1EFF579179FAB5FABD5FFA1A37A2E5EED8E2D9C3868180B35110C

C9600F6FAoriginRegion=eu-west-1originCreation=20220323130656

[9] https://www.capital.fr/entreprises-marches/ratio-de-sharpe-1380962

[10] https://www.lafinancepourtous.com/decryptages/finance-perso/epargne-et-placement/le-couple-
rendement-risque/

[11] Dhankar, Raj and Maheshwari, Supriya, Behavioural Finance: A New Paradigm to Explain
Momentum Effect (May 27, 2016). Available at SSRN: https://ssrn.com/abstract=2785520 or
[http://dx.doi.org/10.2139/ssrn.2785520](http://dx.doi.org/10.2139/ssrn.2785520) [12] https://en.wikipedia.org/wiki/Momentum _f inance_

[13] https://fr.wikipedia.org/wiki/Ratio _deSharpe_

[14] https://www.investopedia.com/terms/m/marketindex.asp

[15] https://risk.edhec.edu/sites/risk/files/edhec-working-paper-equal-or-value-weighting-

f 1403680766170 _.pdf_

[16] Fahlenbrach, 2022, Sustainable and Entrepreneurial Finance

[17] https://fr.wikipedia.org/wiki/Seconde _intifada_

[18] https://www.vuibert.fr/system/files/ressources/9782311401745-theorie-du-portefeuille.pdf

[19] https://israel-canada.co.il/en/about-us-english/

[20] https://fr.wikipedia.org/wiki/Euro _etDanemark_

[21] [http://www.nasdaqomxnordic.com/shares/microsite?Instrument=CSE3431](http://www.nasdaqomxnordic.com/shares/microsite?Instrument=CSE3431)

[22] https://fr.wikipedia.org/wiki/Conflit _isra_ é _lo_ − _libanaisde_ 2006

[23] Maillard, S., Roncalli, T., Teïletche, J. (2010). The properties of equally weighted risk
contribution portfolios. The Journal of Portfolio Management, 36(4), 60-70

[24] https://www.investopedia.com/terms/s/security.asp


Mia Frey, Elodie Savioz
March 24, 2022

[25] https://www.justetf.com/en/how-to/stoxx-europe-600-etfs.html

[26] https://www.investopedia.com/terms/l/long.asp




# Sources
[1] https://www.vuibert.fr/system/files/ressources/9782311401745-theorie-du-portefeuille.pdf
\


[2] https://www.investopedia.com/terms/s/short.asp#:~:text=A\%20short\%20position\%20refers\%

20to,fall\%20in\%20the\%20short\%20term.
\

[5] https://www.smartcapitalmind.com/what-is-the-small-firm-effect.htm
\

[6] https://www.investopedia.com/ask/answers/022715/are-small-cap-companies-more-risky-

investments-large-cap-companies.asp#:~:text=Small\%2Dcap\%20companies\%20tend\%20to,negative\%
20events\%20and\%20bearish\%20sentiments.
\

[7] https://www.investopedia.com/terms/s/smallfirmeffect.asp#:~:text=The\%20small\%20firm\%

20effect\%20theory\%20holds\%20that\%20smaller\%20companies\%20have,to\%20a\%20large\%

20price\%20appreciation.
\

[8] https://reader.elsevier.com/reader/sd/pii/0304405X81900180?token=E9C6045225284B3

161FF5D4F1B376AA00A1EFF579179FAB5FABD5FFA1A37A2E5EED8E2D9C3868180B35110C

C9600F6FA&originRegion=eu-west-1&originCreation=20220323130656
\

[9] https://www.capital.fr/entreprises-marches/ratio-de-sharpe-1380962
\

[10] https://www.lafinancepourtous.com/decryptages/finance-perso/epargne-et-placement/le-couple-rendement-risque/

\
[11] Dhankar, Raj and Maheshwari, Supriya, Behavioural Finance: A New Paradigm to Explain Momentum Effect (May 27, 2016). Available at SSRN: https://ssrn.com/abstract=2785520 or http://dx.doi.org/10.2139/ssrn.2785520
\
[12] https://en.wikipedia.org/wiki/Momentum_\(finance\)

[13] https://fr.wikipedia.org/wiki/Ratio_de_Sharpe
\

[14] https://www.investopedia.com/terms/m/marketindex.asp

[15] https://risk.edhec.edu/sites/risk/files/edhec-working-paper-equal-or-value-weighting-

f_1403680766170.pdf

[16] Fahlenbrach, 2022, Sustainable and Entrepreneurial Finance

[17] https://fr.wikipedia.org/wiki/Seconde_intifada

[18] https://www.vuibert.fr/system/files/ressources/9782311401745-theorie-du-portefeuille.pdf

[19] https://israel-canada.co.il/en/about-us-english/ 

[20] https://fr.wikipedia.org/wiki/Euro_et_Danemark

[21] http://www.nasdaqomxnordic.com/shares/microsite?Instrument=CSE3431

[22] https://fr.wikipedia.org/wiki/Conflit_israélo-libanais_de_2006

[23] Maillard, S., Roncalli, T., & Teïletche, J. (2010). The properties of equally weighted risk contribution portfolios. The Journal of Portfolio Management, 36(4), 60-70

[24] https://www.investopedia.com/terms/s/security.asp
\

[25] https://www.justetf.com/en/how-to/stoxx-europe-600-etfs.html
\

[26] https://www.investopedia.com/terms/l/long.asp

\end{document}
