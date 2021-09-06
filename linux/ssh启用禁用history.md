---
title: "Ssh启用禁用history"
date: 2020-11-09T22:58:41+08:00
tags: ["linux"]
keywords: 
- ssh
- history
description: "Ssh启用禁用history"
categories: ["ssh"]
draft: false
---

## SSH 启用禁用history

想在terminal中执行上一条命令，按了向上的方向键，但是没反应，以为键盘坏了，确定了下，原来是`history`命令被禁了



在命令行终端依次执行以下命令：

```bsh
set -o | grep history
```

如果上述命令的输出结果为 "history off"， 就在 `~/.bashrc`文件的结尾加上如下命令:

```bsh
set -o history
```

接着执行:

```bsh
echo $HISTFILE
echo $HISTSIZE
echo $HISTFILESIZE
```

如果第一个命令结果为`空`或者 `/dev/null`, 把下列命令加到 `~/.bashrc`结尾:

```bsh
HISTFILE=$HOME/.bash_history
```

如果后面的两个命令任意一个执行结果为0，就把他们的值设置成` > 0`的，如 500:

```bsh
HISTFILESIZE=500
HISTSIZE=500
```

切记，不要忘了在保存`~/.bashrc`后执行 `source .bashrc`