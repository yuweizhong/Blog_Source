---
layout: '''resolveInstanceMethod和resolveClassMethod'
title: resolveInstanceMethod和resolveClassMethod
date: 2018-03-02 18:33:15
tags: method
---


在iOS中，消息发送是通过，objc_msgSend(id, SEL, ...)来实现的。
首先会在对象的类对象的cache,method list以及父类对象的cache,method list依次查找SEL对应的IMP。
如果向一个对象发送无法它无法处理的消息，一般会出现crash。

通过实现动态决议和消息转发。只有当动态决议无法决议selector的实现，才会尝试进行消息转发，可以避免一些crash情况。

```
//针对实例方法
+ (BOOL)resolveInstanceMethod:(SEL)name {
	 NSLog(@" >> Instance resolving %@", NSStringFromSelector(name)); 
	 if (name == @selector(MissMethod)) { 
	 /*
	 **就能在运行期动态地为 name 这个 selector 添加实现：dynamicMethodIMP。class_addMethod是运行时函数，所以需要导入头文件：objc/runtime.h。
	 */
	 class_addMethod([self class], name, (IMP)dynamicMethodIMP, "v@:");
	  return YES; } 
	  return [super resolveInstanceMethod:name]; 
	}

//针对类方法
 + (BOOL)resolveClassMethod:(SEL)name { 
 	NSLog(@" >> Class resolving %@", NSStringFromSelector(name)); 
 	return [super resolveClassMethod:name]; 
 }
```


