---
title: python中map的排序以及取出map中取最大最小值
date: 2020-07-20 18:06:46
tags: map函数
categories: python
---

**map排序：**
<!-- more -->
```
1.按key排序：
items=dict.items()
items.sort()
 
sorted(dict.items(),key=lambda x:x[0],reverse=False)
 
2.按value排序
sorted(dict.items(),key=lambda x:x[1],reverse=False)
 
(ps:在python2.x中还是有cmp函数的,在3.x中已经没有了,但是引入了
import operator       #首先要导入运算符模块
operator.gt(1,2)      #意思是greater than（大于）
operator.ge(1,2)      #意思是greater and equal（大于等于）
operator.eq(1,2)      #意思是equal（等于）
operator.le(1,2)      #意思是less and equal（小于等于）
operator.lt(1,2)      #意思是less than（小于）
)
map取最大最小值：
方法一：
max(dict,key=dict.get)
min(dict,key=dict.get)
方法二：
min(d.items(), key=lambda x: x[1])
min(d.items(), key=lambda x: x[1][0]
min(d.items(), key=lambda x: x[1])[1]
```
**实践：取零售价、促销价、会员价中价格最低的标志位。**
```
def get_min_price(raw_retail_price,raw_promo_price,raw_vip_price):
    price_type = {}
    if raw_retail_price > 0:
        price_type["0"] = raw_retail_price
    if raw_promo_price > 0:
        price_type["1"] = raw_promo_price
    if raw_vip_price > 0:
        price_type["2"] = raw_vip_price
    discount_type = "0"
    if price_type:
        discount_type = min(price_type, key=price_type.get)
    return str(discount_type)
```
