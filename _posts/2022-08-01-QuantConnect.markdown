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

Example (1): Liquid Value Stocks

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

Example (2): BlackRock ETF Regressional iShares Model

We will start the algorithm by allocating symbols IVV (iShares Core S&P 500) and IEFA (iShares core MSCI EAFE) with the allocated amount. Then, the algorithm will create an empty storage array for changed symbols that get selected for the coarse selection universe. However, One can add additional filtering options for the algorithm if you want additional parameters. For this example, I only allowed a pointer [c] to add IEFA, IJR, IJH, and IEMG with no additional parameters. If the security should get delisted, it will get into the RemovedSecurities array and the allocated capital shall get a 70-30 split between the two ETFs with the most capital (IVV, IEFA).

        from AlgorithmImports import *

        ### <summary>
        ### Universe Selection regression algorithm simulates for Blackrock Advisors, LLC. 
        ### </summary>
        ### <meta name="tag" content="regression test" />
        class UniverseSelectionRegressionAlgorithm(QCAlgorithm):
    
                def Initialize(self):
        
                        self.SetStartDate(2017,3,1)   #Set Start Date
                        self.SetEndDate(2022,11,27)      #Set End Date
                        self.SetCash(1000000)           #Set Strategy Cash
                        # Find more symbols here: http://quantconnect.com/data
                        # security that exists with no mappings
                        self.AddEquity("IVV", Resolution.Daily)
                        # security that doesn't exist until half way in backtest (comes in as IEFA)
                        self.AddEquity("IEFA", Resolution.Daily)

                        self.UniverseSettings.Resolution = Resolution.Daily
                        self.AddUniverse(self.CoarseSelectionFunction)
        
                        self.changedSymbols = []
                        self.changes = None


                def CoarseSelectionFunction(self, coarse):
                        return [ c.Symbol for c in coarse if c.Symbol.Value == "IEFA" or c.Symbol.Value == "IJR" or c.Symbol.Value == "IJH" or                                   c.Symbol.Value == "IEMG" ]

                def OnData(self, data):
                        if self.Transactions.OrdersCount == 0:
                                self.MarketOrder("IVV", 70)
                                self.MarketOrder("IEFA", 30)

                        for kvp in data.Delistings:
                                self.changedSymbols.append(kvp.Key)
        
                        if self.changes is None:
                                return

                        if not all(data.Bars.ContainsKey(x.Symbol) for x in self.changes.AddedSecurities):
                                return 
        
                        for security in self.changes.AddedSecurities:
                                self.Log("{0}: Added Security: {1}".format(self.Time, security.Symbol))
                                self.MarketOnOpenOrder(security.Symbol, 100)

                        for security in self.changes.RemovedSecurities:
                                self.Log("{0}: Removed Security: {1}".format(self.Time, security.Symbol))
                                if security.Symbol not in self.changedSymbols:
                                self.Log("Not in delisted: {0}:".format(security.Symbol))
                                self.MarketOnOpenOrder(security.Symbol, -100)

                        self.changes = None 


                def OnSecuritiesChanged(self, changes):
                        self.changes = changes


                def OnOrderEvent(self, orderEvent):
                        if orderEvent.Status == OrderStatus.Submitted:
                                self.Log("{0}: Submitted: {1}".format(self.Time, self.Transactions.GetOrderById(orderEvent.OrderId)))
                        if orderEvent.Status == OrderStatus.Filled:
                                self.Log("{0}: Filled: {1}".format(self.Time, self.Transactions.GetOrderById(orderEvent.OrderId)))

As we can see, there are a lot of different varieties of portfolios that can be created from using QuantConnect. If you should use one of these tests, you can trade live locally to maximize the CPU output (more parameters will use more data points which might restrict trade executions). 
