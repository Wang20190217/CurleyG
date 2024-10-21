---
layout: post
category: curleyg-flowable
title: Flowable基础篇
tagline: by CurleyG
tag: [flowable]
excerpt: Flowable基础篇
lock: need
---
# Flowable基础篇


![image-20231102190512238](../../../.vuepress/public/images/flowable/image-20231102190512238.png)

课程环境说明：

- JDK8
- Flowable6.7.2
- MySQL8



# 一、基础知识科普

## 1.工作流发展

&emsp;&emsp;**BPM**(BusinessProcessManagement)，业务流程管理是一种管理原则，通常也可以代指BPMS(BusinessProcessManagementSuite)，是一个实现整合不同系统和数据的流程管理软件套件.

&emsp;&emsp;**BPMN**(BusinessProcessModelandNotation)是基于流程图的通用可视化标准。该流程图被设计用于创建业务流程操作的图形化模型。业务流程模型就是图形化对象的网状图，包括活动和用于定义这些活动执行顺序的`流程设计器`。BPMN2.0正式版本于2011年1月3日发布，常见的`工作流引擎`如：Activiti、Flowable、jBPM 都基于 BPMN 2.0 标准。

&emsp;&emsp;然后来看看BPM的发展历程：

![image-20231102192753906](../../../.vuepress/public/images/flowable/image-20231102192753906.png)

说明：最新版本的Flowable7.0.0需要的环境：https://tkjohn.github.io/flowable-userguide/#download

- SpringBoot3
- Spring6
- Java 17

## 2.流程设计器

### 2.1 误区说明

&emsp;&emsp;为什么一定要纠结于找IDEA中安装绘制的插件呢？我们需要的是一个能够帮助我们绘制流程图的工具。即使没有流程设计器我们也可以通过代码来定义流程的。

### 2.2 古老的Eclipse

&emsp;&emsp;在Eclipse中我们可以通过安装插件来绘制流程图。然后导出对应的`bpmn`文件。然后就可以做相关的部署操作了。

![image-20231102194104595](../../../.vuepress/public/images/flowable/image-20231102194104595.png)

这个只做简单介绍，了解即可。

### 2.3 官方的FlowableUI

&emsp;&emsp;Flowable官方给我们提供了一个功能完备的基于web应用的流程设计器。可以用于流程相关的操作。具体提供了如下的功能：

- Flowable IDM: 身份管理应用。为所有Flowable UI应用提供单点登录认证功能，并且为拥有IDM管理员权限的用户提供了管理用户、组与权限的功能。
- Flowable Modeler: 让具有建模权限的用户可以创建流程模型、表单、选择表与应用定义。
- Flowable Task: 运行时任务应用。提供了启动流程实例、编辑任务表单、完成任务，以及查询流程实例与任务的功能。
- Flowable Admin: 管理应用。让具有管理员权限的用户可以查询BPMN、DMN、Form及Content引擎，并提供了许多选项用于修改流程实例、任务、作业等。管理应用通过REST API连接至引擎，并与Flowable Task应用及Flowable REST应用一同部署。

&emsp;&emsp;所有其他的应用都需要Flowable IDM提供认证。每个应用的WAR文件可以部署在相同的servlet容器（如Apache Tomcat）中，也可以部署在不同的容器中。由于每个应用使用相同的cookie进行认证，因此应用需要运行在相同的域名下。

### 2.4 BPMN.js自定义

&emsp;&emsp;FlowableUI是官方提供的，针对国内复杂的流程业务需求有时并不能很好的满足企业的工作流的需求。这时我们就可以基于`bpmn.js`来自定义流程设计器，官网地址：https://bpmn.io/toolkit/bpmn-js/walkthrough/

![image-20231102194714321](../../../.vuepress/public/images/flowable/image-20231102194714321.png)

开源的学习资料：https://github.com/LinDaiDai/bpmn-chinese-document/tree/master/LinDaiDai

### 2.5 第三方的设计器

&emsp;&emsp;如何感觉完全基于`bpmn.js`来从零开发一个流程设计器太费时了。也可以找一些开源的别人写好的流程设计器比如：

* https://gitee.com/zhangjinlibra/workflow-engine?_from=gitee_search
* https://gitee.com/MiyueSC/bpmn-process-designer?_from=gitee_search
* 其他的可以自行在GitHub或者gitee上查找



最后我们可以不借助流程设计器来定义流程图。我们可以通过BpmnModle对象来自定义流程图

# 二、流程审批初体验

## 1.FlowableUI安装

&emsp;&emsp;官方提供的FlowableUI是集成在Flowable源码包中的。但是在最新的`7.0.0`中已经移除了。我们需要在`6.7.2`中获取。

![image-20231102202232156](../../../.vuepress/public/images/flowable/image-20231102202232156.png)

https://github.com/flowable/flowable-engine/releases

![image-20231102202409751](../../../.vuepress/public/images/flowable/image-20231102202409751.png)

对应的流程设计器在如下目录中：

![image-20231102203300197](../../../.vuepress/public/images/flowable/image-20231102203300197.png)

然后我们把这个`flowable-ui.war`扔到Tomcat容器中然后启动Tomcat服务即可：`不要部署在有中文的目录下！！！`

![image-20231102203435085](../../../.vuepress/public/images/flowable/image-20231102203435085.png)

![image-20231102203604157](../../../.vuepress/public/images/flowable/image-20231102203604157.png)

访问地址：http://localhost:8080/flowable-ui

![image-20231102203645253](../../../.vuepress/public/images/flowable/image-20231102203645253.png)

登录成功的效果

![image-20231102210928229](../../../.vuepress/public/images/flowable/image-20231102210928229.png)

## 2.用户管理

&emsp;&emsp;流程审批是伴随着`用户`的审批处理的。所以肯定需要用户的参与。我们先通过`身份管理应用程序`来创建两个测试的用户。

![image-20231103004246576](../../../.vuepress/public/images/flowable/image-20231103004246576.png)



点击进入后可以看到在FlowableUI中针对用户这块是从三个维度来管理的

1. 用户
2. 组
3. 权限控制

我们先就创建普通的用户。并分配相关的权限

![image-20231103004453902](../../../.vuepress/public/images/flowable/image-20231103004453902.png)

创建了`zhangsan`并设置了密码为`123`.然后分配相关的权限

![image-20231103004643757](../../../.vuepress/public/images/flowable/image-20231103004643757.png)

![image-20231103004711753](../../../.vuepress/public/images/flowable/image-20231103004711753.png)

![image-20231103004736949](../../../.vuepress/public/images/flowable/image-20231103004736949.png)

然后我们就可以通过`zhangsan`来登录，然后就可以看到相关的功能权限。

![image-20231103004820618](../../../.vuepress/public/images/flowable/image-20231103004820618.png)

同样的操作我们再创建下`lisi`的账号。

![image-20231103005438658](../../../.vuepress/public/images/flowable/image-20231103005438658.png)

## 3.流程定义

&emsp;&emsp;&emsp;有了相关的用户信息。我们就可以来创建流程图了。这块我们需要通过`建模应用程序`来实现。

![image-20231103005603153](../../../.vuepress/public/images/flowable/image-20231103005603153.png)

点击后进入，看到如下的界面：

![image-20231103005634619](../../../.vuepress/public/images/flowable/image-20231103005634619.png)

第一次进入提示我们还没有创建流程模型，我们可以点击右上角的`创建流程`按钮开始创建第一个流程案例。

![image-20231103005855063](../../../.vuepress/public/images/flowable/image-20231103005855063.png)

然后会进入到具体的流程图的绘制界面。

![image-20231103010157647](../../../.vuepress/public/images/flowable/image-20231103010157647.png)

创建第一个请假流程图

![image-20231103010328054](../../../.vuepress/public/images/flowable/image-20231103010328054.png)

然后我们需要分配相关节点的审批人

- 人事审批-- zhangsan 审批
- 经理审批 -- lisi 审批



分配操作

![image-20231103010528808](../../../.vuepress/public/images/flowable/image-20231103010528808.png)

选择`分配给单个用户`

![image-20231103010619697](../../../.vuepress/public/images/flowable/image-20231103010619697.png)

并搜索到`zhangsan`.

![image-20231103010657374](../../../.vuepress/public/images/flowable/image-20231103010657374.png)

相同的操作完成`经理审批`节点的配置，然后就可以保存退出了

![image-20231103010809982](../../../.vuepress/public/images/flowable/image-20231103010809982.png)

![image-20231103010826132](../../../.vuepress/public/images/flowable/image-20231103010826132.png)



![image-20231103010846517](../../../.vuepress/public/images/flowable/image-20231103010846517.png)

到此流程定义完成~



## 4.图标介绍

&emsp;&emsp;BPMN 2.0是业务流程建模符号2.0的缩写。它由Business Process Management Initiative这个非营利协会创建并不断发展。作为一种标识，BPMN 2.0是使用一些**符号**来明确业务流程设计流程图的一整套符号规范，它能增进业务建模时的沟通效率。目前BPMN2.0是最新的版本，它用于在BPM上下文中进行布局和可视化的沟通。接下来我们先来了解在流程设计中常见的 符号。

BPMN2.0的**基本符合**主要包含：

### 4.1 事件图标

&emsp;&emsp;在Flowable中的事件图标启动事件，边界事件,中间事件和结束事件.

![image-20241020181658485](../../../.vuepress/public/images/flowable/image-20241020181658485.png)

### 4.2 活动(任务)图标

&emsp;&emsp;活动是工作或任务的一个通用术语。一个活动可以是一个任务，还可以是一个当前流程的子处理流程； 其次，你还可以为活动指定不同的类型。常见活动如下:

### 4.3 结构图标

&emsp;&emsp;结构图标可以看做是整个流程活动的结构，一个流程中可以包括子流程。常见的结构有：

![image-20241020181724906](../../../.vuepress/public/images/flowable/image-20241020181724906.png)

### 4.4 网关图标

&emsp;&emsp;网关用来处理决策，有几种常用网关需要了解：

![image-20241020181747694](../../../.vuepress/public/images/flowable/image-20241020181747694.png)





## 5.流程部署

&emsp;&emsp;有了流程定义后然后我们可以通过FlowableUI中提供的`应用程序`我们可以来完成部署的相关操作。

![image-20231103011901333](../../../.vuepress/public/images/flowable/image-20231103011901333.png)

进入功能页面后我们可以`创建应用程序`。



![image-20231103011931005](../../../.vuepress/public/images/flowable/image-20231103011931005.png)

然后在弹出的菜单中录入基本信息

![image-20231103012028020](../../../.vuepress/public/images/flowable/image-20231103012028020.png)

![image-20231103012116141](../../../.vuepress/public/images/flowable/image-20231103012116141.png)

点击选择需要绑定的流程定义

![image-20231103012146275](../../../.vuepress/public/images/flowable/image-20231103012146275.png)

![image-20231103012216778](../../../.vuepress/public/images/flowable/image-20231103012216778.png)

![image-20231103012257636](../../../.vuepress/public/images/flowable/image-20231103012257636.png)

点击进入后我们可以点击右上角的`发布`操作来完成部署的行为：

![image-20231103012353734](../../../.vuepress/public/images/flowable/image-20231103012353734.png)

![image-20231103012406828](../../../.vuepress/public/images/flowable/image-20231103012406828.png)

部署出现的时候出现了异常信息：

![image-20231103012744584](../../../.vuepress/public/images/flowable/image-20231103012744584.png)

同时在看控制台出现了如下的问题

![image-20231103015431509](../../../.vuepress/public/images/flowable/image-20231103015431509.png)

原因是部署的项目不能放在有中文的路径下。所以我们调整下`Tomcat`的位置.然后再启动即可，然后在发布就提示发布成功了

![image-20231103020036915](../../../.vuepress/public/images/flowable/image-20231103020036915.png)





## 6.流程审批

&emsp;&emsp;流程部署完毕后我们就可以启动一个流程实例了。然后就可以走流程审批的操作。启动流程实例我们通过`任务应用程序`来处理。

![image-20231103020610521](../../../.vuepress/public/images/flowable/image-20231103020610521.png)

点击启动新的流程

![image-20231103020701529](../../../.vuepress/public/images/flowable/image-20231103020701529.png)

