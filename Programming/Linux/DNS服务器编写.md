[TOC]



# 怎么编写一个DNS 服务器？



**实验环境：**

win 10 , clion 

**实验工具:**

wireshark （需安装NpCap,以捕获本地回环数据）

## 1.功能说明

1.服务器功能

2.过滤功能

3.中继功能

## 2.步骤

### Step 1:项目配置

- 任意挑选一种IDE,我使用的Clion

- 引进动态链接库 --  Ws2_32.lib

  ```C
  #在Clion中的方法是，打开项目的makefile.txt文件，在最后添加
  target_link_libraries(myDNS Ws2_32.lib)
  即可
  #在Dev-C，codeblock中都有自己的配置方法，只要最后编译时不出现这种错误就是可以了
  ```

  ```shell
  CMakeFiles\myDNS.dir/objects.a(main.cpp.obj): In function `main':
  F:/Code/CLion/myDNS/main.cpp:371: undefined reference to `inet_addr@4'
  F:/Code/CLion/myDNS/main.cpp:380: undefined reference to `sendto@24'
  F:/Code/CLion/myDNS/main.cpp:382: undefined reference to `WSAGetLastError@0'
  F:/Code/CLion/myDNS/main.cpp:391: undefined reference to `closesocket@4'
  F:/Code/CLion/myDNS/main.cpp:392: undefined reference to `closesocket@4'
  F:/Code/CLion/myDNS/main.cpp:393: undefined reference to `WSACleanup@0'
  ```

  

### Step 2:引入头文件

- ```C
  #include <time.h>				//获取系统时间
  #include <winsock2.h>			//socket编程头文件
  #include <string>				//本地，网络字节序转换时要用
  #include <iostream>				
  #include <fstream>				
  ```

### Step 3:结构体设计

- DNS报文头

  ```C
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

- 域名解析表节点

  ```C
  typedef struct translate
  {
      string IP;
      string domain;
  }
  ```

- ID转换表节点

  ```C
  typedef struct IDChange
  {
      unsigned short oldID;           //原有的ID
      bool done;                      //标记是否完成解析
      SOCKADDR_IN client;             //请求者套接字地址
  }
  ```

  Note:

  - ID转换表的作用是完成网络ID和本地ID的相互转换
  - 之所以要相互转换，是为了处理多用户并发的情况

### Step 5: 对Winsock服务的初始化

1. 定义一个WSADATA;

   ```c
       WSADATA wsaData;
   ```

2. 初始化

   ```c
       if(WSAStartup(MAKEWORD(2,2),&wsaData) != 0) {
           return  0;
       }
   ```

### Step 6: 定义通信接口SOCKET

1. 定义三个以太网套接字socketaddr_in,一个存储本地DNS服务器信息，一个存储北邮DNS服务器信息

   ```c
    SOCKADDR_IN serverName , localName , clientName; 
   ```

2. 定义两个通信socket,一个负责和本地DNS服务器通信，一个负责和北邮DNS服务器通信

   ```c
    SOCKET socketServer, socketLocal ;       
   ```

3. 初始化socketeaddr_in 和socket

   ```c
       socketServer = socket(AF_INET , SOCK_DGRAM ,IPPROTO_UDP);     //AF——INET--ipv4, SOCK_DGRAM--支持UDP连接
       socketLocal = socket(AF_INET , SOCK_DGRAM ,IPPROTO_UDP) ;
   ```

   ```c
       //设置两个代表本地DNS服务器和外部DNS服务器的以太网套接字
       localName.sin_family = AF_INET;
       localName.sin_port = htons(DNS_PORT);       //htons()将本机无符号短整型（8位）转换成网络字节顺序
       localName.sin_addr.s_addr = inet_addr(LOCAL_ADDRESS) ;     //inet_addr ()将点分十进制转换成长整数型
   
       serverName.sin_family = AF_INET ;
       serverName.sin_port = htons(DNS_PORT);
       serverName.sin_addr.s_addr= inet_addr(outerDNS);
   ```

   Note:

   为什么不用初始化clientName？clientName使用默认参数

### Step 7 : 工作流程

1. #### 运行DNS服务程序

2. #### 打开一个或多个终端，输入指令

   ```shell
   nslookup 域名
   ```

   此时程序通过socketLocal接口，接收到clientName发来的DNS请求报文

3. #### 获取请求报文中携带的域名信息

4. #### 在域名解析表中查找是否有该域名

5. #### 分情况：

   - ##### Case 1: 没有找到    -->

     - ID转换，并在ID转换表中注册记录
     - 转发到北邮DNS服务器 
     - 接收来自北邮DNS服务的DNS响应报文
     - ID转换，找到对应记录，获取源主机信息
     - 将DNS响应报文，回发给源主机

   - ##### Case 2: 找到了  -->

     - ID转换，并在ID转换表中注册记录
     - 拷贝整个请求报文，因为该报文的很多信息还是有用的，只需要修改和添加部分内容就可以发回去
     - 修改标志位，回答数域
     - 构造Answer部分，包括：服务器名长，TYPE , CLASS , TIME_ALIVE ,IP 地址长度，IP地址
     - 将两部分组装在一起，即原报文在前，Answer部分在后

      

