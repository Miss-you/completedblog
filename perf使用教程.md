#perf使用教程

##Perf简介

Perf是Linux kernel中的系统性能优化工具，perf基本原理的话是在CPU的PMU register中Get/Set performance counters来获得诸如instructions executed，cache-missed suffered，branches mispredicted等信息。

> perf本身的工具有很多，这里主要介绍个人在查询程序性能问题时使用的一些工具
> 包括perf list、perf stat、perf record、perf report

##perf list

使用perf之前肯定要知道perf能监控哪些性能指标吧？那么就要使用perf list进行查看，通常使用的指标是cpu-clock/task-clock等，具体要根据需要来判断

```
 $ perf list 
 List of pre-defined events (to be used in -e): 
 cpu-cycles OR cycles [Hardware event] 
 instructions [Hardware event] 
…
 cpu-clock [Software event] 
 task-clock [Software event] 
 context-switches OR cs [Software event] 
…
 ext4:ext4_allocate_inode [Tracepoint event] 
 kmem:kmalloc [Tracepoint event] 
 module:module_load [Tracepoint event] 
 workqueue:workqueue_execution [Tracepoint event] 
 sched:sched_{wakeup,switch} [Tracepoint event] 
 syscalls:sys_{enter,exit}_epoll_wait [Tracepoint event] 
…
```
不同内核版本列出的结果不一样多...不过基本是够用的，但是无论多少，我们可以基本将其分为三类

1. **Hardware Event** 是由 PMU 硬件产生的事件，比如 cache 命中，当您需要了解程序对硬件特性的使用情况时，便需要对这些事件进行采样
2. **Software Event** 是内核软件产生的事件，比如进程切换，tick 数等 
3. **Tracepoint event** 是内核中的静态 tracepoint 所触发的事件，这些 tracepoint 用来判断程序运行期间内核的行为细节，比如 slab 分配器的分配次数等

具体监控哪个变量的话，譬如使用后面的perf report工具，则加**-e 监控指标**，如

```
perf report -e cpu-clock ls
监控运行ls命令时的cpu时钟占用监控
```

##perf stat

刚刚知道了可以监控哪些事件，但是事件这么多，该如何下手呢？

解决问题的时候有条理才解决的更快，所以面对一个性能问题的时候，最好采用自顶向下的策略。先整体看看该程序运行时各种统计事件的大概，再针对某些方向深入细节。而不要一下子扎进琐碎细节，会一叶障目的。

整体监测代码性能就需要使用**perf stat**这个工具，该工具主要是从全局上监控，可以看到程序导致性能瓶颈主要是什么原因。因为不同的程序导致其性能瓶颈的原因不同，譬如有些程序慢是由于计算量大，而有些程序是由于频繁的I/O导致性能瓶颈，他们的优化方式不同。**perf stat通过概括精简的方式提供被调试程序运行的整体情况和汇总数据。**

`使用方法`

```
perf stats 程序
譬如
perf stat ./gw --gtpu-ip 172.31.24.58 --sgw-s11-ip 172.31.24.250 --zmq-ip 172.31.31.174 --sgi-if eth1 --teid 1 --mysql 172.31.20.157 -cgw
```
程序运行完之后，然后使用**ctrl+c**来终止程序（若程序自动终止则不用），之后，perf便会打印出监控事件结果，类似结果如下：

```
Performance counter stats for './gw --gtpu-ip 172.31.24.58 --sgw-s11-ip 172.31.24.250 --zmq-ip 172.31.31.174 --sgi-if eth1 --teid 1 --mysql 172.31.20.157 -cgw':

       1773.651816 task-clock (msec)         #    0.016 CPUs utilized          
            79,054 context-switches          #    0.045 M/sec                  
               757 cpu-migrations            #    0.427 K/sec                  
            16,368 page-faults               #    0.009 M/sec                  
   <not supported> cycles                  
   <not supported> stalled-cycles-frontend
   <not supported> stalled-cycles-backend  
   <not supported> instructions            
   <not supported> branches                
   <not supported> branch-misses          

     109.795527410 seconds time elapsed
```

