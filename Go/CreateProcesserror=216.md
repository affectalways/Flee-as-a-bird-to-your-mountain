## CreateProcess error=216, 该版本的 %1 与你运行的 Windows 版本不兼容。请查看计算机的系统信息，然后联系软件发布者

#### 目录

- [问题描述](#问题描述)
- [解决](#解决)



#### 问题描述

```go
package src

import "fmt"

func main() {
	fmt.Println("hello!")
}

```

```
CreateProcess error=216, 该版本的 %1 与你运行的 Windows 版本不兼容。请查看计算机的系统信息，然后联系软件发布者。
```





#### 解决

入口的第一行改为main

```go
package main

import "fmt"

func main() {
	fmt.Println("hello!")
}

```

