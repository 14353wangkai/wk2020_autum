## JMM和JVM内存模型的区别

- jvm内存模型其实就是操作系统的内存模型
- jmm的内存模型指的是java堆栈，方法区，程序计数器这些java安排的逻辑区域

## java 内存管理

- 堆
  - 几乎所有实例对象都是在堆里面分配的
  - java中，堆的逻辑内存也是连续分配的
- 栈
  - 所有局部变量，方法调用都是在栈中完成的
  - 栈空间无疑都是连续分配的，既然如此，为什么要分配堆和栈？
    - 堆中的内存是通过GC回收才能释放的，由于实例对象回收的时候不一定是连续的，因此堆中可能会有一个个“窟窿”
    - 栈只能通过弹出的形式来释放内存
  - 本地方法栈
    - native方法的栈
  - 虚拟机栈
    - java方法执行时候进行的压栈
    - 包含局部变量表
    - 操作数
    - 动态链接
    - 方法出口
- 程序计数器
  - 压栈，弹栈，函数跳转，分支的实现
- 方法区
  - 对象的哈希码
  - 和永久代并不是等价的，永久代是方法区的一个实现方式
  - 通过`-XX:MaxPermSize`设置上
  - 元空间代替永久代：保留类型信息，将常量池，静态变量移出
- 给不同线程分配对象的时候，可能会在TLAB上先分配一段，当TLAB不够分配的时候，才会向堆中分配
- 在TLAB中使用指针碰撞来分配内存，在堆中分配用



## 各类内存问题

### 溢出

- 方法区溢出
- 栈溢出
- 堆溢出

### 泄漏

- 有不该被引用的对象还在被引用

​	

## jvm参数

- `-Xms,-Xmx`:
- `-XX:PermSize=256m -XX:MaxPermSize=256m`
- `XX:NewSize=10144m -XX:MaxNewSize=10144m -XX:SurvivorRatio=10`

## java 元空间

- 相比于永久代，移除了常量池和静态变量。	