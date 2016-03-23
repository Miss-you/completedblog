前一段时间因为开题的事情一直耽搁了我搞Linux的进度，搞的我之前学的东西都遗忘了，很烦躁的说，现在抽个时间把之前所学的做个小节。文章内容主要总结于《Linux程序设计第3版》。
1.Linux进程与线程
       Linux进程创建一个新线程时，线程将拥有自己的栈（因为线程有自己的局部变量），但与它的创建者共享全局变量、文件描述符、信号句柄和当前目录状态。
Linux通过fork创建子进程与创建线程之间是有区别的：fork创建出该进程的一份拷贝，这个新进程拥有自己的变量和自己的PID，它的时间调度是独立的，它的执行几乎完全独立于父进程。
进程可以看成一个资源的基本单位，而线程是程序调度的基本单位，一个进程内部的线程之间共享进程获得的时间片。
 
2._REENTRANT宏
       在一个多线程程序里，默认情况下，只有一个errno变量供所有的线程共享。在一个线程准备获取刚才的错误代码时，该变量很容易被另一个线程中的函数调用所改变。类似的问题还存在于fputs之类的函数中，这些函数通常用一个单独的全局性区域来缓存输出数据。
       为解决这个问题，需要使用可重入的例程。可重入代码可以被多次调用而仍然工作正常。编写的多线程程序，通过定义宏_REENTRANT来告诉编译器我们需要可重入功能，这个宏的定义必须出现于程序中的任何#include语句之前。
       _REENTRANT为我们做三件事情，并且做的非常优雅：
（1）它会对部分函数重新定义它们的可安全重入的版本，这些函数名字一般不会发生改变，只是会在函数名后面添加_r字符串，如函数名gethostbyname变成gethostbyname_r。
（2）stdio.h中原来以宏的形式实现的一些函数将变成可安全重入函数。
（3）在error.h中定义的变量error现在将成为一个函数调用，它能够以一种安全的多线程方式来获取真正的errno的值。
 
3.线程的基本函数
大多数pthread_XXX系列的函数在失败时，并未遵循UNIX函数的惯例返回-1，这种情况在UNIX函数中属于一少部分。如果调用成功，则返回值是0，如果失败则返回错误代码。
 
1.线程创建：
#include <pthread.h>
int pthread_create(pthread_t *thread, pthread_attr_t *attr, void *(*start_routine)(void *), void *arg);
参数说明：
thread：指向pthread_create类型的指针，用于引用新创建的线程。
attr：用于设置线程的属性，一般不需要特殊的属性，所以可以简单地设置为NULL。
*(*start_routine)(void *)：传递新线程所要执行的函数地址。
arg：新线程所要执行的函数的参数。
调用如果成功，则返回值是0，如果失败则返回错误代码。
 
2.线程终止
#include <pthread.h>
void pthread_exit(void *retval);
参数说明：
retval：返回指针，指向线程向要返回的某个对象。
线程通过调用pthread_exit函数终止执行，并返回一个指向某对象的指针。注意：绝不能用它返回一个指向局部变量的指针，因为线程调用该函数后，这个局部变量就不存在了，这将引起严重的程序漏洞。
 
3.线程同步
#include <pthread.h>
int pthread_join(pthread_t th, void **thread_return);
参数说明：
th：将要等待的张璐，线程通过pthread_create返回的标识符来指定。
thread_return：一个指针，指向另一个指针，而后者指向线程的返回值。
 
一个简单的多线程Demo（thread1.c）：
[cpp] view plaincopy
#include <pthread.h>  
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
  
void *thread_function(void *arg);  
  
char message[] = "Hello World";  
  
int main()  
{  
    int res;  
    pthread_t a_thread;  
    void *thread_result;  
  
    res = pthread_create(&a_thread, NULL, thread_function, (void *)message);  
    if (res != 0)  
    {  
        perror("Thread creation failed!");  
        exit(EXIT_FAILURE);  
    }  
  
    printf("Waiting for thread to finish.../n");  
      
    res = pthread_join(a_thread, &thread_result);  
    if (res != 0)  
    {  
        perror("Thread join failed!/n");  
        exit(EXIT_FAILURE);  
    }  
  
    printf("Thread joined, it returned %s/n", (char *)thread_result);  
    printf("Message is now %s/n", message);  
  
    exit(EXIT_FAILURE);  
}  
  
