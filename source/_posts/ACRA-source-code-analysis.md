---
title: ACRA源码分析
date: 2016-02-14 17:48:00
categories: Android
tags: [Android]
---
[ACRA](https://github.com/ACRA/acra)是一个开源的Android平台程序崩溃信息收集小程序，可以嵌入到Android Project中，当该程序崩溃的时候ACRA能够在进程彻底结束前收集崩溃状态时的该应用和设备的各种信息，发送到搭建好的服务端，便于开发者进行程序错误信息的收集，开发者可以更好的改进程序提高兼容性。本文分析的是ACRA 4.8版本的源码。

<!-- more -->
# ACRA的使用方法

在分析源码前，先介绍下ACRA如何应用于一个Android Project之中。下面的代码是[GitHub ACRA wiki](https://github.com/ACRA/acra/wiki/BasicSetup)上的一段。

``` Java
import org.acra.*;
import org.acra.annotation.*;

@ReportsCrashes(
    formUri = "http://www.backendofyourchoice.com/reportpath"
)
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // The following line triggers the initialization of ACRA
        ACRA.init(this);
    }
}
```

使用者需要设置一下服务端的URL，再在程序的Application类里加上`ACRA.init(this)`就全部设置好了。现在当这个App崩溃的时候会自动发送崩溃信息到设定好的URL。

`ACRA.init(this)`究竟做了一些什么呢？

# ACRA.init()

最终调用的`init`方法的申明是
``` Java
public static void init(Application app, ACRAConfiguration config, boolean checkReportsOnApplicationStart)
```
在`org\acra\ACRA.java`类中，是个静态方法。后面两个参数都是偏好设置相关的，config相关的部分我们不关注，这里关注核心部分。

`init`中首先会对当前设备运行系统的API level和传入的Application进行一些check。

``` Java
boolean supportedAndroidVersion = (Build.VERSION.SDK_INT >= Build.VERSION_CODES.FROYO);
if (!supportedAndroidVersion){
    log.w(LOG_TAG, "ACRA 4.7.0+ requires Froyo or greater. ACRA is disabled and will NOT catch crashes or send messages.");
}

if (mApplication != null) {
    log.w(LOG_TAG, "ACRA#init called more than once. Won't do anything more.");
    return;
}
```

随后会new一个`ErrorReporter`对象出来，核心功能都在`ErrorReporter`中被注册。注意此时都运行在`Application.onCreate()`之中。

``` Java
final ErrorReporter errorReporter = new ErrorReporter(mApplication, configProxy, prefs, enableAcra, supportedAndroidVersion, !senderServiceProcess);
```

# ErrorReporter

在`org\acra\ErrorReporter.java`中，`ErrorReporter`的申明

``` Java
public class ErrorReporter implements Thread.UncaughtExceptionHandler
```

这里最关键的是`ErrorReporter`实现了`Thread.UncaughtExceptionHandler`接口，这个接口有一个`uncaughtException`方法需要实现。

在文档里这个interface的描述是
> Implemented by objects that want to handle cases where a thread is being terminated by an uncaught exception. Upon such termination, the handler is notified of the terminating thread and causal exception. If there is no explicit handler set then the thread's group is the default handler.

简单的说就是这个thread里所有的未被catch的exception会被传入到这个接口的方法里执行。这里也是ACRA的功能最核心的部分，拿到未被catch的exception，获取此时的设备信息然后打包发送到服务端

``` Java
@Override
public void uncaughtException(Thread t, Throwable e) {
    ......
    new ReportBuilder()
        .uncaughtExceptionThread(t)
        .exception(e)
        .endApplication()
        .build(reportExecutor);
    ......
}
```

这里把出错的`Thread`和抛出的对象`Throwable`传入`ReportBuilder`，最后再把这个`ReportBuilder`传入`reportExecutor`进行执行，比较绕。

`RerportBuilder`这个类有4个private变量，除了`Thread`和`Throwable`再附带点文字描述的信息，没有很特别的东西。
``` Java
private String message;
private Thread uncaughtExceptionThread;
private Throwable exception;
private final Map<String, String> customData = new HashMap<String, String>();
```

组合完了`ReportBuilder`后调用`build(reportExecutor)`，这个方法最后调用的是`ReportExecutor.execute(final ReportBuilder reportBuilder)`。这个方法同样是ACRA的核心代码之一。

# ReportExecutor.execute()

排除掉config相关的一些操作，包括给用户弹窗提醒等，这个方法内的核心部分实际上只有以下代码（默认为`SILENT`模式，默默的发送消息给服务端不提示用户）。

大致步骤为生成发送给服务端的数据，持久化保存数据，启动一个运行在新的process的`Service`，这个`Service`读取持久化存储的数据通过HTTP把数据发送到服务端。

## CrashReportData

``` Java
final CrashReportData crashReportData = crashReportDataFactory.createCrashData(reportBuilder);

// Always write the report file

final String reportFileName = getReportFileName(crashReportData);
saveCrashReportFile(reportFileName, crashReportData);
```

首先创真正有价值的`CrashReportData`，这里用了个工厂模式，把包含了`Thread`和`Throwable`的`reportBuilder`传给工厂类的方法，返回`CrashReportData`。

``` Java
public final class CrashReportData extends EnumMap<ReportField, String> {
    ......
    public JSONObject toJSON() throws JSONReportException {
        return JSONReportBuilder.buildJSONReport(this);
    }
}
```

`CrashReportData`继承了一个`EnumMap`，对应了崩溃信息的各种键值对，包括系统版本，各个硬件的信息，屏幕尺寸等等，当然肯定包含了函数栈调用信息。这些数据的获取方法一部分可以通过简单的调用系统API获取，另一些部分的获取方式比较复杂，都在`org.acra.collector`中，这些信息不详细说明。

`toJSON`方法方便把这个`EnumMap`转成文本格式的json。

## Sender

``` Java
if (reportingInteractionMode == ReportingInteractionMode.SILENT
        || reportingInteractionMode == ReportingInteractionMode.TOAST
        || prefs.getBoolean(ACRA.PREF_ALWAYS_ACCEPT, false)) {

    // Approve and then send reports now
    startSendingReports(sendOnlySilentReports, true);
    ......
}
```

``` Java
private void startSendingReports(boolean onlySendSilentReports, boolean approveReportsFirst) {
    if (enabled) {
        final SenderServiceStarter starter = new SenderServiceStarter(context, config);
        starter.startService(onlySendSilentReports, approveReportsFirst);
    } else {
        ACRA.log.w(LOG_TAG, "Would be sending reports, but ACRA is disabled");
    }
}
```

调用网络发送部分的策略是启动一个新的`Service`组件，读取持久化存储的数据，然后发送到服务端。至于为什么不直接开一个新的thread调用这个发送操作是有原因的。

* Android 4.4.4 默认的http client库是基于okhttp的，在获取HTTP返回信息的时候为了防止阻塞会开新的thread。
* Android 6.0使用了`ART`替代`Dalvik`开始，在shutdown的过程中不能开新的thread，否则又会跑出异常`java.lang.InternalError: Thread starting during runtime shutdown`。

因此ACRA的解决办法就是开新的process的Service调用http，这个解决方案的讨论过程见：

* [ACRA/acra#327](https://github.com/ACRA/acra/issues/327)
* [ACRA/acra#329](https://github.com/ACRA/acra/issues/329)
* [ACRA/acra#344](https://github.com/ACRA/acra/issues/344)

这个问题还有另外两种解决办法。一种是使用Android 4.4.4之前默认的Apache http client，不会开新的thread。还有一种解决办法是先持久化存储，等下次程序启动的时候再读取存储发送错误信息。