![image-20231103020739370](../../../.vuepress/public/images/flowable/image-20231103020739370.png)

启动流程后进入到`人事审批`的阶段

![image-20231103020822180](../../../.vuepress/public/images/flowable/image-20231103020822180.png)

点击`显示流程`可以看到当前的状态

![image-20231103020847143](../../../.vuepress/public/images/flowable/image-20231103020847143.png)

然后我们可以切换到`zhangsan`账号登录后来审批操作

![image-20231103020945527](../../../.vuepress/public/images/flowable/image-20231103020945527.png)

点击完成后，点击`流程`可以看到对应的流程情况

![image-20231103021032057](../../../.vuepress/public/images/flowable/image-20231103021032057.png)

![image-20231103021044215](../../../.vuepress/public/images/flowable/image-20231103021044215.png)



然后切换到`lisi`登录来继续审批

![image-20231103021122853](../../../.vuepress/public/images/flowable/image-20231103021122853.png)

点击`完成`后，审批操作完成。到此FlowableUI的流程审批操作就演示完成。

# 三、流程审批操作

## 1.案例环境

&emsp;&emsp;创建一个基本的SpringBoot项目。相关的版本如下：

- SpringBoot2.6.13
- JDK8
- Flowable6.7.2

对应的依赖如下：

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--Flowable的核心依赖-->
        <dependency>
            <groupId>org.flowable</groupId>
            <artifactId>flowable-spring-boot-starter</artifactId>
            <version>6.7.2</version>
        </dependency>
        <!-- MySQL的依赖 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.14</version>
        </dependency>
        <!-- 单元测试 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
        <!-- 日志相关 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.21</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.21</version>
        </dependency>
```



## 2.流程部署

&emsp;&emsp;流程部署我们可以在Spring的环境下部署。也可以在非Spring的环境下部署。下面先展示非Spring环境下的部署操作：

```java
    /**
     * 不通过Spring容器。我们单独的构建ProcessEngine对象来实现部署的操作
     */
    @Test
    void contextLoads() {
        // 1.流程引擎的配置
        ProcessEngineConfiguration cfg = new StandaloneProcessEngineConfiguration()
                .setJdbcUrl("jdbc:mysql://localhost:3306/flowable-learn?serverTimezone=UTC&nullCatalogMeansCurrent=true")
                .setJdbcUsername("root")
                .setJdbcPassword("123456")
                .setJdbcDriver("com.mysql.cj.jdbc.Driver")
                .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);
        // 2.构建流程引擎对象
        ProcessEngine processEngine = cfg.buildProcessEngine();
        System.out.println(processEngine);
        Deployment deploy = processEngine.getRepositoryService().createDeployment()
                .addClasspathResource("process/HolidayDemo1.bpmn20.xml")
                .name("第一个流程案例")
                .deploy();
        System.out.println(deploy.getId());
    }
```

方法执行成功后会在这三张表中记录相关的部署信息

- act_ge_bytearray:记录流程定义的资源信息。xml和流程图的图片信息 （是否生成图片取决于xml中是否配置节点配置的坐标信息）
- act_re_deployment:流程部署表，记录这次的部署行为（一次可能部署多个流程）
- act_re_procdef:流程定义表，记录这次部署动作对应的流程定义信息



**databaseSchemaUpdate**：用于设置流程引擎启动关闭时使用的数据库表结构控制策略

- `false` (默认): 当引擎启动时，检查数据库表结构的版本是否匹配库文件版本。版本不匹配时抛出异常。
- `true`: 构建引擎时，检查并在需要时更新表结构。表结构不存在则会创建。
- `create-drop`: 引擎创建时创建表结构，并在引擎关闭时删除表结构。



`注意`:要注意区分流程部署和流程定义的关系

- 一次部署操作可以部署多个流程定义

```java
// 2.构建流程引擎对象
ProcessEngine processEngine = cfg.buildProcessEngine();
System.out.println(processEngine);
Deployment deploy = processEngine.getRepositoryService().createDeployment()
        .addClasspathResource("process/HolidayDemo1.bpmn20.xml") // 部署一个流程
        .addClasspathResource("process/消息中间事件.bpmn20.xml")   // 部署第二个流程
        .name("第一个流程案例")
        .deploy();
```

![image-20231103193919523](../../../.vuepress/public/images/flowable/image-20231103193919523.png)



&emsp;&emsp;当然上面的操作我们是自己定义`ProcessEngine`和`ProcessEngineConfiguration`来实现流程引擎对象的获取操作，我们完全可以把这些初始化的操作交给Spring容器来管理。接下来我们看看在SpringBoot项目中怎么简化处理。首先依赖我们上面已经添加了。然后需要在`application.yml`中配置如下信息：

```yml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/flowable-learn?serverTimezone=UTC&nullCatalogMeansCurrent=true
    username: root
    password: 123456
    hikari:
      minimum-idle: 5
      idle-timeout: 600000
      maximum-pool-size: 10
      auto-commit: true
      pool-name: MyHikariCP
      max-lifetime: 1800000
      connection-timeout: 30000
      connection-test-query: SELECT 1
flowable:
  async-executor-activate: true #关闭定时任务JOB
  #  将databaseSchemaUpdate设置为true。当Flowable发现库与数据库表结构不一致时，会自动将数据库表结构升级至新版本。
  database-schema-update: true
server:
  port: 8082
logging:
  level:
    org:
      flowable: debug
```

注意在url中需要添加`nullCatalogMeansCurrent=true`属性。不然启动服务的时候会出现如下问题

![image-20231103201208204](../../../.vuepress/public/images/flowable/image-20231103201208204.png)



具体部署流程的代码,就可以直接从Spring容器中获取`ProcessEngine`对象了。大大简化了相关的代码

```java
    @Autowired
    private ProcessEngine processEngine;

    /**
     * 流程部署
     */
    @Test
    void deployFlow(){
        Deployment deploy = processEngine.getRepositoryService().createDeployment()
                .addClasspathResource("process/HolidayDemo1.bpmn20.xml") // 部署一个流程
                .name("第一个流程案例")
                .deploy();
        System.out.println(deploy.getId());
    }
```



## 3.启动流程实例

&emsp;&emsp;流程部署完成后我们就可以发起一个流程实例。具体的操作如下：

```java
    /**
     * 发起流程
     */
    @Test
    void startProcess(){
        // 发起流程需要通过RuntimeService来实现
        RuntimeService runtimeService = processEngine.getRuntimeService();
        // act_re_procdef 表中的id
        String processId = "HolidayDemo1:1:89fad01e-7a42-11ee-b574-c03c59ad2248";
        // 根据流程定义Id启动 返回的是当前启动的流程实例 ProcessInstance
        //ProcessInstance processInstance = runtimeService.startProcessInstanceById(processId);
        //System.out.println("processInstance.getId() = " + processInstance.getId());
        String processKey = "HolidayDemo1";
        runtimeService.startProcessInstanceByKey(processKey);
    }
```

启动流程需要通过`RuntimeService`来实现。同时在启动流程的时候有两个方法可以调用：

- startProcessInstanceById: 对应于act_re_procdef 表中的id
- startProcessInstanceByKey: 对应于act_re_procdef 表中的key

启动流程后根据我们的定义。流程会走到`人事审批`。

![image-20231103212958751](../../../.vuepress/public/images/flowable/image-20231103212958751.png)

这时我们可以在`act_ru_task`表中看到对应的记录。`act_ru_task`记录的都是当前待办的记录信息

![image-20231103213051698](../../../.vuepress/public/images/flowable/image-20231103213051698.png)

还有一个要注意的：每启动一个流程实例那么就会在`act_hi_procinst`表中维护一条记录。然后在`act_ru_execution`会记录流程的分支

流程定义和流程实例的关系：

- 流程定义：Java中的类
- 流程实例：Java中的对象





## 4.流程审批

&emsp;&emsp;流程启动后会进入到相关的审批节点。上面的案例中就会进入到`人事审批`节点。审批人是`zhangsan`,也就是`zhangsan`有了一条待办信息。这时候我们可以查询下`zhangsan`有哪些待办任务。

```java
    /**
     * 待办任务查询
     */
    @Test
    void findTask(){
        // 任务查询这块我们可以通过 TaskService 来实现
        TaskService taskService = processEngine.getTaskService();
        // 查询的其实就是 act_ru_task 中的记录
        List<Task> list = taskService.createTaskQuery()
                .taskAssignee("zhangsan") // 根据审批人来查询
                .list();// 返回多条记录
        for (Task task : list) {
            System.out.println(task.getId());
        }
    }
```

查询出了对应的两条记录

![image-20231103214442516](../../../.vuepress/public/images/flowable/image-20231103214442516.png)

找到了需要审批的任务。我们就可以根据 `taskId`来完成审批的操作

```java
    /**
     * 完成任务的审批
     */
    @Test
    void completeTask(){
        TaskService taskService = processEngine.getTaskService();
        // 需要审批的任务 Id
        String taskId = "926b41f8-7a4c-11ee-97fd-c03c59ad2248";
        taskService.complete(taskId); // 通过complete方法完成审批
    }
```

审批通过后就会进入到`经理审批`的节点。

![image-20231103214703397](../../../.vuepress/public/images/flowable/image-20231103214703397.png)

相同的操作可以完成`经理审批`的待办处理。然后就完成了第一个流程的审批处理操作了。



## 5.各个Service服务

&emsp;&emsp;Service是工作流引擎提供用于进行工作流部署、执行、管理的服务接口，我们使用这些接口可以就是操作服务对应的数据表



![image-20220319223019186](../../../.vuepress/public/images/flowable/image-20220319223019186.png)



### 5.1 Service创建方式

通过ProcessEngine创建Service

方式如下：

 ```java
