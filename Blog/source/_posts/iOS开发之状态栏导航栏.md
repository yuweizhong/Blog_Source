---
title: iOS开发之状态栏导航栏
date: 2017-08-26 18:41:44
tags:
---

# **iOS开发之状态栏导航栏**

---

`​`状态栏
======
## 全局设置
将info.plist文件的
```
Viewcontroller-based status bar ppearance
```
设置为NO，即可开启全局设置，也就是说你在VC中对状态栏的控制都将无效
```
//设置状态栏的字体颜色模式
> [[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleLightContent];
//设置状态栏是否隐藏
[[UIApplication sharedApplication] setStatusBarHidden:YES];
UIStatusBarStyleDefault和UIStatusBarStyleLightContent，前者是默认的黑色，而后者是白色。
```

## 分页面设置
将info.plist文件的
```
Viewcontroller-based status bar ppearance
```
设置为YES，即可开启由VC来控制状态栏的功能，在这种模式下，全局的设置将无效！！所以我们必须逐个页面对状态栏进行设置，否则状态栏将维持默认的黑色字体和默认为显示状态。
1) 当VC不在UINavigationController中时，在VC中添加一个方法

```
- (UIStatusBarStyle)preferredStatusBarStyle
{
 //返回白色
 return UIStatusBarStyleLightContent;
 //返回黑色
 //return UIStatusBarStyleDefault;
}
```
保险起见，在view的某个加载阶段比如viewWillAppear中，执行：
```
[self setNeedsStatusBarAppearanceUpdate];
```
2) 当VC在UINavigationController中时，VC并不能通过1)的方式控制状态栏的颜色，详见本文后面的参考资料，那么这个时候，有一个trick的方法可以在VC中间接的控制
```
self.navigationController.navigationBar.barStyle = UIBarStyleBlack;
```

##导航栏
===

导航栏背景控制 ：
```
[[UINavigationBar appearance] setBarTintColor:[UIColor yellowColor]];
```
导航栏背景图片 ：

```
[[UINavigationBar appearance] setBackgroundImage:[UIImage imageNamed:@ "nav_bg.png" ]  forBarMetrics:UIBarMetricsDefault]; （图片64px覆盖状态栏）
```

导航栏标题设置 ：
```
NSShadow *shadow = [[NSShadow alloc] init]; 
 shadow.shadowColor = [UIColor colorWithRed:0.0 green:0.0 blue:0.0 alpha:0.8]; 
 shadow.shadowOffset = CGSizeMake(0, 1); 
 [[UINavigationBar appearance] setTitleTextAttributes: [NSDictionary dictionaryWithObjectsAndKeys: 
 [UIColor colorWithRed:245.0/255.0 green:245.0/255.0 blue:245.0/255.0 alpha:1.0], NSForegroundColorAttributeName, 
 shadow, NSShadowAttributeName, 
 [UIFont fontWithName:@ "HelveticaNeue-CondensedBlack" size:21.0], NSFontAttributeName, nil]];
```
导航栏按钮颜色设置 ：
```
[[UINavigationBar appearance] setTintColor:[UIColor whiteColor]];
```
导航栏下控件定位 ：
```
self.automaticallyAdjustsScrollViewInsets =NO;（scrollView布局设置）
if (on) { (控件自动下移64)

 self.edgesForExtendedLayout = UIRectEdgeNone;

 }else{

 self.edgesForExtendedLayout = UIRectEdgeAll;

}
```
导航栏图标左移
```
//创建UIBarButtonSystemItemFixedSpace
UIBarButtonItem * spaceItem = [[UIBarButtonItem alloc]initWithBarButtonSystemItem:UIBarButtonSystemItemFixedSpace target:nil action:nil];
//将宽度设为负值
spaceItem.width = -15;
//将两个BarButtonItem都返回给NavigationItem
self.navigationItem.leftBarButtonItems = @[spaceItem,leftBarBtn];
```

全屏滑动：

```
id target = self.interactivePopGestureRecognizer.delegate;
// 创建全屏滑动手势，调用系统自带滑动手势的target的action方法
 UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:target action:@selector(handleNavigationTransition:)];
 // 设置手势代理，拦截手势触发
pan.delegate = self;
// 给导航控制器的view添加全屏滑动手势
  [self.view addGestureRecognizer:pan];
 // 禁止使用系统自带的滑动手势
self.interactivePopGestureRecognizer.enabled = NO;
```

