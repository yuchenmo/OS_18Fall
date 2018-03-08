# lec 3 SPOC Discussion

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
-  BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

    512字节的主引导记录和硬盘分区表；操作系统内核显然不止512字节

- 比较UEFI和BIOS的区别。

    1. UEFI将系统启动文件与操作系统本身隔离，起到保护作用
    2. 可以支持2TB以上硬盘

## 3.2 系统启动流程

- 分区引导扇区的结束标志是什么？

    0x55, 0xAA

- 在UEFI中的可信启动有什么作用？

    使用数字签名保证安全性

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？

    中断是外部（软/硬）的响应
    异常是执行时不可预料的意外（从而导致的某些失败）的响应
    系统调用是系统程序主动调用指令的响应

-  中断、异常和系统调用的处理流程有什么异同？

    同：都需要进入内核态，都需要设置中断向量
    异：太多…课件上那个图的不同路径区别太大，例如只有异常需要异常服务例程，只有中断来自外部设备等

- 以ucore lab8的answer为例，uCore的系统调用有哪些？大致的功能分类有哪些？

    有以下代码列出的22个，主要功能分为文件操作、进程管理、内存分配、外设输出等。
    static int (*syscalls[])(uint32_t arg[]) = {
        [SYS_exit]              sys_exit,
        [SYS_fork]              sys_fork,
        [SYS_wait]              sys_wait,
        [SYS_exec]              sys_exec,
        [SYS_yield]             sys_yield,
        [SYS_kill]              sys_kill,
        [SYS_getpid]            sys_getpid,
        [SYS_putc]              sys_putc,
        [SYS_pgdir]             sys_pgdir,
        [SYS_gettime]           sys_gettime,
        [SYS_lab6_set_priority] sys_lab6_set_priority,
        [SYS_sleep]             sys_sleep,
        [SYS_open]              sys_open,
        [SYS_close]             sys_close,
        [SYS_read]              sys_read,
        [SYS_write]             sys_write,
        [SYS_seek]              sys_seek,
        [SYS_fstat]             sys_fstat,
        [SYS_fsync]             sys_fsync,
        [SYS_getcwd]            sys_getcwd,
        [SYS_getdirentry]       sys_getdirentry,
        [SYS_dup]               sys_dup,
    };


## 3.4 linux系统调用分析
-  通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。

 
## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？

    指令不同（表象），特权级与堆栈不同（本质）

- 通过分析`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较函数调用与系统调用的堆栈操作有什么不同？

    系统调用：

