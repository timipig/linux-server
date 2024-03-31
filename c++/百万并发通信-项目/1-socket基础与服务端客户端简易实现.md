## Socket建立简单tcp客户端和服务端

![image-20231230170302286](E:\note\linux-server\picture\image-20231230170302286.png)

客户端和服务端主要实现上面的几种功能

<font color='red'>注意 windows参考unix套接字的设计，所以window和Linux的套接字差距很小，函数名和参数含义都相同，可能仅有返回值不同</font>

### 初始化准备

1.在使用windows socket的时候需要 这相当于是一个启动的示例

使用windows socket需要链接ws2_32.lib

```c++
    WORD ver = MAKEWORD(2, 2);
    WSADATA dat;
    WSAStartup(ver, &dat);
	//
	//具体代码写到中间
	//
    WSACleanup();
    return 0;
```

2.在Linux中使用套接字无需初始化上下文

### linux socket公共API

#### 建立socket

```c++
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
/*
成功时返回文件描述符，失败时返回-1
domain: 套接字中使用的协议族（Protocol Family）
type: 套接字数据传输的类型信息
protocol: 计算机间通信中使用的具体协议信息
*/
```

**1.协议族**

通过 socket 函数的第一个参数传递套接字中使用的协议分类信息。此协议分类信息称为协议族，可分成如下几类

>   头文件 sys/socket.h 中声明的协议族

| 名称      | 协议族               |
| --------- | -------------------- |
| PF_INET   | IPV4 互联网协议族    |
| PF_INET6  | IPV6 互联网协议族    |
| PF_LOCAL  | 本地通信 Unix 协议族 |
| PF_PACKET | 底层套接字的协议族   |
| PF_IPX    | IPX Novel 协议族     |

套接字中采用的最终的协议信息是通过 socket 函数的第三个参数传递的。在指定的协议族范围内通过第一个参数和第二个参数决定第三个参数（如选择ipv4协议族加上面向连接的方式，则第三个参数可以选择传入0代表tcp协议）

**2.套接字类型**

套接字类型指的是套接字数据的传输方式

- 面向连接的套接字 SOCK_STREAM（建议去看看tcp ip协议） 

> 特点：
>
> 过程中数据不会消失
> 按序传输数据
> 传输的数据不存在数据边界（Boundary） 
>
> 数据被缓存在套接字的内部，
>
> 如：传输数据的计算机通过调用3次 write 函数传递了 100 字节的数据，但是接受数据的计算机仅仅通过调用 1 次 read 函数调用就接受了全部 100 个字节。 
>
> **可靠地、按序传递的、基于字节的面向连接的数据传输方式的套接字。**  

- 面向消息的套接字 SOCK_DGRAM

>   特点：不可靠的、不按序传递的、以数据的高速传输为目的套接字。  

#### 网络地址与转换

ip地址代表着计算机在网络中的唯一地址，端口则是为了区分同一计算机上面的不同进程，tcp与udp可以使用同一端口号，但是相同的协议不能使用同一端口号

1. **表示ipv4的结构体**

   ```c++
   struct sockaddr_in
   {
   	sa_family_t sin_family; //地址族（Address Family）
   	uint16_t sin_port; //16 位 TCP/UDP 端口号
   	struct in_addr sin_addr; //32位 IP 地址
   	char sin_zero[8]; //不使用
   };
   
   //in_addr结构体， 表明in_addr是32位的
   typedef uint32_t in_addr_t;
   struct in_addr
     {
       in_addr_t s_addr;
     };
   
   ```

   > 成员分析：
   >
   > - 地址族
   >
   > | 地址族（Address Family） | 含义                               |
   > | ------------------------ | ---------------------------------- |
   > | AF_INET                  | IPV4用的地址族                     |
   > | AF_INET6                 | IPV6用的地址族                     |
   > | AF_LOCAL                 | 本地通信中采用的 Unix 协议的地址族 |
   >
   > - 成员 sin_port
   >
   >   该成员保存 16 位端口号，重点在于，它以网络字节序保存。
   >
   > - 成员 sin_addr
   >
   >   该成员保存 32 为IP地址信息，且也以网络字节序保存
   >
   > - 成员 sin_zero
   >
   >   无特殊含义。只是为结构体 sockaddr_in 结构体变量地址值将以如下方式传递给 bind 函数。 

虽然我们常用的结构体是sockaddr_in，但是最后bind传入的参数依然是

sockaddr类型的地址

<font color='red'>**为什么我们要这么做？**</font>

具体原因我们可以通过查看源码关于sockaddr 结构体的定义

```c++
struct sockaddr
{
sa_family_t sin_family; //地址族
char sa_data[14]; //地址信息
}
```

