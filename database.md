# [MySQL数据库面试题总结](https://blog.csdn.net/adminpd/article/details/122910606)

## 一、数据库设计

### 1. [六大范式](https://blog.csdn.net/weixin_43433032/article/details/89293663)

#### 1.1. 第一范式 1NF
> 属于第一范式关系的<font color='red'>所有属性都不可再分</font>，即数据项不可分。

第一范式强调数据表的<font color='red'>原子性</font>，是其他范式的基础。

**规范化**：一个低一级的关系模式通过模式分解可以转化为若干个高一级范式的关系模式的集合，这个过程叫做规范化。

#### 1.2. 第二范式 2NF
> 若某关系R属于第一范式，且<font color='red'>每一个非主属性完全函数依赖于任何一个候选码</font>，则关系R属于第二范式。

<font color='red'>**候选码**</font>： 若关系中的某一属性组的值能唯一地标识一个元组，而其子集不能，则称该属性组为候选码。若一个关系中有多个候选码，则选定其中一个为主码。

*例如在学生表中，学号和姓名都可以唯一标识一个元组，故该表的候选码为学号和姓名，主码我们可以随便选定其中一个，则选学号为主码。*

**主属性**：所有<font color='red'>候选码的属性称为主属性</font>。不包含在任何候选码中的属性称为非主属性或非码属性。

*学生表中，学号和姓名就是该关系的主属性，年龄和性别就是非主属性。*

**函数依赖**： 设R(U)是属性集U上的关系模式，X、Y是U的子集。若对于R(U)的任意一个可能的关系r，r中不可能存在两个元组在X上的属性值相等，而在Y上的属性值不等，则称Y函数依赖于X或X函数确定Y。

**完全函数依赖**： 设R(U)是属性集U上的关系模式，X、Y是U的子集。如果Y函数依赖于X，且对于X的任何一个真子集X’，都有Y不函数依赖于X’，则称Y对X完全函数依赖。记作：如果Y函数依赖于X，但Y不完全函数依赖于X，则称Y对X部分函数依赖。

![Lapland](https://raw.githubusercontent.com/ww-1009/interview/main/img/database/functional_dependency.png "Lapland")

<font color='red'>**判断一个关系是否属于第二范式**</font>：

1. 找出数据表中的所有码；
2. 找出所有主属性和非主属性；
3. 判断所有的非主属性对码的部分函数依赖。

#### 1.3. 第三范式 3NF
> 非主属性既不传递依赖于码，也不部分依赖于码。

**理解**： 第三范式要求在满足第二范式的基础上，任何非主属性不依赖于其他非主属性，即在第二范式的基础上，消除了传递依赖。

#### 1.4. BC范式 BCFN
> 关系模式R<U,F>中，若每一个决定因素都包含码，则R<U,F>属于BCFN。

**理解**： 根据定义我们可以得到结论，一个满足BC范式的关系模式有：

1. 所有非主属性对每一个码都是完全函数依赖；
2. 所有主属性对每一个不包含它的码也是完全函数依赖；
3. 没有任何属性完全函数依赖于非码的任何一组属性。

*例如有关系模式C(Cno, Cname, Pcno)，Cno, Cname, Pcno依次表示课程号、课程名、选修课。可知关系C只有一个码Cno，且没有任何属性对Cno部分函数依赖或传递函数依赖，所以关系C属于第三范式，同时Cno是C中的唯一决定因素，所以C也属于BC范式。*

#### 1.5. 第四范式 4NF
>限制关系模式的属性之间不允许有非平凡且非函数依赖的多值依赖。

**理解**： 显然一个关系模式是4NF，则必为BCNF。也就是说，当一个表中的非主属性互相独立时（3NF），这些非主属性不应该有多值，若有多值就违反了4NF。

#### 1.6. 第五范式 5NF

1. 必须满足第四范式；
2. 表必须可以分解为较小的表，除非那些表在逻辑上拥有与原始表相同的主键。

### 2. 什么是范式和反范式？

|  名称  |                                     优点                                     |                     缺点                     |
| :----: | :--------------------------------------------------------------------------: | :------------------------------------------: |
|  范式  |         范式化的表减少了数据冗余，数据表更新操作快、占用存储空间少。         | 查询时通常需要多表关联查询，更难进行索引优化 |
| 反范式 | 反范式的过程就是通过冗余数据来提高查询性能，可以减少表关联和更好进行索引优化 |   存在大量冗余数据，并且数据的维护成本更高   |

## 二、索引

> **索引**是对数据库表中一列或多列的值进行排序的数据结构，用于快速访问数据库表中的特定信息。

### 1. 索引分类

1) 从物理结构分为：
   
   * **聚簇索引**指索引的键值的逻辑顺序与表中相应行的物理顺序一致，即每张表只能有一个聚簇索引，也就是我们常说的**主键索引**；
   * **非聚簇索引**的逻辑顺序则与数据行的物理顺序不一致。

