---
title: "Werkzeug 源码解析(1)"
tags: ["flask", "python", "werkzeug"]
keywords: 
- flask
- 源码
- werkzeug
- python
- source code
- hugo
- blog
date: 2020-06-23T23:08:16+08:00
categories: ["python", "werkzeug"]
draft: false
---

# Werkzeug是什么？

`Werkzeug`是一个基于[WSGI](https://affectalways.github.io/posts/wsgi/wsgi_kl_1/)的Web应用框架（说框架可能不合理，[官方文档](https://werkzeug.palletsprojects.com/en/1.0.x/)给出的是`应用程序库`）。想要了解更多请看[官方文档](https://werkzeug.palletsprojects.com/en/1.0.x/)





# 为什么要了解Werkzeug？

因为目前所用的web框架为Flask，而Flask是以Werkzeug为基础的，所以绕不开Werkzeug了。





# 基础知识

1. [WSGI](https://affectalways.github.io/posts/wsgi/wsgi_kl_1/)
2. Python
3. 生成器
4. 非常简单的网络知识