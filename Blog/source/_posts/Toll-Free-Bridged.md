---
title: Toll-Free Bridged
date: 2018-09-06 14:43:41
tags:
---

穿插一个小的知识点 介绍一下Toll-free bridged
>  Toll-free bridging,简称为TFB，是一种允许某些ObjC类与其对应的CoreFoundation类之间可以互换使用的机制。比如 NSString与CFString是桥接(bridged)的, 这意味着可以将任意NSString当做CFString使用，也可以将任意的CFString当做NSString使用。

那么既然有了NSString 为什么还需要CFString 的存在呢？
[官网](https://developer.apple.com/library/archive/documentation/CoreFoundation/Conceptual/CFDesignConcepts/Articles/tollFreeBridgedTypes.html#/apple_ref/doc/uid/TP40010677)解释：
> CFString provides a suite of efficient string-manipulation and string-conversion functions. It offers seamless Unicode support and facilitates the sharing of data between Cocoa and C-based programs
 
 翻译
> CFString提供了一套高效的字符串操作和字符串转换功能。 它提供无缝的Unicode支持，并促进Cocoa和基于C的程序之间的数据共享。

## Toll-Free Bridged Types

![TFB](https://raw.githubusercontent.com/yuweizhong/Resources/master/Img/tfb.png)