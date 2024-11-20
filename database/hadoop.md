# Hadoop

## 1. Hadoop组成
![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/hadoop_maked.png "Lapland")

### 1.1 HDFS架构概述
* **NameNode(nn)**：存储文件的元数据，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等
* **DataNode(dn)**：在本地文件系统存储文件块数据，以及块数据的校验和。
* **DataNode(dn)**：在本地文件系统存储文件块数据，以及块数据的校验和。

### 1.2 YARN架构概述
![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/yarn_maked.png "Lapland")

### 1.3 MapReduce架构概述
MapReduce 将计算过程分为两个阶段：**Map** 和 **Reduce**

1. Map 阶段并行处理输入数据
2. Reduce 阶段对 Map 结果进行汇总

### 1.4 HDFS、YARN、MapReduce 三者关系
![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/triadic_relation.png "Lapland")

### 1.5 集群部署规划
* NameNode 和 SecondaryNameNode 不要安装在同一台服务器
* ResourceManager 也很消耗内存，不要和 NameNode、SecondaryNameNode 配置在
同一台机器上。

![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/deployment_planning.png "Lapland")

## 2. HDFS
### 2.1 HDFS组成架构
![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/HDFS_maked.png "Lapland")

**Client**：就是客户端。

（1）文件切分。文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行上传；

（2）与NameNode交互，获取文件的位置信息；

（3）与DataNode交互，读取或者写入数据；

（4）Client提供一些命令来管理HDFS，比如NameNode格式化；

（5）Client可以通过一些命令来访问HDFS，比如对HDFS增删查改操作；

**Secondary NameNode**：并非NameNode的热备。当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务。

（1）辅助NameNode，分担其工作量，比如定期合并Fsimage和Edits，并推送给NameNode ；

（2）在紧急情况下，可辅助恢复NameNode。

