---
title: Android 多渠道打包实践
comments: true
date: 2020-04-03 22:55:48
categories: Android
tags: Android
description:
---
[TOC]
<!--more-->

## 背景

向日葵 APP 主版本已经越来越庞大，包含了很多业务线功能，每次发版内容复杂，在一些特殊场景下，拉取单独地分支开发，会额外增加维护成本，影响正常的工程，代码正确性不敢保证。一个鲜明的例子是：我们需要向外提供一个轻量级登录模块功能，也就是 Android AAR 代码包，如果单独创建一个 library 开发会特别麻烦，维护库的压力会很大。

在这样的前提下，我们引入 Android 工程共主线差异化打包，共主线的核心其实是利用 productFlavors block 语法，差异化开发，共用主线，多渠道打包。

一句话总结：productFlavors 可以差异化开发，共用主线，多渠道打包

## 差异化构建

在 Library 库的 build.gradle 文件里配置不同的 productFlavors 实现同一个库下差异化版本构建，实现两个不同的产品。例如，向日库登录模块实现2个迭代版本，分别为 full 和 lite，其中 full 为全量代码，lite 为轻量版代码向外提供 aar，配置代码如下：

```groovy
android {
    publishNonDefault true
    flavorDimensions "dimension"
    productFlavors {
        full {
            dimension "dimension"
        }

        lite {
            dimension "dimension"
        }
    }
}
```
## 工程结构

gradle 中有一个 source set 概念，不同产品可以设置不同的 source set，通常 src/main 目录是 IDE 自动帮我们创建的文件夹,因此我们可以在 src 目录下创建 full/lite 这样的目录,目录名需要和 productFlavors 中定义的产品名对应 。
在 src/main 目录下新建两个同级目录 full 与 lite，目录的名称要与 productFlavors 中设定的工程名保持一致。如下所示：

![biz-shell模块多产品构建](https://image1.guazistatic.com/qn191031162602f5518be1df7755c7b227b6089274af5a.jpg)

这样 src/full/java 文件内可以放不同的代码，src/full/res 文件夹内可以放不同的资源文件，同时也可以定义不同的 AndroidManifest 文件，比如申请不同的权限之类。其中，对于 full 和 lite 来说，main 目录下的文件和资源是共享的，而 full 与 lite 下的文件和资源是对应的 product 特有的，在这里可以做一些差异化的编码，达到共主线，差异化逻辑的效果。

## 多渠道打包

assemble 是和 Build Variants 一起结合使用的，而 Build Variants = Build Type(Debug/Release) + Product Flavor，其中 assembleDebug 可以简写为 aDebug 使用，为什么是aDebug，a就是assemble，至于assemble就是gradle的强大之处了。

assemble 命令

```shell
./gradlew assembleDebug 
编译并生成 Debug 包，包含 productFlavors 下所有定义的产品或渠道包
./gradlew assembleRelease 
编译并生成 Release 包,包含 productFlavors 下所有定义的产品或渠道包
./gradlew assembleProductARelease 
编译并生成 Release 包,包含 ProductA 下所有定义的产品或渠道包
./gradlew assembleProductADebug
 编译并生成 Debug 包,包含 ProductA 下所有定义的产品或渠道包
```

## 参考

* [使用gradle的productFlavors实现Android项目多渠道打包](https://zhuanlan.zhihu.com/p/33722674)
* [Gradle中productFlavors使用详解](https://juejin.im/entry/5a586bfaf265da3e2c3808c5)