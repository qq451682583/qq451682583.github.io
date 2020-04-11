---
title: 断点调试 APT 和 Gradle Plugin
comments: true
date: 2020-04-03 22:19:51
categories: Gradle
tags: gradle
description:
---
[TOC]
<!--more-->

## 前言

本文会讲解如何在 IDE（AS、Intellij）下 debug 断点调试注解处理器 APT(Annotation Processing Tool) 和 gradle plugin，帮你摆脱只能打印日志调试的窘境。

## 调试步骤

1. 设置 Remote configuration

* 打开 Edit Configurations，

![toolbar-edit configurations](https://image1.guazistatic.com/qn1911282107012828065451f541cd5512d0dfd9bf8bf8.jpg)

* 点击左上角 + 号，创建一个 Remote configuration，上面名字随便写，我写了 debug，port 端口设置为 5005，点击 ok 即可。

<img desc="设置界面" src="https://image1.guazistatic.com/qn191128210752b85aa9396306c8838e02e502e92c7fbe.jpg" width="80%"/>

2. 在 IDE Terminal 下，输入如下命令：

```
./gradlew clean :moduleName:assembleDebug -Dorg.gradle.debug=true --no-daemon
```

之后这个进程就会一直等待，知道我们 attach 我们的调试进程进来。执行命令后如图：

<img desc="执行等待命令" src="https://image1.guazistatic.com/qn1911282059391cf902285a7c7a8fc3ef15159358051c.jpg" width="80%"/>

3. 下断点

这一步需要你去具体代码里打上断点，比如：android apt 去具体 abstractProcessor 处理器类打断点。

3. 启动远程调试器 

* 点击 debug 按钮连接上远程调试器进行调试

![debug attach](https://image1.guazistatic.com/qn191128212058c8e88a495726ac17909467fb58da15b9.jpg)

* 可以看到 connected to target VM 等信息输出就表示已连接到了远程服务器，之后就是正常的调试了

<img desc="已连接到了远程服务器" src="https://image1.guazistatic.com/qn19112821222494af55a04963cca7187c8c9e5c659103.jpg" width="80%"/>

* 连接上远程服务器之后，可以发现我们刚刚运行的命令已经开始执行了，这时候静代码执行到我们断点处

<img desc="远程代码执行" src="https://image1.guazistatic.com/qn191128212554777807d61da6abaa9a2a0ede3c077daf.jpg" width="80%"/>

最后附上一张调试的图片：

<img desc="断点调试情景图" src="https://image1.guazistatic.com/qn1911282129395953b4742da1182a129b4e2f00cdb5aa.jpg" width="80%"/>

## 遇到的错误

* Android APT 死活不能自动生成？

由于 Android Gradle 构建版本问题引起，之前设置的是 gradle 版本 5.1.1 + android gradle 3.4.1，修改成 4.4 + 3.1.4 解决。建议 3.3.2 + 4.10.1 以下都可以。具体可以参考[这篇博客](https://blog.csdn.net/allenli0413/article/details/90602402)。

## 总结

调试步骤其实很简单，分如下两步即可：

* 新建 remote target
* 在命令行输入执行 `./gradlew --no-daemon -Dorg.gradle.debug=true :moduleName:assembleDebug`
* 之后选择刚刚创建的 remote target，然后点击调试按钮即可

## 参考

* [调试Annotation Processor编译时注解器](https://www.jianshu.com/p/cc369dca20d1)
* [又掌握了一项新技能 - 断点调试 Gradle 插件](https://fucknmb.com/2017/07/05/%E5%8F%88%E6%8E%8C%E6%8F%A1%E4%BA%86%E4%B8%80%E9%A1%B9%E6%96%B0%E6%8A%80%E8%83%BD-%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95Gradle%E6%8F%92%E4%BB%B6/)
* [gradle调试断点](https://fsilence.github.io/2017/04/11/gradle-debug/)
* [Android APT不能自动生成文件](https://blog.csdn.net/allenli0413/article/details/90602402)