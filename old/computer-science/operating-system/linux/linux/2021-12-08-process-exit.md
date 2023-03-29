
#### 示例

- `demo_c/demo_linux_c/wait/wait.c`，通过`wait()`函数回收子进程。
- `demo_c/demo_linux_c/waitpid/waitpid_status.c`，通过`waitpid()`函数回收正常退出的子进程。
- `demo_c/demo_linux_c/waitpid/waitpid_signal.c`，通过`waitpid()`函数回收被信号终止的子进程。
- `demo_c/demo_linux_c/waitpid/waitpid_signal_handler.c`，信号处理函数。

程序运行后，在另外一个终端上，通过`kill -s SIGUSR1 {pid}`命令向子进程发送SIGUSR1信号。可以通过`kill -l`命令查看所有的信号。

测试信号处理函数时，当测试的是信号是SIGSTOP时，并不会像预期的那样输出SIGSTOP信号的值。因为进程收到SIGSTOP后已经停止作业（并没有退出）。如果这时再向进程发送SIGCONT信号。这时进程会恢复作业，并输出`signal no=18`，也就是收到的SIGCONT信号的值。

通过strace命令检测，可以得到下面的内容。进程确实收到了SIGSTOP信号和SIGCONT信号。

```
--- SIGSTOP {si_signo=SIGSTOP, si_code=SI_USER, si_pid=84, si_uid=0} ---
--- stopped by SIGSTOP ---
--- SIGCONT {si_signo=SIGCONT, si_code=SI_USER, si_pid=84, si_uid=0} ---
write(1, "signal no=18\r\n", 14)        = 14
```
