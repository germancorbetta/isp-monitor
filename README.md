# isp-monitor + DNS
ISP monitor using a Raspberry PI with balena.io, Telegraf agent, Speedtest CLI, InfluxDB and Grafana

## Objective

The end goal is to have several network services running on a Raspberry Pi:

![Schema Example](/schema.png)

Services:

- Monitor your internet connection

![Grafana Example](/grafana.png)

- Use Pihole to block unwanted DNS queries, and enable secure and encrypted DNS connections with DNSCrypt-Proxy

![Pihole Example](/pihole.png)


## 0) What do you need:

- Balena CLI installed:
  - Windows/Linux: https://github.com/balena-io/balena-cli/blob/master/INSTALL.md
  - MacOS: run `brew install balena-cli` (require [Homebrew](https://docs.brew.sh/Installation))
- A Raspberry Pi 3b board
- Know your ISP DNS(s) IP address(es)
- Know your Router IP address (usually is 192.168.0.1)
- Know the Raspberry Pi IP in advance if possible, if now it will be displayed in the application dashboard at Balena. (Step 1.c)


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
  - If needed, you will see the Raspberry PI ip address after a few minutes.


## 2) Deploy the code to the Balena application and Device.

***Push and deploy the code:***
  - From a terminal, at the project folder run the commands:
    - `balena login` to login into balena.io
    - `balena push isp-monitor` to push the code to the application
    - If all goes well you should see the three apps deployed and the unicorn in ascii code.

## 3) Management and configuration

***3.a Services Install Confirmation:***
- At the Balena dashboard, access the application `isp-monitor` and on the summary section, at the bottom you should see the ***Services*** section with the following services up and running: (Note: May take a few minutes)

  - Grafana
  - InfluxDB
  - Telegraf
  - Pihole
  - DNSCrypt-Proxy

***3.b Configure InfluxDB Data retention policy***
- From the balena application dashboard, on the lower right side, open a terminal to INFLUX.
- Run the command `influx` and then `CREATE RETENTION POLICY speedtest90days ON speedtest DURATION 90d REPLICATION 1 DEFAULT`. That will create a new retention policy to overwrite the default one, to keep every measurement in the "speedtest" database for up to 90 days before deleting them. You can change those number depending on the size of your SD card.

***3.c Configure Environment variables***
- From the balena application dashboard, on the left, select 'Device Service Variables'
- Click on "add variable"
- Select the service "telegraf"
- Add the corresponing variable for the isp dns servers, router and raspi ip addresses. (dont change the variable name, just the value)

![variables Example](/balena_variables.png)

***3.d Configure Grafana Data source***
- From your computer, Go to https://***RASPI_IP***:3000
  - username: ***admin***
  - password: ***admin***   
- Select "Add data source" and then InfluxDB as the source type.
- Under HTTP, add the URL http://***RASPI_IP***:8086
- Use the following InfluxDB Details:
  - Database: ***speedtest***
  - Database User: ***root***
  - Database Pass: ***password***
  - Method: ***GET***
- At the bottom, click on **SAVE AND TEST**, if you get a green banner, you are good.
- Click on ***BACK*** to go to the main menu and continue with the dashboard.

***3.d The Grafana dashboard***
- Create your own:
  - On the left side of the screen, click on the ***+*** icon and create a new dashboard.
  - Create different panels changing the "measurement" from "FROM" and "field(value)" from "SELECT".
  - Note that some values like the the network speed need to be recalculated by adding the math component. Here is an example: `SELECT last("download_bandwidth")  / 131072 FROM "Speedtest" WHERE ("host" = 'morning-night') AND $timeFilter GROUP BY time($__interval) fill(null)` morning-night is the name of the device, your will be different, click on it and change it.
- Import mine:
  - On the left side of the screen, click on the ***+*** icon and select IMPORT.
  - Select the file `GrafanaDashboard.json` included in this repo.
  - Complete the process
  - `morning-night` is the name of my device, yours will be different. Edit each panel and update that value.

- Import one from the Grafana site:
  - https://grafana.com/grafana/dashboards/12428

You are done!

***Note:*** The RaspberryPi3 LAN port is 10/100 so if your internet connection is above that, any measurement will cap around 100Mbps. You can use an adapter to archive greater speeds.

## 4) DNS server

***4.a Update pihole admin password***
- From the balena app dashboard, open the terminal for the pihole service.
- Run the command `sudo pihole -a -p`
- type and confirm the password

***4.b Access Pihole and update it***
- Open in a browser http://[RASPI_IP]/admin
- click on LOGIN and type the password.
- Go to TOOLS -> UPDATE GRAVITY.
- Click on 'UPDATE'

***4.c Update the devices on your network to user the RASPI_IP as DNS Server***
- Do I have to explain this part?

## 5) Behind the scenes

***MONITORING:***

**Telegraf** will do measure the internet bandwidth using speedtest-cli, also will measure average pings response times from google.com, 8.8.8.8 (google DNS), 1.1.1.1 (CloudFlare) your router, and the ISP DNS servers. Finally will perform dns queries to the ISP dns servers, google and cloudflre and measure the response time among other data capture.

All this information will be stored in the ***Influx*** database.

Finally, ***Grafana*** will query and display the data from Influx in different dashboard and panels we can customize to our needs.

For more information on how to configure Telegraf, visite this [repo](https://github.com/influxdata/telegraf/blob/master/docs/CONFIGURATION.md)

***DNS:***

In short, Pihole block any unwanted DNS queries from the client (your computer assuming you setup the Pihole ip as primary dns server), but also, by the addition of the DNSCrypt Proxy, any DNS is automatically encrypted avoiding DNS Snooping.


## 6th Step: [OPTIONAL] Analyze the results after a long time...
This project has been very useful for me to track and evaluate my ISP, using it I was able to get approval to change the ISP and also request the money back due to having real information about the service outages.
But! having the database inside the Raspberry forced a lot of I/O events that eventually corrupted the SD card a few times in the last 6 months, also losing the entire database. So to avoid the issue, I removed most of the I/O events by just moving to a Raspberry + Cloud solution that I will talk about in another project. As a spoiler, I can confirm that I´m using the same Balena.io + Telegraf on the Raspberry, but having the InfluxDB and on the Cloud using an InfluxDB Cloud Free Tier. 

## Special Thanks
To Mr. [@sbehrends](https://github.com/sbehrends) for the idea and samples that evolved into a stand-alone ISP monitor. (his implementation is even greater with a dedicated cloud server, TLS, etc.)

My original implementation was the speedtest-cli sending JSON to IFTTT and from there to my personal Google Drive but having influxDb+Grafana is way better than Google Drive spreadsheet graphics capability and escalates easier.

To the Ookla team for the speedtest.net CLI tool, more info [Github](https://github.com/teamookla), [Facebook](https://www.facebook.com/speedtest) or [Twitter](https://twitter.com/speedtest).

## Disclaimer
This example is provided as a reference for your own usage and is not to be considered my own product.

By using it, you are approving the license from different products and regulations like Speedtest, Telegraf, InfluxDB, Grafana, Docker, RGPD and more.
This article involves products and technologies which do not form part of my catalog. Technical assistance for such products is limited to this article.