RuntimeService runtimeService = processEngine.getRuntimeService();
RepositoryService repositoryService = processEngine.getRepositoryService();
TaskService taskService = processEngine.getTaskService();
// ...
 ```

### 5.2 Service总览

| service名称       | service作用              |
| ----------------- | ------------------------ |
| RepositoryService | Flowable的资源管理类     |
| RuntimeService    | Flowable的流程运行管理类 |
| TaskService       | Flowable的任务管理类     |
| HistoryService    | Flowable的历史管理类     |
| ManagerService    | Flowable的引擎管理类     |

简单介绍：

**RepositoryService**

&emsp;&emsp;是activiti的资源管理类，提供了管理和控制流程发布包和流程定义的操作。使用工作流建模工具设计的业务流程图需要使用此service将流程定义文件的内容部署到计算机。

除了部署流程定义以外还可以：查询引擎中的发布包和流程定义。

&emsp;&emsp;暂停或激活发布包，对应全部和特定流程定义。 暂停意味着它们不能再执行任何操作了，激活是对应的反向操作。获得多种资源，像是包含在发布包里的文件， 或引擎自动生成的流程图。

&emsp;&emsp;获得流程定义的pojo版本， 可以用来通过java解析流程，而不必通过xml。

**RuntimeService**

&emsp;&emsp;Activiti的流程运行管理类。可以从这个服务类中获取很多关于流程执行相关的信息

**TaskService**

&emsp;&emsp;Activiti的任务管理类。可以从这个类中获取任务的信息。

**HistoryService**

&emsp;&emsp;Flowable的历史管理类，可以查询历史信息，执行流程时，引擎会保存很多数据（根据配置），比如流程实例启动时间，任务的参与者， 完成任务的时间，每个流程实例的执行路径，等等。 这个服务主要通过查询功能来获得这些数据。

**ManagementService**

&emsp;&emsp;Activiti的引擎管理类，提供了对Flowable 流程引擎的管理和维护功能，这些功能不在工作流驱动的应用程序中使用，主要用于 Flowable 系统的日常维护。



# 四、相关表结构介绍

&emsp;&emsp;接下来我们结合上面演示的第一个审批流程来详细的介绍下Flowable中的相关表结构的含义和作用。

## 1.表结构介绍

支持的数据库有：

| Activiti数据库类型 | 示例JDBC URL                                                 | 备注                                     |
| ------------------ | ------------------------------------------------------------ | ---------------------------------------- |
| h2                 | jdbc:h2:tcp://localhost/activiti                             | 默认配置的数据库                         |
| mysql              | jdbc:mysql://localhost:3306/activiti?autoReconnect=true      | 已使用mysql-connector-java数据库驱动测试 |
| oracle             | jdbc:oracle:thin:@localhost:1521:xe                          |                                          |
| postgres           | jdbc:postgresql://localhost:5432/activiti                    |                                          |
| db2                | jdbc:db2://localhost:50000/activiti                          |                                          |
| mssql              | jdbc:sqlserver://localhost:1433;databaseName=activiti (jdbc.driver=com.microsoft.sqlserver.jdbc.SQLServerDriver) *OR* jdbc:jtds:sqlserver://localhost:1433/activiti (jdbc.driver=net.sourceforge.jtds.jdbc.Driver) | 已使用Microsoft JDBC Driver 4.0 (sqljdb  |

&emsp;&emsp;工作流程的相关操作都是操作存储在对应的表结构中，为了能更好的弄清楚Flowable的实现原理和细节，我们有必要先弄清楚Flowable的相关表结构及其作用。在Flowable中的表结构在初始化的时候会创建五类表结构，具体如下：

* **ACT_RE** ：'RE'表示 repository。 这个前缀的表包含了流程定义和流程静态资源 （图片，规则，等等）。
* **ACT_RU**：'RU'表示 runtime。 这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。 Flowable只在流程实例执行过程中保存这些数据， 在流程结束时就会删除这些记录。 这样运行时表可以一直很小速度很快。
* **ACT_HI**：'HI'表示 history。 这些表包含历史数据，比如历史流程实例， 变量，任务等等。
* **ACT_GE**： GE 表示 general。 通用数据， 用于不同场景下
* **ACT_ID:**   ’ID’表示identity(组织机构)。这些表包含标识的信息，如用户，用户组，等等。



具体的表结构的含义:

| **表分类**   | **表名**              | **解释**                                           |
| ------------ | --------------------- | -------------------------------------------------- |
| 一般数据     |                       |                                                    |
|              | [ACT_GE_BYTEARRAY]    | 通用的流程定义和流程资源                           |
|              | [ACT_GE_PROPERTY]     | 系统相关属性                                       |
| 流程历史记录 |                       |                                                    |
|              | [ACT_HI_ACTINST]      | 历史的流程实例                                     |
|              | [ACT_HI_ATTACHMENT]   | 历史的流程附件                                     |
|              | [ACT_HI_COMMENT]      | 历史的说明性信息                                   |
|              | [ACT_HI_DETAIL]       | 历史的流程运行中的细节信息                         |
|              | [ACT_HI_IDENTITYLINK] | 历史的流程运行过程中用户关系                       |
|              | [ACT_HI_PROCINST]     | 历史的流程实例                                     |
|              | [ACT_HI_TASKINST]     | 历史的任务实例                                     |
|              | [ACT_HI_VARINST]      | 历史的流程运行中的变量信息                         |
| 流程定义表   |                       |                                                    |
|              | [ACT_RE_DEPLOYMENT]   | 部署单元信息                                       |
|              | [ACT_RE_MODEL]        | 模型信息                                           |
|              | [ACT_RE_PROCDEF]      | 已部署的流程定义                                   |
| 运行实例表   |                       |                                                    |
|              | [ACT_RU_EVENT_SUBSCR] | 运行时事件                                         |
|              | [ACT_RU_EXECUTION]    | 运行时流程执行实例                                 |
|              | [ACT_RU_IDENTITYLINK] | 运行时用户关系信息，存储任务节点与参与者的相关信息 |
|              | [ACT_RU_JOB]          | 运行时作业                                         |
|              | [ACT_RU_TASK]         | 运行时任务                                         |
|              | [ACT_RU_VARIABLE]     | 运行时变量表                                       |
|              | [ACT_RU_ACTINST]      | 正在执行的活动实例数据的表                         |
| 用户用户组表 |                       |                                                    |
|              | [ACT_ID_BYTEARRAY]    | 二进制数据表                                       |
|              | [ACT_ID_GROUP]        | 用户组信息表                                       |
|              | [ACT_ID_INFO]         | 用户信息详情表                                     |
|              | [ACT_ID_MEMBERSHIP]   | 人与组关系表                                       |
|              | [ACT_ID_PRIV]         | 权限表                                             |
|              | [ACT_ID_PRIV_MAPPING] | 用户或组权限关系表                                 |
|              | [ACT_ID_PROPERTY]     | 属性表                                             |
|              | [ACT_ID_TOKEN]        | 记录用户的token信息                                |
|              | [ACT_ID_USER]         | 用户表                                             |

## 2.部署流程

 1.ACT_GE_BYTEARRAY 资源表

| 字段           | 类型           | 主键 | 说明                       | 备注                                                         |
| -------------- | -------------- | ---- | -------------------------- | ------------------------------------------------------------ |
| ID_            | NVARCHAR2(64)  | Y    | 主键                       |                                                              |
| REV_           | INTEGER        | N    | 数据版本                   | Activiti 有可能会被频繁修改数据库表，加入字段，用来表示该数据被操作的次数 |
| NAME_          | NVARCHAR2(255) | N    | 资源名称                   |                                                              |
| DEPLOYMENT_ID_ | NVARCHAR2(64)  | N    | 部署序号                   | 部署序号，一次部署可以部署多个资源，该字段与部署表 ACT_RE_DEPLOYMENT 的主键关联 |
| BYTES_         | BLOB           | N    | 资源内容                   |                                                              |
| GENERATED_     | NUMBER(1)      | N    | 是否是由流程自动产生的资源 | 0 表示 false，1 表示 true                                    |

#### 2.ACT_GE_PROPERTY 属性表

| 字段   | 类型           | 主键 | 说明       | 备注 |
| ------ | -------------- | ---- | ---------- | ---- |
| NAME_  | NVARCHAR2(64)  | Y    | 属性名称   |      |
| VALUE_ | NVARCHAR2(300) | N    | 属性值     |      |
| REV_   | INTEGER        | N    | 数据版本号 |      |

**数据如下**

| 属性名                               | 值              | 描述                             |
| ------------------------------------ | --------------- | -------------------------------- |
| batch.schema.version                 | 6.7.2.0         | 批处理模式版本                   |
| cfg.execution-related-entities-count | true            | 是否启用与执行相关的实体计数功能 |
| cfg.task-related-entities-count      | true            | 是否启用与任务相关的实体计数功能 |
| common.schema.version                | 6.7.2.0         | 通用表的架构版本                 |
| entitylink.schema.version            | 6.7.2.0         | 实体链接表的架构版本             |
| eventsubscription.schema.version     | 6.7.2.0         | 事件订阅表的架构版本             |
| identitylink.schema.version          | 6.7.2.0         | 身份链接表的架构版本             |
| job.schema.version                   | 6.7.2.0         | 作业表的架构版本                 |
| next.dbid                            | 5001            | 下一个生成的数据库 ID 值         |
| schema.history                       | create(6.7.2.0) | 数据库架构的历史变化记录         |
| schema.version                       | 6.7.2.0         | 整体数据库架构的当前版本         |
| task.schema.version                  | 6.7.2.0         | 任务相关表的架构版本             |
| variable.schema.version              | 6.7.2.0         | 变量表的架构版本                 |

#### 3. ACT_RE_DEPLOYMENT 部署数据表

| 字段                  | 类型           | 主键 | 说明                                       | 备注                                               |
| --------------------- | -------------- | ---- | ------------------------------------------ | -------------------------------------------------- |
| ID_                   | NVARCHAR2(64)  | Y    | 部署序号                                   | 部署的唯一标识符                                   |
| NAME_                 | NVARCHAR2(255) | N    | 部署的名称，可以为空                       |                                                    |
| CATEGORY_             | NVARCHAR2(255) | N    | 部署的类别，通常用于分类管理不同类型的部署 | 流程定义的 Namespace 就是类别                      |
| KEY_                  | NVARCHAR2(255) | N    | 流程定义 ID                                | 部署的唯一业务键，通常用于唯一标识该部署的业务含义 |
| TENANT_ID_            | NVARCHAR2(255) | N    | 租户ID                                     | 租户标识符，用于支持多租户的场景                   |
| DEPLOY_TIME_          | TIMESTAMP(6)   | N    | 部署时间                                   | 部署的时间戳，记录部署发生的时间                   |
| DERIVED_FROM_         | NVARCHAR2(64)  | N    |                                            | 指示该部署是否从其他部署派生的来源部署标识符       |
| DERIVED_FROM_ROOT_    | NVARCHAR2(64)  | N    |                                            | 派生部署的根标识符，用于追踪部署的根来源           |
| PARENT_DEPLOYMENT_ID_ | NVARCHAR2(255) | N    |                                            | 父部署的标识符，表示此部署是哪个部署的子部署       |
| ENGINE_VERSION_       | NVARCHAR2(255) | N    | 引擎版本                                   | 表示该部署使用的引擎版本                           |

#### 4. ACT_RE_PROCDEF 流程定义表

| 字段                    | 类型            | 主键 | 说明                           | 备注                                                         |
| ----------------------- | --------------- | ---- | ------------------------------ | ------------------------------------------------------------ |
| ID_                     | NVARCHAR2(64)   | Y    | 主键                           |                                                              |
| REV_                    | INTEGER         | N    | 数据版本号                     |                                                              |
| CATEGORY_               | NVARCHAR2(255)  | N    | 流程定义分类                   | 读取 xml 文件中程的 targetNamespace 值                       |
| NAME_                   | NVARCHAR2(255)  | N    | 流程定义的名称                 | 读取流程文件中 process 元素的 name 属性                      |
| KEY_                    | NVARCHAR2(255)  | N    | 流程定义 key                   | 读取流程文件中 process 元素的 id 属性                        |
| VERSION_                | INTEGER         | N    | 版本                           |                                                              |
| DEPLOYMENT_ID_          | NVARCHAR2(64)   | N    | 部署 ID                        | 流程定义对应的部署数据 ID                                    |
| RESOURCE_NAME_          | NVARCHAR2(2000) | N    | bpmn 文件名称                  | 一般为流程文件的相对路径                                     |
| DGRM_RESOURCE_NAME_     | VARCHAR2(4000)  | N    | 流程定义对应的流程图资源名称   |                                                              |
| DESCRIPTION_            | NVARCHAR2(2000) | N    | 说明                           | 流程说明                                                     |
| HAS_START_FORM_KEY_     | NUMBER(1)       | N    | 是否存在开始节点 formKey       | start 节点是否存在 formKey 0 否 1 是                         |
| HAS_GRAPHICAL_NOTATION_ | NUMBER(1)       | N    | 表示流程定义是否包含图形化表示 | `true`：表示该流程定义包含图形化表示，即可以在 Flowable 的模型设计器或其他可视化工具中看到该流程的图形化表示（如流程图、泳道图等）。`false`：表示该流程定义没有图形化表示，可能仅包含流程的定义而没有与其对应的图形信息。1 激活、2 中止 |
| SUSPENSION_STATE_       | INTEGER         | N    | 流程定义状态                   | **`1` - 激活状态 (Active)**: **`2` - 挂起状态 (Suspended)**: |
| TENANT_ID_              | NVARCHAR2(255)  | N    | 租户ID                         | 租户标识符，用于支持多租户的场景                             |
| ENGINE_VERSION_         | NVARCHAR2(255)  | N    |                                | 引擎版本                                                     |
| DERIVED_FROM_           | NVARCHAR2(64)   | N    |                                | 指示该部署是否从其他部署派生的来源部署标识符                 |
| DERIVED_FROM_ROOT_      | NVARCHAR2(64)   | N    |                                | 派生部署的根标识符，用于追踪部署的根来源                     |
| DERIVED_VERSION_        | INTEGER         | N    |                                | 表示当前流程定义在派生链中的版本号                           |

**注意**

业务流程定义数据表。此表和ACT_RE_DEPLOYMENT是多对一的关系，即，一个部署的bar包里可能包含多个流程定义文件，每个流程定义文件都会有一条记录在ACT_REPROCDEF表内，每个流程定义的数据，都会对于ACT_GE_BYTEARRAY表内的一个资源文件和PNG图片文件。和ACT_GE_BYTEARRAY的关联是通过程序用ACT_GE_BYTEARRAY.NAME与ACT_RE_PROCDEF.NAME_完成的



## 3.挂起和激活

&emsp;&emsp;部署的流程默认的状态为激活，如果我们暂时不想使用该定义的流程，那么可以挂起该流程。当然该流程定义下边所有的流程实例全部暂停。

流程定义为挂起状态，该流程定义将不允许启动新的流程实例，同时该流程定义下的所有的流程实例都将全部挂起暂停执行。

```java
/**
     * 挂起流程
     */
    @Test
    public void test05(){
        RepositoryService repositoryService = processEngine.getRepositoryService();
        ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery()
                .processDefinitionId("holiday:1:4")
                .singleResult();
        // 获取流程定义的状态
        boolean suspended = processDefinition.isSuspended();
        System.out.println("suspended = " + suspended);
        if(suspended){
            // 表示被挂起
            System.out.println("激活流程定义");
            repositoryService.activateProcessDefinitionById("holiday:1:4",true,null);
        }else{
            // 表示激活状态
            System.out.println("挂起流程");
            repositoryService.suspendProcessDefinitionById("holiday:1:4",true,null);
        }
    }


    /**
     * 挂起流程实例
     */
    @Test
    void suspendInstance(){
        // 挂起流程实例
        runtimeService.suspendProcessInstanceById("a7ae5680-7ba3-11ee-809a-c03c59ad2248");
        // 激活流程实例
        //runtimeService.activateProcessInstanceById("a7ae5680-7ba3-11ee-809a-c03c59ad2248");
    }
