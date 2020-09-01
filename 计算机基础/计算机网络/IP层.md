## 报文头

![preview](https://picb.zhimg.com/v2-a8702bdb6e9cf9fd29e824ac07542067_r.jpg)



## ip子网计算

- ABCD子网
- 根据子网掩码计算主机数量



## ping指令相关的协议是什么





## ICMP

- 最常用的两个命令：ping 和 tracert
  - ping 用来测试网络可达性
  - tracert 用来显示到达目的主机的路径。
- ICMP简单介绍
  - ICMP就是一个“错误侦测与回报机制”，其目的就是让我们能够检测网路的连线状况，也能确保连线的准确性。
  - **当路由器在处理一个数据包的过程中发生了意外，可以通过ICMP向数据包的源端报告有关事件。**
- ICMP其他功能
  - 侦测远端主机是否存在，建立及维护路由资料，重导资料传送路径（[ICMP重定向](https://baike.baidu.com/item/ICMP重定向)），资料[流量控制](https://baike.baidu.com/item/流量控制)。
  - ICMP在沟通之中，主要是透过不同的[类别](https://baike.baidu.com/item/类别)(Type)与[代码](https://baike.baidu.com/item/代码)(Code) 让机器来识别不同的连线状况。



