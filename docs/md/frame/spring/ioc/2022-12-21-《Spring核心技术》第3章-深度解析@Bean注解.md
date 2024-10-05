---
layout: post
category: binghe-code-spring
title: 第03章：深度解析@Bean注解
tagline: by CurleyG
tag: [spring,ioc,aop,transaction,springmvc]
excerpt: 第03章：深度解析@Bean注解
lock: need
---

# 《Spring核心技术》第03章-驱动型注解：深度解析@Bean注解

作者：冰河
<br/>星球：[http://m6z.cn/6aeFbs](http://m6z.cn/6aeFbs)
<br/>博客：[https://binghe.gitcode.host](https://binghe.gitcode.host)
<br/>文章汇总：[https://binghe.gitcode.host/md/all/all.html](https://binghe.gitcode.host/md/all/all.html)
<br/>源码地址：[https://github.com/binghe001/spring-annotation-book/tree/master/spring-annotation-chapter-03](https://github.com/binghe001/spring-annotation-book/tree/master/spring-annotation-chapter-03)

> 沉淀，成长，突破，帮助他人，成就自我。

**大家好，我是冰河~~**

------

* **本章难度**：★★★★☆

* **本章重点**：进一步了解@Bean注解的使用方法和如何避免踩坑，并在源码级别彻底理解和吃透@Bean注解的执行流程。

------

本章目录如下所示：

* 学习指引
* 注解说明
  * 注解源码
  * 使用场景
* 使用案例
  * 案例描述
  * 案例实现
  * 案例测试
* 源码时序图
  * 注册Bean的流程
  * 调用初始化方法
  * 调用销毁方法
* 源码解析
  * 注册Bean的流程
  * 调用初始化方法
  * 调用销毁方法
* 总结
* 思考
* VIP服务

## 一、学习指引

`@Bean注解的实现其实没你想象的那么简单！`

翻看Spring的源码时，发现@Bean注解的源码上标注了`Since: 3.0`，也就是说，@Bean注解是Spring从3.0版本开始提供的源码。@Bean注解可以标注在方法上，将当前方法的返回值注入到IOC容器中，也可以标注到注解上，作为元注解使用。

还是那句话：如果只想做CRUD程序员，对于@Bean注解了解到这里就已经可以了，如果想进一步突破自己，让自己的技术能力更上一层楼，则继续往下看。

## 二、注解说明

`关于@Bean注解的一点点说明~~`

@Bean注解可以说是使用Spring开发应用程序时，使用的比较多的一个注解，尤其是基于SpringBoot开发应用程序时，@Bean注解使用的就很多了。@Bean注解可以标注到方法上，将当前方法的返回值注入到IOC容器中。@Bean注解也可以标注到注解上，作为元注解使用。

### 2.1 注解源码

本节，主要对@Bean注解的源码进行简单的剖析。

@Bean注解的源码详见：org.springframework.context.annotation.Bean，如下所示。

```java
/**
 * @author Rod Johnson
 * @author Costin Leau
 * @author Chris Beams
 * @author Juergen Hoeller
 * @author Sam Brannen
 * @since 3.0
 */
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {
	//Since: 4.3.3
	@AliasFor("name")
	String[] value() default {};

	@AliasFor("value")
	String[] name() default {};
	
    //Since: 5.1
	boolean autowireCandidate() default true;

	String initMethod() default "";

	String destroyMethod() default AbstractBeanDefinition.INFER_METHOD;
}
```

从@Bean源码的注释可以看出，@Bean注解是Spring从3.0版本开始提供的注解，注解中各个属性的含义如下所示。

* name：String[]数组类型，指定注入到IOC容器中的Bean的名称，可以指定多个名称。如果不指定name属性和value属性的值，则注入到IOC容器中的Bean的名称默认是方法的名称。
* value：String[]数组类型，从Spring 4.3.3版本开始提供的@Bean的属性，作用与name属性相同。
* autowireCandidate：boolean类型，表示是否支持自动按照类型注入到其他的Bean中。此属性会影响@Autowired注解，不会响应@Resource注解，默认为true，表示支持自动按照类型注入到其他的Bean中。
* initMethod：指定初始化的方法。
* destroyMethod：指定销毁的方法。

### 2.2 使用场景

在使用Spring的注解开发应用程序时，如果是我们自己开发的类，可以在类上标注@Component注解（也可以是@Repository、@Service、@Controller等注解），将类注入到IOC容器中。但是，有时很多类不是我们自己写的，而是依赖的第三方的类库，此时就无法在类上标注@Component等注解了，此时就需要使用@Bean注解将其注入到IOC容器中。

也就是说，@Bean注解适用于将第三方提供的类注入到IOC容器中的场景。使用@Bean注解创建的Bean对象默认是单实例Bean对象。

## 三、使用案例

`整个案例来玩玩儿吧！`

### 3.1 案例描述

使用@Bean注解将User类的对象注入到IOC容器中，打印初始化和销毁方法，并验证使用@Bean注解创建的Bean对象默认是单实例Bean。

### 3.2 案例实现

使用@Bean注解可以将类对象注入到IOC容器中，并且@Bean注解的initMethod属性可以指定初始化的方法，destroyMethod属性可以指定销毁的方法。当IOC容器对Bean对象进行初始化时，会执行@Bean注解的initMethod属性指定的方法，当IOC容器在关闭时，会执行@Bean注解的destroyMethod属性指定的方法，触发销毁方法的逻辑。

**注意：上述逻辑仅限于注入到IOC容器中的单例Bean对象。如果是单实例Bean，则IOC容器启动时，就会创建Bean对象，IOC容器关闭时，销毁Bean对象。如果是多实例Bean，IOC容器在启动时，不会创建Bean对象，在每次从IOC容器中获取Bean对象时，都会创建新的Bean对象返回，IOC容器关闭时，也不会销毁对象。也就是说，如果是多实例Bean，IOC容器不会管理Bean对象。**

本节，就以单实例Bean为例来实现案例程序，具体的实现步骤如下所示。

（1）新建注入到IOC容器中的User类

源码详见：spring-annotation-chapter-03工程下的io.binghe.spring.annotation.chapter03.bean.User，如下所示。

```java
public class User {

    private final Logger logger = LoggerFactory.getLogger(User.class);

    public User(){
        logger.info("执行构造方法...");
    }

    public void init(){
        logger.info("执行初始化方法...");
    }

    public void destroy(){
        logger.info("执行销毁方法...");
    }
}
```

IOC容器启动时，会创建User类的对象并调用init()方法进行初始化。IOC容器关闭时，会销毁User类的对象，并调用destroy()方法执行销毁逻辑。

可以看到，User类的实现比较简单，提供了一个无参构造方法，一个表示初始化的init()方法和一个表示销毁的destroy()方法，每个方法中都打印了相应的日志来说明调用了对应的方法。

（2）创建Spring的配置类BeanConfig

源码详见：spring-annotation-chapter-03工程下的io.binghe.spring.annotation.chapter03.config.BeanConfig，如下所示。

```java
@Configuration
@ComponentScan(basePackages = "io.binghe.spring.annotation.chapter03")
public class BeanConfig {
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public User user(){
        return new User();
    }
}
```

可以看到，在BeanConfig类上标注了@Configuration注解，说明BeanConfig类是一个Spring的配置类，并且在BeanConfig类上标注了@ComponentScan注解，指定要扫描的包为`io.binghe.spring.annotation.chapter03`。

在BeanConfig类中定义了一个user()方法，返回一个User对象。在user()方法上标注了@Bean注解，并通过initMethod属性指定的初始化方法为User类的init()方法，通过destroyMethod属性指定的销毁方法为User类的destroy()方法。

（3）创建案例测试类BeanTest

源码详见：spring-annotation-chapter-03工程下的io.binghe.spring.annotation.chapter03.BeanTest，如下所示。

```java
public class BeanTest {
    private static final Logger LOGGER = LoggerFactory.getLogger(BeanTest.class);
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(BeanConfig.class);
        LOGGER.info("IOC容器启动完成....");
        User user1 = context.getBean(User.class);
        User user2 = context.getBean(User.class);
        LOGGER.info("user1是否等于user2===>>>{}", (user1 == user2));
        context.close();
    }
}
```

可以看到，在BeanTest类中，通过BeanConfig配置类创建了IOC容器，从IOC容器中两次获取User对象，分别赋值给user1和user2，打印user1是否等于user2的日志，并关闭IOC容器。

### 3.3 案例测试

运行BeanTest类的main()方法，输出的日志信息如下所示。

```bash
- 执行构造方法...
- 执行初始化方法...
- IOC容器启动完成....
- user1是否等于user2===>>>true
- 执行销毁方法...
```

可以看到，IOC容器在启动时，就创建了User对象，并调用了@Bean注解的initMethod方法指定的初始化方法，IOC容器在关闭时，调用@Bean注解的destroyMethod属性指定的销毁方法。并且通过打印的user1是否等于user2的日志为true，可以说明使用@Bean注解默认创建的Bean对象是单实例Bean，每个类在IOC容器中只会存在一个单实例Bean对象。

## 四、源码时序图

`结合时序图理解源码会事半功倍，你觉得呢？`

本节，就以源码时序图的方式，直观的感受下@Bean注解在Spring源码层面的执行流程。本节，主要从注册Bean的流程、调用初始化方法和调用销毁方法三个方面解析@Bean的源码时序图。

### 4.1 注册Bean的流程

@Bean注解在Spring源码层面注册Bean的执行流程如图3-1~图3-2所示。

![图3-1](https://binghe.gitcode.host/assets/images/frame/spring/ioc/spring-core-2022-12-21-001.png)

![图3-2](https://binghe.gitcode.host/assets/images/frame/spring/ioc/spring-core-2022-12-21-002.png)

由图3-1和图3-2可以看出，@Bean注解在Spring源码层面的执行流程会涉及到BeanTest类、AnnotationConfigApplicationContext类、AbstractApplicationContext类、PostProcessorRegistrationDelegate类、ConfigurationClassPostProcessor类、ConfigurationClassParser类、ConfigurationClassBeanDefinitionReader类和DefaultListableBeanFactory类。具体的源码执行细节参见源码解析部分。

### 4.2 调用初始化方法

@Bean注解在Spring源码层面调用初始化方法的执行流程如图3-3~3-4所示。

![图3-3](https://binghe.gitcode.host/assets/images/frame/spring/ioc/spring-core-2022-12-21-003.png)



![图3-4](https://binghe.gitcode.host/assets/images/frame/spring/ioc/spring-core-2022-12-21-004.png)

由图3-3~3-4可以看出，@Bean注解在Spring源码层面调用初始化方法会涉及到BeanTest类、AnnotationConfigApplicationContext类、AbstractApplicationContext类、DefaultListableBeanFactory类、AbstractBeanFactory类、AbstractAutowireCapableBeanFactory类和User类，具体的源码执行细节参见源码解析部分。

### 4.3 调用销毁方法

@Bean注解在Spring源码层面调用销毁方法的执行流程如图3-5所示。

![图3-5](https://binghe.gitcode.host/assets/images/frame/spring/ioc/spring-core-2022-12-21-005.png)

由图3-5可以看出，@Bean注解在Spring源码层面调用销毁方法会涉及到BeanTest类、AbstractApplicationContext类、DefaultListableBeanFactory类、DefaultSingletonBeanRegistry类、DisposableBeanAdapter类和User类，具体的源码执行细节参见源码解析部分。

## 五、源码解析

`源码时序图整清楚了，那就整源码解析呗！`

@Bean注解在Spring源码层面的执行流程，结合源码执行的时序图，会理解的更加深刻。本节，同样会从注册Bean的流程、调用初始化方法和调用销毁方法三个方面解析@Bean的源码执行流程。

### 5.1 注册Bean的流程

本节，就简单介绍下@Bean注解在Spring源码层面注册Bean的源码执行流程，结合源码执行的时序图，会理解的更加深刻，可以结合图3-1~3-2进行理解，具体解析步骤如下所示。

（1）运行案例程序启动类

案例程序启动类源码详见：spring-annotation-chapter-03工程下的io.binghe.spring.annotation.chapter03.BeanTest，运行BeanTest类的main()方法。

在BeanTest类的main()方法中调用了AnnotationConfigApplicationContext类的构造方法，并传入了ComponentScanConfig类的Class对象来创建IOC容器。接下来，会进入AnnotationConfigApplicationContext类的构造方法。

（2）解析AnnotationConfigApplicationContext类的AnnotationConfigApplicationContext(Class<?>... componentClasses)构造方法

源码详见：org.springframework.context.annotation.AnnotationConfigApplicationContext#AnnotationConfigApplicationContext(Class<?>... componentClasses)。

```java
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
    this();
    register(componentClasses);
    refresh();
}
```

可以看到，在上述构造方法中，调用了refresh()方法来刷新IOC容器。

（3）解析AbstractApplicationContext类的refresh()方法

源码详见：org.springframework.context.support.AbstractApplicationContext#refresh()。

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        //############省略其他代码##############
        try {
            //############省略其他代码##############
            invokeBeanFactoryPostProcessors(beanFactory);
           //############省略其他代码##############
        }catch (BeansException ex) {
            //############省略其他代码##############
        }finally {
            //############省略其他代码##############
        }
    }
}
```

refresh()方法是Spring中一个非常重要的方法，很多重要的功能和特性都是通过refresh()方法进行注入的。可以看到，在refresh()方法中，调用了invokeBeanFactoryPostProcessors()方法。

（4）解析AbstractApplicationContext类的invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory)方法

源码详见：org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory)。

```java
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
    if (!NativeDetector.inNativeImage() && beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }
}
```

可以看到，在AbstractApplicationContext类的invokeBeanFactoryPostProcessors()方法中调用了PostProcessorRegistrationDelegate类的invokeBeanFactoryPostProcessors()方法。

（5）解析PostProcessorRegistrationDelegate类的invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors)方法

源码详见：org.springframework.context.support.PostProcessorRegistrationDelegate#invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors)。

由于方法的源码比较长，这里，只关注当前最核心的逻辑，如下所示。

```java
public static void invokeBeanFactoryPostProcessors(
    ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

    //############省略其他代码##############
    List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();

    // First, invoke the BeanDefinitionRegistryPostProcessors that implement PriorityOrdered.
    String[] postProcessorNames =
        beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
    for (String ppName : postProcessorNames) {
        if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
            currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
            processedBeans.add(ppName);
        }
    }
    sortPostProcessors(currentRegistryProcessors, beanFactory);
    registryProcessors.addAll(currentRegistryProcessors);
    invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
    currentRegistryProcessors.clear();

    // Next, invoke the BeanDefinitionRegistryPostProcessors that implement Ordered.
    postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
    for (String ppName : postProcessorNames) {
        if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
            currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
            processedBeans.add(ppName);
        }
    }
    sortPostProcessors(currentRegistryProcessors, beanFactory);
    registryProcessors.addAll(currentRegistryProcessors);
    invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
    currentRegistryProcessors.clear();

    // Finally, invoke all other BeanDefinitionRegistryPostProcessors until no further ones appear.
    boolean reiterate = true;
    while (reiterate) {
        reiterate = false;
        postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
        for (String ppName : postProcessorNames) {
            if (!processedBeans.contains(ppName)) {
                currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                processedBeans.add(ppName);
                reiterate = true;
            }
        }
        sortPostProcessors(currentRegistryProcessors, beanFactory);
        registryProcessors.addAll(currentRegistryProcessors);
        invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());
        currentRegistryProcessors.clear();
    }
    //############省略其他代码##############
}
```

可以看到，在PostProcessorRegistrationDelegate类的invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors)方法中，BeanDefinitionRegistryPostProcessor的实现类在执行逻辑上会有先后顺序，并且最终都会调用invokeBeanDefinitionRegistryPostProcessors()方法。

（6）解析PostProcessorRegistrationDelegate类的invokeBeanDefinitionRegistryPostProcessors(Collection<? extends BeanDefinitionRegistryPostProcessor> postProcessors, BeanDefinitionRegistry registry, ApplicationStartup applicationStartup)方法

源码详见：org.springframework.context.support.PostProcessorRegistrationDelegate#invokeBeanDefinitionRegistryPostProcessors(Collection<? extends BeanDefinitionRegistryPostProcessor> postProcessors, BeanDefinitionRegistry registry, ApplicationStartup applicationStartup)。

```java
private static void invokeBeanDefinitionRegistryPostProcessors(
    Collection<? extends BeanDefinitionRegistryPostProcessor> postProcessors, BeanDefinitionRegistry registry, ApplicationStartup applicationStartup) {

    for (BeanDefinitionRegistryPostProcessor postProcessor : postProcessors) {
        StartupStep postProcessBeanDefRegistry = applicationStartup.start("spring.context.beandef-registry.post-process")
            .tag("postProcessor", postProcessor::toString);
        postProcessor.postProcessBeanDefinitionRegistry(registry);
        postProcessBeanDefRegistry.end();
    }
}
```

可以看到，在invokeBeanDefinitionRegistryPostProcessors()方法中，会循环遍历postProcessors集合中的每个元素，调用postProcessBeanDefinitionRegistry()方法注册Bean的定义信息。

（7）解析ConfigurationClassPostProcessor类的postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry)方法

源码详见：org.springframework.context.annotation.ConfigurationClassPostProcessor#postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry)。

```java
@Override
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
	//##########省略其他代码###################
    processConfigBeanDefinitions(registry);
}
```

可以看到，在postProcessBeanDefinitionRegistry()方法中，会调用processConfigBeanDefinitions()方法。

（8）解析ConfigurationClassPostProcessor类的processConfigBeanDefinitions(BeanDefinitionRegistry registry)方法

源码详见：org.springframework.context.annotation.ConfigurationClassPostProcessor#processConfigBeanDefinitions(BeanDefinitionRegistry registry)。

这里，重点关注方法中的如下逻辑。

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    //############省略其他代码#################
    // Parse each @Configuration class
    ConfigurationClassParser parser = new ConfigurationClassParser(
        this.metadataReaderFactory, this.problemReporter, this.environment,
        this.resourceLoader, this.componentScanBeanNameGenerator, registry);
    
    Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
    Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
    do {
        StartupStep processConfig = this.applicationStartup.start("spring.context.config-classes.parse");
        parser.parse(candidates);
        parser.validate();
        //############省略其他代码#################
        this.reader.loadBeanDefinitions(configClasses);
        alreadyParsed.addAll(configClasses);
        processConfig.tag("classCount", () -> String.valueOf(configClasses.size())).end();
        //############省略其他代码#################
    }
    while (!candidates.isEmpty());
    //############省略其他代码#################
}
```

可以看到，在processConfigBeanDefinitions()方法中，创建了一个ConfigurationClassParser类型的对象parser，并且调用了parser的parse()方法来解析类的配置信息。

（9）解析ConfigurationClassParser类的parse(Set<BeanDefinitionHolder> configCandidates)方法

源码详见：org.springframework.context.annotation.ConfigurationClassParser#parse(Set<BeanDefinitionHolder> configCandidates)。

```java
public void parse(Set<BeanDefinitionHolder> configCandidates) {
    for (BeanDefinitionHolder holder : configCandidates) {
        BeanDefinition bd = holder.getBeanDefinition();
        try {
            if (bd instanceof AnnotatedBeanDefinition) {
                parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());
            }
            else if (bd instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) bd).hasBeanClass()) {
                parse(((AbstractBeanDefinition) bd).getBeanClass(), holder.getBeanName());
            }
            else {
                parse(bd.getBeanClassName(), holder.getBeanName());
            }
        }
        catch (BeanDefinitionStoreException ex) {
            throw ex;
        }
        catch (Throwable ex) {
            throw new BeanDefinitionStoreException(
                "Failed to parse configuration class [" + bd.getBeanClassName() + "]", ex);
        }
    }
    this.deferredImportSelectorHandler.process();
}
```

可以看到，在ConfigurationClassParser类的parse(Set<BeanDefinitionHolder> configCandidates)方法中，调用了类中的另一个parse()方法。

（10）解析ConfigurationClassParser类的parse(AnnotationMetadata metadata, String beanName)方法

源码详见：org.springframework.context.annotation.ConfigurationClassParser#parse(AnnotationMetadata metadata, String beanName)

```java
protected final void parse(AnnotationMetadata metadata, String beanName) throws IOException {
    processConfigurationClass(new ConfigurationClass(metadata, beanName), DEFAULT_EXCLUSION_FILTER);
}
```

可以看到，上述parse()方法的实现比较简单，直接调用了processConfigurationClass()方法。

（11）解析ConfigurationClassParser类的processConfigurationClass(ConfigurationClass configClass, Predicate<String> filter)方法

源码详见：org.springframework.context.annotation.ConfigurationClassParser#processConfigurationClass(ConfigurationClass configClass, Predicate<String> filter)。

```java
protected void processConfigurationClass(ConfigurationClass configClass, Predicate<String> filter) throws IOException {
    //###############省略其他代码####################
    SourceClass sourceClass = asSourceClass(configClass, filter);
    do {
        sourceClass = doProcessConfigurationClass(configClass, sourceClass, filter);
    }
    while (sourceClass != null);
    this.configurationClasses.put(configClass, configClass);
}
```

可以看到，在processConfigurationClass()方法中，会通过do-while()循环获取配置类和其父类的注解信息，SourceClass类中会封装配置类上注解的详细信息。在在processConfigurationClass()方法中，调用了doProcessConfigurationClass()方法。

（12）解析ConfigurationClassParser类的doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)方法

源码详见：org.springframework.context.annotation.ConfigurationClassParser#doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)。

```java
protected final SourceClass doProcessConfigurationClass(
    ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)
    throws IOException {
    //##################省略其他代码##################
    // Process individual @Bean methods
    Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
    for (MethodMetadata methodMetadata : beanMethods) {
        configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
    }
    //##################省略其他代码##################
    // No superclass -> processing is complete
    return null;
}
```

这里，只关注与@Bean注解相关的方法，可以看到，在doProcessConfigurationClass()方法中调用了retrieveBeanMethodMetadata()方法来解析sourceClass中有关@Bean注解的属性信息。

（13）解析ConfigurationClassParser类的retrieveBeanMethodMetadata(SourceClass sourceClass)方法

源码详见：org.springframework.context.annotation.ConfigurationClassParser#retrieveBeanMethodMetadata(SourceClass sourceClass)。

```java
private Set<MethodMetadata> retrieveBeanMethodMetadata(SourceClass sourceClass) {
    AnnotationMetadata original = sourceClass.getMetadata();
    Set<MethodMetadata> beanMethods = original.getAnnotatedMethods(Bean.class.getName());
    if (beanMethods.size() > 1 && original instanceof StandardAnnotationMetadata) {
        try {
            AnnotationMetadata asm = this.metadataReaderFactory.getMetadataReader(original.getClassName()).getAnnotationMetadata();
            Set<MethodMetadata> asmMethods = asm.getAnnotatedMethods(Bean.class.getName());
            if (asmMethods.size() >= beanMethods.size()) {
                Set<MethodMetadata> selectedMethods = new LinkedHashSet<>(asmMethods.size());
                for (MethodMetadata asmMethod : asmMethods) {
                    for (MethodMetadata beanMethod : beanMethods) {
                        if (beanMethod.getMethodName().equals(asmMethod.getMethodName())) {
                            selectedMethods.add(beanMethod);
                            break;
                        }
                    }
                }
                if (selectedMethods.size() == beanMethods.size()) {
                    beanMethods = selectedMethods;
                }
            }
        }
        catch (IOException ex) {
            logger.debug("Failed to read class file via ASM for determining @Bean method order", ex);
        }
    }
    return beanMethods;
}
```

可以看到，在retrieveBeanMethodMetadata()方法中主要是解析@Bean注解，并且将解析到的方法元数据存入 Set<MethodMetadata>集合中并返回。

（14）回到ConfigurationClassParser类的doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)方法。

