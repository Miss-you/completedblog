#Ubuntu14.10系统 apt-get update失败解决办法

> 实验室服务器使用的是Ubuntu14.10，使用apt-get的时候发现ubuntu和阿里云均已经不提供该版本的源，所以需要找到其他的替代源，找到这篇文章解决了我的问题，现在转载一下，方便以后查阅

笔者使用的ubuntu版本是14.10，属于非LTS（长期支持版本），因此前一段时间还可以使用apt-get update来更新源，现在已提示更新失败，无法下载，无法访问了。现提供一种解决思路供大家参考。如网友有其他有效方法，可以一块讨论。

####第一步

Ubutun版本的更新比较快，目前只有10.04,12.04,14.04，以及后续的16.04会支持长期维护，时间长达3-5年，而其他常规版本的维护期比较短，基本是一年以内。而笔者使用的14.10，已经停止更新了好一段时间，因为平时用的还可以，所以也就没怎么更新，直到今天要安装一些东西了，才发现update不能用了。

####第二步
在网上找了很多Ubuntu14.10对应的源的列表，然后把它们加入到了系统的源列表中，可还是不行。但是，所有停止维护的版本都可以使用old源。所以在元列表中把原来的地址改为带有old源的就可以了。

####第三步
首先，备份系统中的源列表，打开终端，输入：**sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup**

![image](http://img.bitscn.com/upimg/allimg/c151225/14510154b921Z-36128.jpg)


####第四步
输入：sudo gedit /etc/apt/sources.list，打开源列表文件，**（这里我的做法跟文章不一样，我这里是直接在releases前面加old-即可，具体思路是这样，寻找一个可用的源，而有人注册了old-releases.ubuntu.com这个域名来提供ubuntu镜像源服务，这个源就是工具包可能会比较老，请注意版本，若需要最新版请手动去工具官网或者github手动下载源码、编译、安装）**ctrl+A，然后delete，删除全部内容，然后把下面的地址复制到该文件中


```
deb http://old-releases.ubuntu.com/ubuntu utopic main restricted universe multiverse   
  
deb http://old-releases.ubuntu.com/ubuntu utopic-security main restricted universe multiverse   
  
deb http://old-releases.ubuntu.com/ubuntu utopic-updates main restricted universe multiverse   
  
deb http://old-releases.ubuntu.com/ubuntu utopic-proposed main restricted universe multiverse   
  
deb http://old-releases.ubuntu.com/ubuntu utopic-backports main restricted universe multiverse   
  
deb-src http://old-releases.ubuntu.com/ubuntu utopic main restricted universe multiverse   
  
deb-src http://old-releases.ubuntu.com/ubuntu utopic-security main restricted universe multiverse   
  
deb-src http://old-releases.ubuntu.com/ubuntu utopic-updates main restricted universe multiverse   
  
deb-src http://old-releases.ubuntu.com/ubuntu utopic-proposed main restricted universe multiverse   
  
deb-src http://old-releases.ubuntu.com/ubuntu utopic-backports main restricted universe multiverse 
```

####第五步

需要注意的是上面地址中的 **utopic是ubuntu系统版本的名称**，我的ubuntu系统是14.10，对应的版本名称是utopic。只要把这里的utopic换车你自己系统版本的名称即可，如果不知道版本名称的话，可以运行以下命令获得：lsb_release -a，其中，Codename就是了。


####第六步

保存好源列表文件后，进入到终端，再输入以下命令：**sudo apt-get update **，看更换镜像源是否成功