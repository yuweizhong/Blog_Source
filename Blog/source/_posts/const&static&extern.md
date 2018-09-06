---
title: '''const&static&extern'''
date: 2017-9-15 20:00:34
tags:
---

### 1、const
用于常量修饰
    （1）const用来修饰右边的基本变量或指针变量
    （2）被修饰的变量只读，不能被修改
```
	int  const  *p   //  *p只读 ;p变量

	int  * const  p  // *p变量 ; p只读

  const  int   * const p //p和*p都只读

  int  const  * const  p   //p和*p都只读 

  NSString * const myStr = @"myStrStr"; //声明字符串的用法
```


### 2、宏定义
#define 预定义

>   	在预处理时（编译前）进行文本替换，没有类型，不做任何类型检查
>
>   	能够定义一些函数方法等等，const做不到的
>
>   	大量的宏定义会导致编译时间长、二进制文件过大等等

### 3、static
静态变量修饰

####     修饰局部变量：

>     1.延长局部变量的生命周期,程序结束才会销毁。
> 
>     2.局部变量只会生成一份内存,只会初始化一次。
> 
>     3.改变局部变量的作用域。

####     修饰全局变量

>     1.只能在本文件中访问,修改全局变量的作用域,生命周期不会改
> 
>     2.避免重复定义全局变量

staic && const联合使用

```
  // 声明一个静态的全局只读常量（当前文件）
   static NSString * const myStr = @"myStrStr"; 
```
### 4、extern

>     用来获取全局变量(包括全局静态变量)的值，不能用于定义变量，定义和分配内存都在原来类中。
>
>     会在当前文件中先查找是否有改变量，其次再去全局查找
    
extern && const联合使用

```
  //全局共享的变量
  A.h
  NSString * const myStr = @"myStrStr";
  B.h
  extern NSString * const myStr;
```

### 5、UIKIT_EXTERN

如果多个.m文件需要用到同一个变量的情况，在pch文件中写上面这行代码,就相当于为每一个.m文件都写这行代码
```
     //例：苹果 系统预置的通知
 
     UIKIT_EXTERN NSString *const UIKeyboardWillShowNotification;

     UIKIT_EXTERN NSString *const UIKeyboardDidShowNotification;

     UIKIT_EXTERN NSString *const UIKeyboardWillHideNotification;
 
     UIKIT_EXTERN NSString *const UIKeyboardDidHideNotification;// UIKIT_EXTERN,是经过处理的extern
 ```  

    