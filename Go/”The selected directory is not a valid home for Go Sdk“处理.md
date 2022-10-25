# "The selected directory is not a valid home f

# or Go Sdk"处理



![](https://raw.githubusercontent.com/affectalways/Flee-as-a-bird-to-your-mountain/main/img/gosdk.webp)

昨天go从1.15升级到1.17后发现goland一直无法配置sdk，提示"The selected directory is not a valid home for Go Sdk"如上图

```
检查环境变量无问题

GOROOT 安装路径无问题

cmd 测试 golang 命令正常使用

重启电脑无法解决
```



### 解决办法

1.执行go version 找到自己安装的详细版本
2.编辑{GOROOT}/src/runtime/internal/sys/zversion.go文件
添加

```
const TheVersion = `go1.17.4`
```

3.重启goland即可发现