1. 1773.651816 task-clock是指程序运行期间占用了xx的任务时钟周期，该值高，说明程序的多数时间花费在 CPU 计算上而非 IO
2. 79,054 context-switches是指程序运行期间发生了xx次上下文切换，记录了程序运行过程中发生了多少次进程切换，频繁的进程切换是应该避免的。（有进程进程间频繁切换，或者内核态与用户态频繁切换）
3. 757 cpu-migrations 是指程序运行期间发生了xx次CPU迁移，即用户程序原本在一个CPU上运行，后来迁移到另一个CPU
4. 16,368 page-faults 是指程序发生了xx次页错误
5. 其他可以监控的譬如分支预测、cache命中等

##perf record

前面通过**perf stat**获得了程序性能瓶颈类型，之后，假设你已经知道哪个进程需要优化**（若不知道则需要使用perf top进行进一步监控，这里由于个人没有使用过，所以不作介绍）**，那么下一步就是对该进程进行细粒度的分析，分析在长长的程序代码中究竟是哪几段代码、哪几个函数需要修改呢？**这便需要使用 perf record 记录单个函数级别的统计信息，并使用 perf report 来显示统计结果。**

调优应该将注意力集中到百分比高的热点代码片段上，假如一段代码只占用整个程序运行时间的 0.1%，就算将其优化到仅剩一条机器指令，恐怕也只能将整体的程序性能提高 0.1%。

> 好钢用在刀刃上

仍以之前的gw程序为例，假设要监控的指标为cpu-clock

```
perf record -e cpu-clock -g ./gw --gtpu-ip 172.31.24.58 --sgw-s11-ip 172.31.24.250 --zmq-ip 172.31.31.174 --sgi-if eth1 --teid 1 --mysql 172.31.20.157 -cgw
```

1. **-g选项是告诉perf record额外记录函数的调用关系**，因为原本perf record记录大都是库函数，直接看库函数，大多数情况下，你的代码肯定没有标准库的性能好对吧？除非是针对产品进行特定优化，所以就需要知道是哪些函数频繁调用这些库函数，通过减少不必要的调用次数来提升性能
2. -e cpu-clock 指perf record监控的指标为cpu周期
3. 程序运行完之后，perf record会生成一个名为perf.data的文件（缺省值），如果之前已有，那么之前的perf.data文件会变为perf.data.old文件
4. 获得这个perf.data文件之后，我们其实还不能直接查看，下面就需要perf report工具进行查看

##perf report

前面通过`perf record`工具获得了某一进程的指标监控数据perf.data，下面就需要使用**perf report**工具查看该文件

使用方法

```
perf report -i perf-report生成的文件
譬如
perf report -i perf.data
```
上面使用`perf record`获得的数据的结果如下

```
+   4.93%  gw  libcurl-gnutls.so.4.3.0  [.] 0x000000000001e1e0                                                                                                                                
+   4.93%  gw  [kernel.kallsyms]        [k] eventfd_write                                                                                                                                     +   2.96%  gw  [kernel.kallsyms]        [k] ipt_do_table                                                                                                                                      +   2.46%  gw  [kernel.kallsyms]        [k] xen_hypercall_event_channel_op                                                                                                                    ?  1.97%  gw  libc-2.19.so             [.] _int_malloc                                                                                                                                       +   1.97%  gw  libc-2.19.so             [.] __clock_gettime                                                                                                                                   ?  1.97%  gw  gw                       [.] nwGtpv2cHandleInitialReq(NwGtpv2cStack*, unsigned int, MqPackage&)                                                                                ?  1.97%  gw  [kernel.kallsyms]        [k] pvclock_clocksource_read                                                                                                                          ?  1.97%  gw  [kernel.kallsyms]        [k] ip_finish_output                                                                                                                                  ?  1.97%  gw  [kernel.kallsyms]        [k] ixgbevf_xmit_frame                                                                                                                                
+   1.48%  gw  [kernel.kallsyms]        [k] kmem_cache_alloc_trace                                                                                                                            ?  1.48%  gw  [kernel.kallsyms]        [k] sk_run_filter      
```

