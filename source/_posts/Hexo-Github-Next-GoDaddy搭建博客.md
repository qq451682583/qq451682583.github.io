---
title: Hexo+Github+Next+GoDaddy搭建博客
date: 2019-07-13 18:18:40
tags:
    - Hexo
    - GitHub
    - Next
    - Atom
categories: 工具
---

![](http://ww4.sinaimg.cn/large/006cL6EBjw1f5z9r5crsqj30ys0hujts.jpg)
<!--more-->

## **简介**
---

  - <font size=4>Hexo 是高效的静态站点生成框架，她基于 Node.js。 通过 Hexo 你可以轻松地使用 Markdown 编写文章，除了 Markdown 本身的语法之外，还可以使用 Hexo 提供的 标签插件 来快速的插入特定形式的内容。[hexo github链接](https://github.com/hexojs/hexo)<p>
  - <font size=4>Github开源社区[Github](https://github.com/)
  - <font size=4>Github Pages,用于介绍托管在GitHub的项目， 不过，由于他的空间免费稳定，用来做搭建一个博客再好不过了。[Github Pages](https://pages.github.com/)
  - <font size=4>Hexo主题中star最多的一种[Next](http://theme-next.iissnan.com/)

## **准备工作**
---
- <font size=4>GitHub(创建仓库,SSH,pages关联hexo,仓库分支)[配置教程](http://blog.sina.com.cn/s/blog_6e572cd60101qls0.html))<p>
  -注：Github Pages的Repository名字是特定的(github账号.github.io)。
 - <font size=4>安转Node.js([Node.js官网](https://nodejs.org/en/)下载相应平台的最新版本)<p>
 - <font size=4>安装[Git](https://git-scm.com/download)<p>
 - <font size=4>Atom等MD编辑器.
 ## **开始安装Hexo**
 ---
 <font size=4>Node.js和Git都安装好后,首先创建一个文件夹,如blog,用户自定义的hexo安装文件夹,然后cd进入blog目录里安装Hexo。

 <font size=4>执行如下命令安装Hexo：

 ```
 cd 你的Hexo安装目录(blog) 执行

sudo npm install-g hexo

 ```

 <font size=4>然后初始化hexo,命令:

 `hexo init`

 <font size=4>OK，至此，全部安装工作已经完成！blog就是你的博客根目录，所有的操作都在里面进行.之后的操作都需要cd 到你的blog目录.


 ## **静态页面**
 ---
 <font size=4>上一步我们已经安装了hexo,生成了blog.<p>
 之后在blog目录,找到_config.yml文件,这就是我们的站点配置文件,可以在此文件中修改我们的配置信息.修改配置文件的教程不多说[传送门](https://hexo.io/docs/configuration.html)

 <font size=4>修改的差不多,就可以本地预览下我们的网站了:

 生成静态页:

 `hexo generate`(简写hexo g)

 本地启动:

 `hexo server`(简写hexo s)

 浏览器输入(localhost:4000),预览我们的博客了,如果打不开,有可能端口被占用了,这时候我们可以指定端口号,命令如下:

 `hexo s -p 4001`(根据指定的端口号浏览博客)

 ## **部署到GitHub上**
 ---

上一步我们已经可以本地预览我们的blog了,当然本地看到是远远不够,现在开始关联github让我们的blog可以分享给更多的小伙伴.


#### 建立GitHub仓库


进入github账号,创建与你用户名对应的仓库，仓库名必须为[your_user_name.github.io]，固定写法不可修改

之后修改我们的配置文件_config.yml,翻到最下面,增加github配置:

```
deploy:

     type: git

     repo: 你自己的仓库地址

     branch: master
```

然后执行命令(记住所有的操作都是在你的blog目录下)：

`npm install hexo-deployer-git --save`

最后执行:

`hexo deploy`(博客部署,提交到仓库)

#### Hexo目录结构
```
.
├── .deploy_git：#将public文件夹的内容提交到Github后生成，内容与public文件夹基本一致
├── node_modules：#用来存储已安装的各类依赖包
├── public  #将source文件夹里的Markdown文档，转换成index.html。再结合主题进行渲染，就是我们最终看到的博客
├── scaffolds #模板文件夹,当您新建文章时，根据 scaffold生成文件,包含page，post，draft三种模板，分别对应 页面、要发布的文章、草稿
├── source  #资源文件夹,用来存放图片、Markdown文档（文章、草稿）、各种页面（分类、关于页面等）
|   └── _posts #博客文章目录
└── themes #主题 
├── _config.yml   #网站的配置信息。标题、网站名称等
├── db.json：#source解析所得到的
├── package.json  # 用来查看Hexo的版本以及相关依赖包的版本
```
#### source ， public 和 .deploy_git

```
这三者的关系大致是：source -> public -> .deploy_git

执行hexo generate，根据source，更新 public。
执行hexo deploy，根据public，更新 .deploy_git。
```

[Hexo配置](https://hexo.io/zh-cn/docs/configuration)

### Hexo会默认安装

- hexo：主程序
- hexo-deployer-git：实现git部署方式
- hexo-generator-archive：存档页面生成器
- hexo-generator-category：分类页面生成器
- hexo-generator-index：index生成器
- hexo-generator-tag：标签页面生成器
- hexo-renderer-ejs：支持EJS渲染
- hexo-renderer-marked：Markdown引擎
- hexo-renderer-stylus：支持stylus渲染
- hexo-server：支持本地预览，默认地址 localhost:4000

之后我们就可以在浏览器输入[your_user_name.github.io],好了赶快撰写文章,分享你的blog吧.

## **Hexo常用命令**
---

```
hexo new"postName" #新建文章

hexo new page"pageName" #新建页面
```
[传送门](https://segmentfault.com/a/1190000002632530)


## **Next主题**
---

网站部署好后,开始设置Next主题


#### 安装Next主题

```
cd your-hexo-site
git clone https://github.com/iissnan/hexo-theme-next themes/next

```
#### 启用主题

和所有的主题启动一样,打开_config.yml,找到theme字段,修改为next

之后我们可以hexo g ,hexo s,localhost:4000本地浏览一下

#### 主题类型设置


选择 Scheme

Scheme 是 NexT 提供的一种特性，借助于 Scheme，NexT 为你提供多种不同的外观。同时，几乎所有的配置都可以 在 Scheme 之间共用。目前 NexT 支持三种 Scheme，他们是：
  - Muse - 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白<p>
  -  Mist - Muse 的紧凑版本，整洁有序的单栏外观<p>
  -  Pisces - 双栏 Scheme，小家碧玉似的清新<p>
  -  Scheme 的切换通过更改 主题配置文件，搜索 scheme 关键字。 你会看到有三行 scheme 的配置，将你需用启用的 scheme 前面注释 # 即可

这里我启用的是Mist

#### Next配置修改

 - 侧边栏头像  `avatar: /uploads/avatar.jpg(你头像的路径)`<p>
 - 菜单   `menu`<p>
 - 设置语言  `language: zh-Hans`<p>
 - 修改背景色 next->source->css->_schemes(启用哪个类型选哪个文件夹)->Mist->index.styl 最上边第一行加`body { background:url(/images/background.jpg)(图片路径,或者#ffffff颜色);}`<p>
 - 等等修改[官网教程](http://theme-next.iissnan.com/getting-started.html)<p>


 #### MarkDown语法简单介绍

![MarkDown](https://user-images.githubusercontent.com/11883853/68834565-10fb2180-06f1-11ea-9af1-d977ac74aee7.png)

<blockquote class="blockquote-center">参考链接</blockquote>

- [Hexo中文文档](https://hexo.io/zh-cn/docs/)
- [绑定github个人博客到GoDaddy](https://segmentfault.com/a/1190000002632530)
- [超详细Hexo+Github Page搭建技术博客教程](https://segmentfault.com/a/1190000017986794)
- [利用Hexo在多台电脑上提交和更新github pages博客](https://www.jianshu.com/p/0b1fccce74e0)
- [Themes](https://github.com/hexojs/hexo/wiki)
- [GoDaddy](https://dcc.godaddy.com/manage/lzhblog.site/dns)