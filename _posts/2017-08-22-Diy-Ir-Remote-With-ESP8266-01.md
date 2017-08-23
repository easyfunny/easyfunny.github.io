---
layout: post
title: Diy Ir Remote With ESP8266-01
categories: [diy, esp8266]
description: A very cheap Wifi-based Ir Remote
keywords: 8266, ir remote
---

## Intro
Recently, I made a Wifi-based Ir Remote with ESP8266-1 that controls my LeTv. 

## About ESP8266-01
ESP8266-01 is the cheapest and smallest MCU with Wifi module that I can afford which costs about 9 CNY (USD 1.5) **in China**. Here is some description from [SparkFun](https://www.sparkfun.com/products/13678):

>The ESP8266 WiFi Module is a self contained SOC with integrated TCP/IP protocol stack that can give any microcontroller access to your WiFi network. The ESP8266 is capable of either hosting an application or offloading all Wi-Fi networking functions from another application processor. Each ESP8266 module comes pre-programmed with an AT command set firmware, meaning, you can simply hook this up to your Arduino device and get about as much WiFi-ability as a WiFi Shield offers (and that’s just out of the box)! The ESP8266 module is an extremely cost effective board with a huge, and ever growing, community.

**Here are some pictures from the Internet:**

![](https://cdn.instructables.com/FE9/58ZS/IJX7FON7/FE958ZSIJX7FON7.MEDIUM.jpg)![](https://cdn.sparkfun.com/assets/parts/1/1/1/2/9/13678-02.jpg)
## Step 1: Gather Your Materials.
You will need the following:

* ESP8266-01 Module.
* Infrared LED(s).
* S8050 (Or other NPN Epitaxial Silicon Transistor).
* A resistor from 330 to 10k Ohm.
* IR Receiver such as the VS1838B (For learning). 
* Some female du-pont connectors.
* USB to TTL Serial Cable(For program downloading).
* A 3.3v power supply(e.g.I use a AMS-1117 3.3).
* A breadboard.
* A PCB board for programing(ESP8266-01 is not breadboard friendly).

## Step 2: Scheme and Wiring
S8050, known as Q1 in the picture, plays a role like a switch that controls the LED1 to flash.

![](http://blog.2the.top/images/posts/8266ir/2.png)
![](http://blog.2the.top/images/posts/8266ir/1.png)

## Step 3: Software
The code is on my github. [ESP8266IRRemote](https://github.com/easyfunny/ESP8266_IR_Remote). I used some libraries, such as:

* [WiFiManager](https://github.com/tzapu/WiFiManager), a magical wifi configuration tool.
* [IRremoteESP8266](https://github.com/markszabo/IRremoteESP8266), a tool to learn/send IR codes.
* ESP8266WebServer, a simple web server to handle our requests.
* [esp8266IRServer](https://github.com/Daniel-t/esp8266IRServer), especially thanks to Daniel-t. He provides the main code. And I just changed a little.
* etc.

## Step 4: How to use
1. Change the host name, SSID, and wifi password at line 425/426(maybe).
```cpp
	WiFi.hostname("ESP8266_IR");
	wifiManager.autoConnect("ESP8266_IR", "1234567890");
```
2. Build and download to ESP8266 with serial cable.
3. Connect your phone or laptop to SSID named *ESP8266_IR* with password *1234567890*. And follow the guide to connect your ESP8266 to your wifi router. It looks like:
![ESP8266 WiFi Captive Portal Homepage](http://i.imgur.com/YPvW9eql.png) ![ESP8266 WiFi Captive Portal Configuration](http://i.imgur.com/oicWJ4gl.png)
4. Find out IP address by follow command. For example, my IP address is _192.168.100.18_.
```shell
arp -a (for Windows, Linux and MacOS)
``` 
or 
```shell
ping ESP8266_IR.lan (For MacOS)
```

5. If you are Windows, modify IP address to yours in file _/html/upload.html_. Save and open this file in your web browser. And upload all files(*success.html* should be the first one to upload) in _html_ dir __EXCEPT__ *upload.html* and *upload.sh*.
![](http://blog.2the.top/images/posts/8266ir/upload.html.jpg)
6. If you are MacOS or Linux, modify IP address to yours in file _/html/upload.sh_. And run command in terminal
```shell
./upload.sh
```
7. Open browser on your phone and access *http://YOURIPADDRESS*. For example, *http://192.168.100.18* or *http://ESP8266_IR.lan*.
![](http://blog.2the.top/images/posts/8266ir/3.PNG)
8. OK, it's working on my LeTV. 

## Step 5: Learn your IR remote CODE.
1. Press any button on your remote controller, toward to ESP8266.
2. Open your web browser and access *http://YOURIPADDRESS/learn*. For example, *http://192.168.100.18/learn* or *http://ESP8266_IR.lan/learn*.
3. You will find your **code**, **bits** and **protocal**.
4. Modify *config.json* to your code.
5. Upload *config.json* to your ESP8266.
6. Enjoy it.
