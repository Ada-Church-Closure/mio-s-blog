+++
date = '2025-11-16T13:07:06+08:00'
draft = false
title = 'MiniDB'

+++

# Mio'sDB

> 这是一个由Java实现的轻量级的关系型数据库.
>
> 基于 socket 的 server 和 client进行前后端的通信.
>
> 原博客教程及仓库:
>
> https://shinya.click/projects/mydb/mydb0

## 运行方式

注意首先需要在 pom.xml 中调整编译版本，如果导入 IDE，请更改项目的编译版本以适应你的 JDK

首先执行以下命令编译源码：

```shell
 mvn compile
```

接着执行以下命令以 /tmp/mydb 作为路径创建数据库：

```shell
 mvn exec:java -Dexec.mainClass="top.guoziyang.mydb.backend.Launcher" -Dexec.args="-create /tmp/mydb"
```

随后通过以下命令以默认参数启动数据库服务：

```shell
 mvn exec:java -Dexec.mainClass="top.guoziyang.mydb.backend.Launcher" -Dexec.args="-open /tmp/mydb"
```

这时数据库服务就已经启动在本机的 9999 端口。重新启动一个终端，执行以下命令启动客户端连接数据库：

```shell
 mvn exec:java -Dexec.mainClass="top.guoziyang.mydb.client.Launcher"
```

会启动一个交互式命令行，就可以在这里输入类 SQL 语法，回车会发送语句到服务，并输出执行的结果。

# 数据库原理

> 我个人认为现在的后端,应当以数据库为中心.
>
> 这是借鉴的一个简单项目,手写一个数据库,利用java语言.
>
> 原博客教程及仓库:
>
> https://shinya.click/projects/mydb/mydb0

## 0.Overview

前后端利用socket通信进行交互.

### 核心模块

#### Transaction Manager 事务管理器

​	管理事务的生命周期,实现ACID特性.

​	**交互：** 与 **Version Manager (VM)** 协作来实现隔离性和回滚，并通知 **Data Manager (DM)** 何时需要将变更写入磁盘。

#### Data Manager 数据管理器

​	管理读写,物理存储和持久性.

#### Version Manager 版本管理器

​	实现多版本并发控制,MVCC.

#### Index Manager 索引管理器

​	管理索引,实现B+树或者B树.

#### Table Manager 表管理器

​	比较上层的一个组件,管理table,存储table的元信息.

> 本教程的实现顺序是 TM -> DM -> VM -> IM -> TBM

## 1.实现事务管理器 TM

> ​	你可以理解,我们的事务管理器,只是一直在做一些关于文件的维护,维护当前处理的每一个事务的状态,同时要支持并发的访问.

### XID

xid唯一标识了一个事务.

自增 不重复 

0:超级事务,可以在没有申请事务的情况下进行.--->状态一直是**commited**

> TransactionManager **维护了一个 XID 格式的文件**，用来**记录各个事务的状态**。MYDB 中，每个事务都有下面的三种状态：
>
> 1. **active**，正在进行，尚未结束
> 2. **committed**，已提交
> 3. **aborted**，已撤销（回滚）

> XID 文件给每个事务分配了一个字节的空间，用来保存其状态。同时，在 XID 文件的头部，还保存了一个 8 字节的数字，记录了这个 XID 文件管理的事务的个数。于是，事务 xid 在文件中的状态就存储在 (xid-1)+8 字节处，xid-1 是因为 xid 0（Super XID）的状态不需要记录。

基本的接口:

```java
  // 开启事务
      long begin();
      // 提交事务
      void commit(long xid);
      // 取消事务
      void abort(long xid);
      // 查询事务状态
      boolean isActive(long xid);
      boolean isCommitted(long xid);
      boolean isAborted(long xid);
      // 关闭事务管理器
      void close();
```

> 我们这里使用Java的 NIO来进行文件的读写.

### 核心特性：直接内存映射 (Memory-Mapped I/O)

`FileChannel` 最强大的特性之一是它支持 **直接内存映射 I/O (Memory-Mapped I/O, MappedByteBuffer)**。

- **工作原理：** 你可以将文件的一部分甚至整个文件，**映射 (map)** 到 Java 虚拟机的内存中。
- **好处：**
  - 一旦映射完成，对文件内容的读写就像读写内存中的数组一样简单。
  - 操作系统负责在内存和磁盘之间同步数据，**避免了传统 I/O 中数据在内核缓冲区和用户缓冲区之间的多次拷贝**（这是一个巨大的性能提升点）。

对于你的 **Data Manager (DM)** 来说，这非常理想，因为 DM 需要处理大量的随机读写，而内存映射能极大地加速对数据页的访问。

## 2.实现数据管理器 DM

### 实现引用计数缓存框架/共享内存数组(Java没有指针)

> 我们接下来去实现DM.

1.分页管理DB.

2.管理日志文件,发生错误时可以进行恢复.

3.抽象DB文件给上层使用,提供缓存.

