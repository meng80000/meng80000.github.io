---
title: python自动发微信监控报警
date: 2020-07-20 17:40:14
tags: 邮件

categories: python
---

我们每个人每天都是在用微信，在程序开发过程中，我们会需要监控我们的程序，发短信监控收费，发邮件懒得看，发微信是最好的方式，而且是免费的。发现个非常好用的python库：wxpy。wxpy基于itchat，使用了 Web 微信的通讯协议，实现了微信登录、收发消息、搜索好友、数据统计等功能。

<!-- more -->
官方文档：[chats.html](https://wxpy.readthedocs.io/zh/latest/chats.html)

安装wxpy包：```pip install wxpy```

###### 一开始扫码登录，程序会保存一个.pkl文件，这个文件是程序自动保存的，下次就不需要扫码了。
```
# -*- encoding=utf-8 -*-
""" 微信报警功能"""
 
 
from wxpy import *
 
# 发给多个好友
def wxSendMsgToFriends(name_list,content):
 """
 :param name_list: 名字列表
 :param content:内容
 :return:
 """
 # 缓存实现自动登录
 bot = Bot(cache_path=True)
 try:
 for i in range(0,len(name_list)):
  my_friend = bot.friends().search(name_list[i])[0]
  my_friend.send(content)
 
 except Exception as e:
 print("{0}".format(str(e)))
 
 
# 发给机器人自己，在文件传输助手收到消息
 
def wxSendMsgToSelf(content):
 """
 :param content: 内容
 :return:
 """
 # 缓存实现自动登录
 bot = Bot(cache_path=True)
 
 # 向文件传输助手发送消息
 bot.file_helper.send(content)
 

if __name__ == '__main__':
 
 # 名字列表
 name_list=['张三','李四']
 # 发送内容
 content="微信报警功能测试"
 wxSendMsgToFriends(name_list,content)
```
###### wxpy 不仅可以发文本内容，也可以发图片，文件，视频等。感觉很方便，感兴趣的朋友可以去尝试。
![1.png](1.png)
###### 本文链接：[https://blog.csdn.net/u013421629/article/details/85626600](https://blog.csdn.net/u013421629/article/details/85626600)

