## mybatis

- 开源的orm框架
- 抽象了jdbc代码，提供了一套api和数据库交互
- 优势
  - 创建数据源
  - 创建mapper
  - 直接调用，自动转换为需要的pojo
- zebra工具
  - 在entity和mapper中修改配置

- 当出现大量重复语句的时候，不是一次次轮循访问，而是考虑将多次查询语句缓存，



```xml
<update id="updateCorpusStatus">
    update corpus_data
    <trim prefix="set" suffixOverrides=",">
        <foreach collection="list" item="" index=""
    </trim>
     where task_id IN
    <foreach collection="dataList" index="dataId" open="(" separator="," close=")" item="dataId">
        #{dataId}
    </foreach>
</update>
    
```