> 这就是DAO层的核心,向下会直接管理数据,并且会为上层提供调用的接口.

我们引入**缓存系统**,而我们管理的方式是**引用计数**.

> **为什么不使用LRU算法?**
>
> ​	*某个时刻缓存满了，缓存驱逐了一个资源，这时上层模块想要将某个资源强制刷回数据源，这个资源恰好是刚刚被驱逐的资源。那么上层模块就发现，这个数据在缓存里消失了，这时候就陷入了一种尴尬的境地：是否有必要做回源操作？*
>
> 1. 不回源。由于没法确定缓存被驱逐的时间，更没法确定被驱逐之后数据项是否被修改，这样是极其不安全的
> 2. 回源。如果数据项被驱逐时的数据和现在又是相同的，那就是一次无效回源
> 3. 放回缓存里，等下次被驱逐时回源。看起来解决了问题，但是此时缓存已经满了，这意味着你还需要驱逐一个资源才能放进去。这有可能会导致缓存抖动问题

release释放引用,引用为0的时候,就自动驱逐这个资源.

```java
  // 尝试获取该资源
              if(maxResource > 0 && count == maxResource) {
                  lock.unlock();
                  // 缓存已满，抛出异常
                  throw Error.CacheFullException;
              }
```

缓存满的时候,直接结束.

这里的所有资源,我们都是使用三张**hash table**来管理的.

### 数据页的缓存和管理问题

我们怎么操作文件系统?

> **读写和缓存**都是以**page**作为**基本单位**.

数据即文件,我们的数据都存放在磁盘文件内部.

> 这里缓存的实现,就是比较基本的缓存思想.

#### 页面管理

db的第一页存放一些元数据,每次打开的时候,和上一次进行校验,看是否合法

剩下的就是正常的数据页.

2bytes 存放当前页面内部的空闲位置的偏移,在做insert的时候,我们主要就是使用这里的数据

### 日志文件和恢复策略

> 这里的恢复思想和文件系统的崩溃恢复是类似的.
>
> MYDB 提供了崩溃后的数据恢复功能。DM 层在每次对底层数据操作时，都会记录一条日志到磁盘上。在数据库奔溃之后，再次启动时，可以根据日志的内容，恢复数据文件，保证其一致性。

```txt
  [XChecksum][Log1][Log2][Log3]...[LogN][BadTail]
```

一个基本的日志文件的组成:

1.Checksum对所有log做校验和.

2.BadTail就是崩溃时没有写完的日志数据.

> “Write-Ahead Logging(WAL)”的意思是：**在数据页面的实际修改（写入主数据文件）之前，必须先把描述这个修改的记录（日志）写入并持久化到日志文件。**

#### 恢复策略

Data Manager 为上层提供了两种操作: insert and update with no delete why?

And the same as file system,we use the strategy of WAL.

> 对于两种数据操作，DM 记录的日志如下：
>
> - (Ti, I, A, x)，表示事务 Ti 在 A 位置插入了一条数据 x
> - (Ti, U, A, oldx, newx)，表示事务 Ti 将 A 位置的数据，从 oldx 更新成 newx

非并发的情况下,log可能是这样:

```txt
  (Ti, x, x), ..., (Ti, x, x), (Tj, x, x), ..., (Tj, x, x), (Tk, x, x), ..., (Tk, x, x)
```

> #### 单线程
>
> 由于单线程，Ti、Tj 和 Tk 的日志永远不会相交。这种情况下利用日志恢复很简单，假设日志中最后一个事务是 Ti：
>
> 1. 对 Ti 之前所有的事务的日志，进行重做（redo）
> 2. 接着检查 Ti 的状态（XID 文件），如果 Ti 的状态是已完成（包括 committed 和 aborted,这里意味着这里的一个log是完整的.），就将 Ti 重做，否则进行撤销（undo）
>
> 接着，是如何对事务 T 进行 redo：
>
> 1. 正序扫描事务 T 的所有日志
> 2. 如果日志是插入操作 (Ti, I, A, x)，就将 x 重新插入 A 位置
> 3. 如果日志是更新操作 (Ti, U, A, oldx, newx)，就将 A 位置的值设置为 newx
>
> undo 也很好理解：
>
> 1. 倒序扫描事务 T 的所有日志
> 2. 如果日志是插入操作 (Ti, I, A, x)，就将 A 位置的数据删除
> 3. 如果日志是更新操作 (Ti, U, A, oldx, newx)，就将 A 位置的值设置为 oldx
>
> 注意，MYDB 中其实没有真正的删除操作，对于插入操作的 undo，只是将其中的标志位设置为 invalid。对于删除的探讨将在 VM 一节中进行。

考察几种多线程的情况:

```txt
 T1 begin
 T2 begin
 T2 U(x)
 T1 R(x)
 ...
 T1 commit
 T2 Running
 MYDB break down
```

两个事务都在进行的时候,一个事务读取了另外一个事务的更新的内容,但是breakdown的时候,T2应当被撤销.

