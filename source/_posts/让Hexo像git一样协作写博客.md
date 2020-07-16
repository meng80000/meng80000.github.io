---
title: 迁移hexo到新电脑
date: 2020-07-16 15:22:09
tags: 博客
categories: hexo
---

![1](1.png)

完成Hexo本地运行后，会在本地文件里生成一个public文件夹。public文件夹内是根据.md生成的html文件，也就博客的静态文件。
通常情况下，我们执行：


`$ hexo d`

<!-- more -->
就是把public文件夹下的文件同步到github，然后就能通过 https://username.github.io/ 访问博客了。所以，我们的思路其实就是把静态文件和Hexo环境，分别存在username.github.io的master和hexo分支上。

由于`hexo d`上传部署到github的其实是hexo编译后的文件，是用来生成网页的，不包含源文件。

![3](2.png)

也就是上传的是在本地目录里自动生成的`.deploy_git`里面。

其他文件 ，包括我们写在source 里面的，和配置文件，主题文件，都没有上传到github

![3](3.png)

所以可以利用git的分支管理，将源文件上传到github的另一个分支即可。

###上传分支

首先，先在github上新建一个hexo分支，如图：

![4](4.png)

然后在这个仓库的settings中，选择默认分支为hexo分支（这样每次同步的时候就不用指定分支，比较方便）。

![5](5.png)

然后在本地的任意目录下，打开git bash，

`git clone git@github.com:ZJUFangzh/ZJUFangzh.github.io.git`

将其克隆到本地，因为默认分支已经设成了hexo，所以clone时只clone了hexo。

接下来在克隆到本地的 `xxxx.github.io` 中，把除了.git 文件夹外的所有文件都删掉

把之前我们写的博客源文件全部复制过来，除了`.deploy_git`。这里应该说一句，复制过来的源文件应该有一个`.gitignore`，用来忽略一些不需要的文件，如果没有的话，自己新建一个，在里面写上如下，表示这些类型文件不需要git：

```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

**注意，如果你之前克隆过theme中的主题文件，那么应该把主题文件中的.git文件夹删掉，因为git不能嵌套上传，最好是显示隐藏文件，检查一下有没有，否则上传的时候会出错，导致你的主题文件无法上传，这样你的配置在别的电脑上就用不了了。**

而后
```
git add .
git commit –m "add branch"
git push 
```
这样就上传完了，可以去你的github上看一看hexo分支有没有上传上去，其中`node_modules`、`public`、`db.json`已经被忽略掉了，没有关系，不需要上传的，因为在别的电脑上需要重新输入命令安装 。

![5](5.png)

这样就上传完了。

### 更换电脑操作

一样的，跟之前的环境搭建一样，

- 安装git

    `sudo apt-get install git`

- 设置git全局邮箱和用户名
    ```
    git config --global user.name "yourgithubname"
    git config --global user.email "yourgithubemail"
    ```
- 设置ssh key
    ```
    ssh-keygen -t rsa -C "youremail"
    #生成后填到github和coding上（有coding平台的话）
    #验证是否成功
    ssh -T git@github.com
    ssh -T git@git.coding.net #(有coding平台的话)
    ```
- 安装nodejs
    ```
    sudo apt-get install nodejs
    sudo apt-get install npm
    ```
- 安装hexo

    `sudo npm install hexo-cli -g`
    
但是已经不需要初始化了，

直接在任意文件夹下，

`git clone git@………………`

然后进入克隆到的文件夹：

    cd xxx.github.io
    npm install
    npm install hexo-deployer-git --save

生成，部署：
 
    hexo g
    hexo d

    
然后就可以开始写你的新博客了

    hexo new newpage

Tips:

1. 不要忘了，每次写完最好都把源文件上传一下
    ```
    git add .
    git commit –m "描述xxxx"
    git push 
    ```

1. 如果是在已经编辑过的电脑上，已经有clone文件夹了，那么，每次只要和远端同步一下就行了
    ```
    git pull
   ```
- master分支

    修改_config.yml，再提交便可。
    ```
    deploy:
      type: git
      repo: https://github.com/username/username.github.io.git
      branch: master
    ```
### hexo分支

简单的说就是把Hexo环境push到hexo分支：
1. 创建仓库，http://xxx.github.io；
2. 创建两个分支：master 与 hexo；
3. 设置hexo为默认分支（因为我们只需要手动管理这个分支上的Hexo网站文件）；
4. 使用git clone git@github.com:xxx/xxx.github.io.git拷贝仓库；
5. 在本地文件夹下通过Git bash依次执行npm install hexo、hexo init、npm install 和 npm install hexo-deployer-git（此时当前分支应显示为hexo）
6. 修改_config.yml中的deploy参数，分支应为master；
7. 依次执行git add .、git commit -m "..."、git push origin hexo提交网站相关的文件；
8. 执行hexo g -d生成网站并部署到GitHub上。

***以上前四步的目的是，在github上建立一个新的repo，并且把目录同步到本地。
执行第五步的时候，Hexo会生成一个新的.git，并且覆盖了上文提到的.git...这会导致没法push到hexo分支...正确的做法是，在hexo init前复制.git在完成hexo init后再黏贴回来覆盖新生成的.git***

***注意：需要使用git push把源文件推到分支上***
```
$ git add .
$ git commit -m "xxxx"
$ git push origin hexo
```

