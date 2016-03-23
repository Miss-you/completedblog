#sudo: unable to resolve host ubuntu

> 环境：ubuntu

我用的是ubuntu,修改了计算机的名字，当运行sudo ...之后出现如下提示：

```
sudo: unable to resolve host ubuntu
```

提示不能解析主机ubuntu，在/etc/hosts中存放了网址的解析，计算机上网时，先访问这个文件。所以修改/etc/hosts文件

**解决办法**

```
sudo vim /etc/hosts
```
添加如下：

```
127.0.0.1 ubuntu  #ubuntu是主机名。
```
保存之后，解决！