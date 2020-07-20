---
title: python collections 模块中的 deque
date: 2020-07-20 17:49:10
tags: 模块
categories: python
---

## collections.deque介绍

collections 是 python 内建的一个集合模块，里面封装了许多集合类，其中队列相关的集合只有一个：deque。 
deque 是双边队列（double-ended queue），具有队列和栈的性质，在 list 的基础上增加了移动、旋转和增删等。

<!-- more -->

##### 常用方法

```
d = collections.deque([])
d.append('a') # 在最右边添加一个元素，此时 d=deque('a')
d.appendleft('b') # 在最左边添加一个元素，此时 d=deque(['b', 'a'])
d.extend(['c','d']) # 在最右边添加所有元素，此时 d=deque(['b', 'a', 'c', 'd'])
d.extendleft(['e','f']) # 在最左边添加所有元素，此时 d=deque(['f', 'e', 'b', 'a', 'c', 'd'])
d.pop() # 将最右边的元素取出，返回 'd'，此时 d=deque(['f', 'e', 'b', 'a', 'c'])
d.popleft() # 将最左边的元素取出，返回 'f'，此时 d=deque(['e', 'b', 'a', 'c'])
d.rotate(-2) # 向左旋转两个位置（正数则向右旋转），此时 d=deque(['a', 'c', 'e', 'b'])
d.count('a') # 队列中'a'的个数，返回 1
d.remove('c') # 从队列中将'c'删除，此时 d=deque(['a', 'e', 'b'])
d.reverse() # 将队列倒序，此时 d=deque(['b', 'e', 'a'])
```

##### 应用

1. 可以使用 deque 的旋转来制作跑马灯：
```
import collections
import sys
import time

def marquee(length=50, speed=1, direction=1):
    """
    生成一个简单的跑马灯
    length:总长
    speed：每0.1秒的移动速度
    direction:0为向左，1为向右
    """
    if direction == 1:
        array = '>'
    else:
        array = '<'
    que = collections.deque([array])
    que.extend(['-'] * (length - 1)) # 形如'>------'
    while True:
        print('%s' % ''.join(que))
        if direction == 1:
            que.rotate(1 * speed)
        else:
            que.rotate(-1 * speed)
        sys.stdout.flush()
        time.sleep(0.1)

if '__main__' == __name__:
    marquee()
```

2. 可以使用deque的旋转来解决约瑟夫问题：
```
""" 约瑟夫算法
据说著名犹太历史学家 Josephus 有过以下的故事：
在罗马人占领桥塔帕特后，39个犹太人与 Josephus 及他的朋友躲到一个洞中，
39个犹太人决定宁愿死也不要被敌人抓到，于是决定了一个自杀方式，41个人排成一个圆圈，
由第1个人开始报数，每报数到第3人该人就必须自杀，然后再由下一个重新报数，
直到所有人都自杀身亡为止。然而 Josephus 和他的朋友并不想自杀，
问他俩安排的哪两个位置可以逃过这场死亡游戏？
"""
import collections
def ysf(a, b):
    d = collections.deque(range(1, a+1)) # 将每个人依次编号，放入到队列中
    while d:
        d.rotate(-b) # 队列向左旋转b步
        print(d.pop()) # 将最右边的删除，即自杀的人

if __name__ == '__main__':
    ysf(41,3) # 输出的是自杀的顺序。最后两个是16和31，说明这两个位置可以保证他俩的安全。
```

3. deque是现成安全的。
```
import collections
import threading
import time

candle = collections.deque(xrange(10))
print candle

def poper(direction, next_item):
    while True:
        try:
            next = next_item()
        except IndexError:
            break
        else:
            print '%s : %s' % (direction, next)
            time.sleep(0.1)
    print "done %s" % direction
    return

left = threading.Thread(target=poper, args=('left', candle.popleft))
right = threading.Thread(target=poper, args=('right', candle.pop))

left.start()
right.start()

left.join()
right.join()

结果：
deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
left : 0
right : 9
right : 8
left : 1
right : 7
left : 2
left : 3
right : 6
left : 4
right : 5
done right
done left
```

###### 原文链接：https://blog.csdn.net/happyrocking/article/details/80058623
