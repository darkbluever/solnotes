title: Human Resource Machine 记录
date: 2018-03-02 15:00:06
tags: game

---

## 开始

开始玩Human Resource Machine 还是在2016年，当时是听一期内核恐慌的时候知道的，听起来很有意思就入坑了。

> Human Resource Machine 是一个解谜游戏，玩家使用某种简陋的汇编语言，控制小职员去完成各种工作任务。每一关除完成任务外都还有两个目标，一个是要使用尽可能少的指令（不多于 N 个指令），另一个是在运行时使用尽可能少的步骤（不多于 M 个步骤）；

游戏介绍我就偷懒直接引用 JAMES PAN的博客[Human Resource Machine 通关大吉](https://blog.jamespan.me/posts/human-resource-machine)了。

我玩这个游戏的时候，中途中断了两次，基本都是因为没有达到优化目标，而我又不想开始下一关，就卡住了……一次是在三排序，一次是在斐波那契参观者。每次都是过了很久之后，想起来又再尝试，可能因为之前思路已经忘记，重新玩的时候往往顺利的通过了，没有遇到之前的问题。

说来说去，写这篇博客的目的还是看了 JAMES PAN 的博客以后，自己也想记录下游戏里写的代码，下面回归正题。

<!-- more -->

## 第一年 收发室

仅为熟悉  `INOBX`，`OUTBOX` 指令设计的教学关卡。

★★

```assembly
; YEAR 1

INBOX
OUTBOX
INBOX
OUTBOX
INBOX
OUTBOX
```

## 第二年 繁忙的收发室

引入了跳转命令，可以实现循环。

★☆

```assembly
; YEAR 2 SIZE CHALLENGE

a:
	INBOX   
	OUTBOX
	JUMP     a
```



不过使用简单循环无法满足执行指令数的挑战，所以需要展开循环，减少跳转指令的比例。

☆★

```assembly
; YEAR 2 SPEED CHALLENGE

a:
	INBOX
	OUTBOX
	INBOX
	OUTBOX
	JUMP     a
```



## 第三年 复印楼层

引入了 `COPYFROM` 指令，可以使用经过抽象简化的寄存器了。

★★

```assembly
; YEAR 3

COPYFROM 4
OUTBOX
COPYFROM 0
OUTBOX
COPYFROM 3
OUTBOX
```



## 第四年 解扰码器

引入了 `COPYTO` 指令。

★★

```assembly
; YEAR 4

a:
	INBOX
	COPYTO   0
	INBOX
	OUTBOX
	COPYFROM 0
	OUTBOX
	JUMP     a
```



## 第五年 咖啡时间

背景新闻：本市局部仍处于停电之中，有关部门已介入调查。



## 第六年 多雨之夏

增加了 `ADD` 指令。

★★

```assembly
; YEAR 6

a:
	INBOX
	COPYTO   0
	INBOX
	ADD      0
	OUTBOX
	JUMP     a
```



## 第七年 零之杀手

增加了 `JUMPZ` 指令。

★★

```assembly
; YEAR 7

a:
	INBOX
	JUMPZ    a
	OUTBOX
	JUMP     a
```



## 第八年 三倍扩大室

★★

```assembly
; YEAR 8

a:
	INBOX
	COPYTO    0
	ADD       0
	ADD       0
	OUTBOX
	JUMP      a
```



## 第九年 保护零行动

按照直觉直接写出来的代码只能满足指令条数的挑战，所以还需要减少一些不必要的循环才能完成执行速度的挑战。

★☆

```assembly
; YEAR 9 SIZE CHALLENGE

a:
	INBOX
	JUMPZ    b
	JUMP     a
b:
	OUTBOX
	JUMP     a
```

☆★

```assembly
; YEAR 9 SPEED CHALLENGE

a:
	INBOX
	JUMPZ    b
	JUMP     a
b:
	OUTBOX
c:
	INOBX
	JUMPZ    b
	JUMP     c
```



## 第十年 八倍扩大器套件

★★

```assembly
; YEAR 10

a:
	INBOX
	COPYTO   0
	ADD      0
	COPYTO   0
	ADD      0
	COPYTO   0
	ADD      0
	OUTBOX
	JUMP     a
```



## 第十一年 Sub 走廊

引入了 `SUB` 指令。

★★

```assembly
; YEAR 11

a:
	INBOX
	COPYTO   0
	INBOX
	COPYTO   1
	SUB      0
	OUTBOX
	COPYFROM 0
	SUB      1
	OUTBOX
	JUMP     a
```



## 第十二年 四十倍扩大器

★★

```assembly
; YEAR 40

a:
	INBOX
	COPYTO   0
	ADD      0
	COPYTO   1
	ADD      1
	ADD      0
	COPYTO   0
	ADD      0
	COPYTO   0
	ADD      0
	COPYTO   0
	ADD      0
	OUTBOX
	JUMP     a
```



## 第十三年 平等化室

增加了注释指令，适当展开循环后可以减少跳转，完成指令条数和执行速度的挑战。

★★

```assembly
; YEAR 13

a:
	INBOX
	COPYTO   0
	INBOX
	SUB      0
	JUMPZ    b
	JUMP     c
b:
	COPYFROM 0
	OUTBOX
c:
	INBOX
	COPYTO   0
	INBOX
	SUB      0
	JUMPZ    b
	JUMP     c

```



## 第十四年 最大化室

引入了 `JUMPN` 指令。

★★

```assembly
; YEAR 14

a:
	INBOX
	COPYTO   0
	INBOX
	SUB      0
	JUMPN    b
	ADD      0
	JUMP     c
b:
	COPYFROM 0
c:
	OUTBOX
	JUMP     a
```



## 第十五年 员工斗志的注入

背景新闻：一批庞大的机器军队已经包围了本市，这些机器拒绝移动，也拒绝谈判，有关部门正在调查。



## 第十六年 绝对正能量

★★

```assembly
; YEAR 16

a:
	INBOX
	JUMPN    b
	JUMP     c
b:
	COPYTO   0
	SUB      0
	SUB      0
c:
	OUTBOX
	INBOX
	JUMPN    b
	JUMP     c
```



## 第十七年 VIP 休息室

★★

```assembly
; YEAR 17

a:
	INBOX
	JUMPN    c
	INBOX
	JUMPN    d
b:
	COPYFROM 4
	OUTBOX
	JUMP     a
c:
	INBOX
	JUMPN    b
d:
	COPYFROM 5
	OUTBOX
	JUMP     a
```



## 第十八年 公休海滩天堂

没 get 到信息



## 第十九年 倒计时

引入了 `BUMPUP` 和 `BUMPDN` 指令。

第一份代码逻辑比较直接，指令数比较少。第二份代码为正数和负数各写了一套逻辑，避免每次循环都判断正负，指令冗余多一些，但是执行效果更高。

（这一关的房间地面上有个机械臂……）

★☆

```assembly
; YEAR 19 SIZE CHALLENGE

a:
	INBOX
	COPYTO   0
b:
	OUTBOX
	COPYFROM 0
	JUMPZ    a
	JUMPN    c
	BUMPDN   0
	JUMP     d
c:
	BUMPUP   0
d:
	JUMP     b
```



☆★

```assembly
; YEAR 19 SPEED CHALLENGE

a:
	INBOX
	JUMPZ    e
	JUMPN    c
	COPYTO   0
b:
	OUTBOX
	BUMPDN   0
	JUMPZ    e
	JUMP     b
c:
	COPYTO   0
d:
	OUTBOX
	BUMPUP   0
	JUMPZ    e
	JUMP     d
e:
	OUTBOX
	JUMP     a
```



## 第二十年 乘法研讨会

引入寄存器标签。

第一份代码逻辑简单直接，满足指令数要求。我自己基于第一份代码莫改了一番，展开两层循环，达到了执行速度的要求，不过会受测试数据集的影响，只在目前的测试数据上可以满足要求。所以还是参考了 JAMES PAN 的实现，先比较两个数的大小，比较大的作为被乘数，小的作为乘数，这样可以减少循环次数，是更通用的做法。

★☆

```assembly
; YEAR 20 SIZE CHALLENGE
a:
    INBOX   
    COPYTO   1
    INBOX   
    COPYTO   2
    COPYFROM 9
    COPYTO   0
b:
    BUMPDN   1
    JUMPN    c
    COPYFROM 2
    ADD      0
    COPYTO   0
    JUMP     b
c:
    COPYFROM 0
    OUTBOX  
    JUMP     a
```



☆★

```assembly
; YEAR 20 SPEED CHALLENGE

a:
b:
    INBOX   
    JUMPZ    h
    COPYTO   1
    INBOX   
    JUMPZ    i
    COPYTO   2
    SUB      1
    JUMPN    d
    COPYFROM 2
    COPYTO   0
    BUMPDN   1
c:
    BUMPDN   1
    JUMPN    g
    COPYFROM 2
    ADD      0
    COPYTO   0
    JUMP     c
d:
    COPYFROM 1
    COPYTO   0
    BUMPDN   2
e:
    BUMPDN   2
    JUMPN    f
    COPYFROM 1
    ADD      0
    COPYTO   0
    JUMP     e
f:
g:
    COPYFROM 0
    OUTBOX  
    JUMP     b
h:
    INBOX   
i:
    COPYFROM 9
    OUTBOX  
    JUMP     a
```



## 第二十一年 以零结尾的求和

第一份简单逻辑的代码可以满足指令条数的要求，然后增加对0长度的字符串的特殊处理以后，满足了执行速度的要求。

★☆

```assembly
; YEAR 21 SIZE CHALLENGE

a:
	COPYFROM 5
	COPYTO   0
b:
	INBOX
	JUMPZ    c
	ADD      0
	COPYTO   0
	JUMP     b
c:
	COPYFROM 0
	OUTBOX
	JUMP     a
```



☆★

```assembly
; YEAR 21 SPEED CHALLENGE

a:
	INBOX
	JUMPZ    d
	COPYTO   0
b:
	INBOX
	JUMPZ    c
	ADD      0
	JUMP     b
c:
	COPYFROM 0
d:
	OUTBOX
	JUMP     a
```



## 第二十二年 斐波那契参观者

这一关我只写出了满足执行速度的代码，然后就卡住了，过了很久以后再回来玩的时候，参考 JAMPES PAN 的代码完成了指令条数的要求。

我使用了3个寄存器放斐波那契数列迭代的中间结果，通过 `A+B->C `, `B+C->A` , `C+A->B` 的循环得到新的结果，不过 JAMES PAN 的代码是通过 `A+B->B` , `B-A->A` 的循环就完成了斐波那契数列的计算，所以指令数也满足了要求。

☆★

```assembly
; YEAR 22 SPEED CHALLENGE

a:
	INBOX
	COPYTO   0
	COPYFROM 9
	COPYTO   1
	BUMPUP   1
	COPYTO   2
	OUTBOX
	COPYFROM 1
	OUTBOX
b:
	COPYFROM 1
	ADD      2
	COPYTO   2
	SUB      0
	JUMPN    c
	JUMPZ    c
	JUMP     a
c:
	ADD      0
	OUTBOX
	COPYFROM 2
	ADD      3
	COPYTO   1
	SUB      0
	JUMPN    d
	JUMPZ    d
	JUMP     a
d:
	ADD      0
	OUTBOX
	COPYFROM 1
	ADD      3
	COPYTO   2
	SUB      0
	JUMPN    e
	JUMPZ    e
	JUMP     a
e:
	ADD      0
	OUTBOX
	JUMP     b
```



★★

```assembly
; YEAR 22
a:
    INBOX   
    COPYTO   0
    COPYFROM 9
    COPYTO   1
    COPYTO   2
    BUMPUP   2
b:
    SUB      0
    JUMPN    d
    JUMPZ    c
    JUMP     a
c:
d:
    COPYFROM 2
    ADD      1
    COPYTO   2
    SUB      1
    COPYTO   1
    OUTBOX  
    COPYFROM 2
    JUMP     b
```



## 第二十三年 最小的数字

★★

```assembly
; YEAR 23

a:
	INBOX
	COPYTO   0
b:
	INBOX
	JUMPZ    d
	SUB      0
	JUMPN    c
	JUMP     b
c:
	ADD      0
	COPYTO   0
	JUMP     b
d:
	COPYFROM 0
	OUTBOX
	JUMP     a
```



## 第二十四年 模运算模块

★★

```assembly
; YEAR 24

a:
	INBOX
	COPYTO   0
	INBOX
	COPYTO   1
	COPYFROM 0
b:
	SUB      1
	JUMPN    c
	JUMPZ    d
	JUMP     b
c:
	ADD      1
d:
	OUTBOX
	JUMP     a
```



## 第二十五年 累加的倒计时

★★

```assembly
; YEAR 25

a:
	INBOX
	JUMPZ    d
	COPYTO   0
b:
	COPYTO   1
	BUMPDN   0
	JUMPZ    c
	ADD      1
	JUMP     b
c:
	COPYFROM 1
d:
	OUTBOX
	JUMP     a
```



## 第二十六年 小小的除法

★★

```assembly
; YEAR 26

a:
	INBOX
	COPYTO   0
	INBOX
	COPYTO   1
	COPYFROM 0
	COPYTO   2
b:
	COPYFROM 0
	SUB      1
	JUMPN    c
	COPYTO   0
	BUMPUP   2
	JUMP     b
c:
	COPYFROM 2
	OUTBOX
	JUMP     a
```



## 第二十七年 深夜石油

深夜，窗外出现了巨大的眼球和非常多的发光的眼镜，疑似机器人入侵。



## 第二十八年 三排序

按照简单逻辑处理，通过比较把三个数原地排序，最后按照预设的顺序输出即可。

★☆

```assembly
; YEAR 28 SIZE CHALLENGE

a:
	INBOX
	COPYTO   0
	INBOX
	COPYTO   1
	INBOx
	COPYTO   2
b:
	COPYFROM 2
	SUB      1
	JUMPN    c
	JUMPZ    c
	ADD      1
	COPYTO   7
	COPYFROM 1
	COPYTO   2
	COPYFROM 7
	COPYTO   1
c:
	COPYFROM 1
	SUB      0
	JUMPN    d
	JUMPZ    d
	ADD      0
	COPYTO   6
	COPYFROM 0
	COPYTO   1
	COPYFROM 6
	COPYTO   0
	JUMP     b
d:
	COPYFROM 2
	OUTBOX
	COPYFROM 1
	OUTBOX
	COPYFROM 0
	OUTBOX
	JUMP     a
```



这一关如果想满足执行速度的要求，需要把循环完全展开，为每种排序结果写一段输出逻辑。执行过程类似于决策树，按照三个数比较结果选择路线，最终叶子节点包含了这种排序结果的输出顺序。而且要尽量利用程序的执行顺序，最大程度上减少跳转次数。我第一次玩的时候，因为循环全部展开以后实在太乱了，所以有一处多余的跳转，造成执行步数没满足要求，过了很久重新玩的时候，在纸上理清了执行路径，才通过了。

☆★

```assembly
; YEAR 28 SPEED CHANLLENGE

a:
	INBOX
	COPYTO   0
	INBOX
	COPYTO   1
	INBOx
	COPYTO   2
	SUB      1
	; 2 < 1
	JUMPN    d
	; 2 > 1
	COPYFROM 0
	SUB      2
	JUMPN    b
	; 1 < 2 < 0
	COPYFROM 1
	OUTBOX
	COPYFROM 2
	OUTBOX
	COPYFROM 0
	OUTBOX
	JUMP a
b:
	; 2 > 1 && 2 > 0
	COPYFROM 1
	SUB      0
	; 1 < 0 < 2
	JUMPN    c
	; 0 < 1 < 2
	COPYFROM 0
	OUTBOX
	COPYFROM 1
	OUTBOX
	COPYFROM 2
	OUTBOX
	JUMP a
c:
	; 1 < 0 < 2
	COPYFROM 1
	OUTBOX
	COPYFROM 0
	OUTBOX
	COPYFROM 2
	OUTBOX
	JUMP a
d:
	; 2 < 1
	COPYFROM 1
	SUB      0
	; 2 < 1 < 0
	JUMPN    f
	; 2 < 1 && 0 < 1
	COPYFROM 2
	SUB      0
	; 2 < 0 < 1
	JUMPN    e
	; 0 < 2 < 1
	COPYFROM 0
	OUTBOX
	COPYFROM 2
	OUTBOX
	COPYFROM 1
	OUTBOX
	JUMP a
e:
	; 2 < 0 < 1
	COPYFROM 2
	OUTBOX
	COPYFROM 0
	OUTBOX
	COPYFROM 1
	OUTBOX
	JUMP a
f:
	; 2 < 1 < 0
	COPYFROM 2
	OUTBOX
	COPYFROM 1
	OUTBOX
	COPYFROM 0
	OUTBOX
	JUMP a
```

PS：过了这一关，玩家角色的头发就白了，还戴上了眼镜……



## 第二十九年 仓库楼层

增加了寄存器取址的能力。

★★

```assembly
; YEAR 29

a:
	INBOX
	COPYTO   10
	COPYFROM [10]
	OUTBOX
	JUMP     a
```



## 第三十年 串存储楼层

★★

```assembly
; year 30

a:
	INPUT
	COPYTO   24
b:
	COPYFROM 24
	JUMPZ    a
	OUTBOX
	BUMPUP   24
	JUMP     b
```



## 第三十一年 串的反转

★★

```assembly
; YEAR 31

a:
	INBOX
	JUMPZ    b
	COPYTO   [14]
	BUMPUP   14
	JUMP     a
b:
	BUMPDN   14
	COPYFROM [14]
	OUTBOX
	COPYFROM 14
	JUMPZ    a
	JUMP     b
```



## 第三十二年 库存报告

★☆

```assembly
; YEAR 32 SIZE CHALLENGE

a:
    INBOX
    COPYTO   15
    COPYFROM 14
    COPYTO   19
    COPYTO   18
b:
    COPYFROM [19]
    JUMPZ    e
    SUB      15
    JUMPZ    d
c:
    BUMPUP   19
    JUMP     b
d:
    BUMPUP   18
    JUMP     c
e:
    COPYFROM 18
    OUTBOX
    JUMP     a
```



☆★

```assembly
; YEAR 32 SPEED CHALLENGE

a:
	INBOX
	COPYTO   15
	COPYFROM 14
	COPYTO   19
	COPYTO   18
b:
	COPYFROM [19]
	JUMPZ    d
	SUB      15
	JUMPZ    c
	BUMPUP   19
	JUMP     b
c:
	BUMPUP   18
	BUMPUP   19
	JUMP     b
d:
	COPYFROM 18
	OUTBOX
	JUMP     a
```



## 第三十三年 王五哪儿去了？

王五从楼上掉下去了。背景新闻：伪装成女主持人的机器人报道，之前关于机器军队的新闻等于假……

这是已经开始渗透进人类社会了么……



## 第三十四年 元音焚化炉

（地板上有一只机械臂，还有两个螺母，看起来很像眼睛。）

★★

```assembly
; YEAR 34

a:
	COPYFROM 5
	COPYTO   6
	INBOX
	COPYTO   7
b:
	COPYFROM [6]
	JUMPZ    c
	SUB      7
	JUMPZ    a
	BUMPUP   6
	JUMP     b
c:
	COPYFROM 7
	OUTBOX
	JUMP     a
```



## 第三十五年 删除重复项

（屋子的窗户打破了一大块，墙壁也破了……）

按照直观的方式写了一份逻辑简单直接的代码，达成了指令数的要求。

★☆

```assembly
; YEAR 35 SIZE CHALLENGE

a:
	COPYFROM 14
	COPYTO   13
b:
	INBOX
	COPYTO   [13]
	COPYFROM 13
	COPYTO   14
c:
	BUMPDN   14
	JUMPN    d
	COPYFROM [14]
	SUB      [13]
	JUMPZ    b
	JUMP     c
d:
	COPYFROM [13]
	OUTBOX
	BUMPUP   13
	JUMP     b
```



展开循环，对前几个输入项特殊处理，可以减少部分逻辑，最终满足了执行速度的要求。

☆★

```assembly
; YEAR 35 SPEED CHALLENGE

a:
	INBOX
	COPYTO   0
	OUTBOX
	BUMPUP   14
	COPYTO   13
	INBOX
	COPYTO   [13]
	SUB      0
	JUMPZ    b
	COPYFROM [13]
	OUTBOX
	BUMPUP   13
b:
	COPYFROM 13
	COPYTO   14
c:
	BUMPDN   14
	INBOX
	COPYTO   [13]
	SUB      [14]
	JUMPZ    b
d:
	BUMPDN   14
	JUMPN    e
	COPYFROM [14]
	SUB      [13]
	JUMPZ    b
	JUMP     d
e:
	COPYFROM [13]
	OUTBOX
	BUMPUP   13
	JUMP     b
```



## 第三十六年 字母饼干

★★

```assembly
; YEAR 36

a:
	COPYFROM 23
	COPYTO   24
	COPYTO   21
b:
	INBOX
	COPYTO   [23]
	JUMPZ    c
	BUMPUP   23
	JUMP     b
c:
	BUMPUP   23
	COPYTO   22
d:
	INOBX
	COPYTO   [23]
	JUMPZ    e
	BUMPUP   23
	JUMP     d
e:
	COPYFROM 22
	COPYTO   23
f:
	COPYFROM [22]
	JUMPZ    g
	COPYFROM [24]
	JUMPZ    h
	SUB      [22]
	JUMPZ    i
	JUMPN    h
g:
	COPYFROM [23]
	JUMPZ    b
	OUTBOX
	BUMPUP   23
	JUMP     g
h:
	COPYFROM [21]
	JUMPZ    b
	OUTBOX
	BUMPUP   21
	JUMP     h
i:
	BUMPUP   24
	BUMPUP   22
	JUMP     f
```



## 第三十七年 拾荒者之链

这一关通过地板上的格子给出了一个链表，只要简单使用就可以了。

★★

```assembly
; YEAR 37

a:
	INBOX
b:
	COPYTO   7
	COPYFROM [7]
	OUTBOX
	BUMPUP   7
	COPYFROM [7]
	JUMPN    a
	JUMP     b
```



## 第三十八年 数位炸弹

通过直观的写法可以满足指令条数的要求。

★☆

```assembly
; YEAR 38 SIZE CHALLENGE

a:
	COPYFROM 9
	COPYTO   2
	COPYTO   5
	COPYTO   8
	INBOX
	COPYTO   0
b:
	COPYFROM 0
	SUB      11
	JUMPN    c
	COPYTO   0
	BUMPUP   2
	JUMP     b
c:
	COPYFROM 2
	JUMPZ    d
	OUTBOX
d:
	COPYFROM 0
	SUB      10
	JUMPN    e
	COPYTO   0
	BUMPUP   5
	JUMP     d
e:
	COPYFROM 5
	ADD      2
	JUMPZ    f
	SUB      2
	OUTBOX
f:
	COPYFROM 0
	OUTBOX
	JUMP     a
```



如果要满足执行速度的要求，还需要做一些优化。例如：

- 优先判断 INBOX 值是否小于10，是否小于100，从而跳过部分比较逻辑。
- 还有通过减少比较次数，直接使用地板上提供的步长是100和10，这里我自己处理一下，构造了300和30作为步长。比较时先使用300作为步长，在 `x < 300`  以后使用100作为步长，不过还是没有满足要求。我开始尝试了使用200和20作为步长，不过因为与100和10取值比较接近，所以优化效果有限，然后将步长改为了300和30。
- 在这个基础上，对  `x < 300` 的场景进一步优化，不使用步长循环，直接做两次 SUB 100，就可以直接得到结果了。

☆★

```assembly
; YEAR 38 SPEED CHALLENGE

a:
	COPYFROM 11
	ADD      11
	ADD      11
	COPYTO   8
	COPYFROM 10
	ADD      10
	ADD      10
	COPYTO   7
	COPYFROM 9
	COPYTO   6
	BUMPUP   6
	BUMPUP   6
	BUMPUP   6
b:
	INBOX
	COPYTO   0
	SUB      10
	JUMPN    k
	COPYFROM 9
	COPYTO   1
	COPYTO   2
	COPYFROM 0
	SUB      11
	JUMPN    g
	COPYTO   0
	BUMPUP   1
c:
	COPYFROM 0
	SUB      8
	JUMPN    d
	COPYTO   0
	COPYFROM 1
	ADD      6
	COPYTO   1
	JUMP     c
d:
	COPYFROM 0
	SUB      11
	JUMPN    f
	COPYTO   0
	SUB      11
	JUMPN    e
	COPYTO   0
	BUMPUP   1
e:
	BUMPUP   1
f:
	COPYFROM 1
	OUTBOX
g:
	COPYFROM 0
	SUB      7
	JUMPN    h
	COPYTO   0
	COPYFROM 2
	ADD      6
	COPYTO   2
	JUMP     g
h:
	COPYFROM 0
	SUB      10
	JUMPN
	COPYTO   0
	SUB      10
	JUMPN    
	COPYTO   0
	BUMPUP   2
i:
	BUMPUP   2
j:
	COPYFROM 2
	OUTBOX
k:
	COPYFROM 0
	OUTBOX
	JUMP     b
```



## 第三十九年 重设坐标

★★

```assembly
; YEAR 39

a:
	COPYFROM 14
	COPYTO   13
	INBOX
b:
	SUB      15
	JUMPN    c
	COPYTO   0
	BUMPUP   13
	COPYFROM 0
	JUMP     b
c:
	ADD      15
	OUTBOX
	COPYFROM 13
	OUTBOX
	JUMP     a
```



## 第四十年 质数工厂

用汇编写质因数分解，想想还是挺有意思的。因为只有一些简单指令，所以肯定也是使用简单逻辑完成。我的思路是：

1.维护一个质数生成器，依序生成质数，设为 A

2.取待检测数 X（初始为 INBOX 值），判断是否可以整除 A，无法整除跳到步骤1，获取下一个质数；可以整除跳到步骤 3

3.将 A 输出至 OUTBOX，将上一步除法的商设置为 X, 若 X = 1，结束循环；否则返回步骤 2

为了减少指令数，质数生成器shi使用从2开始的自然数生成器替代了，剩余逻辑写起来倒是比较简单，根据直观感觉可以写出逻辑简单且满足指令条数要求的代码。

★☆

```assembly
; YEAR 40 SIZE CHALLENGE

a:
	INBOX
	COPYTO   1
	COPYTO   0
	COPYFROM 24
	COPYTO   20
	BUMPUP   20
	BUMPUP   20
b:
	COPYFROM 24
	COPYTO   21
c:
	COPYFROM 0
	SUB      20
	JUMPZ    e
	JUMPN    d
	COPYTO   0
	BUMPUP   21
	JUMP     c
d:
	BUMPUP   20
	COPYFROM 1
	COPYTO   0
	JUMP     b
e:
	COPYFROM 20
	OUTBOX
	COPYFROM 21
	JUMPZ    a
	BUMPUP   21
	COPYTO   0
	COPYTO   1
	JUMP     b
```



为了满足执行速度的要求，需要对代码进一步优化，可以优化的点包括：

- 使用质数生成器代替目前的自然数生成器，可以减少一些不必要的数字的检测，减少循环。但检测生成的数是否是质数逻辑复杂，在 INBOX 中数字比较小的情况下，对执行效率的可能会有负面影响。
- 优化判断整除逻辑，减少比较次数。可以考虑判断奇偶数，判断每一位上数字只和是否可以被3整除，结尾是否5或0等。但可使用的指令有限，判断逻辑较复杂。此处还需要用到整除场景下的商，所以做除法无法避免。尝试考虑减少做除法的比较次数，展开循环，减少跳转逻辑占比。

做完这个简单的优化以后，满足执行速度要求了……

☆★

```assembly
; YEAR 40 SPEED CHALLENGE

a:
	INBOX
	COPYTO   1
	COPYTO   0
	COPYFROM 24
	COPYTO   20
	BUMPUP   20
	BUMPUP   20
b:
	COPYFROM 24
	COPYTO   21
c:
	COPYFROM 0
	SUB      20
	JUMPZ    f
	JUMPN    e
	SUB      20
	JUMPZ    d
	JUMPN    e
	COPYTO   0
	BUMPUP   21
	BUMPUP   21
	JUMP     c
d:
	BUMPUP   21
	JUMP     f
e:
	BUMPUP   20
	COPYFROM 1
	COPYTO   0
	JUMP     b
f:
	COPYFROM 20
	OUTBOX
	COPYFROM 21
	JUMPZ    a
	BUMPUP   21
	COPYTO   0
	COPYTO   1
	JUMP     b
```



## 第四十一年 排序楼层

排序。先试着写了一个选择排序，满足了指令条数的要求。

★☆

指令条数目标：34   实际：32

执行速度目标：714 实际：816

```assembly
; YEAR 41 SELECTION SORT

a:
	; init
	COPYFROM 24
	COPYTO   20
b:
	; input
	INBOX
	JUMPZ    c
	COPYTO   [20]
	BUMPUP   20
	JUMP     b
c:
	; sorting
	BUMPDN   20
	COPYTO   22
d:
	COPYFROM 20
	JUMPZ    h
	COPYTO   21
	BUMPDN   21
e:
	COPYFROM [20]
	SUB      [21]
	JUMPN    f
	; swap
	COPYFROM [20]
	COPYTO   23
	COPYFROM [21]
	COPYTO   [20]
	COPYFROM 23
	COPYTO   [21]
f:
	BUMPDN   21
	JUMPN    g
	JUMP     e
g:
	BUMPDN   20
	JUMP     d
h:
	; output
	COPYFROM [22]
	OUTBOX
	BUMPDN   22
	JUMPN    a
	JUMP     h
```



上面代码的执行速度与目标值差距不算小，应该不是优化当前逻辑可以实现的。看来稳定 O(n^2) 复杂度的方法无法满足执行速度的要求。

插入排序的最好时间是 O(n)，也许可以尝试一下。于是我写了一个插入冒泡排序，因为基于现有指令，冒泡换位和依次移位后插入的复杂度差不多，为了简化逻辑，避免使用过多间接寻址，我使用了冒泡换位的位移逻辑。

☆★

指令条数目标：34   实际：48

执行速度目标：714 实际：643

```assembly
; YEAR 41 INSERTION SORT

a:
	INBOX
	COPYTO   0
	INBOX
	JUMPZ    c
	COPYTO   1
	SUB      0
	JUMPN    b
	; swap 0 and 1
	COPYFROM 0
	COPYTO   23
	COPYFROM 1
	COPYTO   0
	COPYFROM 23
	COPYTO   1
b:
	JUMP     d
c:
	COPYFROM 0
	OUTBOX
	JUMP     a
d:
	; init
	COPYFROM 24
	COPYTO   20
	BUMPUP   20
	BUMPUP   20
e:
	; sorting
	COPYFROM 20
	COPYTO   21
	COPYTO   22
	BUMPDN   22
	INBOX
	JUMPZ    g
	COPYTO   [20]
	BUMPUP   20
f:
	COPYFROM [21]
	SUB      [22]
	JUMPN    e
	; swap
	COPYFROM [21]
	COPYTO   23
	COPYFROM [22]
	COPYTO   [21]
	COPYFROM 23
	COPYTO   [22]
	BUMPDN   21
	BUMPDN   22
	JUMPN    e
	JUMP     f
g:
	BUMPDN   20
h:
	COPYFROM [20]
	OUTBOX
	BUMPDN   20
	JUMPN    a
	JUMP     h
```



其他常用的排序算法，时间复杂度可能满足需求的，我只想到希尔排序有希望满足这一关的空间复杂度（地板上只有25个格子）。大多排序算法需要用到特殊的数据结构，或是需要用到高级语言的操作符。例如快速排序需要有类似函数调用栈或者数组、队列等结构。

我尝试写了一版希尔排序的代码，不过指令条数和执行速度都未满足要求……应该是因为可用的基础指令比较少，高级语言的一些低成本操作在用这些指令实现的时候，成本会高很多。

wikipedia 伪代码：

```c
// shell sort
void shell_sort(int arr[], int len) {
	int gap, i, j;
	int temp;
	for (gap = len >> 1; gap > 0; gap >>= 1)
		for (i = gap; i < len; i++) {
			temp = arr[i];
			for (j = i - gap; j >= 0 && arr[j] > temp; j -= gap)
				arr[j + gap] = arr[j];
			arr[j + gap] = temp;
		}
}
```



☆☆

指令条数目标：34   实际：58

执行速度目标：714 实际：1149

```assembly
; YEAR 41 SHELL SORT

a:
	; init
	COPYFROM 24
	COPYTO   20
b:
	; input
	INBOX
	JUMPZ    b1
	COPYTO   [20]
	BUMPUP   20
	JUMP     b
b1:
	COPYFROM 20
	COPYTO   21
c:
	; init step
	COPYFROM 24
	COPYTO   22
d:
	; step
	BUMPUP   22
	BUMPDN   21
	; stop when step < 1
	JUMPZ    j
	SUB      22
	JUMPZ    e
	JUMPN    e
	JUMP     d
e:
	; sorting
	; i
	COPYFROM 21
	COPYTO   22
f:
	; temp
	COPYFROM [22]
	COPYTO   19
g:
	; j
	COPYFROM 22
	SUB      21
	COPYTO   23
	JUMPN    i
h:
	COPYFROM [23]
	SUB      19
	JUMPN    i
	JUMPZ    i
	COPYFROM 23
	ADD      21
	COPYTO   18
	COPYFROM [23]
	COPYTO   [18]
	COPYFROM 23
	SUB      21
	COPYTO   23
	JUMPN    i
	JUMP     h
i:
	COPYFROM 23
	ADD      21
	COPYTO   23
	COPYFROM 19
	COPYTO   [23]
	BUMPUP   22
	SUB      20
	JUMPZ    c
	JUMP     f
j:
	; init output
	COPYFROM 24
	COPYTO   21
	BUMPDN   20

k:
	; output
	COPYFROM [21]
	OUTBOX
	BUMPUP   21
	BUMPDN   20
	JUMPN    a
	JUMP     k
```



## 第四十二年 程序结束。恭喜。

用户角色的职位被计算机取代，大楼外面人类反抗军正在对抗着机械军队。用户角色走出大楼，离开了画面。



## 结束

玩了一遍以后，觉得这个游戏最棒的地方，就是非常恰当的抽象，让没有计算机基础的玩家也能玩得很开心，除了最后一关需要一点排序的知识……

最后有一点遗憾就是剧情没有弄得太明白，看起来有一些职场和科技的讽刺意味在里面，不知道有没有更多含义。
