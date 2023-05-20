---
layout: post
title:  "Crypto Derivatives: Tokenized Treasury Futures Contracts"
date:   2023-05-05 14:34:25
categories: jekyll update
tags: 
image: /assets/article_images/2014-11-30-mediator/night-track.jpg
image2: /assets/article_images/2014-11-30-mediator/night-track.jpg
---
## Modeling Tokenized Treasury Futures Contracts
---
US Treasury Bond futures and options are deeply liquid and efficient tools for hedging interest rate risk, potentially enhancing income, adjusting portfolio duration, interest rate speculation and spread trading. Tokenized treasuries are a soaring topic this past year due to the increase in Crypto money market funds that exceed $500 million (Coindesk).

As mentioned above, Treasuries ad other option derivatives can play a vital role in risk management due to the efficiency and adjustments for portfolio rebalance.

## Tokenized Treasury Futures Contract
Let's value treasury futures contract using QuantLib. The treasury futures contract gives the buyer the right to buy the underlying by the time the contract expires. The underlying is usually delivered from a basket of securities. So in order to properly value the futures contract, we would need to find the deliverable. Here we start by doing a naive calculation by constructing a fictional security. We will see what is wrong about this approach. As a next step we will perform the cheapest to deliver calculation and subsequently use that deliverable to value the same contract.

## Build Yield Curve
As a first step, we build the treasury curve out of the treasury securities such as T-Bills, T-Notes and Treasury bonds. 

## Treasury Futures
Here we want to understand how to model treasury futures contract. Let us look at the TYZ5, the treasury futures on the 10 year note for delivery in December 2015. The notional deliverable is a 10-year 6% coupon note. In reality, the seller of the futures contract can deliver from a basket of securities.
For now, lets assume that the deliverable is actually a 6% coupon 10-year note issued as of the calculation date. Let us construct a 10 year treasury note and value this security. The futures price for the TYZ5 is $127.0625.

## Cheapest To Deliver
Above we used a fictional 6% coupon bond as the deliverable. In reality, the deliverable is picked from a basket of securities based on what is the cheapest to deliver. Cheapest to deliver is not the
Treasury Futures Contract 182
cheapest in price. The seller of the futures contract, has to buy the delivery security from the market and sell it at an adjusted futures price. The adjusted futures price is given as:
Adjusted Futures Price = Futures Price x Conversion Factor
The gain or loss to the seller is given by the basis,
Basis = Cash Price - Adjusted Futures Price
So the cheapest to deliver is expected to be the security with the lowest basis. The conversion factor for a security is the price of the security with a 6% yield. Let us look at a basket of securities that is underlying this futures contract to understand this aspect.

## Conclusion
In this chapter, we looked into understanding and valuing treasury futures contracts. We used the QuantLib FixedRateBondForward class in order to model the futures contract. But, we also made a cheapest to deliver calculation to figure out the deliverable.

