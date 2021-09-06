---
title: "Hugo 让 GitHub Pages被索引到"
date: 2020-06-17T21:00:29+08:00
tags: ["hugo"]
keywords: 
- hugo
- blog
- 博客
- 建站
- seo
- 被索引
description: "Hugo 让 GitHub Pages被索引到"
categories: ["blog", "hugo"]
draft: false
---

## 居然都找不到！

其实Blog我主要是用来做一些记录，突然看到有博客提到GitHub Pages屏蔽了百度的爬虫，所以百度是搜索不到GitHub Pages上的网页的。

什么？所以百度搜索搜不到我的个人博客？不死心，尝试一下，找得到github、gist主页，但是就是找不到github.io后缀的网页。除了百度之外，我还尝试了Bing、Google，都没有。



## 如何让搜索引擎索引到呢？

发现流行的搜索引擎居然都找不到博客，那就需要赶紧找解决方法。Google和Bing还不清楚是怎么回事，但网上流传的email回复内容都点明了Github Pages禁止了百度爬虫的爬去，似乎原因是百度爬虫爬得太过于频繁，会严重影响服务器性能。针对百度爬虫的问题，大家找了很多方法。自建服务器托管博客、将博客放在Gitlab上或者CDN方法都不在我的选择范围内，因为暂时我还没有购买服务器或者域名的打算，所以决定放弃百度……但不管怎么样Bing和Google还是要设置好的！



### Google

#### 添加资源

在google搜索页面输入“site:affectalways.github.io”就可以看到这个网页是否被google索引到，如果没被索引到，在搜索结果页面就会直接提示你使用[Google Search Console](<https://search.google.com/search-console?utm_source=about-page&resource_id=https://affectalways.github.io/>)。登录后，如果是首次使用在Search Console中以下界面中选择“网页”类型资源，并将博客完整url填入其中，我填入“https://affectalways.github.io”。注意http或者https，www等最好能完全正确。

![hugo_seo_1.png](https://github.com/affectalways/affectalways.github.io/blob/master/images/hugo/hugo_seo/hugo_seo_1.png?raw=true)



如果已经添加过资源，则需要点击左上角的按钮，然后和上面一样地添加资源即可。

![hugo_seo_2.png](https://github.com/affectalways/affectalways.github.io/blob/master/images/hugo/hugo_seo/hugo_seo_2.png?raw=true)



资源添加后，需要验证你对该网站有所有权。Google提供了几种方法，我选择了三种方式：

##### HTML验证文件上传

只需要根据要求，下载HTML验证文件，把文件放在站点根目录的static目录下（以本网站为例：affectalways/static）



##### HTML标记

`config.toml`中的`googleSiteVerification = ""` 设置为 `googleSiteVerification = 不为空`

找到`themes/meme/layouts/partials/head.html`中的

```
{{- with .Site.Params.googleSiteVerification }}

        <meta name="google-site-verification" content="" />

{{- end }}
```

把`content`内容改为给定的内容，然后执行`hugo`命令，就可以验证了



##### Google Analytics

先到[Google Analytics](<https://marketingplatform.google.com/about/analytics/>)创建一个账号，并登录。

新建一个资源，填完后获得`tracking code`。

更新`config.toml`文件，把`enableGoogleAnalytics`设为`true`，`trackingCodeType`设为`gtag`（两个选择gtag和analytics，因为affectalways.github.io使用的是`Google Analytics给定的gtag.js`，所以设置为gtag），`trackingID`设为获取到的`tracking code`。然后执行`hugo`命令，就可以验证了。





#### 站点地图

在左侧点击“站点地图”，并在右侧点添加/测试站点地图，并添加url，我的是`https://affectalways.github.io/sitemap.xml`





### Bing

相似地，在[Bing网站管理](https://www.bing.com/webmaster/home)登陆、添加网站url。

然后在左侧点击“配置我的网站>Sitemaps”，并在右侧加上sitemap的url，点击提交。





### 百度不死心的尝试

> 不死心的失败了