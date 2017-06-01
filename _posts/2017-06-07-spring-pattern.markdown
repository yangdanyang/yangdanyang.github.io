---
layout:     post
title:      "设计模式总结之Spring中使用到的设计模式"
subtitle:   " \"Spring Design Pattern\""
date:       2017-06-07 
author:     "Yang"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - Design Pattern
---
## 工厂模式
### IOC容器
    Spring中IOC容器会以某种方式加载Configurtion Metadata(通常也就是XML格式的配置信
    息)，然后根据这些信息绑定整个系统的对象，最终组装成一个可用的基于轻量级容器的应用系统。
    这个阶段主要分为容器启动阶段和Bean实例化阶段。
    
### 容器启动阶段

    容器启动伊始，首先会通过某种途径加载XML格式的配置信息，容器需要依赖某些工具类
    (BeanDefinitionReader)对加载的XML格式的配置信息进行解析和分析，并将分析后的信息编
    组为相应的BeanDefinition,最后把这些保存了bean定义必要信息的BeanDefinition,注册到相应的
    BeanDefinitionRegistry,这样容器启动工作就完成了。总体来说，该阶段所做的工作可以认为是准
    备性的，重点更加侧重于对象管理信息的收集。当然，一些验证性或者辅助性的工作也可以在这个阶段完成。

### Bean实例化阶段
    经过第一阶段也就是容器准备阶段后，现在所有的bean定义信息都通过BeanDefinition的方式注册到了
    BeanDefinitionRegistry中。当某个请求方通过容器的getBean方法明确地请求某个对象，或者因依赖
    关系容器需要隐式地调用getBean方法时，就会触发第二阶段的活动。
    该阶段，容器会首先检查所请求的对象之前是否已经初始化。如果没有，则会根据注册的BeanDefinition
    所提供的信息实例化被请求对象，并为其注入依赖。如果该对象实现了某些回调接口，也会根据回调接口的要
    求来装配它。当该对象装配完毕之后，容器会立即将其返回请求方法使用。如果说第一阶段只是根据图纸装配
    生产线的话，那么第二阶段就是使用装配好的生产线来生产具体的产品了。
    
### 工厂模式的使用
    以上IOC容器动态生成对象的流程，而其中Bean的实例化阶段就使用到了工厂模式。每一个受管的对象，在
    对象中都会有一个BeanDefinition的实例(instance)与之相对应，该BeanDefinition的实例负责保存
    对象的所有必要信息，包括其对应的对象的class类型，是否是抽象类，构造方法参数以及其他属性等。当客
    户端向BeanFactory请求相应对象的时候，BeanFactory会通过这些信息并利用Java反射机制动态生成并
    为客户端返回一个完备可用的对象实例。

## 代理模式

### AOP基本概念
     Joinpoint：
        系统运行之前，AOP的功能都需要织入到OOP的功能模块中。要进行这种织入过程，我们需
     要知道在系统的哪些执行点上进行织入操作，这些将要在其之上进行织入操作的系统执行点
     就称为Joinpoint
     Pointcut：
       Pointcut概念代表的是JoinPoint的表述方式。我们使用自然语言声明了一Pointcut,
       该Pointcut指定了系统中符合条件的一组Joinpoint.
     Advice:
       Advice是单一横切关注点逻辑的载体，它代表将会织入到Joinpoint的横切逻辑。例如数
       据的校验，权限的管理

### AOP动态代理(代理模式应用):
       JDK1.3之后引入了一种称之为动态代理的机制，使用该机制，我们可以为指定的接口在系统运行期间动
    态地生成代理对象，从而帮助我们走出最初使用静态代理实现AOP的窘境(针对每一个目标对象，都要创
    建一个对应的代理类)。动态代理机制的实现主要由一个类和一个接口组成。当Proxy动态生成的代理对
    象上相应的接口方法被调用时，对应的InvocationHandler就会拦截相应的方法调用，并进行处理。
    InvocationHandler就是我们事实现横切逻辑的地方，它是横切逻辑的载体，作用跟Advice是一样的。
    默认情况下，如果Spring AOP发现目标对象实现了相应的interface，则采用动态代理机制为其生成代
    理对象实例。而如果目标对象没有实现任何Interface,Spring AOP会尝试使用一个称为CGLIB的开源
    的动态字节码生成类库，为目标对象生成动态的代理对象实例。
      整个过程的本质就是Spring根据配置文件，利用反射和目标对象实现所的接口创建了代理对象。然后将代
    理对象返回，与原对象进行替换，从而实现了动态代理。