```

具体的实现其实就是更新了流程定义表中的字段

![image-20220321210010518](../../../.vuepress/public/images/flowable/image-20220321210010518.png)

而且通过REV_字段来控制数据安全，也是一种乐观锁的体现了，如果要启动一个已经挂起的流程就会出现如下的错误

![image-20220321211858122](../../../.vuepress/public/images/flowable/image-20220321211858122.png)



## 4.启动流程

&emsp;&emsp;当我们启动了一个流程实例后，会在ACT_RU_*对应的表结构中操作,运行时实例涉及的表结构共10张：

* ACT_RU_DEADLETTER_JOB  正在运行的任务表
* ACT_RU_EVENT_SUBSCR 运行时事件
* ACT_RU_EXECUTION 运行时流程执行实例
* ACT_RU_HISTORY_JOB  历史作业表
* ACT_RU_IDENTITYLINK 运行时用户关系信息
* ACT_RU_JOB 运行时作业表
* ACT_RU_SUSPENDED_JOB 暂停作业表
* ACT_RU_TASK  运行时任务表
* ACT_RU_TIMER_JOB 定时作业表
* ACT_RU_VARIABLE  运行时变量表



&emsp;&emsp;启动一个流程实例的时候涉及到的表有

* ACT_RU_EXECUTION 运行时流程实例 (执行流) 表，存储流程实例和指向流程实例当前状态的指针（称为执行）
* ACT_RU_IDENTITYLINK 运行时用户关系信息
* ACT_RU_TASK  运行时任务表
* ACT_RU_VARIABLE  运行时变量表



> 注意：上面由于但是运行时的表，在审批结束数据会清空



#### 1.ACT_RU_EXECUTION 运行时流程实例 (执行流) 表

| 字段                  | 类型           | 主键 | 说明               | 备注 |
| --------------------- | -------------- | ---- | ------------------ | ---- |
| ID_                   | NVARCHAR2(64)  | Y    | 主键               |      |
| REV_                  | INTEGER        | N    | 数据版本           |      |
| PROC_INST_ID_         | NVARCHAR2(64)  | N    | 流程实例 ID        |      |
| BUSINESS_KEY_         | NVARCHAR2(255) | N    | 业务主键 ID        |      |
| PARENT_ID_            | NVARCHAR2(64)  | N    | 父执行流的 ID      |      |
| PROC_DEF_ID_          | NVARCHAR2(64)  | N    | 流程定义的数据 ID  |      |
| SUPER_EXEC_           | NVARCHAR2(64)  | N    |                    |      |
| ROOT_PROC_INST_ID_    | NVARCHAR2(64)  | N    |                    |      |
| ACT_ID_               | NVARCHAR2(255) | N    | 节点实例 ID        |      |
| IS_ACTIVE_            | NUMBER(1)      | N    | 是否存活           |      |
| IS_CONCURRENT_        | NUMBER(1)      | N    | 执行流是否正在并行 |      |
| IS_SCOPE_             | NUMBER(1)      | N    |                    |      |
| IS_EVENT_SCOPE_       | NUMBER(1)      | N    |                    |      |
| IS_MI_ROOT_           | NUMBER(1)      | N    |                    |      |
| SUSPENSION_STATE_     | INTEGER        | N    | 流程终端状态       |      |
| CACHED_ENT_STATE_     | INTEGER        | N    |                    |      |
| TENANT_ID_            | NVARCHAR2(255) | N    |                    |      |
| NAME_                 | NVARCHAR2(255) | N    |                    |      |
| START_TIME_           | TIMESTAMP(6)   | N    | 开始时间           |      |
| START_USER_ID_        | NVARCHAR2(255) | N    |                    |      |
| LOCK_TIME_            | TIMESTAMP(6)   | N    |                    |      |
| IS_COUNT_ENABLED_     | NUMBER(1)      | N    |                    |      |
| EVT_SUBSCR_COUNT_     | INTEGER        | N    |                    |      |
| TASK_COUNT_           | INTEGER        | N    |                    |      |
| JOB_COUNT_            | INTEGER        | N    |                    |      |
| TIMER_JOB_COUNT_      | INTEGER        | N    |                    |      |
| SUSP_JOB_COUNT_       | INTEGER        | N    |                    |      |
| DEADLETTER_JOB_COUNT_ | INTEGER        | N    |                    |      |
| VAR_COUNT_            | INTEGER        | N    |                    |      |
| ID_LINK_COUNT_        | INTEGER        | N    |                    |      |

创建流程实例后对应的表结构的数据

![image-20220322133108405](../../../.vuepress/public/images/flowable/image-20220322133108405.png)

![image-20220322133219534](../../../.vuepress/public/images/flowable/image-20220322133219534.png)



#### 2.ACT_RU_TASK  运行时任务表

| 字段              | 类型            | 主键 | 说明                 | 备注                  |
| ----------------- | --------------- | ---- | -------------------- | --------------------- |
| ID_               | NVARCHAR2(64)   | Y    | 主键                 |                       |
| REV_              | INTEGER         | N    | 数据版本             |                       |
| EXECUTION_ID_     | NVARCHAR2(64)   | N    | 任务所在的执行流 ID  |                       |
| PROC_INST_ID_     | NVARCHAR2(64)   | N    | 流程实例 ID          |                       |
| PROC_DEF_ID_      | NVARCHAR2(64)   | N    | 流程定义数据 ID      |                       |
| NAME_             | NVARCHAR2(255)  | N    | 任务名称             |                       |
| PARENT_TASK_ID_   | NVARCHAR2(64)   | N    | 父任务 ID            |                       |
| DESCRIPTION_      | NVARCHAR2(2000) | N    | 说明                 |                       |
| TASK_DEF_KEY_     | NVARCHAR2(255)  | N    | 任务定义的 ID 值     |                       |
| OWNER_            | NVARCHAR2(255)  | N    | 任务拥有人           |                       |
| ASSIGNEE_         | NVARCHAR2(255)  | N    | 被指派执行该任务的人 |                       |
| DELEGATION_       | NVARCHAR2(64)   | N    |                      |                       |
| PRIORITY_         | INTEGER         | N    |                      |                       |
| CREATE_TIME_      | TIMESTAMP(6)    | N    | 创建时间             |                       |
| DUE_DATE_         | TIMESTAMP(6)    | N    | 耗时                 |                       |
|                   |                 |      |                      |                       |
| CATEGORY_         | NVARCHAR2(255)  | N    |                      |                       |
| SUSPENSION_STATE_ | INTEGER         | N    | 是否挂起             | 1 代表激活 2 代表挂起 |
| TENANT_ID_        | NVARCHAR2(255)  | N    |                      |                       |
| FORM_KEY_         | NVARCHAR2(255)  | N    |                      |                       |

创建流程实例后对应的表结构的数据

![image-20220322133307195](../../../.vuepress/public/images/flowable/image-20220322133307195.png)

![image-20220322133335326](../../../.vuepress/public/images/flowable/image-20220322133335326.png)



#### 3.ACT_RU_VARIABLE  运行时变量表

| 字段           | 类型            | 主键 | 说明名称                                                  | 备注备注                                                     |
| -------------- | --------------- | ---- | --------------------------------------------------------- | ------------------------------------------------------------ |
| ID_            | NVARCHAR2(64)   | Y    | 主键主键                                                  |                                                              |
| REV_REV_       | INTEGER         | N    | 数据版本版本号                                            |                                                              |
| TYPE_TYPE_     | NVARCHAR2(255)  | N    | 参数类型                                                  | 可以是基本的类型，也可以用户自行扩展可以是基本的类型，也可以用户自行扩展 |
| NAME_          | NVARCHAR2(255)  | N    | 参数名称                                                  |                                                              |
| EXECUTION_ID_  | NVARCHAR2(64)   | N    | 参数执行ID                                                |                                                              |
| PROC_INST_ID_  | NVARCHAR2(64)   | N    | 流程实例ID                                                |                                                              |
| TASK_ID_       | NVARCHAR2(64)   | N    | 任务ID                                                    |                                                              |
| BYTEARRAY_ID_  | NVARCHAR2(64)   | N    | 资源ID                                                    |                                                              |
| DOUBLE_DOUBLE_ | NUMBER(*,10)    | N    | 参数为 double，则保存在该字段中                           |                                                              |
| LONG_          | NUMBER(19)      | N    | 参数为 long，则保存在该字段中参数为long，则保存在该字段中 |                                                              |
| TEXT_          | NVARCHAR2(2000) | N    | 用户保存文本类型的参数值                                  |                                                              |
| TEXT2_         | NVARCHAR2(2000) | N    | 用户保存文本类型的参数值                                  |                                                              |

创建流程实例后对应的表结构的数据

![image-20220322133406398](../../../.vuepress/public/images/flowable/image-20220322133406398.png)

![image-20220322133439827](../../../.vuepress/public/images/flowable/image-20220322133439827.png)



#### 4.ACT_RU_IDENTITYLINK 运行时用户关系信息

| 字段          | 类型           | 主键 | 说明         | 备注                                                       |
| ------------- | -------------- | ---- | ------------ | ---------------------------------------------------------- |
| ID_           | NVARCHAR2(64)  | Y    | 主键         |                                                            |
| REV_          | INTEGER        | N    | 数据版本     |                                                            |
| GROUP_ID_     | NVARCHAR2(255) | N    | 用户组 ID    |                                                            |
| TYPE_         | NVARCHAR2(255) | N    | 关系数据类型 | assignee 支配人 (组)、candidate 候选人 (组)、 owner 拥有人 |
| USER_ID_      | NVARCHAR2(255) | N    | 用户 ID      |                                                            |
| TASK_ID_      | NVARCHAR2(64)  | N    | 任务 ID      |                                                            |
| PROC_INST_ID_ | NVARCHAR2(64)  | N    | 流程定义 ID  |                                                            |
| PROC_DEF_ID_  | NVARCHAR2(64)  | N    | 属性 ID      |                                                            |

创建流程实例后对应的表结构的数据:

![image-20220322133501720](../../../.vuepress/public/images/flowable/image-20220322133501720.png)

## 5.流程审批

&emsp;&emsp;上面的流程已经流转到了zhangsan这个用户这里，然后可以开始审批了

```java
// 获取流程引擎对象
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        TaskService taskService = processEngine.getTaskService();
        Task task = taskService.createTaskQuery()
                .processDefinitionId("holiday:1:4")
                .taskAssignee("zhangsan")
                .singleResult();
        // 添加流程变量
        Map<String,Object> variables = new HashMap<>();
        variables.put("approved",false); // 拒绝请假
        // 完成任务
        taskService.complete(task.getId(),variables);
