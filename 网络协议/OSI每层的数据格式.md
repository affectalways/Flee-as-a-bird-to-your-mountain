### OSI每层的数据格式

第一层：物理层，二进制传输，bit(比特流)
第二层：数据链路层，介质访问，frame(帧)
第三层：网络层，确定地址和最佳路径,报(包)
第四层：传输层，端到端连接，segment(段)
第五层：会话层，互连主机通信
第六层：表示层，数据表示
第七层：应用层，为应用程序提供网络服务

五至七层为节点传输，发送和接收消息。
数据发送时，从第七层传到第一层，接收数据则相反。
上三层总称应用层，用来控制软件方面。下四层总称数据流层，用来管理硬件。