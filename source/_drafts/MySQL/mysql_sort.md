# 基础整理

基本上，对于数据库的基本认识，都是围绕着索引、锁、事物来展开的。如何确保自身是否算是对数据库有了基础的掌握，可做如下的回顾（MySQL）：

> 1. 能够正常的手写DDL、DML
> 2. 对
> 3. 对索引的理解
>    1. 索引数据结构
>       1. 哈希索引
>       2. B+树索引
>       3. 倒排索引
>    2. 索引类型
>       1. 物理实现
>          1. 聚集索引
>          2. 非聚集索引
>       2. 功能实现
>          1. 唯一索引
>          2. 普通索引
>          3. 联合索引
>          4. 前缀索引
>             1. 主要是针对于字符串
>          5. 覆盖索引
>    3. 索引失效分析
> 4. 对事物的理解及具体实现
>    1. 原子性
>       * redo log
>    2. 一致性
>       * undo log
>    3. 隔离型
>       * 锁
>    4. 持久性
>       * redo log
> 5. 分库分表
>    1. 分片策略
>    2. 分布式主键
>    3. 跨库join
> 6. 性能调优
>    1. 慢查询日志
>    2. 执行计划
>    3. SQL分析
>    4. SQL优化

* [x] 索引
* [ ] 锁
* [ ] 事物
* [ ] 分库分表
* [ ] 性能调优

## 数据库的范式

| Sno  | Sname  | Sdept  | Mname  | Cname  | Grade |
| :--: | :----: | :----: | :----: | :----: | :---: |
|  1   | 学生-1 | 学院-1 | 院长-1 | 课程-1 |  90   |
|  2   | 学生-2 | 学院-2 | 院长-2 | 课程-2 |  80   |
|  2   | 学生-2 | 学院-2 | 院长-2 | 课程-1 |  100  |
|  3   | 学生-3 | 学院-2 | 院长-2 | 课程-2 |  95   |

不符合范式的关系，会产生很多异常，主要有以下四种异常：

- 冗余数据：例如 `学生-2` 出现了两次。
- 修改异常：修改了一个记录中的信息，但是另一个记录中相同的信息却没有被修改。
- 删除异常：删除一个信息，那么也会丢失其它信息。例如删除了 `课程-1` 需要删除第一行和第三行，那么 `学生-1` 的信息就会丢失。
- 插入异常：例如想要插入一个学生的信息，如果这个学生还没选课，那么就无法插入。

范式理论是为了解决以上提到四种异常。

高级别范式的依赖于低级别的范式，1NF 是最低级别的范式。

- 超键：能唯一标识元组的属性集叫做超键。
- 候选键：如果超键不包括多余的属性，那么这个超键就是候选键。
- 主键：用户可以从候选键中选择一个作为主键。
- 外键：如果数据表 R1 中的某属性集不是 R1 的主键，而是另一个数据表 R2 的主键，那么这个属性集就是数据表 R1 的外键。
- 主属性：包含在任一候选键中的属性称为主属性。
- 非主属性：与主属性相对，指的是不包含在任何一个候选键中的属性。

> 超键和候选件均属于一个集合，可包含多个属性也可包含一个属性，但是主键，仅仅是集合中的一个元素（可多个属性，可单个属性，能代表这条记录即可）

### 1NF

**1NF 指的是数据库表中的任何属性都是原子性的，不可再分**。这很好理解，我们在设计某个字段的时候，对于字段 X 来说，就不能把字段 X 拆分成字段 X-1 和字段 X-2。事实上，任何的 DBMS 都会满足第一范式的要求，不会将字段进行拆分。

### 2NF

**2NF 指的数据表里的非主属性都要和这个数据表的候选键有完全依赖关系**。==所谓完全依赖不同于部分依赖，也就是不能仅依赖候选键的一部分属性，而必须依赖其全部属性==。

### 3NF

**3NF在满足2NF的基础上, 对任何非主属性都不传递依赖于主属性**, 也就是说不能存在非主属性 A 依赖于非主属性 B，非主属性 B 依赖于候选键的情况。

我们用球员 player 表举例子，这张表包含的属性包括球员编号、姓名、球队名称和球队主教练。现在，我们把属性之间的依赖关系画出来，如下图所示：

