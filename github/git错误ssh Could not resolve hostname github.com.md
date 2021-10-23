# git错误：ssh: Could not resolve hostname github.com

1. ping github.com，发现压根ping不通
2. 打开网络管理，编辑dns，将本地dns服务器修改为8.8.8.8（谷歌的）或114.114.114.114（国内的）
3. 然后ping github.com就可以了

