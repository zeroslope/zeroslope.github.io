---
title: 'CASPP2e:shell_lab'
date: 2018-04-08 15:10:35
tags:
---

``` c
/* 时刻观看帮助函数 */
void clearjob(struct job_t *job);
void initjobs(struct job_t *jobs);
int maxjid(struct job_t *jobs); 
int addjob(struct job_t *jobs, pid_t pid, int state, char *cmdline);
int deletejob(struct job_t *jobs, pid_t pid); 
pid_t fgpid(struct job_t *jobs);
struct job_t *getjobpid(struct job_t *jobs, pid_t pid);
struct job_t *getjobjid(struct job_t *jobs, int jid); 
int pid2jid(pid_t pid); 
void listjobs(struct job_t *jobs);
```

rate问题的解决？ P520

3-4的时候

``` shell
$ ./tsh
tsh> /bin/echo -e

tsh> ./myspin 1 &
[2] (26916) ./myspin 1 &
tsh> jobs
[1] (26913) Foreground /bin/echo -e
[2] (26916) Running ./myspin 1 &
tsh> quit
```

可以看到结束的进程并没有被回收，所以如果需要通过04，还需要处理子进程的`SIGCHLD`信号，根据handsout的内容实现`waitfg`函数。

> One of the tricky parts of the assignment is deciding on the allocation of work between the waitfg and sigchld handler functions. We recommend the following approach: 
> \- In waitfg,use a busy loop around the sleep function. 
> \- In sigchldhandler,use exactly one callto waitpid.
>
> While other solutions are possible, such as calling waitpid in both waitfg and sigchld handler, these can be very confusing. It is simpler to do all reaping in the handler.

### trace05

在执行trace05时，发现程序并不是在后台运行，而是执行结束后再输出下一个提示符，导致在输入`jobs`命令的时候，已经没有程序在运行,也就是程序并没有在后台运行。

``` c
void sigchld_handler(int sig) 
{
    pid_t pid;

    while((pid = waitpid(-1, NULL, 0)) > 0) {
        if(verbose)
            printf("pid stopped = %d\n", pid);
        deletejob(jobs, pid);
    }
    if(errno != ECHILD)
        unix_error("waitpid error");
    return;
}
```

经过艰难的debug,发现问题在SIGCHLD的处理上,这个处理是之前参照`P520`的例子所写的，为了终止掉以结束的进程，他会挂起调用进程的执行，直到等待集合中的一个进程变成已终止或者被停止。改变`waitpid`的第三个参数后，可以通过测试，虽然这样有很大的问题，等之后的例子再进行修改……

``` c

```

### trace06

查询了一下`kill`函数的用法：

 成功执行时，返回0；失败返回-1，errno被设为以下的某个值 

1. EINVAL：指定的信号码无效（参数 sig 不合法） EPERM；权限不够无法传送信号给指定进程 
2. ESRCH：参数 pid 所指定的进程或进程组不存在

然后直接`kill`掉当前在前台运行的程序，并把对应的job删掉即可。

### trace07

07和06似乎并没有多大差距。

### trace08

到了08之后，修改了`sigtstp_handler`函数如下：

``` c

```

但发现测试时，`jobs`命令并没有显示被挂起的进程，这和前面简陋的`sigchld_handler`函数有关，他会把每个终止的子进程删除掉而不考虑`job`的`status`，所以需要再进行修改。

### trace09/10

要注意判断`bg/fg`后面的参数是否合法,在输出顺序上有一些问题。 10 and 14

#### rtest

``` shell
./sdriver.pl -t trace10.txt -s ./tshref -a "-p"
#
# trace10.txt - Process fg builtin command. 
#
tsh> ./myspin 4 &
[1] (6900) ./myspin 4 &
tsh> fg %1
Job [1] (6900) stopped by signal 20
tsh> jobs
[1] (6900) Stopped ./myspin 4 &
tsh> fg %1
tsh> jobs
```

#### test

``` shell
./sdriver.pl -t trace10.txt -s ./tsh -a "-p"
#
# trace10.txt - Process fg builtin command. 
#
tsh> ./myspin 4 &
[1] (6877) ./myspin 4 &
tsh> fg %1
tsh> jobs
Job [1] (6877) stopped by signal 20
[1] (6877) Stopped ./myspin 4 &
tsh> fg %1
tsh> jobs
```

#### - trace16

本来觉得已经没问题了，打算放手不管了……

然而第二天帮别人调代码发现自己信号处理有问题，

比如，trace11的结果，对job的

``` shell
./sdriver.pl -t trace11.txt -s ./tsh -a "-p"
#
# trace11.txt - Forward SIGINT to every process in foreground process group
#
tsh> ./mysplit 4
Job [0] (11821) terminated by signal 2
tsh> /bin/ps a

./sdriver.pl -t trace11.txt -s ./tshref -a "-p"
#
# trace11.txt - Forward SIGINT to every process in foreground process group
#
tsh> ./mysplit 4
Job [1] (11841) terminated by signal 2
tsh> /bin/ps a
```

在此检查了一下，发现`trace14`里面是要区分`job`和`progress`的，做了修改