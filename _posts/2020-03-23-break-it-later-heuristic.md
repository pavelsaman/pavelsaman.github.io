---
layout: page
title:  "Break it Later Heuristic"
date:   2020-03-23 07:07:19 +0200
categories: testing
---

Having read a great book about exploratory testing by [Elisabeth Hendrickson](https://www.amazon.com/dp/1937785025/ref=cm_sw_em_r_mt_dp_U_dTfEEb8F0QTAK), I can't help thinking about more heuristics in testing and how to apply them in my day-to-day testing practice. I'll explain one heuristic I call "break it later", which originates from me having seen a couple of similar bugs and problems that are easily uncovered by following this heuristic.

Many times, an app you're testing has a particular flow, a number of steps a user has to go through to accomplish whatever the app is designed for. It could be a call center app where the agent goes from looking up the customer on the other side of the line to other steps like updating their personal information all the way to screens with new product offerings. Or it could be an e-shop where a user goes through a few steps in cart - add a product into cart, select payment and delivery methods, fill in invoice and delivery information, and finally see a thank you page with a short summary of what's been bought.

A lot of people, I have seen it in people with not much experience in testing, focus on only one step they think could be affected by a particular change. Let's take the following as an example. Say there's a minimum limit on how much money a user can spend in an eshop, say a minimum total price of an order is 200 (I'll leave the currency choice up to your imagination). Many times, people would focus on the first step, that is adding products into the cart in a way they either break this requirement or not. If they see the cart does what it should (notification to the user for example) when the rule is broken and doesn't do what is shouldn't when the rule is not broken, they consider it a passed test.

But what about all the other steps? Maybe a quantity could be changes in one of the later steps. So you make the first step with a valid order and decrease the amount of products later on so the order doesn't conform to the requirement anymore. What happens? Maybe the order cannot be finished at all? Either way, this is what I call "break it later" heuristic.

If you are lucky, other people will help you with testing, be it programmers who try out the app after finishing implementing a new feature, project managers because they want to see it, or clients because they closely work with you and you ask them to give it a try before going live. These groups of people will likely focus on the obvious, that is the first step, not on "breaking it later". As a tester, you should also estimate what testing could be done by others and focus on what's not done by others, so you effecively use your time.

Of course, it's only a tiny step to making this heuristic more general, and trying to break it sooner rather than later. If that could be applied to an app you're testing, it's likely something you should do as well. Either way, think about what other people don't have on their mind, and as a team of people, you'll cover much more cases.
