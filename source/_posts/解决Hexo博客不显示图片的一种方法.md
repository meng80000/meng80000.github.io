---
title: 解决Hexo博客不显示图片的一种方法
date: 2020-07-16 16:49:22
tags: 博客
categories: hexo
---

## 修改博客配置

修改博客根目录中_config.yml文件的配置项 `post_asset_folder` 为`true`：

<!-- more -->

`post_asset_folder: true`

完成此设置后，当你通过hexo new 文件名新建博客后，会产生一个和文件同名的文件夹。

![image](1.png)

## 下载插件

在博客根目录中下使用npm安装插件：

```
npm install https://github.com/CodeFalling/hexo-asset-image --save
```

## 示例

当文章需要添加图片时，将需要添加的图片放入同名的文件夹中，直接写图片名称就行（否则通过相对路径索引到该图片）。

例如我在上方修改博客配置中展示的那张图片的md源码为：
```
![示例](1.png)
```
使用命令hexo s开启服务，若无报错在本地可以看到图片在博客中正确显示。


