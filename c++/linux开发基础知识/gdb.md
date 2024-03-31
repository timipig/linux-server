# Linux gdb

## 什么是gdb

参考网站：[分类: gdb | 守望的个人博客 (yanbinghu.com)](https://www.yanbinghu.com/categories/gdb/)

gdb是GNU开源组织发布的一个强大的Unix/Linux下的程序调试工具，通过shell操作，可以实现各类IDE类似的调试功能。

## gdb主要作用

- 启动程序，程序猿可以自定义地运行程序

- 让被调试的程序在指定的断点处停住，便于分析

- 当程序被停住时，可以检查此时程序中所发生的事

- 动态地改变程序的执行环境



## gdb的使用

### 源码查看

#### 进入gdb

```shell
g++ -g XXX.cpp -o main	#编译程序 需要加上-g的参数，程序才可以被调试
./main		#执行程序
gdb main 	#进入gdb
```

#### 打印源码

**直接打印源码**

`list`可以简写为`l`

直接输入l可从第一行开始显示源码，继续输入l，可列出后面的源码。后面也可以跟上+或者-，分别表示要列出上一次列出源码的后面部分或者前面部分。

**列出指定行附近源码**

`l 行号`

l后面可以跟行号，表明要列出附近的源码

```shell
(gdb) l 9
4       {
5           return a + b;
6       }
7
8       int main()
9       {
10          int sum[5]={0, 0, 0, 0, 0};
11          int arr1[5]={1, 2, 3, 4, 5};
12          int arr2[5]={1, 1, 1, 1, 1};
13
```

**列出指定函数附近的源码**

`l 函数名`

```shell
(gdb) l add
1       #include <iostream>
2
3       int add(int a , int b)
4       {
5           return a + b;
6       }
7
8       int main()
9       {
10          int sum[5]={0, 0, 0, 0, 0};
```

**设置源码一次列出行数**

`set listsize 20`

`set unlimited` 	列出就没有限制了，但源码如果较长，查看将会不便

```shell
(gdb) set listsize 20
(gdb) show listsize
Number of source lines gdb will list by default is 20.
```

**列出指定行之间的源码**

`list first,last`

first和list可以省略

`list first,` 或者 `list ,last`

**列出指定文件的源码**

前面执行l命令时，默认列出main.c的源码，想要查看指定文件的源码使用`location`

其中`location`可以是文件名加行号或函数名，也可以指定文件指定行之间

```shell
(gbd) l xxx.cpp:1
(gdb) l xxx.cpp:函数名
(gdb) l xxx.cpp:1,xxx.cpp:9
```

#### 指定源码路径

在查看源码之前，首先要确保我们的程序能够关联到源码，一般来说，我们在自己的机器上加上`-g`参数编译完之后，使用gdb都能查看到源码，但是会出现特殊情况。

**源码被移走**

可以使用`dir`命令指定源码路径

**更换源码目录**

使用`set substitute-path from to`将原来的路径替换为新的路径

使用`readelf`命令知道原来的源码路径`readelf main -p .debug_str`

```shell
readelf main -p .debug_str
  [     0]  long unsigned int
  [    12]  short int
  [    1c]  /home/hyb/workspaces/gdb/sourceCode
  [    40]  main.c
#（显示部分内容）
```

main为你将要调试的程序名，这里我们可以看到原来的路径，那么我们现在替换掉它：

```shell
(gdb) set substitute-path /home/hyb/workspaces/gdb/sourceCode /home/hyb/workspaces/gdb/sourceCode/temp
(gdb) show substitute-path
List of all source path substitution rules:
  `/home/hyb/workspaces/gdb/sourceCode' -> `/home/hyb/workspaces/gdb/sourceCode/temp'.
(gdb)
```

设置完成后，可以通过`show substitute-path`来查看设置结果。这样它也能在正确的路径查找源码啦。

需要注意的是，这里**对路径做了字符串替换**，那么如果你有多个路径，可以做多个替换。甚至可以对指定文件路径进行替换。

最后你也可以通过`unset substitute-path [path]`取消替换。

#### 编辑源码

为了避免已经启动了调试之后，需要编辑源码，又不想退出，可以直接在gdb模式下编辑源码，它默认使用的编辑器是/bin/ex，但是你的机器上可能没有这个编辑器，或者你想使用自己熟悉的编辑器，那么可以通过下面的方式进行设置：

```shell
EDITOR=/usr/bin/vim
export EDITOR
```

`/usr/bin/vim`可以替换为你熟悉的编辑器的路径，如果你不知道你的编辑器在什么位置，可借助`whereis`命令或者`witch`命令查看：

```shell
whereis vim
which vim
```

设置之后，就可以在gdb调试模式下进行编辑源码了，使用命令`edit location`，例如：

```shell
(gdb)edit 3		#编辑第三行
(gdb)edit 函数名
(gdb)edit 文件名:5		#编辑文件第五行
```

这里的location和前面介绍的一样，可以跟指定文件的特定行或指定文件的指定函数。 编辑完保存后，别忘了重新编译程序。

```shell
(gdb)shell g++ -g xxx.cpp -o main
```

这里要注意，为了在gdb调试模式下执行shell命令，需要在命令之前加上shell，表明这是一条shell命令。这样就能在不用退出GDB调试模式的情况下编译程序了。

#### 另一种模式

启动时，带上`tui(Text User Interface)`参数，会有意想不到的效果，它会将调试在多个文本窗口呈现：

```shell
gdb main -tui
```

或者使用ctrl + x + a 也可以呈现出来多个文本窗口，上面是源码，下面是命令

#### 退出gdb

```shell
(gdb)quit
```

### 启动调试

#### 哪类程序可被调试

##### gdb文件

对于C程序来说，需要在编译时加上`-g`参数，保留调试信息，否则不能使用GDB进行调试。
但如果不是自己编译的程序，并不知道是否带有`-g`参数，如何判断一个文件是否带有调试信息呢？

```shell
gdb helloword
Reading symbols from helloWorld...(no debugging symbols found)...done.
```

如果没有调试信息，会提示no debugging symbols found。
如果是下面的提示：

```shell
Reading symbols from helloWorld...done.
```

则可以进行调试

##### readelf查看段信息

`readelf -S 文件名|grep debug`

##### file查看strip状况

下面的情况也是不可调试的：

```shell
file main
main: (省略前面内容) stripped
```

如果最后是`stripped`，则说明该文件的符号表信息和调试信息已被去除，不能使用gdb调试。但是`not stripped`的情况并不能说明能够被调试。

#### 调试未运行程序

程序还未启动时，可有多种方式启动调试。

##### 调试启动无参程序

`gdb main`,输入`run`命令，即可运行程序。

##### 调试启动带参程序

假设有以下程序，启动时需要带参数：

```c
#include<stdio.h>
int main(int argc,char *argv[])
{
    if(1 >= argc)
    {
        printf("usage:hello name\n");
        return 0;
    }
    printf("Hello World %s!\n",argv[1]);
    return 0 ;
}
```

编译

```shell
gcc -g -o hello hello.c
```

需要设置参数启动调试

```shell
gdb hello
(gdb)run YYX
Starting program: /root/test/hello YYX
Hello World YYX!
[Inferior 1 (process 18264) exited normally]
Missing separate debuginfos, use: debuginfo-install glibc-2.17-326.el7_9.x86_64
(gdb)
```

只需要run的时候带上参数即可。
或者使用`set args`，然后在用run启动：

```shell
gdb hello
(gdb) set args YYX
(gdb) run
Starting program: /root/test/hello YYX
Hello World YYX!
[Inferior 1 (process 18271) exited normally]
Missing separate debuginfos, use: debuginfo-install glibc-2.17-326.el7_9.x86_64
(gdb)
```

##### 调试core文件

当程序core dump时，可能会产生core文件，它能够很大程序帮助我们定位问题。但前提是系统没有限制core文件的产生。可以使用命令`ulimit -c`查看：

```shell
ulimit -c
0
```

如果结果是0，意味着即便程序core dump了也不会有core文件留下。需要让core文件能够产生：

```shell
ulimit -c unlimied		#表示不限制core文件大小
ulimit -c 10			#设置最大大小，单位为块，一块默认为512字节
```

上面两种方式可选其一。第一种无限制，第二种指定最大产生的大小。
调试core文件也很简单：

```shell
gdb 程序文件名 core文件名
```

具体可参考《[linux常用命令-开发调试篇](https://www.yanbinghu.com/2018/09/26/61877.html)》gdb部分。

#### 调试已运行程序

首先使用`ps`命令找到id进程

```shell
ps -ef|grep 进程名
[root@localhost test]# ps -ef|grep hello
root     18284 17848  0 02:52 pts/0    00:00:00 grep --color=auto hello
```

此处17848为进程号

##### attach方式

```shell
gdb
(gdb) attach 17848
```

可能会有下面的错误提示：

```shell
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Operation not permitted.
```

解决方法，切换到root用户：
将/etc/sysctl.d/10-ptrace.conf中的

`kernel.yama.ptrace_scope = 1`修改为`kernel.yama.ptrace_scope = 0`

##### 直接调试相关id进程

使用`gdb program pid`方式

```shell
gdb hello 17848
#或者
gdb hello --pid 17848
#或者
gdb -pid id号
```

此时运行的进程处**于暂停状态，你可以选择打了断点后在输入c，继续continue**

##### 已运行程序没有调试信息

为了节省磁盘空间，已经运行的程序通常没有调试信息。但如果又不能停止当前程序重新启动调试，那怎么办呢？还有办法，那就是同样的代码，再编译出一个带调试信息的版本。然后使用和前面提到的方式操作。对于attach方式，在attach之前，使用file命令即可：

```shell
gdb
(gdb) file hello
Reading symbols from /root/test/hello...done.
(gdb) attach 17848
```

### 断点设置

#### 为何要设置断点

在介绍之前，我们首先需要了解，为什么需要设置断点。我们在指定位置设置断点之后，程序运行到该位置将会“暂停”，这个时候我们就可以对程序进行更多的操作，比如查看变量内容，堆栈情况等等，以帮助我们调试程序。

#### 查看已设置的断点

可以使用`info breakpoints`查看已设置断点

```shell
gdb hello
(gdb) info breakpoints
```

它将会列出所有已设置的断点，每一个断点都有一个标号，用来代表这个断点。

#### 设置断点

断点设置有多种方式，分别应用于不同的场景。示例程序如下：

```c
//test.c
#include<stdio.h>
void printNum(int a)
{
    printf("printNum\n");
    while(a > 0)
    {
        printf("%d\n",a);
        a--;
    }
}
void printNum2(int a,int num)
{
    printf("printNum\n");
    while(a > num && a>0)
    {
        printf("%d\n",a);
        a--;
    }
}
int div(int a,int b)
{
    printf("a=%d,b=%d\n",a,b);
    int temp = a/b;
    return temp;
}
int main(int argc,char *argv[])
{
    printNum2(12,5);
    printNum(10);
    div(10,0);
    return 0;
}
```

编译

```shell
gcc -g -o test test.c
```

##### 根据行号设置断点

```shell
(gdb) b 10 	#break 可简写为b
#或者
(gdb) b test.c:10
```

##### 根据函数名设置断点

```shell
(gdb) b printNum
```

##### 根据条件设置断点

假设程序某处发生崩溃，而崩溃的原因怀疑是某个地方出现了非期望的值，那么你就可以在这里断点观察，当出现该非法值时，程序断住。这个时候我们可以借助gdb来设置条件断点

```shell
(gdb) break test.c:23 if b==0
```

当在b等于0时，程序将会在第23行断住。
它和`condition`有着类似的作用，假设上面的断点号为1，那么：

```shell
(gdb) break test.c:23 if b==0
Breakpoint 1 at 0x400624: file test.c, line 23.
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000400624 in div at test.c:23
        stop only if b==0
(gdb) condition 1 b==1
(gdb) info breakpoints
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000400624 in div at test.c:23
        stop only if b==1
(gdb)
```

会使得b等于0时，产生断点1。而实际上可以很方便地用来改变断点产生的条件，例如，之前设置b==0时产生该断点，那么**使用condition可以修改断点产生的条件**。

##### 根据规则设置断点

例如需要对所有调用printNum函数都设置断点，可以使用下面的方式：

```shell
(gdb) rbreak printNum*
```

所有以printNum开头的函数都设置了断点。而下面是对所有函数设置断点：

```shell
#用法: rbreak file:regex
(gdb) rbreak .
(gdb) rbreak test.c:.			#对test.c中的所有函数设置断点
(gdb) rbreak test.c:^print	#对以print开头的函数设置断点
```

##### 设置临时断点

假设某处的断点只想生效一次，那么可以设置临时断点，这样断点后面就不复存在了：

```shell
(gdb) tbreak test.c:10		#在第十行设置临时断点
```

##### 跳过多次设置的断点

假如有某个地方，我们知道可能出错，但是前面30次都没有问题，虽然在该处设置了断点，但是想跳过前面30次，可以使用下面的方式：

```shell
(gdb) ignore 1 30
```

其中，1是要忽略的断点号，可以通过前面的方式查找到，30是需要跳过的次数。这样设置之后，会跳过前面30次。再次通过info breakpoints可以看到：

```shell
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x00000000004005e8 in printNum2 at test.c:16
    ignore next 30 hits
```

##### 根据表达式值变化产生断点

有时候需要观察某个值或表达式，知道它什么时候发生变化了，这个时候我们可以借助watch命令。

```shell
(gdb) watch a
```

这个时候，让程序继续运行，如果a的值发生变化，则会打印相关内容，如：

```shell
Hardware watchpoint 2: a
Old value = 12
New value = 11
```

但是这里要特别注意的是，程序必须运行起来，否则会出现：

```shell
No symbol "a" in current context.
```

因为程序没有运行，当前上下文也就没有相关变量信息。

`rwatch`和`awatch`同样可以设置观察点前者是当变量值被读时断住，后者是被读或者被改写时断住。

#### 禁用或启动断点

有些断点暂时不想使用，但又不想删除，可以暂时禁用或启用。

```shell
#bnum为breakpoint Num缩写
(gdb) disable	#禁用所有断点
(gdb) disable bnum 	#禁用标号为bnum的断点
(gdb) enable	#启用所有断点
(gdb) enable bnum	#启用标号为bnum的断点
(gdb) enable delete bnum	#启动标号为bnum的断点，并且在之后删除该断点
```

#### 断点清除

断点清除主要用到clear和delete命令。

```shell
(gdb) clear   #删除当前行所有breakpoints
(gdb) clear function  #删除函数名为function处的断点
(gdb) clear filename:function #删除文件filename中函数function处的断点
(gdb) clear lineNum #删除行号为lineNum处的断点
(gdb) clear f:lename：lineNum #删除文件filename中行号为lineNum处的断点
(gdb) delete  #删除所有breakpoints,watchpoints和catchpoints
(gdb) delete bnum #删除断点号为bnum的断点
```

### 变量查看

GDB调试最大的目的之一就是走查代码，查看运行结果是否符合预期。既然如此，我们就不得不了解一些查看各种类型变量的方法，以帮助我们进一步定位问题。

辅助代码如下：

```c
//testGdb.c
#include<stdio.h>
#include<stdlib.h>
#include"testGdb.h"
int main(void)
{
    int a = 10; //整型
	int b[] = {1,2,3,5};  //数组
    char c[] = "hello,shouwang";//字符数组
	/*申请内存，失败时退出*/    
	int *d = (int*)malloc(a*sizeof(int));
	if(NULL == d)
	{
		printf("malloc error\n");
		return -1;
	}
	/*赋值*/
    for(int i=0; i < 10;i++)
	{
		d[i] = i;
	}
	free(d);
	d = NULL;
	float e = 8.5f;
	return 0;
}
```

```c
//testGdb.h
int a = 11;
```

编译：

```shell
gcc -g -o testGdb testGdb.o
```

#### 查看变量

##### 打印基本类型变量、数组、字符数组

最常见的使用便是使用print（可简写为p）打印变量内容。

```shell
(gdb) p a
$1 = 10
(gdb) p b
$2 = {1, 2, 3, 5}
(gdb) p c
$3 = "hello,shouwang"
(gdb)
```

当然有时候，多个函数或者多个文件会有同一个变量名，这个时候可以在前面加上文件名或者函数名来区分：

```shell
(gdb) p 'testGdb.h'::a
$1 = 11
(gdb) p 'main'::b
$2 = {1, 2, 3, 5}
(gdb)
```

##### 打印指针指向内容

使用上面的方式打印指针指向的内容，打印出来的只是指针地址而已

```shell
(gdb) p d
$1 = (int *) 0x602010
(gdb)
```

如果想要打印指针指向的内容，需要解引用：

```shell
(gdb) p *d
$2 = 0
(gdb) p *d@10
$3 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
(gdb)
```

从上面可以看到，仅仅使用*只能打印第一个值，如果要打印多个值，后面跟上@并加上要打印的长度。
或者@后面跟上变量值：

```shell
(gdb) p *d@a
$2 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
(gdb)
```

由于a的值为10，并且是作为整型指针数据长度，因此后面可以直接跟着a，也可以打印出所有内容。

另外值得一提的是，$可表示上一个变量，而假设此时有一个链表linkNode，它有next成员代表下一个节点，则可使用下面方式不断打印链表内容：

```shell
(gdb) p *linkNode
#(这里显示linkNode节点内容)
(gdb) p *$.next
#(这里显示linkNode节点下一个节点的内容)
```

如果想要查看前面数组的内容，可以将下标一个一个累加，还可以定义一个类似UNIX环境变量，例如：

```shell
(gdb) set $index=0
(gdb) p b[$index++]
$11 = 1
(gdb) p b[$index++]
$12 = 2
(gdb) p b[$index++]
$13 = 3
```

这样就不需要每次修改下标去打印啦。

#### 按照特定格式打印变量

对于简单的数据，print默认的打印方式已经足够了，它会根据变量类型的格式打印出来，但是有时候这还不够，我们需要更多的格式控制。常见格式控制字符如下：

- x 按十六进制格式显示变量。
- d 按十进制格式显示变量。
- u 按十六进制格式显示无符号整型。
- o 按八进制格式显示变量。
- t 按二进制格式显示变量。
- a 按十六进制格式显示变量。
- c 按字符格式显示变量。
- f 按浮点数格式显示变量。

还是以辅助程序来说明，正常方式打印字符数组c：

```shell
(gdb) p c
$18 = "hello,shouwang"
```

但是如果我们要查看它的十六进制格式打印呢？

```shell
(gdb) p/x c
$19 = {0x68, 0x65, 0x6c, 0x6c, 0x6f, 0x2c, 0x73, 0x68, 0x6f, 0x75, 0x77, 0x61, 
  0x6e, 0x67, 0x0}
(gdb)
```

但是如果我们想用这种方式查看浮点数的二进制格式是怎样的是不行的，因为直接打印它首先会被转换成整型，因此最终会得到8：

```shell
(gdb) p e
$1 = 8.5
(gdb) p/t e
$2 = 1000
(gdb)
```

那么就需要另外一种查看方式了。

#### 查看内存内容

examine(简写为x)可以用来查看内存地址中的值。语法如下：

```shell
x/[n][f][u] addr
```

其中：

- n 表示要显示的内存单元数，默认值为1
- f 表示要打印的格式，前面已经提到了格式控制字符
- u 要打印的单元长度
- addr 内存地址

单元类型常见有如下：

- b 字节
- h 半字，即双字节
- w 字，即四字节
- g 八字节

我们通过一个实例来看，假如我们要把float变量e按照二进制方式打印，并且打印单位是一字节：

```shell
(gdb) x/4tb &e
0x7fffffffdbd4:	00000000	00000000	00001000	01000001
(gdb)
```

可以看到，变量e的四个字节都以二进制的方式打印出来了。

#### 自动显示变量内容

假设我们希望程序断住时，就显示某个变量的值，可以使用display命令。

```shell
(gdb) display e
1: e = 8.5
```

那么每次程序断住时，就会打印e的值。要查看哪些变量被设置了display，可以使用：

```shell
(gdb) info display
Auto-display expressions now in effect:
Num Enb Expression
1:   y  b
2:   y  e
```

如果想要清除可以使用

```shell
delete display num #num为前面变量前的编号,不带num时清除所有。
```

或者去使能：

```shell
disable display num  #num为前面变量前的编号，不带num时去使能所有
```

#### 查看寄存器内容

```shell
(gdb)info registers
rax            0x0	0
rbx            0x0	0
rcx            0x7ffff7dd1b00	140737351850752
rdx            0x0	0
rsi            0x7ffff7dd1b30	140737351850800
rdi            0xffffffff	4294967295
rbp            0x7fffffffdc10	0x7fffffffdc10
#(内容过多未显示完全)
```

### 单步调试

#### 数据准备

```c++
/*gdbStep.c*/
#include<stdio.h>
/*计算简单乘法,这里没有考虑溢出*/
int add(int a, int b)
{
    int c = a + b;
    return c;
}
/*打印从0到num-1的数*/
int count(int num)
{
    int i = 0;
    if(0 > num)
        return 0;
    while(i < num)
    {
        printf("%d\n",i);
        i++;
    }
    return i;
}
int main(void)
{
    int a = 3;
    int b = 7;
    printf("it will calc a + b\n");
    int c = add(a,b);
    printf("%d + %d = %d\n",a,b,c);
    count(c);
    return 0;
}
```

编译

```shell
gcc -g -o gdbStep gdbStep.c
```

#### 单步执行-next

next命令（可简写为n）用于在程序断住后，继续执行下一条语句，假设已经启动调试，并在第25行停住，如果要继续执行，则使用n执行下一条语句，如果后面跟上数字num，则表示执行该命令num次，就达到继续执行n行的效果了：

```shell
gdb gdbStep		#启动调试
(gdb)b 25		#将断点设置在25行
(gdb)run		#运行程序
Starting program: /root/gdb/gdbStep
Breakpoint 1, main () at gdbStep.c:25
25          int b = 7;
(gdb)n			#单步执行
26          printf("it will calc a + b\n");
(gdb)n 2		#执行两次
it will calc a + b
28          printf("%d + %d = %d\n",a,b,c);
(gdb)
```

从上面的执行结果可以看到，在25行处断住，执行n之后，运行到26行，运行n 2之后，运行到28行，但是没有进入add函数内部。

#### 单步进入-step

可以使用step命令（可简写为s），它可以单步跟踪到函数内部，但前提是该函数有调试信息并且有源码信息。

```shell
gdb gdbStep    	#启动调试
(gdb) b 25      #在25行设置断点
Breakpoint 1 at 0x4005d3: file gdbStep.c, line 25.
(gdb) run       #运行程序
Breakpoint 1, main () at gdbStep.c:25
25	    int b = 7;
(gdb) s          
26	    printf("it will calc a + b\n");
(gdb) s     #单步进入，但是并没有该函数的源文件信息
_IO_puts (str=0x4006b8 "it will calc a + b") at ioputs.c:33
33	ioputs.c: No such file or directory.
(gdb) finish    #继续完成该函数调用
Run till exit from #0  _IO_puts (str=0x4006b8 "it will calc a + b")
    at ioputs.c:33
it will calc a + b
main () at gdbStep.c:27
27	    int c = add(a,b);
Value returned is $1 = 19
(gdb) s        #单步进入，现在已经进入到了add函数内部
add (a=13, b=57) at gdbStep.c:6
6	    int c = a + b;
```

s命令会尝试进入函数，但是如果没有该函数源码，需要跳过该函数执行，可使用**finish命令**，继续后面的执行。如果没有函数调用，s的作用与n的作用并无差别，仅仅是继续执行下一行。它后面也可以跟数字，表明要执行的次数。

其次它还有一个选项，用来设置当遇到没有调试信息的函数，s命令是否跳过该函数，而执行后面的。默认情况下，它是会跳过的，即step-mode值是off：

```shell
(gdb) show step-mode
Mode of the step operation is off.
(gdb) set step-mode on
(gdb) set step-mode off
```

还有一个与step相关的命令是stepi（可简写为si），它与step不同的是，**每次执行一条机器指令**：

```shell
(gdb) si
0x000000000040058a      6           int c = a + b;
(gdb) display/i $pc
1: x/i $pc
=> 0x40058a <add+13>:   mov    -0x14(%rbp),%edx
(gdb)
```

#### 继续执行到下一个断点-continue

可能打了多处断点，或者断点打在循环内，这个时候，想跳过这个断点，甚至跳过多次断点继续执行该怎么做呢？可以使用continue命令（可简写为c）或者fg，它会继续执行程序，直到再次遇到断点处：

```shell
gdb gdbStep
(gdb)b 18    #在count函数循环内打断点
(gdb)run
Breakpoint 1, count (num=10) at gdbStep.c:18
18	        i++;
(gdb) c      #继续运行，直到下一次断住
Continuing.
1

Breakpoint 1, count (num=10) at gdbStep.c:18
18	        i++;
(gdb) fg     #继续运行，直到下一次断住
Continuing.
2

Breakpoint 1, count (num=10) at gdbStep.c:18
18	        i++;
(gdb) c 3    #跳过三次
Will ignore next 2 crossings of breakpoint 1.  Continuing.
3
4
5

Breakpoint 1, count (num=10) at gdbStep.c:18
18	        i++;
```

#### 继续运行到指定位置-until

假如在25行停住了，现在想要运行到29行停住，就可以使用until命令（可简写为u）：

```shell
gdb gdbStep
(gdb)b 25
(gdb)run
(gdb)u 29
it will calc a + b
3 + 7 = 10
main () at gdbStep.c:29
29	    count(c);
(gdb)
```

在执行u 29之后，它在29行停住了。它利用的是**临时断点**。

#### 跳过执行-skip

skip可以在step时跳过一些不想关注的函数或者某个文件的代码：

```shell
(gdb) b 27
Breakpoint 1 at 0x4005fd: file gdbStep.c, line 27.
(gdb) skip function add		#step时跳过add函数
Function add will be skipped when stepping.
(gdb) info skip		#查看step情况
Num     Type           Enb What
1       function       y   add
(gdb) run
Starting program: /root/gdb/gdbStep
it will calc a + b

Breakpoint 1, main () at gdbStep.c:27
27          int c = add(a,b);
Missing separate debuginfos, use: debuginfo-install glibc-2.17-326.el7_9.x86_64
(gdb) s
28          printf("%d + %d = %d\n",a,b,c);
(gdb)
```

可以看到，再使用skip之后，使用step将不会进入add函数。
step也后面也可以跟文件：

```shell
(gdb)skip file gdbStep.c
```

这样gdbStep.c中的函数都不会进入。

其他相关命令：

- `skip delete [num] `删除skip
- `skip enable [num]`使能skip（负责控制信号的输入和输出）
- `skip diable [num]`去使能skip

其中num是前面通过info skip看到的num值，上面可以带或不带该值，如果不带num，则针对所有skip，如果带上了，则只针对某一个skip。



## gdb远程调试

一般适用与在嵌入式环境中适用，gdb不仅可以调试本地上的程序，同时还可以使用gdb server来调试远程主机上的程序

前提是可执行文件，在远程一定能够执行也就是说远程的可执行文件一定要有全部附近依赖库，且带有调试信息，然后本地环境一定要有源码，这样才能进行远程调试

### gdb server

#### 1.安装gdbserver

使用Linux远程调试目标机器上面的程序时，目标机器需要安装gdbserver，启动测试程序后与本地gdb进行通讯
Ubuntu下安装命令如下：

```
sduo apt-get install gdbserver
```


CentOS安装方式：

```
sudo yum install gdb-server
```



#### 2 配置防火墙

为了能够顺利连接，可以暂时关闭防火墙。



### gdb server的启动与连接

#### 1.gdb server启动

在目标机器上启动gdbserver：
localhost就是本地ip
4242是未被占用的端口，可以自由指定
app.out就是可执行程序名

```
gdbserver localhost:4242 app.out
```

启动成功之后就会监听对应端口，等待连接

```
$> gdbserver localhost:4242 app.out 
Process /home/lhq/Desktop/EOS/14-c-asm/app.out created; pid = 31633
Listening on port 4242
```

如果程序正在运行可以通过查找对应进程的进程号后输入如下指令进行连接：

```
sudo gdbserver localhost:4242 --attach PID

```

```
$> sudo gdbserver localhost:4242 --attach 33201
[sudo] lhq 的密码： 
Attached; pid = 33201
Listening on port 4242
```

#### 2.gdb连接

服务器监听之后，客户端便可以发起连接请求了，操作如下：

```
$> gdb 
GNU gdb (Ubuntu 9.2-0ubuntu1~20.04) 9.2
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".


(gdb) target remote localhost:4242
Remote debugging using localhost:4242
Reading /home/lhq/Desktop/EOS/14-c-asm/app.out from remote target...
warning: File transfers from remote targets can be slow. Use "set sysroot" to access files locally instead.
Reading /home/lhq/Desktop/EOS/14-c-asm/app.out from remote target...
Reading symbols from target:/home/lhq/Desktop/EOS/14-c-asm/app.out...
0x08049000 in _start ()
```

gdb 启动之后输入指令：target remote 目标主机IP:端口
此处是本地演示，连接到了本地端口，连接之后就可以进行进行调试了，和本地效果完全一致。



服务端当前状态

```
$> gdbserver localhost:4242 app.out         
Process /home/lhq/Desktop/EOS/14-c-asm/app.out created; pid = 32189
Listening on port 4242
Remote debugging from host 127.0.0.1, port 40230（新增）

```

客户端：

```
(gdb) b main.c:c_func 
Breakpoint 1 at 0x8049049: file main.c, line 8.
(gdb) c
Continuing.

Breakpoint 1, c_func () at main.c:8
8       {
(gdb) q
A debugging session is active.

```

