# 第3章 Flowable工作流引擎配置

## 1.Flowable工作流引擎的配置

**ProcessEngine**：工作流引擎对象，是Flowable工作流引擎的核心。主要用于管理和执行流程的引擎，同时还负责创建，部署，执行和监控流程。我的理解这个对象更多的像一个工作流引擎的服务对象，他可以提供各种各样的服务对象，帮助我们更好的了解（定义流程，部署发布流程，启动流程，执行流程任务，监控流程状态，结束，查询历史，管理流程）这样一个完整的生命周期。

**ProcessEngineConfiguration**:流程引擎核心配置类，主要用于构建，配置流程引擎。ProcessEngineConfiguration当中提供了很多对流程引擎配置的属性。可以让用户更加灵活的控制和优化流程引擎的性能和行为。



Flowable启动两个条件：

- 创建一个流程引擎配置对象ProcessEngineConfiguration的实例并进行属性配置
- 通过流程引擎配置对象创建一个工作流引擎对象ProcessEngine实例



### 流程引擎核心配置对象ProcessEngineConfiguration

流程引擎中需要配置的方面包含：

- 数据库配置
- 事务配置
- 工作流引擎内置服务配置

**ProcessEngineConfiguration**是一个抽象类它继承父类**AbstractEngineConfiguration**