void *thread_function(void *arg)  
{  
    printf("thread_function is running. Argument was %s/n", (char *)arg);  
    sleep(3);  
    strcpy(message, "Bye!");  
    pthread_exit("Thank you for your CPU time!");  
}  
 
编译这个程序时，需要定义宏_REENTRANT：
gcc -D_REENTRANT thread1.c -o thread1 –lpthread
 
运行这个程序：
$ ./thread1输出：
thread_function is running. Argument was Hello World
Waiting for thread to finish...
Thread joined, it returned Thank you for your CPU time!
Message is now Bye!
 
这个例子值得我们去花时间理解，因为它将作为几个例子的基础。
 
pthread_exit(void *retval)本身返回的就是指向某个对象的指针，因此，pthread_join(pthread_t th, void **thread_return);中的thread_return是二级指针，指向线程返回值的指针。
可以看到，我们创建的新线程修改的数组message的值，而原先的线程也可以访问该数组。如果我们调用的是fork而不是pthread_create，就不会有这样的效果了。原因是fork创建子进程之后，子进程会拷贝父进程，两者分离，相互不干扰，而线程之间则是共享进程的相关资源。
 
4.线程的同时执行
接下来，我们来编写一个程序，以验证两个线程的执行是同时进行的。当然，如果是在一个单处理器系统上，线程的同时执行就需要靠CPU在线程之间的快速切换来实现了。
我们的程序需要利用一个原理：即除了局部变量外，所有其他的变量在一个进程中的所有线程之间是共享的。
在这个程序中，我们是在两个线程之间使用轮询技术，这种方式称为忙等待，所以它的效率会很低。在本文的后续部分，我们将介绍一种更好的解决办法。
 
       下面的代码中，两个线程会不断的轮询判断flag的值是否满足各自的要求。
[cpp] view plaincopy
#include <pthread.h>  
#include <stdio.h>  
#include <stdlib.h>  
  
int flag = 1;  
  
void *thread_function(void *arg);  
  
int main()  
{  
    int res;  
    pthread_t a_thread;  
    void *thread_result;  
    int count = 1;  
  
    res = pthread_create(&a_thread, NULL, thread_function, NULL);  
    if (res != 0)  
    {  
        perror("Thread creation failed");  
        exit(EXIT_FAILURE);  
    }  
  
    while (count++ <= 20)  
    {  
        if (flag == 1)  
        {  
            printf ("1");  
            flag = 2;  
        }  
        else  
        {  
            sleep(1);  
        }  
    }  
  
    printf("/nWaiting for thread to finish.../n");  
    res = pthread_join(a_thread, &thread_result);  
    if (res != 0)  
    {  
        perror("Thread join failed");  
        exit(EXIT_FAILURE);  
    }  
  
    exit(EXIT_SUCCESS);  
}  
  
void *thread_function(void *arg)  
{  
    int count = 1;  
  
    while (count++ <= 20)  
    {  
        if (flag == 2)  
        {  
            printf("2");  
            flag = 1;  
        }  
        else  
        {  
            sleep(1);  
        }  
    }  
}  
 
编译这个程序：
gcc -D_REENTRANT thread2.c -o thread2 –lpthread
 
运行这个程序：
$ ./thread2
121212121212121212
Waiting for thread to finish...
 
5.线程的同步
在上述示例中，我们采用轮询的方式在两个线程之间不停地切换是非常笨拙且没有效率的实现方式，幸运的是，专门有一级设计好的函数为我们提供更好的控制线程执行和访问代码临界区的方法。
本小节将介绍两个线程同步的基本方法：信号量和互斥量。这两种方法很相似，事实上，它们可以互相通过对方来实现。但在实际的应用中，对于一些情况，可能使用信号量或互斥量中的一个更符合问题的语义，并且效果更好。
 
5.1用信号量进行同步
1.信号量创建
#include <semaphore.h>
int sem_init(sem_t *sem, int pshared, unsigned int value);
参数说明：
sem：信号量对象。
pshared：控制信号量的类型，0表示这个信号量是当前进程的局部信号量，否则，这个信号量就可以在多个进程之间共享。
value：信号量的初始值。
 
2.信号量控制
#include <semaphore.h>
int sem_wait(sem_t *sem);
int sem_post(sem_t *sem);
 
