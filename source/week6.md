# week6

## 实验内容

1. 汇编和C的互相调用方法

   在例程基础上，在汇编与C程序中各添加一个简单带参数的函数调用，让两种语言撰写的程序实现混合调用，功能可自定义。

2. ELF文件格式

   依照书上方法，分析你修改的这个可执行文件

3. 使用Loader加载ELF文件

4. 如何加载并扩展内核

5. 设计题：修改启动代码，在引导过程中在屏幕上画出一个你喜欢的ASCII图案，并将第三章的内存管理功能代码、你自己设计的中断代码集成到你的kernel文件目录管理中，并建立makefile文件，编译成内核，并引导

## 实验问题

1. 汇编和C的调用方法是怎样的？

2. 描述ELF文件格式以及作用

3. 描述从Loader引导ELF的原理

4. 一个内核要能基本使用应该扩展哪些功能，怎么扩展？

5. 怎么管理内核文件目录？

## 实验步骤

### 启动显示ASCII图案

1. 通过neofetch或figlet工具获取图案（什么图案都可以）

   ![image-20211024100021459](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211024100021459.png)

2. 在kernel/start.c中编写函数，显示图案

   ![image-20211024100125866](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211024100125866.png)

3. 在kernel/kernel.asm中extern声明printBootLogo函数，同时在合适的位置调用（需要修改disp_str函数实现超出屏幕范围翻页的功能）

   ![image-20211024100256029](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211024100256029.png)

### 中断函数集成

以实现的键盘中断为例

1. 修改代码，使kernel进入死循环（不然hlt被中断唤醒后会继续执行hlt后面的指令，导致出现错误）

   ![image-20211024100817477](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211024100817477.png)

2. 修改hwint01函数，改成自己的中断处理代码

   ![image-20211024100854184](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211024100854184.png)

3. 测试程序，每次按键盘，发现颜色变化

   ![image-20211024101142507](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211024101142507.png)

### 内存管理集成

为了简单展示修改Makefile，我们将创建mm文件夹，将前面内存管理的函数放入该文件夹的pageManage.asm文件中

1. 将代码拷贝进入pageManage.asm中，注意声明标号以及添加所需要的数据结构

   ![image-20211024101341949](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211024101341949.png)

   需要注意的是，对数据的引用无需使用```xxx equ _xxx - $$```，直接使用```_xxx```即可，请思考为什么。

2. 在kernel/kernel.asm中添加测试函数并调用，被测试函数记得extern声明。

   ![image-20211024102148285](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211024102148285.png)

3. 修改Makefile，在OBJS中添加```mm/pageManage.o```，在文件末尾添加对该文件的汇编命令

   ![image-20211024101730081](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211024101730081.png)

4. 测试程序，由于之前验证过其正确性，因此正常显示输出即可判定其工作正常。

   ![image-20211024102119780](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211024102119780.png)

   在断点处分别查看页表的映射情况，alloc_pages和free_pages也正常工作

   ![image-20211024102329706](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211024102329706.png)

## FAQ

### Q: 出现undefined reference to `__stack_chk_fail`错误，如何解决？

需要在 `Makefile` 中的 `$(CFLAGS)` 后面加上 `-fno-stack-protector`，即不需要栈保护

![image-20211024102407702](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211024102407702.png)

### Q：出现xxx is incompatible with i386:x86-64 output

有些同学的Ubuntu系统是64位，而不是32位，那么gcc默认编译的目标文件是64位，无法和32位汇编文件汇编出的目标文件进行链接，需要在Makefile的CFLAGS后面添加```-m32```，LDFLAGS后面添加```-m elf_i386 ```

### Q：如何方便的在标号声明处和定义处进行跳转？

由于建立了文件目录，浏览代码时不容易查找标号对应的定义，对于使用vim的同学，推荐使用ctags工具，方便标号声明和定义的跳转，对于仍在使用自带编辑器的同学，可以尝试vscode。

### Q：myprint不好用，如何使用printf打印输出？

我们采用动态链接的方法将libc.so和程序链接起来。下面操作环境为Ubuntu14.04，gcc 4.8.4

首先包含<stdio.h>头文件

![image-20211030090059779](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211030090059779.png)

修改Makefile，在链接参数中添加```--dynamic-linker /lib/ld-linux.so.2 -lc```

![image-20211030090138171](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211030090138171.png)

make后运行，可以看到正常打印

![image-20211030090318628](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211030090318628.png)

```{admonition} 提示

当你阅读到chapter5/f/start.c时，请留意memcpy第一个参数，尝试修改将gdt前面的&符号去掉，查看结果是否一致并思考为什么，这同时会帮助你理解在C语言中声明指针和数组后，在汇编语言中想要使用它们时有什么不同

```
