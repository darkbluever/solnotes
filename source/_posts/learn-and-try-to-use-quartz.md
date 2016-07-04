title: "Quartz部署及开发学习"
date: 2015-10-27 09:51:12
categories:
- study
tags:
- Quartz
---

## 初识
之前有个项目想做用户自定义规则的定时任务，类似于Crontab的效果。当时调研了一下，基于JAVA的开源作业调度框架Quartz可能是技术和文档最成熟的选择。最近项目中用到了Quartz，所以看了下文档，简单学习了Quartz的部署和开发。

<!-- more -->

## 了解
Quartz是个作业调度框架，相比于Crontab支持秒级别的时间粒度，还支持作业的持久化和集群化的部署。
Quartz的主要元素有：
- Scheduler ：调度器，根据触发器的设置调度作业，和JobStore交互
- Trigger ：触发器，触发作业，可以在触发的时候覆盖作业的数据
- Job ：作业，处理具体工作
- JobDetail ：包含作业数据，定义具体的作业实例
- Scheduler Listener, Job Listener，Trigger Listener ：各种Listener

![Quartz元素](http://7xiium.com1.z0.glb.clouddn.com/Quartz.png)

## 持久化
Quartz的项目中有持久化相关的DB建表SQL，我使用的MySQL InnoDB的建表语句，不过项目中的表结构和字段都没有注释，不太方便理解，表的说明还能通过google查到一些，字段的说明就比较少了，我测试了一下Quartz的持久化，结合google到的一些Quartz工作机制，给Quartz的主要字段增加了注释，记录如下。
```
--
-- In your Quartz properties file, you'll need to set
-- org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate
--
--
-- By: Ron Cordell - roncordell
--  I didn't see this anywhere, so I thought I'd post it here. This is the script from Quartz to create the tables in a MySQL database, modified to use INNODB instead of MYISAM.

DROP TABLE IF EXISTS QRTZ_LOCKS;
DROP TABLE IF EXISTS QRTZ_SCHEDULER_STATE;
DROP TABLE IF EXISTS QRTZ_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_FIRED_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_SIMPLE_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_CRON_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_BLOB_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_SIMPROP_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_PAUSED_TRIGGER_GRPS;
DROP TABLE IF EXISTS QRTZ_CALENDARS;
DROP TABLE IF EXISTS QRTZ_JOB_DETAILS;

CREATE TABLE QRTZ_LOCKS (
SCHED_NAME VARCHAR(120) NOT NULL COMMENT 'Scheduler名称，同一集群下的Scheduler实例名称相同，Instance_Id不同',
LOCK_NAME VARCHAR(40) NOT NULL COMMENT '锁名称，TRIGGER_ACCESS，STATE_ACCESS，JOB_ACCESS，CALENDAR_ACCESS，MISFIRE_ACCESS',
PRIMARY KEY (SCHED_NAME,LOCK_NAME))
ENGINE=InnoDB COMMENT '存储锁以及获得锁的调度器名称。获取的锁不存在时创建，获得锁的调度器可以操作相应数据';

CREATE TABLE QRTZ_SCHEDULER_STATE (
SCHED_NAME VARCHAR(120) NOT NULL COMMENT 'Scheduler名称',
INSTANCE_NAME VARCHAR(200) NOT NULL COMMENT 'Scheduler实例的唯一标识，配置文件中的Instance Id',
LAST_CHECKIN_TIME BIGINT(13) NOT NULL COMMENT '最后检入时间',
CHECKIN_INTERVAL BIGINT(13) NOT NULL COMMENT 'Scheduler 实例检入到数据库中的频率，单位毫秒',
PRIMARY KEY (SCHED_NAME,INSTANCE_NAME))
ENGINE=InnoDB COMMENT '存储少量的有关Scheduler的状态信息，和别的Scheduler实例';

CREATE TABLE QRTZ_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL COMMENT 'Scheduler名称',
TRIGGER_NAME VARCHAR(200) NOT NULL COMMENT 'Trigger key',
TRIGGER_GROUP VARCHAR(200) NOT NULL COMMENT 'Trigger group 名称',
JOB_NAME VARCHAR(200) NOT NULL COMMENT 'Job key',
JOB_GROUP VARCHAR(200) NOT NULL COMMENT 'Job group 名称',
DESCRIPTION VARCHAR(250) NULL COMMENT 'Trigger description， .withDescription()方法传入的string',
NEXT_FIRE_TIME BIGINT(13) NULL COMMENT '下一次触发时间',
PREV_FIRE_TIME BIGINT(13) NULL COMMENT '上一次触发时间，默认-1',
PRIORITY INTEGER NULL COMMENT 'Trigger 优先级，默认5',
TRIGGER_STATE VARCHAR(16) NOT NULL COMMENT 'Trigger状态，ACQUIRED：正常运行；PAUSED：暂停；BLOCKED：阻塞；ERROR：错误；（待补全） ',
TRIGGER_TYPE VARCHAR(8) NOT NULL COMMENT 'Cron 或 Simple',
START_TIME BIGINT(13) NOT NULL COMMENT 'Trigger开始i时间',
END_TIME BIGINT(13) NULL COMMENT 'Trigger结束时间',
CALENDAR_NAME VARCHAR(200) NULL COMMENT 'Trigger关联的 Calendar name',
MISFIRE_INSTR SMALLINT(2) NULL COMMENT 'misfire规则id',
JOB_DATA BLOB NULL COMMENT '存储Trigger的JobDataMap等',
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,JOB_NAME,JOB_GROUP)
REFERENCES QRTZ_JOB_DETAILS(SCHED_NAME,JOB_NAME,JOB_GROUP))
ENGINE=InnoDB COMMENT '存储已配置的Trigger的信息';

CREATE TABLE QRTZ_FIRED_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL COMMENT 'Scheduler名称',
ENTRY_ID VARCHAR(95) NOT NULL COMMENT '',
TRIGGER_NAME VARCHAR(200) NOT NULL COMMENT 'Trigger key',
TRIGGER_GROUP VARCHAR(200) NOT NULL COMMENT 'Trigger group 名称',
INSTANCE_NAME VARCHAR(200) NOT NULL COMMENT 'Scheduler实例的唯一标识（应该是完成这次调度的Scheduler标识，待多实例环境测试验证）',
FIRED_TIME BIGINT(13) NOT NULL COMMENT '触发时间',
SCHED_TIME BIGINT(13) NOT NULL COMMENT '（疑似下一次触发时间，待验证）',
PRIORITY INTEGER NOT NULL COMMENT 'Trigger 优先级',
STATE VARCHAR(16) NOT NULL COMMENT 'Trigger状态',
JOB_NAME VARCHAR(200) NULL COMMENT 'Job key',
JOB_GROUP VARCHAR(200) NULL COMMENT 'Job group 名称',
IS_NONCONCURRENT VARCHAR(1) NULL COMMENT '是否不允许并发',
REQUESTS_RECOVERY VARCHAR(1) NULL COMMENT 'Scheduler实例发生故障时，故障恢复节点会检测故障的Scheduler正在调度的任务是否需要recovery，如果需要会添加一个只执行一次的simple trigger重新触发',
PRIMARY KEY (SCHED_NAME,ENTRY_ID))
ENGINE=InnoDB COMMENT '存储与已触发的Trigger相关的状态信息，以及相联Job的执行信息';


CREATE TABLE QRTZ_SIMPLE_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL COMMENT 'Scheduler名称',
TRIGGER_NAME VARCHAR(200) NOT NULL COMMENT 'Trigger key',
TRIGGER_GROUP VARCHAR(200) NOT NULL COMMENT 'Trigger group 名称',
REPEAT_COUNT BIGINT(7) NOT NULL COMMENT '重复次数',
REPEAT_INTERVAL BIGINT(12) NOT NULL COMMENT '重复间隔',
TIMES_TRIGGERED BIGINT(10) NOT NULL COMMENT '触发次数',
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP))
ENGINE=InnoDB COMMENT '存储简单的Trigger，包括重复次数、间隔、以及已触的次数';

CREATE TABLE QRTZ_CRON_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL COMMENT 'Scheduler名称',
TRIGGER_NAME VARCHAR(200) NOT NULL COMMENT 'Trigger key',
TRIGGER_GROUP VARCHAR(200) NOT NULL COMMENT 'Trigger group 名称',
CRON_EXPRESSION VARCHAR(120) NOT NULL COMMENT '调度规则',
TIME_ZONE_ID VARCHAR(80) COMMENT '时区',
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP))
ENGINE=InnoDB COMMENT '存储CronTrigger，包括Cron表达式和时区信息';

CREATE TABLE QRTZ_BLOB_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL COMMENT 'Scheduler名称',
TRIGGER_NAME VARCHAR(200) NOT NULL COMMENT 'Trigger key',
TRIGGER_GROUP VARCHAR(200) NOT NULL COMMENT 'Trigger group 名称',
BLOB_DATA BLOB NULL COMMENT '对于用户自定义的Trigger信息，无法提前设计字段，所以序列化后使用BLOB存储',
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
INDEX (SCHED_NAME,TRIGGER_NAME, TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP))
ENGINE=InnoDB COMMENT '存储用户自定义的Trigger';

CREATE TABLE QRTZ_SIMPROP_TRIGGERS
  (
    SCHED_NAME VARCHAR(120) NOT NULL COMMENT 'Scheduler名称',
    TRIGGER_NAME VARCHAR(200) NOT NULL COMMENT 'Trigger key',
    TRIGGER_GROUP VARCHAR(200) NOT NULL COMMENT 'Trigger group 名称',
    STR_PROP_1 VARCHAR(512) NULL COMMENT '字符串属性，占位用，下同',
    STR_PROP_2 VARCHAR(512) NULL,
    STR_PROP_3 VARCHAR(512) NULL,
    INT_PROP_1 INT NULL,
    INT_PROP_2 INT NULL,
    LONG_PROP_1 BIGINT NULL,
    LONG_PROP_2 BIGINT NULL,
    DEC_PROP_1 NUMERIC(13,4) NULL,
    DEC_PROP_2 NUMERIC(13,4) NULL,
    BOOL_PROP_1 VARCHAR(1) NULL,
    BOOL_PROP_2 VARCHAR(1) NULL,
    PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
    FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
    REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP))
ENGINE=InnoDB COMMENT '存储trigger属性信息';

CREATE TABLE QRTZ_PAUSED_TRIGGER_GRPS (
SCHED_NAME VARCHAR(120) NOT NULL COMMENT 'Scheduler名称',
TRIGGER_GROUP VARCHAR(200) NOT NULL COMMENT 'Trigger group 名称',
PRIMARY KEY (SCHED_NAME,TRIGGER_GROUP))
ENGINE=InnoDB COMMENT '存储已暂停的Trigger组的信息';

CREATE TABLE QRTZ_CALENDARS (
SCHED_NAME VARCHAR(120) NOT NULL COMMENT 'Scheduler名称',
CALENDAR_NAME VARCHAR(200) NOT NULL COMMENT 'Calendar 名称',
CALENDAR BLOB NOT NULL COMMENT 'Calendar 数据',
PRIMARY KEY (SCHED_NAME,CALENDAR_NAME))
ENGINE=InnoDB COMMENT '存储Quartz的Calendar信息';

CREATE TABLE QRTZ_JOB_DETAILS(
SCHED_NAME VARCHAR(120) NOT NULL COMMENT 'Scheduler名称',
JOB_NAME VARCHAR(200) NOT NULL COMMENT 'Job key',
JOB_GROUP VARCHAR(200) NOT NULL COMMENT 'Job group 名称',
DESCRIPTION VARCHAR(250) NULL COMMENT 'Job 描述， .withDescription()方法传入的string',
JOB_CLASS_NAME VARCHAR(250) NOT NULL COMMENT '实现Job的类名，trigger触发时调度此类的execute方法',
IS_DURABLE VARCHAR(1) NOT NULL COMMENT '为true时，Job相关的trigger完成以后，Job数据继续保留',
IS_NONCONCURRENT VARCHAR(1) NOT NULL COMMENT '是否不允许并发，为true时，如果下一次的触发事件到了，而上一次的job执行还未结束，则后续的触发会放入队列等待',
IS_UPDATE_DATA VARCHAR(1) NOT NULL COMMENT '是否在多次调度之间更新JobDataMap',
REQUESTS_RECOVERY VARCHAR(1) NOT NULL COMMENT 'Scheduler实例发生故障时，故障恢复节点会检测故障的Scheduler正在调度的任务是否需要recovery，如果需要会添加一个只执行一次的simple trigger重新触发',
JOB_DATA BLOB NULL COMMENT '存储JobDataMap等',
PRIMARY KEY (SCHED_NAME,JOB_NAME,JOB_GROUP))
ENGINE=InnoDB COMMENT '存储每一个已配置的Job的详细信息';


CREATE INDEX IDX_QRTZ_J_REQ_RECOVERY ON QRTZ_JOB_DETAILS(SCHED_NAME,REQUESTS_RECOVERY);
CREATE INDEX IDX_QRTZ_J_GRP ON QRTZ_JOB_DETAILS(SCHED_NAME,JOB_GROUP);

CREATE INDEX IDX_QRTZ_T_J ON QRTZ_TRIGGERS(SCHED_NAME,JOB_NAME,JOB_GROUP);
CREATE INDEX IDX_QRTZ_T_JG ON QRTZ_TRIGGERS(SCHED_NAME,JOB_GROUP);
CREATE INDEX IDX_QRTZ_T_C ON QRTZ_TRIGGERS(SCHED_NAME,CALENDAR_NAME);
CREATE INDEX IDX_QRTZ_T_G ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_GROUP);
CREATE INDEX IDX_QRTZ_T_STATE ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_STATE);
CREATE INDEX IDX_QRTZ_T_N_STATE ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP,TRIGGER_STATE);
CREATE INDEX IDX_QRTZ_T_N_G_STATE ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_GROUP,TRIGGER_STATE);
CREATE INDEX IDX_QRTZ_T_NEXT_FIRE_TIME ON QRTZ_TRIGGERS(SCHED_NAME,NEXT_FIRE_TIME);
CREATE INDEX IDX_QRTZ_T_NFT_ST ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_STATE,NEXT_FIRE_TIME);
CREATE INDEX IDX_QRTZ_T_NFT_MISFIRE ON QRTZ_TRIGGERS(SCHED_NAME,MISFIRE_INSTR,NEXT_FIRE_TIME);
CREATE INDEX IDX_QRTZ_T_NFT_ST_MISFIRE ON QRTZ_TRIGGERS(SCHED_NAME,MISFIRE_INSTR,NEXT_FIRE_TIME,TRIGGER_STATE);
CREATE INDEX IDX_QRTZ_T_NFT_ST_MISFIRE_GRP ON QRTZ_TRIGGERS(SCHED_NAME,MISFIRE_INSTR,NEXT_FIRE_TIME,TRIGGER_GROUP,TRIGGER_STATE);

CREATE INDEX IDX_QRTZ_FT_TRIG_INST_NAME ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,INSTANCE_NAME);
CREATE INDEX IDX_QRTZ_FT_INST_JOB_REQ_RCVRY ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,INSTANCE_NAME,REQUESTS_RECOVERY);
CREATE INDEX IDX_QRTZ_FT_J_G ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,JOB_NAME,JOB_GROUP);
CREATE INDEX IDX_QRTZ_FT_JG ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,JOB_GROUP);
CREATE INDEX IDX_QRTZ_FT_T_G ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP);
CREATE INDEX IDX_QRTZ_FT_TG ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,TRIGGER_GROUP);

commit;
```

## 配置
Spring的showcase项目中有Quartz集群化配置示例，可以根据项目需要做些调整。
```
#============================================================================
# Configure Main Scheduler Properties
#============================================================================
org.quartz.scheduler.instanceName = MyClusteredScheduler
org.quartz.scheduler.instanceId = AUTO
org.quartz.scheduler.skipUpdateCheck = true
#============================================================================
# Configure ThreadPool
#============================================================================
org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount = 25
org.quartz.threadPool.threadPriority = 5
#============================================================================
# Configure JobStore
#============================================================================
org.quartz.jobStore.misfireThreshold = 60000
org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate
org.quartz.jobStore.useProperties = false
org.quartz.jobStore.tablePrefix = QRTZ_
org.quartz.jobStore.isClustered = true
org.quartz.jobStore.clusterCheckinInterval = 20000
#============================================================================
# Configure DataSources
#============================================================================
#defined in applicationContext-quartz.xml
```
applicationContext-quartz.xml配置如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
                        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.0.xsd"
       default-lazy-init="false">
    <description>Quartz的定时集群任务配置</description>

    <bean id="quartzDataSource" class="org.apache.tomcat.jdbc.pool.DataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driver}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
    </bean>

    <!-- Quartz集群Scheduler -->
    <bean id="clusterQuartzScheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
        <!--  quartz配置文件路径-->
        <property name="configLocation" value="classpath:/quartz.properties" />
        <!-- 启动时延期3秒开始任务 -->
        <property name="startupDelay" value="2" />
        <property name="autoStartup" value="true" />
        <!-- 保存Job数据到数据库所需的数据源 -->
        <property name="dataSource" ref="quartzDataSource" />
        <!-- Job接受applicationContext的成员变量名 -->
        <property name="applicationContextSchedulerContextKey" value="applicationContext" />
        <property name="overwriteExistingJobs" value="true" />
    </bean>
</beans>
```

## 集群调度机制
集群调度机制部分，gklifg的博客<sup>[\[1\]](#ref1)</sup>中做了很细致的分析和说明，我在此直接引用部分内容。

![Quartz集群](http://img.blog.csdn.net/20140606160024250?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ2tsaWZn/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
Quartz的集群化配置是基于持久化的，需要使用JDBC等持久化到数据库的方式。因为Quartz集群中的各个节点并不感知其他节点的存在,只是通过数据库来进行间接的沟通。

![Quartz集群调度机制](http://img.blog.csdn.net/20140606160208906?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ2tsaWZn/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
一个调度器实例在执行涉及到分布式问题的数据库操作之前，需要先获取QRTZ_LOCKS表中相应的锁，获取到锁的实例可以进行相应的操作并更新相关数据的状态，事务提交以后，锁释放以供其他实例获取。

通过这样的方式避免多个实例同时修改数据可能引起的问题。

## 疑问
在单元测试的时候，我发现如果给Quartz设置了startupDelay，那么如果在Quartz开始调度任务之前有Job被添加到了调度器，且Quartz开始调度时恰好有符合触发条件的Trigger，那么最初的1秒内调度器会检测到3次待触发的Trigger，并分配给不同的Worker执行。目前我没有找到原因，也不排除是自己的配置或者测试环境存在问题，这个需要后续跟进观察。

## 总结
总体来说，Quartz是一个非常强大的作业调度框架，Trigger可以包含JobDataMap的设计方式更是让我眼前一亮，在taskgo的设计和开发过程中会有很多可以借鉴Quartz的地方。

## Update 2016-02-19
最近看到有一篇文章介绍当当开发的elastic-job项目，目标是无中心化的分布式定时调度框架。重写了Quartz的基于数据库的分布式功能，改用Zookeeper实现注册中心，代码已经在Github开源，有时间可以学习一下。

## reference
<span id="ref1">1. gklifg [quartz集群调度机制调研及源码分析](http://demo.netfoucs.com/gklifg/article/details/27090179) </span>
2. [Quartz应用与集群原理分析](http://tech.meituan.com/mt-crm-quartz.html)
3. [详解当当网的分布式作业框架elastic-job](http://www.infoq.com/cn/articles/dangdang-distributed-work-framework-elastic-job)