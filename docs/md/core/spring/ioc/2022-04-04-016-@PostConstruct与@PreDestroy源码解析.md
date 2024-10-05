---
layout: post
category: curleyg-spring-ioc
title: 第15章：@PostConstruct与@PreDestroy源码的执行过程
tagline: by CurleyG
tag: [spring.spring-ioc,curleyg-spring-ioc]
excerpt: 在前面的《[【String注解驱动开发】你真的了解@PostConstruct注解和@PreDestroy注解吗？](https://mp.weixin.qq.com/s?__biz=Mzg3MzE1NTIzNA==&mid=2247485015&idx=1&sn=d9b98808a43f72655bf2be51270c4587&chksm=cee5199af992908c45e3801904013f17714b79dc60f6272c699361f7af4681f7ce3548fb8abf&token=1099992343&lang=zh_CN#rd)》一文中，我们简单的介绍了@PostConstruct注解与@PreDestroy注解的用法，有不少小伙伴纷纷留言说：在Spring中，@PostConstruct注解与@PreDestroy注解标注的方法是在哪里调用的呀？相信大家应该都挺好奇的吧，那今天我们就来一起分析下@PostConstruct注解与@PreDestroy注解的执行过程吧！
lock: need
---

# 《Spring注解驱动开发》第15章：@PostConstruct与@PreDestroy源码的执行过程

## 写在前面

> 在前面的《[【String注解驱动开发】你真的了解@PostConstruct注解和@PreDestroy注解吗？](https://mp.weixin.qq.com/s?__biz=Mzg3MzE1NTIzNA==&mid=2247485015&idx=1&sn=d9b98808a43f72655bf2be51270c4587&chksm=cee5199af992908c45e3801904013f17714b79dc60f6272c699361f7af4681f7ce3548fb8abf&token=1099992343&lang=zh_CN#rd)》一文中，我们简单的介绍了@PostConstruct注解与@PreDestroy注解的用法，有不少小伙伴纷纷留言说：在Spring中，@PostConstruct注解与@PreDestroy注解标注的方法是在哪里调用的呀？相信大家应该都挺好奇的吧，那今天我们就来一起分析下@PostConstruct注解与@PreDestroy注解的执行过程吧！
>
> 项目工程源码已经提交到GitHub：[https://github.com/binghe001/spring-annotation](https://github.com/binghe001/spring-annotation)

## 注解说明

@PostConstruct，@PreDestroy是Java规范JSR-250引入的注解，定义了对象的创建和销毁工作，同一期规范中还有注解@Resource，Spring也支持了这些注解。

在Spring中，@PostConstruct，@PreDestroy注解的解析是通过BeanPostProcessor实现的，具体的解析类是org.springframework.context.annotation.CommonAnnotationBeanPostProcessor，其父类是org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor，Spring官方说明了该类对JSR-250中@PostConstruct，@PreDestroy，@Resource注解的支持。

> Spring's org.springframework.context.annotation.CommonAnnotationBeanPostProcessor supports the JSR-250 javax.annotation.PostConstruct and javax.annotation.PreDestroy annotations out of the box, as init annotation and destroy annotation, respectively. Furthermore, it also supports the javax.annotation.Resource annotation for annotation-driven injection of named beans.

## 调用过程

具体过程是，IOC容器先解析各个组件的定义信息，解析到@PostConstruct，@PreDestroy的时候，定义为生命周期相关的方法，组装组件的定义信息等待初始化；在创建组件时，创建组件并且属性赋值完成之后，在执行各类初始化方法之前，从容器中找出所有BeanPostProcessor的实现类，其中包括InitDestroyAnnotationBeanPostProcessor，执行所有BeanPostProcessor的postProcessBeforeInitialization方法，在InitDestroyAnnotationBeanPostProcessor中就是找出被@PostConstruct修饰的方法的定义信息，并执行被@PostConstruct标记的方法。

## 调用分析

**@PostConstruct的调用链如下：**

![](https://binghe.gitcode.host/assets/images/core/spring/ioc/2022-04-04-005.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(String, Object, RootBeanDefinition)初始化流程中，先执行org.springframework.beans.factory.config.BeanPostProcessor.postProcessBeforeInitialization(Object, String)方法，然后再执行初始化方法：

```java
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
	if (System.getSecurityManager() != null) {
		AccessController.doPrivileged(new PrivilegedAction<Object>() {
			@Override
			public Object run() {
				invokeAwareMethods(beanName, bean);
				return null;
			}
		}, getAccessControlContext());
	}
	else {
		invokeAwareMethods(beanName, bean);
	}
 
	Object wrappedBean = bean;
	if (mbd == null || !mbd.isSynthetic()) {
		// 在执行初始化方法之前：先执行org.springframework.beans.factory.config.BeanPostProcessor.postProcessBeforeInitialization(Object, String)方法
		wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
	}
 
	try {
		//执行InitializingBean的初始化方法和init-method指定的初始化方法
		invokeInitMethods(beanName, wrappedBean, mbd);
	}
	catch (Throwable ex) {
		throw new BeanCreationException(
				(mbd != null ? mbd.getResourceDescription() : null),
				beanName, "Invocation of init method failed", ex);
	}
 
	if (mbd == null || !mbd.isSynthetic()) {
		wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
	}
	return wrappedBean;
}
```

org.springframework.beans.factory.config.BeanPostProcessor.postProcessBeforeInitialization(Object, String)的说明如下：

> Apply this BeanPostProcessor to the given new bean instance before any bean initialization callbacks (like InitializingBean's afterPropertiesSet or a custom init-method). The bean will already be populated with property values. The returned bean instance may be a wrapper around the original.

调用时机： 在组件创建完属性复制完成之后，调用组件初始化方法之前；

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInitialization(Object, String)的具体流程如下。

```java
@Override
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
		throws BeansException {
	
	Object result = existingBean;
	for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
		//遍历所有BeanPostProcessor的实现类，执行BeanPostProcessor的postProcessBeforeInitialization
		//在InitDestroyAnnotationBeanPostProcessor中的实现是找出@PostConstruct标记的方法的定义信息，并执行
		result = beanProcessor.postProcessBeforeInitialization(result, beanName);
		if (result == null) {
			return result;
		}
	}
	return result;
}
```

**@PreDestroy调用链如下：**

![](https://binghe.gitcode.host/assets/images/core/spring/ioc/2022-04-04-005.png)

@PreDestroy是通过org.springframework.beans.factory.config.DestructionAwareBeanPostProcessor.postProcessBeforeDestruction(Object, String)被调用（InitDestroyAnnotationBeanPostProcessor实现了该接口），该方法的说明如下：

> Apply this BeanPostProcessor to the given bean instance before its destruction. Can invoke custom destruction callbacks.
>
> Like DisposableBean's destroy and a custom destroy method, this callback just applies to singleton beans in the factory (including inner beans).

**调用时机： 该方法在组件的销毁之前调用；**

org.springframework.beans.factory.support.DisposableBeanAdapter.destroy()的执行流程如下：

```java
@Override
public void destroy() {
	if (!CollectionUtils.isEmpty(this.beanPostProcessors)) {
		//调用所有DestructionAwareBeanPostProcessor的postProcessBeforeDestruction方法
		for (DestructionAwareBeanPostProcessor processor : this.beanPostProcessors) {
			processor.postProcessBeforeDestruction(this.bean, this.beanName);
		}
	}
 
	if (this.invokeDisposableBean) {
		if (logger.isDebugEnabled()) {
			logger.debug("Invoking destroy() on bean with name '" + this.beanName + "'");
		}
		try {
			if (System.getSecurityManager() != null) {
				AccessController.doPrivileged(new PrivilegedExceptionAction<Object>() {
					@Override
					public Object run() throws Exception {
						((DisposableBean) bean).destroy();
						return null;
				}
				}, acc);
			}
			else {
				//调用DisposableBean的销毁方法
				((DisposableBean) bean).destroy();
			}
		}
		catch (Throwable ex) {
				String msg = "Invocation of destroy method failed on bean with name '" + this.beanName + "'";
			if (logger.isDebugEnabled()) {
				logger.warn(msg, ex);
			}
			else {
				logger.warn(msg + ": " + ex);
			}
		}
	}
 
	//调用自定义的销毁方法
	if (this.destroyMethod != null) {
		invokeCustomDestroyMethod(this.destroyMethod);
	}
	else if (this.destroyMethodName != null) {
		Method methodToCall = determineDestroyMethod();
		if (methodToCall != null) {
			invokeCustomDestroyMethod(methodToCall);
		}
	}
}
```

所以是先调用DestructionAwareBeanPostProcessor的postProcessBeforeDestruction(@PreDestroy标记的方法被调用)，再是DisposableBean的destory方法，最后是自定义销毁方法。

<font color="#FF0000">**好了，咱们今天就聊到这儿吧！别忘了给个在看和转发，让更多的人看到，一起学习一起进步！！**</font>

> 项目工程源码已经提交到GitHub：[https://github.com/binghe001/spring-annotation](https://github.com/binghe001/spring-annotation)

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






