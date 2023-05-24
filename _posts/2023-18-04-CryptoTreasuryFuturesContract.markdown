---
layout: post
title:  "Crypto Derivatives: Tokenized Treasury Futures Contracts"
date:   2023-05-18 14:34:25
categories: jekyll update
tags: 
image: /assets/article_images/2014-11-30-mediator/night-track.jpg
image2: /assets/article_images/2014-11-30-mediator/night-track.jpg
---
# Modeling Tokenized Treasury Futures Contracts
---
US Treasury Bond futures and options are deeply liquid and efficient tools for hedging interest rate risk, potentially enhancing income, adjusting portfolio duration, interest rate speculation and spread trading. Tokenized treasuries are a soaring topic this past year due to the increase in Crypto money market funds that exceed $500 million (Coindesk).

As mentioned above, Treasuries ad other option derivatives can play a vital role in risk management due to the efficiency and adjustments for portfolio rebalance.

# Design Architecture
Lets assume that the token deliverable is actually a 6% coupon 10-year note issued as of the calculation date. Let us construct a 10 year token treasury note and value this security. The futures price for the ZNZ3 is $1000.00.

## Yield Curve
As a first step, we build the treasury curve out of the tokenized treasury securities.

## Treasury Futures
Here we want to understand how to model tokenized treasury futures contract. Let us look at the ZNZ3, we'll call our tokenized treasury futures a 10-year (CZNZ3). The tokenized treasury futures on the 10 year note for delivery in December 2025. The notional deliverable is a 10-year 6% coupon note. Therefore, our tokenized treasury future is a 10-year note that matures on December 2025.

## Cheapest To Deliver
For this example, the tokenized deliverable is picked from a basket of securities based on what is the cheapest to deliver. The seller of the token futures contract has to buy the delivery security from the crypto exchange and sell it at an adjusted futures price. Referencing the CME, the adjusted price is: Adjusted Futures Price = Futures Price x Conversion Factor The gain or loss to the seller is given by the basis, Basis = Cash Price - Adjusted Futures Price. So the cheapest to deliver is expected to be the security with the lowest basis. The conversion factor for a security is the price of the security with a 6% yield. Let us look at a basket of securities that is underlying this futures contract to understand this aspect.

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
    marketQuotes = [999.1092, 999.5352,  999.9083, 999.6439, 999.0276,
              999.6987, 999.0330, 999.1297, 1000.5862, 1000.3765]

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

    def tokenTreasuryFuturesContract(issueDates, bondMaturity, couponRate, couponFrequency = ql.Period(6, ql.Months), dayCount = ql.ActualActual(ql.ActualActual.Bond), calendar = ql.UnitedStates()):
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

    deliverable = tokenTreasuryFuturesContract(issueDates, bondMaturity, couponRate, couponFrequency, dayCount, calendar)
    bondEngine = ql.DiscountingBondEngine(yieldCurveHandle)
    deliverable.setPricingEngine(bondEngine)

    futures_price = 1000.00
    cleanPrice = futures_price*yieldCurve.discount(deliveryDate)
    zspread = ql.BondFunctions.zSpread(deliverable, cleanPrice, yieldCurve, dayCount, ql.Compounded, ql.Semiannual, calcDate) * 10000 
    print("Z-Spread =%3.0fbp" % (zspread))
    
    Z-Spread =-191bp

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

    print("%-20s: %4.4f" % ("Minimum Basis", minBasis)) 
    print("%-20s: %4.4f" % ("Coupon", ctd_info[0]))
    print("%-20s: %4.4f" % ("Maturity", ctd_info[1])) 
    print("%-20s: %4.4f" % ("Price", ctd_info[2]))
    
    Minimum Basis                  = -1204.876684
    Coupon                         = 2.250000
    Maturity                       = December 15th, 2025
    Price                          = 100.375000
    
# Micro Options on Token Futures Contracts
Options on token treasury futures (10 Yr Note ZNZ3) can be valued using the Black formula. Let us value a Call option maturing on December 25, 2025, with a strike of $1,119. The current futures price is 1,200 and the volatility is 20%. The risk free rate as of December 1, 2015 is 0.105%. Let us value this Call option as of December 1, 2015.

    # Crypto Treasuries Futures Contracts
    import QuantLib as ql
    import math

    calendar = ql.UnitedStates()
    business_convention = ql.ModifiedFollowing
    settlement_days = 0
    dayCount = ql.ActualActual(ql.ActualActual.Bond)

    interestRate = 0.0011
    calcDate = ql.Date(1, 1, 2022)
    yieldCurve = ql.FlatForward(calcDate, interestRate, dayCount, ql.Continuous)

    ql.Settings.instance().evaluationDate = calcDate
    T = 31/365

    maturity = ql.Date(28, 1, 2023)

    strike = 1000
    spot = 25860
    volatility = 0.20
    type = ql.Option.Call

    discount = yieldCurve.discount(maturity)

    time = yieldCurve.dayCounter().yearFraction(calcDate, maturity)

    stddev = volatility*math.sqrt(time)

    strikepayoff = ql.PlainVanillaPayoff(type, strike)

    black = ql.BlackCalculator(strikepayoff, spot, stddev, discount)

    print("%-20s: %4.4f" %("Option values on Treasury Futures Put", black.value()))
    print("%-20s: %4.4f" %("Delta", black.delta(spot)))
    print("%-20s: %4.4f" %("Gamma", black.gamma(spot)))
    print("%-20s: %4.4f" %("Theta", black.theta(spot, T)))
    print("%-20s: %4.4f" %("Vega", black.vega(T)))
    print("%-20s: %4.4f" %("Rho", black.rho( T)))

    Option values on Treasury Futures Put: 24830.3928
    Delta               : 0.9988
    Gamma               : 0.0000
    Theta               : 348.3931
    Vega                : 0.0000
    Rho                 : 84.8304



## Conclusion
In this article, we created a tokenized treasury futures contract from an underlying U.S Treasury note. We looked into understanding and valuing treasury futures contracts. We used the QuantLib FixedRateBondForward class in order to model the tokenized futures contract. In addition, we also used synthetic market data to properly structure the tokenized futures contract between two parties.

## Bibliography

U.S. treasury Bond Overview - CME Group. Futures &amp; Options Trading for Risk Management - CME Group. (n.d.). 
    https://www.cmegroup.com/markets/interest-rates/us-treasury/30-year-us-treasury-bond.html 
    
Ballabio, L. (2020). Implementing quantlib: Quantitative finance in C++: An inside look at the architecture of the Quantlib Library. A Lean Pub.
