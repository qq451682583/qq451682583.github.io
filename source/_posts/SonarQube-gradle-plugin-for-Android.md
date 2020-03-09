---
title: SonarQube gradle plugin for Android
comments: true
date: 2020-01-16 15:32:57
categories: Plugin
tags: Gradle
description: 
---

[TOC]
<!--more-->

> [User documentation](https://github.com/SonarSource/sonar-scanner-gradle)

# 前言

本文包含了官方SonarQube Plugin介绍、自定义适合多组件plugin、官方SonarQube Plugin实现多组件等内容，你可以直接使用tech-sonar plugin来实现多组件工程sonar统计。其中，tech-sonar插件是基于[官方SonarQube plugin](https://github.com/SonarSource/sonar-scanner-gradle)二次开发的。

# 1、SonarQube Plugin如何工作

当将插件应用于项目时，他将向该项目添加sonarqube task，它还将添加到项目及其所有子项目SonarQube extension。对于多模块项目，该插件仅适用于调用它的第一个项目。建议：在`subprojects {}`中添加sonar配置对所有子module生效！

* SonarQube Extension

    SonarQube扩展可以使用DSL轻松配置项目，你可以在DSL里设置自定义属性，详细使用属性请看[这里](https://docs.sonarqube.org/latest/analysis/analysis-parameters/)。

* SonarQube Task 

    SonarQube task的名称为`sonarqube`，因此可以通过调用`./gradlew sonarqube`来执行。他从项目及其所有子项目中收集信息，生成分析属性。然后，它使用所有这些属性运行SonarQube分析。task依赖于于所有项目的compile和test task（跳过的项目除外）。如果跳过所有项目（通过向sonarqube DSL添加skipProject = true），则不会执行分析。
    
# 2、tech-sonar plugin

对于目前多组件结构的Android工程，官方SonarQube Plugin插件只能实现项目整体统计，不能单独生成某个模块的报告。因此，在官方plugin基础上开发了适合多组件的tech-sonar 插件。

## 使用

### 1. 添加依赖和配置

根project build.gradle 文件

```java
    dependencies {
        classpath 'xxxx'
    }
    
    sonarConfig {
        onlyFullReport = findProperty('sonarOnlyFull') ?: false 
        hostUrl = 'xxxx'
        login = 'xxx'
        projectKey = 'xxx'
        projectName = 'xxxx'
        projectVersion = VERSION_NAME
        ignoredProjects = ['xxxx', 'xxxx']
        enableTestTask = false
    }
```
字段说明：

* onlyFullReport：一个标记，用来指示tech-sonar plugin内部来执行task使用
* hostUrl：服务端url
* login：登录签名
* projectKey：总工程key，子模块的key默认为 「rootProjectName:moduleName」
* projectName：总工程name，子模块的name默认为 「rootProjectName:moduleName」
* projectVersion：项目版本号
* ignoredProjects：要忽略的模块
* enableTestTask：是否要执行测试任务

### 2. 执行

* ./gradlew sonarqube -PsonarOnlyFull=true 生成项目sonar总报告，但不包含所有module sonar总报告
* ./gradlew sonarFullReport 生成所有module的sonar报告，但不包含项目总报告
* ./gradlew :moduleName:sonarqube 生成单个module的sonar报告


### 3. Jenkins配置


项目-> Configure -> Build -> 执行 shell 添加如下command即可

```java
./gradlew clean & ./gradlew sonarqube -PsonarOnlyFull=true & ./gradlew sonarFullReport
```

---

# 3、官方SonarQube实现多模块方案

*提示：这部分内容只是为了展示官方插件实现多组件工程统计，实现起来比较丑陋，如果不感兴趣，可以忽略这部分！*

## 3.1、单个module

```java
// 当要生成2.3项目总sonar结果，需要去掉这个依赖plugin，在根module的build.gradle中添加一次即可！
apply plugin: 'org.sonarqube'

// List of modules that skip by sonarqube plugin
def ignoredBySonar = [
        'app'
]

sonarqube {
    properties {
        property "sonar.host.url", "xxx"
        property "sonar.login", "xxx"
        property "sonar.language", "java"
        // 工程名：根工程:module名称
        property "sonar.projectName", "${rootProject.name}:${project.name}"
        property "sonar.projectKey", "${rootProject.name}:${project.name}"
        // 项目版本号
        property "sonar.projectVersion", '1.0.0'

        // Defines where the java files are
        property "sonar.sourceEncoding", "UTF-8"
        //property "sonar.sources", "src/main/java"
        properties["sonar.sources"] = android.sourceSets.main.java.srcDirs

        // Analyze tests classes
        property "sonar.exclusions", "src/test/java, src/androidTest/java"
        property "sonar.java.test.binaries", "build/intermediates/classes/debug"
    }

    skipProject = ignoredBySonar.contains(project.name)
}

// disable all test task for sub project
project.afterEvaluate {
    project.tasks.all { task ->
        def ignoreTask = task.name.contains("test") || task.name.contains("Test")
        if (ignoreTask) {
            //println "disable test task  : + ${task.name}"
            task.setEnabled false
        }
    }
}

task sonarExec(dependsOn: ['sonarqube']) {

}

gradle.ext.ignoredBySonar = ignoredBySonar
```

以上配置可以单独写到一个xx.gradle文件（如android-sonarqube.gradle），并依赖到root project build.gradle的subprojects中。即可生成单个module的sonar结果，如：`./gradlew :moduleOne:sonarqube`。

```java
subprojects {
    apply from: rootProject.getRootDir().getAbsolutePath() + "/scripts/sonar/android-sonarqube.gradle"
}
```

## 3.2、一次所有单个module

```java
apply plugin: 'org.sonarqube'

task sonarFullReport() {
    group = 'Reporting'
    description = 'Generates an aggregate report from all subprojects'

    // Get list of projects which should be included in the report
    def projects = []
    subprojects.each { prj ->
        if (!gradle.ignoredBySonar.contains(prj.name)) {
            projects.add(prj)
        }
    }

    //noinspection GrUnresolvedAccess
    dependsOn(projects.sonarExec)
}
```

以上配置可以单独写到一个xx.gradle文件（如android-sonarqube-full.gradle），并依赖到root project build.gradle中。即可一次生成所有单个module的sonar结果，等同于分开执行`./gradlew moduleOne:sonarqube`，`./gradlew moduleTwo:sonarqube`...

```java
apply from: 'scripts/sonar/android-sonarqube-full.gradle'
```

## 3.3、汇总

Generates an aggregate report from all subprojects

```java
apply plugin: 'org.sonarqube'
subprojects {
    // 配置所有子module sonar DSL
    apply from: rootProject.getRootDir().getAbsolutePath() + "/scripts/sonar/android-sonarqube.gradle"
}

/**
 * Generates an aggregate report from all subprojects.
 */
sonarqube {
    properties {
        property "sonar.host.url", "xxxx"
        property "sonar.login", "xxx"
        property "sonar.language", "java"
        property "sonar.projectName", "xxx"
        property "sonar.projectKey", "xxx"
        property "sonar.projectVersion", VERSION_NAME

        property "sonar.sourceEncoding", "UTF-8"
        property "sonar.sources", "src/main/java"
        property "sonar.exclusions", "src/test/java, src/androidTest/java"
        property "sonar.java.test.binaries", "build/intermediates/classes/debug"
    }
}
```

生成项目sonar总报告，需要为所有module和根module配置sonar DSL，在根module的build.gradle配置，执行`./gradlew sonarqube`即可生成当前项目sonar总报告。

# 4、问题

* Q：`Cannot add extension with name 'sonarqube', as there is an extension already registered with that name.`

> 由于sonarqube plugin插件只能在工程中apply一次，所以上面抽取的2个文件android-sonarqube.gradl和android-sonarqube-full.gradle都关联了插件，会报错。目前还不能同时要统计2.2、2.3，这个问题目前还在研究中。。。也可以在Stack Overflow上关注这个 [问题](https://stackoverflow.com/questions/48866378/make-sonarqube-gradle-plugin-available-in-root-project-and-subprojects)！
「后续」：这个问题已经通过tech-sonar plugin解决！！！


# 5、参考

- [sample-android-sonarqube](https://github.com/sogilis/sonarqube-for-android-example/blob/master/android-sonarqube.gradle)
- [SonarSource/sonarqube](https://github.com/SonarSource/sonarqube)
- [SonarSource/sonar-scanner-gradle](https://github.com/SonarSource/sonar-scanner-gradle)
- [User documentation](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Gradle)
- [SonarQubeExtension-analysis properties](https://docs.sonarqube.org/latest/analysis/analysis-parameters/)
- [weikipeng/Android-SonarQube-MultiProject](https://github.com/weikipeng/Android-SonarQube-MultiProject/blob/master/build.gradle)
- [sonar.gradle](https://github.com/hallatech/gradle-sonar-multi-atg-project/blob/master/sonar.gradle)
- [android-app/build.gradle](https://github.com/sogilis/sonarqube-for-android-example/blob/master/android-app/build.gradle)
- [sonar.gradle](https://github.com/hallatech/gradle-sonar-multi-atg-project/blob/master/sonar.gradle)
- [Medium/Android quality with SonarQube](https://medium.com/@nielsz/android-quality-with-sonarqube-bf907e614aed)
- [hallatech/gradle-sonar-multi-atg-project](https://github.com/hallatech/gradle-sonar-multi-atg-project/blob/master/README.md)

# 6、联系我

 * Email: hy04150829@gmail.com







