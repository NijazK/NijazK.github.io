---
layout: post
title:  "Modeling Callable Bonds"
date:   2023-04-18 14:34:25
categories: jekyll update
tags: 
image: /assets/article_images/2014-11-30-mediator_features/night-track.JPG
image2: /assets/article_images/2014-11-30-mediator_features/night-track-mobile.JPG
---
Modeling a Callable Bond using QuantLib and Singleton Design Patterns
---
A callable bond can also be viewed as a redeemable bond, meaning that the issuer can can redeem before it reaches maturity date. Another to look at this is as an American option. While European options do not let the buyer redeem before the maturity date, American options allow the buyer to redeem before maturity date.

These bonds generally come with certain restrictions on the call option. For example, the bonds may not be able to be redeemed in a specified initial period of their lifespan. In addition, some bonds allow the redemption of the bonds only in the case of some extraordinary events.

Callable bonds are attractive products if interest rates are expected to fall. In this case, the issuer can redeem the callable bonds and issue new bonds with lower coupon rates.

## Example

Let's assume that AAPL (Apple Inc.) issues bonds with a face value of $1000 and a coupon rate of 5% while the current interest rate is 4% with a maturity of 10 years. This means that the bonds could be redeemed before maturity. From year 1 to year 5, there is no call option. Years 5-10 can be redeemed if interest rates decrease to refinance their debt and issue new bonds with more favorable coupon rates. The Hull-White model calculates the price of a derivative security as a function of the entire yield curve rather than a single rate (

Basic Valuation 

Price(Callable Bond) = Price(Plain - Vanilla Bond) - Price(Call Option)

Where

* Price (Plain – Vanilla Bond) – the price of a plain-vanilla bond that shares similar features with the (callable) bond.
* Price (Call Option) – the price of a call option to redeem the bond before maturity.

## Pricing Callable Bonds with Hull White Single-Factor.

Let's use the Hull-White model to price the Callable Bond. The Hull-White model assumes that short rates have a normal distribution and that the short rates are subject to mean reversion. Volatility is thus likely to be low when short rates are near zero, which is reflected in a larger mean reversion in the model. The Hull-White model calculates the price of a derivative security through a static yield curve (fixed rate). 

QuantLib Implementation (Python)

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


