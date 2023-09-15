# Time Series Analysis for Portfolio Diversification under the Bayesian Framework

## Introduction

The project aims to perform a time series analysis of three major GPU  companies (Nvidia, AMD, Intel), to forecast their volatility using the well known GARCH model and lastly to propose possible strategies of  Portfolio Diversification based on the asset precision.

## The data

We've obtained historical stock data for Nvidia, AMD, and Intel from [this Kaggle dataset](https://www.kaggle.com/datasets/kapturovalexander/nvidia-amd-intel-asus-msi-share-prices?select=NVIDIA+(1999+-11.07.2023).csv). Our analysis focuses on the common time frame between January 25, 1999, and July 10, 2023, during which we computed both the *Return* and *Log Return* for each day.

<div>
  <img src=".\img\modified_dataset.png" alt="modified dataset" width=100%\>
<\div>

Here is a visualization of the three time series with their relatives *Log Returns*:

![](.\img\time_series_plot.png)

It's intriguing to observe that there are certain dates or periods, both in the *Adj.Close* and *LogReturns* features, during which all three time series exhibit similar spike behavior.

## The model

To forecast the volatility we followed the **GARCH** (**Generalized Autoregressive Conditional Heteroskedasticity**) model, which defines $\sigma^2_t$ as:
$$
\sigma^2_t = \omega + \sum\limits_{i=1}^I\alpha_i(y_{t-1}-\mu)^2 + \sum\limits_{i=1}^I\beta_i\sigma^2_{t-1} \quad\text{where }I\text{ is the number of lags}
$$
This was our setup:
$$
\begin{aligned}
\text{likelihood}&\begin{cases}
y \sim N(\mu,\sigma^{-2})\\
\end{cases}\\
\\
\text{priors}&\begin{cases}
\mu \sim N(0,100^{-2})\\
\omega \sim U(0,10)\\
\alpha \sim U(0,1)\\
\beta \sim U(0,1)
\end{cases}
\end{aligned}
$$
And here it is in R code:

```R
garch_model_code <- "
model
{
  # Likelihood
  for (t in 1:N) {
    y[t] ~ dnorm(mu, 1/pow(sigma[t], 2))
  }
  sigma[1] ~ dunif(0,10)
  for(t in 2:N) {
    sigma[t] <- sqrt(omega + alpha * pow(y[t-1] - mu, 2) + beta * pow(sigma[t-1], 2))
  }
  # Priors
  mu ~ dnorm(0, 0.01)
  omega ~ dunif(0, 10)
  alpha ~ dunif(0, 1)
  beta ~ dunif(0, 1)
}
"
```

## Forcasting Volatility

<div style="display: flex; flex-direction: row;">
    <img src=".\img\nvidia_forecasting.png" alt="nvidia forcasting" width="350" />
    	<img src=".\img\amd_forecasting.png" alt="amd forcasting" width="350" />
    	<img src=".\img\intel_forecasting.png" alt="intel forcasting" width="350" />
</div>

From the three plots above, it appears that all three of our GARCH models effectively captured the volatility between January 25, 1999, and July 10, 2023.

It's also worth noting that both our models for AMD and Intel have demonstrated an unexpected ability to forecast volatility, even for the period we initially excluded from our analysis (1980-1999).

<div style="display: flex; flex-direction: row;">
    <img src=".\img\amd_old_forecasting.png" alt="AMD old forecast" width="500" />
    <img src=".\img\intel_old_forecasting.png" alt="Intel old forecast" width="500" />
</div>

## The Portfolio Diversification strategies

- **Blind All-In** (Baseline): in which we give at each stock a random weight (Blind) and we trade all our budget (All-In).
- **Univariate diagonal (UD) portfolio**: this was a strategy proposed in a 2002 [paper](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=798247) by Momtchil Pojarliev and Wolfgang Polasek. It consists in weighting each stocks by the precision (inverse of $\sigma^2$) of the next day.
- **Optimized UD**: This is a personal adjustment to the **Univariate diagonal portfolio strategy** in which we don’t always buy all stocks each day, but we implement a decision strategy for which we buy a stock only if its $p_t$ (the forecasted precision) is above a certain threshold $\tilde{p}$.

## Results

Using the forcasted values for $\sigma^2$ and the proposed strategies we manage to increase the expected return by 23.08% in the first case and by 83.59% in the second one!

It also seems that the resulting portfolios have a better **Sharpe Ratio** score.

| Strategy     | Return | Sharpe Ratio | Return Increase from Baseline (%) | Sharpe Ratio Increase from Baseline (%) |
| ------------ | ------ | ------------ | --------------------------------- | --------------------------------------- |
| Blind All-In | 6.84   | 0.07         | 0.00                              | 0.00                                    |
| UD           | 8.42   | 0.09         | 23.08                             | 27.26                                   |
| Optimized UD | 12.56  | 0.13         | 83.59                             | 86.91                                   |

**Note**: To perform our evaluations we obtained the most recent two months of data for each company from [Yahoo Finance](https://finance.yahoo.com/), spanning from July 10, 2023, to September 8, 2023.

## Used Technologies

![RStudio](https://img.shields.io/badge/RStudio-4285F4?style=for-the-badge&logo=rstudio&logoColor=white)
![R](https://img.shields.io/badge/r-%23276DC3.svg?style=for-the-badge&logo=r&logoColor=white)
