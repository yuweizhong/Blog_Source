---
title: RunTime
date: 2018-09-04 11:13:41
tags: iOS
---

# RunTime
RunTime，简称运行时，是用C和汇编编写的一套底层API，是Objective-C面向对象和动态机制的基础。
[开放源码地址](https://opensource.apple.com/source/objc4/)

[API地址](https://developer.apple.com/documentation/objectivec/objective_c_runtime#//apple_ref/doc/uid/TP40001418-CH1g-126286)

## 1.类实例（objc_object)
objc_object结构体定义

```
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;

```
类实例的isa指针指向其所在类对象

## 2.类对象（objc_class）
objc_class结构体定义
```
typedef struct objc_class *Class;

struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;     //父类指针
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;     //类名字
    long version                                             OBJC2_UNAVAILABLE;     //版本
    long info                                                OBJC2_UNAVAILABLE;     //信息
    long instance_size                                       OBJC2_UNAVAILABLE;     //实例大小
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;     //变量列表
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE; //方法列表
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;     //缓存
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;     //遵守的协议列表
#endif

} OBJC2_UNAVAILABLE;

```
struct objc_class结构体定义了很多变量（标注如上），类对象就是一个结构体struct objc_class，这个结构体存放的数据称为元数据(metadata)，
对象中的元数据存储的都是如何创建一个实例的相关信息，那么类对象和类方法应该从哪里创建呢？
就是从isa指针指向的结构体创建，类对象的isa指针指向的我们称之为元类(metaclass)，元类中保存了创建类对象以及类方法所需的所有信息。

## 3.元类（Meta Class）

元类(Meta Class)是一个类对象的类。所有的类自身也是一个对象，我们可以向这个对象发送消息(即调用类方法)。
为了调用类方法，这个类的isa指针必须指向一个包含这些类方法的一个objc_class结构体。这就引出了meta-class的概念，元类中保存了创建类对象以及类方法所需的所有信息。
任何NSObject继承体系下的meta-class都使用NSObject的meta-class作为自己的所属类，而基类的meta-class的isa指针是指向它自己。

类实例，类对象和元类的整个结构图所示:
![image](http://note.youdao.com/favicon.ico)
通过上图我们可以看出整个体系构成了一个自闭环，struct objc_object结构体实例它的isa指针指向类对象，
类对象的isa指针指向了元类，super_class指针指向了父类的类对象，
而元类的super_class指针指向了父类的元类，那元类的isa指针又指向了自己。

## 4.Method(objc_method)
```
runtime.h
/// An opaque type that represents a method in a class definition.代表类定义中一个方法的不透明类型
typedef struct objc_method *Method;
struct objc_method {
    SEL method_name                  OBJC2_UNAVAILABLE;  //方法名
    char *method_types               OBJC2_UNAVAILABLE;  //方法类型
    IMP method_imp                   OBJC2_UNAVAILABLE;  //方法实现

```
    其中 SEL和IMP其实都是Method的属性。
### SEL(object_selector)
```
Objc.h
/// An opaque type that represents a method selector.代表一个方法的不透明类型
typedef struct objc_selector *SEL;

```
selector是方法选择器，可以理解为区分方法的 ID，而这个 ID 的数据结构是SEL,selector是SEL的一个实例.

> 同一个类，selector不能重复
> 不同的类，selector可以重复

### IMP
```
/// A pointer to the function of a method implementation.  指向一个方法实现的指针
typedef id (*IMP)(id, SEL, ...); 
#endif

```
指向最终实现程序的内存地址的指针

## 类缓存（object_cache)
当Objective-C运行时通过跟踪它的isa指针检查对象时，它可以找到一个实现许多方法的对象。然而，你可能只调用它们的一小部分，并且每次查找时，搜索所有选择器的类分派表没有意义。所以类实现一个缓存，每当你搜索一个类分派表，并找到相应的选择器，它把它放入它的缓存。所以当objc_msgSend查找一个类的选择器，它首先搜索类缓存。这是基于这样的理论：如果你在类上调用一个消息，你可能以后再次调用该消息。
为了加速消息分发， 系统会对方法和对应的地址进行缓存，就放在上述的objc_cache，所以在实际运行中，大部分常用的方法都是会被缓存起来的，Runtime系统实际上非常快，接近直接执行内存地址的程序速度。

## 分类Category（object_category）
```
struct category_t { 
    const char *name;                           //className
    classref_t cls;                             //要扩展的类对象，在runtime阶段通过name对应到类对象
    struct method_list_t *instanceMethods;      //给类添加的实例方法列表
    struct method_list_t *classMethods;         //类类添加的类方法列表
    struct protocol_list_t *protocols;          //实现的所有协议列表
    struct property_list_t *instanceProperties; //所有属性 非实例变量，通过objc_setAssociatedObject和objc_getAssociatedObject增加
};

```




## 一.Runtime --- 消息传递机制
在OC中，消息和方法的绑定是在运行时进行的，一个对象的方法像这样[obj foo]，编译器转成消息发送objc_msgSend(obj, foo)。消息传递机制的关键在于编译器对每个类和对象的结构的构建，每个类结构包含两个基本元素：指向父类的指针和类调度表。这个表罗列了他们定义的有明确类特征的方法的地址的方法选择器。

![image](http://note.youdao.com/favicon.ico)
当一个消息传递给一个对象的时候，消息函数沿着这个对象的isa指针在调度表找到它建立起方法选择器的类结构。如果它不能在这里发现选择器，obic_msgSend根据指针找到它的父类，在父类的调度表中寻找选择器。连续失败导致objc_msgSend沿着类继承结构直到寻找到NSObject类。一旦确定选择器的位置，函数调用表中的方法并且把它传给接收对象的数据结构。

objc_msgSend 方法定义：

```
OBJC_EXPORT id objc_msgSend(id self, SEL op, ...)
```

具体消息传递在Runtime时执行的流程是这样的（具体类对象/元类的定义可以参见[这里](http://note.youdao.com/)）：

1. 首先，通过obj的isa指针找到它的 class ;
2. 在 class 的 method list 找 foo 方法 ;
3. 如果 class 中没到 foo，继续往它的 superclass 中找 ;
4. 一旦找到 foo 这个函数，就去执行它的实现IMP 。

>  但是如果每次都按照这个流程执行的话，会有一个问题：一个class 往往只有 20% 的函数会被经常调用，可能占总调用次数的 80% 。每个消息都需要遍历一次objc_method_list，效率明显低下，很不合理。
--- 。如果把经常被调用的函数缓存下来，那可以大大提高函数查询的效率。这也就是objc_class 中另一个重要成员objc_cache 做的事情 - 再找到foo 之后，把foo 的method_name 作为key ，method_imp作为value 给存起来。当再次收到foo 消息的时候，可以直接在cache 里找到，避免去遍历objc_method_list。从前面的源代码可以看到objc_cache是存在objc_class 结构体中的。

## 二.Runtime --- 消息转发
![image](http://note.youdao.com/favicon.ico)

由上图可以看到，消息转发中有3次可以调整的机会，防止unrecognized selector报错，

### 动态方法解析
首先，Objective-C运行时会调用 +resolveInstanceMethod:或者 +resolveClassMethod:，让你有机会提供一个函数实现。如果你添加了函数并返回YES， 那运行时系统就会重新启动一次消息发送的过程。

### 备用接收者
如果目标对象实现了-forwardingTargetForSelector:，Runtime 这时就会调用这个方法，给你把这个消息转发给其他对象的机会。

### 完整消息转发
如果在上一步还不能处理未知消息，则唯一能做的就是启用完整的消息转发机制了。
首先它会发送-methodSignatureForSelector:消息获得函数的参数和返回值类型。如果-methodSignatureForSelector:返回nil ，Runtime则会发出 -doesNotRecognizeSelector: 消息，程序这时也就挂掉了。如果返回了一个函数签名，Runtime就会创建一个NSInvocation 对象并发送 -forwardInvocation:消息给目标对象。

从打印结果来看，我们实现了完整的转发。通过签名，Runtime生成了一个对象anInvocation，发送给了forwardInvocation，我们在forwardInvocation方法里面让Person对象去执行了foo函数。签名参数v@:怎么解释呢，这里苹果文档Type [Encodings有详细的解释](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1)。
以上就是Runtime的三次转发流程。下面我们讲讲Runtime的实际应用。


##RunTime --- 应用

关联对象(Objective-C Associated Objects)给分类增加属性
方法魔法(Method Swizzling)方法添加和替换和KVO实现
消息转发(热更新)解决Bug(JSPatch)
实现NSCoding的自动归档和自动解档
实现字典和模型的自动转换(MJExtension)

swizzling应该只在+load中完成。 在 Objective-C 的运行时中，每个类有两个方法都会自动调用。+load 是在一个类被初始装载时调用，+initialize 是在应用第一次调用该类的类方法或实例方法前调用的。两个方法都是可选的，并且只有在方法被实现的情况下才会被调用。
swizzling应该只在dispatch_once 中完成,由于swizzling 改变了全局的状态，所以我们需要确保每个预防措施在运行时都是可用的。原子操作就是这样一个用于确保代码只会被执行一次的预防措施，就算是在不同的线程中也能确保代码只执行一次。Grand Central Dispatch 的 dispatch_once满足了所需要的需求，并且应该被当做使用swizzling 的初始化单例方法的标准。