**级联回滚.**

> I.一个正在进行的事务,不应当读取其余的未提交的事务的数据!

```txt
 T1 begin
 T2 begin
 T1 set x = x+1 // 产生的日志为 (T1, U, A, 0, 1)
 T2 set x = x+1 // 产生的日志为 (T1, U, A, 1, 2)
 T2 commit
 T1 Running
 MYDB break down
```

breakdown的时候,T2已经提交事务,但是T1还在进行.

那么我们恢复的时候,对于T2进行重做,对于T1撤销,这个顺序无论怎样,结果都是0或者2.

> II.一个正在进行的事务,不应当修改其余未提交的事务的数据!(但其实是可以的)

实际上VM层可以保证这两点的实现.

> 我们只要保证:
>
> 1. **重做所有崩溃时已完成**（committed 或 aborted）的**事务**
> 2. **撤销所有崩溃时未完成（active）的事务**

### 索引和Data Item的抽象

> 页面索引，缓存了每一页的空闲空间。用于在上层模块进行插入操作时，能够快速找到一个合适空间的页面，而无需从磁盘或者缓存中检查每一个页面的信息。
>
> MYDB 用一个比较粗略的算法实现了页面索引，将一页的空间划分成了 40 个区间。在启动时，就会遍历所有的页面信息，获取页面的空闲空间，安排到这 40 个区间中。insert 在请求一个页时，会首先将所需的空间向上取整，映射到某一个区间，随后取出这个区间的任何一页，都可以满足需求。

## 3.实现version manager VM

> **版本控制器**? **事务** 和 **数据版本管理**的核心.
>
> 要实现版本管理器的主要问题就是解决并发!

### 2PL 和 MVCC

​	**2PL (Two-Phase Locking, 两阶段锁定)** 和 **MVCC (Multi-Version Concurrency Control, 多版本并发控制)** 是现代数据库用来保证事务 **隔离性 (Isolation)** 的主要方法.

​	我们定义冲突:

​		两个不同的事务,对于一块资源同时**update**或者一个update另一个read--->冲突.

#### 核心原理：两阶段 2PL

顾名思义，2PL 将事务的锁定行为分为两个严格的阶段：

1. 增长阶段 (Growing Phase)

- **行为：** 事务可以 **获取 (Acquire)** 任何它需要的锁，但 **不能释放** 任何已持有的锁。
- **目的：** 确保事务在执行过程中，它所需的所有资源都能被安全地保护起来。

1. 收缩阶段 (Shrinking Phase)

- **行为：** 事务可以 **释放 (Release)** 锁，但 **不能再获取** 任何新的锁。
- **目的：** 一旦事务进入收缩阶段，它不能再请求新的资源，这保证了锁的释放是单向的，从而保证了 **可串行化 (Serializability)** 的隔离级别。

锁的类型

1. **共享锁 (S-Lock / Read Lock)：** 允许多个事务同时读取数据。

2. **排他锁 (X-Lock / Write Lock)：** 只允许一个事务修改数据，阻止所有其他事务的读写。

3. > 比如上面我们就会使用排他锁,两个正在进行的事务不能同时或者至少有一个在进行update操作.

优点和缺点

|  **特点**  |                           **描述**                           |
| :--------: | :----------------------------------------------------------: |
| **优点：** |   **隔离性强：** 很容易实现最高的 **可串行化** 隔离级别。    |
| **缺点：** | **低并发性：** 读操作（即使是共享锁）也会阻塞写操作，反之亦然，降低了系统的并行处理能力。 |
| **缺点：** | **死锁 (Deadlock)：** 事务可能因为互相等待对方释放锁而陷入僵局，需要额外的死锁检测和解决机制。 |

#### MVCC

> 在介绍 MVCC 之前，首先明确**记录和版本**的概念。
>
> DM 层向上层提供了数据项（Data Item）的概念，VM 通过管理所有的数据项，向上层提供了记录（**Entry**）的概念。上层模块通过 **VM 操作数据的最小单位，就是记录。**VM 则在其内部，**为每个记录，维护了多个版本（Version）。**每当上层模块对某个记录进行修改时，VM 就会为这个记录创建一个新的版本。
>
> MYDB 通过 MVCC，降低了事务的阻塞概率。譬如，T1 想要更新记录 X 的值，于是 T1 需要首先获取 X 的锁，接着更新，也就是创建了一个新的 X 的版本，假设为 x3。假设 T1 还没有释放 X 的锁时，T2 想要读取 X 的值，这时候就不会阻塞，MYDB 会返回一个较老版本的 X(**所以叫多版本并发控制**)，例如 x2。这样最后执行的结果，就等价于，T2 先执行(**也就是T2会读取到没有更新的值,我们的两个事务本来没有前后逻辑的要求.**)，T1 后执行，调度序列依然是可串行化的。如果 X 没有一个更老的版本，那只能等待 T1 释放锁了。所以只是降低了概率。
>
> 还记得我们在第四章中，为了保证数据的可恢复，VM 层传递到 DM 的操作序列需要满足以下两个规则：
>
> > 规定 1：正在进行的事务，不会读取其他任何未提交的事务产生的数据。(**这是由MVCC实现的,你想读取的时候,我们返回给你一个老版本的数据.**) 规定 2：正在进行的事务，不会修改其他任何未提交的事务修改或产生的数据。(**直接使用2PL中的排他锁实现的.**)
>
> 由于 2PL 和 MVCC，我们可以看到，这两个条件都被很轻易地满足了。