QuantLib Implementation (C++)

    /* -*- mode: c++; tab-width: 4; indent-tabs-mode: nil; c-basic-offset: 4 -*- */

    /*!
     Copyright (C) 2008 Allen Kuo

     This file is part of QuantLib, a free-software/open-source library
     for financial quantitative analysts and developers - http://quantlib.org/

     QuantLib is free software: you can redistribute it and/or modify it
     under the terms of the QuantLib license.  You should have received a
     copy of the license along with this program; if not, please email
     <quantlib-dev@lists.sf.net>. The license is also available online at
     <http://quantlib.org/license.shtml>.

     This program is distributed in the hope that it will be useful, but WITHOUT
     ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
     FOR A PARTICULAR PURPOSE.  See the license for more details.
     */

    /* This example sets up a callable fixed rate bond with a Hull White pricing
       engine and compares to Bloomberg's Hull White price/yield calculations.
    */

    #include <ql/qldefines.hpp>
    #if !defined(BOOST_ALL_NO_LIB) && defined(BOOST_MSVC)
    #  include <ql/auto_link.hpp>
    #endif
    #include <ql/experimental/callablebonds/callablebond.hpp>
    #include <ql/experimental/callablebonds/treecallablebondengine.hpp>
    #include <ql/models/shortrate/onefactormodels/hullwhite.hpp>
    #include <ql/termstructures/yield/flatforward.hpp>
    #include <ql/time/calendars/unitedstates.hpp>
    #include <ql/time/daycounters/actualactual.hpp>

    #include <vector>
    #include <cmath>
    #include <iomanip>
    #include <iostream>

    using namespace std;
    using namespace QuantLib;

    #if defined(QL_ENABLE_SESSIONS)
    namespace QuantLib {
        ThreadKey sessionId() { return {}; }
    }
    #endif


    ext::shared_ptr<YieldTermStructure>
        flatRate(const Date& today,
                 const ext::shared_ptr<Quote>& forward,
                 const DayCounter& dc,
                 const Compounding& compounding,
                 const Frequency& frequency) {
        return ext::shared_ptr<YieldTermStructure>(
                                           new FlatForward(today,
                                                           Handle<Quote>(forward),
                                                           dc,
                                                           compounding,
                                                           frequency));
    }


    ext::shared_ptr<YieldTermStructure>
        flatRate(const Date& today,
                 Rate forward,
                 const DayCounter& dc,
                 const Compounding &compounding,
                 const Frequency &frequency) {
        return flatRate(today,
                ext::shared_ptr<Quote>(new SimpleQuote(forward)),
                dc,
                compounding,
                frequency);
    }


    int main(int, char* [])
    {
        try {


            Date today = Date(16,October,2007);
            Settings::instance().evaluationDate() = today;

            cout <<  endl;
            cout << "Pricing a callable fixed rate bond using" << endl;
            cout << "Hull White model w/ reversion parameter = 0.03" << endl;
            cout << "BAC4.65 09/15/12  ISIN: US06060WBJ36" << endl;
            cout << "roughly five year tenor, ";
            cout << "quarterly coupon and call dates" << endl;
            cout << "reference date is : " << today << endl << endl;

            /* Bloomberg OAS1: "N" model (Hull White)
               varying volatility parameter

               The curve entered into Bloomberg OAS1 is a flat curve,
               at constant yield = 5.5%, semiannual compounding.
               Assume here OAS1 curve uses an ACT/ACT day counter,
               as documented in PFC1 as a "default" in the latter case.
            */

            // set up a flat curve corresponding to Bloomberg flat curve

            Rate bbCurveRate = 0.055;
            DayCounter bbDayCounter = ActualActual(ActualActual::Bond);
            InterestRate bbIR(bbCurveRate,bbDayCounter,Compounded,Semiannual);

            Handle<YieldTermStructure> termStructure(flatRate(today,
                                                              bbIR.rate(),
                                                              bbIR.dayCounter(),
                                                              bbIR.compounding(),
                                                              bbIR.frequency()));

            // set up the call schedule

            CallabilitySchedule callSchedule;
            Real callPrice = 100.;
            Size numberOfCallDates = 24;
            Date callDate = Date(15,September,2006);

            for (Size i=0; i< numberOfCallDates; i++) {
                Calendar nullCalendar = NullCalendar();

                Bond::Price myPrice(callPrice, Bond::Price::Clean);
                callSchedule.push_back(
                    ext::make_shared<Callability>(
                                        myPrice,
                                        Callability::Call,
                                        callDate ));
                callDate = nullCalendar.advance(callDate, 3, Months);
            }


            // set up the callable bond

            Date dated = Date(16,September,2004);
            Date issue = dated;
            Date maturity = Date(15,September,2012);
            Natural settlementDays = 3;  // Bloomberg OAS1 settle is Oct 19, 2007
            Calendar bondCalendar = UnitedStates(UnitedStates::GovernmentBond);
            Real coupon = .0465;
            Frequency frequency = Quarterly;
            Real redemption = 100.0;
            Real faceAmount = 100.0;

            /* The 30/360 day counter Bloomberg uses for this bond cannot
               reproduce the US Bond/ISMA (constant) cashflows used in PFC1.
               Therefore use ActAct(Bond)
            */
            DayCounter bondDayCounter = ActualActual(ActualActual::Bond);

            // PFC1 shows no indication dates are being adjusted
            // for weekends/holidays for vanilla bonds
            BusinessDayConvention accrualConvention = Unadjusted;
            BusinessDayConvention paymentConvention = Unadjusted;

            Schedule sch(dated, maturity, Period(frequency), bondCalendar,
                         accrualConvention, accrualConvention,
                         DateGeneration::Backward, false);

            Size maxIterations = 1000;
            Real accuracy = 1e-8;
            Integer gridIntervals = 40;
            Real reversionParameter = .03;

            // output price/yield results for varying volatility parameter

            Real sigma = QL_EPSILON; // core dumps if zero on Cygwin

            ext::shared_ptr<ShortRateModel> hw0(
                           new HullWhite(termStructure,reversionParameter,sigma));

            ext::shared_ptr<PricingEngine> engine0(
                          new TreeCallableFixedRateBondEngine(hw0,gridIntervals));

            CallableFixedRateBond callableBond(settlementDays, faceAmount, sch,
                                               vector<Rate>(1, coupon),
                                               bondDayCounter, paymentConvention,
                                               redemption, issue, callSchedule);
            callableBond.setPricingEngine(engine0);

            cout << setprecision(2)
                 << showpoint
                 << fixed
                 << "sigma/vol (%) = "
                 << 100.*sigma
                 << endl;

            cout << "QuantLib price/yld (%)  ";
            cout << callableBond.cleanPrice() << " / "
                 << 100. * callableBond.yield(bondDayCounter,
                                              Compounded,
                                              frequency,
                                              accuracy,
                                              maxIterations)
                 << endl;

            cout << "Bloomberg price/yld (%) ";
            cout << "96.50 / 5.47"
                 << endl
                 << endl;

            sigma = .01;

            cout << "sigma/vol (%) = " << 100.*sigma << endl;

            ext::shared_ptr<ShortRateModel> hw1(
                           new HullWhite(termStructure,reversionParameter,sigma));

            ext::shared_ptr<PricingEngine> engine1(
                          new TreeCallableFixedRateBondEngine(hw1,gridIntervals));

            callableBond.setPricingEngine(engine1);

            cout << "QuantLib price/yld (%)  ";
            cout << callableBond.cleanPrice() << " / "
                 << 100.* callableBond.yield(bondDayCounter,
                                             Compounded,
                                             frequency,
                                             accuracy,
                                             maxIterations)
                 << endl;

            cout << "Bloomberg price/yld (%) ";
            cout << "95.68 / 5.66"
                 << endl
                 << endl;

            ////////////////////

            sigma = .03;

            ext::shared_ptr<ShortRateModel> hw2(
                         new HullWhite(termStructure, reversionParameter, sigma));

            ext::shared_ptr<PricingEngine> engine2(
                          new TreeCallableFixedRateBondEngine(hw2,gridIntervals));

            callableBond.setPricingEngine(engine2);

            cout << "sigma/vol (%) = "
                 << 100.*sigma
                 << endl;

            cout << "QuantLib price/yld (%)  ";
            cout << callableBond.cleanPrice() << " / "
                 << 100. * callableBond.yield(bondDayCounter,
                                              Compounded,
                                              frequency,
                                              accuracy,
                                              maxIterations)
                 << endl;

            cout << "Bloomberg price/yld (%) ";
            cout << "92.34 / 6.49"
                 << endl
                 << endl;

            ////////////////////////////

            sigma = .06;

            ext::shared_ptr<ShortRateModel> hw3(
                         new HullWhite(termStructure, reversionParameter, sigma));

            ext::shared_ptr<PricingEngine> engine3(
                          new TreeCallableFixedRateBondEngine(hw3,gridIntervals));

            callableBond.setPricingEngine(engine3);

            cout << "sigma/vol (%) = "
                 << 100.*sigma
                 << endl;

            cout << "QuantLib price/yld (%)  ";
            cout << callableBond.cleanPrice() << " / "
                 << 100. * callableBond.yield(bondDayCounter,
                                              Compounded,
                                              frequency,
                                              accuracy,
                                              maxIterations)
                 << endl;

            cout << "Bloomberg price/yld (%) ";
            cout << "87.16 / 7.83"
                 << endl
                 << endl;

            /////////////////////////

            sigma = .12;

            ext::shared_ptr<ShortRateModel> hw4(
                         new HullWhite(termStructure, reversionParameter, sigma));

            ext::shared_ptr<PricingEngine> engine4(
                          new TreeCallableFixedRateBondEngine(hw4,gridIntervals));

            callableBond.setPricingEngine(engine4);

            cout << "sigma/vol (%) = "
                 << 100.*sigma
                 << endl;

            cout << "QuantLib price/yld (%)  ";
            cout << callableBond.cleanPrice() << " / "
                 << 100.* callableBond.yield(bondDayCounter,
                                             Compounded,
                                             frequency,
                                             accuracy,
                                             maxIterations)
                 << endl;

            cout << "Bloomberg price/yld (%) ";
            cout << "77.31 / 10.65"
                 << endl
                 << endl;

            return 0;

        } catch (std::exception& e) {
            std::cerr << e.what() << std::endl;
            return 1;
        } catch (...) {
            std::cerr << "unknown error" << std::endl;
            return 1;
        }
    }
