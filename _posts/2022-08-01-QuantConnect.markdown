---
layout: post
title:  "QuantConnect Algorithm Research"
date:   2021-12-29 14:34:25
categories: jekyll update
tags: 
image: /assets/article_images/2022-08-01-QuantConnect/QuantConnect.jpg
---
## QuantConnect Algorithm Research

QuantConnect is an open-source financial technology platform that is currently paving the way for the future of trading. The platform is comprised of three main parts: Cloud Research, Backtesting, Parameter Optimization, and Live Trading. 

This repository aims at illustrating the convenience of backtesting and active portfolio management using the Python engine. More specifically, it will ustilize the backtesting and cloud research platforms that are supported by QuantConnect. Popular trading strategies include: Buy and hold, Pairs trading, ETF weighing, mean-reversion, and momentum-based strategies. I should also include that the repository does contain course solutions for the equities, forex, and options course. 

First Example: Liquid Value Stocks

        from AlgorithmImports import *
        from Selection.FundamentalUniverseSelectionModel import FundamentalUniverseSelectionModel
        class LiquidValueStocks(QCAlgorithm):

            def Initialize(self):
              self.SetStartDate(2016, 10, 1)
              self.SetEndDate(2017, 10, 1)
              self.SetCash(100000)
              self.UniverseSettings.Resolution = Resolution.Hour
              self.AddUniverseSelection(LiquidValueUniverseSelectionModel())
              self.AddAlpha(NullAlphaModel())
              self.SetPortfolioConstruction(EqualWeightingPortfolioConstructionModel())
              self.SetExecution(ImmediateExecutionModel())
            
Here is an example of the program class where we initialize the start date, end date, and any dat structures that will aid in the backtest such as self.UniverSettings.resolution = Resolution.Hour, which will parse the data in an hourly fashion to correct the algorithm if need be (more data points are required for hourly than daily). 

        class LiquidValueUniverseSelectionModel(FundamentalUniverseSelectionModel):
    
            def __init__(self):
              super().__init__(True, None)
              self.lastMonth = -1 
    
            def SelectCoarse(self, algorithm, coarse):
        
              #1. If it isn't time to update data, return the previous symbols 
              if self.lastMonth == algorithm.Time.month:
                return Universe.Unchanged
        
              #2. Update self.lastMonth with current month to make sure only process once per month
              self.lastMonth = algorithm.Time.month
        
              #3. Sort symbols by dollar volume and if they have fundamental data, in descending order
              sortedByDollarVolume = sorted([x for x in coarse if x.HasFundamentalData], key=lambda x: x.DollarVolume, reverse=True)
        
              #4. Return the top 100 Symbols by Dollar Volume 
              return [x.Symbol for x in sortedByDollarVolume[:100]]


This portion of the algorithm will execute the desired paramters of the quantitative trader. The beuaty within thos framework is the quantitative trader can set boundaries as to where they buy (stocks, sector, asset classes), whom they buy from (brokers), and what they buy (paramters such as PE Ratio, Earnings volume, and Trading volume). In the above example, we pass a function SelectCoarse(self, algorithm, coarse) with parameters of self, algorthm, and caorse universe data to set parameters on asset classes that are stocks, and that have relatively low dollar volume, which will correlate with a lower PE Ratio. 