​	MVCC 是一种 **乐观 (Optimistic)** 的并发控制机制，它避免了读写之间的锁竞争，通过 **保存数据的多个版本** 来实现隔离。

核心原理：版本和快照

1. **不阻塞读取：** 当一个事务 T_{read} 读取数据时，它不会申请锁，而是获取一个 **数据快照 (Snapshot)**。
2. **多版本存储：** 当一个写入事务 T_{write} 修改数据时，它不会覆盖旧数据，而是创建一个数据的 **新版本**。旧版本被保留下来。
3. **可见性判断：** 每个数据版本都标记有创建它的事务 ID（T*{start}）和删除它的事务 ID（T*{end}）。事务 T_{read} 根据其启动时的 **Read View**（活跃事务集合），判断哪个版本对它是“可见”的。
   - 简单来说，读取事务 T_{read} 只会看到在它开始之前就已经提交的数据版本。

隔离性实现

MVCC 特别适合实现 **读提交 (Read Committed)** 和 **可重复读 (Repeatable Read)** 等隔离级别：

- **读取事务** 永远不会被 **写入事务** 阻塞。
- **写入事务** 只在提交时进行冲突检查或锁定，但不会阻塞读取。

优点和缺点

| **特点**   | **描述**                                                     |
| ---------- | ------------------------------------------------------------ |
| **优点：** | **高并发性：** 读操作和写操作很少互相阻塞，极大地提高了 OLTP (在线事务处理) 的性能。 |
| **优点：** | **无死锁（读写）：** 读事务不需要锁，因此不会发生读写之间的死锁。 |
| **缺点：** | **存储开销：** 需要额外的存储空间来保存数据的多个历史版本。  |
| **缺点：** | **垃圾回收 (GC)：** 需要一个 **Version Manager (VM)** 机制来定期清理（垃圾回收）那些不再被任何活跃事务引用的旧版本数据。 |

### 事务的隔离级别

#### 读提交

当一个记录的最新版本被加锁,有另一个事务尝试进行读取的时候,会返回旧的版本数据,那么新版本对于这个事务来说就是,**不可见**的,这就是 **版本可见性** 的问题.

​	**最低隔离度**:读取**数据**时,**只能读取已经提交的事务产生的数据**.

> MYDB 实现读提交，为每个版本维护了两个变量，就是上面提到的 XMIN 和 XMAX：
>
> - XMIN：创建该版本的事务编号
> - XMAX：删除该版本的事务编号
>
> XMIN 应当在版本创建时填写，而 XMAX 则在版本被删除，或者有新版本出现时填写。
>
> XMAX 这个变量，也就解释了为什么 DM 层不提供删除操作，当想删除一个版本时，只需要设置其 XMAX，这样，这个版本对每一个 XMAX 之后的事务都是不可见的，也就等价于删除了。
>
> **利用版本可见性,我们就实现了删除.**

我们判断一个记录对于某个事务t是否是可见的:

```java
 /**
     * 读提交,判断一个版本对事务t是否可见
     * @param tm 事务管理器
     * @param t 事务
     * @param e 记录
     * @return 版本是否对事务可见
     */
    private static boolean readCommitted(TransactionManager tm, Transaction t, Entry e) {
        long xid = t.xid;
        long xmin = e.getXmin();
        long xmax = e.getXmax();
        // 自己创建的记录且未删除
        if(xmin == xid && xmax == 0) return true;

        // 创建该记录的事务已提交
        if(tm.isCommitted(xmin)) {
            // 并且还未被删除,就是可见的
            if(xmax == 0) return true;
            // 否则,如果删除该记录的事务不是自己且未提交,也是可见的
            // 因为如果提交了,说明记录被删除了,对当前事务不可见
            if(xmax != xid) {
                if(!tm.isCommitted(xmax)) {
                    return true;
                }
            }
        }
        return false;
    }
```

##### 读提交的问题:不可重复读/幻读

###### Non-Repeatable Read:

​	之前的问题:在一个事务内,我对于一个记录访问了两次:

​	1.第一次访问的时候,返回的是旧版本.

​	2.第二次事务已经读提交,变得可见,返回新版本,两次不一样.

