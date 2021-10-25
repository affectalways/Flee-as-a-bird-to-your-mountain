# Python subprocess执行xfreerdp命令遇到问题

## 背景

​		使用python的subprocess执行xfreerdp命令，远程登录windows，需要根据subprocess的返回值(stdout\stderr = subprocess.PIPE)判断是否登录成功，但是运行程序输出的结果同在linux服务器执行命令输出的结果相比，程序输出的结果总是**丢失**后面最关键的部分（**截图是不方便给的，可以自己手动复现**）



## 问题解析思路

​		既然是丢数据了，肯定是subprocess这块存在问题，因为就是从他创建的管道读取的，看了下subprocess的官方文档

> https://docs.python.org/zh-cn/3/library/subprocess.html
>
> bufsize* 将在 [`open()`](https://docs.python.org/zh-cn/3/library/functions.html#open) 函数创建了 stdin/stdout/stderr 管道文件对象时作为对应的参数供应:
>
> - `0` 表示不使用缓冲区 （读取与写入是一个系统调用并且可以返回短内容）
> - `1` 表示行缓冲（只有 `universal_newlines=True` 时才有用，例如，在文本模式中）
> - 任何其他正值表示使用一个约为对应大小的缓冲区
> - 负的 *bufsize* （默认）表示使用系统默认的 io.DEFAULT_BUFFER_SIZE。

​		所以根据最后一条，看一下 `io.DEFAULT_BUFFER_SIZE=8192`，这是远远不够的。

​		为了解决这个问题，也不能改默认值（会出问题的）。

​		所以只能从命令方面下手，因为xfreerdp提供了/log-filters参数，能对命令执行过滤，不过使用比较奇葩

```
DISPLAY=:0 xfreerdp /v:**** /u:***** /p:***** /cert-ignore +sec-nla /drive:远程共享目录,/临时共享目录 /log-filters:com.freerdp.core.capabilities:ERROR,com.winpr.sspi.NTLM:ERROR
```

```
以上对tag为com.freerdp.core.capabilities、com.winpr.sspi.NTLM进行过滤，只允许他们ERROR级别的展示，但是并不影响其他tag，其他的还是可以正常展示！！！
```



> 推荐：
>
> https://stackoverflow.com/questions/18194374/default-buffer-size-for-a-file-on-linux
>
> https://docs.python.org/zh-cn/3/library/subprocess.html
>
> https://www.hardening-consulting.com/en/posts/20193001-freerdp-and-wlog.html