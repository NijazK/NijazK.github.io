---
layout: post
title:  "Modeling Caps and Floors"
date:   2023-05-05 14:34:25
categories: jekyll update
tags: 
image: /assets/article_images/2014-11-30-mediator/night-track.jpg
image2: /assets/article_images/2014-11-30-mediator/night-track.jpg
---
## Modeling Caps and Floors 
---


## Caps

A "cap" is a type of derivative that gives the holder the right, but not the obligation, to buy an underlying asset at a predetermined price on or before a specified date (the "expiration date"). The underlying asset in a cap can be anything from a stock or commodity to a currency or interest rate. For this example, let's say that an investor is concerned about a potential decline with the price of AAPL stock over the next ten months. The investor could purchase a 10-month AAPL stock price cap with a strike price of $125. If the price of AAPL stock falls below $125 at any point over the next ten months, the investor can exercise the option and buy the stock at $125, regardless of the actual market price. If the price of AAPL stock never falls below $125, the investor simply lets the option expire worthless.

## Floors

A "floor" is the opposite of a cap. It is a type of derivative that gives the holder the right, but not the obligation, to sell a underlying asset at a predetermined price on or before a specified date (the "expiration date"). As with a cap, the underlying asset in a floor can be anything from a stock or commodity to a currency or interest rate. For this example, let's say that an investor is concerned about a potential increase in the price of AAPL stock over the next ten months. The investor could purchase a 10-month AAPL stock price floor with a strike price of $125. If the price of AAPL stock rises above $125 at any point over the next six months, the investor can exercise the option and sell the stock at $125, regardless of the actual market price. If the price of AAPL stock never rises above $125, the investor simply lets the option expire worthless.

## Pricing

For this example, we price the cap and floor using constant volatility and we will use the Black-Scholes pricing engine. Also, this example does not price an equity cap and floor but rather an interest rate cap and floor. 

Black Model is used to price the interest rate cap and floor because of the log-normal distribution and volatility caps and floors follow.
![](https://github.com/NijazK/nijazk.github.io/assets/75659218/e403c037-14ab-4071-9a28-4b69dd7688d2)
P(0,T) is today's discount factor for T, F is the forward price of the rate, K is the strike, N is the standard normal CDF.
![](https://github.com/NijazK/nijazk.github.io/assets/75659218/0549d2ca-190c-4c8d-8129-1bef310325ad)
![](https://github.com/NijazK/nijazk.github.io/assets/75659218/8eb3ddcc-beb2-4e8c-91e9-ecd8171d9826)
LIBOR rates equal to ![](https://github.com/NijazK/nijazk.github.io/assets/75659218/f5c7f3ce-8f2b-4454-8f4f-1fa878aca6bd)

## QuantLib Implementation (Python)

    # # Caps and Floors
    #
    # This file is part of QuantLib, a free-software/open-source library
    # for financial quantitative analysts and developers - https://www.quantlib.org/
    #
    # QuantLib is free software: you can redistribute it and/or modify it under the
    # terms of the QuantLib license.  You should have received a copy of the
    # license along with this program; if not, please email
    # <quantlib-dev@lists.sf.net>. The license is also available online at
    # <https://www.quantlib.org/license.shtml>.
    #
    # This program is distributed in the hope that it will be useful, but WITHOUT
    # ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
    # FOR A PARTICULAR PURPOSE.  See the license for more details.

    import QuantLib as ql

    calcDate = ql.Date(14, 6, 2016)
    ql.Settings.instance().evaluationDate = calcDate

    dates = [ql.Date(14,6,2016), ql.Date(14,9,2016),
             ql.Date(14,12,2016), ql.Date(14,6,2017),
             ql.Date(14,6,2019), ql.Date(14,6,2021),
             ql.Date(15,6,2026), ql.Date(16,6,2031),
             ql.Date(16,6,2036), ql.Date(14,6,2046)]

    yields = [0.000000, 0.006616, 0.007049, 0.007795,
              0.009599, 0.011203, 0.015068, 0.017583,
              0.018998, 0.020080]

    dayCount = ql.ActualActual(ql.ActualActual.Bond)
    calendar = ql.UnitedStates(ql.UnitedStates.GovernmentBond)
    interpolation = ql.Linear()
    compounding = ql.Compounded
    compoundingFrequency = ql.Annual
    term_structure = ql.ZeroCurve(dates, yields, dayCount, calendar, interpolation, compounding, compoundingFrequency)
    ts_handle = ql.YieldTermStructureHandle(term_structure)

    start_date = ql.Date(14, 6, 2016)
    end_date = ql.Date(14, 6 , 2026)
    period = ql.Period(3, ql.Months)
    buss_convention = ql.ModifiedFollowing
    rule = ql.DateGeneration.Forward
    end_of_month = False
    schedule = ql.Schedule(start_date, end_date, period, calendar, buss_convention, buss_convention, rule, end_of_month)

    iborIndex = ql.USDLibor(ql.Period(3, ql.Months), ts_handle)
    iborIndex.addFixing(ql.Date(10,6,2016), 0.0065560)
    ibor_leg = ql.IborLeg([1000000], schedule, iborIndex)

    strike = 0.02
    cap = ql.Cap(ibor_leg, [strike])
    vols = ql.QuoteHandle(ql.SimpleQuote(0.547295))
    engine = ql.BlackCapFloorEngine(ts_handle, vols)
    cap.setPricingEngine(engine)
    print("Value of Caps given constant volatility:", cap.NPV())
    
    Value of Caps given constant volatility: 54361.469655539724
