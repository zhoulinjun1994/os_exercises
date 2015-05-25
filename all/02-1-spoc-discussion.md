#lec 3 SPOC Discussion

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
 1. 比较UEFI和BIOS的区别。
 1. 描述PXE的大致启动流程。

## 3.2 系统启动流程
 1. 了解NTLDR的启动流程。
 1. 了解GRUB的启动流程。
 1. 比较NTLDR和GRUB的功能有差异。
 1. 了解u-boot的功能。

## 3.3 中断、异常和系统调用比较
 1. 举例说明Linux中有哪些中断，哪些异常？
 1. Linux的系统调用有哪些？大致的功能分类有哪些？  (w2l1)

```
  + 采分点：说明了Linux的大致数量（上百个），说明了Linux系统调用的主要分类（文件操作，进程管理，内存管理等）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 进程控制、文件系统控制、系统控制、内存管理、网络管理、socket控制、用户管理、进程间通信
 举例：创建进程、读写文件等190多项。
 
 1. 以ucore lab8的answer为例，uCore的系统调用有哪些？大致的功能分类有哪些？(w2l1)
 
 ```
  + 采分点：说明了ucore的大致数量（二十几个），说明了ucore系统调用的主要分类（文件操作，进程管理，内存管理等）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
 lab8 answer中syscall.c文件夹共有二十几个系统调用函数，包含了文件操作、进程控制、内存控制等多个方面的函数，有如下函数
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
 这里包含了一些创建，运行，中止进程，打开，关闭，读写，查找文件等函数。

 
## 3.4 linux系统调用分析
 1. 通过分析[lab1_ex0]( )了解Linux应用的系统调用编写和含义。(w2l1)
 

 ```
  + 采分点：说明了objdump，nm，file的大致用途，说明了系统调用的具体含义
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 
 ```
 lab1_ex0.s是一个输出hello world的程序，其中调用了系统调用中的标准输出函数。
 objdump是用于查看目标文件或者可执行的目标文件的构成的GCC工具，
 nm是用来查看指定程序中的符号表相关内容的工具，
 file是用于查看文件的工具
 
 1. 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(w2l1)
 

 ```
  + 采分点：说明了strace的大致用途，说明了系统调用的具体执行过程（包括应用，CPU硬件，操作系统的执行过程）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
系统调用是通过软中断指令 INT 0x80 实现的，而这条INT 0x80指令就被封装在C库的函数中。
INT 0x80 这条指令的执行会让系统跳转到一个预设的内核空间地址，它指向系统调用处理程序，即system_call函数。
system_call函数通过系统调用号查找系统调用表sys_call_table。软中断指令INT 0x80执行时，系统调用号会被放入 eax 
寄存器中，system_call函数可以读取eax寄存器获取，然后将其乘以4，生成偏移地
址，然后以sys_call_table为基址，基址加上偏移地址，就可以得到具体的系统调用
服务例程的地址。
然后就到了系统调用服务例程了。系统调用服务例程只会从堆栈里获取参数，所以在system_call执行前，会先将参数存放在寄存器中，system_call
执行时会首先将这些寄存器压入堆栈。system_call退出后，用户可以从寄存器中获得参数。
 
## 3.5 ucore系统调用分析
 1. ucore的系统调用中参数传递代码分析。
 1. ucore的系统调用中返回结果的传递代码分析。
 1. 以ucore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
 1. 以ucore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。
 
## 3.6 请分析函数调用和系统调用的区别
 1. 请从代码编写和执行过程来说明。
   1. 说明`int`、`iret`、`call`和`ret`的指令准确功能
 
