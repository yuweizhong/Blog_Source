---
title: iOS设备旋转及页面横竖屏
date: 2018-09-28 13:59:41
tags: 横竖屏，Orientations 
---

在做SDK的时候遇到一个问题 ，要求页面进入SDK内部时候只支持竖屏，因此对横竖屏及设备旋转做一次记录...

## 1、项目工程整体设置
需要设置整个项目支持的设备方向：

![image](https://raw.githubusercontent.com/yuweizhong/Resources/master/Img/orientationsSetting.png)


## 2、项目内部个别页面支持旋转
```
// 是否支持旋转
- (BOOL)shouldAutorotate { 
    return YES;
} 
//支持旋转的方向（YES后调用）
- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return UIInterfaceOrientationMaskPortrait; 
} 
//一开始出现时候设备方向
- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation { 
    return UIInterfaceOrientationPortrait; 
}
```
设置完这个后，发现并没有什么效果，遂发现...

>  这个方法的设置必须在根控制器，如果不在跟控制器（UITabBarController 或 UINavigationController）中设置，VC中重写shouldAutorotate都不会调用的，并且往下push的VC中shouldAutorotate也都是不会调用的！


（由于又是SDK的缘故，然后想到了以下方案）

对UINavigationController添加了一个分类，实现了设备旋转的几个方法，这样，只需要在VC内部去实现下面几个方法即可....（贴代码）

```
/*
**UINavigationController + category
*/
// 是否支持旋转
- (BOOL)shouldAutorotate { 
    return [[self.viewControllers lastObject] shouldAutorotate];
} 
//支持旋转的方向（YES后调用）
- (NSUInteger)supportedInterfaceOrientations {
    return [[self.viewControllers lastObject] supportedInterfaceOrientations]; 
} 
//一开始出现时候设备方向
- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation { 
return [[self.viewControllers lastObject] preferredInterfaceOrientationForPresentation]; 
    
}

```

然后又出现问题了，当横屏进入页面时候，内部则会展示横屏页面下效果，这是我们不希望出现的。
> 因为 shouldAutorotate这些方法只会在 设备旋转时候调用，如果需要页面只支持竖屏，我们需要强制改变设备方向

## 2、项目内部个别页面强制横竖屏
（贴代码）
```
if([[UIDevice currentDevice] responseToSelector:@selector(setOrientation:)]){
        NSNumber *value = [NSNumber numberWithInt:UIInterfaceOrientationPortrait];
        [[UIDevice currentDevice] setValue:value forKey:@"orientation"];
    }
```
> 注：对页面强制旋转时候，需要设置shouldAutorotate方法YES，否则设备无法正常旋转，导致没有效果！一般再页面push 的时候进行设置，保证进入页面显示即为竖屏。

页面旋转方向参考

```
typedef NS_ENUM(NSInteger, UIDeviceOrientation) {   
    UIDeviceOrientationUnknown,    
    UIDeviceOrientationPortrait,            // Device oriented vertically, home button on the bottom  
    UIDeviceOrientationPortraitUpsideDown,  // Device oriented vertically, home button on the top   
    UIDeviceOrientationLandscapeLeft,       // Device oriented horizontally, home button on the right   
    UIDeviceOrientationLandscapeRight,      // Device oriented horizontally, home button on the left   
    UIDeviceOrientationFaceUp,              // Device oriented flat, face up    UIDeviceOrientationFaceDown             // Device oriented flat, face down
};
```
## 3、view transform
 在设置页面横竖屏的时候，也可以直接将view进行旋转动画的操作。（贴代码）
```
/设置statusBar
[[UIApplication sharedApplication] setStatusBarOrientation:orientation];

UIDeviceOrientation orientation = [UIDevice currentDevice].orientation;
if(UIDeviceOrientationIsLandscape(orientation)){
    [UIView animateWithDuration:0.35 delay:0.0 options:UIViewAnimationOptionBeginFormCurrentStat animations:^{
    //对navigationController.view 进行强制旋转
    self.navigationController.view.transform = (orientation == UIInterfaceOrientationLandscapeLeft)? CGAffineTransformMakeRotation(-M_PI/2) : CGAffineTransformMakeRotation(M_PI/2);
    self.navigationController.view.bounds = CGRectMake(0, 0, SCREEN_HEIGHT, SCREEN_WIDTH);
    } completion:nil];
}
```
> 对于隐藏导航栏的情况只需要对self.view进行强制旋转，以上旋转可能会导致iOS9 之后出现 状态栏不显示的情况，需要确保Info.plist中的【View controller-based status bar appearance】为YES，然后重写viewController的- (BOOL)prefersStatusBarHidden，返回值是NO即可。同时，View的强制旋转需要设置)shouldAutorotate方法返回NO

---

好了，记录调整一次设备横竖屏及设备旋转的问题....


