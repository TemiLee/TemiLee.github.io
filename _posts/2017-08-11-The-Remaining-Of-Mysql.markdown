---
layout: post
title:  "The Remaining Of Mysql"
date:   2017-08-10 16:54:13
tags: MYSQL
author: Temi Lee
---

有关MySql的一些扩展工具和优化策略

<br/>

**Explain:** 查看SQL语句的执行策略

**eg:** 
{% highlight java %}
explain select count(1) as num ,d.band as band from (  select a.id as id ,c.band as band from d_sup_goods_crawlings_log a,         d_sup_goods_crawlings_log b,         d_bi_goods_band c ,d_sup_goods_crawlings d          WHERE  a.crawlings_id = b.crawlings_id and a.crawlings_id = d.id and d.goods_id = c.goods_id   AND a.jd_price < b.jd_price           ) d group by d.band ;`
{% endhighlight %}

{% highlight java %}

+----+-------------+------------+------+----------------------+--------------+---------+----------------------+---------+----------------------------------------------------+
| id | select_type | table      | type | possible_keys        | key          | key_len | ref                  | rows    | Extra                                              |
+----+-------------+------------+------+----------------------+--------------+---------+----------------------+---------+----------------------------------------------------+
|  1 | PRIMARY     | <derived2> | ALL  | NULL                 | NULL         | NULL    | NULL                 | 1627290 | Using temporary; Using filesort                    |
|  2 | DERIVED     | c          | ALL  | NULL                 | NULL         | NULL    | NULL                 |   20090 | NULL                                               |
|  2 | DERIVED     | d          | ref  | PRIMARY,idx_goods_id | idx_goods_id | 4       | suppliers.c.goods_id |       1 | Using where; Using index                           |
|  2 | DERIVED     | b          | ALL  | idx_crawlings_id     | NULL         | NULL    | NULL                 |      12 | Using where; Using join buffer (Block Nested Loop) |
|  2 | DERIVED     | a          | ALL  | idx_crawlings_id     | NULL         | NULL    | NULL                 |      12 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+------------+------+----------------------+--------------+---------+----------------------+---------+----------------------------------------------------+
5 rows in set (0.13 sec)
 
{% endhighlight %}

**﻿id:** 表示语句执行的顺序

    ID如果相同，可以认为是一组，从上往下顺序执行
    在每组中，其中ID越大，优先级越高，越早执行

**select_type:** 查询类型
- ﻿simple 它表示简单的select,没有union和子查询
- ﻿primary 嵌套查询中最外面的一层
- ﻿union ﻿联合查询的第二个或者说是后面那一个

**﻿table:** 行所用的表，有别名展示别名

**﻿type:** ﻿连接类型,性能从好到坏依次为:
- ﻿system 表仅有一行,const类型的特列
- ﻿const 表最多有一行匹配,表示使用﻿primary key 或者unique索引,因为只有一行数据所以很快
- ﻿eq_ref mysql手册: 
> ﻿对于每个来自于前面的表的行组合，从该表中读取一行。这可能是最好的联接类型，除了const类型。它用在一个索引的所有部分被联接使用并且索引是UNIQUE或PRIMARY KEY<br/>
   {% highlight java %}
   explain select * from d_sup_goods_crawlings a , d_sup_goods_crawlings_log b  where a.id = b.crawlings_id ;
   +----+-------------+-------+--------+------------------+---------+---------+--------------------------+------+-------+
   | id | select_type | table | type   | possible_keys    | key     | key_len | ref                      | rows | Extra |
   +----+-------------+-------+--------+------------------+---------+---------+--------------------------+------+-------+
   |  1 | SIMPLE      | b     | ALL    | idx_crawlings_id | NULL    | NULL    | NULL                     |   12 | NULL  |
   |  1 | SIMPLE      | a     | eq_ref | PRIMARY          | PRIMARY | 4       | suppliers.b.crawlings_id |    1 | NULL  |
   +----+-------------+-------+--------+------------------+---------+---------+--------------------------+------+-------+
   
   //mysql会选取表数据较少的一个表使用全表扫描，另在一张表使用eq_ref 链接，
   {% endhighlight %}
- ﻿ref 
   > ﻿对于每个来自于前面的表的行组合，所有有匹配索引值的行将从这张表中读取。如果联接只使用键的最左边的前缀，或如果键不是UNIQUE或PRIMARY KEY（换句话说，如果联接不能基于关键字选择单个行的话），则使用ref。如果使用的键仅仅匹配少量行，该联接类型是不错的。
- ﻿ref_or_null
   > 表现同﻿ref，但是添加了MySQL可以专门搜索包含NULL值的行。在解决子查询中经常使用该联接类型的优化。
- ﻿index_merge
   > ﻿该联接类型表示使用了索引合并优化方法。在这种情况下，key列包含了使用的索引的清单，key_len包含了使用的索引的最长的关键元素。
- ﻿range ﻿给定范围内的检索(in语句)
- ﻿index 和all类似，﻿除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小,并且索引也有可能换存在内存中。
- ﻿ ALL ,执行表扫描
**﻿possible_keys:** 可能用到的索引
**﻿keys:** 查询使用的索引
**﻿key_len:** 索引的长度
**﻿ref:** ﻿使用哪个列或常数与key一起从表中选择行
**﻿rows** ﻿MYSQL执行查询的行数，﻿数值越大越不好，说明没有用好索引
**﻿Extra:** ﻿MySQL解决查询的详细信息
    - ﻿using index:﻿只使用索引树中的信息而不需要进一步搜索读取实际的行来检索表中的信息,即索引包含了要检索的所有信息。
       {% highlight java %}
       explain select goods_id  from d_sup_goods where goods_id = 122 ;
       +----+-------------+-------------+-------+----------------------------------+---------+---------+-------+------+-------------+
       | id | select_type | table       | type  | possible_keys                    | key     | key_len | ref   | rows | Extra       |
       +----+-------------+-------------+-------+----------------------------------+---------+---------+-------+------+-------------+
       |  1 | SIMPLE      | d_sup_goods | const | PRIMARY,user_goods_on_sale_index | PRIMARY | 3       | const |    1 | Using index |
       +----+-------------+-------------+-------+----------------------------------+---------+---------+-------+------+-------------+
       1 row in set (0.03 sec)
       
       explain select goods_id,goods_name  from d_sup_goods where goods_id = 122 ;
       +----+-------------+-------------+-------+----------------------------------+---------+---------+-------+------+-------+
       | id | select_type | table       | type  | possible_keys                    | key     | key_len | ref   | rows | Extra |
       +----+-------------+-------------+-------+----------------------------------+---------+---------+-------+------+-------+
       |  1 | SIMPLE      | d_sup_goods | const | PRIMARY,user_goods_on_sale_index | PRIMARY | 3       | const |    1 | NULL  |
       +----+-------------+-------------+-------+----------------------------------+---------+---------+-------+------+-------+
       1 row in set (0.13 sec)
       
       {% endhighlight %}

    - ﻿using where:
    - ﻿ Using index for group-by:
    
    
***

**﻿查看索引**

    mysql> show index from tblname;
    mysql> show keys from tblname;

{% highlight java %}
    mysql> show index from d_sup_goods \G
    *************************** 1. row ***************************
            Table: d_sup_goods
       Non_unique: 0
         Key_name: PRIMARY
     Seq_in_index: 1
      Column_name: goods_id
        Collation: A
      Cardinality: 474851
         Sub_part: NULL
           Packed: NULL
             Null:
       Index_type: BTREE
          Comment:
    Index_comment:  
    *************************** 2. row ***************************
            Table: d_sup_goods
       Non_unique: 1
         Key_name: user_goods_on_sale_index
     Seq_in_index: 1
      Column_name: goods_id
        Collation: A
      Cardinality: 474851
         Sub_part: NULL
           Packed: NULL
             Null:
       Index_type: BTREE
          Comment:
    Index_comment:
    *************************** 3. row ***************************
            Table: d_sup_goods
       Non_unique: 1
         Key_name: user_goods_on_sale_index
     Seq_in_index: 2
      Column_name: is_delete
        Collation: A
      Cardinality: 474851
         Sub_part: NULL
           Packed: NULL
             Null:
       Index_type: BTREE
          Comment:
    Index_comment:

{% endhighlight %}

- ﻿Table: 表名称
-  Non_unique 唯一索引:0 ，非唯一索引:1
- ﻿Key_name: ﻿索引的名称
- ﻿Seq_in_index:﻿联合索引中的列序列号，从1开始。
- ﻿Column_name: 索引使用的列名称
- ﻿Collation: 索引的存储方式，﻿值‘A’（升序）或NULL（无分类）
- ﻿Cardinality: 索引的可用性(﻿索引中唯一值的数目的估计值)
- ﻿Sub_part: 如果列是部分被编入索引，﻿则为被编入索引的字符的数目。如果整列被编入索引，则为NULL。
- ﻿Packed: 索引的压缩方式，﻿如果没有被压缩，则为NULL。
- ﻿Null: 索引含有null:﻿YES
- ﻿Index_type: 索引的类型(﻿BTREE, FULLTEXT, HASH, RTREE)

**SHOW ENGINES :** 查看mysql支持的存储引擎
{% highlight java %}
    +--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
    | Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
    +--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
    | MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
    | MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
    | CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
    | BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
    | MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
    | FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
    | ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
    | InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
    | PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
    +--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
    9 rows in set (0.00 sec)
{% endhighlight %}

- Engine 引擎名称
- Suuport YSE表示支持，DEFAULT表示为默认引擎
- Commnet 描述信息
- Transaction  是否支持事务
- XA 是否支持分布式事务
- Savepoint 是否支持保存点，支持事务的引擎才支持保存点

