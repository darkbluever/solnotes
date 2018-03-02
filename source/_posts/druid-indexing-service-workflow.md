title: "Druid indexing-service 执行流程梳理"
date: 2018-02-02 20:31:12
categories:
- druid
tags:
- druid
---

最近在开发、测试 Druid 的 Spark-Batch-Indexing 扩展的时候遇到一点问题：提交离线导入任务以后，overlord 节点的 api 都处于404的状态，无法响应正常请求。为了定位问题，梳理了一下 Druid 的 indexing-service 执行流程，记录如下。

<!-- more -->

## Overlord

Overlord 节点负责接收任务，协调和分配任务，为任务创建锁，并返回任务状态给任务发送方。

Overlord 节点在启动后，会初始化一个 event loop（ TaskQueue.start ），TaskQueue 是 Task 生产者和消费者之间的接口，接收的 Task 会基本按照 FIFO 的顺序消费。我们提交一个任务的时候，会发送到 `http://<overlord_ip>:<overlord_port>/druid/indexer/v1/task`，OverlordResource.taskPost()会处理这个请求，将其写入 TaskQueue。TaskQueue 在执行事件循环主体 manage 的时候，会获取并消费这个任务，在判断 taskIsReady 后，调用RemoteTaskRunner.run() 方法执行 task。后续循环的时候，会判断这个任务是否已经在执行了，以及是否需要 shutdown。

RemoteTaskRunner 在执行 run 方法的时候，会判断接收到的任务的状态， 如果已经在执行了，就直接返回 Future，否则会执行 addPendingTask 方法，加入 PendingTasks Queue，然后触发 runPendingTasks 方法。在 runPendingTasks 方法中，会通过多线程的方式，从 PendingTasks Queue (ConcurrentSkipListMap) 获取 task，然后尝试指派给 worker 执行。此处会先从tryAssignTasks ( ConcurrentMap ) 获取锁，然后根据 worker-select-strategy 选择 worker，然后尝试获取 worker 的锁，并在 ZK 中声明。然后将 task 从 PendingTasks Queue 中移除，写入 RunningTasks Queue，并发出 task status changed 的通知，然后和 ZK 同步 task 状态，在这个 task 开始实际执行之前，不再指派另外的 task，避免超出 worker 的 capacity。



## MiddleManager

middle manager 节点是执行提交任务的工作节点。middle manager 将任务分发到 peons 运行，一个 peon 在一个单独的 jvm 中运行。原因是我们通过单独的 jvm 对任务做资源隔离和日志隔离。一个 peon 在一个时间只能运行一个任务，然而，一个 middle manager 可以管理多个 peon。

middle manager 节点启动后，会初始化一个 Monitor ( WorkerTaskMonitor.start ）, WorkerTaskMonitor 启动时，会注册一些 Listener，包括 RunListener ( ZK )，LocationListener ( TaskRunner )。在有新任务创建、任务状态变更等事件时，会触发这些 Listener 的监听事件，然后会创建相应的 Notice，写入本地的 notices (BlockingQueue)，WorkerTaskMonitor 的 mainLoop 方法会消费 notices 中的事件。

在有新任务创建以后，WorkerTaskMonitor 会创建 RunNotice，消费时，会先将 task 的状态更新为 running，然后调用 ForkingTaskRunner.run() 方法执行任务，并增加 FutureCallback，在任务执行成功或失败后，创建相应的 StatusNotice。ForkingTaskRunner.run() 执行时，会构造一个 java 命令，将 peon 放到一个单独的 jvm 中运行，并保持一个 ProcessHolder 以持有 Process Object，和其输出的 File Object，便于后续使用。在 peon 执行时，ForkingTaskRunner 会同步 peon 的输出，当 peon 执行结束后，根据输出判断执行状态，更新任务状态，销毁 peon 运行所用的资源，释放端口。