1. **事务 A** 开始，并执行第一次查询：`SELECT * FROM users WHERE id = 10;` 得到数据 R_1。
2. **事务 B** 紧接着修改了这条记录：`UPDATE users SET balance = 500 WHERE id = 10;` 并 **提交 (COMMIT)** 事务。
3. **事务 A** 再次执行同样的查询：`SELECT * FROM users WHERE id = 10;` 得到数据 R_2。
4. **结果：** R_1 \neq R_2。事务 A 在 R_1 和 R_2 之间看到了另一个已提交事务 B 的修改。

###### Phantom Read:

​	多次执行同一个范围查询的时候,两次出现的集合不一样,其实和上面是类似的:

1. **事务 A** 开始，执行第一次范围查询：`SELECT COUNT(*) FROM products WHERE type = 'Electronics';` 得到结果 C_1。
2. **事务 B** 紧接着插入了一条满足查询条件的新记录：`INSERT INTO products (name, type) VALUES ('Phone', 'Electronics');` 并 **提交 (COMMIT)** 事务。
3. **事务 A** 再次执行同样的范围查询：`SELECT COUNT(*) FROM products WHERE type = 'Electronics';` 得到结果 C_2。
4. **结果：** C_1 \neq C_2。事务 A 发现了一个 **幻影行 (Phantom Row)**，这条新行是事务 B 插入的

> 多帅的名字!

| **异常行为**   | **隔离级别**                   | **如何解决？**                                               | **机制**                                             |
| -------------- | ------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------- |
| **不可重复读** | **可重复读 (Repeatable Read)** | 事务 A 在第一次读取时，会锁定或快照这条记录，阻止事务 B 的修改或看不到事务 B 的修改。 | 2PL (行锁) 或 MVCC (事务快照)                        |
| **幻读**       | **可串行化 (Serializable)**    | 事务 A 在第一次范围查询时，需要锁定 **整个查询范围**，阻止事务 B 在这个范围内进行 **INSERT**。 | **间隙锁 (Gap Lock)** 或 **谓词锁 (Predicate Lock)** |

#### 解决不可重复读,引入新的隔离级别:可重复读

> > **事务只能读取它开始时，就已经结束的那些事务产生的数据版本.**
>
> 这条规定，增加于，事务需要忽略：
>
> 1. 在本事务后开始的事务的数据;
> 2. 本事务开始时还是 active 状态的事务的数据
> 3. **就是非常保守,我只认定在我之前就结束的记录.**

> ​	对于**第一条**，只需要**比较事务 ID**，即可确定。而对于第二条，则需要在事务 Ti 开始时，记录下当前活跃的所有事务 **SP(Ti)**，如果记录的某个版本，**XMIN** 在 **SP(Ti)** 中，也应当对 Ti 不可见。
>
> ​	也就是事务开始的时候,要保存一个快照.

我们先使用一个**hashmap**保存所有活跃事务的id:

```java
 /**
     * 创建一个新的事务对象,并根据隔离级别生成快照,snapshot中包含了所有在该事务开始时活跃的事务id
     * @param xid 事务id
     * @param level 事务隔离级别
     * @param active 当前活跃的事务列表
     * @return 新的事务对象
     */
    public static Transaction newTransaction(long xid, int level, Map<Long, Transaction> active) {
        Transaction t = new Transaction();
        t.xid = xid;
        t.level = level;
        if(level != 0) {
            t.snapshot = new HashMap<>();
            for(Long x : active.keySet()) {
                t.snapshot.put(x, true);
            }
        }
        return t;
    }
```

我们可重复读级别可见性的逻辑就是:

```java
 /**
     * 可重复读,判断一个版本对事务t是否可见
     * @param tm 事务管理器
     * @param t 事务
     * @param e 记录
     * @return 版本是否对事务可见,在可重复读隔离级别下
     */
    private static boolean repeatableRead(TransactionManager tm, Transaction t, Entry e) {
        long xid = t.xid;
        long xmin = e.getXmin();
        long xmax = e.getXmax();
        // 自己创建的记录且未删除
        if(xmin == xid && xmax == 0) return true;

        // 创建该记录的事务已提交,且在当前事务开始前就已经提交,且不在当前事务的活跃快照中
        if(tm.isCommitted(xmin) && xmin < xid && !t.isInSnapshot(xmin)) {
            // 并且还未被删除,就是可见的
            if(xmax == 0) return true;
            // 否则,如果删除该记录的事务不是自己且未提交,或者在当前事务的活跃快照中,也是可见的
            // 说白了,在我可重复读级别看来,你就是没删除也没修改
            if(xmax != xid) {
                if(!tm.isCommitted(xmax) || xmax > xid || t.isInSnapshot(xmax)) {
                    return true;
                }
            }
        }
        return false;
    }
```

#### 真正实现VM

> ​	说到**版本跳跃**之前，顺便提一嘴，**MVCC** 的实现，使得 **MYDB** 在**撤销或是回滚事务**很简单：只需要将这个事务标记为 **aborted** 即可。根据前一章提到的可见性，每个事务都只能看到其他 committed 的**事务**所产生的数据，一个 aborted 事务产生的数据，就不会对其他事务产生任何影响了，也就相当于，这个事务不曾存在过。

