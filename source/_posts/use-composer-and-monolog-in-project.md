title: "在项目中使用composer和monolog的记录"
date: 2015-09-24 12:39:50
categories:
- 工程
tags:
- composer
- monolog
---


## 缘起
目前在做一个比较独立的项目模块，有了一定的时间和自主性，所以针对目前项目开发过程中的弊端，我尝试使用一些增加对代码掌控能力的工具。

<!-- more -->

### PHPUnit
首先在项目中引入的是PHPUnit，之前因为赶工期，开发过程中一直没有引入单元测试。而且因为QA资源比较紧张，所以我们的很多功能都是自己测试的，增加了很多人力成本不说，很多地方测试得还不够细致，导致漏掉了bug。

在这种情况下，单元测试的环节必须得到重视。这样可以减少代码逻辑变更时的重复测试成本，大部分常见问题也可以在这一阶段被发现、解决。

PHPUnit的手册很详尽的记录了不同平台的安装方法，编写测试用例的简单示例以及各种特性，就不再赘述了。

### composer
在安装PHPUnit的过程中，发现了composer。之前管理第三方库或者组件的时候，一直是手工管理的，版本更新比较麻烦，处理起依赖也不太方便，所以准备以后使用composer管理依赖，正好可以借这次机会尝试。

composer的使用比较简单，本地安装后，在项目里增加composer.json文件，里面描述项目的依赖库，然后使用
<code>php composer.phar install</code>即可将所需库下载到vendor目录下，然后composer会生成一个composer.lock文件，里面记录了已下载的第三方库版本。将这个文件托管到svn或者git，这样在另外一个环境部署的时候，composer会根据composer.lock文件中记录的版本下载指定的第三方库，确保项目稳定运行。

这样看来，composer配合docker，将是搭建开发或测试环境的好方案，可以方便的解决软件环境和项目环境的依赖。有机会可以尝试一下。

如果需要新版本的第三方库，只需执行php composer.phar update或者php composer.phar update [package name]即可更新全部库或指定库，当然不会超过composer.json文件中限制的版本范围。

另外，composer还提供了ventor/autoload.php自动加载类，可以方便的加载用composer安装到ventor目录的依赖。

### monolog
在安装composer的过程中，发现了monolog，果然生命在于折腾。之前项目里面的日志实现是基于一个简单的写入本地文件的工具类实现的。服务器规模扩展的话将引起日志分散的问题，且缺少错误信息报警机制。

monolog的安装非常方便，直接使用composer安装即可。如果不使用composer，从github直接下载代码也是可以的。

monolog的逻辑并不复杂，核心概念只有几项，包括logger, channel, handler, formatter, processors等。学习过程记录如下：
- logger是日志的记录者，每条日志生成一个record。
- 创建logger时可以定义一个channel，体现在record的内容中，方便过滤。
- logger可以有多个handler，通过代理将record的具体处理方法交给handler实现。handler用一个栈存储，添加record的时候由栈顶的handler先处理。每个handler判断新增的record是否符合自己的log level，设定是否允许后续的handler继续处理。
- 每个handler有一个formatter，formatter可以将record内容清洗为符合handler需求的格式。
- processors可以添加到logger也可以添加到handler。processor以record为参数，返回处理后的record。processor的逻辑通常是向record的extra数组添加内容。在logger中，processor在handler之前处理record，在handler中，processor在formatter之前处理record。

如图所示：
![](http://7xiium.com1.z0.glb.clouddn.com/Monolog.png)


monolog提供了很多handler以及相应的formatter和processor，用户还可以根据需要自己添加handler、formatter、processors等，非常灵活。


## 后记
这篇博客拖了好久好久才写完，一方面是因为平时杂事比较多，总觉得腾不出手来写；另一方面也是因为自己执行力不足，很多计划好的事情都没有开始，时间没有用好。

不管怎样，还是要从眼前的事情一点一点做起，坚持写博客，养成总结的习惯，恢复更新，继续上路。