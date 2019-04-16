# 实验三 RIP和OSPF路由协议的配置及协议流程分析

## 一 . 网络拓扑

![未命名文件 (1)](C:\Users\xxx\Downloads\未命名文件 (1).jpg)



1. 为每个设备命名

2. 选择插口芯片

   ![1553677481368](C:\Users\xxx\AppData\Roaming\Typora\typora-user-images\1553677481368.png)

   - 每台PC机自带两个Fastethernet接口，
   - R1-R4的slot0选择PA-C7200-IO-FE的芯片
   - R1-R5的slot1选择PA-C7200-IO-FE的芯片
   - **特别的，将R5的slot2设置为PA-2FE-TX,以支持两台PC机的接入**

3. 连接网络，为每一个串口分配ip地址和子网掩码，注意连线两端的接口类型必需一致

   - 关于**IP地址的设计**，在Q&A部分有说明

## 二 . 实验步骤

### 1. 设计拓扑结构 

### 2. 编写.net文件

```shell
autostart = False

[localhost]
    port = 7200
    udp = 10000
    workingdir = ..\tmp\
    
    [[router R1]]
        image = ..\ios\unzip-c7200-is-mz.122-37.bin
        model = 7200
        console = 3001
        npe = npe-400
        ram = 64
        confreg = 0x2102
        exec_area = 64
        mmap = False
        slot0 = PA-C7200-IO-FE
        slot1 = PA-4T
        f0/0 = PC1 f0/0
        s1/0 = R2 s1/0
        s1/1 = R5 s1/0
        s1/2 = R4 s1/2
        idlepc = 0x6067d100
    
    
    [[router R2]]
        image = ..\ios\unzip-c7200-is-mz.122-37.bin
        model = 7200
        console = 3002
        npe = npe-400
        ram = 64
        confreg = 0x2102
        exec_area = 64
        mmap = False
        slot0 = PA-C7200-IO-FE
        slot1 = PA-4T
        f0/0 = PC2 f0/0
        s1/1 = R5 s1/3
        s1/2 = R3 s1/2
        idlepc = 0x605c345c
    
    
    [[router R3]]
        image = ..\ios\unzip-c7200-is-mz.122-37.bin
        model = 7200
        console = 3003
        npe = npe-400
        ram = 64
        confreg = 0x2102
        exec_area = 64
        mmap = False
        slot0 = PA-C7200-IO-FE
        slot1 = PA-4T
        f0/0 = PC3 f0/0
        s1/1 = R5 s1/2
        s1/0 = R4 s1/0
        idlepc = 0x605c34a0
    
    
    [[router R4]]
        image = ..\ios\unzip-c7200-is-mz.122-37.bin
        model = 7200
        console = 3004
        npe = npe-400
        ram = 64
        confreg = 0x2102
        exec_area = 64
        mmap = False
        slot0 = PA-C7200-IO-FE
        slot1 = PA-4T
        f0/0 = PC4 f0/0
        s1/1 = R5 s1/1
        idlepc = 0x605c33fc
    
    
    [[router R5]]
        image = ..\ios\unzip-c7200-is-mz.122-37.bin
        model = 7200
        console = 3005
        npe = npe-400
        ram = 64
        confreg = 0x2102
        exec_area = 64
        mmap = False
        slot2 = PA-2FE-TX
        slot1 = PA-4T
        f2/0 = PC6 f0/0
        f2/1 = PC5 f0/0
        idlepc = 0x6067b6c0
    
    
    [[router PC1]]
        model = 2621
        ram = 20
        image = ..\ios\unzip-c2600-i-mz.121-3.T.bin
        mmap = False
        confreg = 0x2102
        console = 3006
        idlepc = 0x80357140
    
    [[router PC2]]
        model = 2621
        ram = 20
        image = ..\ios\unzip-c2600-i-mz.121-3.T.bin
        mmap = False
        confreg = 0x2102
        console = 3007
        idlepc = 0x803772d0
    
    [[router PC3]]
        model = 2621
        ram = 20
        image = ..\ios\unzip-c2600-i-mz.121-3.T.bin
        mmap = False
        confreg = 0x2102
        console = 3008
        idlepc = 0x802c0730
    
    [[router PC4]]
        model = 2621
        ram = 20
        image = ..\ios\unzip-c2600-i-mz.121-3.T.bin
        mmap = False
        confreg = 0x2102
        console = 3009
        idlepc = 0x803553bc
    
    [[router PC5]]
        model = 2621
        ram = 20
        image = ..\ios\unzip-c2600-i-mz.121-3.T.bin
        mmap = False
        confreg = 0x2102
        console = 30010
        idlepc = 0x802c0730
    
    
    
    [[router PC6]]
        model = 2621
        ram = 20
        image = ..\ios\unzip-c2600-i-mz.121-3.T.bin
        mmap = False
        confreg = 0x2102
        console = 30011
        idlepc = 0x802c0730
```



