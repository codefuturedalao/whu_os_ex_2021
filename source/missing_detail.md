## Why [0x0000fffffff0] f000:fff0 instead of ffff:0000?



![image-20210923123532962](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210923123532962.png)

## Why 0x7c00?

[阮一峰网络日志：为什么主引导记录的内存地址是0x7C00？](http://www.ruanyifeng.com/blog/2015/09/0x7c00.html)

## Why 0100h in com

## Why we need rpl?

## How does BIOS establish IVT?

你可能会奇怪，BIOS如何建立起低地址1KB的中断向量表和相应的中断例程？它又是怎么知道访问硬件的？毕竟，即使是它，要访问硬件也要通过端口一级的途径。

答案是，BIOS可能会为一些简单的外围设备提供初始化代码和功能调用代码，并填写中断向量表，但也有一些BIOS中断是由外部设备接口自己建立的。

首先，每个外部设备接口，包括各种板卡，如网卡、显卡、键盘接口电路、硬件控制器等，都有自己的只读存储器（ROM），类似于BIOS芯片，这些ROM提供了他自己的功能例程，以及本设备的初始化代码。按照规范，前两个单元的内容是0x55和0xAA，第三个单元是本ROM中以512字节为单位的代码长度；从第四个单元开始，就是实际的ROM代码。

其次，我们知道，从内存物理地址A0000开始，到FFFFF结束，有相当一部分空间是留给外围设备的，如果设备存在，那么，它自带的ROM会映射到分配给它的地址范围内。在计算机启动期间，BIOS程序会以2KB为单位搜索内存地址C0000~E0000之间的区域。当他发现某个区域的头两个字节是0x55和0xAA时，那意味着该区域有ROM代码存在，是有效的。接着对该区域做累加和检查，看结果是否和第三个单元相符，如果相符，则从第四个单元进入。这是处理器执行的是硬件自带的程序指令，这些指令初始化外部设备的相关寄存器和工作状态，最后，填写相关的中断向量表，使他们指向自带的中断处理过程。

![image-20210923115930672](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210923115930672.png)

详情参考：[https://en.wikipedia.org/wiki/BIOS](https://en.wikipedia.org/wiki/BIOS)