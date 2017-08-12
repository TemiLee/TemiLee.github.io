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