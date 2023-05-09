---
layout: post
title:  "Modeling Callable Bonds"
date:   2023-04-18 14:34:25
categories: jekyll update
tags: 
image: /assets/article_images/2014-11-30-mediator_features/night-track.JPG
image2: /assets/article_images/2014-11-30-mediator_features/night-track-mobile.JPG
---
Modeling a Callable Bond using QuantLib.
---
A callable bond can also be viewed as a redeemable bond, meaning that the issuer can can redeem before it reaches maturity date. Another to look at this is as an American option. While European options do not let the buyer redeem before the maturity date, American options allow the buyer to redeem before maturity date.

These bonds generally come with certain restrictions on the call option. For example, the bonds may not be able to be redeemed in a specified initial period of their lifespan. In addition, some bonds allow the redemption of the bonds only in the case of some extraordinary events.

Callable bonds are attractive products if interest rates are expected to fall. In this case, the issuer can redeem the callable bonds and issue new bonds with lower coupon rates.

Example

Let's assume that AAPL (Apple Inc.) issues bonds with a face value of $1000 and a coupon rate of 5% while the current interest rate is 4% with a maturity of 10 years. This means that the bonds could be redeemed before maturity. From year 1 to year 5, there is no call option. Years 5-10 can be redeemed if interest rates decrease to refinance their debt and issue new bonds with more favorable coupon rates.

Basic Valuation 

Price(Callable Bond) = Price(Plain - Vanilla Bond) - Price(Call Option)

Where

* Price (Plain – Vanilla Bond) – the price of a plain-vanilla bond that shares similar features with the (callable) bond.

* Price (Call Option) – the price of a call option to redeem the bond before maturity.

QuantLib Implementation 

In QuantLib, modeling a Callable bond is much like a fixed rate bond with the additional input of a call or put schedule.

    # # Callable bonds (Calls)
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
    import numpy as np
    calcDate = ql.Date(16, 8, 2006) 
    ql.Settings.instance().evaluationDate = calcDate

    dayCount = ql.ActualActual(ql.ActualActual.Bond)
    rate = 0.0465
    termStructure = ql.FlatForward(calcDate, rate, dayCount, ql.Compounded, ql.Semiannual)
    term_Structure_Handle = ql.RelinkableYieldTermStructureHandle(termStructure)

    callabilitySchedule = ql.CallabilitySchedule()
    callPrice = 100.0
    callDate = ql.Date(15, ql.September, 2006); 
    nc = ql.NullCalendar()

    # Number of calldates is 24
    for i in range(0, 24):
        callabilityPrice  = ql.BondPrice(callPrice, ql.BondPrice.Clean)
        callabilitySchedule.append(ql.Callability(callabilityPrice, ql.Callability.Call, callDate))
        callDate = nc.advance(callDate, 3, ql.Months)

    issueDate = ql.Date(16, ql.September, 2004)
    maturityDate = ql.Date(15, ql.September, 2012)
    calendar = ql.UnitedStates(ql.UnitedStates.GovernmentBond)
    tenor = ql.Period(ql.Quarterly)
    accrualConvention = ql.Unadjusted
    schedule = ql.Schedule(issueDate, maturityDate, tenor, calendar, accrualConvention, accrualConvention, ql.DateGeneration.Backward, False)

    settlement_days = 3
    faceAmount = 100
    accrual_daycount = ql.ActualActual(ql.ActualActual.Bond)
    coupon = 0.025
    bond = ql.CallableFixedRateBond(settlement_days, faceAmount, schedule, [coupon], accrual_daycount, ql.Following, faceAmount, issueDate, callabilitySchedule)


    maxIterations = 1000
    accuracy = 1e-8
    gridIntervals = 40
    reversionParameter = .03


    def value_bond(a, s, grid_points, bond): 
        model = ql.HullWhite(term_Structure_Handle, a, s)
        engine = ql.TreeCallableFixedRateBondEngine(model, grid_points) 
        bond.setPricingEngine(engine)
        return bond

    def value_bond2(reversionParameter, s, gridIntervals, bond): 
        model2 = ql.HullWhite(term_Structure_Handle, reversionParameter, s)
        engine2 = ql.TreeCallableFixedRateBondEngine(model2, gridIntervals) 
        bond.setPricingEngine(engine2)
        return bond

    # 3% mean reversion 15% volatility
    value_bond(0.03, 0.15, 40, bond) 
    print("Bond price using clean price: ", bond.cleanPrice())
    print("Bond price using net present value: ", bond.NPV())

    # 6% mean reversion and 20% volatility
    value_bond2(0.06, 0.20, 40, bond) 
    print("Bond price using clean price: ", bond.cleanPrice())
    print("Bond price using net present value: ", bond.NPV())
