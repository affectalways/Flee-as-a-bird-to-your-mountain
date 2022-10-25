## go: go.mod file not found in current directory or any parent directory； see ‘go help modules‘

#### 目录

- [问题描述](#问题描述)
- [解决办法](#解决办法)





#### 问题描述

![](https://raw.githubusercontent.com/affectalways/Flee-as-a-bird-to-your-mountain/main/img/go.mod1.png)

当go在goland初次运行时提示如下信息：

```
go: go.mod file not found in current directory or any parent directory; see 'go help modules'
```



#### 解决办法

翻了下文档，发现是 **跳过新手教程的错误玩法**

在新建GO项目后，要在新建的项目文件夹内执行

```
go mod init {项目名}
```

![](https://raw.githubusercontent.com/affectalways/Flee-as-a-bird-to-your-mountain/main/img/go.mod2.png)



然后进入再用goland执行就可以了

![](D:\git_code\Flee-as-a-bird-to-your-mountain\Go\images\go.mod3.png)