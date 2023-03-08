---
layout: post
title:  "Mean Reversion Cyclicals Stocks"
date:   2023-01-18 14:34:25
categories: jekyll update
tags: 
image: /assets/article_images/2023-18-01-FixedRateModel/FixedRateModel.jpg
image2: /assets/article_images/2023-18-01-FixedRateModel/FixedRateModel-mobile.jpg
---
Using QuantConnect to model a Mean Reversion trading strategy for cyclicla stocks.

## Cyclical Stocks.
Cyclical stocks are stocks that are affected by macroeconomic changes. For example, a car manufacturer's stock should be doing better in a good economy because many consumers might be able to afford newer cars during a good economic cycle. On the other hand, car manufacturers will not be doing so well when the economy is in a recession because buying new cars isn't readily available to many consumers during a recession. This results in higher volatility as well as market inefficiencies. 

I also should note that just because a stock is in a cyclical asset bracket does not mean the asset itself is cyclical. For example, Ford (F) is considered a defensive stock because of its low volatility (we will discuss the measurements further in the article) and dividend payment history. In addition, we will designate tech stocks as cyclical because of the bull run of 2016 - 2021 and the quick turnaround from 2022, which seems to still trend downwards. The mean reversion portfolio construction will trade volatility levels with respect to the historic volatility levels' average.

## Mean Reversion.
Mean-reversion or Reversion is a theoretical process in finance that suggests when a stock will eventually average out the longer the process. However, in the short-run, we can take advatnage of pricing miscalculations because within the given framework we can either short overpriced assets and/or buy lower priced (long). This is usually said in the matter "Buy low and sell high". The contexts that measure the mean can be through the volatility, (p/e) ratio, and the average return. For certain assets have underlying factors that influence the factors such as interest rates. 

## Trading Strategy
The basis for this strategy is to combine the certain cyclical assets and utilize mean reversion to construct a long-only portfolio. Therefore, this strategy should only be implemented if the asset is 1. Designated cyclical and 2. the asset has a relative low p/e ratio with respect to its historic p/e ratio levels.

![](https://user-images.githubusercontent.com/75659218/223863587-e7793a6c-8499-45d2-bef8-d63760073f04.png)
Using the equity curve for these specific assets in a 1 year 2020 - 2021 has a magnitude of < 1. However, the returns for this portfolio should be measured in smaller quantities, but for this example and the sakle of simplicity, we will measure the trading startegy as a constant 12 month long position. The reason for the stagnation is due to the lack of movement (< 1 ) standard deviation within the p/e ratio and thus the asset is held on for extended period.

## Backtest Results 
Through several iterations of backtests, I have found that typically a 1-month to couple week timeframe suits best for this strategy. 

The algorithm framework below is the mean reversion class used in QuantConnect. The three cyclical asset classes are: Technology, Healthcare, and Consumer.

    from AlgorithmImports import *
    from Portfolio.MeanReversionPortfolioConstructionModel import *

    class MeanReversionPortfolioAlgorithm(QCAlgorithm):
      '''Example algorithm of using MeanReversionPortfolioConstructionModel'''

      def Initialize(self):
          # Set starting date, cash and ending date of the backtest
          self.SetStartDate(2020, 1, 1)
          self.SetEndDate(2021, 1, 1)
          self.SetCash(100000000)

          self.SetSecurityInitializer(lambda security: security.SetMarketPrice(self.GetLastKnownPrice(security)))
        
          # Subscribe to data of the selected stocks
          self.techSymbols = [self.AddEquity(ticker, Resolution.Daily).Symbol for ticker in ["GOOGL", "TSM", "ZM", "AMD"] ]
          self.healthSymbols = [self.AddEquity(ticker, Resolution.Daily).Symbol for ticker in ["NVO", "GILD", "UTHR", "VRTX"] ]
          self.consumerSymbols = [self.AddEquity(ticker, Resolution.Daily).Symbol for ticker in ["HSY", "KR", "TSLA", "KDP"] ]
        

          self.AddAlpha(ConstantAlphaModel(InsightType.Price, InsightDirection.Up, timedelta(1)))
          self.SetPortfolioConstruction(MeanReversionPortfolioConstructionModel())


## 1-Month Trading Backtest
The results for the 1-month trading strategy has a loss rate 24% and win rate of 76%. This is inlfated due to the notion that we pre-selected certain assets (knowing the cyclical > 1 standard deviation) and thus know that the probability of winning trades tends to be higher. 
![](https://user-images.githubusercontent.com/75659218/223870578-bf2c4eac-a5c4-4619-bef0-98499d6a6d67.png)

## 2-Week Backtest Results
Assume a portfolio of $1 billion USD (1,000,000,000) and we want to optimize 1%, or $10 million to a two week strategy that uses cyclical assets using the mean reversion framework.
<img width="715" alt="Screenshot 2023-03-08 at 3 10 43 PM" src="https://user-images.githubusercontent.com/75659218/223873447-b59b94de-fdc9-451f-b71e-de1caa103532.png">

## 6-Month Backtest Results
For this backtest, we use 10x more liquidity than the above example to compensate transaction fees as well as use volume to our advantage. 
![](https://user-images.githubusercontent.com/75659218/223874500-2008e477-b36f-477b-8bc8-3cf35775b30b.png)
We see that an extended period for the mean reversion trading strategy is more suitable than a weekly or singular month because we the algorithm can emit better results the more timesteps it gets fed. We can also see the Sharpe ratio is much higher in a 6-month backtest than both 1-month and 2-week combined.

## 1-Year Backtest result
We'll assume the $1 billion USD portfolio example from before and only allocate 10% of the portfolio to this strategy ($100,000,000). 
![](https://user-images.githubusercontent.com/75659218/223875940-5b6bf96a-8adf-4d3d-b1d9-fd3442043179.png)
As we see, the sharpe ratio is just over 1 (still better than the 1-month and bi-weekly example!) with a compounding return of 34.37% and 65% win rate across 12 months.

## Conclusions
Mean reversion can be a great strategy when the correct parameters are being exercised. For this scimple exercise, I used parameters such as magnitude of pe/e ratio, 





















