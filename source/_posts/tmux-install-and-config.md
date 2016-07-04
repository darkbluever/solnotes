title: "tmux安装配置记录"
date: 2015-07-22 16:52:16
categories:
- study
tags:
- tools
---

之前早就听过screen，一直没有尝试，今天有空准备试用一下。然而搜索screen相关教程的时候，看到很多tmux和screen的对比<sup>[\[1\]](#ref1) [\[2\]](#ref2)</sup>，大家普遍对tmux评价很高，认为可以作为screen的替代品，而且tmux的pane划分也很吸引我，于是我决定直接从tmux开始上手。

<!-- more -->

之前我也在James Pan的博客<sup>[\[3\]](#ref3)</sup>和其他别的博客里面看到过tmux相关的文章，不过一直没有什么直观的感受，这次一边安装配置，一边又重新看了之前读过的博客。感觉之前看过的东西完全没有留下印象，直到这一次动手试了才有比较直观的感受，果然是生命在于折腾。

tmux最吸引我的地方是session deattch和attach。因为工作过程中需要保持多个ssh连接，每天上班开始都要新建这些连接，而且往往吃完午饭回来就发现都断开了，又要重新连接，之前的工作状态完全被打断。所以希望通过tmux保留工作环境。其次才是pane分块带来的一些效率提升和乐趣。

## 安装
我在自己用的Ubuntu上面用apt-get安装的tmux，比较简单。但是公司的CentOS上的yum没有找到tmux，所以使用源码安装。

tmux需要libeven2.0.10以上版本，所以我先安装了libevent2.0.22，然后在tmux目录执行configure的时候指明了gcc compiler的include路径和linker的library路径，具体命令如下：
```
CFLAGS="-I/usr/local/libevent2/include" LDFLAGS="-L/usr/local/libevent2/lib" ./configure --prefix=/usr/local/tmux
```

执行tmux的时候找不到libevent2.so.5，在/usr/lib64建立了软连接后运行成功。


## 配置
tmux的默认配置文件在~/.tmux.conf，已经有很多人提供了自己的配置，我参考了几份配置文件以后整理了一份还算易用的配置。

```
#此类配置可以在命令行模式中输入show-options -g查询
set-option -g base-index 1                        #窗口的初始序号；默认为0，这里设置为1
set-option -g display-time 5000                   #提示信息的持续时间；设置足够的时间以避免看不清提示，单位为毫秒
set-option -g repeat-time 1000                    #控制台激活后的持续时间；设置合适的时间以避免每次操作都要先激活控制台，单位为毫秒
#set-option -g status-interval 1                  #状态栏刷新频率
set-option -g status-keys vi                      #操作状态栏时的默认键盘布局；可以设置为vi或emacs
set-option -g status-utf8 on                      #开启状态栏的UTF-8支持
set-option -g history-limit 65535                 #历史行数
set-option -g status on                           #显示状态栏
set-option -g mouse-select-pane on                #鼠标可以选中pane
set-option -g status-bg black                     #状态栏背景色
set-option -g status-fg cyan                      #状态栏前景色
set-option -g message-attr dim                    #


#此类设置可以在命令行模式中输入show-window-options -g查询
set-window-option -g mode-keys vi                 #复制模式中的默认键盘布局；可以设置为vi或emacs
set-window-option -g utf8 on                      #开启窗口的UTF-8支持
set-window-option -g pane-base-index 1            #pane初始序号，默认为0，设置为1
set-window-option -g mode-mouse on                #鼠标滚轮可以使用
set-window-option -g window-status-fg red         #窗口状态栏前景色
set-window-option -g window-status-bg default     #窗口状态栏背景色
set-window-option -g window-status-attr dim       #

#-- colorscheme --#
# statusbar
set-option -g status-left '#[fg=red] [#[fg=blue]#I #[fg=red]]#[default]'
set-option -g status-right "#[fg=red][ #[fg=green]%H:%M #[fg=blue]%a %m-%d #[fg=red]] #[default]"
set-window-option -g window-status-format '#[fg=blue,bold]#I #T#[default] '
set-window-option -g window-status-current-format '#[fg=red] [#[fg=green]#I #T#[fg=red]]#[default] '
set-option -g status-left-length 40
set-option -g status-right-length 25


#-- bindkeys --#
# prefix key (Ctrl+a)
set-option -g prefix ^a
unbind ^b
bind a send-prefix

# select pane
bind k selectp -U # above (prefix k)
bind j selectp -D # below (prefix j)
bind h selectp -L # left (prefix h)
bind l selectp -R # right (prefix l)
```

## 脚本
tmux可以进入命令行模式，快捷键为:，详细的可用命令与说明可以参考man page<sup>[\[4\]](#4)</sup>。

tmux命令可以写成脚本，并通过配置文件将执行脚本命令（source-file）绑定到某个按键，从而实现快速设置工作环境等目标。

此外，tmux的命令也支持通过tmux + command的方式在shell环境调用，所以也可以将一系列tmux命令写为bash脚本的方式简化常用的初始化类操作。

## ending
在写这篇博客的时候，通过James Pan的文章发现了[Poweline-vim](https://github.com/Lokaltog/vim-powerline)，一款很强大很漂亮的vim工具栏插件，tmux貌似也能用上。我是个不折不扣的工具控，比较喜欢折腾，一定要找机会试一试。不过现在已经用了不少时间了，堆了一些待办的事情，只能往后推一推了。


## reference
<span id="ref1">1.toy [从 screen 切换到 tmux](https://linuxtoy.org/archives/from-screen-to-tmux.html)</span>
<span id="ref2">2.foocoder [终端环境之tmux](http://foocoder.com/blog/zhong-duan-huan-jing-zhi-tmux.html/)</span>
<span id="ref3">3.James Pan [Tmux、Luit 杂谈](http://blog.jamespan.me/2015/06/12/luit-with-tmux/)</span>
<span id="ref4">4.[linux man page](http://linux.die.net/man/1/tmux)</span>