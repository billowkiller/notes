###信号
#### 产生信号的条件

1. 当用户按某些终端键，引发终端产生信号。`Ctrl+C`产生中断信号(`SIGINT`)。
2. 硬件异常产生信号：除数为0、无效内存引用等等。
3. 进程调用`kill(2)`函数可将信号发送给另外一个进程或进程组。接受和发送信号进程的所有者必须相同，或发送信号进程的所有者是超级用户。
4. 用户可用`kill(1)`命令将信号发送给其他进程
5. 当检测到某种软件条件已经发生，并应将其通知有关进程时产生信号。如`SIGURG`(在网络连接上传来带外数据时产生)、`SIGPIPE`（在管道的读进程已终止后，一个进程写此管道时产生）、以及`SIGALRM`(进程所设置的闹钟时钟超时时产生)。

进程必须告诉内核“在此信号出现时，请执行下列操作”：忽略、捕捉、执行系统默认动作。

####信号函数

最简单的接口是`signal`函数
	
	//成功返回信号以前的处理配置，若出错则返回SIG_ERR
	void (*signal(int signo, void(*func)(int)))(int);
	// for simplicity
	typedef void Sigfunc(int);
	Sigfunc *signal(int, Sigfunc *);

`func`的值是常量`SIG_IGN`、`SIG_DFL`或要调用的函数地址。

函数的限制：不改变信号的处理方式就不能确定信号的当前处理方式。经典代码：

	if(signal(SIGINT, SIG_IGN) != SIG_IGN)
		signal(SIGINT, sig_int);

- `exec`函数将原先设置为要捕捉的信号都更改为它们的默认动作，其他信号的状态不变。例如shell`后台进程对中断和退出信号的处理方式设置为忽略。
- 进程调用`fork`时，子进程继承父进程的信号处理方式。


	kill -USER1 7216 //向进程发送SIGUSR1信号
	kill 7216 //向进程发送SIGTERM信号

**中断的系统调用：**
早期Unix系统的一个特性，如果进程在执行一个低速系统调用而阻塞期间捕捉到一个信号，则该系统调用就被中断不再继续执行。
为了帮助应用程序使其不必处理被中断的系统调用，引入了某些被中断系统调用的自动重启动，包括：ioctl，read， readv， write， writev， wait和waitpid。并且允许进程基于每个信号禁用此功能。


**可重入函数：**
进程捕捉到信号并对其进行处理时，进程正在执行的指令序列就被信号处理程序临时中断，它首先执行该信号处理程序中的指令。而该指令可能破坏原进程中的数据信息。
不可重入函数的原因有：

1. 已知它们使用静态数据结构
2. 它们调用`malloc`或`free`。 //malloc通常为它所分配的存储区维护一个链接表。
3. 它们是标准的I/O函数。 //标准I/O库的很多实现都以不可重入方式使用全局数据结构。

并且每个线程只有一个`errno`变量，所以信号处理程序可能会改变其原先值。

**SIGCHLD语义：**

`僵死进程`的概念：凡是父进程没有调用`wait`函数获得子进程终止状态的子进程在终止之后都是僵死进程，这个概念的关键一点就是父进程是否调用了`wait`函数。

一般的,父进程在生成子进程之后会有两种情况,一种是父进程继续去做别的事情,另一种是父进程啥都不做,一直在k子进程退出.`SIGCHLD`信号就是为这第一种情况准备的,它让父进程去做别的事情,而只要父进程注册了处理该信号的函数,在子进程退出时就会调用该函数,在该函数中又可以调用`wait`得到终止的子进程的状态。处理信号的函数执行完后，再继续做父进程的事情.

`SIGCHLD`信号表示，子进程退出时父进程会收到一个k信号，默认的处理是忽略这个信号，而常规的做法是在这个信号处理函数中调用wait函数获取子进程的退出状态。如果`SIGCHLD`被忽略，会产生僵死进程。如果要避免僵死进程，则必须等待子进程。可以调用`signal`或`sigset`将`SIGCHLD`的配置设置为忽略，则绝不会产生僵死进程。使用`sigaction`可设置`SA_NOCLDWAIT`标准避免子进程僵死。

**几个函数：**

- kill函数将信号发送给进程或进程组。raise函数允许进程向自身发送信号。
- alarm()用来设置信号SIGALRM在经过参数seconds指定的秒数后传送给目前的进程。如果参数seconds为0，则之前设置的闹钟会被取消，并将剩下的时间返回。 返回之前闹钟的剩余秒数，如果之前未设闹钟则返回0。
- pause函数使调用进程挂起直到捕捉到一个信号。
- 信号集数据结构sigset_t， sigfillset将信号集各位设置为1，并且返回0。宏定义`#define sigfillset(ptr) (*(ptr)=~(sigset_t)0, 0)`，用逗号运算符作为表达式的返回值。
- sigprocmask函数规定当前阻塞而不能递送给该进程的信号集。
- sigpending函数返回信号集。
- sigaction函数的功能是检查或修改与指定信号相关联的处理动作。此函数取代signal函数。信号处理程序被调用时，操作系统建立的心心好屏蔽字包括正被递送的信号。因此信号多次发生时，会被阻塞到对钱一个信号的处理结束为止。

### 线程与信号 ###

每个线程都有自己的信号屏蔽字，但是幸好的处理是进程中所有线程共享的。这意味着尽管每个线程可以阻止某些信号，但当线程修改了与某个信号相关的处理行为之后，所有的线程都必须共享这个处理行为的改变。

进程中的信号是传递送到单个线程的。如果信号与硬件故障或计时器超时相关，该信号就被发送到引起改事件的线程中去，而其他的信号则被发送到任意一个线程。
