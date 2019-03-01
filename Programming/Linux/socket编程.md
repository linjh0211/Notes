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
socket(AF_INET,SOCK_DGRAM,0)
 //AF——INET--ipv4, SOCK_DGRAM--支持UDP连接    
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

