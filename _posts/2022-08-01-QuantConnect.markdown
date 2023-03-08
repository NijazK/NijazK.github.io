---
layout: post
title:  "Regressional ETFs Trading"
date:   2022-08-15 14:34:25
categories: jekyll update
tags: 
image: /assets/article_images/2022-08-01-QuantConnect/QuantConnect.jpg
image2: /assets/article_images/2022-08-01-QuantConnect/QuantConnect-mobile.jpg
---




## Abstract
We will start the algorithm by allocating symbols IVV (iShares Core S&P 500) and IEFA (iShares core MSCI EAFE) with the allocated amount. Then, the algorithm will create an empty storage array for changed symbols that get selected for the coarse selection universe. However, One can add additional filtering options for the algorithm if you want additional parameters. For this example, I only allowed a pointer [c] to add IEFA, IJR, IJH, and IEMG with no additional parameters. If the security should get delisted, it will get into the RemovedSecurities array and the allocated capital shall get a 70-30 split between the two ETFs with the most capital (IVV, IEFA).

## Methodology 

        {% highlight Python %}
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
        {% endhighlight %}
                                

