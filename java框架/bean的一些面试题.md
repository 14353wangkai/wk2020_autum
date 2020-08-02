## spring中Bean的作用域

**singleton：**

  Spring IoC容器中只会存在一个共享的Bean实例，无论有多少个Bean引用它，始终指向同一对象。Singleton作用域是Spring中的缺省作用域。

**prototype：**

  每次通过Spring容器获取prototype定义的bean时，容器都将创建一个新的Bean实例，每个Bean实例都有自己的属性和状态，而singleton全局只有一个对象。

**request：**

  在一次Http请求中，容器会返回该Bean的同一实例。而对不同的Http请求则会产生新的Bean，而且该bean仅在当前Http Request内有效。

**session：**

  在一次Http Session中，容器会返回该Bean的同一实例。而对不同的Session请求则会创建新的实例，该bean实例仅在当前Session内有效。

**global Session：**

  在一个全局的Http Session中，容器会返回该Bean的同一个实例，仅在使用portlet context时有效。

## bean的生命周期

- 实例化一个Bean，也就是我们通常说的new。
- 按照Spring上下文对实例化的Bean进行配置，也就是IOC注入。
- 如果这个Bean实现了BeanNameAware接口，会调用它实现的setBeanName(String beanId)方法，此处传递的是Spring配置文件中Bean的ID。
- 如果这个Bean实现了BeanFactoryAware接口，会调用它实现的setBeanFactory()，传递的是Spring工厂本身（可以用这个方法获取到其他Bean）。
- 如果这个Bean实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文。
  - 上下文是什么：上下文可以是xml文件
- 如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessBeforeInitialization(Object obj, String s)方法，BeanPostProcessor经常被用作是Bean内容的更改，并且由于这个是在Bean初始化结束时调用After方法，也可用于内存或缓存技术。
- 如果这个Bean在Spring配置文件中配置了init-method属性会自动调用其配置的初始化方法。
- 如果这个Bean关联了BeanPostProcessor接口，将会调用postAfterInitialization(Object obj, String s)方法。
- 当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean接口，会调用其实现的destroy方法。
- 最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。



## spring 的两个注入方式

两种注入方式为：**构造方法注入**和**设值**注入

1. 设值注入与传统的JavaBean的写法更相似，程序员更容易理解、接受，通过setter方式设定依赖关系显得更加直观、明显;
2. 对于复杂的依赖关系，如果采用构造注入，会导致构造器过于臃肿，难以阅读。Spring在创建Bean实例时，需要同时实例化其依赖的全部实例，因而会产生浪费。而使用设置注入，则避免这下问题;
3. **在某些属性可选的情况**下，多参数的构造器更加笨拙，官方更鼓励使用设值注入。
4. 构造注入可以在构造器中决定依赖关系的注入顺序，优先依赖的优先注入。
5. **对于依赖关系无须变化的Bean**，构造注入更有用处，因为没有setter方法，所有的依赖关系全部在构造器内设定，因此，不用担心后续代码对依赖关系的破坏。
6. 构造注入使依赖关系只能在构造器中设定，则只有组件的创建者才能改变组件的依赖关系。对组件的调用者而言，组件内部的依赖关系完全透明，更符合高内聚的原则。
7. 设值注入不会重写构造方法的值。如果我们对同一个变量同时使用了构造方法注入又使用了设置方法注入的话，那么构造方法将不能覆盖由设值方法注入的值。
8. 建议采用以设值注入为主，构造注入为辅的注入策略。对于依赖关系无须变化的注入，尽量采用构造注入;而其他的依赖关系的注入，则考虑采用set注入。

## spring循环依赖问题

- bean初始化的三个阶段
  - createBeanInstance：实例化，其实也就是调用对象的构造方法实例化对象
  - populateBean：填充属性，这一步主要是多bean的依赖属性进行填充
  - initializeBean：调用程序指定的init 方法。

- > 循环依赖就是N个类中循环嵌套引用，如果在日常开发中我们用new 对象的方式发生这种循环new的话，程序会在运行时一直循环调用，直至内存溢出报错。

- 三级缓存解决：dcl单例模式

  ```java
  protected Object getSingleton(String beanName, boolean allowEarlyReference) {
      Object singletonObject = this.singletonObjects.get(beanName);
      //isSingletonCurrentlyInCreation()判断当前单例bean是否正在创建中
      if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
          synchronized (this.singletonObjects) {
              singletonObject = this.earlySingletonObjects.get(beanName);
              //allowEarlyReference 是否允许从singletonFactories中通过getObject拿到对象
              if (singletonObject == null && allowEarlyReference) {
                  ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                  if (singletonFactory != null) {
                      singletonObject = singletonFactory.getObject();
                      //从singletonFactories中移除，并放入earlySingletonObjects中。
                      //其实也就是从三级缓存移动到了二级缓存
                      this.earlySingletonObjects.put(beanName, singletonObject);
                      this.singletonFactories.remove(beanName);
                  }
              }
          }
      }
      return (singletonObject != NULL_OBJECT ? singletonObject : null);
  }
  
  ```

- 关键实现：singletonFactory实现了ObjectFactory， doCreation的时候将beanname会放入bean工厂

  ```java
  
  public interface ObjectFactory<T> {
      T getObject() throws BeansException;
  }
  
  //这个接口在AbstractBeanFactory里实现，并在核心方法doCreateBean（）引用下面的方法:
  //这段代码发生在createBeanInstance之后，populateBean（）之前，也就是说单例对象此时已经被创建出来(调用了构造器)。这个对象已经被生产出来了，此时将这个对象提前曝光出来，让大家使用。
  
  protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
      Assert.notNull(singletonFactory, "Singleton factory must not be null");
      synchronized (this.singletonObjects) {
          if (!this.singletonObjects.containsKey(beanName)) {
              this.singletonFactories.put(beanName, singletonFactory);
              this.earlySingletonObjects.remove(beanName);
              this.registeredSingletons.add(beanName);
          }
      }
  }
  
  ```

  

### BeanFactory 和ApplicationContext的区别

- BeanFactory是Spring中最底层的接口，提供了最简单的容器的功能，只提供了实例化对象和拿对象的功能。而ApplicationContext是Spring的一个更高级的容器，提供了更多的有用的功能。 
- ApplicationContext提供的额外的功能：消息发送、响应机制、统一加载资源的功能**、强大的事件机制、对Web应用的支持**等等
- BeanFactory采用的是**延迟加载的形式来注入Bean**；
- ApplicationContext则相反的，**它是在Ioc启动时就一次性创建所有的Bean**,好处是**可以马上发现Spring配置文件中的错误**，坏处是造成浪费。

