## maven

- 核心功能

  - 项目构建：清理-编译-测试-打包-发布
  - 依赖管理：解决版本管理之间的依赖关系
  - 项目发布

- 设计思想

  - 约定大于配置：动态的事物设计一个标准来统一管理
  - 面向对象设计

- 前辈构件工具：Ant

  - 需要配置源文件来源，编译之后的target文件夹
  - 从过程开始设计

- maven的一系列属性

  

- | groupId      | 项目创建团体或组织的唯一标志符，通常是域名倒写               |
  | ------------ | ------------------------------------------------------------ |
  | artifactId   | 项目名称                                                     |
  | packaging    | artifact打包的方式，如jar、war等等，默认为jar                |
  | version      | artifact的版本，例如0.0.1-SNAPSHOT                           |
  | name         | 表示项目的展现名，在maven生成的文档中使用                    |
  | url          | 表示项目的地址，在maven生成的文档中使用                      |
  | description  | 表示项目的描述，在maven生成的文档中使用                      |
  | dependencies | 表示依赖，在子节点dependency中添加具体依赖的groupId artifactId和version |
  | build        | 表示build配置，包括插件及执行目标等                          |
  | profile      | 多环境配置，为每个环境指定不同的配置加载路径                 |

  jar包和war包

  - jar包可以是方法包，也可以是包含main函数
  - war包需要配置http解析+main+j2ee规范，需要服务器

- maven
  - 下载：给出依赖即可
  - 上传：
    - snapshot
    - 稳定版本
- 具体实现
  - 默认类型
    - 打包成jar
    - 
  - 生命周期插件
    - clean：把原来编译的工程清理
    - default
    - site
  - 执行default中的生命周期回默认把前面的的插件都运行一次
  - 利用插件+目标的方式实现项目生命周期管理

- settings

  