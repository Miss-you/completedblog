#sigma openstack虚拟机搭建指南

> 主要内容是从sigma已有的openstack及已有的img镜像，到搭建一个可以成功运行的GW虚拟机

目录

1. 连接sigma
2. 使用img文件创建镜像
3. 创建虚拟机
4. 使用vnc连接虚拟机
5. 进行相应初级配置，使之可以进行ssh连接
6. 对ip地址、dns、路由表等进行配置
7. 安装依赖库
8. 编译安装

##连接sigma

连接mlab的sigma必须得用里面的一台申请的台式机才可以，台式机现在ip为192.168.2.123，如果不通则需要自行查看

现阶段该台式机使用的是张浩的账号进行登录

远程连接之后，使用PuTTY连接sigma服务器，上面已经有记录，comput为计算节点，controller为控制节点

**但是如果长时间不连接comput或者controller节点，是需要先连接switch board，使能sigma口才可以**

`连接方式查看最后的ERROR1`

以下为账号密码，/左边为登录用户，/右边为密码**（过去的文档中记录的密码已经过期）**

```
172.28.0.2控制节点 
fsp/fsp300@HW
172.28.6.0计算节点
fsp/fsp300@HW1
```

两台节点的root用户信息均为

```
root/cnp200@HW
```

我这里使用的是controller节点

##使用img文件创建镜像

创建镜像这里使用的是已有的ubuntu 14.04 img文件

`trusty-server-cloudimg-amd64-disk1.img`

