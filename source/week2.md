# week2

## 实验内容

1. 认真阅读章节资料，掌握什么是保护模式，弄清关键数据结构：GDT、descriptor、selector、GDTR， 及其之间关系，阅读pm.inc文件中数据结构以及含义，写出对宏Descriptor的分析

2. 调试代码，/a/ 掌握从实模式到保护模式的基本方法，画出代码流程图，如果代码/a/中，第71行有dword前缀和没有前缀，编译出来的代码有区别么，为什么，请调试截图。

3. 调试代码，/b/，掌握GDT的构造与切换，从保护模式切换回实模式方法

4. 调试代码，/c/，掌握LDT切换

5. 调试代码，/d/掌握一致代码段、非一致代码段、数据段的权限访问规则，掌握CPL、DPL、RPL之间关系，以及段间切换的基本方法

6. 调试代码，/e/掌握利用调用门进行特权级变换的转移的基本方法

## 实验步骤

### 配置环境

1. 运行命令```dd if=pmtest1.bin of=a.img bs=512 count=1 conv=notrunc```将pmtest1.bin写入a.img，然后启动bochs，查看结果，可以看到界面右侧出现红色p

   ![image-20210917145432463](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917145432463.png)

2. 下载[freedos](http://bochs.sourceforge.net/guestos/freedos-img.tar.gz)，将解压后的a.img命名为freedos.img，复制到工作目录下（linux中命令行重命名命令为mv）

3. 用bximage生成一个软盘映像，起名为pm.img

4. 修改bochsrc，确保有以下三行。我们来解释一下，这三行说明我们有两个软盘，分别是freedos.img和pm.img，我们将从a盘也就是floppya启动，floppyb会自动成为b盘。

   ![image-20210917145903441](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917145903441.png)

5. 启动bochs。

   ![image-20210917150004023](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917150004023.png)

6. 输入```format b:```格式化B盘，B盘其实就是我们的pm.img镜像文件

   ![image-20210917150137783](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917150137783.png)

   格式化操作相当于将其转化为FAT32文件系统，当我们到terminal查看pm.img内容时，可以看到pm.img中已经写入了一些文件系统相关的东西，不再是那个全为0的它了。

   ![image-20210917150538255](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917150538255.png)

7. 修改pmtest1.asm代码中的```07c00h```为```0100h```

   ![image-20210917150659111](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917150659111.png)

8. 输入命令```nasm pmtest1.asm -o pmtest1.com```将pmtest1.asm编译为com文件

9. 挂载pm.img，并将生成的pmtest.com拷贝到软盘中

   ```
   sudo mkdir /mnt/floppy
   sudo mount -o loop pm.img /mnt/floppy
   sudo cp pmtest1.com /mnt/floppy/
   sudo umount /mnt/floppy
   ```

10. 重新启动bochs，在freedos中运行命令，可以看到界面右侧出现红色字母p

    ![image-20210917151405604](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917151405604.png)

    由于pmtest1中的```jmp $```，你会看到程序未将控制权交还给freedos，不用担心，后面你会看到如何将控制权交还给freedos。

### 升级装备

现在休息一下，可以喝口水什么的。

你可能发现由于我们不知道freedos把程序加载到了哪里，所以在哪里打断点来进行调试对于我们来说非常棘手😔，这里推荐查看一下FAQ中的how to debug in freedos。

现在假设你已经已经知道了如何调试，那么我们后面的任务无非就是在想要打断点的位置加上```xchg bx, bx```，然后启动bochs进行调试即可。

实验的时候，我们需要改动pmtest中的代码，然后汇编、挂载pm.img、拷贝、取消挂载以及启动bochs，这几个重复且枯燥的动作让人心疲力竭，于是这里提供一个简单的脚本，可能可以节省一下我们的时间。（直接使用makefile也可以实现同样的效果）

```shell
#!/bin/sh

# Author : jackson sang
# toy script

sudo mount pm.img -o loop /mnt/floppy
for f in *.asm;
do
	nasm "${f}" -o "${f%.asm}.com"
	if [ $? -ne 0 ]; 
	then
		sudo umount /mnt/floppy
    		echo "Failed to assemble!!!"
    		exit 1
	fi
	sudo cp  "${f%.asm}.com" /mnt/floppy
	rm "${f%.asm}.com"
done

sudo umount /mnt/floppy
bochs
```

上面代码的作用就是将当前文件夹下所有的汇编文件编译为com文件，然后写入pm.img，最后启动bochs。

如果你愿意使用的话，请新建一个build.sh文件，将上面代码拷贝进去，然后使用```sudo chmod +x build.sh```命令使build.sh可执行，之后每次修改完汇编文件，使用命令```./build.sh```即可启动bochs了。

### 课后实验

```{admonition} 提示
课后实验思路仅供参考，故尽可能减少示例图片的出现和文字描述。
```

#### 课后动手改1

1. 编写GDT代码段，写入内存内容

   ![image-20210917200414001](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917200414001.png)

2. 编写LDT代码段，读取GDT内容并打印

   ![image-20210917200457485](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917200457485.png)

3. 增加GDT代码段描述符，以及设置选择符

4. 增加LDT代码段描述符，以及设置选择符

5. 在实模式代码中初始化新增段的描述符

   ![image-20210917200601733](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917200601733.png)

   ![image-20210917200610459](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917200610459.png)

6. 编译执行（请注意修改相应位置来确保你的代码会跳转到新增的两个代码段执行）

   ![image-20210917200728687](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917200728687.png)

#### 课后动手改2

实验代码已经提供了高特权级通过retf跳到低特权级，低特权级通过调用门跳回高特权级的代码，于是我们采用另一种方法，通过设置高特权级代码段为一致性代码段，从而实现低特权级到高特权级的跳转。（哪种方式都是可以的，只要实现不同特权级代码段跳转即可）

1. 定义dpl=3的数据段并定义段描述符和段选择符，同时初始化段描述符。

   ![image-20210924111925899](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210924111925899.png)

2. 定义ring3代码段描述符和段选择符，同时初始化段描述符

3. 编写ring3代码段，实现输出字符串，并通过call指令跳转到ring0代码段

   ![image-20210924110712138](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210924110712138.png)

4. 定义ring0代码段描述符和选择符，同时初始化段描述符

   需要在代码段中设置DA_CCO属性，表示为一致性代码段。

   ![image-20210924111411154](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210924111411154.png)

5. 编写ring0代码段，实现输出字符串

   请注意，由于从ring3跳转到ring0一致性代码段不更改cpl，因此此时代码段相当于运行在特权级3下，向ds加载数据段选择符时，加载了指向dpl=3的数据段选择符。同时由于特权级的问题，也不能直接从改代码段通过```jmp SelectorCode16:0```跳到16位代码段，需要通过retf返回到ring3，之后通过给出的调用门进行返回。

   ![image-20210924112709677](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210924112709677.png)

6. 实验效果

   ![image-20210924112003595](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210924112003595.png)

## 实验练习

1. GDT、Descriptor、Selector、GDTR结构，及其含义是什么？他们的关联关系如何？pm.inc所定义的宏怎么使用？

2. 从实模式到保护模式，关键步骤有哪些？为什么要关中断？为什么要打开A20地址线？从保护模式切换回实模式，又需要哪些步骤？

3. 解释不同权限代码的切换原理，call, jmp，retf使用场景如何，能够互换吗？

4. 课后动手改：

	1. 自定义添加1个GDT代码段、1个LDT代码段，GDT段内要对一个内存数据结构写入一段字符串，然后LDT段内代码段功能为读取并打印该GDT的内容；
	2. 自定义2个GDT代码段A、B，分属于不同特权级，功能自定义，要求实现A-->B的跳转，以及B-->A的跳转。


## FAQ
### Q: No Boot Device

这种情况大概率是因为在做chapter3/a时以a.img为启动盘，但是a.img结尾并没有55aa标志（可使用hexdump命令查看），请拷贝chapter1/a/目录下的a.img（确保具有55aa标志），然后重新将pmtest1.bin重新写入a.img，之后再次启动bochs。

### Q: wrong fs type, bad option, bad superblock on

请先将在freedos中将pm.img进行格式化，mount命令无法直接识别一个img文件，需要挂载可识别的文件系统。

### Q: format b: error

具体原因多种多样，请按照以下步骤排查：

1. 确保bochsrc中添加要求的三行内容，**并删去了之前的floppya : xxxx**。
2. 确保bximage新建了pm.img
3. 确保pm.img有写入权限
4. 确保pm.img的owner和group是当前用户而非root，否则用chown和chgrp命令进行修改。

### Q: how to debug in freedos

1. debug

   本指导提供的freedos-img.tar.gz中c.img中装了debug程序，可以使用debug来调试程序。修改配置文件如下

   ![image-20210917162205973](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917162205973.png)

   运行bochs，可以发现c盘的BIN目录下有debug程序，输入命令```path c:\bin```，进入b盘，输入命令```debug pmtest1.com```便可以进行调试。

2. 加死循环

   在程序开头加上```jmp $```令程序死循环，执行程序后，在bochs调试窗口按ctrl-c，之后修改eip寄存器的值指向程序入口。

3. magic number（推荐）

   在bochsrc中添加```magic_break: enabled=1```

   ![image-20210917162323976](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917162323976.png)

   在程序开头添加命令```xchg bx,bx```

   重新编译运行，可以发现当运行程序时bochs会在程序开头停止，现在可以进行调试了。

   ![image-20210917162706733](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20210917162706733.png)

   ```xchg bx, bx```的机器码为```87 DB```，不知出于什么原因bochsrc遇到87 DB会停止运行，因此你可以将xchg插入到任何你想要调试的地方，达到断点的效果。

### Q: bochs中键盘不响应

   等待或重新启动bochs。
