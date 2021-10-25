# Python格式化Curl返回的Json

------



在 `API` 调试过程中除了使用 `GUI` 工具（类似：`Postman`）我最常使用的就是 `curl` 命令了 ，简单快捷，但是 `curl` 的输出结果不是特别友好，特别是 `json` 格式，会在命令行里输出成一个长字符串，没法看。

```bash

curl https://test.com/api/test | python -m json.tool

```



## 隐藏 curl 统计信息

在使用上面的格式化命令时，curl 会在输出结果前先输出一段统计信息类似：

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   427  100   427    0     0  17300      0 --:--:-- --:--:-- --:--:-- 17791
```



我们可以使用 curl 的 -s 参数来隐藏这段统计信息：

```
curl -s https://test.com/api/test | python -m json.tool

curl -s https://test.com/api/test | json
```