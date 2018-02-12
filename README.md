---
title: "Power calculations and sample size"
# author: "Brett Ory"
thumbnailImagePosition: left
thumbnailImage: http://www.thehindu.com/opinion/open-page/article17118051.ece/alternates/FREE_660/31TH_ILLDEEPAK
coverImage: http://www.thehindu.com/opinion/open-page/article17118051.ece/alternates/FREE_660/31TH_ILLDEEPAK
metaAlignment: center
coverMeta: out
date: 2018-02-10T21:13:14-05:00
categories: ["Father involvement"]
tags: ["Statistical power", "sample size", "surveys", "Stata", "statistical significance"]
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)


```

Last week I ended the blogpost saying I would share results from the Kaggle survey this week, but actually there's something else I want to talk about first 

<br>

## Sample size

In social science research we are mostly working with data collected by someone else. That's because the scale of the survey data we need to be representative of a country or multiple countries is too big and too expensive for one person to collect by themselves. As a result, in most of our research, sample size is not something we can change. And because we don't have any control over sample size, it's not something we tend to think about a lot, other than to calculate standard errors of the estimates. What I want to learn today is what to do if I'm designing my own survey. 

How can I determine the sample size necessary to have enough statistical power to make meaningful calculations?

<br>

## Significance and Power

First some definitions. In frequentist hypothesis testing we are usually asking if an observed phenomenon is different from some number. Where it gets confusing is that there are two possible answers (yes or no, 1 or 0, greater than x or less than x, etc), but four possible test outcomes if we include the possibility that we made a mistake. For example, a woman might take a pregnancy test with one of four outcomes   

1. accurately not pregnant    
2. accurately pregnant    
3. the test might say she is pregnant when she is not (false positive, type 1 error)    
4. the test might say she is not pregnant when she is (false negative, type 2 error) 

These four potential outcomes are illustrated by the following table:

<br>

![](http://cramster-image.s3.amazonaws.com/definitions/stat-1-img-1.png)

<br>

The probability of getting a type 1 error is the significance of the test (denoted by $\alpha$) and the probability of getting a type 2 error is the test's power (denoted by $1 - \beta$). The significance is set by the researcher a priori. This is often $\alpha$ = .05 or .01. Most researchers will be familiar with this. 

Calculating the power of the test is trickier. In order to calculate the power of a test we need to know the value of the alternative hypothesis and the null hypothesis. I'm going to do this with an example from my own research. 

<br>

## The example

### Background

In one of the papers for my dissertation on father involvement in childcare, I find that wives have the power to overrule men's early socialization. When they are young, men see their fathers doing (or not doing) childcare in the home, and as adults men tend to follow in their father's footsteps. If their fathers were highly involved, men are also more likely to be highly involved. But the extent to which men are able to follow their father's example is influenced by their wife. The more hours per week she works, the more strongly he is influenced by the example set by his own father. Conversely, when women don't work at all, it doesn't really matter what men's fathers did--the wife doesn't give room for her husband to act on the example of his father. 

The gatekeeping effect I just described makes some sense given what we know about male and female gender roles. Except there is one counterintuitive aspect of this effect that you can see in the following figure made in Stata:

```
regress dolcctr fhfrc c.fccfrc##c.pwhv2 age edu2 q8_1 q17_1 q18           
margins, at(fccfrc=(-1.5 -0.5 0.5 1.5) pwhv2=(-26.55 0.45 13.45 33.45))


marginsplot, ytitle("men's involvement in childcare") xlabel(none) 	          ///
	legend(order(1 "0 hours" 2 "27 hours" 3 "40 hours" 4 "60 hours")          ///
	title("partner's work hours"))	                                          ///
	xtitle("father involvement in childcare")                                 ///
	title("Own father's involvement and partner's work hours") 
```

![](/img/Graph.jpg)

The figure shows the predicted value of men's share of childcare on a scale from 0-4, where 0 = partners do all the childcare, 2 = men share childcare equally with their partners, and 4 = men do all the childcare themselves. As explained above, the lines indicate that in general, the more men's fathers were involved, the greater share of childcare men do. EXCEPT in the case where men's partners don't work at all (the blue line). Here it appears that men are actually less involved when their fathers were highly involved than when they had fathers who were never involved. What gives? 

*Is the difference between these fathers statistically significant?*

<br>

### Significance

In other words, my model predicts that men whose fathers were highly involved and who have partners who didn't work at all (let's call them group 1) would have an involvement score of 1.02<sup><a href="#fn1" id="ref1">1</a></sup>, compared to the 1.34 predicted involvement score of men whose fathers weren't involved at all and who likewise have partners who don't work (group 0). We can first do a two sample t-test to test if the observed scores are significantly different from each other. 

I ran this t-test in Stata with the following results. (You could also do this in R, but I was working with the data in Stata for this project.)

```
ttest dolcctr, by(group)