```

&emsp;&emsp;在正常处理流程中涉及到的表结构

* ACT_RU_EXECUTION 运行时流程执行实例
* ACT_RU_IDENTITYLINK 运行时用户关系信息
* ACT_RU_TASK  运行时任务表
* ACT_RU_VARIABLE  运行时变量表

ACT_RU_TASK  运行时任务表 :会新生成一条记录

![image-20220322135040119](../../../.vuepress/public/images/flowable/image-20220322135040119.png)

![image-20220322135125703](../../../.vuepress/public/images/flowable/image-20220322135125703.png)

## 6.流程完毕

&emsp;&emsp;一个具体的流程审批完成后相关的数据如下：

首先我们会发现

* ACT_RU_EXECUTION 运行时流程执行实例
* ACT_RU_IDENTITYLINK 运行时用户关系信息
* ACT_RU_TASK  运行时任务表
* ACT_RU_VARIABLE  运行时变量表

这四张表中对应的数据都没有了，也就是这个流程已经不是运行中的流程了。然后在对应的历史表中我们可以看到相关的信息

* ACT_HI_ACTINST  历史活动实例表

* ACT_HI_ATTACHMENT 历史的流程附件
* ACT_HI_COMMENT  历史的说明性信息
* ACT_HI_DETAIL 历史的流程运行中的细节信息
* ACT_HI_IDENTITYLINK 历史的流程运行过程中用户关系
* ACT_HI_PROCINST  历史的流程实例
* ACT_HI_TASKINST  历史的任务实例
* ACT_HI_VARINST  历史的流程运行中的变量信息

**`ACT_HI_ACTINST`** 关注的是单个活动（如任务、事件等）的执行历史，适合审计每个具体任务的执行情况。

**`ACT_HI_PROCINST`** 关注的是整个流程实例的生命周期，适合分析流程执行的整体表现和状态。

在我们上面的处理流程的过程中设计到的历史表有

#### 1.ACT_HI_ACTINST  历史活动实例表

存储每个活动实例的历史信息，记录的是每个任务、事件或其他活动的执行情况。

**包含的信息**：

- 活动 ID、活动类型（如用户任务、服务任务、开始事件等）
- 活动的开始和结束时间
- 活动的执行状态（如已完成、挂起等）
- 流程实例 ID（指向 `ACT_HI_PROCINST` 表，表示该活动属于哪个流程实例）
- 任务执行者的信息（如分配给哪个用户）

**特点**：记录的是活动的详细执行历史，通常用于审计和分析每个任务的执行情况。

| 字段               | 类型            | 主键 | 说明                  | 备注 |
| ------------------ | --------------- | ---- | --------------------- | ---- |
| ID_                | NVARCHAR2(64)   | Y    | 主键                  |      |
| PROC_DEF_ID_       | NVARCHAR2(64)   | N    | 流程定义 ID           |      |
| PROC_INST_ID_      | NVARCHAR2(64)   | N    | 流程实例 ID           |      |
| EXECUTION_ID_      | NVARCHAR2(64)   | N    | 执行 ID               |      |
| ACT_ID_            | NVARCHAR2(255)  | N    | 节点实例 ID           |      |
| TASK_ID_           | NVARCHAR2(64)   | N    | 任务 ID               |      |
| CALL_PROC_INST_ID_ | NVARCHAR2(64)   | N    | 调用外部的流程实例 ID |      |
| ACT_NAME_          | NVARCHAR2(255)  | N    | 节点名称              |      |
| ACT_TYPE_          | NVARCHAR2(255)  | N    | 节点类型              |      |
| ASSIGNEE_          | NVARCHAR2(255)  | N    | 节点签收人            |      |
| START_TIME_        | TIMESTAMP(6)    | N    | 开始时间              |      |
| END_TIME_          | TIMESTAMP(6)    | N    | 结束时间              |      |
| DURATION_          | NUMBER(19)      | N    | 耗时                  |      |
| DELETE_REASON_     | NVARCHAR2(2000) | N    | 删除原因              |      |
| TENANT_ID_         | NVARCHAR2(255)  | N    | 租户ID                |      |

![image-20220322141800554](../../../.vuepress/public/images/flowable/image-20220322141800554.png)

![image-20220322141825065](../../../.vuepress/public/images/flowable/image-20220322141825065.png)



#### 2.ACT_HI_IDENTITYLINK 历史的流程运行过程中用户关系

| 字段          | 类型           | 主键 | 说明        | 备注 |
| ------------- | -------------- | ---- | ----------- | ---- |
| ID_           | NVARCHAR2(64)  | Y    | 主键        |      |
| GROUP_ID_     | NVARCHAR2(255) | N    | 组 ID       |      |
| TYPE_         | NVARCHAR2(255) | N    | 类型        |      |
| USER_ID_      | NVARCHAR2(255) | N    | 用户 ID     |      |
| TASK_ID_      | NVARCHAR2(64)  | N    | 任务 ID     |      |
| PROC_INST_ID_ | NVARCHAR2(64)  | N    | 流程实例 ID |      |



![image-20220322141717826](../../../.vuepress/public/images/flowable/image-20220322141717826.png)

#### 3.ACT_HI_PROCINST  历史的流程实例

**用途**：存储每个流程实例的历史信息，记录的是整个流程实例的生命周期。

**包含的信息**：

- 流程实例 ID、流程定义 ID
- 流程实例的启动和结束时间
- 当前状态（如运行中、已完成、已挂起等）
- 发起流程实例的用户信息

**特点**：记录的是流程实例的总体信息，主要用于跟踪整个流程的执行情况。

| 字段                       | 类型            | 主键 | 说明          | 备注 |
| -------------------------- | --------------- | ---- | ------------- | ---- |
| ID_                        | NVARCHAR2(64)   | Y    | 主键          |      |
| PROC_INST_ID_              | NVARCHAR2(64)   | N    | 流程实例 ID   |      |
| BUSINESS_KEY_              | NVARCHAR2(255)  | N    | 业务主键      |      |
| PROC_DEF_ID_               | NVARCHAR2(64)   | N    | 属性 ID       |      |
| START_TIME_                | TIMESTAMP(6)    | N    | 开始时间      |      |
| END_TIME_                  | TIMESTAMP(6)    | N    | 结束时间      |      |
| DURATION_                  | NUMBER(19)      | N    | 耗时          |      |
| START_USER_ID_             | NVARCHAR2(255)  | N    | 起始人        |      |
| START_ACT_ID_              | NVARCHAR2(255)  | N    | 起始节点      |      |
| END_ACT_ID_                | NVARCHAR2(255)  | N    | 结束节点      |      |
| SUPER_PROCESS_INSTANCE_ID_ | NVARCHAR2(64)   | N    | 父流程实例 ID |      |
| DELETE_REASON_             | NVARCHAR2(2000) | N    | 删除原因      |      |
| TENANT_ID_                 | NVARCHAR2(255)  | N    |               |      |
| NAME_                      | NVARCHAR2(255)  | N    | 名称          |      |



![image-20220322141855401](../../../.vuepress/public/images/flowable/image-20220322141855401.png)



![image-20220322141912602](../../../.vuepress/public/images/flowable/image-20220322141912602.png)



#### 4.ACT_HI_TASKINST  历史的任务实例

| 字段            | 类型            | 主键 | 说明                    | 备注                                   |
| --------------- | --------------- | ---- | ----------------------- | -------------------------------------- |
| ID_             | NVARCHAR2(64)   | Y    | 主键                    |                                        |
| PROC_DEF_ID_    | NVARCHAR2(64)   | N    | 流程定义 ID             |                                        |
| TASK_DEF_KEY_   | NVARCHAR2(255)  | N    | 任务定义的 ID 值        |                                        |
| PROC_INST_ID_   | NVARCHAR2(64)   | N    | 流程实例 ID             |                                        |
| EXECUTION_ID_   | NVARCHAR2(64)   | N    | 执行 ID                 |                                        |
| PARENT_TASK_ID_ | NVARCHAR2(64)   | N    | 父任务 ID               |                                        |
| NAME_           | NVARCHAR2(255)  | N    | 名称                    |                                        |
| DESCRIPTION_    | NVARCHAR2(2000) | N    | 说明                    |                                        |
| OWNER_          | NVARCHAR2(255)  | N    | 实际签收人 任务的拥有者 | 签收人（默认为空，只有在委托时才有值） |
| ASSIGNEE_       | NVARCHAR2(255)  | N    | 被指派执行该任务的人    |                                        |
| START_TIME_     | TIMESTAMP(6)    | N    | 开始时间                |                                        |
| CLAIM_TIME_     | TIMESTAMP(6)    | N    | 提醒时间                |                                        |
| END_TIME_       | TIMESTAMP(6)    | N    | 结束时间                |                                        |
| DURATION_       | NUMBER(19)      | N    | 耗时                    |                                        |
| DELETE_REASON_  | NVARCHAR2(2000) | N    | 删除原因                |                                        |
| PRIORITY_       | INTEGER         | N    | 优先级别                |                                        |
| DUE_DATE_       | TIMESTAMP(6)    | N    | 过期时间                |                                        |
| FORM_KEY_       | NVARCHAR2(255)  | N    | 节点定义的 formkey      |                                        |
| CATEGORY_       | NVARCHAR2(255)  | N    | 类别                    |                                        |
| TENANT_ID_      | NVARCHAR2(255)  | N    |                         |                                        |

![image-20220322142609163](../../../.vuepress/public/images/flowable/image-20220322142609163.png)

![image-20220322142650699](../../../.vuepress/public/images/flowable/image-20220322142650699.png)



#### 5.ACT_HI_VARINST  历史的流程运行中的变量信息

流程变量虽然在任务完成后在流程实例表中会删除，但是在历史表中还是会记录的

| 字段               | 类型               | 主键 | 说明                 | 备注 |
| ------------------ | ------------------ | ---- | -------------------- | ---- |
| ID_                | NVARCHAR2(64)      | Y    | 主键                 |      |
| PROC_INST_ID_      | NVARCHAR2(64)      | N    | 流程实例 ID          |      |
| EXECUTION_ID_      | NVARCHAR2(64)      | N    | 指定 ID              |      |
| TASK_ID_           | NVARCHAR2(64)      | N    | 任务 ID              |      |
| NAME_              | NVARCHAR2(255)     | N    | 名称                 |      |
| VAR_TYPE_          | NVARCHAR2(100)     | N    | 参数类型             |      |
| REV_               | INTEGER            | N    | 数据版本             |      |
| BYTEARRAY_ID_      | NVARCHAR2(64)      | N    | 字节表 ID            |      |
| DOUBLE_            | NUMBER(*,10)       | N    | 存储 double 类型数据 |      |
| LONG_              | NUMBER(*,10)       | N    | 存储 long 类型数据   |      |
| TEXT_              | NVARCHAR2(2000)    | N    |                      |      |
| TEXT2_             | NVARCHAR2(2000)    | N    |                      |      |
| CREATE_TIME_       | TIMESTAMP(6)(2000) | N    |                      |      |
| LAST_UPDATED_TIME_ | TIMESTAMP(6)(2000) | N    |                      |      |



![image-20220322142756867](../../../.vuepress/public/images/flowable/image-20220322142756867.png)



# 五、任务分配

&emsp;&emsp;在做流程定义的时候我们需要给相关的`用户节点`指派对应的处理人。在Flowable中提供了三种分配的方式，我们来分别介绍

## 1.固定分配

&emsp;&emsp;固定分配就是我们前面介绍的，在绘制流程图或者直接在流程文件中通过Assignee来指定的方式.

![image-20231104134548306](../../../.vuepress/public/images/flowable/image-20231104134548306.png)

## 2.表达式

&emsp;&emsp;Flowable使用UEL进行表达式解析。UEL代表`Unified Expression Language`，是EE6规范的一部分（查看[EE6规范](http://docs.oracle.com/javaee/6/tutorial/doc/gjddd.html)了解更多信息）。为了在所有环境上支持UEL标准的所有最新特性，我们使用JUEL的修改版本。

&emsp;&emsp;表达式可以用于例如[Java服务任务 Java Service tasks](http://jeecg.com/activiti5.21/#bpmnJavaServiceTaskXML), [执行监听器 Execution Listeners](http://jeecg.com/activiti5.21/#executionListeners), [任务监听器 Task Listeners](http://jeecg.com/activiti5.21/#taskListeners) 与 [条件流 Conditional sequence flows](http://jeecg.com/activiti5.21/#conditionalSequenceFlowXml)。尽管有值表达式与方法表达式两种表达式，通过Flowable的抽象，使它们都可以在需要`expression`（表达式）的地方使用。

>${myVar}
>${myBean.myProperty}

### 2.1 值表达式

&emsp;&emsp;我们在处理的位置通过UEL表达式来占位。

![image-20231104134801444](../../../.vuepress/public/images/flowable/image-20231104134801444.png)



然后做流程的部署和启动操作：

```java
/**
 * 流程部署操作
 */
