## sql语法

- 筛选配合
  - join+on进行筛选
    - left join：保留左表的所有内容，跟右表进行匹配，没匹配的列值为null
    - right join：同理，保留右表
    - out join: 笛卡尔积
    - natural join: 将两个表的所有同名列进行，可以通过inner join实现
    
      



- join执行顺序

  - 相关语句，假设left_table有n行x列，right_table有m行y列

    ```sql
    SELECT <row_list>
     FROM <left_table>
       <inner|left|right> JOIN <right_table>
         ON <join condition>
           WHERE <where_condition>
    ```

    

  - 在from的时候，先将两个表进行笛卡尔积，得到一张(n*m) * (x+y)的表格，但是该表格不会持久化，假设叫做`vt1`

  - 在on的时候，根据 `join condition`对`vt1`进行筛选，然后将符合规范的数据插入到新的表`vt2`中

  - 在join的时候，对vt2中的数据进行添加，

