title: "svn工作流管理个人实践"
date: 2015-05-04 16:42:51
categories:
- 工程
tags:
- svn
- git
- 工程
---

## 起
这篇博客起源于一次发布事故的反思。五一之前因为修复一个线上bug，紧急发布了一个补丁，结果发布的代码中包含了正在开发中的新功能的代码，导致了线上部分功能受影响，服务出错不可用。

<!-- more -->

## 承
在回滚线上代码并紧急修复了bug之后，我们有必要重新思考一下这次出现的问题。
- 代码没有做有效的管理，修复bug的时候没有从trunk新建issue分支
- 发布代码上线的时候没有经过充分的测试
- 因为是内部产品且所修复模块用户量不大，工程师略有松懈，发布比较仓促
- 发布的时候线上代码没有进行备份

## 转
经过反思，我们重新梳理了对svn工作流的管理。

![](http://nvie.com/img/git-model@2x.png "Vincent Driessen - a-successful-git-branching-model")

借鉴 Vincent Driessen 的[a-successful-git-branching-model](http://nvie.com/posts/a-successful-git-branching-model/)，我们将svn的工作流分为以下几个部分。

### 发布
trunk作为项目的主分支，只维护可发布的稳定代码。开发不在trunk上进行，只有要发布的时候才把代码合并到trunk。

所有发布都基于trunk的代码进行，发布时新建tag以标记发布代码的快照。

### 开发
除了trunk以外，版本库最重要的一个分支是dev分支。dev分支主要用于日常的开发工作。除了release和fix bug的分支，其他的分支不直接合并到trunk，而是合并到dev分支。

### 预发布
当dev分支的代码处在某个逻辑闭环的稳定状态时，可以基于dev分支新建预发布分支，可以命名为release-\*，其中\*为要发布的版本号。

预发布分支经过测试和bug修复稳定下来以后，需要合并回dev分支和trunk，然后trunk的代码可以用于正式发布新的版本。

### bug修复
当在线上版本发现bug后，基于trunk新建修复特定bug的分支，可以命名为issue-\*，其中\*部分使用bug追踪系统的bug编号。

bug修复并测试通过以后，将issue分支合并回trunk和dev分支。

### 新特性开发
当有某种特定的新功能需要开发时，可以在基于dev新建的特定分支上进行，这样的分支可以命名为feature-\*，其中\*为新特性的代号。

如果需要包括新特性，则将其合并到dev分支。如果最终不需要的话可以删除此分支。

## 合
基于svn的工作流有很多种，以上只是其中一种实践。这样可以充分隔离开不同的功能和版本之间的代码，在保证开发效率的同时，确保发布代码的稳定性。

Git的branch和tag、merge操作都更简洁高效，不过目前公司使用的主流工具还是svn，所以本篇博客主要基于svn，不足之处还请大家指出。

## reference
1. Vincent Driessen [a-successful-git-branching-model](http://nvie.com/posts/a-successful-git-branching-model/)
2. ruanyifeng [Git分支管理策略](http://www.ruanyifeng.com/blog/2012/07/git.html)
3. oldratlee [Git工作流指南](https://github.com/oldratlee/translations/blob/master/git-workflows-and-tutorials/README.md)
