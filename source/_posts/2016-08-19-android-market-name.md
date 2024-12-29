---
title: 从adb获取Android设备的市场名和外观图片
date: 2016-08-19 01:39:36
categories: Android
tags: [Android]
---
全球一共有多少款Android设备呢？根据[Google注册在案的数据](https://support.google.com/googleplay/answer/1727131)，截止2016年8月5日，总计有12518款不同的Android设备型号，这里还没包括几年前中国山寨厂商Android设备。Android设备型号总量这个数字，每天都在增长。

接触过Android开发的人也许遇到，或者想到过，拿到一台Android设备，如何能够得到它在市场售卖时候的设备型号名称。
打开Android系统的“设置”->“关于手机”，可以看到设备的基础信息，其中有型号一栏。对于一部分手机，型号名和市场名是一致或者接近的，但有相当一部分设备的型号名和市场叫法没有关联很小，甚至任何关系。例如华为Mate7在系统中的型号名是 `HUAWEI MT7-CL00` ，一般人很难联想到Mate7；三星同一款市场叫法的手机，在不同国家发行其模型名不一样，Galaxy Note4在中国发行的版本的型号名是 `SM-N9108V` ，笔者在第一次拿到这款设备的时候，如果不看包装盒，几乎无法从系统中获取到该手机的市面叫法。

笔者最近在开发[Appetizer.io](https://www.appetizer.io)时遇到一个需求，需要根据adb能获取到的Android设备数据，得到设备的样例图片和市场名。这种类似的需求场景，在Android相关的开发中有可能碰到，笔者在此分享一下实现该需求的思路与方法。

<!--more-->

# Android设备型号相关参数

在Android系统的根目录下，有文件 `/system/build.prop` ，这个文件是Android系统构建时产生的，描述了系统软件和设备硬件相关的名称和参数。Android系统所有获取这些参数的API的根源，都是从这个文件获取。

下面一段，是通过 `adb shell` 获取的 **小米4LTE** 型号手机 `/system/build.prop` 中的部分内容。从中可以看出，小米的 `ro.product.model` 描述与市面叫法接近， `ro.product.name` 和 `ro.product.device` 类似昵称，通常是内部开发代号， `ro.product.brand` 和 `ro.product.manufacturer` 分别指品牌和代工商，对于小米来说都是 *Xiaomi* ，存在一些设备的品牌和代工商是不同公司，例如Google的Nexus系列。

``` Shell
weilundeMacBook-Pro:~ azard$ adb shell
shell@cancro:/ $ cat /system/build.prop | grep ro.product
ro.product.model=MI 4LTE
ro.product.brand=Xiaomi
ro.product.name=cancro
ro.product.device=cancro
ro.product.board=MSM8974
# ro.product.cpu.abi and ro.product.cpu.abi2 are obsolete,
# use ro.product.cpu.abilist instead.
ro.product.cpu.abi=armeabi-v7a
ro.product.cpu.abi2=armeabi
ro.product.cpu.abilist=armeabi-v7a,armeabi
ro.product.cpu.abilist32=armeabi-v7a,armeabi
ro.product.cpu.abilist64=
ro.product.locale=zh-CN
# ro.build.product is obsolete; use ro.product.device
ro.product.manufacturer=Xiaomi
ro.product.cuptsm=XIAOMI|ESE|02|01
```

这里插一段花絮，在被root的Android系统中， `/system/build.prop` 可以被修改。有些手机游戏的活动只针对特定型号的设备，当年《炉石传说》有三星Galaxy手机专用卡背，玩家可以通过运行在PC上的Android模拟器，生成指定的 `build.prop` ，伪造成三星Galaxy手机获得游戏卡背。
![](http://wx2.sinaimg.cn/large/989ea82cly1g1t0q9s3j5j20cv0csq9w.jpg)

# 获取设备外观图片

如何从互联网得到一部手机的外观图片，通过搜索引擎当然是最好的办法。但根据adb能够得到的数据，准确的查找到一张手机的外观图，最好的方法还是使用专业的图片库。

笔者在网络上爬行，发现了移动设备数据库[DeviceAtlas](https://deviceatlas.com/resources)，这个库收录了各种型号的移动设备的硬件参数，但监狱该库收录的设备型号并不完全，例如三星只在某些国家发型的型号没有收录，而且很多设备没有外观图片，因此放弃使用该库作为设备外观图片的来源。

[GSMArena](http://www.gsmarena.com/)是一个偏向媒体性质的网站，同样收录了各种型号的移动设备，从数量上GSMArena没有收录完全一种设备在各种国家发售的型号，但是该网站的设备外观图片覆盖面广且清晰。最终选择使用此网站作为图片源，复用GitHub上的开爬虫项目[gsmarenacrawler](https://github.com/sanbornsen/gsmarenacrawler)，将所有的设备图片全都爬到本地，外观图片名并非 `/system/build.prop` 中的任何一项，而是接近市场大众的通俗叫法，我们定义为 **市场名** 。

# 获取设备市场名

若要从 `/system/build.prop` 的信息映射到图片名，首先要得到一台设备的市场通俗叫法。鉴于之前提到的，很多设备的型号名，设备名与市场叫法没有任何联系，这一步必须使用外部数据表帮助建立映射。

GitHub上维护的开源项目[AndroidDeviceNames](https://github.com/jaredrummler/AndroidDeviceNames)满足了这个需求场景，提供了嵌入Android的SDK直接获取设备的市场名，可以用于一些需要显示市场名的App上。这个项目实际上是存储了一个大型的从 `/system/build.prop` 映射到设备市场名的 json 格式的数据结构。但是考虑到这是私人维护的项目，设备不一定齐全，也不一定能保证持续更新，需要更加权威的数据。

好在Google提供了使用在Google Play的设备市场名数据表格，来源是手机开放联盟的注册信息（[链接1](https://support.google.com/googleplay/answer/1727131)，[链接2](https://support.google.com/googleplay/android-developer/answer/6154891)）。表格中有四列，是每台注册设备的 `ro.product.manufacturer`，`ro.product.model`，`ro.product.device`，以及市场通俗名。表格中的三星Galaxy Note4就有二十多个不同的型号，应该是最权威的收录。

这样，就得到了一个 `[ro.product.device, ro.product.model] --> market_name` 的大表。

# 市场名与图片的模糊匹配

Google的表中的市场名，与GSMArena的图片名在内容和格式上有些许不同，需要进行模糊匹配。笔者根据特征做了如下操作：

1. 图片名对中扩线，下扩线和空格进行切分，数字和字母切分开，字母统一转成小写，对于每个图片名得到字符串数组 `listA`
2. 市场名对空格进行切分，数字和字母切分，字母统一转成小写，加上品牌名，对于每个市场名得到字符串数组 `listB`
3. 将每个图片名的字符串数组 `listA` 与市场名 `listB` 进行配对，进行如下算法

``` Python
score = 0.0;
count = 0;
for market_word in listB:
    for pic_word in listA:
        if (market_word.find(pic_word)):
            count += 1;
            break;
score = count / len(listB)
```

对于每个市场名，得分最高的图片名即为该模糊匹配算法的最佳匹配项。

这样得到了 `market_name --> picture` 的大表，与之前得到的表结合，最终得到了两张表：

1. `[ro.product.device, ro.product.model] --> market_name`
2. `[ro.product.device, ro.product.model] --> picture`

通过这两张表，就可以根据 `/system/build.prop` 中的信息得到设备的市场名和外观图片。
