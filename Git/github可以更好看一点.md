# Github可以更好看一点

## 表情

Github的Markdown语法支持添加emoji表情，输入不同的符号码（两个冒号包围的字符）可以显示出不同的表情。

比如`:blush:`，可以显示😊。

具体每一个表情的符号码，可以查询GitHub的官方网页[http://www.emoji-cheat-sheet.com](http://www.emoji-cheat-sheet.com/)。

但是这个网页每次都打开**奇慢**。。。

有位大佬整理了一下，大家可以直接在此查看[emoji](https://github.com/guodongxiaren/README/blob/master/emoji.md)。



## 图标

可以参考网站[shields.io](https://shields.io/)



## 目录

平常的 markdown 语法支持自动生成目录，使用 `[toc]` 或者 `@[toc]` ，markdown 编辑器就能根据 **标题** 形成目录，也就是说，目录是根据标题语法 `#` 索引的。

GitHub 不支持目录这一语法，如果你的 ReadMe 太长，可以用如下方法生成目录索引。

比如你的标题如下：

```text
# Heading One
# Heading Two
## AAA
## bbb
```

效果如下：

![image-20211014230643560](https://github.com/affectalways/Flee-as-a-bird-to-your-mountain/blob/main/github/pictures/%E6%95%88%E6%9E%9C%E5%9B%BE.png?raw=true)

可以使用语法`[](#)`链接某个标题，比如：

```text
# Contents
- [Heading One](#heading-one)
- [Heading Two](#heading-two)
	- [AAA](#aaa)
	- [bbb](#bbb)
```

就可以得到下面的目录效果：

![](https://github.com/affectalways/Flee-as-a-bird-to-your-mountain/blob/main/github/pictures/%E6%95%88%E6%9E%9C%E5%9B%BE2.png?raw=true)

但是有几个注意事项：

```
- [] 里面的内容可以随便写，但最好和链接标题一致
- () 内无论链接哪一级标题，都只写一个 `#`
- () 内链接的标题英文全部是小写，只能链接一个单词，多个单词需要用 `-` 连接成一个单词，当然中文也可以链接
- 可以使用 `Tab` 键描述不同级别的标题
```

