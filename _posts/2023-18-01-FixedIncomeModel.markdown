---
layout: post
title:  "Modeling Options Commodities using Black 76 Formula"
date:   2023-01-18 14:34:25
categories: jekyll update
tags: 
image: /assets/article_images/2023-18-01-FixedRateModel/FixedRateModel.jpg
image2: /assets/article_images/2023-18-01-FixedRateModel/FixedRateModel-mobile.jpg
---
Modeling Options commodities with Black 76 and QuantLib

####  Mathematics and Assumptions
* Use the Black-Scholes equation to price commodity futures options.
* We will not model commodities as the same as equity options because the prices do not fluctuate as much. Therefore, the commodiities will not be in a stochastic process.

* Futures prices are log-normally distributed.
* All options will be exercised at maturity.
* Volatility is not constant (depends on time to maturity)

#### Use a stochastic process to model the time, t, and Ft:
![](https://user-images.githubusercontent.com/75659218/224573587-684e8b7d-b46a-43b6-8c73-83ae5826fb09.png)

#### Valuing a Call option
![](https://user-images.githubusercontent.com/75659218/224573614-6a3ae7e1-174d-478a-ad22-c6d8659e45f1.png)

#### Valuing a Put option
![](https://user-images.githubusercontent.com/75659218/224573629-42c28a23-4861-4131-889d-2cafc7ac0fd5.png)


#### Processes
![](https://user-images.githubusercontent.com/75659218/224573660-27211e77-a54f-4a8c-93a3-8a5e8e900af3.png)
![](https://user-images.githubusercontent.com/75659218/224573687-a43e9bad-e3b7-485c-a77d-0afcde68e3f4.png)

#### Walkthrough example
Let's assume that we want to price a 3000 copper future on a call option.  The date till maturity (T) is 31 days with a volatility of 35 or 0.35 and annual interest of 0.005. The strike price (X) is 3000 while the futures price is 2900.

Let's solve for D1 to fnd D2 first.

where F = 2900 and X = 3000
The logarithm base e for F/X or futures price over strike price is: -0.03390....


Sigma squared with respect to T/2 is: 0.0052019625
volaitlity is 0.35 multiplied to interest rate and time to maturity 31/365 = 0.08493

solve for T gives us: 0.1019996

The result for D1 is:
![](https://user-images.githubusercontent.com/75659218/224573853-aad96967-3dea-467f-8bf7-69fc9e3c2e97.png)

Solving for D2:
![](https://user-images.githubusercontent.com/75659218/224573872-8f0ee28b-c7bd-4416-b384-a2a95370b9ef.png)


through cumulative normal distribution, we get
(d1) = 0.38922098
(d2) = 0.35073028

Terminal price = (2900 * 0.38922098) - (3000 * 0.308547) = 203.099842

Let's assume discount factor of 0.99

Calculate priceof option
0.99 * 203.099842 = 201.10
        
#### QuantLib Implementation
Instantiate the global data parameters and nacessary library packages.

        import QuantLib as ql
        import pandas as pd
        import math
        
        # Global Data
        calendar = ql.UnitedStates()
        businessConvention = ql.ModifiedFollowing
        settlementDays = 0
        daysCount = ql.ActualActual(ql.ActualActual.Bond)
        
#### Treasury Notes Futures Contract (2-Year Note)
Option on Treasury Futures Contracts. For these examples, we will assume the same volatility, strike, and spot price to showcase the differentiation of the option greeks.

##### 2-Year note yields.

![](https://user-images.githubusercontent.com/75659218/224574097-5ad33cb3-d4dc-43bf-9588-9d1856a1ce89.png)

        # 2-Year Note 

        interestRate = 0.003
        calcDate = ql.Date(1,12,2023)
        yieldCurve = ql.FlatForward(calcDate, interestRate, daysCount, ql.Compounded, ql.Continuous)

        ql.Settings.instance().evaluationDate = calcDate
        optionMaturityDate = ql.Date(24,12,2025)
        strike = 100
        spot = 125 # spot price is the futures price
        volatility = 20/100.
        optionType = ql.Option.Call

        discount = yieldCurve.discount(optionMaturityDate)
        strikepayoff = ql.PlainVanillaPayoff(optionType, strike)
        T = yieldCurve.dayCounter().yearFraction(calcDate, optionMaturityDate)

        stddev = volatility*math.sqrt(T)

        black = ql.BlackCalculator(strikepayoff, spot, stddev, discount)

        print("%-20s: %4.4f" %("Option Price for Treasury Futures Contract (2-Year Note)", black.value() )) 
        print("%-20s: %4.4f" %("Delta", black.delta(spot)))
        print("%-20s: %4.4f" %("Gamma", black.gamma(spot)))
        print("%-20s: %4.4f" %("Theta", black.theta(spot, T))) 
        print("%-20s: %4.4f" %("Vega", black.vega(T)))
        print("%-20s: %4.4f" %("Rho", black.rho( T)))


##### 3-Year Note yields.

![](https://user-images.githubusercontent.com/75659218/224574195-d0fc3fef-4bc8-4744-ad65-a64c995a1173.png)

        # 3-Year Note

        interestRate = 0.003
        calcDate = ql.Date(1,12,2023)
        yieldCurve = ql.FlatForward(calcDate, interestRate, daysCount, ql.Compounded, ql.Continuous)

        ql.Settings.instance().evaluationDate = calcDate
        optionMaturityDate = ql.Date(24,12,2026)
        strike = 100
        spot = 125 # spot price is the futures price
        volatility = 20/100.
        optionType = ql.Option.Call

        discount = yieldCurve.discount(optionMaturityDate)
        strikepayoff = ql.PlainVanillaPayoff(optionType, strike)
        T = yieldCurve.dayCounter().yearFraction(calcDate, optionMaturityDate)

        stddev = volatility*math.sqrt(T)
        black = ql.BlackCalculator(strikepayoff, spot, stddev, discount)

        print("%-20s: %4.4f" %("Option Price for Treasury Futures Contract (3-Year Note)", black.value() )) 
        print("%-20s: %4.4f" %("Delta", black.delta(spot)))
        print("%-20s: %4.4f" %("Gamma", black.gamma(spot)))
        print("%-20s: %4.4f" %("Theta", black.theta(spot, T))) 
        print("%-20s: %4.4f" %("Vega", black.vega(T)))
        print("%-20s: %4.4f" %("Rho", black.rho( T)))
        

##### 5-Year Note yields.

![](https://user-images.githubusercontent.com/75659218/224574396-fcabecb6-5d88-40bc-9c22-c627bfdd871f.png)

        # 5-Year Note

        interestRate = 0.003
        calcDate = ql.Date(1,12,2023)
        yieldCurve = ql.FlatForward(calcDate, interestRate, daysCount, ql.Compounded, ql.Continuous)

        ql.Settings.instance().evaluationDate = calcDate
        optionMaturityDate = ql.Date(24,12,2028)
        strike = 100
        spot = 125 # spot price is the futures price
        volatility = 20/100.
        optionType = ql.Option.Call

        discount = yieldCurve.discount(optionMaturityDate)
        strikepayoff = ql.PlainVanillaPayoff(optionType, strike)
        T = yieldCurve.dayCounter().yearFraction(calcDate, optionMaturityDate)

        stddev = volatility*math.sqrt(T)

        black = ql.BlackCalculator(strikepayoff, spot, stddev, discount)

        print("%-20s: %4.4f" %("Option Price for Treasury Futures Contract (5-Year Note)", black.value() )) 
        print("%-20s: %4.4f" %("Delta", black.delta(spot)))
        print("%-20s: %4.4f" %("Gamma", black.gamma(spot)))
        print("%-20s: %4.4f" %("Theta", black.theta(spot, T))) 
        print("%-20s: %4.4f" %("Vega", black.vega(T)))
        print("%-20s: %4.4f" %("Rho", black.rho( T)))

##### 10-Year Note yields.

![](https://user-images.githubusercontent.com/75659218/224574450-07f61d4e-6d35-4de5-aa40-2f56fba68e28.png)

        # 10-Year Note

        interestRate = 0.003
        calcDate = ql.Date(1,12,2023)
        yieldCurve = ql.FlatForward(calcDate, interestRate, daysCount, ql.Compounded, ql.Continuous)

        ql.Settings.instance().evaluationDate = calcDate
        optionMaturityDate = ql.Date(24,12,2033)
        strike = 100
        spot = 125 # futures price
        volatility = 20/100.
        optionType = ql.Option.Call

        discount = yieldCurve.discount(optionMaturityDate)
        strikepayoff = ql.PlainVanillaPayoff(optionType, strike)
        T = yieldCurve.dayCounter().yearFraction(calcDate, optionMaturityDate)

        stddev = volatility*math.sqrt(T)
        black = ql.BlackCalculator(strikepayoff, spot, stddev, discount)

        print("%-20s: %4.4f" %("Option Price for Treasury Futures Contract (10-Year Note)", black.value() )) 
        print("%-20s: %4.4f" %("Delta", black.delta(spot)))
        print("%-20s: %4.4f" %("Gamma", black.gamma(spot)))
        print("%-20s: %4.4f" %("Theta", black.theta(spot, T))) 
        print("%-20s: %4.4f" %("Vega", black.vega(T)))
        print("%-20s: %4.4f" %("Rho", black.rho( T)))
        
##### Natural Gas Futures Option.

![](https://user-images.githubusercontent.com/75659218/224574536-8723d9c2-8a64-4b54-9403-7b65eba753bc.png)

        # Natural Gas Futures Option

        interestRate = 0.0015
        calcDate = ql.Date(23,9,2015)
        yieldCurve = ql.FlatForward(calcDate, interestRate, daysCount, ql.Compounded, ql.Continuous)

        ql.Settings.instance().evaluationDate = calcDate
        T = 31/365.

        strike = 3.3
        spot = 2.4360
        volatility = 0.4251
        contract = ql.Option.Call

        discount = yieldCurve.discount(T)
        strikepayoff = ql.PlainVanillaPayoff(contract, strike)
        stddev = volatility*math.sqrt(T)

        strikepayoff = ql.PlainVanillaPayoff(contract, strike)
        black = ql.BlackCalculator(strikepayoff, spot, stddev, discount)

        print("%-20s: %4.4f" %("Option Price for Natural Gas Futures ", black.value() )) 
        print("%-20s: %4.4f" %("Delta", black.delta(spot)))
        print("%-20s: %4.4f" %("Gamma", black.gamma(spot)))
        print("%-20s: %4.4f" %("Theta", black.theta(spot, T))) 
        print("%-20s: %4.4f" %("Vega", black.vega(T)))
        print("%-20s: %4.4f" %("Rho", black.rho( T)))        
  
##### Gold Futures Options (31 days).

![](https://user-images.githubusercontent.com/75659218/224574813-1b3c9b85-1798-40bb-8e39-2e1790e9e8ef.png)

        # Gold Futures Option for April 2023
        # Time to maturity is 31 days; 31/365

        interestRate = 0.0015
        calcDate = ql.Date(23,9,2015)
        yieldCurve = ql.FlatForward(calcDate, interestRate, daysCount, ql.Compounded, ql.Continuous)

        ql.Settings.instance().evaluationDate = calcDate
        T = 31/365.

        strike = 2000
        spot = 1867.20
        volatility = 0.11
        contract = ql.Option.Call

        discount = yieldCurve.discount(T)
        strikepayoff = ql.PlainVanillaPayoff(contract, strike)
        stddev = volatility*math.sqrt(T)

        strikepayoff = ql.PlainVanillaPayoff(contract, strike)
        black = ql.BlackCalculator(strikepayoff, spot, stddev, discount)

        print("%-20s: %4.4f" %("Option Price for Natural Gas Futures ", black.value() )) 
        print("%-20s: %4.4f" %("Delta", black.delta(spot)))
        print("%-20s: %4.4f" %("Gamma", black.gamma(spot)))
        print("%-20s: %4.4f" %("Theta", black.theta(spot, T))) 
        print("%-20s: %4.4f" %("Vega", black.vega(T)))
        print("%-20s: %4.4f" %("Rho", black.rho( T)))

