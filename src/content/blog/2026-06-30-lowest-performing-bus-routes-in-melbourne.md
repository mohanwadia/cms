---
author: Mohan Wadia
pubDatetime: 2026-06-30
modDatetime: 2026-06-30
title: Lowest Performing Bus Routes in Melbourne
featured: true
draft: false
tags:
  - Article
---
Drivers hate to see empty buses, while regular bus commuters hate to be in overcrowded buses. 

The 6th objective of [Victoria's Bus Plan (2021)](https://www.vic.gov.au/sites/default/files/2023-09/victorias-bus-plan-bus-reform-roadmap.pdf) is to create a more resource-efficient network. I believe it's important to regularly examine the productivity of all bus routes and make changes that create a more useful network for more people.

> 6. Deliver better value for money – ensuring value for money and continual service improvement under existing and new contracts with bus operators, manufacturers and infrastructure partners."

## Part I: Method

1. Service Duration: The number of hours in a normal week a service runs
2. Service Distance: The total distance covered by the route in a normal week.
3. Service Stops: The total non-unique number of stops visited by a route in a normal week.

For each of the metrics, I used a normal week in February 2026 without public holidays, and are inclusive of bidirectional travel to justly penalize routes that may only operate in one direction. 

The following is a simplification of loading the GTFS data using `gtfs-kit`, which was later used to calculate the three metrics. 

```
import gtfs-kit as gk
feed = gk.read_feed("C:/Users/Administrator/Desktop/PT/gtfs/4/google_transit.zip", dist_units="m")
dates = ["20260202", "20260203", "20260204", "20260205", "20260206", "20260207", "20260208"]
trip_stats = feed.compute_trip_stats()
route_stats = gk.routes.compute_route_stats(feed, dates, trip_stats
```

I then used `statsmodels.api` on the three metrics, deploying an Ordinar Least Squares Regression to find the R-squared and F-statistic values.

```
import statsmodels.api as sm
X = df['patronage']
y = df['service_hours']
X = sm.add_constant(X)
model = sm.OLS(y, X, missing='drop').fit()
print(model.summary())
```

## Part II: Findings


| Independent Variable | **R-squared Value** | **F-statistic Value** |
| -------------------- | ------------------- | --------------------- |
| 1. Service Duration | 0.797 | 1314 |
| 2. Service Distance | 0.750 | 1005 |
| 3. Service Stops | 0.762 | 1086 |


As a result, it can be determined that the number of service hours of a route is the strongest predictor of patronage. Therefore, I will use Service Duration as the independent variable in the following visual analysis. 

![image.png](/image-11.png)

![image.png](/image-10.png)

![image.png](/image-12.png)

## Part III: So what?

The orbitals (901/902/903) are the most resource intensive routes, however their patronage is proportional to the number of service hours run. The 703, 900, 907, and 906 follow which are all SmartBuses. It's clear that the SmartBus program has been a decade-long success by creating new high-performing routes, however their extreme length has proved tricky to update as demand has increased over time and bounced back over COVID.


| Route | 2018 | 2019 | 2020 | 2021 | 2022 | 2023 |
| ----- | --------- | --------- | --------- | --------- | --------- | --------- |
| 901 | 2,825,852 | 4,395,718 | 2,417,336 | 2,616,200 | 3,551,823 | 4,329,626 |
| 902 | 2,964,959 | 4,616,532 | 2,538,664 | 2,692,073 | 3,513,353 | 4,418,050 |
| 903 | 3,770,586 | 5,752,859 | 2,874,268 | 3,302,643 | 4,387,537 | 5,548,020 |


The next greatest buses with overperforming patronage are the 246, 733, 828, 402, 767, and 737. The [733 and 767](https://www.premier.vic.gov.au/community-gets-board-extra-bus-services) both got upgraded in 2022, and the 246 and 402 runs an elite 10-minute off-peak frequency. Both the 737 and 828 are due for an upgrade, with the 737 running a worse than 30 minute off-peak frequency. 

Meanwhile, 788 stands alone as the most underperforming from the top 50 most served routes, running over 50km and taking 100+ minutes with a frequency of between 30-40 minutes 7 days and great span. 

The patronage per hour histogram provides a mean of 18.3 passengers/hour, with an IQR of 9.5 to 24.2 passengers/hour. There is a large range of routes (82 to be exact) that have a patronage per service hour at less than half of the average. 

The Route 601 as an outlier with a historic 261 passengers per hour. This may be from a variety of factors, including limited stops, a high speed of 29km/h, lengthy bus lanes, and possible low fare avoidance due to most connecting with rail.