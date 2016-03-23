#安装zmq库

> 环境：ubuntu

```
（CAUTION， build ZMQ）To build on UNIX-like systems
If you have free choice, the most comfortable OS for developing with ZeroMQ is probably Ubuntu.
Make sure that libtool, pkg-config, build-essential, autoconf, and automake are installed.
Check whether uuid-dev package, uuid/e2fsprogs RPM or equivalent on your system is installed.
Unpack the .tar.gz source archive.
Run ./configure, followed by make.
To install ZeroMQ system-wide run sudo make install.
On Linux, run sudo ldconfig after installing ZeroMQ.
To see configuration options, run ./configure --help. Read INSTALL for more details.
```

先引用官方安装教程，简述一下自己的安装zmq过程

1. 安装zmq首先需要基本依赖库，libtool, pkg-config, build-essential, autoconf, automake，相应安装一下即可
2. 其次需要uuid-dev库（当然uuid/e2fsprogs库也可以）
3. 上面没有写的还有一个依赖库，zmq库会依赖其进行加密（当然你可以在安装的时候通过设置configure文件来选择不使用加密也可以，相当于一个可选库），这个库是libsodium
4. 之后就安装zmq库和czmq库（我的程序需要）

1.2.条就不说明如何安装，从3.开始

去官网下载libsodium库源码，然后编译安装

```
wget https://github.com/jedisct1/libsodium/releases/download/1.0.8/libsodium-1.0.8.tar.gz
tar xvzf libsodium-1.0.8.tar.gz
./autogen.sh
./configure
make && make check
make install
```

去官网下载zmq和czmq库源码，然后编译安装,**注意，zmq库要先于czmq安装**

安装zmq

```
wget http://download.zeromq.org/zeromq-4.1.4.tar.gz
tar xvzf zeromq-4.1.4.tar.gz
./autogen.sh && ./configure && make -j 4
make check && make install && sudo ldconfig
```

安装czmq

```
wget https://github.com/zeromq/czmq/archive/v3.0.2.tar.gz
tar xvzf czmq-3.0.2.tar.gz 
./autogen.sh && ./configure
make -j 4 && make check
make install
ldconfig
```

这样就可以使用了


##遇到的问题

**ERROR1**

```
error：./configure: line 19267: syntax error near unexpected token `sodium,'
./configure: line 19267: `    PKG_CHECK_MODULES(sodium, libsodium, have_sodium_library="yes")'
```

一种解决办法是把/usr/share/aclocal/pkg.m4文件拷贝到/usr/share下面其他有版本的aclocal-x.x.x下面
譬如

```
cp /usr/share/aclocal/pkg.m4 /usr/share/aclocal-1.13/
```

**如果不行的话，确认libsodium库安装成功之后，将zmq的文件删掉，重新编译安装一次**

**ERROR2**

```
make[1]: Leaving directory `/home/ubuntu/EPC/gw/ctl'
g++ -o gw ./obj/*.o -levent -lsctp -lpthread -lzmq -lczmq -lrt -lcurl -lmemcached 
./obj/NwGtpv2c.o: In function `initSae0MqConn(NwGtpv2cIfT*)':
/home/ubuntu/EPC/gw/nw-gtpv2c/NwGtpv2c.cpp:1031: undefined reference to `zmq_ctx_new'
/home/ubuntu/EPC/gw/nw-gtpv2c/NwGtpv2c.cpp:1069: undefined reference to `zmq_ctx_destroy'
collect2: error: ld returned 1 exit status
make: *** [all] Error 1
```
这个问题有两种情况：
1. 编译的时候没有引用zmq库，即需要在gcc 后面加-lzmq
2. 安装的库过老，或者是安装了多个库，而引用的是较老的库，导致某些API无法使用

第一种情况举例

```
gcc -o -g -Wall test ./obj/*.o -lzmq 
```
第一种情况在使用的时候，链接zmq库即可

第二种情况，有可能是，你手动下载源码安装了zmq库一次，又使用apt-get安装了一个比较老的库，这时候ubuntu系统下会默认引用apt-get安装的库，所以需要卸载掉apt-get安装的库

```
apt-get remove libzmq-dev
```