sem_post的作用是以原子操作的方式给信号量的值加1。
       sem_wait的作用是以原子操作的方式给信号量的值减1，但它会等到信号量非0时才会开始减法操作。如果对值为0的信号量调用sem_wait，这个函数就会等待，直到有线程增加了该信号量的值使其不再为0。
 
3.信号量销毁
#include <semaphore.h>
int sem_destory(sem_t *sem);
这个函数的作用是，用完信号量后对它进行清理，清理该信号量所拥有的资源。如果你试图清理的信号量正被一些线程等待，就会收到一个错误。
与大多数Linux函数一样，这些函数在成功时都返回0。
 
下面编码实现输入字符串，统计每行的字符个数，以“end”结束输入：
[cpp] view plaincopy
#include <pthread.h>  
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
#include <semaphore.h>  
  
#define SIZE 1024  
  
void *thread_function(void *arg);  
  
char buffer[SIZE];  
sem_t sem;  
  
int main()  
{  
    int res;  
    pthread_t a_thread;  
    void *thread_result;  
  
    res = sem_init(&sem, 0, 0);  
    if (res != 0)  
    {  
        perror("Sem init failed");  
        exit(EXIT_FAILURE);  
    }  
  
    res = pthread_create(&a_thread, NULL, thread_function, NULL);  
    if (res != 0)  
    {  
        perror("Thread create failed");  
        exit(EXIT_FAILURE);  
    }  
  
    printf("Input some text. Enter 'end' to finish/n");  
  
    while (scanf("%s", buffer))  
    {  
        sem_post(&sem);  
        if (strncmp("end", buffer, 3) == 0)  
            break;  
    }  
  
    printf ("/nWaiting for thread to finish.../n");  
  
    res = pthread_join(a_thread, &thread_result);  
    if (res != 0)  
    {  
        perror("Thread join failed");  
        exit(EXIT_FAILURE);  
    }  
  
    printf ("Thread join/n");  
  
    sem_destroy(&sem);  
  
    exit(EXIT_SUCCESS);  
}  
  
void *thread_function(void *arg)  
{  
    sem_wait(&sem);  
    while (strncmp("end", buffer, 3) != 0)  
    {  
        printf("You input %d characters/n", strlen(buffer));  
        sem_wait(&sem);  
    }  
    pthread_exit(NULL);  
}  
 
编译这个程序：
gcc -D_REENTRANT thread2.c -o thread2 –lpthread
 
运行这个程序：
$ ./thread3
Input some text. Enter 'end' to finish
123
You input 3 characters
1234
You input 4 characters
12345
You input 5 characters
end
 
Waiting for thread to finish…
Thread join
 
       通过使用信号量，我们阻塞了统计字符个数的线程，这个程序似乎对快速的文本输入和悠闲的暂停都很适用，比之前的轮询解决方案效率上有了本质的提高。
 
5.2用互斥量进行线程同步
       另一种用在多线程程序中同步访问的方法是使用互斥量。它允许程序员锁住某个对象，使得每次只能有一个线程访问它。为了控制对关键代码的访问，必须在进入这段代码之前锁住一个互斥量，然后在完成操作之后解锁它。
       用于互斥量的基本函数和用于信号量的函数非常相似：
#include <pthread.h>
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t, *mutexattr);
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
int pthread_mutex_destory(pthread_mutex_t *mutex);
与其他函数一样，成功时返回0，失败时将返回错误代码，但这些函数并不设置errno，所以必须对函数的返回代码进行检查。互斥量的属性设置这里不讨论，因此设置成NULL。
我们用互斥量来重写刚才的代码如下：
[cpp] view plaincopy
#include <pthread.h>  
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
#include <semaphore.h>  
  
#define SIZE 1024  
char buffer[SIZE];  
  
void *thread_function(void *arg);  
pthread_mutex_t mutex;  
  
int main()  
{  
    int res;  
    pthread_t a_thread;  
    void *thread_result;  
  
    res = pthread_mutex_init(&mutex, NULL);  
    if (res != 0)  
    {  
        perror("Mutex init failed!");  
        exit(EXIT_FAILURE);  
    }  
  
    res = pthread_create(&a_thread, NULL, thread_function, NULL);  
    if (res != 0)  
    {  
        perror("Thread create failed!");  
        exit(EXIT_FAILURE);  
    }  
  
    printf("Input some text. Enter 'end' to finish/n");  
  
    while (1)  
    {  
        pthread_mutex_lock(&mutex);  
        scanf("%s", buffer);  
        pthread_mutex_unlock(&mutex);  
        if (strncmp("end", buffer, 3) == 0)  
            break;  
        sleep(1);  
    }  
  
    res = pthread_join(a_thread, &thread_result);  
    if (res != 0)  
    {  
        perror("Thread join failed!");  
        exit(EXIT_FAILURE);  
    }  
  
    printf("Thread joined/n");  
  
    pthread_mutex_destroy(&mutex);  
  
    exit(EXIT_SUCCESS);  
}  
  
