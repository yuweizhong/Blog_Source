---
title: OC Basis
date: 2017-11-03 16:41:44
tags:
---

### 1、Objective-C动态时特性
    OC语言动态时的特性主要表现在：动态类型、动态绑定、动态加载。
- 动态类型，区分于静态类型（int、float...），静态类型在编译时决定变量类型，类型不符合就会报错。动态类型即在运行时决定变量类型，OC中的id类型，任何对象都可以被id类型指针所指，到运行时才决定对象的类型。

- 动态绑定，基于动态类型，在某个实例对象被确定之后，其属性和方法也就被确定了。OC中的方法都是消息转发机制，当我们向一个对象发送息发送-respondsToSelector:或者 -instancesRespondToSelector:等来确定对象是否可以对某个方法做出响应时，在消息转发机制处理之前，对应的类 的+resolveClassMethod:和+resolveInstanceMethod:将会被调用，可以有机会动态地向类或者实例添加新的方法（避免崩溃），也即类的实现是可以动态绑定；isKindOfClass类似。

- 动态加载，主要是icon图片的加载会根据设备状态自动选取@2x @3x图片进行显示。

### 2、delegate、datasource&block
- delegate是委托回调，在OC中是一个类委托另一个类实现某个方法，一个对象会对他的delegate发送消息，执行delegate里的方法
- datasource是数据源，主要和数据内容有关，一般为cell的属性和各种构造函数，是对数据源的处理。
- block是简单的回调，相比delegate代码更加集中统一，写法更加简练，但要注意循环引用。在公共接口和方法较多情况下可以选择delegate，在异步回调（AFNetWorking...）或者简单的回调选择block，block涉及到栈内存和堆内存的拷贝，运行成本较大；而delegate只是一个对象指针的保存和回调。

### 3、让对象实现copy功能
实现copy功能，则需实现 NSCopying 协议。如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 NSCopying与 NSMutableCopying协议。
声明 NSCopying 协议
```
<NSCopying>
```
实现 NSCopying 协议
```
- (id)copyWithZone:(NSZone *)zone;
```

### 4、深拷贝&浅拷贝
- 浅拷贝：指针拷贝，内存地址并未改变，只是将对象内存地址多了一个引用，在拷贝结束之后，两个对象的值不仅相同，而且对象所指的内存地址都是一样的。

- 深拷贝：内容拷贝，拷贝一个对象的具体内容，拷贝结束之后，两个对象的值虽然是相同的，但是指向的内存地址是不同的。两个对象之间也互不影响，互不干扰。

- 非集合类对象：对一个不可变对象进行copy操作，只仅仅是指针复制，进行mutableCopy操作，是内容复制；而对一个可变对象进行copy和mutableCopy操作，都是内容复制。