### 3. 修改.cmd文件

```shell
@echo off
call bin/script/copyright.cmd
title 控制台,请不要关闭本窗口！
echo * bupt 计网实验三                                                           *
echo *=============================================================================*
cd tmp
"../bin/dynagen/dynagen.exe" ..\net\3.net
```



### 4.启动模拟器

1. 运行虚拟服务XP&2003.bat

2. 运行3.cmd

3. 查看设备状态

   ```
   list
   ```

4. 启动所有设备

   ```shell
   start \all
   #如果没有分配idlepc值，则用
   #idlepc get 设备名 
   #获取 ，选择最优idlepc
   #保存
   #idlepc save 设备名 db
   ```

5. 连接到所有设备

   ```shell
   telnet 设备
   ```

### 5. 子网内PC与路由器相通

- ###### 配置PC1

```shell
#回车，输入no
en 			#获取权限
configure t		#进入配置模式
int f0/0	#选择f0/0接口进行配置
ip add 1.0.220.1 255.0.0.0
no shutdown
```

- ###### 同样的方式，配置R1

```shell
#回车，输入no
en 			#获取权限
configure t		#进入配置模式
int f0/0	#选择f0/0接口进行配置
ip add 1.0.220.2 255.0.0.0
no shutdown
```

- ###### 测试

```shell
#R1,exit退到Route#模式
ping 1.0.220.1
#若能Ping通，说明子网1.0.0.0配好了
```

- ###### 配置其它5个路由和PC直连的子网并测试



### 6. 路由器之间互通

- ###### 以R1,R2互通为例，配置R1


```shell
#退到Route(config)# 
interface s1/0		#进入s1/0接口的配置，注意是s0/1
clock rate ?		#查看可用速率
clock rate 9600		#物理层-->配置速率； 只配一端，R3不用配
encapsulation ppp	#数据链路层 --> 选择协议，两端协议应一样
					#可以不设置，默认DHCP协议
ip add 11.0.220.1 255.0.0.0	#分配ip

```

- ###### 配置R3

```shell
#退到Route(config)# 
interface s1/0		#进入s1/0接口的配置，注意是s0/1
encapsulation ppp	#数据链路层 --> 选择协议，两端协议应一样
					#可以不设置，默认DHCP协议
ip add 11.0.200.2 255.0.0.0	#分配ip
```

- ###### 测试

```shell
#R3,exit退到Route#模式
ping 11.0.220.1
#若能Ping通，说明子网11.0.0.0配好了
```

- ###### 配置其它7个路由和路由直连的子网并测试



### 7.配置RIP路由协议

- ###### 为PC设置缺省路由

  ```shell
  #进入每台PC，在Route(config)# 下设置路由
  ip route 0.0.0.0 0.0.0.0 f0/0
  ```

- ###### 逐个路由配置RIP协议

  ```shell
  #进入路由器的configure界面
  router rip			 #启动rip协议
  version 2 			 #转换为rip 2版本
  network 相邻子网网段号	#宣告网段
  ...
  neighbor 相邻主机IP	  #广播单播报文
  ...
  ```

- ###### Debug观察RIP协议过程

  ```shell
  debug ip rip
  ```

- ###### Note:用到的指令

  - 进入**每个接口**，去除**水平分隔**

  ```shell
  no ip split-horizen
  ```

  - 显示**路由表**

  ```shell
  show ip route		#在router#下输入
  ```



### 8.更改为OSPF路由协议

- ###### 去除RIP

  ```shell
  #在configure界面下输入
  no router rip
  ```

- ###### 逐个路由配置OSPF协议

  ```shell
  router ospf 10  	#10是进程号
  network 子网号 掩码	area 0	#宣告网段
  ...
  int s1/1	#进入每个接口配置
  #设置hello信息和dead信息的时间间隔ip 
  ip ospf hello-interval 5
  ip ospf dead-interval 20
  ```

