---
title: "Hugo 添加评论功能"
date: 2020-06-17T21:48:44+08:00
tags: ["hugo"]
keywords: 
- hugo
- blog
- 博客
- 建站
- 评论
description: "Hugo 添加评论功能"
categories: ["blog", "hugo"]
draft: false
---

# 不能评论!

[Hugo](https://gohugo.io/) 是一个强大的静态网页生成器，使用 go 开发，速度贼快。不过 Hugo 还比较年轻，没有像 [NexT](https://theme-next.iissnan.com/) 那样功能强大，成熟稳定的主题。所以很多东西需要自己动手撸。

比如评论系统。

> 评论系统排名

> valine > gitalk > gitment > livere > 畅言



## 关于Valine

![valine.png](https://github.com/affectalways/affectalways.github.io/blob/master/images/hugo/hugo_comment/valine.png?raw=true)

- 官方网址： <<https://valine.js.org/>>

- 其特性不一一赘述，官方网址有介绍

  

## 添加评论区

目前博客使用的主题是meme

不同的主题可能有所区别，但原理大都类似

> ### 原理？
>
> Hugo 会将 **Markdown 文档** 按照 **主题 (包括 HTML 模板、CSS、JavaScript 等)** 编译成静态网页
>
> 那么我们只需要将 `Valine`作为一个 `<div>` 插入到 HTML 模板中，然后在 `config.toml` 中添加相关配置，就可以添加`评论区`了



## Leancloud相关配置

评论系统依赖于`leancloud`，所以需要先在`leancloud`中进行相关的准备工作。

1. [登录](https://leancloud.cn/dashboard/login.html#/signin) 或 [注册](https://leancloud.cn/dashboard/login.html#/signup) LeanCloud

2. 登录成功后，进入后台点击左上角的创建应用：

   ![leancloud.png](https://github.com/affectalways/affectalways.github.io/blob/master/images/hugo/hugo_comment/leancloud.png?raw=true)

3. 创建好应用，进入应用，左边栏找到 **设置** ，然后点击 **应用Key**，此时记录出现的 **App ID** 和 **App Key**，后面配置文件中会用到：

   ![lc_key.png](https://github.com/affectalways/affectalways.github.io/blob/master/images/hugo/hugo_comment/leancloud_key.png?raw=true)

   

4. 因为评论和文章阅读数统计依赖于存储，所以还需要建立两个新的存储 `Class`，左边栏找到并点击 **存储**，点击 **创建Class**:

   ![lc_class.png](https://github.com/affectalways/affectalways.github.io/blob/master/images/hugo/hugo_comment/leancloud_class.png?raw=true)

   

5. 创建两个存储Class，分别命名为: `Counter` 和 `Comment`;

6. 还需要为应用添加安全域名，左边栏点击 **设置**，找到 **安全中心**，点击后会看到 **安全域名** 设置框，输入博客使用的域名，点击保存即可：

![lc_safe.png](https://github.com/affectalways/affectalways.github.io/blob/master/images/hugo/hugo_comment/leancloud_safe.png?raw=true)





### config.toml开启comment

1. 将`enableComments = false`设置为`enableComments = true`
2. 将`enableValine = false`设置为`enableValine = true`

> 添加 **Valine** 参数项：

```
## Valine
    enableValine = true
    valineAppId = "************"
    valineAppKey = "*****************"
    valinePlaceholder = "Just go go"
    valinePath = ""
    valineAvatar = "mm"
    valineMeta = ["nick", "mail", "link"]
    valinePageSize = 10
    valineLang = "zh-cn"
    valineVisitor = false
    valineHighlight = true
    valineAvatarForce = false
    valineRecordIP = false
    valineServerURLs = ""
    valineEmojiCDN = ""
    valineEmojiMaps = {}
    valineEnableQQ = false
    valineRequiredFields = []
```

上面几项内容的含义，这里简单一说，具体还是要看 [Valine官网中配置相关的内容](https://valine.js.org/configuration.html)：

| 参数       | 用途                                                         |
| ---------- | ------------------------------------------------------------ |
| enable     | 这是用于主题中配置的，不是官方Valine的参数，**true**时控制开启此评论系统 |
| appId      | 这是在 [leancloud](https://leancloud.cn/) 后台应用中获取的，也就是上面提到的 **App ID** |
| appKey     | 这是在 [leancloud](https://leancloud.cn/) 后台应用中获取的，也就是上面提到的 **App Key** |
| notify     | 用于控制是否开启邮件通知功能，具体参考[邮件提醒配置](https://github.com/xCss/Valine/wiki/Valine-%E8%AF%84%E8%AE%BA%E7%B3%BB%E7%BB%9F%E4%B8%AD%E7%9A%84%E9%82%AE%E4%BB%B6%E6%8F%90%E9%86%92%E8%AE%BE%E7%BD%AE) |
| verify     | 用于控制是否开启评论验证码功能                               |
| avatar     | 用于配置评论项中用户头像样式，有多种选择：mm, identicon, monsterid, wavatar, retro, hide。详细参考：[头像配置](https://valine.js.org/avatar.html) |
| placehoder | 评论框的提示符                                               |
| visitor    | 控制是否开启文章阅读数的统计功能i, 详情阅读[文章阅读数统计](https://valine.js.org/visitor.html) |





### 修改主题文件

主要是修改主题中评论相关的布局文件 `themes\meme\layouts\partials\components\comments.html`，按照 [Valine快速开始](https://valine.js.org/quickstart.html) 添加 **Valine** 相关代码，找到以下位置

```
{{ if .Site.Params.enableValine }}
{{- end }}
```

添加的 **Valine** 评论的代码如下：

```
{{ if .Site.Params.enableValine }}
            <div id="vcomments"></div>
			<script src="//cdn1.lncld.net/static/js/3.0.4/av-min.js"></script>
			  <script src='//unpkg.com/valine/dist/Valine.min.js'></script>
			  <script type="text/javascript">
				new Valine({
					el: '#vcomments' ,
					appId: '{{ .Site.Params.valineAppId }}',
					appKey: '{{ .Site.Params.valineAppKey }}',
					notify: '{{ .Site.Params.valineNotify }}', 
					verify: '{{ .Site.Params.valineVerify }}', 
					avatar:'{{ .Site.Params.valineAvatar }}', 
					placeholder: '{{ .Site.Params.valinePlaceholder }}',
					visitor: '{{ .Site.Params.valineVisitor }}'
				});
			  </script>
        {{ end }}
```

可以看到上述代码中引用了配置文件中的相关参数，这样以后修改配置就不用修改代码了，只需要改配置文件 `config.toml`。