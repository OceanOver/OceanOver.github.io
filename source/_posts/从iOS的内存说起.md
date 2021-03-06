title: 从iOS的内存说起
date: 2015-09-19 16:30:22
tags: 
- 内存 
---

### 当启动一个app

当启动一个app程序，系统会把开启的那个app程序从ROM里面拷贝到内存（RAM）*`RAM的访问速度要远高内存卡(Flash)或ROM`*，然后从内存里面执行。<!-- more -->

## 内存分区:可以分为5个区

   - **栈区**（stack）— 这个一般由编译器操作，会存一些局部变量，函数跳转地址，现场保护等等

   - **堆区**（heap） — 一般由程序员管理，比如alloc申请内存

   - **全局区**（静态区）（static）—，全局变量和静态变量的存储是放在一块的，初始化的全局变量和静态变量在一块区域， 未初始化的全局变量和未初始化的静态变量在相邻的另一块区域。 - 程序结束后有系统释放。注意：全局区又可分为未初始化全局区：.bss段 和 初始化全局区：data段。举例：int a;未初始化的。int a = 10;已初始化的。
    
   - **常量区** — 常量字符串就是放在这里的

   - **代码区** — 存放代码，即app程序会拷贝到这里*`app程序从ROM里面拷贝到内存（RAM）`*。
   
### 编程中内存相关
<font color=#E95044>当一个app启动后，代码区，常量区，全局区大小已固定，因此指向这些区的指针不会产生崩溃性的错误。而堆区和栈区是时时刻刻变化的（堆的创建销毁，栈的弹入弹出），所以当使用一个指针指向这两个区里面的内存时，一定要注意内存是否已经被释放，否则会产生程序崩溃（编程中很常见）。</font>


### 参考图 
![](/memery.png)