# Linux使用

## 

## Q1:双系统时间更新？

- #### 原因

UTC即Universal Time Coordinated，协调世界时

GMT 即Greenwich Mean Time，格林尼治平时

Windows 与 Mac/Linux 缺省看待系统硬件时间的方式是不一样的：

Windows把系统硬件时间当作本地时间(local time)，即操作系统中显示的时间跟B[IOS](http://m.2cto.com/kf/yidong/iphone/)中显示的时间是一样的。

Linux/Unix/Mac把硬件时间当作 UTC，操作[系统](http://m.2cto.com/os/)中显示的时间是硬件时间经过换算得来的，比如说北京时间是GMT+8，则系统中显示时间是硬件时间+8。

- #### 同步方法 

  ```shell
  sudo pacman -S ntpdate 					#安装ntpdate
  
  sudo ntpdate time.windows.com			#时间同步 
  	
  sudo hwclock --localtime --systohc		#时间同步到硬件 
  ```

  

