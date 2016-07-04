title: "taskgo开发手册 -1- 项目结构与命令行参数解析"
date: 2015-10-23 10:25:49
categories:
- 开发手册
tags:
- golang
---


## 项目结构
在设计命令行参数之前，我先折腾了一番项目结构。因为是第一次做GO的项目，所以对build方式不太熟悉，只是知道main包会生成可执行程序，其他的包会生成库。这样的结构对一个只有一个可执行程序的项目来说已经足够了，但是我希望在taskgo的项目中包含server、worker、client三种可执行程序，就需要调整一下项目结构。

<!-- more -->

开始的时候，我给server、worker、client各自创建了一个目录，准备每个目录下面创建一个main package，但是这样的结构因为在项目目录下没有扩展名是go的程序文件，所以没办法直接使用go install构建，也没办法通过go get安装，需要自己写安装脚本，用户安装的时候也更麻烦一些。这和我期望的简单易用不太符合，所以这种方案被暂时搁置。

然后，我参考了etcd的项目结构，准备以server作为项目的主程序，将server的main包放在项目的根目录中。然后server、worker、client的具体实现分别写在对应的库中，worker和client单独创建main包生成工具程序。server可以用go install直接构建，worker和client使用命令单独构建。这样项目的逻辑就更加清晰，单独拆分出来的server、worker、client也更便于复用。

## 命令行参数设计目标
设计命令行参数的时候，我参考了之前用过的gearman，再结合自己对server的设想，草拟了一份参数表。
预期的参数包含：
- h: --help, 输出帮助信息
- V: --version, 输出版本信息
- v: --verbose, 调试信息等级，等级越高输出信息越具体。
- h: --host, server的主机名或ip地址
- p: --port, server监听的端口号
- u: --user, 以特定用户的权限执行
- d: --deamon, 以deamon程序方式运行
- l: --log, 日志文件路径
- P: --pid-file, pid文件路径
- s: --signal, 信号，reload，stop， start
- r: --retry, job执行失败的重试次数
- S: --save < seconds > < changes >, 持久化设置
- 后续待补充

目前只想到了上述的参数，计划按照顺序依次实现。后续持续添加其他需要的参数。

## 实现
参数解析使用flag包实现，声明一个server config的结构体，然后使用flag解析命令行参数填充。用flagSet定义可能更清楚一些，不过我准备先用flag.xxx的方式实现了。
flag包貌似不支持‘--xxx’的形式，更复杂的命令行参数解析可以使用[gooptions](https://github.com/voxelbrain/goptions)实现。

之前计划写taskgo的时候是想按照模块设计和记录的，不过拖延症发作，半年一直没有进展，我觉得还是先写一个demo，然后不断调整更实际一些，最近用了Quartz，觉得有很多可以借鉴的地方，后面如果有合适的特性会尽量加进来。