这里，还是重点看下方法中与@Bean注解有关的代码片段，如下所示。

```java
Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
for (MethodMetadata methodMetadata : beanMethods) {
    configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
}
```

调用retrieveBeanMethodMetadata()方法获取到标注了@Bean注解的方法的元数据集合后，遍历方法的元数据集合，将方法的元数据methodMetadata和配置类configClass传入BeanMethod类的构造方法，创建BeanMethod对象，并调用configClass的addBeanMethod()方法传入创建的BeanMethod对象。

configClass的addBeanMethod()方法的源码详见：org.springframework.context.annotation.ConfigurationClass#addBeanMethod(BeanMethod method)，如下所示。

```java
void addBeanMethod(BeanMethod method) {
    this.beanMethods.add(method);
}
```

可以看到，在addBeanMethod()方法中，调用了beanMethods的add()方法添加BeanMethod对象。

beanMethods的源码详见：org.springframework.context.annotation.ConfigurationClass#beanMethods，如下所示。

```java
private final Set<BeanMethod> beanMethods = new LinkedHashSet<>();
```

可以看到，beanMethods是一个LinkedHashSet类型的集合。也就是说，在ConfigurationClassParser类的doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)方法中，会将解析出的标注了@Bean注解的元数据封装成BeanMethod对象，添加到一个LinkedHashSet类型的beanMethods集合中。

