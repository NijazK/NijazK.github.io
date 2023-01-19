---
layout: post
title:  "Fixed Rate Models (C++, Python)"
date:   2023-01-18 14:34:25
categories: jekyll update
tags: 
image: /assets/article_images/2023-18-01-FixedRateModel/FixedRateModel.jpg
---
Modeling fixed rate instruments using industry standard practices.

#### Fixed Rate Bond
Let's assume a simple example for fixed rate bonds. Consider a bond with a par value of $100000, that pays 7% coupon semi-annually, issued on January 15th, 2015 and set to mature on January 15th, 2016. The bond will pay a coupon on July 15th, 2015 and January 15th, 2016. The par amount of 100000 will also be paid on the January 15th, 2016. The spot rates for the bond is 0.6% semi annually and 0.8% anually. 
First, we construct a yield curve with given spot rates using ZeroCurve from QuantLib. After, we will build a fixed rate object and in doing so, we need a stored array of coupon payments so the FixedRateBond() can pass as parameters. 

FixedRateBond():

    coupon_rate = .07
    coupons = [coupon_rate]
    settlement_days = 0
    face_value = 100000
    fixed_rate_bond = FixedRateBond(settlement_days, face_value, schedule, coupons, day_count)

Now, lets; use the QuantLib valuation engine DiscountBondEngine() to properly price the example bond. The DiscountBondEngine() takes the yield curve object as an argument

DiscountBondEngine():

    bond_engine = DiscountingBondEngine(spot_curve_handle)
    fixed_rate_bond.setPricingEngine(bond_engine)

Final yields:

    print(fixed_rate_bond)
    99310.83761241747
    

#### Callable Bonds
A callable bond is a type of fixed instrument that provides the issuer of the bond with the right to redeem the bond before its maturity date. Callable bonds usually have restrictions such as bonds may not be able to be redeemed ina specified initial period of their lifespan. A simple example of a callable bond is assume we have a 10 year maturity bond where the first 5 years of maturity have not call option, but the 6th - 10th year have a reddemable call option if the interest rate decreases. This is how we find value of a callable bond Price(Callable bond) = Price(plain - Vanilla Option) - Price(call Option) where the price of the vanilla bond shares similarities with the callable bond and the price of the call option to redeem the bond before maturity.

#### Modeling a Callable Bond
Callable bonds have similarities with fixed rate bonds with an extra parameter, which is a call or put scheduler. For this example, we assume a flat yield curve of 3.5%. The call price is at $1,000 and we use the container Callability.Call because it is a call option not a put option.

        callability_schedule = CallabilitySchedule()
        call_price = 1000.0
        call_date = Date(15, September, 2016); 
        null_calendar = NullCalendar();
        for i in range(0,24):
            callability_price  = ql.BondPrice(call_price, ql.BondPrice.Clean)
            callability_schedule.append(ql.Callability(callability_price, Callability.Call, call_date))
            call_date = null_calendar.advance(call_date, 3, Months)
            
We then initialize the callable bond with the CallableFixedRateBond class within QuantLib, which accepts inputs same as a fixed rate bond with additional call or put schedule.
        
        bond = CallableFixedRateBond(settlement_days, face_amount, schedule, [coupon], accrual_daycount, Following, face_amount, issue_date, callability_schedule)
        
Now we need an interest rate model for the callable bond because we need to take in two additional parameters; \mu as a mean reversion (3%) and volatility \sigma (12%). 

        
        def value_bond(a, s, grid_points, bond): 
            model = HullWhite(ts_handle, a, s)
            engine = TreeCallableFixedRateBondEngine(model, grid_points) 
            bond.setPricingEngine(engine)
            return bond
            
        value_bond(0.03, 0.12, 40, bond) 
        print("Bond price: ")
        bond.cleanPrice()

![](https://user-images.githubusercontent.com/75659218/213588206-11fb5739-b695-469e-8036-7f2c21f29c9e.png)

As we see the yield curve has a downward slope because if the volatility is increased, the bond value has a higher chance of being callable.

        
        
        
        
        
        
        
        
        
        
        
        
        
        
