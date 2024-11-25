# 1. Yarn资源调度器

## 1.1 Yarn 基础架构
    YARN 主要由 ResourceManager、NodeManager、ApplicationMaster 和 Container 等组件构成。

![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/yarn_infrastructure.png "Lapland")

## 1.2 yarn 工作机制
![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/yarn_mechanism.png "Lapland")

1. MR 程序提交到客户端所在的节点。
2. YarnRunner 向 ResourceManager 申请一个 Application。
3. RM 将该应用程序的资源路径返回给 YarnRunner。
4. 该程序将运行所需资源提交到 HDFS 上。
5. 程序资源提交完毕后，申请运行 mrAppMaster。
6. RM 将用户的请求初始化成一个 Task。
7. 其中一个 NodeManager 领取到 Task 任务。
8. 该 NodeManager 创建容器 Container，并产生 MRAppmaster。
9. Container 从 HDFS 上拷贝资源到本地。
10. MRAppmaster 向 RM 申请运行 MapTask 资源。
11. RM 将运行 MapTask 任务分配给另外两个 NodeManager，另两个 NodeManager 分别领取任务并创建容器。
12. MR 向两个接收到任务的 NodeManager 发送程序启动脚本，这两个 NodeManager分别启动 MapTask，MapTask 对数据分区排序。
13. MrAppMaster 等待所有 MapTask 运行完毕后，向 RM 申请容器，运行 ReduceTask。
14. ReduceTask 向 MapTask 获取相应分区的数据。
15. 程序运行完毕后，MR 会向 RM 申请注销自己。


## 1.3 作业提交
![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/triadic_relation.png "Lapland")

作业提交过程之YARN

![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/yarn_JobSubmission.png "Lapland")

作业提交过程之HDFS & MapReduce

![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/yarn_MR_submission.png "Lapland")

作业提交全过程详解
1. 作业提交

    第 1 步：Client 调用 job.waitForCompletion 方法，向整个集群提交 MapReduce 作业。

    第 2 步：Client 向 RM 申请一个作业 id。

    第 3 步：RM 给 Client 返回该 job 资源的提交路径和作业 id。

    第 4 步：Client 提交 jar 包、切片信息和配置文件到指定的资源提交路径。

    第 5 步：Client 提交完资源后，向 RM 申请运行 MrAppMaster。

2. 作业初始化
   
    第 6 步：当 RM 收到 Client 的请求后，将该 job 添加到容量调度器中。

    第 7 步：某一个空闲的 NM 领取到该 Job。

    第 8 步：该 NM 创建 Container，并产生 MRAppmaster。

    第 9 步：下载 Client 提交的资源到本地。

3. 任务分配
   
    第 10 步：MrAppMaster 向 RM 申请运行多个 MapTask 任务资源。

    第 11 步：RM 将运行 MapTask 任务分配给另外两个 NodeManager，另两个 NodeManager分别领取任务并创建容器。

4. 任务运行
   
    第 12 步：MR 向两个接收到任务的 NodeManager 发送程序启动脚本，这两个NodeManager 分别启动 MapTask，MapTask 对数据分区排序。

    第13步：MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。

    第 14 步：ReduceTask 向 MapTask 获取相应分区的数据。

    第 15 步：程序运行完毕后，MR 会向 RM 申请注销自己。

5. 进度和状态更新
   
    YARN 中的任务将其进度和状态(包括 counter)返回给应用管理器, 客户端每秒(通过mapreduce.client.progressmonitor.pollinterval 设置)向应用管理器请求进度更新, 展示给用户。

6. 作业完成
   
    除了向应用管理器请求作业进度外, 客户端每 5 秒都会通过调用 waitForCompletion()来检查作业是否完成。时间间隔可以通过 mapreduce.client.completion.pollinterval 来设置。作业完成之后, 应用管理器和 Container 会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。

## 1.4 Yarn 调度器和调度算法
Apache Hadoop3.1.3 默认的资源调度器是 Capacity Scheduler。CDH 框架默认调度器是 Fair Scheduler。

### 1.4.1 先进先出调度器（FIFO）
FIFO 调度器（First In First Out）：单队列，根据提交作业的先后顺序，先来先服务。

![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/yarn_FIFO.png "Lapland")

**优点**：简单易懂；

**缺点**：不支持多队列，生产环境很少使用；

### 1.4.2 容量调度器（Capacity Scheduler）

**容量调度器特点**

![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/yarn_CapacityScheduler.png "Lapland")

