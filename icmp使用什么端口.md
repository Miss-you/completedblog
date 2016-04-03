#ICMP使用什么端口

> ICMP使用什么端口？PING操作又是使用什么端口？

1. ICMP是Internet控制信息协议（ICMP），是IP组的一个整合部分。通过IP包传送的ICMP信息主要用户涉及网络操作或错误操作的不可达信息。ICMP包发送是不可靠的，所以主机不能依靠接收ICMP包解决任何网络问题。ICMP不像TCP/UDP有端口，但它确实含有两个域：类型type和代码code。但是这个域的作用与TCP/UDP的端口作用也完全不同。
2. Ping用到了ICMP协议

##延伸：关于ICMP echo (PING操作)

1. 首先查询本地arp cache信息，看是否有对方的mac地址和IP地址映射条目记录
2. 如果没有，则发起一个arp请求广播包，等待对方告知具体的mac地址
3. 收到arp相应包之后，获得某个IP对应的具体mac地址，有了物理地址之后才可以开始通信了，同时对ip-mac地址做一个本地cache
4. 发出icmp echo request包，收到icmp echo reply