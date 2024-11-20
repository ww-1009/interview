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
