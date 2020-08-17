---
title: Pandas到sql在重复主键上失败
date: 2020-08-17 11:46:37
tags:
        - 数据分析
        - pandas
categories: python
---

### 问题：
    使用pandasdf.to_sql()函数追加到现有表。
    设置了if_exists='append'，但是表有主键。    
    在尝试对现有表进行append操作时，希望执行与insert ignore等效的操作，这样可以避免重复的条目错误。  
    对于pandas，这是可能的吗？还是需要编写一个显式查询？

<!-- more -->

### 目前解决方案：

```python
# 方法一：mysql
insert_sql = """
    insert into tableA(ename,cname,sex,role,company)
    SELECT %s,%s,%s,%s 
    FROM DUAL
    WHERE NOT EXISTS (SELECT ename FROM tableA
    WHERE ename=%s and cname=%s)
    """
```

```python
# 方法二：pandas
import pandas as pd
from sqlalchemy.exc import SQLAlchemyError, IntegrityError

df = pd.read_excel("data.xlsx", sheet_name="people").fillna("")
df.columns = ["ename", "cname", "sex", "role", "company"]

# 主键冲突直接写入失败。
df.to_sql("tableA", db_mysql.engine, if_exists="append", index=False)

# 将df切分单条形式的df入库。
for i in range(len(df)):
    try:
        df.iloc[i:i+1].to_sql("tableA", db_mysql.engine, if_exists="append",index=False)
    except IntegrityError:
        print("已存在", df.iloc[i:i+1][["ename", "cname"]])
```

