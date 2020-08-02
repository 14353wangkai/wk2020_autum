## MySQL和MyBatis

### 数据库部署原则

- 单实例，多数据库结构
  - 所有数据库共享物理资源
  - 连接数
- 主从原则
  
  - 1主多从，故障的时候，从数据库变为主数据库
- 多机房原则
  
  - 防止自然灾难
  
  

### mysql特点

- 关系型数据库
- 一致性
  - mvcc
  - 事物（acid）
  - b+tree 索引：支持范围索引
  - 自适应哈希缓存
- 从库和主库是有时间差的
  - 在业务要求主从数据库完全一致的时候，要设计屏障逻辑
- 文档，图片，音频视频等不适合存储在数据库中，适合存储在云盘中

### 结构化与非结构化存储

- 结构化存储：读多写少，对数据一致性要求高
- key-value存储：配合结构化数据，如缓存，分布式
- elasticSearch：倒排索引查询文档（索引的反义词，倒排索引就是通过文档内的单词为索引，单词所在文档为value），复杂条件查场景
- hbase：列存储，key查询，水平扩容，im消息



### 最佳实践

- mysql读写分离
  - 采用主从结构，读qps可以水平扩容
  - 单表数据体量过大的时候需要进行分表甚至分库
- mysql+redis
  - 热点数据
- mysql+es
  - 将mysql的一些数据同步到es中，在复杂条件查询的时候调用es



### 命名规范

- 英文单词小写，多个单词利用下划线划分
- 可以缩写，但是缩写一定要是有共识的
- 不能用拼音

### 索引命名规范

- pk开头代表主见等



### 库设计

- 存储引擎：有限innodb
- 字符集：有限使用utf8??4，防止emoji信息丢失

### 表设计

- 一致性
- 不要用TEXT，BLOB类型，避免不了责需要拆表
- 大字段和冷门数据需要进行分表
- 字段越小越好，避免null列，提高索引效率



### 索引规范

- 避免使用临时表
- 有利于分组和排序+范围索引

## 索引原理

- 等值查询：hash
- 范围查询：b+
- 排序：b+



## 建立索引原则

- 索引列区分度高，不要以性别之类的属性来区分
  - SELECT COUNT(distinct column_name)/count(*) from table_name
- 整体索引占用空间小
- 前缀索引
  - 长类型：TEXT等
  - 前缀区分度高
  - Select count(distinct left(column_name, prefix_lenght)/count(*) from table_name)
- 组合作引：最左匹配原则

- 索引覆盖：查找的数据回生成一个索引，防止回表
- 化繁为简
  - 不建议两张表以上的join
  - 减少锁的竞争
- 事物简单，拒绝大sql，大事物，大批量
  - 拒绝嵌套rpc
  - 大事务
- 不要用 select *，减少io
- 避免触发器，函数，存储过程的使用，由应用代码来完成这些逻辑，降低耦合
- limit高效分页
  - 不建议 limit offset, size
  - 建议 id> offset, limit size, 减少数据查询
- like避免前置%



## expain 

- 慢查询分析工具

- 常用于看select，参数

  - select_type：是联合查询，子查询还是普通产讯

  - type：关键的关键字

    - const：通过一次索引能找到
- eq_ref唯一性索引扫描，对于每个索引key，只有一个记录匹配
    - ref：非唯一索引扫描，返回匹配某个关键字的所有行
- range：范围查询，常用于in，>, <, between
    - Index：索引无用
- all：全表扫描

- Partition：用到了联合索引
- passible_key
- Filtered: 查找出来的数据经过server筛选之后剩下多少

- 新版本支持update分析

- 出现的问题

  - 通过order by导致全表查询
  - 大事务导致的case
    - 事务嵌套耗时网络请求导致数据库更改操作超时
  - 主从延迟导致
    - 对延迟敏感的业务

## 数据库

- 分表
  - 分表策略
    - 水平分：仍然保持数据表结构，但是将一部分数据放入到另一个表中。
    - 垂直分：将某些属性抽取出来，形成新的表
- 分库
- 读写分离
  - 有主数据库和从数据库
  - 写(update, insert, delete)请求是访问主数据库，读请求是访问从数据库
  - 从数据库会有操作日志，在主数据库完成操作的一段时间之后，从数据库才会进行更新
