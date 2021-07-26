### 示例表

```mysql
CREATE TABLE `employees` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(24) NOT NULL DEFAULT '' COMMENT '姓名',
  `age` int(11) NOT NULL DEFAULT '0' COMMENT '年龄',
  `position` varchar(20) NOT NULL DEFAULT '' COMMENT '职位',
  `hire_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间',
  PRIMARY KEY (`id`),
  KEY `idx_name_age_position` (`name`,`age`,`position`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='员工记录表';

INSERT INTO employees(name,age,position,hire_time) VALUES('LiLei',22,'manager',NOW());
INSERT INTO employees(name,age,position,hire_time) VALUES('HanMeimei', 23,'dev',NOW());
INSERT INTO employees(name,age,position,hire_time) VALUES('Lucy',23,'dev',NOW());

-- 插入一些示例数据
drop procedure if exists insert_emp; 
delimiter ;;
create procedure insert_emp()        
begin
  declare i int;                    
  set i=1;                          
  while(i<=100000)do                 
    insert into employees(name,age,position) values(CONCAT('zhuge',i),i,'dev');  
    set i=i+1;                       
  end while;
end;;
delimiter ;
call insert_emp();
```

### 优化案例

1.联合索引第一个字段走范围查找，便不会走索引，优化器内部分析出cost可能还不如all查找更高，因为要回表。

```mysql
EXPLAIN SELECT * FROM employees WHERE name > 'LiLei' AND age = 22 AND position ='manager';
```

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/8B6932DA0FCA46D0A473192F7832275B/98405](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/98405)

强制走索引，来检验实际情况下二者的查找效率。

```mysql
EXPLAIN SELECT * FROM employees force index(idx_name_age_position) WHERE name > 'LiLei' AND age = 22 AND position ='manager';
```

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/E9D3568E1D6C4D1EBC587DF5EF7E8612/98401](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/98401)

```mysql
-- 关闭查询缓存
set global query_cache_size=0;  
set global query_cache_type=0;
-- 执行时间0.333s
SELECT * FROM employees WHERE name > 'LiLei';
-- 执行时间0.444s
SELECT * FROM employees force index(idx_name_age_position) WHERE name > 'LiLei';
```

覆盖索引可以优化，使得执行器最终走了索引查找range。

```mysql
EXPLAIN SELECT name,age,position FROM employees WHERE name > 'LiLei' AND age = 22 AND position ='manager';
```

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/F19176D7224B46F18D4513EC68028072/98402](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/98402)

2.`in` 和 `where ...or`在数据量大的情况下走索引，小的情况下可能直接all。

```mysql
EXPLAIN SELECT * FROM employees WHERE name in ('LiLei','HanMeimei','Lucy') AND age = 22 AND position ='manager';
EXPLAIN SELECT * FROM employees WHERE (name = 'LiLei' or name = 'HanMeimei') AND age = 22 AND position ='manager';
```

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/4DE59AA7C5C24BC0AF31A4C38C894197/98403](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/98403)

此时如果表中仅有三条数据，则

![0](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/98406)

3.like kk%，一般会走索引

```mysql
EXPLAIN SELECT * FROM employees WHERE name like 'LiLei%' AND age = 22 AND position ='manager';
```

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/16D4DEA03F0D4012A060E25410DE4C75/98404](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/98404)

#### 索引下推

这里引入新的概念，索引下推ICP，一种优化手段。按照以往的思想，如果前列字段的条件导致后列字段失去有序性，则后续字段无法利用索引树，这在使用范围查找时（如`>`条件）是这样的，原因在于`ref_equal`需要比较的次数太多了，有可能还不如`all`效率高。而像`like kk%`这样的查找，优化器觉得范围相较`>`会小很多，因此就在给定的range范围内继续利用索引查找。

索引下推的过程为，在索引遍历过程中，对索引中包含的所有字段先做判断，过滤掉不符合条件的记录之后再回表，可以有效的减少回表次数。

### trace优化跟踪

利用trace工具可以分析优化器究竟如何计算cost，到最终选取的索引究竟为何。

```mysql
set session optimizer_trace="enabled=on",end_markers_in_json=on;  --开启trace
select * from employees where name > 'a' order by position;
SELECT * FROM information_schema.OPTIMIZER_TRACE;
```

```json
查看trace字段：
{
  "steps": [
    {
      "join_preparation": {    --第一阶段：SQL准备阶段，格式化sql
        "select#": 1,
        "steps": [
          {
            "expanded_query": "/* select#1 */ select `employees`.`id` AS `id`,`employees`.`name` AS `name`,`employees`.`age` AS `age`,`employees`.`position` AS `position`,`employees`.`hire_time` AS `hire_time` from `employees` where (`employees`.`name` > 'a') order by `employees`.`position`"
          }
        ] /* steps */
      } /* join_preparation */
    },
    {
      "join_optimization": {    --第二阶段：SQL优化阶段
        "select#": 1,
        "steps": [
          {
            "condition_processing": {    --条件处理
              "condition": "WHERE",
              "original_condition": "(`employees`.`name` > 'a')",
              "steps": [
                {
                  "transformation": "equality_propagation",
                  "resulting_condition": "(`employees`.`name` > 'a')"
                },
                {
                  "transformation": "constant_propagation",
                  "resulting_condition": "(`employees`.`name` > 'a')"
                },
                {
                  "transformation": "trivial_condition_removal",
                  "resulting_condition": "(`employees`.`name` > 'a')"
                }
              ] /* steps */
            } /* condition_processing */
          },
          {
            "substitute_generated_columns": {
            } /* substitute_generated_columns */
          },
          {
            "table_dependencies": [    --表依赖详情
              {
                "table": "`employees`",
                "row_may_be_null": false,
                "map_bit": 0,
                "depends_on_map_bits": [
                ] /* depends_on_map_bits */
              }
            ] /* table_dependencies */
          },
          {
            "ref_optimizer_key_uses": [
            ] /* ref_optimizer_key_uses */
          },
          {
            "rows_estimation": [    --预估表的访问成本
              {
                "table": "`employees`",
                "range_analysis": {
                  "table_scan": {     --全表扫描情况
                    "rows": 10123,    --扫描行数
                    "cost": 2054.7    --查询成本
                  } /* table_scan */,
                  "potential_range_indexes": [    --查询可能使用的索引
                    {
                      "index": "PRIMARY",    --主键索引
                      "usable": false,
                      "cause": "not_applicable"
                    },
                    {
                      "index": "idx_name_age_position",    --辅助索引
                      "usable": true,
                      "key_parts": [
                        "name",
                        "age",
                        "position",
                        "id"
                      ] /* key_parts */
                    }
                  ] /* potential_range_indexes */,
                  "setup_range_conditions": [
                  ] /* setup_range_conditions */,
                  "group_index_range": {
                    "chosen": false,
                    "cause": "not_group_by_or_distinct"
                  } /* group_index_range */,
                  "analyzing_range_alternatives": {    --分析各个索引使用成本
                    "range_scan_alternatives": [
                      {
                        "index": "idx_name_age_position",
                        "ranges": [
                          "a < name"      --索引使用范围
                        ] /* ranges */,
                        "index_dives_for_eq_ranges": true,
                        "rowid_ordered": false,    --使用该索引获取的记录是否按照主键排序
                        "using_mrr": false,
                        "index_only": false,       --是否使用覆盖索引
                        "rows": 5061,              --索引扫描行数
                        "cost": 6074.2,            --索引使用成本
                        "chosen": false,           --是否选择该索引
                        "cause": "cost"
                      }
                    ] /* range_scan_alternatives */,
                    "analyzing_roworder_intersect": {
                      "usable": false,
                      "cause": "too_few_roworder_scans"
                    } /* analyzing_roworder_intersect */
                  } /* analyzing_range_alternatives */
                } /* range_analysis */
              }
            ] /* rows_estimation */
          },
          {
            "considered_execution_plans": [
              {
                "plan_prefix": [
                ] /* plan_prefix */,
                "table": "`employees`",
                "best_access_path": {    --最优访问路径
                  "considered_access_paths": [   --最终选择的访问路径
                    {
                      "rows_to_scan": 10123,
                      "access_type": "scan",     --访问类型：为scan，全表扫描
                      "resulting_rows": 10123,
                      "cost": 2052.6,
                      "chosen": true,            --确定选择
                      "use_tmp_table": true
                    }
                  ] /* considered_access_paths */
                } /* best_access_path */,
                "condition_filtering_pct": 100,
                "rows_for_plan": 10123,
                "cost_for_plan": 2052.6,
                "sort_cost": 10123,
                "new_cost_for_plan": 12176,
                "chosen": true
              }
            ] /* considered_execution_plans */
          },
          {
            "attaching_conditions_to_tables": {
              "original_condition": "(`employees`.`name` > 'a')",
              "attached_conditions_computation": [
              ] /* attached_conditions_computation */,
              "attached_conditions_summary": [
                {
                  "table": "`employees`",
                  "attached": "(`employees`.`name` > 'a')"
                }
              ] /* attached_conditions_summary */
            } /* attaching_conditions_to_tables */
          },
          {
            "clause_processing": {
              "clause": "ORDER BY",
              "original_clause": "`employees`.`position`",
              "items": [
                {
                  "item": "`employees`.`position`"
                }
              ] /* items */,
              "resulting_clause_is_simple": true,
              "resulting_clause": "`employees`.`position`"
            } /* clause_processing */
          },
          {
            "reconsidering_access_paths_for_index_ordering": {
              "clause": "ORDER BY",
              "steps": [
              ] /* steps */,
              "index_order_summary": {
                "table": "`employees`",
                "index_provides_order": false,
                "order_direction": "undefined",
                "index": "unknown",
                "plan_changed": false
              } /* index_order_summary */
            } /* reconsidering_access_paths_for_index_ordering */
          },
          {
            "refine_plan": [
              {
                "table": "`employees`"
              }
            ] /* refine_plan */
          }
        ] /* steps */
      } /* join_optimization */
    },
    {
      "join_execution": {    --第三阶段：SQL执行阶段
        "select#": 1,
        "steps": [
        ] /* steps */
      } /* join_execution */
    }
  ] /* steps */
}

结论：全表扫描的成本低于索引扫描，所以mysql最终选择全表扫描

mysql> select * from employees where name > 'zzz' order by position;
mysql> SELECT * FROM information_schema.OPTIMIZER_TRACE;

查看trace字段可知索引扫描的成本低于全表扫描，所以mysql最终选择索引扫描

mysql> set session optimizer_trace="enabled=off";    --关闭trace
```

### order by与group by索引优化

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/63C5EA98994C4AD58CF23A257E801DAC/75983](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/75983)

首先可以看到，name索引是走了，另外extra也说明利用了age索引。

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/1E4EFE167A0E4C4BB9300FA187C6785C/75979](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/75979)

extra的`using filesort`说明利用了buffer sort内存进行数据排序。name部分索引只是帮助减小了导入的数据量。

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/6FAFF326F23D4CC089342E373EB5BFA4/75993](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/75993)

按照左前列原则，所有索引字段都用于了排序。

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/E5777614254F4078BAD3DAFE73A9C08F/76003](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/76003)

颠倒了顺序，无法从索引获取到排好序的数据，因此走了ref查询，将range范围内的数据load进内存，执行filesort。

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/D527776A026843BCA882D2168965B70F/76025](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/76025)

age为常量，相当于在指定name和age下根据position进行排序。

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/D2231D574F9A4F5CBFF48C9F7CE8BC6C/76038](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/76038)

部分字段降序，违背了索引有序性，因此使用filesort。MySQL8版本中，支持降序索引，使得这样的查询也可以走索引。

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/0799A27E2D06431293F4310B2867CD07/76049](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/76049)

两个range导致无序性。

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/5186BA058F4C43F1A3B6DB3A40E876A4/76074](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/76074)

通过覆盖索引，"强制"其走索引。

![https://note.youdao.com/yws/public/resource/d2e8a0ae8c9dc2a45c799b771a5899f6/xmlnote/04F67CC2719C41EC91C0D2A25BAB4CC7/76079](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/76079)

`using index`扫描索引本身（最多再回表一次）完成排序，`using filesort`将数据load内存中进行排序。使用覆盖索引可以减少回表。

`group by`与`order by`类似，也是需要先执行一次排序，再分组。**注意**，能使用where过滤的不要使用having。

### Filesort

- 单路排序：一次性取出所有字段，在sort buffer中按照某一条件，执行排序，排序完后的数据即结果。sort_mode信息里显示< sort_key, additional_fields >
- 双路排序：将排序字段和主键id取出来，执行排序，后再回表得到结果集。sort_mode信息里显示< sort_key, rowid >

MySQL 通过比较系统变量 `max_length_for_sort_data`(**默认1KB) 的大小和需要查询的字段总大小来判断使用哪种排序模式。

- 如果 字段的总长度小于max_length_for_sort_data ，那么使用 单路排序模式；
- 如果 字段的总长度大于max_length_for_sort_data ，那么使用 双路排序模式。

#### 单路排序

1. 从索引name找到第一个满足 name = ‘zhuge’ 条件的主键 id
2. 根据主键 id 取出整行，**取出所有字段的值，存入 sort_buffer 中**
3. 从索引name找到下一个满足 name = ‘zhuge’ 条件的主键 id
4. 重复步骤 2、3 直到不满足 name = ‘zhuge’ 
5. 对 sort_buffer 中的数据按照字段 position 进行排序
6. 返回结果给客户端

#### 双路排序

1. 从索引 name 找到第一个满足 name = ‘zhuge’  的主键id
2. 根据主键 id 取出整行，**把排序字段 position 和主键 id 这两个字段放到 sort buffer 中**
3. 从索引 name 取下一个满足 name = ‘zhuge’  记录的主键 id
4. 重复 3、4 直到不满足 name = ‘zhuge’ 
5. 对 sort_buffer 中的字段 position 和主键 id 按照字段 position 进行排序
6. 遍历排序好的 id 和字段 position，按照 id 的值**回到原表**中取出 所有字段的值返回给客户端

综上，单路排序是一种空间换时间的做法，如果sort buffer较大，则内存中排好序后所得数据就是所需结果，否则还需要回表。一般不通过改变sort buffer大小或者修改`max_length_for_sort_data`大小来改变filesort模式，因为会影响到其他因素。而是通过覆盖索引，选取少量的字段进入sort buffer中。

### **索引设计原则**

**1、代码先行，索引后上**

主体业务功能开发完毕，把涉及到该表相关sql都要拿出来分析之后再建立索引，譬如哪些sql会涉及到高并发，着重优化这些sql语句的查询时间。

**2、联合索引尽量覆盖条件**

比如可以设计一个或者两三个联合索引(尽量少建单值索引)，让每一个联合索引都尽量去包含sql语句里的where、order by、group by的字段，还要确保这些联合索引的字段顺序尽量满足sql查询的最左前缀原则。单值索引，一方面，如果建立的太多的话，占用空间，另一方面，单值索引筛选出来的范围仍然很大。

**3、不要在小基数字段上建立索引**

索引基数是指这个字段在表里总共有多少个不同的值，比如一张表总共100万行记录，其中有个性别字段，其值不是男就是女，那么该字段的基数就是2。

**4、长字符串我们可以采用前缀索引**

尽量对字段类型较小的列设计索引，比如说什么tinyint之类的，因为字段类型较小的话，占用磁盘空间也会比较小，此时你在搜索的时候性能也会比较好一点。

当然，这个所谓的字段类型小一点的列，也不是绝对的，很多时候你就是要针对varchar(255)这种字段建立索引，对于这种varchar(255)的大字段可能会比较占用磁盘空间，可以稍微优化下，比如针对这个字段的前20个字符建立索引，就是说，对这个字段里的每个值的前20个字符放在索引树里，类似于 KEY index(name(20),age,position)。

此时你在where条件里搜索的时候，如果是根据name字段来搜索，那么此时就会先到索引树里根据name字段的前20个字符去搜索，定位到之后前20个字符的前缀匹配的部分数据之后，再回到聚簇索引提取出来完整的name字段值进行比对。mysql内部人员分析出varchar类型的字段，前20个字符已经能够很好的区分了，换句话说，`index(name(20),age,position)`这种索引可以将范围缩小到很小。

但是假如你要是order by name，那么此时你的name因为在索引树里仅仅包含了前20个字符，所以这个排序是没法用上索引的， group by也是同理。所以这里大家要对前缀索引有一个了解。

**5、where与order by冲突时优先where**

在where和order by出现索引设计冲突时，到底是针对where去设计索引，还是针对order by设计索引？一般这种时候往往都是让where条件去使用索引来快速筛选出来一部分指定的数据，接着再进行排序。因为大多数情况基于索引进行where筛选往往可以最快速度筛选出你要的少部分数据，然后做排序的成本可能会小很多。

**6、基于[慢查询][1]做优化**

可以根据监控后台的一些慢sql，针对这些慢查询做特定的索引优化。

[1]: https://blog.csdn.net/qq_40884473/article/details/89455740	"慢查询"

### **索引设计实战**

以社交场景APP来举例，我们一般会去搜索一些好友，这里面就涉及到对用户信息的筛选，这里肯定就是对用户user表搜索了，这个表一般来说数据量会比较大，我们先不考虑分库分表的情况，比如，我们一般会筛选地区(省市)，性别，年龄，身高，爱好之类的，有的APP可能用户还有评分，比如用户的受欢迎程度评分，我们可能还会根据评分来排序等等。

对于后台程序来说除了过滤用户的各种条件，还需要分页之类的处理，可能会生成类似sql语句执行：

```mysql
select xx from user where xx=xx and xx=xx order by xx limit xx,xx
```

对于这种情况如何合理设计索引了，比如用户可能经常会根据省市优先筛选同城的用户，还有根据性别去筛选，那我们是否应该设计一个联合索引 `(province,city,sex)` 了？这些字段好像基数都不大，其实是应该的，因为这些字段查询太频繁了。

假设又有用户根据年龄范围去筛选了，比如 

```mysql
where  province=xx and city=xx and age>=xx and age<=xx
```

我们尝试着把age字段加入联合索引 `(province,city,sex,age)`，注意，一般这种范围查找的条件都要放在最后，之前讲过联合索引范围之后条件的是不能用索引的，但是对于当前这种情况依然用不到age这个索引字段，因为用户没有筛选sex字段，那怎么优化了？其实我们可以这么来优化下sql的写法：

```mysql
where  province=xx and city=xx and sex in ('female','male') and age>=xx and age<=xx
```

对于爱好之类的字段也可以类似sex字段处理，所以可以把爱好字段也加入索引 `(province,city,sex,hobby,age)` 

假设可能还有一个筛选条件，比如要筛选最近一周登录过的用户，一般大家肯定希望跟活跃用户交友了，这样能尽快收到反馈，对应后台sql可能是这样：

```mysql
where  province=xx and city=xx and sex in ('female','male') and age>=xx and age<=xx and latest_login_time>= xx
```

那我们是否能把 `latest_login_time` 字段也加入索引了？比如  `(province,city,sex,hobby,age,latest_login_time)` ，显然是不行的，那怎么来优化这种情况了？其实我们可以试着再设计一个字段`is_login_in_latest_7_days`，用户如果一周内有登录值就为1，否则为0，那么我们就可以把索引设计成 `(province,city,sex,hobby,is_login_in_latest_7_days,age)`  来满足上面那种场景了！

一般来说，通过这么一个多字段的索引是能够过滤掉绝大部分数据的，就保留小部分数据下来基于磁盘文件进行order by语句的排序，最后基于limit进行分页，那么一般性能还是比较高的。

不过有时可能用户会这么来查询，就查下受欢迎度较高的女性，比如

```mysql
where  sex = 'female'  order by score limit xx,xx
```

那么上面那个索引是很难用上的，不能把太多的字段以及太多的值都用 in 语句拼接到sql里的，那怎么办了？其实我们可以再设计一个辅助的联合索引，比如 `(sex,score)`，这样就能满足查询要求了。

以上就是给大家讲的一些索引设计的思路了，核心思想就是，尽量利用一两个复杂的多字段联合索引，抗下你80%以上的查询，然后用一两个辅助索引尽量抗下剩余的一些非典型查询，保证这种大数据量表的查询尽可能多的都能充分利用索引，这样就能保证你的查询速度和性能了！