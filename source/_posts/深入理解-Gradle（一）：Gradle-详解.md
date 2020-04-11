---
title: 深入理解 Gradle（一）：Gradle 详解
comments: true
date: 2020-04-03 23:01:10
categories:
tags:
description:
---
[TOC]
<!--more-->

# 一、前言

## 1.1 什么是构建？

构建，叫 build 也好，叫 make 也行，根绝外部输入的信息然后干一堆事情，最后得到几个产出物（Artifact）。

Gradle 是当前非常「劲爆」的构建工具，介绍 Gradle 之前，先说说一些之前使用过的构建工具们。在 Gradle 爆红之前，常用的构建工具是 Ant，然后进化到 Maven。Ant 和 Maven 这两个工具在出来时也算方便，但是二者都有些缺点。比如，编译规则是用 XML 来编写的，XML 虽然通俗易懂，但是很难在其中描述 if/else 这样有不同条件的任务。

## 1.2 构建历程

* Ant 

<img desc="gradle_ant_eclipse" src="http://wanghaoxun.com/img/gradle_ant_eclipse.jpg" width="60%">

* Maven

<img desc="gradle_maven_retrofit" src="http://wanghaoxun.com/img/gradle_maven_retrofit.jpg" width="60%">

你会发现 Ant 和 Maven 配置复杂，依赖混乱，几十人的团队协作开发编辑不敢想象，甚至都怀疑这玩意是给人用的？（黑人脸，what？？？）