void *thread_function(void *arg)  
{  
    sleep(1);  
  
    while (1)  
    {  
        pthread_mutex_lock(&mutex);  
        printf("You input %d characters/n", strlen(buffer));  
        pthread_mutex_unlock(&mutex);  
        if (strncmp("end", buffer, 3) == 0)  
            break;  
        sleep(1);  
    }  
}  
 
编译这个程序：
gcc -D_REENTRANT thread4.c -o thread4 –lpthread
 
运行这个程序：
$ ./thread4
Input some text. Enter 'end' to finish
123
You input 3 characters
1234
You input 4 characters
12345
You input 5 characters
end
You input 3 characters
Thread joined
6.线程的属性
之前我们并未谈及到线程的属性，可以控制的线程属性是非常多的，这里面只列举一些常用的。
如在前面的示例中，我们都使用的pthread_join同步线程，但其实有些情况下，我们并不需要。如：主线程为服务线程，而第二个线程为数据备份线程，备份工作完成之后，第二个线程可以直接终止了，它没有必要再返回到主线程中。因此，我们可以创建一个“脱离线程”。
 
下面介绍几个常用的函数：
(1)int pthread_attr_init (pthread_attr_t* attr);
功能：对线程属性变量的初始化。
attr：线程属性。
函数返回值：成功：0，失败：-1
 
(2) int pthread_attr_setscope (pthread_attr_t* attr, int scope);
功能：设置线程绑定属性。
attr：线程属性。
scope：PTHREAD_SCOPE_SYSTEM(绑定)；PTHREAD_SCOPE_PROCESS(非绑定)
函数返回值：成功：0，失败：-1
 
(3) int pthread_attr_setdetachstate (pthread_attr_t* attr, int detachstate);
功能：设置线程分离属性。
attr：线程属性。
detachstate：PTHREAD_CREATE_DETACHED(分离)；PTHREAD_CREATE_JOINABLE（非分离）
函数返回值：成功：0，失败：-1
 
(4) int pthread_attr_setschedpolicy(pthread_attr_t* attr, int policy);
功能：设置创建线程的调度策略。
attr：线程属性；
policy：线程调度策略：SCHED_FIFO、SCHED_RR和SCHED_OTHER。
函数返回值：成功：0，失败：-1
 
(5) int pthread_attr_setschedparam (pthread_attr_t* attr, struct sched_param* param);
功能：设置线程优先级。
attr：线程属性。
param：线程优先级。
函数返回值：成功：0，失败：-1
 
(6) int pthread_attr_destroy (pthread_attr_t* attr);
功能：对线程属性变量的销毁。
attr：线程属性。
函数返回值：成功：0，失败：-1
 
(7)其他
int pthread_attr_setguardsize(pthread_attr_t* attr,size_t guardsize);//设置新创建线程栈的保护区大小。
int pthread_attr_setinheritsched(pthread_attr_t* attr, int inheritsched);//决定怎样设置新创建线程的调度属性。
int pthread_attr_setstack(pthread_attr_t* attr, void* stackader,size_t stacksize);//两者共同决定了线程栈的基地址以及堆栈的最小尺寸（以字节为单位）。
int pthread_attr_setstackaddr(pthread_attr_t* attr, void* stackader);//决定了新创建线程的栈的基地址。
int pthread_attr_setstacksize(pthread_attr_t* attr, size_t stacksize);//决定了新创建线程的栈的最小尺寸（以字节为单位）。
例：创建优先级为10的线程。
pthread_attr_t attr;
struct sched_param param;
pthread_attr_init(&attr);
pthread_attr_setscope (&attr, PTHREAD_SCOPE_SYSTEM); //绑定
pthread_attr_setdetachstate (&attr, PTHREAD_CREATE_DETACHED); //分离
pthread_attr_setschedpolicy(&attr, SCHED_RR);
param.sched_priority = 10;
pthread_attr_setschedparam(&attr, &param);
pthread_create(xxx, &attr, xxx, xxx);
pthread_attr_destroy(&attr);
 
