# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
- x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

- 答：BIOS从磁盘中第一个扇区读出bootloader。为了增加灵活性，磁盘上有多种多样的文件系统，BIOS无法认识所有的文件系统代码，因此约定读入加载程序，用加载程序来识别磁盘上文件系统。

- 比较UEFI和BIOS的区别。

- 答：相比BIOS，UEFI通过保护预启动或预引导进程，抵御bootkit攻击，从而提高安全性;缩短了启动时间和从休眠状态恢复的时间;支持容量超过2.2 TB的驱动器;支持64位的现代固件设备驱动程序，系统在启动过程中可以使用它们来对超过172亿GB的内存进行寻址;UEFI硬件可与BIOS结合使用。

- 理解rcore中的Berkeley BootLoader (BBL)的功能。

  

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？
- 答：55AAH
- x86中在UEFI中的可信启动有什么作用？
- 答：安全性更强、开机速度快、支持大硬盘
- RV中BBL的启动过程大致包括哪些内容？
- 答：初始化RAM、初始化串口、检测处理器类型、设置Linux启动参数、调用Linux内核映像

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？
- 答：中断是来自硬件设备的处理请求;异常是非法指令或者其他原因导致当前指令执行失败后的处理请求;系统调用是应用程序主动向操作系统发出的服务请求。
- 中断、异常和系统调用的处理流程有什么异同？
- 答：相同：无论是发生异常、中断，还是系统调用，都需要由硬件保存现场和中断号，转到内核态，进入中断向量表，查找对应的设备驱动程序地址、异常服务例程地址，或找到系统调用表，并在其中查找对应的系统调用实现的起始地址。处理完毕之后，再进行现场的切换，回到用户态继续执行程序。不同：中断是由外设引起，异步响应，持续，对用户应用程序是透明的;异常是应用程序意想不到的行为，同步响应，杀死或者重新执行意想不到的应用程序指令;系统调用是应用程序请求操作系统提供服务，异步或同步响应，等待和持续。
- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？
- 答：功能大致分为4类：
- 进程管理：包括 fork/exit/wait/exec/yield/kill/getpid/sleep
- 文件操作：包括 open/close/read/write/seek/fstat/fsync/getcwd/getdirentry/dup
- 内存管理：pgdir命令
- 外设输出：putc命令

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。


## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？
- 答：汇编指令：系统调用使用INT和IRET指令;函数调用使用CALL和RET指令。安全性：系统调用有堆栈和特权级的转换过程，函数调用没有这样的过程，系统调用相对更为安全。性能：时间方面，系统调用比函数调用要做更多和特权级切换的工作，需要更多时间开销;空间方面，如果函数调用采用静态编译，往往需要大量空间开销，此时系统调用更有优势
- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