1. 多队列：每个队列可配置一定的资源量，每个队列采用FIFO调度策略。
2. 容量保证：管理员可为每个队列设置资源最低保证和资源使用上限
3. 灵活性：如果一个队列中的资源有剩余，可以暂时共享给那些需要资源的队列，而一旦该队列有新的应用程序提交，则其他队列借调的资源会归还给该队列。
4. 多租户：
   
   支持多用户共享集群和多应用程序同时运行。

    为了防止同一个用户的作业独占队列中的资源，该调度器会对<font color='red'>同一用户提交的作业所占资源量进行限定。</font>

**容量调度器资源分配算法**

![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/yarn_CSTree.png "Lapland")

1. **队列资源分配**:
    从root开始，使用深度优先算法，<font color='red'>优先选择资源占用率最低</font>的队列分配资源。
2. **作业资源分配**:
    默认按照提交作业的<font color='red'>优先级</font>和<font color='red'>提交时间</font>顺序分配资源。
3. **容器资源分配**:
   
    按照容器的<font color='red'>优先级</font>分配资源；

    如果优先级相同，按照<font color='red'>数据本地性原则</font>：
    * 任务和数据在同一节点
    * 任务和数据在同一机架
    * 任务和数据不在同一节点也不在同一机架

### 1.4.3 公平调度器（Fair Scheduler）
同队列所有任务共享资源，在时间尺度上获得公平的资源

* **与容量调度器相同点**
  
  1. 多队列：支持多队列多作业
  2. 容量保证：管理员可为每个队列设置资源最低保证和资源使用上线
  3. 灵活性：如果一个队列中的资源有剩余，可以暂时共享给那些需要资源的队列，而一旦该队列有新的应用程序提交，则其他队列借调的资源会归还给该队列。
  4. 多租户：支持多用户共享集群和多应用程序同时运行；为了防止同一个用户的作业独占队列中的资源，该调度器会对同一用户提交的作业所占资源量进行限定。

* **与容量调度器不同点**
  1. 核心调度策略不同
   
        容量调度器：优先选择资源利用率低的队列

        公平调度器：优先选择对资源的缺额比例大的
  2. 每个队列可以单独设置资源分配方式
   
        容量调度器：FIFO、 DRF

        公平调度器：FIFO、FAIR、DRF

**公平调度器——缺额**

* 公平调度器设计目标是：在时间尺度上，所有作业获得公平的资源。某一时刻一个作业应获资源和实际获取资源的差距叫“缺额”
* 调度器会优先为缺额大的作业分配资源

**公平调度器队列资源分配方式**

**Fair 策略**（默认）是一种基于最大最小公平算法实现的资源多路复用方式，默认情况下，每个队列内部采用该方式分配资源。这意味着，如果一个队列中有两个应用程序同时运行，则每个应用程序可得到1/2的资源；如果三个应用程序同时运行，则每个应用程序可得到1/3的资源。

### 1.5 Yarn 常用命令
#### 1.5.1 yarn application 查看任务
1. 列出所有 Application：
   ```
   yarn application -list
   ```
2. 根据 Application 状态过滤：yarn application -list -appStates （所有状态：ALL、NEW、NEW_SAVING、SUBMITTED、ACCEPTED、RUNNING、FINISHED、FAILED、KILLED）
   ```
   yarn application -list -appStates FINISHED
   ```
3. Kill 掉 Application：
   ```
   yarn application -kill application_1612577921195_0001
   ```

#### 1.5.2 yarn logs 查看日志
1. 查询 Application 日志：yarn logs -applicationId <ApplicationId>
   ```
   yarn logs -applicationId application_1612577921195_0001
    ```
2. 查询 Container 日志：yarn logs -applicationId <ApplicationId> -containerId <ContainerId> 
   ```
   yarn logs -applicationId application_1612577921195_0001 -containerId container_1612577921195_0001_01_000001
   ```

#### 1.5.3 yarn applicationattempt 查看尝试运行的任务
1. 列出所有 Application 尝试的列表：yarn applicationattempt -list <ApplicationId>
    ```
    yarn applicationattempt -list application_1612577921195_0001
    ```
2. 打印 ApplicationAttemp 状态：yarn applicationattempt -status <ApplicationAttemptId>
   ```
   yarn applicationattempt -status appattempt_1612577921195_0001_000001
   ```

#### 1.5.4 yarn container 查看容器
1. 列出所有 Container：yarn container -list <ApplicationAttemptId>
   ```
   yarn container -list appattempt_1612577921195_0001_000001
   ```
2. 打印 Container 状态：yarn container -status <ContainerId>
    ```
    yarn container -status container_1612577921195_0001_01_000001
    ```

#### 1.5.5 yarn node 查看节点状态
1. 列出所有节点：yarn node -list -all
    ```
    yarn node -list -all
    ```
