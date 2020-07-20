---
title: Python的时间转换
date: 2020-07-20 18:19:24
tags: 模块
categories: python
---
###### 一、获取当前时间：

<!-- more -->

```
import datetime,time
now = time.strftime("%Y-%m-%d %H:%M:%S")
print now, type(now)
now = datetime.datetime.now()
print now, type(now)

# 2019-10-24 14:19:51 <type 'str'>
# 2019-10-24 14:19:51.615000 <type 'datetime.datetime'>
```
###### 二、时间比较：
- **`包含时分秒`**
```
stime = '2019-10-27 14:26:23'
eime = '2019-10-28 18:29:55'
now = datetime.datetime.now ()

start_time = datetime.datetime.strptime (stime, '%Y-%m-%d %H:%M:%S')
end_time = datetime.datetime.strptime (eime, '%Y-%m-%d %H:%M:%S')

print now, type(now)
print start_time, type(start_time)
print end_time, type(end_time)
print start_time<=now
print end_time>=now

# 2019-10-24 14:24:23.127000 <type 'datetime.datetime'>
# 2019-10-27 14:26:23 <type 'datetime.datetime'>
# 2019-10-28 18:29:55 <type 'datetime.datetime'>
# False
# True
```
- **`不包含时分秒`**
```
stime = '2019-10-27 14:26:23'
eime = '2019-10-28 18:29:55'
now = datetime.datetime.now ().date()

start_time = datetime.datetime.strptime (stime, '%Y-%m-%d %H:%M:%S').date()
end_time = datetime.datetime.strptime (eime, '%Y-%m-%d %H:%M:%S').date()

print now, type(now)
print start_time, type(start_time)
print end_time, type(end_time)
print start_time<=now
print end_time>=now

# 2019-10-24 <type 'datetime.date'>
# 2019-10-27 <type 'datetime.date'>
# 2019-10-28 <type 'datetime.date'>
# False
# True
```
###### 三、各种转换demo：
1. **`时间戳转datetime类型`**
```
import datetime, time

res1 = datetime.datetime.fromtimestamp(time.time())
res2 = datetime.date.fromtimestamp(time.time())
print res1, type(res1)
print res2, type(res2)
# 2019-10-24 14:54:20.837000 <type 'datetime.datetime'>
# 2019-10-24 <type 'datetime.date'>
```
2. **` datetime类型转时间戳`**
```
res1 = time.mktime(datetime.datetime.now().timetuple())
res2 = time.mktime(datetime.datetime.now().date().timetuple())
print res1, type(res1)
print res2, type(res2)
# 1571901284.0 <type 'float'>
# 1571846400.0 <type 'float'>
```
3. **`datetime类型转字符串`**
```
res1 = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
res2 = datetime.datetime.now().strftime("%Y-%m-%d")
print res1, type(res1)
print res2, type(res2)
# 2019-10-24 15:12:55 <type 'str'>
# 2019-10-24 <type 'str'>
```
4. **`字符串转datetime类型`**
```
res1 = datetime.datetime.strptime("2019-10-24 10:20:20","%Y-%m-%d %H:%M:%S")
res2 = datetime.datetime.strptime("2019-10-24","%Y-%m-%d")
print res1, type(res1)
print res2, type(res2)
# 2019-10-24 10:20:20 <type 'datetime.datetime'>
# 2019-10-24 00:00:00 <type 'datetime.datetime'>
```
5. **`字符串转时间戳`**
```
res1 = time.mktime(time.strptime("2019-10-24 14:51:00","%Y-%m-%d %H:%M:%S"))
res2 = time.mktime(time.strptime("2019-10-24","%Y-%m-%d"))
print res1, type(res1)
print res2, type(res2)
# 1571899860.0 <type 'float'>
# 1571846400.0 <type 'float'>
```
6. **`时间戳转字符串`**
```
res1 = time.strftime("%Y-%m-%d %H:%M:%S",time.localtime(time.time()))
res2 = time.strftime("%Y-%m-%d",time.localtime(time.time()))
print res1, type(res1)
print res2, type(res2)
# 2019-10-24 14:54:20 <type 'str'>
# 2019-10-24 <type 'str'>
```
7. **`字符串转元祖格式`**
```
res = time.strptime('2019-11-11 11:11:11',"%Y-%m-%d %H:%M:%S")
print res, type(res)
# time.struct_time(tm_year=2019, tm_mon=11, tm_mday=11, tm_hour=11, tm_min=11, tm_sec=11, tm_wday=0, tm_yday=315, tm_isdst=-1) <type 'time.struct_time'>
```
8. **`元祖格式转字符串`**
```
res = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())
print res, type(res)
# 2019-10-24 15:31:55 <type 'str'>
```
9. **`元祖转时间戳`**
```
res = time.mktime(time.localtime())
print res, type(res)
# 1571902315.0 <type 'float'>
```
10. **`时间戳转元祖`**
```
res = time.gmtime(time.time())
print res, type(res)
# time.struct_time(tm_year=2019, tm_mon=10, tm_mday=24, tm_hour=7, tm_min=31, tm_sec=55, tm_wday=3, tm_yday=297, tm_isdst=0) <type 'time.struct_time'>
```
###### 四、回到过去和未来：
```
now = datetime.datetime.now()

print now
print now + datetime.timedelta(days=2)
print now + datetime.timedelta(days=-2)
print now + datetime.timedelta(hours=3)
print now + datetime.timedelta(hours=-3)
print now.replace(year=2018,month=11,day=11,hour=11,minute=11,second=11)

# 2019-10-26 15:23:21.328000
# 2019-10-22 15:23:21.328000
# 2019-10-24 18:23:21.328000
# 2019-10-24 12:23:21.328000
# 2018-11-11 11:11:11.328000
```
