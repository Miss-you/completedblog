#查看linux函数栈

###废话
函数运行在栈中，函数栈有大小，函数调用深度有限制，`所以不要随便在多次重复递归调用的函数里面加超大的数组变量！`

###查看方法

ulimit -a

```
feiqianyousadeMacBook-Pro:~ yousa$ ulimit -a
core file size          (blocks, -c) 0
core文件大小
data seg size           (kbytes, -d) unlimited
file size               (blocks, -f) unlimited
max locked memory       (kbytes, -l) unlimited
max memory size         (kbytes, -m) unlimited
open files                      (-n) 256
pipe size            (512 bytes, -p) 1
stack size              (kbytes, -s) 8192
函数栈大小，8M，注意是8M！！！
cpu time               (seconds, -t) unlimited
max user processes              (-u) 709
最大用户进程数
virtual memory          (kbytes, -v) unlimited
虚拟内存大小
```

其他还不太懂

直接查看函数栈大小
`ulimit -s`

```
feiqianyousadeMacBook-Pro:~ yousa$ ulimit -s
8192
```

修改函数栈大小至10000 KB
`ulimit -s 10000`

###其他

修改其他一些项就可以按照`ulimit -a`列出来的那样，修改即可

比如修改core file大小，`ulimit -a 1000`

**core file设置之后，程序core dump后就会产生相应core文件，那么便可以使用gdb工具查看程序core dump原因，基本上大部分都可以追踪定位查看（除非：1、开了-o2以上优化。2、调用动态库，挂在动态库里面）**
