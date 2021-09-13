## Introduction

nasm是一个x86架构的汇编器，支持一系列目标文件格式，如a.out、elf、obj、COFF等。

## Running Nasm

如果想要汇编一个文件，命令格式如下：

```shell
nasm -f <format> <filename> [-o <output>]
```

比如想要将myfile.asm汇编为ELF格式的目标文件，可以使用以下命令：

```shell
nasm -f elf myfile.asm -o myfile.elf
```

当不指定目标文件类型时，默认生成bin格式的目标文件。因此可以简单的使用```nasm boot.asm -o boot.bin```命令，即可生成boot.bin文件。（其实还可以更简单，当不指定目标文件名称时，nasm会去掉源文件名的后缀作为目标文件名，即生成boot文件）

### -i选项：包含文件搜索目录

当NASM看到````%include```或```%pathsearch```命令时，除了会在当前目录下搜索，也会在-i选项指定的目录下搜索文件，因此可以包含一些文件作为库文件。例如：

```shell
nasm -ic:\macrolib\ -f obj myfile.asm
```

### -p选项：预包含一个文件

使用-p选项，等同于将%include命令放置在文件头

例如：

```shell
nasm myfile.asm -p myinc.inc
```

等同于```nams myfile.asm```，同时将```%include "myinc.inc"```放在文件最开始

### -d选项：预定义宏

使用-d选项，等同于将%define命令放置在文件头

### NASM的几个要点

1. 大小写敏感

2. 访问内存中数据时需要加上方括号，否则统一视为地址

3. 不存储变量类型

   有些汇编器将变量名加冒号视为标签，不加冒号视为变量，同时并自动记忆变量类型，对于这种类型的汇编器，一下汇编指令是正确的

   ```assembly
   var dw 0
   ....
   mov var, 2
   ```

   但是nasm中只记忆var的地址信息，并不记忆其类型，因此需要显式地指出其类型，如```mov word [var], 2```。

4. 没有assume命令

5. 不支持内存模型

   代码编写人员需要自己明确什么时候使用far call，是么使用使用near call，同时正确使用ret和retf指令。

## NASM Language

通常nasm每一行包含以下要素

```assembly
label:    instruction operands        ; comment
```

通常来说，上面的域均为可选的，有时候可以没有标签和评论，对于一些指令也不需要有操作数。

在nasm，标签后面的冒号是可选的，如果你不小心将lodsb指令写为lodab，nasm会将其视为一个标签而不会报错。

nasm中标识符（即变量名）可以由字母，数字，`_`， `$`， `#`， `@`， `~`， `.` 和`?`组成，只有数字、.、下划线和问好可以作为首字母，标识符最大长度为4095个字符。

### 伪指令

伪指令并不是真正的x86机器指令，而是为了方便使用由汇编器自定义的指令。

nasm中的伪指令包括`DB`, `DW`, `DD`, `DQ`, `DT`, `DO`, `DY` 和 `DZ`; `RESB`, `RESW`, `RESD`, `RESQ`, `REST`, `RESO`, `RESY`和`RESZ`; `INCBIN，EQU`和 `TIMES` 。

#### Dx：声明初始化的数据

他们可以用以下方式使用

```assembly
db    0x55                ; just the byte 0x55 
      db    0x55,0x56,0x57      ; three bytes in succession 
      db    'a',0x55            ; character constants are OK 
      db    'hello',13,10,'$'   ; so are string constants 
      dw    0x1234              ; 0x34 0x12 
      dw    'a'                 ; 0x61 0x00 (it's just a number) 
      dw    'ab'                ; 0x61 0x62 (character constant) 
      dw    'abc'               ; 0x61 0x62 0x63 0x00 (string) 
      dd    0x12345678          ; 0x78 0x56 0x34 0x12 
      dd    1.234567e20         ; floating-point constant 
      dq    0x123456789abcdef0  ; eight byte constant 
      dq    1.234567e20         ; double-precision float 
      dt    1.234567e20         ; extended-precision float
```

#### RESx：声明未初始化的数据

RESx命令用于BSS section中，声明未初始化的存储空间，例子如下：

```assembly
buffer:         resb    64              ; reserve 64 bytes 
wordvar:        resw    1               ; reserve a word 
realarray       resq    10              ; array of ten reals 
ymmval:         resy    1               ; one YMM register 
zmmvals:        resz    32              ; 32 ZMM registers
```

nasm 2.15版本后开始支持masm的语法？和DUP，因此可以用一下命令实现同样的效果

```assembly
buffer:         db      64 dup (?)      ; reserve 64 bytes 
wordvar:        dw      ?               ; reserve a word 
realarray       dq      10 dup (?)      ; array of ten reals 
ymmval:         dy      ?               ; one YMM register 
zmmvals:        dz      32 dup (?)      ; 32 ZMM registers
```

#### INCBIN：包含外部二进制文件

#### EQU：定义常量

EQU给指定的符号赋值常量，例子：

```assembly
message         db      'hello, world' 
msglen          equ     $-message
```

#### TIMES：重复指令或数据

指令格式为```times expression data/inst```，例子如下

```assembly
zerobuf:        times 64 db 0


buffer: db      'hello, world' 
        times 64-$+buffer db ' '
        
times 100 movsb
```

### 有效地址（effective addresses）

effective address是指令中指向内存的操作数，在nasm中，有效地址被放在方括号内。例子如下：

```assembly
wordvar dw      123 
        mov     ax,[wordvar] 
        mov     ax,[wordvar+1] 
        mov     ax,[es:wordvar+bx]
```

### 常量（Constant）

#### 数值常量

通过加后缀H/X,D/T,Q/O,B/Y来表示十六进制、十进制、八进制和二进制。或者可以通过加0x/0h前缀来表示十六进制、0d/ot表示十进制，0o/0q表示八进制以及0b/0y表示二进制。例子如下：

```assembly
 mov     ax,200          ; decimal 
        mov     ax,0200         ; still decimal 
        mov     ax,0200d        ; explicitly decimal 
        mov     ax,0d200        ; also decimal 
        mov     ax,0c8h         ; hex 
        mov     ax,$0c8         ; hex again: the 0 is required 
        mov     ax,0xc8         ; hex yet again 
        mov     ax,0hc8         ; still hex 
        mov     ax,310q         ; octal 
        mov     ax,310o         ; octal again 
        mov     ax,0o310        ; octal yet again 
        mov     ax,0q310        ; octal yet again 
        mov     ax,11001000b    ; binary 
        mov     ax,1100_1000b   ; same binary constant 
        mov     ax,1100_1000y   ; same binary constant once more 
        mov     ax,0b1100_1000  ; same binary constant yet again 
        mov     ax,0y1100_1000  ; same binary constant yet again
```

#### 字符常量

在nasm中单引号和双引号没有区别

### SEG和WRT

当程序具有多个段时，可以用seg来获取标签的段地址，标签本身代表其偏移地址。

当多个段出现重叠时，wrt用于获取标签相对指定段的偏移地址。

### Critical Expressions

### 局部标签

以点号开头的标签为局部标签，他和之前非局部标签联系在一起。

## The Nasm Preprocessor

#### Single-Line Macros

以%开头，字符\表示多行

可以重载有参数的宏，不能重载无参数的宏，大小写敏感，具有防递归调用的机制

=号用于计算值

&号用于转换为字符串

+号用于可变参数

%idefine不区分大小写



#### Multi-line Macros：%macro

```assembly
%macro  prologue 1 

        push    ebp 
        mov     ebp,esp 
        sub     esp,%1 

%endmacro
```

宏名后面跟参数的数量，使用%加数字来引用参数