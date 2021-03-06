---
title: "限流算法中的令牌跟漏斗区别。Zuul作用"
date: 2022-06-07T16:21:10+08:00
draft: false
author: 插画师
tags: ["Java"]
categories: ["Tech"]
---

## 1.高并发3把利器
1. **缓存：** 缓存的目的是提升系统访问速度和增大系统处理容量；
2. **降级：** 降级是当服务器压力剧增的情况下，根据当前业务情况及流量对一些服务和页面有策略的降级，以此释放服务器资源以保证核心任务的正常运行；
3. **限流：** 限流的目的是通过对并发访问/请求进行限速，或者对一个时间窗口的请求进行限速来保护系统，一旦达到限制速率则可以拒绝服务、排队或者等等、降级等处理。
## 2.限流算法
常用的限流算法有**令牌桶**和**漏桶**。
#### 漏桶算法
把请求比作是水，水来了都先放进桶里，并以限定的速度出水，当水来得过猛而出水不够快时就会导致水直接溢出，即拒绝服务。

![](/限流算法中的令牌跟漏斗区别。Zuul作用/1.png)

漏斗有一个进水口和一个出水口，出水口以一定速率出水，并且有一个最大的出水速率：
**在漏斗中没有水的时候：**

- 如果进水速率小于等于最大出水速率，那么出水速率等于进水速率，此时不会积水；
- 如果进水速率大于最大出水速率，那么漏斗以最大出水速率出水，此时多余的水会积在漏斗中

**在漏斗中有水的时候：**
	- 出水口以最大速率出水
	- 如果漏斗未满，且有进水的话，那么这些水会记在漏斗中
	- 如果漏斗已满，且有进水的话，那么这些水会溢出到漏斗之外

#### 令牌桶算法
对于很多场景来说，除了要求能够限制数据的平均传输速率外，还要求某种程度的突发传输。这时候漏桶算法可能就不合适了，令牌桶算法更为合适。
	令牌桶算法的原理是*系统以恒定的速率产生令牌，然后把令牌放到令牌桶中，令牌桶有一个容量，当令牌桶满了的时候，再向其中放入令牌，那么多余的令牌将会被丢弃；当想要处理一个请求时，需要从令牌桶种取出一个令牌，如果此时令牌桶种没有令牌，那么则拒绝该请求*。



![](/限流算法中的令牌跟漏斗区别。Zuul作用/2.png)



​	令牌桶算法具有预消费的能力

## 3.令牌桶算法VS漏桶算法
- **令牌桶：**
	**生产令牌的速度是恒定的，而请求去拿令牌是没有速度限制的。这意味着，面对瞬时大流量，该算法可以在短时间内请求到大量令牌，而且拿令牌的过程并不是消耗很大的事情。**
- **漏桶：**
	**漏桶的出水速度是恒定的，那么意味着如果有瞬时大流量的话，将有大部分请求被丢弃（也就是桶溢出）。**

**不论是令牌桶还是漏桶，都是为了保证大部分流量能够正常使用，而牺牲掉了少部分流量，这是合理的。如果因为极少部分的流量需要保证的话，那么就有可能导致系统达到极限而挂掉，得不偿失。**

## 4.Zuul的作用
**Zuul是一个网关，网关的作用一般是：**
1. 对外提供统一的REST API接口，收缩所有的服务到服务网关后面；
2. 放置到对外访问的前端，可以做权限验证；
3. 可以做均衡负载器；
4. 可以做服务路由的功能。
