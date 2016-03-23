#苹果电脑MAC安装nginx教程

> 使用homebrew安装，**提醒需要翻墙**，最后面介绍的源码安装方式不需要翻墙，只需要在墙内搞到源码即可，建议先确认一下自己能否翻墙，如果可以翻墙就用第一种方法，如果不能就用第二种。。。

```
sudo brew search nginx
sudo brew install nginx
```

启动nginx ，`sudo nginx`启动nginx ;然后，访问localhost:8080 发现已出现nginx的欢迎页面了，说明nginx安装成功

**备注： ln -s  /usr/local/sbin/nginx /usr/bin/nginx 做个软连接。**

常用的指令有： 

1. nginx -V 查看版本，以及配置文件地址
2. nginx -v 查看版本
3. nginx -c filename 指定配置文件
4. nginx -h 帮助

###重新加载配置|重启|停止|退出 nginx

`nginx -s reload|reopen|stop|quit`
###打开 nginx

`sudo nginx`
###测试配置是否有语法错误

`nginx -t`

另外附上Mac安装brew命令：

```
curl -LsSf http://github.com/mxcl/homebrew/tarball/master | sudo tar xvz -C/usr/local --strip 1
```

当brew安装成功后，就可以随意安装自己想要的软件了，例如wget，命令如下：

`sudo brew install wget ` 

卸载的话，命令如下：

`sudo brew uninstall wget`

查看安装软件的话，命令如下：

`sudo brew search /apache*/`

注意/apache*/是使用的正则表达式，用/分割。

> 使用源文件进行安装，**可以不需要翻墙**

以下是在mac os x 10.11.3 安装nginx步骤(10.9安装方式没什么变化，可以放心安装)

##1.安装PCRE

```
1、Download latest PCRE. 
2、安装
$ cd ~/Downloads
##当然你的pcre源文件是下载到了Downloads目录
$ tar xvzf pcre-8.5
##版本可能不同pcre-8.5
$ cd pcre-8.5
$ sudo ./configure --prefix=/usr/local
$ sudo make
$ sudo make install
```

##2.安装Nginx

```
1、Download latest nginx from Nginx.org. 
2、安装
$ cd ~/Downloads
$ tar xvzf nginx-1.6.0.tar.gz
$ cd nginx-1.6.0
$ sudo ./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-cc-opt="-Wno-deprecated-declarations"
$ sudo make
$ sudo make install
```

##3.开启Nginx

```
1、将/usr/local/nginx/sbin加入到环境变量里
2、运行
$ sudo nginx 
3、打开浏览器 http://localhost，如果看到如下界面表明nginx启动正常了
```

##4.停止

`$ sudo nginx -s stop`
