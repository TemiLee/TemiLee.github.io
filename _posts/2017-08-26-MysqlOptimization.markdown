---
layout: post
title:  "Mysql Optimization"
date:   2017-08-26 05:01:29 +0100
tags: [MYSQL]
author: Temi Lee
modifyauthor: Temi Lee
lastmodify: 2017-09-05 01:27:29
---


MYSQL 零散知识点：
```
sql 语句的长度限制：默认单条 SQL 语句大小限制wei：1M (可通过修改配置文件 my.ini 中的 max_allowed_packet = 6M
参数修改）

Mysql in 参数数量没有限制 （ Oracle 9i 中个数不能超过256,Oracle 10g个数不能超过1000 ）

```


有利于 `Mysql` 性能的几点建议:

- 字段
    - 尽量使用TINYINT、SMALLINT、MEDIUM_INT作为整数类型而非INT，如果非负则加上UNSIGNED
    - 使用枚举或整数代替字符串类型
    - 尽量使用TIMESTAMP而非DATETIME
    - 单表不要有太多字段，建议在20以内
    - 避免使用NULL字段，很难查询优化且占用额外索引空间
    - 用整型来存IP

- 索引
    - 避免在WHERE子句中对字段进行NULL值判断，否则将导致引擎放弃使用索引而进行全表扫描
    - 字符字段只建前缀索引
    - 字符字段不要做主键
    - 不用外键，由程序保证约束

- 查询SQL
    - 通过开启慢查询日志来找出较慢的SQL
    - sql语句尽可能简单：`一条sql只能在一个cpu运算；` 大语句拆小语句，减少锁时间；一条大sql可以堵死整个库
    - OR改写成IN：`OR的效率是n级别，IN的效率是log(n)级别，` in的个数建议控制在200以内
    - 不做列运算：SELECT id WHERE age + 1 = 10，任何对列的操作都将导致表扫描，它包括数据库教程函数、计算表达式等等，查询时要尽可能将操作移至等号右边
    - 不用函数和触发器，在应用程序实现
    - 避免%xxx式查询
    - 使用同类型进行比较，比如用'123'和'123'比，123和123比 (???)
    - 尽量避免在WHERE子句中使用 != 或 <> 操作符，否则将引擎放弃使用索引而进行全表扫描



- 系统调优参数
    - back_log：back_log值指出在MySQL暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中。也就是说，如果MySql的连接数据达到max_connections时，新来的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即back_log，如果等待连接的数量超过back_log，将不被授予连接资源。可以从默认的50升至500
    - wait_timeout：数据库连接闲置时间，闲置连接会占用内存资源。可以从默认的8小时减到半小时
    - max_user_connection: 最大连接数，默认为0无上限，最好设一个合理上限
    - thread_concurrency：并发线程数，设为CPU核数的两倍
    - skip_name_resolve：禁止对外部连接进行DNS解析，消除DNS解析时间，但需要所有远程主机用IP访问
    - key_buffer_size：索引块的缓存大小，增加会提升索引处理速度，对MyISAM表性能影响最大。对于内存4G左右，可设为256M或384M，通过查询show status like 'key_read%'，保证key_reads / key_read_requests在0.1%以下最好
    - innodb_buffer_pool_size：缓存数据块和索引块，对InnoDB表性能影响最大。通过查询show status like 'Innodb_buffer_pool_read%'，保证 (Innodb_buffer_pool_read_requests – Innodb_buffer_pool_reads) / Innodb_buffer_pool_read_requests越高越好
    - innodb_additional_mem_pool_size：InnoDB存储引擎用来存放数据字典信息以及一些内部数据结构的内存空间大小，当数据库对象非常多的时候，适当调整该参数的大小以确保所有数据都能存放在内存中提高访问效率，当过小的时候，MySQL会记录Warning信息到数据库的错误日志中，这时就需要该调整这个参数大小
    - innodb_log_buffer_size：InnoDB存储引擎的事务日志所使用的缓冲区，一般来说不建议超过32MB
    - query_cache_size：缓存MySQL中的ResultSet，也就是一条SQL语句执行的结果集，所以仅仅只能针对select语句。当某个表的数据有任何任何变化，都会导致所有引用了该表的select语句在Query Cache中的缓存数据失效。所以，当我们的数据变化非常频繁的情况下，使用Query Cache可能会得不偿失。根据命中率(Qcache_hits/(Qcache_hits+Qcache_inserts)*100))进行调整，一般不建议太大，256MB可能已经差不多了，大型的配置型静态数据可适当调大.`可以通过命令show status like 'Qcache_%'查看目前系统Query catch使用大小`
    - read_buffer_size：MySql读入缓冲区大小。对表进行顺序扫描的请求将分配一个读入缓冲区，MySql会为它分配一段内存缓冲区。如果对表的顺序扫描请求非常频繁，可以通过增加该变量值以及内存缓冲区大小提高其性能
    - sort_buffer_size：MySql执行排序使用的缓冲大小。如果想要增加ORDER BY的速度，首先看是否可以让MySQL使用索引而不是额外的排序阶段。如果不能，可以尝试增加sort_buffer_size变量的大小
    - read_rnd_buffer_size：MySql的随机读缓冲区大小。当按任意顺序读取行时(例如，按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，MySql会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。但MySql会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大。
    - record_buffer：每个进行一个顺序扫描的线程为其扫描的每张表分配这个大小的一个缓冲区。如果你做很多顺序扫描，可能想要增加该值
    - thread_cache_size：保存当前没有与连接关联但是准备为后面新的连接服务的线程，可以快速响应连接的线程请求而无需创建新的
    - table_cache：类似于thread_cache_size，但用来缓存表文件，对InnoDB效果不大，主要用于MyISAM