将该文件拷贝进controller节点，我这里是拷贝到了**/home/fsp/img/**目录下

> 使用openstack进行譬如镜像创建或者虚拟机创建的时候，需要先执行eport配置脚本，脚本在/root/env_sweet.sh

使用root用户先运行export配置脚本

```
source /root/env_sweet.sh
```

创建镜像

```
glance image-create --name ubuntu14.04 --container-format bare --disk-format qcow2 --is-public True --file /home/fsp/img/trusty-server-cloudimg-amd64-disk1.img
--name 是指镜像名字起名叫 ubuntu14.04
--disk-format ubuntu镜像的.img文件基本为qcow2类型
--file 是指取哪个目录下的文件
```

很快镜像就创建好了，下面就可以创建虚拟机了

##创建虚拟机

虽然已经创建好镜像，但是创建虚拟机是需要该镜像的ID，先查看ID

```
glance image-list
```

找到NAME为自己创建的镜像，这里是ubuntu14.04，可以看到它的ID是**bd5218c1-e6d0-4028-9be3-779e1cdbde85**

然后知道ID号之后，使用该镜像创建虚拟机，这里直接使用一个已经写好的的给GW使用的模板创建（因为GW虚拟机使用的是两个网卡，这里还有一些其他的设置）

```
nova boot --flavor 4 --hint hyperThreadAffinity=none --hint vcpuAffinity=[1]  --image bd5218c1-e6d0-4028-9be3-779e1cdbde85   --nic net-id=23a4bcd0-84ea-4190-8068-5ba66dcb087b --nic net-id=21f4e0fd-1911-46a3-9e08-03c8c0f7dcb7 ubuntu14.04_gw

ubuntu14.04_gw 为虚拟机名字
--image 为使用的镜像ID
```
使用命令查看虚拟机状态

```
nova list
```

##使用vnc连接虚拟机

VNC是在192.168.2.123该台式机上，D：vncviewer，安装的话在《Sigma一体机xxx》文档中有华为自己修改过的VNC工具，使用该VNC工具

登录前需要查看VNC的登录端口号，在controller虚拟机上查看虚拟机ID

```
virsh list
这里基本是ID instance_ID，需要的是ID
```

刚新建的环境一般是虚拟机最后一个，也可以使用另一个命令查看

```
nova show NAME
NAME是虚拟机的名字
```
这里可以获得所需虚拟机的instance_ID，然后再通过virsh list找到相应ID号，这里找到ID号为7

再使用命令查看VCN的登录端口号

```
virsh vncdisplay 7
172.28.0.2：2
```

前面**nova show NAME**的时候，还可以获得虚拟机的id号（这俩ID不一样），然后使用命令获取密码

```
nova get-vnc-console xxx novnc
xxx为虚拟机的id号
然后得到一串字符串，选中字符串最后，这里是=encode-xxxxx=|
选中'xxxxx='部分，然后这个值用程序进行计算，就可以获得vnc登录密码
```

新建一个test.py文件，加入如下内容保存

```
import base64
import sys

print (base64.b64decode(sys.argv[1]))
```

运行

```
python test.py xxxxx=
然后就可以获得密码，这个密码就是用VNC登录虚拟机所需要的密码
```

这个密码较短时间内就会过期，所以需要快速操作

> 注意，该镜像需要有可以使用用户名-密码登录，否则无法进入

**如果镜像本身没有用户名-密码登录的用户，那么可以使用guestfish工具对镜像进行修改，加入用户。**

简单思路就是修改一些配置文件，添加用户，这个可以咨询许强

##进行相应初级配置，使之可以进行ssh连接

因为这是一个内网，创建使用ifconfig创建一个ip就可以使用内网的电脑进行ssh连接了

```
ifconfig eth1 192.168.2.189 netmask 255.255.255.0
```

这样就可以连接192.168.2.189虚拟机了

##对ip地址、dns、路由表等进行配置

对默认路由表修改需要使用vnc进行修改

```
#删除默认路由
sudo route del default
#添加新的默认路由
sudo route add default gw 192.168.2.1
```

刚创建的虚拟机可能无法联网，是因为没有设置DNS服务器，所以设置dns服务器

```
sudo vim /etc/resolv.conf
添加
nameserver 1.2.4.8
#国内
nameserver 8.8.8.8
nameserver 8.8.4.4
#国外
```

GW正常工作是需要虚拟机开启路由转发功能

```
sudo vim /etc/sysctl.conf 
net.ipv4.ip_forward=1
```

这里如果还需要连接真实基站的话，还需要对网关gateway的路由表进行修改

##安装依赖库

下面的安装顺序建议按序安装

```
#安装gcc等基础工具
sudo apt-get install build-depgcc
apt-get install autoconf
apt-get install libtool

#安装iptables开发库
sudo apt-get install iptables-dev

#安装libevent开发库
apt-get install libevent-dev

#安装sctp开发库
apt-get install libsctp-dev

#安装zmq和czmq库
#先安装libsodium库
wget https://github.com/jedisct1/libsodium/releases/download/1.0.8/libsodium-1.0.8.tar.gz
tar xvzf libsodium-1.0.8.tar.gz
./autogen.sh
./configure
make && make check
make install
#再安装zmq库
wget http://download.zeromq.org/zeromq-4.1.4.tar.gz
tar xvzf zeromq-4.1.4.tar.gz
./autogen.sh && ./configure && make -j 4
make check && make install && sudo ldconfig
#最后安装czmq库
wget https://github.com/zeromq/czmq/archive/v3.0.2.tar.gz
tar xvzf czmq-3.0.2.tar.gz 
./autogen.sh && ./configure
make -j 4 && make check
make install
ldconfig

#安装libcurl库
apt-get install libcurl4-nss-dev

#安装memcache库
apt-get install libmemcached-dev

```

具体请查看《依赖库问题记录》

##编译安装

安装gtpu模块

```
#进入root用户
sudo su

make clean;make
需要先清理一下，再编译安装，该操作会自动卸载模块、清空iptables、然后删除各种.ko.so文件，编译内核模块和.so文件，进行相应安装，最后自动加载xt_GTPU模块

使用dmesg -c查看安装是否成功
dmesg -c
成功的话可以查看到模块成功加载的打印
lsmod
成功的话，可以查看到系统记录到了模块已经成功加载
```

安装gw

```
make clean;make
```

**CAUTION**

这里有个需要注意的就是，编译安装成功之后，需要对gtpu目录和gw目录进行授权，让其他普通用户也可以运行里面的脚本

```
chmod -R 0777 /gtpu
chmod -R 0777 /gtpu
或者是
chmod -R 0777 *
```

否则gtpu里面的脚本，在自动部署的时候有时候会执行失败

##问题记录

**ERROR：解决调试机172网段不通问题**

远程连接使用PuTTY，连接switch board，上面有链路记录

连接交换板串口(COM1)：波特率115200

登录交换板。用户名guest，密码Visit@7*24，进入之后切换成root模式

```
su -l //携带了环境变量
Admin@7*24 //密码
```

sigma base口使能，执行命令

```
baseset unshutdown 1/2/0
```