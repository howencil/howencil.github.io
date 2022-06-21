---
title: "Redis的数据淘汰策略"
date: 2022-06-21T15:18:49+08:00
draft: false
author: 插画师
tags: ["Redis"]
categories: ["Tech"]
---
# Redis的数据淘汰策略
### 1.缓存淘汰策略主要分为3类
1.  `noevition`
2.  `volatile`
3.  `allkeys`

### noevition：
`Redis`的默认策略。只接收读请求，不执行写请求（会直接返回错误）。
### volatile：
**volatile只淘汰设置了过期时间的key。**
1. **volatile-lru：** 根据lru算法淘汰设置了过期时间的key，**lru算法优先删除最近最少使用的key。**
2. **volatile-ttl:** 根据key的过期时间的长短 淘汰设置了过期时间的key，**过期时间越小的key优先被删除。**
3. **volatile-random:** **随机淘汰设置了过期时间的key。**

### allkeys
**allkeys对所有key无差别淘汰**
1. **allkeys-lru:** 根据lru算法淘汰所有的key，**最近最少使用的key优先被删除。**
2. **allkeys-random:** **随机淘汰所有的key。**

