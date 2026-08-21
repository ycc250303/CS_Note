> 拆自 [黑马MySQL.md](./归档/黑马MySQL.md)（已归档，不再维护），原文未改。事务原理见 [事务.md](./事务.md)。

# InnoDB引擎

## 逻辑存储结构

* 表空间（ibd文件）：一个mysql可以对应多个表空间，用于存储记录、索引等数据
* 段：分为数据段、索引段、回滚段，InnoDB是索引组织表，数据段是B+树的子节点、索引段是B+树的非子节点
* 区：表空间的单元结构，每个区大小为1M。默认情况页大小为16K，一个区有64个连续的页
* 页：InnoDB引擎磁盘管理的最小单元。为保证页的连续性，每次从磁盘申请4-5个区
* 行：数据按照行进行存放

![1761366885846](image/黑马MySQL/1761366885846.png)

## 内存架构

![1772460218997](image/黑马MySQL/1772460218997.png)

### 左侧为内存结构

* Buffer Pool（缓冲池）：缓存磁盘上经常操作的真实数据，curd时，先操作缓冲池的数据，然后再以一定频率刷新到磁盘，减少磁盘IO
  * 缓冲池以Page页为单位
  * free page：空闲页，未被使用
  * change page：被使用页，数据没有被修改过
  * dirty page：脏页，数据被修改过但是与磁盘中的不一致
* Change Buffer（更改缓冲区）：针对于非唯一二级索引页，执行DML语句时，如果数据也不在缓冲池中，不会直接操作磁盘，而是将数据变更存在更改缓冲区中，未来数据被读取时，再将数据合并恢复到缓冲池中，刷新到磁盘中。
  * 二级索引通常是非唯一的，并且插入顺序相对随机，删除和更新都可能影响索引树中不相邻的二级索引，每次都操作磁盘会造成大量IO
* Log Buffer（日志缓冲区）：保存要写入磁盘中log日志的数据，默认大小16MB，日志会定期刷新到磁盘中
  * `innodb_log_buffer_size`：日志缓冲区大小（`show variables like '%log_bnuffer_size%'`）
  * `innodb_flush_log_at_trx_commit`：日志刷新磁盘时机
    * 0:每秒将日志写入并刷新到磁盘一次
    * 1:日志在每次事务提交时写入磁盘
    * 2:日志在每次事务提交后写入，每秒刷新到磁盘
* Adaptive Hash Index（自适应哈希索引）：优化对缓冲池的查询，引擎会监控对表上个各索引页的查询，如果发现哈希索引可以提升速度，则建立哈希索引（系统自动完成）

### 右侧为磁盘结构

* System TableSpace（系统表空间）：更改缓冲区的存储区域，如果表是在系统表空间而不是表文件/通用表空间创建的，也可能包含存储数据
  * 参数：`innodb_data_file_path`
* File-Per-Table Tablespaces（独立表空间）：每个表的文件表空间包含单个InnoDB表的数据和引擎索引，并存储在文件系统上的单个数据文件中
  * 参数：`innodb_file_per_table`
* General Tablespaces（通用表空间）：创建表时，可以指定该表空间
  * 创建： `create tablespace 表空间名 add datafile 文件名 engine = engine_name`
  * 使用：`create table xxx tablepace 关联的表空间 `
* Undo Tablespaces（撤销表空间）：用于存储undo log日志，默认创建两个大小为16MB的
* Temporary Tablespaces（临时表空间）：存储用户创建的临时表等数据
* Doublewrite Buffer Files（双写缓冲区）：引擎将数据页从Buffer Pool刷新到磁盘前，先将数据页写入这里，便于系统异常时回复
  * #ib_16384_0.dblwr
  * #ib_16384_1 .dblwr
* Redo Log（重做日志）：用于实现事务的持久性。
  * ib_logfile0
  * ib_logfile1

## 后台线程

![1761370962670](image/黑马MySQL/1761370962670.png)

* 将缓冲池数据在合适时机刷新到磁盘中
* 分类
  * Master Thread：核心后台线程，负责调度其他线程，还负责将缓冲池的数据异步刷新到磁盘中，and脏页刷新，合并插入缓存，undo页回收
  * IO Thread：负责IO请求回调（引擎使用了大量AIO处理IO请求）
    ![1761371132861](image/黑马MySQL/1761371132861.png)
  * Purge Thread：回收事务以及提交的undo log
  * Page Cleaner Thread：协助Master Thread刷新脏页到磁盘

## 与 MyISAM 区别

### 索引

| 项目         | MyISAM   | InnoDB   |
| ------------ | -------- | -------- |
| 索引结构     | B+Tree   | B+Tree   |
| 主键索引     | 非聚簇   | 聚簇     |
| 叶子节点     | 数据地址 | 整行数据 |
| 二级索引存储 | 数据地址 | 主键值   |
| 是否回表     | ❌       | ✅       |
| 数据与索引   | 分离     | 一体     |