@Test
public void test1(){
    // 1.获取ProcessEngine对象
    // 2.完成流程的部署操作 需要通过RepositoryService来完成
    RepositoryService repositoryService = processEngine.getRepositoryService();
    // 3.完成部署操作
    Deployment deploy = repositoryService.createDeployment()
            .addClasspathResource("flow/test2.bpmn20.xml")
            .name("请假流程-流程变量")
            .deploy(); // 是一个流程部署的行为 可以部署多个流程定义的
    System.out.println(deploy.getId());
    System.out.println(deploy.getName());
}
```

然后我们发起请假流程：

```java
/**
 * 发起一个流程
 */
@Test
public void test3(){
    ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
    // 发起流程 需要通过 runtimeService来实现
    RuntimeService runtimeService = engine.getRuntimeService();
    // 通过流程定义ID来启动流程  返回的是流程实例对象
    ProcessInstance processInstance = runtimeService
            .startProcessInstanceById("test01:1:12503");
    System.out.println("processInstance.getId() = " + processInstance.getId());
    System.out.println("processInstance.getDeploymentId() = " + processInstance.getDeploymentId());
    System.out.println("processInstance.getDescription() = " + processInstance.getDescription());
}
```

我们发起流程后。根据流程的设计应该需要进入到人事审批。但是呢。审批的用户是`${assign1}`是一个流程变量。那么还没有赋值的情况下。那么系统是没有办法识别的。

![image-20230521154221003](../../../.vuepress/public/images/flowable/image-20230521154221003.png)

我们需要在进入该节点前对流程变量赋值

```java
/**
 * 发起一个流程
 */
@Test
public void test3(){
    ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
    // 发起流程 需要通过 runtimeService来实现
    RuntimeService runtimeService = engine.getRuntimeService();
    // 对流程变量做赋值操作
    Map<String,Object> map = new HashMap<>();
    map.put("assgin1","张三");
    map.put("assign2","李四");
    // 通过流程定义ID来启动流程  返回的是流程实例对象
    ProcessInstance processInstance = runtimeService
            .startProcessInstanceById("test01:1:12503",map);
    System.out.println("processInstance.getId() = " + processInstance.getId());
    System.out.println("processInstance.getDeploymentId() = " + processInstance.getDeploymentId());
    System.out.println("processInstance.getDescription() = " + processInstance.getDescription());
}
```

然后我们就可以看到对应的表结构中的待办记录

![image-20230521154633222](../../../.vuepress/public/images/flowable/image-20230521154633222.png)

同时需要了解 ： ACT_RU_VARIABLE

![image-20230521154720358](../../../.vuepress/public/images/flowable/image-20230521154720358.png)



### 2.2 方法表达式



&emsp;&emsp;**方法表达式 Method expression**: 调用一个方法，可以带或不带参数。**当调用不带参数的方法时，要确保在方法名后添加空括号（以避免与值表达式混淆）。**传递的参数可以是字面值(literal value)，也可以是表达式，它们会被自动解析。例如：

```java
${printer.print()}
${myBean.getAssignee()}
${myBean.addNewOrder('orderName')}
${myBean.doSomething(myVar, execution)}
```

&emsp;&emsp;myBean是Spring容器中的个Bean对象，表示调用的是bean的addNewOrder方法.我们通过案例来演示下。我们先定义对应的Service

先定义Bean

```java
@Component
public class MyBean {

    public String getAssignee(){
        System.out.println("本方法执行了....");
        return "波哥";
    }
}
```

然后在绘制流程图的时候就可以对应的指派了。

![image-20231104135125671](../../../.vuepress/public/images/flowable/image-20231104135125671.png)

然后我们先部署流程

```java
/**
 * 流程部署操作
 */
@Test
public void test1(){
    // 1.获取ProcessEngine对象
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 2.完成流程的部署操作 需要通过RepositoryService来完成
    RepositoryService repositoryService = processEngine.getRepositoryService();
    // 3.完成部署操作
    Deployment deploy = repositoryService.createDeployment()
            .addClasspathResource("flow/test3.bpmn20.xml")
            .name("请假流程-方法表达式")
            .deploy(); // 是一个流程部署的行为 可以部署多个流程定义的
    System.out.println(deploy.getId());
    System.out.println(deploy.getName());
}
```

然后我们发起新的流程。注意在这块我们可以不用添加流程变量信息了。因为`人事审批节点`的审批人是通过流程方法来赋值的

```java
/**
 * 发起一个流程
 */
@Test
public void test3(){
    ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
    // 发起流程 需要通过 runtimeService来实现
    RuntimeService runtimeService = engine.getRuntimeService();
    // 通过流程定义ID来启动流程  返回的是流程实例对象
    ProcessInstance processInstance = runtimeService
            .startProcessInstanceById("test01:2:27503");
    System.out.println("processInstance.getId() = " + processInstance.getId());
    System.out.println("processInstance.getDeploymentId() = " + processInstance.getDeploymentId());
    System.out.println("processInstance.getDescription() = " + processInstance.getDescription());
}
```

可以看到操作成功。方法表达式被执行了

![image-20231104135157316](../../../.vuepress/public/images/flowable/image-20231104135157316.png)

同时待办中的审批人就是方法表达式返回的结果

![image-20231104135231458](../../../.vuepress/public/images/flowable/image-20231104135231458.png)



## 3.监听器分配

&emsp;&emsp;可以使用监听器来完成很多Flowable的流程业务。我们在此处使用监听器来完成负责人的指定，那么我们在流程设计的时候就不需要指定assignee。创建自定义监听器：

```java 
public class MyFirstListener implements TaskListener {
    /**
     * 监听器触发的回调方法
     * @param delegateTask
     */
    @Override
    public void notify(DelegateTask delegateTask) {
        System.out.println("---->自定义的监听器执行了");
        if(EVENTNAME_CREATE.equals(delegateTask.getEventName())){
            // 表示是Task的创建事件被触发了
            // 指定当前Task节点的处理人
            delegateTask.setAssignee("boge666");
        }

    }
}
```

在配置流程的时候关联监听器。注意对应的事件。`CREATE`

![image-20231104135618783](../../../.vuepress/public/images/flowable/image-20231104135618783.png)

然后我们部署和启动流程后。可以看到自定义的监听器触发了

![image-20231104135646708](../../../.vuepress/public/images/flowable/image-20231104135646708.png)

而且待办中的任务的处理人就是监听器中设置的信息

![image-20231104135707799](../../../.vuepress/public/images/flowable/image-20231104135707799.png)



# 六、流程变量

&emsp;&emsp;`流程变量`可以用将数据添加到流程的运行时状态中，或者更具体地说，变量作用域中。改变实体的各种API可以用来更新这些附加的变量。一般来说，一个变量由一个名称和一个值组成。名称用于在整个流程中识别变量。例如，如果一个活动（activity）设置了一个名为 *var* 的变量，那么后续活动中可以通过使用这个名称来访问它。变量的值是一个 Java 对象。

## 1.运行时变量

&emsp;&emsp;流程实例运行时的变量，存入act_ru_variable表中。在流程实例运行结束时，此实例的变量在表中删除。在流程实例创建及启动时，可设置流程变量。所有的`startProcessInstanceXXX`方法都有一个可选参数用于设置变量。例如，`RuntimeService`中

```java
ProcessInstance startProcessInstanceById(String processDefinitionId, Map<String, Object> variables);
```

&emsp;&emsp;也可以在流程执行中加入变量。例如，(*RuntimeService*):

```java
    void setVariable(String executionId, String variableName, Object value);
    void setVariableLocal(String executionId, String variableName, Object value);
    void setVariables(String executionId, Map<String, ? extends Object> variables);
    void setVariablesLocal(String executionId, Map<String, ? extends Object> variables);
```

&emsp;&emsp;读取变量方法:

```java
    Map<String, Object> getVariables(String executionId);
    Map<String, Object> getVariablesLocal(String executionId);
    Map<String, Object> getVariables(String executionId, Collection<String> variableNames);
    Map<String, Object> getVariablesLocal(String executionId, Collection<String> variableNames);
    Object getVariable(String executionId, String variableName);
    <T> T getVariable(String executionId, String variableName, Class<T> variableClass);
```

**注意**：由于流程实例结束时，对应在运行时表的数据跟着被删除。所以查询一个已经完结流程实例的变量，只能在历史变量表中查找。

&emsp;&emsp;当然运行时变量我们也可以根据对应的作用域把他分为`全局变量`和`局部变量`.

### 1.1 全局变量

&emsp;&emsp;流程变量的默认作用域是流程实例。当一个流程变量的作用域为流程实例时，可以称为 global 变量

注意：如：    Global变量：userId（变量名）、zhangsan（变量值）

&emsp;&emsp;global 变量中变量名不允许重复，设置相同名称的变量，后设置的值会覆盖前设置的变量值。

案例：

定义监听器

```java
public class MySecondListener implements TaskListener {

    @Override
    public void notify(DelegateTask delegateTask) {
        // 获取所有的流程变量
        Map<String, Object> variables = delegateTask.getVariables();
        Set<String> keys = variables.keySet();
        for (String key : keys) {
            Object obj = variables.get(key);
            System.out.println(key + " = " + obj);
            if(obj instanceof  String){
              // 修改 流程变量的信息
              // variables.put(key,obj + ":boge3306"); 直接修改Map中的数据 达不到修改流程变量的效果
              delegateTask.setVariable(key + ":boge3306");
            }
        }
    }
}
```

设计流程

![image-20230606111907152](../../../.vuepress/public/images/flowable/image-20230606111907152.png)

然后完成流程的部署操作

```java
@Test
public void test1(){
    // 1.获取ProcessEngine对象
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 2.完成流程的部署操作 需要通过RepositoryService来完成
    RepositoryService repositoryService = processEngine.getRepositoryService();
    // 3.完成部署操作
    Deployment deploy = repositoryService.createDeployment()
            .addClasspathResource("flow/holiday1.bpmn20.xml")
            .name("请假流程-流程变量")
            .deploy(); // 是一个流程部署的行为 可以部署多个流程定义的
    System.out.println(deploy.getId());
    System.out.println(deploy.getName());
}
```

然后启动流程实例。注意在启动流程实例时我们需要指定相关的流程变量

```java
/**
 * 发起一个流程
 */
@Test
public void test3(){
    ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
    // 发起流程 需要通过 runtimeService来实现
    RuntimeService runtimeService = engine.getRuntimeService();
    // 定义流程变量信息
    Map<String,Object> map = new HashMap<>();
    map.put("assignee1","张三");
    map.put("assignee2","李四");
    map.put("ass1","变量1");
    map.put("ass2",299);
    map.put("ass3","湖南长沙");
    map.put("ass4","波哥666");
    // 通过流程定义ID来启动流程  返回的是流程实例对象
    ProcessInstance processInstance = runtimeService
            .startProcessInstanceById("holiday1:1:42503",map);
    System.out.println("processInstance.getId() = " + processInstance.getId());
    System.out.println("processInstance.getDeploymentId() = " + processInstance.getDeploymentId());
    System.out.println("processInstance.getDescription() = " + processInstance.getDescription());
}
```

启动流程和触发对应的监听器，同时会在`act_ru_variable`中记录当前的变量信息

![image-20230606112144417](../../../.vuepress/public/images/flowable/image-20230606112144417.png)

当然我们也可以通过`RunTimeService`来查询当前对应的流程实例的流程变量信息

```java
/**
 * 查询当前的流程变量信息
 */
