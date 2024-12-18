---
title: 第80章-X：整合SpringCloud Alibaba阶段作业
---

# 《RPC手撸专栏》第80章-X：整合SpringCloud Alibaba阶段作业

作者：冰河
<br/>星球：[http://m6z.cn/6aeFbs](http://m6z.cn/6aeFbs)
<br/>博客：[https://binghe.gitcode.host](https://binghe.gitcode.host)
<br/>文章汇总：[https://binghe.gitcode.host/md/all/all.html](https://binghe.gitcode.host/md/all/all.html)

> 沉淀，成长，突破，帮助他人，成就自我。

**大家好，我是CurleyG~~**

《RPC手撸专栏》到目前为止，源码已经有60+个子工程，完成了：自定义注解、自定义包扫描类、自定义协议、请求与响应协议的封装、服务提供者、服务消费者、注册中心、负载均衡与增强型负载均衡、序列化与反序列化、动态代理、反射机制、心跳机制、重试机制、整合Spring、整合SpringBoot、整合Docker和整合SpringCloud Alibaba，给大家留个整合SpringCloud Alibaba阶段的作业。

## 作业内容

纸上得来终觉浅，绝知此事要躬行，学有所思，学有所用，特布置作业如下：

* 完成《SpringCloud Alibaba实战》专栏的作业。
* 根据《SpringCloud Alibaba实战》专栏的学习内容，自行搭建SpringCloud Alibaba微服务项目，可参考《SpringCloud Alibaba实战》专栏的源码。
* 搭建好SpringCloud Alibaba微服务项目并测试通过后，自行独立将手写的bhrpc框架整合到微服务项目中，可参考《RPC手撸专栏》第80章的内容。
* 对于《SpringCloud Alibaba实战》和《RPC手撸专栏》做好总结。

## 作业要求

为了保证作业的质量，现对《RPC手撸专栏》阶段性作业提出如下要求：

* 认真完成作业，对于不太了解的问题，认真查询资料后自己总结。
* 星球的小伙伴最好都完成作业，为自己而学。
* 作业提交到星球布置的当前作业的评论区即可，我会对作业进行一一点评，对完成作业的小伙伴单独指导项目优化点。
* 作业提交时间：2023年春节前后。

## 写在最后

最后，我想说的是：学习《RPC手撸专栏》一定要塌下心来，一步一个脚印，动手实践，认真思考，遇到不懂的问题，可以直接到星球发布主题进行提问。一定要记住：纸上得来终觉浅，绝知此事要躬行的道理。否则，一味的CP，或者光看不练，到头来不仅失去了学习的意义，到头来更是一无所获。

小伙伴们记住：学习是为自己学习，大家加油吧！

## 关于星球

**冰河技术** 知识星球《SpringCloud Alibaba实战》从零搭建并开发微服务项目已完结，《RPC手撸专栏》已经更新80+篇文章，并将源码的获取方式放到了知识星球中，同时在微信上创建了专门的知识星球群，冰河会在知识星球上和星球群里解答球友的提问。

### 星球提供的服务

冰河整理了星球提供的一些服务，如下所示。

加入星球，你将获得： 

1.学习从零开始手撸可用于实际场景的高性能、可扩展的RPC框架项目

2.学习SpringCloud Alibaba实战项目—从零开发微服务项目 

3.学习高并发、大流量业务场景的解决方案，体验大厂真正的高并发、大流量的业务场景 

4.学习进大厂必备技能：性能调优、并发编程、分布式、微服务、框架源码、中间件开发、项目实战 

5.提供站点 https://binghe.gitcode.host 所有学习内容的指导、帮助 

6.GitHub：https://github.com/binghe001/BingheGuide - 非常有价值的技术资料仓库，包括冰河所有的博客开放案例代码 

7.提供技术问题、系统架构、学习成长、晋升答辩等各项内容的回答 

8.定期的整理和分享出各类专属星球的技术小册、电子书、编程视频、PDF文件 

9.定期组织技术直播分享，传道、授业、解惑，指导阶段瓶颈突破技巧

### 如何加入星球

* **链接** ：打开链接 [http://m6z.cn/6aeFbs](http://m6z.cn/6aeFbs) 加入星球。
* **回复** ：在公众号 **冰河技术** 回复 **星球** 领取优惠券加入星球。

**特别提醒：** 苹果用户进圈或续费，请加微信 **hacker_binghe** 扫二维码，或者去公众号 **冰河技术** 回复 **星球** 扫二维码加入星球。

**好了，今天就到这儿吧，我是冰河，我们下期见~~**

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
