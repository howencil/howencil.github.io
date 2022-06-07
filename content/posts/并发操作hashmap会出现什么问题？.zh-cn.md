---
title: "并发操作hashmap会出现什么问题？"
date: 2022-06-07T15:13:07+08:00
draft: false
author: 插画师
tags: ["Java"]
categories: ["Tech"]
---

> [HashMap多线程并发问题分析-正常和异常的rehash1(阿里) - aspirant - 博客园](https://www.cnblogs.com/aspirant/p/11504389.html)

## 1.可能引发的问题
1. 多线程put后可能导致get死循环；
2. 多线程put时可能会导致元素丢失。

## 2.三种解决方案
1. 用HashTable替换HashMap；
2. 用Collections.synchronizedMap将HashMap包装起来；
3. ConcurrentHashMap替换HashMap。
