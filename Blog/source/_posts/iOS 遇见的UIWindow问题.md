---
title: 'UIWindow遇到的问题'
date: 2017-10-18 19:22:45
tags: UIWindow
---
最近项目里遇到了一个关于UIWindow的问题，便去查找了一下UIWIndow，故作此记录

**UIWindowLevel**
window层级主要有下面三类
```
UIKIT_EXTERN const UIWindowLevel UIWindowLevelNormal;
UIKIT_EXTERN const UIWindowLevel UIWindowLevelAlert;
UIKIT_EXTERN const UIWindowLevel UIWindowLevelStatusBar
```
**keyWindow**

在ios 中用 [UIApplication sharedApplication].windows可以获取应用内所有设置的window：
##### (1)UIWIndow
程序主要的keyWindow，程序展示的窗口
##### (2)UITextEffectsWindow
这是iOS8引入的一个新window，是键盘所在的window。它的windowLevel是10，高于UIWindowLevelNormal。
##### (3)UIRemoteKeyboardWindow
iOS9之后,新增了一个类型为 UIRemoteKeyboardWindow 的窗口用来显示键盘按钮。

当自定义添加window至页面上时（例如添加全局的悬浮窗口）每次弹窗 UIActionSheet或UIAlertView之后，再用[[UIApplication sharedApplication].keyWindow]获取keywindow时候，会取到最外层设置的UIWindow（项目中的全局悬浮窗口），就可能出现问题。

因为在打开UIActionSheet或UIAlertView弹窗时候，程序为了让弹框出现在最外层，会新建一个临时的UIWindow，并且层级最高，并设置为keywindow。在弹框消失后keywindow会自动变成下一层级的UIWindow（当前层级最高的UIwindow），即指向了项目中的全局悬浮窗口。 

**Add：使用iOS8后出现的UIAlertController不会出现上述问题**
