---
title: hexo categories和tags页面不显示解决办法
date: 2020-07-17 09:48:22
tags:
    - 博客
categories: hexo
---
**Hexo 默认是没有 categories 和 tags 的需要**

<!-- more -->
###一、 需要新建tags和categories页面
```
hexo new page "tags" 
```
```
hexo new page "categories"
```

###二、编辑 /tags/index.md /categories/index.md
```
---
title: tags
date: 2020-01-10 16:14:33
type: "tags"
layout: "tags"   #增加tags的布局
---
```
```
---
title: categories
date: 2020-01-10 16:15:43
type: "categories"
layout: "categories"    #增加categories的布局
---
```

###三、给自己的帖子新加配置,增加tags和categories
例如我自己的一篇博客
```
---
title: hexo categories和tags页面不显示解决办法
date: 2020-01-12 17:22:40
tags: 博客
categories: Hexo
---
```