下面实现一个脱离线程的程序，创建一个线程，其属性设置为脱离状态。子线程结束时，要使用pthread_exit，原来的主线程不再等待与子线程重新合并。代码如下：
[cpp] view plaincopy
#include <stdio.h>  
#include <unistd.h>  
#include <stdlib.h>  
#include <pthread.h>  
  
void *thread_function(void *arg);  
  
char message[] = "Hello World";  
int thread_finished = 0;  
  
int main()  
{  
    int res;  
    pthread_t a_thread;  
    pthread_attr_t thread_attr;  
      
    res = pthread_attr_init(&thread_attr);  
    if (res != 0)  
    {  
        perror("Attribute creation failed");  
        exit(EXIT_FAILURE);  
    }  
      
    res = pthread_attr_setdetachstate(&thread_attr, PTHREAD_CREATE_DETACHED);  
    if (res != 0)  
    {  
        perror("Setting detached attribute failed");  
        exit(EXIT_FAILURE);  
    }  
      
    res = pthread_create(&a_thread, &thread_attr, thread_function, (void*)message);  
    if (res != 0)  
    {  
        perror("Thread creation failed");  
        exit(EXIT_FAILURE);  
    }  
      
    pthread_attr_destroy(&thread_attr);  
    while(!thread_finished)  
    {  
        printf("Waiting for thread to say it's finished.../n");  
        sleep(1);  
    }  
      
    printf("Other thread finished, bye!/n");  
    exit(EXIT_SUCCESS);  
}  
  
void *thread_function(void *arg)  
{  
    printf("thread_function is running. Argument was %s/n", (char *)arg);  
    sleep(4);  
    printf("Second thread setting finished flag, and exiting now/n");  
    thread_finished = 1;  
    pthread_exit(NULL);  
}  
 
编译这个程序：
gcc -D_REENTRANT thread5.c -o thread5 –lpthread
 
运行这个程序：
$ ./thread5
thread_function is running. Argument: hello world!
Waiting for thread to finished...
Waiting for thread to finished...
Waiting for thread to finished...
Waiting for thread to finished...
Second thread setting finished flag, and exiting now
Other thread finished!
 
通过设置线程的属性，我们还可以控制线程的调试，其方式与设置脱离状态是一样的。
 
7.取消一个线程
有时，我们想让一个线程可以要求另一个线程终止，线程有方法做到这一点，与信号处理一样，线程可以在被要求终止时改变其行为。
先来看用于请求一个线程终止的函数：
#include <pthread.h>
int pthread_cancel(pthread_t thread);
这个函数简单易懂，提供一个线程标识符，我们就可以发送请求来取消它。
 
线程可以用pthread_setcancelstate设置自己的取消状态。
#include <pthread.h>
int pthread_setcancelstate(int state, int *oldstate);
参数说明：
state：可以是PTHREAD_CANCEL_ENABLE允许线程接收取消请求，也可以是PTHREAD_CANCEL_DISABLE忽略取消请求。
oldstate：获取先前的取消状态。如果对它没兴趣，可以简单地设置为NULL。如果取消请求被接受了，线程可以进入第二个控制层次，用pthread_setcanceltype设置取消类型。
 
#include <pthread.h>
int pthread_setcanceltype(int type, int *oldtype);
参数说明：
type：可以取PTHREAD_CANCEL_ASYNCHRONOUS，它将使得在接收到取消请求后立即采取行动；另一个是PTHREAD_CANCEL_DEFERRED，它将使得在接收到取消请求后，一直等待直到线程执行了下述函数之一后才采取行动：pthread_join、pthread_cond_wait、pthread_cond_timedwait、pthread_testcancel、sem_wait或sigwait。
oldtype：允许保存先前的状态，如果不想知道先前的状态，可以传递NULL。
默认情况下，线程在启动时的取消状态为PTHREAD_CANCEL_ENABLE，取消类型是PTHREAD_CANCEL_DEFERRED。
 
下面编写代码thread6.c，主线程向它创建的线程发送一个取消请求。
[cpp] view plaincopy
#include <pthread.h>  
#include <stdio.h>  
#include <stdlib.h>  
  
void *thread_function(void *arg);  
  
