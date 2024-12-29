---
title: Android跨进程模拟触屏事件(sendevent)
date: 2015-06-13 1:18:07
categories: Android
tags: [Android]
---
**转载请注明出处**
* 这是一篇通俗易懂介绍用软件的方式模拟Android触屏事件，包括其他传感器事件的一篇博文。
* 整个Internet上还没发现一篇**如此友善且更加详细**的博文（Google的文档很详细但不friendly）。
* 用该技术可以实现一些很cool的东西，亦或是干一些坏事。

<!--more-->
# 跨进程模拟触屏事件的作用
有很多在一个Activity中实现虚拟触摸的方法，但是无法做到跨进程虚拟触摸。无论是Google提供的`Monkey`还是`MonkeyRunner`都不能很好的脱离PC进行虚拟触碰，更别说写一个后台进程实现一些tricky的虚拟触碰。

可以看下面这个视频，是我用跨进程虚拟触碰的技术，再结合一双手套（也是我做的），实现不接触屏幕玩O2JAM-U的一个视频。手套上的WiFi模块发包给Android AP，后台一个进程会handle发送的包并实现跨进程虚拟触碰玩游戏。

<embed height="452" width="544" quality="high" allowfullscreen="true" type="application/x-shockwave-flash" src="http://share.acg.tv/flash.swf" flashvars="aid=2440147&page=1" pluginspage="http://www.adobe.com/shockwave/download/download.cgi?P1_Prod_Version=ShockwaveFlash"></embed>

模拟触屏事件也可以做许多坏事，例如侦听支付宝解锁的手势加以模拟之类的，而且属于系统层面的模拟是应用无法分辨的。至于使用这个方法做很cool很tricky的事或是坏事与我无关，在此只介绍原理和技术。

