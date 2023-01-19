---
layout: post
title:  "Fixed Rate Models (C++, Python)"
date:   2021-12-29 14:34:25
categories: jekyll update
tags: 
image: /assets/article_images/2023-18-01-FixedRateModel/FixedRateModel.jpg
---
## Modeling fixed rate instruments using industry standard practices.

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

    fixed_rate_bond.NPV()
    99310.83761241747
    

#### Callable Bonds

