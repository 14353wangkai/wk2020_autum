### Spring中不同名词表示的java对象

- pojo（plain ordinary java object）：朴素Java对象，不受到任何规范约束，不能是继承类，实现类，也不能有任何注解（这一条不做强制要求），pojo用于数据库映射
- po：***persistent object*** 对象持久化之后的名称，一般一个po代表数据表中的一条记录
- bo：一组业务逻辑相关联的po会被封装为一个bo，对外提供接口
- javabean：满足以下要求的java对象（或者满足以下要求的pojo）
  - 所有field都是private
  - 所有field都会有getter和setter（封装性要求，一个数据更名之后，其他代码不用更改）
  - 拥有一个无参构造函数
  - 实现了serializable接口
  - 是一个公共类
- bean和javabean
  - bean泛指外部只了解程序的api，完全不了解内部情况的组件
  - Bean的编写规范包括Bean类的构造方法、定义属性和访问方法编写规则，能统一实现“将方法翻译为属性”这个操作，比如getter和setter就是典型的方法对应属性
- EJB：enterprise javabean，包含三种bean
  - session：每当客户端请求时，容器就会选择一个Session Bean来为客户端服务。Session Bean可以直接访问数据库，但更多时候，它会通过Entity Bean实现数据访问。 这个类一般用**单例模式来实现**，因为每次连接都需要用到它。
  - entity：Entity Bean是域模型对象，用于实现O/R（java对象和数据表记录）映射，负责将数据库中的表记录映射为内存中的Entity对象，事实上，创建一个Entity Bean对象相当于新建一条记录，删除一个 Entity Bean会同时从数据库中删除对应记录，修改一个Entity Bean时，容器会自动将Entity Bean的状态和数据库同步。  
  - message driven：
- DTO：数据传输对象，将PO的部分属性抽取出来，用于传输的对象，比如PO中可能有100个字端，但是一次RPC调用中，可能只有10个字端是需要传输的，此时DTO就是用于存储这些对象
- VO：value object，存活在业务层，目的在于提供一个便于存储业务数据的地方



## spring扩展