1. [.]代表该调用属于用户态，若自己监控的进程为用户态进程，那么这些即主要为用户态的cpu-clock占用的数值，[k]代表属于内核态的调用。
2. 也许有的人会奇怪为什么自己完全是一个用户态的程序为什么还会统计到内核态的指标？**一是用户态程序运行时会受到内核态的影响，若内核态对用户态影响较大，统计内核态信息可以了解到是内核中的哪些行为导致对用户态产生影响；二则是有些用户态程序也需要依赖内核的某些操作，譬如I/O操作**
3.  /+   4.93%  gw  libcurl-gnutls.so.4.3.0  [.] 0x000000000001e1e0 ，左边的加号代表perf已经记录了该调用关系，按enter键可以查看调用关系，**不过由于这个是动态库里的函数，基本查看到的都是一些二进制数值：P**
4. perf监控gw进程结果记录到很多内核调用，说明gw进程在运行过程中，有可能被内核态任务频繁中断，应尽量避免这种情况，**对于这个问题我的解决办法是采用绑核，譬如机器有8个CPU，那么我就绑定内核操作、中断等主要在0-5CPU，GW由于有两个线程，分别绑定到6、7CPU上**

##实践

这里使用我在实验中程序在某一场景CPU占用率飙升的问题作为示例

###1.perf stat整体定位性能瓶颈

CPU飙升场景与正常场景使用perf stat对比差异

执行

```
perf stat ./gw --gtpu-ip 172.31.24.58 --sgw-s11-ip 172.31.24.250 --zmq-ip 172.31.31.174 --sgi-if eth1 --teid 1 --mysql 172.31.20.157 -cgw
```

`CPU飙升场景`

```
 Performance counter stats for './gw --gtpu-ip 172.31.24.58 --sgw-s11-ip 172.31.24.250 --zmq-ip 172.31.31.174 --sgi-if eth1 --teid 1 --mysql 172.31.20.157 -cgw':

       1773.651816 task-clock (msec)         #    0.016 CPUs utilized          
            79,054 context-switches          #    0.045 M/sec                  
               757 cpu-migrations            #    0.427 K/sec                  
            16,368 page-faults               #    0.009 M/sec                  
   <not supported> cycles                  
   <not supported> stalled-cycles-frontend
   <not supported> stalled-cycles-backend  
   <not supported> instructions            
   <not supported> branches                
   <not supported> branch-misses          

     109.795527410 seconds time elapsed
```     
`正常场景`
   
```     
 Performance counter stats for './gw --gtpu-ip 172.31.24.58 --sgw-s11-ip 172.31.24.250 --zmq-ip 172.31.31.174 --sgi-if eth1 --teid 1 --mysql 172.31.20.157 -cgw':

       1186.728996 task-clock (msec)         #    0.018 CPUs utilized          
            78,284 context-switches          #    0.066 M/sec                  
                69 cpu-migrations            #    0.058 K/sec                  
            16,368 page-faults               #    0.014 M/sec                  
   <not supported> cycles                  
   <not supported> stalled-cycles-frontend
   <not supported> stalled-cycles-backend  
   <not supported> instructions            
   <not supported> branches                
   <not supported> branch-misses          

      64.456003339 seconds time elapsed
```

通过对比可以发现：

1. task-clock异常场景比正常场景占用率高许多，说明程序CPU占用率提升
2. cpu-migrations异常场景比正常场景占用率高许多，说明进程发生了较频繁的从一个CPU迁移到另一个CPU


###2.perf record+perf report单点定位进程本身问题

通过perf stat整体上监控进程性能问题之后，使用perf record等对进程本身进行监控

执行

```
perf record -e task-clock -g ./gw --gtpu-ip 172.31.24.58 --sgw-s11-ip 172.31.24.250 --zmq-ip 172.31.31.174 --sgi-if eth1 --teid 1 --mysql 172.31.20.157 -cgw

perf report -i perf.data
```

结果如下