# 注意事项
* 我写的触屏事件注入的demo代码在
[https://github.com/Azard/VTouch/blob/master/app/src/main/java/me/azard/vtouch/event/Nexus5.java](https://github.com/Azard/VTouch/blob/master/app/src/main/java/me/azard/vtouch/event/Nexus5.java)，对应的手机型号是Nexus5。
* 上面的demo能在Android 4.4.4下正常运行，但无法在Android 5.0以上使用，在[stackoverflow](http://stackoverflow.com/questions/27496968/inject-touch-screen-events-android-5-0-dev-input-eventx)上有人说是由于SELinux的原因造成的，但是我进行相应操作依然没有能够在Android 5.0以上成功运行。如果有相关解决办法还希望在博文下方留言。
* 上面的demo调用了[RootTools 3.5](https://github.com/Stericson/RootTools)包装执行Android的Linux层命令，最新版本的`RootTools`的API和我的demo调用的API会有区别。
* 不同的手机的硬件中断触发事件略有区别，可以在`adb`下使用`getevent`进行测试。
* `getevent`得到的数值是16进制的，`sendevent`输入的参数是10进制的，注意进行转换。
* 如果依然没有效果，尝试先修改文件权限，在su后调用`chmod 777 /dev/input/event[x]`。
* 使用`RootTools`库执行Linux层命令，不要使用`Runtime.getRuntime().exec()`。

# 触屏事件
在Android中，所有的传感器在发生事件时，都会向Linux层对应传感器的某个文件发送一个event，这整个流程我不是十分清楚也不能妄自吹比。

以我个人的角度进行抽象，可以认为任何传感器的操作会产生一个硬件中断，让系统调用一个sendevent，后续Android如何处理该event就不在本文的考虑之内，因为本文只讨论如何用软件的方式模拟硬件产生的信息。

## 关于getevent
在Android提供的可执行工具中，包含一个`getevent`，侦听系统接收到的event，在`/system/bin/`下面，可以用adb查看一下。（关于adb如何配置请参考其他文章）

Nexus5手机在adb shell下运行getevent，会出现如下信息：
> shell@hammerhead:/ $ getevent
> getevent
> add device 1: /dev/input/event5
>   name:     "msm8974-taiko-mtp-snd-card Headset Jack"
> add device 2: /dev/input/event4
>   name:     "msm8974-taiko-mtp-snd-card Button Jack"
> add device 3: /dev/input/event3
>   name:     "hs_detect"
> add device 4: /dev/input/event1
>   name:     "touch_dev"
> add device 5: /dev/input/event0
>   name:     "qpnp_pon"
> add device 6: /dev/input/event2
>   name:     "gpio-keys"

很显然，上面的每个device就相当于一个传感器，对应一个`/dev/input/event[x]`文件。


然后，按一下Nexus5的开机按钮，adb shell上会显示：
> /dev/input/event0: 0001 0074 00000001
> /dev/input/event0: 0000 0000 00000000
> /dev/input/event0: 0001 0074 00000000
> /dev/input/event0: 0000 0000 00000000

多次尝试，可以发现按下会出现前面两行，松开会出现后面两行。

稍微轻轻触碰一下屏幕，会显示8个event（我有次很快按出了只显示6个event的情况，没有第4、5行）：
> /dev/input/event1: 0003 0039 00003c0f
> /dev/input/event1: 0003 0035 00000303
> /dev/input/event1: 0003 0036 000004df
> /dev/input/event1: 0003 003a 00000030
> /dev/input/event1: 0003 0030 00000004
> /dev/input/event1: 0000 0000 00000000
> /dev/input/event1: 0003 0039 ffffffff
> /dev/input/event1: 0000 0000 00000000

同样多次尝试，发现长按会先显示前6行，最后2行是释放时会出现的event。

调用`getevent -l`做同样的操作,返回的信息会以常量的名字出现，而不是具体的数值：
> /dev/input/event0: EV_KEY     KEY_POWER       DOWN
> /dev/input/event0: EV_SYN     SYN_REPORT      00000000
> /dev/input/event0: EV_KEY     KEY_POWER       UP
> /dev/input/event0: EV_SYN     SYN_REPORT      00000000

下面是触屏事件的：
> /dev/input/event1: EV_ABS     ABS_MT_TRACKING_ID  00003c19
> /dev/input/event1: EV_ABS     ABS_MT_POSITION_X   00000333
> /dev/input/event1: EV_ABS     ABS_MT_POSITION_Y   000004d6
> /dev/input/event1: EV_ABS     ABS_MT_PRESSURE     00000033
> /dev/input/event1: EV_ABS     ABS_MT_TOUCH_MAJOR  00000005
> /dev/input/event1: EV_SYN     SYN_REPORT          00000000
> /dev/input/event1: EV_ABS     ABS_MT_TRACKING_ID  ffffffff
> /dev/input/event1: EV_SYN     SYN_REPORT          00000000

粗略的发现，单次触屏事件的第2、3个event是坐标，4、5似乎和长按有关，因为接触屏幕时间很短的话是没有第4、5个event的。`EV_SYN`是信号量同步相关的东西，可以理解为一个transaction的commit操作。

* 具体关于`getevent`的参数可以使用`getevent -h`进行查看，也可以参考Google提供的文档：
[http://source.android.com/devices/input/getevent.html](http://source.android.com/devices/input/getevent.html)
* 关于这些event的常量名的具体意义，可以参考Google的另一个很完整的文档：
[http://source.android.com/devices/input/touch-devices.html](http://source.android.com/devices/input/touch-devices.html)

## 关于sendevent
首先先尝试一下最简单的电源按钮，试图唤醒手机。
根据`getevent`的数据，将其数值翻译成10进制，对应的KEY_POWER是`0x0074`，也就是10进制下的`116`。在adb shell中输入如下命令（并不需要很快连续输入，可以慢慢来）：
```
shell@hammerhead:/ $ sendevent /dev/input/event0 1 116 1
shell@hammerhead:/ $ sendevent /dev/input/event0 0 0 0
shell@hammerhead:/ $ sendevent /dev/input/event0 1 116 0
shell@hammerhead:/ $ sendevent /dev/input/event0 0 0 0
```
如果尝试了，会发现只输入前2行手机屏幕就会亮。但是如果不继续输入后面2行，系统会认为用户一直按住了电源键，因此这时在物理按下一次电源键，手机屏幕不会关闭，需要再按两次才行。

可以试想一下，开个包含handle事件的App在后台运行，触发handle后执行这个操作，可以实现虚拟触碰方式的关机。

# 分析一些高级的操作
在上面引用的[Google的第二个文档](http://source.android.com/devices/input/touch-devices.html)中，**Touch Device Driver Requirements** 这一节的第三小节，介绍了Multi-touch设备的event相关的一些参数，显然现在所有的Android设备都支持多点触控，我的Nexus5也是。

我稍微翻译下这一小节：
> * `ABS_MT_POSITION_X`: (必须) 报告触碰的X坐标。
> * `ABS_MT_POSITION_Y`: (必须) 报告触碰的Y坐标。
> * `ABS_MT_PRESSURE`: (可选) 报告触碰的压力大小或者信号强度。
> * `ABS_MT_TOUCH_MAJOR`: (可选) 报告触碰的代表性区域, 或者触碰的最长的尺寸。
> * `ABS_MT_TOUCH_MINOR`: (可选) 报告触碰的最小的尺寸。 如果`ABS_MT_TOUCH_MAJOR`报告的是区域测量，则不使用。
> * `ABS_MT_WIDTH_MAJOR`: (可选) 报告触碰本身的代表性区域，或者触碰本身的最大尺寸。 如果触碰的尺寸不知道则不使用。
> * `ABS_MT_WIDTH_MINOR`: (可选) 报告触碰本身的最小尺寸。 如果触碰的尺寸不知道则不使用。
> * `ABS_MT_ORIENTATION`: (可选) 报告触碰的方向。
> * `ABS_MT_DISTANCE`: (可选) 报告触碰本身和表面的距离。
> * `ABS_MT_TOOL_TYPE`: (可选) 报告触碰是`MT_TOOL_FINGER`还是`MT_TOOL_PEN`.
> * `ABS_MT_TRACKING_ID`: (可选) 报告触碰的跟踪ID。跟踪ID是一个非负的任意整数，用来分辨多个同时的操作。例如，当多个手指触碰设备，在手指还在屏幕上时每个手指绑定一个独立的跟踪ID，当手指离开屏幕后，跟踪ID可能被重新使用。
> * `ABS_MT_SLOT`: (可选) 报告触碰的slot ID，在使用Linux多点触碰协议B的情况下使用。查看Linnux多点触碰协议文档获取详细信息。

我的最终目标是实现一个能够实现多点在任意坐标短按，长按的库。

## event分析
经过我使用Nexus5多次实验并结合文档，发现短按长按的区别只是释放和按下的间隔的区别，在event上没有本质区别，多点触碰会增加一个`ABS_MT_SLOT`相关的event。

## 单点触碰event流
单点触碰比较容易，可以使用：
> ABS_MT_TRACKING_ID
> ABS_MT_POSITION_X
> ABS_MT_POSITION_Y
> SYN_REPORT

作为一次event流提交，`ABS_MT_PRESSURE`，`ABS_MT_TOUCH_MAJOR`都是可选的，如果要模拟压力大小和触摸区域大小的话可以加上这2个event。

## 单点释放event流：
> ABS_MT_TRACKING_ID -1
> SYN_REPORT

## 多点触碰event流：
多点触碰涉及到`ABS_MT_SLOT`的概念，这个并不是很难理解，在每次指定触碰的同时，增加一个`ABS_MT_SLOT`。在释放前，也要注明`ABS_MT_SLOT`。并且在默认情况下，`ABS_MT_SLOT`为0。这样也就能理解单点触碰所在的`ABS_MT_SLOT`一直为0，单点触碰只是多点触碰的一个特例。
需要注意的是，`ABS_MT_SLOT`一定要再`ABS_MT_TRACKING_ID`前定义，不然会出问题，使用的是之前的slot。
> ABS_MT_SLOT
> ABS_MT_TRACKING_ID
> ABS_MT_POSITION_X
> ABS_MT_POSITION_Y
> SYN_REPORT

## 多点释放event流：
> ABS_MT_SLOT
> ABS_MT_TRACKING_ID -1
> SYN_REPORT

## 滑动操作的event流：
根据我的实现，滑动操作其实并不难。实际上是间隔取样某一个`ABS_MT_TRACKING_ID`的多次触碰。只需先指定`ABS_MT_TRACKING_ID`跟所在的`ABS_MT_SLOT`，再改变X，Y坐标（如果某个坐标没变则不需要改变），然后同步信号量即可。

# 实现一些高级的操作
拥有了上一节的知识，实现起来的步骤就很清晰了。具体各个常量名和数值的关系，可以调用`getevent -lp`和`getevent -p`进行对比查看，这里发现`ABS_MT_SLOT`的范围是0到9，最多支持10点触控。
下面给一点干货。

下面一段代码调用了[RootTools 3.5](https://github.com/Stericson/RootTools)帮助进行Linux层命令调用，如果直接使用`Process`和`Runtime.getRuntime().exec()`进行Linux层的调用会间歇性出问题，我还没搞清楚为什么，如果有想法欢迎在博文下留言。
``` Java
package me.azard.vtouch.event;

import com.stericson.RootTools.RootTools;
import com.stericson.RootTools.exceptions.RootDeniedException;
import com.stericson.RootTools.execution.CommandCapture;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Vevent {

    public void executeCommand(String command) throws InterruptedException, IOException, TimeoutException, RootDeniedException {
        CommandCapture cmd = new CommandCapture(0, command);
        RootTools.getShell(true).add(cmd);
    }

    public void sendevent(String event_num, int param_1, int param_2, int param_3) {
        try {
            executeCommand(String.format("sendevent /dev/input/%s %d %d %d", event_num, param_1, param_2, param_3));
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        } catch (RootDeniedException e) {
            e.printStackTrace();
        }
    }
}
```
对Nexus5的event进行包装（如果需要增加触碰范围和压力大小可自行添加）：
``` Java
package me.azard.vtouch.event;

public class Nexus5 {

    Vevent vevent = new Vevent();

    static int EV_SYN = 0;
    static int EV_ABS = 3;

    static int SYN_REPORT = 0;
    static int ABS_MT_SLOT = 47;
    static int ABS_MT_TOUCH_MAJOR = 48;
    static int ABS_MT_POSITION_X = 53;
    static int ABS_MT_POSITION_Y = 54;
    static int ABS_MT_TRACKING_ID = 57;
    static int ABS_MT_PRESSURE = 58;


    public void touch(int finger_index, int x, int y) {
        vevent.sendevent("event1", EV_ABS, ABS_MT_SLOT, finger_index);
        vevent.sendevent("event1", EV_ABS, ABS_MT_TRACKING_ID, finger_index);
        vevent.sendevent("event1", EV_ABS, ABS_MT_POSITION_X, x);
        vevent.sendevent("event1", EV_ABS, ABS_MT_POSITION_Y, y);
        vevent.sendevent("event1", EV_SYN, SYN_REPORT, 0);
    }

    public void release(int finger_index) {
        vevent.sendevent("event1", EV_ABS, ABS_MT_SLOT, finger_index);
        vevent.sendevent("event1", EV_ABS, ABS_MT_TRACKING_ID, -1);
        vevent.sendevent("event1", EV_SYN, SYN_REPORT, 0);
    }
}
```

调用可以这样
``` Java
Nexus5 mNexus5 = new Nexus5();
mNexus5.touch(0, 208, 346);
mNexus5.release(0);
```

长按只需要晚点执行release即可，滑动只需要连续执行touch即可。

# 写在后面
这篇文章是因为我需要实现这个功能，但整个Internet还没有很friendly的文章和代码进行参考，只有各种片段，因此我对整个`sendevent`的操作进行了一个整理。

如果这篇博文对你的工作有所帮助，或者减少了查看各种资料的时间，可以通过**博客右边的支付宝链接或者二维码**对我进行资助，1元、2元意思意思即可。