@Test
public void testVal(){
    ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
    RuntimeService runtimeService = engine.getRuntimeService();
    // 获取流程变量信息  获取某个流程实例的变量信息
    Map<String, VariableInstance> variableInstances =
            runtimeService.getVariableInstances("50008");
    Set<String> keys = variableInstances.keySet();
    for (String key : keys) {
        System.out.println(key+"="+variableInstances.get(key));
    }
}
```



###  1.2 局部变量

&emsp;&emsp;任务和执行实例仅仅是针对一个任务和一个执行实例范围，范围没有流程实例大， 称为 local 变量。

&emsp;&emsp;Local 变量由于在不同的任务或不同的执行实例中，作用域互不影响，变量名可以相同没有影响。Local 变量名也可以和 global 变量名相同，没有影响。

我们通过RuntimeService 设置的Local变量绑定的是 executionId。在该流程中有效

```java 
    @Test
    public void test4(){
        ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
        // 待办查询 执行中的任务处理通过 TaskService来实现
        TaskService taskService = engine.getTaskService();
        RuntimeService runtimeService = engine.getRuntimeService();
        runtimeService.setVariableLocal("60004","orderId","100001");
        runtimeService.setVariableLocal("60004","price",6666);
    }
```

我们还可以通过TaskService来绑定本地流程变量。需要指定对应的taskId

```java 
@Test
public void test41(){
    ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
    // 待办查询 执行中的任务处理通过 TaskService来实现
    TaskService taskService = engine.getTaskService();
    taskService.setVariableLocal("60007","wechat","boge3306");
}
```

然后通过测试演示我们可以看到通过TaskService绑定的Local变量的作用域只是在当前的Task有效。而通过RuntimeService绑定的Local变量作用的访问是executionId。

需要注意：executionId和processInstanceId的区别



## 2.历史变量

&emsp;&emsp;**历史变量**，存入`act_hi_varinst`表中。在流程启动时，流程变量会同时存入历史变量表中；在流程结束时，历史表中的变量仍然存在。可理解为“永久代”的流程变量。

&emsp;&emsp;获取已完成的、id为’XXX’的流程实例中，所有的`HistoricVariableInstances`（历史变量实例），并以变量名排序。

```java
historyService.createHistoricVariableInstanceQuery()
    .processInstanceId("XXX")
    .orderByVariableName.desc()
    .list();
```



# 七、身份服务

&emsp;&emsp;在流程定义中在任务结点的 assignee 固定设置任务负责人，在流程定义时将参与者固定设置在.bpmn 文件中，如果临时任务负责人变更则需要修改流程定义，系统可扩展性差。针对这种情况可以给任务设置多个候选人或者候选人组，可以从候选人中选择参与者来完成任务。

## 1.候选人

### 1.1 定义流程图

&emsp;&emsp;定义流程图，同时指定候选人，多个候选人会通过`,`连接

![image-20220325095959489](../../../.vuepress/public/images/flowable/image-20220325095959489.png)



### 1.2 部署和启动流程实例

&emsp;&emsp;部署流程，并且在启动流程实例的时候对UEL表达式赋值

```java
    /**
     * 部署流程
     */
    @Test
    public void deploy(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        RepositoryService repositoryService = processEngine.getRepositoryService();

        Deployment deploy = repositoryService.createDeployment()
                .addClasspathResource("请假流程-候选人.bpmn20.xml")
                .name("请求流程-候选人")
                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
        System.out.println(deploy.getName());
    }

    /**
     * 启动流程实例
     */
    @Test
    public void runProcess(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        RuntimeService runtimeService = processEngine.getRuntimeService();
        // 给流程定义中的UEL表达式赋值
        Map<String,Object> variables = new HashMap<>();
        variables.put("candidate1","张三");
        variables.put("candidate2","李四");
        variables.put("candidate3","王五");
        runtimeService.startProcessInstanceById("holiday-candidate:1:4",variables);
    }
```

&emsp;&emsp;在对应的表结构中我们可以看到流程变量已经有了，但是对于的Task的Assignee还是为空。

![image-20220325101054787](../../../.vuepress/public/images/flowable/image-20220325101054787.png)





![image-20220325102600573](../../../.vuepress/public/images/flowable/image-20220325102600573.png)





### 1.3 任务的查询

&emsp;&emsp;根据当前登录的用户，查询对应的候选任务

```java
   /**
     * 根据登录的用户查询对应的可以拾取的任务
     *
     */
    @Test
    public void queryTaskCandidate(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        TaskService taskService = processEngine.getTaskService();
        List<Task> list = taskService.createTaskQuery()
                //.processInstanceId("2501")
                .processDefinitionId("holiday-candidate:1:4")   
                .taskCandidateUser("李四") # 注意
                .list();
        for (Task task : list) {
            System.out.println("task.getId() = " + task.getId());
            System.out.println("task.getName() = " + task.getName());
        }
    }
```



### 1.4 任务的拾取

&emsp;&emsp;知道了我有可拾取的任务后，拾取任务。

```java
/**
     * 拾取任务
     *    一个候选人拾取了这个任务之后其他的用户就没有办法拾取这个任务了
     *    所以如果一个用户拾取了任务之后又不想处理了，那么可以退还
     */
    @Test
    public void claimTaskCandidate(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        TaskService taskService = processEngine.getTaskService();
        Task task = taskService.createTaskQuery()
                //.processInstanceId("2501")
                .processDefinitionId("holiday-candidate:1:4")
                .taskCandidateUser("李四")
                .singleResult();
        if(task != null){
            // 拾取对应的任务
            taskService.claim(task.getId(),"李四");
            System.out.println("任务拾取成功");
        }
    }
```



![image-20220325103624344](../../../.vuepress/public/images/flowable/image-20220325103624344.png)



### 1.5 任务的归还

&emsp;&emsp;拾取任务后不想操作那么就归还任务

```java
    /**
     * 退还任务
     *    一个候选人拾取了这个任务之后其他的用户就没有办法拾取这个任务了
     *    所以如果一个用户拾取了任务之后又不想处理了，那么可以退还
     */
    @Test
    public void unclaimTaskCandidate(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        TaskService taskService = processEngine.getTaskService();
        Task task = taskService.createTaskQuery()
                //.processInstanceId("2501")
                .processDefinitionId("holiday-candidate:1:4")
                .taskAssignee("张三")
                .singleResult();
        if(task != null){
            // 拾取对应的任务
            taskService.unclaim(task.getId());
            System.out.println("归还拾取成功");
        }
    }
```





### 1.6 任务的交接

&emsp;&emsp;拾取任务后如果不想操作也不想归还可以直接交接给另外一个人来处理

```java
   /**
     * 任务的交接
     *    如果我获取了任务，但是不想执行，那么我可以把这个任务交接给其他的用户
     */
    @Test
    public void taskCandidate(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        TaskService taskService = processEngine.getTaskService();
        Task task = taskService.createTaskQuery()
                //.processInstanceId("2501")
                .processDefinitionId("holiday-candidate:1:4")
                .taskAssignee("李四")
                .singleResult();
        if(task != null){
            // 任务的交接
            taskService.setAssignee(task.getId(),"王五");
            System.out.println("任务交接给了王五");
        }
    }
```





### 1.7 任务的完成

&emsp;&emsp;正常的任务处理

```java
   /**
     * 完成任务
     */
    @Test
    public void completeTask(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        TaskService taskService = processEngine.getTaskService();
        Task task = taskService.createTaskQuery()
                //.processInstanceId("2501")
                .processDefinitionId("holiday-candidate:1:4")
                .taskAssignee("王五")
                .singleResult();
        if(task != null){
            // 完成任务
            taskService.complete(task.getId());
            System.out.println("完成Task");
        }
    }

```







## 2.候选人组

&emsp;&emsp;当候选人很多的情况下，我们可以分组来处理。先创建组，然后把用户分配到这个组中。

### 2.1 管理用户和组

#### 2.1.1 用户管理

&emsp;&emsp;我们需要先单独维护用户信息。后台对应的表结构是`ACT_ID_USER`.

```java
   /**
     * 维护用户
     */
    @Test
    public void createUser(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        // 通过 IdentityService 完成相关的用户和组的管理
        IdentityService identityService = processEngine.getIdentityService();
        User user = identityService.newUser("田佳");
        user.setFirstName("田");
        user.setLastName("jia");
        user.setEmail("tianjia@qq.com");
        identityService.saveUser(user);
    }
```



![image-20220325110324815](../../../.vuepress/public/images/flowable/image-20220325110324815.png)



#### 2.1.2 Group管理

&emsp;&emsp;维护对应的Group信息，后台对应的表结构是`ACT_ID_GROUP`

```java
    /**
     * 创建用户组
     */
    @Test
    public void createGroup(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        IdentityService identityService = processEngine.getIdentityService();
        // 创建Group对象并指定相关的信息
        Group group = identityService.newGroup("group2");
        group.setName("开发部");
        group.setType("type1");
        // 创建Group对应的表结构数据
        identityService.saveGroup(group);

    }
```

![image-20220325110408435](../../../.vuepress/public/images/flowable/image-20220325110408435.png)



#### 2.1.3 用户分配组

&emsp;&emsp;用户和组是一个多对多的关联关联，我们需要做相关的分配，后台对应的表结构是`ACT_ID_MEMBERSHIP`

```java
    /**
     * 将用户分配给对应的Group
     */
    @Test
    public void userGroup(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        IdentityService identityService = processEngine.getIdentityService();
        // 根据组的编号找到对应的Group对象
        Group group = identityService.createGroupQuery().groupId("group1").singleResult();
        List<User> list = identityService.createUserQuery().list();
        for (User user : list) {
            // 将用户分配给对应的组
            identityService.createMembership(user.getId(),group.getId());
        }
    }
```



![image-20220325110511848](../../../.vuepress/public/images/flowable/image-20220325110511848.png)





### 2.2 候选人组应用

&emsp;&emsp;搞清楚了用户和用户组的关系后我们就可以来使用候选人组的应用了

#### 2.2.1 创建流程图

![image-20220325111013641](../../../.vuepress/public/images/flowable/image-20220325111013641.png)



![image-20220325110952527](../../../.vuepress/public/images/flowable/image-20220325110952527.png)







#### 2.2.2 流程的部署运行

&emsp;&emsp;然后我们把流程部署和运行，注意对UEL表达式赋值，关联上Group

```java
/**
     * 部署流程
     */
    @Test
    public void deploy(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        RepositoryService repositoryService = processEngine.getRepositoryService();

        Deployment deploy = repositoryService.createDeployment()
                .addClasspathResource("请假流程-候选人组.bpmn20.xml")
                .name("请求流程-候选人")
                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
        System.out.println(deploy.getName());
    }

    /**
     * 启动流程实例
     */
    @Test
    public void runProcess(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        IdentityService identityService = processEngine.getIdentityService();
        Group group = identityService.createGroupQuery().groupId("group1").singleResult();
        RuntimeService runtimeService = processEngine.getRuntimeService();
        // 给流程定义中的UEL表达式赋值
        Map<String,Object> variables = new HashMap<>();
        // variables.put("g1","group1");
        variables.put("g1",group.getId()); // 给流程定义中的UEL表达式赋值
        runtimeService.startProcessInstanceById("holiday-group:1:17504",variables);
    }
