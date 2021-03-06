# week8

## 实验内容

1. 多进程问题，如何扩展单进程到多进程，如何扩展中断支持多进程？
2. 如何实现系统调用
3. 进程调度问题，弄清楚实现调度的基本思路

## 实验问题

1. 在单进程的基础上扩展实现多进程要考虑哪些问题？

2. 画出以下关键技术的流程图：

   初始化多进程控制块的过程、扩展初始化LDT和TSS

3. 如何修改时钟中断来支持多进程管理，画出新的流程图。
4. 系统调用的基本框架是如何的，应该包含哪些基本功能，画出流程图。
5. 如何操控可编程计数器？
6. 进程调度的框架是怎样的？优先级调度如何实现？
7. 动手做：修改例子程序的调度算法，模拟实现一个多级反馈队列调度算法，并用其尝试调度5-8个任务。注意，抢占问题，注意时间片问题。
8. 思考题：从用户态进程读和写内核段的数据，看能否成功，是否会触发保护，并解释原因。

## 实验步骤

给出一个简化版多级反馈队列实现，没有到达时间和任务所需时间片的概念。

1. 头文件中添加相应数据结构，主要有三个队列和对应的时间片，队列用数组来实现，存储u32类型的pid，优先级越高的队列对应时间片越少。

   ![image-20211118201522411](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211118201522411.png)

2. 在kernel_main中初始化数据

   ![image-20211118201700353](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211118201700353.png)

3. 最开始向高优先级队列中加入A和B两个进程

   ![image-20211118201721768](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211118201721768.png)

4. 修改schedule，下面只给出部分代码，思路为寻找第一个不为-1的元素，找到后查看进程剩余时间片，如果大于0则进行调度，小于等于0则移到下一级队列中（通过数组元素移动）。

   ![image-20211118201821933](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211118201821933.png)

5. 令C在tick为12时，即A和B都进入中优先级队列时到达，进行抢占

   ![image-20211118202726711](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211118202726711.png)

6. 查看结果，发现A和B各自运行完5个时间片后，进入中优先级队列，A运行两个时间片后C到达进行抢占，C运行5个时间片后进入中优先级队列，A继续运行8个时间片，之后转入低优先级队列，B开始运行，然后C，循环往复。

   ![image-20211118202753608](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211118202753608.png)

## FAQ

