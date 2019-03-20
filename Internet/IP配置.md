# 实验一：配置IP

## 1. 准备材料

- 使用dynamips模拟Cisco交换机进行实验
- 下载好需要使用的.net 和.cmd文件（.net 文件放到net 文件夹，.cmd放到根目录下）

## 2.启动模拟环境

- 运行“虚拟服务*p & 2003.bat”

- 运行“bupt_sw.cmd”

  Note:这两个窗口在之后的进程中都不能关

## 3.查看设备状态并启动

```shell
list   #查看所有设备状态
start SW1	#让所有设备都跑起来
start SW2
start CP1
start CP2
start CP3
```

- #### idlepc值

  第一次启动设备的时候要设置idlepc值

  n每个设备的idlepc值标识这个设备的cpu占用率

  n每一类设备只有在该类的第一个设备启动时需要设置idlepc值，以后再启动，或启动其它同类设备都不需要再重设；

  ```shell
  idlepc get 设备名  #获取设备的idlepc值 
  优先选择带*的，再选择数值大的
  idlepc save 设备名 db 	#将设备的idlepc值保存到数据库
  ```

- #### telnet  设备名

  这是真正的连入(启动)设备,需要打开SW1,SW2,PC1,PC2,PC3

  ```shell
  telnet SW1
  或者在cmd中
  telnet 127.0.0.1 设备端口号 #需要开启telnet服务
  ```

  Note: 配置完IP之后 ，也不要关闭窗口，设备才能运行

## 4. 配置IP

```shel
回车 --> no -->回车    # 进入Route >
```

```shel
enable 				# 获取配置权限，Route #
					# Route# 模式下可以进行
						show running-config(可以用缩写sh run) ,显示ip配置信息
						ping
```

```shell
configure t 		#进入Route(Config) #
```

```shell
interface f0/0		#进入配置ip端口号 Route(config-if) #
ip add ip地址(cp1配置为1.1.1.1,CP2配置为1.1.1.2) 子网掩码	
no shutdown  		#让端口一直运行
```

```shell
#输入exit 返回到 Route(config) #
no dpc run #让配置生效
```

## 5. 测试

```shell
#在PC1下，按ctrl + z，加到Route #状态
ping 1.1.1.1  #ping 本机，通了，说明ip地址配上了
pint 1.1.1.2  #ping PC2
			  #  . 表示丢包 ！表示成功
```



# Q & A

### 1. 拓扑结构怎么看？

从实验的拓扑结构来看,CP1 和CP2互通，需要SW1 运行

与CP3的互通需要SW2运行



### 2.实验有哪些现象？

- 开启SW1之后 ，第一次ping ，所有包都是丢的
- 第二次ping ，丢一个包
  - 原因是开始时转换表为空，发了一个包之后 ，才被记录在转换表中







