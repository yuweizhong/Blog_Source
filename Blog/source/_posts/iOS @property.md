---
layout: 'ios'
title: 'iOS @property'
date: 2017-10-06 15:50:36
tags: iOS property copy weak strong
---
对于property，iOS开发者并不会陌生，无时无刻不在接触可是**@property 的本质是什么？**
> @property = ivar + getter + setter;


“属性” (property)有两大概念：ivar（实例变量）、存取方法（access method ＝ getter + setter）
“属性” (property)作为 Objective-C 的一项特性，主要的作用就在于封装对象中的数据。 Objective-C 对象通常会把其所需要的数据保存为各种实例变量。实例变量一般通过“存取方法”(access method)来访问。其中，“获取方法” (getter)用于读取变量值，而“设置方法” (setter)用于写入变量值。这个概念已经定型，并且经由“属性”这一特性而成为 Objective-C 2.0 的一部分。 
property在runtime中是objc_property_t定义如下:
```
typedef struct objc_property *objc_property_t;
```
而objc_property是一个结构体，包括name和attributes，定义如下：
```
struct property_t {
     const char *name;
     const char *attributes;
 };
```
而attributes本质是objc_property_attribute_t，定义了property的一些属性，定义如下：
```
/// Defines a property attribute
typedef struct {
    const char *name;           /**< The name of the attribute */
    const char *value;          /**< The value of the attribute (usually empty) */
} objc_property_attribute_t;
```

#### @property属性关键字
-  nonatomic ---原子性
在默认情况下，由编译器合成的方法会通过锁定机制确保其原子性(atomicity)。如果属性具备 nonatomic 特质，则不使用自旋锁。请注意，尽管没有名为“atomic”的特质(如果某属性不具备 nonatomic 特质，那它就是“原子的” ( atomic) )，但是仍然可以在属性特质中写明这一点，编译器不会报错。若是自己定义存取方法，那么就应该遵从与属性特质相符的原子性。

- readwrite(读写)、readonly (只读) --- 读/写权限

- assign、strong、 weak、unsafe_unretained、copy --- 内存管理语义
1. assign
简单赋值，不更改索引计数。 对基础数据类型 （例如NSInteger，CGFloat）和C数据类型（int, float, double, char, 等），适用简单数据类型。
2. strong(强引用)
ARC下的一个关键字。被强引用指向的内存不会被释放。强引用会对引用计数+1，从而拓展对象的生命周期。（retain调用的ARC版本）
3. weak弱引用：
弱引用是一种特殊的引用类型。它不会增加引用计数因而不会拓展对象的生命周期。当没有强引用指向对象时，弱引用会被置为nil。在对象回收时，weak具有安全性——指针将自动被设置为nil，避免野指针的出现
4. _unsafe_unretained:
与weak类似，只是没有强引用指向对象时，_unsafe_unretained不会被置为nil；
5. copy
建立一个索引计数为1的对象，然后释放旧对象。一般适用于存在可变对象的类型（例 NSString NSDictionary...）
6. atomic
atomic意为操作是原子的，意味着只有一个线程访问实例变量.属性的存取方法是线程安全的,很少使用，因为比较影响效率，性能比nonatomic差。
7. nonatomic
非原子的，可以被多个线程访问。它的效率比atomic快。禁止多线程，变量保护，提高性能，不能保证在多线程环境下的安全性，在单线程和明确只有一个线程访问的情况下广泛使用。
8.@synthesize 和 @dynamic
@property 有两个对应的词，一个是 @synthesize，一个是 @dynamic。如果 @synthesize 和 @dynamic都没写，那么默认的就是 @syntheszie var = _var;
@synthesize ：如果你没有手动实现 setter 方法和 getter 方法，那么编译器会自动为你加上这两个方法。
```
@syntheszie var = _var;
```
@dynamic 告诉编译器：属性的 setter 与 getter 方法由用户自己实现，不自动生成。假如一个属性被声明为 @dynamic var，然后你没有提供 @setter 方法和 @getter 方法，编译的时候没问题，但是当程序运行到 instance.var = someVar，由于缺 setter 方法会导致程序崩溃；或者当运行到 someVar = var 时，由于缺 getter方法同样会导致崩溃。编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定。


**A. 什么情况使用 weak 关键字，相比 assign 有什么不同？**

##### 什么情况使用 weak 关键字？
在 ARC 中,在有可能出现循环引用的时候,往往要通过让其中一端使用 weak 来解决,比如: delegate 代理属性
自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用 weak,自定义 IBOutlet 控件属性一般也使用 weak；当然，也可以使用strong

##### 不同点：
weak 此特质表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同assign类似， 然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。 而 assign 的“设置方法”只会执行针对“纯量类型” (scalar type，例如 CGFloat 或 NSlnteger 等)的简单赋值操作。

assign 可以用非 OC 对象,而 weak 必须用于 OC 对象

**B. 怎么用 copy 关键字？**

用途：

NSString、NSArray、NSDictionary 等等经常使用copy关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary，他们之间可能进行赋值操作，为确保对象中的字符串值不会无意间变动，应该在设置新属性值时拷贝一份。
block 也经常使用 copy 关键字,block 使用 copy 是从 MRC 遗留下来的“传统”,在 MRC 中,方法内部的 block 是在栈区的,使用 copy 可以把它放到堆区.在 ARC 中写不写都行；

copy 此特质所表达的所属关系与 strong 类似。然而设置方法并不保留新值，而是将其“拷贝” (copy)。 当属性类型为 NSString 时，经常用此特质来保护其封装性，因为传递给设置方法的新值有可能指向一个 NSMutableString 类的实例。这个类是 NSString 的子类，表示一种可修改其值的字符串，此时若是不拷贝字符串，那么设置完属性之后，字符串的值就可能会在对象不知情的情况下遭人更改。所以，这时就要拷贝一份“不可变” (immutable)的字符串，确保对象中的字符串值不会无意间变动。只要实现属性所用的对象是“可变的” (mutable)，就应该在设置新属性值时拷贝一份。

**C.用@property声明的NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？**

str1使用copy关键字 str2使用strong关键字
```
-(void)test
{
    NSMutableString*str=[NSMutableString stringWithFormat:@"helloworld"];
    self.str1=str;
    self.str2=str;
    NSLog(@"str:%p--%p",str,&str);
    NSLog(@"copy_str:%p--%p",_str1,&_str1);
    NSLog(@"strong_str:%p--%p",_str2,&_str2);
}
```
运行结果

```
2017-09-16 15:21:55.910 APP[1236:78490] str:0x600000075cc0--0x7fff5bd7c9d8
2017-09-16 15:21:55.910 APP[1236:78490] copy_str:0x600000026e60--0x7fc66cc06358
2017-09-16 15:21:55.911 APP[1236:78490] strong_str:0x600000075cc0--0x7fc66cc06360
```
经过copy关键字修饰过的str1没有因为str的变化而变化，经过strong修饰过的str2却随str的改变而改变。对于希望字符串的值不跟着str变化，使用用copy来设置string的属性。如果希望字串的值跟着赋值的字串的值变化，可以使用strong。

**D.为什么不用assign修饰一个对象，而可以修饰基本数据类型？
assign修饰的对象（编译的时候会产生警告：Assigning retained object to unsafe property; object will be released after assignment）在释放之后，指针的地址还是存在的，并没有被置为nil，造成野指针。对象分配在堆上的某块内存，如果在后续的内存分配中，刚好分到了这块地址，程序就会crash。而基础数据类型是分配在栈上，栈的内存会由系统自己自动处理回收，不会造成野指针。

