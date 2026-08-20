> 拆自 [黑马MySQL.md](./黑马MySQL.md)，原文未改。

# 索引与SQL优化

## 索引

![1772161345603](image/黑马MySQL/1772161345603.png)

### 结构

* 为什么 InnoDB 存储引擎选择使用 B+Tree 索引结构？

  * 相对于二叉树，层级更少，搜索效率高
  * 对于 B-Tree，无论是叶子节点还是非叶子节点，都会保存数据，这样导致一页中存储的键值减少，指针也跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低
  * 相对于 Hash 索引，B+Tree 支持范围匹配及排序操作
* 复杂度

  * 插入（I/O操作）：$O(\log_{n/2}(N))$，n为节点中指针的最大数量，N是被索引文件中的记录数量

### 语法

```sql
# 创建索引
create [unique|fulltext] index index_name on table_name(column_name1, column_name2, ...);
# unique: 唯一索引
# fulltext: 全文索引

# 查看索引
show index from table_name;

# 删除索引
drop index index_name on table_name;
```

### 性能分析

#### 查看执行频次

```sql
show global status lick 'Com_______';
```

* 几个重要的指标

| Variable_name | Value            |
| ------------- | ---------------- |
| Com_delete    | 删除语句执行次数 |
| Com_insert    | 插入语句执行次数 |
| Com_update    | 更新语句执行次数 |
| Com_select    | 查询语句执行次数 |

#### 慢查询日志

* 查看慢查询日志开关

```sql
show variables like '%slow_query_log%';
```

* 若未开启，需要在MySQL的配置文件中 `/etc/my.cnf`添加以下配置

```sql
# 开启慢查询日志
slow_query_log = 1

# 设置慢查询日志的时间阈值，单位为秒
long_query_time = 1
```

* 慢查询文件存放位置 `/var/lib/mysql/localhost-slow.log`
* `tail -f localhost-slow.log`实时查看慢查询日志

#### profile详情分析

* 检查profile是否开启

```sql
select @@have_profiling;
```

* 开启profile

```sql
set profiling = 1;
```

* 查看profile

```sql
show profiles;
```

输入格式如下:

| Quert_ID | Duration | Query                    |
| -------- | -------- | ------------------------ |
| 1        | 0.000123 | select * from table_name |
| 2        | 0.000123 | select * from table_name |

* 查看特定查询耗时情况

```sql
show profile for query query_id;
```

#### explain执行计划

* explain或desc命令可以获取MySQL如何执行select语句信息，包括select语句执行过程中表如何连接和连接的顺序，是否用到索引

```sql
explain select * from table_name where condition;
```

* 执行计划

| 名称                    | 含义                                                                                                                                                                                                                                                                                                                                                     |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id                      | select查询的序列号，表示查询中执行select子句或者是操作表的顺序(id相同，执行顺序从上往下；id不同，id值越大，优先级越高)                                                                                                                                                                                                                                   |
| select_type             | select的类型<br />SIMPLE：不使用表连接或者子查询<br />PRIMARY：主查询<br />UNION：UNION中的第二个或者后面的查询语句 SUBQUERY：SELECT/WHERE之后包含了子查询                                                                                                                                                                                               |
| **table**         | 进行查询的表                                                                                                                                                                                                                                                                                                                                             |
| partitions              |                                                                                                                                                                                                                                                                                                                                                          |
| **type**          | 表示数据扫描类型，性能从高到低依次为：<br />const：结果只有一条的主键/唯一索引<br />eq_ref：唯一索引<br />ref：非唯一索引<br />range：索引范围扫描<br />index：全索引扫描<br />all：全表扫描                                                                                                                                                             |
| **possible_keys** | 可能应用在这张表的索引                                                                                                                                                                                                                                                                                                                                   |
| **key**           | 实际使用的索引(没用则为NULL)                                                                                                                                                                                                                                                                                                                             |
| **key_len**       | 索引中使用的字节数，该值为索引字段最大可能长度而非实际长度                                                                                                                                                                                                                                                                                               |
| ref                     |                                                                                                                                                                                                                                                                                                                                                          |
| **rows**          | MySQL认为必须要执行查询的行数，预估值                                                                                                                                                                                                                                                                                                                    |
| filtered                | 返回结果行数占需读取行数的百分比                                                                                                                                                                                                                                                                                                                         |
| **Extra**         | 执行情况的描述和说明<br />using index condition:查找使用了索引，但是需要回表查询<br />using where;using index:查找使用了索引，需要的数据在索引列都能找到，不需要回表<br />using filesort：查询语句中包含group by操作，且无法用索引完成排序，需要用排序算法进行，效率很低<br />using temporary：使用临时表保存中间结果，常见于排序order by和分组group by |

### 使用规则

#### 最左前缀法则

