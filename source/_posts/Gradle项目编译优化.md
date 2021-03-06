---
title: Gradle项目编译优化
comments: true
date: 2020-04-03 22:25:29
categories: Gradle
tags: gradle
description:
---
[TOC]
<!--more-->

# 一、当 Gradle 遇上我们的项目时

那些我们遇到的问题们：

* 解释下什么是 Gradle ?
* Gradle 原理是是样子的？
* 那些错误该怎么处理？
* 怎么解决编译慢的问题？

# 二、基本内容

## 2.1 基本知识

1. 这三个是不一样滴

gradle & gradlew & android plugin gradlew 

```
# Makefile - C/C++ 组织动态链接库和文件
# Java 简单点
javac HelloWorld.java
java HelloWorld
```

* Eclipse-Ant 编译，没准备让人看懂
* Maven-XML，编译 XML 大几千行，像 Spring 项目，基本没法编辑
* Gradle-Groovy 像我们 build.gradle 文件，很多 DSL，符合程序员的风格

2. Gradle 讲解

* configuration -> Task module A -> compile A -> Jar 

<img desc="gradle-执行流程" src="http://wanghaoxun.com/img/Gradle 执行流程.jpg" width="60%"/>

* Task 执行

Input -> Task Java Compile -> .class 文件

TODO2：gradle-task执行.jpg

* AGP-Android Gradle Plugin

```
classpath 'com.android.tools.build:gradle:3.5.0'
```
AGP 代码也是个 jar包，用了就会生成很多 Task

TODO3：gradle-agp-souce.jpg

Android 和 Java 不一样，它有 Resource 文件，要生成 resource id-R.id 文件，还有 Manifest，AIDL 等等，AGP 添加了很多 Task，最后生成 module aar 包。AGP 就是帮你在 Android 中把包打出来。

TODO4：gradle-agp-task.jpg

3. gradle 和 gradlew 不是一个东西！！！

位置不一样，gradlew gradle wrapper 不是一个东西

TODO5：gradle-gradlew.jpg

4. 在 Android Studio 里点 RUN AS 将注入 init-script，运行结果和 shell 里会有不同混合执行会很慢！！！

我们可以在命令行或者 AS 里点击运行工程，不建议同时使用。点击 RUN AS 会通过 init-script 往里面注入很多复杂的 debug，性能相关，打广告等等。所以，在 AS 中可能快可能慢。

5. sdk list gradle 

最新 gradle 5.6.1，我们现在用的是 3.4.1。gradle 每个版本性能最高提高 10%，可能吗？？？

## 2.2 项目现状

1. why so so so slowwwwww?

编译失败了，clean 完了，在运行失败了，已经运行了 28 分钟左右。

* 模块太多
* 资源太多
* 什么都多！！！
* APT 太慢
* AOP 太慢

* 2557 tasks executed in 89 projects in 9m 56.538s

项目代码 13多万行，编译模块几十个，APT（ButterKnife，EventBus，Room等等），AOP 也有几个。

2. Official solution 

官方更新了这么多版本，0.x.x -> 5.6.x 版本，应该提升很多吧，实际呢？呵呵，下面列举一些有用的：

1. Parallel 并行
2. Compiler daemon 不要每次创建新进程，使用后台进程
3. Big RAM(memory)!!! 更大的内存
4. Build Cache 增量编译 cache
5. incremental build 编译走增量
6. Compile avoidance 很多编译不用走全量
7. ABI 

* Parallel-Gradle Task 并行执行

格式化输出 task 运行过程，可以看到有 8 条线也就是 8 个线程同时运行。电脑 CPU 有 4核，超线程到 8 核，gradle 把所有 CPU 沾满之后，就会变得很卡。自己可以配置调节少些，电脑舒服点。

TODO5：gradle-task-parallel.jpg

* Incremental Build 增量编译

TODO2：gradle-task执行.jpg

只编译 changed 的文件，增量编译。

TODO6：gradle-task-incremental-build.jpg

* ABI 级别-implement 

3.x 版本之后升级成了 implement 来隔离模块，compile 时，修改底层模块，会造成依赖链上的模块都编译，十几分钟又过去了。。。compile 需要 9m，implement 只需要 5m。

