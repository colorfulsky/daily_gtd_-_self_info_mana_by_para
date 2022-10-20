> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/youzi_yun/article/details/97621355)

最近用 presto 引擎查数据，发现了语法和 MYSQL，PG 的稍许区别，写此文章留念~~

### 文章目录

*   [1 数据类型](#1__5)
*   [2 SELECT 搜索查询](#2_SELECT__18)
*   *   [2.1 with 子句](#21_with__51)
    *   [2.2 GROUP BY 子句](#22_GROUP_BY__82)
    *   *   [2.2.1 GROUP BY](#221_GROUP_BY_84)
        *   [2.2.2 GROUPING SETS](#222_GROUPING_SETS_95)
        *   [2.2.3 CUBE](#223_CUBE_150)
        *   [2.2.4 ROLLUP](#224_ROLLUP_173)
        *   [2.2.5 group by, clue, rollup 区别](#225_group_by_clue_rollup_188)
        *   [2.2.6 group sets, clue, rollup 组合使用](#226_group_sets_clue_rollup__214)
        *   [2.2.7 ALL 和 DISTINCT 的使用](#227_ALL__DISTINCT__239)
    *   [2.3 HAVING](#23_HAVING_287)
    *   [2.4 UNION，INTERSECT， EXCEPT](#24_UNIONINTERSECT_EXCEPT_300)
    *   *   [2.4.1 UNION](#241_UNION_322)
        *   [2.4.2 INTERSECT](#242_INTERSECT_336)
        *   [2.4.3 EXCEPT](#243_EXCEPT_346)
    *   [2.5 ORDER BY 排序](#25_ORDER_BY__357)
    *   [2.6 LIMIT](#26_LIMIT_370)
*   [3 其他常用 SQL](#3_SQL_375)
*   *   [3.1 SCHEMA 操作](#31_SCHEMA__376)
    *   *   [3.1.1 重命名 SCHEMA](#311__SCHEMA_378)
        *   [3.1.2 创建 SCHEMA](#312__SCHEMA_386)
        *   [3.1.2 删除 SCHEMA](#312__SCHEMA_405)
    *   [3.2 TABLE 操作](#32_TABLE__420)
    *   *   [3.2.1 创建 TABLE](#321__TABLE_421)
        *   [3.2.2 查看建表语句](#322__463)
        *   [3.2.3 修改 TABLE](#323__TABLE_469)
        *   [3.2.4 删除 TABLE](#324__TABLE_482)
        *   [3.2.5 CREATE TABLE AS 使用搜索结果建新表](#325_CREATE_TABLE_AS__496)
    *   [3.3 ANALYZE 统计表和列信息](#33_ANALYZE__513)
    *   [3.4 CALL 调用存储过程](#34_CALL__524)
    *   [3.5 START TRANSACTION，ROLLBACK，COMMIT 事务](#35_START_TRANSACTIONROLLBACKCOMMIT__540)
    *   [3.6 DELETE 删除](#36_DELETE__567)
    *   [3.8 PREPARE, EXECUTE, DEALLOCATE PREPARE](#38_PREPARE_EXECUTE_DEALLOCATE_PREPARE_582)
    *   [3.9 INSERT 插入数据](#39__INSERT__605)
    *   [3.10 查看数据仓库目录，SCHEMA, TABLE，COLUMN](#310_SCHEMA_TABLECOLUMN_625)

  

presto 是 Facebook 主持下运营的开源的分布式 SQL 查询引擎，用于针对各种大小（从千兆字节到千兆字节）的数据源运行交互式分析查询。本文主要介绍常用 SQL，具体可参考官方文档：

[https://prestodb.github.io/docs/current/](https://prestodb.github.io/docs/current/)

1 数据类型
======

*   Boolean: true, false
*   Integer: tinyint, smallint, integer, bigint
*   Floating-Point: real, double
*   Fixed-Precision：DECIMAL
*   String：varchar, char, varbinary, json
*   Date and Time: date, time, time with time zone, timestamp, timestamp with time zone, interval year to month, interval day to second
*   Structural: array, map, row
*   Network Address: ipaddress
*   HyperLogLog: HyperLogLog, P4HyperLogLog
*   Quantile Digest: QDigest

2 SELECT 搜索查询
=============

```
[ WITH with_query [, ...] ]
SELECT [ ALL | DISTINCT ] select_expr [, ...]
[ FROM from_item [, ...] ]
[ WHERE condition ]
[ GROUP BY [ ALL | DISTINCT ] grouping_element [, ...] ]
[ HAVING condition]
[ { UNION | INTERSECT | EXCEPT } [ ALL | DISTINCT ] select ]
[ ORDER BY expression [ ASC | DESC ] [, ...] ]
[ LIMIT [ count | ALL ] ]

以下是这些参数可能的格式
- from_item
    table_name [ [ AS ] alias [ ( column_alias [, ...] ) ] ]
    from_item join_type from_item [ ON join_condition | USING ( join_column [, ...] ) ]

- join_type
    [ INNER ] JOIN
    LEFT [ OUTER ] JOIN
    RIGHT [ OUTER ] JOIN
    FULL [ OUTER ] JOIN
    CROSS JOIN
    
- grouping_element
    ()
    expression
    GROUPING SETS ( ( column [, ...] ) [, ...] )
    CUBE ( column [, ...] )
    ROLLUP ( column [, ...] )
```

2.1 with 子句
-----------

with 定义要在查询中使用的命名关系

```
WITH x AS (SELECT a, MAX(b) AS b FROM t GROUP BY a)
SELECT a, b FROM x;

等同于
SELECT a, b
FROM (
  SELECT a, MAX(b) AS b FROM t GROUP BY a
) AS x;


也可以用于多条定义
WITH
  t1 AS (SELECT a, MAX(b) AS b FROM x GROUP BY a),
  t2 AS (SELECT a, AVG(d) AS d FROM y GROUP BY a)
SELECT t1.*, t2.*
FROM t1
JOIN t2 ON t1.a = t2.a;


也可以链式使用
WITH
  x AS (SELECT a FROM t),
  y AS (SELECT a AS b FROM x),
  z AS (SELECT b AS c FROM y)
SELECT c FROM z;
```

2.2 GROUP BY 子句
---------------

### 2.2.1 GROUP BY

当在 select 语句中使用 group by 时，所有输出表达式都必须是聚合函数或 group by 子句中存在的列。

```
按字段nationkey分组，并查出各组数量，以下两种写法是一致的，by 2 代表以输出表达式第2列做分组
SELECT count(*), nationkey FROM customer GROUP BY 2;
SELECT count(*), nationkey FROM customer GROUP BY nationkey;

也可以不输出指定分组的列，如下
SELECT count(*) FROM customer GROUP BY mktsegment;
```

### 2.2.2 GROUPING SETS

可以指定多个列进行分组，结果列中不属于分组列的将被设置为 NUll。  
具有复杂分组语法 (GROUPING SETS, CUBE 或 ROLLUP) 的查询只从基础数据源读取一次，而使用 UNION ALL 的查询将读取基础数据三次。这就是当数据源不具有确定性时，使用 UNION ALL 的查询可能会产生不一致的结果的原因。

```
有一个表：SELECT * FROM shipping;

origin_state | origin_zip | destination_state | destination_zip | package_weight
--------------+------------+-------------------+-----------------+----------------
 California   |      94131 | New Jersey        |            8648 |             13
 California   |      94131 | New Jersey        |            8540 |             42
 New Jersey   |       7081 | Connecticut       |            6708 |            225
 California   |      90210 | Connecticut       |            6927 |           1337
 California   |      94131 | Colorado          |           80302 |              5
 New York     |      10002 | New Jersey        |            8540 |              3
(6 rows)

SELECT origin_state, origin_zip, destination_state, sum(package_weight)
FROM shipping
GROUP BY GROUPING SETS (
    (origin_state),
    (origin_state, origin_zip),
    (destination_state));
    
这个的查询在逻辑上等同于多个分组查询的union all：

SELECT origin_state, NULL, NULL, sum(package_weight)
FROM shipping GROUP BY origin_state

UNION ALL

SELECT origin_state, origin_zip, NULL, sum(package_weight)
FROM shipping GROUP BY origin_state, origin_zip

UNION ALL

SELECT NULL, NULL, destination_state, sum(package_weight)
FROM shipping GROUP BY destination_state;


结果如下：
 origin_state | origin_zip | destination_state | _col0
--------------+------------+-------------------+-------
 New Jersey   | NULL       | NULL              |   225
 California   | NULL       | NULL              |  1397
 New York     | NULL       | NULL              |     3
 California   |      90210 | NULL              |  1337
 California   |      94131 | NULL              |    60
 New Jersey   |       7081 | NULL              |   225
 New York     |      10002 | NULL              |     3
 NULL         | NULL       | Colorado          |     5
 NULL         | NULL       | New Jersey        |    58
 NULL         | NULL       | Connecticut       |  1562
(10 rows)
```

### 2.2.3 CUBE

为给定的列生成所有可能的分组，比如 (origin_state, destination_state) 的可能分组为 (origin_state, destination_state),  
(origin_state),  
(destination_state),  
()

```
SELECT origin_state, destination_state, sum(package_weight)
FROM shipping
GROUP BY CUBE (origin_state, destination_state);

等同于

SELECT origin_state, destination_state, sum(package_weight)
FROM shipping
GROUP BY GROUPING SETS (
    (origin_state, destination_state),
    (origin_state),
    (destination_state),
    ());
```

### 2.2.4 ROLLUP

为给定的列集生成部分可能的分类汇总

```
SELECT origin_state, origin_zip, sum(package_weight)
FROM shipping
GROUP BY ROLLUP (origin_state, origin_zip);

等同于

SELECT origin_state, origin_zip, sum(package_weight)
FROM shipping
GROUP BY GROUPING SETS ((origin_state, origin_zip), (origin_state), ());
```

### 2.2.5 group by, clue, rollup 区别

比如按字段 1，2，3 来分组，group 只会聚合 1，2，3 分组，clue 会展示所有层级分组，rollup 只会展示 1 以下所有分组  
用列表标识会更直观

group by

<table><thead><tr><th>1</th><th>2</th><th>3</th></tr></thead><tbody></tbody></table>

clue

<table><thead><tr><th>1</th><th>2</th><th>3</th></tr></thead><tbody><tr><td>1</td><td>2</td><td></td></tr><tr><td>1</td><td></td><td></td></tr><tr><td></td><td>2</td><td>3</td></tr><tr><td></td><td>2</td><td></td></tr><tr><td></td><td></td><td>3</td></tr><tr><td></td><td></td><td></td></tr></tbody></table>

rollup

<table><thead><tr><th>1</th><th>2</th><th>3</th></tr></thead><tbody><tr><td>1</td><td>2</td><td></td></tr><tr><td>1</td><td></td><td></td></tr><tr><td></td><td></td><td></td></tr></tbody></table>

### 2.2.6 group sets, clue, rollup 组合使用

```
SELECT origin_state, destination_state, origin_zip, sum(package_weight)
FROM shipping
GROUP BY
    GROUPING SETS ((origin_state, destination_state)),
    ROLLUP (origin_zip);

等同于

SELECT origin_state, destination_state, origin_zip, sum(package_weight)
FROM shipping
GROUP BY
    GROUPING SETS ((origin_state, destination_state)),
    GROUPING SETS ((origin_zip), ());
    
逻辑上等同于

SELECT origin_state, destination_state, origin_zip, sum(package_weight)
FROM shipping
GROUP BY GROUPING SETS (
    (origin_state, destination_state, origin_zip),
    (origin_state, destination_state));
```

### 2.2.7 ALL 和 DISTINCT 的使用

在复杂的组合搜索中 ALL 和 DISTINCT 作用很强大，ALL 代表全部输出，DISTINCT 代表去重后输出

```
SELECT origin_state, destination_state, origin_zip, sum(package_weight)
FROM shipping
GROUP BY ALL
    CUBE (origin_state, destination_state),
    ROLLUP (origin_state, origin_zip);

等同于

SELECT origin_state, destination_state, origin_zip, sum(package_weight)
FROM shipping
GROUP BY GROUPING SETS (
    (origin_state, destination_state, origin_zip),
    (origin_state, origin_zip),
    (origin_state, destination_state, origin_zip),
    (origin_state, origin_zip),
    (origin_state, destination_state),
    (origin_state),
    (origin_state, destination_state),
    (origin_state),
    (origin_state, destination_state),
    (origin_state),
    (destination_state),
    ());
```

```
SELECT origin_state, destination_state, origin_zip, sum(package_weight)
FROM shipping
GROUP BY DISTINCT
    CUBE (origin_state, destination_state),
    ROLLUP (origin_state, origin_zip);

等同于

SELECT origin_state, destination_state, origin_zip, sum(package_weight)
FROM shipping
GROUP BY GROUPING SETS (
    (origin_state, destination_state, origin_zip),
    (origin_state, origin_zip),
    (origin_state, destination_state),
    (origin_state),
    (destination_state),
    ());
```

2.3 HAVING
----------

HAVING 与聚合函数和 GROUP BY 一起使用，以过滤 GROUP BY。

```
从customer表中选择帐户余额大于指定值的组
SELECT count(*), mktsegment, nationkey,
       CAST(sum(acctbal) AS bigint) AS totalbal
FROM customer
GROUP BY mktsegment, nationkey
HAVING sum(acctbal) > 5700000
ORDER BY totalbal DESC;
```

2.4 UNION，INTERSECT， EXCEPT
---------------------------

```
query UNION [ALL | DISTINCT] query
query INTERSECT [DISTINCT] query
query EXCEPT [DISTINCT] query

- all: 最终结果集中包括所有行
- distinct: 组合结果集中只包含唯一的行
- 如果两者都未指定，则行为默认为distinct。

区别
- intersect或except不支持all参数。
- 除非通过括号明确指定顺序，否则将从左到右处理多个集合操作
- INTERSECT 优先级高于UNION和EXCEPT
    比如：
    A UNION B INTERSECT C EXCEPT D 
    等同于
    A UNION (B INTERSECT C) EXCEPT D
```

### 2.4.1 UNION

```
以下结果返回13和42
SELECT 13 UNION SELECT 42;

以下结果返回13和42
SELECT 13 UNION SELECT * FROM (VALUES 42, 13);

以下结果返回13，42 和 13
SELECT 13 UNION ALL SELECT * FROM (VALUES 42, 13);
```

### 2.4.2 INTERSECT

使用 INTERSECT 代表返回的最终结果集为：INTERSECT 之前的结果与 INTERSECT 查出的结果取交集

```
比如以下结果返回 13：
SELECT * FROM (VALUES 13, 42)
INTERSECT
SELECT 13;
```

### 2.4.3 EXCEPT

使用 EXCEPT 代表返回的最终结果集中排除 EXCEPT 查出的结果

```
比如以下结果返回 42：
SELECT * FROM (VALUES 13, 42)
EXCEPT
SELECT 13;
```

2.5 ORDER BY 排序
---------------

一般放到 SELECT 语句的最后，或在 HAVING 之前， 默认 ASC NULLS LAST，

```
ORDER BY expression [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...]

- ASC： 默认从小到大正序排序
- DESC： 倒序
- NULLS FIRST： NULL 值最大
- NULLS LAST： 默认 NULL 值最小
```

2.6 LIMIT
---------

limit 5 代表只输出 5 条结果，limit all 代表全部输出，没有数量限制

3 其他常用 SQL
==========

3.1 SCHEMA 操作
-------------

### 3.1.1 重命名 SCHEMA

```
ALTER SCHEMA name RENAME TO new_name

eg: 将 web 重命名为 traffic
ALTER SCHEMA web RENAME TO traffic
```

### 3.1.2 创建 SCHEMA

```
CREATE SCHEMA [ IF NOT EXISTS ] schema_name
[ WITH ( property_name = expression [, ...] ) ]

- IF NOT EXISTS 比较安全，防止SCHEMA已存在的报错
- WITH 可以给SCHEMA添加属性，通过以下SQL可以查看所有属性：
SELECT * FROM system.metadata.schema_properties

eg: 
创建一个名为web的SCHEMA
CREATE SCHEMA web
创建一个在hive目录下名为sales的SCHEMA
CREATE SCHEMA hive.sales
如果名为traffic的SCHEMA不存在那么创建它
CREATE SCHEMA IF NOT EXISTS traffic
```

### 3.1.2 删除 SCHEMA

```
DROP SCHEMA [ IF EXISTS ] schema_name

- IF EXISTS 可以防止SCHEMA不存在时的报错

eg:
删除名为web的SCHEMA
DROP SCHEMA web
如果名为web的SCHEMA存在，则删除它
DROP SCHEMA IF EXISTS sales
```

3.2 TABLE 操作
------------

### 3.2.1 创建 TABLE

```
CREATE TABLE [ IF NOT EXISTS ]
table_name (
  { column_name data_type [ COMMENT comment ] [ WITH ( property_name = expression [, ...] ) ]
  | LIKE existing_table_name [ { INCLUDING | EXCLUDING } PROPERTIES ] }
  [, ...]
)
[ COMMENT table_comment ]
[ WITH ( property_name = expression [, ...] ) ]


- IF NOT EXISTS 比较安全，防止TABLE已存在的报错
- WITH 可以给TABLE添加属性：
通过以下SQL可以查看所有表属性
SELECT * FROM system.metadata.table_properties
通过以下SQL可以查看所有列属性
SELECT * FROM system.metadata.column_properties
- COMMENT 为表添加注释
- LIKE 可用于在新表中包含来自现有表的所有列。可以指定多个LIKE子句，允许从多个表复制列。


eg:
创建一个名为orders的表， 并添加表注释
CREATE TABLE orders (
  orderkey bigint,
  orderstatus varchar,
  totalprice double,
  orderdate date
)
COMMENT 'A table to keep track of orders.'
WITH (format = 'ORC')

创建一个名为bigger_orders的表，并包含orders表的所有字段
CREATE TABLE bigger_orders (
  another_orderkey bigint,
  LIKE orders,
  another_orderdate date
)
```

### 3.2.2 查看建表语句

```
SHOW CREATE TABLE table_name
```

### 3.2.3 修改 TABLE

```
重命名
ALTER TABLE name RENAME TO new_name
添加字段
ALTER TABLE name ADD COLUMN column_name data_type [ COMMENT comment ] [ WITH ( property_name = expression [, ...] ) ]
删除字段
ALTER TABLE name DROP COLUMN column_name
重命名字段
ALTER TABLE name RENAME COLUMN column_name TO new_column_name
```

### 3.2.4 删除 TABLE

```
DROP TABLE  [ IF EXISTS ] table_name

- IF EXISTS 可以防止TABLE不存在时的报错

eg:
删除名为web的TABLE
DROP TABLE web
如果名为web的TABLE存在，则删除它
DROP TABLE IF EXISTS sales
```

### 3.2.5 CREATE TABLE AS 使用搜索结果建新表

```
CREATE TABLE [ IF NOT EXISTS ] table_name [ ( column_alias, ... ) ]
[ COMMENT table_comment ]
[ WITH ( property_name = expression [, ...] ) ]
AS query
[ WITH [ NO ] DATA ]

eg:
创建一个新表orders_column_aliased，字段order_date, total_price分别来自于表orders的字段orderdate, totalprice
CREATE TABLE orders_column_aliased (order_date, total_price)
AS
SELECT orderdate, totalprice
FROM orders
```

3.3 ANALYZE 统计表和列信息
-------------------

```
统计表和列信息，目前该语句只支持Hive connector。
ANALYZE table_name [ WITH ( property_name = expression [, ...] ) ]

- WITH 可以给查询添加特定属性：
通过以下SQL可以查看所有可以使用的属性
SELECT * FROM system.metadata.analyze_properties
```

3.4 CALL 调用存储过程
---------------

```
调用存储过程，有些连接器，比如 PostgreSQL Connector,有自己的存储过程，不能通过call调用
CALL procedure_name ( [ name => ] expression [, ...] )


eg:
传入必传参数，调用存储过程
CALL test(123, 'apple');
传入命名参数，调用存储过程
CALL test(name => 'apple', id => 123);
不需要传参，调用存储过程
CALL catalog.schema.test();
```

3.5 START TRANSACTION，ROLLBACK，COMMIT 事务
----------------------------------------

```
开启事务 （默认为READ WRITE读写事务）
START TRANSACTION [ mode [, ...] ]
回滚事务
ROLLBACK [ WORK ]
提交事务
COMMIT [ WORK ]

- model 是以下的一种：
ISOLATION LEVEL { READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE }
READ { ONLY | WRITE }

eg:
开始一个事务，默认为READ WRITE读写事务
START TRANSACTION;
开始一个可重复读事务
START TRANSACTION ISOLATION LEVEL REPEATABLE READ;
开始一个读写事务
START TRANSACTION READ WRITE;
开始一个提交读/不可重复读、只读事务
START TRANSACTION ISOLATION LEVEL READ COMMITTED, READ ONLY;
开始一个读写串行化事务
START TRANSACTION READ WRITE, ISOLATION LEVEL SERIALIZABLE;
```

3.6 DELETE 删除
-------------

有的连接器对于删除有限制或者是不支持， 需要看具体的连接器文档

```
DELETE FROM table_name [ WHERE condition ]

eg:
删除lineitem表里的shipmode = 'AIR'的行
DELETE FROM lineitem WHERE shipmode = 'AIR';

删除所有orders里的数据
DELETE FROM orders;
```

3.8 PREPARE, EXECUTE, DEALLOCATE PREPARE
----------------------------------------

```
声明一个名为statement_name的SQL
PREPARE statement_name FROM statement
执行名为statement_name的声明
EXECUTE statement_name [ USING parameter1 [ , parameter2, ... ] ]
删除名为statement_name的声明
DEALLOCATE PREPARE statement_name

eg:
准备一条sql语句
PREPARE my_select2 FROM
SELECT name FROM nation WHERE regionkey = ? and nationkey < ?;

执行这个语句，并加入？的参数
EXECUTE my_select2 USING 1, 3;

以上两句相当于执行下面这条SQL：
SELECT name FROM nation WHERE regionkey = 1 AND nationkey < 3;
```

3.9 INSERT 插入数据
---------------

```
INSERT INTO table_name [ ( column [, ... ] ) ] query

eg：
往orders表里插入数据，数据全部来源于new_orders表
INSERT INTO orders SELECT * FROM new_orders;

往cities表里插入一条数据
INSERT INTO cities VALUES (1, 'San Francisco');

往cities表里插入多条数据
INSERT INTO cities VALUES (2, 'San Jose'), (3, 'Oakland');

指定字段名往nation表里插入多条数据，如果有字段未指定，则用字段默认值， 没有默认值就是null
INSERT INTO nation (nationkey, name, regionkey, comment)
VALUES (26, 'POLAND', 3, 'no comment');
```

3.10 查看数据仓库目录，SCHEMA, TABLE，COLUMN
----------------------------------

```
查看数据仓库第一层目录
SHOW CATALOGS [ LIKE pattern ]

查看所有SCHEMAS
SHOW SCHEMAS [ FROM catalog ] [ LIKE pattern ]

查看schema里的表
SHOW TABLES [ FROM schema ] [ LIKE pattern ]

查看表的字段类型，描述, 搜索出来的结果有：column type extra comment
DESCRIBE table_name
相当于 SHOW COLUMNS from table_name
```
