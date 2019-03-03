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
  > 定时中断用于进程的调度，地址映射（MMU）用于虚拟内存的管理。特权指令至少应该提供中断的管理（使能与禁止）、地址映射管理（页表、TLB、寻址模式修改等）。

- 你理解的x86的实模式和保护模式有什么区别？你认为从实模式切换到保护模式需要注意那些方面？
  > 实模式可以直接使用物理地址，并且没有特权级别；保护模式下不能直接访问物理地址，需要通过段和页管理、访问内存，并设计了不同的特权级被，防止重要程序（如操作系统）被用户程序破坏。

- 物理地址、线性地址、逻辑地址的含义分别是什么？它们之间有什么联系？
  > 物理地址是可访问的硬件地址，线性地址是通过段转换后得到的地址，虚拟地址是用户程序可以使用的地址（每个程序均一样）。

- 你理解的risc-v的特权模式有什么区别？不同模式在地址访问方面有何特征？
  > 有三种：Machine Mode：机器模式，简称M Mode；Hypervisor Mode：虚拟机监视器模式，简称 H Mode；Supervisor Mode：监督（系统）模式，简称S Mode；User Mode：用户模式，简称U Mode。M Mode 可以直接访问物理地址，此时 MMU 不工作；运行在其他模式的程序需要经过 MMU 的地址映射（不同于 MPIS，RSIC-V 的整个地址空间都需要经过页表转换）。

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
  > 这是 C 的 bitfield 特性，表示该成员的位宽。

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

  > 无法理解本题：intr 作为一个无符号数，直接传给 `SETGATE` 宏将导致编译错误。猜测本题含义是考察C的结构体的内存布局和端序。

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

```as
seta20.1:
    inb $0x64, %al
    testb $0x2, %al
    jnz seta20.1
    movb $0xd1, %al
    outb %al, $0x64
seta20.2:
    inb $0x64, %al 
    testb $0x2, %al
    jnz seta20.2
    movb $0xdf, %al
    outb %al, $0x60
```

  > 这段代码用于设置 8042 控制器以打开 A20 地址线，做进入保护模式前的准备。首先读 0x64 端口的数据直到其第二位为 1，然后写入 0x64 端口，再次等待其不忙，而后向 0x60 端口写入控制信号。其中用到了类似 (while a & 0x02 !== 1); 的结构。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

```c
#define assert(x)                                       \
    do {                                                \
        if (!(x)) {                                     \
            panic("assertion failed: %s", #x);          \
        }                                               \
    } while (0)

#define static_assert(x)                                \
    switch (x) { case 0: case (x): ; }
```

  > 这段代码提供的 `static_assert` 在 `x` 为假的情况下，会导致 case 中有两个重复的分支，从而产生编译时错误；`assert` 则在运行时进行求值，如果 `x` 为假则触发内核崩溃。

## 问答题

#### 在配置实验环境时，你遇到了那些问题，是如何解决的。

  > * 编译出的Bootloader 大于 512 B：为 gcc 添加 `-fno-PIC -fno-PIE` 选项
  > * 执行到 LGDT 函数时虚拟机崩溃：修改 linker script 防止 data 区域被破坏

## 参考资料
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
 - [v9 cpu architecture](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)
 - [RISC-V cpu architecture](http://www.riscvbook.com/chinese/)
 - [OS相关经典论文](https://github.com/chyyuu/aos_course_info/blob/master/readinglist.md)
