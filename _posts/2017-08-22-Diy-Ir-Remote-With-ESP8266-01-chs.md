---
layout: post
title: 基于ESP8266-01 Diy万能红外遥控器 
categories: [diy, esp8266]
description: A very cheap Wifi-based Ir Remote
keywords: 8266, ir remote
---

## Intro
最近搞了一个通过Wifi控制的万能红外遥控器来遥控我的乐视TV，主要是用了ESP8266-1这个小玩意儿。

## 关于 ESP8266-01
ESP8266-01 一个非常便宜而且小的Wifi模块，还带MCU。我买的时候9块9包邮，9块9买不了吃亏，买不了上当。

**图:**

![](https://cdn.instructables.com/FE9/58ZS/IJX7FON7/FE958ZSIJX7FON7.MEDIUM.jpg) ![](https://cdn.sparkfun.com/assets/parts/1/1/1/2/9/13678-02.jpg)

## 第 1 步: 材料清单

* ESP8266-01（废话）
* 红外发射管，一个或者多个
* 红外接收器，比如 VS1838B (学习用)
* S8050三极管，或者其他NPN三极管
* 330 到 10k 欧姆电阻，都行
* 电线
* USB to TTL 串口线(下载程序用)
* 3.3V电源
* 面包版
* 下载用的PCB板(ESP8266-01 不能直接插到面包版上)

## 第 2 步: 电路图、连线
**温馨提示: 只能用3.3V电压的电源，5V电源会烧毁模块！**

S8050，就是图中的Q1，用来控制红外发射管LED1的通断。

<img src="http://blog.2the.top/images/posts/8266ir/2.png" width="50%"/><img src="http://blog.2the.top/images/posts/8266ir/1.png" width="50%"/>

红线是3.3v正极，黑线是负极，其他都是信号线。

## 第 3 步: 软件
代码在这里 [ESP8266IRRemote](https://github.com/easyfunny/ESP8266_IR_Remote)。 使用到的库有：

* [WiFiManager](https://github.com/tzapu/WiFiManager) 配置wifi连接的。
* [IRremoteESP8266](https://github.com/markszabo/IRremoteESP8266) 牛逼哄哄的红外库
* ESP8266WebServer Web服务器，用来处理指令
* [esp8266IRServer](https://github.com/Daniel-t/esp8266IRServer)，特别感谢这位，他的代码我基本没怎么改就能用。这哥们好像是个德国人。
* 等等
 
## 第 4 步: 使用说明
1. 在文件471、472行，修改主机名，初始SSID和密码。
```cpp
  WiFi.hostname(HOSTNAME);
  wifiManager.autoConnect(HOSTNAME, "1234567890");
```
2. 编译，用串口线下载。

3. 用手机或电脑连接到wifi，SSID 为 *esp8266-ir* 密码 *1234567890*。然后根据下图连接到你的wifi路由器上：
![ESP8266 WiFi Captive Portal Homepage](http://i.imgur.com/YPvW9eql.png) ![ESP8266 WiFi Captive Portal Configuration](http://i.imgur.com/oicWJ4gl.png)

4. 用各种方法找到你的IP地址. 比如我的是 _192.168.100.18_。
在Windows， Linux， MacOS下 执行：
```shell
arp -a 
``` 
在MacOS下执行：
```shell
ping esp8266-ir.local 
```

5. 如果是Windows系统, 修改 _/html/upload.html_ 中的IP地址，保存后用浏览器打开。 上传 _html_ 文件夹中的全部文件。先上传*success.html*， __不要上传__ *upload.html* 和 *upload.sh*。。
![](http://blog.2the.top/images/posts/8266ir/upload.html.jpg)

6. 如果是MacOS 或者 Linux 系统， 修改 _/html/upload.sh_ 文件中的IP地址。 
![](http://blog.2the.top/images/posts/8266ir/upload.sh.jpg)
运行命令：
```shell
./upload.sh
```

7. 打开浏览器访问 *http://YOURIPADDRESS*. 比如 *http://192.168.100.18* 或者 *http://esp8266-ir.local*.
![](http://blog.2the.top/images/posts/8266ir/3.PNG)

8. Duang，这是我的乐视电视的遥控器。

## 第 5 步: 改成你的遥控器码.
1. 拿你自己的遥控器，冲着ESP8266按一个键。
2. 打开浏览器，访问 *http://YOURIPADDRESS/learn*. 比如 *http://192.168.100.18/learn* 或者 *http://esp8266-ir.local/learn*。
3. json里有 **code**， **bits**， **protocal**。
4. 修改 *config.json* 文件。
5. 上传 *config.json* 文件。
6. Duang。
.local