Two-sample t test with equal variances
------------------------------------------------------------------------------
   Group |     Obs        Mean    Std. Err.   Std. Dev.   [95% Conf. Interval]
---------+--------------------------------------------------------------------
       0 |      19    1.560464     .163706    .7135779     1.21653    1.904397
       1 |      10     1.46119    .1342994    .4246919    1.157384    1.764997
---------+--------------------------------------------------------------------
combined |      29    1.526232    .1156121    .6225902    1.289411    1.763052
---------+--------------------------------------------------------------------
    diff |            .0992732    .2469596                -.407446    .6059924
------------------------------------------------------------------------------
    diff = mean(0) - mean(1)                                      t =   0.4020
Ho: diff = 0                                     degrees of freedom =       27

    Ha: diff < 0                 Ha: diff != 0                 Ha: diff > 0
 Pr(T < t) = 0.6546         Pr(|T| > |t|) = 0.6909          Pr(T > t) = 0.3454

```
t = 0.40. The difference between group 0 and group 1 is not statistically significant, but the sample size is really small. How powerful is this test? If there was a significant difference between groups, would we be able to see it?

<br>

### Power

Let's imagine two normal distributions: one distribution represents group 0 with a mean of 1.34 and one distribution represents group 1 with a mean of 1.02. $\alpha$ is the most extreme 5% of group 1 distribution, and the t-test asks how much of the distribution of group 1 overlaps with group 0. Power = 1 - $\beta$ and $\beta$ is the area of the group 0 curve which is less than the critical value of the group 1 distribution. In other words, 

$$\beta = P(\bar{x} \leq x_{crit} | \mu = 1.34)$$

We can calculate the critical z-score and the probability that an observation is less than or equal to that z-score in R using the following code written by [HyLown Consulting LLC](http://powerandsamplesize.com/Calculators/Compare-2-Means/2-Sample-1-Sided).

```{r}
mu1=1.02
mu0=1.32
kappa=1       ## ratio of sample size between groups
sigma1=.42   
sigma0=.71
alpha=0.05

z=(mu1-mu0)/sqrt(sigma1^2+sigma0^2/kappa)*sqrt(10)
(Power=pnorm(z-qnorm(1-alpha)))
```

$\beta$ is .9975, meaning power is 1 - 0.9975 = 0.003. In other words, our test has virtually no statistical power, and we have a near certainty of making type 2 error. Apparently, a sample size of 10 is too small to say anything about the difference between groups 0 and 1. We have neither significance nor power. 

<br>

### Sample size

The power of my previous test was low, so I would now like to conduct another survey with a larger sample size to test if the difference between means 1.02 and 1.34 is actually significant. 


Working backwards from the calculation I just did, I first need to choose what statistical power level I find acceptable. This is a bit arbitrary, and varies from field to field. In social science power = .80 is considered acceptable, meaning I'm willing to accept that I will incorrectly say there is no effect when there is 20% of the time. 

The sample size in a two-sample t-test is calculated by the following formula:

$$N_1 = (\sigma_1^2 + \sigma_0^2/k)((z_{1-\alpha} + z_{1-\beta})/(\mu_1 - \mu_0))^2$$

```{r}
## these are the same values as above
mu1=1.02
mu0=1.32
kappa=1       
sigma1=.42   
sigma0=.71
alpha=0.05
beta=0.20
(n1=(sigma1^2+sigma0^2/kappa)*((qnorm(1-alpha)+qnorm(1-beta))/(mu1-mu0))^2)
ceiling(n1) # round n1 to a whole number
```


Thus, I would need a group 1 sample size of 47 to see a significant difference between group 1 and group 0. 

This post can be found on [GitHub](https://github.com/brettory/powercalc).

<sup id="fn1">1. These numbers are a bit different than what appears in the figure, because they were calculated by hand as the sum of the direct and interaction effects, ignoring control variables. The figure was calculated using the margins command in Stata which supplies mean values for the control variables. <a href="#ref1" title="Jump back to footnote 1 in the text.">↩</a></sup> 