（15）回到ConfigurationClassPostProcessor类的processConfigBeanDefinitions(BeanDefinitionRegistry registry)方法

源码详见：org.springframework.context.annotation.ConfigurationClassPostProcessor#processConfigBeanDefinitions(BeanDefinitionRegistry registry)。

此时，重点关注下如下源码片段。

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    //############省略其他代码#################
    do {
        //############省略其他代码#################
        this.reader.loadBeanDefinitions(configClasses);
        alreadyParsed.addAll(configClasses);
        processConfig.tag("classCount", () -> String.valueOf(configClasses.size())).end();
        //############省略其他代码#################
    }
    while (!candidates.isEmpty());
    //############省略其他代码#################
}
```

可以看到，在processConfigBeanDefinitions()方法的do-while()循环中，调用了reader的loadBeanDefinitions()方法来加载Bean的定义信息。

（16）解析ConfigurationClassBeanDefinitionReader类的loadBeanDefinitions(Set<ConfigurationClass> configurationModel)方法

源码详见：org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader#loadBeanDefinitions(Set<ConfigurationClass> configurationModel)

```java
public void loadBeanDefinitions(Set<ConfigurationClass> configurationModel) {
    TrackedConditionEvaluator trackedConditionEvaluator = new TrackedConditionEvaluator();
    for (ConfigurationClass configClass : configurationModel) {
        loadBeanDefinitionsForConfigurationClass(configClass, trackedConditionEvaluator);
    }
}
```

可以看到，在loadBeanDefinitions()方法中，会循环遍历传入的configurationModel集合，并调用loadBeanDefinitionsForConfigurationClass()方法处理遍历的每个元素。

（17）解析ConfigurationClassBeanDefinitionReader类的loadBeanDefinitionsForConfigurationClass(ConfigurationClass configClass, TrackedConditionEvaluator trackedConditionEvaluator)方法

源码详见：org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader#loadBeanDefinitionsForConfigurationClass(ConfigurationClass configClass, TrackedConditionEvaluator trackedConditionEvaluator)。

```java
private void loadBeanDefinitionsForConfigurationClass(
    ConfigurationClass configClass, TrackedConditionEvaluator trackedConditionEvaluator) {
    //###############省略其他代码################
    for (BeanMethod beanMethod : configClass.getBeanMethods()) {
        loadBeanDefinitionsForBeanMethod(beanMethod);
    }
	//###############省略其他代码################
}
```

可以看到，在loadBeanDefinitionsForConfigurationClass()方法中，会循环遍历通过configClass获取到的BeanMethod集合，并调用loadBeanDefinitionsForBeanMethod()方法处理遍历的每个BeanMethod对象。

（18）解析ConfigurationClassBeanDefinitionReader类的loadBeanDefinitionsForBeanMethod(BeanMethod beanMethod)方法

源码详见：org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader#loadBeanDefinitionsForBeanMethod(BeanMethod beanMethod)

```java
private void loadBeanDefinitionsForBeanMethod(BeanMethod beanMethod) {
    ConfigurationClass configClass = beanMethod.getConfigurationClass();
    MethodMetadata metadata = beanMethod.getMetadata();
    String methodName = metadata.getMethodName();
    if (this.conditionEvaluator.shouldSkip(metadata, ConfigurationPhase.REGISTER_BEAN)) {
        configClass.skippedBeanMethods.add(methodName);
        return;
    }
    if (configClass.skippedBeanMethods.contains(methodName)) {
        return;
    }
    AnnotationAttributes bean = AnnotationConfigUtils.attributesFor(metadata, Bean.class);
    Assert.state(bean != null, "No @Bean annotation attributes");
    List<String> names = new ArrayList<>(Arrays.asList(bean.getStringArray("name")));
    String beanName = (!names.isEmpty() ? names.remove(0) : methodName);
    for (String alias : names) {
        this.registry.registerAlias(beanName, alias);
    }
    if (isOverriddenByExistingDefinition(beanMethod, beanName)) {
        if (beanName.equals(beanMethod.getConfigurationClass().getBeanName())) {
            throw new BeanDefinitionStoreException(beanMethod.getConfigurationClass().getResource().getDescription(),
                                                   beanName, "Bean name derived from @Bean method '" + beanMethod.getMetadata().getMethodName() +
                                                   "' clashes with bean name for containing configuration class; please make those names unique!");
        }
        return;
    }

    ConfigurationClassBeanDefinition beanDef = new ConfigurationClassBeanDefinition(configClass, metadata, beanName);
    beanDef.setSource(this.sourceExtractor.extractSource(metadata, configClass.getResource()));

    if (metadata.isStatic()) {
        // static @Bean method
        if (configClass.getMetadata() instanceof StandardAnnotationMetadata sam) {
            beanDef.setBeanClass(sam.getIntrospectedClass());
        }
        else {
            beanDef.setBeanClassName(configClass.getMetadata().getClassName());
        }
        beanDef.setUniqueFactoryMethodName(methodName);
    }
    else {
        // instance @Bean method
        beanDef.setFactoryBeanName(configClass.getBeanName());
        beanDef.setUniqueFactoryMethodName(methodName);
    }

    if (metadata instanceof StandardMethodMetadata sam) {
        beanDef.setResolvedFactoryMethod(sam.getIntrospectedMethod());
    }

    beanDef.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_CONSTRUCTOR);
    AnnotationConfigUtils.processCommonDefinitionAnnotations(beanDef, metadata);

    boolean autowireCandidate = bean.getBoolean("autowireCandidate");
    if (!autowireCandidate) {
        beanDef.setAutowireCandidate(false);
    }

    String initMethodName = bean.getString("initMethod");
    if (StringUtils.hasText(initMethodName)) {
        beanDef.setInitMethodName(initMethodName);
    }

    String destroyMethodName = bean.getString("destroyMethod");
    beanDef.setDestroyMethodName(destroyMethodName);
    ScopedProxyMode proxyMode = ScopedProxyMode.NO;
    AnnotationAttributes attributes = AnnotationConfigUtils.attributesFor(metadata, Scope.class);
    if (attributes != null) {
        beanDef.setScope(attributes.getString("value"));
        proxyMode = attributes.getEnum("proxyMode");
        if (proxyMode == ScopedProxyMode.DEFAULT) {
            proxyMode = ScopedProxyMode.NO;
        }
    }
    BeanDefinition beanDefToRegister = beanDef;
    if (proxyMode != ScopedProxyMode.NO) {
        BeanDefinitionHolder proxyDef = ScopedProxyCreator.createScopedProxy(
            new BeanDefinitionHolder(beanDef, beanName), this.registry,
            proxyMode == ScopedProxyMode.TARGET_CLASS);
        beanDefToRegister = new ConfigurationClassBeanDefinition(
            (RootBeanDefinition) proxyDef.getBeanDefinition(), configClass, metadata, beanName);
    }

    if (logger.isTraceEnabled()) {
        logger.trace(String.format("Registering bean definition for @Bean method %s.%s()",
                                   configClass.getMetadata().getClassName(), beanName));
    }
    this.registry.registerBeanDefinition(beanName, beanDefToRegister);
}
```

可以看到，在loadBeanDefinitionsForBeanMethod()方法中解析了@Bean注解中的属性信息，并将解析出的信息封装到一个BeanDefinition对象中，最终会调用registry对象的registerBeanDefinition()方法将封装的BeanDefinition对象注册到IOC容器中。

（19）解析DefaultListableBeanFactory类的registerBeanDefinition(String beanName, BeanDefinition beanDefinition)方法

源码详见：org.springframework.beans.factory.support.DefaultListableBeanFactory#registerBeanDefinition(String beanName, BeanDefinition beanDefinition)

```java
@Override
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
    throws BeanDefinitionStoreException {
	//##############省略其他代码#################
    BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
    if (existingDefinition != null) {
        //##############省略其他代码#################
        this.beanDefinitionMap.put(beanName, beanDefinition);
    }
    else {
        //##############省略其他代码#################
        if (hasBeanCreationStarted()) {
            // Cannot modify startup-time collection elements anymore (for stable iteration)
            synchronized (this.beanDefinitionMap) {
                this.beanDefinitionMap.put(beanName, beanDefinition);
                List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
                updatedDefinitions.addAll(this.beanDefinitionNames);
                updatedDefinitions.add(beanName);
                this.beanDefinitionNames = updatedDefinitions;
                removeManualSingletonName(beanName);
            }
        }
        else {
            // Still in startup registration phase
            this.beanDefinitionMap.put(beanName, beanDefinition);
            this.beanDefinitionNames.add(beanName);
            removeManualSingletonName(beanName);
        }
        this.frozenBeanDefinitionNames = null;
    }
    if (existingDefinition != null || containsSingleton(beanName)) {
        resetBeanDefinition(beanName);
    }
    else if (isConfigurationFrozen()) {
        clearByTypeCache();
    }
}
```

可以看到，Spring会解析这些标注了@Bean注解的方法，将解析出的信息封装成BeanDefinition对象注册到beanDefinitionMap中。这一点和第1章中，注册ConfigurationClassPostProcessor类的Bean定义信息有点类似。

好了，至此，@Bean注解在Spring源码中的执行流程分析完毕。

### 5.2 调用初始化方法

本节，就简单介绍下@Bean注解在Spring源码层面调用初始化方法的源码执行流程，结合源码执行的时序图，会理解的更加深刻，可以结合图3-3~3-4进行理解，具体解析步骤如下所示。

（1）解析BeanTest类的main()方法

在BeanTest类的main()方法中，会调用AnnotationConfigApplicationContext类的构造方法创建IOC容器。

（2）解析AnnotationConfigApplicationContext类的AnnotationConfigApplicationContext(Class<?>... componentClasses)方法

源码详见：org.springframework.context.annotation.AnnotationConfigApplicationContext#AnnotationConfigApplicationContext(Class<?>... componentClasses)。

```java
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
    this();
    register(componentClasses);
    refresh();
}
```

可以看到，在AnnotationConfigApplicationContext类的构造方法中，会调用refresh()方法刷新IOC容器。

（3）解析AbstractApplicationContext类的refresh()方法

源码详见：org.springframework.context.support.AbstractApplicationContext#refresh()。

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
         /*************省略其他代码**************/
        try {
            /*************省略其他代码**************/
            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);
 			/*************省略其他代码**************/
        }
        catch (BeansException ex) {
            /*************省略其他代码**************/
        }
        finally {
            /*************省略其他代码**************/
        }
    }
}
```

