# isp-monitor
ISP monitor using a Raspberry PI with balena.io, Telegraf agent, Speedtest CLI, InfluxDB and Grafana

## Objective
To finish with some automated ISP monitoring system like this:

![Schema Example](/final-schema.png)

And eventually get graphics like this:

![Grafana Example](/grafana-example.png)

## 0) pre requisites

- Balena CLI:
  - Windows/Linux: https://github.com/balena-io/balena-cli/blob/master/INSTALL.md
  - MacOS: run `brew install balena-cli` (require [Homebrew](https://docs.brew.sh/Installation))

## 1) Setup Balena App and device
***1.a Account:***
- Go to [balena.io](https://dashboard.balena-cloud.com/login) and sign up for a account or log in using google or github.

***1.b Application:***
- Once you access the account, create a new application:
  - ***Application Name***: for this demo, use **isp-monitor**
  - ***Device Type***: Select the type od SoC you will use, in my case I have a Raspberry 3b so I will select **Raspberry Pi 3**
  - ***Application Type***: Leave the default (**Starter**)
  - Click in "Create new application"

***1.c Device:***
- Now that the application is ready, lets setup the device:
  - At the application Click **Add device** on the top left corner
  - Leave all the default values and only change (if needed), the Network connection for Ethernet only or Ethernet & Wifi.
  - At the bottom, Click on the button to download the OS image.
  - Unzip the file and burn the .img file to the SD card with [Etcher](https://www.balena.io/etcher/).
  - Insert the SD card into the RaspberryPi, connect it to the network and power it on.
  - After a moment you should see the device on the application's dashboard.


## 2) Deploy the code to the Balena application and Device.

***2.a get and update the code:***
- Download this project and unzip it.
- Open the file `/telegraf/telegraf.conf` and update the following:
  - At line 19, replace **DEVICE_IP** for the raspberry pi IP address.
  - At line 60, replace **ISP_DNS1** and **ISP_DNS2** for the actual ISP dns.
  - At line 65, replace **ISP_DNS1** and **ISP_DNS2** for the actual ISP dns and  **ROUTER_IP** for your router IP.
  - Save and exit the files

***2.b Push and deploy the code:***
  - From a terminal, at the project folder run the commands:
    - `balena login` to login into balena.io
    - `balena push isp-monitor` to push the code to the application
    - If all goes well you should see the three apps deployed and the unicorn in ascii code.

## 3) Management and configuration

***3.a Services Install Confirmation:***
- At the Balena dashboard, access the application `isp-monitor` and on the summary section, at the bottom you should see the ***Services*** section with the following services up and running:
  - Grafana
  - InfluxDB
  - Telegraf

***3.b Configure InfluxDB Data retention policy***
- From the balena application dashboard, on the lower right side, open a terminal to INFLUX.
- Run the command `influx` and then `CREATE RETENTION POLICY speedtest90days ON speedtest DURATION 90d REPLICATION 1 DEFAULT`.
- That would create a new retention policy to overwrite the default one, to keep every measurement in the "speedtest" database for up to 90 days before deleting them.
You can change those number depending on the size of your SD card.


***3.c Configure Grafana Data source***
- From your computer, Go to https://***DEVICE_IP***:3000
  - username: ***admin***
  - password: ***admin***   
- Select "Add data source" and then InfluxDB as the source type.
- Under HTTP, add the URL http://***DEVICE_IP***:8086
- Use the following InfluxDB Details:
  - Database: ***speedtest***
  - Database User: ***root***
  - Database Pass: ***password***
  - Method: ***GET***
- At the bottom, click on **SAVE AND TEST**, if you get a green banner, you are good.
- Click on ***BACK*** to go to the main menu and continue with the dashboard.

***3.d Configure Grafana dashboard***
- On the left side of the screen, click on the ***+*** icon and create a new dashboard.
- Create different panels changing the "measurement" from "FROM" and "field(value)" from "SELECT", here are some examples:
  - Average response time from DNS server: `SELECT mean("query_time_ms") FROM "dns_query" WHERE $timeFilter GROUP BY time($__interval) fill(null)`
  - Raspberry Pi CPU temperature: `SELECT mean("value")  / 1000 FROM "execcputemp" WHERE $timeFilter GROUP BY time($__interval) fill(null)`
- Clic on Apply (Upper right)

You are done!

## 4) How it works?
***Telegraf*** will do several tests, pings and queries to the IPs we setup on the `telegraf.conf` file. These data will be stored in the ***Influx*** DB and using ***Grafana*** we can query and display this data.


## Special Thanks
To Mr. [@sbehrends](https://github.com/sbehrends) for the idea and samples that evolved into a stand-alone ISP monitor. (his implementation is even greater with a dedicated cloud server, TLS, etc.)
My original implementation was the speedtest-cli sending JSON to IFTTT and from there to my personal Google Drive but having influxDb+Grafana is way better than Google Drive spreadsheet graphics capability and escalates easier.

To the Ookla team for the speedtest.net CLI tool, more info [Github](https://github.com/teamookla), [Facebook](https://www.facebook.com/speedtest) or [Twitter](https://twitter.com/speedtest).

## Disclaimer
This example is provided as a reference for your own usage and is not to be considered my own product.
By using it, you are approving the license from different products and regulations like Speedtest, Telegraf, InfluxDB, Grafana, Docker, RGPD and more.
This article involves products and technologies which do not form part of my catalog. Technical assistance for such products is limited to this article.
