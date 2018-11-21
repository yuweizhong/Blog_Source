---
title: ReadyForBAT---UI
date: 2018-10-10 20:25:07
tags:
---

## 生命周期
1. App生命周期
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions{}
                            
- (void)applicationWillResignActive:(UIApplication *)application{}

- (void)applicationDidEnterBackground:(UIApplication *)application{}

- (void)applicationWillEnterForeground:(UIApplication *)application{}

- (void)applicationDidBecomeActive:(UIApplication *)application{}

- (void)applicationWillTerminate:(UIApplication *)application{}
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

2. UIViewController生命周期
```
//非storyboard和xib创建执行初始化操作走该方法
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil {}

//storyboard和xib创建执行初始化操作走该方法
- (instancetype)initWithCoder:(NSCoder *)aDecoder {}

//storyboard和xib加载完成
- (void)awakeFromNib{}

//加载视图 UIViewController对象的view属性被访问到且为空的时候调用，一般由系统调用，用于自定义控制器的view ，[super loadView]默认会创建一个空白的UIView，所以重写方法时不要调用父类方法，避免不必要的开销
- (void)loadView{}

//view载入内存中，可以进行一些初始化操作
- (void)viewDidLoad{}

//view即将展现
- (void)viewWillAppear{}

// view 即将布局Subviews
- (void)viewWillLayoutSubviews {} 

// view 已经布局 Subviews 
- (void)viewDidLayoutSubviews {}

//view显示
- (void)viewDidAppear{}

//view即将消失
- (void)viewWillDisAppear{}

//view消失
- (void)viewDidDisAppear{}

//出现内存警告
- (void)didReceiveMemoryWarning{}

//销毁
- (void)dealloc
```

## 事件传递和响应机制
**事件的产生**
发生触摸事件后，系统会将该事件加入到一个由UIApplication管理的事件队列中,根据队列的特点(FIFO)先产生的事件先处理;

**事件的传递**
- [ ] 事件是从父控件传递到子控件，由UIApplication ->UIWindow -> View

1. 首先判断主窗口（keyWindow）自己是否能接受触摸事件
2. 判断触摸点是否在自己身上
3. 子控件数组中从后往前遍历子控件，重复前面的两个步骤（所谓从后往前遍历子控件，就是首先查找子控件数组中最后一个元素，然后执行1、2步骤）
4. view，比如叫做fitView，那么会把这个事件交给这个fitView，再遍历这个fitView的子控件，直至没有更合适的view为止。
5. 如果没有符合条件的子控件，那么就认为自己最合适处理这个事件，也就是自己是最合适的view。

**事件的响应**
1. 首先判断initial view能否处理这个事件，如果不能则会将事件传递给其上级视图（inital view的superView）；
2. 如果上级视图仍然无法处理则会继续往上传递；一直传递到视图控制器view controller，首先判断视图控制器的根视图view是否能处理此事件；如果不能则接着判断该视图控制器能否处理此事件，如果还是不能则继续向上传 递；（对于第二个图视图控制器本身还在另一个视图控制器中，则继续交给父视图控制器的根视图，如果根视图不能处理则交给父视图控制器处理）；一直到 window，如果window还是不能处理此事件则继续交给application处理，如果最后application还是不能处理此事件则将其丢弃

可见事件的传递和事件的响应顺序正好是相反的 一个是由父视图向下传递，一个是向上传递。

> hitTest:withEvent:方法---事件传递工程中 通过调用子控件的hitTest:withEvent:方法来判断是否由最合适的view来响应事件。所以可以通过重写在父控件的hitTest:withEvent:中返回子控件作为最合适的view！来达到手动控制拦截的目的

> pointInside:withEvent:方法判断点在不在当前view上（方法调用者的坐标系上）如果返回YES，代表点在方法调用者的坐标系上;返回NO代表点不在方法调用者的坐标系上，那么方法调用者也就不能处理事件。

参考 https://www.jianshu.com/p/2e074db792ba

## UIView和 CALayer


## UITableView
1. a.将cell及它的子控件设置为不透明的。b.尽量少用或不用透明图层。c.减少子控件的数量。d.尽量少用addView给Cell动态添加View，可以初始化时就添加，然后通过hide来控制是否显示。e如果Cell内现实的内容来自web,使用异步加载,缓存请求结果这个在我的《iOS多图下载案例》系列文章中有具体讲解原理，这个采用sdwebimage这个第三方框架即可。这几个都是通过轻量级cell来对tableview进行优化。
2. 正确使用reuseIdentifier来重用Cell。
3. 提前计算并缓存好高度（布局），因为heightForRowAtIndexPath:是调用最频繁的方法。
4. 滑动的UITableView时，按需加载对应的内容
