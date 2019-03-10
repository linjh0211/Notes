# SOCKET编程

## Part One  函数

```c
main(argc,argv)
WSAStartup()
MAKEWORD(2,2)
socket()
htons()
inet_addr()
recvfrom()
WSAGetLastError()
sendto()
recvfrom()
```



```c
int WSAStartup ( WORD wVersionRequested, LPWSADATA lpWSAData ); 
使用Socket的程序在使用Socket之前必须调用WSAStartup函数。以后应用程序就可以调用所请求的Socket库中的其它Socket函数了。
头文件 header： 　Winsock2.h
库library：      Ws2_32.lib
```

```C
首先,建立一个WSADATA结构,通常用wsaData
WSADATA wsaData;

然后,调用WSAStartup函数,这个函数是连接应用程序与winsock.dll的第一个调用.其中,第一个参数是WINSOCK 版本号,第二个参数是指向
WSADATA的指针.该函数返回一个INT型值,通过检查这个值来确定初始化是否成功.调用格式如下:WSAStartup(MAKEWORD(2,2),&wsaData),其中
MAKEWORD(2,2)表示使用WINSOCK2版本.wsaData用来存储系统传回的关于WINSOCK的资料.
```



```c
Window环境下：
#include <winsock.h>
int PASCAL FAR bind( SOCKET sockaddr, const struct sockaddr FAR* my_addr,int addrlen);

Linux环境下：
#include <sys/types.h>
#include <sys/socket.h>
/****
*  sockfd：标识一未捆绑套接口的描述字。
*  my_addr：赋予套接口的地址。sockaddr结构定义如下：
*  struct sockaddr{
*    u_short sa_family;
*    char sa_data[14];
*  };
*  addrlen：name名字的长度。
*  返回值：成功返回0，失败返回-1.
****/
int bind( int sockfd , const struct sockaddr * my_addr, socklen_t addrlen);
```



```c
#include <sys/socket.h>
int socket( int af, int type, int protocol);
af：一个地址描述。目前仅支持AF_INET格式，也就是说ARPA Internet地址格式。ipv4
type：指定socket类型。新套接口的类型描述类型，如TCP（SOCK_STREAM）和UDP（SOCK_DGRAM）。常用的socket类型有，SOCK_STREAM、SOCK_DGRAM、SOCK_RAW、SOCK_PACKET、SOCK_SEQPACKET等等。
protocol：顾名思义，就是指定协议。套接口所用的协议。如调用者不想指定，可用0。常用的协议有，IPPROTO_TCP、IPPROTO_UDP、IPPROTO_STCP、IPPROTO_TIPC等，它们分别对应TCP传输协议、UDP传输协议、STCP传输协议、TIPC传输协议。
```

```c
htons()
//将本机无符号短整型数转换成网络字节顺序 
```

```c
inet_addr()
//将点分十进制转换成长整数型数
    
```

```c
int recvfrom(SOCKET s,void *buf,int len,unsigned int flags, struct sockaddr *from,int *fromlen)
//用来接收远程主机经指定的socket传来的数据,并把数据传到由参数buf指向的内存空间,参数len为可接收数据的最大长度.参数flags一般设0,其他数值定义参考recv().参数from用来指定欲传送的网络地址,结构sockaddr请参考bind()函数.参数fromlen为sockaddr的结构长度.
//成功则返回接收到的字符数,失败返回-1.
```



```c
#include <winsock.h>
WSAGetLastError()//是指该函数返回上次发生的网络错误，当一特定的Windows Sockets API函数指出一个错误已经发生，该函数就应调用来获得对应的错误代码
```



```c
定义函数
	int sendto ( int s , const void * msg, int len, unsigned int flags, const
struct sockaddr * to , int tolen ) ;
函数说明
	s 是用来通信的socket接口，to 是目标服务器
	sendto() 用来将数据由指定的socket传给对方主机。参数s为已建好连线的socket,如果利用UDP协议则不需经过连线操作。参数msg指向欲连线的数据内容，参数flags 一般设0，详细描述请参考send()。参数to用来指定欲传送的网络地址，结构sockaddr请参考bind()。参数tolen为sockaddr的结果长度。
返回值
	成功则返回实际传送出去的字符数，失败返回－1，错误原因存于errno 中。
错误代码
	EBADF 参数s非法的socket处理代码。
EFAULT 参数中有一指针指向无法存取的内存空间。
WNOTSOCK canshu s为一文件描述词，非socket。
EINTR 被信号所中断。
EAGAIN 此动作会令进程阻断，但参数s的soket为补课阻断的。
ENOBUFS 系统的缓冲内存不足。
EINVAL 传给系统调用的参数不正确。
```



