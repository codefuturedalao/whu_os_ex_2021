## System Architecture Overview

### Modes of operation

IA-32共有以下四种操作模式：

* 实模式

* 保护模式

* 虚拟8086模式

  该模式允许在保护模式下执行8086程序

* 系统管理模式（system management mode, SMM)

  为操作系统或执行环境提供了电源管理的透明机制，通过激活外部系统中断引脚（SMI#）触发系统管理中断（system managemtn interrupt, SMI）来进入SMM模式。在SMM模式中，处理器切换到分离的地址空间，透明执行指定的代码，然后执行RSM指令返回到之前的模式。

![image-20210921101657821](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921101657821.png)

其中实模式和保护模式是我们关注的重点。所有的Intel 64和32位处理器在加电后都会进入实模式，之后软件切换到保护模式下运行。

系统总览图如下：

![image-20210922112336635](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210922112336635.png)

### EFLAGS

EFLAGS寄存器控制着IO、可屏蔽硬件中断、任务切换和虚拟8086模式等。只有特权代码才允许修改这些位。

![image-20210921102053952](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921102053952.png)

* TF：Trap(bit 8)，为1时表示单步中断模式，处理器在每条指令执行后触发调试异常（debug exception），这允许检查程序每条指令执行的状态。当程序通过```POPF, POPFD, IRET```指令设置了TF位，则后面跟着的指令开始触发调试一场。
* IF：Interrupt enable(bit 9)，控制处理器对可屏蔽中断的响应，如果为1，则表示处理器响应可屏蔽中断，为0表示禁止可屏蔽中断。**IF位不影响异常和不可屏蔽中断。**CPL、IOPL和CR4中的VME位决定IF位是否能被CLI，STI，POPF，POPFD和IRET指令修改。
* IOPL：I/O privilege level field(bits 12 and 13)表示当前运行程序的IO特权级，当前运行程序的CPL必须数值上小于或等于IOPL值才可以访问IO地址空间，只有当CPL为0时，才可以通过POPF和IRET指令修改IOPL位。
* NT：Nested task(bit 14)，控制被中断和被调用任务，当通过call指令、异常或中断进入任务时，处理器置该位为1。当IRET从任务中返回时会检查和修改该位。
* VM：Virtual-8086 mode(bit 17)，置为表示进入虚拟8086模式，清除表示返回保护模式。
* ID：Identification(bit21)，表示是否支持CPUID指令。

### GDTR、LDTR、IDTR和TR

![image-20210921103902353](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921103902353.png)

GDT和IDT都不能称之为段，因为他们既没有段选择符，也没有段描述符。

#### GDTR

当处理器上电或reset时，GDTR的基地址为0，界限为0FFFFH，在进入保护模式之前，新的基地址和界限必须被加载进GDTR中。

#### LDTR

LDTR寄存器包含16位的段选择符、基地址、短界限和描述符属性，当LLDT指令向LDTR中加载段选择符时，基地址、段界限和描述符属性会被自动加载如LDTR中。当发生任务切换时，LDTR自动加载新任务的段选择符和描述符。

当处理器上电或reset时，GDTR的基地址为0，界限为0FFFFH。

#### IDTR

当处理器上电或reset时，GDTR的基地址为0，界限为0FFFFH。

#### TR

TR寄存器包含当前任务TSS的16位的段选择符、基地址、短界限和描述符属性。段选择符指向GDT表中的TSS描述符。

当LTR指令向TR中加载段选择符时，基地址、段界限和描述符属性会被自动加载如TR中。当发生任务切换时，TR自动加载新任务TSS的段选择符和描述符。

### Control Registers

![image-20210921105345794](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921105345794.png)

```{admonition}
在保护模式下0特权级下，MOV指令允许读取和加载控制寄存器。其他特权级下的程序无法读取和修改控制寄存器
```

* CR0：包含控制操作模式和处理器状态的系统控制位
  * PG：Paging(bit 31)：为1时表示启动分页，为0时表示禁止分页。当禁止分页时，所有的线性地址可以直接看作物理地址。当PE位为0时，设置PG位会触发保护异常，即只有操作系统是保护模式时，才可以启动分页。
  * CD：Cache Disable
  * NW：Not Write-through
  * WP：Write Protect(bit 16)，为1时，阻止高特权级程序向只读页面进行写操作；为0时，允许高特权级程序向只读页面写入（忽略U/S位）
  * EM：Emulation(bit 2)，为1时表示处理器没有内部或外部的x87 FPU；为0时表示存在x87 FPU
  * PE：Protection Enable(bit 0)，为1时表示启动保护模式；为0时表示进入是模式。

* CR1：Reserved
* CR2：包含触发缺页异常的线性地址
* CR3：包含页目录的起始物理地址和两个控制位（PCD和PWT）
* CR4：包含一些架构扩展控制位
  * PVI：Protected-Mode Virtual Interrupts(bit 1)，为1时启动对虚拟中断的硬件支持；为0时禁止虚拟中断。
  * PSE：Page Size Extensions(bit 4)，为1时表示页的大小为4MB，为0时表示页的大小为4KB。
  * PAE：Physical Address Extension(bit 5)，为1时，启动物理地址扩展，允许分页产生超过32位的物理地址；为0时，禁止物理地址扩展。
  * VMXE：VMX-Enable Bit(bit 13)，为1时，启动虚拟机操作扩展；为0时，禁止虚拟机操作扩展。

