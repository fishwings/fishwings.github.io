---
author: "fishwings"
date: 2022-02-15
title: "获取goroutine id 的两种实现"
tags: [
    "go",
]
categories: [
    "go",
]
---
﻿# 获取goroutine id 的两种实现

众所周知，在golang并发编程的一些情况下，需要打印一下goroutien的id号，怎么来实现呢？下面提供两种方法：

1. 从runtime.Stack()方法中获取方法栈，然后从栈帧处获取；
2. 获取运行时g指针，反解析出g的结构。g指针时保存在当前goroutine的TLS对象中。

## 1 解析方法栈

```golang

func GoID() int {
    var buf [64]byte
    n := runtime.Stack(buf[:], false)
    // 得到id字符串
    idField := strings.Fields(strings.TrimPrefix(string(buf[:n]), "goroutine "))[0]
    id, err := strconv.Atoi(idField)
    if err != nil {
        panic(fmt.Sprintf("cannot get goroutine id: %v", err))
    }
    return id
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202163715456.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ2Nzk0NTY=,size_16,color_FFFFFF,t_70#pic_center)




## 2 g指针方式

因为go的版本不同，goroutine的结构也不同，推荐使用第三方库来实现。[petermattis/goid](https://github.com/petermattis/goid)。

go get -u github.com/petermattis/goid  

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202163735209.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ2Nzk0NTY=,size_16,color_FFFFFF,t_70#pic_center)
