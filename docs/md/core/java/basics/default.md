---
layout: post
category: curleyg-spring-ioc
title: 第01章：Java概述
tagline: by CurleyG
tag: [java]
excerpt: Java概述
lock: need
---


# Java概述

java是一个完整的平台，有一个庞大的库，其中还包含了很多可重用的代码和一个提供安全性，跨操作系统，可移植性以及可以自动垃圾回收的执行执行环境。

## 1 ) 简单性

相比较于C++来说，java的语法是很接近C++的，但是java舍弃了C++中许多很少使用，难以理解、 易混淆的特性。

> 头文件、 指针运算（甚至指针语法) 结构、 联合、 操作符重载、 虚基类等

## 2 ) 面向对象

1.面向对象

- 面向对象以对象为中心：先把要完成的功能封装成一个一个的对象，通过调用对象的方法或属性来完成功能。
- 优点：不仅关注眼前的事件实现，也关注未来可能发生的事件。具有高度的拓展性和复用性，特点是继承、封装、多态。
- 缺点：如果只是单一的功能实现，面向对象的设计思路会过于繁琐。

  2.面向过程

- 面向过程是以事件为中心，按照我们编写的代码是根据完成一个步骤的过程来进行。
- 优点：根据事情的目的分解出过程，再一步步实施。对于不复杂的事件执行效率快。
- 缺点：只关注眼前事件的实现。

  3.简单概括

  简单地讲， 面向对象设计是一种程序设计技术。它将重点放在教椐（即对象）和对象的接口上。用木匠打一个比方， 一个**面向对象**的木匠始终关注的是所制作的椅子，第二位才是所使用的工具；一个**非面向对象**的木匠首先考虑的是所用的工具。在本质上，Java的面向对象能力与 C++ 是一样的

## 3 ) 分布式

Java程序只要编写一次，就可到处运行。所以Java是一种支持网络应用的分布式语言，其设计特点使其能够处理基于TCP/IP协议如HTTP和FTP等，使得Java应用程序能够通过URL访问网络上的对象，这种访问方式与访问本地文件系统非常相似

## 4 ) 健壮性

Java的健壮性主要体现在其**强类型机制**、**异常处理**、**垃圾回收机制**以及**取消指针算法**等方面。

- Java的强类型检查机制在编译阶段就能发现许多潜在的错误，大大减少了运行时错误的可能性。
- Java的异常处理机制允许开发者在出现异常情况时进行特定的处理，从而避免程序崩溃。
- Java的垃圾回收机制自动管理内存，减少了内存泄漏和溢出的问题。
- Java取消了指针算法，进一步提高了程序的健壮性，用户无需担心内存分配错误或指针溢出的问题。

## 5 ) 安全性

