---
layout: post
category: curleyg-code-rpc
title: 第fix-07章：更新基于Semaphore的限流策略
tagline: by CurleyG
tag: [rpc,mykit-rpc,curleyg-code-rpc]
excerpt: 第fix-07章：更新基于Semaphore的限流策略
lock: need
---

# 《RPC手撸专栏》第fix-07章：更新基于Semaphore的限流策略

作者：冰河
<br/>星球：[http://m6z.cn/6aeFbs](http://m6z.cn/6aeFbs)
<br/>博客1：[https://binghe001.github.io](https://binghe001.github.io)
<br/>博客2：[https://binghe.gitcode.host](https://binghe.gitcode.host)
<br/>文章汇总：[https://binghe.gitcode.host/md/all/all.html](https://binghe.gitcode.host/md/all/all.html)

> 沉淀，成长，突破，帮助他人，成就自我。

**大家好，我是CurleyG~~**

在写《RPC手撸专栏》的过程中，针对专栏版本的代码，在书写的过程中，会提前埋一些坑进去，使各位星球的小伙伴在调试代码的过程中，能够自己去发现问题，并且分析问题，最好也能够自己解决问题。经过自己发现问题->分析问题->解决问题的过程，能够提升大家对于RPC框架源码的参与过程，更重要的是，能够不断提升大家自己发现问题、分析问题和解决问题的能力，这种能够力才是程序员最核心的竞争力。

## 一、问题描述

`本章要解决什么问题呢？`

在设计上，基于Semaphore的限流策略，要实现的逻辑就是在一段时间内，最多允许通过多少个请求，如果请求的数量在这段时间内，达到了上限，就需要触发限流的操作。但是，在实际实现过程中，会出现在这段时间内，达到请求数量的上限时，没有触发限流的问题。

## 二、问题分析

`这个问题是如何产生的呢？`

问题的分析和第fix-05章基于计数器的限流策略中分析问题的流程一样，大体流程还是如下所示。

要想分析问题产生的根源，我们先来看看服务提供者和服务消费者是如何整合服务限流的，在服务提供者和服务消费中，会通过如下的代码逻辑来整合服务限流。

**（1）初始化SPI接口对象**

```java
private boolean enableRateLimiter;
private RateLimiterInvoker rateLimiterInvoker;
private void initRateLimiter(String rateLimiterType, int permits, int milliSeconds) {
    if (enableRateLimiter){
        rateLimiterType = StringUtils.isEmpty(rateLimiterType) ? RpcConstants.DEFAULT_RATELIMITER_INVOKER : rateLimiterType;
        this.rateLimiterInvoker = ExtensionLoader.getExtension(RateLimiterInvoker.class, rateLimiterType);
        this.rateLimiterInvoker.init(permits, milliSeconds);
    }
}
```

**（2）整合限流功能**

```java
if (enableRateLimiter){
    if (rateLimiterInvoker.tryAcquire()){
        try{
            //限流获取到资源后，执行后续的业务逻辑
        }finally {
            rateLimiterInvoker.release();
        }
    }else {
        //执行限流失败的处理策略
    }
}
```

可以看到，在限流操作中，只有获取到资源后，才能继续执行业务逻辑，执行完毕后，会在finally{}代码块中释放资源。

**注意：服务提供者整合服务限流的逻辑位于：bhrpc-provider-common工程下的io.binghe.rpc.provider.common.handler.RpcProviderHandler类中，而服务提供者整合服务限流的逻辑位于：bhrpc-proxy-api工程下的io.binghe.rpc.proxy.api.object.ObjectProxy类中，大家自行查阅对应源码工程中的代码，这里不再赘述。**

接下来，我们再来看看基于计数器的限流策略实现类SemaphoreRateLimiterInvoker，SemaphoreRateLimiterInvoker类的源码详见：bhrpc-ratelimiter-semaphore工程下的io.binghe.rpc.ratelimiter.semaphore.SemaphoreRateLimiterInvoker，修改前的源码如下所示。

```java
@SPIClass
public class SemaphoreRateLimiterInvoker extends AbstractRateLimiterInvoker {
    private final Logger logger = LoggerFactory.getLogger(SemaphoreRateLimiterInvoker.class);
    private Semaphore semaphore;

    @Override
    public void init(int permits, int milliSeconds) {
        super.init(permits, milliSeconds);
        this.semaphore = new Semaphore(permits);
    }

    @Override
    public boolean tryAcquire() {
        logger.info("execute semaphore rate limiter...");
        return semaphore.tryAcquire();
    }

    @Override
    public void release() {
        semaphore.release();
    }
}
```

结合前面的代码分析得出结论：当请求获取到限流资源，执行完业务逻辑后，就会在finally代码块中调用release()方法释放资源，则后续的请求就会再次获取到限流资源，继续执行后续的业务逻辑，达不到限流的效果。

## 三、问题解决

`问题该如何解决呢？`

分析完问题后，解决问题的方案就变得简单了，我们在SemaphoreRateLimiterInvoker类的tryAcquire()方法中加入时间窗口的逻辑，在同一个时间窗口内最多允许通过permits个请求，不对semaphore进行释放资源的操作。当在同一个时间窗口内，请求数达到上限时，就会触发限流逻辑。在同一个时间窗口结束时，统一释放semaphore占用的资源。

修改后的SemaphoreRateLimiterInvoker类的源码如下所示。

```java
@SPIClass
public class SemaphoreRateLimiterInvoker extends AbstractRateLimiterInvoker {
    private final Logger logger = LoggerFactory.getLogger(SemaphoreRateLimiterInvoker.class);
    private Semaphore semaphore;
    private final AtomicInteger currentCounter = new AtomicInteger(0);
    private volatile long lastTimeStamp = System.currentTimeMillis();
    @Override
    public void init(int permits, int milliSeconds) {
        super.init(permits, milliSeconds);
        this.semaphore = new Semaphore(permits);
    }
    @Override
    public boolean tryAcquire() {
        logger.info("execute semaphore rate limiter...");
        //获取当前时间
        long currentTimeStamp = System.currentTimeMillis();
        //超过一个时间周期
        if (currentTimeStamp - lastTimeStamp >= milliSeconds){
            //重置窗口开始时间
            lastTimeStamp = currentTimeStamp;
            //释放所有资源
            semaphore.release(currentCounter.get());
            //重置计数
            currentCounter.set(0);
        }
        boolean result = semaphore.tryAcquire();
        //成功获取资源
        if (result){
            currentCounter.incrementAndGet();
        }
        return result;
    }
    @Override
    public void release() {
        //TODO ignore
    }
}
```

至此，问题解决。

## 四、问题总结

`修改完问题不总结下怎么行？`

我们自己手写的RPC框架不是一蹴而就的，它是一个不断优化和不断调整的过程，冰河也会将这些调整的过程整理好分享给各位星球的小伙伴。

总之，我们写的RPC框架正在一步步实现它该有的功能。

最后，我想说的是：学习《RPC手撸专栏》一定要塌下心来，一步一个脚印，动手实践，认真思考，遇到不懂的问题，可以直接到星球发布主题进行提问。一定要记住：纸上得来终觉浅，绝知此事要躬行的道理。否则，一味的CP，或者光看不练，不仅失去了学习的意义，到头来更是一无所获。

**好了，本章就到这里吧，我是冰河，我们下一章见~~**

## 五、关于星球

大家可以加入 **冰河技术** 知识星球，和星球小伙伴们一起学习《SpringCloud Alibaba实战》专栏和《RPC手撸专栏》，冰河技术知识星球的《RPC手撸专栏》是个连载大几十篇的专栏（目前已更新几十大篇章，110+篇文章，110+工程源码，120+源码Tag分支，真正的企业级、分布式、高并发、高性能、高可用，可扩展的RPC框架，仍在持续更新）。

另外，星球中《企业级大规模分布式调度系统》和《企业级大规模分布式IM系统》也已经提升日程，期待你的加入，与星球小伙伴一起开发企业级中间件项目，一起提升硬核技术！

### 星球提供的服务

冰河整理了星球提供的一些服务，如下所示。

加入星球，你将获得：

1.学习从零开始手撸可用于实际场景的高性能RPC框架项目

2.学习SpringCloud Alibaba实战项目—从零开发微服务项目

3.学习高并发、大流量业务场景的解决方案，体验大厂真正的高并发、大流量的业务场景

4.学习进大厂必备技能：性能调优、并发编程、分布式、微服务、框架源码、中间件开发、项目实战

5.提供站点 https://binghe.gitcode.host 所有学习内容的指导、帮助

6.GitHub：https://github.com/binghe001/BingheGuide - 非常有价值的技术资料仓库，包括冰河所有的博客开放案例代码

7.提供技术问题、系统架构、学习成长、晋升答辩等各项内容的回答

8.定期的整理和分享出各类专属星球的技术小册、电子书、编程视频、PDF文件

9.定期组织技术直播分享，传道、授业、解惑，指导阶段瓶颈突破技巧

### 如何加入星球

加入星球：扫描优惠券二维码即可加入星球。

![sa-2022-04-21-007](https://binghe.gitcode.host/assets/images/microservices/springcloudalibaba/sa-2022-04-28-008.png)


* **扫码** ：通过扫描优惠券二维码加入星球。
* **链接** ：打开链接 [http://m6z.cn/6aeFbs](http://m6z.cn/6aeFbs) 加入星球。
* **回复** ：在公众号 **冰河技术** 回复 **星球** 领取优惠券加入星球。

**特别提醒：** 苹果用户进圈或续费，请加微信 **hacker_binghe** 扫二维码，或者去公众号 **冰河技术** 回复 **星球** 扫二维码加入星球。

**好了，今天就到这儿吧，我是冰河，我们下期见~~**

## 写在最后

如果你觉得冰河写的还不错，请微信搜索并关注「 **冰河技术** 」微信公众号，跟冰河学习高并发、分布式、微服务、大数据、互联网和云原生技术，「 **冰河技术** 」微信公众号更新了大量技术专题，每一篇技术文章干货满满！不少读者已经通过阅读「 **冰河技术** 」微信公众号文章，吊打面试官，成功跳槽到大厂；也有不少读者实现了技术上的飞跃，成为公司的技术骨干！如果你也想像他们一样提升自己的能力，实现技术能力的飞跃，进大厂，升职加薪，那就关注「 **冰河技术** 」微信公众号吧，每天更新超硬核技术干货，让你对如何提升技术能力不再迷茫！


## 加群交流

本群的宗旨是给大家提供一个良好的技术学习交流平台，所以杜绝一切广告！由于微信群人满 100 之后无法加入，请扫描下方二维码先添加作者 “冰河” 微信(hacker_binghe)，备注：`学习加群`。



<div align="center">
    <img src="https://binghe.gitcode.host/images/personal/hacker_binghe.jpg?raw=true" width="180px">
    <div style="font-size: 18px;">冰河微信</div>
    <br/>
</div>



## 公众号

分享各种编程语言、开发技术、分布式与微服务架构、分布式数据库、分布式事务、云原生、大数据与云计算技术和渗透技术。另外，还会分享各种面试题和面试技巧。

<div align="center">
    <img src="https://img-blog.csdnimg.cn/20210426115714643.jpg?raw=true" width="180px">
    <div style="font-size: 18px;">公众号：冰河技术</div>
    <br/>
</div>


## 星球

加入星球 **[冰河技术](http://m6z.cn/6aeFbs)**，可以获得本站点所有学习内容的指导与帮助。如果你遇到不能独立解决的问题，也可以添加冰河的微信：**hacker_binghe**， 我们一起沟通交流。另外，在星球中不只能学到实用的硬核技术，还能学习**实战项目**！

关注 [冰河技术](https://img-blog.csdnimg.cn/20210426115714643.jpg?raw=true)公众号，回复 `星球` 可以获取入场优惠券。

<div align="center">
    <img src="https://binghe.gitcode.host/images/personal/xingqiu.png?raw=true" width="180px">
    <div style="font-size: 18px;">知识星球：冰河技术</div>
    <br/>
</div>
