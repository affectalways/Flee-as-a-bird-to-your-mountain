---
title: "Bug：python第三方库cx_Oracle生成大量xml日志"
date: 2020-11-12T21:48:01+08:00
tags: ["python"]
keywords: 
- python
- blog
- 博客
- cx_Oracle
- log xml
- celery
description: "Bug：python第三方库cx_Oracle生成大量xml日志"
categories: ["python"]
draft: false
---

## Bug：python第三方库cx_Oracle生成大量xml日志

#### 又tm是cx_Oracle！

线上环境突然无法访问前端，后端排查发现服务都挂了，根据经验，用

```
df -Th
```

命令，发现根目录磁盘空间满了。

然后通过

```
du -s * | sort -nr
```

依次进入占用最大空间的目录，发现在`/var/tmp/.../alert/`目录存在大量的`log*.xml`文件，每个xml文件大小为11M，有几千个。

通过谷歌发现，这些多余的日志文件确实是cx_Oracle这个第三方库产生的，但根据搜索到的内容是oracle server产生的，但线上项目并没有用到oracle server，可能是安装的依赖instantclient导致的，因为不开源，也懒得深究了。

就因为不开源，所以就直接写了定时脚本，定时清除。

定时脚本用的是`crontab`，因为具体代码不方便展示，所以就不写了。

并且此处get一个新技能：

```
蠢到将多个命令分开用subprocess执行，第一条命令是进入/var/tmp目录，然后执行查询，再执行删除操作。却发现怎么也删不了，所以用本地的terminal，发现subprocess执行cd命令，后续的subprocess命令执行其实还是在之前的目录执行，所以需要subprocess同时执行多条命令。

subprocess同时执行多个命令

subprocess.Popen("命令1 && 命令2 && ...")
```

