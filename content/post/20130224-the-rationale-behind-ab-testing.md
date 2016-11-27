+++
title = "The rationale behind A/B testing"
date = "2013-02-24"
slug = "the-rationale-behind-ab-testing"
categories = [ "Product Management" ]
tags = [ "A/B Testing"]
+++

A few days ago, while having a casual conversation over a beer, I was totally shocked when I heard something close to: "A/B testing is pointless on small project like ours because testing overhead is too high". Is it? Let's run the numbers :) Assumptions: 

  * The product costs X to operate monthly. 70% of the costs are development team, and 30% is administrative overhead and other costs like infrastructure, customer support and marketing expenses. The development cost obviously becomes much, much smaller percentage of the costs if the project is successful, but here we are taking about small project that still has to fight for success.
  * Typical cost of implementing a feature is 0.5 month of development team work, so in our case it is 0.35*X.
  * Averaged overhead of making new feature A/B testable is 10%. For a team that's used to A/B testing stuff this is likely accurate number, for team that is not routinely doing it the overhead may be a lot higher, at least initially. If a feature that has base cost of 0.35*X, the A/B testing overhead is 0.035*X.
  * Our alternative to A/B testing is picking the variant manually by product owner intuition. I assume absolutely great PO is right 80% of the time.
  * Positive change brings the revenue up by 3%, negative change (a mistake) makes the revenue go down 2%.
  * The product makes X per month, essentially it's on the edge of getting axed.

The question: should we A/B test changes in this scenario? Now something very important, and something people tend to overlook a lot: **anything** you do with the product impacts revenue over a long time. It's not just one month, but possibly event years when the change is active. In order to properly think about A/B testing, we have to take this into account. Let's take 6 month period: 

  * Not running A/B test gives us expected payoff of (0.98*0.2+1.03*0.8)*6*X, which is 6.12*X.
  * Running A/B test means we are almost always right, and gives us payoff of 1.03*6*X, which is 6.18*X.

We made 0.025*X (0.06 minus cost of A/B test) by running the A/B test! Note that the model I used to illustrate the need for A/B testing is highly simplified. For example, in reality there's is some cost associated with delay introduced by testing, the product performance while tests are running is a mix of good and bad variant, the variants between which we are trying to decide may both be good, both bad, or one may be neutral etc etc. The result of this added complexity is not to abandon A/B testing! The goal is not to break even but to make the product make 10 times as much! Bear in mind that any mistake you make early in the product lifecycle will impact your future revenue. The model above shows A/B testing is profitable even if you play not to lose. If you play to win, A/B testing is something you just **have to** be doing.