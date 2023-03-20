---
title: "计算机科学--操作系统--Linux--系统调用"
create_date: 2021-12-16 08:00:00 +0800
date: 2022-01-24 08:00:00 +0800
tags: computer-science operating-system linux
comment: false
show_author_profile: true
show_subscribe: false
---

当启动一个程序之后，程序为了使用系统的相关资源（一般是硬件资源），就必须借助操作系统提供的系统接口（系统调用、syscall）才能操作。用户进程通过系统调用去调用系统资源，也就是，用户空间运行的程序切换到内核中去运行。

在linux系统中，系统调用存放在`/usr/include/asm/unistd.h`文件中。这个文件会根据系统的架构，加载不同的头文件。这些头文件里就是系统调用的声明。

汇编语言、高级语言（c/c++、java、golang）、脚本语言（php、python、nodejs）在运行的时候最终都是调用的系统调用。这些系统函数看上去是c语言风格的定义，但是程序最终变成汇编代码、机器码的时候是一样的。这点反过来想比较好理解，因为芯片是固定的，不可能应付各式各样的编程语言，所以只能是编程语言最终都变成一样的机器语言。

```
#ifndef _ASM_X86_UNISTD_H
#define _ASM_X86_UNISTD_H

/* x32 syscall flag bit */
#define __X32_SYSCALL_BIT       0x40000000

# ifdef __i386__
#  include <asm/unistd_32.h>
# elif defined(__ILP32__)
#  include <asm/unistd_x32.h>
# else
#  include <asm/unistd_64.h>
# endif

#endif /* _ASM_X86_UNISTD_H */
```

下面是`/usr/include/asm/unistd_32.h`文件的最前面的部分。比如：`#define __NR_fork 2`，`__NR_fork`是系统调用的名字，`2`是系统调用号。`__NR_fork`，就是`fork()`，`__NR_write`，就是`write()`，这些函数前面都用过的。

```
#ifndef _ASM_X86_UNISTD_32_H
#define _ASM_X86_UNISTD_32_H 1

#define __NR_restart_syscall 0
#define __NR_exit 1
#define __NR_fork 2
#define __NR_read 3
#define __NR_write 4
#define __NR_open 5
#define __NR_close 6
#define __NR_waitpid 7
#define __NR_creat 8
#define __NR_link 9
#define __NR_unlink 10
#define __NR_execve 11
```

可以使用`man`命令或者通过linux文档查看系统调用的使用方式，就像之前的文档里那样。