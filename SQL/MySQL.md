# MySQL

## 基础

### 查询语言 

- DDL Data Definition Language：定义数据库对象（库、表、列等）
- DML Data Manipulation Language： 数据操作 增删插改 insert update delete
- DCL Data Control Language： 定义访问权限和安全级别，创建用户等 grant
- DQL Data Query Language： 查询记录 select from where

#### DDL

CREATE DATABASE dbname; 创建数据库

SHOW DATABASE; 查看数据库

USE dbname; 切换数据库

DROP DATABASE dbname; 删除库

CREATE TABLE tbname; 建表

DESC tbname; 查看表信息

DROP TABLE tbname; 删除表 可通过日志恢复

TRUNCATE TABLE tbname; 删除表 不可恢复

ALTER TABLE tbname (MODIFY、ADD、DROP、CHANGE) col (信息) 修改表中字段

ALTER TABLE tbname RENAME new-tbname 修改表名



#### DML

INSERT INTO tbname (field1,field2) VALUES(value1, value2); 插入

UPDATE tbname SET field=value WHERE; 更新

 DELETE FROM tbname WHERE; 删除



#### DQL

SELECT FROM WHERE 查询

DISTINCT 去重

ORDER BY 排序

LIMIT 限制

聚合:

	- sum(), count(), max(), min()
	- GROUP BY
	- WITH
	- HAVING 对分类后的结果再进行条件过滤，在where之后

表连接： left-t JOIN right-t ON left-t.col1=right-t.col2

	- 内连接：选出两张表中互相匹配的记录 (默认) INNER JOIN
 - 外连接：不仅选出匹配的记录，也选不匹配的记录 OUTER JOIN
   	- 左外连接，包含左表未匹配记录 LEFT JOIN
   	- 右外连接，包含右表未匹配记录 RIGHT JOIN

子查询：IN、NOT IN、=、!=、EXISTS、NOT EXISTS

​	SELECT * FROM tb WHERE col IN (SELECT t.c FORM t)

联合查询： UNION（去重）、 UNION ALL（不去重）

​	SELECT * FROM ta UNION SELECT * FROM tb



#### DCL

数据库管理员使用



### 帮助

? contents 所有可供查询的分类

? 【category】分类下的内容

? KEYWORDS 关键词的文档



## 数据类型

数值、日期和时间、字符串

### 数值类型

- 严格数值类型
  - INTEGER
  - SMALLINT
  - DECIMAL
  - NUMERIC
- 近似数值类型
  - FLOAT
  - REAL
  - DOUBLE PRECISION
- 扩展数据类型
  - TINYINT
  - MEDIUMINT
  - BIGINT
  - BIT

### 日期时间类型

| YEAR      | YYYY                | 1byte  |
| --------- | ------------------- | ------ |
| TIME      | HH:MM:SS            | 3bytes |
| DATE      | YYYY-MM-DD          | 3bytes |
| DATETIME  | YYYY-MM-DD HH:MM:SS | 8bytes |
| TIMESTAMP | YYYY-MM-DD HH:MM:SS | 4bytes |



### 字符串类型

![img](https://mmbiz.qpic.cn/mmbiz_png/libYRuvULTdUqmGNFicK1sjvE6sqzQdibohhZz22VFtC48SHwolRsTSG1J3nw2b6ETLkJuabr1DayU1diaYfqOvprA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## 进阶

### 存储引擎

功能：

- 并发
- 支持事务
- 完整性约束
- 物理存储
- 支持索引
- 性能帮助

#### MyISAM

- 不支持`事务`操作，ACID 的特性也就不存在了，这一设计是为了性能和效率考虑的。

- 不支持`外键`操作，如果强行增加外键，MySQL 不会报错，只不过外键不起作用。

- MyISAM 默认的锁粒度是`表级锁`，所以并发性能比较差，加锁比较快，锁冲突比较少，不太容易发生死锁的情况。

- MyISAM 会在磁盘上存储三个文件，文件名和表名相同，扩展名分别是 `.frm(存储表定义)`、`.MYD(MYData,存储数据)`、`MYI(MyIndex,存储索引)`。这里需要特别注意的是 MyISAM 只缓存`索引文件`，并不缓存数据文件。

- MyISAM 支持的索引类型有 `全局索引(Full-Text)`、`B-Tree 索引`、`R-Tree 索引`

  Full-Text 索引：它的出现是为了解决针对文本的模糊查询效率较低的问题。

  B-Tree 索引：所有的索引节点都按照平衡树的数据结构来存储，所有的索引数据节点都在叶节点

  R-Tree索引：它的存储方式和 B-Tree 索引有一些区别，主要设计用于存储空间和多维数据的字段做索引,目前的 MySQL 版本仅支持 geometry 类型的字段作索引，相对于 BTREE，RTREE 的优势在于范围查找。

- 数据库所在主机如果宕机，MyISAM 的数据文件容易损坏，而且难以恢复。

- 增删改查性能方面：SELECT 性能较高，适用于查询较多的情况

#### InnoDB

自从 MySQL 5.1 之后，默认的存储引擎变成了 InnoDB 存储引擎，相对于 MyISAM，InnoDB 存储引擎有了较大的改变，它的主要特点是

- 支持事务操作，具有事务 ACID 隔离特性，默认的隔离级别是`可重复读(repetable-read)`、通过`MVCC（并发版本控制）`来实现的。能够解决`脏读`和`不可重复读`的问题。
- InnoDB 支持外键操作。
- InnoDB 默认的锁粒度`行级锁`，并发性能比较好，会发生死锁的情况。
- 和 MyISAM 一样的是，InnoDB 存储引擎也有 `.frm文件存储表结构` 定义，但是不同的是，InnoDB 的表数据与索引数据是存储在一起的，都位于 B+ 数的叶子节点上，而 MyISAM 的表数据和索引数据是分开的。
- InnoDB 有安全的日志文件，这个日志文件用于恢复因数据库崩溃或其他情况导致的数据丢失问题，保证数据的一致性。
- InnoDB 和 MyISAM 支持的索引类型相同，但具体实现因为文件结构的不同有很大差异。
- 增删改查性能方面，果执行大量的增删改操作，推荐使用 InnoDB 存储引擎，它在删除操作时是对行删除，不会重建表。

#### 如何选择

在实际开发过程中，我们往往会根据应用特点选择合适的存储引擎。

- MyISAM：如果应用程序通常以检索为主，只有少量的插入、更新和删除操作，并且对事物的完整性、并发程度不是很高的话，通常建议选择 MyISAM 存储引擎。
- InnoDB：如果使用到外键、需要并发程度较高，数据一致性要求较高，那么通常选择 InnoDB 引擎，一般互联网大厂对并发和数据完整性要求较高，所以一般都使用 InnoDB 存储引擎。



### 索引