可以看到，在AbstractApplicationContext类的refresh()方法中，会调用finishBeanFactoryInitialization()方法实例化未延迟创建的单例Bean。

（4）解析AbstractApplicationContext类的finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory)方法

源码详见：org.springframework.context.support.AbstractApplicationContext#finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory)。

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    /*************省略其他代码**************/
    // Instantiate all remaining (non-lazy-init) singletons.
    beanFactory.preInstantiateSingletons();
}
```

可以看到，在AbstractApplicationContext类的finishBeanFactoryInitialization()方法中，会调用beanFactory的preInstantiateSingletons()方法来创建非懒加载的Bean。

（5）解析DefaultListableBeanFactory类的preInstantiateSingletons()方法

源码详见：org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons()

```java
@Override
public void preInstantiateSingletons() throws BeansException {
   /*************省略其他代码**************/
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
    // Trigger initialization of all non-lazy singleton beans...
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            /*************省略其他代码**************/
            }
            else {
                getBean(beanName);
            }
        }
    }
	/*************省略其他代码**************/
}
```

可以看到，在DefaultListableBeanFactory类的preInstantiateSingletons()方法中，会循环遍历所有非懒加载的单例Bean的名称，调用getBean()方法创建单例Bean对象。

（6）解析AbstractBeanFactory类的getBean(String name)方法

源码详见：org.springframework.beans.factory.support.AbstractBeanFactory#getBean(String name)。

```java
@Override
public Object getBean(String name) throws BeansException {
    return doGetBean(name, null, null, false);
}
```

可以看到，在AbstractBeanFactory类的getBean()方法中，会调用doGetBean()方法创建Bean对象。

（7）解析AbstractBeanFactory类的doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)方法

源码详见：org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)。

```java
protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly) throws BeansException {
    String beanName = transformedBeanName(name);
    Object beanInstance;
    // Eagerly check singleton cache for manually registered singletons.
    Object sharedInstance = getSingleton(beanName);
    if (sharedInstance != null && args == null) {
        /***********省略其他代码*************/
    }
    else {
        /***********省略其他代码*************/
        try {
            /***********省略其他代码*************/
            // Create bean instance.
            if (mbd.isSingleton()) {
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        return createBean(beanName, mbd, args);
                    }
                    catch (BeansException ex) {
                        /***********省略其他代码*************/
                    }
                });
                beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }
			/***********省略其他代码*************/
        }
        catch (BeansException ex) {
           /***********省略其他代码*************/
        }
        finally {
           /***********省略其他代码*************/
        }
    }
    return adaptBeanInstance(name, beanInstance, requiredType);
}
```

可以看到，在AbstractBeanFactory类的doGetBean()方法中，会调用createBean()方法创建Bean对象。

（8）解析AbstractAutowireCapableBeanFactory类的createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)方法

源码详见：org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)。

```java
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
	/***********省略其他代码*************/
    try {
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        if (logger.isTraceEnabled()) {
            logger.trace("Finished creating instance of bean '" + beanName + "'");
        }
        return beanInstance;
    }
    catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
        /***********省略其他代码*************/
    }
    catch (Throwable ex) {
        /***********省略其他代码*************/
    }
}
```

可以看到，在AbstractAutowireCapableBeanFactory类的createBean()方法中，会调用doCreateBean()方法创建Bean对象。

（9）解析AbstractAutowireCapableBeanFactory类的doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)方法

源码详见：org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)。

```java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
	/***********省略其他代码*************/
    Object exposedObject = bean;
    try {
        populateBean(beanName, mbd, instanceWrapper);
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
    catch (Throwable ex) {
       /***********省略其他代码*************/
    }
	/***********省略其他代码*************/
    return exposedObject;
}
```

可以看到，在AbstractAutowireCapableBeanFactory类的doCreateBean()方法中，会调用initializeBean()方法初始化Bean对象。

（10）解析AbstractAutowireCapableBeanFactory类的initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd)方法

源码详见：org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd)。

```java
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
    /***********省略其他代码*************/
    try {
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    catch (Throwable ex) {
       /***********省略其他代码*************/
    }
    /***********省略其他代码*************/
    return wrappedBean;
}
```

可以看到，在AbstractAutowireCapableBeanFactory类的initializeBean()方法中，会调用invokeInitMethods()方法来调用初始化方法。

（11）解析AbstractAutowireCapableBeanFactory类的invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd)方法

源码详见：org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd)。

```java
protected void invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd) throws Throwable {
	/***********省略其他代码*************/
    if (mbd != null && bean.getClass() != NullBean.class) {
        String[] initMethodNames = mbd.getInitMethodNames();
        if (initMethodNames != null) {
            for (String initMethodName : initMethodNames) {
                if (StringUtils.hasLength(initMethodName) &&
                    !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
                    !mbd.hasAnyExternallyManagedInitMethod(initMethodName)) {
                    invokeCustomInitMethod(beanName, bean, mbd, initMethodName);
                }
            }
        }
    }
}
```

可以看到，在AbstractAutowireCapableBeanFactory类的invokeInitMethods()方法中，会调用invokeCustomInitMethod()方法来执行自定义的初始化方法。

（12）解析AbstractAutowireCapableBeanFactory类的invokeCustomInitMethod(String beanName, Object bean, RootBeanDefinition mbd, String initMethodName)方法

源码详见：org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeCustomInitMethod(String beanName, Object bean, RootBeanDefinition mbd, String initMethodName)。

```java
protected void invokeCustomInitMethod(String beanName, Object bean, RootBeanDefinition mbd, String initMethodName)  throws Throwable {
    Method initMethod = (mbd.isNonPublicAccessAllowed() ? BeanUtils.findMethod(bean.getClass(), initMethodName) : ClassUtils.getMethodIfAvailable(bean.getClass(), initMethodName));
	/***********省略其他代码*************/
    Method methodToInvoke = ClassUtils.getInterfaceMethodIfPossible(initMethod, bean.getClass());
    try {
        ReflectionUtils.makeAccessible(methodToInvoke);
        methodToInvoke.invoke(bean);
    }
    catch (InvocationTargetException ex) {
        throw ex.getTargetException();
    }
}
```

可以看到，在AbstractAutowireCapableBeanFactory类的invokeCustomInitMethod()方法中，会通过Java的反射机制调用自定义的初始化方法。在本章的案例程序中，就会调用User类的init()方法。

至此，@Bean注解调用初始化的方法流程分析完毕。

### 5.3 调用销毁方法

本节，就简单介绍下@Bean注解在Spring源码层面调用销毁方法的源码执行流程，结合源码执行的时序图，会理解的更加深刻，可以结合图3-5进行理解，具体解析步骤如下所示。

（1）解析BeanTest类的main()方法

在BeanTest类的main()方法中，会调用context对象的close()方法来关闭IOC容器。如下所示。

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(BeanConfig.class);
    /*********省略其他代码***********/
    context.close();
}
```

