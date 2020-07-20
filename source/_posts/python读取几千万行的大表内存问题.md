---
title: python读取几千万行的大表内存问题
date: 2020-07-20 18:14:25
tags: 数据库
categories: python
---
Python导数据的时候，需要在一个大表上读取很大的结果集。

<!-- more -->

如果用传统的方法，Python的内存会爆掉，传统的读取方式默认在内存里缓存下所有行然后再处理，内存容易溢出

### 解决的方法：

1. 使用SSCursor(流式游标)，避免客户端占用大量内存。(这个cursor实际上没有缓存下来任何数据，它不会读取所有所有到内存中，它的做法是从储存块中读取记录，并且一条一条返回给你。)

2. 使用迭代器而不用fetchall,即省内存又能很快拿到数据。
```
import MySQLdb.cursors
 
conn = MySQLdb.connect(host='ip地址', user='用户名', passwd='密码', db='数据库名', port=3306,
   charset='utf8', cursorclass = MySQLdb.cursors.SSCursor)
cur = conn.cursor()
cur.execute("SELECT * FROM bigtable");
row = cur.fetchone()
while row is not None:
    do something
    row = cur.fetchone()
 
cur.close()
conn.close()
```

### 需要注意的是:

1. 因为SSCursor是没有缓存的游标,结果集只要没取完，这个conn是不能再处理别的sql，包括另外生成一个cursor也不行的。

如果需要干别的，请另外再生成一个连接对象。

2. 每次读取后处理数据要快，不能超过60s，否则mysql将会断开这次连接，也可以修改 SET NET_WRITE_TIMEOUT = xx 来增加超时间隔。
