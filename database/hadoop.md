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


## 2. HDFS组成架构
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