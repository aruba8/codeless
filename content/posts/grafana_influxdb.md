---
title: "Grafana and InfluxDB"
date: 2019-03-20T16:43:44-05:00
draft: true
tags: ["performance", "testing", "gatling", "influxdb"]
---

On my job I has been assigned to a task to measure performance of our applications. I was considering several tools, Gatling, Locust, Jmeter and others. Initially I had selected [Locust](https://locust.io/) but then decided to use [Gatling](https://gatling.io/). Probably later I'll try to explain and will do some comparisons. But for now all you need to know is that I have selected Gatling for my needs. The second issue I had to figure out solution for was where and how to store results and how to analyze them. Despite the fact that gatling has nice report generation, it is still not very convenient to compare and analyze everyday results. So this is how I found InfluxDB and Grafana.


InfluxDB is open source time-series database which fits my needs to store results, Grafana in it's turn is platform for analyzing and monitoring data, visualize them, create dashboards, etc. Altogether it gives us amazing opportunity to get feedback immediately right after you started your tests, so you don't need to wait until all tests are finished, generate graphs, etc. I spent a lot of time trying to find tutorial how to setup them, but most of them were outdated. So I hope this article will save some time for someone. We will setup influxdb + grafana locally using `docker-compose` in this article.

So you will need `docker` and `docker-compose` installed on your machine. For you convenience I've created git repository with ready to go config files for docker-compose. You just need to clone it:

```bash
git clone git://
```

We'll go through configs one by one so you will have some understanding what's going on there. If you run `tree .` command inside cloned repository the output should look like this:

{{< highlight ini >}}
.
├── README.md
├── docker-compose.yml
├── grafana
│   └── Dockerfile
├── influxdb
│   ├── Dockerfile
│   └── influxdb.conf
└── tmpl.json

2 directories, 6 files
{{< / highlight >}}

So first of all I'm going to make this setup working with gatling, gatling can use graphite protocol to write data into influxdb directly over TCP, influxdb in it's order also supports that protocol. In order to achieve it we had to add following section to `influxdb/influxdb.conf` file.

{{< highlight ini >}}

[[graphite]]
  templates = [
    "gatling.*.*.*.* measurement.simulation.request.status.field",
    "gatling.*.users.*.* measurement.simulation.measurement.request.field"
  ]

{{< / highlight >}}

so influxdb will use these templates when adding data, and the whole file `influxdb/influxdb.conf` should have content like showed here:

{{< highlight ini >}}
[[graphite]]
  templates = [
    "gatling.*.*.*.* measurement.simulation.request.status.field",
    "gatling.*.users.*.* measurement.simulation.measurement.request.field"
  ]
  enabled = true
  bind-address = ":2003"
  database = "graphite"
  retention-policy = ""
  protocol = "tcp"
  batch-size = 5000
  batch-pending = 10
  batch-timeout = "1s"
  consistency-level = "one"
  separator = "."
  udp-read-buffer = 0

[meta]
  dir = "/mnt/db/meta"

[data]
  dir = "/mnt/db/data"
  wal-dir = "/mnt/influx/wal"

[hinted-handoff]
  dir = "/mnt/db/hh"
{{< / highlight >}}

that is basically it for the influxdb. We don't need any configs for grafana, we'll make them through UI.