2) 从应用上分为：
   
   * **普通索引**：MySQL 中的基本索引类型，没有什么限制，允许在定义索引的列中插入重复值和空值，纯粹为了提高查询效率。通过 <font color='red'>`ALTER TABLE table_name ADD INDEX index_name (column)`</font> 创建；
   * **唯一索引**：索引列中的值必须是唯一的，但是允许为空值。通过 <font color='red'>`ALTER TABLE table_name ADD UNIQUE index_name (column)`</font> 创建；
   * **主键索引**：特殊的唯一索引，也成聚簇索引，不允许有空值，并由数据库帮我们自动创建；
   * **组合索引**：组合表中多个字段创建的索引，遵守最左前缀匹配规则；
   * **全文索引**：只有在 MyISAM 引擎上才能使用，同时只支持 CHAR、VARCHAR、TEXT 类型字段上使用。

### 2. 索引优缺点

**优点**

* 通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性
* 加快数据的检索速度
* 加速表和表之间的连接
* 使用分组和排序子句进行数据检索时

**缺点**

* 创建和维护索引需要耗费时间，降低了数据的维护速度
* 索引需要占物理空间

### 3. 索引设计原则

* 选择唯一性索引
* 为常作为查询条件的字段建立索引
* 为经常需要排序、分组和联合操作的字段建立索引
* 限制索引的数目
* 小表不建议索引（如数量级在百万以内）
* 尽量使用数据量少的索引
* 删除不再使用或者很少使用的索引

### 4. 索引的数据结构

MySQL常用 **Hash** 和 **B+ 树**索引

* **Hash 索引**底层就是 <font color='red'>Hash 表</font>，进行查询时调用 Hash 函数获取到相应的键值（对应地址），然后回表查询获得实际数据.

* **B+ 树索引**底层实现原理是<font color='red'>多路平衡查找树</font>，对于每一次的查询都是从根节点出发，查询到叶子节点方可以获得所查键值，最后查询判断是否需要回表查询.

#### 4.1. Hash 和 B+ 树索引的区别
**Hash**

* Hash 进行等值查询更快，但<font color='red'>无法进行范围查询</font>，也不支持使用索引进行排序
* Hash <font color='red'>不支持模糊查询</font>以及多列索引的最左前缀匹配，因为 Hash 函数的值不可预测
* Hash 任何时候都避免不了回表查询数据
* 性能不稳定，因为当某个键值存在大量重复时，产生 Hash 碰撞

**B+ Tree**

* <font color='red'>支持范围查询和排序</font>
* 在符合某些条件（聚簇索引、覆盖索引等）时候可以只通过索引完成查询，不需要回表
* 查询效率比较稳定，因为每次查询都是从根节点到叶子节点，且为树的高度

#### 4.2. 为何使用 B+ 树而非 B 树做索引

**B+ 树和 B 树的区别：**

* <font color='red'>B 树非叶子结点和叶子结点都存储数据</font>，因此查询数据时，时间复杂度最好为 O(1)，最坏为 O(log n)。而 <font color='red'>B+ 树只在叶子结点存储数据，非叶子结点存储关键字</font>，且不同非叶子结点的关键字可能重复，因此查询数据时，时间复杂度固定为 O(log n)
* <font color='red'>B+ 树叶子结点之间用链表相互连接</font>，因而只需扫描叶子结点的链表就可以完成一次遍历操作，B 树只能通过中序遍历

## 三、事务

> 数据库的**事务**是一个<font color='red'>不可分割</font>的数据库操作序列，也是<font color='red'>数据库并发控制的基本单位</font>，其执行的结果必须使数据库从一种一致性状态变到另一种一致性状态。事务是逻辑上的一组操作，要么都执行，要么都不执行。

### 1. 四大特征（ACID）

* **原子性**： 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用
* **一致性**： 事务执行前后，数据保持一致，多个事务对同一个数据读取的结果是相同的
* **隔离性**： 并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的
* **持久性**： 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。
  

### 2. 事务的并发问题

* **脏读：一个事务读取到另一个事务尚未提交的数据。** 事务 A 读取事务 B 更新的数据，然后 B 回滚操作，那么 A 读取到的数据是脏数据。

* **不可重复读：一个事务中两次读取的数据的内容不一致。** 事务 A 多次读取同一数据，事务 B 在事务 A 多次读取的过程中，对数据作了更新并提交，导致事务 A 多次读取同一数据时，结果 不一致。

* **幻读：一个事务中两次读取的数据量不一致。** 系统管理员 A 将数据库中所有学生的成绩从具体分数改为 ABCDE 等级，但是系统管理员 B 就在这个时候插入了一条具体分数的记录，当系统管理员 A 改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。

    <font color='red'>*不可重复读侧重于修改，幻读侧重于新增或删除*</font>

