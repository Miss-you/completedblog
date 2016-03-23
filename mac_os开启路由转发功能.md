#mac OS开启路由转发功能

> 主要是我有时候需要用mac进行ps4直播转发，需要mac开启路由功能

环境：MAC OS 10.11

机器：macbook pro

##配置方法

基本步骤跟linux类似，不过变量有些变化

```
sysctl -w net.inet.ip.forwarding=1
```

这样就开启了ipv4报文路由转发

路由转发是做啥的？当电脑开启路由转发功能的时候，电脑收到目标地址不是自己电脑的IP地址，不会丢掉，反而会进行路由搜索，发送给目标地址机器（如果能搜索到的话）或者是发送给电脑的相应指定网关或者默认网关

**上面的方法电脑重启之后就会没有了，若需要固化，即电脑重启配置依然在，需要写入配置文件，其他一些变量的固化修改也是同样操作**

sysctl的一些配置，电脑每次启动的时候就会读取sysctl.conf文件（如果有的话），配置变量，然后其他的缺省值则配置为默认，所以接下来就需要修改/etc/sysctl.conf文件，mac os如果你之前没有建立的话，是搜索不到/etc/sysctl.conf文件的，因为没有啊2333

所以你就需要创建该文件

```
sudo vim /etc/sysctl.conf
```

然后在配置文件中写入相应配置即可

```
net.inet.ip.forwarding=1
修改好之后，:wq保存
```

`注意，mac的某些配置变量名跟linux的配置变量名不同，需要自行查看`

修改好之后，这仅仅是保证电脑启动的时候生效，但是现在并没有生效，所以需要用sysctl配置一下，执行命令，使配置生效即可

```
sysctl -p
```

**linux开启的方法也类似~~~~**

##查看conf变量

使用sysctl查看ipv4路由转发是否开启，`forward`是转发的意思

```
sudo sysctl -a | grep forward
net.inet.ip.forwarding: 0
net.inet6.ip6.forwarding: 0
```

查看sysctl可以改变哪些变量配置

```
man sysctl
然后往下拉，就会有一个关于哪些变量可以修改
```

使用sysctl查看conf变量配置

```
sudo sysctl -a
...
```