```

对应表结构中就有对应的体现

![image-20220325112545719](../../../.vuepress/public/images/flowable/image-20220325112545719.png)





#### 2.2.3 任务的拾取和完成

&emsp;&emsp;然后完成任务的查询拾取和处理操作

```java
/**
     * 根据登录的用户查询对应的可以拾取的任务
     *
     */
    @Test
    public void queryTaskCandidateGroup(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        // 根据当前登录的用户找到对应的组
        IdentityService identityService = processEngine.getIdentityService();
        // 当前用户所在的组
        Group group = identityService.createGroupQuery().groupMember("邓彪").singleResult();

        TaskService taskService = processEngine.getTaskService();
        List<Task> list = taskService.createTaskQuery()
                //.processInstanceId("2501")
                .processDefinitionId("holiday-group:1:17504")
                .taskCandidateGroup(group.getId())
                .list();
        for (Task task : list) {
            System.out.println("task.getId() = " + task.getId());
            System.out.println("task.getName() = " + task.getName());
        }
    }

    /**
     * 拾取任务
     *    一个候选人拾取了这个任务之后其他的用户就没有办法拾取这个任务了
     *    所以如果一个用户拾取了任务之后又不想处理了，那么可以退还
     */
    @Test
    public void claimTaskCandidate(){
        String userId = "田佳";
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        // 根据当前登录的用户找到对应的组
        IdentityService identityService = processEngine.getIdentityService();
        // 当前用户所在的组
        Group group = identityService.createGroupQuery().groupMember(userId).singleResult();
        TaskService taskService = processEngine.getTaskService();
        Task task = taskService.createTaskQuery()
                //.processInstanceId("2501")
                .processDefinitionId("holiday-group:1:17504")
                .taskCandidateGroup(group.getId())
                .singleResult();
        if(task != null) {
            // 任务拾取
            taskService.claim(task.getId(),userId);
            System.out.println("任务拾取成功");
        }
    }  
   /**
     * 完成任务
     */
    @Test
    public void completeTask(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        TaskService taskService = processEngine.getTaskService();
        Task task = taskService.createTaskQuery()
                //.processInstanceId("2501")
                .processDefinitionId("holiday-group:1:17504")
                .taskAssignee("邓彪")
                .singleResult();
        if(task != null){
            // 完成任务
            taskService.complete(task.getId());
            System.out.println("完成Task");
        }
    }
```



# 八、网关服务

网关用来控制流程的流向

## 1.排他网关

&emsp;&emsp;排他网关（exclusive gateway）（也叫*异或网关 XOR gateway*，或者更专业的，*基于数据的排他网关 exclusive data-based gateway*），用于对流程中的**决策**建模。当执行到达这个网关时，会按照所有出口顺序流定义的顺序对它们进行计算。选择第一个条件计算为true的顺序流（当没有设置条件时，认为顺序流为*true*）继续流程。

&emsp;&emsp;**请注意这里出口顺序流的含义与BPMN 2.0中的一般情况不一样。一般情况下，会选择所有条件计算为true的顺序流，并行执行。而使用排他网关时，只会选择一条顺序流。当多条顺序流的条件都计算为true时，会且仅会选择在XML中最先定义的顺序流继续流程。如果没有可选的顺序流，会抛出异常。**

图示

&emsp;&emsp;排他网关用内部带有’X’图标的标准网关（菱形）表示，'X’图标代表*异或*的含义。请注意内部没有图标的网关默认为排他网关。BPMN 2.0规范不允许在同一个流程中混合使用有及没有X的菱形标志。

![image-20220326100630908](../../../.vuepress/public/images/flowable/image-20220326100630908.png)



案例：

![image-20220326103951903](../../../.vuepress/public/images/flowable/image-20220326103951903.png)



```java
   /**
     * 部署流程
     */
    @Test
    public void deploy(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        RepositoryService repositoryService = processEngine.getRepositoryService();

        Deployment deploy = repositoryService.createDeployment()
                .addClasspathResource("请假流程-排他网关.bpmn20.xml")
                .name("请求流程-排他网关")
                .deploy();
        System.out.println("deploy.getId() = " + deploy.getId());
        System.out.println(deploy.getName());
    }

    /**
     * 启动流程实例
     */
    @Test
    public void runProcess(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        RuntimeService runtimeService = processEngine.getRuntimeService();
        // 给流程定义中的UEL表达式赋值
        Map<String,Object> variables = new HashMap<>();
        // variables.put("g1","group1");
        variables.put("num",3); // 给流程定义中的UEL表达式赋值
        runtimeService.startProcessInstanceById("holiday-exclusive:1:4",variables);
    }


    /**
     * 启动流程实例
     */
    @Test
    public void setVariables(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        RuntimeService runtimeService = processEngine.getRuntimeService();
        // 给流程定义中的UEL表达式赋值
        Map<String,Object> variables = new HashMap<>();
        // variables.put("g1","group1");
        variables.put("num",4); // 给流程定义中的UEL表达式赋值
        runtimeService.setVariables("12503",variables);
    }



    /**
     * 完成任务
     */
    @Test
    public void completeTask(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        TaskService taskService = processEngine.getTaskService();
        Task task = taskService.createTaskQuery()
                //.processInstanceId("2501")
                .processDefinitionId("holiday-exclusive:1:4")
                .taskAssignee("zhangsan")
                .singleResult();
        if(task != null){
            // 完成任务
            taskService.complete(task.getId());
            System.out.println("完成Task");
        }
    }
```





如果从网关出去的线所有条件都不满足的情况下会抛出系统异常，

![image-20220326104744181](../../../.vuepress/public/images/flowable/image-20220326104744181.png)



但是要注意任务没有介绍，还是原来的任务，我们可以重置流程变量

```java
    @Test
    public void setVariables(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        RuntimeService runtimeService = processEngine.getRuntimeService();
        // 给流程定义中的UEL表达式赋值
        Map<String,Object> variables = new HashMap<>();
        // variables.put("g1","group1");
        variables.put("num",4); // 给流程定义中的UEL表达式赋值
        runtimeService.setVariables("12503",variables);
    }
```







## 2.并行网关

&emsp;&emsp;并行网关允许将流程分成多条分支，也可以把多条分支汇聚到一起，并行网关的功能是基于进入和外出顺序流的：

* fork分支：并行后的所有外出顺序流，为每个顺序流都创建一个并发分支。

* join汇聚： 所有到达并行网关，在此等待的进入分支， 直到所有进入顺序流的分支都到达以后， 流程就会通过汇聚网关。

&emsp;&emsp;注意，如果同一个并行网关有多个进入和多个外出顺序流， 它就同时具有分支和汇聚功能。 这时，网关会先汇聚所有进入的顺序流，然后再切分成多个并行分支。

**与其他网关的主要区别是，并行网关不会解析条件。** **即使顺序流中定义了条件，也会被忽略。**



案例：

![image-20220326110341232](../../../.vuepress/public/images/flowable/image-20220326110341232.png)

当我们执行了创建请假单后，到并行网关的位置的时候，在ACT_RU_TASK表中就有两条记录

![image-20220326111359504](../../../.vuepress/public/images/flowable/image-20220326111359504.png)

然后同时在ACT_RU_EXECUTION中有三条记录，一个任务对应的有两个执行实例

![image-20220326111453630](../../../.vuepress/public/images/flowable/image-20220326111453630.png)







## 3.包含网关

&emsp;包含网关可以看做是`排他网关`和`并行网关`的结合体。 和排他网关一样，你可以在外出顺序流上定义条件，包含网关会解析它们。 但是主要的区别是包含网关可以选择多于一条顺序流，这和并行网关一样。

包含网关的功能是基于进入和外出顺序流的：

* 分支： 所有外出顺序流的条件都会被解析，结果为true的顺序流会以并行方式继续执行， 会为每个顺序流创建一个分支。

* 汇聚：所有并行分支到达包含网关，会进入等待状态， 直到每个包含流程token的进入顺序流的分支都到达。 这是与并行网关的最大不同。换句话说，包含网关只会等待被选中执行了的进入顺序流。 在汇聚之后，流程会穿过包含网关继续执行。



![image-20220326112720089](../../../.vuepress/public/images/flowable/image-20220326112720089.png)





## 4.事件网关

&emsp;&emsp;事件网关允许根据事件判断流向。网关的每个外出顺序流都要连接到一个中间捕获事件。 当流程到达一个基于事件网关，网关会进入等待状态：会暂停执行。与此同时，会为每个外出顺序流创建相对的事件订阅。

&emsp;&emsp;事件网关的外出顺序流和普通顺序流不同，这些顺序流不会真的"执行"， 相反它们让流程引擎去决定执行到事件网关的流程需要订阅哪些事件。 要考虑以下条件：

1. 事件网关必须有两条或以上外出顺序流；
2. 事件网关后，只能使用intermediateCatchEvent类型（activiti不支持基于事件网关后连接ReceiveTask）
3. 连接到事件网关的中间捕获事件必须只有一个入口顺序流。

# 九、总结

## 查询sql

```sql
#	流程开启时，首先流程历史表ACT_HI_PROCINST 会产生一条数据，
select * from ACT_HI_PROCINST where id_='0c660f05-8ee3-11ef-a426-025cc22bef59'

#查询这个流程实例的结束时间是否结束
select END_TIME_ from ACT_HI_PROCINST where id_='0c660f05-8ee3-11ef-a426-025cc22bef59'

#如果没有结束，正在执行的任务表ACT_RU_TASK 大概率存在数据。（获取当前流程实例正在等待执行的任务）
select * from ACT_RU_TASK where PROC_INST_ID_='0c660f05-8ee3-11ef-a426-025cc22bef59'

#一条 ACT_RU_EXECUTION 记录（即一个执行路径）可以进入多个活动节点，因此在 ACT_RU_ACTINST 中会有多个对应的活动节点记录。
#如果没有结束，ACT_RU_EXECUTION 存在完整的流程实例执行的流程顺序。管理流程实例的执行状态，包括整个流程实例的主执行路径、并行执行路径等。 
select * from ACT_RU_EXECUTION  where PROC_INST_ID_='0c660f05-8ee3-11ef-a426-025cc22bef59' ORDER BY START_TIME_;

#表记录具体活动节点的执行情况，即执行路径进入了哪个具体的活动
select * from ACT_RU_ACTINST  where PROC_INST_ID_='0c660f05-8ee3-11ef-a426-025cc22bef59'  ORDER BY START_TIME_;
#当流程执行到用户任务（userTask）节点时，会在 ACT_RU_ACTINST 表中记录该活动实例。
#ACT_RU_ACTINST.TASK_ID_：这个字段存储的是任务的 ID，它与 ACT_RU_TASK.ID_ 字段对应。
#也可以通过流程实例 ID 来关联，表示该任务属于哪个流程实例。

#查询活动节点绑定的任务
select * from ACT_RU_TASK where id_="0c66361a-8ee3-11ef-a426-025cc22bef59"

#ACT_RU_ACTINST.EXECUTION_ID_：指向 ACT_RU_EXECUTION.ID_ 字段,表示活动实例属于哪个执行路径。
SELECT * FROM ACT_RU_EXECUTION WHERE ID_ IN (SELECT EXECUTION_ID_ FROM ACT_RU_ACTINST WHERE ID_ = '0c660f07-8ee3-11ef-a426-025cc22bef59');

# ACT_RU_VARIABLE 表中的变量是与 ACT_RU_EXECUTION 中的执行路径（Execution）相关联的。关联字段：ACT_RU_VARIABLE.EXECUTION_ID_ 关联到 ACT_RU_EXECUTION.ID_。
#任务可以通过其 TASK_ID_ 直接访问 ACT_RU_VARIABLE 表中的变量，前提是这些变量与任务直接关联（即 TASK_ID_ 字段已填充）
#注意：当一个变量的 TASK_ID_ 字段被设置时，这个变量是特定于某个任务的，只有这个任务可以直接访问它，没有值的就是全局变量
SELECT * FROM ACT_RU_VARIABLE WHERE EXECUTION_ID_ = (SELECT EXECUTION_ID_ FROM ACT_RU_TASK WHERE ID_ = 'someTaskId');

SELECT * FROM ACT_RU_VARIABLE WHERE TASK_ID_ = 'someTaskId';

#ACT_RU_TASK 表：
#关系：ACT_RU_IDENTITYLINK.TASK_ID_ 与 ACT_RU_TASK.ID_ 关联。表示某个任务的身份链接。
#用途：查找某个任务的身份链接
SELECT * FROM ACT_RU_IDENTITYLINK WHERE TASK_ID_ = 'someTaskId';

ACT_RU_EXECUTION 表：

#关系：ACT_RU_IDENTITYLINK.PROC_INST_ID_ 与 ACT_RU_EXECUTION.PROC_INST_ID_ 关联，表示与某个流程实例相关的身份链接。
#用途：可以用于控制流程实例的权限，确保只有特定用户或组能够查看或参与该实例。

#ACT_RU_VARIABLE 表（间接关系）：
#关系：虽然 ACT_RU_VARIABLE 不直接与 ACT_RU_IDENTITYLINK 关联，但它可能与任务和流程实例共享数据，而身份链接则管理对这些数据的访问权限。
#用途：可以帮助控制哪些用户或组可以访问特定的变量。
```













