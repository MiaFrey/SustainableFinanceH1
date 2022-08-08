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
unique identifier (instead of the NAME), as certain securities could share the same name, but have different ISINs. Our mission is to identify the European securities with available scope 1,2 & 3 emissions.

To identify these firms of interest, we merge the scope 1-3 emission data sets with the regional data on ISIN and drop any rows for which the region is different from Europe (’EUR’). We then collect the list of securities’ ISINs for each scope and take the intersection of securities for which we have data in all three scopes (inner join). In our case, the securities are already the same (without duplicates), so there is no need to drop further data. We thus start our analysis with
2’739 different securities (sample size).

For the formatting of our data, we decided to import and transpose the data sets of interest, while only keeping the securities found above. That way, we have the rows as our time indicator (per month, reformatted from string to date type) and the columns represent the securities (ISIN). We saved the cleaned data into new excel files locally, to reduce the compilation time when running the code repeatedly. Lastly, once having calculated the returns, we drop the month December 1999 in our imported data sets, to remain consistent, as this month is "lost" upon calculating the returns. Except for the data set size, for which we use the December 1999 data in later exercises (See below an example of a cleaned data set, see Figure below.. Further, we make sure to also replace infinite values with NaNs at this point, as these undefined values else would create noise.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183402126-c9d54b9c-b753-4f0c-89d7-7c79da1a4b5f.png" width="800">
</p>


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
    <img src="https://user-images.githubusercontent.com/110820736/183404762-5584ff00-5e64-4331-a588-baf9f439ba1f.png" width="400">
</p>

The data points on the graph above represent the securities. The higher the return the better. We can observe that the annualized average return of many of them is positive. Only a few of them have a negative value. As mentioned in the summary statistics table, the average of all those returns is 0.13. Furthermore, we can observe two outliers. We will go more into depth about these in exercise 3. For the volatility we used the built-in Python function of the standard deviation and multiplied this by the square root of 12 to obtain the annualized volatility per individual asset. This is one of the best ways to measure the risk because it is easier to interpret than the variance [Rockinger, 2020]. Thus, the lower the volatility the better, since it is synonymous with a smaller risk.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183404948-151d0a9b-d9cd-4721-9888-cda7afc21bc0.png" width="400">
</p>

We find that the graph of the volatility (figure above) looks similar to the one of the return. This
shows the highly positive correlation between these two measures (risk and return) which will be
discussed in point 1.2. Again, the two outliers can be nicely observed with high volatility. This
already hints at the fact that their large returns probably stem from a one-time occurrence that
greatly increases the average, since this metric is not robust to extremes. The average of all the
annualized volatilities is 0.40.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183405022-d9e50f80-cf8e-4aa4-a860-df872dfb3aa0.png" width="800">
</p>

- **Compute the correlation between individual returns and volatility and comment
on the observed correlation.**

The computed correlation between the individual annualized average returns and volatility is 0.79. It is strongly positive and almost equal to 1. This is in line with the theory because we know that the return acts as compensation for the risk taken. Meaning the higher the volatility, the higher the risk ought to be. If this were not the case, then it would be easy to invest with small risk and gain high returns, so we would all be rich. At this stage, it is also important to look at the correlation between the securities, especially since portfolios will be created in the following exercises. The higher the correlation between the assets, the more difficult it is to benefit from diversification. If the assets are perfectly negatively correlated, then there exists a portfolio such that the expected risk is zero. In reality, this is rarely the case. If the portfolio is big, then the variance terms are diversified away but not the covariance terms. Hence, diversification can eliminate some, but not all the risk of individual securities. There is a systematic risk (or market risk) that defines the lower bound of our portfolio risk.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183405306-9ce6061c-479c-4a87-b81e-0beeffca8cd6.png" width="800">
</p>

In the heatmap above, we observe that our assets are mostly positively correlated.
But there are not perfectly positive and we have some negative correlation coefficients as well,
indicating that we can benefit from diversification (although not fully).

## Exercise 2

- **Form an equally-weighted and value-weighted portfolio with monthly rebalancing.**

The equally-weighted portfolio is one where all constituent firms have an equal value [Fahlenbrach, 2022]. To form it, each month is taken in turn and the securities which are traded during this month (total_traded) are kept (except NaN). These are the securities used to build our monthly portfolio. Then a weight of (w_i = 1/total_traded) is allocated to each of the securities to obtain the equally-weighted portfolio.

The value-weighted portfolio is composed of investing a proportion in the outstanding value of each stock. This is done, for instance, by SP 500 or NASDAQ [Fahlenbrach, 2022]. To form it, each month is taken in turn and the securities which are traded during this month (total_traded) are kept and we look at the market capitalization of each company (market_cap_i), given in the size data set. Seeing as though we invest at time t, we only know the size of the company at time t. But the return of the portfolio is only realised at time t + 1. This is why the firm return, return_t+1, is used to compute the return of the portfolio, in combination with the size at time t. Hence, the portfolio return is equal to the sum of weights based on time t and returns in period t + 1. The market capitalization changes each month, thus the weight of the asset does as well, seeing as though we rebalance the portfolio each month.

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183417128-6b085495-8194-4775-b31b-cb18110ec854.png" width="800">
</p>

- **Report the following statistics for both portfolios: annualized average return, annualized volatility, minimum return, maximum return, and Sharpe ratio.**

We find the following summary statistics for each portfolio:


<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183417961-6d518797-f9e9-4146-98f9-7c6ea4fcac3d.png" width="400"> <img src="https://user-images.githubusercontent.com/110820736/183418006-afb15de0-3469-4d90-addb-842b72f15f87.png" width="400">
</p>

<p align="center">
    <img src="https://user-images.githubusercontent.com/110820736/183418044-5916ea11-6b43-41e4-b34d-568de8b34fbd.png" width="400"> <img src="https://user-images.githubusercontent.com/110820736/183418072-cd892c50-b746-40cc-abda-2292dcf1c741.png" width="400">
</p>

We observe that the equally-weighted portfolio has a higher return, volatility, and Sharp ratio than the value-weighted one, as well as a lower minimum and higher maximum return. This last point can be linked to the fact that the equally-weighted portfolio has higher volatility so the range between the minimum and maximum is also bigger.

Plyakha et al. also found that with monthly rebalancing the equally-weighted portfolio outperforms the value-weighted one when it comes to the return and the Sharpe ratio. They found that this is partly because the equally-weighted portfolio is more exposed to the systematic risk factor and the other part is due to the difference in alpha (as defined by the authors). This comes from the fact that we rebalance the equally-weighted to maintain constant weights [Plyakha, 2014]. Additionally, the equally-weighted portfolio gives bigger weights to small firms and thus can have a bigger return since smaller companies tend to have higher returns.

The return of the value-weighted portfolio is rather low, especially if we consider that the risk is almost the same as for the equally-weighted portfolio which gives more than twice the return. This also explains why the Sharpe ratio is more than twice as high.

To compute the Sharpe ratio the average of the monthly European risk-free rates is taken and then multiplied by 12 to annualize it. The reward-to-variability ratio measures the expected return per unit of risk [Fahlenbrach, 2022]. Hence, the higher the ratio, the better. Given our data, it would be better to invest in the equally-weighted portfolio with a Sharpe ratio of 0.65 compared to only 0.29 for the value-weighted.

- **Plot the time series of returns for both portfolios.**

In the above plots we can see that the range of the y-axis is bigger in positive terms for the equally-weighted portfolio, again indicating higher returns and volatility. Moreover, the financial crisis in 2008 can be well observed on both graphs with a huge drop of returns and characterises the minimum of the time series.

# Exercise 3
**Supposed that you invested 100% of your wealth in the asset with the highest annualized average return computed in point 1.**

- **Compare the annualized average return and annualized volatility of this one-asset portfolio with the equally-weighted and value-weighted portfolios?**



- **What explains the differences between a one-asset portfolio and a portfolio composed of many stocks?**


- **What if you invest 100% of your wealth in the asset with the highest average return computed over the first 2 years?**

