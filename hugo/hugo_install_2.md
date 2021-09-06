---
title: "Hugo blog（2）"
date: 2020-06-16T23:30:53+08:00
tags: ["hugo"]
keywords: 
- hugo
- blog
- 博客
- 建站
- 安装
description: "Hugo 建站（2）Blog"
categories: ["blog", "hugo"]
draft: false
---

# Introduction

正如上一篇说的，我选择了[Hugo](https://gohugo.io/)作为静态网页生成器。为何要放弃Hexo，换成Hugo呢？

主要是出于以下考虑：

> 1.Hugo是一个用go写的静态网页生成器，它被提及最多的优点就是它生成网站的速度快。
>
> 2.同样得益于go，Hugo的安装配置看上去也并不麻烦；Hugo可以很轻松的被编译成二进制文件安装、运行。不必像Hexo一样依赖Node.js，也不必安装一堆依赖

1. Hugo的安装配置看上去也并不麻烦，同样得益于go，



# 安装

直接在[这里](https://github.com/gohugoio/hugo/releases)选择合适的版本。

下载好后解压，将解压出来的可执行文件 (格式为 .exe)，放到自己喜欢的目录下。直接可以使用，不需要安装。

但要记住一定要将你选择的文件夹路径加入到环境变量 `PATH` 中。

> PS：由于theme选择的是meme，需要下载extended版本的hugo；



# 初始化

下面，初始化博客路径。首先需要选择一个路径来存放我们的博客，在你选好的路径下执行：

```
hugo new site myBlog
```

这条命令会创建一个名为**myBlog**（可以使用任意名字）的文件夹来存放你的博客。执行 **cd myBlog** 命令进入文件夹。

此时目录结构应该是这样的

```
.
└── myBlog
    ├── config.toml / config.yaml / config.json
    ├── content
    │   └── ...
    ├── layouts
    │   └── ...
    ├── themes
    │   └── ...
    ├── static
    │   └── ...
    ├── archetypes
    │   └── ...
    ├── data
    │   └── ...
    └── ...
```

其中：

1. `config.toml` 是网站的配置文件，Hugo还可使用 `config.yaml` 或者 `config.json` 进行配置。
2. `content` 文件夹中存放所有的网站内容，可在此文件夹中建立其他子文件夹，即为子模块。
3. `layouts` 文件夹存放 `.html` 格式的模板。模板确定了静态网站渲染的样式。
4. `themes` 文件夹存放网站使用的theme主题模板。
5. `static` 文件夹存放未来网站使用的静态内容，比如图片、css、JavaScript等。当Hugo生成静态网站时，该文件夹中的所有内容会原封不动的被复制。
6. `archetypes` 文件夹存放网站预设置的文件模板头部，当使用 `hugo new` 时即可生成一个带有该头部的实例。
7. `data` 文件夹用来存储Hugo生成网站时应用的配置文件。配置文件可以是YAML，JSON或者TOML格式。



# 配置theme

可以在[这里](https://themes.gohugo.io/)找自己喜欢的主题。我暂时选择有搜索功能的meme，将主题clone到`themes`目录下：

```
git clone https://github.com/忘了/meme.git themes/meme
```

然后将`themes/meme/exampleSite/config.toml`模板配置文件复制到根目录，然后根据此文件来配置你的设置。

> PS：一定要把config.yaml中的theme修改为你使用的主体名称。比如我用的主题是meme，config.yaml文件就设置theme="meme"



# 创建新页面

创建一个新页面

```
hugo new about.md
```

此时 `content` 文件夹下就多了一个 `about.md` 文件，打开文件就可以看到时间、文件名等信息已经自动生成了

```
---
title: "about"
date: 2020-06-16T23:30:53+08:00
draft: true
---
```

两条 `---` 间的信息是文章的配置信息，有的信息是自动生成的 (如：`title`、`date` 等)，简单介绍以下各项配置

> - **以下项目是自动生成的:**
> - `title:` # 文章标题
> - `date:` # 写作时间
> - `draft:` # 是否为草稿，如果为 `true` 需要在命令中加入 `--buildDrafts` 参数才会生成这个文档
> - **以下项目需要自行添加:**
> - `description:` # 描述
> - `tags:` # 标签，用于文章分类
> - 等等

`自动生成` 和 `执行添加` 的内容并不是绝对的，你可以根据自己的喜好配置模板文件 `archetypes/default.md`



# 生成网站

设置完`config.toml` 后我们执行以下命令

```
hugo server --buildDrafts -w
```

此时你就可以在 `http://localhost:1313` 访问到你的博客了。

此时你的博客目录下就会多出一个`public`目录，这是Hugo生成的网站。



简单介绍一下两个参数：

- `--buildDrafts`: 生成被标记为 「草稿」 的文档
- `-w`: 监控更改，如果发生更改直接显示到博客上 

> PS：但此时只能在本地访问 (相当于预览博客，如果与期望值不符，可以随时更改)，如果想发布到 `Github Pages` 上需要先将文章配置信息中的 `draft:` 改为 `false` ，
>
> 然后执行命令
>
> ```
> hugo
> ```
>
> 

# 

# GitHub Pages部署

参考[这里](https://help.github.com/articles/user-organization-and-project-pages/)，在Github Pages有四种类型，而对于非组织型用户来说有两种，一种是用户的个人网站，网页域名为 `username.github.io`，另一种为Project的主页，网页域名为 `username.github.io/projectname`。Github Pages对于Project主页的源码要求有了修改，现在也可以放置在master上，之前版本中必须放在`gh-pages` 分支上，不过这里暂且不提，主要还是关心用户个人主页。

这就需要你在Github上建立一个以 `username.github.io` 为名称的repository，对于我来说就是 `affectalways.github.io`。此外，需要将Hugo生成的所有静态网页push到这个repository的master分支上。现在就可以用这个域名打开个人网站了。

Hugo没有提供自动发布到GitHub Pages的功能。需要将`public`中的内容手动上传到Github上。

首先执行命令`cd public`进入到`public`目录，然后执行

```
git init
git remote add origin https://github.com/[Github 用户名]/[Github 用户名].github.io.git
git add .
git commit -m "[介绍，随便写点什么，比如日期]"
git push （若是第一次发布，需要用到--set-upstream）
```