### 2.2 HDFS文件块大小
HDFS中的文件在物理上是分块存储（Block），块的大小可以通过配置参数 (dfs.blocksize）来规定，<font color='red'>默认大小在Hadoop2.x/3.x版本中是128M，1.x版本中是64M</font>。

![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/dfs_blocksize.png "Lapland")

**思考：为什么块的大小不能设置太小，也不能设置太大？**

（1）HDFS的块设置**太小**，<font color='red'>会增加寻址时间</font>，程序一直在找块的开始位置；

（2）如果块设置的**太大**，从<font color='red'>磁盘传输数据的时间</font>会明显<font color='red'>大于定位这个块开始位置所需的时间</font>。导致程序在处理这块数据时，会非常慢。

<font color='red'>**总结：HDFS块的大小设置主要取决于磁盘传输速率。**</font>

### 2.3 HDFS的Shell操作
hadoop fs 具体命令 OR hdfs dfs 具体命令两个是完全相同的。

#### 2.3.1 上传
1. -moveFromLocal：从本地**剪切**粘贴到 HDFS
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ vim shuguo.txt
    输入：
    shuguo

    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -moveFromLocal ./shuguo.txt /sanguo
    ```
2. -copyFromLocal：从本地文件系统中拷贝文件到 HDFS 路径去
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ vim weiguo.txt
    输入：
    weiguo

    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -copyFromLocal weiguo.txt /sanguo
    ```
3. -put：等同于 copyFromLocal，生产环境更习惯用 put
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ vim wuguo.txt
    输入：
    wuguo

    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -put ./wuguo.txt /sanguo
    ```
4. -appendToFile：追加一个文件到已经存在的文件末尾
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ vim liubei.txt
    输入：
    liubei

    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -appendToFile liubei.txt /sanguo/shuguo.txt
    ```

#### 2.3.2 下载
1. -copyToLocal：从 HDFS 拷贝到本地
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -copyToLocal 
    /sanguo/shuguo.txt ./
    ```

2. -get：等同于 copyToLocal，生产环境更习惯用 get
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -get 
    /sanguo/shuguo.txt ./shuguo2.txt
    ```

#### 2.3.3 直接操作
1. -ls: 显示目录信息
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -ls /sanguo
    ```

2. -cat：显示文件内容
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -cat /sanguo/shuguo.txt
    ```
3. -chgrp、-chmod、-chown：Linux 文件系统中的用法一样，修改文件所属权限
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -chmod 666 
    /sanguo/shuguo.txt
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -chown atguigu:atguigu 
    /sanguo/shuguo.txt
    ```
4. -mkdir：创建路径
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -mkdir /jinguo
    ```
5. -cp：从 HDFS 的一个路径拷贝到 HDFS 的另一个路径
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -cp /sanguo/shuguo.txt /jinguo
    ```

6. -mv：在 HDFS 目录中移动文件
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -mv /sanguo/wuguo.txt /jinguo
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -mv /sanguo/weiguo.txt /jinguo
    ```
7. -tail：显示一个文件的末尾 1kb 的数据
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -tail /jinguo/shuguo.txt
    ```
8. -rm：删除文件或文件夹
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -rm /sanguo/shuguo.txt
    ```
9. -rm -r：递归删除目录及目录里面内容
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -rm -r /sanguo
    ```
10. -du 统计文件夹的大小信息
    ```
    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -du -s -h /jinguo
    27 81 /jinguo

    [atguigu@hadoop102 hadoop-3.1.3]$ hadoop fs -du -h /jinguo
    14 42 /jinguo/shuguo.txt
    7 21 /jinguo/weiguo.txt
    6 18 /jinguo/wuguo.tx
    ```
    说明：27 表示文件大小；81 表示 27*3 个副本；/jinguo 表示查看的目录

11. -setrep：设置 HDFS 中文件的副本数量
    
    这里设置的副本数只是记录在 NameNode 的元数据中，是否真的会有这么多副本，还得看 DataNode 的数量。因为目前只有 3 台设备，最多也就 3 个副本，只有节点数的增加到 10台时，副本数才能达到 10。

### 2.4 HDFS 的读写流程
#### 2.4.1 HDFS 写数据流程

![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/HDFS_write.png "Lapland")

1. 客户端通过 Distributed FileSystem 模块向 NameNode 请求上传文件，NameNode 检查目标文件是否已存在，父目录是否存在。
2. NameNode 返回是否可以上传。
3. 客户端请求第一个 Block 上传到哪几个 DataNode 服务器上。
4. NameNode 返回 3 个 DataNode 节点，分别为 dn1、dn2、dn3。
5. 客户端通过 FSDataOutputStream 模块请求 dn1 上传数据，dn1 收到请求会继续调用dn2，然后 dn2 调用 dn3，将这个通信管道建立完成。
6. dn1、dn2、dn3 逐级应答客户端。
7. 客户端开始往 dn1 上传第一个 Block（先从磁盘读取数据放到一个本地内存缓存），以 Packet 为单位，dn1 收到一个 Packet 就会传给 dn2，dn2 传给 dn3；dn1 <font color='red'>每传一个 packet会放入一个应答队列等待应答</font>。
8. 当一个 Block 传输完成之后，客户端再次请求 NameNode 上传第二个 Block 的服务器。（重复执行 3-7 步）。

#### 2.4.2 HDFS 读数据流程

![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/hadoop/HDFS_read.png "Lapland")

1.客户端通过 DistributedFileSystem 向 NameNode 请求下载文件，NameNode 通过查询元数据，找到文件块所在的 DataNode 地址。
2. 挑选一台 DataNode（就近原则，然后随机）服务器，请求读取数据。
3. DataNode 开始传输数据给客户端（从磁盘里面读取数据输入流，以 Packet 为单位来做校验）。
4. 客户端以 Packet 为单位接收，先在本地缓存，然后写入目标文件。

### 2.5 NameNode 和 SecondaryNameNode
