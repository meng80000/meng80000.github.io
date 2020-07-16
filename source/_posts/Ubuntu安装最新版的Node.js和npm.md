---
title: Ubuntu 中安装最新版的 Node.js 和 npm
date: 2020-07-15 17:55:33
tags:
        - 环境
        - ubuntu
categories: ubuntu
---

#### 配置
### 1. 卸载已安装的Node和npm ！！！
    ```
    sudo apt remove npm  //卸载npm
    sudo apt remove node //卸载node

    cd /usr/local/bin   //进入该目录中，若有node或者npm文件，将他删除删除
    ```
### 2. 下载安装node.js
去[node.js官网](https://nodejs.org/en/download/current/)下载最新版或者最稳定版的node.js Linux安装包
![imagl](1.png)

<!-- more -->
1. 下载解压

    `tar -xJf node-v8.5.0-linux-x64.tar.xz`  //解压命令

2. 然后把解压的文件移动到opt文件夹下

    `mv node-v8.5.0-linux-x64 /opt/`   //移动到opt目录下

3. 建立链接到 /usr/local/bin/ 目录

    `sudo ln -s /opt/node-v8.5.0-linux-x64/bin/node /usr/local/bin/node`    

4. 然后跟npm建立执行链接

    `sudo ln -s /opt/node-v8.5.0-linux-x64/bin/npm /usr/local/bin/npm`

此时，我们的环境搭建已经完毕


![imagl](2.png)


### 最后再补充一下设置淘宝镜像

    sudo npm config set registry https://registry.npm.taobao.org   //设置淘宝镜像
    source ~/.bashrc       //使修改立即生效


*转载：https://blog.csdn.net/mrwangweijin/article/details/78106955*