![image-20241030000238179](https://curleyg-1311489005.cos.ap-shanghai.myqcloud.com/202410300002070.png)

AbstractEngineConfiguration是实现工作流引擎配置的顶层抽象类，除了ProcessEngineConfiguration外，它还有其他的子类

- EventRegistryEngineConfiguration：事件注册引擎配置对象
- FormEngineConfiguration:表单引擎配置对象
- IdmEngineConfiguration：身份引擎配置对象
- DmnEngineConfiguration:决策引擎配置对象



AbstractEngineConfiguration主要用于配置Flowable引擎中使用的各种数据库信息，包括（数据库类型，数据库驱动，数据库URL，数据库用户名，密码等）。我们可以根据不同的需求可以对配置对象进行改造。例如我们在进行flowable适配国产数据库的时间就可以对AbstractEngineConfiguration进行定制。

ProcessEngineConfiguration是所有工作流引擎配置类的父类，它有一个子类ProcessEngineConfigurationImpl（抽象类），**ProcessEngineConfigurationImpl**有四个直接子类：

![](https://curleyg-1311489005.cos.ap-shanghai.myqcloud.com/202410300043645.png)

**StandaloneProcessEngineConfiguration**：标准工作流引擎配置类。使用这个类当中流程引擎配置对象，会对数据库事务进行管理，默认情况下启动流程引擎还会检查数据库结构和版本是否版本一致。

**SpringProcessEngineConfiguration**：Spring环境工作流引擎配置对象，当Spring和flowable整合时，可以使用此类。它提供了以下功能：

- 创建流程引擎实例对象后自动的部署流程配置的流程文档（需要设置），
- 设置工作流引擎连接的数据源，
- 事务管理器等

由Spring代理创建工作流引擎对象和事务管理对象，将Flowable引擎嵌入业务系统，实现业务系统事务和Flowable事务的统一性管理。

**JtaProcessEngineConfiguration**：JTA工作流引擎配置类，该类不使用Flowable事务，适用于JTA管理事务

**MultiSchemaMultiTenantProcessEngineConfiguration**：多数据库多租户工作流引擎配置类。Flowable通过此类提供路由机制，当工作流引擎需要连接多个数据库的时候，客户无需关心需要连接那个数据库，该类会通过路由规则自动的选择需要连接的数据库。

**StandaloneProcessEngineConfiguration**子类**StandaloneInMemProcessEngineConfiguration**适用于单元测试，默认采用H2数据库，由Flowable处理事务。



ProcessEngineConfigurationImpl实现类四个接口：

![image-20241030010025439](C:\Users\CurleyG\AppData\Roaming\Typora\typora-user-images\image-20241030010025439.png)

*ScriptingEngineAwareEngineConfiguration*：用于配置Flowable引擎的脚本引擎。脚本引擎用于执行一些脚本任务，比如执行脚本任务节点中的脚本

*HasExpressionManagerEngineConfiguration*：用于配置Flowable引擎的表达式管理器。表达式管理器用于解释和执行表达式。比如流程定义中条件表达式

*HasVariableTypes*：用于配置Flowable引擎的变量类型。变量类型用于定义流程实例中的变量的数据类型，比如字符串、整数、日期。json等

*HasVariableServiceConfiguration*:用于配置流程引擎的变量服务。变量服务用于管理流程实例中的变量，主要操作包括：设置变量获取变量删除变量等操作。

ProcessEngineConfiguration中创建流程配置对象的方法如下：

方法一:默认加载classpath下的flowable.cfg.xml bean配置文件，利用Spring的依赖注入从工厂对象中获取processEngineConfiguration名称的bean对象

```java
public static ProcessEngineConfiguration createProcessEngineConfigurationFromResourceDefault() {
        return createProcessEngineConfigurationFromResource("flowable.cfg.xml", "processEngineConfiguration");
    }


```

方法二:加载指定配置文件，从工厂对象中获取processEngineConfiguration名称的bean对象

```java
public static ProcessEngineConfiguration createProcessEngineConfigurationFromResourceDefault() {
        return createProcessEngineConfigurationFromResource("flowable.cfg.xml", "processEngineConfiguration");
    }


```

方法三：前两种方法的事迹调用方法，

```java
public static ProcessEngineConfiguration createProcessEngineConfigurationFromResource(String resource) {
    return createProcessEngineConfigurationFromResource(resource, "processEngineConfiguration");
}
```

方法三：加载resource路径的配置文件，获取名称为beanName的对象实力

```java
public static ProcessEngineConfiguration createProcessEngineConfigurationFromResource(String resource, String beanName) {
    return (ProcessEngineConfiguration) BeansConfigurationHelper.parseEngineConfigurationFromResource(resource, beanName);
}
//通过源码解释代码
//BeansConfigurationHelper.parseEngineConfigurationFromResource.(resource, beanName) 方法如下

    public static AbstractEngineConfiguration parseEngineConfigurationFromResource(String resource, String beanName) {
        //访问classpath下resource参数路径配置文件，转换成Resource对象
        Resource springResource = new ClassPathResource(resource);
        return parseEngineConfiguration(springResource, beanName);
    }

 //这段代码的主要作用是通过读取Spring资源文件（XML配置文件）来解析并创建一个AbstractEngineConfiguration对象
    public static AbstractEngineConfiguration parseEngineConfiguration(Resource springResource, String beanName) {
        //DefaultListableBeanFactory作为Spring的bean工厂 ，用于管理Bean的实例化和依赖关系
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        //XmlBeanDefinitionReader用于读取和解析Spring XML配置文件，并将解析后的Bean定义加载到beanFactory中
        XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
        //设置XML验证模式：
        xmlBeanDefinitionReader.setValidationMode(XmlBeanDefinitionReader.VALIDATION_XSD);
        //加载配置文件
        xmlBeanDefinitionReader.loadBeanDefinitions(springResource);

        // 处理非单例Bean的类型：
        // 使用getBeansOfType方法获取所有BeanFactoryPostProcessor类型的Bean实例，但不会立刻初始化FactoryBeans。如果没有定义BeanFactoryPostProcessor，则默认创建一个PropertyPlaceholderConfigurer实例来处理Bean属性的占位符替换
        Collection<BeanFactoryPostProcessor> factoryPostProcessors = beanFactory.getBeansOfType(BeanFactoryPostProcessor.class, true, false).values();
        if (factoryPostProcessors.isEmpty()) {
            factoryPostProcessors = Collections.singleton(new PropertyPlaceholderConfigurer());
        }
        //对每个BeanFactoryPostProcessor实例调用postProcessBeanFactory方法，完成Bean工厂的后置处理，例如属性占位符的替换、Bean定义的调整等。
        for (BeanFactoryPostProcessor factoryPostProcessor : factoryPostProcessors) {
            factoryPostProcessor.postProcessBeanFactory(beanFactory);
        }
        //通过beanName在beanFactory中获取AbstractEngineConfiguration实例对象
        AbstractEngineConfiguration engineConfiguration = (AbstractEngineConfiguration) beanFactory.getBean(beanName);
        //将beanFactory封装成SpringBeanFactoryProxyMap并赋值给engineConfiguration，确保配置类可以访问beanFactory中的其他Bean
        engineConfiguration.setBeans(new SpringBeanFactoryProxyMap(beanFactory));
       // 返回engineConfiguration
        return engineConfiguration;
    }
```

方法四：通过输入流加载任意路径下的文件，获取默认名称的bean

```java

public static ProcessEngineConfiguration createProcessEngineConfigurationFromInputStream(InputStream inputStream) {
    return createProcessEngineConfigurationFromInputStream(inputStream, "processEngineConfiguration");
}
```

方法五:通过输入流加载任意路径下的文件，获取指定名称的bean

```java
public static ProcessEngineConfiguration createProcessEngineConfigurationFromInputStream(InputStream inputStream, String beanName) {
    return (ProcessEngineConfiguration) BeansConfigurationHelper.parseEngineConfigurationFromInputStream(inputStream, beanName);
}

    public static AbstractEngineConfiguration parseEngineConfigurationFromInputStream(InputStream inputStream, String beanName) {
        //将文件输入流转换成Resource对象
        Resource springResource = new InputStreamResource(inputStream);
        //源码解释在方法二中
        return parseEngineConfiguration(springResource, beanName);
    }

```

方法六：不加载任何的配置文件，所有属性均硬编码在代码中

```java
public static ProcessEngineConfiguration createStandaloneProcessEngineConfiguration() {
    //创建StandaloneProcessEngineConfiguration对象实力并返回
    return new StandaloneProcessEngineConfiguration();
}

//解释源码
public class StandaloneProcessEngineConfiguration extends ProcessEngineConfigurationImpl {

    @Override
    public CommandInterceptor createTransactionInterceptor() {
        return null;
    }

}

//其所有的属性均已在 ProcessEngineConfiguration中设置

```

方法七：该方法也是基于硬编码的方式创建工作流对象，也不加载任何配置。

```java

public static ProcessEngineConfiguration createStandaloneInMemProcessEngineConfiguration() {
    return new StandaloneInMemProcessEngineConfiguration();
}
//StandaloneInMemProcessEngineConfiguration是StandaloneProcessEngineConfiguration的子类，实现了StandaloneInMemProcessEngineConfiguration方法，指定了一些属性的默认值

public class StandaloneInMemProcessEngineConfiguration extends StandaloneProcessEngineConfiguration {

    public StandaloneInMemProcessEngineConfiguration() {
        //默认数据库表结构的更新策略
        this.databaseSchemaUpdate = DB_SCHEMA_UPDATE_CREATE_DROP;
        //默认数据库Url
        this.jdbcUrl = "jdbc:h2:mem:flowable";
    }
}

```

数据库更新策略有以下几种：

**`DB_SCHEMA_UPDATE_CREATE_DROP`**：启动时创建数据库表，关闭时删除表。这在开发和测试环境中非常有用，因为每次运行时都会重建表以确保表结构是最新的。

**`DB_SCHEMA_UPDATE_TRUE`**：如果数据库表已经存在，Flowable会自动更新它们（例如，添加新列或更改表结构），但不会删除现有数据。这是生产环境的常见设置，可以自动适应表结构的变化。

**`DB_SCHEMA_UPDATE_FALSE`**：Flowable 不会对数据库表进行任何操作，假设表结构已经手动创建和配置好了。这适合严格控制的生产环境，防止应用程序无意更改数据库结构。

## 2.工作流引擎对象ProcessEngine

一个ProcessEngine实例代表一个工作流引擎。ProcessEngineti中提供了很多获取对应工作流引擎的服务对象的API，比如

**`getRepositoryService`**：

- **获取对象**：`RepositoryService`
- **作用**：管理和操作流程定义及相关资源。可以用于部署、查询、删除流程定义，以及管理流程模型、部署资源等。该服务主要负责流程的元数据管理，不涉及实际的流程实例。
- **常见功能**：流程定义部署、查询已部署流程、管理流程模型等。

**`getTaskService`**：

- **获取对象**：`TaskService`
- **作用**：管理用户任务，是操作任务的核心服务。通过它可以创建、查询、完成任务，还可以设置任务的分配人、优先级等。用于日常的任务管理。
- **常见功能**：分配任务、完成任务、查询待办任务等。

**`getRuntimeService`**：

- **获取对象**：`RuntimeService`
- **作用**：管理流程实例的运行时数据。可以用于启动流程实例、管理流程变量、查询流程实例和执行的相关信息。该服务专注于流程实例的操作和管理。
- **常见功能**：启动流程实例、查询流程实例、管理和获取流程变量。

**`getHistoryService`**：

- **获取对象**：`HistoryService`
- **作用**：管理历史数据。用于查询流程实例和任务的历史信息，例如流程实例完成的时间、任务的处理人、任务完成的状态等。历史数据通常在审计、报告和跟踪等场景中使用。
- **常见功能**：查询已完成的流程实例和任务，获取历史数据和流程轨迹。

**`getManagementService`**：

- **获取对象**：`ManagementService`
- **作用**：提供引擎管理和维护功能。用于查询和操作一些流程引擎内部的状态信息和配置，通常在调试和维护时使用。主要是辅助性功能，不直接涉及流程管理。

ProcessEngine的创建的方式：

方式一：ProcessEngineConfiguration的buildProcessEngine()方法创建

```java
  //ProcessEngineConfiguration父类定义
public abstract ProcessEngine buildProcessEngine();
  
  
 // ProcessEngineConfigurationImpl子类实现
@Override
public ProcessEngine buildProcessEngine() {
    // 初始化引擎配置
    init();

    // 创建ProcessEngineImpl对象（流程引擎的核心实现类），并传入当前配置对象
    ProcessEngineImpl processEngine = new ProcessEngineImpl(this);

    // 判断是否在创建引擎后启动执行器
    if (handleProcessEngineExecutorsAfterEngineCreate) {
        // 启动流程引擎中的执行器，管理异步任务和定时任务
        processEngine.startExecutors();
    }

    // 检查是否启用了Flowable 5兼容模式并且存在兼容处理器
    if (flowable5CompatibilityEnabled && flowable5CompatibilityHandler != null) {
        // 使用命令模式，执行兼容模式下的流程引擎初始化
        commandExecutor.execute(new Command<Void>() {

            @Override
            public Void execute(CommandContext commandContext) {
                // 获取Flowable 5的原始流程引擎，兼容旧版引擎的操作
                flowable5CompatibilityHandler.getRawProcessEngine();
                return null;
            }
        });
    }

    // 在流程引擎初始化完成后进行后处理操作
    postProcessEngineInitialisation();

    // 返回构建完成的流程引擎实例
    return processEngine;
}

```

方式二：通过ProcessEngines创建

ProcessEngines是一个流程引擎的工具类，主要用于创建和关闭工作流引擎。并且所有创建的ProcessEngine实例都会注册到ProcessEngines中。在其中维护了一个Map key为实例对象的名称，value 为ProcessEngine实例对象。通过ProcessEngines创建ProcessEngine对象有以下两种方式

方式一：通过init()方法创建

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

public static ProcessEngine getDefaultProcessEngine() {
        return getProcessEngine(NAME_DEFAULT);
}

public static ProcessEngine getProcessEngine(String processEngineName) {
    //判断是否已初始化
        if (!isInitialized()) {
            //没有初始化进行初始化
            init();
        }
    //从map中获取key 为processEngineName的ProcessEngine实例
 return processEngines.get(processEngineName);
}


public static synchronized void init() {
    // 判断是否已初始化
    if (!isInitialized()) {
        // 如果processEngines为空，则创建一个新的HashMap用于存储流程引擎
        if (processEngines == null) {
            processEngines = new HashMap<>();
        }

        // 获取当前类加载器
        ClassLoader classLoader = ReflectUtil.getClassLoader();
        Enumeration<URL> resources = null;
        try {
            // 查找classpath下的“flowable.cfg.xml”配置文件
            resources = classLoader.getResources("flowable.cfg.xml");
        } catch (IOException e) {
            throw new FlowableIllegalArgumentException("在classpath中检索flowable.cfg.xml资源时出现问题：" + System.getProperty("java.class.path"), e);
        }

        // 使用Set去重配置URL，避免重复加载相同的URL
        Set<URL> configUrls = new HashSet<>();
        while (resources.hasMoreElements()) {
            configUrls.add(resources.nextElement());
        }
        
        // 遍历每个配置URL，并初始化流程引擎
        for (URL resource : configUrls) {
            LOGGER.info("使用配置文件 '{}' 初始化流程引擎", resource);
            initProcessEngineFromResource(resource);
        }

        try {
            // 查找classpath下的“flowable-context.xml”Spring配置文件
            resources = classLoader.getResources("flowable-context.xml");
        } catch (IOException e) {
            throw new FlowableIllegalArgumentException("在classpath中检索flowable-context.xml资源时出现问题：" + System.getProperty("java.class.path"), e);
        }
        
        // 遍历每个Spring配置文件URL，并通过Spring配置初始化流程引擎
        while (resources.hasMoreElements()) {
            URL resource = resources.nextElement();
            LOGGER.info("使用Spring配置文件 '{}' 初始化流程引擎", resource);
            initProcessEngineFromSpringResource(resource);
        }

        // 设置初始化完成标志
        setInitialized(true);
    } else {
        LOGGER.info("流程引擎已初始化");
    }
}

```

## 3.Flowable工作流引擎配置文件

Flowable配置文件主要分为两种配置方式：

第一种：遵循Flowable配置风格

这种方式是Flowable默认的配置方式，使用这种配置方式的配置文件名称默认为flowanle.cfg.xml。加载这类配置文件的方法为

ProcessEngines中的initProcessEngineFromSpringResource(resource);

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--数据源配置-->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
        <property name="driverClassName" value="org.h2.Driver"/>
        <property name="url" value="jdbc:h2:mem:flowable"/>
        <property name="username" value="sa"/>
        <property name="password" value=""/>
    </bean>
    
    <!-- Flowable流程引擎-->
    <bean id="processEngineConfiguration"
          class="org.flowable.engine.impl.cfg.StandaloneProcessEngineConfiguration">
        <!-- 数据源 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 数据库更新策略 -->
        <property name="databaseSchemaUpdate" value="drop-create"/>

        <!-- 表示是否激活异步执行器。如果设置为true，则会启用异步执行器；如果设置为false，则禁用异步执行器。默认值为true。 -->
        <property name="asyncExecutorActivate" value="false"/>
        <!-- 表示异步执行器的配置。该配置用于配置异步执行器的行为和属性，例如线程池大小、队列容量等。可以通过配置该属性来指定具体的异步执行器配置。 -->
        <property name="asyncExecutorConfiguration" ref="asyncJobExecutorConfiguration"/>
        <!-- 表示异步执行器的实例。异步执行器用于执行异步任务，例如异步消息发送和定时任务。可以通过配置该属性来指定具体的异步执行器实例。 -->
        <property name="asyncExecutor" ref="asyncJobExecutor"/>
        <!-- 表示异步任务执行器的实例。异步任务执行器用于执行异步任务，例如异步方法调用和异步事件处理。可以通过配置该属性来指定具体的异步任务执行器实例。 -->
        <property name="asyncTaskExecutor" ref="asyncTaskExecutor"/>
        <!-- 表示异步任务执行器的配置。该配置用于配置异步任务执行器的行为和属性，例如线程池大小、队列容量等。可以通过配置该属性来指定具体的异步任务执行器配置。 -->
        <property name="asyncExecutorTaskExecutorConfiguration" ref="asyncExecutorTaskExecutorConfiguration"/>

        <!-- 此属性用于启用或禁用异步历史记录功能。如果设置为true，则Flowable将以异步方式处理历史记录数据。默认值为false。 -->
        <property name="asyncHistoryEnabled" value="false"/>
        <!-- 这是一个配置类，用于定义异步历史记录执行器的行为。您可以配置异步历史记录执行器的线程池大小、最大等待时间等。 -->
        <!--
        <property name="asyncHistoryExecutorConfiguration" ref="asyncHistoryJobExecutorConfiguration"/>
        -->
        <!-- 此属性用于控制异步历史记录执行器是否在引擎启动时自动激活。如果设置为true，则引擎启动时将自动激活异步历史记录执行器。默认值为true。 -->
        <property name="asyncHistoryExecutorActivate" value="true"/>
        <!-- 此属性用于配置异步历史记录执行器的实现类。异步历史记录执行器负责处理历史记录数据的异步处理。可以使用asyncHistoryExecutorConfiguration来配置它的行为。默认情况下，Flowable使用DefaultAsyncHistoryExecutor作为异步历史记录执行器。 -->
        <!--
        <property name="asyncHistoryExecutor" ref="asyncHistoryExecutor"/>
        -->
        <!-- 此属性用于配置异步历史记录执行器的线程池。异步历史记录执行器使用线程池来执行异步历史记录任务。以使用asyncHistoryExecutorConfiguration来配置它的行为。默认情况下，Flowable使用DefaultAsyncHistoryTaskExecutor作为异步历史记录执行器的线程池。 -->
        <!--
        <property name="asyncHistoryTaskExecutor" ref="asyncHistoryTaskExecutor"/>
        -->
        <!-- The number of retries for a job. -->
        <property name="asyncExecutorNumberOfRetries" value="3"/>    <!-- asyncExecutorNumberOfRetries -->
    </bean>

    <bean id="asyncJobExecutorConfiguration"
          class="org.flowable.job.service.impl.asyncexecutor.AsyncJobExecutorConfiguration">
        <!-- 异步作业在获取时被锁定的时间。在此期间，没有其他异步 executor 会尝试获取和锁定此作业. -->
        <property name="asyncJobLockTime">
            <bean class="java.time.Duration" factory-method="ofHours">
                <constructor-arg value="1"/>
            </bean>
        </property>
        <!-- 是否启动线程以获取异步作业。可以用于启动仍然执行来自该实例的作业的引擎实例，但不自己获取新作业。 -->
        <property name="asyncJobAcquisitionEnabled" value="true"/>
        <!-- 是否启动线程以获取定时器作业。 -->

        <property name="timerJobAcquisitionEnabled" value="true"/>
        <!-- 是否启动线程以重置过期的作业。 -->
        <property name="resetExpiredJobEnabled" value="true"/>
        <!-- 是否在启动或关闭时解锁当前执行器所拥有的作业（具有相同lockOwner）。 -->
        <property name="unlockOwnedJobs" value="true"/>
        <!-- 是否启用获取定时器作业的runnable。 -->
        <property name="timerRunnableNeeded" value="true"/>

        <!-- 异步作业获取线程的名称。 -->
        <property name="acquireRunnableThreadName" value="acquireRunnableThreadName"/>

        <!-- 重置过期作业的线程名称。 -->
        <property name="resetExpiredRunnableName" value="resetExpiredRunnableName"/>

        <!-- 定时器作业迁移的线程池大小。 -->
        <property name="moveTimerExecutorPoolSize" value="4"/>

        <!-- 每次获取操作中应获取的最大定时器/历史作业数。 -->
        <property name="maxTimerJobsPerAcquisition" value="512"/>

        <!-- 每次获取操作中应获取的最大异步/历史作业数。 -->
        <property name="maxAsyncJobsDuePerAcquisition" value="512"/>

        <!-- 定时器获取线程在执行下一次获取逻辑之前的等待时间。 -->
        <property name="defaultTimerJobAcquireWaitTime"> <!-- asyncExecutorDefaultTimerJobAcquireWaitTime -->
            <bean class="java.time.Duration" factory-method="ofSeconds">
                <constructor-arg value="10"/>
            </bean>
        </property>

        <!-- 异步作业获取线程在执行下一次获取逻辑之前的等待时间。 -->
        <property name="defaultAsyncJobAcquireWaitTime"> <!-- asyncExecutorDefaultAsyncJobAcquireWaitTime -->
            <bean class="java.time.Duration" factory-method="ofSeconds">
                <constructor-arg value="10"/>
            </bean>
        </property>

        <!-- 当队列已满时获取线程的等待时间。 -->
        <property name="defaultQueueSizeFullWaitTime"> <!-- asyncExecutorDefaultQueueSizeFullWaitTime -->
            <bean class="java.time.Duration" factory-method="ofSeconds">
                <constructor-arg value="5"/>
            </bean>
        </property>

        <!-- 是否使用全局获取锁。 -->
        <property name="globalAcquireLockEnabled" value="true"/>

        <!-- 可用于全局获取锁的前缀，设置不同前缀可使不同引擎/执行器不竞争相同的锁。 -->
        <property name="globalAcquireLockPrefix" value=""/>

        <!-- 异步作业获取线程尝试获取全局锁的等待时间。 -->
        <property name="asyncJobsGlobalLockWaitTime">
            <bean class="java.time.Duration" factory-method="ofMinutes">
                <constructor-arg value="1"/>
            </bean>
        </property>

        <!-- 异步作业获取线程检查全局锁是否已释放的轮询速率。 -->
        <property name="asyncJobsGlobalLockPollRate">
            <bean class="java.time.Duration" factory-method="ofMillis">
                <constructor-arg value="500"/>
            </bean>
        </property>

        <!-- 在上一次全局锁获取时间后锁被强制获取的时间。如果其他节点未正确释放锁（因崩溃），则另一节点可以获取锁。 -->
        <property name="asyncJobsGlobalLockForceAcquireAfter">
            <bean class="java.time.Duration" factory-method="ofMinutes">
                <constructor-arg value="10"/>
            </bean>
        </property>

        <!-- 定时器作业获取线程尝试获取全局锁的等待时间。 -->
        <property name="timerLockWaitTime">
            <bean class="java.time.Duration" factory-method="ofMinutes">
                <constructor-arg value="1"/>
            </bean>
        </property>

        <!-- 定时器作业获取线程检查全局锁是否已释放的轮询速率。 -->
        <property name="timerLockPollRate">
            <bean class="java.time.Duration" factory-method="ofMillis">
                <constructor-arg value="500"/>
            </bean>
        </property>

        <!-- 在上一次全局锁获取时间后锁被强制获取的时间。如果其他节点未正确释放锁（因崩溃），则另一节点可以获取锁。 -->
        <property name="timerLockForceAcquireAfter">
            <bean class="java.time.Duration" factory-method="ofMinutes">
                <constructor-arg value="10"/>
            </bean>
        </property>

        <!-- 定时器作业在被获取时锁定的时间，在此期间其他异步执行器不会尝试获取该作业。 -->
        <property name="timerLockTime"> <!-- asyncExecutorTimerLockTimeInMillis -->
            <bean class="java.time.Duration" factory-method="ofHours">
                <constructor-arg value="1"/>
            </bean>
        </property>

        <!-- 重置过期作业线程在执行下一次重置逻辑前的等待时间。过期作业指被锁定但未完成的作业。重置后作业再次可用。 -->
        <property name="resetExpiredJobsInterval"> <!-- asyncExecutorResetExpiredJobsInterval -->
            <bean class="java.time.Duration" factory-method="ofMinutes">
                <constructor-arg value="1"/>
            </bean>
        </property>

        <!-- 每轮重置周期中重置过期作业的数量。 -->
        <property name="resetExpiredJobsPageSize" value="3"/> <!-- asyncExecutorResetExpiredJobsPageSize -->

        <!-- 异步执行器在解锁作业时使用的租户ID。 -->
        <property name="tenantId" value=""/>
    </bean>
    <!--
        <bean id="asyncHistoryJobExecutorConfiguration"
              class="org.flowable.job.service.impl.asyncexecutor.AsyncJobExecutorConfiguration">
            <property name="acquireRunnableThreadName" value="acquireRunnableThreadName2"/>
            <property name="resetExpiredRunnableName" value="resetExpiredRunnableName2"/>
        </bean>
    -->
    <bean id="asyncJobExecutor" class="org.flowable.job.service.impl.asyncexecutor.DefaultAsyncJobExecutor">
        <!-- 引用的异步执行器配置 -->
        <property name="configuration" ref="asyncJobExecutorConfiguration"/>
        <!-- 引用的异步任务执行器 -->
        <property name="taskExecutor" ref="asyncTaskExecutor"/>
    </bean>

    <bean id="asyncExecutorTaskExecutorConfiguration"
          class="org.flowable.common.engine.impl.async.AsyncTaskExecutorConfiguration">
        <!-- 作业执行队列的大小 -->
        <property name="queueSize" value="2048"/> <!-- asyncExecutorThreadPoolQueueSize -->
        <!-- 作业执行线程池的最小线程数 -->
        <property name="corePoolSize" value="8"/> <!-- asyncExecutorCorePoolSize -->
        <!-- 作业执行线程池的最大线程数 -->
        <property name="maxPoolSize" value="8"/> <!-- asyncExecutorMaxPoolSize -->
        <!-- 作业执行线程在被销毁前应保持存活的时间 -->
        <property name="keepAlive"> <!-- asyncExecutorThreadKeepAliveTime -->
            <bean class="java.time.Duration" factory-method="ofSeconds">
                <constructor-arg value="5"/>
            </bean>
        </property>
        <!-- 作业执行线程池在关闭时等待的时间 -->
        <property name="awaitTerminationPeriod"> <!-- asyncExecutorSecondsToWaitOnShutdown -->
            <bean class="java.time.Duration" factory-method="ofSeconds">
                <constructor-arg value="60"/>
            </bean>
        </property>
        <!-- 是否允许核心线程超时（用于线程缩减） -->
        <property name="allowCoreThreadTimeout" value="true"/>
        <!-- 线程池线程的命名模式 -->
        <property name="threadPoolNamingPattern" value="flowable-async-job-executor-thread-%d"/>
    </bean>

    <bean id="asyncTaskExecutor" class="org.flowable.common.engine.impl.async.DefaultAsyncTaskExecutor"
          init-method="start">
        <!-- 引用的异步任务执行器配置 -->
        <constructor-arg name="configuration" ref="asyncExecutorTaskExecutorConfiguration"/>
    </bean>


    <!--
        <bean id="asyncHistoryExecutor"
              class="org.flowable.job.service.impl.asyncexecutor.DefaultAsyncHistoryJobExecutor">
            <constructor-arg name="configuration" ref="asyncHistoryJobExecutorConfiguration"/>
        </bean>
    
        <bean id="asyncHistoryTaskExecutor"
              class="org.flowable.common.engine.impl.async.DefaultAsyncTaskExecutor" init-method="start">
            <constructor-arg name="configuration" ref="asyncExecutorTaskExecutorConfiguration"/>
        </bean>
    -->

</beans>

```

第二种：Spring配置风格

这种方式是遵循spring配置风格， 这种配置方式文件名称可以自定义事务管理器