从上面的结构体我们可以很清晰的看到该结构体将ip地址 端口都用一个字符数组来输入显然对于用户操作起来很麻烦，所以我们最终选择构建ip地址 端口的时候使用的是sockaddr_in结构体，然后在bind的时候将其类型转化为sockaddr结构体来使用，因为两个结构体在内存的存放是一致的。

2. **网络字节序与地址序的转换**

首先我们需要了解不同的cpu中，数据在内存保持方式不同

![image-20231231154436227](E:\note\linux-server\picture\image-20231231154436227.png)



> 如何区分本机是大端还是小端？
>
> ```c++
> int a = 0x12345678;
> char* b;
> b = (char *)&a;
> //b[0]是低地址
> //b[4]是高地址
>     if (b[0] == 0x12)
>         cout << "mini" << endl;
>     else if (b[0] == 0x78)
>         cout << "big" << endl;
> ```



自然也引发了一种问题就是两台主机一个采用不同的方式来解析数据，显然是有问题的，所以再进行网络通信的时候应该确定同一的方式，在网络中传输信息都是从低地址开始传输。

如用户A大端序传输a = 0x1234，低地址为0x12, 高地址为0x34，先发送 0x12, 再发送0x34，

用户B小端序，接受的0x12在低地址，0x34在高地址，组合起来就变成了0x3412



**相关API**

**主机序到网络序序的转换**

```
unsigned short htons(unsigned short);//适用与端口号
unsigned short ntohs(unsigned short);
unsigned long htonl(unsigned long);//适用与ip地址
unsigned long ntohl(unsigned long);
通过函数名称掌握其功能，只需要了解：
htons 的 h 代表主机（host）字节序。
htons 的 n 代表网络（network）字节序。
s 代表 short - 2字节
l 代表 long - 4字节
```

**字符串ip地址转换为sockaddr_in中需要的整型**

```c++
#include <arpa/inet.h>
in_addr_t inet_addr(const char *string);
//错误则返回INADDR_NONE,该函数还能检测输入的ip string的有效性


#include <arpa/inet.h>
int inet_aton(const char *string, struct in_addr *addr);
/*
成功时返回 1 ，失败时返回 0
string: 含有需要转换的IP地址信息的字符串地址值
addr: 将保存转换结果的 in_addr 结构体变量的地址值
*/

//inet_aton函数将更方便，因为它将转化之后的结果直接保存进了 in_addr 结构体中，也就是sockaddr_in中存放ipv4地址的函数

#include <arpa/inet.h>
char *inet_ntoa(struct in_addr adr);
/*
该函数将通过参数传入的整数型IP地址（需要网络序的ip地址哟）转换为字符串格式并返回。
但要小心，返回值为 char 指针，返回字符串地址意味着字符串已经保存在内存空间，但是该函数未向程序员要求分配内存，而是在内部申请了内存保存了字符串。也就是说调用了该函数候要立即把信息复制到其他内存空间。因此，若再次调用 inet_ntoa 函数，则有可能覆盖之前保存的字符串信息。总之，再次调用 inet_ntoa 函数前返回的字符串地址是有效的。若需要长期保存，则应该将字符串复制到其他内存空间*/
```

**网络地址初始化的示例：**

```c++
struct sockaddr_in addr;
char *serv_ip = "211.217.168.13"; //声明IP地址族
char *serv_port = "9190"; //声明端口号字符串

memset(&addr, 0, sizeof(addr)); 
//结构体变量 addr 的所有成员初始化为0
addr.sin_family = AF_INET; //制定地址族

addr.sin_addr.s_addr = inet_addr(serv_ip);
//基于字符串的IP地址初始化，将字符串转化为整数形式的IP地址

addr.sin_port = htons(atoi(serv_port)); 
//基于字符串的IP地址端口号初始化，atoi是将字符串转化为整数，htos将主机上的整数型转化为网络字节序
```

#### 收发数据

首先看 send 函数定义：

```c++
#include <sys/socket.h>
ssize_t send(int sockfd, const void *buf, size_t nbytes, int flags);
/*
成功时返回发送的字节数，失败时返回 -1
sockfd: 表示与数据传输对象的连接的套接字和文件描述符
buf: 保存带传输数据的缓冲地址值
nbytes: 待传输字节数
flags: 传输数据时指定的可选项信息
/*
```

下面是 recv 函数的定义：  

```c++
#include <sys/socket.h>
ssize_t recv(int sockfd, void *buf, size_t nbytes, int flags);
/*
成功时返回接收的字节数（收到 EOF 返回 0），失败时返回 -1
sockfd: 表示数据接受对象的连接的套接字文件描述符
buf: 保存接受数据的缓冲地址值
nbytes: 可接收的最大字节数
flags: 接收数据时指定的可选项参数
*/
```