- ###### debug 观察OSPF工作过程

  ```shell
  debug ip ospf events #输出ospf工作过程的各种事件（除了泛洪事件）
  debug ip ospf flood #查看泛洪事件
  sh ip ospf neighbor #查看ospf的邻居
  
  ```



### 9.实验结果

- #### rip

  - R1

    ![1555142525641](C:\Users\xxx\AppData\Roaming\Typora\typora-user-images\1555142525641.png)

  - R2

    ![1555142604352](C:\Users\xxx\AppData\Roaming\Typora\typora-user-images\1555142604352.png)

  - R3

    ![1555142646941](C:\Users\xxx\AppData\Roaming\Typora\typora-user-images\1555142646941.png)

  - R4

    ![1555142686778](C:\Users\xxx\AppData\Roaming\Typora\typora-user-images\1555142686778.png)

  - R5

    ![1555142721370](C:\Users\xxx\AppData\Roaming\Typora\typora-user-images\1555142721370.png)

  - **结果分析**：

    R1-R5的路由表中，添加了所有子网，说明实验成功。

- #### ospf

  - R1

    ![1555141197263](C:\Users\xxx\AppData\Roaming\Typora\typora-user-images\1555141197263.png)

  - R2

    ![1555141287931](C:\Users\xxx\AppData\Roaming\Typora\typora-user-images\1555141287931.png)

  - R3

    ![1555141354691](C:\Users\xxx\AppData\Roaming\Typora\typora-user-images\1555141354691.png)

  - R4

    ![1555141403317](C:\Users\xxx\AppData\Roaming\Typora\typora-user-images\1555141403317.png)

  - R5

    ![1555141450998](C:\Users\xxx\AppData\Roaming\Typora\typora-user-images\1555141450998.png)

  - **结果分析**：

    R1-R5的路由表中，添加了所有子网，说明实验成功。



#### Q1：IP地址的选定？

**A:** 计算三类IP地址的网段号范围：

- 第一类：1.0.0.0  - 127.0.0.0
- 第二类：128.0.0.0 - 191.255.0.0
- 第三类：192.0.0.0 - 223.255.255.0

选用第一类IP地址的网段号一共有127个，可以组建127个子网，对于实验要求来说，已经足够了

- ###### I**P设置**：

子网1：网段号1.0.0.0   IP: 1.0.220.1  , ...

子网2：网段号2.0.0.0   IP: 2.0.220.1  , ...

**Note:**考虑到RIP配置使用的是默认子网掩码255.0.0.0，不会造成影响



#### Q2 ：.net文件写错

![1554893200929](C:\Users\xxx\AppData\Roaming\Typora\typora-user-images\1554893200929.png)



**A:** 开始时，我以为将R5的slot0芯片换成C7200-IO-2F，就可以支持两个Fastethernet接口，结果报错了。

**解决方案**：尝试了

```
1.   slot0=PA-C7200-IO-2FE
	 f0/0 = PC5
	 f0/1 = PC6
2.   slot0=PA-C7200-IO-GE-E
	 e0/0 = PC5
	 g0/0 = PC6
```

都不行,最终选择抛弃slot0,用slot2接入

```
    slot2 = PA-2FE-TX
    f2/0 = PC5 f0/0
    f2/1 = PC6 f0/0
```

#### Q3: 链接两端端口的工作模式不一致？

使用R5的 f2/0 ,PC5的f0/0之间的工作模式总是对不上，改了也没有什么用！！！？？？

```
duplex mismatch discovered on FastEthernet0/0 (not full duplex), with Router FastEthernet2/1 (full duplex)
```

**忽略这个问题**



#### Q4:PC机的配置信息不能保存？

**原因：**没有更改文件内容的权限

**A:** 需要将/tmp文件夹中的临时文件（.flash文件）删除，新建时就可以保存了。

**Note:**

```shell
#在router#下输入 保存
write
```

#### Q5:ping 不通？

case 1 : R1 -- 5.0.0.0

case 2 : R1 -- 6.0.0.0

**原因分析**：我直接放弃了rip，转身做ospf，做完之后查看router table，发现所有子网都加入了。所以问题应该出在我配置rip的时候不小心写少了指令，可能是少了neighbor 5.0.220.1

和neighbor 6.0.220.1

**方案**:重新配置了一遍rip，发现全通了。