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
![image.png](attachment:image.png)

#### Valuing a Call option
![image-2.png](attachment:image-2.png)

#### Valuing a Put option
![image-3.png](attachment:image-3.png)

#### Processes
![image-4.png](attachment:image-4.png)
![image-5.png](attachment:image-5.png)
        
        
        
        
        
        
        
        
        
        
        
        
        
        