（2）解析AbstractApplicationContext类的close()方法

源码详见：org.springframework.context.support.AbstractApplicationContext#close()

```java
@Override
public void close() {
    synchronized (this.startupShutdownMonitor) {
        doClose();
        /*********省略其他代码***********/
    }
}
```

可以看到，在AbstractApplicationContext类的close()方法中，调用了doClose()方法来关闭IOC容器。

（3）解析AbstractApplicationContext类的doClose()方法

源码详见：org.springframework.context.support.AbstractApplicationContext#doClose()。

```java
protected void doClose() {
    // Check whether an actual close attempt is necessary...
    if (this.active.get() && this.closed.compareAndSet(false, true)) {
         /*********省略其他代码***********/
        // Destroy all cached singletons in the context's BeanFactory.
        destroyBeans();
 		/*********省略其他代码***********/
    }
}
```

可以看到，在AbstractApplicationContext类的doClose()方法中，会调用destroyBeans()方法销毁所有的单例Bean。

（4）解析AbstractApplicationContext类的destroyBeans()方法

源码详见：org.springframework.context.support.AbstractApplicationContext#destroyBeans()。

```java
protected void destroyBeans() {
    getBeanFactory().destroySingletons();
}
```

可以看到，在AbstractApplicationContext类的destroyBeans()方法中，调用了beanFactory的destroySingletons()方法来销毁单例Bean。

