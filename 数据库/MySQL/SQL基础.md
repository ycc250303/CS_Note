> 拆自 [黑马MySQL.md](./黑马MySQL.md)，原文未改。

# SQL基础

## 概述

* 关系型数据库（RDBMS）
  * 概念：建立在关系模型基础上，由多张相互连接的二维表组成的数据库
  * 特点：
    * 使用表存储数据，格式统一，便于维护
    * 使用SQL语言进行操作，标准统一，使用方便

## SQL

* SQL分类

| 分类 | 全称                       | 说明                                                   |
| ---- | -------------------------- | ------------------------------------------------------ |
| DDL  | Data Definition Language   | 数据定义语言，用来定义数据库对象(数据库，表，字段)     |
| DML  | Data Manipulation Language | 数据操作语言，用来对数据库表中的数据进行增删改         |
| DQL  | Data Query Language        | 数据查询语言，用来查询数据库中表的记录                 |
| DCL  | Data Control Language      | 数据控制语言，用来创建数据库用户、控制数据库的访问权限 |

### DDL

* 数据库操作

```sql
# 查询
# 查询所有数据库
show databases;
# 查询当前数据库
show database();

# 创建
create database [if not exists] <数据库名> [default charset 字符集] [collate 排序规划]

# 删除
drop database [if exists] 数据库名;

# 使用
use 数据库名;
```

* 表操作-创建&查询

```sql
# 查询表
# 查询当前数据库所有表
show tables;

# 查询表结构
desc 表名;

# 查询指定表的建表语句
show create table 表名;

# 创建表
create table <表名>(
    <字段1> 字段1类型 [comment 字段1注释],
    <字段2> 字段2类型 [comment 字段2注释],
    <字段3> 字段3类型 [comment 字段3注释],
    ...,
    <字段n> 字段n类型 [comment 字段n注释]
) [comment 表注释];
```

* 表操作-修改&删除

```sql
# 添加字段
alter table 表名 add 字段名 类型（长度） [comment 注释] [约束];

# 修改数据类型
alter table 表名 modify 字段名 新数据类型(长度);

# 修改字段名和字段类型
alter table 表名 change 旧字段名 新字段名 类型(长度) [comment 注释] [约束];

# 修改表名
alter table 表名 rename to 新表名;

# 删除表
drop table [if exists] 表名;

# 删除表并重建
truncate table 表名;
```

* 数据类型
  * 数值类型表

| 类型        | 大小    | 有符号(SIGNED)范围                                  | 无符号(UNSIGNED)范围                                  | 描述           |
| ----------- | ------- | --------------------------------------------------- | ----------------------------------------------------- | -------------- |
| TINYINT     | 1 byte  | (-128, 127)                                         | (0, 255)                                              | 小整数值       |
| SMALLINT    | 2 bytes | (-32768, 32767)                                     | (0, 65535)                                            | 大整数值       |
| MEDIUMINT   | 3 bytes | (-8388608, 8388607)                                 | (0, 16777215)                                         | 大整数值       |
| INT/INTEGER | 4 bytes | (-2147483648, 2147483647)                           | (0, 4294967295)                                       | 大整数值       |
| BIGINT      | 8 bytes | (-2^63, 2^63-1)                                     | (0, 2^64-1)                                           | 极大整数值     |
| FLOAT       | 4 bytes | (-3.402823466E+38, 3.402823466351E+38)              | 0和(1.175494351E-38, 3.402823466E+38)                 | 单精度浮点数值 |
| DOUBLE      | 8 bytes | (-1.7976931348623157E+308, 1.7976931348623157E+308) | 0和(2.2250738585072014E-308, 1.7976931348623157E+308) | 双精度浮点数值 |
| DECIMAL     | -       | 依赖于M(精度)和D(标度)                              | 依赖于M(精度)和D(标度)                                | 精确定点数     |

* 字符串类型表

| 分类       | 类型       | 大小                  | 描述                         |
| ---------- | ---------- | --------------------- | ---------------------------- |
| 字符串类型 | CHAR       | 0-255 bytes           | 定长字符串                   |
|            | VARCHAR    | 0-65535 bytes         | 变长字符串                   |
|            | TINYBLOB   | 0-255 bytes           | 不超过256字符的二进制数据    |
|            | TINYTEXT   | 0-255 bytes           | 短文本字符串                 |
|            | BLOB       | 0-65,535 bytes        | 二进制形式的长文本数据       |
|            | TEXT       | 0-65,535 bytes        | 长文本数据                   |
|            | MEDIUMBLOB | 0-16,777,215 bytes    | 二进制形式的中等长度文本数据 |
|            | MEDIUMTEXT | 0-16,777,215 bytes    | 中等长度文本数据             |
|            | LONGBLOB   | 0-4,294,967,295 bytes | 二进制形式的极大文本数据     |
|            | LONGTEXT   | 0-4,294,967,295 bytes | 极大文本数据                 |