-集合类对象：不可变对象copy相同，进行mutableCopy操作时，虽然数组内存地址发生了变化，但是数组元素的内存地址并没有发生变化（单层深拷贝）；对于可变对象，copy和mutableCopy都是内容复制，内存地址变化，但是数组元素的内存地址并没有发生变化（单层深拷贝）
集合类对象和非集合类对象的copy与mutablecopy
建议参照此篇分析 与网上大多数看法不一致 但个人感觉可信度较高 [https://joeshang.github.io/]

demo1:
NSString *origStr = @"123456";
NSString *strCopy = [origStr copy];
NSMutableString *strMtCopy = [origStr mutablecopy];

demo2:
NSMutableStr *origStr = [NSMutableString stringWithFormat:@"123456"];
NSString *strCopy = [origStr copy];
NSMutableStr *strMuCopy = [origStr mutablecopy];

demo3:
NSArray *arr = @[@"a", @"b",@"c"];
NSArray *arrCopy = [array copy];
NSMutableArray *arrayMCopy = [array mutableCopy];

demo4: 
NSMutableArray *arr =[ [NSMutableArray alloc] initWithObject:@"1",@"2",@"3",nil];
NSArray *arrCopy = [array copy]; 
NSMutableArray *arrayMCopy = [array mutableCopy];

### 5、进程&线程 同步&异步 并行&串行
1、进程是计算机操作系统分配资源的单位；线程是进程的基本执行单元，一个进程内的所有任务都在线程中执行；两者都是操作系统体积功的程序运行的基本单元，知识资源管理方式不同。进程有独立的地址空间，而线程只是一个进程中所执行的任务。线程有自己的堆栈空间和局部变量，没有单独的地址空间。一个进程内部可以有多个线程并发执行。
2、同步和异步是在线程单元中的概念。同步指在当前线程执行任务，不开启新的线程；异步指开启新的线程执行任务，不影响当前线程任务的执行。
3、并行与串行是线程内任务的执行方式。并行指多个任务同时执行，串行指按顺序执行，执行完一个任务后执行下一个（并发指处理多任务的能力，可以是多任务在同一时间段内执行，并不一定同时执行多个任务）。

### 6、线程间通信
1、NSThread
```
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait;
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait NS_AVAILABLE(10_5, 2_0);
```
2、GCD
```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
     
 })
 dispatch_async(dispatch_get_main_queue(), ^{
 
 })
```
GCD其他函数方法
```
//延迟方法
 dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(/*延迟时间*/  *NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
   }
//单例
  static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
       //只执行一次的代码
    });
 ```
3、NSOperation
```
//创建线程队列
NSOperstionQueue *queue =[[NSOperationQueue alloc] init];
NSOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
    //线程任务
}];
[queue addOperation：operation];
//获取main队列
NSOperstionQueue *mainQueue = [NSOperationQueue mainQueue];
[mainQueue addOperationWithBlock:^{
   //线程任务 
}];
 ```
 [多线程介绍可见这篇连接较详细](https://www.jianshu.com/p/6e6f4e005a0b)

 
### 7、iOS数据持久化
-  plist文件
-  Perference（UserDefaule）
-  NSKeyedArchiver（归档）：遵从NSCoding协议，实现解码和编码两个方法
-  SQLite3：使用三方库FMDB较为方便
-  CoreData

### 8、App生命周期
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    // Override point for customization after application launch.
    NSLog(@"didFinishLaunchingWithOptions");
    return YES;
}
                            
- (void)applicationWillResignActive:(UIApplication *)application
{
    // Sent when the application is about to move from active to inactive state. This can occur for certain types of temporary interruptions (such as an incoming phone call or SMS message) or when the user quits the application and it begins the transition to the background state.
    // Use this method to pause ongoing tasks, disable timers, and throttle down OpenGL ES frame rates. Games should use this method to pause the game.
    NSLog(@"WillResignActive");
}

- (void)applicationDidEnterBackground:(UIApplication *)application
{
    // Use this method to release shared resources, save user data, invalidate timers, and store enough application state information to restore your application to its current state in case it is terminated later. 
    // If your application supports background execution, this method is called instead of applicationWillTerminate: when the user quits.
    NSLog(@"DidEnterBackground");
}

- (void)applicationWillEnterForeground:(UIApplication *)application
{
    // Called as part of the transition from the background to the inactive state; here you can undo many of the changes made on entering the background.
    NSLog(@"WillEnterForeground");
}

- (void)applicationDidBecomeActive:(UIApplication *)application
{
    // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
    NSLog(@"DidBecomeActive");
}

- (void)applicationWillTerminate:(UIApplication *)application
{
    // Called when the application is about to terminate. Save data if appropriate. See also applicationDidEnterBackground:.
    NSLog(@"WillTerminate");
}
```
**首次启动程序**
didFinishLaunchingWithOptions
DidBecomeActive
**从前台到后台**
WillResignActive
DidEnterBackground
**从后台到前台**
WillEnterForeground
DidBecomeActive

### 9、NSCache&NSDictionary
两者基本用法相似。NSCache是线程安全的，可以指定缓存的限额，当缓存超出限额或者内存不足时NSCache会自动释放内存；NSMutableDictionary线程不安全。NScache相比功能更强大一些。
```
// 缓存中总共可以存储多少条
_cache.countLimit = 5;
// 缓存的数据总量为多少
_cache.totalCostLimit = 1024 * 5;

//delegate
//当缓存被移除的时候执行
- (void)cache:(NSCache *)cache willEvictObject:(id)obj{
    NSLog(@"缓存移除  %@",obj);
}
```

### 10、description
description方法默认返回对象的描述信息(默认实现是返回类名和对象的内存地址)，重写description方法之后可以用来输出对象内部变量的值。调用NSLog(@"%@", object);这会自动调用object的description方法来输出Object的描述信息.
**Attention：不要在description方法中同时使用%@和self，否则会陷入死循环，循环调用description方法**

### 11、ARC
   ARC：Automatic Reference Counting 自动引用计数，苹果引入的一种内存管理机制。只要是为了解决MRC下手动创建和释放内存而造成的内存泄漏程序崩溃等问题。
### 12、assign VS weak ；__block VS __weak
    assign用于修饰基础类型 float int 等等，
    weak用于修饰delegate block等对象类型，weak修饰的对象在对象废弃后会自动取消引用计数并置为nil，不会造成野指针；
    __block不管是ARC还是MRC模式下都可以使用，可以修饰对象，还可以修饰基本数据类型。__block对象可以在block中被重新赋值，__weak不可以。__weak只能在ARC模式下使用，也只能修饰对象（NSString），不能修饰基本数据类型（int）。
### 13、KVC KVO NSNotifiction delegate block
    KVC: key value coding，根据key 的值访问属性
    KVO：观察者模式，对某个对象添加观察 实现oberserveValueForKeyPath：
    NSNotifiction：一对多的通知方式
    delegate：代理，一对一模式，将某些方法教给代理去执行
    block：delegate的灵活化方式，函数式编程的一种形式
    KVO一般的使用场景是数据，需求是数据变化，比如股票价格变化，我们一般使用KVO（观察者模式）。delegate一般的使用场景是行为，需求是需要别人帮我做一件事情，比如买卖股票，我们一般使用delegate。Notification一般是进行全局通知，比如利好消息一出，通知大家去买入。delegate是强关联，就是委托和代理双方互相知道，你委托别人买股票你就需要知道经纪人，经纪人也不要知道自己的顾客。Notification是弱关联，利好消息发出，你不需要知道是谁发的也可以做出相应的反应，同理发消息的人也不需要知道接收的人也可以正常发出消息。

    一般在block中修改变量都需要事先加__block进行修饰。
    在非arc中，__block修饰的变量的引用计算是不变的。
    在arc中，会引用到，并且计算+1；
    非arc下可使用（arc直接使用__weak即可）
### 14、MVC vs MVVM
    ...
### 15、NSNumber 和 NSInteger
    NSNumber是类 ，而NSInteger是一个基本类型
### 16、如何重写类方法
    1、在子类中实现一个同基类名字一样的静态方法
    2、在调用的时候不要使用类名调用，而是使用[self class]的方式调用。原理，用类名调用是早绑定，在编译期绑定，用[self class]是晚绑定，在运行时决定调用哪个方法。
### 17、category/extension/继承
    category是在现有类的基础上添加新的方法，利用objective-c 的动态运行时分配机制，可以为现有类添加新方法。可以在分类中添加方法和成员变量，但是添加的成员变量不会自动生成setter和getter方法，需要在实现部分给出实现。
    extension就是匿名的分类，只有头文件没有实现文件。只能扩展方法，不能添加成员变量。扩展的方法只能在原类中实现。例如你扩展NSString,那么你只能在NSString的.m实现（这是不可能的），所以尽量少用扩展。用分类就可以了
    在iOS中继承是单继承，既只能有一个父类。在继承中，子类可以使用父类的方法和变量，当子类想对本类或者父类的变量进行初始化，那么需要重写init（）方法 。父类也可以访问子类的方法和成员变量。、
### 18、将一个自定义对象序列化至磁盘
	自定义对象实现<NSCoding>协议，实现-(instancetype)initWithCoder:(NSCoder *)aDecoder 方法和-(void)encodeWithCoder:(NSCoder *)aCoder方法
	使用NSKeyedArchiver进行读写就可以了。

