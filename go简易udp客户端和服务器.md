#go简易udp socket客户端和服务器

##1.Socket编程

以前使用Socket编程时，一般是如下步骤

1. 建立socket，socket
2. 绑定socket，bind
3. 监听，listen
4. 接受连接，accept
5. 接受/发送，recv/send

Go语言对其进行了抽象和封装，刚开始接触有可能不太适应（譬如我第一天用的时候觉得API好难找……建议参考文档），后来发现用起来很爽

简单来说，客户端省去了很多！客户端只需要调用net.Dial()即可，服务器我这里还需要摸索一下，但是也是很简单了，不过流程感觉没简化-  -

**废话不多说，直接上代码**

##2.Server端
```
import (
	"os"
	"fmt"
	"net"
)

func checkError(err error){
	if 	err != nil {
		fmt.Println("Error: %s", err.Error())
		os.Exit(1)	
	}
}

func recvUDPMsg(conn *net.UDPConn){
	var buf [20]byte
	
	n, raddr, err := conn.ReadFromUDP(buf[0:])
	if err != nil {
		return	
	}
	
	fmt.Println("msg is ", string(buf[0:n]))
	
	//WriteToUDP
	//func (c *UDPConn) WriteToUDP(b []byte, addr *UDPAddr) (int, error) 
	_, err = conn.WriteToUDP([]byte("nice to see u"), raddr)
	checkError(err)
}

func main() {
	udp_addr, err := net.ResolveUDPAddr("udp", ":11110")
	checkError(err)	
	
	conn, err := net.ListenUDP("udp", udp_addr)
	defer conn.Close()
	checkError(err)	
	
	//go recvUDPMsg(conn)
	recvUDPMsg(conn)
}
```

流程

1. 先通过net.ResolveUDPAddr创建监听地址
2. net.ListenUDP创建监听链接
3. 然后通过conn.ReadFromUDP和conn.WriteToUDP收发UDP报文


##3.Client端

```
package main

import (
	"os"
	"fmt"
	"net"
//	"io"
)

func main() {
	conn, err := net.Dial("udp", "127.0.0.1:11110")
	defer conn.Close()
	if err != nil {
		os.Exit(1)	
	}
	
	conn.Write([]byte("Hello world!"))	
	
	fmt.Println("send msg")
	
	var msg [20]byte
	conn.Read(msg[0:])
	
	fmt.Println("msg is", string(msg[0:10]))
}
```

客户端非常简单，先通过net.Dial("udp", "127.0.0.1:11110")，建立发送报文至本机11110端口的socket，然后使用conn.Write和conn.Read收发包，当然conn.ReadFromUDP和conn.WriteToUDP也是可以的

