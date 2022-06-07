---
title: "ForkJoinPool和一般的线程池有什么区别？"
date: 2022-06-07T15:03:47+08:00
draft: false
author: 插画师
tags: ["Java"]
categories: ["Tech"]
---

## 说在前面
首先是结论：
1. ForkJoinPool不是为了替代ExecutorService，而是他的补充，在某些场景下性能比ExecutorService更好；
2. ForkJoinPool主要用于实现“分而治之“的算法，特别是分治之后递归调用的函数，例如quick sort等；
3. ForkJoinPool最适合的是计算密集型的任务（也就是CPU密集型），如果存在IO，线程间同步，sleep()等会造成线程长时间阻塞的情况时，最好配合使用ManagedBlocker。