* 最左前缀法则：查询从索引的最左列开始，并且不跳过中间的索引。
* 如果索引了多列（联合索引），要遵守最左前缀法则。
* 如果跳过某一列，该列后面的字段将失效
* 联合索引中，如果出现范围查询(<,>)，范围查询右侧的索引将会失效(尽量使用带等于号的查询)

#### 索引失效情况

* 对索引使用左模糊匹配
  * 左模糊无法按照索引排列的顺序查找
* 对索引使用函数
  * 索引保存的是字段原始值
  * MySQL 8.0 后，增加函数索引，可以针对函数计算结果建立索引
* 对索引进行表达式计算
  * 同上
* 对索引隐式类型转换
  * 索引字段是字符串，查询输入是整数，索引失效（**字符串转化为整形，索引失效**）
  * 索引字段是整形，查询输入是字符串，索引不失效
  * **MySQL 遇到字符串和数字比较时，会将字符串转化为数字  `select "10" > 9` 的结果为1**
* 联合索引非最左匹配
* 用or分割开的条件，如果or前的条件中的列有索引，而后面的列没有索引
* MySQL评估索引比全表扫描慢

#### SQL提示

优化数据库查询的方法，在SQL语句中添加提示，告诉MySQL如何执行查询。

```sql
# 建议 使用索引
select * from table_name use index(index_name);

# 不使用索引
select * from table_name ignore index(index_name);

# 强制使用索引
select * from table_name force index(index_name);
```

#### 覆盖索引

* 查询使用了索引，并且需要返回的列在索引中能够全部找到

#### 前缀索引

* 将字符串的一部分前缀提取出来建立索引，节约索引空间，提高效率

```sql
create index index_name on table_name(column_name(length));
```

* 选择性：不重复的索引值（基数）和数据表的记录总数的比值，选择性越高，索引的效果越好
* 计算选择性

```sql
# 1
select count(distinct column_name)/count(*) from table_name;

select count(distinct substring(column_name,left,right))/count(*) from table_name;
```

#### 单列&联合索引

* 单列索引：一个索引只包含单个列
* 联合索引：一个索引包含多个列
* 存在多个查询条件，建议使用联合索引

### 设计原则

1. 数据量较大（100万以上），查询比较频繁
2. 为常作为查询条件、排序、分组(where、order by、group by)的字段建立索引
3. 选择区分度高的列作为索引，尽量使用唯一索引
4. 对字符串类型的字段可以建立前缀索引
5. 尽量使用联合索引，避免回表
6. 控制索引数量
7. 如果索引不能存储NULL值，创建表时用NOT NULL约束

### MySQL 8.x 中的索引新特性

- **隐藏索引（不可见索引）**

  - 索引仍然维护，但优化器不会使用，常用于灰度下线索引或排查问题。
  - 主键索引不能设置为隐藏。
- **降序索引**

  - ySQL 8.x 之前虽然支持 `DESC` 语法，但底层仍是升序索引；8.x 起真正支持物理降序索引。
  - 在多列混合升/降序排序场景下可以更好利用索引，减少 `filesort`。
- **函数索引**

  - 从 8.0.13 起支持在索引中使用表达式或函数结果（如对 `LOWER(col)` 建索引），可避免因函数计算导致索引失效。

### 索引下推（Index Condition Pushdown, ICP）

- **基本思想**

  - 允许存储引擎在遍历索引时，提前执行部分 `WHERE` 条件过滤，减少回表次数和 Server 层的数据传输量。
  - 可以理解为：把一部分原本在 Server 层做的条件判断“下推”到存储引擎层。
- **典型示例**

