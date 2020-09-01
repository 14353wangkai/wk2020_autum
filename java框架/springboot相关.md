## SpringBoot的常用注解

- springBootApplication：是@SpringBootConfiguration, @EnableConfiguration @ComponentScan 三个注解的组合，用于标志它是一个springBoot应用

  

- EnableAutoConfiguration

- Configuration

  - 用来代替applicationContext.xml配置文件， 所有这个**配置文件里面能做到的事情都可以通过这个注解所在类来进行注册**。

    

- SpringBootConfiguration

  - 用来修饰是Spring Boot配置，或者可以

- ComponentScan

  - 代替配置文件中的component-scan配置
  - 自动扫描包路径下用@Component注解修饰的bean

- Conditional

  - 用来标志一个spring bean或者configuration的配置文件

  



## 自动配置过程

- https://jingyan.baidu.com/album/fdbd4277a277edb89e3f48fa.html?picindex=4
- 加载主配置类，开启了自动配置功能@EnableAutoConfiguration
- @EnableAutoConfiguration 利用AutoConfigurationImportSelector给容器中导入一些组件。
- selectImport的函数如下

- 通过protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,    AnnotationAttributes attributes)获取候选的配置，这个是扫描所有jar包类路径下"META-INF/spring.factories"
  
  - *利用spring-boot-starter依赖的包在meta-inf那里都会有一个spring.factories文件
- 然后把扫描到的这些文件包装成Properties对象。
- 从properties中获取到EnableAutoConfiguration.class类名对应的值，然后把他们添加在容器中，因为enableautoconfig是一个注解，通过get类加载器的方式可以获取对应的值？
  
  - <img src="/Users/wangkai/Library/Application Support/typora-user-images/image-20200715081134914.png" alt="image-20200715081134914" style="zoom: 50%;" />
- 每个配置文件都有一个@configurationProperties对应里面的各个配置属性
  
  - 通过contitionalOnClass













## springApplicationLoader调用链

- ```java
  //初始化入口
  	public SpringApplication(Object... sources) {
  		initialize(sources);
  	}
  	
  	private void initialize(Object[] sources) {
  		if (sources != null && sources.length > 0) {
  			this.sources.addAll(Arrays.asList(sources));
  		}
  		this.webEnvironment = deduceWebEnvironment();
  		setInitializers((Collection) getSpringFactoriesInstances(
  				ApplicationContextInitializer.class));
  		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
        this.mainApplicationClass = deduceMainApplicationClass();
  	}
  ```

  

  - SpringApplication: 启动构造器，入参为 primarySources，就是ApplicationLoader.class，可以传入多个加载类，这里指分析一个
  - 利用deduceFromClasspath解析web应用类型，也就是应该启动什么样的web服务
    - 如果是handler：返回反应式栈（webflux reactive）
    - 如果是servlet：返回(webmvc servlet)
    - 如果是none：返回application context
  - 设置初始化器和监听器
  - 将启动类的加载器添加到main集合里

- 调用run方法

  - 加载上下文：

    - ```
      ConfigurableEnvironment environment = prepareEnvironment(listeners,
            applicationArguments);
      ```