# week4

## 实验内容

1. 理解中断与异常的机制 

2. 调试8259A的编程基本例程 

3. 调试时钟中断例程 

4. 实现一个自定义的中断向量，功能可自由设想 

## 实验步骤

### 添加IDT表项

添加21号中断对应的中断门，同时初始化8259A打开相应的中断

![image-20211003202534811](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211003202534811.png)

### 编写中断处理程序

编写处理程序，每次接收到键盘输入，修改字符的颜色

![image-20211003202653007](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211003202653007.png)

### 查看实验结果

每次敲击键盘，可以看到右上角颜色发生变化。

![image-20211003202911027](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211003202911027.png)

## FAQ

### Q：编写键盘中断，为什么运行程序后还没按下键盘就执行了键盘中断程序

可能是因为enter键松开时，键盘中断被程序所收到，导致执行键盘中断程序

### Q：为什么只能按下键盘触发一次键盘中断？

请按以下步骤排查：

1. 检查是否发送了EOI
2. 检查是否使用命令```in al, 0x60```清除了键盘缓冲区

### Q：是否可以使用BIOS中断？

进入保护模式后，中断从原来的从IVT找中断向量变成了根据中断向量号索引IDT中的门描述符，无法在使用int指令调用BIOS中断。