TODO7：gradle-abi.jpg

**我们的增量编译 == 全量**

猜猜有多少生效？？？

* Parallel
* Compiler daemon

也就这两个可以留着！what？听我仔细道来：

甚至有时候编译时，会执行 GC。：）

# 三、Our Problem--rerun-tasks

**运行项目 20多分钟，最后失败了？一脸懵逼。。。**

## 3.1 gradle 的问题（APT、ABI、Version）

* 如果想知道 compile 执行过程，可以执行如下 command:

```
# 会把 gradle 执行结果上传到服务器，帮助我们更加精细化的分析编译中遇到的问题
./gradlew run --scan
```

TODO8：gradle-task-run-scan.jpg

* 改一行底层 core 模块，一个八竿子打不着的模块产生了编译？

### 3.1.1 ABI

正常 ABI 编译过程

TODO9：gradle-abi-normal.jpg 

失效 ABI

Core -> module A -> module B

只改了 core，造成了 module A 中使用 APT 组件（EventBus等）产生了编译，再造成了 module B 产生了编译。

TODO10：gradle-abi-invalid.jpg 

### 3.1.2 APT

* APT 产生了问题

Java 语法很少，为什么从 96年活到现在，就是因为可以动态生成代码，扩充很多功能。

看到下面这种图，你可能会想到，如果我们 APT 生成的文件每次都不一样，那么整个编译就毁了。

TODO11：gradle-task-apt.jpg 

**防止 APT 生成随机文件**

TODO12：gradle-apt-fix.jpg 

自己写的 APT 组件可以修复下，EventBus 就没法修复了。解决：简单粗暴点，在 debug 环境下把 EventBus 关闭了，哈哈。

* ABI 好了，增量编译又失效了？

apt 修复好了，但是和 core 模块相关的，增量编译时会产生一次全量编译。

TODO13：gradle-apt-processor-path.jpg

要使用 apt，需要告诉 gradle apt 文件在哪里？扫描 apt，在扫描文件看有没有变化。

```
javaCompileOptions {
    annotationProcessorOptions {
        // compile 项目模块的源文件
        includeCompileClasspath = true
    }
}

```

由于 includeCompileClasspath 为 true，当你修改模块一行代码，会造成 apt 重新全量编译，整个模块全量编译，天呐！

TODO14：gradle-apt-processor-path-fix.jpg

* APT 增量编译

gradle 5.x 之后 APT 支持增量编译，官方文档：https://blog.gradle.org/incremental-compiler-avoidance

TODO15：gradle-apt-increment.jpg

* RenderScript Src 设置错误导致 core 重复编译

```
renderscript.srcDirs = ['src']
```

renderscript 编译造成了 resource 重新编译，引起了 R 文件重新编译，于是我们项目又开始了重新编译，很可怕！可以算工伤。

### 3.1.3 AGP & Gradle version

* 升级 AGP 与 Gradle 

Gradle 和 AGP 不一样，Gradle 支持增量编译，AGP 不一定支持增量编译，只能同时升级。

TODO15：gradle-vs-agp.jpg

**以上 gradle 修改后，该一行 core 代码，编译时间从 9:00 -> 5:00 **

## 3.2 AGP-Transform 优化

优化完 gradle，在看来 AGP，Google 坑挺多，如图，dexBuider & dexMerger 2个任务会卡很久

TODO16：gradle-apg-transform.jpg

### 3.2.1 Transform 流程

TODO17: gradle-agp-transform-flow.jpg

之前加一点功能，需要 hook gradle task，加一些任务进去，hook debugCompileClass,aptClass 等，特别恶心，版本兼容性也不好。所以开放了 transform API 来解决，

* Android Transform API

看到 transform API，甚至怀疑都是照着 task API 抄的！

TODO18: gradle-apg-transform-api.jpg

* DexArchive & DexBuilder & DexMerger

TODO19: gradle-apg-transform-dex.jpg

分为 DirInput 和 JarInput 2 种情况，顶层项目 CRMApp （apply andorid plugin）生成 dir，其它所有 module 都会变成 jar 包，传进去 transform 里。