![Nick Young](http://wanghaoxun.com/img/black_face.jpeg)

**怎么办？怎么办？自然是编程来解决。。。**

* Gradle 

<img desc="gradle_xrk_crm" src="http://wanghaoxun.com/img/gradle_xrk_crm.jpg" width="60%">

Gradle 作为一种很方便的构建工具，可以轻松解决多版本配置、构建依赖等问题，最主要它支持用 Grooovy、Java、Kotlin 来编写，可以书写自己的变量、函数。在使用 Gradle 之前，我们可能还有几个小要求：

1. 这种编程不要搞得太复杂，几行代码就轻松把要做的事情描述出来就最好不过了。所以 Gradle 选择了 Groovy，Groovy 基于 Java 并拓展了 Java，Groovy 说白了就是把写 Java 程序变得像写脚本一样简单。写完就可以执行，Groovy 内部会将其编译成 Java class 然后启动虚拟机来执行。
2. 除了可以用灵活的语言来写构建规则外，Gradle 另外一个特点就是它是一种 DSL，即 Domain Specific Language，领域相关语言。什么是 DSL？说白了它是某个行业中的行话。类似徐克导演的《智取威虎山》中就很有典型的 DSL 使用描述，例如：

> 土匪：天王盖地虎！（你好大的胆！敢来气你的祖宗？）    
杨子荣：宝塔镇河妖！（要是那样，叫我从山上摔死，掉河里淹死。）    
土匪：野鸡闷头钻，哪能上天王山！（你不是正牌的。）    
杨子荣：地上有的是米，喂呀，有根底！（老子是正牌的，老牌的。）

Gradle 中也有类似的行话，比如 sourceSet 代表源文件的集合等... 太多了，记不住？没关系，后面我们会通过手册来慢慢了解一些常用的。

**一句行话可以包含很多意思，在这个行当里的人一听就懂，不用解释。另外，基于外行，我们甚至可以建立一个模板，使用者只要往这个模板里填必须的内容，Gradle 就可以非常漂亮的完成工作，得到我们想要的东西。**

## 1.3 Gradle 小总结

至此，大家应该明白要真正学会 Gradle 怕是离不开下面两个基础知识：

1. Groovy，基于 Java，了解 Groovy 语言是掌握 Gradle 的基础
2. Gradle 作为一个工具，它的行话和它的「为人处事」的原则

# 二、Groovy 介绍

Groovy 是一种动态语言，它和 Java 一样，也运行在 Java 虚拟机中。恩？？对，你没听错，你可以认为 Groovy 拓展了 Java 语言，比如，Groovy 对自己的定义就是：Groovy 是在 Java 平台上，具有像 Python，Ruby 和 Smalltalk 语言特性的灵活动态语言，Groovy 保证了这些特性像 Java 语法一样被 Java 开发者使用。

除了语言和 Java 相通外，Groovy 有时候又像一种脚本语言，当你执行 Groovy 脚本时，Groovy 会先将其编译成 Java 类字节码，然后通过 JVM 来执行这个 Java 类，图示展示了 Java、Groovy和 JVM 之间的关系。

![Groovy 和 Java 关系图](http://wanghaoxun.com/img/Groovy关系图.png)

**实际上，在 Groovy Code 在真正执行的时候已经变成了 Java 字节码，所以 JVM 根本不知道自己运行的是 Groovy 代码。**

下面将集中讲解 Groovy 和 Gradle 打交道时一些常用的知识点。

## 2.1 Groovy 开发环境

根据 [Groovy 官网](http://www.groovy-lang.org/download.html#gvm) 介绍部署 Groovy 开发环境。

macOS 安装

```sh
brew install groovy
```

Groovy 也有一套 [GDK](http://www.groovy-lang.org/api.html)

<img desc="Groovy-GDK" src="http://wanghaoxun.com/img/Groovy-GDK.jpg" width="60%">

[Gradle 源代码地址](https://github.com/gradle/gradle)

## 2.2 一些前提知识

* Groovy 注释标记和 Java 一样，支持 // 或者 /**/
* Groovy 语句可以不用分号结尾
* Groovy 中支持动态类型，即定义变量的时候可以不指定其类型。Groovy 中，变量定义可以使用关键字 def。注意，虽然 def 不是必须的，但是为了代码清晰，建议还是使用 def 关键字

```groovy
   def variable1 = 1   // 可以不使用分号结尾 
   def varable2 = "I am a person"
   def int x = 1   // 变量定义时，也可以直接指定类型
```

* 函数定义时，参数的类型也可以不指定

```groovy
    String testFunction(arg1,arg2){// 无需指定参数类型 
      ...
    }
```

* 除了变量定义可以不指定类型外，Groovy 中函数的返回值也可以是无类型的

```groovy
    // 无类型的函数定义，必须使用 def 关键字
    
    def nonReturnTypeFunc(){
        last_line   // 最后一行代码的执行结果就是本函数的返回值 
    }
     
    // 如果指定了函数返回类型，则可不必加 def 关键字来定义函数 
    String getString(){
       return "I am a string"
    }
```

其实，所谓的无返回类型的函数，我估计内部都是按返回 Object 类型来处理的。毕竟，Groovy 是基于 Java 的，而且最终会转成 Java Code 运行在 JVM 上

* 函数返回值：Groovy 的函数里，可以不使用 return xxx 来设置 xxx 为函数返回值。如果不使用 return 语句的话，则函数里最后一句代码的执行结果被设置成返回值。

```groovy
    // 下面这个函数的返回值是字符串"getSomething return value"
    def getSomething() {
     
        "getSomething return value" // 如果这是最后一行代码，则返回类型为 String
     
        1000 // 如果这是最后一行代码，则返回类型为 Integer
    }
```

注意，如果函数定义时候指明了返回值类型的话，函数中则必须返回正确的数据类型，否则运行时报错。如果使用了动态类型的话，你就可以返回任何类型了。

* Groovy 对字符串支持相当强大，充分吸收了一些脚本语言的优点：

1. 单引号''中的内容严格对应 Java 中的 String，不对 $ 符号进行转义

```groovy
   def singleQuote='I am $ dolloar'  // 输出就是 I am $ dolloar
```
2. 双引号""的内容则和脚本语言的处理有点像，如果字符中有 $ 号的话，则它会$ 表达式先求值

```groovy
   def doubleQuoteWithoutDollar = "I am one dollar" // 输出 I am one dollar
   def x = 1
   def doubleQuoteWithDollar = "I am $x dolloar" // 输出 I am 1 dolloar
```

3. 三个引号'''xxx'''中的字符串支持随意换行 比如

```groovy
   def multieLines = ''' begin
     line  1 
     line  2
     end '''
```

* 最后，除了每行代码不用加分号外，Groovy 中函数调用的时候还可以不加括号。

```groovy
    println("test") ---> println "test"
```

**注意，虽然写代码的时候，对于函数调用可以不带括号，但是 Groovy 经常把属性和函数调用混淆。比如**

```groovy
    def getSomething(){
       "hello"
    }
```

getSomething()   // 如果不加括号的话，Groovy 会误认为 getSomething 是一个变量。

所以，调用函数要不要带括号，我个人意见是如果这个函数是 Groovy API 或者 Gradle API 中比较常用的，比如 println，就可以不带括号。否则还是带括号。Groovy 自己也没有太好的办法解决这个问题，只能兵来将挡水来土掩了。

## 2.3 Groovy 中的数据类型

Groovy 中的数据类型我们就介绍三种和 Java 不太一样的：

* 一个是 Java 中的基本数据类型
* 另外一个是 Groovy 中的容器类
* 最后一个非常重要的是闭包

### 2.3.1 基本数据类型

作为动态语言，Groovy 世界中的所有事物都是对象。所以，int，boolean 这些 Java 中的基本数据类型，在 Groovy 代码中其实对应的是它们的包装数据类型。比如 int 对应为 Integer，boolean 对应为 Boolean。比如下图中的代码执行结果：

<img desc="基本数据类型" src="http://wanghaoxun.com/img/groovy-basic-data-type.jpg" width="60%">

### 2.3.2 容器类

Groovy 中的容器类很简单，就三种：

* List：链表，其底层对应 Java 中的 List 接口，一般用 ArrayList 作为真正的实现类
* Map：键 - 值表，其底层对应 Java 中的 LinkedHashMap
* Range：范围，它其实是 List 的一种拓展

了解容器简单例子：

1. List 类

```groovy
    变量定义：List 变量由 [] 定义，比如 
    
    def aList = [5,'string',true] //List 由 [] 定义，其元素可以是任何对象 
    
    变量存取：可以直接通过索引存取，而且不用担心索引越界。如果索引超过当前链表长度，List 会自动 
    往该索引添加元素 
    
    assert aList[1] == 'string'
    assert aList[5] == null // 第 6 个元素为空 
    aList[100] = 100  // 设置第 101 个元素的值为 100
    assert aList[100] == 100
    
    那么，aList 到现在为止有多少个元素呢？
    
    println aList.size  ===> 结果是 101
```
2. Map 类

```groovy
    容器变量定义 
    
    变量定义：Map 变量由 [:] 定义，比如 
    
    def aMap = ['key1':'value1','key2':true] 
    
    Map 由 [:] 定义，注意其中的冒号。冒号左边是 key，右边是 Value。key 必须是字符串，value 可以是任何对象。另外，key 可以用''或""包起来，也可以不用引号包起来。比如 
    
    def aNewMap = [key1:"value",key2:true] // 其中的 key1 和 key2 默认被 
    处理成字符串"key1"和"key2"
    
    不过 Key 要是不使用引号包起来的话，也会带来一定混淆，比如 
    
    def key1 = "wowo"
    def aConfusedMap = [key1:"who am i?"]
    
    aConfuseMap 中的 key1 到底是"key1"还是变量 key1 的值“wowo”？显然，答案是字符串"key1"。如果要是"wowo"的话，则 aConfusedMap 的定义必须设置成：
    
    def aConfusedMap = [(key1):"who am i?"]
    
    Map 中元素的存取更加方便，它支持多种方法：
    
    println aMap.keyName    <== 这种表达方法好像 key 就是 aMap 的一个成员变量一样 
    println aMap['keyName'] <== 这种表达方法更传统一点 
    aMap.anotherkey = "i am map"  <== 为 map 添加新元素
```
3. Range 类

Range 是 Groovy 对 List 的一种扩展，变量定义和大体的使用方法如下：

```groovy
    def aRange = 1..5  <==Range 类型的变量 由 begin 值 + 两个点 +end 值表示 
                          左边这个 aRange 包含 1,2,3,4,5 这 5 个值 
    
    如果不想包含最后一个元素，则 
    
    def aRangeWithoutEnd = 1..<5  <== 包含 1,2,3,4 这 4 个元素 
    println aRange.from
    println aRange.to
```

### 2.3.3 Groovy API 的一些秘笈

了解 Groovy 的语法，是离不开 SDK 的，Groovy 是动态语言，所以要使用它的 SDK 也需要掌握一些小诀窍。

Groovy 的 API 文档地址：http://www.groovy-lang.org/api.html

以上面介绍的 Range 为例，我们该如何查阅 SDK 更好的使用它呢？

* 先找到 Range 类，它位于 [groovy.lang](http://www.groovy-lang.org/api.html) 包中：

有了 API 文档，你可以使用里面的函数了，不过，细心地你可以发现我们刚才代码中用到的 Range.from/to 属性值，在 Range API 文档中并没有这两个成员变量。

<img desc="gradle-groovy-sdk-range" src="http://wanghaoxun.com/img/gradle-groovy-sdk-range.jpg" width="60%">

文档中没有说明 Range 有 from 和 to 这两个属性，但是却有 getFrom 和 getTo 两个函数，What？？？原来：

根据 Groovy 的原则，如果一个类中有名为 xxyyzz 这样的属性（其实就是成员变量），Groovy 会自动为它添加 getXX 和 setXX 两个函数，用于获取和设置 xxyyzz 属性值。

**注意：get 和 set 后第一个字母是大写的！**

所以，当你看到 Range 中有 getFrom 和 getTo 这两个函数时候，就得知道潜规则下，Range 有 from 和 to 这两个属性。当然，由于它们不可以被外界设置，所以没有公开 setFrom 和 setTo 函数。

## 2.4 闭包

### 2.4.1 闭包的样子

闭包，英文叫 Closure，是 Groovy 中非常重要的一个数据类型或者说一种概念了。闭包的历史来源，种种好处我就不说了。我们直接看怎么使用它！

**闭包，是一种数据类型，它代表了一段可执行的代码。其外形如下：**

```
    def aClosure = {// 闭包是一段代码，所以需要用花括号括起来..  
        Stringparam1, int param2 ->  // 这个箭头很关键。箭头前面是参数定义，箭头后面是代码  
        println "this is code" // 这是代码，最后一句是返回值，  
       // 也可以使用 return，和 Groovy 中普通函数一样  
    }  
```

简而言之，Closure 的定义格式是：

```
    def xxx = {paramters -> code}  // 或者  
    def xxx = {无参数，纯 code}  这种 case 不需要 -> 符号
```

补充一点，有没有想到我分享 Java8 时提到的 Lambda 表达式呢？哈哈。。。

**从 C/C++ 语言的角度看，闭包和函数指针很像**。闭包定义好，要调用它的方法就是：

闭包对象.call(参数)  或者更像函数指针调用的方法：

闭包对象 (参数)

比如：

```
    aClosure.call("this is string",100)  或者  
    aClosure("this is string", 100)  
```

如果闭包没定义参数的话，则隐含有一个参数，这个参数名字叫 it，和 this 的作用类似。it 代表闭包的参数。

比如：

```
    def greeting = { "Hello, $it!" }
    assert greeting('Nelson') == 'Hello, Nelson!'
```

等同于：

```
    def greeting = { it -> "Hello, $it!" }
    assert greeting('Patrick') == 'Hello, Patrick!'
```


### 2.4.2 Closure 使用中的注意点

1. 省略圆括号

闭包在 Groovy 中大量使用，比如很多类都定义了一些函数，这些函数最后一个参数都是一个闭包。比如：

```
    public static <T> List<T> each(List<T> self, Closure closure)
```

上面这个函数表示针对 List 的每一个元素都会调用 closure 做一些处理。这里的 closure，就有点回调函数的感觉。但是，在使用这个 each 函数的时候，我们传递一个怎样的 Closure 进去呢？比如：

```
    def iamList = [1,2,3,4,5]  // 定义一个 List
    iamList.each {  // 调用它的 each，这段代码的格式看不懂了吧？each 是个函数，圆括号去哪了？
          println it
    }
```

上面代码，有 2个知识点：

* **each 函数调用的圆括号不见了！**原来，Groovy 中，当函数的最后一个参数是闭包的话，可以省略圆括号。比如

```
    def  testClosure(int a1,String b1, Closure closure) {
          //do something
          closure() // 调用闭包 
    }
    那么调用的时候，就可以免括号！
    testClosure (4, "test", {
       println "i am in closure"
    } )  // 外层的括号可以不写..
    // 简写
    testClosure 4,"test",{println "i am in closure"}
```

注意，这个特点非常重要，因为以后在 Gradle 中经常出现如下这样的代码：

<img desc="Gradle 简单闭包示例" src="http://wanghaoxun.com/img/gradle-groovy-closure-hello.png"/>

经常碰见图示这样的没有圆括号的代码。省略圆括号虽然使得代码简洁，看起来更像脚本语言，但是它这经常会让我疑惑，以 doLast 为例，完整的代码应该按下面这种写法：

```
    doLast({
       println 'Hello world!'
    })
```

有了圆括号，你会知道 doLast 只是把一个 Closure 对象传了进去。很明显，它不代表这段脚本解析到 doLast 的时候就会调用 println 'Hello world!' 。

但是把圆括号去掉后，就感觉好像 println 'Hello world!' 立即就会被调用一样！

2. 如何确定 Closure 的参数

另外一个比较让人头疼的地方是，closure 的参数该怎么搞？还是刚才的 each 函数：

```
    public static <T> List<T> each(List<T> self, Closure closure)
```

使用：

```
    def iamList = [1,2,3,4,5]  // 定义一个 List 变量 
    iamList.each{  // 调用它的 each 函数，只要传入一个 Closure 就可以了。
      println it
    }
```

* **对于 each 所需要的 Closure，它的参数是什么？有多少个参数？返回值是什么？**

Closure 虽然方便，但是它一定会和使用它的上下文有极强的关联。要不，作为类似回调这样的东西，该如何知道调用者传递什么参数给 Closure 呢？

此问题如何破解？只能通过查询 [API](http://docs.groovy-lang.org/latest/html/gapi/index.html?org/codehaus/groovy/runtime/DefaultGroovyMethods.html)  文档才能了解上下文语义。如下图：

从整体上知道 groovy SDK 对 JDK 做了扩展，这些都是 GDK 提供的新类，这些方法并没有加到 JDK 中对应的类中，那是如何直接调用它扩展的那些方法呢？例如，Class DefaultGroovyMethods 类中为我们任意对象都提供了一个 each 方法。

<img desc="each 方法" src="http://wanghaoxun.com/img/groovy-defaultmethods-each.jpg" width="60%"/>

<img desc="findAll 方法" src="http://wanghaoxun.com/img/groovy-defaultmethods-findall.jpg" width="60%"/>

* each 函数说明中，将给指定的 closure 传递 Set 中的每一个 item。所以，closure 的参数只有一个。
* findAll 中，绝对抓瞎了。一个是没说明往 Closure 里传什么。另外没说明 Closure 的返回值是什么.....

**对 Map 的 findAll 而言，Closure 可以有两个参数。findAll 会将 Key 和 Value 分别传进去。并且，Closure 返回 true，表示该元素是自己想要的。返回 false 表示该元素不是自己要找的。** 示意代码如下所示：

```
    def aMap = [k1:'value1',k2:true]
    def results = aMap.findAll {
        key, value -> 
        println "key=$key, value=$value"
        if(key == 'k1') 
            return true
        return false
    }
```

> Closure 的使用有点坑，很大程度上依赖你对 API 的熟悉程度，so，最初阶段，SDK 查询是少不了的！

## 2.5 脚本类、文件 I/O 和 XML 操作

最后，来看一下 Groovy 中比较高级的用法。

### 2.5.1 脚本类

1. 脚本中 import 其他类

Groovy 中可以像 Java 那样写 package，然后写类。比如在文件夹 com/nelson/groovy/chap2 目录中放一个文件，叫 Article.groovy，如代码所示（演示-chap2 代码）：

创建了 Article.groovy 文件，ArticleTest.groovy 再 import 了 Article 类，然后创建了 Article 类型的对象，接着调用它的 print 函数。

在 groovy 中，系统自带会加载当前目录 / 子目录下的 xxx.groovy 文件。所以，当执行 groovy ArticleTest.groovy 的时候，ArticleTest.groovy import 的 Article 类能被自动搜索并加载到。

2. 脚本到底是什么

Java 中，我们最熟悉的是类。但是我们在 Java 的一个源码文件中，不能不写 class（interface 或者其他....），而 Groovy 可以像写脚本一样，把要做的事情都写在 xxx.groovy 中，而且可以通过 groovy xxx.groovy 直接执行这个脚本。这到底是怎么搞的？

既然是基于 Java 的，Groovy 会先把 xxx.groovy 中的内容转换成一个 Java 类。比如：chap2/Inner.groovy 文件

**执行 groovyc -d classes Inner.groovy**，groovyc 是 groovy 的编译命令， -d classes 用于将编译得到的 class 文件拷贝到 classes 文件夹下，Inner.groovy 脚本转换得到的 java class，可以用 jd-gui 反编译它的代码，也可以用 IEA 直接查看。

```
import groovy.lang.Binding;
import groovy.lang.Script;
import org.codehaus.groovy.runtime.InvokerHelper;
import org.codehaus.groovy.runtime.callsite.CallSite;

public class Inner extends Script {
    public Inner() {
        CallSite[] var1 = $getCallSiteArray();
    }

    public Inner(Binding context) {
        CallSite[] var2 = $getCallSiteArray();
        super(context);
    }

    public static void main(String... args) {
        CallSite[] var1 = $getCallSiteArray();
        var1[0].call(InvokerHelper.class, Inner.class, args);
    }

    public Object run() {
        CallSite[] var1 = $getCallSiteArray();
        return var1[1].callCurrent(this, "Hello Groovy!");
    }
}
```

* Inner.groovy 被转换成了一个 Script 类，它从 script 派生。
* 每一个脚本都会生成一个 static main 函数。这样，当我们 groovy Inner.groovy 的时候，其实就是用 java 去执行这个 main 函数
* 脚本中的所有代码都会放到 run 函数中。比如，println 'Hello Groovy'，这句代码实际上是包含在 run 函数里的。
* 如果脚本中定义了函数，则函数会被定义在 Inner 类中

3. 脚本中的变量和作用域

上面说了脚本的代码其实都会被放到 run 函数中去执行，那么在 Groovy 的脚本中，很重要的一点就是脚本中定义的变量和它的作用域。举例：

Field.groovy 文件

```
/**
 * 2.5.1 脚本中的变量和作用域
 */
def x = 1

def printx() {
    println x
}

printx()
```

**运行脚本会报错，groovy.lang.MissingPropertyException: No such property: x for class: Field**

反编译后：

```
public class Field extends Script {
    public Field() {
        CallSite[] var1 = $getCallSiteArray();
    }

    public Field(Binding context) {
        CallSite[] var2 = $getCallSiteArray();
        super(context);
    }

    public static void main(String... args) {
        CallSite[] var1 = $getCallSiteArray();
        var1[0].call(InvokerHelper.class, Field.class, args);
    }

    public Object run() {
        CallSite[] var1 = $getCallSiteArray();
        Object x = 1;
        return x;
    }

    public Object printx() {
        CallSite[] var1 = $getCallSiteArray();
        return var1[1].callCurrent(this, var1[2].callGroovyObjectGetProperty(this));
    }
}
```

* printx 被定义成 Field 类的成员函数
* def x = 1，这句话是在 run 中创建的。所以，x = 1 从代码上看好像是在整个脚本中定义的，但实际上 printx 访问不了它。printx 是 test 成员函数，除非 x 也被定义成 test 的成员函数，否则 printx 不能访问它。

那么，如何使得 printx 能访问 x 呢？很简单，定义的时候不要加类型和 def。演示 chap2/Field2.groovy 源文件和反编译文件

x 也没有被定义成 Field2 的成员函数，而是在 run 的执行过程中，将 x 作为一个属性添加到 Field2 **实例对象**中了。然后在 printx 中，先获取这个属性。

**注意，Groovy 文档说 x = 1 这种定义将使得 x 变成 Filed2 的成员变量，但从反编译情况来看，这是不对的。。。**

虽然 printx 可以访问 x 变量了，但是假如有其他脚本却无法访问 x 变量。因为它不是 Field2 的成员变量。

```
public class Field3 extends Script {
    Object x;

    public Field3() {
       ...
    }

    public static void main(String... args) {
        CallSite[] var1 = $getCallSiteArray();
        var1[0].call(InvokerHelper.class, Field3.class, args);
    }

    public Object run() {
        CallSite[] var1 = $getCallSiteArray();
        Object var10000 = null;
        return !__$stMC && !BytecodeInterface8.disabledStandardMetaClass() ? this.printx() : var1[1].callCurrent(this);
    }

    public Object printx() {
        CallSite[] var1 = $getCallSiteArray();
        return var1[2].callCurrent(this, this.x);
    }
}
```

可以看到 Field3.groovy 中的 x 已经变成了 Field3 类的成员属性了。如此，我们可以在 script 中定义那些需要输出给外部脚本或类使用的变量了！

> 这里演示 Field2.groovy，Field3.groovy，InvokeField.groovy 文件来说明成员属性！

### 2.5.2 文件 I/O

本节介绍 Groovy 的文件 I/O 操作。虽然比 Java 看起来简单，但要理解起来其实比较难。尤其是当你要自己查 SDK 并编写代码的时候。

整体说来，Groovy 的 I/O 操作是在原有 Java I/O 操作上进行了更为简单方便的封装，并且使用 Closure 来简化代码编写。主要封装了如下一些类：

* java.io.File class: http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/File.html
* java.io.InputStream class: http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/InputStream.html
* java.io.OutputStream class: http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/OutputStream.html
* java.io.Reader class: http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/Reader.html
* java.io.Writer class: http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/Writer.html
* java.nio.file.Path class: http://docs.groovy-lang.org/latest/html/groovy-jdk/java/nio/file/Path.html

1. 读文件

Groovy 中，文件读操作简单到令人发指：

def targetFile = new File(文件名)  <==File 对象还是要创建的

看看 Groovy 定义的 API：

* 读该文件中的每一行：eachLine 的唯一参数是一个 Closure。Closure 的参数是文件每一行的内容

内部实现肯定是 Groovy 打开这个文件，然后读取文件的一行，然后调用 Closure...

演示 File.groovy 文件

* 直接得到文件内容

```
targetFile.getBytes() // 文件内容一次性读出，返回类型为 byte[]
```

注意前面提到的 getter 和 setter 函数，这里可以直接使用 targetFile.bytes //...

* 使用 InputStream 

```
def ism = targetFile.newInputStream()

ism.close()
```

* 使用闭包操作 inputStream，以后会在 Gradle 里经常看到这种写法。。

```
targetFile.withInputStream { ism ->
    // 操作 ism，不用 close，groovy 会自动替你 close
}
```

确实简单到发指，一开始也死活不理解 withInputStream 是个啥意思。所以，请再看的各位老铁牢记 Groovy I/O 操作相关类的 SDK 地址~

2. 写文件

和读文件差不多，不再啰嗦，这里举个例子，告诉你如何 copy 文件，演示 Copy.groovy 文件：

```
def srcFile = new File('src/com/nelson/groovy/chap2/source.txt')
def targetFile = new File('src/com/nelson/groovy/chap2/dest.txt')
targetFile.withOutputStream { os ->
    srcFile.withInputStream { ins ->
        // 利用 OutputStream 的 << 操作符重载，完成从 Inputstream 到 OutputStream 的输出
        os << ins
    }
}
```

关于 OutputStream 的 << 操作符重载，查看 SDK 文档之后才知道：

http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/OutputStream.html

再一次向极致简单致敬。以后，怕是 SDK 离不开手了，各位。。。

### 2.5.3 XML 操作

除了 I/O 异常简单之外，Groovy 中的 XML 操作也极致得很。Groovy 中，XML 的解析提供了和 XPath 类似的方法，名为 GPath。这是一个类，提供相应 API。关于 XPath，请看 [WIKI](https://en.wikipedia.org/wiki/XPath)。

演示 Xml.groovy 文件

详细操作就不在这展开了，用到了各位在查 SDK 完成吧~

### 2.6 更多

作为一门语言，Groovy 是复杂的，是需要深入学习和钻研的。一本厚书都无法描述 Groovy 的方方面面。

从使用的角度看，尤其是又限定在 Gradle 这个领域内，能用到的 Groovy 中一些简单的知识。

# 三、Gradle 介绍

<img src="http://wanghaoxun.com/img/gradlephant@2x.png" align="right" width="280"/>

现在正式进入 Gradle。Gradle 是一个工具，同时它也是一个编程框架。前面也提到过，使用这个工具可以完成 app 的编译打包等工作。当然你也可以用它干其他的事情。

Gradle 是什么？学习它到什么地步就可以了？

看待问题的时候，所站的角度非常重要！

-> 当你把 Gradle 当工具看的时候，我们只想着如何用好它。会写、写好配置脚本就 OK
-> 当你把它当做编程框架看的时候，你可能需要学习很多更深入的内容。

今天我们把它当工具看，明天因为需求发生变化，我们可能又得把它当编程框架看。

## 3.1 Gradle 开发环境部署

Gradle 官网：http://gradle.org/，最新 Gradle 6.0.1

文档：https://docs.gradle.org/current/release-notes.html，其中的 User Guide 和 DSL Reference 很关键，User Guide 就是介绍 Gradle 的一本书，而 DSL Reference 是 Gradle API 的说明。

MacOs 安装：

* 下载 Gradle：http://gradle.org/gradle-download/  选择Complete distribution和Binary only distribution都行。然后解压到指定目录
* 设置 ~/.bashrc 把 Gradle 加到 PATH 里
* 执行 source ~/.bashrc 初始化环境
* 执行 gradle --version 显示 Gradle 版本号

注意，为什么说 Gradle 是一个编程框架？来看它提供的 API 文档：

https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html

原来，我们编写所谓的编译脚本，其实就是玩 Gradle 的 API.... 所以它从更底层意义上看，是一个编程框架！

既然是编程框架，我在讲解 Gradle 的时候，尽量会从 API 的角度来介绍。有些读者肯定会不耐烦，为嘛这么费事？

**从我个人的经历来看：因为我从网上学习到的资料来看，几乎全是从脚本的角度来介绍 Gradle，结果学习一通下来，只记住参数怎么配置，却不知道它们都是函数调用，都是严格对应相关 API 的。**

而从 API 角度来看待 Gradle 的话，有了 SDK 文档，你就可以编程。编程是靠记住一行行代码来实现的吗？不是，是在你掌握大体流程，然后根据 SDK + API 来完成的！

其实，Gradle 自己的 User Guide 也明确说了：

**Build scripts are code**

## 3.2 基本组件

Gradle 是一个框架，它定义一套自己的游戏规则。我们要玩转 Gradle，必须要遵守它设计的规则。下面我们来讲讲 Gradle 的基本组件：

Gradle 中，每一个待编译的工程都叫一个 Project。每一个 Project 在构建的时候都包含一系列的 Task。比如一个 Android APK 的编译可能包含：**Java 源码编译 Task、资源编译 Task、JNI 编译 Task、lint 检查 Task、打包生成 APK 的 Task、签名 Task 等。**

一个 Project 到底包含多少个 Task，其实是由编译脚本指定的插件决定。插件是什么呢？插件就是用来定义 Task，并具体执行这些 Task 的东西。

Gradle 是一个框架，作为框架，它负责定义流程和规则。而具体的编译工作则是通过插件的方式来完成的。比如**编译 Java 有 Java 插件，编译 Groovy 有 Groovy 插件，编译 Android APP 有 Android APP 插件，编译 Android Library 有 Android Library 插件**

到现在为止，你知道 Gradle 中每一个待编译的工程都是一个 Project，一个具体的编译过程是由一个一个的 Task 来定义和执行的。

### 3.2.1 一个重要的例子

演示向日葵项目

<img desc="向日葵工程目录结构" src="http://wanghaoxun.com/img/gradle-xrk-project.jpg" width="60%"/>

* BIZ(biz-task,biz-transfer)、BIZService(biz-core,biz-form,biz-lego) 是 Android Library。其中，BIZ 下面的 library 依赖 BIZService library
* CRMApp 和 MiniApp 是 Android APP。这些 App 和 SDK 有依赖关系。CRMApp 依赖所有的 BIZ library。

问题1：请回答问题，在上面这个例子中，有多少个 Project？

答案是：每一个 Library 和每一个 App 都是单独的 Project。根据 Gradle 的要求，每一个 Project 在其根目录下都需要有一个 build.gradle。build.gradle 文件就是该 Project 的编译脚本，类似于 Makefile。


问题2：这么多 project，我们要独立编译他们的话，得 cd 某个 project，然后执行 gradle xxx，这会很麻烦，这么多 project，每个都得重复执行。可不可以在工程下执行 gradle assemble 把这些 Project 的东西都编译出来呢？

答案自然是可以。在 Gradle 中，这叫 Multi-Projects Build。在项目下添加个 settings.gradle 文件，名字必须是 settings.gradle。它里边用来告诉 Gradle，这个 multiprojects 包含多少个子 Project。

```
include ModuleA,ModuleB...
```

### 3.2.2 Gradle 命令介绍

1. gradle projects 查看工程信息

`./gradlew projects` 可以查看 multi projects 到底包含多少个子 Project

2. gradle tasks 查看任务信息

`./gradlew project-path:tasks` 查看某个 Project 包含哪些 Task 信息。

Android Library 对应的插件定义了好多 Task。每种插件定义的 Task 都不尽相同，这就是所谓的 Domain Specific，需要我们对相关领域有比较多的了解。

3. gradle task-name 执行任务

上面列出了好多任务，这时候就可以通过 gradle 任务名来执行某个任务。这和 make xxx 很像。比如：

* gradle clean 是执行清理任务，和 make clean 类似。
* gradle properites 用来查看所有属性信息。

这里要强调一点：Task 和 Task 之间往往是有关系的，这就是所谓的依赖关系。比如，assemble task 就依赖其他 task 先执行，assemble 才能完成最终的输出。

大家先了解这么多，等后面介绍如何写 gradle 脚本的时候，这就是调用几个函数的事情，Nothing Special!

## 3.3 Gradle 工作流程

Gradle 的工作流程其实蛮简单，用一个图来表示：

![Gradle 工作流程](http://wanghaoxun.com/gradle%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.png)

Gradle 工作包含三个阶段：

* 首先是初始化阶段。对我们前面的 multi-project build 而言，就是执行 settings.gradle
* Initiliazation phase 的下一个阶段是 Configration 阶段
* Configration 阶段的目标是解析每个 project 中的 build.gradle。比如 multi-project build 例子中，解析每个子目录中的 build.gradle。在这两个阶段之间，我们可以加一些定制化的 Hook。这当然是通过 API 来添加的
* Configuration 阶段完了后，整个 build 的 project 以及内部的 Task 关系就确定了。恩？前面说过，一个 Project 包含很多 Task，每个 Task 之间有依赖关系。Configuration 会建立一个有向图来描述 Task 之间的依赖关系。所以，我们可以添加一个 HOOK，即当 Task 关系图建立好后，执行一些操作
* 最后一个阶段就是执行任务了。当然，任务执行完后，我们还可以加 Hook

关于 Gradle 的工作流程，你只要记住：

* Gradle 有一个初始化流程，这个时候 settings.gradle 会执行。
* 在配置阶段，每个 Project 都会被解析，其内部的任务也会被添加到一个有向图里，用于解决执行过程中的依赖关系。
* 然后才是执行阶段。你在 gradle xxx 中指定什么任务，gradle 就会将这个 xxx 任务链上的所有任务全部按依赖顺序执行一遍！

Gradle 对应项目执行流程图：

<img src="http://wanghaoxun.com/img/Gradle 执行流程.jpg" width="60%"/>

**接下来告诉你怎么写代码！！！**

## Gradle 编程模型及 API 实例详解

**https://docs.gradle.org/current/dsl/ 这个文档很重要，在强调一遍~**

Gradle 基于 Groovy，Groovy 又基于 Java。所以，Gradle 执行的时候和 Groovy 一样，会把脚本转换成 Java 对象。Gradle 主要有三种对象，这三种对象和三种不同的脚本文件对应，在 gradle 执行的时候，会将脚本转换成对应的对端：

* Gradle 对象：当我们执行 gradle xxx 或者什么的时候，gradle 会从默认的配置脚本中构造出一个 Gradle 对象。在整个执行过程中，只有这么一个对象。Gradle 对象的数据类型就是 Gradle。我们一般很少去定制这个默认的配置脚本。
* Project 对象：每一个 build.gradle 会转换成一个 Project 对象。
* Settings 对象：显然，每一个 settings.gradle 都会转换成一个 Settings 对象。

注意，对于其他 gradle 文件，除非定义了 class，否则会转换成一个实现了 Script 接口的对象。这一点和 Groovy 的脚本类相似。

当我们执行 gradle 的时候，gradle 首先是按顺序解析各个 gradle 文件。这里边就有所所谓的生命周期的问题，即先解析谁，后解析谁。图示是 Gradle 文档中对生命周期的介绍：结合上一节的内容，相信大家都能看明白了。现在只需要看红框里的内容：

![Gradle-Lifecycle](http://wanghaoxun.com/gradle-lifecycle.png)

文档地址：https://docs.gradle.org/current/dsl/org.gradle.api.Project.html

## 3.4.1 Gradle 对象

我们先来看 Gradle 对象，它有哪些属性呢？跳转到文档，看看：https://docs.gradle.org/current/dsl/org.gradle.api.invocation.Gradle.html

可以尝试在 Setting.gradle 和 CRMApp build.gradle 中打印查看 gradle 详细信息：

* 可发现 gradle 对象的 hashCode 一样
* HomeDir 是在哪个目录存储的 gradle 可执行程序 
* User Home Dir：是 gradle 自己设置的目录，里边存储了一些配置文件，以及编译过程中的缓存文件，生成的类文件，编译中依赖的插件等等。~/.gradle 目录

## 3.4.2 Project 对象

每一个 build.gradle 文件都会转换成一个 Project 对象。在 Gradle 术语中，Project 对象对应的是 Build Script。

Project 包含若干 Tasks。另外，由于 Project 对应具体的工程，所以需要为 Project 加载所需要的插件，比如为 Java 工程加载 Java 插件。其实，一个 Project 包含多少 Task 往往是插件决定的。

所以，在 Project 中，我们要：

* 加载插件
* 不同插件有不同的行话，即不同的配置。例如我们要在 Project 中配置好 code 目录，这样插件就知道从哪里读取源文件等
* 设置属性

1. 加载插件

Project 的 API 位于 https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html 加载插件是调用它的 apply 函数。apply 其实是 Project 实现的

PluginAware 接口定义的：

apply 的用法：

apply 是一个函数，此处调用的是一个 apply 重载函数。注意，Groovy 支持函数调用的时候通过 参数名 1: 参数值 1，参数名 2：参数值 2 的方式来传递参数

```
// Project implements PluginAware #apply(..) 函数生命
void apply(Map<String, ?> options)

// 使用
apply plugin: 'com.android.application'
```

问题：除了加载二进制的插件（上面的插件其实都是下载了对应的 jar 包，这也是通常意义上我们所理解的插件），还可以加载一个 gradle 文件。为什么要加载 gradle 文件呢？

其实这和代码的模块划分有关。一般而言，我会把一些通用的函数放到一个名叫 utils.gradle 文件里。然后在其他工程的 build.gradle 来加载这个 utils.gradle。这样，通过一些处理，我就可以调用 utils.gradle 中定义的函数了。

加载 utils.gradle 插件的代码如下：

```
apply from: rootProject.getRootDir().getAbsolutePath() + "/utils.gradle"  
```

utils.gradle 是我封装的一个 gradle 脚本，里边定义了一些方便函数，比如读取 AndroidManifest.xml 中的 versionName，或者是 copy jar 包/APK
包到指定的目录。

问题：那么，apply 最后一个函数到底支持哪些参数呢？

还得查阅文档来看说明，我这里不遗余力的列出 API 图片，就是希望大家在写脚本的时候，碰到不会的，一定要去查看 API 文档！这点很重要！！！

2. 设置属性

如果是单个脚本，则不需要考虑属性的跨脚本传播，但是 Gradle 往往包含不止一个 build.gradle 文件，比如我设置的 utils.gradle，settings.gradle。如何在多个脚本中设置属性呢？

Gradle 提供了一种名为 extra property 的方法。extra property 是额外属性的意思，在第一次定义该属性的时候需要通过 ext 前缀来标示它是一个额外的属性。定义好之后，后面的存取就不需要 ext 前缀了。ext 属性支持 Project 和 Gradle 对象。即 Project 和 Gradle 对象都可以设置 ext 属性。

举个栗子：

* 在 setting.gradle 文件为 Gradle 对象设置一些外置属性，

```
def initGradleEnvironment(){  
    // 属性值从 local.properites 中读取  
    Propertiesproperties = new Properties()  
    File propertyFile = new File(rootDir.getAbsolutePath() +"/local.properties")  
    properties.load(propertyFile.newDataInputStream())  
    //gradle 就是 gradle 对象。它默认是 Settings 和 Project 的成员变量。可直接获取  
   //ext 前缀，表明操作的是外置属性。api 是一个新的属性名。前面说过，只在  
   // 第一次定义或者设置它的时候需要 ext 前缀  
    gradle.ext.api = properties.getProperty('sdk.api')  
     
    println gradle.api  // 再次存取 api 的时候，就不需要 ext 前缀了  
    ......  
} 

// 调用初始化
initGradleEnvironment()
```

* 再强化栗子，utils.gradle 文件

```
// utils.gradle 中定义了一个获取 AndroidManifests.xmlversionName 的函数  
def  getVersionName(){  
   // 问题1：下面这行代码中的 project 是谁？  
   defxmlFile = project.file("AndroidManifest.xml")  
   defrootManifest = new XmlSlurper().parse(xmlFile)  
   returnrootManifest['@android:versionName']    
}  
// 现在，想把这个 API 输出到各个 Project。由于这个 utils.gradle 会被每一个 Project Apply，所以  
// 我可以把 getVersionName 定义成一个 closure，然后赋值到一个外部属性  

// 问题2：下面的 ext 是谁的 ext？  
ext{ // 此段花括号中代码是闭包  
    // 除了 ext.xxx=value 这种定义方法外，还可以使用 ext{}这种书写方法。  
    //ext{}不是 ext(Closure) 对应的函数调用。但是 ext{}中的{}确实是闭包。  
    getVersionName = this.&getVersionName  
 } 
```

先来理清楚一个问题，Project 和 utils.gradle 对于的 Script 的对象的关系是：

* 当一个 Project apply 一个 gradle 文件的时候，这个 gradle 文件会转换成一个 Script 对象。这个，相信大家都已经知道了。
* Script 中有一个 delegate 对象，这个 delegate 默认是加载（即调用 apply）它的 Project 对象。但是，在 apply 函数中，有一个 from 参数，还有一个 to 参数。通过 to 参数，你可以把 delegate 对象指定为别的东西。
* delegate 对象是什么意思？当你在 Script 中操作一些不是 Script 自己定义的变量，或者函数时候，gradle 会到 Script 的 delegate 对象去找，看看有没有定义这些变量或函数。

现在你知道问题 1,2 和答案了：

* 问题 1：project 就是加载 utils.gradle 的 project。由于 b_sales_android 有多个 project，所以 utils.gradle 会分别加载到 project 中。所以，getVersionName 才不用区分到底是哪个 project。反正一个 project 有一个 utils.gradle 对应的 Script。
* 问题 2：ext：自然就是 Project 对应的 ext 了。此处为 Project 添加了一些 closure。那么，在 Project 中就可以调用 getVersionName 函数了

一些思考：

* Java 和 Groovy 中：我们会把常用的函数放到一个辅助类和公共类中，然后在别的地方 import 并调用它们。
* 但是在 Gradle，更正规的方法是在 xxx.gradle 中定义插件。然后通过添加 Task 的方式来完成工作。gradle 的 user guide 有详细介绍如何实现自己的插件。

3. Task 介绍

Task 是 Gradle 中的一种数据类型，它代表了一些要执行或者要干的工作。不同的插件可以添加不同的 Task。每一个 Task 都需要和一个 Project 关联。

Task 的 API 文档位于 https://docs.gradle.org/current/dsl/org.gradle.api.Task.html 关于 Task，我这里简单介绍下 build.gradle 中怎么写它，以及 Task 中一些常见的类型。

```
//Task 是和 Project 关联的，所以，我们要利用 Project 的 task 函数来创建一个 Task  
task myTask  <==myTask 是新建 Task 的名字  
task myTask { configure closure }  
task myType << { task action } <== 注意，<< 符号是 doLast 的缩写  
task myTask(type: SomeType)  
task myTask(type: SomeType) { configure closure } 
```

上述代码中都用了 Project 的一个函数，名为 task，注意：

* 一个 Task 包含若干 Action。所以，Task 有 doFirst 和 doLast 两个函数，用于添加需要最先执行的 Action 和需要和需要最后执行的 Action。Action 就是一个闭包。
* Task 创建的时候可以指定 Type，通过type: 名字表达。这是什么意思呢？其实就是告诉 Gradle，这个新建的 Task 对象会从哪个基类 Task 派生。比如，Gradle 本身提供了一些通用的 Task，最常见的有 Copy 任务。Copy 是 Gradle 中的一个类。当我们：task myTask(type:Copy)的时候，创建的 Task 就是一个 Copy Task。
* 当我们使用 task myTask{ xxx}的时候。花括号是一个 closure。这会导致 gradle 在创建这个 Task 之后，返回给用户之前，会先执行 closure 的内容。
* 当我们使用 task myTask << {xxx}的时候，我们创建了一个 Task 对象，同时把 closure 做为一个 action 加到这个 Task 的 action 队列中，并且告诉它“最后才执行这个 closure”（注意，<< 符号是 doLast 的代表）。

**讲了这么多东西，都有点烦了。是的，Gradle 用一整本书来讲都嫌不够呢！**

## 3.4.3 b_sales_android 实例

现在正是开始通过例子来介绍怎么玩 gradle。这里要特别强调一点，根据 Gradle 的哲学。gradle 文件中包含一些所谓的 Script Block（姑且这么称它）。Script Block 作用是让我们来配置相关的信息。不同的 SB 有不同的需要配置的东西。这也是我最早说的行话。比如，源码对应的 SB，就需要我们配置源码在哪个文件夹里。关于 SB，我们后面将见识到！

b_sales_android 是一个 multi project。下面包含多个 Project。对于这种 Project，请大家回想下我们该创建哪些文件？

* settings.gradle 是必不可少的
* 根目录下的 build.gradle。这个我们没讲过，因为 b_sales_android 的根目录本身不包含代码，而是包含其他子 project。
* 每个 project 目录下包含对于的 build.gradle
* 另外，我把常用的函数封装到一个名为 utils.gradle 的脚本里了。

马上一个一个来看它们：

1. utils.gradle

用来放一些获取版本号、拷贝文件等公共函数

```
import groovy.util.XmlSlurper  // 解析 XML 时候要引入这个 groovy 的 package  
 
def getVersionName() {
   defxmlFile = project.file("AndroidManifest.xml")  
   defrootManifest = new XmlSlurper().parse(xmlFile)  
   returnrootManifest['@android:versionName']    
}  
   
// 对于 android library 编译，我会 disable 所有的 debug 编译任务  
def disableDebugBuild() {  
  //project.tasks 包含了所有的 tasks，下面的 findAll 是寻找那些名字中带 debug 的 Task。  
  // 返回值保存到 targetTasks 容器中  
  def targetTasks = project.tasks.findAll{task ->  
     task.name.contains("Debug")  
  }  
  // 对满足条件的 task，设置它为 disable。如此这般，这个 Task 就不会被执行  
 targetTasks.each{  
     println"disable debug task  :${it.name}"  
    it.setEnabled false  
  }  
}  
// 将函数设置为 extra 属性中去，这样，加载 utils.gradle 的 Project 就能调用此文件中定义的函数了  
ext{  
   getVersionName = this.&getVersionName 
   disableDebugBuild = this.&disableDebugBuild  
}  
```

2. setting.gradle

这个文件中我们该干什么？调用 include 把需要包含的子 Project 加进来。代码如下

```
// 添加子 Project 信息  
include ':CRMApp'
include ':biz-dev'
include ':biz-test'
...

/**
  这个函数的目的是 
  1  解析一个名为 local.properties 的文件，读取 AndroidSDK 和 NDK 的路径 
  2  获取最终产出物目录的路径。这样，编译完的 apk 或者 jar 包将拷贝到这个最终产出物目录中 
  3 获取 Android SDK 指定编译的版本 
*/  
def initXRKGradleEnvironment() {   
   println"initialize xrk Gradle Environment ....."  
   Properties properties = new Properties()  
   //local.properites 也放在 posdevice 目录下  
   File propertyFile = new File(rootDir.getAbsolutePath()+ "/local.properties")  
   properties.load(propertyFile.newDataInputStream())  
    /* 
      根据 Project、Gradle 生命周期的介绍，settings 对象的创建位于具体 Project 创建之前 
      而 Gradle 底对象已经创建好了。所以，我们把 local.properties 的信息读出来后，通过 
     extra 属性的方式设置到 gradle 对象中 
      而具体 Project 在执行的时候，就可以直接从 gradle 对象中得到这些属性了！ 
    */  
    gradle.ext.api =properties.getProperty('sdk.api')  
    gradle.ext.sdkDir =properties.getProperty('sdk.dir')  
     gradle.ext.ndkDir =properties.getProperty('ndk.dir')  
     gradle.ext.localDir =properties.getProperty('local.dir')  
    // 指定 debugkeystore 文件的位置，debug 版 apk 签名的时候会用到  
    gradle.ext.debugKeystore= properties.getProperty('debug.keystore')  
     ......  
    println"initialize xrk Gradle Environment completed..."  
}  
// 初始化  
initXRKGradleEnvironment()
```

注意，对于 Android 来说，local.properties 文件是必须的，它的内容如下：

```
// 注意，根据 Android Gradle 的规范，只有下面两个属性是必须的，其余都是我自己加的  
sdk.dir=/Users/Nelson/Library/Android/sdk
ndk.dir=/Users/Nelson/Library/Android/android-ndk-r16b
sdk.api=android-21
// 本地签名绝对地址
debug.keystore=/Users/Nelson/Downloads/guazi/tools/mykeystore.jks  
// 生成 library artifact 对应目录地址
local.dir=/Users/Nelson/Downloads/guazi/xrk-flat-dir/  
```
3. b_sales_android build.gradle 

作为 multi-project 根目录，一般情况下，它的 build.gradle 是做一些全局配置。来看我们的 build.gradle

```
// 下面这个 subprojects{}就是一个 Script Block  
subprojects {  
  println"Configure for $project.name" // 遍历子 Project，project 变量对应每个子 Project  
  buildscript {  // 这也是一个 SB  
    repositories {//repositories 是一个 SB  
       ///jcenter 是一个函数，表示编译过程中依赖的库，所需的插件可以在 jcenter 仓库中  
       // 下载。  
       jcenter()  
    }  
    dependencies { //SB  
        //dependencies 表示我们编译的时候，依赖 android 开发的 gradle 插件。插件对应的  
       //class path 是 com.android.tools.build。版本是 3.1.4  
        classpath'com.android.tools.build:gradle:3.1.4'  
    }  
   // 为每个子 Project 加载 utils.gradle 。当然，这句话可以放到 buildscript 花括号之后  
   applyfrom: rootProject.getRootDir().getAbsolutePath() + "/utils.gradle"  
 }//buildscript 结束  
}  
```

**SB 在 Gradle 的 API 文档中也是有的。先来看 Gradle 定义了哪些 SB。**

https://docs.gradle.org/current/dsl/org.gradle.api.Project.html

你看，subprojects、dependencies、repositories 都是 SB。那么 SB 到底是什么？它是怎么完成所谓配置的呢？

仔细研究，你会发现 SB 后面都需要跟一个花括号，而花括号，恩，我们感觉里边可能一个 Closure。由于文档中这些 SB 的 Description 都有“Configure xxx for this project”，所以很可能 subprojects 是一个函数，然后其参数是一个 Closure。是这样的吗？

Absolutely right。只是这些函数你直接到 Project API 里不一定能找全。不过要是你好奇心重，不妨到 https://docs.gradle.org/current/javadoc/ 选择 Index 这一项，然后 ctrl+f，输入任何一个 Block，你都会找到对应的函数。比如我替你找了几个 API：演示去文档查找，例如 buildscript https://docs.gradle.org/current/javadoc/index-all.html#I:B

**特别提示：当你下次看到一个不认识的 SB 的时候，就去看 API 吧。**

下面来解释代码中的各个 SB：

* subprojects：它会遍历 b_sales_android 中的每个子 Project。在它的 Closure 中，默认参数是子 Project 对应的 Project 对象。由于其他 SB 都在 subprojects 花括号中，所以相当于对每个 Project 都配置了一些信息。
* buildscript：它的 closure 是在一个类型为 ScriptHandler 的对象上执行的。主要用来所依赖的 classpath 等信息。通过查看 ScriptHandler API 可知，在 buildscript SB 中，你可以调用 ScriptHandler 提供的 repositories(Closure )、dependencies(Closure) 函数。这也是为什么 repositories 和 dependencies 两个 SB 为什么要放在 buildscript 的花括号中的原因。明白了？这就是所谓的行话，得知道规矩。不知道规矩你就乱了。记不住规矩，又不知道查 SDK，那么就彻底抓瞎，只能到网上到处找答案了！
* 关于 repositories 和 dependencies，大家直接看 API 吧。后面碰到了具体代码我们再来介绍

4. BIZ-xxx build.gradle

BIZ-xxx 是一个 Android Library。按 Google 的想法，Android Library 编译出来的应该是一个 AAR 文件。但是加入这时我们的项目有些特殊，我需要发布 xxx.jar 包给其他人使用。jar 在编译过程中会生成，但是它不属于 Android Library 的标准输出。在这种情况下，我需要在编译完成后，主动 copy jar 包到我自己设计的产出物目录中。

BIZ 下的工程都是一个个 Android Library。按 Google 的想法，Android Library 编译出来的应该是一个 AAR 文件。

问题：我现在需要发布 biz-xxx.jar 包给其他人使用，怎么办？

jar 在编译过程中会生成，但是它不属于 Android Library 的标准输出。在这种情况下，需要在编译完成后，主动 copy jar 包到我自己设计的产出目录中。

```
// Library 工程必须加载此插件。注意，加载了 Android 插件就不要加载 Java 插件了。因为 Android  
// 插件本身就是拓展了 Java 插件  
apply plugin: 'com.android.library'   
//android 的编译，增加了一种新类型的 ScriptBlock-->android  
android {  
       // 我在 local.properties 中设置的 API 版本号，就可以一次设置，多个 Project 使用了  
      // 借助我特意设计的 gradle.ext.api 属性  
       compileSdkVersion = gradle.api 
       sourceSets{ // 配置源码路径。这个 sourceSets 是 Java 插件引入的  
           main{ //main：Android 也用了  
               manifest.srcFile 'AndroidManifest.xml' // 这是一个函数，设置 manifest.srcFile  
               aidl.srcDirs=['src'] // 设置 aidl 文件的目录  
               java.srcDirs=['src'] // 设置 java 文件的目录  
            }  
        }  
        
        dependencies {  // 配置依赖关系  
          // compile 表示编译和运行时候需要的 jar 包，fileTree 是一个函数，  
         // dir:'libs'，表示搜索目录的名称是 libs。include:['*.jar']，表示搜索目录下满足 *.jar 名字的 jar  
         // 包都作为依赖 jar 文件  
           compile fileTree(dir: 'libs', include: ['*.jar'])  
       }  
}  //android SB 配置完了  

// 前面说了，我要把 jar 包拷贝到指定的目录。对于 Android 编译，我一般指定 gradle assemble  
// 它默认编译 debug 和 release 两种输出。所以，下面这个段代码表示：  
//tasks 代表一个 Projects 中的所有 Task，是一个容器。getByName 表示找到指定名称的任务。  
// 我这里要找的 assemble 任务，然后我通过 doLast 添加了一个 Action。这个 Action 就是 copy  
// 产出物到我设置的目标目录中去  

tasks.getByName("assemble"){  
   it.doLast{  
       println "$project.name: After assemble, jar libs are copied tolocal repository"  
        copyOutput(true)
        
        // 具体函数内部如下：
        from('build/intermediates/bundles/release/')
        into('build/outputs/')
        include('classes.jar')
        rename ('classes.jar', 'myLib.jar')
        into('release/') //you can change this directory where you want to copy your .jar
     }  
}  
/* 
  因为我的项目只提供最终的 release 编译出来的 Jar 包给其他人，所以不需要编译 debug 版的东西 
  当 Project 创建完所有任务的有向图后，我通过 afterEvaluate 函数设置一个回调 Closure。在这个回调 
  Closure 里，我 disable 了所有 Debug 的 Task 
*/  
project.afterEvaluate{  
    disableDebugBuild()  
}  
```

Android 自己定义了很多 ScriptBlock，Android 定义的 DSL 参考文档在：

**Android Gradle Plugin release notes**

https://developer.android.com/studio/releases/gradle-plugin

这条很重要，请惠存！里面包含了：[Android Plugin DSL Reference](http://google.github.io/android-gradle-dsl/current/)、[Configure Your Build](https://developer.android.com/studio/build/index.html)、[Gradle DSL Reference](https://docs.gradle.org/current/dsl/)

AGP-Android Gradle Plugin 版本号

<img desc="AGP 版本对比" src="http://wanghaoxun.com/img/gradle-agp-version.jpg" width="60%"/>

5. CRMApp build.gradle

再来看看 APK 的 build，它内部包含了编译（class，NDK），还有签名。

问题：根据项目需求，我们开发只能签 debug 版本的，而 release 版的签名得发布 unsigned 包给 leader 签名，怎么办？

```
apply plugin: 'com.android.application'  //APK 编译必须加载这个插件  
android {  
    compileSdkVersion gradle.api  
    buildToolsVersion "22.0.1"  
    sourceSets{ 
        main{  
            // 设置 jni 和 java 目录
            jni.srcDirs = []  
            java.srcDirs=['src']  
               
        }  
    }//main 结束  
    signingConfigs { // 设置签名信息配置  
        debug {  // 如果我们在 local.properties 设置使用特殊的 keystore，则使用它  
           // 下面这些设置，无非是函数调用.... 请务必阅读 API 文档  
           if(project.gradle.debugKeystore != null){  
              storeFile file("file://${project.gradle.debugKeystore}")  
              storePassword "android"  
              keyAlias "androiddebugkey"  
              keyPassword "android"  
           }  
        }  
    }//signingConfigs 结束  
    
    buildTypes {  
        debug {  
            signingConfig signingConfigs.debug  
            jniDebuggable false  
        }  
    }//buildTypes 结束  
    
    repositories {  
        flatDir {//flatDir：告诉 gradle，编译中依赖的 jar 包存储在 dirs 指定的目录  
           name "minsheng-gradle-local-repository" dirs gradle.LOCAL_JAR_OUT //LOCAL_JAR_OUT 是我存放编译出来的 jar 包的位置  
        }  
    }//repositories 结束  
}//android 结束  
/* 
   创建一个 Task，类型是 Exec，这表明它会执行一个命令。我这里让他执行 ndk 的 
   ndk-build 命令，用于编译 ndk。关于 Exec 类型的 Task，请自行脑补 Gradle 的 API 
*/  
// 注意此处创建 task 的方法，是直接{}喔，那么它后面的 tasks.withType(JavaCompile)  
// 设置的依赖关系，还有意义吗？Think！如果你能想明白，gradle 掌握也就差不多了  
task buildNative(type: Exec, description: 'CompileJNI source via NDK') {  
       if(project.gradle.ndkDir == null) // 看看有没有指定 ndk.dir 路径  
          println "CANNOT Build NDK"  
       else{  
            commandLine "/${project.gradle.ndkDir}/ndk-build",  
               '-C', file('jni').absolutePath,  
               '-j', Runtime.runtime.availableProcessors(),  
               'all', 'NDK_DEBUG=0'  
        }  
  }  
 tasks.withType(JavaCompile) {  
       compileTask -> compileTask.dependsOn buildNative  
  }  
  .....
```

6. 再来个实例演示

问题：现在 App 有个特点，有三个版本，分别是 debug、release 和 demo。这三个版本对应的代码都一样，但是在运行的时候会跳转到 debug、release 或者 demo 的逻辑上。

办法：在编译 build、release 和 demo 版本前，在 build.gradle 中自动设置 runtime_config 的内容。

```
apply plugin: 'com.android.application'  // 加载 APP 插件  

    signingConfigs {
        debug {
            ...
        }
    }
    /* 
     最关键的内容来了： buildTypes ScriptBlock。
     buildTypes 和上面的 signingConfigs，当我们在 build.gradle 中通过{}配置它的时候， 
     其背后的所代表的对象是 NamedDomainObjectContainer<BuildType> 和 
     NamedDomainObjectContainer<SigningConfig> 
     注意，NamedDomainObjectContainer<BuildType 或者 SigningConfig> 是一种容器， 
     容器的元素是 BuildType 或者 SigningConfig。我们在 debug{}要填充 BuildType 或者 
    SigningConfig 所包的元素，比如 storePassword 就是 SigningConfig 类的成员。而 proguardFile 等 
    是 BuildType 的成员。 
    那么，为什么要使用 NamedDomainObjectContainer 这种数据结构呢？因为往这种容器里 
    添加元素可以采用这样的方法： 比如 signingConfig 为例 
    
    signingConfigs {// 这是一个 NamedDomainObjectContainer<SigningConfig> 
       test1{// 新建一个名为 test1 的 SigningConfig 元素，然后添加到容器里 
         // 在这个花括号中设置 SigningConfig 的成员变量的值 
       } 
      test2{// 新建一个名为 test2 的 SigningConfig 元素，然后添加到容器里 
         // 在这个花括号中设置 SigningConfig 的成员变量的值 
      } 
    } 
    在 buildTypes 中，Android 默认为这几个 NamedDomainObjectContainer 添加了 
    debug 和 release 对应的对象。如果我们再添加别的名字的东西，那么 gradle assemble 的时候 
    也会编译这个名字的 apk 出来。比如，我添加一个名为 test 的 buildTypes，那么 gradle assemble 
    就会编译一个 xxx-test-yy.apk。在此，test 就好像 debug、release 一样。 
   */  
   buildTypes{  
        debug{ // 修改 debug 的 signingConfig 为 signingConfig.debug 配置  
            signingConfig signingConfigs.debug  
        }  
        demo{ //demo 版需要混淆  
            proguardFile 'proguard-project.txt'  
            signingConfig signingConfigs.debug  
        }  
       // release 版没有设置，所以默认没有签名，没有混淆  
    }  
    
    // 来看如何动态生成 runtime_config 文件  
   def runtime_config_file = 'assets/runtime_config'  
   /* 
   我们在 gradle 解析完整个任务之后，找到对应的 Task，然后在里边添加一个 doFirst Action 
   这样能确保编译开始的时候，我们就把 runtime_config 文件准备好了。 
   注意，必须在 afterEvaluate 里边才能做，否则 gradle 没有建立完任务有向图，你是找不到 
   什么 preDebugBuild 之类的任务的 
   */  
   project.afterEvaluate{  
      // 找到 preDebugBuild 任务，然后添加一个 Action   
      tasks.getByName("preDebugBuild"){  
           it.doFirst{  
               println "generate debug configuration for ${project.name}"  
               def configFile = new File(runtime_config_file)  
               configFile.withOutputStream{os->  
                   os << I am Debug\n'  // 往配置文件里写 I am Debug  
                }  
           }  
        }  
       // 找到 preReleaseBuild 任务  
       tasks.getByName("preReleaseBuild"){  
           it.doFirst{  
               println "generate release configuration for ${project.name}"  
               def configFile = new File(runtime_config_file)  
               configFile.withOutputStream{os->  
                   os << I am release\n'  
               }  
           }  
        }  
       // 找到 preDemoBuild。这个任务明显是因为我们在 buildType 里添加了一个 demo 的元素  
      // 所以 Android APP 插件自动为我们生成的  
       tasks.getByName("preDemoBuild"){  
           it.doFirst{  
               println "generate offlinedemo configuration for${project.name}"  
               def configFile = new File(runtime_config_file)  
               configFile.withOutputStream{os->  
                   os << I am Demo\n'  
               }  
            }  
        }  
    }  
}  
 .....//copyOutput  
```

why？为什么我知道有 preXXXBuild 这样的任务？

**./gradlew tasks --all 查看所有任务。然后，多尝试几次，直到成功**

# 四、锦囊文档集合

## 所有文档集合

1. Groovy-最新版本 v2.5.8
    * [Groovy 官网](http://www.groovy-lang.org/download.html#gvm) 
    * [Groovy 的 API 文档地址](http://www.groovy-lang.org/api.html)
    
    Groovy I/O
    
    * [java.io.File class](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/File.html)
    * [java.io.InputStream class](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/InputStream.html)
    * [java.io.OutputStream class](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/OutputStream.html)
    * [java.io.Reader class](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/Reader.html)
    * [java.io.Writer class](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/io/Writer.html)
    * [java.nio.file.Path class](http://docs.groovy-lang.org/latest/html/groovy-jdk/java/nio/file/Path.html)

2. Gradle--最新版本 v6.0.1

    * [Gradle 官网](http://gradle.org/)
    * [Gradle DSL Reference](https://docs.gradle.org/current/dsl/)
    * [Gradle-Project](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html)
    * [Gradle-Task](https://docs.gradle.org/current/dsl/org.gradle.api.Task.html)
    * [Gradle javadoc](https://docs.gradle.org/current/javadoc/)
    
3. AGP-Android Gradle Plugin

    * [Android Gradle Plugin release notes-Google Gradle 总入口](https://developer.android.com/studio/releases/gradle-plugin)
    * [Android Plugin DSL Reference](http://google.github.io/android-gradle-dsl/current/)
    * [Configure Your Build](https://developer.android.com/studio/build/index.html)
    * [Gradle DSL Reference](https://docs.gradle.org/current/dsl/)


# 五、总结

啰里啰嗦的讲了这么多 Gradle 知识，解开了它神秘的面纱，回想以前配置项目 Gradle 一直是我的心病，不知道如何下手，只能网上搜索，碰运气。走了不少弯路，直到完整学习了下 Gradle 框架，了解了套路，现在求解问题的思路也和以前不一样了：

* 最开始的时候，我一直把 gradle 当做脚本看。然后到处到网上找怎么配置 gradle。可能能编译成功，但是完全不知道为什么。比如 NameDomainObjectContainer，为什么有 debug、release。能自己加别的吗？不知道怎么加，没有章法，没有参考。出了问题只能 google，找到一个解法，试一试，成功就不管。这么搞，心里不踏实。
* 另外，对语法不熟悉，尤其是 Groovy 语法，虽然看了下快速教材，但总感觉一到 gradle 就看不懂。主要问题还是闭包，比如 Groovy 那一节写得文件拷贝的例子中的 withOutputStream，还有 gradle 中的 withType，都是些啥玩意啊？
* 所以后来下决心先把 Groovy 学会，主要是把自己暴露在闭包里边。另外，Groovy 是一门语言，总得有 SDK 说明吧。写了几个例子，慢慢体会到 Groovy 的好处，也熟悉 Groovy 的语法了。
* 接着开始看 Gradle。Gradle 有几本书，我看过 Gradle in Action。说实话，看得非常痛苦。现在想起来，Gradle 其实比较简单，知道它的生命周期，知道它怎么解析脚本，知道它的 API，几乎很快就能干活。而 Gradle In Action 一上来就很细，而且没有从 API 角度介绍。说个很有趣的事情，书中有个类似下面的例子。

```
task myTask  <<  {
   println ' I am myTask'
}
```

书中说，如果代码没有加 <<，则这个任务在脚本 initialization（也就是你无论执行什么任务，这个任务都会被执行，I am myTask都会被输出）的时候执行，如果加了<<，则在 gradle myTask 后才执行。

换成现在我们理解的：

这和我们调用 task 这个函数的方式有关！如果没有 <<，则闭包在 task 函数返回前会执行，而如果加了 <<，则变成调用 myTask.doLast 添加一个 Action 了，自然它会等到 grdle myTask 的时候才会执行！

**API 说清楚了，如果你把 Gradle 当做编程框架来看，对于我们这些程序猿来说，写这几百行代码，那还算事嘛？？？**

完结~ 下一篇我们讲解项目实战 Gradle 编译那些事，敬请等候。。。


---
**备注：Groovy 章节对应的代码在我本地，GitHub repo，嗯.. 先安全检查下再传上去吧，你懂的。。。**

# 六、参考

Blog

* [用 Gradle 构建 Android 和 Java by Google](https://cn.udacity.com/course/gradle-for-android-and-java--ud867)
* [邓平凡 | 深入理解 Android（一）：Gradle 详解](https://www.infoq.cn/article/android-in-depth-gradle/?utm_source=infoq&utm_campaign=user_page&utm_medium=link)
* [Gradle DSL Version 6.0.1](https://docs.gradle.org/current/dsl/)

UDACITY

* [Udacity | 用 Gradle 构建 Android 和 Java](https://cn.udacity.com/course/gradle-for-android-and-java--ud867)

API

* [Groovy核心类源码讲解(下)](https://www.imooc.com/article/44169)