#linux基础指令（三）scp

> linux的scp命令可以在linux之间复制文件和目录（linux和mac之间也可以）
 

================== 
scp 命令 
================== 
scp 可以在 2个 linux 主机间复制文件； 


##命令基本格式：
 
```
scp [可选参数] file_source file_target 
       
NAME
     scp -- secure copy (remote file copy program)

SYNOPSIS
     scp [-12346BCEpqrv] [-c cipher] [-F ssh_config] [-i identity_file]
         [-l limit] [-o ssh_option] [-P port] [-S program]
         [[user@]host1:]file1 ... [[user@]host2:]file2
```

常用参数：

- -r，复制目录用
- -i identity_file，如果连接该虚拟机需要密钥，则需要该命令
- -v，和大多数 linux 命令中的 -v 意思一样 , 用来显示进度 . 可以用来查看连接 , 认证 , 或是配置错误 . 
- -C，使能压缩选项 . 
- -P，选择端口 . 注意 -p 已经被 rcp 使用 . 
- -4，强行使用 IPV4 地址 . 
- -6强行使用 IPV6 地址 .

====== 

##常用操作示例

###从本地复制到远端

命令格式

```
scp local_file remote_username@remote_ip:remote_folder 

例如
sudo scp ./tokyo.pem  ubuntu@54.238.202.90:/home/ubuntu/
```

###从远端复制到本地

命令格式

```
scp remote_username@remote_ip:remote_folder local_file 

例如
sudo scp ubuntu@54.238.202.90:/home/ubuntu/tokyo.pem ./tokyo1.pem
或者
sudo scp ubuntu@54.238.202.90:/home/ubuntu/tokyo.pem ./
```
**如果是目录，则在拷贝目录路径前加-r即可**

###实际使用示例

从本地拷贝到远端

```

feiqianyousadeMacBook-Pro:key-2 yousa$ sudo scp -i ./tokyo.pem tokyo.pem  ubuntu@54.238.202.90:/home/ubuntu/
tokyo.pem                                     100% 1692     1.7KB/s   00:00
feiqianyousadeMacBook-Pro:key-2 yousa$
```

从远端拷贝回来本地

```
feiqianyousadeMacBook-Pro:key-2 yousa$ sudo scp -i ./tokyo.pem  ubuntu@54.238.202.90:/home/ubuntu/tokyo.pem tokyo1.pem
tokyo.pem                                     100% 1692     1.7KB/s   00:00
feiqianyousadeMacBook-Pro:key-2 yousa$
```
===

##注意两点：
1.如果远程服务器防火墙有特殊限制，scp便要走特殊端口，具体用什么端口视情况而定，命令格式如下：

```
#scp -p 4588 remote@www.abc.com:/usr/local/sin.sh /home/administrator
```
2.使用scp要注意所使用的用户是否具有可读取远程服务器相应文件的权限。