```
root@ip-172-31-24-250:/home/ubuntu/EPC/gw#

+   4.93%  gw  libcurl-gnutls.so.4.3.0  [.] 0x000000000001e1e0                                                                                                                                
+   4.93%  gw  [kernel.kallsyms]        [k] eventfd_write                                                                                                                                     +   2.96%  gw  [kernel.kallsyms]        [k] ipt_do_table                                                                                                                                      +   2.46%  gw  [kernel.kallsyms]        [k] xen_hypercall_event_channel_op                                                                                                                    ?  1.97%  gw  libc-2.19.so             [.] _int_malloc                                                                                                                                       +   1.97%  gw  libc-2.19.so             [.] __clock_gettime                                                                                                                                   ?  1.97%  gw  gw                       [.] nwGtpv2cHandleInitialReq(NwGtpv2cStack*, unsigned int, MqPackage&)                                                                                ?  1.97%  gw  [kernel.kallsyms]        [k] pvclock_clocksource_read                                                                                                                          ?  1.97%  gw  [kernel.kallsyms]        [k] ip_finish_output                                                                                                                                  ?  1.97%  gw  [kernel.kallsyms]        [k] ixgbevf_xmit_frame                                                                                                                                
+   1.48%  gw  [kernel.kallsyms]        [k] kmem_cache_alloc_trace                                                                                                                            ?  1.48%  gw  [kernel.kallsyms]        [k] sk_run_filter                                                                                                                                     ?  1.48%  gw  [kernel.kallsyms]        [k] fib_table_lookup                                                                                                                                  ?  1.48%  gw  [kernel.kallsyms]        [k] _raw_spin_unlock_irqrestore                                                                                                                       ?  0.99%  gw  libpthread-2.19.so       [.] __libc_fcntl                                                                                                                                      
+   0.99%  gw  libc-2.19.so             [.] vfprintf                                                                                                                                          ?  0.99%  gw  libc-2.19.so             [.] malloc                                                                                                                                            ?  0.99%  gw  libc-2.19.so             [.] free                                                                                                                                              ?  0.99%  gw  libc-2.19.so             [.] inet_ntop                                                                                                                                         ?  0.99%  gw  libc-2.19.so             [.] inet_pton                                                                                                                                        
+   0.99%  gw  [vdso]                   [.] 0x0000000000000ca1                                                                                                                                ?  0.99%  gw  [kernel.kallsyms]        [k] __do_softirq                                                                                                                                      ?  0.99%  gw  [kernel.kallsyms]        [k] ksize                                                                                                                                             ?  0.99%  gw  [kernel.kallsyms]        [k] kfree                                                                                                                                            
+   0.99%  gw  [kernel.kallsyms]        [k] fput                                                                                                                                              ?  0.99%  gw  [kernel.kallsyms]        [k] d_alloc_pseudo                                                                                                                                    ?  0.99%  gw  [kernel.kallsyms]        [k] sys_socket                                                                                                                                        ?  0.99%  gw  [kernel.kallsyms]        [k] datagram_poll                                                                                                                                     ?  0.99%  gw  [kernel.kallsyms]        [k] skb_network_protocol                                                                                                                              
+   0.99%  gw  [kernel.kallsyms]        [k] __dev_queue_xmit
```

1. libcurl-gnutls.so.4.3.0 它本身功能是一个数据上报，但是占用较高的CPU，说明调用该库存在问题（代码本身问题），**需要对调用该库的代码进行检查**
2. libc-2.19.so [.] _int_malloc 这是常用的malloc操作，由于对代码比较熟悉，在这个过程中不应该有这么多次申请内存，说明代码本身有问题，**需要对申请动态内存的代码进行检查**
3. __clock_gettime 这个是由于计时需要，需要频繁获取时间，通常是指 gettimeofday()函数
4. 整个统计显示有很多task-clock占用是由于**kernel.kallsyms**导致，同时也验证了对perf stat获得的数据的初步判断，即CPU飙升也与频繁的CPU迁移（内核态中断用户态操作）导致，解决办法是采用CPU绑核

###perf工具很好用，要善用这个利器