send 和 recv 函数都是最后一个参数是收发数据的可选项，该选项可以用位或（bit OR）运算符（| 运
算符）同时传递多个信息。
send & recv 函数的可选项意义

| 可选项（Option） | 含义                                                         | send | recv |
| ---------------- | ------------------------------------------------------------ | ---- | ---- |
| MSG_OOB          | 用于传输带外数据（Out-of-band data）                         | O    | O    |
| MSG_PEEK         | 验证输入缓冲中是否存在接受的数据                             | X    | O    |
| MSG_DONTROUTE    | 数据传输过程中不参照本地路由（Routing）表，在本 地（Local）网络中寻找目的地 | O    | X    |

后续讲到这里再细讲

### tcp服务端的简单实现（Linux版本）

#### 1.tcp服务器的基本实现逻辑与API

![image-20231231193131659](E:\note\linux-server\picture\image-20231231193131659.png)

1.先创建一个socket ，分配协议族和协议

2.分配套接字地址

> 需要分配套接字的地址，才能让客户端连接 
>
> ```c++
> #include <sys/socket.h>
> int bind(int sockfd, struct sockaddr *myaddr, socklen_t addrlen);
> //成功时返回0，失败时返回-1
> ```
>



3.受理客户端连接请求

> 已经调用了 bind 函数给套接字分配地址，接下来就是要通过调用 listen 函数进入等待连接请求状态。
> 只有调用了 listen 函数，客户端才能进入可发出连接请求的状态。换言之，这时客户端才能调用
> connect函数 ，**此时客户端调用connect函数就意味着进入连接请求等待队列，还需要服务端调用accept函数才能与服务端交换数据**
>
> ```c++
> #include <sys/socket.h>
> int listen(int sockfd, int backlog);
> //成功时返回0，失败时返回-1
> //sock: 希望进入等待连接请求状态的套接字文件描述符，传递的描述符套接字参数称为服务端套接字
> //backlog: 连接请求等待队列的长度，若为5，则队列长度为5，表示最多使5个连接请求进入队列
> ```
>



3.接受远程连接

> accept函数受理连接请求等待队列中待处理的客户端连接请求。函数调用成功时，accept函数内部将产生用于数据I/O的套接字，并返回其文件描述符。需要强调的是，套接字是自动创建的,并自动与发起连接请求的客户端建立连接，可以直接对该套接字进行读写操作
>
> ![image-20240104215946913](E:\note\linux-server\picture\image-20240104215946913.png)
>
> ```c++
> #include <sys/socket.h>
> int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
> /*
> 成功时返回文件描述符，失败时返回-1
> sock: 服务端套接字的文件描述符
> addr: 保存发起连接请求的客户端地址信息的变量地址值
> addrlen: 第二个参数addr结构体的长度，但是存放有长度的变量地址。
> */
> ```
>

#### 2.服务端源码编译与分析

服务端源码[server.cpp](www.baidu.com)（源码比较简单且有足够的注释）

编译源码,生成可执行文件main

```c++
g++ -g server.cpp -o main
```

运行可执行程序

```c++
./main
```

执行结果



### tcp客户端的实现（linux版本）

#### 1.tcp客户端的基本实现逻辑与API

![image-20240116225133003](C:\Users\19193\AppData\Roaming\Typora\typora-user-images\image-20240116225133003.png)

与服务端相比，区别就在于「请求连接」，他是创建客户端套接字后向服务端发起的连接请求。服务端
调用 listen 函数后创建连接请求等待队列，之后客户端即可请求连接。  

```c++
#include <sys/socket.h>
int connect(int sock, struct sockaddr *servaddr, socklen_t addrlen);
/*
成功时返回0，失败返回-1
sock:客户端套接字文件描述符
servaddr: 保存目标服务器端地址信息的变量地址值
addrlen: 以字节为单位传递给第二个结构体参数 servaddr 的变量地址长度
*/

```

客户端调用 connect 函数后，发生以下函数之一才会返回（完成函数调用）:
1.服务端接受连接请求
2.发生断网等一场状况而中断连接请求
注意：接受连接不代表服务端调用 accept 函数，其实只是服务器端把连接请求信息记录到等待队列。
因此 connect 函数返回后并不应该立即进行数据交换  

#### 2.客户端源码编译与分析

服务端源码[client](www.baidu.com)（源码比较简单且有足够的注释）

编译源码,生成可执行文件main

```c++
g++ -g client.cpp -o main
```

运行可执行程序

```c++
./main
```

执行结果