（5）解析DefaultListableBeanFactory类的destroySingletons()方法

源码详见：org.springframework.beans.factory.support.DefaultListableBeanFactory#destroySingletons()。

```java
@Override
public void destroySingletons() {
    super.destroySingletons();
    /*********省略其他代码***********/
}
```

可以看到，在DefaultListableBeanFactory类的destroySingletons()方法中，会调用父类的destroySingletons()方法。

（6）解析DefaultSingletonBeanRegistry类的destroySingletons()方法

源码详见：org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#destroySingletons()。

```java
public void destroySingletons() {
    /*********省略其他代码***********/
    for (int i = disposableBeanNames.length - 1; i >= 0; i--) {
        destroySingleton(disposableBeanNames[i]);
    }
	/*********省略其他代码***********/
}
```

可以看到，在DefaultSingletonBeanRegistry类的destroySingletons()方法中，会调用destroySingleton()方法销毁指定的单例Bean对象。

（7）解析DefaultListableBeanFactory类的destroySingleton(String beanName)方法

源码详见：org.springframework.beans.factory.support.DefaultListableBeanFactory#destroySingleton(String beanName)。

```java
@Override
public void destroySingleton(String beanName) {
    super.destroySingleton(beanName);
    /*********省略其他代码***********/
}
```

