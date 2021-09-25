---
title: Raspberry Pi使用sds011传感器记录空气质量
tags: raspberrypi sds011
key: raspberrypi-sds011
comments: true
---

使用SDS011传感器获取空气质量数值(PM2.5/PM10)

<!--more-->

- 准备一个SDS011的传感器，像我一样的硬件白痴可以直接用usb接口；
- 在[github](https://gist.github.com/kadamski/92653913a53baf9dd1a8)下一段别人写好的代码，需要注意几点：
  - 这段代码是python2写的，如果要在python3使用，需要根据报错的位置做修改；
  - ser.port改成usb设备(`/dev/ttyUSB0`)；
  - 代码的流程作者在回复中说了，根据自己的需要修改即可。例如我是每隔一段时间将数值记录到文件的末尾，那么把原来直接用print输出的函数改掉，再加一段file操作即可。