- 事务
  - ACID：原子（undo日志：出错回滚，临界区只有一个线程），一致（操作前后保持逻辑一致），隔离（保证多事务并行的结果和串行的结果是一样的），持续（redo 日志）
  
  - 如何保证事务持久性
    - redo log：一些事物完成之后，写回os buffer，在os buffer中再往硬盘中写入
    - undo log：给事务一致性提供保障的日志
    
  - 为什么要有事务
    
    - 保持数据的一致性，方便回滚
    
  - CAP原理
    - 一致性，可用性，分区容错性，只能三选二
    - 分区容错基本是必要的，因此一般是二选一，对于银行，商城这样的，是宁愿可用性低，也要保留一致性，对于实时性高的应用场景（想不起来），可用性（只要保障每次请求都会有返回，但是并不需要保障是最新的数据）又是必要的，一致性到可以忽略
    
  - 四大隔离等级：读未提交，读已提交，重复读（mvcc，数据后面接版本号），序列化（加锁）
  
  - 快照读和当前读
  
    - 对于需要修改数据库的操作而言，读取数据都是“当前读”
      - MVCC+next-key locks：next-key locks由record locks(索引加锁/行锁) 和 gap locks(间隙锁，每次锁住的不光是需要使用的数据，还会锁住这些数据附近的数据)的结合，next-key lock 会锁定范围和自身行，比如select...where id<6，锁定的是小于6的行和等于6的行 
    - 对于普通的查询而言，读取数据都是“快照读”，禁止更新类似cas，更新完发现和当前版本号不一致，那么在事务提交之后，该
      - 查询时需要同时满足以下两个条件
        　　1、查找数据版本号，早于（小于等于）当前事务id的数据行。 这样可以确保事务读取的数据是事务之前已经存在的。或者是当前事务插入或修改的。
          　　2、查找**删除大于当前事务版本号的记录**。 这样确保取出来的数据在当前事务开启之前没有被删除。
  
  - 不同隔离等级导致的读问题：
    - 脏读：发生在读未提交，也就是事务1更改某数据之后尚未提交，事务2就可以对数据进行读取，此时事务1发生错误，进行回滚，又将数据改回去，导致事务2读取到无效数据。
      - 问题场景：银行取钱，A给银行卡X打了100元，B读取发现银行卡多了100，然后出现故障，A事务尚未提交就进行了回滚，那么B实际上没有多钱。
    - 不可重复读：事务操作的时候，对同一个索引的数据读取出不同的结果，出现在事务1对某数据修改完进行提交之后，事务2读取该数据，然后事务1又对数据进行第二次修改，当事务2对该数据再次读取的时候，就会
    - 幻读：快照读可以解决幻读，当前读必须建立间隙锁来锁住对应位置的内容。
    
    
    
  - 注解：@Transactional
  
  - spring中的事务



- 什么时候不匹配索引
  - 最左前缀不满足
  - 使用函数
  - 用or连接
- 









```c++
#include<bits/stdc++.h>
using namespace std;


void backtrack(vector<int>& res, vector<int>& output, int first, int len){
    if (first == len) {
        int tmp = 0;
        for(int i = 0;i < output.size();i++){
            int num = output[i];
            tmp = tmp*10 + num;
        } 
        res.push_back(tmp);
        return;
    }   
    for (int i = first; i < len; ++i) {
        swap(output[i], output[first]);
        backtrack(res, output, first + 1, len);
        swap(output[i], output[first]);
    }
}
void permute(vector<int>& nums) {
    vector<int> res;
    backtrack(res, nums, 0, (int)nums.size());
    int cnt = 0;
    for(int i = 0;i < res.size();i++){
        int tmp = res[i];
        if(tmp % 7 == 0)
            cnt++;
    }
    cout << cnt;
}
int main() {
    string s;
    cin >> s;
    vector<int> nums;
    for(int i = 0;i < s.length();i++){
        if(s[i] <= '9' && s[i] >= '0')
            nums.push_back(s[i]-'0');
    } 
    permute(nums);
	return 0;
}
```





```java
.addFormDataPart("taskId", taskId)
                .addFormDataPart("xmlPath", xmlPath)
                .addFormDataPart("modelRoot", modelRoot)
                .addFormDataPart("filterNum", filterNum)
                .addFormDataPart("notagHDFSRoot", notagHDFSRoot)
                .addFormDataPart("outputHDFSRoot", outputHDFSRoot)
                .addFormDataPart("notagFile","notagFile",
```