```sql
CREATE TABLE `user` (
  `id` int NOT NULL AUTO_INCREMENT,
  `username` varchar(20) NOT NULL,
  `zipcode` varchar(20) NOT NULL,
  `birthdate` date NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_username_birthdate` (`zipcode`,`birthdate`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 查询 zipcode 为 431200 且生日在 3 月的用户
SELECT * FROM user 
WHERE zipcode = '431200' 
  AND MONTH(birthdate) = 3;
```

- **没有 ICP 时**

  - 存储引擎层先根据 `zipcode` 索引字段找到所有 `zipcode = '431200'` 的主键 ID，然后回表读取完整记录。
  - Server 层再根据 `MONTH(birthdate) = 3` 条件做二次筛选。
- **启用 ICP 后**

  - 存储引擎层在扫描二级索引时就可以根据 `MONTH(birthdate) = 3` 过滤掉大部分不符合条件的记录，再做回表。
- **适用场景简记**

  - 适用于 InnoDB / MyISAM，引擎层能利用到二级索引时收益最大。
  - 常见于访问类型为 `range` / `ref` / `eq_ref` / `ref_or_null` 的范围查询。
  - 子查询临时表、无索引的临时表、部分存储过程场景下不会使用 ICP。

## SQL优化

### 插入数据

* insert语句优化

  * 批量插入(500-1000条)
  * 手动提交事务
  * 主键顺序插入
* 大批量数据插入——load

```sql
# 客户端连接服务器时加上参数
mysql --loacal-infile -u root -p
# 设置全局参数，开启从本地加载文件导入数据的开关
set global local_infile = 1;
# 检查开关
select @@local_infile
# 执行load指令
# 字段用逗号分隔，每行用'\n'分隔
load data local infile 'root/sql1.log' into table 'tb_user' fields terminated by ',' lines terminated by '\n'
```

### 主键优化

* InnoDB引擎中，表数据是根据主键顺序存放的（索引组织表）
* 页分裂、页合并
* 设计原则
  * 尽量降低主键长度
  * 插入数据时顺序插入、使用自增逐渐
  * 尽量避免对主键的修改

### order by优化

| 排序           | 说明                                                |
| -------------- | --------------------------------------------------- |
| using filesort | 表索引/全表扫描，读取满足条件的数据，在缓冲区中排序 |
| using index    | 有序索引直接返回有序数据，效率高                    |

* 根据排序字段建立索引，遵循最左前缀法则
* 尽量使用覆盖索引
* 多字段+同时存在升降序，在创建索引时要说明
* 不可避免需要filesort时，可以增大排序缓冲区大小

### group by优化

* 分组也可用索引，遵循最左前缀法则

### limit优化

* 大数据情况下效率很低
* `limit offset, size` 比 `limit size` 慢，offset越大越慢（`limit size` 等价于 `limit 0, size`）
* 原因：Server层会从引擎层取出 offset+size 条数据，再丢弃前offset条，只保留size条
  * 走主键索引：取出完整行再丢弃，`select *` 时拷贝整行开销更大
  * 走非主键索引：每条还要回表；offset过大时优化器可能改成全表扫描（type=ALL）
* 覆盖索引 + 子查询优化（延迟关联）：先只查主键，再回表取完整行

```sql
# 走主键：先定位起始id，再按主键范围取
select * from page
where id >= (select id from page order by id limit 6000000, 1)
order by id limit 10;

# 走非主键：子查询只取id（覆盖索引，不回表），再按id关联取整行
select * from page t1,
  (select id from page order by user_name limit 6000000, 100) t2
where t1.id = t2.id;
```

* 上述优化只能减少回表/拷贝整行，仍要扫描并丢弃offset条，offset极大时收益有限

### 深度分页优化

* 深度分页：offset到百万、千万级时，LIMIT性能急剧下降；MySQL/ES都无法根治，只能规避
* 全表导出（同步到ES/Hive）
  * 不要一次性`select *`，也不要用`limit offset, size`循环翻页
  * 按主键分批，用上一批最大id作为下一批条件（seek/游标分页）
  * 每次走主键定位后再向后扫描，无论翻到哪一批耗时都稳定

```sql
select * from page where id > last_id order by id limit 100;
```
* 用户分页展示
  * 搜索/筛选优先用ES，并限制结果总数（如1万以内）
  * 必须用MySQL时，也限制返回数量（如1k以内），才能勉强支持跳页
  * 更好的做法：只支持上一页/下一页或瀑布流，配合`start_id`分批获取，查询速度与页码无关
* 数据量长期只有千级，直接用`limit offset, size`即可

### count优化

* count()：统计符合条件的记录中，参数不为NULL的行数
  * `count(字段)`：该字段为NULL的行不计入
  * `count(1)` / `count(*)`：参数恒不为NULL，统计全部行数（`count(*)` 会被转成 `count(0)`，与 `count(1)` 无性能差异）
* MyISAM：表的meta信息中存了row_count，无where条件时直接返回，O(1)
  * 带where后也要扫描，与InnoDB无区别
* InnoDB：支持事务+MVCC，同一时刻不同会话看到的行数可能不同，无法维护单一row_count，只能逐行扫描计数
* 执行过程（InnoDB）：Server层维护count变量，循环向引擎读记录，参数不为NULL则+1
  * 有二级索引时，优化器优先扫key_len最小的二级索引（比聚簇索引小，I/O更少）
  * 无二级索引时才扫主键索引
  * `count(主键)`：要读出主键值再判断是否为NULL
  * `count(1)` / `count(*)`：不读字段值，读到一行就+1
  * `count(普通字段)`：全表扫描，效率最差
* 效率：`count(字段)` < `count(主键id)` < `count(1)` ≈ `count(*)`
  * `count(主键)` 略慢于 `count(1)` 只发生在表上没有二级索引、只能扫聚簇索引时
  * 有二级索引时，`count(主键)` / `count(1)` / `count(*)` 执行过程相同
* 大表 `count(*)` 优化
  * 不需要精确值：`show table status` 或 `explain` 的rows做估算
  * 需要精确值：单独一张计数表，插入/删除时同步维护计数字段
* 尽量给表建二级索引；不要用 `count(字段)` 统计总行数，若要统计该字段非NULL行数，给该字段建二级索引

### update优化

* update语句加的是行锁
* 根据索引字段更新，否则行锁会变成表锁
