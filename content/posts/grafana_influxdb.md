---
title: "Grafana and InfluxDB"
date: 2019-03-20T16:43:44-05:00
draft: true
tags: ["performance", "testing", "gatling", "influxdb"]
---

On my job I has been assigned to do measure performance of our applications. I was considering several tools, Gatling, Locust, Jmeter and others. Initially I had selected [Locust](https://locust.io/) but then decided to use [Gatling](https://gatling.io/). Probably later I'll try to explain and will do some comparisons. But for now all you need to know is that I have selected Gatling for my needs. The second issue I had to figure out solution for was where and how to store results and how to analyze them. Despite the fact that gatling has nice report generation, it is still not very convenient to compare and analyze everyday results. So this is how I found InfluxDB and Grafana.


InfluxDB is open source timeseries database which fits my needs to store results, Grafana in it's turn is platform for analyzing and monitoring. I spent a lot of time trying to find tutorial how to setup them, but most of them were outdated. So I hope this article will save some time for someone. We will setup influxdb + grafana locally using `docker-compose` in this article.

So you will need docker installed on your machine. 