* 日期类型表

| 分类     | 类型      | 大小    | 范围                                       | 格式                | 描述                     |
| -------- | --------- | ------- | ------------------------------------------ | ------------------- | ------------------------ |
| 日期类型 | DATE      | 3 bytes | 1000-01-01 至 9999-12-31                   | YYYY-MM-DD          | 日期值                   |
|          | TIME      | 3 bytes | -838:59:59 至 838:59:59                    | HH:MM:SS            | 时间值或持续时间         |
|          | YEAR      | 1 byte  | 1901 至 2155                               | YYYY                | 年份值                   |
|          | DATETIME  | 8 bytes | 1000-01-01 00:00:00 至 9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值         |
|          | TIMESTAMP | 4 bytes | 1970-01-01 00:00:01 至 2038-01-19 03:14:07 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值(时间戳) |

#### 字段类型选择常见问题

- **整数类型 `UNSIGNED` 的作用**

  - 不允许负值的无符号整数，可以将正整数的上限提高一倍。
  - 对于从 0 开始递增的 ID 列，使用 `UNSIGNED` 非常适合。
- **`VARCHAR(10)` 和 `VARCHAR(100)` 的差异**

  - 存储相同字符串时，占用磁盘空间基本一致。
  - 但在内存中操作时，通常会按照字段定义长度分配内存，过大的长度会带来额外内存开销。
- **`DECIMAL` vs `FLOAT/DOUBLE`**

  - `DECIMAL` 是定点数，适合存储精确小数（如金额），避免浮点误差。
  - `FLOAT/DOUBLE` 是浮点数，只能存近似值，主要用于对精度要求不高、但计算性能要求高的场景。
- **`DATETIME` vs `TIMESTAMP`**

  - `DATETIME` 不含时区信息，占 8 字节，时间范围更大（到 9999 年），常用于业务逻辑上的“绝对时间”。
  - `TIMESTAMP` 带时区语义，占 4 字节，范围较小（到 2038 年），更适合多时区或审计场景，但有时区转换开销。
- **`NULL` vs `''`（空字符串）**

  - `NULL` 表示“缺失/未知”，不等于任何值，比较运算结果为 `NULL`，需要用 `IS NULL` / `IS NOT NULL` 判断。
  - `''` 是已知的空字符串，参与比较和聚合：`COUNT(col)` 会统计它，`SUM` 会视为 0。
- **MySQL 中的布尔类型**

  - 没有真正的 `BOOLEAN` 类型，通常使用 `TINYINT(1)` 存储布尔值，约定 0 / 1 表示假 / 真。
- **手机号用 `INT` 还是 `VARCHAR` 存储？**

  - 推荐使用 `VARCHAR`：
    - 手机号不参与算术运算；
    - 可能包含前导 0、国家区号等，用整数容易丢失信息或被错误格式化。

### DML

* 插入

```sql
# 给指定字段添加数据
insert into <表名> (<字段名1>, <字段名2>, ...) values (<值1>, <值2>, ...);

# 给全部字段添加数据
insert into <表名> values (<值1>, <值2>, ...);

# 批量添加数据
# 方式1: 指定字段
insert into <表名> (<字段名1>, <字段名2>, ...) 
values 
(<值1>, <值2>, ...),
(<值1>, <值2>, ...),
(<值1>, <值2>, ...);

# 方式2: 全部字段
insert into <表名> 
values 
(<值1>, <值2>, ...),
(<值1>, <值2>, ...),
(<值1>, <值2>, ...);
```

* 更新

```sql
# 修改数据
update 表名 set 字段名1 = 值1,字段名2 = 值2,...[where 条件];
```

* 删除

```sql
delete from 表名 [where 条件];
# delete 语句不能删除某个字段的值
```

### DQL

* DQL 用于查询数据库表中的记录

```sql
# dql - 基本查询结构+执行顺序
# 4
select  
    <字段列表>
# 1
from
    <表名列表>
# 2
where
    <条件列表>
# 3
group by
    <分组字段列表>
having
    <分组后条件列表>
# 5
order by
    <排序字段列表>
# 6
limit
    <分页参数>;
```

