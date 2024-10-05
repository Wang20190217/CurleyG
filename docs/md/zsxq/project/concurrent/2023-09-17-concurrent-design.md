---
title: 实战高并发设计模式
---

# 《并发设计模式》开篇-全新的开始：用讲故事的形式带你彻底吃透并发设计模式

作者：冰河
<br/>星球：[http://m6z.cn/6aeFbs](http://m6z.cn/6aeFbs)
<br/>博客：[https://binghe.gitcode.host](https://binghe.gitcode.host)
<br/>文章汇总：[https://binghe.gitcode.host/md/all/all.html](https://binghe.gitcode.host/md/all/all.html)
<br/>源码获取地址：[https://t.zsxq.com/0dhvFs5oR](https://t.zsxq.com/0dhvFs5oR)

> 沉淀，成长，突破，帮助他人，成就自我。

* 本章难度：★★☆☆☆
* 本章重点：对全新的《并发设计模式》专栏进行总体介绍，整体以故事线的形式来贯穿整个专栏，介绍专栏故事场景下涉及到的人物关系，以及主人翁在整个专栏学习过程中，一路打怪升级的过程。

<iframe src="//player.bilibili.com/player.html?aid=278789619&bvid=BV1xw411P7Z9&cid=1342542828&p=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width="100%" height="480"> </iframe>

**大家好，我是冰河~~**

小菜本是一名211、985毕业的研究生，明明可以靠脸吃饭，却偏偏要入行程序员。虽说没啥工作经验吧，但毕竟在学校里也算是个风云人物，什么学生会主席啦、宿管会会长啦，反正是学校的社团都有他的身影，也会积极参加校外活动，一来二去，还真成为了学校的风云人物，也正是借着自己在学校的传奇经历，小菜面试上了一家互联网大厂实习，从此开启了在并发编程领域一路打怪升级的过程。

## 一、专栏介绍

没错，《并发设计模式》是一个全新的专栏，写什么的呢？看专栏名字就知道了，写的是与并发相关的设计模式，一想到并发，小伙伴们脑海里蹦出的关键词可能是：枯燥，难学，学不会，记不住，学过就忘，明明记得看到过，可就是想不起来在哪看到过等等。

<div align="center">
    <img src="https://binghe.gitcode.host/assets/images/core/concurrent/2023-09-17-001.png?raw=true" width="30%">
    <br/>
</div>

所以，为了让小伙伴们更轻松、高效的学习并发编程，冰河推出了这个《并发设计模式》专栏，整个专栏会以事件驱动、以故事线的形式来贯穿始末，让大家以看故事的形式轻松学习《并发设计模式》，并且在整个专栏过程中，会穿插不少实际业务项目场景，也会带着大家手写核心源码，让大家更好的从项目实战角度学习并发设计模式。所以说，对于并发编程来说，你可以轻松、高效的学会，吃透并发设计模式的核心知识。

## 二、人物介绍

既然是讲故事嘛，那肯定是要有故事人物啦。正所谓以人为本嘛，咱们先来定几个专栏场景中的人物。

* 小菜：小菜可不得了，整个专栏实际上都是围绕着小菜这个核心人物，在并发编程领域一路打怪升级的经历来写的。小菜本是一名211、985毕业的研究生，明明可以靠脸吃饭，却偏偏要入行程序员。虽说没啥工作经验吧，但毕竟在学校里也算是个风云人物，什么学生会主席啦、宿管会会长啦，反正是学校的社团都有他的身影，也会积极参加校外活动，一来二去，还真成为了学校的风云人物。也正是借着自己在学校的传奇经历，小菜面试上了一家互联网大厂实习，从此开启了在并发编程领域一路打怪升级的过程。
* 老王：小菜在大厂的直属领导，是一位有才华，技术能力极强，从发型看上去就很资深的资深技术专家，在并发编程领域，就没有老王搞不定的问题。虽说老王在公司的Title很高，但是对新人和下属还是比较和蔼的，尤其是下属遇到自己不能解决的问题时，老王都会耐心的为其讲解，必要时，还会自己手搓代码为其演示（这老王人咋这么好，我在工作中咋特么就遇不到呢？）。所以，在工作中，小菜遇到自己解决不了的问题时，会经常问老王。
* 产品经理：提起产品经理，可能大部分小伙伴心里都是比较厌恶的，没错，小菜也非常“痛恨”他，具体为何，在后续故事情节会有。
* 大Boos：基本不怎么出场，一出场就是王炸。

总体任务关系也并不是很复杂，大家看图：

<div align="center">
    <img src="https://binghe.gitcode.host/assets/images/core/concurrent/2023-09-17-002.png?raw=true" width="50%">
    <br/>
</div>


大Boos虽说在整个故事线中出现的不对，但他却是能够决定所有人升职加薪和去留问题。产品经理嘛，每次会议新需求的形式出现，并且大部分会在小菜刚要下班时“及时”出现，久而久之，会造成小菜心理上的厌烦。老王，就是前文说的从发型就能看出来非常资深的资深技术专家，为人正直，对待下属和新人比较和蔼，有耐心。小菜，就是前文说的刚毕业的大学生，明明可以靠脸吃饭，就非要入行程序员，比较”痛恨“产品经理。

## 三、专栏安排

这次，咱们换种方式，以小菜在并发编程领域一路打怪升级的过程为线索，以场景故事的形式贯穿始末，专栏会穿插大量的图解，让大家从轻松、愉快的氛围中学习并发设计模式，主要涵盖的内容如图所示。

<div align="center">
    <img src="https://binghe.gitcode.host/assets/images/core/concurrent/2023-09-17-003.png?raw=true" width="70%">
    <br/>
</div>

可以看到，整个专栏会涵盖12种最常用，也是最核心的并发设计模式， 每一种并发设计模式都会深入真实场景案例去讲解，让大家在学习并发设计模式的过程中，知其然，知其所以然，更要知其如何在实际项目场景中落地。

## 四、需求驱动

## 查看全文

专栏中的每一种设计模式都是以事件驱动的形式去讲解，这里的事件，我们可以理解成需求，正所谓无需求，不设计，无设计不编码。以场景故事的形式设计需求，从需求角度设计代码，再到最终实现，整个过程都会记录小菜在整个故事情节中的经验积累和心态变化，估计这也是大部分程序员在职场的心路历程。

每一种设计模式都会配套合适的真实场景案例，加上“老王”耐心的讲解，小菜下班后自己不断总结和思考，随后，将其使用到真实场景中。整个过程，实际上就是需求在驱动。

在整个场景过程中，我们舍弃了整个研发过程中比较繁琐的流程，将其简化成最核心的流程，如图所示。

<div align="center">
    <img src="https://binghe.gitcode.host/assets/images/core/concurrent/2023-09-17-004.png?raw=true" width="70%">
    <br/>
</div>

（1）产品经理设计需求原型，将设计出来的需求原型，交给公司的老王进行评估。

（2）老王拿到需求原型后，一顿梳理，梳理来梳理去，可行，就将其交给小菜进行开发。当然，老王是很负责任的，交给小菜的时候，就会给小菜讲清楚需求和实现方式。

（3）小菜作为刚毕业的职场新人，拿到新需求后自然显得比较生疏和懵逼，不用慌，有老王在，怕啥？小菜在梳理需求和实现方式的时候，会不断跟老王进行沟通，讨教具体的实现方式。在这个过程中，老王更是为其耐心的讲解（老王真特么是个大好人啊！），小菜也是边听边记。

（4）小菜将需求和实现方式了解清楚后，就会尝试去动手开发了，当然实际过程中可能还会遇到问题，此时老王还是会耐心讲解的（老王是真特么好啊），经过不懈的努力，小菜终于把一个个需求实现了，最终交付验收，老板也比较满意。

## 五、打怪升级

小菜一步步梳理需求，使用并发设计模式实现一个个需求的过程，其实就是一个打怪升级的过程。当然，在这个过程中，会不断有老王的辅助和讲解，在整个过程中，小王也比较享受攻克难题，实现需求的自豪感，就这么不断进行着良性循环。久而久之，小菜的技术能力以及并发编程能力，都会得到很大的提升，能独当一面，使用并发设计模式独立负责完整的需求开发。

<div align="center">
    <img src="https://binghe.gitcode.host/assets/images/core/concurrent/2023-09-17-005.png?raw=true" width="70%">
    <br/>
</div>

最终，小菜被大Boos赏识，顺利转正实现加薪。

## 六、总结

本章开始我们就进入了一个全新的专栏了，这个专栏是除项目实战专栏外的一个技术专栏——《并发设计模式》，本章主要是从总体上介绍了《并发设计模式》专栏的概要内容，整体会以小菜在并发编程领域一路打怪升级的过程为线索，以场景故事贯穿始末，以需求驱动的方式，在老王的帮助下，基于并发设计模式来完成各种不同的需求。在整个过程中，小菜的技术能力和并发编程能力得到了极大的提升，最终被老板认可，完美实现转正和加薪。

最后，可以在评论区写下你学完本章节的收获，祝大家都能学有所成，我们一起搞定高并发设计模式。

**好了，让我们一起开启一个新专栏之旅吧，愿大家都能有所收获，我是冰河，我们下期见~~**

## 星球服务

加入星球，你将获得：

1.项目学习：微服务入门必备的SpringCloud  Alibaba实战项目、手写RPC项目—所有大厂都需要的项目【含上百个经典面试题】、深度解析Spring6核心技术—只要学习Java就必须深度掌握的框架【含数十个经典思考题】、Seckill秒杀系统项目—进大厂必备高并发、高性能和高可用技能。

2.框架源码：手写RPC项目—所有大厂都需要的项目【含上百个经典面试题】、深度解析Spring6核心技术—只要学习Java就必须深度掌握的框架【含数十个经典思考题】。

3.硬核技术：深入理解高并发系列（全册）、深入理解JVM系列（全册）、深入浅出Java设计模式（全册）、MySQL核心知识（全册）。

4.技术小册：深入理解高并发编程（第1版）、深入理解高并发编程（第2版）、从零开始手写RPC框架、SpringCloud  Alibaba实战、冰河的渗透实战笔记、MySQL核心知识手册、Spring IOC核心技术、Nginx核心技术、面经手册等。

5.技术与就业指导：提供相关就业辅导和未来发展指引，冰河从初级程序员不断沉淀，成长，突破，一路成长为互联网资深技术专家，相信我的经历和经验对你有所帮助。

冰河的知识星球是一个简单、干净、纯粹交流技术的星球，不吹水，目前加入享5折优惠，价值远超门票。加入星球的用户，记得添加冰河微信：hacker_binghe，冰河拉你进星球专属VIP交流群。

## 星球重磅福利

跟冰河一起从根本上提升自己的技术能力，架构思维和设计思路，以及突破自身职场瓶颈，冰河特推出重大优惠活动，扫码领券进行星球，**直接立减149元，相当于5折，** 这已经是星球最大优惠力度！

<div align="center">
    <img src="https://binghe.gitcode.host/images/personal/xingqiu_149.png?raw=true" width="80%">
    <br/>
</div>

领券加入星球，跟冰河一起学习《SpringCloud Alibaba实战》、《手撸RPC专栏》和《Spring6核心技术》，更有已经上新的《大规模分布式Seckill秒杀系统》，从零开始介绍原理、设计架构、手撸代码。后续更有硬核中间件项目和业务项目，而这些都是你升职加薪必备的基础技能。

**100多元就能学这么多硬核技术、中间件项目和大厂秒杀系统，如果是我，我会买他个终身会员！**

## 其他方式加入星球

* **链接** ：打开链接 [http://m6z.cn/6aeFbs](http://m6z.cn/6aeFbs) 加入星球。
* **回复** ：在公众号 **冰河技术** 回复 **星球** 领取优惠券加入星球。

**特别提醒：** 苹果用户进圈或续费，请加微信 **hacker_binghe** 扫二维码，或者去公众号 **冰河技术** 回复 **星球** 扫二维码加入星球。

## 星球规划

后续冰河还会在星球更新大规模中间件项目和深度剖析核心技术的专栏，目前已经规划的专栏如下所示。

### 中间件项目

* 《大规模分布式定时调度中间件项目实战（非Demo）》：全程手撸代码。
* 《大规模分布式IM（即时通讯）项目实战（非Demo）》：全程手撸代码。
* 《大规模分布式网关项目实战（非Demo）》：全程手撸代码。
* 《手写Redis》：全程手撸代码。
* 《手写JVM》全程手撸代码。

### 超硬核项目

* 《从零落地秒杀系统项目》：全程手撸代码，在阿里云实现压测（**已上新**）。
* 《大规模电商系统商品详情页项目》：全程手撸代码，在阿里云实现压测。
* 其他待规划的实战项目，小伙伴们也可以提一些自己想学的，想一起手撸的实战项目。。。


既然星球规划了这么多内容，那么肯定就会有小伙伴们提出疑问：这么多内容，能更新完吗？我的回答就是：一个个攻破呗，咱这星球干就干真实中间件项目，剖析硬核技术和项目，不做Demo。初衷就是能够让小伙伴们学到真正的核心技术，不再只是简单的做CRUD开发。所以，每个专栏都会是硬核内容，像《SpringCloud Alibaba实战》、《手撸RPC专栏》和《Spring6核心技术》就是很好的示例。后续的专栏只会比这些更加硬核，杜绝Demo开发。

小伙伴们跟着冰河认真学习，多动手，多思考，多分析，多总结，有问题及时在星球提问，相信在技术层面，都会有所提高。将学到的知识和技术及时运用到实际的工作当中，学以致用。星球中不少小伙伴都成为了公司的核心技术骨干，实现了升职加薪的目标。

## 联系冰河

### 加群交流

本群的宗旨是给大家提供一个良好的技术学习交流平台，所以杜绝一切广告！由于微信群人满 100 之后无法加入，请扫描下方二维码先添加作者 “冰河” 微信(hacker_binghe)，备注：`星球编号`。



<div align="center">
    <img src="https://binghe.gitcode.host/images/personal/hacker_binghe.jpg?raw=true" width="180px">
    <div style="font-size: 18px;">冰河微信</div>
    <br/>
</div>



### 公众号

分享各种编程语言、开发技术、分布式与微服务架构、分布式数据库、分布式事务、云原生、大数据与云计算技术和渗透技术。另外，还会分享各种面试题和面试技巧。内容在 **冰河技术** 微信公众号首发，强烈建议大家关注。

<div align="center">
    <img src="https://binghe.gitcode.host/images/personal/ice_wechat.jpg?raw=true" width="180px">
    <div style="font-size: 18px;">公众号：冰河技术</div>
    <br/>
</div>


### 视频号

定期分享各种编程语言、开发技术、分布式与微服务架构、分布式数据库、分布式事务、云原生、大数据与云计算技术和渗透技术。另外，还会分享各种面试题和面试技巧。

<div align="center">
    <img src="https://binghe.gitcode.host/images/personal/ice_video.png?raw=true" width="180px">
    <div style="font-size: 18px;">视频号：冰河技术</div>
    <br/>
</div>



### 星球

加入星球 **[冰河技术](http://m6z.cn/6aeFbs)**，可以获得本站点所有学习内容的指导与帮助。如果你遇到不能独立解决的问题，也可以添加冰河的微信：**hacker_binghe**， 我们一起沟通交流。另外，在星球中不只能学到实用的硬核技术，还能学习**实战项目**！

关注 [冰河技术](https://img-blog.csdnimg.cn/20210426115714643.jpg?raw=true)公众号，回复 `星球` 可以获取入场优惠券。

<div align="center">
    <img src="https://binghe.gitcode.host/images/personal/xingqiu.png?raw=true" width="180px">
    <div style="font-size: 18px;">知识星球：冰河技术</div>
    <br/>
</div>