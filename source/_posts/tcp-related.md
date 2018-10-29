---
title: 从socket api的几个方法看tcp机制
date: 2018-10-29 20:37:21
tags: tcp
---
## 背景问题
在使用rpc框架时，设置socket参数时会使用到下面的api：
soccket.setReceiveBufferSize()
socket.setSendBufferSize()

socket.setTcpNoDelay()
socket.setSoLinger()
socket.setSoTimeout()

socket.setKeepAlive()
socket.setTrafficClass()
socket.setReuseAddress()
socket.setPerformancePreferences()

这些参数对我们在进行网络编程时起到了什么作用，不同的参数设置会有什么影响程序的表现呢？


## tcp协议

#### 拥塞控制



#### 滑动窗口&流量控制



## 应用

#### rpc框架中的使用



