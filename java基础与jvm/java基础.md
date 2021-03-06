## java基础类型

## java类加载过程

- 从字节码到class：类加载
- 链接
  - 验证：文件格式验证，元数据验证，字节码验证，符号引用验证
  - 准备：初始化静态数据
  - 解析：将符号引用变成直接引用
- 初始化：
  - 一般是等函数调用的时候才会有这个

## 双亲委派构造

- 类构造的时候首先将这个任务交由bootstrap (lib)，extention (lib/ext)，application classloader(本地classpath)完成，如果这些都无法完成，才会load当前类。
- 所谓用户自定义类加载器，其实就是用户自己写的类
- 好处在于
  - 不会重复加载重复的方法和变量
  - 安全一点，防止黑客通过写一个完全同名的类来覆盖现有的类，比如在当前目录下写了一个java.lang.String，如果不是双亲委派，当前目录下程序引用的可能就不是jdk中的String类了。

- 打破双亲委派的例子
  - 双亲委派构造要求：只要父类加载器中能加载，就一定从父类中加载。
  - tomcat为了实现jsp和webapp加载器的隔离，打破了这恶个要求



## 在什么时候会进行类的初始化

- 遇到new、getstatic、setstatic或者invokestatic这4个字节码指令时，对应的java代码场景为：new一个关键字或者一个实例化对象时、读取或设置一个静态字段时(final修饰、已在编译期把结果放入常量池的除外)、调用一个类的静态方法时。

- 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没进行初始化，需要先调用其初始化方法进行初始化。

- 当初始化一个类时，如果其父类还未进行初始化，会先触发其父类的初始化。

- 当虚拟机启动时，用户需要指定一个要执行的主类(包含main()方法的类)，虚拟机会先初始化这个类。

- 当使用JDK 1.7等动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化 








## 垃圾回收器

### 传统

- 标注-清理：标注为垃圾之后，回收器进行回收，容易产生碎片
- copy：每次只能使用一半的空间，在垃圾清理的时候，将非垃圾的放在一堆，垃圾的放在另一堆。
- 标注-压缩：标记完之后，将垃圾放在一堆，非垃圾放在一堆

### G1

- 划分为一个个区域eden，大对象，垃圾。
- eden:s1:s2是811，新生代1/3, 老年代 2/3 
- mixed-gc
- 什么时候full gc：期望停顿时间太小，回收的垃圾没有产生垃圾速度快





### OOM问题

- 

- 需要使用java排查工具？👴没有用过。



## 强软弱虚引用



## java注解

- 元注解
  - type
  - retention
  - document
  - 

