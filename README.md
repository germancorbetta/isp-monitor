# isp-monitor
ISP monitor using a Raspberry PI with balena.io, Telegraf agent, Speedtest CLI, InfluxDB and Grafana

## Objective
To finish with some automated ISP monitoring system like this

![Grafana Example](/grafana-example.png)

## 1st Step: balena.io account
Create an account in balena.io to get your Raspberry up and running.
Follow balena.io "Get Started" page until you have your account with a balena Application and your Raspberry Pi device in it.
For the purpose of this guide, let´s assume that your balena.io application name is "First-App-RaspberryPi3".

Note: balena applications are like a group of devices that will eventually deploy the same services.

Note 2: we will assume that you finished the balena "get started" section with balena-cli installed in your main computer (not the Raspberry).


## 2nd Step: Understanding the code...
These are just some hardcoded values you may change depending on your implementation:
- My ISP DNS: 181.45.64.77
- My Router: 192.168.0.1
- Database: speedtest
- Database User: root
- Database Pass: password
- Raspberry static IP: 192.168.0.123

The project is a Docker Compose file that will install:
- InfluxDB as the open source time-oriented data base, listening in port 8086, persisting values upon restarts and create a "speedtest" database with root:password credentials.
- Grafana as the open source graphic UI that you´ll need to configure at the end, listening in port 3000.
- Telegraf as the capture agent to gather files or even run external processes.
- SpeedTest-cli as the command-line tool to analyze your ISP. More info in speedtest.net.

## 3rd Step: Run it!
Download this repository files in your main computer (not the Raspberry).
Go to the root folder.
And run: 
```$ balena push First-App-RaspberryPi3```

## 4th Step: Start your dashboards in Grafana
Wait for a few minutes until it´s fully deployed.
Additional logs available at balena.io if you access the device.
You´ll notice that balena.io shows 3 services running at the device, "Influx", "Grafana" and "Telegraf", you may read logs or SSH to them from there.
Once it´s working, just go to http://192.168.0.123:3000 (default user admin:admin).
Configure Grafana to read from the InfluxDB "http://192.168.0.123:8086" with the InfluxDB Details (user: root, password: password).
And start creating some custom Dashboard!

NOTE: Doing the Grafana query for some graphic, just select the "measurement" and the "field" (this last one is sometimes not needed) to read specific values in your timelines.

## 5th Step: [OPTIONAL] Limit your influxDB measurements
You may have a limited storage for your measurements, so you may change the database default retention policy (how long to wait until deleting old data) to make sure it´s not affecting your Raspberry Pi microSD capacity. To do so, just go to balena.io and log into the Influx Docker through SSH (not the Host) and run the following SQL inside influx.
```
$ influx
> CREATE RETENTION POLICY speedtest90days ON speedtest DURATION 90d REPLICATION 1 DEFAULT
```
That would create a new retention policy to overwrite the default one, to keep every measurement in the "speedtest" database for up to 90 days before deleting them.

## Special Thanks
To Mr. [@sbehrends](https://github.com/sbehrends) for the idea and samples that evolved into a stand-alone ISP monitor. (his implementation is even greater with a dedicated cloud server, TLS, etc.)
My original implementation was the speedtest-cli sending JSON to IFTTT and from there to my personal Google Drive but having influxDb+Grafana is way better than Google Drive spreadsheet graphics capability and escalates easier.

To the Ookla team for the speedtest.net CLI tool, more info [Github](https://github.com/teamookla), [Facebook](https://www.facebook.com/speedtest) or [Twitter](https://twitter.com/speedtest).

## Disclaimer
This example is provided as a reference for your own usage and is not to be considered my own product.
By using it, you are approving the license from different products and regulations like Speedtest, Telegraf, InfluxDB, Grafana, Docker, RGPD and more.
This article involves products and technologies which do not form part of my catalog. Technical assistance for such products is limited to this article.