### System Instruction Summary

系统指令只允许在特权级0上执行，其他特权级不允许执行。

![image-20210921113540448](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921113540448.png)

## Protected-Mode Memory Management

### Overview

IA-32中的内存管理机制被分为两部分：分段和分页。**在保护模式中，分段模式自动开启，没有控制位来禁止分段。**而分页机制则是可选的。

![image-20210921114706298](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921114706298.png)

### Segment

分段机制可用来实现一系列系统设计，这些设计包含简单使用分段的平坦模型到使用分段建立了健壮的操作环境的多分段模型。

* Basic Flat Model

  代码段和数据段基地址从0开始，界限为4GB，访问不存在内存设备的物理地址也不会触发异常，需要分页机制来实现进程和进程的内存空间隔离与保护。

  ![image-20210921115600711](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921115600711.png)

* Protected Flat Model

  设置了段界限，避开了不存在物理内存的物理地址，需要分页机制来实现进程和进程的内存空间隔离与保护。

  ![image-20210921115733505](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921115733505.png)

* Multi-segmented Model

  充分使用了分段机制，每个程序拥有自己的段描述符表和段，每个段都做到了充分的隔离。

  ![image-20210921115853091](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921115853091.png)

#### Segment Selector

![image-20210921144638225](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921144638225.png)

由于GDT的第一个描述符不会被处理器使用，因此指向该描述符的选择符（index和ti为0）被用作null段选择符，可以用来初始化一个未使用的段寄存器，然而，通过null段选择符访问数据以及向CS和SS寄存器加载null段选择符会导致保护异常(#GP)。

#### Segment Registers

为了减少地址转换时间和代码复杂度，处理器提供了6个段寄存器。每一个段寄存器都有一个可见部分和隐藏部分，当段选择符被加载到段寄存器的可见部分，处理器会自动将段寄存器的隐藏部分（段基址、段界限和访问信息）加载进来。这样处理器每次访问内存便不用再去段选择符指向的描述符中获取信息，而是直接从段寄存器中获取。

![image-20210921145351514](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921145351514.png)

当段描述符发生变化，软件有责任重新更新段寄存器的内容，否则段寄存器很可能使用过时的信息来访问内存。

#### Segment Descriptor

段描述符通常由编译器或操作系统创建，而不是应用程序。

![image-20210921150627003](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921150627003.png)

**注：段基址最好为16字节对齐，这样有利于提高性能。**

描述符共有以下几种类型：

* 代码和数据段描述符（S位为1）

  ![image-20210921151102035](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921151102035.png)

* 系统描述符（S位为0）

  * 系统段描述符（指向系统段）
    * LDT段描述符
    * TSS描述符
  * 门描述符（包含指向代码段或TSS的选择符）
    * 调用门
    * 中断门
    * 陷阱门
    * 任务门

  ![image-20210921152527514](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921152527514.png)

### Paging

软件通过MOV指令设置CR0的PG位来启动分页，在此之前要确保CR3中包含页目录的起始地址以及初始化相应内容。

IA-64处理器共支持四种不同的分页方式：

* 32位分页
* PAE分页
* 4级分页
* 5级分页

我们用到的是32位分页。此时CR0.PG=1以及CR4.PAE=0。

当CR4的PSE和PDE的PS位为0，表示采用4KB页大小，寻址方式如下所示：

![image-20210921200917925](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921200917925.png)

![image-20210921201712475](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210921201712475.png)

* R/W：0表示只读，1表示可读写
* U/S：为0表示不允许用户模式访问，只允许特权级为0、1、2的程序访问。为1时，允许所有特权级程序访问。
* A：Accessd，由处理器固件来设置，用来指示此表象所指向的页是否被访问过。
* D：Dirty，由处理器固件来设置，用来指示此表象所指向的页是否被写过数据。
* PAT
* G：如果CR4.PGE=1，决定是否地址转换为全局。

#### Access Rights

对线性地址的访问可以分为两种：

* supervisor-mode access：CPL < 3
* user-mode access：CPL=3

其中对线性地址隐式的访问如从GDT或LDT中加载段选择符被视为supervisor-mode access（不管其CPL为多少）。

其访问权限的判断过于复杂，此处略去，推荐同学们去手册中直接查看。粗略总结就是supervisor-mode access可以访问supervisor和user的地址（由U/S位决定）。但user只能访问自己的，不可以访问supervisor的。

#### Page Fault Exception

缺页异常由两种情况导致：

1. 没有线性地址对应的物理地址转换（P位为0）
2. 有地址转换，但access rights不允许本次访问

## Protection



## Reference

Intel® 64 and IA-32 Architectures Software Developer’s Manual Volume 3 (3A, 3B, 3C & 3D): System Programming Guide