# git status显示中文和解决中文乱码

##### **目录**

- [现象](#现象)
- [原因](#原因)
- [解决办法](#解决办法)



##### 现象

status查看有改动但未提交的文件时总只显示数字串，显示不出中文文件名，非常不方便。如下图：
![img](D:\git_code\Flee-as-a-bird-to-your-mountain\Git\pictures\解决git不显示中文1.png)



##### 原因

在默认设置下，中文文件名在工作区状态输出，中文名不能正确显示，而是显示为八进制的字符编码。



##### 解决办法

将git 配置文件 `core.quotepath`项设置为false。
quotepath表示引用路径
加上`--global`表示全局配置

git bash 终端输入命令：

```
git config --global core.quotepath false
```

要注意的是，这样设置后，你的git bash终端也要设置成中文和utf-8编码。才能正确显示中文，例如：

![](D:\git_code\Flee-as-a-bird-to-your-mountain\Git\pictures\解决git不显示中文2.png)

在git bash的界面中右击空白处，弹出菜单，选择选项->文本->本地Locale，设置为zh_CN，而旁边的字符集选框选为UTF-8。

英文显示则是：
Options->Text->Locale改为zh_CN，Character set改为UTF-8

如图：

![](D:\git_code\Flee-as-a-bird-to-your-mountain\Git\pictures\解决git不显示中文3.png)

