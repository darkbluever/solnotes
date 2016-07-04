title: "MySQL高可用笔记 -1- Replication基础"
date: 2016-02-19 11:47:14
categories:
- study
tags:
- mysql
- mysql replication
---

## 前言
MySQL Replication 可以将 master 节点的所有改变准确的复制到 slave 节点，按照复制方式可以分为 statement-based replication 和 row-based replication。

<!-- statement-based replication 是基于statement的复制，在语句少，数据变更大的时候效果更好，row-based replication 是基于数据行的复制，在语句多，数据变更相对少的时候效果更好。 -->

Replication 作用：
- 异地多活
- 水平扩展，负载均衡
- 使用 slave 节点进行备份、数据分析等
- 配置 delayed slave，避免误操作的影响

<!-- more -->

## 建立 Replication 的基础步骤
建立一个最基础的 Replication 需要3个步骤，
1. 设置 master
2. 设置 slave
3. 将 slave 连接到 master

### 设置 master
最基础的 master 设置是设置server ID 和配置 binlog。server ID 是区分两个 server 的唯一标识，binlog 记录了 master 上的所有变化，通过在 slave 重放 master 的 binlog，可以将 master 的改变更新到 slave。

设置 server ID 和 binlog 需要停止 mysqld 服务，然后修改 my.cnf 配置文件。

一个简单的示例配置如下：
```
[mysqld]
...
server-id     = 1
log-bin       = master-bin
log-bin-index = master-bin.index
```

log-bin 设置了 binlog 文件的 basename，如果 log-bin 设置的值中包含了扩展名，那么扩展名部分将被忽略，只有 basename 部分会被使用。log-bin 的默认值是 hostname-bin，如果 server 的 hostname 改变，binlog 的文件名也将变更，不过在 binlog index 文件中会被记录。

log-bin-index 配置了 binlog index 文件的名称，文件中存储了 binlog 文件的索引。log-bin-index 的默认值是 binlog 文件的 basename，扩展名是 index。如果没有指定 log-bin 的值，那 log-bin-index 的默认值就将是 hostname-bin.index，如果管理员修改了 hostname，那么 mysqld 将找不到之前的 binlog index 文件，并假设其不存在，这样会重新生成新的 binlog 文件，因为之前的已经无法访问到了。

server ID 是识别server的唯一标识，如果设置重复的话，在 slave 连接到 master 的时候将会触发错误。

设置好以后重启 mysqld 就可以使 Replication 设置生效了。

接下来，slave 就可以连接到这台 master 进行复制了。这时，master server 需要一个具有 replication 相关权限的用户来进行复制相关的操作。

操作示例如下：
```
master > CREATE USER repl_user;
master > GRANT REPLICATION SLAVE ON *.*
      -> TO repl_user IDENTIFIED BY 'cptbtptp';
```

*REPLICATION SLAVE 权限这是赋予了用户获取 master 上的 binlog 的 dump 文件的权利，完全可以将其赋予一个现有用户，不过将 Replication 用户和 app 相关用户分隔开会更加方便安全。*


## 设置 slave
接下来需要配置 slave 的 server ID 和 relay-log相关配置。

relay-log、relay-log-index 项和 log-bin，log-bin-index 比较像，默认值分别是 hostname-relay-bin 和 hostname-relay-bin.index。在 hostname 变更的时候也会有相同的问题。

简单的示例配置如下：
```
[mysqld]
...
server-id       = 2
relay-log       = slave-relay-bin
relay-log-index = slave-relay-bin.index
```


## 连接 slave 和 master
最后一步是将 slave 连接到 master，开始复制。

最简单的方式是使用 CHANGE MASTER TO 和 START SLAVE 命令设置并开始复制。CHANGE MASTER TO 命令需要至少4个参数：
- a hostname
- a port number
- a user account on the master with replication slave privileges
- a password for the user account

简单示例如下：
```
slave > CHANGE MASTER TO
     ->   MASTER_HOST = 'master-1',
     ->   MASTER_PORT = 3306,
     ->   MASTER_USER = 'repl_user',
     ->   MASTER_PASSWORD = 'cptbtptp';

slave > START SLAVE;
```

其中 MASTER_HOST 参数可以是 hostname 也可以是IP地址，如果是 hostname 会通过 DNS lookup 解析成IP地址。

如果 master 已经运行一段时间并有很多数据了，更好的方式是先创建 master 或 已存在的 slave 的备份，记录备份数据的 binlog position。在 slave 导入备份，并在执行 CHANGE MASTER TO 命令的时候，指明 MASTER_LOG_FILE 和 MASTER_LOG_POS 参数，从备份的数据之后进行复制，这样可以减少复制的时间，并降低 master 的压力。


## 结语
通过以上步骤，可以建立最基础的 MySQL Replication，更多的设置还需要参考 MySQL 手册。
