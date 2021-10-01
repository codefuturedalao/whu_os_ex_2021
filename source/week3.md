# week3

## 实验内容

1. 认真阅读章节资料，掌握什么是分页机制
2. 调试代码，掌握分页机制基本方法与思路
	* 代码3.22中，212行---237行，设置断点调试这几个循环，分析究竟在这里做了什么？
3. 掌握PDE，PTE的计算方法
	* 动手画一画这个映射图
	* 为什么代码3.22里面，PDE初始化添加了一个PageTblBase(Line 212)，而PTE初始化时候没有类似的基地址呢（Line224）？
4. 熟悉如何获取当前系统内存布局的方法
5. 掌握内存地址映射关系的切换
	* 画出流程图
6. 基础题：依据实验的代码，
	* 自定义一个函数，给定一个虚拟地址，能够返回该地址从虚拟地址到物理地址的计算过程，如果该地址不存在，则返回一个错误提示。
	* 完善分页管理功能，补充alloc_pages, free_pages两个函数功能
7. 进阶题（选做）
	* 设计一个内存管理器，选择其一实现：首次适应算法、最佳适应算法、伙伴算法，要求实现内存的分配与回收。（提示，均按照页为最小单位进行分配、对于空闲空间管理可采用位图法或者双向链表法管理）

## 实验步骤

```{admonition} 提示

课后实验思路仅供参考，故尽可能减少示例图片的出现和文字描述。鼓励更加完善的内存管理方式

```

### 虚拟地址到物理地址

#### 编写程序

![image-20211001083449615](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211001083449615.png)

程序可以大致分为以下几个部分：

* 从cr3寄存器获取页目录物理地址
* 通过掩码和右移得到线性地址高10位，左移两位（乘4）作为索引，寻找对应的页目录项，从页目录项中得到页表物理地址
* 通过掩码和右移得到线性地址中间10位，左移两位（乘4）作为索引，寻找对应的页表项，从页表项中得到页物理地址高20位
* 与低12位拼接，可得到对应物理地址

注：出于简化的目的，当页目录项或页表项不存在时，返回物理地址全为1，表示出错。

#### 编写测试代码

通过TestL2P函数来对代码进行测试，打印相关信息。

分别在PSwitch前后查看LinearAddrDemo对应的物理地址，查看是否发生变化

![image-20211001084158851](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211001084158851.png)

#### 调用测试函数，查看输出

![image-20211001084130800](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211001084130800.png)

### Alloc_pages函数

```{admonition} 注解

函数的重点是建立线性地址和物理地址的映射，不在于如何管理线性空间，因此简单的选取可用线性地址返回即可

```

#### 物理页管理

* 定义位图

  通过在数据段定义位图的形式，假设0~1MB物理内存均被占用，1MB~2MB物理内存均未分配

  ![image-20211001085007382](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211001085007382.png)

* 寻找空闲页并返回物理地址

  通过bts寻找位图中第一个0比特，将其位置乘以4KB得到物理地址，如果没有可用空间，直接停机（简化操作）

  ![image-20211001085214251](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211001085214251.png)

#### 虚拟页管理

* 假设线性地址从0x8000_0000开始分配

  ![image-20211001085357967](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211001085357967.png)

* 获取线性地址，然后查找对应页目录项，如果页目录项对应的页表不存在，则创建页表

  ![image-20211001092124162](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211001092124162.png)

* 修改对应页表项，将alloc_a_4k_page返回的物理地址和相应属性写入。

  ![image-20211001092143705](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211001092143705.png)

### Free_pages函数

给定线性地址和页大小（不用考虑是否越界的问题），修改对应页表项和页目录项，取消映射关系。

同学们自己coding⌨练练手

### 测试两个函数

![image-20211001092026739](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211001092026739.png)

* 查看alloc_pages前地址映射关系

  ![image-20211001090427737](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211001090427737.png)

* 查看alloc_pages后地址映射关系

  ![image-20211001090519049](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211001090519049.png)

* 查看free_pages后地址映射关系

  ![image-20211001090623863](https://sql-markdown-picture.oss-cn-beijing.aliyuncs.com/img/image-20211001090623863.png)

## FAQ

