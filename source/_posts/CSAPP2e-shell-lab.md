---
title: 'CASPP2e:shell_lab'
date: 2018-04-08 15:10:35
tags: CSAPP
---

# 实验介绍

实现一个简单的Unix shell程序，熟悉进程的控制和信号的处理。

# 实验内容

对tsh.c中没有填写的函数进行填写，使得该shell能处理前后台运行程序，能够处理ctrl+z,ctrl+c等信号。需要实现的函数有如下：

``` 
1. eval()：该函数的主要功能是对用户输入的参数进行解析并运行计算。如果用户输入内建的命令行(quit,bg,fg,jobs)那么立即进行。否则，fork一个新的子进程并且将该任务在子进程的上下文中运行。如果该任务是前台任务，那么需要等到它运行结束才返回。 [70 lines]
2. builtin_cmd(): 该函数主要用来判断cmd是否是内建指令，如果是则立即执行，不是则返回 [25lines]
3. do_bgfg()：主要执行bg和fg指令功能 [50 lines]
4. waitfg()：实现等待前台程序运行结束 [20 lines]
5. sigchld_handler()：响应SIGCHLD [80 lines]
6. sigint_handler()：响应SIGINT（ctrl-c）信号 [15 lines]
7. sigtstp_handler()：响应SIGTSTP（ctrl-z）信号 [15 lines]
```

# 实验过程

## - 时刻查看帮助函数！

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

## - 总体思路

先通过CSAPP2e中P503给出的示例函数，实现初步的框架，然后根据`trace01-16`的数据逐步修改、完善自己的程序，是一个渐进增强的过程。

### Trace01-02

直接使用了P503的模板来实现`eval()`初步的函数，通过了`Trace01-02`的测试。

该函数的主要功能是对用户输入的参数进行解析并运行计算。如果用户输入内建的命令行（quit，bg，fg，jobs）那么立即执行。 
否则，fork一个新的子进程并且将该任务在子进程的上下文中运行。如果该任务是前台任务那么需要等到它运行结束才返回。

1. 注意每个子进程必须用户自己独一无二的进程组id，通过在fork()之后的子进程中Setpgid(0,0)实现，这样当我们向前台程序发送ctrl+c 或ctrl+z命令时，才不会影响到后台程序。如果没有这一步，则所有的子进程与当前的tsh shell进程为同一个进程组，发送信号时，前后台的子进程均会收到。
2. 在fork()新进程前后要阻塞SIGCHLD信号，防止出现竞争（race）这种经典的同步错误，如果不阻塞会出现子进程先结束从jobs中删除，然后再执行到主进程addjob的竞争问题。相关解释和方法见CSAPP P519页。

``` c
void eval(char *cmdline) 
{
    char *argv[MAXARGS]; /* Arguement list execve() */
    char buf[MAXLINE]; /* Holds modified command line */
    int bg;
    pid_t pid;
    sigset_t mask; /* race */

    strcpy(buf, cmdline);
    bg = parseline(buf, argv);
    if(argv[0] == NULL)
        return ; /* Ignore empty lines*/
    
    /* test &*/
    // printf("bg: %d\n", bg);

    if(!builtin_cmd(argv)) {
        sigemptyset(&mask);
        sigaddset(&mask, SIGCHLD);
        sigprocmask(SIG_BLOCK, &mask, NULL);

        if((pid = Fork()) == 0) {
            sigprocmask(SIG_UNBLOCK, &mask, NULL);
            setpgid(0, 0);
            if(execve(argv[0], argv, environ) < 0) {
                printf("%s: Command not found.\n", argv[0]);
                exit(0);
            }
        }

        addjob(jobs, pid, bg ? BG : FG, cmdline);
        sigprocmask(SIG_UNBLOCK, &mask, NULL);

        if(!bg) {
            waitfg(pid);
        } else {
            printf("[%d] (%d) %s", pid2jid(pid), pid, cmdline);
        }
    }
    return;
}
```

### Trace03-04

在用前面的程序运行`test04`时,产生了如下的结果：

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

``` c
void waitfg(pid_t pid)
{
    while(pid == fgpid(jobs)) {
        sleep(0);
    }
    return;
}
```

`waitfg`函数利用忙循环的形式实现，虽然课本中提到这样会对性能有一定的影响，但这是最简单、最明了的一种实现形式。