* 基础查询

```sql
# dql - 查询示例
# 1. 查询多个字段
# 方式1: 指定字段
select <字段1>, <字段2>, <字段3> from <表名>;
# 方式2: 所有字段
select * from <表名>;

# 2. 设置别名
select 
    <字段1> [as] <别名1>,
    <字段2> [as] <别名2>
from <表名>;

# 3. 去除重复记录
select distinct <字段列表> from <表名>;
```

* 条件查询

```sql
# dql - where条件查询
# 1. 语法
select <字段列表> from <表名> where <条件列表>;
```

比较运算符表

| 运算符              | 功能     | 示例                            |
| ------------------- | -------- | ------------------------------- |
| >                   | 大于     | `where age > 18`              |
| >=                  | 大于等于 | `where score >= 60`           |
| <                   | 小于     | `where price < 100`           |
| <=                  | 小于等于 | `where quantity <= 10`        |
| =                   | 等于     | `where name = '张三'`         |
| <> 或 !=            | 不等于   | `where status != 0`           |
| BETWEEN ... AND ... | 在范围内 | `where age between 18 and 30` |
| IN(...)             | 在列表中 | `where id in (1,3,5)`         |
| LIKE                | 模糊匹配 | `where name like '张%'`       |
| IS NULL             | 是NULL   | `where email is null`         |

逻辑运算符表

| 运算符    | 功能 | 示例                                       |
| --------- | ---- | ------------------------------------------ |
| AND 或 && | 并且 | `where age > 18 and status = 1`          |
| OR 或\|\| | 或者 | `where role = 'admin' or role = 'super'` |
| NOT 或 !  | 非   | `where not is_deleted`                   |

优先级：NOT>AND>OR

* 聚合函数

将一列数据作为一个整体，进行纵向计算
常见聚合函数：

| 函数  | 功能     |
| ----- | -------- |
| count | 统计数量 |
| max   | 最大值   |
| min   | 最小值   |
| avg   | 平均值   |
| sum   | 求和     |

```sql
select 聚合函数（字段列表） from 表名;
```

* 分组查询

```sql
select <字段列表> 
from <表名> 
[where <条件>] 
group by <分组字段名> 
[having <分组后过滤条件>];
```

* where与having区别

  * 执行时机不同:
    * where在分组之前过滤,不满足条件不参与分组
    * having在分组之后过滤,对分组结果进行筛选
  * 判断条件不同:
    * where不能使用聚合函数
    * having可以使用聚合函数
* 执行顺序：where>聚合函数>having
* 分组之后，查询的字段一般为聚合函数和分组字段，查询其他字段无意义
* 排序查询

```sql
# 排序方式:asc-升序（默认），desc-降序

select 字段列表 from 表名 order by 字段1 排序方式1,字段2 排序方式2;
```

* 分页查询

```sql
# 起始索引从0开始，起始索引 = （查询页码-1）* 每页显示记录数
# 如果查询的是第一页数据，起始索引可以省略，直接简写为limit 10
select 字段列表 from 表名 limit 起始索引,查询记录数;
```

### DCL

* DCL用于管理数据库用户、控制数据库的访问权限
* 用户管理

```sql
# dcl - 用户管理
# 1. 查询用户
use mysql;
select * from user;

# 2. 创建用户
create user '<用户名>'@'<主机名>' identified by '<密码>';

# 3. 修改用户密码
alter user '<用户名>'@'<主机名>' identified with mysql_native_password by '<新密码>';

# 4. 删除用户
drop user '<用户名>'@'<主机名>';
```

* 权限控制

常见权限

| 权限                | 说明               |
| ------------------- | ------------------ |
| ALL, ALL PRIVILEGES | 所有权限           |
| SELECT              | 查询数据           |
| INSERT              | 插入数据           |
| UPDATE              | 修改数据           |
| DELETE              | 删除数据           |
| ALTER               | 修改表             |
| DROP                | 删除数据库/表/视图 |
| CREATE              | 创建数据库/表      |

```sql
# 查询用户'admin'@'localhost'的权限
show grants for 'admin'@'localhost';

# 授予用户'dev'@'%'对test_db.orders表的SELECT, INSERT权限
grant select, insert on test_db.orders to 'dev'@'%';

# 撤销用户'guest'@'localhost'对mydb.*的所有权限
revoke all privileges on mydb.* from 'guest'@'localhost';
```