### 3. 四种隔离级别

* Read Uncommitted（读取未提交内容）
   > 在该隔离级别，所有事务都可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其他级别好多少。读取未提交的数据，也被称之为脏读（Dirty Read）。
* Read Committed（读取提交内容）
   > 这是大多数数据库系统的默认隔离级别（但不是 MySQL 默认的）。它满足了隔离的简单定义：**一个事务只能看见已经提交事务所做的改变**。这种隔离级别也支持所谓的不可重复读（Nonrepeatable Read），因为同一事务的其他实例在该实例处理其间可能会有新的 commit，所以同一 select 可能返回不同结果。
* Repeatable Read（可重读）
   >这是 MySQL 的默认事务隔离级别，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。不过理论上，这会导致另一个棘手的问题：幻读 （Phantom Read）。
* Serializable（可串行化）
   > 通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争。

|     隔离级别     | 脏读  | 不可重复读 | 幻影读 |
| :--------------: | :---: | :--------: | :----: |
| Read Uncommitted |   √   |     √      |   √    |
|  Read Committed  |   ×   |     √      |   √    |
| Repeatable Read  |   ×   |     ×      |   √    |
|   Serializable   |   ×   |     ×      |   ×    |

事务隔离机制的实现基于**锁机制**和**并发调度**。其中并发调度使用的是MVVC（多版本并发控制），通过保存修改的旧版本信息来支持并发一致性读和回滚等特性。

## 四、存储
### 1. [数据库引擎](https://blog.csdn.net/qq_21531681/article/details/108902419)

> 数据库存储引擎是数据库底层软件组织，数据库管理系统（DBMS）使用数据引擎进行创建、查询、更新和删除数据。不同的存储引擎提供不同的存储机制、索引技巧、锁定水平等功能，使用不同的存储引擎，还可以 获得特定的功能。现在许多不同的数据库管理系统都支持多种不同的数据引擎。MySQL**的核心就是存储引擎**。

#### 1.1. InnoDB存储引擎

InnoDB是事务型数据库的首选引擎，**支持事务安全表（ACID），支持行锁定和外键**，InnoDB是默认的MySQL引擎。InnoDB主要特性有：

* **InnoDB给MySQL提供了具有提交、回滚和崩溃恢复能力的事物安全（ACID兼容）存储引擎。**
* InnoDB是为处理巨大数据量的最大性能设计。
* InnoDB存储引擎完全与MySQL服务器整合，InnoDB存储引擎为在主内存中缓存数据和索引而维持它自己的缓冲池。
* **InnoDB支持外键完整性约束**
* 存储表中的数据时，每张表的存储都按主键顺序存放，如果没有显示在表定义时指定主键，InnoDB会为每一行生成一个6字节的ROWID，并以此作为主键

#### 1.2. MyISAM存储引擎

MyISAM基于ISAM存储引擎，并对其进行扩展。它是在Web、数据仓储和其他应用环境下最常使用的存储引擎之一。MyISAM拥有较高的插入、查询速度，但<font color='red'>不支持事物</font>。MyISAM主要特性有：

* 当把删除和更新及插入操作混合使用的时候，动态尺寸的行产生更少碎片。

#### 1.3. MEMORY存储引擎

MEMORY存储引擎将表中的数据存储到内存中，未查询和引用其他表数据提供快速访问。MEMORY主要特性有：

* MEMORY存储引擎执行HASH和BTREE缩影
* 可以在一个MEMORY表中有非唯一键值
* MEMORY表使用一个固定的记录长度格式
* MEMORY表内存被存储在内存中，内存是MEMORY表和服务器在查询处理时的空闲中，创建的内部表共享
* 当不再需要MEMORY表的内容时，要释放被MEMORY表使用的内存，应该执行DELETE FROM或TRUNCATE TABLE，或者删除整个表（使用DROP TABLE）

*如果要提供提交、回滚、崩溃恢复能力的事物安全（ACID兼容）能力，并要求实现并发控制，**InnoDB**是一个好的选择*

*如果数据表主要用来插入和查询记录，则**MyISAM**引擎能提供较高的处理效率*

*如果只是临时存放数据，数据量不大，并且不需要较高的数据安全性，可以选择将数据保存在内存中的**Memory**引擎，MySQL中使用该引擎作为临时表，存放查询的中间结果*

*如果只有INSERT和SELECT操作，可以选择Archive，Archive支持高并发的插入操作，但是本身不是事务安全的。Archive非常适合存储归档数据，如记录日志信息可以使用Archive*

#### 1.4. [MyISAM 和 InnoDB 的区别](https://blog.csdn.net/qq_35642036/article/details/82820178)

