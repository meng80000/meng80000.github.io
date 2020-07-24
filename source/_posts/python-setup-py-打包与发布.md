---
title: python setup.py 打包与发布
date: 2020-07-24 17:01:07
tags:
    - 模块
    - sdist
categories: python
---
### setup函数包含的参数解释：

<!-- more -->

```
--name 包名称------------生成的egg名称
--version (-V) 包版本----生成egg包的版本号
--author 程序的作者------包的制作者名字
--author_email 程序的作者的邮箱地址
--maintainer 维护者
--maintainer_email 维护者的邮箱地址
--url 程序的官网地址
--license 程序的授权信息
--description 程序的简单描述-------程序的概要介绍
--long_description 程序的详细描述---程序的详细描述
--platforms 程序适用的软件平台列表
--classifiers 程序的所属分类列表
--keywords 程序的关键字列表
--packages 需要处理的包目录（包含__init__.py的文件夹）-------和setup.py同一目录下搜索各个含有 init.py的包
--py_modules 需要打包的python文件列表
--download_url 程序的下载地址
--cmdclass
--data_files 打包时需要打包的数据文件，如图片，配置文件等
--scripts 安装时需要执行的脚步列表
--package_dir 告诉setuptools哪些目录下的文件被映射到哪个源码包。一个例子：package_dir = {'': 'lib'}，表示“root package”中的模块都在lib 目录中。
--requires 定义依赖哪些模块
--provides定义可以为哪些模块提供依赖
--find_packages() 对于简单工程来说，手动增加packages参数很容易，刚刚我们用到了这个函数，它默认在和setup.py同一目录下搜索各个含有 init.py的包。
                  其实我们可以将包统一放在一个src目录中，另外，这个包内可能还有aaa.txt文件和data数据文件夹。另外，也可以排除一些特定的包
find_packages(exclude=[".tests", ".tests.", "tests.", "tests"])
--install_requires = ["requests"] 需要安装的依赖包
--entry_points 动态发现服务和插件
```

### 在同级目录下建立 setup.py

```python
# encoding=utf-8
from setuptools import setup, find_packages, Extension

setup(
    name='sdk_tool',    # 安装包名
    version='1.0.3',    # 打包安装软件的版本号
    description='Harvest DataLab data api service',
    author='xxx',    # 提供与包相关的其他维护者的名字
    author_email='xxx@xxx.cn',    # 他维护者的邮箱
    long_description="flask demo",    # 程序的详细描述
    url='http://datalab.jsfund.cn/',    # 包相关网站主页的的访问地址
    download_url="",  # 下载安装包(zip , exe)的url
    keywords='Financial Data',
    zip_safe=False,    # 提供与包相关的其他维护者的名字
    python_requires=">=3.6",

    # 对于C,C++,Java 等第三方扩展模块一起打包时，需要指定扩展名、扩展源码、以及任何编译/链接 要求（包括目录、链接库等）
    ext_modules = [Extension('data',['data.c'])],

    # 需要处理的包目录（必须包含__init__.py文件）
    # 需要打包的package,使用find_packages 来动态获取package，exclude参数的存在，使打包的时候，排除掉这些文件
    packages = find_packages(exclude=["test", "test.*"]),

    ##########################################
    # 首先告诉程序去哪个目录中找包，因此有了packages参数，
    # 其次，告诉程序我包的起始路径是怎么样的，因此有了package_dir参数
    # 最后，找到包以后，我应该把哪些文件打到包里面，因此有了package_data参数

    # 包含所有src目录下的包 ---------项目中的所有源码和测试用例文件目录一般都存放在统一的src目录下方便管理，默认也是创建src目录	
    # packages = find_packages('src'),
    # package_dir = {'':'src'},
    package_data = {
    # 包含所有.txt文件
    "":[".txt"],
    # 包含data目录下所有的.dat文件
    "test":["data/.dat"],
    },
    ##########################################
    # 需要安装的依赖
    install_requires=[
        "requests>=2.21.0",
        "PyMySQL>=0.9.3",
        "numpy==1.18.5",
        "pandas==0.25.0",
    ],
    # 程序的所属分类列表
    classifiers=[
        'Development Status :: 4 - Beta',
        'Programming Language :: Python :: 3.6',
        'License :: OSI Approved :: BSD License'
    ],
    include_package_data=True,
)
```

如何扩展和嵌入 Python 解释器 ： https://docs.python.org/zh-cn/3/extending/index.html

### 编写安装配置文件 setup.cfg

```
[sdist]
dist-dir = source
dist-dir : 指定发布源码的路径，默认 dist
```
如何编写setup.cfg:

```
Common commands: (see '--help-commands' for more)

  setup.py build      will build the package underneath 'build/'
  setup.py install    will install the package

Global options:
  --verbose (-v)  run verbosely (default)
  --quiet (-q)    run quietly (turns verbosity off)
  --dry-run (-n)  don't actually do anything
  --help (-h)     show detailed help message
  --no-user-cfg   ignore pydistutils.cfg in your home directory

Options for 'sdist' command:
  --template (-t)        name of manifest template file [default: MANIFEST.in]
  --manifest (-m)        name of manifest file [default: MANIFEST]
  --use-defaults         include the default file set in the manifest
                         [default; disable with --no-defaults]
  --no-defaults          don't include the default file set
  --prune                specifically exclude files/directories that should
                         not be distributed (build tree, RCS/CVS dirs, etc.)
                         [default; disable with --no-prune]
  --no-prune             don't automatically exclude anything
  --manifest-only (-o)   just regenerate the manifest and then stop (implies
                         --force-manifest)
  --force-manifest (-f)  forcibly regenerate the manifest and carry on as
                         usual. Deprecated: now the manifest is always
                         regenerated.
  --formats              formats for source distribution (comma-separated
                         list)
  --keep-temp (-k)       keep the distribution tree around after creating
                         archive file(s)
  --dist-dir (-d)        directory to put the source distribution archive(s)
                         in [default: dist]
  --medata-check         Ensure that all required elements of meta-data are
                         supplied. Warn if any missing. [default]
  --owner (-u)           Owner name used when creating a tar file [default:
                         current user]
  --group (-g)           Group name used when creating a tar file [default:
                         current group]
  --help-formats         list available distribution formats

usage: setup.py [global_opts] cmd1 [cmd1_opts] [cmd2 [cmd2_opts] ...]
   or: setup.py --help [cmd1 cmd2 ...]
   or: setup.py --help-commands
   or: setup.py cmd --help
```
 
###  发布软件 (压缩包)
```
linux : python setup.py  sdist
windows : setup.py sdist
```
指定发布格式,同时生成两个压缩包: `python setup.py  sdist --formats=gztar,zip`
windows exe : `python setup.py bdist_wininst`

    --formats:

    zip -> .zip
    gztar -> .tar.gz
    bztar -> .tar.bz2
    ztar -> .tar.Z
    tar -> .tar
 
### 安装源码包，然后你就可以导入了

解压后cd 到解压目录
安装命令 `python setup.py install`
或者安装时保存安装日志： `python setup.py install --record log`

### 安装后删除

1. 安装时记录日志 `python setup.py install --record log`
2. windows : `for /F  %i in (log) do del %i`
     linux : `cat log ｜ xagrs rm －rf`
     
其中： log文件内容是安装目录：

```
E:\...\Lib\site-packages\package.py
E:\...\Lib\site-packages\__pycache__\package.cpython-37.pyc
E:\...\Lib\site-packages\package-1.0-py3.7.egg-info
```