```
* DirInput
* Jar -> 解压 -> class -> D8/DX(脱糖) -> class -> 合并 -> dex

java 1.8 ，Android 真正可以执行 1.6，需要进行处理。
```

如果在 module 中改了一行代码，造成了 class 修改，transform 只知道 jar 包变了，不知道哪个 class 变了，只能解压，编译以下，合并到一个 dex 中，就这么可怕！

### 3.2.2 Transform 增量优化

* Transform-Incremental-Build

1. DexBuilder

TODO20: gradle-transform-incremental-build.jpg

Jar To Dir: 先把 Jar 解压了，变成 Dir，在塞进 Transform。

问题：怎么知道哪个 class 增量编译下？

只能解压出来，计算下 diff，看哪些变化了，哪些被删除了。我们需要算每个文件的 md5 值，要哪个是加的哪个是减的哪个是删的，很慢！

所有的 task 利用线程池并行执行，比如解压计算 CRC32-类似 MD5，好多都是同步执行，这里利用协程并行解压 Diff

```
所有 Module 全量解压 20s -> 2s
Core 增量解压 2s -> 600ms
```

结果：DexBuilder: 30s -> 5s

2. DexMerger

暂时没有好的办法 :(

```
1. API 21 以上，每个 JarInput 作为一个dex，每个Dir 作为一个Dex
2. 21、22 有 100 个 dex 的限制
3. 设置 minSdk 23 能加快 10s
```

5.0,6.0 有个 bug，如果 dex 超过 100个的话，会把所有 dex 合起来再打个大的 dex，打到一起很慢~

可以升级 minSdk 23 来解决，但是升级后，打出来的 dex 数量超多，2000 多个，安装很慢~


## 3.3 Avoid Compile

APT 加速后，9m -> 5m -> 3:30m 还是太慢

* 思考一个问题：Core 模块修改了，关 module A 啥事？

更多是一层校验，其实我们写代码都很规范了，修改 Core 时都会 find usage 以下，所以，基本不会出现找不到函数的情况。

TODO21: gradle-abi-aggressive.jpg

那么依赖 core 的就先别编译了，激进一点的做法，hook 所有task 做处理，同样算下所有 Input 文件的 Diff，当发现没有变化时， 就直接跳过去！目前谨慎的跳过了 compile 编译 task，annotation 这个毒瘤，

TODO22: gradle-abi-avoid-compile.jpg

* AnnotationProcessor 分离

TODO23：gradle-apt-seperate.jpg

目前关联 Core 的模块不用编译了，但是 Core 里面使用了 APT 还是很慢。

假如 Core 模块是纯 Java 代码，不是 KAPT，APT 和 Compile 都是由 JavacCompile Task 执行的。

```
// Google 提供的 API，但有个风险，会造成不拷贝资源文件
android.enableSeparateAnnotationProcessing=true
```

经常造成 Singleton.getxxx，找不到资源的问题。

修复分离后没有 copy res

```
## 3.4 我们还做了很多

1. 增强 InvokerPlugin，修复 Class 重复或 Jar 包找不到

解决 duplicate class 问题，虽然你可以快速编译，但是来个这个幺蛾子错误，又得重新编译，10m 又过去了。省的时间又还给你们了。

2. 兼容处理 AS 和 Shell混编，不需要频繁 clean
3. AnnotationWrap 分离，减少 APT 扫描耗时
4. 等等等
```

## 3.4 我们还能做什么

1. AS 3.5 apply change & InstantRun
2. 类似 Freeline class 和 res 增量
3. Core 分离

* AS 3.5 删除了 InstantRun（插件化都在超 InstantRun 代码），变成了 apply change
* Freeline 虽然可以增量，但是是个危险的操作，Android 每个版本都在改，gradle 也再改，AGP 也再改，各个手机厂商也再改，所以 BUG 特别多，所以你会发现市面上所有的编译插件项目都已经不再维护了。Android 新出的 jetpack 都不支持。

* TODO24: gradle-apt-seperate.jpg
* Core 中心节点依赖分离


### 3.4.1 开发一套支持动态配置快速编译方案

**如何开启**

cp local_config.json.sample local_config.json

json 配置文件

TODO25: gradle-polish-setting.jpg

# 四、还有很长的路要走。。。

。。。