## spring中用到了什么设计模式

- spring IOC
  - 监听器：观察者模式
  - 创建：朴素工厂模式
  - 单例：单例模式
- spring AOP
  - 代理模式
    - 动态代理：动态字节码技术
      - cglib：在接口没有实现类的时候，通过操作类字节码实现代理。
      - jdk：通过接口方式实现代理。proxy.newInstance()

- SpringMVC
  - AOP的应用之一
- 