![img](data:image/jpeg;base64,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)
你能看到球员编号决定了球队名称，同时球队名称决定了球队主教练，非主属性球队主教练就会传递依赖于球员编号，因此不符合 3NF 的要求。

如果要达到 3NF 的要求，需要把数据表拆成下面这样：

球员表的属性包括球员编号、姓名和球队名称；球队表的属性包括球队名称、球队主教练。

### 总结

1NF 需要保证表中每个属性都保持原子性；2NF 需要保证表中的非主属性与候选键完全依赖（需要主键唯一标识一行记录）；3NF 需要保证表中的非主属性与候选键不存在传递依赖。

## 索引

此处仅做对索引的宏观阐述，详细的讲解，具体查阅[索引整理](./mysql)

### 索引类型

1. hash索引
   1. 底层为hash表
2. B+树索引
   1. 底层为B+树
   2. 演变
      1. 有序数组
      2. 二叉搜索树
      3. 平衡二叉搜索树
      4. B树
      5. B+树
         1. 为磁盘或其他直接存取辅助设备设计的一种平衡查找树，在B+树中，所有记录节点都是按照键值的大小顺序存放在同一层的叶子结点上
         2. 一个节点内有多个数据集，每个节点大小为一页，页内都通过一个双向链表进行链接
3. 全文索引



### B+树索引

可以详细分为**聚集索引（聚簇索引）**和**辅助索引（非聚集索引）**。叶子中存放所有的数据，但是

1. 聚集索引叶子节点存放的是一整条记录
2. 非聚集索引存储的是主键的id

### 聚集索引



### 辅助索引(非聚集索引)

**查询方式：**

1. 先通过辅助索引找到对应的主键
2. 根据主键去聚集索引上寻找完整的记录

**示例：**

假设一颗高度为3的辅助索引中查找数据。

1. 在辅助索引树遍历3次找到指定主键
2. 之后在主键索引上还需要遍历3次找到最终需要的记录

#### 索引的选择

应该选择高选择性的字段作为索引

```sql
count(distinct column) / count(*) 近似 1 
show index from table
-- cardinality值 表示索引中不重复记录数量的预估值
```



### 联合索引

对表上多个列进行索引。

譬如建立了多个一个(a,b)索引

1. where a = xxx and b = yyy 可以走索引
2. where a = xxx 可以走索引
3. where b = yyy 不能走该索引

联合索引对第二个键值进行了排序处理。这样方便一些场景的使用

1. 查询用户购物情况，根据user_id（主键）查找出用户的消费记录，然后再根据buy_date进行排序，取出最近三次的消费。
   1. 使用联合索引就可以避免多一次的排序操作。



```sql
explain 中 Extra 中标识 using index -- 表示进行覆盖索引操作
```



### 覆盖索引

定义： **从辅助索引中就可以得到查询的记录**

通常情况下，诸如（a,b）的联合索引是不会选择列b中的查询条件，但是如果是统计操作，并且是覆盖索引，则优化器会走联合索引

```sql
explain select count(*) from buy_log where buy_date >= '2011-01-01' and buy_date <= '2011-02-01';
```



## 锁

## 事物

### 什么是事物

1. 事物是应用程序中一些列的操作，同时这些操作要么全部成功，要么每个操作做的更改都被撤销。
2. 事物会把**数据库从一种一致状态转换为另一种一致状态**。在数据库提交工作时，可以确保要么所有的修改都已经保存，要么所有的修改都不保存。同时事物具有四个特性：ACID
   1. A（原子性）
      - 事务中包含的各操作要么都做，要么都不做 
   2. C（一致性）
      - 事物将数据库从一种一致的状态转化为下一种一致性的状态
      - 同时事物开始之前和事物开始之后，数据库的完整性约束没有被破坏
   3. I（隔离性）
      - 要求每个读写事物的对象对其他事物的操作对象能相互分离，也就是该事物提交前对其他事物都不可见，通常通过锁来实现。
      - 设立了不同程度的隔离级别，通过适度的破坏一致性，得以提高性能
   4. D（持久性）
      - 事物一旦提交，其结果就是永久性的。也就是需要保存到磁盘

### 隔离级别

