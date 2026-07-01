---
author: Mohan Wadia
pubDatetime: 2026-07-01
modDatetime: 2026-07-01
title: Which Melbourne Bus Routes Are Over- and Under-Performing?
slug: patronage-analysis
featured: true
draft: false
tags:
  - Article
description: An analysis of Melbourne's bus route performance using GTFS and patronage data
---
I believe it's important to regularly examine the productivity of all bus routes, making resource-effective changes to create a more useful network for more people. [Victoria's Bus Plan (2021)](https://www.vic.gov.au/sites/default/files/2023-09/victorias-bus-plan-bus-reform-roadmap.pdf) agrees with this.

> 6. Deliver better value for money – ensuring value for money and continual service improvement under existing and new contracts with bus operators, manufacturers and infrastructure partners.

I set out to see which Bus Routes are under-performing relative to the resources provided, as well as which routes require more resources to sustain demand.

# Methodology

It's important to normalize patronage data per route in order to make comparisons. I decided to test patronage data against the following three metrics to see which has the strongest correlation. 

1. Service Duration: The total number of timetabled service hours
2. Service Distance: The total timetabled distance covered
3. Service Stops: The number of timetabled non-unique stops served

I used a normal week in February 2026 without public holidays, and summated service hours for each direction.

The following is a simplification of loading the GTFS data using `gtfs-kit`, which was later used to calculate the three metrics. 

```python file="gtfs-kit.py"
import gtfs-kit as gk
feed = gk.read_feed("C:/Users/Administrator/Desktop/PT/gtfs/4/google_transit.zip", dist_units="m")
dates = [str(d) for d in range(20260202, 20260202 + 7)]
trip_stats = feed.compute_trip_stats()
route_stats = gk.routes.compute_route_stats(feed, dates, trip_stats
```

I then used `statsmodels.api` on the three metrics, deploying an Ordinary Least Squares Regression to find the R-squared and F-statistic values.

```python file="statsmodels.py"
import statsmodels.api as sm
X = df['service_hours']
y = df['patronage']
X = sm.add_constant(X)
model = sm.OLS(y, X, missing='drop').fit()
print(model.summary())
```

# Findings


| Independent Variable | **R-squared Value** | **F-statistic Value** | **P value** |
| -------------------- | ------------------- | --------------------- | ----------- |
| 1. Service Duration | 0.797 | 1314 | <0.001 |
| 2. Service Distance | 0.750 | 1005 | <0.001 |
| 3. Service Stops | 0.762 | 1086 | <0.001 |


As a result, the number of service hours of a route has the strongest association with patronage. Therefore, I will use Service Duration as the independent variable in the following visual analysis. 

![Figure 1](../../../public/images/scatter_50.png)

The orbitals (901/902/903) are the most resource intensive routes, however their patronage is proportional to the number of service hours run. The 703, 900, 907, and 906 follow which are all SmartBuses. It's clear that the SmartBus program has been a decade-long success by creating new high-performing routes, however their extreme length has proved tricky to update as demand has increased over time and bounced back after COVID as seen in the table below.


| Route | 2018 | 2019 | 2020 | 2021 | 2022 | 2023 |
| ----- | --------- | --------- | --------- | --------- | --------- | --------- |
| 901 | 2,825,852 | 4,395,718 | 2,417,336 | 2,616,200 | 3,551,823 | 4,329,626 |
| 902 | 2,964,959 | 4,616,532 | 2,538,664 | 2,692,073 | 3,513,353 | 4,418,050 |
| 903 | 3,770,586 | 5,752,859 | 2,874,268 | 3,302,643 | 4,387,537 | 5,548,020 |


The next greatest buses with overperforming patronage are the 246, 733, 828, 402, 767, and 737. The [733 and 767](https://www.premier.vic.gov.au/community-gets-board-extra-bus-services) both got upgraded in 2022, and the 246 and 402 runs an elite 10-minute off-peak frequency. Both the 737 and 828 are due for an upgrade, with the 737 running a worse than 30 minute off-peak frequency. 

Meanwhile, 788 stands alone as the most underperforming from the top 50 most served routes, running over 50km and taking 100+ minutes with a frequency of between 30-40 minutes over a solid span.

![Figure 2](../../../public/images/histogram.png)

The patronage per hour histogram provides a mean of 18.3 passengers/hour. The Route 601 stands as an outlier with a historic 261 passengers per hour. I suspect this is the result of a variety of factors, including limited stops, an average speed of 29km/h, lengthy bus lanes, and possible low fare avoidance due to most connecting with rail.

![Figure 3](../../../public/images/scatter_all.png)

There is a large range of routes (82 to be exact) that have a patronage per service hour at less than half of the average. This is also seen in the histogram, with an IQR of 9.5 to 24.2 passengers/hour. While there are other goals for routes such as maximizing coverage, low patronage is a key indicator that the route may be underperforming.

# So What?

By isolating service hours as one variable, patronage data assists in evaluating the performance of a route. However, there is more nuance to this that requires data that isn't currently publicly available. 

Is an off-peak and weekend patronage drop proportional to the service hours provided? Moreover, how have recent upgrades in service hours affect patronage, and can the discussed model accurately predict said change? And looking at the other two metrics discussed, can service distance or number of stops become more useful metrics by looking at local area coverage and unique coverage?