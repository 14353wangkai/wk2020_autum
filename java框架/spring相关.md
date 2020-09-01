## bean是什么

- 一个规范，满足以下四个原则的class都叫做bean
  - 成员变量都是private
  - 每个变量都有getter/setter
  - 有无参构造函数
  - 实现序列化(`serializable`)接口，便于从内存到硬盘的持久化。

## 传统对象控制 vs 控制反转容器

- 传统（放在一堆）：一个对象所属的类如果组合了大量的其他类对象，相当于每两个类之间都有一条线联系着。

  - 比如依赖的一个类的路径发生了改变，或者其中某个方法更改了名字，那么所有与这个改变类相关

   

- ioc：通过bean约束+上下文反射+工厂模式，做到耦合尽量低。

  - IOC像spring提供的一个**统筹根节点**，通过这个根来管理每一个bean的创建和实现。

  - 通过配置的方式，将依赖上下文告诉ioc，比如依赖对象的类名等信息，ioc会以map的形式，将bean类和上下文信息关联，因此bean类一定要有一个唯一标识，比如xml的id，如果用注解，那么就一定要在包扫描的时候，能够唯一确定对应的类。

  - 在加载一个bean实例的时候，首先加载上下文

    - 如果是xml，ioc会根据beanid找到对应的上下文
    - 如果是注解，ioc会扫描找到`component`,`service`,`controller`,`component`标注的类，通过byname（匹配类的名字）或者bytype（匹配所属祖先类）对应类的上下文

  - 此时我们知道要加载的bean类的类名，我们知道，通过类名，反射机制可以通过`getconstructor`来获取构造器，也可以通过`Method[] methods = getmethods`来获取方法名，然后通过`method.invoke()`来调用对应方法。

  - 回顾bean的约束，每个私有变量都拥有对应的setter，因此利用反射，我们直接可以对类中的所有变量进行注入

    

- 总结：

  - 我们可以看到，在ioc注入的过程中，唯一的变量就是上下文信息，而这些信息，要么可以通过修改xml做到通知所有相关bean，要么是注解的方式，什么都不用改（注解一般也会对应一个map）
  - 这得益于反射的优越性，因为反射提供的getName，getConstructor, getMethod这些方法具有极强的泛化能力，只需要类加载器，就能获取所有的类方法



## 面向切面编程 AOP

- spring还根据“对修改关闭，对扩展开放的”AOP机制

- AOP的实现，是为了解决：“相同类型的组件，除了共同的核心功能以外，还会有不同的额外功能”，比如**日志、事务管理和安全这样的系统服务**经常会作为增强功能融入到一些现有的应用中

- 为了防止代码冗余和逻辑混乱（核心逻辑被增强功能淹没），AOP将增强服务理解为一个无限大的“切面”，如果有什么类用到了这个增强，那么就会和这个切面相交

- xml里面aop的写法，原理就是给动态代理的method.invoke方法

- 注解也是差不多的道理

  - @Aspect

    

  - 切点声明

    - @Pointcut("execution(* com.work.service.*.*(..))")
      - 代表一种excution类型的pointcut，声明AOP的拦截包 

    ```java
    @Pointcut("execution(* com.work.service.*.*(..))")
    public void Pointcut(){
      System.out.println("声明切入点");
    }
    ```

  - 增强类型

    - @Before

    - @After

    - @Around

      ``` 
      @Around("pointcut()")
      public Object around(ProceedingJointPoint point) throws Throwable{
        long start = System.CurrentTimeMillis();
        Object result = point.proceed();
        long end = System.CurrentTimeMillis();
        System.out.println(end - start);
        return result;
      }
      ```

      

    - Throws

    - Introduction

  

  ​    @After("Pointcut()")
  ​    public void After(){
  ​        System.out.println("后置通知：方法之后执行");
  ​    }
  ​    @Before("Pointcut()")
  ​    public void Before(){
  ​        System.out.println("前置通知：方法之前执行");
  ​    }
  ​    @AfterReturning("Pointcut()")
  ​    public void AfterReturning(){
  ​        System.out.println("后置通知：方法之后执行ing");
  ​    }  

  - 动态代理
    - 默认：实现了接口，就用jdk.proxy，没有实现，就用cglib
    - 

## spring bean管理　

　以<bean id="airplane" class="spring.Airplane"/>为例，spring在启动的时候，会创建应用上下文容器（如xml），而所有的bean都是在创建应用上下文容器的时候进行加载的，大致流程就是，应用上下文对象会根据我们传入的配置文件路径去加载这个配置文件，然后解析配置文件的<beans>标签下的<bean>标签，然后会对每个bean标签进行解析，这时会根据我们在bean标签中配置的属性（这里我们只定义了id和class）给每一个bean实例化一个BeanDefinition，同时会把这些BeanDefinition对象放入到应用上下文中的一个List<BeanDefinition>集合中，接着就是对List<BeanDefinition>进行循环并且通过class的值通过反射，实例化bean，最后将实例化的bean维护到一个map中，map的key就是bean的id，map的value就是bean的实例化对象，最后我们就可以通过id来获取我们想要的bean了，但是这里只是简单的介绍了bean的加载，应用上下文所做的事情远不止这些，还有对懒加载bean的维护，对bean之间依赖关系的维护（就是我们常说的依赖关系，其实也是通过一个Map<String, Set<String>>类型ConcurrentHashMap来维护的）等等。



## spring事务的四种实现

- 编程式事务管理：需要手动编写代码，在实际开发中很少使用
- 声明式事务管理：
- 基于TransactionProxyFactoryBean的方式，需要为每个进行事务管理的类做相应配置
- 基于AspectJ的XML方式，不需要改动类，在XML文件中配置好即可

## 事务传播模型