​    Java的存储分配模型是它防御恶意代码的主要方法之一。Java没有指针，所以程序员不能得到隐蔽起来的内幕和伪造指针去指向存储器。更重要的是，Java编译程序不处理存储安排决策，所以程序员不能通过查看[声明](https://baike.baidu.com/item/声明/13130358?fromModule=lemma_inlink)去猜测类的实际存储安排。编译的Java代码中的存储引用在运行时由Java解释程序决定实际存储地址。

Java运行系统使用字节码验证过程来保证装载到网络上的代码不违背任何Java语言限制。这个安全机制部分包括类如何从网上装载。例如，装载的类是放在分开的名字空间而不是局部类，预防恶意的小应用程序用它自己的版本来代替标准Java类。

## 6 ) 体系结构中立

​    Java的特点之一是其体系结构中立性。

​    这意味着Java解释器生成与体系结构无关的字节码指令。当Java源程序被编译时，它会被转化为一种高层次的、与机器无关的字节码格式。这种格式的设计初衷是在虚拟机上运行，并由与机器相关的运行调试器来实现执行。因此，只要存在Java运行时系统，Java程序便可以在任何处理器上运行，无论该处理器是基于何种体系结构。

## 7 ) 可移植性

​    Java并不依赖平台，用Java编写的程序可以运用到任何操作系统上。

## 8 ) 解释型

‌   **[Java]是一种既包含编译特性也包含解释特性的语言。**‌ Java程序首先被编译成字节码，然后通过[Java虚拟机]（JVM）进行解释执行或[即时编译]（JIT），这使得Java既可以说成是编译型语言，也可以说是解释性语言‌。

Java的编译过程是将[源代码]（*.java文件）编译成字节码（*.class文件）。这些字节码包含了运行程序所需的所有信息，但本身不可直接执行，必须通过JVM来解释执行‌。

在执行过程中，JVM会采用两种主要的方式来提高性能：解释执行和即时编译。解释执行是指JVM直接解释字节码并执行；即时编译（JIT）则是在运行时将热点代码编译成本地机器码，从而提高执行效率‌。

此外，Java还具有跨平台的能力，能够在任何支持JVM的平台上运行，这得益于其字节码的设计，使得Java程序可以在不同的操作系统上无缝运行‌

## 9 ) 高性能

高性能的体现只要是即时编译器的作用.

即时编译器已经非常出色，以至于成了传统编译器的竞争对手。在某些情况下，甚至超越了传统编译器，原因是它们含有更多的可用信息。

例如，

即时编译器可以监控经常执行哪些代码并优化这些代码以提高速度。更为复杂的优化是消除函数调用（即“内联”）。

即时编译器知道哪些类已经加载 = 基于当前加载的类集， 如果特定的函数不会被覆盖，就可以使用内联。必要时，还可以撤销优化。

## 10 ) 多线程

Java是多线程语言，它可以同时执行多个程序，能处理不同任务。能够更好利用多核处理器的性能。

## 11 ) 动态性

Java本质为静态语言，而不是动态语言。动态语言显著的特点是在程序运行时，可以改变程序结构或变量类型，典型的动态语言有Python、ruby、javascript等。Java不是动态语言，但Java具有一定的动态性，表现在以下几个方面:

> 程序运行时,可以改变程序得结构或变量类型.典型语言:

- Python,Ruby,JavaScript等.
- 如下JavaScript代码

```javascript
function test(){
    var s = "var a=3;var b=5;alert(a+b);";
    eval(s);
}
```

- C,C++,Java不是动态语言,但Java有一定的动态性,我们可以利用反射机制,字节码操作获得类似动态语言的特性
- Java的动态性让编程的时候更加灵活

### 1.反射机制

反射机制指的是可以在运行期间加载一些知道名字的类
对于任意一个已加载的类,都能够知道这个类的所有属性和方法;对于任意一个对象,都能调用它的任意一个方法或属性

```java
Class c = Class.forName("com.test.User");
```

类加载完之后,在堆内存中会产生一个Class类的对象(一个类只有一个Class对象),这个对象包含了完整的类的结构信息,我们可以通过这个对象看到类的结果

Class类介绍

- java.lang.Class类十分特殊,用来表示java中类型(class/interface/enum/annotation/primitive type/void)本身

  Class类的对象包含了某个被加载类的结构,一个被加载的类对应一个Class对象
  当一个class被加载,或当加载器(class loader)的defineClass()被JVM调用,JVM便会自动产生一个Class对象

- Class类是Reflection的根源

  针对任何你想动态加载,运行的类,只有先获得相应的Class对象

User bean:

```java
package com.lorinda.bean;

public class User {
    
    private int id;
    private int age;
    private String uname;
    
    public User(int id, int age, String uname) {
        super();
        this.id = id;
        this.age = age;
        this.uname = uname;
    }
    
    public User() {
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getUname() {
        return uname;
    }

    public void setUname(String uname) {
        this.uname = uname;
    }
     
}
```

测试各种类型对应Class对象的获取方式:

```java
/**
 * 测试各种类型对应Class对象的获取方式
 * @author Matrix42
 *
 */
public class ReflectionDemo01 {

    public static void main(String[] args) {

        String path = "com.lorinda.bean.User";
        
        try {
            Class<?> clazz = Class.forName(path);
            System.out.println(clazz);              //class com.lorinda.bean.User
            System.out.println(clazz.hashCode());   //366712642
            //同样的类只会被加载一次
            Class<?> clazz2 = Class.forName(path);
            System.out.println(clazz2.hashCode());  //366712642
            
            Class<String> strClazz = String.class;  //类名.class
            
            Class<?> strClazz2 = path.getClass();   //对象.getClass();
            
            System.out.println(strClazz==strClazz2);//true
            
            Class<?> intClazz = int.class;
            
            int[] arr01 = new int[10];
            int[] arr02 = new int [30];
            
            int[][] arr03 = new int[30][3];
            
            //数组的Class对象只与类型和维度有关
            System.out.println(arr01.getClass()==arr02.getClass()); //true
            
            System.out.println(arr01.getClass().hashCode());        //1829164700
            System.out.println(arr03.getClass().hashCode());        //2018699554
            
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
```

Class类的对象如何获取

- 对于对象可以使用getClass()
- 使用Class.forName() (最常使用)
- 使用.class

反射机制的常见作用

- 动态加载类,动态获取类的信息(属性,方法,构造器)
- 动态构造对象
- 动态调用类和对象的任意方法,构造器
- 动态调用和处理属性
- 获取泛型信息
- 处理注解

获取方法,属性,构造器等的信息

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

/**
 * 获取方法,属性,构造器等的信息
 * @author Matrix42
 *
 */
public class ReflectionDemo02 {

    public static void main(String[] args) {
       
        String path = "com.lorinda.bean.User";
        
        try {
            Class<?> clazz = Class.forName(path);

            //获取类的名字
            System.out.println(clazz.getName());//获得包名+类名:com.lorinda.bean.User
            System.out.println(clazz.getSimpleName());//获得类名:User
            
            //获取属性信息
            //Field[] fields = clazz.getFields();//只能获取public的field
            Field[] fields = clazz.getDeclaredFields();//获得所有的field
            Field field = clazz.getDeclaredField("uname");//根据名字获取field
            
            for(Field temp:fields){
                System.out.println("属性: "+temp);
            }
            
            //获取方法
            Method[] methods = clazz.getDeclaredMethods();
            Method method01 = clazz.getDeclaredMethod("getUname", null);
            //如果方法有参数,则必须传递参数类型对应的Class对象
            Method method02 = clazz.getDeclaredMethod("setUname", String.class);
            
            for(Method m:methods){
                System.out.println("方法: "+m);
            }
            
            //获得构造器信息
            Constructor[] constructors = clazz.getDeclaredConstructors();
            //单独获取,无参
            Constructor c1 = clazz.getDeclaredConstructor(null);
            System.out.println("构造器: "+c1);
            //单独获取,有参
            Constructor c2 = clazz.getDeclaredConstructor(int.class,int.class,String.class);
            System.out.println("构造器: "+c2);
            for(Constructor c:constructors){
                System.out.println("构造器: "+c);
            }   
            
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (SecurityException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }

    }
```

通过反射动态操作构造器,方法,属性

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import com.lorinda.bean.User;

/**
 * 通过反射动态操作构造器,方法,属性
 * @auther Matrix42
 */
public class ReflectionDemo03 {
  
    public static void main(String[] args) {
        
        String path = "com.lorinda.bean.User";
        
        try {
            Class clazz = Class.forName(path);
            
            //动态操作构造器
            User u = (User) clazz.newInstance();    //调用了User的无参构造方法
            
            Constructor<User> c = clazz.getConstructor(int.class,int.class,String.class);
            
            User u2 = c.newInstance(1000,20,"Matrix42");
            System.out.println(u2.getUname());
            
            //通过反射调用普通方法
            //好处:方法名,参数都可以是变量,可以从数据库读取
            User u3 = (User) clazz.newInstance();
            Method method = clazz.getDeclaredMethod("setUname", String.class);
            method.invoke(u3, "Matrix42");
            System.out.println(u3.getUname());
            
            //通过反射操作属性
            User u4 = (User) clazz.newInstance();
            Field f = clazz.getDeclaredField("uname");
            f.setAccessible(true);
            f.set(u4, "24xirtaM");  
            //默认会报错,添加f.setAccessible(true);关闭安全检查
            //can not access a member of class com.lorinda.bean.User with modifiers "private"
            System.out.println(u4.getUname());  //正常调用
            System.out.println(f.get(u4));      //通过反射调用  
            
        } catch (Exception e) {
            e.printStackTrace();
        } 
    
    }

}
```

反射机制性能问题

> 注意使用反射机制最大的问题就是性能问题，当你获得灵活性的时候也会牺牲你的性能，若大量使用反射则性能会大幅下降。

- setAccessible

- - 启用和禁用安全检查的开关,值为true则表示反射的对象在使用时应取消Java语言访问检查.值为fals则表示反射的对象应该实施Java语言访问检查.并不是为true就能访问,为false就不能访问

- 禁止安全检查,可以提高反射的运行速度

- 可以考虑使用:cglib/javasssist字节码操作

反射性能测试:

```java
import java.lang.reflect.Method;
import com.lorinda.bean.User;

public class ReflectionDemo04 {
    
    public static void test01(){
        
        User user = new User();
        
        long startTime = System.currentTimeMillis();
        
        for(int i=0;i<1000000000L;i++){
            user.getUname();
        }
        
        long endTime = System.currentTimeMillis();
        
        //421ms
        System.out.println("普通方法调用,执行10亿次,耗时:"+(endTime-startTime)+"ms");
       
    }
    
    public static void test02() throws Exception{
        
        User user = new User();
        Class clazz = user.getClass();
        Method m = clazz.getDeclaredMethod("getUname", null);
        
        long startTime = System.currentTimeMillis();
        
        for(int i=0;i<1000000000L;i++){
            m.invoke(user, null);
        }
        
        long endTime = System.currentTimeMillis();
        
        //1650ms
        System.out.println("反射动态调用,执行10亿次,耗时:"+(endTime-startTime)+"ms");
       
    }
    
    public static void test03() throws Exception{
        
        User user = new User();
        Class clazz = user.getClass();
        Method m = clazz.getDeclaredMethod("getUname", null);
        m.setAccessible(true);
        
        long startTime = System.currentTimeMillis();
        
        for(int i=0;i<1000000000L;i++){
            m.invoke(user, null);
        }
        
        long endTime = System.currentTimeMillis();
        
        //1153ms
        System.out.println("反射动态调用,跳过安全检查,执行10亿次,耗时:"+(endTime-startTime)+"ms");
       
    }

    public static void main(String[] args) throws Exception {
        
        test01();
        test02();
        test03();
        
    }

}
```

可以看出在java8中使用安全检查的反射耗时大约是普通调用的4倍,不使用安全检查是普通调用的2.5倍

反射操作泛型(Generic)

- Java采用**泛型擦除机制**来引入泛型.Java中泛型仅仅是给编译器javac使用的,**确保数据的安全性和免去强制类型转换的麻烦**.但是,一旦编译完成,所有**和泛型有关的类型全部擦除**.
- 为了通过反射操作这些类型以迎合实际开发的需要,Java就新增了ParameterizedType,GenericArrayType,TypeVariable和WildcardType几种类型来代表不能被归一到Class类中的类型但是又和原始类型齐名的类型.
- ParameterizedType:表示一种参数化类型,比如

```java
 Collection<String>
```

- GenericArrayType:表示一种元素类型是参数化类型或者类型变量的数组类型
- TypeVariable:是各种类型变量的公共父接口
- WildcardType:表示一种通配符类型表达式,比如?,? extends Number,? super Integer [wildcard就是通配符的意思]

通过反射读取泛型

```java
import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.List;
import java.util.Map;
import com.lorinda.bean.User;

/**
 * 通过反射读取泛型
 * @author Matrix42
 *
 */
public class ReflectionDemo05 {
    
    public void test01(Map<String, User> map,List<User> list){
        System.out.println("ReflectionDemo05.test02");
    }
    
    public Map<Integer, User>test02(){
        System.out.println("ReflectionDemo05.test2");
        return null;
    }

    public static void main(String[] args) {
        
        try {
            
            //获取指定方法参数泛型信息
            Method m = ReflectionDemo05.class.getMethod("test01", Map.class,List.class);
            Type[] t = m.getGenericParameterTypes();
            for(Type paramType:t){
                System.out.println("#"+paramType);
                if(paramType instanceof ParameterizedType){
                    Type[] genericTypes = ((ParameterizedType)paramType).getActualTypeArguments();
                    for(Type genericType:genericTypes){
                        System.out.println("泛型类型: "+genericType);
                    }
                }
            }
            /*
               #java.util.Map<java.lang.String, com.lorinda.bean.User>
                                            泛型类型: class java.lang.String
                                            泛型类型: class com.lorinda.bean.User
               #java.util.List<com.lorinda.bean.User>
                                            泛型类型: class com.lorinda.bean.User
             */
            //获得指定方法返回值泛型信息
            Method m2 = ReflectionDemo05.class.getMethod("test02", null);
            Type returnType = m2.getGenericReturnType();
            if(returnType instanceof ParameterizedType){
                Type[] genericTypes = ((ParameterizedType)returnType).getActualTypeArguments();
                for(Type genericType:genericTypes){
                    System.out.println("返回值,泛型类型: "+genericType);
                }
            }
            /*
                                       返回值,泛型类型: class java.lang.Integer
                                       返回值,泛型类型: class com.lorinda.bean.User
           */
               
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

}
```

反射操作注解

Student类:

```java
package com.lorinda.bean;

import com.demo.util.MField;
import com.demo.util.MTable;

@MTable("tb_student")
public class MStudent {

    @MField(columnName="id",type="int",length=10)
    private int id;
    @MField(columnName="sname",type="varchar",length=10)
    private String studentName;
    @MField(columnName="age",type="int",length=3)
    private int age;
    
    public MStudent(int id, String studentName, int age) {
        super();
        this.id = id;
        this.studentName = studentName;
        this.age = age;
    }
    
    public MStudent() {
    }

    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getStudentName() {
        return studentName;
    }
    public void setStudentName(String studentName) {
        this.studentName = studentName;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
      
}
```

Table注解:

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(value={ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface MTable {

    String value();
    
}
```

Field注解:

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(value={ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface MField {

    String columnName();
    String type();
    int length();
    
}
```

Demo06 通过反射读取注解

```java
import java.lang.annotation.Annotation;
import java.lang.reflect.Field;

public class ReflectionDemo06 {

    public static void main(String[] args) {

        try {
            Class clazz = Class.forName("com.lorinda.bean.MStudent");
            
            //获得类的所有有效注解
            Annotation[] annotations = clazz.getAnnotations();
            for(Annotation a:annotations){
                System.out.println(a);
            }
            
            //获得类的指定注解
            MTable table = (MTable) clazz.getAnnotation(MTable.class);
            System.out.println(table.value());
            
            //获得类的属性的注解
            Field f = clazz.getDeclaredField("studentName");
            MField field = f.getAnnotation(MField.class);
            System.out.println(field.columnName()+"--"+field.type()+"--"+field.length());
            
            //可以根据获得的表名,字段的信息,拼出DDL语句,然后使用JDBC执行这个SQL,在数据库中生成相关的表
            
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

}
```

### 2.动态字节码操作

- JAVA动态性的两种常见实现方式

  字节码操作

  反射

- 运行时操作字节码可以让我们实现如下功能

  动态生成新的类

  动态改变某个类的结构(添加/删除/修改 新的属性/方法)

- 优势

  比反射开销小,性能高

  JAVAasist性能高于反射,低于asm

常见的字节码操作类库

- BCEL

  Byte Code Engineering Library (BCEL), 这是Apache Software Foundation 的 Jakarta 项目的一部分.BCEL是Java classworking广泛使用的一种框,它可以让您深入JVM汇编语言进行类操作的细节.BCEL与Javassist有不同的处理字节码方法,BCEL在实际的JVM指令层次上进行操作(BCEI拥有丰富的JVM指令级支持)而Javassist所强调的是源代码级别的工作

- ASM

  是一个轻量级ava字节码操作框架,直接涉及量到VM底层的操作和指令

- CGLIB(Code Generation Library)

  是一个强大的,高性能,高质量的Code生成类库,基于ASM实现

- Javassist

  是一个开源的分析、编辑和创建Jaw字节码的类库.性能较ASM差,跟cglib差不多,但是使用简单.很多开源框架都在使用它

### 3.动态编译

- Java 6.0 引入了编译机制

- 动态编译的应用场景:

  - 可以做一个浏览器端编写java代码,上传服务器编译和运行的在线评测系统
  - 服务器动态加载某些类文件进行编译
    ​

- 动态编译的两种做法:

  - 通过Runtime调用javac,启动新的进程去操作(6.0之前,不是真正的动态编译)

    Runtime run = Runtime.getRuntime();

    Process process = run.exec("javac -cp d:/myjava/Helloworld.java")

  - 通过JavaCompiler动态编译
    ​

- 通过JavaCompiler动态编译

  JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();

  int result = compiler.run(null, null, null, "f:/HelloWorld.java");
  Parameters:
  in "standard" input; use System.in if null
  out "standard" output; use System.out if null
  err "standard" error; use System.err if null
  arguments arguments to pass to the tool

栗子:

```java
JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
int result = compiler.run(null, null, null, "f:/HelloWorld.java");
System.out.println(result==0?"编译成功":"编译失败");
```

2.动态运行编译好的类

- 通过Runtime.getRuntime()运行启动新的进程运行

```java
JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
int result = compiler.run(null, null, null, "f:/HelloWorld.java");
System.out.println(result==0?"编译成功":"编译失败");
Runtime run = Runtime.getRuntime();
Process process = run.exec("java -cp f: HelloWorld");
        
BufferedReader w = new BufferedReader(new InputStreamReader(process.getInputStream()));
System.out.println(w.readLine());
```

- 通过反射运行编译好的类

```java
import java.io.IOException;
import java.lang.reflect.Method;
import java.net.URL;
import java.net.URLClassLoader;
import javax.tools.JavaCompiler;
import javax.tools.ToolProvider;

public class DynamicCompile {

    public static void main(String[] args) throws IOException {

        JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
        int result = compiler.run(null, null, null, "f:/HelloWorld.java");
        System.out.println(result==0?"编译成功":"编译失败");
    
        try {
            URL[] urls = new URL[]{new URL("file:/"+"f:/")};
            URLClassLoader loader = new URLClassLoader(urls);
            Class<?> c = loader.loadClass("HelloWorld");
            Method m = c.getMethod("main", String[].class);
            m.invoke(null, (Object)new String[]{});//静态方法不用谢调用的对象
            //加Object强制转换的原因
            //由于可变参数是JDK5.0之后才有　m.invoke(null, new String[]{"23","34"});
            //编译器会把它编译成m.invoke(null,"23","34");的格式,会发生参数不匹配的问题
            //带数组的参数都这样做
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 4.执行其他脚本代码

1.脚本引擎执行JavaScript代码

- 脚本引擎介绍

  使得Java应用程序可以通过一套固定的接口与各种脚本引擎交互,从而达到在Java平台上调用各种脚本语言的目的

  Java脚本API是连通Java平台和脚本语言的桥梁

  可以吧一些复杂异变的业务逻辑交给脚本语言处理,这又大大提高了开发效率

- 获得脚本引擎对象

```java
ScriptEngineManager sem = new ScriptEngineManager();
        ScriptEngine engine = sem.getEngineByName("javascript");
```

- Java脚本API为开发者提供了如下功能:

  - 获取脚本程序输入,通过脚本引擎运行脚本并返回运行结果,

    这是最核心的接口

    注意是:接口 Java可以使用各种不同的实现,从而通用的调用js,groovy,python等脚本

    Rhino是一种使用Java语言写的JavaScript的开源实现,原先由Mozilla开发,现在被集成进入JDK6.0以及以上版本

  - 通过脚本引擎的运行上下文在脚本和Java平台间交换数据

  - 通过Java应用程序调用函数脚本

a.js

```javascript
function test(){
  var a = 3;
  var b = 4;
  print("invoke js file:"+(a+b));
}
test();
import java.io.FileReader;
import java.net.URL;
import javax.script.Invocable;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;

public class JavaScript {

  public static void main(String[] args) throws Exception {

  //获得脚本引擎对象
  ScriptEngineManager sem = new ScriptEngineManager();
  ScriptEngine engine = sem.getEngineByName("javascript");

  //定义变量,存储在引擎上下文中
  engine.put("msg", "I am a good man!");

  String str = "var user = {name:'Matrix42',age:18,schools:['清华大学','北京大学']};";

  str+= "print(user.name);";

  engine.eval(str);

  engine.eval("msg = 'Ha Ha';");

  System.out.println(engine.get("msg"));

  //定义函数
  engine.eval("function add(a,b){var sum = a + b; return sum;}");

  //取得调用接口
  Invocable jsInvocable = (Invocable) engine;

  //执行脚本中定义的方法
  Object result = jsInvocable.invokeFunction("add", new Object[]{13,20});

System.out.println(result);

//导入其他java包,使用其他包中的java类,如果想要深入了解可以学习Rhino
//查资料说jdk8好像不支持,jdk7支持
//- If you need JavaScript, use Java 7. 
//- If you need Java 8, don't use JavaScript. 
/*String jsCode = "importPackage(java.util);var list=Arrays.asList([\"北京大学\",\"清华大学\"]);";

engine.eval(jsCode);

List<String> list = (List<String>) engine.get("list");

for(String string:list){
    System.out.println(string);
}*/

//执行一个js文件(将js放到src下即可)
URL url = JavaScript.class.getClassLoader().getResource("a.js");
FileReader fr = new FileReader(url.getPath());
engine.eval(fr);
fr.close();
}

}
```