### Trace05

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
void sigchld_handler(int sig) 
{
    pid_t pid;

    while((pid = waitpid(-1, NULL, WNOHANG|WUNTRACED)) > 0) {
        if(verbose)
            printf("pid stopped = %d\n", pid);
        deletejob(jobs, pid);
    }
    return;
}
```

> `WNOHANG|WUNTRACED`: 立即返回，如果等待集合中没有任何子进程被停止或已终止，那么返回值为0，或者返回值等于那个被停止的或者已终止的子进程的PID。

### Trace06-07

查询了一下`kill`函数的用法：

 成功执行时，返回0；失败返回-1，errno被设为以下的某个值 

1. EINVAL：指定的信号码无效（参数 sig 不合法） EPERM；权限不够无法传送信号给指定进程 
2. ESRCH：参数 pid 所指定的进程或进程组不存在

然后直接`kill`掉当前在前台运行的程序，并把对应的job删掉即可。

``` c
void sigint_handler(int sig) 
{
    pid_t pid = fgpid(jobs);
    /* no fg job running */
    if(pid <= 0) 
        return;
    if(kill(-pid, SIGKILL) < 0) {
        unix_error("error when kill in sigint_handler");
        return;
    }
    printf("Job [%d] (%d) terminated by signal %d\n", pid2jid(pid), pid, sig);
    deletejob(jobs, pid);
    return;
}
```

（**这里的实现有问题，`kill`函数发出的信号不对，应该为`SIGINT`；**

**而且下面的`deletejob`函数会和`sigchld_handler`中的`deletejob`函数发生冲突，很可能会发生删除两次的情况，此时这里的输出会输出`Job [0] xxxxx`；**

**如果遇到这种问题，可能就是发生了这个问题，需仔细要检查一下信号的处理逻辑。**

**有两种解决方案，阻塞信号或者修改逻辑**）

### Trace08

到了08之后，修改了`sigtstp_handler`函数如下：

``` c
void sigtstp_handler(int sig) 
{
    pid_t pid = fgpid(jobs);
    if(pid <= 0)
        return;
    if(kill(-pid, SIGTSTP) < 0) {
        unix_error("error when kill in sigtstp_handler");
        return;
    }
    // printf("Job [%d] (%d) stopped by signal %d\n", pid2jid(pid), pid, sig);
    struct job_t *this_job = getjobpid(jobs, pid);
    this_job->state = ST;
    return;
}
```

但发现测试时，`jobs`命令并没有显示被挂起的进程，这和前面简陋的`sigchld_handler`函数有关，他会把每个终止的子进程删除掉而不考虑`job`的`status`，所以需要再进行修改。

通过`P496`给出的函数，对每个工作的状态进行判断,对每种不同的返回状态进行判断。

- WIFEXITED(status): 
  如果进程是正常返回即为true，通过调用exit()或者return返回
- WIFSIGNALED(status)： 
  如果进程因为捕获一个信号而终止的，则返回true
- WTERMSIG(status)： 
  当WIFSIGNALED(status)为真时，设置该值，返回导致当前状态的信号编号
- WIFSTOPPED(status)： 
  如果返回的进程当前是被停止，则为true
- WSTOPSIG(status)： 
  返回引起进程停止的信号

``` c
void sigchld_handler(int sig) 
{
    pid_t pid;
    int status;
    while((pid = waitpid(-1, &status, WNOHANG|WUNTRACED)) > 0) {
        if(WIFSTOPPED(status)) {
            printf("Job [%d] (%d) stopped by signal %d\n", pid2jid(pid), pid, WSTOPSIG(status));
            struct job_t *this_job = getjobpid(jobs, pid);
            if(this_job == NULL) {
                app_error("pid not exist");
                return ;
            }
            this_job -> state = ST;
        } else {
            if(WIFSIGNALED(status)) {
                printf("Job [%d] (%d) terminated by signal %d\n", pid2jid(pid), pid, WTERMSIG(status));
                deletejob(jobs, pid);
            } else {
                deletejob(jobs, pid);
            }
        }
    }
    return;
}
```

~~在这个函数仍然有瑕疵……之后会进行修改~~

### trace09/10 - 16

要注意判断`bg/fg`后面的参数是否合法(很烦),在输出顺序上有一些问题，如下面的`rtest`和`test`。

``` c
void do_bgfg(char **argv) 
{
    if(argv[1] == NULL) {
        printf("%s command requires PID or %%jobid argument\n", argv[0]);
        return;
    }
    int jid;
    pid_t pid;
    struct job_t *this_job = NULL; 
    if(*argv[1] == '%') {
        jid = atoi((char *)argv[1]+1);
        if(jid <= 0) {
            printf("%s command requires PID or %%jobid argument\n", argv[0]);
            return;
        }
        this_job = getjobjid(jobs, jid);
        if (this_job == NULL) {
            printf("%s: no such job\n", argv[1]);
            return;
        }
        pid = this_job->pid;
    } else {
        int pid = atoi((char *)argv[1]);
        if(pid <= 0) {
            printf("%s command requires PID or %%jobid argument\n", argv[0]);
            return;
        }
        this_job = getjobpid(jobs, pid);
        if (this_job == NULL) {
            printf("%s: no such job\n", argv[1]);
            return;
        }
        jid = this_job->jid;
    }

    if(!strcmp(argv[0], "bg")) {
        kill(-pid, SIGCONT);
        this_job->state = BG;
        printf("[%d] (%d) %s", jid, pid, (char *)this_job->cmdline);
    } else {
        if(!strcmp(argv[0], "fg")) {
            kill(-pid, SIGCONT);
            this_job->state = FG;
            waitfg(pid);
        }
    }

    return;
}
```

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

### 续

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

在信号的处理上有很大的问题，就是Trace06-07中下面写的

> **而且下面的`deletejob`函数会和`sigchld_handler`中的`deletejob`函数发生冲突，很可能会发生删除两次的情况，此时这里的输出会输出`Job [0] xxxxx`；如果遇到这种问题，可能就是发生了这个问题，需仔细要检查一下信号的处理逻辑。有两种解决方案，阻塞信号或者修改逻辑**）

更改了所有信号的处理逻辑，把全部的`deletejob`都放到`sigchld_handler`中处理，这样就不会出现重复删除的情况。

在此基础上又检查了一下，发现`trace14`里面是要区分`job`和`progress`的，做了修改。

# - 总结

总体而言是一个循序渐进的过程，通过对不同的输入和结果来对程序进行不断的修改，加深对整个流程的理解。

这里面的错误不容易出现，需要多次运行才能够发现，发现bug是一件很难的事情，当然debug也很难。