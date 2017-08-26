---
layout: post
title:  "Mysql Optimization"
date:   2017-08-26 05:01:29 +0100
tags: [MYSQL]
author: Temi Lee
<!-- modifyauthor: Temi Lee -->
<!-- lastmodify: 2017-08-08 01:27:29 -->
---

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
    