---
title: python3 pandas to_sql填坑
date: 2020-07-15 18:33:15
tags:
        - 数据分析
        - python
categories: python
---

## 为什么要使用to_sql方法
<!-- more -->
- 表结构如下：
    ```sql
    CREATE TABLE `my_balance` (
      `id` int(11) NOT NULL AUTO_INCREMENT,
      `balance` decimal(20,4) NOT NULL COMMENT '余额',
      `account` varchar(30) DEFAULT NULL COMMENT '账户',
      `charges` decimal(10,4) NOT NULL COMMENT '交易手续费/每笔',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8 COMMENT='账户表'
    ```

- 现在要向my_balance表中插入一下数据，下面我们来看三段代码：

1. 普通pymysql sql insert 写法（强烈不推介这种写法）
    ```python
import pymysql.cursors
# 建立数据库连接
db = pymysql.connect(host='127.0.0.1', user='root',
                     passwd='root', db='tushare', charset='utf8')
balance = 11500.0000
account = 8888
charges = 0.0005
# insert 语句
sql = f"insert into `tushare`.`my_balance` ( `balance`, `account`, `charges`) values ( '{balance}', '{account}', '{charges}')"
# 执行新增或更新数据操作,返回受影响的行数
cursor = db.cursor()
res_row = cursor.execute(sql)
# 提交事务
db.commit()
# 关闭链接
db.close()
    ```
**优点：代码比较直观易懂，适合初学者读
缺点：每次一个insert语句都要复制一堆代码，代码复用性为0.**

2. pandas拼接insert写法（这种写法有BUG）
    ```python
import pandas as pd
import pymysql.cursors


def init_db():
    # 建立数据库连接
    db = pymysql.connect(host='127.0.0.1', user='root',
                         passwd='root', db='tushare', charset='utf8')
    return db


def insert_data_pandasType(table_name, valuses, db):
    '''
    向数据库中新增数据
    table_name：库名
    valuses：字段名称和对应数据，组成的数组
    '''
    pop = pd.Series(valuses)
    into = ','.join(pop.keys().values)
    val = ','.join(str(v) for v in pop.values)
    sql = f"insert into {table_name}({into}) values({val})"
    cursor = db.cursor()
    return cursor.execute(sql)

# 执行SQL
db = init_db()
values = {'balance': 11400.0000, 'account': '8866.ZH', 'charges': 0.0005}
insert_data_pandasType('my_balance', values, db)
db.commit()
db.close()
    ```
   
    'account': '8866.ZH'问题出在这里，错误如下：
    
    You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'ZH,0.0005)

所以只能存储没带标点符号的字符串~！

so,优缺点就不说了，这BUG太明显，压根不能用~！

3. to_sql写法

```python
import pandas as pd
from sqlalchemy import create_engine
engine = create_engine(
    "mysql+pymysql://root:root@localhost:3306/tushare?charset=utf8")
df = pd.DataFrame({'balance': [10000.000], 'account':
                   ['13579.SH'], 'charges': [0.0005]})
df.to_sql(name='my_balance', con=engine, if_exists='append',
            index=False, index_label='id')
```

你没有看错，就这么几行就搞定了insert了，是不是非常爽~！但是有几点需要注意下：

**错误信息：Execution failed on sql ‘SELECT name FROM sqlite_master WHERE type=‘table’ AND name=?;’: not all arguments converted during string formatting

数据库链接不再使用pymysql，而改用sqlalchemy，con=engine 而不是con=db 官方文档**

最最最坑的地方出现了，大家注意：

错误：No module named ‘MySQLdb’

百度到的代码，基本99%都是这么写的：

    mysql+mysqldb://root:root@localhost:3306/tushare?charset=utf8

但是，如果按照如上写法，在python3.6（我的python版本）环境下会出现找不到mysqldb模块错误！
正确的写法如下，因为python3将mysqldb改为pymysql了！！！

    mysql+pymysql://root:root@localhost:3306/tushare?charset=utf8
    

    if_exists 的参数说明
    fail的意思如果表存在，啥也不做
    replace的意思，如果表存在，删了表，再建立一个新表，把数据插入
    append的意思，如果表存在，把数据插入，如果表不存在创建一个表！！
    index=False, index_label=‘id’



如果你数据库中主键名称为index则不用设置这两项，如果不是，请按照主键名称设置！

附上一篇文章：to_sql与普通insert性能对比测试

## 关于to_sql事务问题
看到这里，有编程经验的朋友，一定会发现，使用to_sql的代码中，没有看到事务，在有多个insert或者update的情况时，如果回滚提交呢？从下面这篇文章中寻找答案吧~！

[关于to_sql事务](https://capelastegui.wordpress.com/2018/05/21/commit-and-rollback-with-pandas-dataframe-to_sql/)

不能翻墙的小伙伴请看下面代码示例：
```python
from sqlalchemy import create_engine
engine = create_engine(
    "mysql+pymysql://root:root@localhost:3306/tushare?charset=utf8")
df = pd.DataFrame({'balance': [10000.000], 'account':
                   ['13579.SH'], 'charges': [0.0005]})
df1 = pd.DataFrame({'balance': [1234.000], 'account':
                    ['17886.SH'], 'charges': [0.0005]})
# 开始事物
with engine.begin() as conn:
    df.to_sql(name='my_balance', con=conn, if_exists='append',
              index=False, index_label='id')
    # 故意出现错误的代码，测试事物回滚
    df1.to_sql(name='my_balance', con=conn, if_exists='append')

```

————————————————
*原文链接：https://blog.csdn.net/qnloft/java/article/details/87979937*
