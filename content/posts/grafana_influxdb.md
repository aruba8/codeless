---
title: "Grafana and InfluxDB"
date: 2019-03-23T16:43:44-05:00
draft: false
tags: ["performance", "testing", "gatling", "influxdb"]
---

On my job I has been assigned to a task to measure performance of our applications. I was considering several tools, Gatling, Locust, Jmeter and others. Initially I had selected [Locust](https://locust.io/) but then decided to use [Gatling](https://gatling.io/). Probably later I'll try to explain and will do some comparisons. But for now all you need to know is that I have selected Gatling for my needs. The second issue I had to figure out solution for was where and how to store results and how to analyze them. Despite the fact that gatling has nice report generation, it is still not very convenient to compare and analyze everyday results. So this is how I found InfluxDB and Grafana.


InfluxDB is open source time-series database which fits my needs to store results, Grafana in it's turn is platform for analyzing and monitoring data, visualize them, create dashboards, etc. Altogether it gives us amazing opportunity to get feedback immediately right after you started your tests, so you don't need to wait until all tests are finished, generate graphs, etc. I spent a lot of time trying to find tutorial how to setup them, but most of them were outdated. So I hope this article will save some time for someone. We will setup influxdb + grafana locally using `docker-compose` in this article.

So you will need `docker` and `docker-compose` installed on your machine. For you convenience I've created git repository with ready to go config files for docker-compose. You just need to clone it:

{{< highlight bash>}}
git clone git@github.com:biomaks/grafana-influx-kit.git
{{< / highlight >}}

We'll go through configs so you will have some understanding what's going on there. If you run `tree .` command inside cloned repository the output should look like this:

{{< highlight bash >}}
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

{{< highlight ini "linenos=table" >}}
[[graphite]]
  templates = [
    "gatling.*.*.*.* measurement.simulation.request.status.field",
    "gatling.*.users.*.* measurement.simulation.measurement.request.field"
  ]
  enabled = true
  bind-address = ":2003"

[meta]
  dir = "/mnt/db/meta"

[data]
  dir = "/mnt/db/data"
  wal-dir = "/mnt/influx/wal"

[hinted-handoff]
  dir = "/mnt/db/hh"
{{< / highlight >}}

that is basically it for the influxdb. We don't need any configs for grafana, we'll make them through the UI. 

At this point we are ready to start our services. I just want to make a note here, that I used specific versions of influxdb and grafana in my docker files instead of using `:latest` versions. I made it by purpose to ensure that my setup will continue working in the future.
Influxdb will use `8086` and `2003` ports, grafana will be running on `3000` port. So make sure they are available

Let's run it:
{{< highlight bash>}}
docker-compose up -d 
{{< / highlight >}}


Once docker containers up, we should create database in influxdb and for this article we will use default admin user. Let's connect to influxdb using `influx` command in order to create database:

{{< highlight bash>}}
  ~$ docker-compose exec influxdb influx
  Connected to http://localhost:8086 version 1.7.4
  InfluxDB shell version: 1.7.4
  Enter an InfluxQL query
  >
{{< / highlight >}}

creating database is simple as (we will use `graphite` name for it):
{{< highlight bash>}}
create database graphite
{{< / highlight >}}

now we done with influxdb so we can close influxdb console (ctrl+d). Grafana should be available on `localhos:3000` default user/password are `admin/admin` it will ask you to change password on first login. So once you are in, you need to add data source and create dashboard.

Follow theses steps to add our influxdb as data source:

1. Go to Configuration -> Data Sources where click on `Add data source` 
2. Choose InfluxDB
3. Set url `http://localhost:8086` and select access `Browser` in HTTP section
4. Set database `graphite` and user/password as `admin/admin` in InfluxDB Details

If everything is correct you will see `Data source is working` after click on `Save and Test` button.

Last thing for today, we need to add dashboard.

1. Hover `+` icon on the left sidebar and select `Import` option. 
2. Copy and paste the content of `tmpl.json` file into opened JSON form input and click `Load` button 
3. Confirm options by clicking green `Import` button

So now, you are ready to run you gatling tests and see how it goes in real time.
