## 3.Q && A

### Q1:  关于ID转换

- #### 为什么要ID转换？

  我们编写的DNS服务器，在接收到一个DNS报文时，会先将其转换成内部的ID，一旦出了我们程序 ，发出去之前，要先把它转换成外部的ID。这样处理的好处是，如果程序操作需要用做ID做下标，比较方便。

- #### 怎么转换

  ##### Case 1: 接收到报文之后 ，怎么ID转换？

1. 用memcpy()函数，获取网络报文中的ID(此时为网络序)，即拷贝前8位
2. 用ntohs()函数,转成本地序，定义一个短整形，保存起来
3. 定义register()函数，将原ID,发送请求的主机等信息注册入ID转换表中，并返回该item的下标。
4. 用htons()函数，将该item的下标转成网络序

   ##### Case 2:要发送报文时，怎么ID转换？

5. 用memcpy()函数，获取网络报文中的ID(此时为网络序)，即拷贝前8位
6. 用ntohs()函数,转成本地序，定义一个短整形，保存起来（通过这个ID，可以在ID转换表中找到最初发送DNS请求的主机信息，因为它就是对应条目的下标）
7. 就ID转换表中该条目的标志位改为TURE





### Q2: 如何从报文中提取所需字段（如域名，IP）？

- #### 获取域名

  ##### 已知

1. recvfrom()函数的返回值iRecv表示接收到的字节数，即DNS报文长度；
2. 接收到的字节流存储在recv中（字节流的内容为DNS报文内容）

   ##### 方法

![1551961726724](C:\Users\XXX\AppData\Roaming\Typora\typora-user-images\1551961726724.png)

**Step1 拷贝内存**: 

​	从recv的第12个字节开始，拷贝iRecv-(12+4)个字节，拷贝的内容就是域名

​	**Note**: DNS报文头部的大小为12，域名之后还有Type（2字节）和Class（2字节），一共16字节，每个字节是8位

**Step2 域名转换** :

得到的域名中含有不可读的控制位，先过滤掉头尾，再将中间的控制位转成 “ . ”

- ### 获取IP

  （方法和获取域名差不多）

### Q3: 多客户端并发

允许多个客户端（可能会位于不同的多个计算机） 的并发查询，即：允许第一个查询尚未得到答案 前就启动处理另外一个客户端查询请求.

DNS报文头部的ID字段的作用，区别哪个响应包对应哪个请求包。因而不会乱掉。

Note:不需要用到线程



### 	Q4:参数问题？

说明：dnsrelay [-d | -dd] [<dns-server>] [<db-file>]





### Q5:为什么要字节序转换？

**A：**一般操作系统都是小端，而通讯协议是大端的。



### Q6:过滤掉反向DNS报文

- ###### What

  使用nslookup命令查寻IP时，会先后发三个上DNS协议包

  - 反向IP解析包：向DNS服务器发送请求的IP地址，返回类型为SOA的DNS报文，可以通过查询目标DNS服务器的SOA纪录得到对应的域名
  - IPV4请求包：
  - IPV6请求包：



### Q7:区域文件

- ###### What

  每台DNS服务器下存储若干区域文件，每个区域文件对应一个域名。区域文件下包含多种类型的纪录，如SOA,A/AAAA,MX,CNAME,PTR等。

- **A\AAAA记录格式**

  ![untitled](C:\Users\xxx\Pictures\untitled.png)

- **Note**

  - 一台真正的DNS服务器，在本地应该包括各种类型的纪录文件。在本实验中，我们只实现A类型的记录处理
  - 实验中的DNS反向解析，解析不到对应的域名，是因为在北邮的DNS服务器中没有对应的PRT纪录

### Q8：C++的多线程编程

**具体问题：**没办法使用c++11的thread类，引用了<thread>头文件，也没有用

![1554982215621](C:\Users\xxx\AppData\Roaming\Typora\typora-user-images\1554982215621.png)



**原因**：电脑原先的编译器是mingw5.2版本的，不支持<thread>，重新安装8.2版本的就可以支持了。

###### 补充问题：join 和detach的区别

- join 子线程会阻塞主线程，必须等子线程执行完，主线程才能执行
- detach 子线程和主线程交替执行



### Q9：clientName的作用？

- ###### recvfrom 函数

  ```c
  int recvfrom(SOCKET s,void *buf,int len,unsigned int flags, struct sockaddr *from,int *fromlen)
  /*	s 是通信的接口
  	buf 是接收信息的缓存
  	len 是接收信息的长度
  	flags 一般为0
  	from 可以为null, 也可以是一个sockaddr*类型的套接字，用于存储 信息从哪里发过来
  	formlen 是sockaddr的长度
  */
  ```

- ###### clienName的作用：

  - 用于记录DNS报文的源地址，即谁发的DNS报文
  - 当只在本机测试时，clientName就是localName
  - 当有外部主机访问时，clientName就是外部主机

## 4.参考代码

#### 



