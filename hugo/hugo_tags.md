---
title: "Hugo 添加tags等分类"
date: 2020-06-17T20:19:37+08:00
tags: ["hugo"]
keywords: 
- hugo
- blog
- 博客
- 建站
description: "Hugo 添加tags等分类"
categories: ["blog", "hugo"]
draft: false
---

# Hugo 

Hugo是支持用户自定义分类的，这个称为taxonomy，可以来对网页内容进行逻辑划分，详情可以在[这里](https://gohugo.io/content-management/taxonomies/)查看。

分类taxonomy有3个概念：

- **Taxonomy 分类**: 可以用来对内容进行分类的类别
- **Term 术语**: 分类的一个键
- **Value 值**: 分配给这个Term的具体内容

例如我需要增加3个分类，分别是：

- tag：文章标签
- topic：文章主题/文章系列
- category：文章分类

以tag为例，则对应Taxonomy是tag，Term是具体标签内容比如hugo，Value是打上这个标签的对应网页。



### 配置分类

需要在 `config.toml` 中增加分类。还是这个例子，则需要增加如下内容：

```
[taxonomies]
tag = "tags"
category = "categories"
```

而将每个post的头部也相应增加对应的分类，例如这篇的头部就相应为：

```
title: "Hugo添加tags等分类"
date: 2020-06-17T20:19:37+08:00
tags: ["hugo"]
draft: true
```

当然实际上，Hugo默认会产生 `tags` 和 `categories` 的分类，如果只需要这两个，可以不用在 `config.toml` 中声明就在post头部使用。



### 分类集合查看

使用分类taxonomy之后，Hugo会使用分类的模板 (taxonomy templates) 来自动生成一个显示所有分类的term术语的网页以及一个显示该术语的所有value内容列表网页。

还是以tag为例：

`example.com/tags/` 会列出tags中的所有术语；

`example.com/tags/docker` 会列出tags标为docker的所有网页列表。





> 额外知识点：

### keywords、description

meta标签的一个很重要的功能就是设置关键字，来帮助你的主页被各大搜索引擎登录，提高网站的访问量。在这个功能中，最重要的就是对Keywords和description的设置。因为按照搜索引擎的工作原理,搜索引擎首先派出机器人自动检索页面中的keywords和decription，并将其加入到自己的数据库，然后再根据关键词的密度将网站排序。因此，我们必须设置好关键字，来提高页面的搜索点击率。使用如下：

```
keywords: 
- hugo
- blog
- 博客
- 建站
description: "Hugo 添加tags等分类"
```

> keywords需要进行配置：
>
> 在`themes\meme\layouts\partials\header.html`的
>
> ```
> <header class="header"{{ if and (eq .Site.Params.headerLayout "flex") 
> ```
>
> 内部添加
>
> <meta content="{{ delimit .Keywords ", " }}" name="keywords">
> 就可以了



> description是hugo支持的，不需要配置

