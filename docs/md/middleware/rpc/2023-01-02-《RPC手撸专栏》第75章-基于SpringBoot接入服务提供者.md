---
title: 第75章：基于SpringBoot接入服务提供者
pay: https://articles.zsxq.com/id_8fwr1wu3jx9p.html
---

# 《RPC手撸专栏》第75章：基于SpringBoot接入服务提供者

作者：冰河
<br/>星球：[http://m6z.cn/6aeFbs](http://m6z.cn/6aeFbs)
<br/>博客1：[https://binghe001.github.io](https://binghe001.github.io)
<br/>博客2：[https://binghe.gitcode.host](https://binghe.gitcode.host)
<br/>文章汇总：[https://binghe.gitcode.host/md/all/all.html](https://binghe.gitcode.host/md/all/all.html)

> 沉淀，成长，突破，帮助他人，成就自我。

**大家好，我是CurleyG~~**

在整合Spring篇章，我们已经实现了服务提供者整合Spring，并基于Spring XML和注解的方式实现了如何接入服务提供者。同时，也实现了服务消费者整合Spring，并且基于Spring XML和注解的方式实现了接入服务消费者的功能。在整合SpringBoot的篇章，我们实现了服务提供者整合SpringBoot的功能。

## 一、前言

`如何基于SpringBoot接入服务提供者呢？`

目前，我们自己手写的RPC框架已经完成了整体设计、服务提供者的实现、服务消费者的实现、注册中心的实现、负载均衡的实现、SPI扩展序列化机制、SPI扩展动态代理机制、SPI扩展反射机制、SPI扩展负载均衡策略、SPI扩展增强型负载均衡策略、SPI扩展实现注册中心、心跳机制、增强型心跳机制、重试机制和整合Spring等篇章。

在整合SpringBoot的篇章中，实现了服务提供者整合SpringBoot的功能。

## 二、目标

`目标很明确：基于SpringBoot接入服务提供者!`

在整合SpringBoot篇章中，我们一起实现了服务提供者整合SpringBoot，但是只是实现了RPC框架的服务提供者整合SpringBoot的功能，整合后怎么使用，也就是还没有实现如何在基于SpringBoot开发的真实项目中如何使用RPC框架。

本章，我们就一起实现基于SpringBoot接入服务提供者。

## 三、设计

`如果让你设计基于SpringBoot接入服务提供者的流程，你会怎么设计呢？`

基于SpringBoot接入服务提供者的流程如图75-1所示。

![图75-1](https://binghe.gitcode.host/assets/images/middleware/rpc/rpc-2023-01-02-001.png)

由图75-1可以看出如下信息：

（1）服务提供者会通过自定义类扫描器整合注册中心，将服务注册到注册中心。

（2）服务注册到注册中心的元数据，例如服务的名称、服务的版本号、服务地址、服务端口和服务分组等信息，元数据会贯穿整个服务的注册与发现流程。

（3）服务注册与发现SPI接口对外提供服务注册与发现的方法，服务提供者通过自定义扫描器会调用服务注册与发现SPI接口的方法实现服务注册功能。

（4）基于服务注册与发现的SPI接口，服务提供者会基于SPI接口实现多个服务注册与发现的实现类，每个实现类对应着一种注册中心服务。

（5）服务消费者会通过服务注册与发现的SPI接口订阅注册中心的服务，会从注册中心获取到服务提供者发布的服务信息，实现服务发现的功能。

（6）服务消费者从注册中心获取到服务提供者发布的服务信息后，会基于SPI机制动态加载普通算法（我们将第42章~第50章实现的负载均衡算法统称为普通算法）、基于增强型加权随机算法、基于增强型加权轮询算法、基于增强型加权Hash算法、基于增强型加权源IP地址Hash算法、基于增强型Zookeeper一致性Hash算法和最少连接数算法的负载均衡策略，从多个服务中选择一个进行远程网络连接。

（7）服务消费者会直接与根据基于SPI机制动态加载的负载均衡策略选择出的服务提供者建立连接，实现数据交互。也就是说，后续服务消费者会与服务提供者直接实现数据交互。

（8）服务消费者向服务提供者发送心跳ping消息，服务提供者响应服务消费者pong消息。服务提供者向服务消费者发送心跳ping消息，服务消费者向服务提供者响应pong消息。

（9）服务消费者发送心跳和服务提供者发送心跳，定时任务的时间间隔都是配置化的。

（10）服务提供者与服务消费者除了手动实现定时任务来实现心跳检测外，还基于Netty的IdleStateHandler实现了心跳检测机制。

（11）服务消费者支持服务订阅的重试机制、服务消费者连接服务提供者支持重试机制。

（12）服务提供者支持以Java原生程序方式和整合Spring的方式提供服务，并且实现了基于Spring XML和Spring注解的方式接入RPC框架的服务提供者。

（13）服务消费者支持以Java原生程序方式和整合Spring的方式提供服务。并且实现了基于Spring XML和Spring注解的方式接入RPC框架的服务消费者。

（14）服务提供者支持整合SpringBoot的功能，并支持基于SpringBoot接入服务提供者。

## 四、实现

`说了这么多，具体要怎么实现呢？`

### 核心类实现关系

基于SpringBoot接入服务提供者的核心类关系如图75-2所示。

![图75-2](https://binghe.gitcode.host/assets/images/middleware/rpc/rpc-2023-01-02-002.png)

## 查看完整文章

加入[冰河技术](http://m6z.cn/6aeFbs)知识星球，解锁完整技术文章与完整代码
