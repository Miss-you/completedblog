#go简易tcp socket客户端和服务器

##1.Socket编程

以前使用Socket编程时，一般是如下步骤

1. 建立socket，socket
2. 绑定socket，bind
3. 监听，listen
4. 接受连接，accept
5. 接受/发送，recv/send

go tcp版真的很省事

> 服务端：

就是Listen、Accept、Read/Write

> 客户端

就是Dial、Read/Write

直接上代码

##2.Server端

```
package main

import (
	"fmt"
	"net"
	"os"
)

func checkError(err error){
	if 	err != nil {
		fmt.Println("Error: %s", err.Error())
		os.Exit(1)	
	}
}

func recvConnMsg(conn net.Conn) {
//	var buf [50]byte
	buf := make([]byte, 50) 
	
	defer conn.Close()
	
	for {
		n, err := conn.Read(buf)
		
		if err != nil {
			fmt.Println("conn closed")
			return	
		}	
		
		//fmt.Println("recv msg:", buf[0:n])
		fmt.Println("recv msg:", string(buf[0:n]))
	}	
}

func main() {
	listen_sock, err := net.Listen("tcp", "localhost:10000")
	checkError(err)
	defer listen_sock.Close()
	
	for {
		new_conn, err := listen_sock.Accept()
		if err != nil {
			continue	
		}	
		
		go recvConnMsg(new_conn)
	}
		
}
```

##3.Client端

```
package main

import (
	"fmt"
	"net"
	"os"
)

func checkError(err error){
	if 	err != nil {
		fmt.Println("Error: %s", err.Error())
		os.Exit(1)	
	}
}

func main() {
	conn, err := net.Dial("tcp", "127.0.0.1:10000")
	checkError(err)
	defer conn.Close()	
	
	conn.Write([]byte("Hello world!"))	
	
	fmt.Println("send msg")
}
```