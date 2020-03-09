---
title: FrieBase调研
comments: true
date: 2020-02-12 12:19:22
categories:
tags:
description:
---

[TOC]
<!--more-->

**Firebase简介**

- [将 Firebase 添加到您的 Android 项目](https://firebase.google.cn/docs/android/setup)
- [FireBase Packges](https://firebase.google.cn/docs/reference/android/packages) 

Firebase是谷歌旗下的一个强大的工具，使用Firebase需要翻墙。Firebase提供了以下几大功能：

**app埋点：Analytics**
```
应用内数据上报，帮助分析用户在app内的行为
```
**云消息推送：Firebase Cloud Message**
```
即：FCM，帮助app推送通知
```
**身份验证：Authentication**
```
方便的实现google登录，facebook登录，twitter登录，github登录，邮箱登录，电话登录以及自定义验证登录
```
**实时数据库：Database和最新的Firestore**
```
无需搭建服务器就能拥有一个实时的数据库，可以用来保存自己想要保存的任何数据。
```
**云仓库：Cloud Storage**
```
无需搭建服务器就能拥有一个云仓库，可以用来保存文件，如图片、音频、视频。不过免费版最多保存1个G的文件。
```
**app崩溃报告：Firebase Crashlytics**
```
自动记录应用内崩溃信息，只需简单的几步，就可以将Firebase Crashlytics添加到安卓工程中，然后Firebase Crashlytics就会自动的收集应用内崩溃信息，包括错误类型，代码定位等等，非常的方便实用
```
**Firebase远程配置：Remote Config**
```
相当于在服务器上设置几个key-value字段，我们在应用内可以请求这几个字段，通过value值设置我们的app。

比如：将app页面的背景色放在远程配置中，启动app时拉取远程配置，根据远程配置中的value值设置页面背景色。这样就实现了动态配置app的背景色。
读者可能会疑惑：使用实时数据库是不是也一样能实现这个功能？只要在数据库里设置几个用于app配置的字段就可以了。或者使用云仓库是不是也能实现这个功能？保存一个用于app配置的文件，每次打开app拉取此文件，然后根据读取的内容动态配置app就可以了?

是的，这两种方法一样可以实现远程配置。只是用Firebase远程配置实现的话，对app的性能影响最低，实现起来也更优雅。
```

**A/B测试**
```
通过Firebase远程配置的A/B测试，帮助了解哪种配置用户更喜欢。

比如：如果你想了解用户更喜欢红色风格的充值页面还是绿色风格的充值页面，那么你就可以使用A/B测试，在远程配置中设置A方案：红色和B方案：绿色，并设置A方案和B方案的比例，比如各占50%。这样用户拉取配置的时候，50%的用户会拉取到红色，50%的用户会拉取到绿色。再配合在用户充值时埋点，分析出红色页面和绿色页面的充值比例。这样你就可以选出更好的方案，获得更多的收益。
```
**动态链接：Dynamic Link**
```
生成一个根据不同的场景响应不同行为的链接。

比如：你想要为app添加一个房间内邀请好友的功能，如果好友也安装了此app，点击分享链接就进入此房间，如果好友没有安装此app，那就跳转到Google Play下载页面（或者你自定义的任何页面），如果好友没有安装此app，而且他是苹果手机，那就跳转到苹果商店的应用下载页面。这个功能就可以使用Firebase的动态链接实现。
```

**邀请：Firebase invites**
```
邀请好友，基于Firebase动态链接。使用邀请功能让用户邀请好友下载或打开app更加的方便。
```
**AdWords**
```Java
帮助投放app，就是给钱让谷歌给你打广告。主要有以下几种渠道：

1. 买关键词

平时我们用搜索引擎的时候，搜索的结果中一般都有几条广告。这就是广告主买关键词的作用。广告主买一些关键词，当用户用谷歌搜索这些关键词的时候，就展示你的app下载链接。

买个Google Play关键词，用户一搜Google Play就搜到你的应用了，是不是很舒服。当然，没有这么简单，你想得到别人也想得到。买Google Play关键词的人非常多，Google的策略大致就是价高者得。所以越火的关键词就会越贵，想要赚钱就需要权衡广告投入成本和应用的收益。

2. 买应用内广告或网页广告

访问一些app或者网站时，边边角角会有一些广告弹出，这也是广告主买的。
```
**AdMob**
```
在自己的app里面打广告，赚取收益。和AdWords对应着看，举个例子：你用AdWords买一个广告位，展示一千次自己app的广告给了谷歌1美元，再使用AdMob展示别人的广告赚取收益，展示了一千次赚取了0.1美元。

以上只是Firebase的大部分功能，Firebase还有机器学习套件，网页托管等等功能。
```

## **接入Firebase Crashlytics**

[官方Demo](https://github.com/firebase/quickstart-android)

[官方文档](https://firebase.google.com/docs/crashlytics/?hl=zh-CN)

[Firebase 控制台](https://console.firebase.google.com/project/_/crashlytics)

#### 一、配置Android应用并下载google-service.json文件



```
在Firebase控制台新建项目（需要使用Google账户登录），配置好Android应用后下载google-service.json文件，将google-service.json文件添加到项目的app目录下即可。
```
- 添加项目
- 添加Android应用
- 下载google-service.json文件加入项目

#### 二、集成Crashlytics SDK

- 项目级 build.gradle：添加classpath和maven

```Gradle
buildscript {
    repositories {
        ...
        maven { url 'https://maven.google.com'  }
        maven { url 'https://maven.fabric.io/public' }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.3'
        classpath 'com.google.gms:google-services:4.1.0'
        classpath 'io.fabric.tools:gradle:1.25.1'
    }
}

allprojects {
    repositories {
        ...
        maven { url 'https://maven.google.com' }
    }
}

...
```
- 应用级 build.gradle(<项目>/<应用模块>/build.gradle)，添加implementation和apply plugin

```Gradle
...
//Twitter将Fabric卖给Google了
apply plugin: 'io.fabric'
android{
    ...
}
dependencies {
    ...
    implementation 'com.google.firebase:firebase-core:16.0.3'
    implementation 'com.crashlytics.sdk.android:crashlytics:2.9.5'
}
apply plugin: 'com.google.gms.google-services'
```

做完这一步后，就可以启动app并制造一个crash，到Firebase 控制台中的 Crashlytics页面查看是否有数据上报即可。Firebase的数据上报实时性很高，尤其是新建的项目，数据量很少，出现crash后几秒钟就能在Firebase Crashlytics的平台上看到了。

建议在debug模式下添加 ext.alwaysUpdateBuildId = false标志来阻止 Crashlytics 不断更新其构建 ID，优化日常开发的编译速度。

```Gradle
android {
  ...
  buildTypes {
    debug {
      ext.alwaysUpdateBuildId = false
    }
}
```

- （可选） Crashlytics SDK NDK Crash监控

```Gradle
Firebase Crashlytics的官方文档只列出来了Java代码的Crash监控使用方式，并没有提及NDK Crash的监控。
对于大部分Android开发者来说，NDK的Crash也确实没有监控的必要，但是考拉这边用了很多第三方so库，这些so库也是有必要监控起来的。
Firebase Crashlytics是由Firebase收购Fabric而来的项目，技术方案也几乎没有变化，所以可以用Fabric监控NDK Crash的使用方式使用Firebase Crashlytics。
Fabric NDK Crash Reporting

在应用级 build.gradle 中，添加 Crashlytics NDK Crash Reporting 相关配置

apply plugin: 'com.android.application'
apply plugin: 'io.fabric'

// set NDK Crash Reporting enable
crashlytics {
    enableNdk true
}

dependencies {
    // ...
    implementation 'com.google.firebase:firebase-core:16.0.6'

    implementation 'com.crashlytics.sdk.android:crashlytics:2.9.8'
    // Add ndk dependency
    implementation 'com.crashlytics.sdk.android:crashlytics-ndk:2.0.5'
}
```

在manifest中声明Crashlytics手动初始化

```XML
<!--手动初始化firebase Crashlytics sdk-->
<meta-data
    android:name="firebase_crashl
        ytics_collection_enabled"
    android:value="false"/>

```
根据自身app启动流程，选择合适时机手动初始化Crashlytics
```Java
private void configureCrashReporting() {
        CrashlyticsCore crashlyticsCore = new CrashlyticsCore.Builder()
                .disabled(BuildConfig.DEBUG)
                .build();
        Fabric.with(this, new Crashlytics.Builder().core(crashlyticsCore).build());
    }
```
如果是本地源码编译的so而不是直接使用第三方提供的so，可以生成并上传符号表来辅助分析crash信息，执行 ./gradlew crashlyticsUploadSymbolsRelease即可上传符号表。

#### 三、错误数据上报分析


![错误面板](https://img-blog.csdn.net/20180906100828240?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FscGluaXN0V2FuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

通过以上步骤的配置，现在Firebase Crashlytics已经可以正常工作了。并且还附带了发生Crash的时间，机型，系统版本，应用版本等信息以供分析。

![](https://img-blog.csdnimg.cn/20190127174334220.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbmdjaGVueHVhbmZlbmc=,size_16,color_FFFFFF,t_70)


除了这些基础信息，我们可以自定义添加更多数据,比如:

```Java
//用户id
Crashlytics.setUserIdentifier(builder.getUserId());
//用户渠道
Crashlytics.setString("Channel", builder.getChannelInfo());
//用户补丁版本
Crashlytics.setString("Version", builder.getUserTag());
//用户当前WebView UA
Crashlytics.setString("WebView UA", WebViewSettings.getUserAgent());
//打包机名称
Crashlytics.setString("BuildHost", BuildInfo.BUILD_HOST);
//打包时间
Crashlytics.setString("BuildTime", BuildInfo.BUILD_TIME);
//最新提交commit id
Crashlytics.setString("GitLog", BuildInfo.BUILD_GIT_LOG);
```
这些自定义的数据可以在，Firebase Crashlytics的“键”选项卡下面看到。
![](https://img-blog.csdnimg.cn/20190127175450347.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbmdjaGVueHVhbmZlbmc=,size_16,color_FFFFFF,t_70)

通过[setUserIdentifier]()所设置的用户Id还可以用于搜索。

![](https://img-blog.csdnimg.cn/20190127175929869.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbmdjaGVueHVhbmZlbmc=,size_16,color_FFFFFF,t_70)

还可以通过[Crashlytics.log]()方法，打入自定义日志用于分析。

![](https://img-blog.csdnimg.cn/20190127180206168.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbmdjaGVueHVhbmZlbmc=,size_16,color_FFFFFF,t_70)

开发过程中，还会遇到通过[try catch]()捕获了异常，不造成崩溃，但是又希望能够统计上报该异常的情况。这个时候，可以使用[Crashlytics.logException(throwable)]()方法将异常统计上来。
在Firebase的过滤条件中选择，不严重的事件类型，即可过滤该异常。

![](https://img-blog.csdnimg.cn/20190127180623407.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpbmdjaGVueHVhbmZlbmc=,size_16,color_FFFFFF,t_70)

如果之前的项目使用的是Fabric Crashlytics，那你可以直接使用[Fabric迁移流程](https://fabric.io/firebase_migration)来快速迁移到Firebase Crashlytics。


## **Firebase比较有用的地方**

- 支持对每个用户设置唯一标识，我们可以用服务传递回来的用户id 或者 手机号码，然后再排查问题的时候通过这个唯一标识去定位，很方便。

    [Crashlytics.setUserIdentifier("user123456789");]()

- 添加自定义日志消息

    [Crashlytics.log(Log.DEBUG, "customCrashLog" , e.toString());]()
    
    第一个参数int 类型，可以填info，debug，error ；我们可以通过这些 类型，区分日志的重要程度；第二个参数为 自定义 key。

    要为导致崩溃的事件提供更多背景信息，您可以向您的应用添加自定义 Crashlytics 日志。Crashlytics 会将日志与您的崩溃数据相关联，并在 [Firebase 控制台](https://console.firebase.google.com/project/_/crashlytics)中显示这些日志。

    在 Android 上，请使用 Crashlytics.log 来帮助查明问题。Crashlytics.log 既可以将日志写入崩溃报告并执行 Log.println()，也可以仅继续写入下一个崩溃报告：

    - 崩溃报告和 Log.println：Crashlytics.log(Log.DEBUG, "tag", "message");
    - 仅崩溃报告：Crashlytics.log("message");

- 记录非严重异常

    除了自动报告您的应用中出现的崩溃，Crashlytics 还可让您记录非严重异常。

    在 Android 上，这意味着您可以在应用的 catch 块中记录已捕获到的异常：

    ```Java
    try {
        methodThatThrows();
    } catch (Exception e) {
        Crashlytics.logException(e);
        // handle your exception here
    }
    ```

    所有记录的异常在 Firebase 控制台中均显示为非严重问题。问题摘要中会包含您通常从崩溃中可以获得的所有状态信息，以及按 Android 版本和硬件设备细分的数据。

    Crashlytics 在一个专用的后台线程中处理异常，所以对您的应用性能的影响极小。为了减少用户的网络流量，Crashlytics 会将已记录的异常汇集到一起，并在下次应用启动时批量发送这些异常。