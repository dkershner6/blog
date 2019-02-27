---
title: When to use Azure Functions v1 in an App Service Plan for Cost Reasons
category: "Azure Functions"
cover: costassumptions.jpg
author: Derek Kershner
---

### Update: 2/26/19

I have moved all of my functions to Azure Functions v2 (.Net Core) and have a [new comparison](/azure-functions-consumption-vs-app-service-vs-kubernetes) for those functions.

***

### Common Question

When to move on from the consumption plan is actually a tougher question to answer than I thought it would be. There are almost no analyses online, and the [one that I found](https://www.azurefromthetrenches.com/azure-functions-scaling-with-a-dedicated-app-service-plan/) had inaccurate information.

### Common Misconception

In [that article](https://www.azurefromthetrenches.com/azure-functions-scaling-with-a-dedicated-app-service-plan/), James does a good job of comparing the performance (and [updates the performance later on](https://www.azurefromthetrenches.com/azure-functions-significant-improvements-in-http-trigger-scaling/) as well), but falls flat on his cost conclusions. The reason for this is people tend to focus on execution cost only.

![Cost Assumptions](./costassumptions.jpg)

Now, this all appears fine until some simple math: $130 / 640 = ~$0.20/million - that $0.20 number looks an awful lot like one of [Microsoft's consumption costs](https://azure.microsoft.com/en-us/pricing/details/functions/).

![Functions Cost](./functionscost.jpg)

But, as you see in red, it totally ignores the other costs associated with functions, that can often be as high as **85% of the cost** or higher.

### Fixing Assumptions

Since his function was very simple, we can safely assume it would use the minimum memory per function of 128MB.  Given an average runtime of 0.581s per run, we can multiply these all together: 640M * (0.581s * 0.125GB) = 46,480,000 GB-seconds of memory use.  Since each one of these costs $0.000016, we can multiply those two as well: 46.48M GB-s * $0.000016 = $**743.68**.  As you can see, this is about 85% of the now total cost of $873.68.  This is obviously not cost effective, since the performance compares well to the 4x S1 app service at that velocity, costing $292.80/mo (and actually performs better).

### Updating the numbers

Later on James [updates his Azure Function performance numbers](https://www.azurefromthetrenches.com/azure-functions-significant-improvements-in-http-trigger-scaling/), so we should update too.  Taking the Stock Functions section, we can see that the performance of Azure Functions has significantly improved. Instead of 
a response time of 589ms, we now see it shoot all the way down to 69.5ms. (The throughput also improves, but I am going to keep the execution count the same to match the other app service data set).  Since Azure rounds up to 100ms minimum, so will we. 640M * (0.1s * 0.125GB) = 8,000,000 GB-seconds of memory use * $0.000016 = $128 - a **MASSIVE** savings.  This now represents a little less than half of the cost of $258.  This is also much more comparable to the $292 (and the added performance covers the $32 nicely - 11-14% in both cases).

### What does this mean?

It means, that with the vastly improved performance of consumption functions, the cost difference between them and the standard app service plans is not significant (anymore, because it was massive previously).

If you have an unpredictable use pattern, don't be afraid to use consumption functions, as they are likely the better choice, even at large scale.

If you have a predictable dataset, and can get away with Basic App Service plans, you likely will save money doing so, so long as the aggregate monthly execution count is > 160M or so (the 640M / the 4 instances).

If you have need of an _individual_ function running more swiftly, app service plans are your only means of doing so.

Hope this helps you reach a conclusion on how to build your functions!
