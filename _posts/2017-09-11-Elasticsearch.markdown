---
layout: post
title:  "JDK 1.8 Feature"
date:   2017-09-06 05:01:29 +0100
tags: [JAVA]
author: Temi Lee
<!-- modifyauthor: Temi Lee -->
<!-- lastmodify: 2017-08-08 01:27:29 -->
published: false
---


Elasticsearch的RESTfulAPI
GET用来获得请求对象的当前 状态，POST来改变对象的当前状态，PUT创建一个对象，而DELETE销毁对象，另外还有个HEAD 请求仅仅用来获取对象的基础信息。

GET http://127.0.0.1:9200/ : 这个命令用来获取Elasticsearch的基本信息。
GET http://127.0.0.1:9200/_cluster/state/nodes/ : 这个命令获取集群中节点的信息。
POST http://127.0.0.1:9200/_cluster/nodes/_shutdown : 这个命令向集群中所有节点发送一个shutdown请求。

curl -XGET http://127.0.0.1:9200/_cluster/health?pretty
curl -XPOST http://127.0.0.1:9200/_cluster/nodes/_shutdown
curl –XPOST http://127.0.0.1:9200/_cluster/nodes/BlrmMvBdSKiCeYGsiHijdg/_shutdown


curl -XPUT http://localhost:9200/blog/article/1 -d '{"title":"New version of Elasticsearch released!", "content": "Version 1.0 released today!", "tags": ["announce", "elasticsearch", "release"] }'