* InnoDB 支持事务，而 MyISAM 不支持。
* InnoDB 支持外键，而 MyISAM 不支持。因此将一个含有外键的 InnoDB 表 转为 MyISAM 表会失败。
* InnoDB 和 MyISAM 均支持 B+ Tree 数据结构的索引。但 InnoDB 是聚集索引，而 MyISAM 是非聚集索引。
* InnoDB 不保存表中数据行数，执行 `select count(*) from table` 时需要全表扫描。而 MyISAM 用一个变量记录了整个表的行数，速度相当快（注意不能有 WHERE 子句）。

   <font color='red'>*因为InnoDB的事务特性，在同一时刻表中的行数对于不同的事务而言是不一样的。*</font>
* InnoDB 支持表、行（默认）级锁，而 MyISAM 支持表级锁。

  <font color='red'>*InnoDB 的行锁是基于索引实现的，而不是物理行记录上。即访问如果没有命中索引，则也无法使用行锁，将要退化为表锁。*</font>

* InnoDB 必须有唯一索引（如主键），如果没有指定，就会自动寻找或生产一个隐藏列 Row_id 来充当默认主键，而 Myisam 可以没有主键。
* Innodb 不支持全文索引，而 MyISAM 支持全文索引，查询效率上 MyISAM 要高；

###  2. 存储结构

**页（Page）：**

>InnoDB 将物理磁盘划分为**页（page）**，每页的大小默认为 16 KB，**页是最小的存储单位**。页根据上层应用的需要，如索引、日志等，分为很多的格式。我们主要说**数据页**，也就是存储实际数据的页。

**区（Extent）：**

> 如果只有页这一个层次的话，页的个数是非常多的，存储空间的分配和回收都会很麻烦，因为要维护这么多的页的状态是非常麻烦的。
> 所以，InnoDB 又引入了**区（Extent)** 的概念。一个区默认是 64 个连续的页组成的，也就是 1MB。通过 Extent 对存储空间的分配和回收就比较容易了。

**段（Segment）：**

>B+ 树的叶子节点存放的是我们的具体数据，非叶子结点是索引页。所以 B+ 树将数据分为了两部分，叶子节点部分和非叶子节点部分，也就我们要介绍的段 Segment，也就是说 InnoBD 中每一个索引都会创建两个 Segment 来存放对应的两部分数据。

>Segment 是一种逻辑上的组织，其层次结构从上到下一次为 Segment、Extent、Page。

## 其他知识

### [SQL约束](https://blog.csdn.net/qq_43551373/article/details/87865739)

>SQL 约束用于规定表中的数据规则。如果存在违反约束的数据行为，行为会被约束终止。

* **非空约束(NOT NULL)** - 指示某列不能存储 NULL 值。
* **唯一约束(UNIQUE)** - 保证某列的每行必须有唯一的值。
* **主键约束(PRIMARY KEY)** - NOT NULL 和 UNIQUE 的结合。确保某列（或两个列多个列的结合）有唯一标识，有助于更容易更快速地找到表中的一个特定的记录。

   <font color='red'>*请注意，每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束。*</font>

* **外键约束(FOREIGN KEY)** - 保证一个表中的数据匹配另一个表中的值的参照完整性。
* **检查约束(CHECK)** - 保证列中的值符合指定的条件。
* **默认约束(DEFAULT)** - 规定没有给列赋值时的默认值。

### [聚簇索引和非聚簇索引](https://zhuanlan.zhihu.com/p/528069989)

#### 聚簇索引

* 聚簇索引（Clustered Index）一般指的是主键索引（如果存在主键索引的话），聚簇索引也被称之为聚集索引。

* 聚簇索引在 InnoDB 中是使用 B+ 树实现的。在聚簇索引的**叶子节点**直接存储用户信息的**内存地址**，我们使用内存地址可以直接找到相应的行数据。

#### 非聚簇索引

* 非聚簇索引在 InnoDB 引擎中，也叫二级索引

* 在非聚簇索引的**叶子节点**上存储的并不是真正的行数据，而是**主键 ID**，所以当我们使用非聚簇索引进行查询时，首先会得到一个主键 ID，然后再使用主键 ID 去聚簇索引上找到真正的行数据，我们把这个过程称之为回表查询。

#### 总结

* 聚簇索引叶子节点存储的是行数据；而非聚簇索引叶子节点存储的是聚簇索引（通常是主键 ID）。
* 聚簇索引查询效率更高，而非聚簇索引需要进行回表查询，因此性能不如聚簇索引。
* 聚簇索引一般为主键索引，而主键一个表中只能有一个，因此聚簇索引一个表中也只能有一个，而非聚簇索引则没有数量上的限制。

### varchar 和 char

* char 是一个定长字段；varchar 是变长的
* 检索效率上来讲,char > varchar