根据对事物的四大特性的介绍可以得出，隔离性和一致性是有一定的冲突的，有时候为了提高性能，会适度的破坏一致性。 譬如当前的事物对数据做的修改还没有提交，但是另一个事物已经能够看到对这条数据的修改。也就是所谓的**脏读**，其只能在**读未提交**的隔离级别下发生。

#### 事物并发存在的问题

1. **脏读**	

   1. 假设银行中A，B的存款分别为10000，此时有两个事物1、2。1用于操作A向B转账500元。2用于操作统计A和B的总金额

      ![image-20210305212113218](https://i.loli.net/2021/03/05/QKowXd3YpJcbSUI.png)

      我们不考虑任何加锁机制，仅仅从程序运行的角度来看，事务 1 执行成功之后，A 成功转了 100 元到 B 账户，这时 A 余额还剩 9500 元，B 余额剩 10500，总和为 2000；但是事务 2 的统计求和算出来的结果却是 A + B = 19500。这种现象通俗来讲就是：**还没有提交的事物被其他事物读取到了**。我们称之为**脏读**

2. **不可重复读**

   ![image-20210305212559979](https://i.loli.net/2021/03/05/CnuObQk7eAz93Md.png)

   同样的道理，还是上述的例子，事物1在事物2查询A的余额之后更新了A的余额，然后事物B重新读取A的余额，然后就会发现在事物2期间，前后两次读取的结果不同。通俗来讲就是：==在用一个事物中，同样的记录读取两遍，但是结果却不一致。==

3. **幻读**

   幻读的命名有些魔幻。

   ![image-20210305213137969](/Users/weikunkun/Library/Application Support/typora-user-images/image-20210305213137969.png)

   对于幻读的解释：同一个事物中，同样的条件，第一次和第二次读出来结果的记录数量不同。就像是平时出现的了幻象，明明刚开始看到的是这样的，但是再细看，诶！少了个东西，闹鬼了。

   **幻读和不可重复读区别**

   1. 简单来理解：
      1. 不可重复读是在某个 事物中，对**同一条**结果每次的读取结果不同。
      2. 幻读是指在某个事物中，前后两次读取同一个**范围**返回数量不同。

   这么理解也仅仅是便于理解，实际不准确。可能存在另一个事务先插入一条记录然后再删除一条记录的情况，这个时候两次查询得到的记录数也是一样的，但这也是幻读，所以严格点的说法应该是：**两次读取得到的结果集不一样**

   不怎么理解，但是总结之后不难感觉到：

   	1. 不可重复读是因为其他事物执行了update操作。
   	2. 幻读是因为其他事物执行了insert或delete操作。

   ![image-20210305214207658](https://i.loli.net/2021/03/05/SyqYezbpXZoBUQn.png)
   
4. **丢失更新**

   1. 第一类丢失更新（回滚覆盖）

      ![image-20210305225045045](https://i.loli.net/2021/03/05/lmGZbnotdspYO5D.png)

   2. 第二类丢失更新（提交覆盖）

      1. 假设两个事务同时对 A 的余额进行修改，他们都查出 A 的当前余额为 1000，然后事务 2 修改 A 的余额，将 A 的余额加 100 变成 1100 并提交，这个时候 A 的余额应该是 1100，但是这个时候事务 1 并不知道 A 的余额已经变动，而是继续在 1000 的基础上进行减 100 的操作并提交事务，就这样事务 2 的提交被覆盖掉了，事务 1 提交之后 A 的余额变成了 900 元。这就是说事务 1 的提交覆盖了事务 2 的提交，事务 2 的 UPDATE 操作完全丢失了，整个过程如下图所示

         ![image-20210305225113327](https://i.loli.net/2021/03/05/rdaHYBDclQnx7iv.png)

         

         

#### 隔离级别

上述那么多的并发情况下事物可能出现的问题，该如何解决嘞。最简单的方法就等同于我们写一个线程安全的方法一个道理：**加锁**

1. 写的时候不允许其他事务读，读的时候不允许其他事务写，这样是不是就完美解决了？确实如此。这其实就是四大隔离级别里的 **序列化**，在序列化隔离级别下，可以保证事务的安全执行，数据库的一致性得以保障，但是它大大降低了事务的并发能力，性能最低。

为了调和事务的安全性和性能之间的冲突，适当的降低隔离级别，可以有效的提高数据库的并发性能。于是便有了四种不同的隔离级别：

- 读未提交（Read Uncommitted）：可以读取未提交的记录，会出现脏读，幻读，不可重复读，所有并发问题都可能遇到；
- 读已提交（Read Committed）：事务中只能看到已提交的修改，不会出现脏读现象，但是会出现幻读，不可重复读；（大多数数据库的默认隔离级别都是 RC，但是 MySQL InnoDb 默认是 RR）
- 可重复读（Repeatable Read）：MySQL InnoDb 默认的隔离级别，解决了不可重复读问题，但是任然存在幻读问题；（MySQL 的实现有差异，后面介绍）
- 序列化（Serializable）：最高隔离级别，啥并发问题都没有。



#### InnoDB隔离级别

其默认的隔离级别是`repeatable read`，和标准的SQL不同的是，InnoDB存储引擎在**可重复读**隔离级别下，使用**next-key lock**锁算法，因此避免幻读的产生。

>  因为“幻读”的这个“读”字在 MySQL 里本身就存在歧义，这个“读”到底指的是快照读，还是当前读？如果是快照读，MySQL 通过版本号来保证同一个事务里每次查询得到的结果集都是一致的；如果是当前读，MySQL 通过 Next-key locks 保证其他事务无法插入新的数据，从而避免幻读问题。当然，如果你的场景里一会是快照读，一会是当前读，导致幻读现象，MySQL 也只能表示自己很无奈了。

#### 传统的隔离级别

上面说了很多，其实我一直在刻意的避免谈到锁，因为隔离级别和锁本身就是两个东西，SQL 规范中定义的四种隔离级别，分别是为了解决事务并发时可能遇到的四种问题，至于如何解决，实现方式是什么，规范中并没有严格定义。锁作为最简单最显而易见的实现方式，可能被广为人知，所以大家在讨论某个隔离级别的时候，往往会说这个隔离级别的加锁方式是什么样的。其实，锁只是实现隔离级别的几种方式之一，除了锁，实现并发问题的方式还有[时间戳](https://en.wikipedia.org/wiki/Timestamp-based_concurrency_control)，[多版本控制](https://en.wikipedia.org/wiki/Multiversion_concurrency_control)等等，这些也可以称为[无锁的并发控制](https://en.wikipedia.org/wiki/Non-lock_concurrency_control)。

传统的隔离级别是基于锁实现的，这种方式叫做 **基于锁的并发控制（Lock-Based Concurrent Control，简写 LBCC）**。通过对读写操作加不同的锁，以及对释放锁的时机进行不同的控制，就可以实现四种隔离级别。传统的锁有两种：读操作通常加共享锁（Share locks，S锁，又叫读锁），写操作加排它锁（Exclusive locks，X锁，又叫写锁）；加了共享锁的记录，其他事务也可以读，但不能写；加了排它锁的记录，其他事务既不能读，也不能写。另外，对于锁的粒度，又分为行锁和表锁，行锁只锁某行记录，对其他行的操作不受影响，表锁会锁住整张表，所有对这个表的操作都受影响。

归纳起来，四种隔离级别的加锁策略如下：

- 读未提交（Read Uncommitted）：事务读不阻塞其他事务读和写，事务写阻塞其他事务写但不阻塞读；通过对写操作加 “持续X锁”，对读操作不加锁 实现；
- 读已提交（Read Committed）：事务读不会阻塞其他事务读和写，事务写会阻塞其他事务读和写；通过对写操作加 “持续X锁”，对读操作加 “临时S锁” 实现；不会出现脏读；
- 可重复读（Repeatable Read）：事务读会阻塞其他事务事务写但不阻塞读，事务写会阻塞其他事务读和写；通过对写操作加 “持续X锁”，对读操作加 “持续S锁” 实现；
- 序列化（Serializable）：为了解决幻读问题，行级锁做不到，需使用表级锁。

结合上面介绍的每种隔离级别分别是用来解决事务并发中的什么问题，再来看看它的加锁策略其实都挺有意思的。其中 **读未提交** 网上有很多人认为不需要加任何锁，这其实是错误的，我们上面讲过，有一种并发问题在任何隔离级别下都不允许存在，那就是第一类丢失更新（回滚覆盖），如果不对写操作加 X 锁，当两个事务同时去写某条记录时，可能会出现丢失更新问题，[这里](http://www.jianshu.com/p/71a79d838443) 有一个例子可以看到写操作不加 X 锁发生了回滚覆盖。再看 **读已提交**，它是为了解决脏读问题，只能读取已提交的记录，要怎么做才可以保证事务中的读操作读到的记录都是已提交的呢？很简单，对读操作加上 S 锁，这样如果其他事务有正在写的操作，必须等待写操作提交之后才能读，因为 S 和 X 互斥，如果在读的过程中其他事务想写，也必须等事务读完之后才可以。这里的 S 锁是一个临时 S 锁，表示事务读完之后立即释放该锁，可以让其他事务继续写，如果事务再读的话，就可能读到不一样的记录，这就是 **不可重复读** 了。为了让事务可以重复读，加在读操作的 S 锁变成了持续 S 锁，也就是直到事务结束时才释放该锁，这可以保证整个事务过程中，其他事务无法进行写操作，所以每次读出来的记录是一样的。最后，**序列化** 隔离级别下单纯的使用行锁已经实现不了，因为行锁不能阻止其他事务的插入操作，这就会导致幻读问题，这种情况下，我们可以把锁加到表上（也可以通过范围锁来实现，但是表锁就相当于把表的整个范围锁住，也算是特殊的范围锁吧）。

从上面的描述可以看出，通过对**锁的类型（读锁还是写锁）**，**锁的粒度（行锁还是表锁）**，**持有锁的时间（临时锁还是持续锁）**合理的进行组合，就可以实现四种不同的隔离级别。

这四种不同的加锁策略实际上又称为 **封锁协议（Locking Protocol）**，所谓协议，就是说不论加锁还是释放锁都得按照特定的规则来。**读未提交** 的加锁策略又称为 **一级封锁协议**，后面的分别是二级，三级，**序列化** 的加锁策略又称为 **四级封锁协议**。

其中三级封锁协议在事务的过程中为写操作加持续 X 锁，为读操作加持续 S 锁，并且在事务结束时才对锁进行释放，像这种加锁和解锁明确的分成两个阶段我们把它称作 **两段锁协议（2-phase locking，简称 2PL）**。在两段锁协议中规定，加锁阶段只允许加锁，不允许解锁；而解锁阶段只允许解锁，不允许加锁。这种方式虽然无法避免死锁，但是两段锁协议可以保证事务的并发调度是串行化的（关于串行化是一个非常重要的概念，尤其是在数据恢复和备份的时候）。在两段锁协议中，还有一种特殊的形式，叫 **一次封锁**，意思是指在事务开始的时候，将事务可能遇到的数据全部一次锁住，再在事务结束时全部一次释放，这种方式可以有效的避免死锁发生。但是这在数据库系统中并不适用，因为事务开始时并不知道这个事务要用到哪些数据，一般在应用程序中使用的比较多。



# MySQL实战45讲笔记

## 基础

从数据库的架构和阐述了sql的执行都经过哪些内容，之后讲解了**redolog**和**binlog**。接下来阐述了**隔离级别**、**索引**、**锁**的原理和机制。这些都是数据库的基础信息。后面通过实战的方式，阐述了MySQL中的详细讲解

## 01、基础架构：一条SQL查询语句是如何执行的

![image-20210219105910297](https://i.loli.net/2021/02/19/iq8A3nCxu6wyV9G.png)



### service层

1. 分析器
2. 优化器
3. 执行器

### 存储引擎层

1. 存储引擎

### 查询过程

1. 执行前先通过连接器连接数据库
2. 分析器进行**词法、语法的分析**，分析出这是一条查询语句
3. 优化器进行优化，优化查询，譬如选择索引等
4. 执行器进行具体的执行，找到该条数据，返回

可能涉及查询缓存。

## 02、日志系统：一条SQL是更新语句是如何执行的

执行过程和查询的过程是一样的。不过是因为更新问题，所以查询缓存会被删除。同时还会涉及两个很重要的日志模块儿：

1. **redolog(重做日志)**
   1. innodb引擎特有的日志
2. **binlog(归档日志)**
   1. sevice层自己的日志

### redolog

如果每一次的更新操作都需要写进磁盘，然后磁盘也要找到对应的那条记录，然后再更新，整个过程 IO 成本、查找成本都很高。为了解决这个问题，MySQL 的设计者就用了类似酒店掌柜粉板的思路来提升更新效率。

而粉板和账本配合的整个过程，其实就是 MySQL 里经常说到的 WAL 技术，WAL 的全称是 Write-Ahead Logging，它的关键点就是先写日志，再写磁盘，也就是先写粉板，等不忙的时候再写账本。

具体来说，当有一条记录需要更新的时候，InnoDB 引擎就会先把记录写到 redo log（粉板）里面，并更新内存，这个时候更新就算完成了。同时，InnoDB 引擎会在适当的时候，将这个操作记录更新到磁盘里面，而这个更新往往是在系统比较空闲的时候做，这就像打烊以后掌柜做的事。

如果今天赊账的不多，掌柜可以等打烊后再整理。但如果某天赊账的特别多，粉板写满了，又怎么办呢？这个时候掌柜只好放下手中的活儿，把粉板中的一部分赊账记录更新到账本中，然后把这些记录从粉板上擦掉，为记新账腾出空间。与此类似，InnoDB 的 redo log 是固定大小的，比如可以配置为一组 4 个文件，每个文件的大小是 1GB，那么这块“粉板”总共就可以记录 4GB 的操作。从头开始写，写到末尾就又回到开头循环写，如下面这个图所示。

![image-20210219220227427](https://i.loli.net/2021/02/19/pmF6iNIrxbuDwtY.png)

write pos 和 checkpoint 之间的是“粉板”上还空着的部分，可以用来记录新的操作。如果 write pos 追上 checkpoint，表示“粉板”满了，这时候不能再执行新的更新，得停下来先擦掉一些记录，把 checkpoint 推进一下。



有了 redo log，InnoDB 就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为 **crash-safe**。



### binlog

MySQL整体来看，一共分为两层：

1. service层	
   1. 负责MySQL功能层面的事情
2. 存储引擎层
   1. 负责数据存储相关的事宜

**redolog**是innodb存储引擎独有的日志，同时service层也有自己的日志：**binlog**



最开始 MySQL 里并没有 InnoDB 引擎。**MySQL 自带的引擎是 MyISAM，但是 MyISAM 没有 crash-safe 的能力，binlog 日志只能用于归档**。而 InnoDB 是另一个公司以插件形式引入 MySQL 的，既然只依靠 binlog 是没有 crash-safe 能力的，所以 InnoDB 使用另外一套日志系统——也就是 redo log 来实现 crash-safe 能力。



### redolog和binlog的总结

1. redo log 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。
2. redo log 是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。
3. redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。



![image-20210220081629575](https://i.loli.net/2021/02/20/y4Yp9bDUSNjcBCX.png)

有了对这两个日志的概念性理解，我们再来看执行器和 InnoDB 引擎在执行这个简单的 update 语句时的内部流程。

1. 执行器先找引擎取 ID=2 这一行。ID 是主键，引擎直接用树搜索找到这一行。如果 ID=2 这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回。
2. 执行器拿到引擎给的行数据，把这个值加上 1，比如原来是 N，现在就是 N+1，得到新的一行数据，再调用引擎接口写入这行新数据。
3. 引擎将这行新数据更新到内存中，同时将这个更新操作记录到 redo log 里面，此时 redo log 处于 prepare 状态。然后告知执行器执行完成了，随时可以提交事务。
4. 执行器生成这个操作的 binlog，并把 binlog 写入磁盘。
5. 执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（commit）状态，更新完成。





## 13、为什么表数据删除掉一半，表文件大小不变



delete之后，索引之间会变得不紧凑，所以，虽然数据量少了，但是索引整体的结构没有变，所以表文件可能不会变化，

使用

```sql
alter table t engine=InnoDB
```

也就是重建表的意思

> 要收缩一个表，只是 delete 掉表里面不用的数据的话，表文件的大小是不会变的，你还要通过 alter table 命令重建表，才能达到表文件变小的目的。我跟你介绍了重建表的两种实现方式，Online DDL 的方式是可以考虑在业务低峰期使用的，而 MySQL 5.5 及之前的版本，这个命令是会阻塞 DML 的，这个你需要特别小心



## 14、count(*)这么慢，该怎么办



