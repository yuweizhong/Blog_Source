---
title: iOS webview实践优化
date: 2018-11-21 14:45:55
tags: webview
---

苹果为开发者提供给内嵌webview，UIWebView在iOS2.0之后就存在了，在iOS8.0之后苹果又为我们提供了WKWebView，性比来说，WKWebview性能高，稳定性好，占用的内存比较小。对于支持iOS8之后的应用，可以考虑使用WKWebView替换使用。

同时，在App开发中，内嵌WebView始终占有着一席之地。它能以较低的成本实现Android、iOS和Web的复用（跨平台），也可以冠冕堂皇的突破苹果对热更新的封锁。然而便利性的同时，WebView的性能体验却备受质疑，导致很多客户端中需要动态更新等页面时不得不采用其他方案（WEEX、ReactNative...）

---

## WebView性能

从性能上来看，经常能听到很多人吐槽webview：白屏、打开速度比native慢、一直loading.....
从用户角度来看，加载webview大致经历一下几个过程：
- 交互无反馈
- 到达新的页面，页面白屏
- 页面基本框架出现，但是没有数据；页面处于loading状态
- 出现所需的数据

从程序角度看，初始化webview则经历一下几个过程：
![image]()

因此，如何优化Webview，就从缩小这几个部分的时间来看。


## WebView初始化

 当App首次打开时，默认是并不初始化浏览器内核的；只有当创建WebView实例的时候，才会创建WebView的基础框架。所以与浏览器不同，App中打开WebView的第一步并不是建立连接，而是启动浏览器内核。

在浏览器中，我们输入地址时（甚至在之前），浏览器就可以开始加载页面。
而在客户端中，客户端需要先花费时间初始化WebView完成后，才开始加载。

[手机QQ Hybrid架构](https://mp.weixin.qq.com/s/evzDnTsHrAr2b9jcevwBzA?)


## 建立连接/服务器处理

- DNS采用和客户端API相同的域名------客户端首次打开都会请求某个域名，其DNS将会被系统缓存，当重新打开WebView的时候，如果请求了不同的域名，需要重新获取对应域名的IP，也是一部分时间的消耗。
- Eag && Last-Modified-----webview缓存策略优化，判断服务器数据更新状态判断是否加载本地缓存
- Transfer-Encoding: chunked -----如果采用普通方式输出页面，则页面会在服务器请求完所有API并处理完成后开始传输。浏览器要在后端所有API都加载完成后才能开始解析。如果采用chunk-encoding: chunked，并优先将页面的静态部分输出；然后处理API请求，并最终返回页面，可以让后端的API请求和前端的资源加载同时进行。两者的总共后端时间并没有区别，但是可以提升首字节速度，从而让前端加载资源和后端加载API不互相阻塞 
- 前端框架、脚本执行、js/css标签格式、js解析/编译/执行......


## webview内存消耗


## webview 安全


参考 
    [WebView性能、体验分析与优化](https://tech.meituan.com/WebViewPerf.html)
    [手机QQ Hybrid架构](https://mp.weixin.qq.com/s/evzDnTsHrAr2b9jcevwBzA?)