##### MVCC版本跳跃问题

```txt
T1 begin
T2 begin
R1(X) // T1 读取 x0
R2(X) // T2 读取 x0
U1(X) // T1 将 X 更新到 x1
T1 commit
U2(X) // T2 将 X 更新到 x2
T2 commit
```

> 中间跳跃了一个x1的版本,但是你可以看出来,如果是读提交的隔离级别,就是没有问题的.
>
> 但是可重复读是不允许的.

> Ti 不可见的 Tj，有两种情况：
>
> 1. XID(Tj) > XID(Ti) 这个事务在我之后**创建**.
> 2. Tj in SP(Ti)          这个事务在**活跃快照**之中.
>
> 于是**版本跳跃的检查**也就很简单了，取出要修改的数据 **X** 的最新**提交版本**，并检查该最新版本的创建者对当前事务是否可见：

```java
// 是否有版本跳跃的问题
    public static boolean isVersionSkip(TransactionManager tm, Transaction t, Entry e) {
        long xmax = e.getXmax();
        if(t.level == 0) {
            // 读提交不存在问题
            return false;
        } else {
            // 之前有一个事务:
            //      修改了并且已经提交 && 这个事务还是我不可见的事务!
            return tm.isCommitted(xmax) && (xmax > t.xid || t.isInSnapshot(xmax));
        }
    }
```

##### 死锁检测:解决2PL带来的问题

> ​	当一个事务等待另外一个事务释放lock的时候,这个关系我们称为一个有向边,每次有 **等待** 的时候,就加上一条边,然后检测图中是否出现了环,出现的话,就要撤销刚刚增加的这个事务.

```java
/**
 * 维护了一个依赖等待图，以进行死锁检测.
 * 这里比较像OS中的死锁检测问题.
 */
public class LockTable {
    
    private Map<Long, List<Long>> x2u;  // 某个XID已经获得的资源的UID列表
    private Map<Long, Long> u2x;        // UID被某个XID持有
    private Map<Long, List<Long>> wait; // 正在等待UID的XID列表
    private Map<Long, Lock> waitLock;   // 正在等待资源的XID的锁
    private Map<Long, Long> waitU;      // XID正在等待的UID
    private Lock lock;
...
```

## 4.实现index manager 索引管理器

在实现之前,我们先来理解一下B+ Tree:

> `B+树`是一种数据结构，是一个N叉排序树，每个节点通常有多个孩子，一棵`B+树`包含根节点、内部节点和叶子节点。根节点可能是一个叶子节点， 也可能是一个包含两个或两个以上孩子节点的节点。
>
> `B+树`通常用于数据库和操作系统的`文件系统`中。NTFS、ReiserFS、NSS、XFS、JFS、ReFS和BFS等文件系统都在使用`B+树`作为元数据索引。`B+树`的特点是能够保持数据稳定有序， 其插入与修改拥有较稳定的对数时间复杂度。`B+树`元素自底向上插入。
>
> 自己找图去看也行.

1.n棵子树就有n个关键字,一个关键字对应一颗子树(或者比孩子的个数小1).

2.所有叶子节点包含全部关键字的信息.(所有的数据都会存放在叶子节点中)

> 非叶子节点含其子树中的**最大关键字**.

3.两个指针,一个指向root(随机查找),一个指向最小节点(从最小开始顺序查找).

> 除此之外B+树还有以下的要求:
>
> - B+树包含2种类型的结点：内部结点（也称索引结点）和叶子结点。根结点本身即可以是内部结点，也可以是叶子结点。根结点的关键字个数最少可以只有1个。
> - B+树与B树最大的不同是内部结点不保存数据，只用于索引，所有数据（或者说记录）都保存在叶子结点中。
> - m阶B+树表示了内部结点最多有m-1个关键字（或者说内部结点最多有m个子树），阶数m同时限制了叶子结点最多存储m-1个记录。
> - 内部结点中的key都按照从小到大的顺序排列，对于内部结点中的一个key，左树中的所有key都小于它，右子树中的key都大于等于它。叶子结点中的记录也按照key的大小排列。
> - 每个叶子结点都存有相邻叶子结点的指针，叶子结点本身依关键字的大小自小而大顺序链接。

那么B+树就会有这样的特性:

> - 所有关键字都出现在叶子节点的链表中（稠密索引），且链表中的关键字恰好是有序的；
> - 不可能在非叶子节点命中；
> - 非叶子节点相当于叶子节点的索引（稀疏索引），叶子节点相当于是存储（关键字）数据的数据层；
> - 更适合文件索引系统；

核心就是查询和insert操作:

查询:

> **1)** 从最小关键字起**顺序查找**;
>
> **2)** 从**根节点**开始，进行**随机查找**;
>
> ​	在查找时，若非终端节点上的关键字等于给定值，并不终止，而是继续向下直到叶子节点。因此，在`B+树`中，不管查找成功与否，每次查找都是走了一条从根到叶子节点的路径。其余同`B-树`的查找类似。

