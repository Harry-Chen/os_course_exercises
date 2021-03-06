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
-  请描述在“计算机组成原理课”上，同学们做的MIPS CPU是从按复位键开始到可以接收按键输入之间的启动过程。
-  x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？
  > BIOS 无法读取文件系统，也不能知道操作系统的引导方式。因此需要从第一个扇区读取 Bootloader 并执行。
- 比较UEFI和BIOS的区别。
  > BIOS 是实模式的，UEFI 工作在保护模式，两者可访问的地址空间和可使用的硬件有区别。BIOS 仅提供有限的硬件支持，UEFI 能够为设备加载驱动。
- 理解rcore中的Berkeley BootLoader (BBL)的功能。
  > BBL 是 proxy kernel，工作在 H Mode，能够为上层 S Mode 的操作系统提供一致的底层 IO 接口。使得操作系统的实现不必涉及太多平台相关的内容，解除了操作系统和硬件的耦合。

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？
  > `0x55 0xAA`
- x86中在UEFI中的可信启动有什么作用？
  > 从第一级引导程序开始逐步验证，从而保证启动的系统是可信的、安全的
- RV中BBL的启动过程大致包括哪些内容？
  > 探测硬件、建立 DTB 设备树、进入 S 模式（加载内核）

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？
  > * 中断：外部意外的响应
  > * 异常：指令执行意外的响应
  > * 系统调用：系统调用指令的响应
-  中断、异常和系统调用的处理流程有什么异同？
  > 都会进入内核态并执行异常服务例程，但中断是异步的，后两者是同步的
- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？
  > * 文件读写： `open, read, close, write, seek, fstat, fsync, getcwd, getdireentry, dup`
  > * 进程管理： `fork, exit, wait, exec, yield, kill, getpid, sleep`
  > * 其他：`pgdir, putc`

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。

 
## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？
  > 系统调用会有特权级别转换，函数调用没有。系统调用由操作系统完成，函数调用只在程序内部。
- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
  > `int` 与 `iret` 指令事实上是异常的触发/返回，会使用一个额外的异常栈，并会将当前处理器一些状态（比如 `SS:EIP`）压栈，处理完毕后再弹出。而 `call` 与 `ret` 指令只是将函数返回地址压/弹栈。
- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较RISC-V中函数调用与系统调用的堆栈操作有什么不同？
  > `ecall` 触发异常，此时系统需要自行保存现场，错误地址和原因等会被 CPU 保存在控制寄存器中，最后靠 `eret` 恢复到触发异常的地址。而 `jal` 和 `jalr` 只是把下一条指令的地址保存在 `$ra` 中然后跳转。


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
