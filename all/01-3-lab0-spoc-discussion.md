# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？
- 答：在硬件设计上至少支持时钟中断、地址映射、MMU、稳定的存储介质;同时，操作系统应当提供中断使能、触发软中断，设置内存寻址模式，设置页表等内存管理，执行I/O操作等文件系统相关的特权指令。

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？
- 实模式将整个物理内存看成分段的区域，程序代码和数据位于不同区域，系统程序和用户程序没有区别对待，而且每一个指针都是指向“实在”的物理地址;而保护模式下物理内存地址不能直接被程序访问，程序内部的地址要由操作系统转化为物理地址去访问。实模式下可访问的物理内存空间不能超过1MB，且无法发挥Intel 80386以上级别的32位CPU的4GB内存管理能力;而保护模式下80386的全部32根地址线有效，可寻址高达4G字节的线性地址空间和物理地址空间，可访问64TB（有2^14个段，每个段 最大空间为2^32字节）的逻辑地址空间，可采用分段存储管理机制和分页存储管理机制。

-物理地址：是指出目前CPU外部地址总线上的寻址物理内存的地址信号，是地址变换的最终结果地址。线性地址：程式代码会产生逻辑地址，或说是段中的偏移地址，加上相应段的基地址就生成了一个线性地址。逻辑地址：是指由程式产生的和段相关的偏移地址部分。

- 

- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？
- RISC-V定义了四种特权级模式：用户模式，管理员模式，监督模式，特权模式。机器模式下的代码是固有可信的，因为它可以在低层次访问机器的实现;用户模式和管理员模式被分别用于传统应用程序和操作系统;而监督模式则是为了支持虚拟机监视器。 

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

- “:”后的数字表示每一个域在结构体中所占的位数

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

- 0x20003

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。
- 代码：
- struct context {
-   uint32_t eip;
-   uint32_t esp;
-   uint32_t ebx;
-   uint32_t ecx;
-   uint32_t edx;
-   uint32_t esi;
-   uint32_t edi;
-   uint32_t ebp;
- };

- 开始时，栈顶是返回地址，下面（esp-4）是from（因为参数是从右往左压栈的），再下面是to。系统先从栈中取出from，然后把该函数的返回地址弹出，保存到from->eip中。然后依次保存各个通用寄存器（段寄存器不需要保存，因为内核线程之间这些寄存器都一样）。因为eax中保存的总是返回值，所以可以不保存它，简化代码。之后就是从栈中再取出to，恢复通用寄存器，最后把to->eip入栈，保证返回之后能跳转到正确地址。
2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。
- 用途：
- 利用宏进行复杂数据结构中的数据访问；
- 利用宏进行数据类型转换；如 to_struct
- 常用功能的代码片段优化；如 ROUNDDOWN, SetPageDirty

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