## Code

    import QuantLib as ql
    import math
    from pandas import DataFrame
    import numpy as np

    calcDate = ql.Date(30,11,2020)
    ql.Settings.instance().evaluationDate = calcDate
    dayCount = ql.ActualActual(ql.ActualActual.Bond)
    calendar = ql.UnitedStates()
    bussinessConvention = ql.Following
    settlement = 0
    faceAmount = 100000
    coupon = ql.Period(ql.Semiannual)

    ## Build Yield Curve
    marketQuotes = [99.1092, 99.5352,  99.9083, 99.6439, 99.0276,
              99.6987, 99.0330, 99.1297, 100.5862, 100.3765]

    couponRates = [0.0000, 0.0000, 0.0000, 0.0000, 0.00875,
                    0.0125, 0.01625, 0.02, 0.0225, 0.03]

    maturities = [ql.Date(24,12,2020), ql.Date(25,2,2021),
                      ql.Date(26,5,2021), ql.Date(10,11,2021),
                      ql.Date(30,11,2022), ql.Date(15,11,2023),
                      ql.Date(30,11,2024), ql.Date(30,11,2026),
                      ql.Date(15,11,2029), ql.Date(15,11,2050)]

    issues = [ql.Date(25,6,2015), ql.Date(27,8,2015),
                   ql.Date(28,5,2015), ql.Date(12,11,2015),
                   ql.Date(30,11,2015), ql.Date(16,11,2015),
                   ql.Date(30,11,2015), ql.Date(30,11,2015),
                   ql.Date(16,11,2015), ql.Date(16,11,2015)]

    couponFrequency = ql.Period(6, ql.Months)

    bondHelpers = []
    for coupon, issues, maturities, price in zip(couponRates, issues, maturities, marketQuotes): 
        schedule = ql.Schedule(calcDate, maturities, couponFrequency, calendar, bussinessConvention, bussinessConvention, ql.DateGeneration.Backward, False)
        helper = ql.FixedRateBondHelper(ql.QuoteHandle(ql.SimpleQuote(price)), settlement, faceAmount, schedule, [coupon], dayCount, bussinessConvention)
        bondHelpers.append(helper)

    yieldCurve = ql.PiecewiseCubicZero(calcDate, bondHelpers, dayCount) 
    yieldCurveHandle = ql.YieldTermStructureHandle(yieldCurve)   

    def treasuryFuturesContract(issueDates, bondMaturity, couponRate, couponFrequency = ql.Period(6, ql.Months), dayCount = ql.ActualActual(ql.ActualActual.Bond), calendar = ql.UnitedStates()):
        faceAmount = 100000.
        settlement = 0
        schedule = ql.Schedule(issueDates, bondMaturity, couponFrequency, calendar, ql.ModifiedFollowing, ql.ModifiedFollowing, ql.DateGeneration.Forward, False)
        security = ql.FixedRateBond(settlement, faceAmount, schedule, [coupon], dayCount)
        return security

    issueDates = calcDate
    deliveryDate = ql.Date(1,12,2020)

    bondMaturity = issueDates + ql.Period(10, ql.Years)
    dayCount = ql.ActualActual(ql.ActualActual.Bond)
    couponFrequency = ql.Period(6, ql.Months)
    couponRate = 5/100.

    deliverable = treasuryFuturesContract(issueDates, bondMaturity, couponRate, couponFrequency, dayCount, calendar)
    bondEngine = ql.DiscountingBondEngine(yieldCurveHandle)
    deliverable.setPricingEngine(bondEngine)

    futures_price = 127.00
    cleanPrice = futures_price*yieldCurve.discount(deliveryDate)
    zspread = ql.BondFunctions.zSpread(deliverable, cleanPrice, yieldCurve, dayCount, ql.Compounded, ql.Semiannual, calcDate) * 10000 
    print("Z-Spread =%3.0fbp" % (zspread))

    dayCount = ql.ActualActual(ql.ActualActual.Bond)
    basket = [(1.625, ql.Date(15,8,2022), 97.921875),
                      (1.625, ql.Date(15,11,2022), 97.671875),
                      (1.75, ql.Date(30,9,2022), 98.546875),
                      (1.75, ql.Date(15,5,2023), 97.984375),
                      (1.875, ql.Date(31,8,2022), 99.375),
                      (1.875, ql.Date(31,10,2022),99.296875),
                      (2.0, ql.Date(31,7,2022), 100.265625),
                      (2.0, ql.Date(15,2,2023), 100.0625),
                      (2.0, ql.Date(15,2,2025), 98.296875),
                      (2.0, ql.Date(15,8,2025), 98.09375),
                      (2.125, ql.Date(30,6,2022), 101.06250),
                      (2.125, ql.Date(15,5,2025),99.25),
                      (2.25, ql.Date(15,11,2024),100.546875),
                      (2.25, ql.Date(15,11,2025),100.375),
                      (2.375, ql.Date(15,8,2024),101.671875),
                      (2.5, ql.Date(15,8,2023),103.25),
                      (2.5, ql.Date(15,5,2024),102.796875),
                      (2.75, ql.Date(15,11,2023),105.0625),
                      (2.75, ql.Date(15,2,2024),104.875)]


    securities = []
    minBasis = 100; 
    minBasisIndex = -1 
    for i, b in enumerate(basket):
        coupon, maturity, price = b
        issue = maturity - ql.Period(10, ql.Years)
        s = treasuryFuturesContract(issue, maturity, coupon/100.)
        bondEngine = ql.DiscountingBondEngine(yieldCurveHandle)
        s.setPricingEngine(bondEngine)
        cf = ql.BondFunctions.cleanPrice(s, 0.06, dayCount, ql.Compounded, ql.Semiannual, calcDate)/100.
        adjusted_futures_price = futures_price * cf
        basis = price - adjusted_futures_price
        if basis < minBasis: 
            minBasis = basis 
            minBasisIndex = i
        securities.append((s,cf))

        ctd_info = basket[minBasisIndex]
        ctd_bond,ctd_cf = securities[minBasisIndex] 
        ctd_price = ctd_info[2]

    print("%-30s = %lf" % ("Minimum Basis", minBasis)) 
    print("%-30s = %lf" % ("Conversion Factor", ctd_cf)) 
    print("%-30s = %lf" % ("Coupon", ctd_info[0]))
    print("%-30s = %s" % ("Maturity", ctd_info[1])) 
    print("%-30s = %lf" % ("Price", ctd_info[2]))



