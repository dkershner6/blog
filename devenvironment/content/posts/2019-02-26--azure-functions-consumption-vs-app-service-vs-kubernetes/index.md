---
title: Azure Functions v2 Consumption vs App Service Plan vs Kubernetes - Where to Host for optimal cost and performance?
category: "Azure Functions"
cover: kubernetes.png
author: Derek Kershner
---

### Note

This comparison applies to functions on v2 (.Net Core). There is a large difference in performance between the two, and these conclusions should not be held over to v1. I do have a [similar-ish comparison for v1](/when-to-use-app-service-plan/).

***

### Some Background

Over the course of switching to v2, I noticed that code was executing much faster, even on tiny VMs like are used for consumption. This is a [cornerstone](https://visualstudiomagazine.com/articles/2018/08/22/bing-net-core.aspx) of the technology, so this is less than surprising, but it does affect how cost is measured amongst hosting possibilities, notably for cases where you pay by the millisecond.

I often find the best way to compare something is just to move it around in production. No simulation, and so long as it is done correctly, you are comparing literal apples to apples. I have an application that spans 6 azure function apps, uses durable functions, and has both long and short running functions. It is fairly CPU intensive, but not overly so. The app also has a steady, predictable load, which caters itself well to non-consumption based models, because it has an api rate limiter on the primary target for the app. This seems like a perfect app for varied testing of the environments, but it is a single app and **your mileage may vary**. If you have a burstier app, it would lend itself more to the consumption side.

I added in Kubernetes to the comparison, because docker is now working pretty well with Functions, but this obviously requires a bit more effort than the other two. I will do a write up on this later on, but we can compare performance and cost.

### Methodology

Since consumption is the hardest to measure, I am going to have the app run in that environment for a full month, and tally the cost associated. For the other two, I will approximate how you could comfortably run the app in a correctly sized VM/app service by running the app inside it for a few days.

## Testing

### App Service Plan (Windows)

This is actually how I was originally hosting this app, so I know for a fact that it runs well on 1x B2 instance. It uses about 60% of the CPU on a regular basis and can spike up to 80-90%, but nothing that would cause too much concern. Note that I was using the Basic tier, since it is cheaper and my load was fairly predictable, but an S2 would also do the job if you have a bit of a burstier load.

At time of writing, cost of a B2 instance in West US 2 is **$109.50/month**. On a CPU intensive task (long running) an average duration of **1.45min** was observed. On an active function that performs more than one async task, an average duration of **11.42s** was observed. On a rapid, since task function, an average response of **2.01ms** was observed.

### Kubernetes (Linux/Docker)

Behind Azure Kubernetes Service is actually just normal VMs that are priced as Ubuntu or CentOS Linux. In addition to the VM, you require one 32GB SSD and some networking items per node. I will approximate this cost at $6/mo, although you may get some for free, depending on how many other VMs you have. For a VM size, we could use the A2 v2, which would be the same as the app service plan's, but since the pricing is close for me, I recommend springing for the D2 v3. At time of writing, one West US 2 D2 v3 VM node costs $70.08/month, making for a total cost of **$76.08/month**.  This VM handled the load of the app with ease, hovering around 25-30% CPU, barely peaking above 50%. The CPU intensive task ran in a brisk **1.05min**. The multi-threaded task ran in **8.36s**. The rapid task ran in **1.49ms**.

### Consumption (Windows)

Across 6 different Function apps, each running variable amount of instances, some cold starts, all cause the performance of the app to be decidedly more spiky here. The long running task runs in **1.7min**. The multi-threaded task ran in **16.42s**. The rapid task ran in **3.35ms**. Over the course of the month, (this was easy to calculate thanks to that new billing tool), the execution time has cost $28.90 and the executions have cost $0.96. There is a little file IO as well mixed in, and best I can approximate is that it costs $0.20.  This is a total cost of **$30.06** for the month.

## Conclusion

As far as I can tell, with the increased performance of .Net Core under its belt, there is no scenario that beats the consumption model of Azure functions. This was a much closer comparison with .Net Framework, but thanks to both general improvements in Functions themselves and the new framework, it is no longer even close.  Even if a theoretical D1 v3 VM existed for Kubernetes to use, it would still need to auto scale 2 nodes on occasion, and would cost a theoretical $41 even if it didn't.

Now, consumption functions are certainly not without their shortcomings and limits, and the performance is spikier and generally slower, but it also has the ability to scale without any custom rules or intervention on your part.