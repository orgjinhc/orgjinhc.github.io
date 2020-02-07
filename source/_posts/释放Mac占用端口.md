---
title: Mac系统释放被占用端口问题
author: 靳宏财
top: true
cover: true
toc: true
summary: 日常工作中,进场会遇到端口被占用情况,通过blog记录一下解决思路
categories: Mac
tags:
  - 问题合集
date: 2020-01-19 03:25:00
---

# 端口占用问题

结论:问题很常见,只要是涉及到独立进程的工具或应用都会涉及到默认的端口已经被占用情况,常见的手段就是释放对应端口或变更默认端口号

## 问题描述

### 问题复现

放假第一天,一定要从写Blog开始,但是当查看排版和样式的时候出现了一个小问题

![问题](https://lion-heart.online/blog/2020-01-20-080616.png)





## 解决思路

### 先查看所有端口/4000端口,是否存在被占用情况

![查看端口](http://lion-heart.online/blog/2020-01-20-080711.png)

马上定位问题,接下我们就来解决它

### 终断它,并重新查看

![](http://lion-heart.online/blog/2020-01-20-%E9%87%8A%E6%94%BE%E7%AB%AF%E5%8F%A3-%E9%87%8D%E6%96%B0%E6%9F%A5%E7%9C%8B.png)

4000端口已经中断,我们重新查看发现4000端口已无使用

## 重新启动

![启动成功](http://lion-heart.online/blog/2020-01-20-081710.png)

启动成功,搞定



## 附查看本机ip地址方法

### 一、通过终端查看

![终端查看](http://lion-heart.online/blog/2020-02-07-034458.png)

### 二、系统设置

![系统设置](http://lion-heart.online/blog/2020-02-07-034808.png)

![系统设置](http://lion-heart.online/blog/2020-02-07-034758.png)

![TCP:IP](http://lion-heart.online/blog/2020-02-07-034822.png)

