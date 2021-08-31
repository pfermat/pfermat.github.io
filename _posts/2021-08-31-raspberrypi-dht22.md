---
title: Raspberry Pi使用dht22传感器记录温度湿度
tags: raspberrypi dht22
key: raspberrypi-dht22
comments: true
---

最近夏天经常开关空调，于是就想知道室内的温度和湿度数值是多少，恰好有树莓派和usb显示屏，一番搜索之后发现实现起来还是很简单的。

<!--more-->

## 硬件
- 树莓派
- DHT22传感器
- 显示屏

传感器主要有硬件和软件两方面的问题：硬件方面我完全不懂，所以根据搜索结果买了一个焊好电路板的传感器；软件方面同样需要根据个人的不同能力，最简单的是类似DHT22这种随手一搜就有大量实例的传感器。

随后根据树莓派的型号找到相应的GPIO接口图，将正负极与GPIO线插上。

## 软件
读取DHT22的数据需要使用python下的Adafruit_DHT库，因此需要在系统中安装python之后，再使用pip安装Adafruit_DHT。如果途中出现报错，可能是python-dev或者基础的编译工具链没装。
准备完毕后，获取当前温度湿度就一句：
{% highlight python %}
humidity, temperature = Adafruit_DHT.read_retry(sensor, pin)
{% endhighlight %}
接下来按照实际使用情况，可以在python中用while定时获取数据，也可以使用其他工具，数据可以随取随丢也可以一直记录在log文件中。

## 参考资料
- [树莓派编程（三） — 使用DHT22测温湿度](https://www.ccarea.cn/archives/482)
- [树莓派感知温度和湿度，详细记录树莓派使用DHT22传感器](https://www.labno3.com/2021/03/21/raspberry-pi-humidity-sensor-using-the-dht22/)
