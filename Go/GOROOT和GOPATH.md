# GOROOT和GOPATH

GOROOT和GOPATH都是环境变量

- GOROOT：GOROOT就是Go的安装目录（类似于Java的JDK）

- GOPATH是一个环境变量，用来表明你写的`go`项目的存放路径。

  - 使用msi安装的golang,会为你配置默认的环境变量，我们需要修改`GOPATH`的路径，设置为一个比较明显的路径。

  ```
  创建GOPATH目录，在目录下创建三个文件夹
  bin:用来存放编译后生成的可执行文件
  pkg:用来存放编译后生成的归档文件
  src:用来存放源码文件
  ```

  - 在进行`Go`语言开发的时候，我们的代码总是会保存在`$GOPATH/src`目录下。在工程经过`go build`、`go install`或`go get`等指令后，会将下载的第三方包源代码文件放在`$GOPATH/src`目录下， 产生的二进制可执行文件放在 `$GOPATH/bin`目录下，生成的中间缓存文件会被保存在 `$GOPATH/pkg` 下。

  - 如果我们使用版本管理工具（`Version Control System`，`VCS`。常用如`Git`）来管理我们的项目代码时，我们只需要添加`$GOPATH/src`目录的源代码即可。`bin` 和 `pkg` 目录的内容无需版本控制。