insert:显然**insert**操作只会在叶子节点上进行.

> [直接看这篇文章](https://ivanzz1001.github.io/records/post/data-structure/2018/06/16/ds-bplustree#31-b树插入示例)

和B-树的区别:

> 一棵m阶的`B+树`和m阶的`B树`的异同点在于：
>
> - 所有的**叶子节点中包含了全部关键字的信息**，即指向含有这些关键字记录的**指针**，且**叶子节点**本身依关键字的大小`自小而大`的**顺序链接**。（而`B-树`的叶子节点并没有包括全部需要查找的信息）
> - 所有的**非终端节点**可以看成是**索引**部分，节点中仅含有其子树根节点中最大（或最小）关键字。（而`B-树`的非终端节点也包含需要查找的有效信息）

一个问题:

> **`B+树`主要适用于索引操作。为什么说`B+树`比`B-树`更适合实际应用于操作系统的文件索引和数据库索引？**
>
> - `B+树`的磁盘读写代价更低: `B+树`的**内部节点并没有指向关键字具体信息的指针**。因此其内部节点相对`B-树`更小。如果把所有同一内部节点的关键字存放在同一盘块中，那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就越多。相对来说IO读写次数也就降低了。举个例子：假设磁盘中的一个盘块容纳16bytes，而一个关键字2bytes, 一个关键字具体信息指针2bytes。一棵9阶`B-树`(一个节点最多8个关键字）的内部节点需要2个盘块。而`B+树`内部节点只需要1个盘块。当需要把内部节点读入内存的时候，`B-树`就比`B+树`多一次盘块查找时间（在磁盘中就是盘片旋转时间）
> - **就是因为在寻找一个叶子节点的时候,前面的内部节点没有存储具体的信息,而仅仅是一些index,这是很好的.**
> - B+树的**查询效率更加稳定**: 由于非终节点并不是最终指向文件内容的节点，而只是叶子节点中关键字的索引。**所以任何关键字的查找必须走一条从根节点到叶子节点的路**。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。
> - **因为只有叶子节点存储了信息,所以大家要走的路是类似的.**
> - `B+树`所有的Data域在叶子节点，一般来说都会进行一个优化，就是将所有的叶子节点用指针串起来。这样遍历叶子节点就能获得全部数据，这样就能进行**区间访问**了。
> - **我们之前提到的两个指针.**

什么是聚簇索引:

> ​	在《数据库原理》里面，对**聚簇索引**的解释是： **聚簇索引的顺序就是数据的物理存储顺序**； 而对**非聚簇索引**的解释是： **索引顺序与数据物理排列无关**。正是因为如此，所以一个表最多只能有一个聚簇索引。直观上来说，聚簇索引的叶子节点就是数据节点； 而非聚簇索引的叶子节点仍然是索引节点，只不过是指向对应数据块的指针。

## 5.实现table manager 字段与表管理器 TBM

### SQL解析器

实现了以下的SQL:

```sql
<begin statement> # 开启事务,设置隔离级别
    begin [isolation level (read committedrepeatable read)]
        begin isolation level read committed

<commit statement>	# 提交事务
    commit

<abort statement>	# 回滚事务
    abort

<create statement>	# 创建table
    create table <table name>
    <field name> <field type>
    <field name> <field type>
    ...
    <field name> <field type>
    [(index <field name list>)]
        create table students
        id int32,
        name string,
        age int32,
        (index id name)

<drop statement>	# 废弃table
    drop table <table name>
        drop table students

<select statement>	# 查询语句
    select (*<field name list>) from <table name> [<where statement>]
        select * from student where id = 1
        select name from student where id > 1 and id < 4
        select name, age, id from student where id = 12

<insert statement>	# 插入语句
    insert into <table name> values <value list>
        insert into student values 5 "Zhang Yuanjia" 22

<delete statement>	# 删除
    delete from <table name> <where statement>
        delete from student where name = "Zhang Yuanjia"

<update statement>	# 更新
    update <table name> set <field name>=<value> [<where statement>]
        update student set name = "ZYJ" where id = 5

<where statement>	# 条件查询
    where <field name> (><=) <value> [(andor) <field name> (><=) <value>]
        where age > 10 or age < 3
					# 字段和表名
<field name> <table name>
    [a-zA-Z][a-zA-Z0-9_]*
					# 字段类型
<field type>
    int32 int64 string

<value>
    .*
```

解析就是parser对于输入的语句做逐字节的解析,并且切割成token,根据首个token来包装成不同的类.

### 表和字段的管理

基本的结构:

```java
/**
 * Table 维护了表结构
 * 二进制结构如下：
 * [TableName][NextTable]
 * [Field1Uid][Field2Uid]...[FieldNUid]
 */
public class Table {
    TableManager tbm;
    long uid;
    String name;
    byte status;
    long nextUid;
    List<Field> fields = new ArrayList<>();
    ...
    }
```

一个字段的结构:

```java
/**
 * field 表示字段信息
 * 二进制格式为：
 * [FieldName][TypeName][IndexUid] [字段名][类型][是否建立了index索引(有index,这个字段会直接指向索引二叉树的root)]
 * 如果field无索引，IndexUid为0
 * 字段信息直接保存在一个entry内部.
 */
public class Field {
    long uid;
    private Table tb;
    String fieldName;
    String fieldType;
    private long index;
    private BPlusTree bt;
    ...
}
```

一个database中存在多张table,我们使用链表的形式把这些table连接起来:

```txt
[TableName][NextTable]
[Field1Uid][Field2Uid]...[FieldNUid]
```

> ​	这里由于每个 **Entry** 中的数据，字节数是确定的，于是无需保存字段的个数。根据 UID 从 Entry 中读取表数据的过程和读取字段的过程类似。
>
> ​	对表和字段的操作，有一个很重要的步骤，就是计算 Where 条件的范围，目前 MYDB 的 Where 只支持两个条件的与和或。例如有条件的 Delete，计算 Where，最终就需要获取到条件范围内所有的 UID。MYDB 只支持已索引字段作为 Where 的条件。计算 Where 的范围，具体可以查看 Table 的 `parseWhere()` 和 `calWhere()` 方法，以及 Field 类的 `calExp()` 方法。
>
> ​	由于 TBM 的表管理，使用的是链表串起的 Table 结构，所以就必须保存一个链表的头节点，即第一个表的 UID，这样在 MYDB 启动时，才能快速找到表信息。
>
> ​	MYDB 使用 Booter 类和 bt 文件，来管理 MYDB 的启动信息，虽然现在所需的启动信息，只有一个：头表的 UID。**Booter 类对外提供了两个方法：load 和 update，并保证了其原子性。**
>
> ​	**update 在修改 bt 文件内容时，没有直接对 bt 文件进行修改，而是首先将内容写入一个 bt_tmp 文件中，随后将这个文件重命名为 bt 文件。以期通过操作系统重命名文件的原子性，来保证操作的原子性。**

要用一个**bt**文件保存第一个table的uid,因为我们要快速找到table的信息:

```java
// 记录第一个表的uid
public class Booter {
    public static final String BOOTER_SUFFIX = ".bt";
    public static final String BOOTER_TMP_SUFFIX = ".bt_tmp";

    String path;
    File file;
	...
}
```

最终TBM实现给server使用的所有接口:

```java
// 这里就已经是提供给最外层的Server所使用的接口了.
public interface TableManager {
    BeginRes begin(Begin begin);
    byte[] commit(long xid) throws Exception;
    byte[] abort(long xid);

    byte[] show(long xid);
    byte[] create(long xid, Create create) throws Exception;

    byte[] insert(long xid, Insert insert) throws Exception;
    byte[] read(long xid, Select select) throws Exception;
    byte[] update(long xid, Update update) throws Exception;
    byte[] delete(long xid, Delete delete) throws Exception;

    // 创建新表使用的是头插法,每次创建的时候,都要更新bt文件.
    public static TableManager create(String path, VersionManager vm, DataManager dm) {
        Booter booter = Booter.create(path);
        booter.update(Parser.long2Byte(0));
        return new TableManagerImpl(vm, dm, booter);
    }

    public static TableManager open(String path, VersionManager vm, DataManager dm) {
        Booter booter = Booter.open(path);
        return new TableManagerImpl(vm, dm, booter);
    }
}
```

## 6.最终实现软件:Server和Client的通信

> 通信是利用Socket.

### 通信

用**Package**作为一个基本的结构:

```java
public class Package {
    byte[] data;
    Exception err;

    public Package(byte[] data, Exception err) {
        this.data = data;
        this.err = err;
    }

    public byte[] getData() {
        return data;
    }

    public Exception getErr() {
        return err;
    }
}
```

发送前**编码**,收到之后会进行**解码**的操作.

基本格式:(其实就是字节数组前面加了一个1.)

```java
[Flag][data]
```

利用Encoder的类:

```java
public class Encoder {

    public byte[] encode(Package pkg) {
        if(pkg.getErr() != null) {
            Exception err = pkg.getErr();
            String msg = "Intern server error!";
            if(err.getMessage() != null) {
                msg = err.getMessage();
            }
            // 1,表明出错.
            return Bytes.concat(new byte[]{1}, msg.getBytes());
        } else {
            // 0,表示数据本身没错.
            return Bytes.concat(new byte[]{0}, pkg.getData());
        }
    }

    public Package decode(byte[] data) throws Exception {
        if(data.length < 1) {
            throw Error.InvalidPkgDataException;
        }
        if(data[0] == 0) {
            return new Package(Arrays.copyOfRange(data, 1, data.length), null);
        } else if(data[0] == 1) {
            return new Package(null, new RuntimeException(new String(Arrays.copyOfRange(data, 1, data.length))));
        } else {
            throw Error.InvalidPkgDataException;
        }
    }
}
```

> END.
