---
title: sphinx支持markdown
date: 2020-07-15 17:50:20
tags:
        - sphinx
        - python
categories: python
---

#### 配置

1. 要为Markdown支持配置Sphinx项目，请按照以下步骤进行：
    
    `pip install --upgrade recommonmark`
    注
    这里解释的配置需要重新注册版本0.5.0或更高版本。
<!-- more -->

2. config文件中：

    ```python
    extensions = ['recommonmark']
    ```
3. 如果您想使用扩展名以外的Markdown文件.md，调整source_suffix变量。下面的示例将Sphinx配置为解析所有扩展名为.md和.txt作为标记：

    ```python
    source_suffix = {
        '.rst': 'restructuredtext',
        '.txt': 'markdown',
        '.md': 'markdown',
    }
    ```
*参考：https://www.sphinx-doc.org/en/master/usage/markdown.html*
