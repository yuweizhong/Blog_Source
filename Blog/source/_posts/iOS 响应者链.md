---
title: iOS 响应者链
date: 2018-04-26 16:32:10
tags: UIResponder
---
# iOS 响应者链

谈到iOS内的响应者链，首先想到的肯定是 UIResponder对象，用户点击屏幕，传递响应事件，直至对响应事件做出反应。

而这其中具体的事件响应又是如何呢？

UIResponder对象
=============

UIResponder是事件的响应类，里面提供了那些需要响应并处理事件的一组接口，包括触摸事件（touch events）和运动事件（motion events）。包括我们熟悉的UIKit类中的UIApplication、UIView、UIViewController等几个类都是直接继承与UIResponder类，这些对象都是一系列响应对象（响应者）。

UIResponder对象有一个nextResponder属性

`@property(nonatomic, readonly, nullable) UIResponder *nextResponder;`

对于nextResponder的理解，应该是这样的：

* UIView

  UIView的nextResponder是管理该view的VC或是其父视图superView
* UIViewController
* 若该VC是由某个VC presented的，则该VC的nextResponder则是发起presented的那个VC；
* 其余情况下VC的nextResponder则是UIWindow；
* UIWindow

  其nextResponder为UIApplication对象
* 
* UIApplication

 其nextResponder为AppDelegate，对于AppDelegate对象则返回nil

响应链工作
-----

 UIView中有一个方法

`- (nullable UIView *)hitTest:(CGPoint)point withEvent:(nullable UIEvent *)event;`

**UIKit就是通过基于视图的hit-testing来确定touch事件发生的位置，再将touch的位置和视图层级中的view边界进行对比，判断事件的响应。**

而符合响应者的要求包括以下几条：

* 响应者 hidden属性不为YES；
* 响应者 透明度不为 0；
* 响应者的 userInteractionEnabled 不为NO;
* 响应者在事件touch区域内；
* 遍历响应者subview，从上至下遍历，直至找到合适的view作响应者。

例如对于以下层级视图，当用户选择其重叠区域点击时，响应者流程如下：

![ViewController_RedViewAndGreenView.png](https://raw.githubusercontent.com/yuweizhong/Resources/master/Img/redAndGreen.png)

1. AppDelegate的 window收到事件，开始执行hitTest方法，符合要求，开始遍历子view；
2. 事件传到VC.view上，并开始执行hitTest方法，符合要求，开始遍历子view；
3. VC.view上有两个子view(RedView和GreenView)，GreenView在RedView之上，所以GreenView先执行hitTest方法，若符合响应者要求，（开始遍历子view，没有子View）则返回GreenView；若GreenView不符合响应者要求，则RedView执行hitTest方法，判断是否符合响应者要求.....
4. 返回符合响应者要求的View

