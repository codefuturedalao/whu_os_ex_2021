# week7

## 实验内容

1. 掌握进程相关数据结构的定义方法：
   进程控制块(进程表)、进程结构体、进程相关的GDT/LDT、进程相关的TSS，以及数据结构的关系
2. 掌握构造进程的关键技术：
   初始化进程控制块的过程、初始化GDT和TSS、实现进程的启动
3. 进程的现场保护与切换，弄清楚需要哪些关键数据结构与步骤
   时钟中断与进程调度关系，现场保护与恢复机理，从ring0-->ring1的上下文切换方法，中断重入机理

## 实验问题

1. 描述进程数据结构的定义与含义：
   进程控制块(进程表)、进程结构体、进程相关的GDT/LDT、进程相关的TSS，画出数据结构的关系图
2. 画出以下关键技术的流程图：
   初始化进程控制块的过程、初始化GDT和TSS、实现进程的启动
3. 怎么实现进程的现场保护与恢复？
4. 为什么需要从ring0-->ring1，怎么实现？
5. 进程为什么要中断重入，具体怎么实现，画出流程图？

## 实验步骤

![diy](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/diy.png)

本次实验没有太多实操，重在理解，这里给大家更新一个小的脚本，免去每次都需要修改Makefile来增加"-fno-stack-protector"以及更新bochsrc。

```shell
#!/bin/sh

# Author : jackson sang
# another toy script

if test "$#" -eq 0;
then
	for file in `ls .`
	do
		if test -d $file
		then
			echo "deal with dir "$file
			sed -i 's/-c -fno-builtin/-c -fno-stack-protector -fno-builtin/' $file"/Makefile"
			cp bochsrc $file"/bochsrc"
			cd $file
			ctags -R . #optional
			cd ../
		fi

	done
else
	if test -d $1
	then
		echo "deal with dir "$1
		sed -i 's/-c -fno-builtin/-c -fno-stack-protector -fno-builtin/' $1"/Makefile"
		cp bochsrc $1"/bochsrc"
		cd $1
		ctags -R . #optional
		make image	#may be
		bochs
		cd ../	
	fi
fi
```

使用方法为将想要替换的bochsrc和shell脚本放在chapterX文件夹下

![image-20211101152535848](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211101152535848.png)

然后直接运行```./update.sh```进行所有目录下Makefile修改和bochsrc的替换，或者```./update xxx```单独对某个文件下的Makefile修改以及bochsrc的替换，同时执行```make image```和```bochs```命令启动bochs。

## FAQ