int main()  
{  
    int res;  
    pthread_t a_thread;  
    void *thread_result;  
  
    res = pthread_create(&a_thread, NULL, thread_function, NULL);  
    if (res != 0)  
    {  
        perror("Thread create failed!");  
        exit(EXIT_FAILURE);  
    }  
  
    sleep(4);  
    printf("Canceling thread.../n");  
  
    res = pthread_cancel(a_thread);  
    if (res != 0)  
    {  
        perror("Thread cancel failed!");  
        exit(EXIT_FAILURE);  
    }  
  
    printf ("Waiting for thread to finished.../n");  
      
    res = pthread_join(a_thread, &thread_result);  
    if (res != 0)  
    {  
        perror ("Thread join failed!");  
        exit(EXIT_FAILURE);  
    }  
  
    printf("Thread canceled!");  
  
    exit(EXIT_FAILURE);  
}  
  
void *thread_function(void *arg)  
{  
    int i;  
    int res;  
  
    res = pthread_setcancelstate(PTHREAD_CANCEL_ENABLE, NULL);  
    if (res != 0)  
    {  
        perror("Thread setcancelstate failed!");  
        exit(EXIT_FAILURE);  
    }  
  
    res = pthread_setcanceltype(PTHREAD_CANCEL_DEFERRED, NULL);  
    if (res != 0)  
    {  
        perror("Thread setcanceltype failed!");  
        exit(EXIT_FAILURE);  
    }  
  
    printf("thread_function is running.../n");  
  
    for (i = 0; i < 10; i++)  
    {  
        printf("Thread is still running (%d).../n", i);  
        sleep(1);  
    }  
    pthread_exit(0);  
}  
 
编译这个程序：
gcc -D_REENTRANT thread6.c -o thread6 –lpthread
 
运行这个程序：
$ ./thread6
thread_function is running...
Thread is still running (0)...
Thread is still running (1)...
Thread is still running (2)...
Thread is still running (3)...
Canceling thread...
Waiting for thread to finished...
 
8.多线程
之前，我们所编写的代码里面都仅仅是创建了一个线程，现在我们来演示一下如何创建一个多线程的程序。
[cpp] view plaincopy
#include <pthread.h>  
#include <stdio.h>  
#include <stdlib.h>  
  
#define NUM 6  
  
void *thread_function(void *arg);  
  
int main()  
{  
    int res;  
    pthread_t a_thread[NUM];  
    void *thread_result;  
    int index;  
  
    for (index = 0; index < NUM; ++index) {  
        res = pthread_create(&a_thread[index], NULL, thread_function, (void *)index);  
        if (res != 0)  
        {  
            perror("Thread create failed!");  
            exit(EXIT_FAILURE);  
        }  
        sleep(1);  
    }  
  
    printf("Waiting for threads to finished.../n");  
  
    for (index = NUM - 1; index >= 0; --index)  
    {  
        res = pthread_join(a_thread[index], &thread_result);  
        if (res == 0)  
        {  
            printf("Picked up a thread:%d/n", index + 1);  
        }  
        else  
        {  
            perror("pthread_join failed/n");  
        }  
    }  
  
    printf("All done/n");  
      
    exit(EXIT_SUCCESS);  
}  
  
void *thread_function(void *arg)  
{  
    int my_number = (int)arg;  
    int rand_num;  
  
    printf("thread_function is running. Argument was %d/n", my_number);  
    rand_num = 1 + (int)(9.0 * rand()/(RAND_MAX + 1.0));  
    sleep(rand_num);  
    printf("Bye from %d/n", my_number);  
    pthread_exit(NULL);  
}  
编译这个程序：
gcc -D_REENTRANT thread7.c -o thread7 –lpthread
 
运行这个程序：
$ ./thread7
thread_function is running. Argument was 0
thread_function is running. Argument was 1
thread_function is running. Argument was 2
thread_function is running. Argument was 3
thread_function is running. Argument was 4
Bye from 1
thread_function is running. Argument was 5
Waiting for threads to finished...
Bye from 5
Picked up a thread:6
Bye from 0
Bye from 2
Bye from 3
Bye from 4
Picked up a thread:5
Picked up a thread:4
Picked up a thread:3
Picked up a thread:2
Picked up a thread:1
All done
 
9.小结
本文主要介绍了Linux环境下的多线程编程，介绍了信号量和互斥量、线程属性控制、线程同步、线程终止、取消线程及多线程并发。
本文比较简单，只作为初学Linux多线程编程入门之用。