```c
recvfrom（经socket接收数据）

函数原型:int recvfrom(SOCKET s,void *buf,int len,unsigned int flags, struct sockaddr *from,int *fromlen);

相关函数 recv，recvmsg，send，sendto，socket
函数说明:
参数from是发送方,s是接收方
recv()用来接收远程主机经指定的socket传来的数据,并把数据传到由参数buf指向的内存空间,参数len为可接收数据的最大长度.参数flags一般设0,其他数值定义参考recv().结构sockaddr请参考bind()函数.参数fromlen为sockaddr的结构长度.

返回值:成功则返回接收到的字符数,失败返回-1.
错误代码
EBADF 参数s非合法的socket处理代码/
EFAULT 参数中有一指针指向无法存取的内存空间。
ENOTSOCK 参数s为一文件描述词，非socket。
EINTR 被信号所中断。
EAGAIN 此动作会令进程阻断，但参数s的socket为不可阻断。
ENOBUFS 系统的缓冲内存不足
ENOMEM 核心内存不足
EINVAL 传给系统调用的参数不正确。
```



## Part  Two  结构体

```c
sockaddr_in
socket
wsadata
DNSHeader
```



```c
    struct sockaddr_in
    {
        u8 sin_len;					#sockaddr_in的长度
        u8 sin_family;			#协议族类型
        u16 sin_port;			#16位端口号 
        struct in_addr sin_addr;		#32位ip地址
        char sin_zero[8]; 	#未使用
    }
```

```c
struct socket
{
     socket_state  state; // socket state      
     short   type ; // socket type
     unsigned long  flags; // socket flags
     struct fasync_struct  *fasync_list;
     wait_queue_head_t wait;
     struct file *file;
     struct sock *sock;  // socket在网络层的表示；
     const struct proto_ops *ops;
}
```

```c
   typedef struct DNSHeader   
    {   
        unsigned short ID;   
        unsigned short Flags;   
        unsigned short QuestNum;   
        unsigned short AnswerNum;   
        unsigned short AuthorNum;   
        unsigned short AdditionNum;   
    }
```



## Part 3 导入动态链接库

```c
#在Makefile文件最后面加上
target_link_libraries(项目名 .dll名字)
```



## PART 4 DNS报文格式

### Note:

1. recvbuf 是除了dns header之外的部分

   recvbuf =dns header +  dns具体内容

### Case 1: 在本地域名解析表中找到域名

【对比query报文和answer报文】

![1551690717595](C:\Users\XXX\AppData\Roaming\Typora\typora-user-images\1551690717595.png)



1. *定义transaction ID( unsigned short ) 

2. 修改flag位 为0x8180

![1551664378078](C:\Users\XXX\AppData\Roaming\Typora\typora-user-images\1551664378078.png)

3. 修改Question 为0 或1

4. Answer域，Authority ，Additional不需要修改

5. 构造DNS Answer部分

   ![1551666555401](C:\Users\XXX\AppData\Roaming\Typora\typora-user-images\1551666555401.png)

   

   (1) 前4个字节为 0xc00c ，具体是什么，我也不太知道

   (2) Type 定义为0x0001

   (3) Class 定义为0x0001

   (4) timelive 定义为0x0000 007b

   (5)  Data length 定义为0x0004

   (6) 之后的16个字节为查询到的ip地址

### Case 2 在域名解析表中找不到

##### 1.ID转换

将接收到的ID，转成本地字节序，再将主机信息一并存入ID转换表中, 再换成自己设计的id（需要转成网络字节序）

![1551693265207](C:\Users\XXX\AppData\Roaming\Typora\typora-user-images\1551693265207.png)

##### 2.转发DNS报文

##### 3.接收DNS报文

##### 4.ID转换

获取接收到的ID，转成本地字节序，再到ID 转换表中去查找得到原来的ID，和请求主机的信息。 

将recvbuf中的ID 换掉 ，转发回请求主机





 