可以看到，在DefaultListableBeanFactory类的destroySingleton()方法中，会调用父类的destroySingleton()方法销毁指定的单例Bean。

（8）解析DefaultSingletonBeanRegistry类的destroySingleton(String beanName)方法

源码详见：org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#destroySingleton(String beanName)。

```java
public void destroySingleton(String beanName) {
    removeSingleton(beanName);
    DisposableBean disposableBean;
    synchronized (this.disposableBeans) {
        disposableBean = (DisposableBean) this.disposableBeans.remove(beanName);
    }
    destroyBean(beanName, disposableBean);
}
```

可以看到，在DefaultSingletonBeanRegistry类的destroySingleton(方法中，会调用destroyBean()方法来销毁指定的单例Bean对象。

（9）解析DefaultSingletonBeanRegistry类的destroyBean(String beanName, @Nullable DisposableBean bean)方法

源码详见：org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#destroyBean(String beanName, @Nullable DisposableBean bean)。

```java
protected void destroyBean(String beanName, @Nullable DisposableBean bean) {
    /***********省略其他代码***********/
    if (bean != null) {
        try {
            bean.destroy();
        }
        catch (Throwable ex) {
            /***********省略其他代码***********/
        }
    }
    /***********省略其他代码***********/
}
```

可以看到，在DefaultSingletonBeanRegistry类的destroyBean()方法中，会调用bean对象的destroy()方法销毁Bean对象。

（10）解析DisposableBeanAdapter类的destroy()方法

源码详见：org.springframework.beans.factory.support.DisposableBeanAdapter#destroy()。

```java
@Override
public void destroy() {
    /***********省略其他代码***********/
    else if (this.destroyMethods != null) {
        for (Method destroyMethod : this.destroyMethods) {
            invokeCustomDestroyMethod(destroyMethod);
        }
    }
    /***********省略其他代码***********/
}
```

可以看到，在DisposableBeanAdapter类的destroy()方法中，会调用invokeCustomDestroyMethod()方法执行自定义的销毁方法。

（11）解析DisposableBeanAdapter类的invokeCustomDestroyMethod(Method destroyMethod)方法

源码详见：org.springframework.beans.factory.support.DisposableBeanAdapter#invokeCustomDestroyMethod(Method destroyMethod)。

```java
private void invokeCustomDestroyMethod(Method destroyMethod) {
    /***********省略其他代码***********/
    try {
        ReflectionUtils.makeAccessible(destroyMethod);
        destroyMethod.invoke(this.bean, args);
    } catch (InvocationTargetException ex) {
        /***********省略其他代码***********/
    } catch (Throwable ex) {
        /***********省略其他代码***********/
    }
}
```

可以看到，在DisposableBeanAdapter类的invokeCustomDestroyMethod()中，最终会通过Java的反射技术调用自定义的销毁方法。在本章的案例程序中，最终会调用User类的destroy()方法。

至此，@Bean注解在Spring源码层面调用销毁方法的源码执行流程分析完毕。

## 六、总结

`@Bean注解讲完了，我们一起总结下吧！`

本章，主要对@Bean注解进行了系统性的介绍。首先，对@Bean注解的源码和使用场景进行了简单的介绍。随后，给出了@Bean注解的使用案例。接下来，对@Bean注解在Spring源码层面的执行时序图和源码流程进行了详解的介绍。

## 七、思考

`既然学完了，就开始思考几个问题吧？`

* @Bean注解为何也是先将Bean定义信息注册到IOC容器中呢？这样做的好处是什么呢？
* @Bean注解标注的方法被Spring解析后将Bean定义信息注册到IOC容器中后，何时会对注册的信息进行实例化呢？实例化的流程是怎样的呢？
* Spring是如何调用@Bean注解中使用initMethod和destroyMethod属性指定的方法的？调用的流程是怎样的呢？

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

