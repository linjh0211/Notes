# 自定义Graph库

[TOC]



## 1.功能需求

设计和实现一个图形函数库，具有绘制直线 段、任意圆弧、椭圆弧、多边形区域的阴影填 充和颜色填充等功能（仅调用画点函数）, 要 求显示功能效果并能使用所设计库函数显示完 成者名字。提交设计报告。 

```c
 Windows API:   setpixel(hdc,x,y,color) 
```

## 2.需求分析

1. 程序应该分成两部分：一个头文件mygraph.h 和一个应用程序的 main.c文件
   - mygraph.h文件完成所有图形函数的定义
   - main.c实例一个窗口类，应用mygraph.h提供的接口，完成绘制直线段、任意圆弧、椭圆弧、多边形区域的阴影填充和颜色填充等功能。
2. 成果展示：
   - 用类似水印的方式打印一句话
   - 在代码中修改选项绘制不同图形

## 3.基础知识

#### 1. WinMain()函数说明？

```c
int WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,PSTR szCmdLine, int iCmdShow)
/*
1.头文件 <windows.h>
2.第一个参数 HINSTANCE hInstance
　　HINSTANCE为实例句柄，句柄无非是一个数值，程序用它来标识某些东西。在此该句柄唯一标识我们这个程序。
　第二个参数 HINSTANCE hPrevInstance
　 前一个实例句柄，在win32程序中这一概念已不再采用，因此WinMain的第二个参数通常是NULL。
　第三个参数 PSTR szCmdLine
　　PSTR是一个字符指针，用来运行程序的命令行，有些程序在启动时用它来把文件装入内存。
　第四个参数 int iCmdShow
　　　　用来指明程序最初如何显示：正常显示、最大化到全屏、最小化到任务栏。
*/
```

#### 2.win32如何画一个窗口？

- ##### step1   定义窗口事件处理函数

  ```c
  LRESULT CALLBACK WndProc (HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam)
  /*
  HWND hwnd, // 窗口的句柄
  UINT uMsg, // 消息的ID号
  WPARAM wParam, // 消息所对应的参数
  LPARAM lParam // 消息所对应的参数
  */
  ```

- ##### step2 初始化窗口类型参数

  ```c
     WNDCLASS   wndclass; //定义一个WNDCLASS型结构
  
  	typedef struct _WNDCLASS { 
      UINT    style;  //窗口类的风格
      WNDPROC lpfnWndProc; //窗口处理函数
      int     cbClsExtra; //窗口类附加数据大小，为0
      int     cbWndExtra; //窗口附加数据大小，为0
      HANDLE  hInstance; //当前应用程序的实例句柄，为mainWin的第一个参数
      HICON   hIcon; //窗口图标,使用LoadIcon()
      HCURSOR hCursor; //窗口的鼠标，使用LoadCursor()
      HBRUSH  hbrBackground; //窗口的背景画刷使用GetStockObject()
      LPCTSTR lpszMenuName; //菜单，为NULL
      LPCTSTR lpszClassName; //窗口类名，为一个char[]，命名为“窗口”，就可以
      } WNDCLASS;
  ```

- ##### step3 注册窗口

  ```c
  RegisterClass()
  ```

- ##### step4 创建窗口

  ```c
  HWND CreateWindow(
  LPCTSTR lpClassName, //窗口类型名称
  LPCTSTR lpWindowName, //窗口名称
  DWORD dwStyle, //窗口类型
  int x, //窗口的左上角X坐边，可以设置为CW_USEDEFAUL，默认设置
  int y, //窗口的左上角X坐边
  int nWidth, //窗口的宽度
  int nHeight, //窗口的高度
  HWND hWndParent, //父窗口句柄,为null
  HMENU hMenu, //窗口菜单句柄,为null
  HANDLE hInstance, //应用程序的实例句柄
  LPVOID lpParam //创建的参数，一般为NULL
  );
  ```

- ##### step5 显示窗口

  ```c
  ShowWindow (hwnd, iCmdShow) ;        //在显示器上显示窗口
  ```

- ##### step6  刷新窗口

  ```c
  UpdateWindow (hwnd) ;                //绘制窗口
  ```

  

#### 3.窗口事件处理函数怎么写？

```c
//这是wndProc函数的基本内容
LRESULT CALLBACK WndProc(HWND hwnd ,UINT message, WPARAM wparam,LPARAM lParam){
    switch(message){
        case WM_DESTROY:
            PostQuitMessage(0);
            return 0;
//        case WM_PAINT:
    }
    return DefWindowProc(hwnd,message,wparam,lParam);
}
```









## 4.实现

## 5. Q&A 

#### 1.Q:main()和WinMain()的区别？

**A:** main()用在console程序 ，如果要用到gui,最好是有WinMain();

#### 2.ADD方法画直线的问题

**A:**需要主要增量计算必需是double类型，注意类型转换的问题

#### 3.Bresenham方法画直线的问题

**A:**课件中只给出了斜率为0~1取值的情况，还应考虑-1~0及斜率绝对值大于1的情况。

####  4.画阴影时，为什么有MN个点，就一定是MN条边？

**A:**  当只考虑一个外环，一个内环或者没有内环的情况时，有MN 个点就一定是MN条边。











